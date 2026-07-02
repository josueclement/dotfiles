# ContentDialog, Overlay & InfoBar (host services)

Three service+host controls that share one pattern:

1. Place the host control (`ContentDialog` / `Overlay` / `InfoBar`) in MainWindow's root `Panel`,
   `x:Name`d (see `setup.md`).
2. `RegisterHost(...)` it once at startup in `App.OnFrameworkInitializationCompleted`.
3. Inject the service into a ViewModel and call `await service.ShowAsync(...)`.

Calling `ShowAsync` before `RegisterHost` throws `InvalidOperationException`.

---

## ContentDialog — modal dialog

**`IContentDialogService`** (`…Services`):
- `void RegisterHost(ContentDialog dialog)`
- `Task<DialogResult> ShowMessageAsync(string title, string message, string closeButtonText = "OK")`
- `Task<DialogResult> ShowAsync(Action<ContentDialog> configure)`
- `Task HideAsync()`

**`ContentDialog`** props (set inside `configure`): `Title`, `Content` (`object?` — string or any
control), `PrimaryButtonText` / `SecondaryButtonText` / `CloseButtonText`, matching
`…ButtonCommand`s and `Is…ButtonEnabled` flags, `DefaultButton` (`None`|`Primary`|`Secondary`|`Close`),
`IconData` : `Geometry?`, `IconBrush` : `IBrush?`. Escape / overlay-click close with `None`.

**Enums** (`…Controls.ContentDialog`): `DialogResult { None, Primary, Secondary, Close }`,
`DefaultButton { None, Primary, Secondary, Close }`.

For a **Yes/No confirmation**, set `PrimaryButtonText = "Yes"` and `CloseButtonText = "No"`, then treat
**only `DialogResult.Primary`** as confirmed — dismissing with Esc or an overlay click returns `None`
(not `Close`), so `== DialogResult.Primary` is the correct check.

`ShowAsync` resets the host to a clean state before applying your `configure`, so only set what you need.

```csharp
// Simple confirmation
var result = await _dialogService.ShowAsync(dialog =>
{
    dialog.Title = "Information";
    dialog.Content = "This is a simple message dialog.";
    dialog.IconData = IconService.CreateGeometry(Icon.info, IconType.regular);
    dialog.PrimaryButtonText = "OK";
});
if (result == DialogResult.Primary) { /* ... */ }

// Rich content with three buttons + a coloured icon
await _dialogService.ShowAsync(dialog =>
{
    dialog.Title = "Confirm Action";
    dialog.IconData = IconService.CreateGeometry(Icon.warning, IconType.regular);
    dialog.IconBrush = new SolidColorBrush(Color.Parse("#F59E0B"));
    dialog.Content = new StackPanel { Spacing = 8, Children =
        { new TextBlock { Text = "Are you sure?", TextWrapping = TextWrapping.Wrap } } };
    dialog.PrimaryButtonText = "Confirm";
    dialog.SecondaryButtonText = "Maybe Later";
    dialog.CloseButtonText = "Cancel";
});
```

You can capture input by putting a control in `Content` and reading it after the await:
`var box = new TextBox { PasswordChar = '•' }; dialog.Content = box; … if (result == DialogResult.Primary) use(box.Text);`

---

## Overlay — modal busy/progress layer

**`IOverlayService`** (`…Services`): `void RegisterHost(Overlay presenter)`,
`Task ShowAsync(Control control)`, `Task HideAsync()`. **`Overlay`** props: `IsOpen` : `bool`,
`OverlayBrush` : `IBrush?` (default translucent black).

`ShowAsync` takes any `Control` as content — typically a small progress card you build and then mutate
while a task runs, then `HideAsync()`.

```csharp
var card = new ProgressCard { Title = "Processing", IsIndeterminate = true, Message = "Starting…" };
await _overlayService.ShowAsync(card);
try
{
    card.IsIndeterminate = false; card.Progress = 50; card.Message = "Halfway…";
    await DoWorkAsync();
}
finally { await _overlayService.HideAsync(); }
```

`ProgressCard` here is your own `ContentControl` subclass (the tester's `ProgressOverlayCard` exposes
`Title`, `Message`, `IsIndeterminate`, `Progress`, `Minimum`, `Maximum` StyledProperties and templates a
`ProgressBar`). Guard the command with an `IsBusy` flag and `NotifyCanExecuteChanged()` to prevent
re-entry.

---

## InfoBar — inline notification

**`IInfoBarService`** (`…Services`): `void RegisterHost(InfoBar infoBar)`,
`Task ShowAsync(Action<InfoBar>? configure = null)`, `Task HideAsync()`. **`InfoBar`** props: `Title`,
`Message`, `Severity` : `InfoBarSeverity`, `IsOpen`. **Enum** (`…Controls.InfoBar`):
`InfoBarSeverity { Info, Success, Warning, Error }`.

```csharp
await _infoBarService.ShowAsync(o =>
{
    o.Title = "Success";
    o.Message = "The operation completed successfully.";
    o.Severity = InfoBarSeverity.Success;
});
// later: await _infoBarService.HideAsync();
```

---

## Injecting the services

```csharp
public class MyPageViewModel(
    IContentDialogService dialogService,
    IOverlayService overlayService,
    IInfoBarService infoBarService) : ObservableObject { /* store & use */ }
```

## Common mistakes

- **`ShowAsync` before `RegisterHost`** → `InvalidOperationException`. Wire hosts at startup.
- Reusing one dialog config across calls — not needed; `ShowAsync` resets the host each time.
- Not wrapping overlay work in `try/finally` — an exception mid-task leaves the overlay stuck open.
- `using Carbon.Avalonia.Desktop.Controls.ContentDialog;` is required to see `DialogResult` /
  `DefaultButton`; `…Controls.InfoBar;` for `InfoBarSeverity`.
