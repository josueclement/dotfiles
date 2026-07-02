# Ribbon

Office-style ribbon in `Carbon.Avalonia.Desktop.Controls.Ribbon`
(`xmlns:carbon="using:Carbon.Avalonia.Desktop.Controls.Ribbon"`). Hierarchy:
`Ribbon` → `RibbonTab`s → `RibbonGroup`s → buttons.

## Public API

- **`Ribbon`** : `Tabs` : `AvaloniaList<RibbonTab>` (`[Content]`), `SelectedTab` : `RibbonTab?` (TwoWay),
  `SelectedIndex` : `int` (TwoWay, default 0).
- **`RibbonTab`** : `Header` : `string?`, `Groups` : `AvaloniaList<RibbonGroup>` (`[Content]`).
- **`RibbonGroup`** : `Header` : `string?`, `Items` : `AvaloniaList<Control>` (`[Content]`).
- **`RibbonButton`** : `Header`, `IconData` : `Geometry?`, `Command` : `ICommand?`, `CommandParameter`.
- **`RibbonToggleButton`** : as button + `IsChecked` : `bool` (TwoWay).
- **`RibbonDropDownButton`** : `Header`, `IconData`, `IsDropDownOpen` : `bool` (TwoWay),
  `Items` : `AvaloniaList<RibbonMenuItem>` (`[Content]`).
- **`RibbonMenuItem`** (an `AvaloniaObject`, not a control) : `Header`, `IconData`, `Command`,
  `CommandParameter`.

## Example

```xml
<DockPanel>
  <carbon:Ribbon DockPanel.Dock="Top" SelectedIndex="{Binding SelectedTabIndex, Mode=TwoWay}">
    <carbon:RibbonTab Header="Home">
      <carbon:RibbonGroup Header="File">
        <carbon:RibbonButton Header="New"  IconData="{pia:IconGeometry Icon=file_text}"   Command="{Binding NewCommand}" />
        <carbon:RibbonButton Header="Save" IconData="{pia:IconGeometry Icon=floppy_disk}" Command="{Binding SaveCommand}" />
      </carbon:RibbonGroup>

      <carbon:RibbonGroup Header="Clipboard">
        <carbon:RibbonDropDownButton Header="Paste" IconData="{pia:IconGeometry Icon=clipboard}">
          <carbon:RibbonMenuItem Header="Paste"         Command="{Binding PasteCommand}" />
          <carbon:RibbonMenuItem Header="Paste Special" Command="{Binding PasteSpecialCommand}" />
        </carbon:RibbonDropDownButton>
      </carbon:RibbonGroup>

      <carbon:RibbonGroup Header="Format">
        <carbon:RibbonToggleButton Header="Bold" IconData="{pia:IconGeometry Icon=text_b}"
                                   IsChecked="{Binding IsBoldActive}" Command="{Binding ToggleBoldCommand}" />
      </carbon:RibbonGroup>
    </carbon:RibbonTab>

    <carbon:RibbonTab Header="Insert">
      <carbon:RibbonGroup Header="Elements">
        <carbon:RibbonButton Header="Image" IconData="{pia:IconGeometry Icon=image}" Command="{Binding InsertImageCommand}" />
      </carbon:RibbonGroup>
    </carbon:RibbonTab>
  </carbon:Ribbon>

  <!-- your content fills the rest of the DockPanel -->
</DockPanel>
```

ViewModel is plain `ObservableObject` — one `RelayCommand` per button, plus `bool` toggle properties and
`SelectedTabIndex`:

```csharp
public class EditorRibbonViewModel : ObservableObject
{
    public EditorRibbonViewModel()
    {
        NewCommand        = new RelayCommand(() => Status = "New");
        ToggleBoldCommand = new RelayCommand(() => Status = IsBoldActive ? "Bold on" : "Bold off");
    }
    public int SelectedTabIndex { get; set => SetProperty(ref field, value); }
    public bool IsBoldActive    { get; set => SetProperty(ref field, value); }
    public string Status        { get; set => SetProperty(ref field, value); } = "Ready";
    public IRelayCommand NewCommand { get; }
    public IRelayCommand ToggleBoldCommand { get; }
}
```

## Common mistakes

- Wrong `xmlns` — ribbon types are under `…Controls.Ribbon`, not `…Controls`.
- Putting arbitrary controls directly in a `RibbonDropDownButton` — its content is `RibbonMenuItem`s.
- `RibbonMenuItem` isn't a `Control`; you can't style it like one, only set its properties.
