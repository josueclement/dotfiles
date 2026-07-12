# FEATURE-006 — `dotnet-solution-config` skill

**Status:** DONE

_Single-phase item. Planned via `/interview` (v7); build with `/build FEATURE-006`. Build this **before** FEATURE-007, which cross-references it._

## Objective

Add a reusable Claude Code skill, `dotnet-solution-config`, that scaffolds the three
solution-root shared-config files into a .NET solution: the full C# `.editorconfig`
(code-style + naming + analyzer severities + line endings), `Directory.Build.props`
(solution-wide csproj defaults), and `Directory.Packages.props` (Central Package
Management skeleton).

## Context

The source content already exists, verbatim, in the house reference repos. The goal is to
turn it into a skill so any future .NET solution gets the same shared config for free. This is
a documentation/prompt artifact in the `dotfiles2` repo — no build/test step.

Two overlaps with existing skills were resolved in the interview:
- `git-repo-hygiene` ships a **minimal** line-endings-only `.editorconfig`. The Enigma
  `.editorconfig` is a **strict superset** (it already contains the LF/charset/final-newline
  rules). This skill owns the full file; `git-repo-hygiene` gets a one-line pointer to it.
- `dotnet-solution-setup` currently says put shared csproj defaults in "every csproj". Enigma
  centralizes them in `Directory.Build.props`. This skill owns `Directory.Build.props`;
  `dotnet-solution-setup` gets a cross-reference.

**Reference sources (copy from these when building):**
- `/home/jo/Dev/Enigma.Cryptography/.editorconfig` — the full file (identical byte-for-byte in
  `Enigma.LicenseManager`).
- `/home/jo/Dev/Enigma.Cryptography/Directory.Build.props` — the shared defaults.
- `/home/jo/Dev/Enigma.Cryptography/Directory.Packages.props` and
  `/home/jo/Dev/Enigma.LicenseManager/Directory.Packages.props` — the latter shows the per-group
  "held-back ecosystem" comments to fold into the skeleton.
- `/home/jo/Dev/Enigma.Cryptography/global.json` — optional SDK pin to mention.

## Design decisions (validated in the interview)

1. **Name:** `dotnet-solution-config` (sits beside the existing `dotnet-solution-setup`; `name` matches the directory).
2. **Split:** one skill covering all three files (not three separate skills) — they are set up together at the solution root.
3. **`.editorconfig` ownership:** this skill ships the **full** Enigma `.editorconfig` verbatim; it is the single authoritative code-style file. `git-repo-hygiene`'s minimal one stays for non-.NET-code / text-only repos.
4. **`Directory.Build.props` ownership:** this skill is the home for solution-wide csproj defaults (`LangVersion`, `Nullable`, `TreatWarningsAsErrors`, `EnforceCodeStyleInBuild`, `Authors`, `Copyright`). `Authors`/`Copyright` are **parameterized placeholders**.
5. **`Directory.Packages.props`:** ship a **CPM skeleton** (`ManagePackageVersionsCentrally=true` + commented example `ItemGroup`s), not a verbatim package list — the package set is project-specific. Include the "held-back version-coupled ecosystem" comment as guidance.
6. **`global.json`:** mentioned as an **optional** extra (SDK pin), not one of the three core files.
7. **`.gitattributes`/`.gitignore`:** out of scope — cross-reference `git-repo-hygiene`, do not duplicate.
8. **House conventions applied:** frontmatter is only `name`+`description`; body ends with `## Common mistakes`; templates bundled un-dotted under `templates/`; **create-only-if-missing**, never clobber; cross-reference sibling skills by bare name rather than restating them.
9. **Skill type:** Reference-type skill; verification = retrieval/dry-run at build time.

## Scope

Directory `/home/jo/dotfiles2/.claude/skills/dotnet-solution-config/`:

- **`SKILL.md`** — new. Frontmatter + body (sections below).
- **`templates/editorconfig`** — new. Verbatim full Enigma `.editorconfig` (from `/home/jo/Dev/Enigma.Cryptography/.editorconfig`).
- **`templates/Directory.Build.props`** — new. Verbatim Enigma `Directory.Build.props`, with `Authors`/`Copyright` as placeholders.
- **`templates/Directory.Packages.props`** — new. CPM skeleton: `ManagePackageVersionsCentrally=true` + commented example groups (Core / CLI / Desktop / Tests) + the held-back-ecosystem comment.

### SKILL.md content

- Frontmatter:
  - `name: dotnet-solution-config`
  - `description: Use when setting up a .NET solution's shared root config — the full C# .editorconfig (code style, naming, analyzer severities), Directory.Build.props (solution-wide build defaults), and Directory.Packages.props (Central Package Management).`
- Body sections:
  1. `# .NET solution shared config` + one-line rule ("Give a .NET solution one authoritative code-style file and centralized build/package defaults at the solution root.").
  2. **When to use / Not for** — new or existing .NET solution wanting shared code-style + centralized build/package config. Not for `.gitignore`/`.gitattributes`/CRLF (see `git-repo-hygiene`), not for per-project scaffolding (see `dotnet-solution-setup`).
  3. **The three files** — table: `.editorconfig` (full C# code style + naming + analyzer severities + LF/charset), `Directory.Build.props` (shared csproj defaults + author metadata), `Directory.Packages.props` (CPM skeleton).
  4. **Workflow** — create-only-if-missing: copy each `templates/*` to its target name only if absent; if present, leave it and report/diff. Fill the `Authors`/`Copyright` placeholders.
  5. **`## Directory.Build.props`** — the shared defaults it carries and why they live here (single source of truth); note this supersedes per-csproj defaults (`dotnet-solution-setup` defers here).
  6. **`## Central Package Management`** — the skeleton, the pinned-version model (`<PackageVersion>` central, no `Version` on project `<PackageReference>`), and the held-back-ecosystem guidance.
  7. Cross-references: `git-repo-hygiene` (line-endings/`.gitattributes`/`.gitignore`), `dotnet-solution-setup` (project scaffolding + TFMs), `xunit-v3` (test packages), `dotnet-release` (which consumes the CPM file for updates).
  8. **`## Bundled files`** — links to `templates/`.
  9. **`## Common mistakes`** — table: dotting a template filename; clobbering a customized config; splitting the editorconfig across skills; putting shared defaults back in every csproj; adding `Version` to a project `<PackageReference>` under CPM; bumping coupled Avalonia/Carbon/Phosphor versions individually.
  10. Close with the "match an existing solution's conventions and flag divergence" caveat.

### Cross-reference edits to existing skills (part of this feature)

- **`/home/jo/dotfiles2/.claude/skills/git-repo-hygiene/SKILL.md`** — add one line: for a .NET solution wanting full C# code-style, use `dotnet-solution-config`'s `.editorconfig` instead of the minimal one here.
- **`/home/jo/dotfiles2/.claude/skills/dotnet-solution-setup/SKILL.md`** — point the "csproj defaults (every project)" guidance at `Directory.Build.props` via `dotnet-solution-config` (keep the values, move their home).

## Acceptance criteria

1. `dotnet-solution-config/SKILL.md` exists with valid `name`+`description`-only frontmatter (`name` matches the directory) and the sections above.
2. `templates/editorconfig`, `templates/Directory.Build.props`, `templates/Directory.Packages.props` exist with the intended content (editorconfig byte-for-byte from Enigma; props with `Authors`/`Copyright` placeholders; packages a CPM skeleton).
3. SKILL.md documents create-only-if-missing behavior and defers line-endings/`.gitattributes`/`.gitignore` to `git-repo-hygiene`.
4. The one-line cross-reference is added to both `git-repo-hygiene/SKILL.md` and `dotnet-solution-setup/SKILL.md`.
5. Dry-run: a fresh agent, given the skill and a sample .NET solution, scaffolds the three files without clobbering any pre-existing ones.

## Notes & follow-ups

- No build/test step (documentation/prompt artifact): DoD criteria 1–2 are met by a well-formed skill verified via the dry-run in criterion 5.
- The bundled `templates/*` are authored LF (line-ending recommendation only; owned by the user per `dev-workflow` / `git-repo-hygiene`).
- Optional parity follow-up: mention `global.json` SDK pinning as an extra; not one of the three core files.
