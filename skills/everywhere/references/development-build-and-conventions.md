# Everywhere Development Workflow, Build & Coding Conventions

Read this guide before modifying C# or AXAML code, adding localization strings, or running builds/tests in the Everywhere repository (`D:\Source\CSharp\Everywhere`).

---

## 1. Change Discipline & Source of Truth (`AGENTS.md`)

1. **Inspect Code Before Editing**: Design docs under `docs/` may lag behind later refactorings. Always inspect current implementations and call sites first.
2. **Dependency Discipline**: Discuss any new NuGet packages with the user before introducing them.
3. **Clean Up Obsolete Code**: Remove obsolete compatibility layers when they have no real callers and are not required by a public API or serialized contract.
4. **Simplicity by Ownership**: Optimize for the application's real ownership, call paths, and lifetime rather than generic hypothetical consumers. Within trusted nullable-enabled solution code, rely on the type system rather than redundant null checks; validate at external, serialization, reflection, plugin, or native boundaries.

---

## 2. C# Coding Style & Conventions

- **Formatting**: Follow `.editorconfig`. Use `var` for all local variable declarations.
- **Visibility**: Prefer `public` for APIs unless a concrete ownership, lifetime, or safety boundary requires `internal` or `private`. Non-constant fields must be `private` (prefixed with `_camelCase`, e.g., `_isDisposed`).
- **Boolean Naming**: Give all boolean properties, fields, parameters, and locals a semantic prefix (`Is`, `Has`, `Can`, `Should`), e.g., `_isDisposed`, `HasCount`.
- **Member Ordering Within a Type**:
  1. Properties
  2. Events
  3. Fields
  4. Methods
  *(Static or `readonly` members may appear earlier within their category. Place `public` nested types at the top or bottom; `private` nested types at the bottom.)*
- **Expression-Bodied Members**: Use only when the entire declaration and expression fit readably on a single line. Use block bodies if either spans multiple lines.
- **Control Flow**: Prefer guard clauses and early returns over deep nesting.
- **Async Fire-and-Forget**: **Never** write `_ = DoAsync()`. Always use `TaskExtensions.Detach` (e.g., `DoAsync().Detach(IExceptionHandler.DangerouslyIgnoreAllException)` or appropriate handler).
- **Nullability**: Avoid `!` (null-forgiving operator) whenever possible. Use `[NotNullWhen]`, `[MemberNotNull]`, and explicit `if` checks.
- **Records**: Do not retain unused positional-record parameters (they become captured state and affect equality/hashing).
- **Comments & Docs**: Write all code comments, XML `<summary>` (prefer multiline `<summary>`), and technical docs under `docs/` in **English**. Use `<inheritdoc />` when the base/interface contract suffices.

---

## 3. Avalonia UI, Views & Presentation Rules (`docs/References/AvaloniaViewPresentation.md`)

1. **ViewModels vs. Presentation Objects**:
   - Treat ViewModels as heavy. For simple projections or row state, prefer lightweight Presentation objects, models, or custom controls.
   - Keep Presentation object identity stable across incremental updates to avoid binding churn, lost control state, and animation flicker.
2. **UI Thread Affinity**:
   - Marshal background events to the UI thread at the ingress boundary using `Dispatcher.UIThread.PostOnDemand`.
   - Once inside a UI-thread-affine View or Presentation object, keep the model single-threaded (`List<T>` instead of concurrent/immutable collections or locks).
   - `SourceList.Items` (DynamicData) can be read directly as a stable snapshot; update projections incrementally via DynamicData change sets without rebuilding entire collections.
3. **AXAML & Control Design**:
   - Inspect existing controls in `Views/Chat` and `Views/Controls` before creating new AXAML.
   - Prefer `Panel` (shared slot) or `DockPanel` (constrained child + fill child with `HorizontalSpacing`/`VerticalSpacing`) over `Grid` unless row/column sizing is genuinely needed.
   - Create expensive visual trees (terminals, detailed Markdown, diffs) lazily on demand when visible.
   - Use `RenderTransform` and `Opacity` for animations rather than mutating layout dimensions.

---

## 4. Localization (`Everywhere.I18N`)

Everywhere uses source-generated localization APIs from `.resx` files (`src/Everywhere.I18N.SourceGenerator`):
- **Persistent UI State**: Carry `IDynamicLocaleKey` (`DynamicLocaleKey`, `FormattedDynamicLocaleKey`, `AggregateDynamicLocaleKey`, or `DirectLocaleKey` for literal strings like paths) through models and RPC contracts, and bind via `{Binding MessageKey^}` in AXAML so live language switching works automatically.It supports MessagePack serialization—do not resolve it to a string before presentation.
- **One-Shot Snapshots**: Use `LocaleKey.SomeKey.I18N()` only for transient snapshots such as Toast or Dialog text.
- **Key Naming**: Name keys after their semantic meaning and owning area; prefix genuinely shared keys with `Common_`.
- **Logs & CLI Output**: Keep diagnostic logs and CLI stdout/stderr in English; never use raw CLI output as a localized UI message contract.
- **Syncing Translations**: Use `python tools/sync_i18n.py` and `python tools/validate_i18n.py` when managing i18n resources.

---

## 5. Toolchain, Building & Testing (`docs/build.md`)

### Prerequisites
- **.NET SDK**: `11.0.100` (configured in `global.json` with `rollForward: latestPatch`).
- **Git LFS**: Run `git lfs install` (and `git config --global core.longpaths true` on Windows).
- **Platform Solution Filters**:
  - Windows: `Everywhere.Windows.slnx` (or full `Everywhere.slnx`)
  - macOS: `Everywhere.Mac.slnx` (requires Xcode `27.2` + `sudo dotnet workload install macos --version 11.0.100`; avoid unversioned `dotnet workload update`)
  - Linux: `Everywhere.Linux.slnx`

### Common Build & Test Commands
```bash
# Build a specific project or platform solution filter
dotnet build Everywhere.Windows.slnx

# Run core unit tests
dotnet test tests/Everywhere.Core.Tests/Everywhere.Core.Tests.csproj

# Run specific test suites (e.g., Terminal, AI, Automation, ProcessIsolation, Patcher)
dotnet test tests/Everywhere.Core.Tests/Everywhere.Core.Tests.csproj --no-restore --filter "FullyQualifiedName~Everywhere.Core.Tests.Terminal"
dotnet test tests/Everywhere.AI.Tests/Everywhere.AI.Tests.csproj
dotnet test tests/Everywhere.Automation.Tests/Everywhere.Automation.Tests.csproj
dotnet test tests/Everywhere.ProcessIsolation.Tests/Everywhere.ProcessIsolation.Tests.csproj
dotnet test tests/Everywhere.BuildTask.Patcher.Tests/Everywhere.BuildTask.Patcher.Tests.csproj
```

### Testing & Verification Discipline
- **Test Method Naming**: `PascalCase` with underscores: `Scenario_Action_ExpectedResult` (e.g., `UpdateContent_WhenConditionBecomesFalse_ClearsChild`).
- **No Test-Only Production Hooks**: Do not add methods, constructors, or properties to production types solely for testing; use production entry points, mocks, reflection, or `[UnsafeAccessor]`.
- **Temporary Tests**: Mark one-off investigative tests with `// [TEMPORARY] ...`, keep them unstaged, and list their locations at handoff.
- **Post-Implementation Check**: Use Rider MCP (if available) alongside `dotnet build` / `dotnet test` to verify zero errors, warnings, or code-analysis regressions.
