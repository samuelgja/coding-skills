# Output template for `.coding/coding-guidelines.md`

`learn-coding` writes **one** file: `.coding/coding-guidelines.md`. **No `sources.md`, no evidence file.** `coding` and `fix-me-bug` read this file before every task, so keep it **strict and token-friendly** — short imperative lines, no prose padding, no restating generic best practice the baseline already covers. Only rules *specific to this team*.

---

````markdown
# Coding Guidelines — <owner/repo>

> Learned by learn-coding on <YYYY-MM-DD> from <N> team review comments across <M> recent PRs.
> Re-run learn-coding to refresh.

## Rules — most-enforced first

### Naming & style
- <strict imperative rule>
- ...

### Structure & architecture
- ...

### Error handling
- ...

### Testing
- ...

### PRs & commits
- ...

### <language / framework> specifics
- ...

## Hotspots
Files reviewers flag most — take extra care:
- `<path>` — <what gets flagged here>

## Manual overrides
<!-- Team edits below are preserved across re-runs and WIN over learned rules. -->
````

---

## Rules for filling it in

- **One strict imperative line per rule.** A coder complies just by reading it. Cut filler; add a `≤6-word why` only when not obvious.
- **Most-enforced first** — order by how often team reviewers raise it.
- **Team signal only.** Include a rule only if real team reviewers (OWNER/MEMBER/COLLABORATOR) raised it — ideally more than once, or in recent PRs. Bots and outside commenters don't count.
- **Specific, not generic.** If the strict baseline already covers it, don't restate it. This file is what makes `coding` *team-specific*.
- **No evidence file.** Don't write `sources.md`. If a rule's origin is genuinely non-obvious you may append one short PR link in parens — but prefer reading *more* comments over documenting them.
- **Recency wins** on conflict: prefer what recent closed/merged PRs enforce.
- **Token-friendly.** This file is loaded before every coding task — keep it lean.
