# 当前可用的模型服务商清单

> 截至 2026-03-01，`~/.config/opencode/opencode.json` + 环境变量的完整服务商列表。

## 服务商总览

| # | Provider ID | 类型 | 认证方式 | 配置位置 | 默认？ |
|---|------------|------|---------|---------|-------|
| ① | `anthropic` | 内置 + 代理 | `opencode.json` 中 apiKey | JSON 显式配置 | ❌ |
| ② | `glm5` | 自定义私有 | `opencode.json` 中 apiKey | JSON 显式配置 | ✅ |
| ③ | `google` | 内置（自动检测） | `GEMINI_API_KEY` 环境变量 | `.zshrc` 环境变量 | ❌ |

---

## ① Anthropic（通过代理）

- **Provider ID**: `anthropic`（内置供应商，不需要 `npm` 和 `models`）
- **代理地址**: `https://us4.ctok.ai/api/v1`
- **认证**: 代理 token `cr_xxx...`
- **模型**: 从 models.dev 自动获取全部 Claude 系列模型
- **使用方式**: `/models` → 选择 Anthropic 下的任意模型

## ② GLM-5 Private（自定义）

- **Provider ID**: `glm5`（自定义，需要 `npm` + `models`）
- **SDK**: `@ai-sdk/openai-compatible`
- **API**: `https://ds-api.yovole.com/v1`
- **模型**: `glm-5`（200K context / 64K output）
- **当前默认**: `"model": "glm5/glm-5"`

## ③ Google Gemini（已固化在配置中）

- **Provider ID**: `google`（内置供应商）
- **SDK**: `@ai-sdk/google`
- **认证**: `opencode.json` 中显式指定的 `apiKey`（为了方便跨机器拷贝，不再依赖 `.zshrc` 环境变量）
- **无需 models 配置**: OpenCode 会自动从 models.dev 获取 Gemini 模型列表
- **模型**: 包括：
  - `gemini-2.5-flash` / `gemini-2.5-pro`
  - `gemini-3.1-pro-preview-*`
  - 等等
- **使用方式**: `/models` → 选择 Google 下的任意模型

## 切换默认模型

修改 `opencode.json` 中的 `"model"` 字段：

```json
"model": "glm5/glm-5"                              // 当前：GLM-5
"model": "anthropic/claude-sonnet-4-20250514"        // 切换到 Claude Sonnet 4
"model": "google/gemini-2.5-flash"                   // 切换到 Gemini 2.5 Flash
```

或在 TUI 中使用 `/models` 命令交互式切换。
