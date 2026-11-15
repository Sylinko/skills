# Everywhere UI Navigation & Step-by-Step Operation Guide

Use this guide when a user asks **how to use or operate Everywhere in the UI** (e.g., *"Where is the button to...?"*, *"How do I open the settings window?"*, *"How do I add an API key or MCP server?"*, *"Why did the chat window disappear when I clicked outside?"*).

---

## 1. The Four UI Surfaces of Everywhere

Everywhere does not use a single monolithic window. Instead, it separates quick in-place interaction from configuration across **four distinct UI surfaces**:

| UI Surface | Purpose | How to Open / Summon |
| :--- | :--- | :--- |
| **1. System Tray / Menu Bar Icon** (`MainTrayIcon`) | Background entry point in the Windows System Tray or macOS Menu Bar. | Always visible while Everywhere is running.<ul><li>**Single-click**: Opens the **Floating Chat Window**.</li><li>**Double-click** (within 300ms): Opens the **Main Window** (Dashboard & Settings).</li><li>**Right-click**: Opens context menu (*Open Chat Window*, *Open Main Window*, *Enable Chat Window Shortcut* checkbox, *Exit*).</li></ul> |
| **2. Floating Chat Window** (`ChatWindow`) | Compact, summonable AI chat overlay for daily conversations, visual context capture, and tool execution. | <ul><li>Press global shortcut **`Ctrl + Shift + E`** (default).</li><li>**Single-click** the System Tray icon, or right-click Tray icon → **Open Chat Window**.</li><li>Click **Start Chat** (`MessageCircle` icon) at the top-right of the **Main Window → Home** page.</li></ul> |
| **3. Main Window** (`MainView`) | Full dashboard for configuring Custom Assistants, Prompts, Plugins/MCP, Skills, Cloud Sync, and Settings. | <ul><li>**Double-click** the System Tray icon.</li><li>Right-click the System Tray icon → **Open Main Window**.</li><li>Launch the Everywhere app executable a second time while it is already running.</li></ul> |
| **4. Floating Text Selection Toolbar** (`TextSelectionToolbarWindow`) | Compact popup bar that appears automatically near selected text in any application. | Select text with the mouse or keyboard in any non-excluded application (when enabled under **Main Window → Settings → Text Selection**). |

---

## 2. Floating Chat Window (`ChatWindow`) — Visual Layout & Controls

The Floating Chat Window is organized vertically into **Title Bar**, **History Drawer** (collapsible), **Message & Strategy Area**, and **Chat Input Area**.

### 2.1 Title Bar (`TitleBarBorder`)
- **Left Side**:
  - **Evy Mascot** (`v:Evy`): Animated mascot avatar that displays a thinking/working animation while the assistant is busy.
  - **Chat Topic Title**: Shows the current chat title (automatically generated after the first turn if enabled, or `"New Chat"`).
- **Right Side Action Buttons** (left to right):
  1. **Software Update Button** *(conditional)*: Appears only when a new version of Everywhere is ready to install.
  2. **Temporary Chat Toggle** (`Ghost` icon, Hotkey: **`Ctrl + G`**):
     - Visible before messages are sent, or while the current chat is in Temporary mode.
     - When active (highlighted), the conversation is **not saved** to persistent history (`chat.db`).
     - Clicking it again inside an active temporary chat **persists (saves)** that chat to permanent history.
  3. **History Drawer Toggle** (`History` clock-arrow icon, Hotkey: **`Ctrl + H`**):
     - Opens/closes the **Chat History Drawer** (`ResponsiveDrawer`).
     - Displays an indicator dot or counter badge when background chat sessions are actively generating or have unread notifications.
  4. **New Chat Button** (`Plus` `+` icon, Hotkey: **`Ctrl + N`**):
     - Starts a fresh conversation immediately.
  5. **Pin Window Toggle** (`Pin` icon, Hotkey: **`Ctrl + D`**):
     - Cycles through **three pin states**:
       - **Unpinned** (`PinOff` icon): The Chat Window automatically hides when you click outside it (loses focus).
       - **Pinned / Always on Top** (`Pin` / 45°-rotated `Pin` icon): Keeps the Chat Window visible on screen even when you click into other apps.
  6. **Close Button** (`X` icon, Hotkey: **`Ctrl + W`**):
     - Hides/closes the Floating Chat Window (Everywhere continues running in the System Tray).

### 2.2 Chat History Drawer (`Ctrl + H`)
When opened via the `History` button (`Ctrl + H`), the drawer slides out at the top/side of the Chat Window:
- **Top Bar**:
  - **View Filter Dropdown** (top-left): Switch between **Recent** (normal saved chats), **Temporary** (active in-memory temporary sessions), and **Archived**.
  - **Search Button** (`Search` magnifying glass icon, top-right, Hotkey: **`Ctrl + Shift + F`**): Opens the search input box. Inside the search box, click the **`TextSearch` icon** on the right to toggle full-text search across message contents (instead of titles only).
  - **Bulk Actions** (when checkboxes are selected): **Select All**, **Deselect**, and **Trash (`Trash2`)** to batch-delete selected threads.
- **Per-Chat Item Actions** (hover buttons or **Right-Click Context Menu** on any history item):
  - **Rename** (`Pencil` icon): Edit the conversation title inline (`Enter` to save).
  - **Pin / Unpin** (`Pin` / `PinOff` icon): Pin important conversations to the top of the Recent list.
  - **Export Markdown** (`Upload` icon): Export the full conversation transcript as a `.md` file.
  - **Manage Chat Sync** (`Cloud` icon): Open Cloud Sync management in the Main Window.
  - **Archive / Unarchive** (`Archive` / `ArchiveRestore` icon): Move the chat into or out of the Archived tab.
  - **Persist** (`Save` floppy icon, *Temporary view only*): Convert a temporary chat into a permanently saved chat.
  - **Delete Permanently** (`Trash2` icon): Permanently delete the conversation.

### 2.3 Message Area, Strategy Recommendations & Turn Navigator
- **Strategy Recommendations**:
  - **Empty / New Chat**: Up to 4 recommended **Strategy Cards** (`240px` cards with icon, title, and description) are displayed in the center of the window based on your currently attached screen context or selected files.
  - **Ongoing Chat**: Recommended strategies shrink into a horizontal row of compact **Strategy Chips** just above the input box.
  - Clicking a Strategy card/chip attaches that strategy (shown as a tag inside the input box) and pre-fills or runs its workflow.
- **In-Chat Text Search Bar**:
  - Appears at the top-right of the message list when searching within the active conversation (`Enter` = Next match, `Shift + Enter` = Previous match, `Escape` = Close, plus a scope toggle button).
- **Message Bubble Actions** (below each message turn):
  - **User Messages**: Edit message (creates a new conversation branch), Copy text, Branch navigation (`< 1/2 >` when edited).
  - **Assistant Messages**: Collapsible **Thinking / Reasoning** block, **Tool Call cards** (with arguments, output preview, and interactive `Ask` approval buttons when confirmation is required), Copy response, Regenerate (`RefreshCw`), Branch switcher, and **Chat Statistics** (Token count, TTFT, tokens/sec).
- **Turn Navigator (`ChatTurnNavigator`)**:
  - Vertical rail along the right edge of the message list showing visual markers for each turn. Hovering over markers previews the turn; clicking jumps directly to that turn.
- **Todo List Bar (`ListChecks` icon)**:
  - Appears as an expandable bar right above the input area whenever the AI agent uses `manage_todo` to track multi-step tasks, displaying `Completed / Total` progress.

### 2.4 Chat Input Area (`ChatInputArea`)
Located at the bottom of the Chat Window:
1. **Attachment Strip** (top of the input box):
   - Horizontal scrollable strip displaying currently attached **Visual Elements** (window/control snapshots), **Screenshots**, **Selected Text**, or **Files**. Each badge has an `X` button on hover to remove it.
2. **Text Box**:
   - Type your prompt here. Supports **Drag & Drop** of files directly onto the input area (`FileUp` overlay) and `Ctrl + V` to paste images/files from the clipboard.
   - If a **Strategy** is active, its pill badge appears inline at the start of the text box (click its `X` to clear the strategy).
3. **Bottom-Left Toolbar** (three controls from left to right):
   - **Add Attachment (`Paperclip` icon)**: Clicking opens a flyout menu showing current attachment count `(n/50)` and **4 capture actions**:
     1. **Pick Visual Element** (`SquareMousePointer` icon): Interactively hover and click any window or UI control on screen to attach its accessibility tree + screenshot.
     2. **Take Screenshot** (`Fullscreen` icon): Drag a rectangle or snap to a window/element to attach a screenshot image.
     3. **Paste from Clipboard** (`ClipboardPaste` icon): Attach current clipboard image, file, or text.
     4. **Attach File** (`File` icon): Open the system file picker to attach local documents or images.
   - **Custom Assistant Selector (`ChatCustomAssistantSelector`)**:
     - Dropdown pill showing the currently selected AI assistant (icon + name). Click to switch between your configured Custom Assistants at any time.
   - **Tool Approval Selector (`ToolApprovalSelector`)**:
     - Visible when the active assistant supports tool calling. Click to switch permission mode:
       - **`Ask`**: Prompts you in the chat before executing mutating or sensitive tools.
       - **`Auto`**: Automatically approves safe/configured operations and consults the `ToolApproval` system assistant.
       - **`FullAccess`**: Approves all tool calls without prompting.
     - *Note*: If you change this while the AI is mid-turn, a banner appears above the input box with a **Switch now** button to apply it immediately to the running turn.
4. **Bottom-Right Controls**:
   - **Context Usage Ring (`ContextUsageRing`)**:
     - Circular progress ring that appears once the conversation has messages.
     - **Click the ring** to open a flyout showing **Used %**, **Total / Context Limit Tokens**, **Cached Tokens**, and a **Compact Context** button to manually compress/summarize conversation history right away.
   - **Send / Cancel Button**:
     - **Send** (`Send` paper-plane icon): Sends the message (`Enter` or `Ctrl + Enter`).
     - **Cancel** (Square stop icon): Appears while the assistant is generating; click to abort generation immediately.

---

## 3. Main Window (`MainView`) — Sidebar Navigation & Pages

Open the Main Window by **double-clicking the System Tray icon** (or right-clicking the Tray icon → **Open Main Window**).
The left **Navigation Sidebar** (which collapses into a bottom navigation bar if the window is narrower than `700px`) contains **6 main sections** plus a **User Profile / Cloud Sync footer** at the bottom-left:

```text
Main Window Sidebar
├── 🏠 Home (House icon)
├── 🤖 Custom Assistants (Bot icon)
├── 📖 Prompt Manager (BookText icon)
├── 🔨 Chat Plugins (Hammer icon)  --> [Built-in Tab] | [MCP Tab]
├── 📦 Skills (Box icon)
├── ⚙️ Settings (Cog icon, expandable)
│    ├── Common
│    ├── Display
│    ├── Shortcut
│    ├── Proxy
│    ├── Chat Window
│    ├── System Assistant
│    └── Text Selection
└── 👤 User Profile / Cloud Account (Bottom-left footer button)
```

### 3.1 Home (`House` icon)
- **Top-Right**: **Start Chat** primary button (`MessageCircle` icon) — opens the Floating Chat Window.
- **Dashboard Cards & Charts**: Displays usage metrics (Total Topics, Turns/Branches, Token Consumption, etc.) and activity charts.
- **Bottom Footer Quick Links**:
  - **Welcome Page**: Re-launches the initial Onboarding / Setup Wizard (OOBE).
  - **What's New**: Opens the version changelog dialog.
  - **GitHub Repo**, **Open Source Licenses**, **Official Docs**, **Discord**, **QQ Group**.

### 3.2 Custom Assistants (`Bot` icon)
Where you configure AI models, API keys, system prompts, and per-assistant tool overrides:
- **Left Pane (`240px`)**:
  - **New (`+`)** button at top-right of the list header: Creates a new Custom Assistant.
  - **Draggable Assistant List**: Hover over any assistant to reveal the **`GripVertical` drag handle** on the left; drag up or down to reorder how assistants appear in the Chat Window selector.
- **Right Pane Header** (for the selected assistant):
  - **Check Connectivity** (`Zap` lightning icon): Sends a test request to verify your API Key, Endpoint, and Model ID.
  - **Duplicate** (`Copy` icon): Clones the selected assistant configuration.
  - **Delete** (`Trash2` icon): Deletes the selected assistant.
- **Right Pane Configuration Tabs**:
  - **Identity Header**: Set Assistant **Icon** (colored avatar picker), **Name**, and **Description**.
  - **Model Configuration**: Switch between **Official Model Provider** (Everywhere Cloud credits) or **Custom Model Provider** (choose **Schema**: `OpenAIResponses`, `OpenAI`, `Anthropic`, `Google`, `DeepSeek`, `Moonshot`, `MiniMax`, `Mistral`, `Ollama`; enter **Endpoint**, **API Key** via API Key Manager, **Model ID**, **Max Tokens**, **Reasoning Effort**, vision/tool toggles, Temperature/TopP).
  - **Prompt Configuration**: Select which prompt from **Prompt Manager** this assistant uses (defaults to `Default`).
  - **Tool Settings**: Override global plugin states specifically for this assistant.
  - **Advanced Settings**: Configure `VisualContextLengthLimit` and `ContextCompressionThreshold`.

### 3.3 Prompt Manager (`BookText` icon)
Manage reusable system prompts and `{Placeholder}` templates:
- **Left Pane (`260px`)**:
  - **New Prompt (`+`)** button, **Search box**, and list of prompts (the built-in **Default** prompt is pinned at the top with a green badge).
- **Right Pane**:
  - **Header Buttons**: **Copy** (copies rendered preview to clipboard), **Edit** (`Pencil` icon), and **Delete** (`Trash2` icon, disabled for built-in Default).
  - **Diagnostic Banners**: Non-dismissible warning cards appear automatically if a custom prompt omits `{SkillsPrompt}` or places volatile `{Time}` / `{VisualContext}` variables before static instructions (which breaks LLM prompt caching).
  - **References Section**: Shows which Custom Assistants are currently linked to this prompt.
  - **Editor Mode** (when clicking **New** or **Edit**): Supports both a **Guided Editor** (structured Role, Tone, Context & Capabilities toggles) and an **Advanced Template Editor** with a clickable `{Placeholder}` sidebar (`{OS}`, `{Time}`, `{Language}`, `{WorkingDirectory}`, `{VisualContext}`, `{System}` / `{BuiltinPrompt}`, `{SkillsPrompt}`, `{Variables}`).

### 3.4 Chat Plugins (`Hammer` icon)
Manage built-in tools and external **MCP (Model Context Protocol)** servers:
- **Left Pane (`320px`) — Two Tabs at the Top**:
  1. **Built-in Tab (`Box` icon)**:
     - Lists Everywhere's built-in tool plugins: **Essential**, **Visual Context**, **File System**, **Terminal**, **Web**, **OfficeCLI**, and **Everything** *(Windows)* / **System** *(macOS)*.
     - Each row has an **Enable/Disable toggle switch** on the right, and a yellow **Warning icon (`TriangleAlert`)** if required settings are missing (for example, when **Web** is enabled without a search provider API key).
  2. **MCP Tab (MCP icon)**:
     - Selecting the MCP tab reveals two action buttons in the left pane header:
       - **Add (`+` icon)**: Opens a dialog to configure a new MCP server (**Stdio** command/args/env or **HTTP / SSE** endpoint/headers).
       - **Import (`Import` icon)**: Import MCP server definitions directly from standard MCP JSON configuration text/files.
     - Each MCP server row displays a **Status Dot** (**Green** = running, **Gray** = stopped) and an **Enable toggle switch**.
- **Right Pane (Plugin Details)**:
  - **MCP Action Bar** *(visible when an MCP server is selected)*:
    - **Start (`Play` icon) / Stop (`Square` icon)**: Manually start or stop the MCP server process/connection.
    - **Edit (`SquarePen` icon)**: Modify the MCP transport, command, environment variables, or URL.
    - **Remove (`Trash2` icon)**: Delete the MCP server.
    - **Bypass Approval Toggle** & **Runtime Log Viewer**: View live stdout/stderr connection logs from the MCP server.
  - **Settings Tab** *(visible when the plugin has settings, e.g., Web Search, File System, Terminal)*:
    - Configure plugin-specific options (such as **Web Search** provider: *Tavily*, *Brave*, *BoCha*, *Jina*, *UniFuncs*, *Google*, or *SearXNG* + API Key).
  - **Functions Tab**:
    - Lists every individual tool function exposed by the plugin, its permission category badge (Read-Only, File Write, Shell Execute, Network, etc.), an individual **Enable/Disable toggle**, and a **Bypass Approval** toggle.

### 3.5 Skills (`Box` icon)
Inspect and manage Agent Skills (`SKILL.md`) discovered across your system:
- **Left Pane (`400px`)**:
  - **Source Filter Dropdown**: Filter by *All*, *Built-in*, *Everywhere* (`~/.everywhere/skills`), *Agents* (`~/.agents/skills`), *Claude*, *Codex*, *Copilot*, *Cursor*, or *Gemini*.
  - **Search Box** & **Rescan (`RefreshCw` icon)** button: Click Rescan after adding or editing `SKILL.md` files on disk.
  - **Skill Groups**: Expandable groups by source. Each skill row displays validation warnings (`TriangleAlert` if frontmatter or folder name is invalid) and an **Enable/Disable toggle switch**.
- **Right Pane**:
  - Shows the skill's description, canonical `skill://` URI / local path, license, metadata table, validation diagnostics, and rendered Markdown body.

### 3.6 Settings (`Cog` icon)
Clicking **Settings** expands 7 sub-pages in the sidebar:
1. **Common** (`Blocks` icon):
   - Software Update check button & Automatic Update toggle, Launch at Startup toggle, **Hosts Status** card & **Windows Service Mode** toggle (for interacting with Administrator windows on Windows), Update Channel (Stable / Preview), Statistics & Diagnostic Data toggles, and Debug Features.
2. **Display** (`MonitorCog` icon):
   - UI Language (`en-US`, `zh-hans`, `zh-hant`, `ja`, etc.), Theme (`System`, `Light`, `Dark`), Accent Color, Font Family, and Font Size (`-1` to `3`).
3. **Shortcut** (`Keyboard` icon):
   - Record custom global hotkeys for **Chat Window** (default `Ctrl+Shift+E`), **Pick Visual Element**, and **Take Screenshot**.
4. **Proxy** (`WifiCog` icon):
   - **Enable Custom Proxy** toggle, **Endpoint** (e.g., `http://127.0.0.1:7890` or `socks5://127.0.0.1:1080`), **Bypass on Local**, **Bypass List**, and **Use Authentication** (Username / Password).
5. **Chat Window** (`MessageCircle` icon):
   - Default **Window Pin Mode**, **Temporary Chat Mode**, **Automatically Generate Title**, **Always Start New Chat**, **Automatically Add Focused Element**, **Show Chat Statistics**, **Automatic Link Previews**, **Turn Navigation Mode**, and **Visual Animations**.
6. **System Assistant** (`Sparkles` icon):
   - Configure which model powers background system tasks (**Title Generation**, **Tool Approval** + *Allow Approval Assistant to Read Files*, **Default Subagent**, and **Image Understanding**). Defaults to `AutoSelect`.
7. **Text Selection** (`TextSelect` icon):
   - **Floating Toolbar** master toggle & live interactive preview, **Toolbar Opacity** & **Adaptive Fade**, **Freeform Chat Input** toggle, **Visible Action Count** & Action Manager, **Automatic Chat Window Attachment** toggle, **Exclude Fullscreen Apps**, **Start Toolbar Chats as Temporary**, and **Excluded Applications** / **Toolbar-Excluded Applications** rule lists.

### 3.7 Cloud Account & Chat Sync Footer (`UserProfilePresenter`)
Located at the **bottom-left corner of the Main Window sidebar**:
- Clicking the user avatar/icon opens the **User Profile Flyout**:
  - **Logged Out**: Click **Login** to sign in to Everywhere Official Cloud via browser OAuth.
  - **Logged In**:
    - Shows your current **Plan Badge**, **Plan Credits**, **Bonus Credits**, **5-Hour / 7-Day Quota Windows**, and **Free Web Search Quota**.
    - Quick buttons: **Manage Subscription**, **Open Dashboard**, **Get Support**, and **Logout**.
    - **Cloud Sync Toggle Switch**: Enables/disables end-to-end encrypted synchronization of chats, assistants, and prompts across devices.
    - **Manage Chat Sync (`Cloud` icon)**: Opens the dedicated **Chat Sync Page** (`ChatSyncPage`) inside the Main Window to inspect cloud storage usage, resolve quota warnings, manage encryption/sync settings, or selectively clean up synced conversations.

---

## 4. "How Do I...?" — Step-by-Step User Recipes

When guiding a user, give them the exact click path below:

### Q1: How do I open the Settings / Main Window? I only see the small Chat Window.
1. Look at the **System Tray** (bottom-right of Windows taskbar) or **Menu Bar** (top-right on macOS) and find the **Everywhere icon**.
2. **Double-click** the Everywhere icon (or **right-click** it and select **Open Main Window**).

### Q2: How do I add my own API Key or configure a model (OpenAI, Claude, DeepSeek, Ollama, etc.)?
1. Open the **Main Window** (double-click the System Tray icon).
2. In the left sidebar, click **Custom Assistants** (`Bot` icon, 2nd item).
3. Click the **`+` (New)** button at the top of the left list (or select an existing assistant).
4. Under **Model Provider**, switch from Official to **Custom** (if applicable), choose the **Schema** (e.g., `OpenAIResponses`, `Anthropic`, `DeepSeek`, `Ollama`), enter the **Endpoint** and **API Key**, and select or type the **Model ID**.
5. Click the **`Zap` (Check Connectivity)** button at the top-right to verify the connection works.

### Q3: How do I keep the Floating Chat Window from disappearing when I click another window?
- **For the current session**: In the top-right title bar of the Floating Chat Window, click the **Pin (`Pin`) icon** (or press **`Ctrl + D`**) until the pin icon is active.
- **Permanently by default**: Open **Main Window → Settings → Chat Window**, and change **Window Pin Mode** so the window stays pinned by default.

### Q4: How do I enable Web Search (`web_search`)?
1. Open the **Main Window → Chat Plugins** (`Hammer` icon).
2. In the left pane under the **Built-in** tab, make sure **Web** is toggled **On**, then click **Web** to open its details on the right.
3. In the right pane under the **Settings** tab, select a **Web Search Provider** (e.g., *Tavily*, *Brave*, *BoCha*, *Jina*, *Google*, *SearXNG*, or *Everywhere Official* if logged into Cloud) and enter your API Key / URL.

### Q5: How do I add or import an MCP Server?
1. Open the **Main Window → Chat Plugins** (`Hammer` icon).
2. At the top of the left pane, switch from **Built-in** to the **MCP** tab.
3. Click **`+` (Add)** to configure a `Stdio` or `HTTP/SSE` server manually, or click **Import** to paste a standard MCP JSON configuration.
4. Once added, toggle the switch **On** (or click the **Play** button in the right header) and check that the status dot turns **Green**.

### Q6: How do I capture a specific window or UI element on screen for the AI to see?
1. Open the **Floating Chat Window** (`Ctrl + Shift + E`).
2. Click the **Paperclip (`Add Attachment`)** button at the bottom-left of the input box.
3. Choose **Pick Visual Element** (to inspect a window/control's UI hierarchy + screenshot) or **Take Screenshot** (to capture an image region).
   *(Tip: You can also bind global hotkeys for both actions in **Main Window → Settings → Shortcut**).*

### Q7: How do I stop the Floating Text Selection Toolbar from popping up in a specific app (e.g., VS Code or a game)?
1. Open **Main Window → Settings → Text Selection**.
2. Scroll down to the **Advanced** section:
   - To hide **only the popup toolbar** while still automatically attaching selected text when you open the Chat Window, add the app to **Toolbar-Excluded Applications**.
   - To completely disable text selection monitoring in that app, add it to **Excluded Applications**.
   - *(On Windows, enter the executable name like `Code.exe` or full path; on macOS, enter the Bundle ID like `com.microsoft.VSCode`).*

### Q8: How do I save a Temporary Chat (`Ghost` mode) so it isn't lost?
- **While still inside the chat**: Click the highlighted **Ghost (`Temporary Chat`)** button in the top-right title bar of the Chat Window (or press **`Ctrl + G`**).
- **From the History Drawer**: Press **`Ctrl + H`** to open the History Drawer, change the top-left filter dropdown from *Recent* to **Temporary**, hover over the chat (or right-click it), and click **Persist (`Save` icon)**.

### Q9: How do I manually compress/summarize a long conversation when context tokens get high?
1. In the bottom-right of the Chat Window input box (next to the Send button), click the circular **Context Usage Ring**.
2. In the popup flyout showing token usage, click **Compact Context**.
