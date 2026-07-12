# FEATURE-007 — `dotnet-release` skill

**Status:** TODO

_Single-phase item. Planned via `/interview` (v7); build with `/build FEATURE-007`. Build **after** FEATURE-006, which this skill cross-references._

## Objective

Add a reusable Claude Code skill, `dotnet-release`, that drives a new NuGet library-version
release end-to-end: bump `<Version>`, update `<PackageReleaseNotes>`, prepend `RELEASENOTES.md`,
refresh the README (badges + "what's new" callout), apply and document selective NuGet package
updates, then **print** (never execute) the out-of-repo runbook — build/test in Release, merge,
tag, `dotnet pack`, `dotnet nuget push`, post-publish verification.

## Context

The release process is a mature, hand-rolled runbook used across the house .NET repos. Today it
lives only inside those repos (`Enigma.LicenseManager/docs/RELEASE.md`) and in the narration of
their completion records. This skill captures it. A release is the **final phase of a
`FEATURE-NNN` work item** under `dev-workflow` (roadmap → plan → done, doc-freshness sweep,
conventional-commit message, never-commit-myself) — this skill defers to `dev-workflow` for all
of that rather than restating it. This is a documentation/prompt artifact — no build/test step.

**Reference sources (template from these when building):**
- `/home/jo/Dev/Enigma.LicenseManager/docs/RELEASE.md` — the canonical runbook (source for `templates/RELEASE.md`).
- `/home/jo/Dev/Enigma.Cryptography/README.md` — badge markdown + "what's new" callout format.
- `/home/jo/Dev/Enigma.Cryptography/RELEASENOTES.md` — release-notes format (newest-first, `# <Name> vX.Y.Z Release Notes`).
- `/home/jo/Dev/Enigma.Cryptography/src/Enigma.Cryptography/Enigma.Cryptography.csproj` — `<Version>`, `<PackageReleaseNotes>`, and packaging metadata (`PackageId`, README/LICENSE pack, `GeneratePackageOnBuild` **absent** = off).
- `/home/jo/Dev/Enigma.Cryptography/docs/done/FEATURE-005.md` and
  `/home/jo/Dev/Enigma.LicenseManager/docs/done/FEATURE-004-PHASE04.md` — narrate the exact per-release edits.

## Design decisions (validated in the interview)

1. **Name:** `dotnet-release` (short, discoverable; `name` matches the directory).
2. **Execution boundary:** the skill makes **in-repo edits only** and **prints** the outward-facing/irreversible commands (`git tag`, `dotnet pack`, `dotnet nuget push`) for the user to run — never executes them. Matches `git-repo-hygiene`/`dev-workflow` (user owns commits/tags/publishes). The NuGet API key is never stored or echoed.
3. **NuGet updates:** run `dotnet list package --outdated`; apply non-coupled bumps in `Directory.Packages.props`; **hold back** the version-coupled ecosystems (`Avalonia.*`, `Carbon.Avalonia.Desktop`, `PhosphorIconsAvalonia`, `AvaloniaUI.DiagnosticsSupport`) unless the user opts in; record each `old → new` transition in `RELEASENOTES.md`.
4. **Tag format:** detect existing `git tag`s and match them; default to **bare `X.Y.Z`** (Enigma.Cryptography's actual tags) for a repo with no tags.
5. **`GeneratePackageOnBuild`:** documented as **OFF** for a publishable library — pack explicitly via the release step (consistent with the "never auto-execute" boundary).
6. **Runbook artifact:** bundle a `templates/RELEASE.md` and offer it into the target repo's `docs/`, **create-only-if-missing** (mirrors `Enigma.LicenseManager`).
7. **App vs library:** library/NuGet-focused; for a non-packable app the `pack`/`push` steps are skipped (version bump + release notes + tag still apply).
8. **Badges (defaults):** NuGet-version + License (MIT) added by default; Downloads badge offered as optional.
9. **What's-new callout / release notes (defaults):** a single blockquote after the intro (`> **What's new in X.Y** — …. See the [release notes](RELEASENOTES.md).`); `RELEASENOTES.md` newest-first, prepended, rename any `(unreleased)` heading to `vX.Y.Z Release Notes`; match the target repo's existing style when present.
10. **House conventions applied:** frontmatter only `name`+`description`; body ends with `## Common mistakes`; cross-reference `dev-workflow` / `dotnet-solution-config` / `git-repo-hygiene` rather than restating; template bundled un-dotted under `templates/`.
11. **Skill type:** Reference/procedure skill; verification = dry-run at build time.

## Scope

Directory `/home/jo/dotfiles2/.claude/skills/dotnet-release/`:

- **`SKILL.md`** — new. Frontmatter + body (sections below).
- **`templates/RELEASE.md`** — new. The human runbook, from `/home/jo/Dev/Enigma.LicenseManager/docs/RELEASE.md` (parameterized for package id / solution name).

### SKILL.md content

- Frontmatter:
  - `name: dotnet-release`
  - `description: Use when releasing a new version of a .NET library/app to NuGet — bumping the version, updating README badges and the what's-new callout, release notes, PackageReleaseNotes, refreshing NuGet packages, and printing the pack/tag/push runbook.`
- Body sections:
  1. `# .NET version release` + one-line rule; note a release is the final phase of a `FEATURE-NNN` item — defer to `dev-workflow`.
  2. **In-repo edits (the skill makes these):** `<Version>` in the library `.csproj`; `<PackageReleaseNotes>` prose (mirrors the top of `RELEASENOTES.md`, ends "See RELEASENOTES.md …"); `RELEASENOTES.md` prepend (rename `(unreleased)` → `vX.Y.Z Release Notes`); README badges (NuGet + License, optional Downloads) and the what's-new blockquote; version appears in badge (auto) + callout + any "supported TFMs" line.
  3. **NuGet package refresh:** `dotnet list package --outdated`; apply non-coupled bumps in `Directory.Packages.props`; hold coupled ecosystems; record `old → new` in `RELEASENOTES.md`. Cross-reference `dotnet-solution-config` for the CPM file.
  4. **Packable-library prerequisites:** `PackageId`; `PackageReadmeFile`/`PackageLicenseFile` + `<None Include="..\..\README.md|LICENSE.md" Pack="true" PackagePath="\" />`; `PackageReleaseNotes`; `GeneratePackageOnBuild` OFF.
  5. **Runbook — print, never run** (the `docs/RELEASE.md` commands): `dotnet build <sln> -c Release`; `dotnet test <sln> -c Release`; merge to the default branch; `git tag <X.Y.Z|vX.Y.Z>` + `git push origin <tag>` (detect/match convention; default bare); `dotnet pack <lib.csproj> -c Release -o ./artifacts`; `dotnet nuget push ./artifacts/<pkg>.nupkg --api-key <KEY> --source https://api.nuget.org/v3/index.json`; post-publish verification (package page, badge resolves, `dotnet add package … --version X.Y.Z`, tag exists). State explicitly the skill prints these; the user runs them.
  6. **App vs library:** non-packable app skips pack/push (version bump + release notes + tag still apply).
  7. Cross-references: `dev-workflow` (commit/branch/tag ownership + doc-freshness sweep + FEATURE-item framing), `dotnet-solution-config` (CPM/`Directory.Packages.props`), `git-repo-hygiene`.
  8. **`## Bundled files`** — link to `templates/RELEASE.md` (offered into `docs/`, create-only-if-missing).
  9. **`## Common mistakes`** — table: auto-pushing to NuGet / running tag/pack/push; bumping coupled Avalonia/Carbon/Phosphor sets individually; forgetting `<PackageReleaseNotes>`; wrong tag prefix (not matching existing tags); `GeneratePackageOnBuild` on; skipping the Release build/test pre-flight; not logging `old → new` package transitions.
  10. Close with the "match an existing repo's release conventions and flag divergence" caveat.

## Acceptance criteria

1. `dotnet-release/SKILL.md` exists with valid `name`+`description`-only frontmatter (`name` matches the directory) and the sections above.
2. `templates/RELEASE.md` exists (parameterized from the LicenseManager runbook).
3. SKILL.md makes **only in-repo edits** and **prints** (never runs) `git tag` / `dotnet pack` / `dotnet nuget push`; no API key is stored or echoed.
4. The NuGet-update policy documents check-outdated + held-back coupled sets + `old → new` logging; tag-format detect-and-match (default bare); `GeneratePackageOnBuild` OFF.
5. SKILL.md cross-references `dev-workflow`, `dotnet-solution-config`, and `git-repo-hygiene`, and does not restate their rules.
6. Dry-run: a fresh agent, given a sample library repo, performs the in-repo edits (version, release notes, README, `PackageReleaseNotes`, selective package updates) and prints the runbook without executing any outward-facing command.

## Notes & follow-ups

- No build/test step (documentation/prompt artifact): DoD criteria 1–2 are met by a well-formed skill verified via the dry-run in criterion 6.
- Depends on FEATURE-006 for the `dotnet-solution-config` cross-reference (build FEATURE-006 first).
- The two references differ on `GeneratePackageOnBuild` (Crypto off, LicenseManager on) and tag prefix (Crypto bare, LicenseManager runbook `v`-prefixed) — the skill standardizes on off + detect/default-bare; note the divergence for anyone aligning LicenseManager later.
