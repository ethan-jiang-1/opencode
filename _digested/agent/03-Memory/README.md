# 03-Memory：记忆与上下文控制

## 目录范围
聚焦上下文预算、历史压缩、输出截断、回放恢复和一致性保护。

## 章节索引
- [3.1_Compaction.md](./3.1_Compaction.md): Compaction 主线（触发/执行/续跑）
- [3.2_Prune_And_Truncate.md](./3.2_Prune_And_Truncate.md): 历史裁剪与截断策略
- [3.3_MessageV2_Model_Adapter.md](./3.3_MessageV2_Model_Adapter.md): 消息到模型适配
- [3.4_SQLite_Projection_And_Eventual_Consistency.md](./3.4_SQLite_Projection_And_Eventual_Consistency.md): 投影与最终一致性
- [3.5_Revert_Time_Travel_And_Diff.md](./3.5_Revert_Time_Travel_And_Diff.md): 回滚与差异
- [3.6_Overflow_Budget_Reservation.md](./3.6_Overflow_Budget_Reservation.md): overflow 预算判决
- [3.7_SessionSummary_Snapshot_Diff_Aggregator.md](./3.7_SessionSummary_Snapshot_Diff_Aggregator.md): 快照差异聚合
- [3.8_FileTime_ReadStamp_And_Optimistic_Concurrency.md](./3.8_FileTime_ReadStamp_And_Optimistic_Concurrency.md): read-then-write 一致性
- [3.9_Truncate_Output_Spool_And_Retention.md](./3.9_Truncate_Output_Spool_And_Retention.md): 大输出落盘与保留
- [3.10_Compaction_Overflow_Replay_And_Media_Strip.md](./3.10_Compaction_Overflow_Replay_And_Media_Strip.md): overflow 回放与媒体剥离
- [3.11_Compaction_Agent_DeepDive.md](./3.11_Compaction_Agent_DeepDive.md): Compaction 附录（提示词与时序细节）

## 收敛状态
- 第二轮已完成：
  - `3.1` 收敛为主文。
  - `3.11` 降为附录，去除重复叙事。
