# Coding practises (strict baseline)

The language-agnostic floor `coding` applies when `.coding/coding-guidelines.md` is silent. Learned team rules and `## Manual overrides` always win over anything here. Each rule is phrased to be **observable in a diff** so it's checkable. Sources: Google eng-practices, trunk-based development, Conventional Commits, OWASP, testing-pyramid guidance, and 2025–2026 AI-code governance.

---

## 1. Source control & PRs
- One self-contained change per PR. Never mix refactor with feature/bugfix — split them. (Reviewers lose the plot; bugs hide in noise.)
- Keep diffs small: ~100 LOC is ideal, ~400 LOC is the soft ceiling. Larger → split. (Defect detection collapses after ~60 min of review; small PRs get real review.)
- Every PR that changes logic includes new/updated tests for that behavior. Untested logic is non-mergeable.
- Conventional Commits: `type(scope): description`. `feat`→minor, `fix`→patch, `BREAKING CHANGE:` footer→major. (Machine-readable history, automated releases.)
- Short-lived branches: merge within ~1–2 days. Wrap incomplete work behind a feature flag so the trunk stays releasable. Remove dead flags.

## 2. Code review readiness (self-review first)
- Read your own diff as a reviewer before requesting review. Fix what you'd comment on.
- Separate must-fix from taste: don't ship something you'd block in review. (If the team marks taste as `Nit:`, mirror that.)
- Don't over-engineer: solve the problem you have now, not a hypothetical one. No speculative abstraction.

## 3. Style & readability
- Guard clauses / early returns over nested conditionals. Flat code is scannable and reviews faster.
- One responsibility per function. Prefer small functions; a function doing several things is several functions.
- Explicit names. Booleans get a verb prefix (`isActive`, `hasPermission`, `canEdit`). No cryptic abbreviations (`usr`→`user`, `cnt`→`count`); domain-standard short names (`id`, `db`, `ctx`, `props`) are fine.
- Delete dead code and commented-out blocks. Version control is the history; don't ship graveyards.
- Comments explain **why**, not **what**. Good names remove the need for most comments.
- Follow the repo's existing style and the language's idioms; match neighboring files. Let the formatter/linter own mechanical style.
- File size: keep modules focused (~200 lines is a useful split trigger). A growing file usually means mixed responsibilities — extract.

## 4. Architecture & modularity
- **Discover before creating.** Search the codebase for an existing function/component/type/pattern first. Reuse > extend > compose > create-new. (Duplication is debt; near-duplicates drift apart.)
- Respect module boundaries and dependency direction; don't reach across layers or create cycles.
- A change belongs in this codebase (not vendored into a library), integrates cleanly, and ships with a real caller — no unused/speculative public APIs.
- Keep units cohesive: one clear purpose, a well-defined interface, understandable without reading internals.

## 5. Testing
- Test the behavior you changed — the test must fail if the code breaks. Coverage of the change, not coverage theater.
- Test pyramid: many fast unit tests, fewer integration tests (use a real DB for query/schema/transaction logic), few E2E (~5–10%, reserved for critical journeys: auth, payment, core path).
- Tests are deterministic: same input → same result regardless of order/clock/network. Mock external deps, use stable selectors and fixed data, proper waits (not sleeps).
- Zero tolerance for flaky tests: quarantine and fix; never normalize "re-run until green."

## 6. CI / quality gates
- Format + lint + typecheck (strict) + tests + coverage must pass **before** the change is done. Run them locally; don't assume.
- Gates are blocking, not advisory. Never leave `main`/trunk red.
- Fast feedback first (lint/types before slow E2E) so failures surface early.

## 7. Security & dependencies
- No secrets in code, config, logs, or tests. Use env/secret managers and short-lived credentials.
- Validate and sanitize external input; parameterize queries; encode output. (Injection is still #1.)
- Dependencies: add deliberately, pin versions, commit the lockfile, prefer maintained packages. Verify a referenced library actually exists and is current (guards against hallucinated/typosquatted deps).
- Least privilege everywhere (tokens, IAM, file perms). Don't broaden scope "to make it work."
- Treat SCA/SAST findings of high severity as blocking.

## 8. Documentation & knowledge
- Update README / reference docs for any user-facing or interface change in the same PR.
- Record architecturally significant decisions as a short in-repo ADR (context, decision, consequences). Supersede — don't rewrite — a decided ADR.
- Prefer self-documenting code; reserve comments for non-obvious *why* and gotchas.

## 9. AI-assisted development (2025–2026)
- AI-generated code gets **equal or greater** scrutiny than human code — it carries more subtle logic/correctness errors. The human author stays fully accountable.
- Read and understand every AI-produced line before committing it. No "looks right, ship it."
- AI output is non-mergeable until it has edge-case tests.
- Don't rely on AI generation for authentication, authorization, or secrets handling without manual security review.
- Disclose AI involvement per the team's policy (commit trailer / PR note) where required.

---

## One-screen checklist (run before "done")
- [ ] One concern, small diff, tests included
- [ ] Guard clauses, explicit names, no dead code, no debug logs
- [ ] Reused existing code where it existed (searched first)
- [ ] Behavior change has a deterministic test
- [ ] Format + lint + types + tests green (ran them)
- [ ] No secrets; deps pinned + lockfile updated
- [ ] Docs/ADR updated if interface or decision changed
- [ ] Conventional Commit message
- [ ] AI-written lines understood, tested, and not in auth/secrets paths
