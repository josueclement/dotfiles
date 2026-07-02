# Theming

Colors live in `Themes/Colors.axaml` (a `ResourceDictionary.ThemeDictionaries` with `Dark` and `Light`
variants); brushes in `Themes/Brushes.axaml` bind each brush to its color via `{DynamicResource}`. Both
are pulled in by `Themes/Fluent.axaml` (the single include you merge — see `setup.md`).

## Rules

- Reference **brushes**, not colors, and always with `{DynamicResource Carbon…Brush}` (never
  `StaticResource`) so the UI re-themes at runtime.
- Every `…Color` key has a matching `…Brush` key (e.g. `CarbonSurfaceColor` → `CarbonSurfaceBrush`).
- Switch theme at runtime by setting `Application.Current.RequestedThemeVariant`
  (`ThemeVariant.Dark` / `ThemeVariant.Light`).

## Brush key catalog (drop the `Color`, add `Brush`)

- **Surfaces**: `CarbonBackground`, `CarbonSurface`, `CarbonSurfaceHigh`, `CarbonSurfaceLow`
- **Borders**: `CarbonBorder`, `CarbonBorderSubtle`
- **Foreground/text**: `CarbonForeground`, `CarbonForegroundSecondary`, `CarbonForegroundTertiary`
- **Accent/selection**: `CarbonAccent`, `CarbonAccentHover`, `CarbonSelection`
- **Inputs**: `CarbonInputBackground`, `CarbonInputBackgroundFocused`, `CarbonInputBackgroundHover`
- **Interaction**: `CarbonHover`, `CarbonPressed`, `CarbonOverlay`
- **Status**: `CarbonSuccess`, `CarbonWarning`, `CarbonError`
- **Status surfaces** (`…Background` / `…Border`): `CarbonInfo*`, `CarbonSuccess*`, `CarbonWarning*`, `CarbonError*`
- **Calendar**: `CarbonCalendarToday`, `CarbonCalendarSelected`, `CarbonCalendarOutOfMonth`,
  `CarbonCalendarAppointment`, `CarbonCalendarGridLine`, `CarbonCalendarCurrentTime`

## Standard-control theming

`Brushes.axaml` also overrides Avalonia FluentTheme keys (`TextControlBackground*`,
`TextControlBorderBrush*`, `ComboBoxBackground*`, …) so plain `TextBox`/`ComboBox`/etc. match the Carbon
look automatically once the theme is merged — you don't style them yourself.

## Example

```xml
<Border Background="{DynamicResource CarbonSurfaceBrush}"
        BorderBrush="{DynamicResource CarbonBorderBrush}"
        BorderThickness="1" CornerRadius="8" Padding="16">
  <TextBlock Text="Hello" Foreground="{DynamicResource CarbonForegroundSecondaryBrush}" />
</Border>
```

Toggle the whole app theme (from a ViewModel):

```csharp
Application.Current!.RequestedThemeVariant =
    isDark ? ThemeVariant.Dark : ThemeVariant.Light;   // using Avalonia.Styling;
```

## Common mistakes

- Using `StaticResource` → the color freezes at load and won't follow a theme switch.
- Hard-coding hex colors instead of the `Carbon*` brushes → inconsistent look and no theme reactivity.
