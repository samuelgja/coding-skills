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

Two files in the **target project's root** (see `references/output-template.md` for the exact format):

| File | Contents |
|------|----------|
| `.coding/coding-guidelines.md` | The distilled, prioritized ruleset — DO / DON'T + why, ordered by enforcement frequency. The single source of truth for `coding`/`fix-me-bug`. |
| `.coding/sources.md` | Evidence — the real review comments behind each rule (author, association, date, link, count). Makes rules auditable, not invented. |

Tell the user to **commit `.coding/`** so the whole team shares one learned standard.

## Workflow

Run these seven steps in order: **Preflight → Authority list → Pass 1 (declared) → Pass 2 (enforced) → Distill → Write `.coding/` → Report.** The one branch: if `gh` auth fails at Preflight, run local-only (config + `git log`) and skip Pass 2.

### 1. Preflight

- Confirm `gh auth status` works and `git` is present. If `gh` is missing/unauthenticated, tell the user and continue **local-only**: build `.coding/` from Pass 1 only (config + `git log`); skip Pass 2 (PR-comment) mining, and mark the `Learned on` header `[LOCAL-ONLY]` so readers know the high-signal review data is absent.
- Resolve the repo: `gh repo view --json nameWithOwner,defaultBranchRef`.
- Check budget before any large mining: `gh api rate_limit --jq '.resources.core | {remaining, limit, reset}'`. If `remaining` is low, reduce `--limit`/page count and say so.

### 2. Build the authority list (who to weight)

Reviewers are not equal. Comments from people who gate merges define the standard. Build the list in priority order:

1. **CODEOWNERS** — the authoritative reviewers. Try `.github/CODEOWNERS`, `CODEOWNERS`, `docs/CODEOWNERS`.
2. **Collaborators with write/admin** — `gh api repos/{owner}/{repo}/collaborators --jq '.[] | {login, role: .role_name}'`.
3. **Top contributors** — proxy for maintainers when the above are unavailable.

Weighting rule: a recurring point from an OWNER/MEMBER outranks a one-off from a drive-by CONTRIBUTOR. Carry the authority set into the `--jq select(...)` filters in Pass 2.

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

Filter to `author_association` ∈ {OWNER, MEMBER, COLLABORATOR} and the authority list. Then find signal by frequency, not volume:

- Most-commented files (convention hotspots) and most-active reviewers (who sets the standard).
- Cluster comments by theme (naming, tests, error handling, structure, …).

Exact commands: see `references/gh-mining.md`.

### 5. Distill + rank

Turn clusters into rules. A rule is worth writing only if it is:

- **Recurring** — said more than once, or by an owner with authority. One-offs go in a low-confidence section, not the main list.
- **Actionable** — a coder can comply: _"name booleans `isX`/`hasX`"_, not _"write clean code."_
- **Observable** — phrased against a diff where possible (_"every logic change adds a test,"_ _"no `console.log` in committed code"_) so it's checkable.
- **Evidence-backed** — at least one real comment in `sources.md`.

Order rules by enforcement frequency (most-flagged first). Reconcile contradictions toward the most recent / highest-authority signal and note the conflict.

### 6. Write `.coding/`

- Create `.coding/` in the project root if absent.
- Write `coding-guidelines.md` and `sources.md` using `references/output-template.md`.
- **Re-run = regenerate**, don't blindly append: rebuild the ruleset, update the `Learned on` date, and preserve any user-added `## Manual overrides` section verbatim (see template).

### 7. Report

Tell the user: repo + branch mined, # PRs / comments analyzed, authority set used, top 5 recurring rules, confidence/gaps, and `gh` rate-limit remaining. Remind them to commit `.coding/`.

## Privacy & safety

- Read-only mining. Never post comments, never push, never alter the target repo.
- `sources.md` quotes public review comments with author handles and links — these are already public on the repo. Don't include anything beyond what the GitHub API returns.

## Common mistakes

| Mistake | Fix |
|---------|-----|
| Dumping every comment | Rank by frequency/authority; keep the top recurring rules. |
| Treating all reviewers equally | Weight CODEOWNERS / OWNER / MEMBER. |
| Inventing rules with no evidence | Every rule needs a `sources.md` entry. Cut the rest. |
| Vague rules ("be consistent") | Make them actionable and observable. |
| Per-PR API loops | Use repo-wide `--paginate` endpoints; gate on `rate_limit`. |
| Appending forever on re-run | Regenerate; keep only the `## Manual overrides` section. |
| Failing hard when `gh` is absent | Degrade to local-only Pass 1 + `git log`; say what was skipped. |

## Reference

- `references/gh-mining.md` — exact, copy-pasteable `gh`/`git` command cookbook (authority list, comment surfaces, frequency ranking, pagination, rate limits).
- `references/output-template.md` — the precise `.coding/coding-guidelines.md` and `.coding/sources.md` format.
