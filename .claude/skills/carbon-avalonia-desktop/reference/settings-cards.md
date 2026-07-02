# SettingsCard & SettingsCardExpander

Both are `TemplatedControl`s in `Carbon.Avalonia.Desktop.Controls`
(`xmlns:carbon="using:Carbon.Avalonia.Desktop.Controls"`). Use them to build settings pages: an icon +
header + description row, with either an inline control, a click command, or expandable content.

## SettingsCard

Properties: `Header` : `string?`, `Description` : `string?`, `IconData` : `Geometry?`,
`Content` : `object?` (the `[Content]` property), `Command` : `ICommand?`,
`CommandParameter` : `object?`.

**Two modes:**
- **Content mode** — put a control inside; it renders on the right of the row.
- **Command mode** — leave `Content` empty and set `Command`; the whole card becomes a clickable button
  (pseudo-classes `:hasContent`, `:pressed` reflect state).

## SettingsCardExpander

Properties: `Header`, `Description`, `IconData`, `Content` (`[Content]`), plus
`IsExpanded` : `bool` (default `false`). The content shows/hides on expand. Also handy as a plain section
container (set `IsExpanded="True"`).

## Example

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:carbon="using:Carbon.Avalonia.Desktop.Controls"
             xmlns:pia="using:PhosphorIconsAvalonia.Markup"
             xmlns:vm="using:MyApp.ViewModels"
             x:Class="MyApp.Views.SettingsPageView"
             x:DataType="vm:SettingsPageViewModel">
  <ScrollViewer>
    <StackPanel Margin="24" Spacing="16">

      <!-- Content mode: inline control -->
      <carbon:SettingsCard Header="Theme"
                           Description="Switch between light and dark mode"
                           IconData="{pia:IconGeometry Icon=moon}">
        <ToggleSwitch IsChecked="{Binding IsDarkTheme, Mode=TwoWay}"
                      OnContent="Dark" OffContent="Light" />
      </carbon:SettingsCard>

      <!-- Command mode: whole card is clickable -->
      <carbon:SettingsCard Header="Appearance"
                           Description="Change theme, colors, and display options"
                           IconData="{pia:IconGeometry Icon=palette}"
                           Command="{Binding CardClickedCommand}"
                           CommandParameter="Appearance" />

      <!-- Expander with arbitrary content -->
      <carbon:SettingsCardExpander Header="Preferences"
                                   Description="Configure display and behavior options"
                                   IconData="{pia:IconGeometry Icon=gear}">
        <StackPanel Spacing="12">
          <TextBox PlaceholderText="Display name" />
          <CheckBox Content="Enable notifications" />
        </StackPanel>
      </carbon:SettingsCardExpander>

    </StackPanel>
  </ScrollViewer>
</UserControl>
```

ViewModel (semi-auto properties + a parameterized command):

```csharp
public class SettingsPageViewModel : ObservableObject
{
    public SettingsPageViewModel() => CardClickedCommand = new RelayCommand<string?>(CardClicked);

    public bool IsDarkTheme { get; set { if (SetProperty(ref field, value)) ApplyTheme(value); } }
    public IRelayCommand CardClickedCommand { get; }

    private void CardClicked(string? key) => LastAction = $"Clicked: {key}";
    public string? LastAction { get; private set => SetProperty(ref field, value); }
}
```

## Common mistakes

- Setting **both** `Content` and `Command` on a `SettingsCard` — command mode only triggers when
  `Content` is empty; put a `Button` in the content if you need a click target alongside other content.
- `IconData` expects a `Geometry` — use `{pia:IconGeometry Icon=…}` or `Geometry.Parse(...)`, not a
  string.
