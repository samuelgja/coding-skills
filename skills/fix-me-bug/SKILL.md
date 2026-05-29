---
name: fix-me-bug
description: Use when fixing any bug, regression, failing test, crash, or unexpected behavior in any language or codebase. Test-first, root-cause bug fixing that also follows the team's learned conventions from .coding/ so the fix doesn't draw new review comments.
---

# fix-me-bug

**Core rule:** No fix without a failing test. The test IS the proof the bug existed and is gone.

**Also:** a fix is still a code change — it gets reviewed. Before editing, read `.coding/coding-guidelines.md` (if present) so the fix matches the team's conventions and doesn't earn a fresh round of review comments. Missing? Apply the generic baseline and continue.

## Step 1: Triage — pick your path

Read the report. Ask: **Can I write a failing test right now?**

**YES** (→ FAST PATH) when any hold:
- Expected vs actual behavior is stated.
- Reproduction steps are clear.
- It's a regression ("used to work").
- The module has clear inputs/outputs.

**NO** (→ DEEP PATH) when:
- Non-deterministic / can't reproduce.
- Expected behavior is ambiguous.
- Unfamiliar system, no idea where it lives.
- Symptoms could have multiple unrelated causes.

**Default to FAST PATH** — most reported bugs already carry enough to write a test.

## FAST PATH: test first

### 1. Write the failing test
- Encode the bug directly: expected behavior vs actual.
- One test, one behavior. Clear name: `test('rejects empty email with validation error')`.
- Use real code, not mocks, unless unavoidable.
- Match the project's existing test style and the conventions in `.coding/`.

### 2. Run it — confirm it FAILS
- Must **fail**, not error. If it errors, fix the test setup first.
- If it **passes**, your understanding is wrong → switch to DEEP PATH.
- Read the failure message carefully — it often names the root cause.

### 3. Fix
- One minimal change targeting the **root cause**, not the symptom.
- Run the test. Passes → step 4. Still fails → re-examine; second failed attempt → DEEP PATH with what you learned. Don't stack fixes.

### 4. Verify
- Full test suite green (catch regressions you introduced).
- Format / lint / typecheck pass.
- The fix obeys `.coding/coding-guidelines.md` + baseline (guard clauses, naming, no debug leftovers). Run the `coding` reviewer check: *"would a reviewer comment on this fix?"*
- Conventional Commit, e.g. `fix(auth): reject empty email`.

## DEEP PATH: investigate, then test

### 1. Reproduce
- Get a minimal, reliable trigger. Can't trigger it → can't fix it.
- Read the full error message + stack trace; they often hold the answer.

### 2. Localize — binary search
- **Regressions:** `git bisect` between known-good and known-bad commits.
- **Data bugs:** probe the midpoint of the data flow. Correct there → bug is downstream; wrong → upstream. Repeat.
- **Logic bugs:** isolate half the suspect path; narrow down which half misbehaves.

Goal: pinpoint the file + function where the bug originates.

### 3. Hypothesize
State it explicitly: **"The bug is [X] because [Y]."** Can't form one → back to step 1 with more logging/probes.

### 4. Test + fix + verify
Same as FAST PATH steps 1–4 — now that you understand it, encode the bug as a test and fix the root cause.

## Multiple bugs
1. One todo per bug.
2. Quick-scan for a shared root cause.
3. Independent bugs in different files → can fix in parallel; related or same-file → sequentially, one full cycle each.
4. Each bug gets its **own** failing test — no shared "fix everything" test.
5. Run the full suite once at the end to catch interactions.

## Escalation — 3 failed attempts? STOP.
Don't attempt fix #4. Ever. On the 3rd failed attempt, escalate — this is not a suggestion. Stacking fixes masks and compounds the real (often architectural) problem. Tell the user:
- What you tried and why each attempt failed.
- Whether this needs a redesign vs a patch.
- Ask how they want to proceed.

| Rationalization | Reality |
|-----------------|---------|
| "I'm very close on attempt #3" | You're not. Escalate. |
| "Just one more quick fix" | That's fix #4. Forbidden. Escalate now. |

## When stuck

| Problem | Action |
|---------|--------|
| Can't reproduce | Add logging at each layer. Run once. Read output. |
| Test passes (expected fail) | Understanding is wrong → DEEP PATH. |
| Fix didn't work | New hypothesis. Don't stack fixes on failed ones. |
| Can't write a test | Narrow to the smallest unit where the *actual* behavior breaks (not a downstream symptom). A valid test FAILS on the buggy code — if it passes, you isolated the wrong thing. |
| Test errors (not fails) | Fix setup: imports, fixtures, assertions. |

## Companion skills
- `learn-coding` generates `.coding/coding-guidelines.md` (the conventions this fix should follow).
- `coding` — same conventions for new code; reuse its reviewer check before declaring the fix done.
