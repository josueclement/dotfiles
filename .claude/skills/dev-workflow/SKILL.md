---
name: dev-workflow
description: Use when planning or implementing tracked work items under this house dev workflow — the roadmap registry (docs/roadmap.md), per-item plan files (docs/plan/<ID>.md), FEATURE-NNN/BUG-NNN work-item IDs, branch naming, the Definition of Done, multi-phase progress reporting, and sub-agent delegation. Loaded by the /interview and /build commands; apply it whenever allocating a work-item ID, writing a plan file, branching for a dev, or reporting a dev as done.
---

# Dev workflow — house planning, tracking & version-control standards

These standards govern *how* tracked development work is planned, recorded, and delivered. They are shared by `/interview` (which plans work, and optionally executes it) and `/build` (which executes a previously planned item). If the existing codebase already enforces its own conventions (documentation layout, branch naming, an existing roadmap or work-item ID scheme, etc.), those take precedence over the defaults here — confirm any such conventions before allocating IDs or creating branches.

**A "dev" is one deliverable unit of work — a single-phase item, or one phase of a multi-phase item.** Each dev gets its own branch, its own roadmap status update, and its own suggested commit message.

## Two flows: planning vs building

This workflow separates *planning* from *building*, so plans can be produced now and implemented later.

- **Planning flow** — produce the artifacts without implementing. Write the roadmap row(s) and plan file(s) with status `TODO` on the project's **base branch** (its default branch); create **no** dev branch. The plan file is the durable contract a later build consumes. `/interview` runs this flow when the user chooses to stop after plans.
- **Build flow** — implement a dev. Create its branch from the base branch **first** (see Version Control), then implement. Any planning artifacts not already committed (roadmap row, plan file) are the first change on that branch; if the plan file already exists on the base branch (planned earlier), the first change on the branch is flipping that item/phase status to `IN PROGRESS`. `/interview` runs this flow when the user chooses to execute now; `/build` always runs it against an already-planned item.

## Planning & Documentation

Track every dev through a stable work-item ID, a persistent roadmap, and one plan file per item.

### Documentation structure (at the solution root)

- `docs/roadmap.md` — the single, persistent registry of **every** work item across the whole project (not just the current task).
- `docs/plan/` — one markdown plan file per work item, holding the full plan details.
- **If this structure does not exist yet** — new project, or the first tracked work item in an existing codebase — create `docs/roadmap.md` (an empty table with the four columns below) and `docs/plan/` before allocating the first ID, after confirming the project doesn't already track work items elsewhere.

### Work-item identifiers

Every unit of work gets a stable ID before anything else happens:

- **Format:** `FEATURE-NNN` or `BUG-NNN`, where `NNN` is a 3-digit, zero-padded number (`001`, `002`, …).
- **Numbering is per type:** features and bugs each have their own counter (`FEATURE-001`, `FEATURE-002`, `BUG-001`, …). Allocate by taking the highest `NNN` of that type in the roadmap's **ID column** and adding 1 — ignore phase rows and IDs mentioned in status or notes text, and never reuse a number (including `ABANDONED` ones). Also glance at `docs/plan/` for stray plan files whose ID isn't in the roadmap.
- **Only `FEATURE` and `BUG`** types are used.
- **Multi-phase items:** when an item is large enough to need several phases, suffix the phase as `-PHASENN` (2-digit): `FEATURE-001-PHASE01`, `FEATURE-001-PHASE02`, … The base item keeps its un-suffixed ID (`FEATURE-001`).
- If the existing project already uses its own roadmap or ID scheme, follow that instead of this one.

### `roadmap.md` — summary only

The roadmap holds **only** a summary table — never plan details. Columns: **ID · Title · Status · Plan** (the path to the item's plan file).

- **Status vocabulary:** `TODO`, `IN PROGRESS`, `DONE`, `ABANDONED`.
- For a multi-phase item, list the item on one row, then one indented `- PHASENN` row per phase, each carrying its own status; the phase rows reference the item's plan file as `(in FEATURE-001.md)`.
- **Keep it current — update the roadmap after every dev** so each item/phase status reflects reality. Add a new item's row when it is created: in the **planning flow** on the base branch; in the **build flow** as the first change on the dev's branch (see Version Control).

```markdown
| ID          | Title               | Status      | Plan                     |
|-------------|---------------------|-------------|--------------------------|
| FEATURE-001 | User authentication | IN PROGRESS | docs/plan/FEATURE-001.md |
| - PHASE01   | Login flow          | DONE        | (in FEATURE-001.md)      |
| - PHASE02   | OAuth providers     | IN PROGRESS | (in FEATURE-001.md)      |
| BUG-001     | Fix token refresh   | DONE        | docs/plan/BUG-001.md     |
```

### Plan files — `docs/plan/<ID>.md`

One file per work item, named after its base ID (`docs/plan/FEATURE-001.md` — no phase suffix in the filename). It contains the full plan, proportional to the work: objective, scope, design, acceptance criteria, and — for multi-phase items — a dedicated section per phase with that phase's steps and status. The roadmap's **Plan** column always points to this file.

### Abandoning or changing direction

If you decide to drop a dev, or to do it a different way, **do not delete its row.** Set its status to `ABANDONED` and, in the status cell, record a short reason and — if applicable — the ID of the item that replaces it. Mirror the same note at the top of the item's plan file.

```markdown
| FEATURE-004 | Custom cache layer | ABANDONED — superseded by a simpler approach; replaced by FEATURE-009 | docs/plan/FEATURE-004.md |
```

## Definition of Done

A dev is **done** only when all of the following hold:

1. The full build succeeds with zero warnings.
2. The entire test suite passes — including the new tests the plan's acceptance criteria call for.
3. Every acceptance criterion in the plan file is met.
4. The roadmap and plan file statuses are updated.

Only after all four may you print the progress table (multi-phase items) and the suggested commit message. Never mark an item `DONE` — or present it as finished — with a failing build or failing tests; report the failure instead.

(For work with no build step or test suite — e.g. documentation or prompt/config changes — criteria 1 and 2 are satisfied by the applicable equivalent: the artifact is well-formed and its acceptance criteria are verified by inspection or a dry run. State explicitly that there was nothing to build or test.)

## Version Control

- **Never commit changes yourself** — leave all commits to the user. Creating the branch (below) is the only git action you perform **that modifies repository state**; read-only commands (`git status`, `git log`, `git diff`, `git branch --list`) are always allowed — use them to verify state before branching.
- **Before starting any dev (build flow), create its branch from the project's default branch** (confirm the base branch first — do not assume `main`; some repos use `master` or another branch), named from the ID under a category folder — `feature/` for features, `bugfix/` for bugs — followed by the ID lowercased and hyphenated: `git switch -c feature/feature-001-<short-slug>` (and `git switch -c bugfix/bug-001-<short-slug>` for a bug). For a phase of a multi-phase item, include the phase: `git switch -c feature/feature-001-phase01-<short-slug>` (one branch per phase). First verify the working tree is clean (`git status --porcelain` prints nothing). If the tree is dirty, a branch with that name already exists, or the project has no git repository yet — stop and ask the user before proceeding.
- **In the planning flow, do not create a dev branch:** the roadmap row(s) and plan file(s) are written on the base branch. Branching is deferred to whoever builds the item later.
- **Branch first, then plan (build flow only):** on the new branch, the first change is the item's roadmap row + plan file if they don't yet exist, or — if they were written earlier by the planning flow — flipping the item/phase status to `IN PROGRESS`. Never leave a build's first change on the previous branch or the default branch.
- **After each dev, write a suggested commit message to the console** (do **not** run it): a short title, a blank line, then a brief bulleted description. Use conventional-commit style with the ID scoped in; **the scope is always the base ID** — `feat(FEATURE-001): …` for features, `fix(BUG-001): …` for bugs — and for a phase, the phase goes in the title (`feat(FEATURE-001): add login flow (PHASE01)`), never in the scope.
- **Multi-phase items: pause between phases.** After printing a phase's progress table and commit message, stop and wait for the user to commit before creating the next phase's branch, so each phase's changes land on their own branch.

### Multi-phase progress reporting

When a phase of a multi-phase item completes, **before** the commit message, print to the console a **status table of all phases** of that item — each phase with its status, the immediate next phase marked `TODO (next)` — so it's clear what is done, what's next, and what remains. Then print the commit title and description **after** the table. (Single-phase devs skip the table and just get the commit message.) When the final phase completes, no row carries the `(next)` marker — all phases show `DONE` — and the item's own roadmap row flips to `DONE` in the same update.

```
FEATURE-001 — User authentication

| Phase   | Title           | Status      |
|---------|-----------------|-------------|
| PHASE01 | Login flow      | DONE        |
| PHASE02 | OAuth providers | TODO (next) |
| PHASE03 | Two-factor auth | TODO        |

Commit title:
  feat(FEATURE-001): add login flow (PHASE01)

Commit description:
  - email/password login with validation
  - session cookie issuance
  - unit tests for the auth service
```

## Sub-Agent Delegation
- For large efforts that decompose cleanly into independent units, delegate those units to sub-agents, each spawned with its own fresh context. This keeps every sub-agent focused on a single, well-scoped portion and improves the quality of each part.
- Only delegate when the split is genuinely clean (minimal cross-dependencies and clear interfaces between parts). If the work is tightly coupled, keep it in a single context to preserve coherence rather than forcing an artificial split.
- When you do delegate: (1) include in each sub-agent's prompt the interface contract it must honor, the relevant validated requirements from the plan, and the exact target paths; (2) sub-agents write code and tests only — all git actions, roadmap/plan updates, and commit messages remain yours; (3) after integrating sub-agent output, run the full build and test suite yourself before declaring the dev done.
