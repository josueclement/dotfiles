# FEATURE-006 — `dotnet-solution-config` skill — DONE

## Summary

Added a new Claude Code skill, `dotnet-solution-config`, that scaffolds the three shared
solution-root config files into a .NET solution: the full C# `.editorconfig` (code style,
naming, analyzer severities, line endings), `Directory.Build.props` (solution-wide csproj
defaults + author metadata), and `Directory.Packages.props` (Central Package Management
skeleton). The skill is create-only-if-missing and never clobbers existing config.

Two ownership overlaps flagged in the plan were resolved with one-line cross-references
(not duplicated content): `git-repo-hygiene` now points at this skill's full `.editorconfig`
for .NET solutions (its own stays the minimal line-endings-only file), and
`dotnet-solution-setup` now points its shared-csproj-defaults guidance at
`Directory.Build.props` here (values kept, home moved).

## Files/modules touched

**Created**
- `.claude/skills/dotnet-solution-config/SKILL.md` — frontmatter (`name`+`description` only) + body: title/rule, When to use / Not for, the three files (+ optional `global.json` note), create-only-if-missing workflow, `Directory.Build.props` section, Central Package Management section, cross-references, bundled files, `## Common mistakes`, closing convention caveat.
- `.claude/skills/dotnet-solution-config/templates/editorconfig` — verbatim (byte-for-byte, verified via `diff`) copy of `/home/jo/Dev/Enigma.Cryptography/.editorconfig`.
- `.claude/skills/dotnet-solution-config/templates/Directory.Build.props` — Enigma `Directory.Build.props` with `Authors`/`Copyright` parameterized as `{{AUTHORS}}`/`{{YEAR}}` placeholders.
- `.claude/skills/dotnet-solution-config/templates/Directory.Packages.props` — CPM skeleton: `ManagePackageVersionsCentrally=true` + commented Core/CLI/Desktop/Tests example groups + held-back-coupled-ecosystem guidance (Avalonia/Carbon/Phosphor).
- `docs/done/FEATURE-006.md` — this record.

**Modified**
- `.claude/skills/git-repo-hygiene/SKILL.md` — one-line pointer in "Not for": a .NET solution wanting a full C# code-style `.editorconfig` uses `dotnet-solution-config` (a strict superset).
- `.claude/skills/dotnet-solution-setup/SKILL.md` — "csproj defaults (every project)" now points shared defaults at `Directory.Build.props` via `dotnet-solution-config` (values unchanged).
- `docs/roadmap.md` — FEATURE-006 → `IN PROGRESS`, then `DONE`.
- `docs/plan/FEATURE-006.md` — status → `IN PROGRESS`, then `DONE`.

## Deviations & follow-ups

- **No deviations from the plan's design decisions.** All 9 validated decisions were honored (name matches directory, single skill for all three files, full `.editorconfig` verbatim, `Directory.Build.props` home for shared defaults, CPM skeleton not a package list, `global.json` mentioned as optional, `.gitattributes`/`.gitignore` deferred to `git-repo-hygiene`, house frontmatter/`## Common mistakes` conventions, reference-type verified by dry-run).
- **`global.json`** — included as the plan's optional parity follow-up: a one-line note under "The three files", explicitly not one of the three core files.
- **`ImplicitUsings`** — the Enigma `Directory.Build.props` (shipped verbatim per the plan) does not set `ImplicitUsings`; `dotnet-solution-setup` keeps documenting `<ImplicitUsings>disable</ImplicitUsings>` as a per-project rule. No conflict, but noting the shared props file does not enforce that house rule.
- **Line endings:** all authored files are LF / UTF-8 (verified — no CRLF anywhere in the skill). Recommendation only; normalization is the user's call per `dev-workflow` / `git-repo-hygiene`.
- **Follow-up:** FEATURE-007 (`dotnet-release`) cross-references this skill's `Directory.Packages.props` as the file it updates; build it next, as planned.

## Build/test evidence

Documentation/prompt artifact — there is no build step or test suite (DoD criteria 1–2 satisfied by the well-formed-artifact equivalent). Verification performed:

- **Well-formedness:** frontmatter is `name`+`description` only, `name: dotnet-solution-config` matches the directory; all planned body sections present; body ends with `## Common mistakes` + closing caveat.
- **Template fidelity:** `templates/editorconfig` is byte-for-byte identical to the Enigma source (`diff` → IDENTICAL). Props/packages templates carry the intended placeholders / CPM skeleton.
- **Line endings:** `grep -lrU $'\r'` over the skill → no CRLF; all files UTF-8/LF.
- **Acceptance criterion 5 — dry-run (independently verified):** a fresh general-purpose agent, given only the skill and a sample .NET solution (with a pre-existing customized `.editorconfig` and no `Directory.Build.props`/`Directory.Packages.props`), scaffolded the two missing files (placeholders filled → "Jane Doe"/2026; CPM enabled) and **left the pre-existing `.editorconfig` untouched** (sentinel line + `indent_style = tab` preserved, 4 lines unchanged), producing a diff for the user to merge instead of overwriting. All acceptance criteria (1–5) met.
