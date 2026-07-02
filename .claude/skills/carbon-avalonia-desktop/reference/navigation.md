# Navigation

`NavigationView` (a `TemplatedControl`) renders a list of `NavigationItem`s (main + footer) and a
selection. `INavigationService` owns the item collections, the current page, and a **`PageFactory`** that
turns a selected `NavigationItem` into a `Control`. Bind the view to the service; drive everything from a
ViewModel.

## Public API

**`NavigationView`** (`…Controls.Navigation`): `Items` / `FooterItems` : `IReadOnlyList<NavigationItem>?`,
`SelectedItem` : `NavigationItem?` (TwoWay), `Logo` : `object?`, `Orientation` : `NavigationOrientation`
(`Vertical` | `Horizontal`, default `Vertical`).

**`NavigationItem`** (`…Controls.Navigation`): `Header` : `string?`, `IconData` : `Geometry?`,
`PageType` : `Type`, `PageViewModelType` : `Type`.

**`INavigationService`** (`…Services`):
- `ObservableCollection<NavigationItem> Items { get; }`, `FooterItems { get; }`
- `NavigationItem? SelectedItem { get; set; }` (setting it navigates)
- `Control? CurrentPage { get; }`
- `Task NavigateToAsync(Control page, object? parameter = null)`
- `Func<NavigationItem, Control> PageFactory { get; set; }`
- `event EventHandler<NavigationFailedEventArgs>? NavigationFailed`

Navigation is serialized with a `SemaphoreSlim(1,1)` — concurrent navigations are dropped. The default
`PageFactory` uses `Activator.CreateInstance`; override it to resolve from DI (below).

## Page lifecycle — `INavigationViewModel`

A page's ViewModel (or view) can implement `INavigationViewModel` (`…Services`):
- `Task<bool> OnDisappearingAsync()` — return **`false` to cancel** navigating away.
- `Task OnAppearingAsync(object? parameter = null)` — receives the `NavigateToAsync` parameter.

## Example — DI-resolved pages

MainWindow XAML binds the view to the service (see `setup.md`). In `MainWindowViewModel`, set a
DI-backed `PageFactory` and add items:

```csharp
using PhosphorIconsAvalonia;                 // IconService, Icon, IconType
using Microsoft.Extensions.DependencyInjection;

public MainWindowViewModel(IServiceProvider services, INavigationService navigation)
{
    Navigation = navigation;

    Navigation.PageFactory = navItem =>
    {
        var page = (Control)services.GetRequiredService(navItem.PageType);
        page.DataContext = services.GetRequiredService(navItem.PageViewModelType);
        return page;
    };

    Navigation.Items.Add(new NavigationItem
    {
        Header = "Home",
        IconData = IconService.CreateGeometry(Icon.house, IconType.regular),  // PhosphorIconsAvalonia
        PageType = typeof(HomePageView),
        PageViewModelType = typeof(HomePageViewModel)
    });
    Navigation.FooterItems.Add(new NavigationItem
    {
        Header = "Settings",
        IconData = IconService.CreateGeometry(Icon.gear, IconType.regular),
        PageType = typeof(SettingsPageView),
        PageViewModelType = typeof(SettingsPageViewModel)
    });

    Navigation.SelectedItem = Navigation.Items[0];   // show the first page on startup
}
public INavigationService Navigation { get; }
```

Setting `SelectedItem` navigates. To open a page that has no nav item, call
`await Navigation.NavigateToAsync(page)` (see below).

`IconData` can also be a raw `Geometry.Parse("M3 17…")` if you don't want the icon pack.

## Example — programmatic navigation with cancel

```csharp
public class EditorPageViewModel : ObservableObject, INavigationViewModel
{
    public async Task<bool> OnDisappearingAsync()
    {
        if (!HasUnsavedChanges) return true;
        var result = await _dialogService.ShowAsync(d =>
        {
            d.Title = "Unsaved Changes";
            d.Content = "Discard your changes and leave?";
            d.PrimaryButtonText = "Discard";
            d.SecondaryButtonText = "Keep Editing";
            d.DefaultButton = DefaultButton.Secondary;
        });
        return result == DialogResult.Primary;   // false ⇒ navigation cancelled
    }

    public Task OnAppearingAsync(object? parameter = null) => Task.CompletedTask;

    private async Task GoToSettings()               // navigate to a page with no nav item
    {
        var page = _services.GetRequiredService<SettingsPageView>();
        page.DataContext = _services.GetRequiredService<SettingsPageViewModel>();
        await _navigation.NavigateToAsync(page);
    }
}
```

## Common mistakes

- Forgetting to set `PageFactory` when you use DI — the default uses `Activator.CreateInstance`, which
  won't inject constructor dependencies.
- Blocking on navigation in a constructor with `.GetAwaiter().GetResult()` works for the initial page but
  prefer awaiting `NavigateToAsync` from an async context otherwise.
- Not subscribing to `NavigationFailed` — page-factory / lifecycle exceptions surface there
  (with a `Phase` string), not as thrown exceptions from `NavigateToAsync`.
