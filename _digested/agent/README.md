# OpenCode Agent 架构手册 (Handbook TOC)

这本手册 (Handbook) 是《OpenCode 架构解析》中最核心、最底层的部分。我们将用上帝视角，配合真实源码定义和时序流转图，彻底拆解一个**基于 Effect-TS 构建、依靠自治的心跳循环 (Loop) 驱动**的新一代 AI Coding Agent。

读完本手册，你将彻底理解 Agent 是如何工作的，以及如何在每一个隐蔽的生命周期里，挂载你的高阶 Harness（业务编排增强网）。

---

## 🗺 核心全局流转图 (Agent Architectural Flow)

下面这幅图描绘了 OpenCode 中，一个任务从输入到被解析、拦截、压缩和执行的全局流转拓扑。**我们在后续的章节中，将像手术刀一样剖开这里的每一个色块。**

```mermaid
graph TD
    %% 外部输入层
    USER((Human / User Input)) -->|CLI/Terminal/MCP| SESSION[Session Prompt: The Engine]

    %% 01 Anatomy 数据层
    subgraph "01-Anatomy: 核心基座数据"
        A_INFO[Agent.Info<br/>包含模型、权限(Permission)、系统提示]
        M_V2[MessageV2 Graph<br/>本地 SQLite 流式存储]
        T_REG[Tools Registry<br/>工具路由表]
    end
    
    A_INFO -.->|定义代理人的边界| SESSION
    M_V2 -.->|提供持久化短时记忆| SESSION

    %% 02 Runtime 运行时主循环
    subgraph "02-Runtime: 自治时钟 (runLoop)"
        LOOP_START{Loop<br/>while(true)}
        
        %% 03 Memory 截断与清洗
        subgraph "03-Memory: 反熵增系统"
            COMPACT[SessionCompaction<br/>通过 'hidden' Agent 生成 summary 截断记忆图]
        end
        
        %% 04 Harness 编排挂载点
        subgraph "04-Harness: 环境与挂载注入"
            HOOK_MSG[Hook: chat.messages.transform<br/>(业务上下文/RAG硬洗)]
            SKILLS[Skills & Environment<br/>动态推入 React/Python 规约]
        end

        SESSION --> LOOP_START
        LOOP_START -->|1. 触发判断| COMPACT
        COMPACT -->|2. 处理前置流转| HOOK_MSG
        HOOK_MSG -->|3. 拼装环境| SKILLS
        SKILLS -->|4. 向下分发| PROCESSOR[SessionProcessor<br/>LLM Stream 处理 / 断点拦截]
        PROCESSOR -->|5. 遇到 ToolCall| TOOL_EXEC{Tool Sandbox<br/>Zod与权限拦截}
        
        TOOL_EXEC -->|6a. 继续| LOOP_START
        TOOL_EXEC -->|6b. DoomLoop / Ask| HUMAN_IN[Human Approval / 打断]
        HUMAN_IN --> LOOP_START
        PROCESSOR -->|5. 纯文本 / Finish| EXIT((返回前端))
    end
```

---

## 📖 目录 (Table of Contents)

本手册的内容严格划分为 4 个篇章，依次递进。推荐按照此顺序阅读：

### [01-Anatomy (底层数据解剖学)](./01-Anatomy/)
拆解 Agent 世界的地基。不再抽象地聊“上下文”，而是直接看被注入的 Zod Models。
- [1.1 Agent State & Permissions](./01-Anatomy/1.1_Agent_Info.md) —— Agent 根本不是 Prompt，而是带沙箱边界的角色。
- [1.2 Message Graph](./01-Anatomy/1.2_Message_Graph.md) —— 为什么它是立体的状态转换图而不是扁平的列表。
- [1.3 Tools Registry](./01-Anatomy/1.3_Tools_Registry.md) —— 给 AI 的“四肢”进行 Zod Schema 强校验。

### [02-Runtime (调度引擎与心跳)](./02-Runtime/)
如果你想知道大模型到底在循环什么，一切尽在此处。
- [2.1 The Loop (`prompt.ts`)](./02-Runtime/2.1_The_Loop.md) —— 解剖 `while(true)` 的每一次固定流转。
- [2.2 Processor (`processor.ts`)](./02-Runtime/2.2_Processor.md) —— 底层 LLM Stream 收发处理与异常自愈机制。

### [03-Memory (记忆控制与反熵增)](./03-Memory/)
解决如何在一周的持久开发后，不让系统的 Token 爆炸。
- [3.1 Context Compaction](./03-Memory/3.1_Compaction.md) —— 极其巧妙的 "幻影 Agent" (`hidden: true`) 压缩设计。

### [04-Harness (定制落地指南与扩展实战)](./04-Harness/)
对于打算将系统私有化、集成研发管线的高级玩家，这里提供了所有的能力注入实战范式。
- [4.1 拦截器钩子实战 (Hooks)](./04-Harness/4.1_Interceptor_Hooks.md) —— 在循环间隙硬塞 RAG 向量知识。
- [4.2 静态能力注入 (Skill)](./04-Harness/4.2_Skill.md) —— 如何通过 Markdown 规范自动干预代码产出。
- [4.3 动态知识拉取 (MCP)](./04-Harness/4.3_MCP.md) —— 连入监控与数据库的按需懒加载 (Pull-based) 终极解法。
- [4.4 外部驱动协议 (ACP)](./04-Harness/4.4_ACP.md) —— 如何让 Zed 这样的外部 IDE / 客户端接管并驱动 Agent 服务端的大脑。
- [4.5 子智能体编排 (A2A/Subagent)](./04-Harness/4.5_A2A_Subagent.md) —— `subtask` 状态折叠与主从协同的代码审计流水线。
- [4.6 终端与命令行注入 (CLI/Bash)](./04-Harness/4.6_CLI_And_Bash.md) —— 底层是如何用 AST 编译器 (Tree-Sitter) 物理拦截 `rm -rf` 等危险指令的。
