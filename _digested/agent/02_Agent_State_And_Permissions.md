# Agent 状态机与权限边界 (Agent State & Permissions)

在 OpenCode 中，Agent 并非指代一个单纯的 “系统提示词 (System Prompt)” 集合。系统架构的精妙之处在于：**一个 Agent 实际上是一套权限边界 (Permission Ruleset)、角色属性 (Role Info) 和工具过滤器的总和**。

它的核心实现在 `packages/opencode/src/agent/agent.ts`。

## 1. Agent.Info 结构解构

每一个被系统承认的 Agent，其核心骨架是由 `Agent.Info` 这个 Zod Schema 强定义的，它至少包含：
- **`mode`**: `primary`（主模型，如 build、plan）、`subagent`（被降级孵化的子模型，如 general、explore）、`all`（均可适用）。
- **`permission`**: 这是一个由 `Permission.Ruleset` 构建的矩阵，这也是 **Agent 的真正灵魂**。
- **`model`**: 指定或覆写的关联大模型。
- **`temperature` / `topP`**: 模型基础超参。
- **`hidden`**: 一些系统级使用的“隐身” Agent，比如用来压缩对话的 `compaction` Agent 和用来起标题的 `title` Agent。

## 2. 系统内置的 Agent 角色详解

OpenCode 开箱即用地在代码库里写死了几种关键 Agent，通过观察它们的 `Permission`，我们可以深刻理解权限系统是如何驾驭大模型的：

### 🗡 `build` Agent (主开发)
这是权限最高的默认执行者。
- **权限边界**: `plan_enter` 允许，`question` 允许。它同时继承了系统级的 `defaults`，这意味着它能随意读取、修改当前工作区（Workspace）的内容。
- **定位**: 脏活累活执行者，能改文件，能跑 Bash 命令。

### 👁 `plan` Agent (架构师/推演者)
这是“多脑分离”理念的实践。很多人在用 AI 工具时，模型经常擅自改代码导致大面积崩溃。`plan` Agent 的出现就是用来解决这个问题的。
- **权限边界**: `edit: { "*": "deny" }`。除了 `.opencode/plans/` 目录外的所有代码文件，一律禁止写入。
- **定位**: 纯粹的代码研究者。当我们需要它梳理屎山代码的调用链时，使用 `plan` 完全不需要担心它把原工程破坏。

### 🔍 `explore` Agent (专注视角的探索者)
- **权限边界**: `* : deny` (屏蔽所有其它能力), 但特批开放了 `grep`, `glob`, `list`, `bash`, `webfetch`。
- **定位**: 一个纯粹的“猎犬”。它是一个 `subagent`，被主 Agent 召唤出来专门负责跨目录找文件。

## 3. 为什么通过 Permission 驾驭 Agent 是最高级的 Harness？

传统的 Harness 喜欢在 Prompt 里加上规则：“**如果你遇到了死循环，请立刻停下**”。但大语言模型在长期执行（Loop）中必然产生顺从性退化，它大概率会无视这条语言指令。

在 OpenCode 中，开发者采用了**物理拦截 (Physical Interception)** 机制：
1. **Tool 层面的判定**: `session/processor.ts` 在收到 LLM 返回的 Tool Call 后，会首先检查 `Permission.evaluate(toolName, ...)`。
2. **Doom Loop 拦截**: 如果 AI 工具触发了死循环匹配逻辑（如系统里的 `doom_loop` 配置），权限引擎会立刻生效（比如设定为 `doom_loop: "ask"`）。此时无论大模型怎么狂热地要在 JSON 里生成 `tool-calls`，框架都在运行时直接掐断它的调用，并将其阻塞，等待人 (Human) 来做批准（Ask）。

**这就是 Agent 框架设计的铁律：永远不要用不确定的自然语言规则，去防范必须要杜绝的系统灾难。用确定性的 Permission Ruleset 来进行沙箱物理隔离。**
