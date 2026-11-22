# Project SDK, Manifest & Repository Packaging

Everywhere mods are built with **`Everywhere.Modding.Sdk`**, packaged as ZIP archives containing `manifest.json`, and optionally published to static HTTP/local repository indexes.

---

## 1. Project Setup Modes

### 1.1 External Community Mods (NuGet MSBuild SDK — Recommended)

Once `Everywhere.Modding.Sdk` is published to NuGet, external mod projects reference it as an MSBuild Project SDK:

```xml
<Project Sdk="Everywhere.Modding.Sdk/1.0.0">

  <PropertyGroup>
    <Version>1.0.0</Version>
    <EverywhereHostVersion>[1.0.0,)</EverywhereHostVersion>
    <EverywhereModId>alice.translator</EverywhereModId>
    <EverywhereModCategory>productivity</EverywhereModCategory>
    <EverywhereModEntryType>Alice.Translator.TranslatorMod</EverywhereModEntryType>
    <EverywhereModAuthor>Alice</EverywhereModAuthor>
    <EverywhereModHomepage>https://github.com/alice/everywhere-translator</EverywhereModHomepage>
    <EverywhereModRepository>https://github.com/alice/everywhere-translator</EverywhereModRepository>
    <EverywhereModLicense>Apache-2.0</EverywhereModLicense>
    <EverywhereModLicenseFile>LICENSE</EverywhereModLicenseFile>
    <EverywhereModIcon>Assets/Icon.svg</EverywhereModIcon>
  </PropertyGroup>

  <ItemGroup>
    <EverywhereModLocalization Include="en">
      <Name>Translator</Name>
      <Description>Translates selected text with custom dictionaries.</Description>
      <Readme>README.md</Readme>
    </EverywhereModLocalization>
    <EverywhereModLocalization Include="zh-hans">
      <Name>翻译助手</Name>
      <Description>使用自定义词典翻译选中文本。</Description>
    </EverywhereModLocalization>
  </ItemGroup>

  <ItemGroup>
    <EmbeddedResource Include="Assets/Icon.svg" />
    <Content Include="LICENSE" CopyToOutputDirectory="PreserveNewest" />
  </ItemGroup>

</Project>
```

You can also use the explicit `<Sdk>` element if you need property evaluation before `Sdk.props`:

```xml
<Project>
  <Sdk Name="Everywhere.Modding.Sdk" Version="1.0.0" />
  ...
</Project>
```

#### Local Host Configuration (`Everywhere.local.props`)

`Sdk.props` sets `<TargetFramework>net11.0</TargetFramework>`, `<SelfContained>false</SelfContained>`, `<CopyLocalLockFileAssemblies>true</CopyLocalLockFileAssemblies>`, and automatically imports `$(EverywhereLocalConfiguration)` (defaulting to `$(MSBuildProjectDirectory)/Everywhere.local.props`) if present.

Keep `Everywhere.local.props` gitignored and commit an `Everywhere.local.props.example`:

```xml
<Project>
  <PropertyGroup>
    <!-- Path to an installed Everywhere directory (or .app bundle on macOS) containing mod-host.json -->
    <EverywhereHost>C:\Program Files\Everywhere</EverywhereHost>
    <EverywhereHostVersion>[1.0.0,)</EverywhereHostVersion>
  </PropertyGroup>
</Project>
```

- On macOS, if `EverywhereHost` ends with `.app`, `HostReferenceSet` resolves `<EverywhereHost>/Contents/MonoBundle/mod-host.json` automatically.
- In CI or scripted builds, pass `-p:EverywhereHost=/path/to/host` on the command line.

### 1.2 In-Repository Built-In / Workspace Mods

Mods developed inside the `Sylinko/Everywhere` repository import the workspace SDK directly instead of NuGet:

```xml
<Project>
  <Import Project="../../src/Everywhere.Modding.Sdk/Sdk/Sdk.props"/>
  <PropertyGroup>
    <TargetFramework>net11.0</TargetFramework>
    <EverywhereModToolsDirectory>$(MSBuildThisFileDirectory)../../src/Everywhere.Modding.Sdk/bin/$(Configuration)/net11.0</EverywhereModToolsDirectory>
    <EverywhereHost Condition="'$(EverywhereHost)' == ''">$(MSBuildThisFileDirectory)../../src/Everywhere.Core/Everywhere.Core.csproj</EverywhereHost>
    ...
  </PropertyGroup>
  <ItemGroup>
    <ProjectReference Include="../../src/Everywhere.Modding.Sdk/Everywhere.Modding.Sdk.csproj"
                      ReferenceOutputAssembly="false" Private="false" ExcludeAssets="all"
                      GlobalPropertiesToRemove="$(_EverywhereHostPropertiesToRemove);TargetFramework"/>
  </ItemGroup>
  <Import Project="../../src/Everywhere.Modding.Sdk/Sdk/Sdk.targets"/>
</Project>
```

> **Note on Source Generators**: `Everywhere.I18N.SourceGenerator` (`src/Build.I18N.targets`) and `Everywhere.Configuration.SourceGenerator` (`[GeneratedSettingsItems]`) are currently workspace project analyzers inside `Sylinko/Everywhere`. Standalone external mods should use standard `.resx` `EmbeddedResource` files (`ResourceManagerLocaleResourceProvider` / `DirectLocaleKey`) and construct `SettingsItem` instances in C#, unless those analyzer packages are separately distributed.

---

## 2. How Host References & `EverywhereHostVersion` Work

During build (`EverywhereImportHost`), the SDK runs `Everywhere.Modding.Sdk import` against `$(EverywhereHost)`:
1. Reads `mod-host.json` (`ModHostDescriptor`: `hostVersion`, `modApiVersion`, `framework`, `runtimeIdentifier`, `entryAssembly`).
2. Resolves all compile-time host assemblies (framework runtime, `Everywhere.Modding.Abstractions.dll`, `Everywhere.Core.dll`, Avalonia, ShadUI, Microsoft.Extensions.AI, etc.) with `Private="false"` so they are **never copied into your mod package**.
3. During `EverywherePack`, `ModPackager` inspects your compiled DLL's assembly references:
   - If your mod uses **only** `Everywhere.Modding.Abstractions` and .NET runtime contracts, `EverywhereHostVersion` is optional (only `EverywhereModApiVersion` is checked).
   - If your mod references host implementation assemblies (such as `Everywhere.Core`, `Avalonia`, `ShadUI`, etc.), **`EverywhereHostVersion` is required** (`MissingHostRange` error if omitted) and must match the build host (`BuildHostOutsideRange` if mismatched).
   - `EverywhereModApiVersion` defaults from `mod-host.json` as `[<major>.<minor>.<patch>,<major+1>.0.0-0)` (or exact `[<prerelease>]` for prerelease APIs).

---

## 3. MSBuild Properties & Items Reference

### Required Properties

| Property | Validation / Notes |
| :--- | :--- |
| `EverywhereModId` | Lowercase identifier matching `^[a-z0-9]+(?:[._-][a-z0-9]+)*$` (e.g. `alice.translator`). **Never use `everywhere.*` or `sylinko.*`** — those are reserved for official mods. |
| `Version` | Valid SemVer string (e.g. `1.0.0`, `1.2.0-beta.1`). |
| `EverywhereModEntryType` | Full CLR type name of your public `IMod` class (e.g. `Alice.Translator.TranslatorMod`). |

### Optional Properties

| Property | Default / Purpose |
| :--- | :--- |
| `EverywhereModPlatform` | `universal` (omitted in `manifest.json` as `UniversalModEntry`). Set e.g. `win-x64`, `win-arm64`, `osx-x64`, `osx-arm64`, `linux-x64`, `linux-arm64` for platform-specific mods (`PlatformModEntry`). |
| `EverywhereModCategory` | `general`. Recommended values: `appearance`, `productivity`, `search`, `automation`, `general`. Passed to `EverywherePack` outputs for index building. |
| `EverywhereHostVersion` | NuGet version range for host compatibility (e.g. `[1.0.0,2.0.0-0)`). Required when referencing host implementation assemblies. |
| `EverywhereModApiVersion` | NuGet version range for `Everywhere.Modding.Abstractions` contract compatibility. Auto-derived from host descriptor if omitted. |
| `EverywhereModPackagePath` | Output `.zip` path. Default: `$(OutputPath)$(EverywhereModId)-$(Version).zip`. |
| `EverywhereModAuthor` | Display author/publisher name (cannot be whitespace if specified). |
| `EverywhereModIcon` | Package-relative icon path (`.svg`, `.png`, `.jpg`, `.jpeg`, or `.webp`), e.g. `Assets/Icon.svg`. |
| `EverywhereModHomepage` | Absolute `http://` or `https://` URL without embedded credentials. |
| `EverywhereModRepository` | Absolute `http://` or `https://` URL of the mod's source repository. |
| `EverywhereModRepositoryDirectory` | Relative path inside the monorepo if applicable (e.g. `mods/my-mod`). |
| `EverywhereModLicense` | SPDX license expression (e.g. `Apache-2.0`, `MIT`, `AGPL-3.0-only`) or `LicenseRef-...`. Required if `EverywhereModLicenseFile` or acceptance is set. |
| `EverywhereModLicenseFile` | Package-relative path to license text bundled in the package (e.g. `LICENSE`). Must be included via `<Content>` / `<None Pack="true">`. |
| `EverywhereModRequireLicenseAcceptance` | `true` / `false`. Prompts user confirmation in the UI before installation when `true`. |

### MSBuild Items

1. **Localizations (`<EverywhereModLocalization>`)**:
   ```xml
   <EverywhereModLocalization Include="en">
     <Name>My Mod</Name>
     <Description>Short catalog summary.</Description>
     <Readme>README.md</Readme>
   </EverywhereModLocalization>
   ```
   - `Include` must be a valid BCP-47 language tag (`^[A-Za-z]{1,8}(?:-[A-Za-z0-9]{1,8})*$`).
   - Root `README.md` (`en` fallback) and `README.<lang>.md` files are automatically discovered and packed unless explicitly excluded with `<Content Remove="..." />` or `Pack="false"`.

2. **Mod Dependencies (`<EverywhereModDependency>`)**:
   ```xml
   <EverywhereModDependency Identity="everywhere.themes"
                            Version="[1.0.0,)"
                            Optional="false"
                            Assemblies="Everywhere.Mods.Themes" />
   ```
   - `Assemblies`: Semicolon- or comma-separated managed assembly names exported by the dependency mod that your mod imports.
   - `ModPackager` automatically **excludes** declared dependency assemblies and their sidecar `.deps.json` / `.pdb` / satellite assemblies from your `.zip`, because the host loads them once in dependency order and resolves them across `AssemblyLoadContext` boundaries.
   - **Important**: Add a reference path or compile-time reference to the dependency mod's assembly during build if you call its C# types directly, while ensuring it is NOT shipped inside your own `.zip`.

3. **Extra Package Files & Native Binaries**:
   - `<Content>` items (with `Pack` != `false`) and `<None Pack="true">` are packed relative to `PackagePath` (or project-relative path).
   - Native libraries from `<RuntimeIdentifier>` NuGet packages (`NativeCopyLocalItems`) are automatically placed under `runtimes/<rid>/native/`.

---

## 4. Packaging & Repository Index Tooling

### Creating the Mod `.zip`

```bash
dotnet build -c Release -t:EverywherePack
```

The SDK stages files into `obj/.../everywhere-pack`, excludes host-owned and dependency-mod assemblies, validates all paths/URLs/licenses, computes integrity metadata, generates `manifest.json` (`schemaVersion: 1`), and emits `PackageCreated` JSON to stdout.

### Validating & Publishing an Index via `Everywhere.Modding.Sdk` CLI

The packaged SDK ships a command-line tool (`tools/net11.0/Everywhere.Modding.Sdk.dll`):

```bash
# 1. Validate an unpacked mod directory's manifest.json
dotnet <sdk-tools-dir>/Everywhere.Modding.Sdk.dll check ./unpacked-mod

# 2. Generate an ECDSA P-256 signing key pair (PEM private key + SPKI base64 public key)
dotnet <sdk-tools-dir>/Everywhere.Modding.Sdk.dll keygen ./mod-signing.key.pem ./mod-signing.pub

# 3. Build (and optionally sign) a repository index.json from one or more mod .zip packages
dotnet <sdk-tools-dir>/Everywhere.Modding.Sdk.dll build-index ./repo-spec.json ./repo/index.json ./mod-signing.key.pem
```

#### `repo-spec.json` Schema (`ModIndexBuildSpec`)

```json
{
  "format": 1,
  "id": "alice.mods",
  "title": {
    "en": "Alice's Mod Repository",
    "zh-hans": "Alice 的模组仓库"
  },
  "description": {
    "en": "Community extensions for Everywhere."
  },
  "mirrors": [],
  "layout": "directory",
  "packages": [
    {
      "path": "./artifacts/alice.translator-1.0.0.zip",
      "publishedAt": 1795300000,
      "category": "productivity",
      "labels": ["verified"],
      "keywords": ["translate", "dictionary"],
      "isDefaultEnabled": false,
      "channels": ["stable"]
    }
  ]
}
```

- `layout`: `"directory"` copies archives and generates sidecar assets into the output directory; `"release"` targets external release artifact URLs (`releaseTag` / `artifactUrl`).
- When signed with `private-key-pem`, `ModIndexBuilder` writes an ECDSA P-256 signature verified by Everywhere's `ModArtifactVerifier` when users add your repository URL + public key in **Mods → Sources**.
