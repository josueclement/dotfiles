# FEATURE-002-PHASE01 — `dev-workflow` skill overhaul — DONE

## Summary

Overhauled the shared `dev-workflow` skill (`.claude/skills/dev-workflow/SKILL.md`) — the single source of
truth both `/interview` and `/build` defer to — to encode the six FEATURE-002 changes at the contract level,
so the two commands (PHASE02, PHASE03) can simply defer to it. Introduced skill versioning at `v1`.

## Files/modules touched

- **Modified** `.claude/skills/dev-workflow/SKILL.md`:
  - Added `**Version: dev-workflow v1.**` under the title; updated the frontmatter description to name
    `docs/done/<ID>.md` and the `CODE-REVIEW-NNN` type.
  - **Two flows / intro**: `/interview` now *only plans* (never builds); `/build` is the sole executor.
  - **Version Control**: dev branches are cut from the **current `HEAD`** (branch or detached commit), never
    the default branch; detached `HEAD` is explicitly accepted; the clean-tree and branch-name-collision
    hard-stops are retained. Added the `review/` branch prefix and the `fix(CODE-REVIEW-NNN): … (PHASE0N)`
    commit scope.
  - **Work-item identifiers**: added `CODE-REVIEW-NNN` (own per-type counter) and the rule that each finding
    is a PHASE, ordered highest-severity first, severity recorded in the phase title/plan; added an example
    roadmap block.
  - **Documentation structure**: added `docs/done/` and included it in the bootstrap step.
  - **New "Completion docs" section** describing `docs/done/<ID>.md` (per-dev, phase-suffixed) with its four
    content sections; it lands in the dev's own commit.
  - **New "Line endings (CRLF)" section**: never fix, never ask, recommendation-only.
  - **Definition of Done**: added criterion 5 (completion doc written); criteria 3–5 apply to no-build work.
- **Created** `docs/plan/FEATURE-002.md` (this item's plan) and `docs/done/` (this record bootstraps it).
- **Modified** `docs/roadmap.md` (added the FEATURE-002 rows).

## Deviations & follow-ups

- No deviations from the plan.
- **Follow-up (out of scope, not done):** the repo has no `.gitattributes` normalizing line endings. Per the
  new CRLF rule this is a recommendation only — adding `* text=auto eol=lf` would prevent CRLF↔LF churn at
  the source. No CRLF churn was observed in this phase's edits.

## Build/test evidence

No build step or test suite (prompt/config markdown). Verified by inspection and dry-run:
- `grep` confirmed the version marker is present, 10 `CODE-REVIEW` references, 6 `docs/done` references, and
  that no `optionally executes` / `execute now` / `stop after plans` text remains; the only `default branch`
  occurrence is the intended "never switch to it" phrasing.
- The skill hot-reloaded with its updated description, confirming well-formed frontmatter.
- All eight PHASE01 plan steps are reflected in the file.
