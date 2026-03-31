# 02-Runtime：运行时主线

## 目录范围
聚焦会话执行的时间流：runLoop、processor、LLM 流桥接、重试、状态、调度闸门。

## 章节索引
- [2.1_The_Loop.md](./2.1_The_Loop.md): runLoop 总览
- [2.2_Processor.md](./2.2_Processor.md): 流事件处理总览
- [2.3_LLM_Bridge_And_Provider_Shims.md](./2.3_LLM_Bridge_And_Provider_Shims.md): LLM 桥接层
- [2.4_Retry_Scheduler_And_Backoff_Policy.md](./2.4_Retry_Scheduler_And_Backoff_Policy.md): 重试与退避
- [2.5_Session_Service_Event_Sourcing.md](./2.5_Session_Service_Event_Sourcing.md): Session 服务与事件写模型
- [2.6_Status_Channel_And_Retry_Surface.md](./2.6_Status_Channel_And_Retry_Surface.md): 状态通道与重试外显
- [2.7_Todo_Sidecar_Channel.md](./2.7_Todo_Sidecar_Channel.md): Todo 侧通道
- [2.8_Processor_Abort_Cleanup_And_Stop_Gates.md](./2.8_Processor_Abort_Cleanup_And_Stop_Gates.md): 中断与清理闸门
- [2.9_RunLoop_Break_And_Continuation_Gates.md](./2.9_RunLoop_Break_And_Continuation_Gates.md): break/continue 判决面
- [2.10_Runner_SingleFlight_And_Busy_Gate.md](./2.10_Runner_SingleFlight_And_Busy_Gate.md): Session 单飞与忙闲互斥
- [2.11_LLM_Stream_Queue_Bridge_And_Cancellation.md](./2.11_LLM_Stream_Queue_Bridge_And_Cancellation.md): AsyncIterable -> Effect Stream
- [2.12_AutoTitle_FirstTurn_Generator.md](./2.12_AutoTitle_FirstTurn_Generator.md): 首轮自动标题
- [2.13_ProviderTransform_Normalization_And_Option_Routing.md](./2.13_ProviderTransform_Normalization_And_Option_Routing.md): provider 兼容变换层

## 建议顺序
`2.1 -> 2.2 -> 2.8 -> 2.9 -> 2.10 -> 2.3 -> 2.11 -> 2.4 -> 2.6`
