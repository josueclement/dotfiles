---
name: dotnet-release
description: Use when releasing a new version of a .NET library/app to NuGet — bumping the version, updating README badges and the what's-new callout, release notes, PackageReleaseNotes, refreshing NuGet packages, and printing the pack/tag/push runbook.
---

# .NET version release

Drive a new NuGet library-version release end-to-end: make the in-repo edits (version, release notes, README, package metadata, selective dependency bumps), then **print** the out-of-repo runbook for the user to run. A release is the **final phase of a `FEATURE-NNN` work item** — defer to **dev-workflow** for the branch, the commit, tag/publish ownership, the roadmap/plan/`done` records, and the documentation-freshness sweep; this skill does not restate them.

## When to use

- Cutting a new version of a .NET **library** that publishes to NuGet — bump the version, refresh notes and README, update dependencies, and produce the pack/tag/push runbook.
- Cutting a new version of a non-packable **app** — the version bump, release notes, and tag still apply; the `pack`/`push` steps are skipped (see *App vs library*).

**Not for:** the shape of `Directory.Packages.props` / CPM itself (see **dotnet-solution-config**) · `.gitignore` / `.gitattributes` / line-ending normalization (see **git-repo-hygiene**) · the roadmap / plan / `done` / commit / branch mechanics of the work item (see **dev-workflow**).

## Execution boundary

The skill makes **in-repo edits only**. It **prints — never runs —** the outward-facing, irreversible commands (`git tag`, `dotnet pack`, `dotnet nuget push`, and the merge/push to the default branch); the user runs them. The NuGet API key is never stored, committed, or echoed. This mirrors **git-repo-hygiene** and **dev-workflow**: the user owns commits, tags, and publishes.

## In-repo edits (the skill makes these)

Given the version `X.Y.Z` being released:

- **Target frameworks** — check the **packable library**'s `<TargetFramework>` / `<TargetFrameworks>` and normalize its modern-.NET set to the current LTS pair. **This is the one in-repo edit the skill proposes and confirms before writing:** show the `old → new` TFM set and wait for the user's OK, because a TFM change moves the library's compatibility surface. Apps/CLIs are left alone (they ship a single TFM).
  - **Preserve** every `netstandard*` target (max consumer compatibility) and every **platform-specific** TFM (one with a `-` suffix — `net8.0-windows`, `net10.0-android`, …); rewriting those would strip the platform surface. If a platform-specific TFM is older than `net8`, **warn** (print a note) for manual review — don't change it.
  - **Normalize the plain-`net` targets to exactly `net8.0;net10.0`** — the two currently-supported LTS releases (net8 LTS through Nov 2026, net10 LTS from Nov 2025), so the modern-.NET set is `net8.0` + `net10.0` **for now**. Add whichever is missing, replace any `net` older than 8 (`net6.0`, `netcoreapp*`), and collapse any other `net` (e.g. `net9.0`) down to the pair. This fires **only when a plain-`net` target already exists** — a `netstandard`-only library is left untouched (never force `net` onto it):

    | before | after |
    |---|---|
    | `net8.0` | `net8.0;net10.0` |
    | `netstandard2.0;net8.0` | `netstandard2.0;net8.0;net10.0` |
    | `net6.0` | `net8.0;net10.0` |
    | `net9.0` | `net8.0;net10.0` |
    | `net10.0` | `net8.0;net10.0` |
    | `netstandard2.0` | `netstandard2.0` (unchanged) |

  - When the result carries more than one TFM, use the plural **`<TargetFrameworks>`** element (`;`-separated — `netstandard*` first, then ascending `net`, then any preserved platform TFM), converting from a singular `<TargetFramework>` if needed.
  - **Log the change:** record the `old → new` TFM set in the release's `RELEASENOTES.md` *Compatibility* sub-section and update the README **"supported target frameworks"** line (below). A **dropped** TFM (replacing a `net` older than 8) is a compatibility break — say so in the notes.
- **`<Version>`** in the packable library `.csproj` → `X.Y.Z`. (A CLI/app in the same solution keeps its own independent `<Version>`.)
- **`<PackageReleaseNotes>`** in that csproj — a short prose summary mirroring the top of `RELEASENOTES.md`, ending with `See RELEASENOTES.md for the full details.` (or the migration guide, for a breaking release).
- **`RELEASENOTES.md`** — prepend the new section (newest-first). If the top section is headed `(unreleased)`, rename it to `<Name> vX.Y.Z Release Notes`; otherwise add a new `# <Name> vX.Y.Z Release Notes` block above the previous one. Match the repo's existing sub-section style (e.g. *New Features · Breaking Changes & Migration · Compatibility · Version*).
- **`README.md`**:
  - **Badges** under the title — NuGet version and License by default; the Downloads badge is optional (offer it):
    ```markdown
    [![NuGet](https://img.shields.io/nuget/v/<PackageId>.svg)](https://www.nuget.org/packages/<PackageId>)
    [![Downloads](https://img.shields.io/nuget/dt/<PackageId>.svg)](https://www.nuget.org/packages/<PackageId>)
    [![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE.md)
    ```
    The NuGet-version badge tracks the published version automatically — no per-release edit.
  - **What's-new callout** — a single blockquote after the intro:
    ```markdown
    > **What's new in X.Y** — <one-line highlight>. See the [release notes](RELEASENOTES.md).
    ```
  - Update any **"supported target frameworks"** line if the TFM set changed this release.

So the actual per-release version edits are usually just `<Version>` in the csproj and the what's-new callout — plus the target-framework normalization on the releases that move the TFM set. The NuGet badge tracks the published version on its own — leave it alone — and the supported-TFMs line changes only if the TFM set moved this release.

## NuGet package refresh

- Run `dotnet list package --outdated` to see what moved.
- Apply the **non-coupled** bumps by editing `<PackageVersion>` entries in `Directory.Packages.props` (see **dotnet-solution-config** for the CPM file's layout).
- **Hold back the version-coupled ecosystems** unless the user opts in — bump each set together, never one package at a time: `Avalonia.*`, `Carbon.Avalonia.Desktop`, `PhosphorIconsAvalonia`, `AvaloniaUI.DiagnosticsSupport`.
- **Record every `old → new` transition** in the release's `RELEASENOTES.md` section (a *Dependencies* bullet), and note any coupled set deliberately held back.

## Packable-library prerequisites

Before the first release (or when verifying an existing one), the library csproj must have:

- `<PackageId>`.
- `<PackageReadmeFile>README.md</PackageReadmeFile>` and `<PackageLicenseFile>LICENSE.md</PackageLicenseFile>`, with the files packed:
  ```xml
  <ItemGroup>
    <None Include="..\..\README.md" Pack="true" PackagePath="\" />
    <None Include="..\..\LICENSE.md" Pack="true" PackagePath="\" />
  </ItemGroup>
  ```
- `<PackageReleaseNotes>` (see above).
- **`GeneratePackageOnBuild` OFF** (absent, or `false`) — a publishable library is packed explicitly by the release step, never on every local build, consistent with the print-don't-run boundary.

## Runbook — print, never run

Print the following for the user to run (these are the `docs/RELEASE.md` steps; the bundled template is the fuller checklist). State explicitly: **the skill prints these; the user runs them.**

```bash
# 1. Pre-flight (Release configuration)
dotnet build <solution> -c Release
dotnet test  <solution> -c Release

# 2. Merge the release branch into the default (published) branch, then locally:
git switch <default-branch> && git pull

# 3. Tag — match existing tags; default bare X.Y.Z
git tag X.Y.Z
git push origin X.Y.Z

# 4. Pack (GeneratePackageOnBuild is off)
dotnet pack <lib.csproj> -c Release -o ./artifacts

# 5. Push to NuGet (API key has push rights; never commit/echo it)
dotnet nuget push ./artifacts/<PackageId>.X.Y.Z.nupkg \
  --api-key <NUGET_API_KEY> \
  --source https://api.nuget.org/v3/index.json
```

Then **post-publish verification**: the package page shows `X.Y.Z`; the README NuGet badge resolves; `dotnet add package <PackageId> --version X.Y.Z` restores; and the tag exists with notes matching `RELEASENOTES.md`.

Detect the specifics rather than assuming them:

- **Tag format** — `git tag` to see the repo's convention (bare `X.Y.Z` vs. `vX.Y.Z`) and match it; default to **bare `X.Y.Z`** for a repo with no tags.
- **Solution vs. single project** — use the solution file (`.slnx` / `.sln`) for the build/test pre-flight; if the repo has no solution (one packable project), use the library `.csproj` instead.
- **Default branch** — the published branch you merge into and pack from; the house repos use `main` or `master`. Confirm with `git remote show origin` (or the local default) if unsure.

The merge/tag/push steps assume an initialized git repo with a remote — skip them for a repo that isn't set up to publish yet.

## App vs library

For a **non-packable app** (no `PackageId`, not published to NuGet), skip the pack/push steps entirely — the version bump, `RELEASENOTES.md`, README callout, and the tag still apply.

## Cross-references

- **dev-workflow** — the release is the final phase of a `FEATURE-NNN` item; it owns branch naming, the never-commit-myself rule, tag/publish ownership, the `docs/roadmap.md` + `docs/plan/` + `docs/done/` records, and the doc-freshness sweep.
- **dotnet-solution-config** — the CPM `Directory.Packages.props` file this skill edits when refreshing dependency versions, and the coupled-ecosystem rule.
- **dotnet-solution-setup** — the authority for choosing a project's target frameworks at creation time; this skill's release-time normalization keeps a multi-targeted library on the current `net8.0` + `net10.0` LTS pair.
- **git-repo-hygiene** — `.gitignore` / `.gitattributes` / line-ending normalization.

## Bundled files

- [`templates/RELEASE.md`](templates/RELEASE.md) — the human release checklist (pre-flight → merge → tag → pack → push → verify). Offer it into the target repo's `docs/RELEASE.md`, **create only if missing** (don't clobber an existing runbook — diff instead), and fill its placeholders (`{{PACKAGE_ID}}`, `{{SOLUTION}}`, `{{LIB_CSPROJ}}`, `{{LIB_DIR}}`, `{{DEFAULT_BRANCH}}`).

## Common mistakes

| Mistake | Fix |
|---------|-----|
| Auto-pushing to NuGet, or running `git tag` / `dotnet pack` / `dotnet nuget push` for the user | The skill only prints them; the user runs the outward-facing commands. |
| Bumping `Avalonia.*` / `Carbon.Avalonia.Desktop` / `PhosphorIconsAvalonia` individually | Bump the coupled ecosystem together, or hold the whole set back. |
| Forgetting `<PackageReleaseNotes>` in the library csproj | Add it, mirroring the top of `RELEASENOTES.md`. |
| Tagging with the wrong prefix (bare vs. `v`) | Detect existing tags and match; default bare `X.Y.Z`. |
| `GeneratePackageOnBuild` left on for a publishable library | Turn it off; pack explicitly in the release step. |
| Skipping the Release-config build/test pre-flight | Always `dotnet build`/`dotnet test -c Release` before tagging. |
| Not logging `old → new` dependency transitions | Record every version change (and held-back set) in `RELEASENOTES.md`. |
| TFM set changed but the README "supported target frameworks" line / `RELEASENOTES.md` *Compatibility* note not updated | Log the `old → new` TFM set and keep the audit trail in sync; flag a dropped TFM as a breaking change. |
| Committing or echoing the NuGet API key | The key is a secret — it never appears in the repo or output. |

If the target repo already has its own release conventions (tag prefix, notes layout, badge set), stay consistent with them and flag the divergence rather than silently imposing these defaults.
