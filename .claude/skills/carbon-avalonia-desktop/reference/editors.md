# Editors

Typed input controls in `Carbon.Avalonia.Desktop.Controls.Editors`
(`xmlns:editors="using:Carbon.Avalonia.Desktop.Controls.Editors"`). All derive from `TextBox`, so every
`TextBox` member (e.g. `PlaceholderText`, `PasswordChar`, `Text`) is available, plus editor extras.

## The controls

| Editor | Binds | Notes |
|---|---|---|
| `IntEditor` `UIntEditor` `LongEditor` `ULongEditor` `ShortEditor` `UShortEditor` | `Value` : `T?` | integer types |
| `SingleEditor` `DoubleEditor` `DecimalEditor` | `Value` : `T?` | floating/decimal; honor `FormatString` |
| `TextEditor` | `Text` (TextBox) | single-line string; no `Value` |
| `MultiLineTextEditor` | `Text` | wraps, `AcceptsReturn=true` |
| `HexadecimalEditor` `Base64Editor` | `Value` : `byte[]?` | binary; render multi-line |

## Common properties

From `BaseEditor` (all editors): `Title` : `string?` (label above the field), `Unit` : `string?`
(trailing unit chip), `LeadingContent` : `object?` (icon/control at the start), `ActionContent` :
`object?` (control at the end), `HasValidationError` : `bool` + `ValidationErrorMessage` : `string?`
(drives the `:error` pseudo-class), `SelectAllTextOnFocus` : `bool` (default `true`).

Numeric editors add (`BaseEditor<T>`): `Value` : `T?` (TwoWay, data-validated), `FormatString` :
`string?` (standard .NET numeric format, e.g. `"N2"`), `NullWhenEmpty` : `bool` (default `false` — when
true, empty text yields `null` instead of `default(T)`). Parsing/formatting uses **InvariantCulture**.

## Example — XAML

```xml
<StackPanel xmlns:editors="using:Carbon.Avalonia.Desktop.Controls.Editors"
            xmlns:pia="using:PhosphorIconsAvalonia.Markup" Spacing="8">

  <editors:ShortEditor Title="Temperature (short)" Value="{Binding ShortValue}" Unit="°C" />

  <editors:UShortEditor Title="Port" Value="{Binding Port}" NullWhenEmpty="True" Unit="port" />

  <editors:DoubleEditor Title="Distance" Value="{Binding Distance}" FormatString="N2" Unit="mm" />

  <editors:TextEditor Title="Search" Text="{Binding Query}">
    <editors:TextEditor.LeadingContent>
      <PathIcon Data="{pia:IconGeometry Icon=magnifying_glass}" Width="16" Height="16" />
    </editors:TextEditor.LeadingContent>
  </editors:TextEditor>

  <editors:TextEditor Title="With action" Text="{Binding Name}">
    <editors:TextEditor.ActionContent>
      <Button Content="Go" Padding="6,2" FontSize="12" />
    </editors:TextEditor.ActionContent>
  </editors:TextEditor>

  <editors:TextEditor Title="Password" PasswordChar="*" />

  <editors:HexadecimalEditor Title="Raw bytes" Value="{Binding Bytes}" />
  <editors:Base64Editor      Title="Encoded"   Value="{Binding Bytes}" />
</StackPanel>
```

## Example — validation via `ObservableValidator`

`Value` is registered with data validation, so DataAnnotations on the bound property light up the
`:error` state automatically. Derive the ViewModel from `ObservableValidator` and pass `true` to
`SetProperty` to trigger validation:

```csharp
using System.ComponentModel.DataAnnotations;

public class FormViewModel : ObservableValidator
{
    [Range(0, 10000)]
    public int? IntValue { get; set { if (SetProperty(ref field, value, true)) { /* react */ } } } = 42;

    [Required, MaxLength(100)]
    public string? TextValue { get; set => SetProperty(ref field, value, true); } = "Hello";

    [Range(typeof(decimal), "0.01", "99999.99")]
    public decimal? Price { get; set => SetProperty(ref field, value, true); } = 99.95m;

    public byte[]? Bytes { get; set => SetProperty(ref field, value); } = [0x0A, 0xFF, 0x1B];
}
```

## Common mistakes

- Binding a numeric editor's `Text` instead of `Value` — bind **`Value`** (typed); `Text` is the raw
  string the base `TextBox` manages.
- Expecting culture-specific parsing — editors use InvariantCulture; `FormatString` controls display
  only.
- For validation styling, the property must be validated (`SetProperty(ref field, value, true)` on an
  `ObservableValidator`), or set `HasValidationError`/`ValidationErrorMessage` yourself.
