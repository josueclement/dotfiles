---
name: phosphor-icons-avalonia
description: Use when adding, displaying, or styling icons in an Avalonia app that uses (or should use) the PhosphorIconsAvalonia NuGet package — installing the package, declaring its XAML namespace, using the IconGeometry or IconSource markup extensions, calling the IconService C# API, choosing an icon style (thin/light/regular/bold/fill), or looking up a Phosphor icon's enum name.
---

# PhosphorIconsAvalonia

## Overview

`PhosphorIconsAvalonia` is an Avalonia library embedding 1,512 Phosphor icons × 5 styles as SVG resources. **Zero setup:** no DI, no service registration, no resource-dictionary or theme includes — install the package, declare one `xmlns`, and use the two XAML markup extensions or the static `IconService`. Icons are the `Icon` enum; styles are the `IconType` enum.

Requires **Avalonia 12**; targets net8.0 / net10.0.

## Setup

1. Install: `dotnet add package PhosphorIconsAvalonia`
2. In any XAML file that uses an icon, declare the namespace. There is **no** `XmlnsDefinition` URI, so the `using:` (or `clr-namespace:`) form is required:

   ```xml
   xmlns:pia="using:PhosphorIconsAvalonia.Markup"
   ```

Nothing else to register or initialize.

## Quick reference

| API | Kind | Returns | Use for | Color from |
|-----|------|---------|---------|------------|
| `IconGeometry` | XAML markup ext | `Geometry` | `PathIcon.Data`, `Path.Data` | host control's `Foreground` |
| `IconSource` | XAML markup ext | `DrawingImage` | `Image.Source` | its `Brush` (**default black**) |
| `IconService.CreateGeometry(Icon, IconType)` | C# | `Geometry` | code-behind geometry | control's `Foreground` |
| `IconService.CreateDrawingImage(Icon, IconType, IBrush)` | C# | `DrawingImage` | code-behind `Image.Source` | the `IBrush` you pass |
| `IconService.GetIconData(Icon, IconType)` | C# | `string` (SVG path `d`) | raw path data | — |
| `IconService.GetIconStream(Icon, IconType)` | C# | `Stream?` | raw embedded SVG | — |

**Which XAML extension?** Inside a `PathIcon`/`Path`, or you want the icon to inherit the control's `Foreground` → **`IconGeometry`**. As an `Image.Source` with an explicit brush → **`IconSource`**.

Both extensions expose `Icon` and `IconType` (default `regular`); `IconSource` also has `Brush` (default **black**).

## XAML usage

```xml
<!-- Geometry: color comes from Foreground, NOT the extension -->
<Button>
    <PathIcon Data="{pia:IconGeometry Icon=gear, IconType=bold}"
              Foreground="Red" Width="20" Height="20"/>
</Button>

<!-- DrawingImage: set Brush, or it renders BLACK (invisible on dark backgrounds) -->
<Image Source="{pia:IconSource Icon=house, IconType=regular, Brush=SteelBlue}"
       Width="24" Height="24"/>
```

## C# usage

```csharp
using PhosphorIconsAvalonia;   // Icon, IconType, IconService (all static, no instance/DI)
using Avalonia.Media;          // Brushes, Geometry, DrawingImage

// As an Image source — pass the brush explicitly:
MyImage.Source = IconService.CreateDrawingImage(Icon.house, IconType.regular, Brushes.SteelBlue);

// As geometry (e.g. Path.Data / PathIcon.Data); color via the host control's Foreground:
Geometry g = IconService.CreateGeometry(Icon.gear, IconType.bold);
```

## Styles (`IconType`)

Members are **lowercase**: `thin`, `light`, `regular`, `bold`, `fill`. Default is `regular`. Write `IconType=bold` in XAML, `IconType.bold` in C#.

## Finding an icon name

Names come from the `Icon` enum: **lowercase, words joined by underscores** (e.g. `arrow_bend_double_up_left`, `youtube_logo`). Don't guess — verify against the bundled `icon-names.txt` in this skill's folder:

```bash
grep -i gear icon-names.txt      # → gear_fine, gear_six, gear, user_circle_gear, user_gear
```

- Browse visually at **phosphoricons.com**, then convert the displayed kebab-case name to the enum member: `arrow-bend-double-up-left` → `arrow_bend_double_up_left`.
- The `lock` icon is a C# keyword: write **`Icon.@lock`** in C# (in XAML just `Icon=lock`).
- `icon-names.txt` lists all 1,512 names, one per line, in enum order.

## Common mistakes

| Symptom | Cause / fix |
|---------|-------------|
| Icon invisible | `IconSource` `Brush` defaults to **black** → set `Brush=` on dark backgrounds. For `IconGeometry`, color comes from the host control's `Foreground`, not the extension. |
| `InvalidOperationException` at runtime | Icon/style resource not found — the enum name is wrong. Verify against `icon-names.txt`. |
| Name not accepted | The enum uses `under_scores`; on-disk SVGs use hyphens, but you always pass the **enum** form (underscores). |
| `IconType.Bold` won't compile | `IconType` members are lowercase: `IconType.bold`. |
| Icon looks truncated | Only the first `<path>` of a multi-path SVG is rendered (library limitation). |

Building the surrounding Avalonia app? See the `avalonia` skill for project/XAML/DI setup.
