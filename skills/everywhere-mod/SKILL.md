---
name: everywhere-mod
description: >-
  Official developer guide, SDK reference, collectible hot-unload rules, extension recipes, and modding policy for building Everywhere mods (.NET 11 & Avalonia UI).
  Use when scaffolding, developing, packaging, testing, signing, publishing, or reviewing mods for Everywhere using Everywhere.Modding.Sdk, IMod/IModContext, ModChatPlugin, OpenSettingsAsync/IModSettingsRegistry, Avalonia UI/navigation, cross-mod dependencies, repository indexes, or checking Everywhere's Modding Policy and Acceptable Use Guidelines.
license: Apache-2.0
metadata:
  author: Sylinko
  version: "1.0.0"
  repository: "https://github.com/Sylinko/Everywhere"
---

# Everywhere Mod Development Guide

**Everywhere** supports runtime-loaded, collectible `.NET 11` mods packaged with **`Everywhere.Modding.Sdk`**. Each mod loads into an isolated, collectible `AssemblyLoadContext` (`AssemblyOwner`), receives host services and lifecycle signals through `IMod` / `IModContext`, and supports hot enable, disable, update, rollback, and uninstall without restarting the application—provided it follows strict resource reversal and GC hygiene rules.

---

## How to Load Reference Documents (Progressive Disclosure)

Keep context usage minimal by reading only the reference documents relevant to your current task.

> **Path Resolution Rule for `read_file`**:
> - **Inside Everywhere (`skill://` URI)**: If this file was loaded from `skill://<source>.everywhere-mod/SKILL.md` (e.g. `skill://builtin.everywhere-mod/SKILL.md`), pass `skill://<source>.everywhere-mod/references/<filename>.md` to `read_file`.
> - **External Coding Agents (Filesystem)**: If this file was loaded from `<dir>/SKILL.md`, pass `<dir>/references/<filename>.md` to your file-reading tool.

1. **[Project SDK, Manifest & Repository Packaging](references/project-sdk-and-packaging.md)**
   - **Read first when creating a mod project, configuring `.csproj` / `Everywhere.local.props`, or publishing packages/indexes.** Covers NuGet `<Project Sdk="Everywhere.Modding.Sdk/...">` vs. in-repo `<Import>`, all MSBuild properties (`EverywhereModId`, `EverywhereModEntryType`, `EverywhereHostVersion`, localizations, dependencies), `EverywherePack`, generated `manifest.json`, and `Everywhere.Modding.Sdk` CLI (`check`, `keygen`, `build-index`).
2. **[Lifecycle, ModResource & Collectible Hot-Unload Rules](references/lifecycle-and-hot-unload.md)**
   - **Mandatory before writing `IMod`, event handlers, background tasks, caches, or UI code.** Covers `IMod`, `IModContext`, `ModResource`, `[RequiresMod]`, hot unload (`ModStopReason.Unload`) vs. exit (`ModStopReason.Exit`), `ModEngine`'s Avalonia `ContextIdle` sweep & GC retry loop, leak-prevention checklist, and `[AvaloniaTest]` unload testing.
3. **[Host Extension Points & End-to-End Recipes](references/extension-points-and-recipes.md)**
   - **Read when implementing real mod features.** End-to-end patterns grounded in Everywhere's reference mods: AI tools (`ModChatPlugin` & `IChatPluginManager`), persistent migrated settings (`context.OpenSettingsAsync` + `IModSettingsRegistry`), navigation pages & `.resx` localization (`MainViewModel.RegisterNavigationItem`, `LocaleManager`), and consuming or designing extensible cross-mod APIs (`<EverywhereModDependency>`, reading target mod READMEs, and returning `IDisposable` handles).
4. **[Mod Ecosystem & Acceptable Use Policy (EN / 中文)](references/mod-policy-and-guidelines.md)**
   - **Mandatory before publishing, reviewing, or declaring mod metadata.** Defines multi-licensing (`AGPL-3.0-only` host + Mod Interface Additional Permission, `Apache-2.0` SDK/Skills), brand and artwork boundaries (`TRADEMARK.md`), reserved `everywhere.*` / `sylinko.*` namespaces, security and content compliance requirements, and platform integrity boundaries (prohibiting modification, concealment, or circumvention of Account, Cloud Sync, Official Model Gateway, Mod Verification, or official repository/website/sponsorship links).

---

## Quick Orientation Cheat Sheet

### Minimal External Mod `.csproj` (`Everywhere.Modding.Sdk`)

```xml
<Project Sdk="Everywhere.Modding.Sdk/1.0.0">

  <PropertyGroup>
    <Version>1.0.0</Version>
    <EverywhereHostVersion>[1.0.0,)</EverywhereHostVersion>
    <EverywhereModId>alice.hello</EverywhereModId>
    <EverywhereModCategory>general</EverywhereModCategory>
    <EverywhereModEntryType>Alice.Hello.HelloMod</EverywhereModEntryType>
    <EverywhereModAuthor>Alice</EverywhereModAuthor>
    <EverywhereModLicense>Apache-2.0</EverywhereModLicense>
  </PropertyGroup>

  <ItemGroup>
    <EverywhereModLocalization Include="en">
      <Name>Hello Everywhere</Name>
      <Description>Adds a sample greeting tool to Everywhere.</Description>
      <Readme>README.md</Readme>
    </EverywhereModLocalization>
  </ItemGroup>

</Project>
```

Pair with a gitignored `Everywhere.local.props` pointing to a mod-enabled Everywhere installation (containing `mod-host.json`), and build the distributable `.zip` with:

```bash
dotnet build -c Release -t:EverywherePack
```

### Extension Surfaces at a Glance

| Goal | Host API / Entry | Reversal Registration |
| :--- | :--- | :--- |
| **AI Chat Tools** | Subclass `ModChatPlugin`, register via `IChatPluginManager.RegisterChatPlugin(...)` | `context.Register(...)` or `DisposeAsync` (use UI thread on `Unload`, skip dispatcher on `Exit`) |
| **Persistent Settings** | `await context.OpenSettingsAsync(...)` (`SettingsEngine<T>`) | Dispose & flush inside `IMod.DisposeAsync()` (`finally` block) for both `Unload` and `Exit` |
| **Mod Settings UI** | `IModSettingsRegistry.Register(context, () => settings.SettingsItems)` | Dispose in `IMod.DisposeAsync()` before saving final settings |
| **Sidebar Navigation Page** | `MainViewModel.RegisterNavigationItem(IMainViewNavigationTopLevelItem)` on `Dispatcher.UIThread` | Pass returned `IAsyncDisposable` to `context.Register(...)` |
| **Localization (`.resx`)** | `LocaleManager.RegisterProvider(assembly, () => new ResourceManagerLocaleResourceProvider(...))` | Pass returned `IDisposable` to `context.Register(...)` |
| **Cross-Mod Extensions** | Declare `<EverywhereModDependency>`, check the target mod's `README.md` for its public API | Pass returned `IDisposable` / `IAsyncDisposable` tokens to `context.Register(...)` |

### Mandatory Engineering and Policy Requirements

1. **Preserve collectible hot unload**: Dispose UI subscriptions, tasks, timers, and host registrations in `DisposeAsync` or `context.Register(...)`, avoid reflection on global `JsonSerializerOptions.Default`, and never block on `Dispatcher.UIThread` when `context.StopReason == ModStopReason.Exit`.
2. **Maintain platform integrity**: Mods must not modify, conceal, intercept, replace, or circumvent Everywhere's Account/Auth, Cloud Sync, Official Model Gateway, Mod Verification/Signing, or built-in official repository, website, attribution, and sponsorship links.
3. **Respect reserved identifiers**: The `everywhere.*` and `sylinko.*` mod ID prefixes are reserved exclusively for official Sylinko mods.
