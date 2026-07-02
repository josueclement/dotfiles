# Docking

IDE-style dockable panes in `Carbon.Avalonia.Desktop.Controls.Docking`
(`xmlns:carbon="using:Carbon.Avalonia.Desktop.Controls.Docking"`). The easiest path is **declarative**:
build a `DockLayoutNode` model tree in the ViewModel and bind it to `DockingHost.LayoutRoot`.

## Public API

**`DockingHost`** (`TemplatedControl`): `LayoutRoot` : `DockLayoutNode?`, `Panes` :
`AvaloniaList<DockPane>` (`[Content]`), methods `SetRootLayout(Control root)` (after template applied),
`ClosePane(DockPane)`.

**Layout model POCOs** (`DockLayoutNode.cs`, not controls):
- `DockSplitModel` : `Orientation` (`Horizontal`|`Vertical`, default Horizontal), `First` / `Second` :
  `DockLayoutNode?`, `FirstSize` / `SecondSize` : `GridLength` (default `1*`).
- `DockTabGroupModel` : `Panes` : `AvaloniaList<DockPaneModel>`, `SelectedPane` : `DockPaneModel?`.
- `DockPaneModel` : `Header` : `string`, `Content` : `object?`, `CanClose` : `bool` (default true),
  `CanMove` : `bool` (default true).

Content-side controls (used by templates / imperative layout): `DockPane`, `DockTabGroup`,
`DockSplitContainer`. Enum `DockPosition { Center, Left, Right, Top, Bottom }`.

`DockPaneModel.Content` is typically a ViewModel; provide a `DataTemplate` for its type so the pane renders
your view.

## Example

```xml
<UserControl xmlns:carbon="using:Carbon.Avalonia.Desktop.Controls.Docking"
             x:DataType="vm:WorkspaceViewModel" ...>
  <UserControl.DataTemplates>
    <DataTemplate DataType="vm:PaneContentViewModel">
      <Border Padding="16">
        <TextBlock Text="{Binding Title}" FontSize="18" FontWeight="SemiBold" />
      </Border>
    </DataTemplate>
  </UserControl.DataTemplates>

  <carbon:DockingHost LayoutRoot="{Binding RootLayout}" />
</UserControl>
```

```csharp
using Avalonia.Controls;   // GridLength, GridUnitType
using Avalonia.Layout;     // Orientation
using Carbon.Avalonia.Desktop.Controls.Docking;

public class WorkspaceViewModel : ObservableObject
{
    public DockLayoutNode RootLayout { get; }

    public WorkspaceViewModel()
    {
        var solution = Pane("Solution", canClose: false, canMove: false);
        var doc1 = Pane("Doc1"); var doc2 = Pane("Doc2");
        var output = Pane("Output"); var debug = Pane("Debug");

        var center = new DockTabGroupModel { SelectedPane = doc1 };
        center.Panes.Add(doc1); center.Panes.Add(doc2);

        var solutionGroup = new DockTabGroupModel { SelectedPane = solution };
        solutionGroup.Panes.Add(solution);

        var bottom = new DockTabGroupModel { SelectedPane = output };
        bottom.Panes.Add(output); bottom.Panes.Add(debug);

        var top = new DockSplitModel
        {
            Orientation = Orientation.Horizontal,
            First = solutionGroup, Second = center,
            FirstSize = new GridLength(200, GridUnitType.Pixel),
            SecondSize = new GridLength(1, GridUnitType.Star)
        };

        RootLayout = new DockSplitModel
        {
            Orientation = Orientation.Vertical,
            First = top, Second = bottom,
            FirstSize = new GridLength(1, GridUnitType.Star),
            SecondSize = new GridLength(200, GridUnitType.Pixel)
        };
    }

    private static DockPaneModel Pane(string header, bool canClose = true, bool canMove = true) =>
        new() { Header = header, CanClose = canClose, CanMove = canMove,
                Content = new PaneContentViewModel { Title = header } };
}

public class PaneContentViewModel { public string Title { get; set; } = ""; }
```

## Common mistakes

- Forgetting a `DataTemplate` for the pane `Content` type → the pane shows the VM's `ToString()`.
- Calling `SetRootLayout` before the template is applied → `InvalidOperationException`. Prefer binding
  `LayoutRoot` instead.
- Mixing up the model POCOs (`Dock*Model`) with the content controls (`DockPane`, `DockTabGroup`) — bind
  the **models** to `LayoutRoot`.
