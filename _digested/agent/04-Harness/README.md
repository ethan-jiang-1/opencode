# 04-Harness：扩展与集成层

## 目录范围
聚焦从外部注入能力到内部工具执行的全链路：Hook、Skill、MCP、ACP、Tool、Command、Plugin、Model Catalog。

## 章节索引（按主题）

### A. 基础总览
- [4.1_Interceptor_Hooks.md](./4.1_Interceptor_Hooks.md)
- [4.2_Skill.md](./4.2_Skill.md)
- [4.3_MCP.md](./4.3_MCP.md)
- [4.4_ACP.md](./4.4_ACP.md)
- [4.5_A2A_Subagent.md](./4.5_A2A_Subagent.md)
- [4.6_The_Bash_Tool.md](./4.6_The_Bash_Tool.md): Bash e2e 主链路
- [4.7_Custom_CLI_Injection.md](./4.7_Custom_CLI_Injection.md): 自定义 CLI 接入主线

### B. 插件与协议
- [4.8_Plugin_Hook_Runtime.md](./4.8_Plugin_Hook_Runtime.md)
- [4.9_MCP_Multi_Transport_And_OAuth.md](./4.9_MCP_Multi_Transport_And_OAuth.md)
- [4.10_ACP_Bidirectional_Translator.md](./4.10_ACP_Bidirectional_Translator.md)
- [4.14_InstructionPrompt_Claim_And_Inheritance.md](./4.14_InstructionPrompt_Claim_And_Inheritance.md)
- [4.15_SystemPrompt_Model_Routing_And_Skill_Gating.md](./4.15_SystemPrompt_Model_Routing_And_Skill_Gating.md)
- [4.16_MCP_Auth_Store_And_URL_Binding.md](./4.16_MCP_Auth_Store_And_URL_Binding.md)
- [4.17_MCP_OAuth_Callback_Server_And_CSRF.md](./4.17_MCP_OAuth_Callback_Server_And_CSRF.md)
- [4.18_ACP_Session_Manager_State_Model.md](./4.18_ACP_Session_Manager_State_Model.md)
- [4.19_Plugin_Loader_Resolve_And_Compatibility.md](./4.19_Plugin_Loader_Resolve_And_Compatibility.md)
- [4.20_Plugin_Config_Patcher_JSONC_Workflow.md](./4.20_Plugin_Config_Patcher_JSONC_Workflow.md)
- [4.21_Plugin_Meta_Fingerprint_And_Theme_Tracking.md](./4.21_Plugin_Meta_Fingerprint_And_Theme_Tracking.md)
- [4.32_ModelsDev_Catalog_Fallback_And_HotRefresh.md](./4.32_ModelsDev_Catalog_Fallback_And_HotRefresh.md)

### C. 工具执行内核
- [4.11_Edit_Tool_Robust_Replacement_Engine.md](./4.11_Edit_Tool_Robust_Replacement_Engine.md)
- [4.12_ApplyPatch_Safe_Pipeline.md](./4.12_ApplyPatch_Safe_Pipeline.md)
- [4.13_Task_Tool_Subagent_Sandboxing.md](./4.13_Task_Tool_Subagent_Sandboxing.md)
- [4.22_ReadTool_Guarded_Slicing_And_Multimodal_Attachment.md](./4.22_ReadTool_Guarded_Slicing_And_Multimodal_Attachment.md)
- [4.23_WriteTool_Diff_Gated_Write_And_Diagnostics_Fanout.md](./4.23_WriteTool_Diff_Gated_Write_And_Diagnostics_Fanout.md)
- [4.24_BashTool_AST_Path_Resolver_And_Permission_Synthesis.md](./4.24_BashTool_AST_Path_Resolver_And_Permission_Synthesis.md): Bash 机制深潜
- [4.29_SessionPrompt_ResolveTools_Context_Bridge.md](./4.29_SessionPrompt_ResolveTools_Context_Bridge.md): 工具桥接内核
- [4.30_BashArity_Command_Intention_Compressor.md](./4.30_BashArity_Command_Intention_Compressor.md)

### D. 输入编译与命令通道
- [4.25_SessionPrompt_ResolvePromptParts_Markdown_Expander.md](./4.25_SessionPrompt_ResolvePromptParts_Markdown_Expander.md)
- [4.26_SessionPrompt_UserPart_File_MCP_Transcoder.md](./4.26_SessionPrompt_UserPart_File_MCP_Transcoder.md)
- [4.27_StructuredOutput_Tool_Enforcement_Lane.md](./4.27_StructuredOutput_Tool_Enforcement_Lane.md)
- [4.28_Command_Template_Compiler_And_Subtask_Lowering.md](./4.28_Command_Template_Compiler_And_Subtask_Lowering.md)
- [4.31_SessionPrompt_ShellImpl_UserCommand_Replay.md](./4.31_SessionPrompt_ShellImpl_UserCommand_Replay.md)

## 收敛状态
- 第二轮已完成：
  - `4.6` 收敛为 e2e 主链路文。
  - `4.24` 保留为 Bash 机制深潜文。
  - `4.7` 收敛为外部 CLI 接入主线。
  - `4.29` 保留为内部桥接实现文。
