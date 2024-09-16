# 启动流

本页介绍了控制流启动时间的不同方法：使用命令生成器、预测意图以及使用流守卫。

## 启动流 {#starting-flows}

流可以由以下 Rasa 组件之一触发：[SingleStepLLMCommandGenerator](components/llm-command-generators.md#singlestepllmcommandgenerator)、[MultiStepLLMCommandGenerator](components/llm-command-generators.md#multistepllmcommandgenerator) 或 [NLUCommandAdapter](components/nlu-command-adapter.md)。

`SingleStepLLMCommandGenerator` 和 `MultiStepLLMCommandGenerator` 均使用每个流的描述、正在运行的对话和其他上下文来决定何时启动流。为每个流编写清晰而独特的描述很重要，可以帮助 LLM 决定触发哪个流，或者在用户未提供足够信息时发出[澄清](dialogue-understanding.md#clarify)命令。

`NLUCommandAdapter` 使用预测意图来启动流。为了通过 `NLUCommandAdapter` 触发流，你需要为流定义 [NLU 触发器](starting-flows.md#nlu-trigger)。

[流守卫](starting-flows.md#flow-guards)是启动流前必须满足的条件。添加流守卫可以对触发流的时间进行额外的控制。

## NLU 触发器 {#nlu-trigger}

`nlu_trigger` 字段是可选的。如果存在，它包含可以启动流的意图列表。

如果不想使用置信度阈值，请列出意图名称：

```yaml title="flows.yml" hl_lines="4-5"
flows:
  my_flow:
    description: "A flow triggered with <intent-name>"
    nlu_trigger:
      - intent: <intent-name>
    steps:
      - action: my_action
```

如果仅希望在置信度高于阈值时触发流，请使用以下语法：

```yaml title="flows.yml" hl_lines="4-7"
flows:
  my_flow:
    description: "A flow triggered with <intent-name>"
    nlu_trigger:
      - intent:
          name: <intent-name>
          confidence_threshold: 0  # threshold value, optional
    steps:
      - action: my_action
```

### NLU 触发器中的多意图 {#multiple-intents-in-nlu-trigger}

可以为 `nlu_trigger` 列出多个意图。如果预测到任何这些意图，流将由 [NLUCommandAdapter](components/nlu-command-adapter.md) 启动。

!!! warning "警告"

    为了实际使用 `nlu_trigger`，你需要在配置文件中的 NLU 管道中将 `LLMCommandGenerator` 添加到 [NLUCommandAdapter](components/nlu-command-adapter.md)  之前。

## 阻止流启动 {#preventing-flows-from-starting}

### 流守卫 {#flow-guards}

流守卫是通过在流定义中添加额外的 `if` 字段来指定的。例如，以下显示用户最新账单的流只有在槽 `authenticated` 和 `email_verified` 都为 `true` 时才会触发。

```yaml title="flows.yml" hl_lines="4"
flows:
  show_latest_bill:
    description: A flow with a flow guard.
    if: slots.authenticated AND slots.email_verified
    steps:
      - action: my_action
```

如果 `if` 键后的[条件](conditions.md)不满足，则无法启动流。但是，也有一些例外情况：

1. 流通过另一个流的 [`link`](flows.md#link) 步骤触发。
2. 流通过另一个流的 [`call`](flows.md#call) 步骤触发。
3. 流已使用 [NLU 触发器](starting-flows.md#nlu-trigger)定义了意图。在这种情况下，意图触发消息（例如 `/initialize_conversation`）可以针对目标意图强制启动流。

如果有一个流应该仅通过 [`link`](flows.md#link) 或 [`call`](flows.md#call) 步骤启动，则可以通过添加 `if: False` 来指定。例如：

```yaml title="flows.yml" hl_lines="4"
flows:
  feedback_form:
    description: A flow should only be linked to.
    if: False
    steps:
      - action: my_action
```
