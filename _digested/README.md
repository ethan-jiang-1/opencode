# OpenCode 架构解析 (The Digest)

**目录目的**:
本 `_digested` 目录及相关文件，旨在为那些对 OpenCode 内部机制感兴趣的高级开发者、架构师或研究人员提供一份**硬核、透彻的系统级解析 (System Digest)**。跳过表面的基础使用教程，我们将直接深入代码的骨架与血肉，去解构这个 "AI Coding Agent" 的核心运作原理。

我们的解析将围绕你提出的两个核心话题展开：**Overall (全局架构)** 与 **Agent (智能体核心引擎)**。理解了这两点，尤其是 Agent Loop，你就拥有了为这个系统编写高质量 Harness (驾驭/强化框架) 的上帝视角。

---

## 1. Overall：系统全局架构鸟瞰 (The "O")

如果你用传统的视角去看 OpenCode，它本质上是一个**基于 C/S (Client-Server) 模型，并用重型函数式响应框架 (Effect-TS) 驱动的本地化微服务系统**。

- **Monorepo 结构**: 系统通过 Bun + Turbo 构建，各个模块被清晰地划分在 `packages/` 下（如 `app/`、`console/`、`opencode/` 等）。
- **C/S 分离**: 它的核心引擎（也就是 `packages/opencode`）作为一个 Server 在本地运行。前端展现可以是 Terminal TUI (终端界面，类似于 `packages/console`)，也可以是 Web 页面、桌面端 (Electron)、移动端、乃至未来可能接入的任何客户端。这种解耦意味着 Agent 的大脑逻辑和 UI 是完全分离的。
- **Effect-TS 驱动**: 这是一个极其关键的技术底座。OpenCode 并没有使用传统的 `Promise`/`async-await` 来做简单的流程控制，而是高度工程化地全程使用了 **Effect**（一种用于构建稳健、可组合及可追溯的异步系统的库）。
    - 所有的资源、生命周期、错误处理都被包裹在 `Effect.Effect<...>` 中。
    - 服务通过 `Layer` 和 `ServiceMap` 注入解耦。这对扩展框架（Harness）有决定性的好处：拦截和接管底层能力（如读写文件、MCP资源访问、LSP分析）变得极其规范和防漏。

---

## 2. Agent：心脏与 Agent Loop 的解剖

Agent 是 OpenCode 的绝对核心。它不是一段简单的单次 API 调用脚本，而是一个具备 "状态机循环特性" 的自治执行流。其核心逻辑位于 `packages/opencode/src/session/prompt.ts` (Loop实现) 和 `packages/opencode/src/agent/agent.ts` (定义) 中。

### 2.1 什么是 Agent？
在系统中，Agent 被定义为一组极其精简的数据集（`Agent.Info`）。它包括：模型指向、特定的 System Prompt 指令、以及最关键的 **Permission (权限动作边界规则)**。
系统内置了 `build`（全权限主开发）、`plan`（只读探索/思考）、`explore` 等角色。角色的差异很大程度上并不只是Prompt的不同，而是权限对 Tool (工具) 可见性的裁决。

### 2.2 解剖 Agent Loop (The Heartbeat)
Agent 执行任何任务（从写一句代码到完成整个需求）的生命线都是一个位于 `prompt.ts` 内的 `runLoop`（死循环 `while (true)`）。它的“一次搏动”包含如下固定过程：

1. **历史梳理与截断 (Compaction Check)**: 每次循环起点，检查历史消息。如果感知到 Token 溢出或者遇到了被标记的 Compact 指令，将触发上下文清洗与压缩，以此保证它在长工作流中不会“失忆”或“Token爆炸”。
2. **任务分配与挂起 (Subtask Hook)**: 检测当前队列是否有子任务需要分配。如果有，将切换流转到对应的子 Agent 上。
3. **环境、技能与插件系统注入 (System Assembly)**: 一切大发神威的基础。每次请求模型前，内部通过 Effect 并发拉取：系统级的环境信息、LSP (语法库感知)、工作区状态、用户自定义的 `Skills`、甚至包括 `MCP` 外部资源。全部进行动态拼装。
4. **模型请求与工具筛选**: 大模型依据被允许的Tools列表输出决策：
   - 如果收到 `tool-calls` -> 执行对应的侧边工具（读写文件、查搜索、运行 Bash），完成后带着执行结果进入下一次循环 (**`continue`**)。
   - 如果收到纯文本/结束标志 -> 任务完成，跳出深渊循环 (**`break`**)，给用户返回结果。

---

## 3. Harness (驾驭/强化框架) 的深层思考：如何做得更合理？

正如你思考的：“明白 Agent 的 loop 之后，就知道怎么给 Agent 补足任何东西后，这个 harness 是怎么做更合理”。

对于如此重型的 Agent，外部的 Harness 不应该仅仅是在前端做 Prompt Wrapper (套壳)，而应该利用其现有的 **插件钩子和权限拦截机制**，将其置于一个可控的“安全网”和“增强网”中：

1. **利用 Permission 打造绝对的护栏 (Safeguard Harness)**:
   Agent Loop 最大的隐患是 **Doom Loop**（死循环盲点：例如反复在一行报错代码上尝试错误的方式而卡死）。合理的 Harness 应该强依赖原生的权限系统。例如利用 `doom_loop: "ask"` 或将关键模块目录设为 `edit: "deny"`。当系统感知到危险行为即将发生时，通过权限机制在 Loop 阶段直接切断执行，唤回外部的主理人或者真实用户（Human-in-the-loop）干预。

2. **利用 Plugin/Effect Layer 实施知识注入 (Capability Harness)**:
   要知道给 Agent “补充知识”，绝非在对话框里发一堆文档。因为在漫长的循环中，普通对话很快会被 Compaction 压碎流失。
   最优解是利用 OpenCode 暴露的 Hook 事件（例如 `Plugin.trigger("experimental.chat.messages.transform")` 或注入自定义 `Skill`）。让你的 Harness 在每次进入 Agent Loop 进行 "System Assembly" 阶段时，都能像“呼吸”一样把最新的业务规范、LSP 类型分析和外部知识强行打入系统 Prompt 顶端。

3. **利用多 Agent / Subtask 实施编排控制 (Workflow Harness)**:
   把所有压力丢给单一庞大 Loop 必将崩溃。更高级的 Harness 应像流水线一样设计调度策略。在 `agent.ts` 中有原生生成 Subagent（`generate`接口）和调用子任务的能力。合理的框架是做一层“主脑控制层”，把长线需求降级拆分，以 Subtask 形式丢进不同的子 Agent Loop 内，并在这个过程中插入“Review 节点”。

总结：洞悉了 OpenCode 此般架构与 Agent Loop 的循环原理，后续所有的二次开发、定制调优与大规模应用 Harness，才不会隔靴搔痒。在每一个 `Effect` 节点和每一次的 `continue/break` 之间找准位置切入，方可锻造出最契合实际业务体系的 AI 工程底座。
