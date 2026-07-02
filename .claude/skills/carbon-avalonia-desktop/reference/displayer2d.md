# Displayer2D

Pan/zoom 2D canvas that renders a collection of shapes (world coordinates), with an interaction strategy.
Namespaces: `Carbon.Avalonia.Desktop.Controls.Displayer2D` (control, `DrawingObject`, `UserInteraction`,
`DragInteraction`), `.Shapes` (concrete shapes), `.Groups` (shape groups).
`xmlns:d2d="using:Carbon.Avalonia.Desktop.Controls.Displayer2D"`.

## Public API

**`Displayer2D`** (`TemplatedControl`):
- `DrawingObjects` : `ObservableCollection<DrawingObject>?`
- `DrawingObjectGroups` : `ObservableCollection<DrawingObjectGroup>?`
- `UserInteraction` : `UserInteraction?` (pan/zoom/drag strategy)
- `BackgroundImage` : `IImage?`
- `ZoomFactor` / `PanX` / `PanY` : `double`
- `WorldMousePosition` : `Point?` (read-only direct property — subscribe via `WorldMousePositionProperty`)
- methods: `Point WorldToCanvas(Point)`, `Point CanvasToWorld(Point)`,
  `void ZoomToFit(Rect worldBounds, double padding = 20)`, `void ZoomToFit(double padding = 20)`
  (fits `BackgroundImage`), `void Refresh()`

**Shapes** (`…Shapes`, all `: Shape : DrawingObject`): `RectangleShape`, `EllipseShape`, `CircleShape`,
`LineShape` (`X2`,`Y2`), `PathShape` (`Geometry`), `TextShape` (`Text`,`FontSize`,`FontFamily`,
`FontWeight`,`Foreground`), `ImageShape` (`Source`). Shared shape props: `X`,`Y`,`Width`,`Height`,
`Rotation`,`ZIndex`,`IsVisible`,`IsFixed` (ignores pan/zoom — for HUD overlays), `Fill`,`Stroke`,
`StrokeThickness`,`FillHover`,`StrokeHover`,`IsMovable`. `EllipseShape`/`CircleShape` add
`CenterX`/`CenterY`.

**Interaction**: `UserInteraction` is the no-op base (override `OnMouseDown/Move/Up/Wheel/…`);
`DragInteraction` (sealed) implements pan, wheel-zoom, double-click zoom-to-fit, and dragging
`IsMovable` shapes.

**Groups** (`…Groups`, `: DrawingObjectGroup`): `LineMovingObjectGroup(x1,y1,x2,y2)`,
`RectangleRoiGroup(ax,ay,bx,by,halfWidth)`; or subclass `DrawingObjectGroup` (populate `Items`, override
`RecalculateCoordinates()` and `UnregisterEvents()`).

## Example — ViewModel supplies shapes

```csharp
using Carbon.Avalonia.Desktop.Controls.Displayer2D;
using Carbon.Avalonia.Desktop.Controls.Displayer2D.Shapes;

public class CanvasViewModel : ObservableObject
{
    public ObservableCollection<DrawingObject> Objects { get; }
    public UserInteraction Interaction { get; } = new DragInteraction();

    public CanvasViewModel() => Objects =
    [
        new RectangleShape { X = 20, Y = 20, Width = 150, Height = 80, Rotation = 45,
            Fill = new SolidColorBrush(Color.Parse("#3574F0")),
            FillHover = new SolidColorBrush(Color.Parse("#7AB0FF")) },
        new LineShape { X = 20, Y = 180, X2 = 340, Y2 = 180,
            Stroke = new SolidColorBrush(Color.Parse("#F75464")), StrokeThickness = 3 },
        new TextShape { X = 14, Y = 14, Text = "HUD label", IsFixed = true, ZIndex = 100,
            Foreground = new SolidColorBrush(Color.Parse("#BCBEC4")) },
    ];
}
```

```xml
<d2d:Displayer2D DrawingObjects="{Binding Objects}"
                 UserInteraction="{Binding Interaction}"
                 Margin="24" />
```

## Example — coordinate readout (code-behind)

`WorldMousePosition` is read-only; to feed live coordinates, convert the pointer position yourself and
listen for changes:

```csharp
private void OnLoaded(object? s, RoutedEventArgs e)
{
    Displayer.ZoomToFit();                                    // fit BackgroundImage
    Displayer.PropertyChanged += (_, ev) =>
    {
        if (ev.Property == Displayer2D.WorldMousePositionProperty && DataContext is MyVm vm)
            vm.UpdateCoordinates((Point?)ev.NewValue);
    };
    RootGrid.PointerMoved += (_, ev) =>
        Displayer.WorldMousePosition = Displayer.CanvasToWorld(ev.GetPosition(Displayer));
}
```

## Common mistakes

- No `UserInteraction` set → the canvas won't pan/zoom/drag. Assign `new DragInteraction()`.
- Expecting shapes to move with drag without `IsMovable = true`.
- Confusing world vs canvas coordinates — shape `X`/`Y` are world coords; use `CanvasToWorld` /
  `WorldToCanvas` to translate pointer positions.
