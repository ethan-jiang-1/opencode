# Processor 引擎与工具调用 (The Processor Engine)

如果说 `prompt.ts` 里的 `runLoop` 是血液系统，那么 `packages/opencode/src/session/processor.ts` 中的 `SessionProcessor` 就是 OpenCode Agent 的**前额叶 (执行引擎)**。

这一部分的代码，展示了一个成熟的 AI Coding Agent 是如何完美对接底层大模型 API（通过 `ai` SDK），安全并且严密地进行 Tool (工具) 分发的。

## 1. Handle 的概念设计

当你每次向大模型发问，或者 Loop 进入下一个迭代周期时，系统并不只是单纯地调一个异步函数。它会创建一个被 Effect 环境包裹的 `Handle` 对象（即处理器实例）。
这个 `Handle` 封装了：
- **`message`**: 这里保存着 `AssistantMsg`。
- **请求 abort 控制器**: 可以被外界中断打断操作。
- **`process()` 方法**: 核心大脑。

## 2. processor.process() 的内部流水线

当 Loop 喂给 `Processor` 全量上下文之后，`process` 方法的执行逻辑如下：

### 阶段A：Stream 拦截与映射
调用 `@ai-sdk/core` 的 `streamText`，或者由于模型不同调用的各家定制接口。Processor 屏蔽了各家大模型的底层返回差异。它通过一系列回调将流事件转化为内部严谨的数据结构：
- `onTextContent`: 捕获思考过程与纯回复，映射给前端用于打字机效果展示。
- `onToolCall`: 拦截！这并不是马上执行代码。系统开始校验该模型宣称调用的工具，它的 `JSON Arguments` 是否合法。

### 阶段B：工具沙箱化执行
在 `prompt.ts` 将所有 `tools` 根据当前 Agent 的 `Permission` 进行过滤后，它会将可用工具的字典传递给 `Processor`。
当 `Processor` 收到一个 `toolCall` 时：
1. 它通过工具注册表 (`ToolRegistry`) 实例化对应的执行逻辑。
2. **阻断判定**: 检查这个 Tool 需要什么权限。如果是 `ask` (如涉及到删除文件、外发网络请求)，则阻塞当前执行图，触发一个 `bus` 消息交由控制端/用户点击确认 (`Approval`)。
3. **Effect 隔离**: 工具的执行是在一个特定的被保护状态中进行的。一旦它向外抛出未处理异常，系统级别的 Layer 将将其捕获，并作为 `Assistant Tool Result` 退回给大语言模型，告诉大模型：“你刚执行的指令引发了系统内部异常，错误栈如下，请重试。”——这让 Agent 具备了“在报错中自我修复”的能力。

## 3. 面向结构化数据的兜底

值得特别注意的是 `Processor` 对于 `StructuredOutput` 工具的降级和劫持处理：
如果你强制要求此次交互只返回 JSON Schema（例如提取总结时），系统会在 `tools` 注入时塞入一个特制的隐性 Tool: `StructuredOutput`。大模型返回的属性必须吻合给定的 Schema，如果不吻合，或者模型死活输出了平铺文本。`Processor` 将不承认这次回答的合理性，主动抛出 `StructuredOutputError` 并触发重试反馈链。

## 4. Harness 设计启示：构建新的神经末梢

理解了 `Processor` 的底层调度，你就会明白在 OpenCode 里加一个能力，千万不要用“调取命令行输出再去解析文字”的粗糙套路。

**如果要给 Agent 挂接新的能力 (Harnessing New Capabilities)**：
你应当去 `packages/opencode/src/tool/` 创建一个新的 Tool 实例，并给它严格定义 `Zod Schema` (它的入参)，以及 `Permission Action` 类别。只要这个 Tool 被注入了底层系统，`Processor` 就会利用它无与伦比的流式解析和错误重试机制帮你把所有的异常屏蔽掉。Agent 便立刻拥有了一个全新的、甚至比原生开发者还稳定的“四肢功能”。
