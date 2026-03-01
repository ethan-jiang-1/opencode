# OpenCode _config 目录

本目录用于存放 OpenCode 的配置参考文档和备份。

## 文件说明

| 文件 | 说明 |
|------|------|
| [model-configuration-guide.md](./model-configuration-guide.md) | 模型配置完整指南，包含添加新模型的方法、模板和故障排查 |
| [current-global-config.json](./current-global-config.json) | 当前全局配置 `~/.config/opencode/opencode.json` 的备份 |

## 实际配置文件位置

- **全局配置**: `~/.config/opencode/opencode.json`
- **认证存储**: `~/.local/share/opencode/auth.json`
- **数据库**: `~/.local/share/opencode/opencode.db`
- **日志**: `~/.local/share/opencode/log/`

> ⚠️ 修改配置请直接编辑 `~/.config/opencode/opencode.json`，本目录仅供参考。
