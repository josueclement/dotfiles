# Setup & bootstrap

How to stand up an Avalonia desktop app that consumes `Carbon.Avalonia.Desktop`, using
`Microsoft.Extensions.Hosting` + DI. All snippets are the working pattern from the library's own tester
app (`CarbonAvaloniaDesktopTester`).

> **Retrofitting an existing app?** You still need all five pieces below: the theme include (step 3),
> service registration (step 2), the host controls in your main window (step 4), and the one-time host
> wiring (step 4). If you already have an `IHost`, just add `AddCarbonServices()` to it.

## 1. Project file

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net10.0</TargetFramework>   <!-- or net8.0 -->
    <Nullable>enable</Nullable>
    <BuiltInComInteropSupport>true</BuiltInComInteropSupport>
    <AvaloniaUseCompiledBindingsByDefault>true</AvaloniaUseCompiledBindingsByDefault>
    <ApplicationIcon>Assets/app.ico</ApplicationIcon>   <!-- app/window icon: see the avalonia skill's "Application icon" section -->
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Carbon.Avalonia.Desktop" Version="0.2.0" />
    <PackageReference Include="Avalonia.Desktop" Version="12.0.2" />
    <PackageReference Include="Avalonia.Themes.Fluent" Version="12.0.2" />
    <PackageReference Include="Avalonia.Fonts.Inter" Version="12.0.2" />
    <PackageReference Include="Microsoft.Extensions.Hosting" Version="10.0.7" />
  </ItemGroup>
  <ItemGroup>
    <AvaloniaResource Include="Assets/**" />
  </ItemGroup>
</Project>
```

`CommunityToolkit.Mvvm`, `Microsoft.Extensions.DependencyInjection`, `PhosphorIconsAvalonia`, and
`Avalonia` itself arrive transitively — no need to reference them explicitly (add
`PhosphorIconsAvalonia`/`CommunityToolkit.Mvvm` explicitly only if you want to pin versions).

## 2. DI registration

The six Carbon services are singletons. This uses C# 14 extension members, but a plain static method
works identically.

```csharp
using Carbon.Avalonia.Desktop.Services;
using Microsoft.Extensions.DependencyInjection;

public static class ServiceCollectionExtensions
{
    extension(IServiceCollection services)
    {
        public void AddCarbonServices()
        {
            services.AddSingleton<IFileDialogService, FileDialogService>();
            services.AddSingleton<IFolderDialogService, FolderDialogService>();
            services.AddSingleton<INavigationService, NavigationService>();
            services.AddSingleton<IContentDialogService, ContentDialogService>();
            services.AddSingleton<IInfoBarService, InfoBarService>();
            services.AddSingleton<IOverlayService, OverlayService>();
        }
    }
}
```

Register your window, page views (Transient — a fresh instance per navigation), and ViewModels
(Singleton — state survives navigation), e.g. `services.AddSingleton<MainWindow>();
services.AddTransient<HomePageView>(); services.AddSingleton<HomePageViewModel>();`.

## 3. `Program.cs` — build the host, then start Avalonia

```csharp
using Avalonia;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

sealed class Program
{
    internal static IHost? AppHost { get; private set; }

    [STAThread]
    public static void Main(string[] args)
    {
        AppHost = Host.CreateDefaultBuilder(args)
            .ConfigureServices((_, services) =>
            {
                services.AddCarbonServices();
                // services.AddSingleton<MainWindow>(); ...pages, viewmodels...
            })
            .Build();

        AppHost.Start();
        BuildAvaloniaApp().StartWithClassicDesktopLifetime(args);

        // Avoid a shutdown deadlock: detach the Avalonia SynchronizationContext
        // before awaiting StopAsync (observed on Windows).
        SynchronizationContext.SetSynchronizationContext(null);
        AppHost.StopAsync().GetAwaiter().GetResult();
        AppHost.Dispose();
    }

    public static AppBuilder BuildAvaloniaApp()
        => AppBuilder.Configure<App>().UsePlatformDetect().WithInterFont().LogToTrace();
}
```

`AppHost` is exposed so `App.axaml.cs` can pull the provider. On Linux you may also want to swallow the
Tmds.DBus `TaskCanceledException` on shutdown (see the tester's `Program.cs`).

## 4. `App.axaml` + `App.axaml.cs`

`App.axaml` — base FluentTheme in `Styles`, the Carbon dictionary as a `ResourceInclude` in `Resources`:

```xml
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="MyApp.App"
             RequestedThemeVariant="Dark">
  <Application.Styles>
    <FluentTheme />
  </Application.Styles>
  <Application.Resources>
    <ResourceDictionary>
      <ResourceDictionary.MergedDictionaries>
        <ResourceInclude Source="avares://Carbon.Avalonia.Desktop/Themes/Fluent.axaml" />
      </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
  </Application.Resources>
</Application>
```

`App.axaml.cs` — resolve window/VM from DI and **register the host controls once**:

```csharp
public override void OnFrameworkInitializationCompleted()
{
    var services = Program.AppHost!.Services;

    if (ApplicationLifetime is IClassicDesktopStyleApplicationLifetime desktop)
    {
        var mainWindow = services.GetRequiredService<MainWindow>();
        mainWindow.DataContext = services.GetRequiredService<MainWindowViewModel>();

        services.GetRequiredService<IContentDialogService>().RegisterHost(mainWindow.HostDialog);
        services.GetRequiredService<IOverlayService>().RegisterHost(mainWindow.HostOverlay);
        services.GetRequiredService<IInfoBarService>().RegisterHost(mainWindow.HostInfoBar);
        services.GetRequiredService<IFileDialogService>().SetStorageProvider(mainWindow.StorageProvider);
        services.GetRequiredService<IFolderDialogService>().SetStorageProvider(mainWindow.StorageProvider);

        desktop.MainWindow = mainWindow;
    }
    base.OnFrameworkInitializationCompleted();
}
```

If you support the Avalonia designer (which bypasses `Program.Main`, leaving `AppHost` null), build a
fallback provider — see the tester's `BuildDesignerServices()`.

## 5. MainWindow — the host `Panel` pattern

The three host controls are siblings in a root `Panel` (which z-stacks its children) layered over your
content. The `x:Name`s must match what `App.axaml.cs` passes to `RegisterHost`.

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:vm="using:MyApp.ViewModels"
        xmlns:controls="using:Carbon.Avalonia.Desktop.Controls"
        xmlns:contentDialog="using:Carbon.Avalonia.Desktop.Controls.ContentDialog"
        xmlns:infoBar="using:Carbon.Avalonia.Desktop.Controls.InfoBar"
        xmlns:nav="using:Carbon.Avalonia.Desktop.Controls.Navigation"
        x:Class="MyApp.Views.MainWindow"
        x:DataType="vm:MainWindowViewModel"
        Icon="/Assets/app.ico"
        Background="{DynamicResource CarbonBackgroundBrush}">
  <Panel>
    <DockPanel>
      <nav:NavigationView DockPanel.Dock="Left"
                          Items="{Binding Navigation.Items}"
                          FooterItems="{Binding Navigation.FooterItems}"
                          SelectedItem="{Binding Navigation.SelectedItem}"
                          Orientation="Vertical" />
      <ContentControl Content="{Binding Navigation.CurrentPage}" />
    </DockPanel>

    <contentDialog:ContentDialog x:Name="HostDialog" />
    <controls:Overlay x:Name="HostOverlay" />
    <infoBar:InfoBar x:Name="HostInfoBar" />
  </Panel>
</Window>
```

Each `x:Name`d host becomes a field on the generated `MainWindow` partial class — that's why
`App.axaml.cs` can pass `mainWindow.HostDialog` / `HostOverlay` / `HostInfoBar` to `RegisterHost`
(you don't declare those fields yourself).

Note the **four separate `xmlns` prefixes** — `ContentDialog`, `Overlay`, `InfoBar` and `NavigationView`
live in different CLR namespaces. A single `using:Carbon.Avalonia.Desktop.Controls` will not resolve
`ContentDialog` or `InfoBar`.

## Common mistakes

- **`StyleInclude` for the theme** → runtime failure. `Themes/Fluent.axaml` is a `ResourceDictionary`;
  use `ResourceInclude` inside `Application.Resources`.
- **Calling `ShowAsync` before `RegisterHost`** → `InvalidOperationException`. Register hosts in
  `OnFrameworkInitializationCompleted` after the window is resolved.
- **Wrong package id** — it is `Carbon.Avalonia.Desktop` (with dots), not `CarbonAvaloniaDesktop`.
- **Forgetting `Avalonia.Desktop` / `Avalonia.Fonts.Inter`** — the library doesn't bring them; without
  them `StartWithClassicDesktopLifetime` / `WithInterFont()` won't resolve.
