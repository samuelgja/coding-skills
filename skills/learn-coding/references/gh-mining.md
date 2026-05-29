# gh / git mining cookbook

Copy-pasteable commands for `learn-coding`. Replace `{owner}/{repo}` only if `gh` can't resolve it from the working directory; `{n}` / `N` are PR numbers you substitute. All commands are **read-only**.

Verify your budget first — a full mine can be hundreds of API calls:

```bash
gh api rate_limit --jq '.resources.core | {remaining, limit, reset}'
```

---

## 0. Resolve the repo

```bash
gh repo view --json nameWithOwner,defaultBranchRef,visibility
# default branch only (for tree/log refs):
gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name'
```

---

## 1. Authority list — who to weight

```bash
# CODEOWNERS (try each location; contents API 404s if missing).
# The raw media type returns file contents directly — no base64 decode, fully portable.
gh api repos/{owner}/{repo}/contents/.github/CODEOWNERS -H "Accept: application/vnd.github.raw"
gh api repos/{owner}/{repo}/contents/CODEOWNERS         -H "Accept: application/vnd.github.raw"
gh api repos/{owner}/{repo}/contents/docs/CODEOWNERS    -H "Accept: application/vnd.github.raw"

# Collaborators with their permission level (maintainers = admin/write)
gh api repos/{owner}/{repo}/collaborators --jq '.[] | {login, role: .role_name}'

# Top contributors (maintainer proxy), most commits first
gh api --paginate --slurp repos/{owner}/{repo}/contributors \
  | jq '[.[][]] | sort_by(-.contributions) | .[] | {login, contributions}'
```

`author_association` enum (used in `select(...)` below): `OWNER`, `MEMBER`, `COLLABORATOR`, `CONTRIBUTOR`, `FIRST_TIMER`, `FIRST_TIME_CONTRIBUTOR`, `MANNEQUIN`, `NONE`.

---

## 2. Pass 1 — declared rules (config + docs)

```bash
# One recursive tree call → list convention-bearing files
gh api repos/{owner}/{repo}/git/trees/HEAD?recursive=1 --jq '.tree[].path
  | select(test("(^|/)(\\.eslintrc|eslint\\.config|\\.prettierrc|prettier\\.config|biome\\.json|\\.editorconfig|\\.rubocop\\.yml|ruff\\.toml|pyproject\\.toml|\\.flake8|\\.golangci|tsconfig\\.json|\\.stylelintrc|CONTRIBUTING\\.md)$"))'

# Fetch a config/doc as raw text (raw media type → file contents directly, no base64)
gh api repos/{owner}/{repo}/contents/CONTRIBUTING.md -H "Accept: application/vnd.github.raw"
gh api repos/{owner}/{repo}/contents/.editorconfig   -H "Accept: application/vnd.github.raw"

# Commit-message convention from history (local clone)
git log -n 200 --pretty=format:'%s'          # subjects only
git log -n 100 --pretty=format:'%s%n%b%n--'  # subjects + bodies

# Commit tooling that enforces a format
gh api repos/{owner}/{repo}/git/trees/HEAD?recursive=1 --jq '.tree[].path
  | select(test("commitlint|\\.gitmessage|\\.husky|\\.commitlintrc"))'

# Required CI checks (lint/test gates)
gh api repos/{owner}/{repo}/contents/.github/workflows --jq '.[].name'   # directory → JSON array (no base64)
gh api repos/{owner}/{repo}/contents/.github/workflows/<file> -H "Accept: application/vnd.github.raw"
```

---

## 3. Pass 2 — enforced rules (PR review comments)

The three surfaces are **not** interchangeable:

| Surface | Endpoint | Signal |
|---------|----------|--------|
| Inline code-line | `/pulls/{n}/comments` | file/line/diff-anchored nitpicks & fixes |
| Review verdict | `/pulls/{n}/reviews` | `CHANGES_REQUESTED` reasons, approvals |
| PR conversation | `/issues/{n}/comments` | general discussion |

### List PRs to mine — both open AND closed/merged

Mine **every** state. Open PRs reveal the team's *current* standards (what reviewers push for right now); closed/merged PRs reveal the *settled* conventions (what got enforced before merge).

```bash
gh pr list --state open   --limit 100 --json number,title,author,createdAt,labels,reviewDecision,url    # OPEN: live, in-flight reviews (current standards)
gh pr list --state merged --limit 200 --json number,title,author,mergedAt,baseRefName,reviewDecision,url # MERGED: settled conventions (enforced before merge)
gh pr list --state closed --limit 100 --json number,state,title,author,closedAt,labels,url              # CLOSED, unmerged only: rejected / abandoned work
gh pr list --state all    --limit 300 --json number,state,title,author,mergedAt,closedAt,labels          # everything in one pass
```

In `gh`, `open` / `closed` / `merged` are **mutually exclusive** states (GraphQL `PullRequestState`). `--state closed` returns only PRs closed **without** merging — it does **not** include merged PRs; use `--state merged` for those, or `--state all` for everything. To capture both current and settled standards, mine `open` + `merged` (add `closed` to see what gets rejected).

### Repo-wide review comments (preferred — one corpus, fewer calls)

The repo-wide endpoints below return review comments from PRs of **every state** (open + closed + merged) in one paginated pass — no need to loop per PR or per state.

```bash
# Team-only filter — reuse in EVERY comment query. Keeps OWNER/MEMBER/COLLABORATOR;
# drops bots ([bot] login or type "Bot": Copilot, dependabot, github-actions) and outside commenters.
TEAM='select((.author_association=="OWNER" or .author_association=="MEMBER" or .author_association=="COLLABORATOR") and (.user.type!="Bot") and ((.user.login|endswith("[bot]"))|not))'

# ALL inline review comments across the repo (any PR state), team only, most RECENT first
gh api --paginate "repos/{owner}/{repo}/pulls/comments?per_page=100&sort=created&direction=desc" \
  --jq ".[] | $TEAM | {user: .user.login, assoc: .author_association, path, line, body, url: .html_url, created_at}"
```

Read **many** comments and bias toward **recent** ones (the `sort=created&direction=desc` above) — recent closed/merged PRs reflect the team's current standard. Drop self-comments (PR author on own PR) when you have the author handle.

### Per-PR (when you need verdicts/context for a specific PR)

```bash
# (uses $TEAM from the block above)
gh api repos/{owner}/{repo}/pulls/{n}/comments --jq ".[] | $TEAM | {user: .user.login, assoc: .author_association, path, line, diff_hunk, body, created_at}"
gh api repos/{owner}/{repo}/pulls/{n}/reviews  --jq ".[] | $TEAM | {user: .user.login, state, body, submitted_at}"
gh api repos/{owner}/{repo}/issues/{n}/comments --jq ".[] | $TEAM | {user: .user.login, assoc: .author_association, body, created_at}"

# Or via gh pr view (auto-resolves repo from cwd)
gh pr view N --json reviews,comments,number,title,author
```

---

## 4. Find the signal — rank by frequency

```bash
# (set $TEAM from §3 first) Convention hotspots: files TEAM reviewers flag most
gh api --paginate repos/{owner}/{repo}/pulls/comments --jq ".[] | $TEAM | .path" | sort | uniq -c | sort -rn | head -20

# Standard-setters: TEAM reviewers by comment volume
gh api --paginate repos/{owner}/{repo}/pulls/comments --jq ".[] | $TEAM | .user.login" | sort | uniq -c | sort -rn | head -20
```

Then cluster the comment bodies by theme (naming, tests, error handling, structure, performance, security) and keep themes that recur or come from the authority list.

---

## 5. Pagination & rate limits

- `--paginate` follows `Link` headers across all pages.
- `--paginate --slurp` merges pages into **one** JSON array (avoids concatenated arrays that break `jq`); post-process with `jq '[.[][]]'` when slurping arrays-of-arrays.
- Re-check `gh api rate_limit` mid-run on big repos; back off if `remaining` is low.

## Field reference

- `/pulls/{n}/comments` returns: `user.login`, `author_association`, `body`, `path`, `line`, `side`, `diff_hunk`, `position`, `in_reply_to_id`, `created_at`, `html_url`, `pull_request_review_id`.
- `/pulls/{n}/reviews` returns: `id`, `user.login`, `body`, `state` (`APPROVED`/`CHANGES_REQUESTED`/`COMMENTED`), `author_association`, `submitted_at`, `commit_id`, `html_url`.
- `gh pr list --state` accepts: `open`, `closed`, `merged`, `all` (`merged` ≠ `closed`).

## No-`gh` / API-only fallbacks

- No local clone for `git log`? Pull subjects via API: `gh api --paginate repos/{owner}/{repo}/commits --jq '.[].commit.message'` (heavier, paginated).
- No `gh` auth at all? Run Pass 1 against the **local** working tree (read config files + `git log` directly) and tell the user PR mining was skipped.
