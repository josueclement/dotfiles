# FEATURE-001 — Decouple planning from execution in the interview workflow

## Objective

Let `/interview` optionally **stop after producing the roadmap items and full plans** (rather than always executing immediately), and add a companion `/build` command that implements a previously planned item on demand. This is the classic plan/apply split: plan several things up front, review/prioritize, and build them later.

## Context

`/interview` currently runs its questioning and then executes the work immediately (Phase 4). The tracking machinery (`docs/roadmap.md`, `docs/plan/<ID>.md`, `FEATURE-NNN`/`BUG-NNN` IDs, branch naming, Definition of Done) already exists but only as inline prose inside `interview.md`. What was missing: (a) a way to stop at the plan, and (b) a command to run a validated plan later.

This feature dogfoods the very workflow it introduces.

## Design decisions (validated)

- **Timing choice at the end of the interview** — after Phase 3 validation, `/interview` writes all roadmap rows + plan files (`TODO`), then asks "execute now, or stop here and build later?".
- **`/build` runs directly and is plan-mode-aware** — reads the already-validated plan and executes; if launched in plan mode it re-presents the plan for a quick `ExitPlanMode` approval. No redundant re-planning.
- **Shared `dev-workflow` skill** — the Execution Standards move out of `interview.md` into a skill both commands load (single source of truth; avoids the twin-sync problem).
- **Plans on base branch, build branches later** — the planning flow writes artifacts on the base branch (`master`); the build flow creates the dev branch and adds implementation. "Branch first, then plan" now applies only to the build flow.
- **Command name `/build`**; selection by ID arg, else list `TODO`/`IN PROGRESS` and ask.

## Scope

- New skill `.claude/skills/dev-workflow/SKILL.md`.
- Edit `.claude/commands/interview.md` (v5 → v6): load the skill, delete inline Execution Standards, add the timing choice.
- New command `.claude/commands/build.md` + global symlink.
- Housekeeping: correct stale memory; optionally remove dangling `interview-dotnet.md` symlink.

## Acceptance criteria (feature-level)

1. `/interview` (v6) references the `dev-workflow` skill everywhere the old Execution Standards were referenced; no orphaned references to a deleted inline section.
2. `/interview` offers an explicit execute-now / stop-after-plans choice, and the stop path writes `TODO` artifacts on the base branch without creating a dev branch.
3. `/build [<ID>]` resolves a target (by arg, or by listing `TODO`/`IN PROGRESS`), reads the plan, is plan-mode-aware, creates the dev branch, and drives the Definition of Done.
4. The `dev-workflow` skill is the single source of truth for the roadmap/plan/ID/branch/DoD conventions and is loaded by both commands.
5. `/build` is globally available via symlink; the dangling `interview-dotnet.md` symlink is resolved.

---

## PHASE01 — Create the `dev-workflow` skill — **DONE**

Extract the Execution Standards body from `interview.md` into `.claude/skills/dev-workflow/SKILL.md`, generalized (audience is any command/agent, not "the interview"), and add a **"Two flows"** section reconciling the planning-vs-build branching rule.

Steps:
1. Author the skill with frontmatter (`name`, `description`) matching the repo's skill format.
2. Include: Two flows · Planning & Documentation · Definition of Done · Version Control (branch-first split by flow) · Multi-phase progress reporting · Sub-Agent Delegation.

Acceptance: skill file exists, well-formed frontmatter, self-contained, resolvable as `dev-workflow`. (No inline copy is removed yet — that happens in PHASE02, so a transient duplication is expected and acceptable.)

## PHASE02 — Refactor `/interview` to use the skill + add the timing choice (v6) — **DONE**

1. Phase 0: `interview v5` → `interview v6`.
2. Phase 1 step 4: also load the `dev-workflow` skill.
3. Delete the inline `# Execution Standards` section; repoint every remaining reference to "the `dev-workflow` skill."
4. Restructure Phase 4: first step asks execute-now vs stop-after-plans; document both paths (stop = artifacts on base branch, print planning commit message, point to `/build`; execute = existing branch-first flow).

Acceptance: criteria 1 and 2 above met; existing plan-mode paragraph retained.

## PHASE03 — Add `/build` command + wiring — **DONE**

1. New `.claude/commands/build.md` (frontmatter + body: announce version, load skill, resolve target, read plan, pre-flight, plan-mode-aware, branch, implement, DoD, progress table + commit message, pause between phases).
2. Symlink `~/.claude/commands/build.md` → repo file.
3. Correct memory `interview-commands.md`; optionally remove dangling `interview-dotnet.md` symlink.

Acceptance: criteria 3 and 5 above met.
