# Everywhere Built-in Plugins, MCP Tools & Tool Approval

## 1. Built-in Chat Plugins & Functions

Everywhere organizes tools into `BuiltInChatPlugin` modules. In Strategy files (`.strategy.md`) and `PluginSettings`, plugins are addressed by their plugin key (e.g., `builtin.file_system` or `file_system`).

### 1.1 `essential` (Essential Plugin — Default Enabled)
Always-available agent orchestration and interaction tools (`ChatFunctionPermissions.None`):
- **`run_subagent`**: Launches an isolated, stateless nested agent (`chatContext.CreateNested(title)`) that has access to the same tools except `run_subagent` (preventing infinite recursion).
  - Parameters: `prompt` (detailed instructions), `title` (concise task title), `specialization` (`"default"` or `"image-understanding"`).
- **`manage_todo`**: Manages a structured task list in the chat UI (`Reset`, `Update`, `Read`, `Clear`).
- **`ask_user_question`**: Presents an interactive question card in the chat UI and waits for the user's answer.

### 1.2 `visual_context` (Visual Context Plugin — Default Enabled)
Inspects, captures, queries, and acts on the live desktop accessibility / UI Automation graph:
- **`list_windows`** (`ScreenRead`): Lists current top-level windows and returns integer visual element IDs for subsequent calls.
- **`capture_visual_element`** (`ScreenRead`): Captures a bitmap screenshot of a specific visual element by ID.
- **`query_visual`** *(Experimental, `ScreenRead`)*: Traverses or queries the live visual graph around target elements using Everywhere's bounded Weighted BFS Snapshot & Build pipeline.
- **`read_visual_text`** *(Experimental, `ScreenRead`)*: Paged UTF-16 text reading from a visual element (supports positive and negative offsets).
- **`execute_visual_actions`** *(Experimental, `ScreenAccess`)*: Executes UI Automation actions (click, invoke, focus, set value, key press, selection) on retained visual element targets, gated by internal consent.

### 1.3 `file_system` (File System Plugin — Default Enabled)
Unified file operations delegated through `FileHandlerContextFactory` (`LocalFileHandler`, `TextFileHandler`, `BinaryFileHandler`, `PdfFileHandler`, and `SkillFileHandler` for `skill://` URIs):
- **`search_files`** (`FileRead`): Regex search for file/directory names under `path` (`filePattern`, `skip`, `maxCount` up to 1000; 20s timeout; ignores common build/hidden folders).
- **`get_file_info`** (`FileRead`): Returns size, timestamps, and attributes for a file or directory (or `skill://` resource).
- **`search_file_content`** (`FileRead`): Regex content search across text files or `skill://` resources.
- **`read_file`** (`FileRead`): Reads lines from text files, extracts text from PDFs (`PdfPig`), reads hex bytes from binary files, or reads `skill://<source>.<skill>/<path>` resources.
- **`transfer_path`** (`FileAccess`): Moves or copies files/directories.
- **`delete_paths`** (`FileAccess`): Deletes specified files/directories.
- **`create_directory`** (`FileAccess`): Creates directories.
- **`apply_patch`** (`FileAccess`): Applies multi-file unified/hunk patches (`PatchParser`, `PatchHunkMatcher`, `PatchReviewSession`, `PatchCommitter`). Gathers user/auto approval decisions across affected files before committing.

### 1.4 `terminal` (Terminal Plugin)
Cross-platform PTY-backed shell execution (`Porta.Pty`):
- **`execute_in_terminal`** (`ShellExecute`, `canBypassApproval: false` for static bypass rules):
  - Detects the configured shell (`pwsh`, `Windows PowerShell`, `zsh`, `bash`, `sh`).
  - Uses **Rich Strategy (OSC 633 shell integration)** when available to track exact command boundaries (`A` prompt start, `B` command ready, `C` execution start, `D;exitCode` completion) and falls back to **None Strategy** (idle detection + VT heuristic cleaning) otherwise.
  - Note for PowerShell: Always chain single-line commands with `;` rather than `&&` when compatibility with Windows PowerShell 5.1 is required.

### 1.5 `web` (Web Plugin — Default Enabled)
Public internet search and webpage extraction:
- **`web_search`** (`NetworkAccess`): Queries the configured Web Search Engine provider (Official Everywhere AI Gateway, Google Custom Search, Brave, Tavily, Bocha, SearXNG, Jina, etc.). Requires a provider to be selected in **Settings → Web Search**.
- **`web_extract`** (`NetworkAccess`): Loads and extracts clean Markdown/readable text from a URL via `IWebBrowserHost` / AngleSharp / ReverseMarkdown.

### 1.6 `officecli` (OfficeCLI Plugin — Default Enabled)
Built-in zero-dependency binary for creating, inspecting, and editing `.docx`, `.xlsx`, and `.pptx` files:
- **`officecli`** (`FileAccess`, `isDefaultBypassApproval: true`):
  - Before invoking `officecli`, the agent MUST call `read_file` on `skill://builtin.officecli/SKILL.md` to learn the L1 (`view`/`get`/`query`/`validate`) → L2 (`add`/`set`/`remove`/`move`/`swap`/`batch`) → L3 (`raw`/`raw-set`) command syntax.

### 1.7 Platform-Specific Built-in Plugins
- **Windows — `everything`** (`EverythingPlugin`):
  - **`search_files`** (`FileRead`): Integrates with Voidtools Everything (`EverythingNet`) for instant NTFS filename search. Automatically shuts down its managed Everything process after 5 minutes of inactivity and registers with `IWatchdogManager`.
- **macOS — `system`** (`SystemPlugin`):
  - Native macOS app integrations: **`manage_reminders`**, **`manage_calendar`**, **`send_email`**, **`open_maps`**, **`manage_notes`**, **`open_url`** (`NetworkAccess`), and **`execute_applescript`** (`ShellExecute`).

---

## 2. MCP (Model Context Protocol) Integration

Everywhere supports connecting external MCP servers (`Settings.Plugin.McpChatPlugins`, keyed by `Guid`):
- **Transports** (`McpTransportConfiguration`):
  - **Stdio**: Launches a local command with arguments, working directory, and environment variables (supervised so child processes do not leak).
  - **HTTP / SSE**: Connects to remote or local HTTP MCP endpoints with optional custom headers/authentication.
- **Lifecycle**: Managed by `ManagedMcpClient` and exposed as `ManagedMcpClientTool` instances inside `ChatPluginManager`. Each MCP server appears as a configurable plugin whose tools participate in the same Tool Approval and `ToolEnablementRulesets` system as built-in plugins.

---

## 3. Tool Approval Architecture (`Ask`, `Auto`, `FullAccess`)

Everywhere enforces a two-layer consent model (`ChatService.InvokeFunctionAsync` outer gate + tool-internal `RequestConsentAsync` inner gate for resolved file paths, shell commands, and UI action batches).

### Approval Modes (`Settings.Plugin.ApprovalMode`)
| Mode | Behavior |
| :--- | :--- |
| **`Ask`** *(Default)* | Prompts the user with an interactive approval card in the chat window (allowing once, session remember, or path/rule remember where supported). |
| **`Auto`** | Delegates pending consent to a private **`ToolApprovalReviewer`** assistant (`SystemAssistantSettings.ToolApproval`). The reviewer inspects `ENVIRONMENT`, `EXECUTION_CONSTRAINTS`, `EFFECTIVE_HISTORY`, `PENDING_ACTION`, and `CONSENT_SCOPE` (and may call `read_file` if `AllowApprovalFileReads` is enabled), then calls `submit_approval`. Auto approvals are **allow-once** (never persisted as permanent bypass rules), and denials return a concise explanation to the main assistant without human fallback. |
| **`FullAccess`** | Bypasses both outer and inner approval gates (requires a local first-use warning acknowledgement in `PersistentState.HasAcknowledgedFullAccess`). Disabled tools, OS privileges, and argument validation remain enforced. |

### Turn Lifetime & Mid-Turn Switching
- A turn captures `ApprovalMode` when `GenerationContext` starts (shared with nested subagents in that turn via `GenerationApprovalState`).
- Changing the global mode while a turn is running shows a banner allowing the user to **Switch now** for the active turn or apply it only to subsequent turns.
