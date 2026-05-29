---
name: coding
description: Use when writing or changing any code, in any language, before and while implementing. Applies the team's learned conventions from .coding/coding-guidelines.md on top of a strict baseline so the change passes review the first time instead of getting re-commented by a coworker or lead. Run learn-coding first to generate the conventions; without them, this falls back to generic baseline rules.
---

# coding

## Overview

Write code that **passes review the first time** — no coworker or team lead re-commenting the same things. Every team codes differently; those differences are captured by `learn-coding` in `.coding/coding-guidelines.md`. This skill reads that file and applies it **on top of** a strict, language-agnostic baseline.

**Core principle:** Generic-correct isn't enough. Match *this team's* conventions, which live in the learned file. The baseline is the floor; the learned rules are what stop the re-comments.

## Step 0 — Read the guidelines (mandatory, do this first)

Before writing or changing any code:

1. Look for `.coding/coding-guidelines.md` in the project root (walk up from the working directory).
2. **Found** → read it fully, including `.coding/sources.md` if you need the reasoning. These rules now take priority. Re-read it at the start of each coding task (it may have changed).
3. **Missing** → say once: _"No `.coding/coding-guidelines.md` found — applying the generic baseline. Run `learn-coding` to make this project-specific."_ Then continue with the baseline. **Do not block.**

Skipping Step 0 is the main failure mode. Reading a generic ruleset when a learned one exists means you'll get the exact comments the team already wrote down.

## Priority order

When rules conflict, higher wins:

```
1. .coding/coding-guidelines.md  → ## Manual overrides   (team's explicit word)
2. .coding/coding-guidelines.md  → learned team rules     (what reviewers enforce)
3. coding-practises.md              → strict baseline         (generic floor)
```

If the learned file contradicts the baseline (e.g. team allows longer functions), **follow the team** — that's the whole point.

## Workflow

### Before writing — discover & reuse
- **Search before you create.** Grep/glob for an existing function, component, type, or pattern that does this. Reuse > extend > compose > create-new (in that order). Don't add a near-duplicate of something that exists. (Baseline §Architecture.)
- Match the surrounding code's style, naming, and structure — read a neighbor file first.

### While writing — apply the rules
- Apply learned rules + baseline together. Key baseline reflexes: guard clauses over nesting; one responsibility per function/file; explicit names (`isX`/`hasX` for booleans, no cryptic abbreviations); no dead/commented-out code; a test for every behavior change; no secrets in code. Full list: `references/coding-practises.md`.
- Keep the change **small and one-concern** — don't mix a refactor into a feature/fix.

### Before claiming done — the reviewer check
Ask, against the learned rules specifically: **"Would a reviewer on this team comment on this?"** Walk the diff and check:
- [ ] Every learned rule that applies to this diff is satisfied.
- [ ] Format / lint / typecheck / tests pass (run them; don't assume).
- [ ] Commit message matches the team's convention (Conventional Commits unless the learned file says otherwise).
- [ ] No new TODOs, debug logs, secrets, or unused/speculative code.

If anything fails, fix it before saying it's done. Evidence before assertions.

## Quick reference — baseline categories

Full rules in `references/coding-practises.md`. The nine categories:

| # | Category | The reflex |
|---|----------|-----------|
| 1 | Source control & PRs | Small (~≤400 LOC), one concern per PR; Conventional Commits. |
| 2 | Code review readiness | Self-review the diff as a reviewer would, first. |
| 3 | Style & readability | Guard clauses; SRP; explicit names; delete dead code. |
| 4 | Architecture & modularity | Reuse first; respect boundaries; no speculative APIs. |
| 5 | Testing | Test the behavior you changed; deterministic, no flaky. |
| 6 | CI / quality gates | Lint + types + tests green before done; never red main. |
| 7 | Security & dependencies | No secrets; pin/verify deps; least privilege. |
| 8 | Documentation & knowledge | Update docs for user-facing changes; comments say *why*. |
| 9 | AI-assisted code | Review AI output at equal/greater scrutiny; verify libs exist. |

## Red flags — stop and fix

These thoughts mean you're about to get re-commented:

| Thought | Reality |
|---------|---------|
| "I'll skip reading `.coding/`, I know good code" | Generic ≠ this team. The learned rules are the ones reviewers already wrote. Read them. |
| "I'll just create a new helper" | Search first — reuse/extend beats a near-duplicate. |
| "I'll add tests later" | Behavior change without a test gets blocked. Add it now. |
| "Nesting is fine here" | Guard clauses. Flat code reviews faster. |
| "Small refactor while I'm here" | One concern per PR. Split it. |
| "Lint will probably pass" | Run it. Assumptions get re-commented. |
| "The learned rule seems wrong" | Follow it (or use `## Manual overrides`); don't silently override the team. |

## Common mistakes

| Mistake | Fix |
|---------|-----|
| Coding before Step 0 | Read `.coding/coding-guidelines.md` first, every task. |
| Applying baseline when a learned rule contradicts it | Learned/team rules win over baseline. |
| Treating the missing-file case as a blocker | It's not — fall back to baseline, suggest `learn-coding`, continue. |
| Declaring done without running checks | Run format/lint/types/tests; show the result. |

## Reference

- `references/coding-practises.md` — the full strict baseline ruleset (the floor, applied when the learned file is silent).
- Companion skills: `learn-coding` (generates `.coding/`), `fix-me-bug` (test-first fixes that also follow `.coding/`).
