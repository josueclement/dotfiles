# FEATURE-010 — dotnet-release MSI profile generation (apps) — DONE

## Summary

Extended the `dotnet-release` skill so that releasing a .NET **app** (never a library) triggers an
opt-in offer to generate a home-made **MSI profile** JSON (consumed downstream by WixSharp), with the
correct GUID semantics: `upgradeCode` stable across every version, `productId` new per version. Added
a bundled template as the single source of truth for the field set/order.

Documentation/prompt change only — no build step and no test suite (verified by inspection + a JSON
structural check against the grounding examples).

## Files/modules touched

**Created**
- `.claude/skills/dotnet-release/templates/msiprofile.template.json` — MSI-profile skeleton with the
  exact field set/order from the examples, using `{{…}}` placeholder tokens. Valid JSON as-is (tokens
  are quoted string values) and after substitution.

**Modified**
- `.claude/skills/dotnet-release/SKILL.md`:
  - Frontmatter `description` extended to cover generating a WixSharp MSI profile for apps.
  - New **"MSI profiles (apps only)"** section (placed after *App vs library*): apps-only opt-in offer
    and its placement (after version bump/notes, before the runbook); `<solution-root>/msiProfiles/`
    (committed); always-version-suffixed filenames + per-app glob detection; GUID contract; first
    profile (two GUIDs, auto-detect + confirm) vs subsequent (clone latest, new `productId` only,
    reuse `upgradeCode`, re-point `releasePath` only on TFM change, stop-and-ask if malformed); local
    GUID generation as the documented boundary exception; JSON-only scope; a **field reference** table
    and a **placeholder legend**.
  - *Execution boundary* — added the "one exception" note (local GUID generation is run, not printed).
  - *App vs library* — one-sentence pointer to the new MSI-profile step.
  - *Bundled files* — entry for the new template.
  - *Common mistakes* — four new rows (library profile; regenerating `upgradeCode`; fabricating GUIDs;
    storing profiles outside `msiProfiles/`).
- `docs/roadmap.md`, `docs/plan/FEATURE-010.md` — status `TODO` → `IN PROGRESS` → `DONE`.

## Deviations & follow-ups

- **Added one placeholder token beyond the plan's listed 12: `{{TARGET_EXE}}`.** The plan (decision 4
  / scope §2) phrases the shortcut target as `[INSTALLDIR]\<App>.exe`, but every grounding example
  targets `[INSTALLDIR]\Draw.App.exe` — the app's **build-output assembly name** (`Draw.App`), which
  differs from the display `appName` (`Draw`). Substituting `appName`/`msiFilename` would have produced
  `Draw.exe` and broken AC5's byte-compatibility. So the exe is documented as the auto-detected
  assembly/output name and carried by a dedicated `{{TARGET_EXE}}` token. Faithful to the plan's own
  grounding examples, not a redesign — flagged here per scope discipline.
- Placeholder **legend** lives in `SKILL.md` (not inside the template) so the template stays pure,
  comment-free JSON — consistent with the plan's "in SKILL.md or alongside".
- `manufacturer` non-ASCII: the template uses a plain `{{MANUFACTURER}}` placeholder; SKILL.md notes
  the examples' `\uXXXX` escaping is cosmetic and not required.
- No CRLF/line-ending issues observed in the touched files.

## Build/test evidence

No build step or test suite — this is a skill (prompt/doc) change. Verified by inspection and a
structural JSON check:

- `templates/msiprofile.template.json` parses as JSON both before and after token substitution.
- After substituting the `Draw` 1.3.0 values (incl. `TARGET_EXE=Draw.App.exe`), the generated profile
  is **full value-equal** to `/mnt/c/Temp/Draw.msiprofile.1.3.0.json` — identical keys, order, and
  shape (AC5).
- Dry-run traced both paths against the Draw example: (a) empty `msiProfiles/` → two GUIDs, filename
  `Draw.msiprofile.<ver>.json`; (b) existing `Draw.msiprofile.1.3.0.json` → clone latest, keep
  `upgradeCode`, new `productId`, bumped `version`.
- `SKILL.md` reviewed end-to-end: new section present, internally consistent, terse house style
  preserved; frontmatter `description` now triggers on MSI-profile generation for apps.

All six acceptance criteria in `docs/plan/FEATURE-010.md` are met.
