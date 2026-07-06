# FEATURE-002-PHASE03 — `/build` current-HEAD branching + CODE-REVIEW + done docs (v2) — DONE

## Summary

Refactored the `/build` command (`.claude/commands/build.md`) from v1 to **v2** to match the skill contract
set in PHASE01: it now branches from the user's **current `HEAD`** (detached `HEAD` allowed; never the
default branch) while retaining the clean-tree and branch-collision hard-stops; resolves and gates
**`CODE-REVIEW-NNN`** items (and their phases); writes the **`docs/done/<ID>.md`** completion doc as part of
the Definition of Done; and never touches CRLF (recommendation-only). This completes FEATURE-002.

## Files/modules touched

- **Modified** `.claude/commands/build.md`:
  - Version line → `Using build v2 by Josué Clément`.
  - Description / Context → planning-vs-build split clarified (`/interview` only plans; `/build` branches
    from current `HEAD`); accepts FEATURE / BUG / CODE-REVIEW.
  - Phase 2 resolution → base and phase ID examples now include `CODE-REVIEW-001` /
    `CODE-REVIEW-001-PHASE01`.
  - Phase 3 pre-flight → current-`HEAD` starting point (detached OK; never the default branch); clean-tree /
    branch-collision stops retained.
  - Phase 4 → branch from current `HEAD` with the `review/` prefix for code reviews; Definition of Done now
    includes writing `docs/done/<ID>.md` (lands in the dev's own commit).
  - Rules → added the "never fix CRLF" recommendation-only rule.
- **Modified** `docs/roadmap.md`, `docs/plan/FEATURE-002.md` (PHASE03 → DONE; FEATURE-002 base row → DONE).

## Deviations & follow-ups

- No deviations from the plan.
- **Follow-up (unchanged across all phases, not done):** repo has no `.gitattributes`; adding
  `* text=auto eol=lf` would prevent CRLF↔LF churn at the source. Recommendation only, per the new rule.
- **Note on branching model:** because `/build` (and this run) branch from current `HEAD`, FEATURE-002's three
  phase branches stack (PHASE02 on PHASE01, PHASE03 on PHASE02). Merging the PHASE03 branch brings in all
  three phases. This is the intended consequence of the "use where I am" decision.

## Build/test evidence

No build step or test suite (prompt/config markdown). Verified by:
- **Adversarial workflow** `verify-build-v2-ecosystem` — 4 independent lenses (build branching/version,
  CODE-REVIEW end-to-end vs the skill, completion-doc/DoD, and whole-ecosystem coherence across skill +
  interview + build) plus an independent verify stage. Result: **0 findings**.
- **Cross-file grep**: version lines are `dev-workflow v1` / `Using interview v7 …` / `Using build v2 …`; no
  stale `build v1` / `interview v6`; the only `default branch` mentions in build.md are the intended "never
  switch to it" phrasing; `CODE-REVIEW`, `review/`, `docs/done`, current-`HEAD`, and the CRLF rule all
  present.
