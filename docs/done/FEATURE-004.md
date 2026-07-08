# FEATURE-004 — `/build` end-of-build documentation freshness sweep — DONE

## Summary

Added an end-of-build **documentation freshness sweep** to `/build`. After a dev meets the
Definition of Done (roadmap + plan updated, `docs/done/<ID>.md` written) and before the
suggested commit message is printed, `/build` now always asks — with concrete, pre-analyzed
candidates — whether to refresh user-facing docs (README, CLAUDE.md / agent-instruction
files, other prose docs) the change may have made stale, folding any accepted edits into the
dev's own commit. The behavior is defined once in the `dev-workflow` skill (build-flow-only)
and referenced from the `/build` command, matching the repo's thin-command / detailed-skill
split. It is a non-blocking safety-net prompt, **not** a new Definition-of-Done criterion.

## Files/modules touched

- **Modified** `.claude/skills/dev-workflow/SKILL.md`
  - Version bumped `v1` → `v2`.
  - Added a new `## Documentation freshness sweep (build flow)` section immediately after the
    Definition of Done (and before Line endings): build-flow-only; fires after DoD 4–5 and
    before the progress table/commit message; scope = README + CLAUDE.md/agent-instruction +
    other prose docs, excluding the workflow's own `docs/roadmap.md` · `docs/plan/` ·
    `docs/done/` and excluding skill/command prompt files; always-ask with concrete analyzed
    candidates + a "none / skip" option; accepted edits fold into the dev's own commit with the
    commit message extended; explicitly non-blocking.
- **Modified** `.claude/commands/build.md`
  - Phase 0 version bumped `v2` → `v3`.
  - Phase 4: inserted a new step 4 invoking the skill's documentation freshness sweep (before
    printing the commit message); renumbered the former step 4 (print table + commit message +
    pause) to step 5.
  - Frontmatter `description` extended with "and ends by prompting whether to refresh
    user-facing docs" to stay truthful.
- **Modified** `docs/roadmap.md` — added the `FEATURE-004` row (created during planning,
  carried onto this branch) and flipped it through `IN PROGRESS` to `DONE`.
- **Created** `docs/plan/FEATURE-004.md` — the plan (written during the `/interview` planning
  flow; status flipped to `DONE` here).
- **Created** `docs/done/FEATURE-004.md` — this completion record.

## Deviations & follow-ups

- **No deviations** — implemented exactly as the plan specified.
- **Uncommitted planning artifacts carried onto the branch.** `/build` was invoked before the
  planning commit was made, so `FEATURE-004`'s planning artifacts (roadmap row + plan file)
  were uncommitted. Per the skill's build flow (uncommitted planning artifacts become the
  first change on the dev's branch), they were carried onto `feature/feature-004-doc-freshness-sweep`
  and land in this dev's commit — there is no separate planning commit.
- **Dogfood of the new sweep.** This build ran the newly-added sweep at its tail. Nothing
  in-scope was stale: the root `README.md` is a bare placeholder ("# dotfiles2") that documents
  none of the commands, and there is no `CLAUDE.md` / `CHANGELOG.md` / `CONTRIBUTING.md` in the
  repo; the edited skill/command files are out of scope by design. So the sweep recommended
  skip and no doc edits were made.
- **Line endings:** no CRLF issues observed — `git diff --check` reported no whitespace/EOL
  errors.
- **Out-of-scope follow-ups** (noted, not acted on): `FEATURE-003` still shows `IN PROGRESS`
  in the roadmap although its branch appears merged in history — consider flipping it to `DONE`
  and adding `docs/done/FEATURE-003.md`; likewise `FEATURE-001`'s phases are `DONE` without
  completion records.

## Build/test evidence

No build step or test suite — this is a prompt/config markdown change, so DoD criteria 1–2 are
met by the no-build/no-test equivalent (well-formed markdown, verified by inspection and a dry
run). Verification performed:

- `git diff --check` → clean (no whitespace/CRLF errors).
- Version markers confirmed: `SKILL.md` → `**Version: dev-workflow v2.**`; `build.md` Phase 0
  → `Using build v3 by Josué Clément`.
- New skill section present at `SKILL.md:107`, positioned after the Definition of Done and
  before `## Line endings (CRLF)`; markdown structure (heading + bullet list) intact.
- `build.md` Phase 4 now has step 4 (documentation freshness sweep) and renumbered step 5
  (print table + commit message + pause), in the correct order.
- `.claude/commands/interview.md` was **not** touched, and the new skill section is explicitly
  labelled "Build flow only — the planning flow never runs this," so `/interview` behavior is
  unchanged.
- All six plan acceptance criteria checked and met.
