# NLU 命令适配器

NLU 命令适配器将经典 NLU 管道（意图和实体）的输出转换为 CALM 可用于执行业务逻辑的命令。

## 工作原理 {#how-the-nlucommandadapter-works}

`NLUCommandAdapter` 使用传统方式启动流，例如使用意图分类器的预测意图。它查看[意图分类器](../../nlu-based-assistants/components.md#intent-classifiers)的预测意图，并尝试找到定义了相应 [NLU 触发器](../starting-flows.md#nlu-trigger)的流。如果流具有与预测意图匹配的 NLU 触发器，并且置信度大于 NLU 触发器中给定的阈值，则 `NLUCommandAdapter` 将返回 `StartFlow` 命令以启动相应流。

## 使用 NLUCommandAdapter {#using-the-nlucommandadapter}

要在对话机器人中使用此组件，请在 `config.yml` 文件中将 `NLUCommandAdapter` 添加到 NLU 管道。你还需要在 NLU 管道中列出一个[意图分类器](../../nlu-based-assistants/components.md#intent-classifiers)。在[此处](overview.md)阅读有关 `config.yml` 文件的更多信息。

```yaml title="config.yml"
pipeline:
# - ...
  - name: NLUCommandAdapter
# - ...
```

## 何时使用 NLUCommandAdapter {#when-to-use-the-nlucommandadapter}

我们建议在两种情况下使用 `NLUCommandAdapter`：

- 你希望使用包含意图的样本的 NLU 数据以及 CALM 范式。使用 `NLUCommandAdapter`，你可以根据预测的意图启动流，前提是你已经有一个可靠的意图分类器。一旦启动流，业务逻辑将像往常一样在 CALM 范式中执行，其中 `LLMCommandGenerator` 预测命令，策略预测下一个最佳动作。
- 你希望通过不每次都对 LLM 进行 API 调用来最大限度地降低成本。与 `LLMCommandGenerator` 相比，`NLUCommandAdapter` 不会对 LLM 进行任何 API 调用。使用 `NLUCommandAdapter` 可以节省一些成本。使用 `NLUCommandAdapter` 时，请确保有一个可靠的意图分类器，否则，将会开启错误的流。

## 自定义 {#customization}

要限制用户消息的长度，你可以设置 `user_input.max_characters`（默认值为 `420` 个字符）。

```yaml title="config.yml" hl_lines="4"
pipeline:
  - name: NLUCommandAdapter
    user_input:
      max_characters: 420
```
