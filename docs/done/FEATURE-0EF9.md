# FEATURE-0EF9 — Random-hex work-item IDs — DONE

## Summary
Replaced the sequential per-type counter in the house work-item ID scheme
(`FEATURE-NNN` / `BUG-NNN` / `CODE-REVIEW-NNN`, 3-digit zero-padded) with a
**random 4-digit uppercase hex value** (`FEATURE-HHHH`, e.g. `FEATURE-3A7F`).
Sequential numbering caused ID collisions when multiple devs allocated "the next
number" concurrently on the same repo; random IDs remove the shared counter so
independently created IDs don't clash on merge. Existing IDs were **not migrated**
— old `NNN` IDs and new `HHHH` IDs coexist. Ordering now comes from roadmap **row
position** (append at bottom), not from the ID.

This item dogfooded the new scheme: its own ID (`FEATURE-0EF9`) was allocated via
the very shell rule the change introduces (`printf '%04X' $((RANDOM % 65536))`),
checked unique against the existing `FEATURE-001…010`.

## Files/modules touched
**Created**
- `docs/plan/FEATURE-0EF9.md` — the plan (from the validated interview summary).
- `docs/done/FEATURE-0EF9.md` — this record.

**Modified**
- `.claude/skills/dev-workflow/SKILL.md` — version `v2 → v3`; rewrote the
  "Work-item identifiers" allocation rules (random hex, shell-generated, per-type
  uniqueness + regenerate-on-collision, phases stay sequential); added the
  "Ordering — the row order is the order / append at bottom" rule; converted
  every example table, branch-name, commit-scope and progress-report sample to
  hex; updated the frontmatter description.
- `.claude/commands/interview.md` — announce line `v7 → v8`; `CODE-REVIEW-NNN`
  → `CODE-REVIEW-HHHH` (3 occurrences).
- `.claude/commands/build.md` — announce line `v4 → v5`; `argument-hint` and the
  Phase-2 base/phase-ID examples converted to hex.
- `.claude/skills/dotnet-release/SKILL.md` — `FEATURE-NNN` → `FEATURE-HHHH`
  (2 occurrences) so the docs stay consistent (beyond the original draft, which
  named only interview/build/dev-workflow).
- `docs/roadmap.md` — appended the `FEATURE-0EF9` row (existing rows untouched).

## Deviations & follow-ups
- **No deviations from the plan.** All six acceptance criteria met.
- **Build-flow reconciliation:** the interview validated the plan (Phase 3) but
  `/build` was invoked before Phase 4 persisted the artifacts, so the roadmap row
  and plan file were written as the first change on this branch at `IN PROGRESS`
  (per the skill's build flow for uncommitted planning artifacts) — not invented.
- **Version-bump scope:** per user decision, all three files were bumped
  (dev-workflow v3, interview v8, build v5), rather than only the skill.
- **Line endings (CRLF):** none noticed; all touched files are LF — no action.
- **Follow-up (out of repo):** the personal memory note `interview-commands.md`
  still references the old versions (interview v6 / dev-workflow v1) and the
  sequential scheme; it should be refreshed to interview v8 / dev-workflow v3 and
  the random-hex scheme. Not part of this commit (it lives outside the repo).

## Build/test evidence
Prompt/documentation-only change — **nothing to compile and no test suite**
(DoD criteria 1–2 satisfied by the no-build/no-test equivalent). Verified by:
- `grep -rn "NNN" .claude/skills .claude/commands` → empty (AC1).
- Allocation section rewritten as specified; old "highest + 1 / never reuse"
  text removed (AC2).
- `grep -rnE '(FEATURE|BUG|CODE-REVIEW)-[0-9]{3}'` over the 4 changed files →
  empty; no sequential example IDs remain (AC3).
- Version/announce lines confirmed: `dev-workflow v3`, `interview v8`, `build v5`
  (AC4).
- `docs/roadmap.md` still shows all 10 original `FEATURE-001…010` rows unchanged
  (AC5).
- Markdown inspected: frontmatter intact, hex example tables aligned and
  well-formed (AC6).
