# Lifecycle, `ModResource` & Collectible Hot-Unload Rules

Everywhere loads every mod assembly into its own collectible `AssemblyLoadContext` (`AssemblyOwner` in `Everywhere.Modding.Hosting`). Users can install, enable, disable, update, roll back, and uninstall mods **at runtime without restarting Everywhere**. If even **one strong GC root** continues referencing a type, object, delegate, or cache entry from a retired mod after unload, the `AssemblyLoadContext` fails to collect, `ModEngine` emits `CollectionUnconfirmed`, and `IModManager.IsRestartRequired` locks further live mod operations until application restart.

---

## 1. Core Contracts (`Everywhere.Modding.Abstractions`)

### `IMod`

```csharp
public interface IMod
{
    ValueTask InitializeAsync(IModContext context, CancellationToken cancellationToken);
}
```

- **Parameterless constructor rule**: `ModActivation` instantiates your `IMod` type via `Activator.CreateInstance`. Constructors **must never** register hooks, publish UI elements, start tasks, or mutate host state. Do all setup in `InitializeAsync`.
- **Disposal rule**: Implement **`IAsyncDisposable`** (preferred) or `IDisposable` whenever your mod owns connections, timers, background tasks, settings engines, or registrations that require ordered teardown. `ModActivation` prefers `IAsyncDisposable` and invokes **only one** disposal interface—even if `InitializeAsync` threw partway through!

### `IModContext`

| Member | Lifetime & Contract |
| :--- | :--- |
| `ModId` | Stable lowercase mod identifier (`string`). |
| `Services` | Activation-scoped `IServiceProvider`. Host singletons remain shared across activations; services closed over mod types stay scoped to this activation. **Stop resolving or using `Services` before `DisposeAsync` finishes.** |
| `Stopping` | `CancellationToken` signaled **before** `DisposeAsync()` is called. Stop accepting new work immediately and observe it in loops/network calls. |
| `StopReason` | `null` while active; `ModStopReason.Unload` during hot disable/update/uninstall; `ModStopReason.Exit` during normal process shutdown. Always check `StopReason` before touching UI dispatchers! |
| `Mods` | `IModCatalog` (`IsAvailable(modId)`, `TryGetInfo(modId, out var info)`). Allows inspecting declared dependencies without creating strong object references across ALCs. |
| `InstallationDirectory` | Read-only unpacked package directory (assets, `manifest.json`, binaries). Never write runtime state here. |
| `DataDirectory` | Writable persistent directory shared across updates/rollbacks of the same `ModId`. Store `settings.json`, user configs, and caches here. |
| `Register(object resource)` | Accepts `IDisposable` or `IAsyncDisposable` (prefers `IAsyncDisposable`). Queues handles to **reverse application changes** during `Unload` or failed `InitializeAsync`. Skipped during `ModStopReason.Exit`. Registration closes once stopping begins; late calls throw `InvalidOperationException`. |

### `ModResource` (Shared Library Static Registrations)

When a **library assembly** shared by multiple consumer mods initializes static hooks or resources on behalf of the assembly itself rather than a single consumer's `IModContext`, register reversal handles on **`ModResource`** instead:

```csharp
ModResource.Register<MyLibraryMarker>(handle);
```

- Handles registered via `ModResource.Register<TOwner>(handle)` stay alive even if one consumer mod unloads, and are disposed in reverse order only when `TOwner`'s assembly itself retires (`Unload`).
- Do **not** pass a framework generic closed over a mod type (e.g. `List<ModType>`): use a marker type defined in the owning mod assembly itself.

### `[RequiresMod("mod.id")]` & Optional Dependencies

`RequiresModAttribute` is **declarative metadata only**—the host does **not** inject runtime guards or suppress JIT compilation of methods referencing an optional dependency!

If your mod declares an optional dependency (`Optional="true"` in `<EverywhereModDependency>`):
1. Check `context.Mods.IsAvailable("optional.mod.id")` first.
2. Put all code that statically references types from the optional dependency assembly into a **separate helper class** (or `[MethodImpl(MethodImplOptions.NoInlining)]` method), so the JIT never tries to load missing assemblies when the optional mod is disabled.

---

## 2. Exact Unload vs. Process-Exit Timeline

Understanding the exact sequence in `ModEngine` explains why teardown order matters:

### Path A: Hot Unload / Update / Disable (`context.StopReason == ModStopReason.Unload`)

1. Consumers stop **before** dependencies (`ModRetirementPlan.ModOrder`).
2. `context.Stopping` is canceled.
3. **`IMod.DisposeAsync()`** runs (under `CallbackTimeout`, default 10s). Stop timers/background loops, await in-flight tasks, and **flush/dispose `SettingsEngine<T>`**.
4. **`context.Register(...)`** handles are disposed in **LIFO order** (reversing UI items, `LocaleManager`, `ThemeRegistry`, etc.).
5. For retiring assemblies (dependencies after consumers): **`ModResource`** assembly handles are disposed in LIFO order.
6. **Avalonia UI Drain & Framework Sweep (`AssemblyRegistry.SweepAvaloniaAsync`)**:
   - When `Application.Current` is running, posts `CollectibleAssemblyCleanup.SweepUI` to `Dispatcher.UIThread` at **`DispatcherPriority.ContextIdle`** (waiting for pending layout/render/detach work to settle).
   - Runs `CollectibleAssemblyCleanup.Sweep` for non-UI framework caches.
7. **`AssemblyRegistry.Release`** clears internal reflection caches (`ModReflectionCache`), refreshes `TypeDescriptor` metadata for each retiring assembly, removes assembly owners, and calls `AssemblyLoadContext.Unload()`.
8. **GC + UI Retry Loop (`ModEngine.ReleaseAsync`)**:
   - Runs `GC.Collect(); GC.WaitForPendingFinalizers(); GC.Collect();`.
   - If any retiring `WeakReference` is still alive, waits exponentially and **re-pumps `DispatcherPriority.ContextIdle` + re-runs `SweepUI` & `Sweep`** (`SweepAliveAvaloniaAsync`) before the next GC pass.
   - If still alive after `CollectionAttempts`, `ModEngine` emits `CollectionUnconfirmed` and sets `IsRestartRequired = true`.

### Path B: Application Exit (`context.StopReason == ModStopReason.Exit`)

1. `context.Stopping` is canceled.
2. **`IMod.DisposeAsync()`** runs sequentially (consumers before dependencies) with no bounded UI phase.
3. **`context.Register(...)` and `ModResource` reversal queues are NOT executed**, and `Avalonia`/`Dispatcher` sweeping is skipped because the UI thread may already be shutting down.
4. **CRITICAL RULE**: Never call `await Dispatcher.UIThread.InvokeAsync(...)` inside `DisposeAsync()` when `context.StopReason == ModStopReason.Exit`! It can hang process shutdown indefinitely. Put settings flushing (`await _settings.DisposeAsync()`) in a `finally` block so user settings always persist on exit even if UI reversal is skipped.

---

## 3. Collectible `AssemblyLoadContext` Leak-Prevention Checklist

Before shipping or accepting any mod, verify every item below:

| Risk Area | What Breaks Hot Unload | Required Pattern |
| :--- | :--- | :--- |
| **Events on host singletons** | `Application.Current.ActualThemeVariantChanged += OnChanged;` or `AppearanceManager.Shared.AccentColorChanged += ...` without `-=` pins the delegate target + mod type. | Always wrap subscription/unsubscription in a disposable passed to `context.Register(...)` or cleaned in `DisposeAsync()`. |
| **Dispatcher & UI timers** | `DispatcherTimer` active during unload, or `Dispatcher.UIThread.Post(...)` firing after unload. | Stop `DispatcherTimer` and unsubscribe `Tick` in `DisposeAsync()`; check `Stopping.IsCancellationRequested` inside posted callbacks. |
| **Background Tasks & Loops** | Unawaited `Task.Run(...)` or `_ = DoWorkAsync()` holding closures over mod objects/ALC. | Pass `context.Stopping` into loops, cancel cleanly, and **`await` all mod-owned tasks in `DisposeAsync()`** before releasing resources. |
| **Avalonia Properties / Styles / Controls** | Custom `StyledProperty` / `AttachedProperty` metadata, style templates, or `MergedDictionaries` entries left attached to `Application.Current` or host windows. | Remove added `ResourceDictionary`/`Styles` on unload; detach `ItemsSource`/`DataContext` (already handled automatically by `ModSettingsRegistry` for settings views); leverage patched `Avalonia` + `CollectibleAssemblyCleanup`. |
| **Reflection & `System.Text.Json`** | Calling `JsonSerializer.Serialize(obj)` with default options populates global `JsonSerializerOptions.Default` internal reflection caches pointing to mod types! | **Always define a mod-owned `[JsonSerializable]` `JsonSerializerContext`** (used by `OpenSettingsAsync` too) and never touch global `JsonSerializerOptions.Default`. |
| **Host DI & `ConditionalWeakTable`** | Manually inserting mod instances/types into host static dictionaries or host singleton properties. | Only register through official reversible host APIs (`IChatPluginManager`, `IModSettingsRegistry`, `MainViewModel`, `LocaleManager`, `ThemeRegistry`), or register custom cleanup via `CollectibleAssemblyCleanup.RegisterCleanup(...)` / `RegisterSweeper(...)`. |
| **Async state machines holding `Assembly` / `Type` across cleanup** | Keeping local `AssemblyLoadContext`, `Type`, or `IMod` variables alive on caller stack frames during GC checks. | Drop references before collection checks; in tests, build and unload inside `[MethodImpl(MethodImplOptions.NoInlining)]` boundaries if testing raw ALCs. |

---

## 4. Writing Automated Unload Tests (`[AvaloniaTest]`)

Every non-trivial mod—especially UI, theme, settings, or plugin mods—should include an integration test that loads the real packaged `.zip` (or build output) through `ModEngine`, exercises its features, unloads it (`ApplyAsync([])`), and confirms zero resident assemblies remain:

```csharp
[AvaloniaTest]
public async Task MyMod_ExerciseAndUnload_CollectsCleanly()
{
    var started = await engine.ApplyAsync([myModDefinition]);
    Assert.That(started.Status, Is.EqualTo(ModApplyStatus.Applied));

    // Exercise tools, settings view creation, UI rendering, etc.
    await Dispatcher.UIThread.InvokeAsync(() => { }, DispatcherPriority.ContextIdle);

    var unloaded = await engine.ApplyAsync([]);
    Assert.That(unloaded.Status, Is.EqualTo(ModApplyStatus.Applied));
    Assert.That(unloaded.ResidentAssemblyNames, Is.Empty);
}
```

Use `[AvaloniaTest]` from **`Avalonia.Headless.NUnit`** (configured with `[assembly: AvaloniaTestApplication(typeof(AvaloniaHeadlessTestApp))]` and `.UseHeadless(...)`) so `Application.Current` and `Dispatcher.UIThread` behave identically to the desktop host during sweep and GC retry passes.
