# Authoring Everywhere Strategies (`.strategy.md`) & Skills (`SKILL.md`)

Everywhere distinguishes between **Skills** (discoverable long-form instruction resources read on demand via `skill://` URIs) and **Strategies** (contextual recommended actions in the chat UI that evaluate live screen/attachment/clipboard context and configure prompt + tool rules for a single user send).

---

## 1. Skills v1 System (`SKILL.md`)

### 1.1 Discovery Roots & Default Enablement
Everywhere automatically watches and indexes `root/<skill-folder>/SKILL.md` (one level deep under each root):

| Source Root (`SkillSourceRoot`) | Path | Default Enabled? | Canonical ID Prefix |
| :--- | :--- | :--- | :--- |
| **`BuiltIn`** | Assembly embedded resources (`Assets/Skills/<folder>/SKILL.md`) | **Yes** | `builtin.<folder>` |
| **`Everywhere`** | `~/.everywhere/skills` | **Yes** | `everywhere.<folder>` |
| **`Agents`** | `~/.agents/skills` | **Yes** | `agents.<folder>` |
| **`Claude`** | `~/.claude/skills` | No (User toggleable) | `claude.<folder>` |
| **`Codex`** | `~/.codex/skills` (or `$CODEX_HOME/skills`) | No (User toggleable) | `codex.<folder>` |
| **`Copilot`** | `~/.copilot/skills` | No (User toggleable) | `copilot.<folder>` |
| **`Cursor`** | `~/.cursor/skills` | No (User toggleable) | `cursor.<folder>` |
| **`Gemini`** | `~/.gemini/skills` | No (User toggleable) | `gemini.<folder>` |

### 1.2 Skill Identity, Validation & `skill://` URIs
- **Skill ID**: `<source-root>.<normalized-folder-name>` (e.g., `everywhere.code-review`, `builtin.officecli`).
- **Frontmatter Validation (`SkillParser`)**:
  - `name` (required): Must match the parent folder name with **exact case** (`StringComparison.Ordinal`), otherwise Everywhere emits a `skill.name_folder_mismatch` warning diagnostic.
  - `description` (required): Concise summary injected into `<skills>` in the system prompt when `{SkillsPrompt}` or `{DefaultSystemPrompt}` is active.
- **Canonical Resource URIs**:
  - Main file: `skill://{source}.{skill}/SKILL.md`
  - Supporting files (`references/`, `assets/`, `examples/`): `skill://{source}.{skill}/{relative-path}` (always using `/` separators).
  - Read via the built-in `file_system` (`read_file`, `get_file_info`, `search_files`, `search_file_content`) tools.
- **Security Boundary**: Skills v1 treats skills strictly as read-only instruction text; Everywhere never automatically executes scripts or grants tool permissions requested inside a `SKILL.md`.

---

## 2. Strategy Engine & `.strategy.md` Format

A Strategy file (`.strategy.md`) defines a single contextual action using YAML frontmatter (`schema: everywhere.strategy/v1`) and a Markdown prompt body.

### 2.1 Top-Level Frontmatter Schema
```yaml
---
schema: everywhere.strategy/v1
id: user.file-manager.summarize-selection   # Stable ID (cannot start with 'builtin.')
from: skill://everywhere.my-skill           # Optional single inheritance source (skill, strategy, markdown, url)
enabled: true                               # Default: true
name: "Summarize selected files"            # Required (after 'from' resolution)
description: "Summarize files selected in the file manager"
icon: FileText                              # Lucide icon name
priority: 80                                # Higher appears earlier (default: 0)

when:                                       # Recommendation condition (default: true)
  all:
    - extra.file_manager.selection.items:
        count:
          min: 1
    - extra.file_manager.selection.items:
        any:
          extension:
            in: [".pdf", ".docx", ".txt", ".md"]

tools:                                      # ToolPatternRulesets (plugin glob -> function glob -> bool)
  builtin.file_system:
    read_file: true
  builtin.web:
    "*": false

preprocessors:                              # Predefined execution-time variable collectors
  - file-manager-selection

systemPrompt: |                             # Optional system prompt override (supports {variables})
  You are a concise document analyst.

options:                                    # Optional evaluation timeout budgets ('ms' or 's')
  matchingTimeout: 300ms
  conditionTimeout: 80ms
  regexTimeout: 50ms
  visualQueryTimeout: 120ms
  extraTimeout: 200ms
---

Please summarize the selected files:

{extra.file_manager.selection.items}
```

### 2.2 `from` Inheritance Rules
- `from` can reference `./SKILL.md`, `skill://{source}.{skill}`, another `.strategy.md` path, or a markdown file (`kind: auto | skill | strategy | markdown | url`).
- Single inheritance only (nested `from` is rejected in v1).
- Current frontmatter fields override source fields; current Markdown body replaces source body if present (otherwise source body is inherited).

### 2.3 `when` Condition DSL Reference
Conditions evaluate to three-valued logic (`bool?`: `true`, `false`, `null` on timeout/unavailable context). Only root `true` recommends the Strategy.

#### Composite & Array Operators
- Composite: `all: [...]`, `any: [...]`, `none: [...]`
- String operators (case-insensitive by default unless `caseSensitive: true`): `equals`, `in`, `contains`, `startsWith`, `endsWith`, `regex`, `glob`
- Numeric / Length / Count operators: `min`, `max`, `equals` (under `length:`, `count:`, or numeric fields)
- Collection item operators: `any: { ... }`, `all: { ... }`, `none: { ... }` (paths inside are relative to each item)

#### Context Paths
| Root Namespace | Supported Paths |
| :--- | :--- |
| **`attachments`** | `attachments.files` (`[].path`, `[].mimeType`, `[].extension`), `attachments.selection.text`, `attachments.text`, `attachments.visual.items` |
| **`clipboard`** | `clipboard.text`, `clipboard.hasText`, `clipboard.hasImage`, `clipboard.files` |
| **`assistant`** | `assistant.id`, `assistant.name`, `assistant.model.id`, `assistant.model.modalities`, `assistant.model.supportsToolCall` |
| **`environment`** | `environment.os` (`"windows"`, `"macos"`, `"linux"`), `environment.architecture`, `environment.locale`, `environment.timeZone`, `environment.currentDate` |
| **`extra`** | **File Manager**: `extra.file_manager.current_folder.path`, `extra.file_manager.current_folder.displayName`, `extra.file_manager.selection.items` (`[].path`, `[].displayName`, `[].kind`, `[].extension`, `[].size`)<br>**Browser**: `extra.browser.active_tab.url`, `extra.browser.active_tab.title`, `extra.browser.active_tab.domain`<br>**Workspace**: `extra.workspace.root`, `extra.workspace.name`, `extra.workspace.git.branch`, `extra.workspace.git.hasChanges` |
| **`visual`** | `visual.exists: { query: "..." }`, `visual.count: { query: "...", min: 1, max: 5 }`, `visual.match: { query: "//TopLevel/@name", contains: "Visual Studio" }` |

### 2.4 Visual Query DSL Syntax (`visual.*`)
An XPath-like subset evaluated against attached visual elements or top-level roots:
- **Steps**: `//Type` (descendant), `/Type` (child), `.` (current attachment context node), `*` (any type).
- **Attributes**: `@name`, `@text` (read on demand), `@process`, `@selected`, `@focused`, `@disabled`, `@readonly`, `@offscreen`, `@password`, `@bounds.x`, `@bounds.y`, `@bounds.width`, `@bounds.height`.
- **Predicates**: `[@focused=true]`, `[contains(@name,'Save')]`, `[matches(@text,'error|warning')]`.
- *Not supported in v1*: XPath axes, namespaces, positional indices (`//Button[3]`), or platform-specific `AutomationId`/`ClassName`.
