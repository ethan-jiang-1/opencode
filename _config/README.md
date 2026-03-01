# OpenCode 跨机器配置模板 (_config)

本目录的核心目的是：**实现 OpenCode 配置的"开箱即用"与跨机器无缝迁移**。

只要把这份配好的 `opencode.json` 拷贝到任何一台新机器上，所有定义好的模型提供商（Anthropic、GLM、Gemini 等）都会**直接可用**，无需重新配置环境变量或执行 `/connect` 登录。

## 文件说明

| 文件 | 说明 |
|------|------|
| [model-configuration-guide.md](./model-configuration-guide.md) | 模型配置完整指南，包含添加新模型的方法、模板和故障排查 |
| [opencode.json](./opencode.json) | 配置模板，可直接拷贝作为 `~/.config/opencode/opencode.json` 使用 |

## 🚀 如何在新机器上使用

1. 在新机器上安装 OpenCode。
2. 将本目录下的 `opencode.json` 文件直接拷贝或覆盖到新机器的 `~/.config/opencode/opencode.json` 路径下。
3. 完成！所有模型已就绪，直接在终端输入 `opencode` 开始使用。

## 设计原则

为了实现 100% 的可移植性，本目录下的配置遵循以下原则：
- **API Key 必须固化在 JSON 中**：坚决避免使用 `.zshrc` 环境变量或 `auth.json`，确保 `opencode.json` 本身就是唯一的事实来源。
- **避免绝对路径**：配置中不包含任何与特定机器用户名 (`/Users/bowhead/`) 相关的硬编码绝对路径。

## 实际配置文件位置（备忘）

- **全局配置**: `~/.config/opencode/opencode.json`（⭐ **核心迁移文件**）
- **认证存储**: `~/.local/share/opencode/auth.json`（本方案中已废弃不用）
- **数据库**: `~/.local/share/opencode/opencode.db`（历史记录，视需求迁移）
- **日志**: `~/.local/share/opencode/log/`
