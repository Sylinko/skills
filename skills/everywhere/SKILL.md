---
name: everywhere
description: >-
  Comprehensive knowledge base for Everywhere (Sylinko's context-aware cross-platform desktop AI assistant) and its C#/.NET 11 codebase.
  Use when guiding users on how to operate the Everywhere UI (Tray icon, Floating Chat Window, Main Window pages, buttons, hotkeys, and step-by-step workflows), answering questions about features, settings, text selection toolbar, custom/system assistants, model providers, Prompt Manager, authoring Strategy Engine (.strategy.md) or Skills (SKILL.md) files, built-in tools/MCP, or troubleshooting; OR when developing, debugging, or reviewing the Everywhere source code (Avalonia UI, Visual Context & Automation, Process Isolation, LLM Request runtime, Terminal PTY, Settings Engine, I18N, and IL patching).
license: Apache-2.0
metadata:
  author: Sylinko
  version: "1.0.0"
  repository: "https://github.com/Sylinko/Everywhere"
---

# Everywhere Knowledge Base & Developer Guide

**Everywhere** (`Sylinko/Everywhere`) is a context-aware, cross-platform desktop AI assistant built with **.NET 11** and **Avalonia UI** for Windows (`10.0.19041+`), macOS (`Sonoma 14.0+`), and Linux (X11, in development). Unlike traditional chat apps, Everywhere perceives live screen context via native UI Automation / Accessibility APIs, offers a floating Text Selection Toolbar, recommends context-driven actions via its **Strategy Engine**, and executes tools in isolated/supervised environments.

---

## How to Load Reference Documents (Progressive Disclosure)

Keep context usage minimal by reading only the reference files relevant to the current task.

> **Path Resolution Rule for `read_file`**:
> - **Inside Everywhere (`skill://` URI)**: If this file was loaded from `skill://<source>.everywhere/SKILL.md` (e.g., `skill://builtin.everywhere/SKILL.md` or `skill://everywhere.everywhere/SKILL.md`), pass `skill://<source>.everywhere/references/<filename>.md` to `read_file`.
> - **External Coding Agents (Filesystem)**: If this file was loaded from a local file path `<dir>/SKILL.md`, pass `<dir>/references/<filename>.md` to your file-reading tool.

### Track A: Product Usage, UI Operation, Configuration & Authoring (For End-Users & In-App Assistant)

1. **[UI Navigation, Window Layout & Step-by-Step Operation Guide](references/ui-navigation-and-operations.md)**
   - **Read first when a user asks how to operate Everywhere or where a feature/button is located in the UI** (e.g., how to open the Main Window vs. Floating Chat Window via the System Tray icon, Chat Window title bar buttons & hotkeys (`Ctrl+G`, `Ctrl+H`, `Ctrl+N`, `Ctrl+D`, `Ctrl+W`), History Drawer (`Ctrl+Shift+F`, Recent/Temporary/Archived views, right-click actions), Input Area controls (`Paperclip` attachment menu, Assistant/Tool Approval selectors, Context Usage Ring & manual compaction), Main Window sidebar pages (`Home`, `Custom Assistants`, `Prompt Manager`, `Chat Plugins` Built-in vs. MCP tabs, `Skills`, `Settings` sub-pages, Cloud Sync footer), and step-by-step *"How Do I...?"* recipes).
2. **[User Guide, Settings, Shortcuts & Storage Paths](references/user-guide-overview-and-settings.md)**
   - Read when the user asks about system requirements, local file/database/log paths (`%APPDATA%/Everywhere`, `~/.everywhere`), global keyboard shortcuts (`Ctrl+Shift+E`), detailed Text Selection floating toolbar settings & app exclusion rules, Chat Window pin/temporary/navigation modes, Windows Service Mode (`HostsStatus`), or telemetry/privacy settings.
3. **[Assistants, Model Providers & Prompt Manager](references/assistants-models-and-prompts.md)**
   - Read when configuring Custom Assistants, System Assistants (`TitleGeneration`, `ToolApproval`, `DefaultSubagent`, `ImageUnderstanding`), model providers/schemas (`OpenAIResponses`, `OpenAI`, `Anthropic`, `Google`, `DeepSeek`, `Moonshot`, `MiniMax`, `Mistral`, `Ollama`, Official Cloud), context compression, or managing prompts and `{Placeholders}` in Prompt Manager (`prompt.db`).
4. **[Built-in Plugins, MCP Tools & Tool Approval](references/plugins-mcp-and-tool-approval.md)**
   - Read when the user asks what tools Everywhere has (`essential`, `visual_context`, `file_system`, `terminal`, `web`, `officecli`, Windows `everything`, macOS `system`), how to configure MCP servers, or how Tool Approval modes (`Ask`, `Auto`, `FullAccess`) and path permissions work.
5. **[Authoring Strategies (`.strategy.md`) & Skills (`SKILL.md`)](references/strategies-and-skills-authoring.md)**
   - Read when creating, debugging, or explaining Everywhere **Strategies** (`everywhere.strategy/v1`, `when` Condition DSL, `visual.exists`/`visual.match` XPath-like queries, `extra.file_manager.*`, `from` inheritance, `tools` rulesets, `preprocessors`) or managing local **Skills** (`~/.everywhere/skills`, `~/.agents/skills`, `skill://` URIs).

### Track B: Source Code Architecture & Engineering (For Contributors & Coding Agents)

6. **[Architecture, Assemblies & Core Subsystems](references/architecture-and-modules.md)**
   - Read when exploring or modifying the Everywhere repository (`Everywhere.slnx`), understanding assembly boundaries, **Process Isolation** (`Main`, `Input Host`, `Automation Host`, `--hosts-control`, `Watchdog`, RPC), **Visual Context & Automation** (Weighted BFS Snapshot vs Build pipeline, `CompositeTarget`, Text Selection monitoring), **LLM Request Runtime** (`StreamRequestAsync`, attempt checkpoint rollback, exception normalization), **Terminal PTY** (`TerminalSession`, `TerminalParser`, OSC 633), **Settings Engine** (`_jsonObj` patching), or **Dependency Patching** (`3rd/` forks, mirror projects, and `MonoMod` static IL weaving).
7. **[Development Workflow, Build & Coding Conventions](references/development-build-and-conventions.md)**
   - Read before writing C# or AXAML code, running builds/tests, adding localization keys (`IDynamicLocaleKey` / `.I18N()`), or working on Avalonia Views/Presentation objects. Summarizes mandatory repository rules (`AGENTS.md`, `AvaloniaViewPresentation.md`, `.NET 11` & macOS Xcode workload setup).

---

## Quick Orientation Cheat Sheet

### UI Surfaces & Navigation Quick Map
| UI Surface | How to Open | Key Controls & Navigation |
| :--- | :--- | :--- |
| **System Tray / Menu Bar Icon** | Always running in tray/menu bar | **Single-click**: Open Floating Chat Window · **Double-click**: Open Main Window · **Right-click**: Context menu (*Open Chat Window*, *Open Main Window*, *Enable Chat Window Shortcut*, *Exit*). |
| **Floating Chat Window** | `Ctrl+Shift+E` or single-click Tray icon | **Title Bar**: Temporary Chat (`Ghost`, `Ctrl+G`), History Drawer (`History`, `Ctrl+H`, search `Ctrl+Shift+F`), New Chat (`+`, `Ctrl+N`), Pin Window (`Pin`, `Ctrl+D`), Close (`X`, `Ctrl+W`).<br>**Input Area**: Attachment menu (`Paperclip`: Pick Visual Element, Screenshot, Paste, File), Custom Assistant selector, Tool Approval selector (`Ask`/`Auto`/`FullAccess`), Context Usage Ring (click to **Compact Context**), Send/Cancel. |
| **Main Window (Dashboard)** | Double-click Tray icon or right-click → *Open Main Window* | **Left Sidebar**: `Home` → `Custom Assistants` (API keys/models) → `Prompt Manager` → `Chat Plugins` (`Built-in` & `MCP` tabs) → `Skills` → `Settings` (`Common`, `Display`, `Shortcut`, `Proxy`, `Chat Window`, `System Assistant`, `Text Selection`) → Bottom-left **User Profile / Cloud Sync** button. |
| **Text Selection Toolbar** | Select text in any non-excluded app | Floating quick-action buttons + overflow menu + inline freeform input (configured in **Main Window → Settings → Text Selection**). |

### Key Local Directories (User Machine)
| Path | Purpose |
| :--- | :--- |
| `~/.everywhere/` | User configuration folder (`ConfigurationFolderPath`): user skills (`~/.everywhere/skills`), strategies, and small editable configs. |
| `%APPDATA%/Everywhere` *(Win)* / `~/Library/Application Support/Everywhere` *(macOS)* | Writable data folder (`WritableFolderPath`): `settings.json`, `db/chat.db`, `db/prompt.db`, logs, `bin/`, and `cache/`. |
| `%LOCALAPPDATA%/Everywhere/.device_id` | Persistent 36-char GUID device identifier. |
| `~/.agents/skills/` | Cross-agent shared skills directory (enabled by default in Everywhere). |

### Core Built-in Plugin Keys & Functions
| Plugin Key | Functions |
| :--- | :--- |
| `essential` | `run_subagent`, `manage_todo`, `ask_user_question` |
| `visual_context` | `list_windows`, `capture_visual_element`, `query_visual`, `read_visual_text`, `execute_visual_actions` |
| `file_system` | `search_files`, `get_file_info`, `search_file_content`, `read_file` (supports local files, PDFs, and `skill://`), `transfer_path`, `delete_paths`, `create_directory`, `apply_patch` |
| `terminal` | `execute_in_terminal` (PTY-backed with OSC 633 shell integration) |
| `web` | `web_search`, `web_extract` |
| `officecli` | `officecli` (single-binary `.docx`/`.xlsx`/`.pptx` tool; read `skill://builtin.officecli/SKILL.md` before use) |
| `everything` *(Win)* | `search_files` (ultra-fast NTFS filename search via Everything engine) |
| `system` *(macOS)* | `manage_reminders`, `manage_calendar`, `send_email`, `open_maps`, `manage_notes`, `open_url`, `execute_applescript` |

### High-Level Architecture Map
```text
Everywhere.Windows / Everywhere.Mac / Everywhere.Linux (Platform Entry & Native Interop)
  │
  ├──► Everywhere.Core (UI, ChatService, StrategyEngine, Skills, Plugins, AI Mixins, SettingsEngine)
  │      ├──► Everywhere.Abstractions (Shared Interfaces, RuntimeConstants, Skill/Strategy Contracts)
  │      ├──► Everywhere.Automation (Platform-neutral VisualElement, VisualContext, Snapshot/Build Pipeline)
  │      ├──► Everywhere.ProcessIsolation (Main <-> Input Host / Automation Host RPC & Resource Handles)
  │      ├──► Everywhere.Prompting (PromptNode, PromptCompactElement, Document Renderers)
  │      ├──► Everywhere.Terminal (PTY Session, VT/CSI/OSC 633 Parser, TerminalRun, TerminalLineBuffer)
  │      └──► Everywhere.Cloud (OAuth, Official Model Gateway, Chat & Resource Sync)
  │
  ├──► Everywhere.Watchdog (Process lifecycle supervision)
  └──► Source Generators & Build Tasks
         ├──► Everywhere.Configuration.SourceGenerator ([GeneratedSettingsItems], JSON Patch Descriptors)
         ├──► Everywhere.I18N.SourceGenerator (Resx -> LocaleKey & IDynamicLocaleKey)
         ├──► Everywhere.ProcessIsolation.SourceGenerator (RPC Proxies & Dispatchers)
         └──► Everywhere.BuildTask.Patcher (Compile-time MonoMod Static IL Weaving forpatches/*)
```
