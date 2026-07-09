# FEATURE-005 — `/build` prints the roadmap when invoked with no ID

**Status:** DONE
**Branch:** `feature/feature-005-build-roadmap-listing` (cut from `master` @ `d772bb2`)

## Summary

Replaced the `AskUserQuestion` picker in the no-argument path of `/build` with a full-roadmap-table print. When `/build` is invoked with no work-item ID, it now reproduces the entire `docs/roadmap.md` table (every item and phase row, all statuses), marks the recommended next build, annotates the next buildable phase of any multi-phase item, and asks the user to reply with the ID in free text — instead of the old ~4-option picker that hid the full picture. The plan-mode note was updated to match the new read-only resolution flow.

## Files/modules touched

**Modified**
- `.claude/commands/build.md`
  - Phase 2, "If I passed nothing" bullet: swapped the `AskUserQuestion` picker for a full-table print with recommended-build marker, per-phase `← next` annotation, an explicit "ask me to reply with the ID" step, and a preserved empty-state branch. (Edit 1)
  - Plan-mode note: replaced the "Phase 2 target-resolution picker (its `AskUserQuestion` is allowed…)" reference with "printing the Phase 2 roadmap table and asking me for the ID (all read-only, allowed before `ExitPlanMode`)". (Edit 2)
  - Phase 0 version bump: `Using build v3 by Josué Clément` → `Using build v4 by Josué Clément`, reflecting the behavior change (follows FEATURE-004's precedent of bumping the version when `/build` changes). (Added post-plan at user request.)
- `docs/roadmap.md` — FEATURE-005 status `TODO` → `DONE`.
- `docs/plan/FEATURE-005.md` — status `TODO` → `DONE`.

**Created**
- `docs/done/FEATURE-005.md` (this file).

## Deviations & follow-ups

- **One addition beyond the plan (at user request):** bumped the Phase 0 version line `v3` → `v4` to reflect the changed `/build` behavior. The plan's two scoped edits landed exactly as written; the `$ARGUMENTS`-passed path, ID resolution, buildable-status gate, and Phases 3–4 were otherwise left untouched (AC7).
- **CRLF / line-ending recommendation (no action taken, per the `dev-workflow` skill):** the working tree carries repo-wide line-ending noise — ~46 files show pure exec-bit mode flips (`100644`→`100755`, from `core.fileMode=true` on the `/mnt/c` WSL mount) and 5 files show CRLF↔LF churn (`build.md`, `dev-workflow/SKILL.md`, `docs/done/FEATURE-004.md`, `docs/plan/FEATURE-004.md`, `docs/roadmap.md`). `build.md` is CRLF in the working tree; my edits preserved its CRLF convention (all 56 lines CRLF, no mixed endings). Recommend the user normalize line endings (the repo's own `git-repo-hygiene` skill / a `.gitattributes` `* text=auto eol=lf` rule + `git add --renormalize`) and consider `git config core.fileMode false`. **This build took no action on either.**
- **Build ran on a dirty tree by explicit user authorization** (the clean-tree precondition was waived for this run). The suggested commit should stage only the FEATURE-005 files, leaving the pre-existing mode/CRLF noise for the user to handle separately.
- **Out-of-scope observation (not addressed):** FEATURE-003 (`git-repo-hygiene` skill) is marked `IN PROGRESS` in the roadmap but was already committed (`3a7863e`) and merged (`85b4044`); it is missing its `docs/done/FEATURE-003.md` completion doc and a `DONE` status flip. Flagged for a future closeout — not part of this dev.

## Build/test evidence

This is a prompt/documentation change with **no build step and no test suite** — DoD criteria 1–2 are satisfied by inspection and dry-run:

- **Content diff verified** with `git diff --ignore-all-space -- .claude/commands/build.md`: exactly the two intended edits, no incidental changes.
- **Dry-run against the current `docs/roadmap.md`:** the new bullet's output is producible — the table has a topmost `IN PROGRESS` item (FEATURE-003) to mark as recommended, and the multi-phase items (FEATURE-001/002, all phases `DONE`) correctly need no `← next` marker.
- **Plan-mode note re-read:** internally consistent, no dangling picker reference.
- **Grep confirmed** the only references to the old no-arg behavior were `build.md` lines 27 & 44 (the `interview.md` `AskUserQuestion` hits are that command's own unrelated usage, out of scope).

All 7 acceptance criteria met.
