# Everywhere Architecture, Assemblies & Core Subsystems

## 1. Solution Structure & Assembly Graph (`Everywhere.slnx`)

Everywhere targets **.NET 11** (`11.0.100`) and **C# 15** with **Runtime Async** enabled in `Directory.Build.targets` (applied after target framework selection so `.NET Standard 2.0` source generators retain standard compilation).

### Core & Platform Assemblies (`src/`)
| Assembly | Role & Responsibilities |
| :--- | :--- |
| `Everywhere.Abstractions` | Foundation interfaces, `RuntimeConstants`, `SkillSourceRoot` / `ISkillManager`, and shared primitives. |
| `Everywhere.Core` | Application heart: Avalonia Views (`Views/`), ViewModels (`ViewModels/`), `ChatService`, `ChatPluginManager` & Built-in plugins, `KernelMixin` AI connectors, `StrategyEngine`, `SkillManager`, `PromptService` (`prompt.db`), `ChatDbContext` (`chat.db`), and `SettingsEngine`. |
| `Everywhere.Automation` | Platform-neutral visual graph engine: `VisualElement`, `VisualContext`, Weighted BFS `Snapshot` pipeline, in-memory `Build` / `CompositeTarget` compression, `VisualQuery`, and text-selection monitoring contracts. |
| `Everywhere.ProcessIsolation` | Multi-process role orchestration (`Main`, `Input Host`, `Automation Host`, `--hosts-control`), authenticated RPC channel, peer verification, signed remote resource IDs, and typed exception transport. |
| `Everywhere.Prompting` | Reusable model-facing structured prompt AST (`PromptNode`, `PromptCompactElement`) and renderers. |
| `Everywhere.Terminal` | Cross-platform PTY terminal emulator (`TerminalSession`, `TerminalParser` VT/CSI/OSC 633, `ExecuteStrategy`, `TerminalRun`, `TerminalLineBuffer`). |
| `Everywhere.Cloud` | OAuth client (`OAuthCloudClient`), official model catalog & AI Gateway, WebDAV/RFC 6578 sync infrastructure. |
| `Everywhere.Watchdog` | Standalone child process supervisor that monitors registered process handles and cleans up orphaned hosts/processes on unexpected exit. |
| `Everywhere.Windows` & `Everywhere.Windows.Interop` | Windows entry point, Win32 / UI Automation (`IVisualElementBackend`) implementation, `EverythingPlugin`, and Windows Task Scheduler Service Mode (`--hosts-control`). |
| `Everywhere.Mac` | macOS entry point (`.NET macOS` workload), AppKit / AXUIElement accessibility backend, native non-activating panels (`MacOSNativePanels.md`), and `SystemPlugin`. |
| `Everywhere.Linux` | Linux X11 / AT-SPI entry point and platform services. |

### Source Generators & Build Tooling
- `Everywhere.Configuration.SourceGenerator`: Generates UI settings items (`[GeneratedSettingsItems]`) and AOT-friendly JSON patch descriptors for `SettingsEngine`.
- `Everywhere.I18N.SourceGenerator`: Generates `LocaleKey`, `DynamicLocaleKey`, and `IDynamicLocaleKey` bindings from `.resx` localization files (`Build.I18N.targets`).
- `Everywhere.ProcessIsolation.SourceGenerator`: Generates strongly typed RPC proxies and dispatchers for cross-process Host interfaces.
- `Everywhere.BuildTask.Patcher` (`patches/`): MSBuild task using `MonoMod.Patcher` for compile-time static IL weaving of third-party assemblies (`Build.Patches.targets`).

---

## 2. Process Isolation & Host Architecture (`docs/ProcessIsolation/`)

To prevent buggy accessibility providers or global input hooks from freezing or crashing the main UI—and to support cross-privilege (UIPI) inspection on Windows—Everywhere splits runtime roles across isolated processes:

1. **`Main` Process**:
   - Owns product state, Avalonia UI, SQLite databases, `settings.json`, Host generation lifecycles, and authenticated RPC client connections.
   - Never holds raw native `VisualElement` pointers; interacts with `Automation Host` exclusively via remote resource handles (`RemoteVisualContext`) and copied result DTOs.
2. **`Input Host` Process**:
   - Owns global keyboard/mouse shortcut registration, shortcut delivery, and shortcut recording hooks.
3. **`Automation Host` Process**:
   - Owns `IVisualElementBackend`, native accessibility clients (UI Automation on Windows, AXUIElement on macOS), `VisualContext` instances, element pickers, target captures, UI automation actions, and the complete **native text-selection monitoring** workflow (gesture detection, debounce, accessibility text read, and clipboard fallback).
   - Serializes operations per `VisualContext` via a single Channel consumer (no thread-pool locks needed on context dictionaries). If a synchronous native COM/AX call ignores its platform timeout and hangs, `Main` can terminate and replace the `Automation Host` process cleanly.
4. **`--hosts-control` & Windows Service Mode**:
   - Short-lived CLI surface for installation and Windows Task Scheduler service-mode management.
5. **`Watchdog` (`Everywhere.Watchdog`)**:
   - Supervises registered process handles (`Automation Host`, `Input Host`, managed `Everything.exe`, etc.) to guarantee teardown if `Main` crashes.

---

## 3. Visual Context & Automation Pipeline (`docs/Automation/`)

Replaced legacy eager `get_visual_tree` dumps with a bounded two-phase pipeline exposed through `VisualQuery`:

1. **Phase 1 — `Snapshot` (Live Platform Read)**:
   - The *only* phase allowed to touch live native accessibility objects.
   - Uses **Weighted BFS** (`TraverseDistance`, direction weights, element type weights, core priority, and visited deduplication) under strict per-call native timeouts plus aggregate elapsed-time, operation, child, failure, and output budgets.
   - Produces a bounded partial observation with explicit degradation statuses (never fails silently on huge or stalled trees).
2. **Phase 2 — `Build` (Pure In-Memory Normalization & Compression)**:
   - Collapses transparent layout containers, projects repetitive/fragmented regions into logical `CompositeTarget` nodes (with honest `Composite` IDs distinct from actionable `VisualElement` IDs), progressively allocates token budgets (`VisualContextLengthLimit`), and renders final text using `PromptCompactElement` (compact XML-like syntax).
   - Atomically publishes target IDs into the turn's ownership batch before returning the RPC response.

---

## 4. LLM Request Runtime & Error Handling (`docs/LLMRequests/`)

- **Single Sequential Loop with Attempt Checkpoints (`KernelMixin.StreamRequestAsync`)**:
  - Emits a small typed union: `ChatRequestUpdate.AttemptStarted(attemptNumber)`, `ChatRequestUpdate.Content(delta)`, and `ChatRequestUpdate.AttemptFailed(failure, retryDelay)`.
  - Before attempt 1, `ChatService` checkpoints `AssistantChatMessage.Count` (source span count) and immutable message metadata.
  - If an attempt fails and is eligible for retry, `ChatService` rolls back only the spans added after the checkpoint (via `Edit`/change-set so `ChatTurnPresentation` prunes `_activityRows` and `_outputRows` incrementally without rebuilding surviving UI rows), restores metadata, displays retry status, waits for `RetryDelay`, and starts the next attempt cleanly.
- **Exception Normalization (`ChatExceptionNormalizer`)**:
  - Extracts structured HTTP status, provider error codes, headers, and response bodies (`ChatExceptionEvidenceExtractor`) across OpenAI, Anthropic, Google, Mistral, and Ollama SDKs, classifying errors into actionable categories (rate limit, auth, quota, context length, transient network, provider overload) with localized user-facing messages.

---

## 5. Terminal Subsystem (`docs/Terminal/`)

- **Single PTY Reader Invariant**: `TerminalSession` owns the `IPtyConnection`, incremental `PtyTextDecoder`, `TerminalParser`, and `TerminalResponseWriter`. Only one reader (`ReadOrIdleAsync` / `WaitForIdleAsync`) consumes the byte stream across shell detection and command execution so parser state (bracketed paste, OSC 633 markers, DA/DSR queries) is never split.
- **Rich (OSC 633) vs. None Strategy**:
  - `Detect` waits specifically for `OSC 633;B` (`CommandReady`), not just any OSC 633 sequence.
  - In **Rich** mode, `OSC 633;C` begins capturing into `activeRun.Output` (`TerminalLineBuffer`), and completion is signaled strictly by `OSC 633;D;<exitCode>` (never by output silence/idle, so `sleep 10` does not prematurely complete).
  - In **None** mode (when shell integration is unavailable), a synthetic `TerminalRun` is used with `ExitCode = null` and idle heuristics.
- **Bounded `TerminalLineBuffer`**: Serves both the LLM (`run.OutputText`) and the Avalonia UI (`run.Output.CopyLines(maxVisibleLines, out version)`), trimming trailing blank lines automatically.

---

## 6. Settings Engine (`docs/SettingsEngine/`)

- **JSON Patch Architecture**: `_jsonObj` (`JsonNode`) is the source of truth for `settings.json`.
- At startup phase `AsyncInitializerIndex.Settings`, `SettingsEngine` runs versioned JSON migrations, patches the singleton `Settings` object graph **in place** (preserving MVVM object and collection references), and observes `PropertyChanged` / `CollectionChanged` to write changes back to `_jsonObj`.
- Unknown JSON keys are preserved by default (supporting forward/backward compatibility and cross-platform subtrees).
- **Important DI Rule**: `IAsyncInitializer` constructors must **not** read final values from `Settings` or subscribe to settings changes in the constructor; consume `Settings` inside `InitializeAsync()` after `AsyncInitializerIndex.Settings` has completed.

---

## 7. Dependency Patching Architecture (`docs/patches.md`)

Everywhere uses three active strategies to customize upstream libraries without polluting public NuGet feeds:
1. **Source Submodule Forks (`3rd/`)**: Used for heavy UI/serialization libraries (`shad-ui`, `Motion.Avalonia`, `ExpressionBinding.Avalonia`, `ClassicDiagnostics.Avalonia`).
2. **Mirror Project Shadowing (`3rd/semantic-kernel-patch/`)**: Recompiles specific connectors (`Connectors.Google`, `Connectors.MistralAI`) by linking unmodified files from the upstream submodule and substituting only patched C# files.
3. **Compile-Time Static IL Weaving (`patches/` + `src/Build.Patches.targets`)**:
   - Uses `Everywhere.BuildTask.Patcher` (`MonoMod.Patcher`) during MSBuild `AfterTargets="ResolveAssemblyReferences"` to weave donor classes (`[MonoModPatch]`, `[MonoModReplace]`, `orig_Method` wrappers) directly into resolved third-party implementation DLLs (`Avalonia.Controls`, `Avalonia.Native`, `Avalonia.Win32`, `AvaloniaEdit`, `SemanticKernel`, `OllamaSharp`, `Microsoft.Extensions.AI.OpenAI`, `Anthropic`).
   - Produces standard, statically rewritten DLLs with zero runtime Harmony hooking overhead, avoiding Windows Defender false positives and preserving Apple/Certum code signing.
   - *Boundary*: Framework references (`FrameworkReferenceName`, e.g., `Microsoft.macOS` with native Objective-C metadata token tables) are never woven by Cecil/MonoMod.
