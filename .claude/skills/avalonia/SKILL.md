---
name: avalonia
description: Use when creating or modifying an Avalonia application — project setup and packages, bootstrapping and DI/IHost wiring, XAML bindings, views and ViewModels, or desktop app lifetime.
---

# Avalonia 12 conventions

House rule: new Avalonia work targets **Avalonia 12** (stable; 12.0.5 as of 2026-07) — never start new work on 11. Apps target `net10.0` (Avalonia 12 requires .NET 8+). MVVM uses CommunityToolkit.Mvvm per the communitytoolkit-mvvm skill — not ReactiveUI.

## Packages (12.x template set)

```xml
<PackageReference Include="Avalonia" Version="12.0.5" />
<PackageReference Include="Avalonia.Desktop" Version="12.0.5" />
<PackageReference Include="Avalonia.Themes.Fluent" Version="12.0.5" />
<PackageReference Include="Avalonia.Fonts.Inter" Version="12.0.5" />
<!-- Dev tools: Avalonia.Diagnostics was REMOVED in 12 -->
<PackageReference Include="AvaloniaUI.DiagnosticsSupport" Version="2.2.3">
  <IncludeAssets Condition="'$(Configuration)' != 'Debug'">None</IncludeAssets>
  <PrivateAssets Condition="'$(Configuration)' != 'Debug'">All</PrivateAssets>
</PackageReference>
```

General csproj properties (LangVersion 14, nullable, no implicit usings): see dotnet-solution-setup.

## Bootstrapping — Avalonia lifetime + IHost

`await host.RunAsync()` alone never shows a window; keep `Main` synchronous and run Avalonia's classic desktop lifetime:

```csharp
// Program.cs
using System;
using Avalonia;

public static class Program
{
    [STAThread]
    public static void Main(string[] args) => BuildAvaloniaApp()
        .StartWithClassicDesktopLifetime(args);

    public static AppBuilder BuildAvaloniaApp()
        => AppBuilder.Configure<App>()
            .UsePlatformDetect()
#if DEBUG
            .WithDeveloperTools()   // 12: replaces window-level this.AttachDevTools()
#endif
            .WithInterFont()
            .LogToTrace();
}
```

```csharp
// App.axaml.cs
using Avalonia;
using Avalonia.Controls.ApplicationLifetimes;
using Avalonia.Markup.Xaml;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

public partial class App : Application
{
    private IHost? _host;

    public override void Initialize() => AvaloniaXamlLoader.Load(this);

    public override void OnFrameworkInitializationCompleted()
    {
        HostApplicationBuilder builder = Host.CreateApplicationBuilder();
        builder.Services.AddSingleton<MainWindowViewModel>();
        // options, logging, hosted services …
        _host = builder.Build();
        _host.Start();

        if (ApplicationLifetime is IClassicDesktopStyleApplicationLifetime desktop)
        {
            desktop.MainWindow = new MainWindow
            {
                DataContext = _host.Services.GetRequiredService<MainWindowViewModel>(),
            };
            desktop.Exit += (_, _) => _host.StopAsync().GetAwaiter().GetResult();
        }

        base.OnFrameworkInitializationCompleted();
    }
}
```

- Resolve root ViewModels/services from `host.Services` — never `new` them; keep XAML code-behind free of service instantiation.
- Long-running background work: `IHostedService`/`BackgroundService` on the builder; marshal UI updates via `Dispatcher.UIThread` (see dotnet-async).

## Compiled bindings (default ON in 12)

- Plain `{Binding}` compiles; the **root of every view and every `DataTemplate` needs `x:DataType`** or the build fails: `x:DataType="vm:MainWindowViewModel"`.
- Per-binding escape hatch for dynamic shapes: `{ReflectionBinding Path}`; scope opt-out: `x:CompileBindings="False"`.
- The template ViewLocator (`IDataTemplate` mapping `FooViewModel` → `FooView`) is still the pattern for VM-first navigation; register it in `<Application.DataTemplates>`.

## Common v11-isms to avoid (all wrong in 12)

| v11 habit | Avalonia 12 |
|---|---|
| `Avalonia.Diagnostics` package + `this.AttachDevTools()` | `AvaloniaUI.DiagnosticsSupport` + `.WithDeveloperTools()` |
| `BindingPlugins.DataValidators.RemoveAt(0)` in App | Obsolete — data-annotations validation is off by default |
| `Avalonia.ReactiveUI` package | Renamed `ReactiveUI.Avalonia` (but house MVVM is CommunityToolkit) |
| Assuming compiled bindings are off / sprinkling `x:CompileBindings="True"` | On by default; just provide `x:DataType` |
| `SystemDecorations`, `ExtendClientAreaChromeHints`, `Clipboard.GetTextAsync()`, `DoDragDrop()` | Removed/renamed (`WindowDecorations`, `TryGetTextAsync()`, `DoDragDropAsync`) |
| Targeting net6.0/net7.0 | Minimum net8.0 — use `net10.0` |

When scaffolding, surface the OS targets (Windows/Linux/macOS) and packaging/deployment decisions — see the checklist in dotnet-solution-setup.

If an existing app is on Avalonia 11, stay consistent with it and flag the divergence — upgrading to 12 is its own work item.
