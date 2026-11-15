# Sylinko Agent Skills (中文说明)

[English Version »](./README.md)

由 [Sylinko](https://github.com/Sylinko) 官方维护的 **[Everywhere](https://github.com/Sylinko/Everywhere)**（具备上下文感知能力的跨平台桌面 AI 助手）官方 [Agent Skills](https://agentskills.io) 仓库。

本仓库遵循开放的 `SKILL.md` 规范，既可直接内置或安装于 **Everywhere 自身的 Skills 系统**中，也可在各类主流 AI 编程助手（Claude Code、Google Antigravity、Cursor、Codex、GitHub Copilot、Windsurf 等）中按需加载。

---

## 📦 包含的 Skills

| Skill 名称 | 路径 | 说明 |
| :--- | :--- | :--- |
| **`everywhere`** | [`skills/everywhere`](./skills/everywhere/SKILL.md) | Everywhere 完整知识库：涵盖产品使用指南、全局设置与快捷键、自定义助手与模型提供商、Prompt Manager、Strategy Engine（`.strategy.md` 编写）、Skills v1 规范、内置插件与 MCP 工具、常见排障，以及 Everywhere 完整源码架构与开发规范。 |

*（即将推出：`everywhere-mod`，专为 Everywhere Mod 与扩展开发提供 SDK、补丁与打包指南。）*

---

## 🚀 快速安装

### 1. 使用 `npx skills` CLI 安装（推荐）

通过 Vercel 的 `skills` CLI 一键安装到本地项目或全局 Agent 目录（如 `~/.agents/skills/`、`.claude/skills/` 等）：

```bash
# 交互式选择安装仓库中的 Skill
npx skills add Sylinko/skills

# 直接指定安装 everywhere Skill
npx skills add Sylinko/skills --skill everywhere
```

### 2. 在 Everywhere 客户端中使用

Everywhere 原生支持自动扫描并加载 `~/.everywhere/skills` 与 `~/.agents/skills`（默认启用），同时也支持发现 `~/.claude/skills`、`~/.codex/skills`、`~/.copilot/skills`、`~/.cursor/skills` 和 `~/.gemini/skills`。

你也可以手动将 `skills/everywhere` 复制到 `~/.everywhere/skills/everywhere`，然后在 Everywhere 的 **Skills（技能）** 页面中查看和管理启用状态。

---

## 🔗 相关链接

- **Everywhere 主仓库**: [https://github.com/Sylinko/Everywhere](https://github.com/Sylinko/Everywhere)
- **官方网站与文档**: [https://everywhere.sylinko.com](https://everywhere.sylinko.com)
- **Discord 社区**: [https://discord.gg/5fyg6nE3yn](https://discord.gg/5fyg6nE3yn)

## 📄 开源协议

本项目基于 [Apache License 2.0](./LICENSE) 开源。
