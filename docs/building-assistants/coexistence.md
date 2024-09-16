# 将基于 NLU 的对话机器人迁移到 CALM

!!! info "3.8 版本新特性"

    共存功能可帮助你从基于 NLU 的对话机器人迁移到[带有语言模型的对话式 AI (CALM) ](../calm.md)，从 `3.8.0` 版本开始可用。

共存允许你同时运行使用[带语言模型的对话式 AI (CALM)](../calm.md) 方法和[基于 NLU 的系统](../nlu-based-assistants/model-configuration.md)的对话机器人，以便以迭代方式从基于 NLU 的对话机器人手迁移到 CALM。这样，你无需一次性迁移，而是可以逐步开始在生产中使用 CALM。

!!! info "信息"

    有关共存功能的深入解释，请参阅[共存](../concepts/business-logic/coexistence.md)。

## 词汇表 {#glossary}

- **CALM**：[CALM](../calm.md)（带语言模型的对话式 AI），Rasa 的新系统，用于快速创建强大且可操控的大型语言模型对话体验。
- **基于 NLU 的系统**：[基于 NLU 的系统](../nlu-based-assistants/model-configuration.md)依靠 NLU 组件来处理传入的用户消息并检测意图和实体。此外，规则和故事用于引导下一步动作。
- **系统**：每当我们使用“两个系统”或“任一系统”等短语时，我们指的是 CALM 或基于 NLU 的系统。
- **技能**：一小段独立的业务逻辑，可让用户实现目标或回答一个或多个问题。技能的示例包括转账、查看账户余额或回答常见问题。通常，技能需要用户在多个步骤中提供多条信息。
- **技能中断**：如果在执行过程中启动另一项技能而不停止先前的技能，则技能会中断。例如，在汇款过程中，用户可能会被要求确认交易，然后再进行最终确认，检查其账户余额或交易历史记录。在许多情况下，最好在中断技能完成后返回到被中断的技能。
- **主题领域**：一组紧密相关且经常同时发生并相互干扰的技能。主题领域的一个例子是“投资”，其中包含提供证券报价、买卖证券、投资组合管理、提供公司季度和年度业绩等技能。

## 入门 {#getting-started}

!!! important "重要"

    在共存中，CALM 和基于 NLU 的系统都存在于同一个对话机器人中，但彼此分离。虽然可以按顺序使用这两个系统，即在 CALM 技能完成后触发基于 NLU 的系统技能，反之亦然，但基于 NLU 的系统技能不可能中断 CALM 技能，反之亦然。技能中断仅限于在其中一个系统内发生。

### 主题领域的识别 {#identification-of-topic-areas}

要开始使用共存，你必须在对话机器人中识别单个技能和至少一个主题领域以进行迁移。识别单个技能很有用，因为它们最常在 CALM 中变成单个流。主题领域很重要，因为它们将应该迁移在一起的技能进行分组，并允许在单个对话会话中轻松中断技能。

例如，如果对话机器人允许转账，并且你想将该技能迁移到 CALM，那么你可能还想迁移：

- 技能
    - 检查余额
    - 查看交易历史记录
    - 添加联系人（如果用户在汇款之前必须先将某人添加为联系人）
- 回答有关汇款主题的常见问题

迁移这些其他技能也将允许用户在开始汇款时启动这些技能。如果其他技能仅保留在基于 NLU 的系统中，则只能在汇款完成或中止后启动它们。

在某些情况下，你可能希望技能在两个系统中都可用，在这种情况下，你还应该在基于 NLU 的系统中保留该技能的副本。将技能保留在基于 NLU 的系统中有两个原因：

- 该技能通常由两个系统中的主题使用。
- 这是一项高风险技能，但由于技能中断只能在一个系统中发生，因此你希望对话机器人仍然能够在两个系统中处理它。

一旦你确定了技能和要迁移的第一个主题区域，你就可以开始编写[流](../concepts/flows.md)。

在接下来的步骤中，我们将了解如何将你的新流与基于 NLU 的系统的其余部分集成。

## 更新 NLU 管道和策略 {#updating-your-nlu-pipeline-and-policies}

首先，你需要决定路由机制是否应基于意图，或者 LLM 是否应决定传入消息应路由到何处。你可以在两个不同的[路由器组件](../concepts/components/coexistence-routers.md)之间进行选择：`IntentBasedRouter` 和 `LLMBasedRouter`。[`IntentBasedRouter`](../concepts/components/coexistence-routers.md#intentbasedrouter) 根据 NLU 组件的预测意图路由消息。[`LLMBasedRouter`](../concepts/components/coexistence-routers.md#llmbasedrouter) 利用 LLM 来决定传入消息是否应路由到基于 NLU 的系统或 CALM。有关这些组件的更多信息，请阅读[路由器组件文档](../concepts/components/coexistence-routers.md)。

决定使用哪个路由器组件后，你需要将其与基于 LLM 的命令生成器和 `FlowPolicy` 一起添加到配置文件中。有关基于 LLM 的命令生成器或 `FlowPolicy` 的更多信息，请查阅[流](../concepts/flows.md)和[对话理解](../concepts/dialogue-understanding.md)文档。

添加组件后，你的 `config.yml` 可能如下所示，具体取决于你选择的路由器以及你使用的基于 NLU 的系统组件：

```yaml
recipe: default.v1
language: en
pipeline:
- name: LLMBasedRouter
  calm_entry:
    sticky: "handles everything around contacts"
- name: WhitespaceTokenizer
- name: CountVectorsFeaturizer
- name: CountVectorsFeaturizer
  analyzer: char_wb
  min_ngram: 1
  max_ngram: 4
- name: LogisticRegressionClassifier
- name: SingleStepLLMCommandGenerator
  llm:
    model_name: gpt-4
    request_timeout: 7
    max_tokens: 256

policies:
- name: FlowPolicy
- name: RulePolicy
- name: MemoizationPolicy
  max_history: 10
- name: TEDPolicy
```

### 将路由槽添加到领域中 {#adding-the-routing-slot-to-your-domain}

路由槽用于存储路由器组件的路由决策，例如，是否应将消息路由到基于 NLU 的系统或 CALM。

请务必将路由槽添加到领域中，如下所示：

```yaml
version: "3.1"

slots:
  route_session_to_calm:
    type: bool
    influence_conversation: false
```

这需要完全按照这种方式进行。它需要是一个布尔槽，以便根据其值捕获三种情况：

- `None`：没有路由处于活动状态，路由器将处于活动状态。
- `True`：路由到 CALM 处于活动状态。
- `False`：路由到基于 NLU 的系统处于活动状态。

当路由槽设置为 `True` 或 `False` 时，路由会话处于活动状态。传入消息将相应地路由。如果路由槽被重置并且没有路由处于活动状态，例如设置为 `None`，路由器将在下一条传入消息时再次处于活动状态。要重置路由槽，可以调用新的默认动作 [`action_reset_routing`](../concepts/default-actions.md#action_reset_routing)（请参阅[此部分](../building-assistants/coexistence.md#managing-routing-resets)）。

`influence_conversation: false` 很重要，因为它不会根据此槽的值触发基于 NLU 的系统策略。

如果你想让其中一个系统从对话一开始就处于活动状态，你可以为槽指定一个 `initial_value`。在路由重置期间，此槽将始终设置为 `None`，而不是其初始值。

!!! warning "注意"

    如果你想要重置路由槽，请调用新的默认动作 `action_reset_routing`。请勿单独更新路由槽的值（例如在自定义动作中），因为这可能会导致对话机器人出现意外行为。如果你的目标是重置路由，请始终使用 `action_reset_routing`。

### 管理路由重置 {#managing-routing-resets}

重置路由槽对于在单个会话中实现多项技能非常重要。要重置路由，需要调用新的默认动作 [`action_reset_routing`](../concepts/default-actions.md#action_reset_routing)。

#### CALM 完成后重置

CALM 完成后重置相对简单。每当所​​有启动的流完成时，CALM 都会触发流 `pattern_completed`。我们可以利用这一事实并稍微调整此模式，如下所示：

```yaml hl_lines="7"
flows:
    pattern_completed:
      description: all flows have been completed and there is nothing else to be done
      name: pattern completed
      steps:
        - action: utter_can_do_something_else
        - action: action_reset_routing
```

在 [`pattern_completed` 的原始定义](../concepts/conversation-repair.md#reference-default-pattern-configuration)中，我们只是询问用户我们是否可以为他们做其他事情。在此版本中，我们还重置了路由。确保将此代码段添加到流中，以覆盖 CALM 附带的 `pattern_completed` 的默认实现。

此设置将处理以下所有情况：

- 用户启动流并完成或取消它。
- 用户按顺序或同时启动多个流并完成或取消它们。
- 用户启动流并在流中询问知识问题或使用闲聊，然后完成或取消流。
- 用户直接使用与知识相关的问题启动 CALM 会话。

#### 基于 NLU 的系统完成后重置

与技能边界定义明确的 CALM 相比，基于 NLU 的系统策略使用更松散连接的业务逻辑片段来描述技能。如果你想允许用户在基于 NLU 的系统中完成一项技能后使用对话机器人的 CALM 部分，则必须在规则和故事中的适当位置添加 [`action_reset_routing`](../concepts/default-actions.md#action_reset_routing)。

通常，这将在不同的故事和规则的末尾。你可能会在每次交互结束时执行一些对话部分。这些部分可能是你可以在其末尾重置路由的好地方，如下所示：

```yaml hl_lines="12"
version: "3.1"

stories:
- story: feedback
  steps:
  - checkpoint: start_of_feedback
  - action: utter_ask_feedback
  - intent: submit_rating
  - action: store_rating
  - action: utter_thank_you_rating
  - action: utter_do_something_else
  - action: action_reset_routing
```

很有可能，当你确定用户刚刚完成一项技能，并且在基于 NLU 的系统中没有其他需要包装的内容（例如中断的技能）时，你必须在多个故事或规则的末尾添加 `action_reset_routing`。

## 管理 CALM 和基于 NLU 的系统之间的共享槽 {#managing-shared-slots-between-calm-and-the-nlu-based-system}

为了防止在 [`action_reset_routing`](../concepts/default-actions.md#action_reset_routing) 期间重置槽，你可以使用选项 [`shared_for_coexistence: True`](../concepts/domain.md#persistence-of-slots-during-coexistence) 配置槽。

或者，你可以将用户个人资料信息存储在数据库中，并在故事和流期间在不同位置检索它并将其临时存储在槽中。在这种情况下，你无需担心共享槽的持久性，因为数据会持久保存在数据库中。

## 自定义动作 {#custom-actions}

将功能移至 CALM 时，你可能需要在不同程度上调整自定义动作。某些概念（例如动态表单）已集成到核心流功能中，应使用流中的分支来实现。最好重新考虑移至 CALM 的所有自定义动作的实现，并评估是否应对其进行调整以更好地适应新范式。

一般而言，也可以在 CALM 和基于 NLU 的系统中同时使用相同的动作。两个系统共享相同的动作服务器。

## 测试对话机器人 {#testing-your-assistant}

使用 CALM 和基于 NLU 的系统测试对话机器人可以通过 [e2e 测试](../production/testing-your-assistant.md#end-to-end-testing)来实现。这些测试的定义不太依赖于底层实现，因此当越来越多的技能随后从基于 NLU 的系统迁移到 CALM 时，不需要进行重大更改。e2e 测试可帮助你跨范式测试技能中断和技能触发器。

你仍然可以使用命令 [`rasa test`](../command-line-interface.md#rasa-test) 仅评估对话机器人的基于 NLU 的系统。
