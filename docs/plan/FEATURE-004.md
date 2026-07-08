# FEATURE-004 — `/build` end-of-build documentation freshness sweep

**Status:** DONE

## Objective

Add an end-of-build **documentation freshness sweep** to `/build`: after a dev meets the
Definition of Done (roadmap + `docs/done/<ID>.md` updated) and *before* the suggested commit
message is printed, always ask — with concrete, pre-analyzed candidates — whether to refresh
README / CLAUDE.md / other prose docs the change may have made stale, folding any accepted
edits into the dev's own commit.

## Context

`/build` (`v2`) is a thin command that defers all end-of-build substance to the
`dev-workflow` skill (`v1`). Today the end sequence is: roadmap + plan statuses updated →
`docs/done/<ID>.md` written (lands in the dev's own commit) → print progress table → print
suggested commit message → pause for the user to commit. Nothing prompts the user to refresh
user-facing documentation, so README / CLAUDE.md / other prose docs can silently fall out of
date as behavior changes. This item adds that safety-net prompt.

The step must be **generic** (works against whatever project `/build` runs in) and
**non-blocking** (a prompt, not a new Definition-of-Done gate). `/interview` is unaffected —
it runs only the planning flow, which never triggers the sweep.

## Design decisions (validated in the interview)

- **Home (thin-command / fat-skill pattern):** define the behavior in `dev-workflow/SKILL.md`
  as a **build-flow-only** step; `build.md` Phase 4 gets a one-line step that defers to it.
- **When it fires:** always — every build ends with the question, even when nothing looks
  stale (informed safety net).
- **Prompt content:** the agent inspects the dev's changes and presents **concrete, analyzed
  candidates** (specific files/sections with a one-line reason each), plus a "none / skip"
  option; if nothing looks stale, it says so and recommends skip.
- **Scope of the sweep:** README files; CLAUDE.md / agent-instruction files (`AGENTS.md`,
  etc.); other prose docs (`CHANGELOG.md`, `CONTRIBUTING.md`, a docs site, other human-facing
  files under `docs/`). **Excluded:** the workflow's own tracked artifacts
  (`docs/roadmap.md`, `docs/plan/`, `docs/done/` — already handled by the DoD) and the
  `.claude` skill/command prompt files.
- **Commit placement:** accepted doc edits land in **this dev's own commit** (alongside the
  completion doc); the printed suggested commit message is extended to mention them.
- **Non-blocking:** never hold a dev's `DONE` status on the sweep.

## Scope

**In scope** — edit two files:

### 1. `.claude/skills/dev-workflow/SKILL.md`
- Bump version line: `**Version: dev-workflow v1.**` → `**Version: dev-workflow v2.**`.
- Add a new section immediately **after** `## Definition of Done` (it references DoD
  criteria 4–5 and sits before the progress-table/commit-message step). Proposed wording:

  ```markdown
  ## Documentation freshness sweep (build flow)

  **Build flow only** — the planning flow never runs this. Once a dev has met the Definition
  of Done (roadmap + plan statuses updated, `docs/done/<ID>.md` written) and **before** you
  print the progress table and the suggested commit message, run a documentation freshness
  sweep so user-facing docs don't silently fall out of date.

  - **Scan the dev's changes** and identify project/user-facing documentation the change may
    have made stale, across: **README** files; **CLAUDE.md** / agent-instruction files
    (`AGENTS.md`, etc.); and **other prose docs** (e.g. `CHANGELOG.md`, `CONTRIBUTING.md`, a
    docs site, other human-facing files under `docs/`). **Exclude the workflow's own tracked
    artifacts** — `docs/roadmap.md`, `docs/plan/`, `docs/done/` are already handled by the
    Definition of Done — and do not treat skill/command prompt files as sweep targets.
  - **Always ask, with concrete candidates.** Surface one `AskUserQuestion` every time (even
    when nothing looks stale). Offer the specific files/sections you recommend updating — one
    option each, with a one-line reason (e.g. "README 'Usage' still shows the old flag") —
    plus a **"none / skip"** option. If the sweep found nothing, say so and recommend skip.
    Never edit docs without asking.
  - **Accepted edits land in this dev's own commit.** If the user selects targets, make those
    edits now; they ride in the same commit as the code and the completion doc, and you extend
    the suggested commit message to mention them. If the user skips, proceed unchanged.
  - **Non-blocking.** This is a safety-net prompt, not a sixth Definition-of-Done criterion —
    never hold a dev's `DONE` status on it.
  ```

### 2. `.claude/commands/build.md`
- Bump Phase 0 line: `Using build v2 by Josué Clément` → `Using build v3 by Josué Clément`.
- In Phase 4, insert a new step between the current step 3 (Definition of Done) and step 4
  (print table + commit message), and renumber. Proposed wording:

  ```markdown
  4. Run the skill's **documentation freshness sweep**: before printing the commit message,
     always ask (with concrete, analyzed candidates) whether to refresh any README /
     CLAUDE.md / other prose docs this dev may have made stale; fold any edits you accept into
     this dev's own commit. (Per the skill — build flow only; non-blocking.)
  5. Print the multi-phase progress table (if applicable) and the suggested commit message,
     then — per the skill — **pause and wait for me to commit** before starting the next dev.
  ```
- Optional minor touch: extend the frontmatter `description:` to mention the doc-freshness
  prompt. Low priority; keep the scope tight.

**Out of scope:** changing the Definition of Done into a gate; touching `/interview`; the
`.claude` skill/command files as sweep *targets*; any code beyond these two markdown files.

## Acceptance criteria (feature-level)

1. `dev-workflow/SKILL.md` has a "Documentation freshness sweep (build flow)" section after
   the Definition of Done specifying: build-flow-only; fires after DoD 4–5 and before the
   progress table/commit message; scope = README + CLAUDE.md/agent-instruction + other prose
   docs, excluding roadmap/plan/done and excluding skill/command files; always-ask with
   concrete analyzed candidates + a skip option; accepted edits fold into the dev's own commit
   with the commit message extended; explicitly non-blocking.
2. `build.md` Phase 4 has a new step invoking that sweep in the correct position, referencing
   the skill (not duplicating its substance).
3. Version markers bumped: build `v2`→`v3`, dev-workflow `v1`→`v2`.
4. The step is generic — no hardcoded repo-specific filenames beyond illustrative examples.
5. `/interview` behavior is unchanged (no wording implies the planning flow runs the sweep).
6. Both files remain well-formed markdown.

## Implementation notes (single-phase)

- **Branch (at `/build` time):** `feature/feature-004-doc-freshness-sweep`, cut from the
  current `HEAD` per the skill's Version Control rules.
- **No build/test suite** — prompt/config markdown. DoD criteria 1–2 are satisfied by the
  no-build/no-test equivalent (well-formed markdown, verified by inspection/dry-run); criteria
  3–5 apply normally.
- **Verification:** re-read both edited files and confirm the six acceptance criteria hold,
  version markers are bumped, the new skill section sits after the Definition of Done,
  `build.md` Phase 4 references it in the right position without duplicating it, no
  `/interview` wording implies the sweep runs during planning, and the prompt text is generic.
