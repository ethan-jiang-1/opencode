# OpenCode 模型配置指南

> 本文档基于 OpenCode 源码及官方文档整理。
> **核心设计理念：可移植性 (Portability)**。所有配置方法均旨在实现"单文件跨机器拷贝即可用"，告别繁琐的环境变量配置。

---

## 目录

1. [配置文件位置与优先级](#1-配置文件位置与优先级)
2. [当前已配置的模型](#2-当前已配置的模型)
3. [配置结构详解](#3-配置结构详解)
4. [添加新模型的三种方式](#4-添加新模型的三种方式)
5. [已知供应商内置 SDK 列表](#5-已知供应商内置-sdk-列表)
6. [常见模型配置模板](#6-常见模型配置模板)
7. [认证管理](#7-认证管理)
8. [故障排查](#8-故障排查)
9. [参考来源](#9-参考来源)

---

## 1. 配置文件位置与优先级

OpenCode 支持多级配置，优先级从高到低：

| 优先级 | 位置 | 说明 |
|--------|------|------|
| 🔴 最高 | 项目目录 `.opencode/opencode.json` | 项目级覆盖 |
| 🟡 | 项目根目录 `opencode.json` | 项目配置 |
| 🟢 | `~/.config/opencode/opencode.json` | **全局配置**（你的当前配置） |
| 🔵 最低 | 系统管理目录 | 企业部署用 |

> [!TIP]
> 全局配置适合放 API Key 和常用模型；项目配置适合放项目专用的覆盖项。

---

## 2. 当前已配置的模型

你的 `~/.config/opencode/opencode.json` 当前配置：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "glm5/glm-5",
  "provider": {
    "anthropic": {
      "options": {
        "baseURL": "https://us4.ctok.ai/api/v1",
        "apiKey": "cr_***"
      }
    },
    "glm5": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "GLM-5 Private",
      "options": {
        "baseURL": "https://ds-api.yovole.com/v1",
        "apiKey": "sk-***"
      },
      "models": {
        "glm-5": {
          "name": "GLM-5",
          "limit": { "context": 200000, "output": 64000 }
        }
      }
    }
  }
}
```

- **默认模型**: `glm5/glm-5`（格式: `<provider-id>/<model-id>`）
- **anthropic**: 通过代理 `us4.ctok.ai` 接入，但未定义具体模型（会从 models.dev 自动拉取）
- **glm5**: 自定义 provider，使用 OpenAI 兼容协议

---

## 3. 配置结构详解

### 顶层字段

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "<provider-id>/<model-id>",       // 默认模型
  "small_model": "<provider-id>/<model-id>",  // 用于标题生成等轻量任务
  "provider": { ... },                        // Provider 配置（核心）
  "disabled_providers": ["provider-id"],       // 禁用指定 provider
  "enabled_providers": ["provider-id"],        // 仅启用指定 provider（白名单模式）
  "agent": { ... },                           // Agent 配置
  "mcp": { ... }                              // MCP Server 配置
}
```

### Provider 配置结构

```json
{
  "provider": {
    "<provider-id>": {
      "npm": "<ai-sdk-package>",      // 必填（自定义 provider）
      "name": "显示名称",               // 可选，UI 中显示的名字
      "options": {
        "baseURL": "https://...",      // API 端点
        "apiKey": "sk-xxx",            // API Key（也可用 /connect 存入 auth.json）
        "headers": { "key": "value" }, // 自定义请求头
        "timeout": 300000              // 超时(ms)，默认 5 分钟；设 false 禁用
      },
      "models": {
        "<model-id>": {
          "name": "模型显示名",
          "limit": {
            "context": 200000,         // 上下文窗口大小（token 数）
            "output": 65536            // 最大输出 token 数
          }
        }
      },
      "whitelist": ["model-1"],        // 仅显示这些模型
      "blacklist": ["model-2"]         // 隐藏这些模型
    }
  }
}
```

### apiKey 环境变量与文件引用语法

配置文件支持两种动态引用（在 `config/paths.ts` 中实现）：

```json
// 引用环境变量（如果变量不存在，替换为空字符串）
"apiKey": "{env:MY_API_KEY}"

// 引用文件内容（支持绝对路径、相对路径和 ~/）
"apiKey": "{file:~/.secrets/my-api-key.txt}"
"apiKey": "{file:/absolute/path/to/key}"
"apiKey": "{file:./relative/to/config-dir/key}"
```

> [!TIP]
> 配置文件支持 **JSONC 格式**（`.jsonc` 扩展名），可以写注释！OpenCode 会同时查找 `opencode.jsonc` 和 `opencode.json`，JSONC 优先。

### npm 包 fallback 链

当 OpenCode 加载模型时，按以下顺序确定使用哪个 AI SDK 包（源码 `provider.ts` L834-838）：

```
model.provider.npm  →  provider.npm  →  已有模型的 npm  →  models.dev 中的 npm  →  "@ai-sdk/openai-compatible"（最终 fallback）
```

即：**如果你什么都不配，默认会用 `@ai-sdk/openai-compatible`**。

### model ID 别名（api.id）

配置中的 model key 和实际发送给 API 的 model ID 可以不同：

```json
"models": {
  "my-friendly-name": {
    "id": "actual-api-model-id",
    "name": "Display Name"
  }
}
```

- `"my-friendly-name"` — 在 OpenCode 内部用作标识（也用于 `"model": "provider/my-friendly-name"`）
- `"id": "actual-api-model-id"` — 实际发送给 LLM API 的 model ID
- 如果不设 `id`，则 key 本身就是 API model ID

---

## 4. 添加新模型的三种方式

### 方式一：内置供应商 + API Key 固化（最便携，推荐）

对于 Anthropic、OpenAI、Google 等内置供应商，如果你希望**跨机器拷贝配置直接使用**，最简单的方法是直接在 `opencode.json` 中配置它们的 `apiKey`：

```jsonc
{
  "provider": {
    "google": {
      // 只要指定 apiKey，其它都不用配！模型列表会自动加载
      "options": {
        "apiKey": "AIzaSyBhOWYa..."
      }
    }
  }
}
```

> [!TIP]
> 虽然内置供应商也支持通过环境变量（如 `GEMINI_API_KEY`）配置，但**直接将 key 固化在 JSON 中是最利于多设备跨环境迁移的方式**，一台机器配好，文件一拷就完事了。

也可以在 TUI 中执行 `/connect` 交互式输入，但这会存到 `~/.local/share/opencode/auth.json` 中，需要拷两个文件。

### 方式二：内置供应商 + 自定义 baseURL（代理场景）

当你需要通过代理调用时，只需覆盖 `baseURL`：

```json
{
  "provider": {
    "anthropic": {
      "options": {
        "baseURL": "https://your-proxy.com/v1",
        "apiKey": "your-key"
      }
    }
  }
}
```

### 方式三：完全自定义 Provider（任何 OpenAI 兼容 API）

```jsonc
{
  "provider": {
    "my-provider": {
      // npm 包名（可省略，默认为 @ai-sdk/openai-compatible）
      "npm": "@ai-sdk/openai-compatible",
      "name": "My Provider Display Name",
      "options": {
        "baseURL": "https://api.example.com/v1",
        // apiKey 可直接写，也可引用环境变量/文件
        "apiKey": "{env:MY_PROVIDER_KEY}",
        // 自定义请求头（可选）
        "headers": { "X-Custom": "value" },
        // 超时设置（可选，默认 5 分钟，设 false 禁用）
        "timeout": 300000
      },
      "models": {
        "model-id": {
          "name": "Model Display Name",
          // id: 如果实际 API 的 model id 与 key 不同，在这里指定
          // "id": "actual-model-id-sent-to-api",
          "limit": {
            "context": 128000,  // 上下文窗口
            "output": 32768     // 最大输出
          },
          // 费用追踪（可选，单位: $/百万token）
          "cost": { "input": 3, "output": 15 },
          // 能力声明（可选，影响 OpenCode 的行为）
          "reasoning": true,         // 是否支持推理/思考
          "tool_call": true,         // 是否支持 tool call（默认 true）
          "temperature": true,       // 是否支持温度调节
          "attachment": false         // 是否支持附件
        }
      },
      // 模型过滤（可选，用于内置供应商）
      "whitelist": ["model-1"],  // 仅显示这些
      "blacklist": ["model-2"]   // 隐藏这些
    }
  }
}
```

> [!IMPORTANT]
> 自定义 provider **必须手动定义 `models`**，否则 OpenCode 不知道有哪些模型可用。内置供应商的模型列表会从 [models.dev](https://models.dev) 自动获取。

### 方式四：使用本地 AI SDK 包（高级）

`npm` 也支持 `file://` 协议，可以加载本地开发的 provider 包：

```json
{
  "provider": {
    "my-local": {
      "npm": "file:///path/to/my-ai-sdk-provider/index.js",
      "name": "My Local Provider",
      "options": { "baseURL": "http://localhost:8000" },
      "models": { "local-model": { "name": "Local" } }
    }
  }
}
```

> 要求：包必须导出以 `create` 开头的函数（如 `createMyProvider`），接受 options 参数，返回 AI SDK 兼容的 Provider 对象。

---

## 5. 已知供应商内置 SDK 列表

以下供应商已内置专用 SDK，配置时可使用对应的 `npm` 包名（或无需 `npm`，自动识别）：

| Provider ID | npm 包 | 认证方式 |
|------------|--------|---------|
| `anthropic` | `@ai-sdk/anthropic` | `ANTHROPIC_API_KEY` 或 `/connect` |
| `openai` | `@ai-sdk/openai` | `OPENAI_API_KEY` 或 `/connect` |
| `google` | `@ai-sdk/google` | `GOOGLE_GENERATIVE_AI_API_KEY` 或 `/connect` |
| `google-vertex` | `@ai-sdk/google-vertex` | GCP 项目认证 |
| `amazon-bedrock` | `@ai-sdk/amazon-bedrock` | AWS 凭证链 |
| `azure` | `@ai-sdk/azure` | `AZURE_API_KEY` 或 `/connect` |
| `openrouter` | `@openrouter/ai-sdk-provider` | `OPENROUTER_API_KEY` 或 `/connect` |
| `xai` | `@ai-sdk/xai` | `XAI_API_KEY` 或 `/connect` |
| `mistral` | `@ai-sdk/mistral` | `MISTRAL_API_KEY` 或 `/connect` |
| `groq` | `@ai-sdk/groq` | `GROQ_API_KEY` 或 `/connect` |
| `deepinfra` | `@ai-sdk/deepinfra` | `DEEPINFRA_API_KEY` 或 `/connect` |
| `cerebras` | `@ai-sdk/cerebras` | `CEREBRAS_API_KEY` 或 `/connect` |
| `cohere` | `@ai-sdk/cohere` | `COHERE_API_KEY` 或 `/connect` |
| `togetherai` | `@ai-sdk/togetherai` | `TOGETHER_AI_API_KEY` 或 `/connect` |
| `perplexity` | `@ai-sdk/perplexity` | `PERPLEXITY_API_KEY` 或 `/connect` |
| `vercel` | `@ai-sdk/vercel` | `/connect` |
| `gitlab` | `@gitlab/gitlab-ai-provider` | `GITLAB_TOKEN` 或 `/connect` |
| `github-copilot` | 内置 | Copilot 认证 |
| 任何 OpenAI 兼容 | `@ai-sdk/openai-compatible` | `apiKey` 配置 |

---

## 6. 常见模型配置模板

### DeepSeek（通过 models.dev 已注册）

```bash
export DEEPSEEK_API_KEY="sk-xxx"
```

或使用 `/connect` 选择 DeepSeek。

### Ollama（本地模型）

```json
{
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": {
        "baseURL": "http://localhost:11434/v1"
      },
      "models": {
        "qwen2.5-coder:32b": {
          "name": "Qwen 2.5 Coder 32B",
          "limit": { "context": 32768, "output": 8192 }
        },
        "deepseek-coder-v2": {
          "name": "DeepSeek Coder V2",
          "limit": { "context": 128000, "output": 8192 }
        }
      }
    }
  }
}
```

### LM Studio（本地）

```json
{
  "provider": {
    "lmstudio": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LM Studio",
      "options": {
        "baseURL": "http://localhost:1234/v1"
      },
      "models": {
        "your-loaded-model": {
          "name": "Your Model",
          "limit": { "context": 32768, "output": 8192 }
        }
      }
    }
  }
}
```

### OpenRouter（聚合网关）

```bash
export OPENROUTER_API_KEY="sk-or-xxx"
```

或使用 `/connect` 选择 OpenRouter，模型列表自动获取。

### 通用代理模式（中转站）—— ⭐ 你当前 GLM-5 的配置模式

这是你最常用的场景。核心要点：
- `npm` 用 `@ai-sdk/openai-compatible`（或省略，自动 fallback 到这个）
- `baseURL` 指向中转站地址
- `apiKey` 填中转站给你的 key
- **必须手动定义每个模型**，特别是 `limit` 字段（告诉 OpenCode 上下文窗口大小）

```jsonc
{
  "provider": {
    "my-proxy": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "API Proxy",
      "options": {
        "baseURL": "https://your-proxy.com/v1",
        "apiKey": "{env:MY_PROXY_KEY}"  // 推荐用环境变量
      },
      "models": {
        "claude-sonnet-4-20250514": {
          "name": "Claude Sonnet 4 (via Proxy)",
          "limit": { "context": 200000, "output": 64000 },
          "reasoning": true,
          "tool_call": true
        },
        "gpt-4o": {
          "name": "GPT-4o (via Proxy)",
          "limit": { "context": 128000, "output": 16384 },
          "tool_call": true
        },
        "deepseek-chat": {
          "name": "DeepSeek V3 (via Proxy)",
          "limit": { "context": 65536, "output": 8192 },
          "tool_call": true
        }
      }
    }
  }
}
```

> [!WARNING]
> **`limit` 字段非常重要！** 如果不设置，默认为 0，OpenCode 不知道能发多少上下文给模型，可能导致上下文管理（compaction 等）行为异常。

### 同一个内置供应商 + 自定义代理

如果你想修改内置供应商（如 `anthropic`）的 baseURL 指向代理，但**保留** models.dev 的自动模型列表：

```json
{
  "provider": {
    "anthropic": {
      "options": {
        "baseURL": "https://your-proxy.com/v1",
        "apiKey": "your-proxy-key"
      }
    }
  }
}
```

不需要手动定义 `models`！因为 `anthropic` 是内置供应商，模型列表会从 models.dev 自动获取，你的 `options` 只是覆盖了连接参数。这就是你当前 `anthropic` 配置的模式。
```

---

## 7. 认证管理

```bash
# 查看已存储的认证
opencode auth list

# 添加新认证（交互式）
# 在 TUI 中执行 /connect

# 认证存储位置
~/.local/share/opencode/auth.json
```

> [!TIP]
> `opencode.json` 中的 `apiKey` 优先级高于 `auth.json`，两者都会被检查。

---

## 8. 故障排查

1. **模型不出现在列表中**
   - 检查 `provider.<id>.models` 是否正确定义
   - 自定义 provider 必须显式列出 models
   - 内置 provider 需正确设置 API Key

2. **认证失败**
   - `opencode auth list` 检查凭证
   - 确保 `provider-id` 与 `/connect` 中的 ID 一致

3. **使用代理但报错**
   - 确认 `baseURL` 格式正确（通常以 `/v1` 结尾）
   - 确认代理支持的 API 协议

4. **切换默认模型**
   - 修改顶层 `"model"` 字段，格式: `"<provider-id>/<model-id>"`
   - 或在 TUI 中按 `/models` 切换

---

## 9. 参考来源

本文档信息来源于以下材料：

| 来源 | 路径/URL | 内容 |
|------|---------|------|
| 配置定义 (Zod Schema) | [`packages/opencode/src/config/config.ts`](file:///Users/bowhead/opencode/packages/opencode/src/config/config.ts) | 完整的 Config.Info / Config.Provider schema 定义 |
| Provider 实现 | [`packages/opencode/src/provider/provider.ts`](file:///Users/bowhead/opencode/packages/opencode/src/provider/provider.ts) | 所有内置 provider 的 SDK 映射 + 自定义加载器 |
| 官方文档 - Providers | [opencode.ai/docs/providers](https://opencode.ai/docs/providers) | 各供应商配置方法、自定义 provider、故障排查 |
| 官方文档 - Intro | [opencode.ai/docs](https://opencode.ai/docs) | 安装、配置、初始化流程 |
| 本目录模板 | `opencode.json` | 包含了通用代理、自定义模型和 Google API 固化的最佳实践模板 |
| 认证存储 | `~/.local/share/opencode/auth.json` | API Key 持久化存储 |
| JSON Schema | [opencode.ai/config.json](https://opencode.ai/config.json) | 配置自动补全与验证 |
