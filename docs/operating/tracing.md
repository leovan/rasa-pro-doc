# 追踪

## 追踪 {#tracing}

分布式追踪追踪请求在分布式系统（在本例中为 Rasa 对话机器人）中流动的过程，将有关请求的数据发送到追踪后端，该后端收集所有追踪数据并允许对其进行检查。追踪数据可帮助你了解请求在单个服务（Rasa 本身）的组件以及不同分布式服务（例如你的动作服务器）中的流动情况。

### 支持的追踪后端/收集器 {#supported-tracing-backendscollectors}

要在 Rasa Pro 中追踪请求，你可以使用 [Jaeger](https://www.jaegertracing.io/) 作为后端，也可以使用 [OTEL 收集器（OpenTelemetry 收集器）](https://opentelemetry.io/docs/collector/)来收集追踪，然后将其发送到你选择的后端。有关说明，请参阅[配置追踪后端或收集器](#configuring-a-tracing-backend-or-collector)。

### Rasa 频道 {#rasa-channels}

使用 [W3C 追踪上下文规范](https://www.w3.org/TR/trace-context/)通过 REST 频道随请求一起发送的追踪上下文用于继续在 Rasa Pro 中进行追踪。

#### Rasa 检查器 {#rasa-inspector}

如果你已在 Rasa Pro 中启用追踪并使用 [Rasa 检查器](../production/inspect-assistant.md)调试工具来尝试你的对话机器人，请注意，除了 `Agent.handle_message` 方法调用的预期追踪跨度之外，追踪后端还将收集 `MessageProcessor.get_tracker` 方法调用的独立追踪跨度。这是预期的行为，因为 Rasa 检查器工具使用 Rasa [HTTP API 端点](../pages/http-api.md)来检索检查器接口所需的对话追踪器。

### 动作服务器 {#action-server}

使用 [W3C 追踪上下文规范](https://www.w3.org/TR/trace-context/)将来自 Rasa Pro 的追踪上下文与请求一起发送到自定义动作服务器，然后用于继续通过自定义动作服务器追踪请求。

通过检测接收自定义动作的 webhook，在动作服务器中继续追踪。请参阅[动作服务器属性](#action-executor-attributes)，了解作为追踪上下文的一部分捕获的属性。

请参阅[追踪事件](#traced-events)，了解 Rasa Pro 中哪些属性可作为追踪上下文的一部分使用的详细信息。

## 追踪可以帮助回答的问题 {#questions-tracing-can-help-answer}

追踪可以通过回答以下问题来帮助解决开发和生产中的问题：

- 如何在不同的组件（即对话理解组件（NLU、`CommandGenerator`、`CommandProcessorComponent`）、策略和动作服务器）之间处理用户消息请求？
- 为什么 Rasa 对话机器人决定执行某个动作？
- 为什么 Rasa 对话机器人响应缓慢？
- 为什么自定义动作执行缓慢？
- OpenAI 提示令牌使用情况如何？
- Rasa 对话机器人在不同流中的性能如何？
- Rasa 对话机器人在不同 LLM 模型中的性能如何？
- Rasa 对话机器人在不同向量存储中的性能如何？

## 配置追踪后端或收集器 {#configuring-a-tracing-backend-or-collector}

要配置追踪后端或收集器，请将 `tracing` 条目添加到你的端点，即在 `endpoints.yml` 文件中，或部署中 Helm 值的相关部分。

### Jaeger {#jaeger}

要配置 Jaeger 追踪后端，请将 `type` 指定为 `jaeger`。

```yaml
tracing:
  type: jaeger
  host: localhost
  port: 6831
  service_name: rasa
  sync_export: ~
```

!!! tip "提示"

    如果你遇到错误“OSError: [Errno 40] Message too long”，请阅读[此处](../installation/troubleshooting.md#oserror-errno-40-message-too-long)的说明以解决问题。

### OTEL 收集器 {#otel-collector}

收集器是以与供应商无关的方式收集追踪信息然后将其转发到各种后端的组件。例如，OpenTelemetry Collector (OTEL) 可以从多个不同的组件和仪表库收集追踪信息，然后将其导出到多个不同的后端，例如 jaeger。

要配置 OTEL 收集器，请将 `type` 指定为 `otlp`。

```yaml
tracing:
  type: otlp
  endpoint: my-otlp-host:4318
  insecure: false
  service_name: rasa
  root_certificates: ./tests/unit/tracing/fixtures/ca.pem
```

## 追踪事件 {#traced-events}

可追踪的 Rasa 服务区域涵盖以下动作所需的动作：

- 训练模型（即每个[图组件](https://rasa.leovan.tech/custom-graph-components#graph-components)的训练）
- 处理消息

### 模型训练 {#model-training}

通过检测 Rasa [`GraphTrainer`](https://rasa.com/docs/rasa/reference/rasa/engine/training/graph_trainer/#graphtrainer-objects) 和 [`GraphNode`](https://rasa.com/docs/rasa/reference/rasa/engine/graph/#graphnode-objects) 类，可以启用模型训练的追踪。

#### `GraphTrainer` 属性 {#graphtrainer-attributes}

在 `GraphTrainer` 训练期间可以检查以下属性：

- 模型配置的 `training_type`：
    - `"NLU"`
    - `"CORE"`
    - `"BOTH"`
    - `"END-TO-END"`
- 模型配置的 `language`
- `config.yml` 文件中使用的 `recipe_name`
- `output_filename`：保存打包模型的位置
- `is_finetuning`：布尔参数，如果为 `True`，则启用增量训练

#### `GraphNode` 属性 {#graphnode-attributes}

在每个图形节点的训练（以及消息处理期间的预测）期间捕获以下属性：

- `node_name`
- `component_class`
- `fn_name`：被调用的组件类的方法

### 消息处理 {#message-handling}

以下 Rasa 类被设计用于在消息处理期间启用追踪：

- [`Agent`](https://rasa.com/docs/rasa/reference/rasa/core/agent/#agent-objects)
- [`MessageProcessor`](https://rasa.com/docs/rasa/reference/rasa/core/processor/#messageprocessor-objects)
- [`TrackerStore`](../production/tracker-stores.md)
- [`LockStore`](../production/lock-stores.md)
- [`SingleStepLLMCommandGenerator`](#singlestepllmcommandgenerator-attributes)
- [`MultiStepLLMCommandGenerator`](#multistepllmcommandgenerator-attributes)
- [`NLUCommandAdapter`](#nlucommandadapter-attributes)
- [`FlowPolicy`](../concepts/policies/flow-policy.md)
- [`IntentlessPolicy`](#intentlesspolicy-attributes)
- [`EnterpriseSearchPolicy`](#enterprisesearchpolicy-attributes)
- [`InformationRetrieval`](#informationretrieval-attributes)
- [`EndpointConfig`](#endpointconfig-attributes)

此外，以下 Python 模块被设计用于在消息处理期间启用追踪：

- 命令处理器模块，即 `CommandProcessorComponent` 利用的工具函数，用于预处理[预测命令](../concepts/dialogue-understanding.md#command-reference)。
- 流执行器模块，即 `FlowPolicy` 利用的工具函数，用于推进流。

也就是说，这些动作现在是可追踪的：

- 接收消息
- 解析消息
- 预测[命令](../concepts/dialogue-understanding.md#command-reference)
- [预处理命令](#command-processor-module-attributes)
- 预测下一个动作
- 运行动作
- 推进[流](#flow-executor-module-attributes)
- 在[向量存储](../concepts/policies/enterprise-search-policy.md#vector-store)中搜索文档以进行企业搜索
- 通过策略（例如 [`IntentlessPolicy`](../concepts/policies/intentless-policy.md) 和 [`EnterpriseSearchPolicy`](../concepts/policies/enterprise-search-policy.md)）生成 LLM 答案
- 追踪[提示 Token 使用情况](#tracing-prompt-token-usage)
- 检索和保存追踪器
- 锁定对话
- 发布到事件代理
- 向动作服务器或 NLG 服务器发出请求
- 将追踪上下文传递给动作服务器

#### 追踪提示 Token 使用情况 {#tracing-prompt-token-usage}

!!! info "3.8 版本新特性"

    从 `3.8.0` 版本开始可以追踪 OpenAI 模型的提示 Token 使用情况。

如果你使用的是 OpenAI 模型，则以下类可以追踪提示 Token 的使用情况：

- `SingleStepLLMCommandGenerator` 类
- `MultiStepLLMCommandGenerator` 类
- `IntentlessPolicy` 类
- `EnterpriseSearchPolicy` 类
- `ContextualResponseRephraser` 类

提示 Token 的使用情况作为追踪上下文的一部分被捕获，可用于检查 LLM 答案生成过程中提示 Token 的使用情况。仅当上述其中一个工具类配置为启用捕获提示 Token 的长度时，才会捕获此信息。例如，可以通过在 `config.yml` 文件中将 `trace_prompt_tokens` 属性设置为 `true`，将 `SingleStepLLMCommandGenerator` 配置为追踪提示 Token 的长度：

```yaml
pipeline:
  - name: SingleStepLLMCommandGenerator
    trace_prompt_tokens: true
```

强烈建议仅在开发中启用提示 Token 追踪，而不是在生产中启用，因为它可能会增加对话机器人响应延迟。

#### `Agent` 属性 {#agent-attributes}

追踪处理消息的 `Agent` 实例会捕获以下属性：

- `input_channel`：频道连接器的名称
- `sender_id`：对话 ID
- `model_id`：模型的唯一标识符
- `model_name`：模型名称

#### `MessageProcessor` 属性 {#messageprocessor-attributes}

在追踪过程中会提取以下 `MessageProcessor` 属性：

- `number_of_events`：追踪器中的事件数
- `action_name`：预测和执行的动作的名称
- `sender_id`：`DialogueStateTracker` 对象的对话 ID
- `message_id`：唯一的消息 ID

后三个属性也会注入到追踪上下文中，并传递给对自定义动作服务器的请求。

#### `TrackerStore` 和 `LockStore` 属性 {#trackerstore--lockstore-attributes}

可观察的 `TrackerStore` 和 `LockStore` 属性包括：

- `number_of_streamed_events`：要流式传输的新事件数量
- `broker_class`：发布新事件的 `EventBroker`
- `lock_store_class`：用于在主动处理消息时锁定对话的锁定存储的名称

#### `SingleStepLLMCommandGenerator` 属性 {#singlestepllmcommandgenerator-attributes}

!!! info "3.9 版本新特性"

    从 `3.9.0` 版本开始可以追踪描述的 `SingleStepLLMCommandGenerator` 属性。

以下属性作为 `SingleStepLLMCommandGenerator` 的追踪上下文的一部分被捕获：

- `class_name`：被检测组件类的名称
- `llm_model`：所用 LLM 的名称
- `llm_type`：所用 LLM 的类型
- `embeddings`：所用嵌入
- `llm_temperature`：用于生成 LLM 答案的温度
- `request_timeout`：LLM 请求的超时
- `llm_engine`：用于生成 LLM 答案的引擎
- `len_prompt_tokens`：提示的 Token 长度（可选，仅支持 OpenAI 模型）。要启用此属性，请参阅[追踪提示 Token 使用](#tracing-prompt-token-usage)部分中的说明。


#### `MultiStepLLMCommandGenerator` 属性 {#multistepllmcommandgenerator-attributes}

!!! info "3.9 版本新特性"

    从 `3.9.0` 版本开始可以追踪描述的 `MultiStepLLMCommandGenerator` 属性。

以下属性作为 `MultiStepLLMCommandGenerator` 的追踪上下文的一部分被捕获：

- `class_name`：被检测组件类的名称
- `llm_model`：所用 LLM 的名称
- `llm_type`：所用 LLM 的类型
- `embeddings`：所用嵌入
- `llm_temperature`：用于生成 LLM 答案的温度
- `request_timeout`：LLM 请求的超时
- `llm_engine`：用于生成 LLM 答案的引擎
- `len_prompt_tokens`：提示的 Token 长度（可选，仅支持 OpenAI 模型）。要启用此属性，请参阅[追踪提示 Token 使用](#tracing-prompt-token-usage)部分中的说明。

#### `NLUCommandAdapter` 属性 {#nlucommandadapter-attributes}

!!! info "3.8 版本新特性"

    从 `3.8.0` 版本开始可以追踪描述的 `NLUCommandAdapter` 属性。

以下属性作为 `NLUCommandAdapter` 追踪上下文的一部分被捕获：

- `commands`：预测的命令
- `intent`：`NLUCommandAdapter` 作为输入接收的用户消息的预测意图

#### 命令处理器模块属性 {#command-processor-module-attributes}

!!! info "3.8 版本新特性"

    从 `3.8.0` 版本开始可以追踪所描述的命令处理器模块属性。

以下属性是作为命令处理器模块函数的追踪上下文的一部分捕获的：

- `execute_commands` 函数：
    - `number_of_events`：追踪器中的事件数
    - `sender_id`：`DialogueStateTracker `对象的对话 ID
- `validate_state_of_commands` 函数：
    - `cleaned_up_commands`：已清理命令的列表
- `clean_up_commands` 函数：
    - `commands`：来自 LLM 答案的原始解析命令的列表
    - `current_context`：对话堆栈的当前上下文
- `remove_duplicated_set_slots` 函数：
    - `resulting_events`：在删除重复的设置槽事件之前的事件列表，请注意，槽值已被删除以防止 PII 泄漏

#### 流执行器模块属性 {#flow-executor-module-attributes}

!!! info "3.8 版本新特性"

    从 `3.8.0` 版本开始可以追踪所描述的流执行器模块属性。

以下属性被捕获为流执行器模块函数的追踪上下文的一部分：

- `advance_flow` 函数：
    - `available_actions`：可用动作列表
    - `current_context`：对话堆栈的当前上下文
- `advance_flows_until_next_action` 函数：
    - `action_name`：要执行的动作的名称
    - `score`：已执行动作的分数
    - `metadata`：预测元数据
    - `events`：事件名称列表（如果可用）
- `run_step` 函数：
    - `step_custom_id`：步骤的自定义 ID（如果可用）
    - `step_description`：步骤的描述（如果可用）
    - `current_flow_id`：当前流的 ID
    - `current_context`：对话堆栈的当前上下文

#### `Policy` 子类属性 {#policy-subclasses-attributes}

!!! info "3.8 版本新特性"

    从 `3.8.0` 版本开始可以追踪所描述的 `Policy` 子类的属性。

以下属性被捕获为 `Policy` 接口子类（例如 `FlowPolicy`、`IntentlessPolicy`、`EnterpriseSearchPolicy`）的追踪上下文的一部分：

- `priority`：进行预测的策略的优先级
- `events`：事件名称列表，无论该策略是否胜过其他策略，都会应用这些事件
- `optional_events`：可选事件名称列表（如果可用），否则为 `None` 如果该策略胜过其他策略，则应用这些事件
- `is_end_to_end_prediction`：一个布尔值，表示预测是否使用了用户消息的文本而不是意图
- `is_no_user_prediction`：一个布尔值，表示预测是否既不使用用户消息的文本也不使用意图
- `diagnostic_data`：中间结果或其他信息，这些信息对于 Rasa 运行不是必需的，但用于调试和微调目的
- `action_metadata`：策略可以传递的其他元数据

#### `IntentlessPolicy` 属性 {#intentlesspolicy-attributes}

!!! info "3.8 版本新特性"

    从 `3.8.0` 版本开始可以追踪描述的 `IntentlessPolicy` 属性。

根据所检测的策略方法，以下属性将作为 `IntentlessPolicy` 的追踪上下文的一部分被捕获：

- `current_context`：顶部对话堆栈框架的上下文，由 `IntentlessPolicy.find_closest_response` 方法作为输入接收
- `ai_response_examples`：适合当前对话的样本响应，由 `IntentlessPolicy.select_response_examples` 方法返回
- `conversation_samples`：由 `IntentlessPolicy.select_few_shot_conversations` 方法返回的对话样本
- `ai_responses`：由 `IntentlessPolicy.extract_ai_responses` 方法从对话样本中提取的 AI 响应
- `llm_response`：由 LLM 模型调用生成的响应，由 `IntentlessPolicy.generate_answer` 方法返回
- `action_name`：要执行的动作的名称，由 `IntentlessPolicy._prediction_result` 作为输入接收方法
- `score`：执行动作的分数，由 `IntentlessPolicy._prediction_result` 方法作为输入接收

此外，`IntentlessPolicy._generate_llm_answer` 捕获与 [`SingleStepLLMCommandGenerator` 类](#singlestepllmcommandgenerator-attributes)相同的属性。

#### `EnterpriseSearchPolicy` 属性 {#enterprisesearchpolicy-attributes}

!!! info "3.8 版本新特性"

    从 `3.8.0` 版本开始可以追踪描述的 `EnterpriseSearchPolicy` 属性。

`EnterpriseSearchPolicy._generate_llm_answer` 方法捕获与 [`SingleStepLLMCommandGenerator` 类](#singlestepllmcommandgenerator-attributes)相同的属性。

#### `InformationRetrieval` 属性 {#informationretrieval-attributes}

!!! info "3.8 版本新特性"

    从 `3.8.0` 版本开始可以追踪所描述的 `InformationRetrieval` 子类的属性。

以下属性作为 `InformationRetrieval` 子类（例如 `Milvus_Store`、`Qdrant_Store`）的追踪上下文的一部分被捕获：

- `query`：用于搜索向量存储的查询
- `document_metadata`：从向量存储中检索到的文档的元数据

#### `EndpointConfig` 属性 {#endpointconfig-attributes}

!!! info "3.8 版本新特性"

    从 `3.8.0` 版本开始可以追踪描述的 `EndpointConfig` 属性。

以下属性作为 `EndpointConfig` 的追踪上下文的一部分被捕获：

- `url`：端点的 URL
- `request_body_size_in_bytes`：请求体的大小（以字节为单位）

## 在动作服务器中追踪 {#tracing-in-the-action-server}

通过检测接收自定义动作和执行自定义动作所涉及的其他类的 webhook，可以追踪 API 请求在流经动作服务器时的情况。

!!! info "3.8 版本新特性"

    现在已配备附加类别来改善动作服务器中的追踪。

以下类已进行检测：

- [`ValidationAction`](../action-server/validation-action.md#validationaction-class)：自定义动作的基类，用于提取和验证可在表单上下文之外设置或更新的槽。
- [`FormValidationAction`](../action-server/validation-action.md#formvalidationaction-class)：自定义动作的基类，用于提取和验证仅在表单上下文中设置的槽。
- [`ActionExecutor`](../operating/tracing.md#action-executor-attributes)：执行自定义动作的类。

### Webhook 属性 {#webhook-attributes}

以下属性作为接收自定义动作的 webhook 的追踪上下文的一部分被捕获；

- `http.method`：用于发出请求的 http 方法
- `http.route`：请求的端点
- `next_action`：要执行的下一个动作的名称
- `version`：使用的 rasa 版本
- `sender_id`：对话的 ID
- `message_id`：唯一的消息 ID

### 动作执行器属性 {#action-executor-attributes}

以下属性作为动作执行器的追踪上下文的一部分被捕获；

- `action_name`：要执行的动作的名称
- `sender_id`：对话的 ID
- `events`：返回事件的列表
- `slots`：由执行的自定义动作填充的槽列表
- `utters`：执行的话语列表

### 槽验证动作属性 {#slot-validation-action-attributes}

以下属性作为槽验证动作的追踪上下文的一部分被捕获；

- `class_name`：被检测的组件类的名称
- `action_name`：要执行的动作的名称
- `sender_id`：对话的 ID
- `events`：返回事件的列表
- `slots`：由执行的自定义动作填充的槽列表
- `utters`：执行的话语列表
- `message_count`：消息数量
- `slots_to_validate`：最近要验证的填充槽列表

### 调试自定义动作性能 {#debugging-custom-actions-performance}

!!! info "3.8 版本新特性"

    你现在可以沿着自定义动作代码继续追踪请求。

现在可以通过追踪自定义动作代码的特定部分来调试自定义动作的性能。这可以通过创建跨度来追踪这些部分的执行来实现。

为了创建更多跨度，你可以从 `ActionExecutorTracerRegister` 组件中检索[追踪器对象](https://opentelemetry.io/docs/languages/python/instrumentation/#acquire-tracer)。

```python
# import the ActionExecutorTracerRegister component
from rasa_sdk.tracing.tracer_register import ActionExecutorTracerRegister
```

要创建 [OTEL 文档](https://opentelemetry.io/docs/languages/python/instrumentation/#creating-spans)中所述的跨度，对应于自定义动作代码特定部分的追踪，你可以在自定义动作的运行方法中嵌入以下代码片段：

```python
# retrieve the tracer object
tracer = ActionExecutorTracerRegister().get_tracer()

# create a span
with tracer.start_as_current_span("span_name") as span:
  # your code here
  span.set_attribute("attribute_name", "attribute_value")
```

例如，实现自定义跨度的完整的自定义动作如下所示：

```python
import requests
import json
from rasa_sdk import Action
from rasa_sdk.tracing.tracer_register import ActionExecutorTracerRegister


class ActionCheckSufficientFunds(Action):
  def name(self):
    return "action_check_sufficient_funds"

  def run(
    self,
    dispatcher: CollectingDispatcher,
    tracker: Tracker,
    domain: Dict[Text, Any]
  ) -> List[Dict[Text, Any]]:
    tracer = ActionExecutorTracerRegister().get_tracer()

    with tracer.start_as_current_span("span_name"):
      balance = 1000 # hardcoded balance from tutorial purposes
      transfer_amount = tracker.get_slot("amount")
      has_sufficient_funds = transfer_amount <= balance

      # set trace attributes
      span.set_attribute("has_sufficient_funds", has_sufficient_funds)

      return [SlotSet("has_sufficient_funds", has_sufficient_funds)]
```

在动作服务器中启用和禁用追踪也以[如下](#enabling--disabling)所述的方式完成。动作服务器也支持[上面](#supported-tracing-backendscollectors)列出的相同追踪后端/收集器。有关进一步说明，请参阅[配置追踪后端或收集器](#configuring-a-tracing-backend-or-collector)。

### 启用/禁用 {#enabling--disabling}

通过[配置受支持的追踪后端](#configuring-a-tracing-backend-or-collector)，Rasa Pro 中会自动启用追踪。无需进一步动作即可启用追踪。

你可以通过将端点文件中的 `tracing:` 配置键留空来禁用追踪。
