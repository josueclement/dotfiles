# FEATURE-002 — Interview-ecosystem improvements

**Status:** IN PROGRESS

## Objective

Apply six behavioral changes to the `/interview` → `/build` → `dev-workflow` ecosystem so that:
`/interview` only ever plans (never builds), all git work happens from the user's current HEAD (including a
detached HEAD), completed devs leave a durable "what has been done" record, code reviews are first-class
tracked work, and CRLF noise is never touched — only flagged.

## Context

The ecosystem was delivered as FEATURE-001 (a `dev-workflow` skill as the single source of truth, an
`/interview` command that plans and optionally executes, and a `/build` executor). This feature refines that
machinery. The `dev-workflow` skill is the shared contract both commands defer to, so it is edited first
(PHASE01) and the two commands follow (PHASE02, PHASE03).

## Design decisions (validated in the interview)

1. **`/interview` is planning-only.** The v6 Phase-4 execute-now/stop-after-plans timing question is removed
   entirely; interview always writes roadmap row(s) + plan file(s) at `TODO` and stops. `/build` is the sole
   executor. (Reverses FEATURE-001-PHASE02's timing choice.)
2. **Branch from current HEAD**, never the default branch — the dev branch is cut from the user's current
   position (branch tip or detached commit). Detached HEAD is accepted (not a stop). The clean-tree and
   branch-name-collision hard-stops are kept.
3. **Completion docs** at `docs/done/<ID>.md` — one per dev (phase-suffixed for phases), containing: summary
   of what changed · files/modules touched · deviations from plan + follow-ups (incl. any CRLF
   recommendation) · build/test evidence. Committed together with the dev; required by the Definition of Done.
4. **Code reviews are tracked work.** New work-item type `CODE-REVIEW-NNN` (own per-type counter). Each
   finding becomes a PHASE, ordered highest-severity first (Critical → Low), severity recorded in the phase
   title and plan. Branch prefix `review/`; commit scope `fix(CODE-REVIEW-NNN): … (PHASE0N)`. `/interview`
   recognizes a code review by inference and confirms before allocating the ID.
5. **CRLF / line endings** — never fixed as part of a dev, never raised as an interview question; if noticed,
   emit a one-line recommendation only (in the completion doc's follow-ups and/or console).
6. **Version the skill** — introduce `dev-workflow v1`. Bump the commands: `/interview` v6 → v7,
   `/build` v1 → v2.

## Scope

- Edit `.claude/skills/dev-workflow/SKILL.md` (PHASE01).
- Edit `.claude/commands/interview.md` — v6 → v7 (PHASE02).
- Edit `.claude/commands/build.md` — v1 → v2 (PHASE03).
- Bootstrap `docs/done/` and land each dev's completion record there (dogfooding the new convention).

## Acceptance criteria (feature-level)

1. `/interview` no longer offers or performs execution — it always ends by writing plans and pointing at
   `/build`; no delivery-timing question remains.
2. Both commands and the skill branch from current HEAD (never the default branch) and accept a detached
   HEAD, while retaining the clean-tree and branch-collision stops.
3. `CODE-REVIEW-NNN` is a recognized work-item type across the skill and both commands, with `review/`
   branches, severity-ordered findings-as-phases, and the documented commit scope.
4. A completion doc `docs/done/<ID>.md` is a Definition-of-Done requirement, described in the skill and
   produced by the build flow; `docs/done/` exists.
5. The CRLF rule (never fix, never ask, recommend-only) is stated in the skill and honored by both commands.
6. Version strings are updated: `dev-workflow v1`, `Using interview v7 by Josué Clément`,
   `Using build v2 by Josué Clément`; no `interview v6` / `build v1` remain.

Because these are prompt/config (markdown) changes with no build or test suite, Definition-of-Done criteria
1–2 are met by the no-build/no-test equivalent: each file is well-formed and its criteria are verified by
inspection and dry-run.

## Phases

### PHASE01 — `dev-workflow` skill overhaul — **DONE**
File: `.claude/skills/dev-workflow/SKILL.md`. Branch: `feature/feature-002-phase01-dev-workflow-skill`.
1. Add version marker `dev-workflow v1`.
2. Two-flows + intro text: planning flow is what `/interview` always runs; build flow is `/build` only.
3. Version Control: base = current HEAD (detached OK), never default branch; keep clean-tree /
   branch-collision stops; planning flow writes on current position; add `review/` prefix + CODE-REVIEW
   commit scope.
4. Work-item identifiers: add `CODE-REVIEW-NNN` and the findings-as-phases / severity-ordering rules.
5. Documentation structure: add `docs/done/` + bootstrap it.
6. New "Completion docs" section (content per decision 3).
7. New "Line endings (CRLF)" rule (decision 5).
8. Definition of Done: add the completion-doc criterion.

### PHASE02 — `/interview` v7 — **DONE**
File: `.claude/commands/interview.md`. Branch: `feature/feature-002-phase02-interview-planning-only`.
1. Version line → `Using interview v7 by Josué Clément`.
2. Description / Role / Mission → planning-only (hand off to `/build`; drop self-implementation).
3. Phase 1 → code-review intake (infer + confirm → `CODE-REVIEW-NNN`); CRLF never a question.
4. Phase 4 rewrite → remove the delivery-timing question; planning flow only; update the plan-mode note.
5. Phase 3 work-item-breakdown wording → include the CODE-REVIEW case.

### PHASE03 — `/build` v2 — **TODO**
File: `.claude/commands/build.md`. Branch: `feature/feature-002-phase03-build-command`.
1. Version line → `Using build v2 by Josué Clément`.
2. Phase 2 resolution → accept `CODE-REVIEW-NNN` / `CODE-REVIEW-NNN-PHASENN`; `review/` example.
3. Phase 3/4 branching → current HEAD (detached OK), never default; keep stops; `review/` prefix; update
   pre-flight and plan-mode note.
4. Build flow → write `docs/done/<ID>.md` on DoD (defer to the skill).
5. CRLF → never fix; recommend-only in the done doc.
