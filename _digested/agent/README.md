# OpenCode Agent 架构手册

本目录进入“收拢阶段（Phase 3）”：
- 已完成：大范围扩张挖掘、章节分层、编号冲突修复、第二轮去重、第三轮模板统一。
- 当前目标：横向索引补全（按源码模块反查章节）与小范围重复裁剪。
- 下一阶段：稳定版本冻结与轻量维护。

## 阅读路径
1. 先读 [01-Anatomy](./01-Anatomy/README.md)
2. 再读 [02-Runtime](./02-Runtime/README.md)
3. 然后读 [03-Memory](./03-Memory/README.md)
4. 最后读 [04-Harness](./04-Harness/README.md)

## 章节导航
- [01-Anatomy](./01-Anatomy/README.md): 静态结构、权限、消息与工具注册
- [02-Runtime](./02-Runtime/README.md): 主循环、处理器、重试、状态与调度
- [03-Memory](./03-Memory/README.md): 压缩、溢出、截断、快照与一致性
- [04-Harness](./04-Harness/README.md): 工具层、协议层、插件层、命令与扩展

## 收敛说明
- 已修复编号冲突：
  - `01-Anatomy/1.2_Permission_Engine.md` -> `01-Anatomy/1.3_Permission_Engine.md`
  - `01-Anatomy/1.3_Tools_Registry.md` -> `01-Anatomy/1.4_Tools_Registry.md`
  - `03-Memory/3.1_Compaction_Agent.md` -> `03-Memory/3.11_Compaction_Agent_DeepDive.md`
- `03-Memory/3.1` 与 `3.11` 暂时保留（总览 vs 深潜），后续做合并裁剪。
- 第三轮已完成：`01/02/03/04` 章节正文统一为 `源码是什么 / 解读是什么 / implication 是什么` 三段模板。
