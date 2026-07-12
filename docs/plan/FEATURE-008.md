# FEATURE-008 — Avalonia app/window icon convention

**Status:** DONE

_Single-phase item. Planned via `/interview` (v7); build with `/build FEATURE-008`. Documentation/prompt change — no build/test step._

## Objective

Add a short **Application icon** convention to the `avalonia` skill: ship one multi-resolution
`.ico` and wire it **two ways** — `<ApplicationIcon>` in the csproj (embeds it in the Windows
`.exe` so Explorer/taskbar show it even when the app isn't running) **and** `Window.Icon` on the
MainWindow (the runtime titlebar/taskbar icon on every platform). Update the
`carbon-avalonia-desktop` setup reference so its inline examples stay consistent.

## Context

The `avalonia` skill is the house canonical Avalonia project-setup reference, but it currently
documents **no** application/window icon at all — nor does any sibling skill (verified: no
`<ApplicationIcon>`, `Window.Icon`, `WindowIcon`, or `<AvaloniaResource>` anywhere in the skills).
So this is **net-new content**, not a rewrite of an existing in-code icon approach.

The point of the change is the better practice that the built **executable file itself** should
carry the icon, not only the running UI. `<ApplicationIcon>` is **Windows-only** — it embeds a
Win32 icon resource into the `.exe` and has no effect on the Linux/macOS binary; on Linux/macOS
the icon seen while the app runs comes from the runtime `Window.Icon`. Wiring both from one shared
`.ico` is what makes the icon appear everywhere it can. `git-repo-hygiene` already marks
`*.ico binary`, so committed icons are handled correctly.

## Design decisions (validated in the interview)

1. **Scope:** one shared `.ico` wired as both `<ApplicationIcon>` (Windows `.exe`) and
   `Window.Icon` (runtime window, all platforms). Not `<ApplicationIcon>` alone.
2. **`.ico` production:** assume the user supplies the file; document only the wiring, plus a
   one-line tip that the `.ico` be multi-resolution (16/32/48/256 px) so the taskbar icon isn't
   blurry. No generation commands / external tooling (would rot, out of the skill's lane).
3. **Cross-platform depth:** a brief **inline** "Windows-only" caveat on `<ApplicationIcon>`;
   **no** dedicated Linux `.desktop` launcher or macOS `.icns`/app-bundle sections (out of scope —
   they pull packaging/bundling into a skill that is about project + XAML + DI setup).
4. **Ripple:** also update `carbon-avalonia-desktop`'s `reference/setup.md` so its csproj and
   MainWindow examples show the icon and point at the avalonia section — kept minimal
   (attribute/property + a one-line pointer), no duplicated prose.
5. **Placement:** a compact new `## Application icon` section in `avalonia/SKILL.md`, after the
   Bootstrapping section (it references both the csproj and the MainWindow).
6. **Asset convention (default applied):** folder `Assets/` (matches Avalonia templates),
   placeholder filename `app.ico`, forward-slash paths, `<AvaloniaResource Include="Assets/**" />`
   glob so `Icon="/Assets/app.ico"` resolves.
7. **House conventions:** terse; cross-reference rather than restate; general csproj properties
   stay owned by `dotnet-solution-setup` — the icon section adds only the icon-specific
   `<ApplicationIcon>` / `<AvaloniaResource>` bits.

## Scope

### 1. `/home/jo/dotfiles2/.claude/skills/avalonia/SKILL.md` — new `## Application icon` section (after Bootstrapping)

Concise section containing:
- One-line intro: ship a multi-resolution `.ico` under `Assets/` (16/32/48/256 px in one file) and
  wire it two ways — executable and running window are separate concerns.
- csproj snippet with `<ApplicationIcon>Assets/app.ico</ApplicationIcon>` in a `PropertyGroup`
  plus `<AvaloniaResource Include="Assets/**" />` in an `ItemGroup`, commented "embeds the icon in
  the Windows `.exe`; Windows-only — no effect on the Linux/macOS binary."
- MainWindow snippet `<Window ... Icon="/Assets/app.ico">`, commented "runtime titlebar/taskbar
  icon on every platform — and the only one that does anything on Linux/macOS."
- Closing line: one `.ico`, referenced from both places; `<ApplicationIcon>` gives the `.exe` its
  icon, `Icon="/Assets/app.ico"` covers the running window everywhere.

### 2. `/home/jo/dotfiles2/.claude/skills/carbon-avalonia-desktop/reference/setup.md` — keep examples consistent

- csproj example (~lines 13–30): add `<ApplicationIcon>Assets/app.ico</ApplicationIcon>` to the
  `PropertyGroup` and an `<ItemGroup><AvaloniaResource Include="Assets/**" /></ItemGroup>`.
- MainWindow `<Window …>` example (~lines 160–186): add `Icon="/Assets/app.ico"`.
- Add a one-line pointer (e.g. `<!-- app/window icon: see the avalonia skill's "Application icon" section -->`); no duplicated prose.

## Acceptance criteria

1. `avalonia/SKILL.md` has a concise `## Application icon` section showing the shared `.ico` wired
   as both `<ApplicationIcon>` (with `<AvaloniaResource Include="Assets/**" />`) and `Window.Icon`,
   including the Windows-only caveat and the multi-resolution tip.
2. `carbon-avalonia-desktop/reference/setup.md`'s csproj and MainWindow examples show the icon and
   point at the avalonia section.
3. Terse house style preserved; no duplication of `dotnet-solution-setup`'s general csproj guidance;
   frontmatter untouched.
4. All snippets are well-formed and paths consistent (`Assets/app.ico`, `/Assets/app.ico`).
5. Roadmap + plan file statuses updated; `docs/done/FEATURE-008.md` written, stating explicitly
   there was nothing to build/test (verified by inspection).

## Notes & follow-ups

- No build/test step (documentation change): DoD criteria 1–2 are satisfied by well-formed
  artifacts whose criteria are verified by inspection — state this in `docs/done/FEATURE-008.md`.
- Deliberately excluded (out of scope per interview): Linux `.desktop` launcher `Icon=` and macOS
  `.icns`/`CFBundleIconFile` app-bundle icons. Candidate future follow-up if full cross-platform
  packaging guidance is ever wanted.
- No new NuGet package — pure MSBuild/XAML; the `.ico` art is user-owned.
