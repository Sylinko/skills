# Sylinko Agent Skills

[前往中文版本 »](./README-zh-cn.md)

Official [Agent Skills](https://agentskills.io) repository maintained by [Sylinko](https://github.com/Sylinko) for **[Everywhere](https://github.com/Sylinko/Everywhere)** — the context-aware desktop AI assistant ("Every moment, Every place. Your AI - Everywhere").

These skills follow the open `SKILL.md` specification and work across **Everywhere's built-in Skills engine** as well as external AI coding agents (Claude Code, Google Antigravity, Cursor, Codex, GitHub Copilot, Windsurf, Cline, etc.).

---

## 📦 Available Skills

| Skill | Path | Description |
| :--- | :--- | :--- |
| **`everywhere`** | [`skills/everywhere`](./skills/everywhere/SKILL.md) | Comprehensive knowledge base for Everywhere: product usage, settings, shortcuts, assistants, model providers, Prompt Manager, Strategy Engine (`.strategy.md`), Skills v1, built-in tools/MCP, troubleshooting, and full C#/.NET codebase architecture. |

*(Coming soon: `everywhere-mod` for authoring and packaging Everywhere mods and plugins.)*

---

## 🚀 Quick Installation

### 1. Install via `npx skills` CLI (Recommended)

Install all skills or a specific skill into your local agent environment (`.agents/skills/`, `~/.agents/skills/`, Claude Code, Cursor, etc.):

```bash
# Install interactively from the repository
npx skills add Sylinko/skills

# Or install the 'everywhere' skill directly
npx skills add Sylinko/skills --skill everywhere
```

### 2. Use directly inside Everywhere

Everywhere natively indexes skills from `~/.everywhere/skills` and `~/.agents/skills` (both enabled by default), as well as `~/.claude/skills`, `~/.codex/skills`, `~/.copilot/skills`, `~/.cursor/skills`, and `~/.gemini/skills`.

To install into Everywhere's managed skill directory manually:

```bash
git clone https://github.com/Sylinko/skills.git
cp -r skills/skills/everywhere ~/.everywhere/skills/everywhere
```

Once installed, verify or toggle the skill in **Everywhere → Skills** page.

---

## 🔗 Related Links

- **Everywhere Repository**: [https://github.com/Sylinko/Everywhere](https://github.com/Sylinko/Everywhere)
- **Official Website & Documentation**: [https://everywhere.sylinko.com](https://everywhere.sylinko.com)
- **Discord Community**: [https://discord.gg/5fyg6nE3yn](https://discord.gg/5fyg6nE3yn)

## 📄 License

Released under the [Apache License 2.0](./LICENSE).
