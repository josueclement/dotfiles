# FEATURE-008 — Avalonia app/window icon convention

**Status:** DONE

## Summary

Added a short, house-style **Application icon** convention to the `avalonia` skill and kept the
`carbon-avalonia-desktop` setup examples consistent with it. Net-new content — no icon guidance
existed in any skill before. The convention: ship one shared multi-resolution `.ico` under
`Assets/` and wire it two ways — `<ApplicationIcon>` in the csproj (embeds a Win32 icon into the
Windows `.exe`, so Explorer/taskbar show it even when the app isn't running; Windows-only) and
`Window.Icon` on the MainWindow (the runtime titlebar/taskbar icon on every platform, and the only
one that does anything on Linux/macOS).

## Files/modules touched

- **Modified** `.claude/skills/avalonia/SKILL.md` — new `## Application icon` section inserted after
  the Bootstrapping section (before "Compiled bindings"): one-line multi-resolution intro, csproj
  snippet (`<ApplicationIcon>` + `<AvaloniaResource Include="Assets/**" />` with the Windows-only
  caveat), MainWindow snippet (`Icon="/Assets/app.ico"`), and a closing line tying the two together.
- **Modified** `.claude/skills/carbon-avalonia-desktop/reference/setup.md` — added
  `<ApplicationIcon>Assets/app.ico</ApplicationIcon>` (with an inline pointer comment to the avalonia
  skill's section) plus an `<AvaloniaResource Include="Assets/**" />` item group to the csproj
  example, and `Icon="/Assets/app.ico"` to the MainWindow `<Window …>` example. No duplicated prose.

## Deviations & follow-ups

- **No deviations** from the plan. All five design decisions and both ripple targets implemented as
  specified; frontmatter untouched; paths consistent (`Assets/app.ico` in csproj, `/Assets/app.ico`
  in XAML).
- **Deliberately out of scope** (per interview): Linux `.desktop` launcher `Icon=` and macOS
  `.icns`/`CFBundleIconFile` app-bundle icons. Candidate future follow-up if full cross-platform
  packaging guidance is ever wanted.
- No new NuGet package — pure MSBuild/XAML; the `.ico` art is user-owned. `git-repo-hygiene` already
  marks `*.ico binary`, so committed icons are handled correctly.
- No CRLF / line-ending issues observed in the touched files.

## Build/test evidence

Documentation/prompt-only change — **nothing to build or run and no test suite applies** (DoD
criteria 1–2 satisfied by the applicable equivalent). Acceptance criteria verified **by inspection**:

1. ✅ `avalonia/SKILL.md` has a concise `## Application icon` section showing the shared `.ico` wired
   as both `<ApplicationIcon>` (with `<AvaloniaResource Include="Assets/**" />`) and `Window.Icon`,
   including the Windows-only caveat and the multi-resolution (16/32/48/256 px) tip.
2. ✅ `carbon-avalonia-desktop/reference/setup.md`'s csproj and MainWindow examples show the icon and
   point at the avalonia section (inline pointer comment).
3. ✅ Terse house style preserved; no duplication of `dotnet-solution-setup`'s general csproj
   guidance; frontmatter untouched.
4. ✅ All snippets well-formed; paths consistent (`Assets/app.ico`, `/Assets/app.ico`).
5. ✅ Roadmap + plan file statuses updated to `DONE`; this completion doc written.
