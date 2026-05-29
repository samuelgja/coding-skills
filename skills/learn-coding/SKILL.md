---
name: learn-coding
description: Use when onboarding to an unfamiliar repo, or when reviewers keep flagging the same things on your pull requests. Learns a team's real coding conventions from the repo's code, config, and PR review comments (weighting owners and maintainers) and writes them to .coding/. Re-run anytime to refresh.
---

# learn-coding

## Overview

A team's real coding standard is mostly **undocumented**. Linters catch the mechanical part. The rest — naming taste, structure, what counts as over-engineered, which patterns get rejected — lives in **pull-request review comments**, where reviewers say _"extract this,"_ _"we don't do it this way,"_ _"add a test for the empty case."_

This skill reads those comments (and the repo's config and code), **weights the people who actually gate merges**, ranks rules by how often they recur, and writes a prioritized ruleset to `.coding/`. The companion `coding` and `fix-me-bug` skills then read it so future code passes review the first time.

**Core principle:** Learn from what reviewers *repeatedly* say, not from generic best-practice lists.

## When to use

- Starting work in an unfamiliar or new-to-you repository.
- Reviewers keep leaving the same kinds of comments on your PRs.
- Onboarding; you want the team's conventions before writing code.
- Periodically, to refresh `.coding/` as conventions drift.

**When not to use:** a brand-new repo with no PR history (there's little to learn — `coding`'s baseline is enough until reviews accumulate).

## What it produces

**One file** in the **target project's root**: `.coding/coding-guidelines.md` — the team's **generalized conventions** distilled from review comments (short imperative DO/DON'T lines, most-enforced first; the pattern behind the comments, not a comment log). It is the single source of truth for `coding` and `fix-me-bug`, so keep it **strict and token-friendly**. See `references/output-template.md` for the exact format.

No separate evidence/sources file — don't write one. Spend that effort reading **more** comments and **more recent closed/merged PRs** instead. Tell the user to **commit `.coding/`** so the whole team shares one learned standard.

## Workflow

**First, post a TODO list of the plan** so the user sees exactly what's coming. Create one todo per step:

1. Preflight — auth, repo, rate limit
2. Build authority list — who counts (maintainers/team)
3. Pass 1 — declared rules (config, CONTRIBUTING, CI, commits)
4. Pass 2 — enforced rules (mine PR review comments)
5. Distill + rank into rules
6. Write `.coding/`
7. Report

Then work the list top to bottom, marking each item done as you go. The one branch: if `gh` auth fails at Preflight, run local-only (config + `git log`) and skip Pass 2.

**Keep the run clean:** prefer the ready-made `jq` commands in `references/gh-mining.md` over ad-hoc scripts, and give a one-line result per step instead of dumping raw output.

### 1. Preflight

- Confirm `gh auth status` works and `git` is present. If `gh` is missing/unauthenticated, tell the user and continue **local-only**: build `.coding/` from Pass 1 only (config + `git log`); skip Pass 2 (PR-comment) mining, and mark the `Learned on` header `[LOCAL-ONLY]` so readers know the high-signal review data is absent.
- Resolve the repo: `gh repo view --json nameWithOwner,defaultBranchRef`.
- Check budget before any large mining: `gh api rate_limit --jq '.resources.core | {remaining, limit, reset}'`. If `remaining` is low, reduce `--limit`/page count and say so.

### 2. Build the authority list (who counts — everyone else is noise)

Reviewers are not equal. On a public repo **anyone** can comment — drive-by users and bots add opinions that are **not** the team's standard. Only count people who gate merges. Build the list in priority order:

1. **CODEOWNERS** — the authoritative reviewers. Try `.github/CODEOWNERS`, `CODEOWNERS`, `docs/CODEOWNERS`.
2. **Collaborators with write/admin** — `gh api repos/{owner}/{repo}/collaborators --jq '.[] | {login, role: .role_name}'`.
3. **Top contributors** — proxy for maintainers when the above are unavailable.

**The filter — apply it to every comment in Pass 2:**
- **KEEP** only `author_association` ∈ {`OWNER`, `MEMBER`, `COLLABORATOR`} — the team.
- **DROP** outside commenters: `CONTRIBUTOR`, `FIRST_TIMER`, `FIRST_TIME_CONTRIBUTOR`, `NONE`, `MANNEQUIN`.
- **DROP all bots**: `user.type == "Bot"`, or login ending in `[bot]` (Copilot, dependabot, github-actions, …).
- **DROP** a PR author commenting on their own PR (self-notes, not review).

Weighting: a recurring point from an `OWNER`/`MEMBER` outranks a one-off from a `COLLABORATOR`. Carry this filter into every Pass 2 `--jq select(...)`.

### 3. Pass 1 — declared rules (what the repo says)

Fast, cheap, high-confidence. Read and summarize:

- Linter/formatter configs (`.eslintrc*`, `eslint.config.*`, `.prettierrc*`, `biome.json`, `ruff.toml`, `pyproject.toml`, `.rubocop.yml`, `.golangci*`, `.editorconfig`, `tsconfig.json` strictness).
- `CONTRIBUTING.md` (and `.github/CONTRIBUTING.md`) — explicit stated rules.
- CI workflows (`.github/workflows/*`) — what's a **required**, merge-blocking check.
- Commit-message convention from history: `git log -n 200 --pretty=format:'%s'` (look for Conventional Commits, ticket prefixes, etc.).

### 4. Pass 2 — enforced rules (what reviewers actually say) — the gold

Mine review comments across **both open AND closed/merged PRs** — they carry different signal:
- **Open PRs** — what reviewers are pushing for *right now*; the team's current, live standards.
- **Closed / merged PRs** — what got enforced and fixed before merge; the historical, settled conventions.

Mine **repo-wide** (one paginated call beats per-PR loops). The repo-wide comment endpoints return comments from PRs of every state, so a single pass already covers open + closed + merged. The three comment surfaces are different and not interchangeable:

- **Inline code-line comments** (`/pulls/{n}/comments` or repo-wide `/pulls/comments`) — the richest signal: tied to a file/line/diff.
- **Review verdicts + summaries** (`/pulls/{n}/reviews`) — `CHANGES_REQUESTED` bodies say why a PR was blocked.
- **PR conversation comments** (`/issues/{n}/comments`) — general discussion.

Apply the Step 2 filter to **every** query — keep only team (OWNER/MEMBER/COLLABORATOR), drop bots and outside commenters. Then find signal by frequency, not volume:

- Most-commented files (convention hotspots) and most-active reviewers (who sets the standard).
- Cluster comments by theme (naming, tests, error handling, structure, …).

Exact commands: see `references/gh-mining.md`.

### 5. Distill + rank

Read **a lot** of comments before distilling — depth beats a tidy summary. Bias toward the **most recent closed/merged PRs**: they reflect the team's current standard, not abandoned old habits. Then turn recurring clusters into rules.

**Generalize the comment into the convention.** A comment is an *instance*; the rule is the *pattern* behind it. Lift the general rule, don't copy the literal note:

| Comment (instance) | Rule (generalized) |
|--------------------|--------------------|
| "this variable is unused" | Remove unused variables, params, and imports. |
| "don't inline this — use the shared type" | Reuse shared/domain types, not inline object shapes. |
| "these listeners are never removed" | Always clean up event listeners / subscriptions. |
| "can we rename this to a clearer plural?" | Name collections with descriptive plural nouns. |

Keep a rule **specific only when the convention itself is specific** (a named type, a required helper, a particular file not to touch). Don't over-generalize a genuine one-off into a sweeping law.

A rule earns a place only if it is:

- **Team-sourced** — raised by a real team reviewer (OWNER/MEMBER/COLLABORATOR), not a bot or outside commenter.
- **Recurring or recent** — said more than once, or enforced in recent PRs. True one-offs → low-confidence section, or cut.
- **Actionable** — a coder can comply: _"name booleans `isX`/`hasX`"_, not _"write clean code."_
- **Observable** — phrased against a diff where possible (_"every logic change adds a test,"_ _"no `console.log` in committed code"_).

Don't restate generic best practice the baseline already covers — only what's *specific to this team*. Order by enforcement frequency (most-flagged first). On conflict, prefer the most recent / highest-authority signal.

### 6. Write `.coding/coding-guidelines.md`

- Create `.coding/` in the project root if absent. Write the **single** file `coding-guidelines.md` using `references/output-template.md`. **No `sources.md`** — don't create one.
- Keep it **strict and token-friendly**: short imperative lines, most-enforced first, no padding (the file is read by an agent before every task).
- **Re-run = regenerate**, don't blindly append: rebuild the ruleset, update the `Learned on` date, and preserve any user-added `## Manual overrides` section verbatim (see template).

### 7. Report

Tell the user (briefly): repo + branch mined, # comments / recent PRs analyzed, team reviewers counted, top 5 rules, and `gh` rate-limit remaining. Remind them to commit `.coding/`.

## Privacy & safety

- Read-only mining. Never post comments, never push, never alter the target repo.
- The guidelines may paraphrase what reviewers enforce; don't include anything beyond what the public GitHub API returns.

## Common mistakes

| Mistake | Fix |
|---------|-----|
| Writing a `sources.md` / evidence file | Don't. One file only: `coding-guidelines.md`. Spend the effort reading more comments. |
| Counting bots / outside commenters | Keep only team (OWNER/MEMBER/COLLABORATOR); drop `[bot]` users and CONTRIBUTOR/NONE. |
| Skimming a few comments | Read many, and weight recent closed/merged PRs — that's where the current standard lives. |
| Restating generic best practice | Only write rules *specific to this team*; the baseline covers the rest. |
| Verbose, padded rules | One strict imperative line each; token-friendly. |
| Per-PR API loops | Use repo-wide `--paginate` endpoints; gate on `rate_limit`. |
| Appending forever on re-run | Regenerate; keep only the `## Manual overrides` section. |
| Failing hard when `gh` is absent | Degrade to local-only Pass 1 + `git log`; say what was skipped. |

## Reference

- `references/gh-mining.md` — exact, copy-pasteable `gh`/`git` command cookbook (team-only + bot filter, recency sort, comment surfaces, frequency ranking, pagination, rate limits).
- `references/output-template.md` — the precise `.coding/coding-guidelines.md` format (single file, strict, token-friendly).
