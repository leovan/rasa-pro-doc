# 无意图策略

替代开放式响应生成，使用 LLM 从领域中选择现有响应。

!!! info "3.7 版本新特性"

    无意图策略是 Rasa 的新[语言模型（CALM）对话式 AI](../../calm.md) 的一部分，从 `3.7.0` 版本开始可用。

无意图策略用于发送在领域中定义但不属于任何流的 `responses`。这有助于有效地处理闲聊、上下文问题和高风险主题。为了提高其性能并使其满足特定要求，你可以自定义策略的提示并添加示例对话。

## 将无意图策略添加到对话机器人 {#adding-the-intentless-policy-to-your-bot}

要将 `IntentlessPolicy` 添加到对话机器人，请将其添加到 `config.yml` 中：

=== "Rasa Pro >= 3.8.x"

    ```yaml title="config.yml"
    policies:
    # ... any other policies you have
    - name: IntentlessPolicy
    ```

=== "Rasa Pro <=3.7.x"

    ```yaml title="config.yml"
    policies:
    # ... any other policies you have
    - name: rasa_plus.ml.IntentlessPolicy
    ```

与所有使用 LLM 的组件一样，你可以配置要使用的提供者和模型以及其他参数。所有这些都在[此处](../components/llm-configuration.md)描述。

如果你想使用 Azure OpenAI 服务，可以按照 [Azure OpenAI 服务](../components/llm-configuration.md#azure-openai-service)部分所述配置必要的参数。

## 自定义提示模板 {#customizing-the-prompt-template}

你可以通过在 `config.yml` 中设置 `prompt` 属性来更改用于生成响应的提示模板：

=== "Rasa Pro >= 3.8.x"

    ```yaml title="config.yml" hl_lines="4"
    policies:
    # - ...
    - name: IntentlessPolicy
        prompt: prompts/intentless-policy-template.jinja2
    ```

=== "Rasa Pro <=3.7.x"

    ```yaml title="config.yml" hl_lines="4"
    policies:
    # - ...
    - name: rasa_plus.ml.IntentlessPolicy
        prompt: prompts/intentless-policy-template.jinja2
    ```

提示是一个 [Jinja2](https://jinja.palletsprojects.com/en/3.0.x/) 模板，可用于自定义提示。提示中提供以下变量：

- `conversations`：从故事中获取的用户与 AI 对话机器人之间的示例对话列表。
- `current_conversation`：与用户的当前对话。
- `responses`：对话机器人可以发送的示例响应列表。

## 指导无意图策略 {#steering-the-intentless-policy}

无意图策略通常可以以零样本方式选择正确的响应。也就是说，无需向 LLM 提供任何示例对话。

但是，你可以通过添加示例对话来提高策略的性能。为此，请将[端到端故事](../../nlu-based-assistants/training-data-format.md#end-to-end-training)添加到 `data/e2e_stories.yml` 的训练数据中。这些对话将用作示例，帮助无意图策略学习。

```yaml title="data/e2e_stories.yml"
- story: currencies
  steps:
    - user: How many different currencies can I hold money in?
    - action: utter_faq_4

- story: automatic transfers travel
  steps:
    - user: Can I add money automatically to my account while traveling?
    - action: utter_faq_5

- story: user gives a reason why they can't visit the branch
  steps:
    - user: I'd like to add my wife to my credit card
    - action: utter_faq_10
    - user: I've got a broken leg
    - action: utter_faq_11
```

## 闲聊 {#chitchat}

在企业环境中，使用纯生成模型来处理闲聊可能并不合适。团队希望确保对话机器人始终与品牌保持一致并传达信息。使用无意图策略，可以定义对话机器人可以发送的经过审查的人工编写的 `responses`。由于无意图策略利用 LLM 并在选择合适的 `response` 时考虑对话的整个上下文，因此它比简单地预测意图并触发固定响应要强大得多。

## 高风险主题 {#high-stakes-topics}

对于处理高风险主题，无意图策略是[企业搜索策略](enterprise-search-policy.md)的强大替代方案。

例如，如果用户对政策、法律条款或保证有疑问，例如：“My situation is X, I need Y. Is that covered by my policy?” 在这些情况下，企业搜索策略的 RAG 方法存在风险。即使提示中存在相关内容，RAG 方法也允许 LLM 对文档进行解释。即使提示中存在相关内容，RAG 方法也允许 LLM 对文档进行解释。用户获得的答案将根据他们问题的确切措辞而有所不同，并且可能会在底层模型发生变化时发生变化。

对于高风险主题，最安全的做法是发送一个独立的、经过审查的答案，而不是依赖生成模型。无意图政策在 CALM 对话机器人中提供了这种功能。请注意，在这些情况下，对话设计至关重要，因为你希望你的回答是独立的、明确的，而不仅仅是“是”或“否”。

### 启用无意图策略 {#enabling-the-intentless-policy}

在 Rasa 中，基于知识的问题被定向到默认的 `pattern_search` 流。默认情况下，它会以 `utter_no_knowledge_base` 响应，从而拒绝请求。但是，你可以对其进行自定义以初始化触发无意图策略的 [`action_trigger_chitchat`](../default-actions.md#action_trigger_chitchat)。

!!! info "信息"

    覆盖 `pattern_search` 时，你可以选择无意图策略或企业搜索策略来处理基于知识的问题。两种策略不能同时启用。

## 插入语 {#interjections}

当流到达 `collect` 步骤并且对话机器人向用户询问信息时，你的用户可能会提出澄清问题、拒绝回答或在流继续时插入。在这些情况下，无意图策略可以根据上下文选择适当的响应，然后允许流继续。

## 测试 {#testing}

编写[端到端测试](../../production/testing-your-assistant.md#end-to-end-testing)可让你评估无意图策略的性能并防止回退。
