# Output template for `.coding/`

`learn-coding` writes these two files to the **target project root**. `coding` and `fix-me-bug` read `coding-guidelines.md`. Keep both files concise and scannable — they get loaded by an agent before every task.

---

## `.coding/coding-guidelines.md`

````markdown
# Coding Guidelines — <owner/repo>

> Learned by `learn-coding` on <YYYY-MM-DD> from <N> PRs / <M> review comments.
> Evidence: see `.coding/sources.md`. Re-run `learn-coding` to refresh.

## How to read this
Rules are ordered by how often the team enforces them — top = most-flagged in review.
Each rule is one actionable line. `[E#]` links to evidence in `sources.md`.

## Team rules (learned)

### Naming & style
- DO: <rule>. <why> `[E1]`
- DON'T: <rule>. <why> `[E2]`

### Structure & architecture
- DO: <rule>. <why> `[E3]`

### Error handling
- ...

### Testing
- DO: <rule>. <why> `[E4]`

### PRs & commits
- DO: <rule>. <why> `[E5]`

### Language / framework specifics
- <stack-specific rules, e.g. React, Go, Python>

## Most-commented areas
Files/paths reviewers touch most (write extra-carefully here):
- `<path>` — <theme> `[E#]`

## Confidence & gaps
- Low-confidence (seen once / single author): <list>
- Not yet observed (baseline applies): <areas with no review signal>

## Manual overrides
<!-- Anything the team writes here is preserved across re-runs and WINS over learned rules. -->
````

**Rules for filling this in**

- Most-enforced rules first. Each rule actionable + observable; attach `[E#]`.
- No rule without evidence in `sources.md`. Generic best practice with no team signal → leave it to the baseline, don't restate it here.
- On contradiction, prefer the most recent / highest-authority signal and note it under Confidence & gaps.
- **Re-run = regenerate**: rebuild everything except the `## Manual overrides` block, which you copy across verbatim. Update the `Learned on` line.

---

## `.coding/sources.md`

````markdown
# Evidence — <owner/repo>

> Real review comments behind each rule in `coding-guidelines.md`. Generated <YYYY-MM-DD>.

## Authority weighting used
Comments from these accounts were weighted highest:
- `@<login>` — CODEOWNERS / OWNER / MEMBER / role
- ...

- **[E1]** "<verbatim comment>" — `@reviewer` (OWNER), PR #123, 2026-04-02. <link>. Seen ~7× across PRs.
- **[E2]** "<verbatim comment>" — `@reviewer` (MEMBER), PR #98. <link>. Seen ~4×.
- ...
````

Keep evidence to the highest-signal comments (the ones that recur or come from authority). Quote verbatim; include author, association, PR #, date, link, and approximate frequency.
