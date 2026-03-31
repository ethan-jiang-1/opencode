# 上下文压缩与记忆管理 (Context Compaction)

作为常青态 (Long-running) 的开发辅助 Agent，上下文爆炸是毁灭性痛点。当你让 Agent 在一个项目里写了一天代码后，对话历史 `msgs` 的文本量极大概率突破 128k 甚至 200k Token。此时带来的灾难是多重的：
1. **推理变蠢**: 大量早期的报错与尝试堆栈会干扰模型的注意力焦点 (Loss in the middle)。
2. **API 拉闸**: 被大模型厂商强制由于超过 Context Window 限制直接抛错截断。

在 OpenCode 中，这个核心机制的实现位于 `packages/opencode/src/session/compaction.ts`（即上文 Loop 中的第二步）。它是维系生命体长久思考的“海马体”。

## 1. 何时触发 Compaction？(Triggering)

在 `prompt.ts` 的 `runLoop` 中，有两个切面会自动检测是否需要触发清洗：
- **`isOverflow` 检查**: 每次执行完，系统都会统计当前保留记录的 Tokens 消耗。如果算上内置 Prompt 达到了当前选择大模型 (比如 Claude 3.5 Sonnet 的限定) 预设的安全阈值以下（并不是等到 100% 满，而是达到一个水位线），便自动打上 `overflow: true` 的标记。
- **人工或任务切换标记**: 用户主动触发“提炼”，或是执行完一个巨大的 `subtask` 结束后要求主动结算。

## 2. 这到底是如何压缩的？（The Dark Subagent）

当你追踪到 `compaction.ts` 内部，就能看到极其精彩的系统级 Harness 是怎么设计的：

**它并不是简单的粗暴截断头部的 `msgs` 数组！**

1. **幻影特工入场**: 系统调集了一个名为 `compaction` 的内置 Agent。这个 Agent 设置了 `hidden: true`，对用户界面透明，并且拥有一个叫 `prompt/compaction.txt` 的专属 System Prompt。
2. **双重处理器启动**: 系统会基于之前留下的那些庞杂的所有 `User`/`Assistant` 的对话流建立一个新的隔离 Processor，但模型不再用它去推进业务，而是要求这个大模型去把**这一大坨历史对话，用一种结构化的摘要机制总结成一小段纯文本（Summary）**。
3. **节点阻断**: 一旦 LLM 提供完了 `Summary`，系统将把这条携带 `summary` 属性的特殊消息安插在历史图中。从此以后，`MessageV2.filterCompacted` (也就是 `loop` 中提取消息的源头) 只要回溯碰到带 `summary` 的消息节点，就会 **`break` (截断)**。之前数百回合的工具执行堆栈（Terminal Bash 日志、废弃代码的修改日志）从下一个循环开始将**永远不会再被发送给大模型**。而模型只看到了它最后留给自己的那一小束“精华摘要”。

## 3. Harness 中的记忆控制

对于系统级架构师，理解了 `Compaction` 的原理，就相当于掌握了 Agent 的失忆症解药：

如果你的 Harness 会向上下文中推入海量的动态日志（例如你外接了 CI/CD 的报错全量 Heapdump）：
1. **不要犹豫，利用 Subtask 隔离它**：让它在一个独立隔离的子循环中解析日志，由于那个任务会被立刻结算，解析产生的大几十兆日志就不会污染根层 (`Root Session`) 的关键路径。
2. **注入自定义的摘要指令**：如果你发现内置的 `compaction.txt` 在压缩你们公司的业务逻辑时经常遗失代码分支的上下文，你可以利用前文提到的 `Plugin Layer`，在 `Compaction Triggered` 的事件生命周期里去替换这个“幻影 Agent”的提示词模板。通过针对性强化，使得压缩产物更符合你的架构需求。
