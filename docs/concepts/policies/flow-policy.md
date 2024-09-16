# 流策略

执行 Rasa 流中定义的业务逻辑。

!!! info "3.7 版本新特性"

    流策略是 Rasa 的新[语言模型（CALM）对话式 AI](../../calm.md) 的一部分，从 `3.7.0` 版本开始可用。

!!! info "将流策略与基于 NLU 的策略添加在一起"

    如果你正在将基于 NLU 的对话机器人逐步地迁移到基于 CALM 的对话机器人，请查看[迁移到 CALM 的指南](../../building-assistants/coexistence.md)以获取有关如何执行此动作的说明。

流策略是一种状态机，可确定性地执行[流](../flows.md)中定义的业务逻辑。

流策略负责监督对话机器人的状态、处理状态转换，并在[对话修复](../conversation-repair.md)需要时启动新流。

## 将流策略添加到对话机器人 {#adding-the-flow-policy-to-your-assistant}

要使用流策略，请将其添加到 `config.yml` 中的策略列表中。

```yaml title="config.yml"
# ...

policies:
  - name: FlowPolicy
```

流策略没有任何额外的配置参数。

## 它是如何工作的？ {#how-does-it-work}

`FlowPolicy` 采用对话堆栈结构（后进先出）以及内部槽来管理对话的状态。

## 管理状态 {#managing-the-state}

流策略使用“对话堆栈”管理状态。每当启动一个流时，它都会被推送到对话堆栈中，并且该堆栈会追踪每个流中的当前位置。对话堆栈遵循“后进先出”顺序，这意味着最近启动的流将首先完成。一旦完成，下一个最近启动的流将继续。

考虑本[教程](../../tutorial.md)中的 `transfer_money` 流：

```yaml title="flows.yml"
flows:
  transfer_money:
    description: |
      This flow lets users send money to friends
      and family, in US Dollars.
    steps:
      - collect: recipient
      - collect: amount
      - action: utter_transfer_complete
```

当对话到达 `collect` 步骤时，对话机器人将向用户询问信息以帮助其填充相应的槽。在第一步中，对话机器人会说“Who would you like to send money to?”然后等待回复。

当用户回复时，[对话理解](../dialogue-understanding.md)组件将生成一系列命令，这些命令将决定接下来会发生什么。在最简单的情况下，用户说“to Jen”之类的话，然后生成命令 `SetSlot("recipient", "Jen")`。

流收集了所需的信息，流策略继续进行下一步。如果用户说“I want to send 100 dollars to Jen”，则会生成命令 `SetSlot("recipient", "Jen")`、`SetSlot("amount", 100)`，流策略将直接跳到流的最后一步。

除了提供对话机器人请求的槽值之外，用户可能会说很多话。他们可能会澄清说他们根本不想汇款，提出澄清问题，或者改变之前说过的话。这些情况由[对话修复](../conversation-repair.md)处理。'

## 启动新流 {#starting-new-flows}

流可以通过多种方式启动：

- 当 Rasa 组件将流放入堆栈时，流就会启动。例如，当 [LLM 命令生成器](../dialogue-understanding.md)确定某个流适合当前对话时，它会将流放入堆栈。
- 一个流可以[“链接”到另一个流](../flows.md#link)，后者将启动链接的流，并在链接的流完成后返回到原始流。
- 在[对话修复](../conversation-repair.md)的情况下，`FlowPolicy` 可以自动添加默认流。
