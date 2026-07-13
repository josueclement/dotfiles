# FEATURE-010 — dotnet-release MSI profile generation (apps)

**Status:** DONE

_Single-phase item. Planned via `/interview` (v7); build with `/build FEATURE-010`. Documentation/prompt change to the `dotnet-release` skill — no build/test step._

## Objective

Extend the `dotnet-release` skill so that, when releasing a .NET **app** (never a library), it
proactively **suggests generating a home-made MSI profile** JSON file (consumed downstream by
WixSharp) and creates it with the correct GUID semantics:

- `upgradeCode` = the app's stable identity GUID — **the same across every version**.
- `productId` = a **new GUID for every version**.

## Context

The `dotnet-release` skill (FEATURE-007) drives a release end-to-end and already distinguishes a
non-packable **app** from a packable **library** (see its *App vs library* section). For apps the
user maintains MSI profile JSONs that WixSharp turns into an installer, but that step is manual and
lives outside the skill. This item folds an **opt-in profile-generation offer** into the app-release
path.

Grounded in four real examples at `/mnt/c/Temp/Draw.msiprofile*.json`, which fix the exact field
set, ordering, and the upgradeCode-stable / productId-per-version contract (`upgradeCode`
= `c5420e69-…` in all four; `productId` differs per file).

## Design decisions (validated in the interview)

1. **Apps only, opt-in.** The offer fires only in the skill's app branch (non-packable, no
   `PackageId`), **after** the version bump / release notes and **before** the printed runbook. The
   skill *suggests*; the user says yes/no. Never for libraries.
2. **Location:** `<solution-root>/msiProfiles/` — create the dir if missing. Profiles are
   **committed** to the repo (the `upgradeCode` must persist across releases).
3. **Filenames:** **always version-suffixed** — `<AppName>.msiprofile.<version>.json`, including the
   first. Existing profiles for *this* app are detected by glob `<AppName>.msiprofile.*.json` (an
   empty dir counts as "first").
4. **First profile** (none exist for this app): generate **two** GUIDs (`upgradeCode` + `productId`)
   via a local generator. Fill fields by **auto-detecting the derivable ones** and
   **confirming/asking the rest**:
   - Auto-detect: `appName` + `msiFilename` (csproj / app name); `releasePath`
     (`<appdir>/bin/Release/<tfm>`); `productIcon` (`Assets/*.ico`); `manufacturer`
     (csproj `<Authors>`/`<Company>`); `version` (the release `X.Y.Z`). If the icon or a Release
     build isn't present yet → leave the field blank and **warn**.
   - Confirm/ask, with defaults: `scope` (=`PerMachine`), `installPath` (=`%ProgramFiles%\<App>`),
     `compression` (=`High`), `outputPath`, `shortcuts` (default Desktop + ProgramMenu, targeting
     `[INSTALLDIR]\<App>.exe`).
5. **Subsequent profile** (≥1 exists): **clone the most-recent** `<AppName>.msiprofile.*.json`
   (highest version), keep `upgradeCode` and all config verbatim, generate a **new `productId`**
   only, set `version` = release version. Re-point `releasePath` **only if** this release changed the
   app's TFM. If the newest existing profile is malformed → stop and ask.
6. **GUID generation — the skill runs it.** A local generator (`uuidgen`, or
   `cat /proc/sys/kernel/random/uuid`, or PowerShell `[guid]::NewGuid()`). Documented as the **one
   exception** to the skill's "prints, never runs" boundary: GUID generation is local and reversible,
   unlike the outward pack/tag/push commands. GUIDs are **never fabricated** by the model.
7. **Scope stops at the JSON profile** — no `.msi` build, no WixSharp invocation.
8. **Template bundled** — `templates/msiprofile.template.json`, the single source of truth for the
   field set/order.

## Scope

### 1. `.claude/skills/dotnet-release/SKILL.md` — new section "MSI profiles (apps only)"

- Place near the existing **"App vs library"** section. Specify decisions 1–7: the apps-only opt-in
  offer and its placement in the flow; the `<solution-root>/msiProfiles/` location + committed;
  always-version-suffixed filenames + per-app glob detection; first-profile (two GUIDs,
  auto-detect + confirm) vs subsequent-profile (clone latest, new `productId`, reuse `upgradeCode`,
  re-point `releasePath` on TFM change); local GUID generation as the documented boundary exception;
  JSON-only scope.
- Include a **field reference** (name → meaning → source) mirroring the template.
- **"App vs library"** section: one sentence pointing to the new MSI-profile step for apps.
- **"Common mistakes"** rows: generating a profile for a *library*; regenerating `upgradeCode` on a
  subsequent release (must reuse the existing one); fabricating GUIDs instead of running a generator;
  storing profiles anywhere other than `<solution-root>/msiProfiles/`.
- **Frontmatter `description`:** extend so the trigger covers "generate MSI profiles for apps".

### 2. `.claude/skills/dotnet-release/templates/msiprofile.template.json` — new bundled file

Skeleton with the **exact field set and order** from the examples, using placeholder tokens, plus a
short placeholder legend (in SKILL.md or alongside).

Reference field order (from `/mnt/c/Temp/Draw.msiprofile.1.3.0.json`):
`appName, installPath, releasePath, scope, version, productId, upgradeCode, manufacturer,
productIcon, compression, outputPath, msiFilename, shortcuts[]`, where each shortcut is
`{ shortcutPath, shortcutName, targetPath, iconPath, arguments }`.

Placeholder tokens: `{{APP_NAME}}`, `{{INSTALL_PATH}}`, `{{RELEASE_PATH}}`, `{{SCOPE}}`,
`{{VERSION}}`, `{{PRODUCT_ID}}`, `{{UPGRADE_CODE}}`, `{{MANUFACTURER}}`, `{{PRODUCT_ICON}}`,
`{{COMPRESSION}}`, `{{OUTPUT_PATH}}`, `{{MSI_FILENAME}}`.

## Acceptance criteria

1. `SKILL.md` has an "MSI profiles (apps only)" section covering: apps-only + opt-in offer and its
   placement; `<solution-root>/msiProfiles/` (committed); always-version-suffixed filenames +
   per-app detection; first-vs-subsequent GUID logic (two GUIDs vs new `productId` only, `upgradeCode`
   reused); auto-detect + confirm field population; clone-latest for subsequent; local GUID
   generation (documented boundary exception); JSON-only scope; field reference.
2. `templates/msiprofile.template.json` exists with the exact field set/order from the examples + a
   placeholder legend.
3. The skill's frontmatter `description` mentions MSI-profile generation for apps.
4. "App vs library", "Common mistakes", and cross-references are updated consistently; terse house
   style preserved.
5. A profile the skill would generate for the `Draw` app is structurally byte-compatible with the
   existing `/mnt/c/Temp/Draw.msiprofile.*.json` files (same keys, order, shape).
6. Roadmap + plan-file statuses updated and `docs/done/FEATURE-010.md` written on completion, stating
   this is a documentation/prompt change with no build/test step (verified by inspection).

## Verification (no build/test suite — prompt file)

- Inspect `SKILL.md`: the new section is present, internally consistent, and the description triggers
  on MSI profiles.
- Validate `templates/msiprofile.template.json` parses as JSON after token substitution, and its key
  set/order matches `/mnt/c/Temp/Draw.msiprofile.1.3.0.json`.
- Dry-run trace both paths against the Draw example: (a) empty `msiProfiles/` → two GUIDs, filename
  `Draw.msiprofile.<ver>.json`; (b) existing `Draw.msiprofile.1.3.0.json` → clone, keep `upgradeCode`,
  new `productId`, bumped `version`.

## Notes & follow-ups

- **Decision that overrode my recommendation:** always version-suffixed filenames (I'd suggested
  matching the examples' first-file-unversioned `Draw.msiprofile.json`; the user chose uniform
  versioning for every file).
- Write valid JSON; a non-ASCII `manufacturer` may be preserved as-is (the examples' `\uXXXX`
  escaping is cosmetic, not required).
- Deliberately out of scope: building the `.msi`, invoking WixSharp, re-detecting all fields on
  subsequent profiles (clone-latest instead), and any library-side behaviour.
