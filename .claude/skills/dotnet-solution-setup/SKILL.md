---
name: dotnet-solution-setup
description: Use when creating a new .NET solution or project, adding a project to a solution, writing or editing csproj files, choosing target frameworks, or wiring application startup and hosting.
---

# .NET solution & project setup

## Solution layout

- Solution format: **`.slnx`** (XML), not legacy `.sln`. The .NET 10 SDK's `dotnet new sln` emits `.slnx` by default; migrate existing files with `dotnet sln <file.sln> migrate` (keep only one of `.sln`/`.slnx` per directory).
- Three top-level directories at the solution root:
  - `src/` — production projects
  - `tests/` — test projects (xUnit v3 — see xunit-v3)
  - `docs/` — markdown documentation

## Target frameworks

| Project type | TFM |
|---|---|
| Library | `netstandard2.0` by default (max consumer compatibility) |
| Application (console, worker, Avalonia, WPF) | `net10.0` (WPF: `net10.0-windows`) |
| Tests | `net10.0`, under `tests/` |

Libraries deviate only when their purpose requires it (`Span<T>`-heavy APIs, source generators, framework-specific surface) — then `net10.0` or multi-target `netstandard2.0;net10.0`, with the reason documented in a `.csproj` comment. A `netstandard2.0` library using C# 14: compiler-only features (`field` keyword, extension blocks) work as-is; add PolySharp to polyfill compiler-known types before using `init`/`required` members or nullable-analysis attributes like `[NotNullWhen]`.

## csproj defaults (every project)

```xml
<PropertyGroup>
  <LangVersion>14</LangVersion>
  <Nullable>enable</Nullable>
  <!-- at minimum: <WarningsAsErrors>nullable</WarningsAsErrors> -->
  <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
  <ImplicitUsings>disable</ImplicitUsings>
</PropertyGroup>
```

- **Never enable `ImplicitUsings`** — every file declares its own `using` directives.
- No `!` null-forgiveness without a justified, commented reason.

## Bootstrapping — every app builds an IHost

All applications wire configuration, logging, lifetime, and DI through `Microsoft.Extensions.Hosting.IHost` from day one. Console apps and workers:

```csharp
using System.Threading.Tasks;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

public static class Program
{
    public static async Task Main(string[] args)
    {
        HostApplicationBuilder builder = Host.CreateApplicationBuilder(args);
        // builder.Services.AddSingleton<IClock, SystemClock>();

        using IHost host = builder.Build();
        await host.RunAsync();
    }
}
```

- `Host.CreateApplicationBuilder` returns `HostApplicationBuilder`: register via `builder.Services`, read config via `builder.Configuration`. There is **no `ConfigureServices` callback** — that belongs to the legacy `IHostBuilder` API (`Host.CreateDefaultBuilder`), which new code does not use.
- **GUI apps must run the UI framework's own lifetime** — `await host.RunAsync()` alone never shows a window. See avalonia and wpf for the exact wiring.
- Resolve the application's root type (main window, root ViewModel, primary service) from `host.Services` — never `new` it up.
- Long-running background work goes through `IHostedService`/`BackgroundService`, not ad-hoc `Task.Run` in `Main` (see dotnet-async).
- Bind configuration through `IOptions<T>`/`IOptionsMonitor<T>` — services don't read `IConfiguration` directly (see dotnet-di-design).

## Decisions to surface when scaffolding

Don't silently default these — ask or state the choice explicitly: deployment model (framework-dependent vs self-contained, trimming/NativeAOT) · installer/packaging (MSIX, zip, `dotnet tool`, NuGet package) · OS targets for GUI apps (Windows/Linux/macOS) · localization (`.resx`) · logging sinks and levels (`Microsoft.Extensions.Logging`) · licensing of NuGet dependencies.

## Common mistakes

| Mistake | Fix |
|---|---|
| `dotnet new sln` kept as `.sln` / new `.sln` files | `.slnx` |
| `Host.CreateDefaultBuilder(args).ConfigureServices(...)` | `Host.CreateApplicationBuilder(args)` + `builder.Services` |
| Library scaffolded as `net10.0` "because it's newest" | `netstandard2.0` unless a documented reason requires more |
| `<ImplicitUsings>enable</ImplicitUsings>` (template default) | Always `disable`, explicit usings per file |
| Tests scaffolded with `dotnet new xunit` (v2) | xUnit v3 per the xunit-v3 skill |

If an existing solution has its own layout or bootstrapping conventions, stay consistent with it and flag the divergence rather than silently mixing styles.
