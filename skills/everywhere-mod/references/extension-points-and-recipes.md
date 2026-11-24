# Host Extension Points & End-to-End Recipes

Everywhere mods interact with the host through standard .NET DI services (`context.Services`) and host extension APIs. Always keep host service references scoped properly and register reversible changes with `context.Register(...)` or clean them up in `IMod.DisposeAsync()`.

---

## 1. Exposing AI Tools (`ModChatPlugin` & `IChatPluginManager`)

To add native tools callable by Everywhere assistants (like `everywhere.everything`):

1. **Subclass `ModChatPlugin`** (pass a stable plugin slug, e.g. `"my_tool"` -> key becomes `"mod.my_tool"`):

```csharp
using System.ComponentModel;
using DynamicData;
using Everywhere.Chat.Permissions;
using Everywhere.Chat.Plugins;
using Everywhere.I18N;
using Tabler.Avalonia;

namespace Alice.MyTools;

public sealed class MyChatPlugin : ModChatPlugin
{
    public override IDynamicLocaleKey HeaderKey { get; } = new DirectLocaleKey("My Tool");
    public override IDynamicLocaleKey DescriptionKey { get; } = new DirectLocaleKey("Provides custom tool functions.");
    public override TablerIconKind? Icon => TablerIconKind.Tool;

    public MyChatPlugin() : base("my_tool")
    {
        _functionsSource.Add(new NativeChatFunction(GreetAsync, ChatFunctionPermissions.None));
    }

    [ChatFunction("greet_user")]
    [Description("Generates a friendly localized greeting for the specified name.")]
    private Task<string> GreetAsync(
        [Description("The user or target name to greet.")] string name,
        CancellationToken cancellationToken = default)
    {
        return Task.FromResult($"Hello from Alice.MyTools, {name}!");
    }
}
```

- **Permissions**: Pick the least-privileged `ChatFunctionPermissions` flag (`None`, `FileRead`, `FileAccess`, `NetworkAccess`, `ShellExecute`, etc.) so Everywhere's `ToolApproval` system prompts users accurately.
- **UI progress feedback**: Accept `[FromChatServices] IChatPluginDisplaySink displaySink` on tool methods when you need rich display blocks.

2. **Register & Dispose Safely** (notice how `EverythingMod` handles `Exit` vs. `Unload`!):

```csharp
private IModContext? _context;
private MyChatPlugin? _plugin;
private IAsyncDisposable? _pluginRegistration;

public ValueTask InitializeAsync(IModContext context, CancellationToken cancellationToken)
{
    _context = context;
    _plugin = new MyChatPlugin();
    _pluginRegistration = context.Services.GetRequiredService<IChatPluginManager>().RegisterChatPlugin(_plugin);
    context.Register(_pluginRegistration);
    return ValueTask.CompletedTask;
}

public async ValueTask DisposeAsync()
{
    if (_pluginRegistration is { } registration)
    {
        if (_context?.StopReason == ModStopReason.Exit)
        {
            await registration.DisposeAsync().ConfigureAwait(false);
        }
        else
        {
            // Remove plugin discovery on UI thread so Chat Plugins settings detach cleanly.
            await Dispatcher.UIThread.InvokeAsync(async () => await registration.DisposeAsync());
        }
    }

    _plugin?.Dispose();
    _pluginRegistration = null;
    _plugin = null;
    _context = null;
}
```

---

## 2. Persistent Settings & Mod Settings UI

Mods persist structured JSON settings inside `context.DataDirectory` using `context.OpenSettingsAsync(...)` (`Everywhere.Configuration`) and expose controls on the Mod Details page via `IModSettingsRegistry`.

### Step A: Observable Settings Model + `JsonSerializerContext`

- Inherit from `ObservableObject` (`CommunityToolkit.Mvvm.ComponentModel`) or implement `INotifyPropertyChanged`.
- **Reserved field**: Never name a serialized JSON property `"Version"` (case-insensitive)—`SettingsEngine` reserves `"Version"` for schema migration tracking.
- Always generate a dedicated `[JsonSerializable(typeof(MySettings))]` `JsonSerializerContext` so `System.Text.Json` never caches mod types on global `JsonSerializerOptions.Default`.

### Step B: Open Settings & Register UI in `InitializeAsync`

```csharp
var settings = new MySettings();
_settings = await context.OpenSettingsAsync(
        settings,
        new SemanticVersion(1),
        MySettingsJsonSerializerContext.Default.Options,
        new MySettingsDescriptors(), // ISettingsDescriptorProvider
        settingsFileName: "settings.json",
        cancellationToken: cancellationToken)
    .ConfigureAwait(false);

if (context.Services.GetService<IModSettingsRegistry>() is { } settingsRegistry)
{
    _settingsRegistration = settingsRegistry.Register(context, () => settings.SettingsItems);
}
```

### Step C: Orderly Teardown in `DisposeAsync`

1. First await `_settingsRegistration?.DisposeAsync()` (detaches UI controls and breaks bindings).
2. Stop any mod services that still write to `settings`.
3. In the `finally` block, `await _settings.DisposeAsync()` so pending changes flush to disk on **both** hot unload (`Unload`) and application exit (`Exit`).

---

## 3. Navigation Pages, Avalonia UI & Localization

### Adding a Main Window Navigation Item (`Everywhere.Mods.Sample` Pattern)

```csharp
await Dispatcher.UIThread.InvokeAsync(() =>
{
    var navigation = context.Services.GetRequiredService<MainViewModel>();
    context.Register(navigation.RegisterNavigationItem(new MyModPage()));
});
```

- `MyModPage` implements `IMainViewNavigationTopLevelItem`.
- Always use `{DynamicResource ...}` for colors and brushes in `.axaml` or C# code (e.g., `WindowBackgroundColor`, `CardBackgroundColor`, `ForegroundColor`, `MutedColor`, `BorderColor`, `PrimaryColor`) so your UI automatically follows light/dark switching and any installed appearance/theme mod.
- Note: Complex compiled `.axaml` requires building via `dotnet` (`Everywhere.Modding.Sdk`). Pure C# Avalonia controls work both in MSBuild projects and lightweight Roslyn environments.

### Registering `.resx` Localization

Register your mod's `.resx` resource assembly with `LocaleManager` and keep the handle in `context.Register(...)`:

```csharp
context.Register(
    LocaleManager.RegisterProvider(
        typeof(MyMod).Assembly,
        static () => new ResourceManagerLocaleResourceProvider(
            "Alice.MyMod.I18N.Strings",
            typeof(MyMod).Assembly)));
```

- Use `IDynamicLocaleKey` (`new DynamicLocaleKey(...)`, `FormattedDynamicLocaleKey`, `DirectLocaleKey`) in persistent UI state and `ModChatPlugin` headers/descriptions so switching Everywhere's UI language updates mod UI dynamically without reloading.

---

## 4. Consuming Dependencies & Designing Extensible Mod APIs

Built-in and community mods can expose public C# APIs or drop-in data folders for other mods to build upon. Because Everywhere loads each mod from its own catalog and package, **the authoritative specification for any specific mod's API lives in that mod's own `README.md`** (visible in Everywhere's Mod Details page or packed inside its `.zip` archive), not in this general SDK guide.

### Consuming Another Mod's API

1. **Inspect the Target Mod's `README.md` & `manifest.json`**: Check its exact `Id`, compatible version range, exported assembly names, public types, and any non-code extension directories (such as custom JSON folders under its `DataDirectory`).
2. **Declare the Dependency in `.csproj`**:
   ```xml
   <EverywhereModDependency Identity="vendor.provider-mod"
                            Version="[1.0.0,)"
                            Optional="false"
                            Assemblies="Vendor.ProviderMod" />
   ```
   - `ModPackager` excludes `Vendor.ProviderMod` from your `.zip`; at runtime `ModEngine` loads `vendor.provider-mod` before your mod and resolves its exported assemblies across `AssemblyLoadContext` boundaries.
3. **Register & Clean Up Handles**:
   - Pass any returned `IDisposable` or `IAsyncDisposable` handle directly to `context.Register(...)` so disabling or unloading your mod automatically revokes your contribution without pinning your `AssemblyLoadContext`.
4. **Isolate Optional Dependencies (`Optional="true"`)**:
   - Always check `context.Mods.IsAvailable("vendor.provider-mod")` before calling into it.
   - Keep code referencing optional dependency types inside a separate class or `[MethodImpl(MethodImplOptions.NoInlining)]` helper (annotated with `[RequiresMod("vendor.provider-mod")]` for documentation clarity) so the JIT does not attempt to load missing assemblies when the dependency is absent.

### Designing Your Own Extensible Mod API

When publishing a library or platform mod intended for downstream mods to extend:
- Keep registration methods static or host-service-backed, validate arguments eagerly, and **return an opaque `IDisposable` or `IAsyncDisposable` token** rather than asking consumers to call `Unregister(instance)`.
- Store only lightweight metadata or weak/removable entries so disposing the returned token removes every reference to the consumer mod's types, delegates, and `AssemblyLoadContext`.
- Notify subscribers via clean events (`Changed`) and document your mod ID, exported `Assemblies`, required `<EverywhereModDependency>` snippet, and minimal registration example in your mod's `README.md`.
