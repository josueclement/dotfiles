---
name: dotnet-solution-config
description: Use when setting up a .NET solution's shared root config — the full C# .editorconfig (code style, naming, analyzer severities), Directory.Build.props (solution-wide build defaults), and Directory.Packages.props (Central Package Management).
---

# .NET solution shared config

Give a .NET solution one authoritative code-style file and centralized build/package defaults at the solution root.

## When to use

- A new or existing .NET solution that wants one shared code-style file plus centralized build and package-version defaults across every project.
- Adding any one of the three files below to a solution that is missing it.

**Not for:** `.gitignore` / `.gitattributes` / CRLF line-ending normalization (see git-repo-hygiene) · per-project scaffolding, target frameworks, or IHost wiring (see dotnet-solution-setup).

## The three files

All three live at the **solution root**, beside the `.slnx`:

| File | Purpose | Source |
|------|---------|--------|
| `.editorconfig` | Full C# code style + naming conventions + analyzer severities, plus LF/charset/final-newline for all files. The single authoritative style file. | `templates/editorconfig` |
| `Directory.Build.props` | Solution-wide csproj defaults (`LangVersion`, `Nullable`, `TreatWarningsAsErrors`, `EnforceCodeStyleInBuild`) + author metadata. | `templates/Directory.Build.props` |
| `Directory.Packages.props` | Central Package Management skeleton — one place for every package version. | `templates/Directory.Packages.props` |

**Optional:** pin the SDK at the same root with a `global.json` (`{ "sdk": { "version": "10.0.100", "rollForward": "latestFeature" } }`). It belongs to the same solution-root config family but is **not** one of the three core files.

## Workflow — create only if missing

For each of the three files, in the target solution root:

1. If the file **does not exist**, copy the matching template to its target name: `templates/editorconfig` → `.editorconfig`, `templates/Directory.Build.props` → `Directory.Build.props`, `templates/Directory.Packages.props` → `Directory.Packages.props`.
2. If the file **already exists**, do **not** overwrite it. Report that it exists and show a diff against the template so the user decides what (if anything) to merge.
3. In the freshly copied `Directory.Build.props`, fill the `{{AUTHORS}}` / `{{YEAR}}` placeholders.

Never clobber an existing config — an established solution's `.editorconfig` or props often carry project-specific rules.

## Directory.Build.props

MSBuild imports `Directory.Build.props` into every project under the solution root automatically, so these defaults live in **one** place instead of being repeated in every `.csproj`:

- `LangVersion` / `Nullable` — the C# language level and nullable context for the whole solution.
- `TreatWarningsAsErrors` — warnings fail the build everywhere (library, CLI, Desktop, tests).
- `EnforceCodeStyleInBuild` — the warning-level `.editorconfig` rules are checked at build time, not just in the IDE.
- `Authors` / `Copyright` — shared package/assembly metadata (the parameterized placeholders).

This is the single source of truth for these values — they do **not** belong in individual csproj files. (dotnet-solution-setup defers here for shared csproj defaults.)

## Central Package Management

`Directory.Packages.props` centralizes every dependency version:

- `<ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>` turns CPM on.
- Each version is declared **once**, centrally, as `<PackageVersion Include="Foo" Version="1.2.3" />`.
- Project `.csproj` files reference packages **without** a `Version`: `<PackageReference Include="Foo" />`. The version comes from the central file, so shared dependencies can't drift between projects.
- **Version-coupled ecosystems** (Avalonia + Avalonia.Desktop + Avalonia.Themes.Fluent + Carbon.Avalonia.Desktop + PhosphorIconsAvalonia + CommunityToolkit.Mvvm) are bumped **together**, never one package at a time — mixing versions within the set breaks the build. The skeleton groups them so the coupling stays visible and the set can be held back until it moves as a unit.

The template ships commented example groups (Core / CLI / Desktop / Tests); uncomment and fill only the ones the solution needs.

## Cross-references

- **git-repo-hygiene** — `.gitignore`, `.gitattributes`, and line-ending (CRLF↔LF) normalization. Its minimal line-endings-only `.editorconfig` is for text-only / non-C# repos; a .NET solution uses the full `.editorconfig` here instead (a strict superset).
- **dotnet-solution-setup** — solution/project layout, target frameworks, and IHost bootstrapping.
- **xunit-v3** — the test-project packages that go in the CPM file's Tests group.
- **dotnet-release** — consumes `Directory.Packages.props` when updating dependency versions.

## Bundled files

- [`templates/editorconfig`](templates/editorconfig) — full C# code style, naming, and analyzer severities (+ LF/charset).
- [`templates/Directory.Build.props`](templates/Directory.Build.props) — solution-wide build defaults + author-metadata placeholders.
- [`templates/Directory.Packages.props`](templates/Directory.Packages.props) — CPM skeleton with commented example groups.

## Common mistakes

| Mistake | Fix |
|---------|-----|
| Naming a bundled template `.editorconfig` (with a dot) | Keep bundled names un-dotted (`templates/editorconfig`); copy to the dotted name at the target. |
| Overwriting a solution's existing `.editorconfig` / props | Create only if missing; diff, don't clobber. |
| Keeping a separate code-style `.editorconfig` per project or split across skills | One authoritative `.editorconfig` at the solution root. |
| Repeating `LangVersion` / `Nullable` / `TreatWarningsAsErrors` in every csproj | Put them once in `Directory.Build.props`. |
| Adding `Version="..."` to a project `<PackageReference>` under CPM | The version lives centrally in `<PackageVersion>`; the reference carries none. |
| Bumping Avalonia / Carbon / Phosphor versions individually | Bump the coupled ecosystem together. |

If an existing solution already has its own root config, stay consistent with it and flag the divergence rather than silently mixing styles.
