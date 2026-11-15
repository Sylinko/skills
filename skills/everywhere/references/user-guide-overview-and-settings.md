# Everywhere User Guide: Overview, Settings, Shortcuts & Storage Paths

## 1. Product Overview & System Requirements

**Everywhere** is a context-aware desktop AI assistant that perceives screen elements, active windows, and text selections in place using native accessibility / UI Automation APIs alongside multimodal vision.

### Minimum System Requirements
| Platform | Minimum OS Version | Notes |
| :--- | :--- | :--- |
| **Windows** | Windows 10 (`10.0.19041.0` / 2004+) | Supports Installer (`Setup.exe`) and Portable (`zip`) distributions; supports Task Scheduler Service Mode for elevated window interaction. |
| **macOS** | macOS Sonoma `14.0+` | Native `.pkg` installers for Apple Silicon (`arm64`) and Intel (`x64`). Requires Accessibility & Screen Recording permissions for visual context. |
| **Linux** | X11 Desktop Environment | In active development. |

---

## 2. Local File, Database & Configuration Paths

Everywhere separates user-editable configurations (`ConfigurationFolderPath`) from application data/databases (`WritableFolderPath`) via `RuntimeConstants`:

| Resource | Windows Path | macOS / Linux Path | Description |
| :--- | :--- | :--- | :--- |
| **Writable Data Root** (`WritableFolderPath`) | `%APPDATA%\Everywhere` | `~/Library/Application Support/Everywhere` *(macOS)* / `~/.config/Everywhere` *(Linux)* | Root directory for settings, databases, logs, and managed binaries. |
| **Settings File** | `<WritableFolderPath>\settings.json` | `<WritableFolderPath>/settings.json` | Human-readable JSON settings patched and observed by `SettingsEngine`. |
| **Databases** | `<WritableFolderPath>\db\chat.db`<br>`<WritableFolderPath>\db\prompt.db` | `<WritableFolderPath>/db/chat.db`<br>`<WritableFolderPath>/db/prompt.db` | SQLite databases for chat history/blobs and Prompt Manager resources. |
| **Managed Binaries** (`BinFolderPath`) | `<WritableFolderPath>\bin` | `<WritableFolderPath>/bin` | Portable runtime binaries managed by Everywhere. |
| **Cache Directory** (`CacheFolderPath`) | `<WritableFolderPath>\cache` | `<WritableFolderPath>/cache` | Image loader cache, model catalog cache, and temporary files (safe to clear). |
| **User Config Root** (`ConfigurationFolderPath`) | `%USERPROFILE%\.everywhere` | `~/.everywhere` | User-managed directory for skills (`~/.everywhere/skills`) and custom configs. |
| **Device ID** | `%LOCALAPPDATA%\Everywhere\.device_id` | `~/.local/share/Everywhere/.device_id` | 36-character GUID v7 generated once per machine. |

---

## 3. Global Shortcuts & Context Capture

Configured under **Settings → Shortcut** (`ShortcutSettings`):

| Shortcut Setting | Default Binding | Action |
| :--- | :--- | :--- |
| **Chat Window** (`Shortcut.ChatWindow`) | `Ctrl + Shift + E` | Summons the floating Everywhere Chat Window near the cursor/active window and optionally attaches the currently focused visual element (`AutomaticallyAddElement`). |
| **Pick Visual Element** (`Shortcut.PickVisualElement`) | *(Unbound by default)* | Enters interactive Screen/Window/Element picker mode to snap and attach a visual element to the chat. |
| **Take Screenshot** (`Shortcut.TakeScreenshot`) | *(Unbound by default)* | Enters interactive screen capture mode (supports element snapping or free-form rectangle selection). |

---

## 4. Text Selection Floating Toolbar & Automatic Attachment

Configured under **Settings → Text Selection** (`TextSelectionSettings`):

Everywhere monitors native text selection gestures across applications (via the isolated `Automation Host` with accessibility reads and clipboard fallback). It offers two independent capabilities:

1. **Floating Selection Toolbar** (`IsToolbarEnabled`):
   - Appears near selected text with quick action buttons (up to `VisibleActionCount`, default `5`, max `12`; remaining actions appear in the overflow menu) and an optional inline freeform chat input (`IsFreeformChatEnabled`, default `true`).
   - **Visual tuning**: `ToolbarOpacity` (`0.1` to `1.0`, default `1.0`) and **Adaptive Opacity** (`IsToolbarOpacityAdaptive`), which fades the toolbar as the pointer moves away.
   - **Temporary Chat Policy** (`ShouldStartToolbarChatsAsTemporary`, default `true`): When enabled, chats started from the floating selection toolbar are temporary and do not clutter persistent history.
2. **Automatic Chat Window Attachment** (`IsAutomaticAttachmentEnabled`):
   - Automatically attaches the currently selected text as a `SelectionAttachment` when the Chat Window is open or summoned, even if the floating toolbar itself is disabled.
3. **Exclusion Rules** (`Advanced` group):
   - **Exclude Fullscreen Apps** (`IsFullscreenExcluded`, default `true`): Suppresses selection monitoring when a fullscreen app/game is active.
   - **Excluded Applications** (`ExcludedApplications`): Completely disables text-selection monitoring in matching apps.
   - **Toolbar-Excluded Applications** (`ToolbarExcludedApplications`): Suppresses only the floating toolbar in matching apps while keeping automatic attachment to the Chat Window working.
   - **Platform Matching Rules**:
     - **Windows**: Case-insensitive match against either the executable file name (e.g., `code.exe`) or full executable path. Both Windows and Mac rule subtrees are preserved in `settings.json` when importing/exporting settings.
     - **macOS**: Case-sensitive exact match against the application Bundle Identifier (e.g., `com.microsoft.VSCode`).

---

## 5. Chat Window Behavior & Display Settings

### Chat Window Settings (`ChatWindowSettings`)
- **Window Pin Mode** (`WindowPinMode`): Controls how the floating chat window pins on screen or auto-hides when losing focus.
- **Temporary Chat Mode** (`TemporaryChatMode`): Controls whether new chats start as temporary by default or remember the last state.
- **Automatically Generate Title** (`AutomaticallyGenerateTitle`, default `true`): Uses the `TitleGeneration` System Assistant to name the chat after the first turn.
- **Always Start New Chat** (`AlwaysStartNewChat`, default `false`): Starts a fresh conversation every time the chat window is summoned.
- **Automatically Add Element** (`AutomaticallyAddElement`, default `true`): Automatically attaches the focused window/element when summoning the chat window.
- **Show Chat Statistics** (`ShowChatStatistics`, default `true`): Displays token usage, latency, and speed metrics on assistant turns.
- **Automatic Link Previews** (`IsAutomaticLinkPreviewEnabled`, default `true`): Loads site `/favicon.ico` (without path/query/fragment) and hover page previews using the configured proxy without sending cookies or running scripts. Links with query strings only show domain info on hover for privacy.
- **Turn Navigation Mode** (`TurnNavigationMode`, default `Fluid`): Controls how turn navigation and previews are rendered in the chat scrollbar/rail.
- **Visual Animations** (`EnableVisualElementPickAnimation`, `EnableVisualContextAnimation`, default `true`): Toggles the visual border/scan effects when picking or reading screen elements.

### Display Settings (`DisplaySettings`)
- **Language** (`Language`): Supports `default` (`en-US`), `zh-hans`, `zh-hant`, `zh-hant-hk`, `ja`, `ko`, `de`, `es`, `fr`, `it`, `pt-br`, `ru`, `tr`, etc.
- **Theme** (`Theme`): System, Light, or Dark mode, plus custom **Accent Color** (`AccentColor`), **Font Family** (`FontFamily`), and **Font Size** (`FontSize` scale `-1` to `3`, mapping to `12px`–`18px`, default `0` = `14px`).

---

## 6. Common Settings, Hosts Architecture & Troubleshooting

Configured under **Settings → Common** (`CommonSettings`):

1. **Software Updates & Channels**:
   - `IsAutomaticUpdateCheckEnabled` (default `true`) and `UpdateChannel` (Stable / Preview).
2. **Startup & Single-Instance Activation**:
   - `IsStartupEnabled` launches Everywhere at login. Secondary launches forward commands to the running Main instance and exit.
3. **Hosts Status & Windows Service Mode** (`HostsStatus`, `HostsServiceMode`):
   - Everywhere isolates native hooks and UI Automation into separate processes (`Input Host` for global shortcuts and `Automation Host` for UI Automation / text selection) supervised by `Main` and `Watchdog`.
   - **Troubleshooting Elevated Windows on Windows**: If shortcuts or visual element picking do not work when an elevated (Run as Administrator) window is focused, enable **Service Mode** in **Settings → Common → Hosts Status**. This registers a Windows Task Scheduler task so the hosts can interact across UIPI privilege boundaries cleanly without running the entire UI as Administrator.
   - If `Input Host` or `Automation Host` crashes or stalls on a hung accessibility provider, you can inspect or restart hosts from **Settings → Common → Hosts Status**.
4. **Telemetry & Privacy** (`DiagnosticData`, `IsStatisticsEnabled`):
   - Everywhere is open-source and never collects chat messages, files, API keys, or personal identifiers.
   - Turning off **Diagnostic data** (`Settings → Common → Diagnostic data`) disables all optional usage and performance telemetry, retaining only minimal anonymous crash stack traces via Sentry.
