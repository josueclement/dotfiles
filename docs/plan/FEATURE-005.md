# FEATURE-005 — `/build` prints the roadmap when invoked with no ID

**Status:** TODO
**Type:** FEATURE (single-phase)
**Branch (at build time):** `feature/feature-005-build-roadmap-listing` (cut from current `HEAD`)

## Objective

When `/build` is invoked with **no work-item ID**, print the full `docs/roadmap.md` table to the console so the user can see every item at a glance and reply with the ID to build — instead of the current `AskUserQuestion` picker, which caps at ~4 options and hides the full picture.

## Problem

Today (`.claude/commands/build.md`, Phase 2, no-arg bullet) `/build` with no ID reads the roadmap and immediately calls `AskUserQuestion` to pick a `TODO`/`IN PROGRESS` item. `AskUserQuestion` is limited to ~4 options, so as the roadmap grows it cannot list every buildable item, and it never surfaces the full context (all statuses, phase rows, what's already done).

## Scope

Single-file prompt change to `.claude/commands/build.md`. **No code, no build, no test suite.** Two edits:

### Edit 1 — Phase 2, "If I passed nothing" bullet (currently line 27)

Replace the `AskUserQuestion` picker with:

- Read `docs/roadmap.md` and **print the full roadmap table** to the console — every item **and** phase row, all statuses (`TODO`, `IN PROGRESS`, `DONE`, `ABANDONED`), reproducing the roadmap's columns (ID · Title · Status · Plan).
- **Mark the recommended next build**: the topmost `IN PROGRESS` item, else the topmost `TODO` (annotation/marker, not a new column).
- For a **multi-phase** item, **annotate the next buildable phase** — the first `TODO`/`IN PROGRESS` phase (e.g. `PHASE02 ← next`), consistent with the skill's multi-phase progress reporting.
- Then **stop and ask the user to reply with the ID** to build. **Do not** call `AskUserQuestion` in this path.
- **Empty state preserved:** if there are no `TODO`/`IN PROGRESS` items, still print the table for context, then say there's nothing to build and stop. Keep the existing "roadmap missing → stop and tell me" behavior.

### Edit 2 — Plan-mode note (currently line 44)

Update the read-only resolution list: replace "the Phase 2 target-resolution picker (its `AskUserQuestion` is allowed before `ExitPlanMode`)" with the new flow — reading `docs/roadmap.md`, printing the table, and asking the user for the ID (all read-only, allowed before `ExitPlanMode`).

### Out of scope (leave unchanged)

The `$ARGUMENTS`-passed path, ID resolution, the buildable-status gate, and everything from Phase 3 onward.

## Acceptance criteria

1. `/build` with no ID prints the full roadmap table (all items/phases, all statuses) to the console.
2. The recommended next build is clearly marked (topmost `IN PROGRESS`, else topmost `TODO`).
3. Multi-phase items annotate the next buildable phase (`← next`).
4. The command then asks for the ID in free text and does **not** call `AskUserQuestion` in the no-arg path.
5. No `TODO`/`IN PROGRESS` items → table still printed, then "nothing to build" + stop; missing roadmap → stop and tell the user.
6. The plan-mode note no longer references the removed picker and matches the new flow.
7. The `$ARGUMENTS`-passed path and all later phases are unchanged.

## Verification (no build/test suite — prompt file)

- Dry-run read of the edited Phase 2 bullet against `docs/roadmap.md`: confirm the described output (full table + recommended marker + `← next` for multi-phase) is producible and the empty-state branch is preserved.
- Re-read the plan-mode note for internal consistency (no dangling picker reference).
- Grep confirms no other file references the old behavior (`build.md` lines 27 & 44 were the only references at plan time).

## Notes

- DoD criteria 1–2 (build/tests) are satisfied by inspection/dry-run since this is a prompt file with no build step; criteria 3–5 (acceptance met, roadmap+plan status updated, `docs/done/FEATURE-005.md` written) apply as normal at build time.
