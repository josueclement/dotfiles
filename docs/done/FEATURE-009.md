# FEATURE-009 — dotnet-release TFM normalization — DONE

_Single-phase item. Built with `/build FEATURE-009` on branch `feature/feature-009-dotnet-release-tfm-normalization`. Documentation/prompt change — no build/test step._

## Summary

Gave the `dotnet-release` skill a **target-framework check/update** step and reconciled
`dotnet-solution-setup` to the same policy. On release, the packable library's TFM set is normalized:
`netstandard*` is preserved, and the plain-`net` portion becomes **exactly `net8.0;net10.0`** (the two
currently-supported LTS releases) — adding whichever is missing, replacing any `net` older than 8, and
collapsing any other `net` (e.g. `net9.0`) to the pair. The change is **proposed and confirmed before
writing** (a departure from the skill's usual auto-edit model, because a TFM change moves the
compatibility surface), and logged `old → new` in `RELEASENOTES.md` *Compatibility* plus the README
"supported target frameworks" line. Edge cases: a `netstandard`-only library is left untouched;
platform-specific TFMs (`-windows`, …) are preserved and only **warned** about if older than net8.

## Files/modules touched

**Modified:**
- `.claude/skills/dotnet-release/SKILL.md` — new **Target frameworks** step at the top of *In-repo
  edits* (partition/normalize algorithm, propose→confirm, singular→plural, case table, RELEASENOTES +
  README wiring); reworded the *In-repo edits* summary sentence; added a `dotnet-solution-setup`
  cross-reference; added a *Common mistakes* row for an unlogged TFM change.
- `.claude/skills/dotnet-release/templates/RELEASE.md` — new pre-release checkbox for the
  net8.0+net10.0 policy, next to "build across all TFMs".
- `.claude/skills/dotnet-solution-setup/SKILL.md` — *Target frameworks* section reconciled: library
  multi-target example `netstandard2.0;net10.0` → `netstandard2.0;net8.0;net10.0`, added the
  net8.0+net10.0 LTS framing and a back-reference to `dotnet-release` (bidirectional cross-ref);
  app/test single-target `net10.0` defaults left unchanged.

**Workflow artifacts:** `docs/roadmap.md` (status → DONE), `docs/plan/FEATURE-009.md` (status → DONE),
`docs/done/FEATURE-009.md` (this file).

## Deviations & follow-ups

- **No deviations** from the approved plan; all 10 design decisions implemented as specified.
- Per plan scope item 3 ("adjust the *Common mistakes* row **if it hard-codes net10.0-only**"), the
  `dotnet-solution-setup` mistakes row ("Library scaffolded as `net10.0` …") was **left unchanged** —
  it steers libraries to `netstandard2.0` by default and does not contradict the net8+net10 policy, so
  no edit was warranted.
- Frontmatter left untouched (AC5). Optional future follow-up: the `dotnet-release` `description` could
  add "target frameworks" for discovery, but the plan scoped frontmatter out — not changed.
- **Line endings:** files are LF; no CRLF churn observed — nothing to recommend.

## Build/test evidence

Documentation/prompt change to skill files — nothing to compile or test (DoD criteria 1–2 satisfied by
well-formed artifacts, verified by inspection). Verification performed:
- `grep` confirmed the stale `netstandard2.0;net10.0` example is gone from `dotnet-solution-setup` and
  the new example/LTS framing/back-reference are present; the `dotnet-release` TFM step, cross-reference,
  mistakes row, and the RELEASE.md checkbox are all present.
- Walked the normalization algorithm against the plan's decision-3 case table plus both edge cases —
  `net8.0`→`net8.0;net10.0`, `netstandard2.0;net8.0`→`netstandard2.0;net8.0;net10.0`,
  `net6.0`/`net9.0`/`net10.0`→`net8.0;net10.0`, `netstandard2.0`→unchanged,
  `net6.0-windows`→untouched + warning — each matches the documented output.
- Confirmed bidirectional cross-references resolve and house terse style / frontmatter preserved.
