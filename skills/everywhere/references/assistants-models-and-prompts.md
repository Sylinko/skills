# Everywhere Assistants, Model Providers & Prompt Manager

## 1. Custom Assistants vs. System Assistants

Everywhere separates conversational personas (**Custom Assistants**) from background utility tasks (**System Assistants**).

### Custom Assistants (`Settings.Model.CustomAssistants`)
Users can create and switch between multiple Custom Assistants (`CustomAssistant`). Each assistant configures:
- **Identity**: `Id` (GUID v7), `Name`, `Description`, and `Icon` (Lucide icon or colored icon).
- **System Prompt Reference** (`SystemPromptId`): References a prompt managed in **Prompt Manager** (`Guid.Empty` uses the built-in default system prompt; non-empty GUIDs reference rows in `prompt.db`).
- **Model Configuration**: Provider, endpoint, API key reference, model ID, temperature/topP, max tokens, and **Reasoning Effort** (`ReasoningEffortValues` such as `none`, `minimal`, `low`, `medium`, `high`, `xhigh` when supported by the model).
- **Tool Call Policy**:
  - `IsToolCallEnabled` (bool): Master toggle for tool calling on this assistant (when `Configuration.SupportsToolCall` is true).
  - `ToolEnablementRulesets`: Optional per-assistant overrides for which plugins/functions are enabled (`null` inherits global plugin settings).
- **Context & Visual Limits** (under *Advanced Settings*):
  - `VisualContextLengthLimit` (default `Balanced`): Controls the token budget allocated when capturing/building visual tree snapshots for this assistant.
  - `ContextCompressionThreshold` (`5`–`95`%, default `80`%): Percentage of the model's declared context limit at which automatic conversation compaction triggers.

### System Assistants (`Settings.SystemAssistant`)
Configured under **Settings → System Assistant** (`SystemAssistantSettings`). Each specialization defaults to `AutoSelect = true` (which automatically selects a suitable model or reuses the active generation's connection) or can be pinned to a specific model configuration:

| Specialization | Property | Purpose |
| :--- | :--- | :--- |
| **Title Generation** | `TitleGeneration` | Automatically generates a concise title after the first turn of a chat (prefers fast nano/mini models when `AutoSelect` is on). |
| **Tool Approval** | `ToolApproval` | Powers the private `ToolApprovalReviewer` agent when Tool Approval Mode is set to `Auto`. Optional experimental toggle: `AllowApprovalFileReads` lets the reviewer call `read_file` during safety review. |
| **Default Subagent** | `DefaultSubagent` | Model used when the main assistant invokes `run_subagent` with `specialization: "default"` (or omitted). |
| **Image Understanding** | `ImageUnderstanding` | Multimodal vision model used when `run_subagent` is invoked with `specialization: "image-understanding"` or when analyzing visual attachments. |

---

## 2. Model Providers, Schemas & Catalog

Everywhere supports both preset providers (`PresetModelTemplates`, synchronized with the **Models.dev** catalog cache) and custom API endpoints:

### Supported Provider Schemas (`ModelProviderSchema`)
1. **`OpenAIResponses`** (`/v1/responses`):
   - Default for **OpenAI** (`gpt-5.5`, `gpt-5.4`, `gpt-5.4-mini`, `gpt-5.4-nano`, `gpt-5.2`, `gpt-5.1`, `gpt-5`, `o3`, `o4-mini`, etc.).
   - Implemented via `OpenAIResponsesKernelMixin` with full reasoning summary and streaming tool-call support.
2. **`OpenAI`** (Chat Completions `/v1/chat/completions`):
   - Used for OpenAI-compatible providers including **DeepSeek**, **Moonshot (Kimi)**, **MiniMax**, **SiliconFlow**, **OpenRouter**, **302.AI**, and custom endpoints.
   - Includes `OpenAICompatibleToolSchemaTransformer` to normalize JSON schemas for strict/compatible providers.
3. **`Anthropic`**:
   - Native Claude SDK integration (`AnthropicKernelMixin`) supporting Claude 4 / 3.7 / 3.5 Sonnet, Haiku, and Opus with extended thinking/reasoning and prompt caching.
4. **`Google`**:
   - Native Gemini connector (`GoogleKernelMixin`) supporting Gemini 2.5 / 3.x Pro, Flash, and Flash-Lite.
5. **`Mistral`**:
   - Native Mistral connector (`MistralKernelMixin`).
6. **`Ollama`**:
   - Local deployment via `OllamaKernelMixin` (`OllamaSharp`), supporting local tool-calling and vision models.
7. **Everywhere Official Cloud (`OfficialConnector`)**:
   - Integrated AI Gateway when signed into an Everywhere Cloud account, providing managed models and official web search without separate API keys.

### API Key Management (`Settings.Model.ApiKeys`)
- API keys are stored separately from assistant definitions and referenced by ID so multiple assistants can share one key without duplicating credentials.

---

## 3. Prompt Manager & Template Placeholders

**Prompt Manager** elevates system prompts to first-class resources stored in `<WritableFolderPath>/db/prompt.db` (`PromptDbContext`).

### Core Rules & Workflows
- **Built-in Default Prompt** (`Guid.Empty`): Virtual, pinned at the top of the Prompt Manager list, and always up to date with Everywhere's latest capabilities. It cannot be edited or deleted directly (copy it to customize).
- **Guided Creation vs. Advanced Editing**:
  - Beginners can create prompts through guided recipe choices (persona, scenario, tone, output style).
  - Power users can switch to the **Advanced Template Editor** at any time with live rendered preview (`PromptTemplateRenderer`) and real-time static diagnostics (`PromptTemplateAnalyzer`).
- **Reverse Reference Protection**: Deleting a prompt checks if any `CustomAssistant` references its GUID; upon confirmation, affected assistants automatically fall back to `Guid.Empty` (the built-in default prompt).
- **Global Prompt Picker**: Users can summon the Prompt Picker to paste **rendered** prompt content (not raw unexpanded placeholders) directly into external applications.

### System Prompt Placeholders (`SystemPromptPlaceholderSource`)
In Prompt Manager templates, placeholders use single curly braces `{PlaceholderName}` (case-sensitive):

| Placeholder | Resolved Value | Notes & Diagnostics |
| :--- | :--- | :--- |
| `{DefaultSystemPrompt}` | Full built-in Everywhere system prompt | Includes `{SkillsPrompt}` and `{Date}` internally. |
| `{SkillsPrompt}` | XML `<skills>...</skills>` index of enabled Skills | **Warning diagnostic (`MissingSkillsPrompt`)**: Emitted if a custom prompt omits both `{DefaultSystemPrompt}` and `{SkillsPrompt}`, because the assistant will not see installed skills! |
| `{Date}` | Current localized long date (`DateTime.Now.ToString("D")`) | Recommended over `{Time}` to preserve LLM KV-cache / prompt cache hit rates across turns. |
| `{Time}` | Current date and time (`DateTime.Now.ToString("F")`) | **Warning diagnostic (`TimeMayReduceCacheHitRate`)**: Using `{Time}` without `{Date}` changes the system prompt every second/minute and invalidates provider prompt caching. |
| `{OS}` | `Environment.OSVersion.ToString()` | Current operating system version string. |
| `{SystemLanguage}` | English name of current UI locale | e.g., `Simplified Chinese`, `English`, `Japanese`. |
| `{WorkingDirectory}` | Current chat working directory | Falls back to `RuntimeConstants.WritableFolderPath` if unset. |
