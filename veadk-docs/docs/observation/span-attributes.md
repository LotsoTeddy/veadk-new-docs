---
title: 埋点字段说明
---

VeADK 的 Span 属性命名和值规范遵循 [OpenTelemetry]() 社区 以及 CozeLoop、APMPlus 等产品要求。VeADK 中的 Span 属性主要分为三类：

- 通用类：将会在所有类型的 Span 中存在
- LLM 类：将会在 `call_llm` Span 中存在
- Tool 类：将会在 `tool_call` Span 中存在

## 通用类

| 序号 | 埋点字段名                                 | 含义                               | 注释                                                                 |
| -- | ------------------------------------- | -------------------------------- | ------------------------------------------------------------------ |
| 1  | `gen_ai.system`                       | 生成式 AI 系统提供商名称，用于识别模型提供方。        | 从 `kwargs["model_provider"]` 提取；若无则为 `"<unknown_model_provider>"`。 |
| 2  | `gen_ai.system.version`               | VeADK 框架版本号，用于版本追踪与兼容性分析。        | 固定返回 `veadk.version.VERSION`。                                      |
| 3  | `gen_ai.agent.name`                   | 智能体（Agent）名称，用于区分不同代理实例。         | 从 `kwargs["agent_name"]` 提取；若无则为 `"<unknown_agent_name>"`。         |
| 4  | `openinference.instrumentation.veadk` | VeADK 的 OpenInference 标准化埋点版本标识。 | 固定返回 `veadk.version.VERSION`。                                      |
| 5  | `gen_ai.app.name`                     | 应用名称，用于按应用组织和过滤遥测数据。             | 从 `kwargs["app_name"]` 提取；若无则为 `"<unknown_app_name>"`。             |
| 6  | `gen_ai.user.id`                      | 用户标识，用于区分用户会话及进行用户级分析。           | 从 `kwargs["user_id"]` 提取；若无则为 `"<unknown_user_id>"`。               |
| 7  | `gen_ai.session.id`                   | 会话标识，用于按会话组织遥测数据。                | 从 `kwargs["session_id"]` 提取；若无则为 `"<unknown_session_id>"`。         |
| 8  | `agent_name`                          | 代理名称（CozeLoop 平台专用字段）。           | 同 `gen_ai.agent.name`，用于 CozeLoop 平台。                              |
| 9  | `agent.name`                          | 代理名称（TLS 平台字段）。                  | 同 `gen_ai.agent.name`，用于 TLS 兼容。                                   |
| 10 | `app_name`                            | 应用名称（CozeLoop 平台专用字段）。           | 同 `gen_ai.app.name`，用于 CozeLoop 平台。                                |
| 11 | `app.name`                            | 应用名称（TLS 平台字段）。                  | 同 `gen_ai.app.name`，用于 TLS 兼容。                                     |
| 12 | `user.id`                             | 用户标识（CozeLoop / TLS 通用字段）。       | 同 `gen_ai.user.id`。                                                |
| 13 | `session.id`                          | 会话标识（CozeLoop / TLS 通用字段）。       | 同 `gen_ai.session.id`。                                             |
| 14 | `cozeloop.report.source`              | CozeLoop 报告来源标识。                 | 固定返回 `"veadk"`，表示来自 VeADK 框架。                                      |
| 15 | `cozeloop.call_type`                  | CozeLoop 调用类型。                   | 从 `kwargs["call_type"]` 提取；若无则为 `None`。                            |

## LLM 类

| 序号 | 埋点字段名                                      | 含义                           | 注释                                                                    |
| -- | ------------------------------------------ | ---------------------------- | --------------------------------------------------------------------- |
| 1  | `gen_ai.request.model`                     | 请求的语言模型名称，用于识别调用的具体模型。       | 从 `params.llm_request.model` 获取；无值则为 `"<unknown_model_name>"`。        |
| 2  | `gen_ai.request.type`                      | LLM 请求类型，标识交互方式。             | 固定返回 `"chat"`，表示对话式交互。                                                |
| 3  | `gen_ai.request.max_tokens`                | 响应最大生成 token 数。              | 从 `params.llm_request.config.max_output_tokens` 获取，用于控制输出长度与成本。       |
| 4  | `gen_ai.request.temperature`               | 采样温度参数，控制生成随机性。              | 从 `params.llm_request.config.temperature` 获取。                         |
| 5  | `gen_ai.request.top_p`                     | nucleus sampling 的 Top-p 参数。 | 从 `params.llm_request.config.top_p` 获取。                               |
| 6  | `gen_ai.request.functions`                 | 请求中定义的函数/工具元数据。              | 提取每个工具的名称、描述及参数定义（CozeLoop 必需）。                                       |
| 7  | `gen_ai.response.model`                    | 实际响应使用的模型名称。                 | 与请求模型一致时表示正常返回。                                                       |
| 8  | `gen_ai.response.stop_reason`              | 响应生成停止原因。                    | 当前返回占位符 `"<no_stop_reason_provided>"`，待后续实现。                          |
| 9  | `gen_ai.response.finish_reason`            | 响应完成原因。                      | 当前返回占位符 `"<no_finish_reason_provided>"`，用于区分自然结束/截断等情况。               |
| 10 | `gen_ai.is_streaming`                      | 是否为流式响应。                     | 暂未实现，返回 `None`。                                                       |
| 11 | `gen_ai.operation.name`                    | 操作名称。                        | 固定返回 `"chat"`，用于统一标识操作类型。                                             |
| 12 | `gen_ai.span.kind`                         | Span 类型。                     | 固定返回 `"llm"`，符合 OpenTelemetry 语义约定。                                   |
| 13 | `gen_ai.prompt`                            | 请求输入内容结构化信息。                 | 按消息顺序记录角色、内容、函数调用、图片等输入。                                              |
| 14 | `gen_ai.completion`                        | 模型响应内容结构化信息。                 | 记录模型生成的文本、函数调用等输出内容。                                                  |
| 15 | `gen_ai.usage.input_tokens`                | 输入 token 数量。                 | 从 `params.llm_response.usage_metadata.prompt_token_count` 提取。         |
| 16 | `gen_ai.usage.output_tokens`               | 输出 token 数量。                 | 从 `params.llm_response.usage_metadata.candidates_token_count` 提取。     |
| 17 | `gen_ai.usage.total_tokens`                | 总 token 数量（输入 + 输出）。         | 从 `params.llm_response.usage_metadata.total_token_count` 提取。          |
| 18 | `gen_ai.usage.cache_creation_input_tokens` | 缓存创建所用 token 数量。             | 从 `params.llm_response.usage_metadata.cached_content_token_count` 提取。 |
| 19 | `gen_ai.usage.cache_read_input_tokens`     | 缓存读取所用 token 数量。             | 从 `params.llm_response.usage_metadata.cached_content_token_count` 提取。 |
| 20 | `gen_ai.messages`                          | 完整对话消息事件。                    | 包括系统指令、用户消息、工具响应和助手回复的结构化事件序列。                                        |
| 21 | `gen_ai.choice`                            | 模型选择事件。                      | 表示模型生成的候选响应（含函数调用或文本内容）。                                              |
| 22 | *(调试字段：未启用)* `input.value`                 | 完整 LLM 请求体。                  | 供调试使用，序列化输出请求对象。                                                      |
| 23 | *(调试字段：未启用)* `output.value`                | 完整 LLM 响应体。                  | 供调试使用，序列化输出响应对象。                                                      |

## Tool 类

| 序号 | 埋点字段名                   | 含义                   | 注释                                                                     |
| -- | ----------------------- | -------------------- | ---------------------------------------------------------------------- |
| 1  | `gen_ai.operation.name` | 操作名称，用于标识工具执行操作类型。   | 固定返回 `"execute_tool"`，统一标识工具调用操作。                                      |
| 2  | `gen_ai.tool.name`      | 工具名称。                | 从 `params.tool.name` 获取；若无则为 `"<unknown_tool_name>"`。*(TLS 平台必需字段)*    |
| 3  | `gen_ai.tool.input`     | 工具输入内容。              | JSON 序列化包含：`name`、`description`、`parameters`，用于记录工具调用参数。*(TLS 平台必需字段)* |
| 4  | `gen_ai.tool.output`    | 工具输出内容。              | JSON 序列化包含：`id`、`name`、`response`，记录工具执行结果。*(TLS 平台必需字段)*              |
| 5  | `cozeloop.input`        | 工具输入（CozeLoop 平台字段）。 | 同 `gen_ai.tool.input`，为 CozeLoop 平台提供专用格式。                             |
| 6  | `cozeloop.output`       | 工具输出（CozeLoop 平台字段）。 | 同 `gen_ai.tool.output`，为 CozeLoop 平台提供专用格式。                            |
| 7  | `gen_ai.span.kind`      | Span 类型。             | 固定返回 `"tool"`，遵循 OpenTelemetry 语义约定。*(APMPlus 平台必需字段)*                 |
| 8  | `gen_ai.input`          | 工具输入（APMPlus 平台字段）。  | 同 `gen_ai.tool.input`，用于 APMPlus 平台埋点。                                 |
| 9  | `gen_ai.output`         | 工具输出（APMPlus 平台字段）。  | 同 `gen_ai.tool.output`，用于 APMPlus 平台埋点。                                |
