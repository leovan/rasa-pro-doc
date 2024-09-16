# 对话理解

对话理解旨在了解人工智能对话机器人的最终用户希望如何进行对话。

!!! info "3.7 版本新特性"

    命令生成器是 Rasa 的新[语言模型 (CALM) 对话式 AI 方法](../calm.md)的一部分，从 `3.7.0` 版本开始可用。

CALM 的对话理解模块将最新的用户消息与对话上下文转换为一组[命令](#command-reference)，对话机器人使用这些命令来执行定义的[业务逻辑](flows.md)。它在[命令生成器](#commandgenerator)的帮助下实现这一点。

## 命令生成器 {#commandgenerator}

命令生成器将最新的用户消息与对话上下文一起作为输入，并将其转换为一组[命令](#command-reference)。目前，有三个命令生成器可用：

1. [`SingleStepLLMCommandGenerator`](components/llm-command-generators.md#singlestepllmcommandgenerator)
2. [`MultiStepLLMCommandGenerator`](components/llm-command-generators.md#multistepllmcommandgenerator)
3. [`NLUCommandAdapter`](components/nlu-command-adapter.md)

要使用其中一个基于 LLM 的命令生成器，请按照[基于 LLM 的命令生成器](components/llm-command-generators.md)页面上的说明进行操作。

## 使用 NLUCommandAdapter 和基于 LLM 的命令生成器 {#using-nlucommandadapter-and-llm-based-command-generators}

如果你想同时利用 LLM 和经典 NLU 管道来预测命令，那么可以在基于 LLM 的命令生成器之前添加 `NLUCommandAdapter`，例如 `SingleStepLLMCommandGenerator`。

```yaml title="config.yml"
pipeline:
# - ...
  - name: NLUCommandAdapter
  - name: SingleStepLLMCommandGenerator
# - ...
```

组件会一个接一个地执行。如果第一个组件（即 `NLUCommandAdapter`）成功预测了 `StartFlow` 命令，则会跳过 `SingleStepLLMCommandGenerator`（即不会调用 LLM）。

通常，如果第一个命令生成器预测了命令，则会跳过管道中接下来的所有其他命令生成器。在向管道添加自定义命令生成器时请记住这一点。

## 命令参考 {#command-reference}

正如其名称所示，`CommandGenerator` 会生成“命令”，然后进行内部处理以触发当前对话的操作。以下是所有受支持命令的参考，指示 AI 对话机器人应该：

### 启动流 {#start-flow}

启动新[流](flows.md)。

### 取消流 {#cancel-flow}

取消当前[流](flows.md)。它为[对话修复中的取消用例](conversation-repair.md#3-cancellations)提供支持。

### 跳过问题 {#skip-question}

拦截旨在绕过流中当前 [`collect` 步骤](flows.md#collect)的用户消息。它为[对话修复中的跳过 `collect` 步骤用例](conversation-repair.md#4-skipping-collect-steps)提供支持。

### 设置槽 {#set-slot}

将[槽](domain.md#slots)设置为给定值。

### 修正槽 {#correct-slots}

将给定[槽](domain.md#slots)的值修改为新值。它为[对话修复中的修正用例](conversation-repair.md#2-corrections)提供支持。

### 澄清 {#clarify}

要求澄清。它支持[对话修复中的澄清用例](conversation-repair.md#7-clarification)。

### 闲聊回复 {#chit-chat-answer}

以闲聊风格回复回复，无论它们是预定义还是自由格式。它为[对话修复中的闲聊用例](conversation-repair.md#5-chitchat)提供支持。

### 知识回复 {#knowledge-answer}

基于知识的自由格式回复。它与[企业搜索策略](policies/enterprise-search-policy.md)协同工作。

### 人工处理 {#human-handoff}

将对话交给人工。

### 错误 {#error}

此命令表示由于[内部错误](conversation-repair.md#8-internal-errors)导致的 AI 对话机器人无法处理对话。

### 无法处理 {#cannot-handle}

此命令表示命令生成器无法生成任何命令。它为[对话修复中的无法处理用例](conversation-repair.md#9-cannot-handle)提供支持。默认情况下，此命令不包含在 `SingleStepLLMCommandGenerator` 提供给 LLM 的提示中，但它包含在 `MultiStepLLMCommandGenerator` 中。

### 更改流 {#change-flow}

此命令表示命令生成器请求更改流。此命令由 `MultiStepLLMCommandGenerator` 单独预测并在内部使用。
