---
name: carbon-avalonia-desktop
description: Use when building, improving, or adding controls to an Avalonia desktop app with the Carbon.Avalonia.Desktop control library (NuGet package Carbon.Avalonia.Desktop) — including installing it into an app that doesn't reference it yet, or theming standard Avalonia controls with it — spanning its SettingsCard/SettingsCardExpander, Overlay, ContentDialog, Editors, NavigationView, InfoBar, Ribbon, CalendarSchedule, Displayer2D, Docking controls, the dialog/overlay/infobar/navigation/file-picker services, DI + host wiring, and Fluent theme/brushes.
---

# Carbon.Avalonia.Desktop

Reusable **Avalonia 12** control library (Fluent-based, Dark + Light) published on nuget.org as
`Carbon.Avalonia.Desktop`. Two patterns to know:

- **Controls** are `TemplatedControl`s used directly in XAML (SettingsCard, Ribbon, editors, …).
- **Services** (`ContentDialog`, `Overlay`, `InfoBar`, navigation, file/folder pickers) are injected via
  DI. The three overlay-style services need a **host control** registered once at startup
  (`RegisterHost`); calling them before that throws `InvalidOperationException`.

MVVM-friendly (ships `CommunityToolkit.Mvvm` transitively) but not MVVM-bound.

## Install

The package targets `net8.0`/`net10.0` and needs Avalonia **12.0.2**. It does **not** bring the Avalonia
*app* packages transitively — add them yourself:

```bash
dotnet add package Carbon.Avalonia.Desktop        # currently 0.2.0
dotnet add package Avalonia.Desktop                # 12.0.2 — desktop lifetime
dotnet add package Avalonia.Themes.Fluent          # 12.0.2 — base FluentTheme
dotnet add package Avalonia.Fonts.Inter            # 12.0.2 — WithInterFont()
```

You get transitively: `Avalonia`, `Avalonia.Themes.Fluent`, `CommunityToolkit.Mvvm`,
`Microsoft.Extensions.DependencyInjection`, `PhosphorIconsAvalonia`, `Enigma.Cryptography`.

## Bootstrap (5 steps — full detail in `reference/setup.md`)

1. **Include the theme in `App.axaml`** — base `<FluentTheme/>` in `Application.Styles`, and the Carbon
   dictionary as a **`ResourceInclude` in `Application.Resources`** (it has a `ResourceDictionary` root,
   so `StyleInclude` is wrong):
   ```xml
   <Application.Styles><FluentTheme /></Application.Styles>
   <Application.Resources>
     <ResourceDictionary>
       <ResourceDictionary.MergedDictionaries>
         <ResourceInclude Source="avares://Carbon.Avalonia.Desktop/Themes/Fluent.axaml" />
       </ResourceDictionary.MergedDictionaries>
     </ResourceDictionary>
   </Application.Resources>
   ```
2. **Register services** in DI (`Microsoft.Extensions.DependencyInjection`): the six Carbon services as
   singletons — see `reference/setup.md` for the `AddCarbonServices()` block.
3. **Place the host controls** as siblings in MainWindow's root `Panel`: `ContentDialog`, `Overlay`,
   `InfoBar`, each `x:Name`d.
4. **Wire hosts once** in `App.OnFrameworkInitializationCompleted`: `RegisterHost(...)` for the three
   host services + `SetStorageProvider(mainWindow.StorageProvider)` for the file/folder services.
5. **Inject services** into ViewModels via constructor and call `await service.ShowAsync(...)`.

## Quick reference

| Control / feature | XAML `xmlns` (`using:…`) | Purpose | Reference |
|---|---|---|---|
| SettingsCard, SettingsCardExpander, Overlay | `Carbon.Avalonia.Desktop.Controls` | Settings rows / modal busy overlay | `settings-cards.md`, `dialogs-overlay-infobar.md` |
| ContentDialog | `Carbon.Avalonia.Desktop.Controls.ContentDialog` | Modal dialog + `DialogResult`/`DefaultButton` | `dialogs-overlay-infobar.md` |
| InfoBar | `Carbon.Avalonia.Desktop.Controls.InfoBar` | Inline notification + `InfoBarSeverity` | `dialogs-overlay-infobar.md` |
| NavigationView, NavigationItem | `Carbon.Avalonia.Desktop.Controls.Navigation` | Side/top nav + page switching | `navigation.md` |
| Editors (Int/Double/Text/Hex/…) | `Carbon.Avalonia.Desktop.Controls.Editors` | Typed, validated input fields | `editors.md` |
| Ribbon, RibbonTab/Group/Button/… | `Carbon.Avalonia.Desktop.Controls.Ribbon` | Office-style ribbon | `ribbon.md` |
| CalendarSchedule | `Carbon.Avalonia.Desktop.Controls.CalendarSchedule` | Week/month appointment view | `calendar-schedule.md` |
| Displayer2D | `Carbon.Avalonia.Desktop.Controls.Displayer2D` | Pan/zoom 2D canvas with shapes | `displayer2d.md` |
| DockingHost | `Carbon.Avalonia.Desktop.Controls.Docking` | IDE-style dockable panes | `docking.md` |
| File/Folder pickers | `Carbon.Avalonia.Desktop.Services` | `IFileDialogService` / `IFolderDialogService` | `file-folder-dialogs.md` |
| CollectionViewSource | `Carbon.Avalonia.Desktop.Data` | Sort/filter/group over a collection | `data-collectionview.md` |
| Theme colors/brushes | — | `Carbon*` `{DynamicResource}` keys | `theming.md` |

## Cross-cutting conventions

- **Icons**: in XAML `xmlns:pia="using:PhosphorIconsAvalonia.Markup"` then
  `IconData="{pia:IconGeometry Icon=gear}"`; in C# `using PhosphorIconsAvalonia;` then
  `IconService.CreateGeometry(Icon.gear, IconType.regular)` (`IconService`/`Icon`/`IconType` all live in
  `PhosphorIconsAvalonia`). Any `Geometry` works too — e.g. `Geometry.Parse("M3 17…")`.
- **Theme brushes**: always `{DynamicResource CarbonXxxBrush}` (never `StaticResource`) so themes switch
  at runtime. Common: `CarbonBackgroundBrush`, `CarbonSurfaceBrush`, `CarbonForegroundSecondaryBrush`.
- **Hit testing**: an element with no `Background` is invisible to the pointer — set
  `Background="Transparent"` on areas that must receive clicks/hover.
- **MVVM (if using CommunityToolkit.Mvvm)**: `ObservableObject`/`ObservableValidator`,
  `RelayCommand`/`AsyncRelayCommand`, and semi-auto properties `public T X { get; set => SetProperty(ref field, value); }`.

## Also use it when

- **The app doesn't reference the package yet.** Adopting the library is in scope — `reference/setup.md`
  covers adding the NuGet package and wiring it up, then using its controls.
- **You're styling standard Avalonia controls** (`TextBox`, `ComboBox`, `Button`, …). Merging the Carbon
  theme restyles them automatically; `reference/theming.md` explains how and lists the `Carbon*` brushes
  to reference from your own markup.

## When NOT to use

- Non-Avalonia UIs — WPF, WinForms, .NET MAUI, or web. This library is Avalonia-only.

Read the relevant `reference/*.md` for full public API, a runnable example, and common mistakes.
