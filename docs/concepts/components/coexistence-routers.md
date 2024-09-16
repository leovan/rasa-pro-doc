# 共存路由器

[CALM 与基于 NLU 的系统共存](../../building-assistants/coexistence.md)取决于路由机制，该机制根据消息的内容将消息路由到任一系统。你可以在两个不同的路由器组件之间进行选择：

- [`IntentBasedRouter`](#intentbasedrouter)：NLU 管道的预测意图用于决定消息应该去往何处。
- [`LLMBasedRouter`](#llmbasedrouter)：此组件利用 LLM 来决定是否应将消息路由到基于 NLU 的系统或 CALM。

你只能在对话机器人中使用其中一个路由器组件。

## IntentBasedRouter {#intentbasedrouter}

`IntentBasedRouter` 使用来自 NLU 组件的预测意图并根据该意图路由消息。需要将路由器添加到配置文件中的管道中。

!!! info "注意"

    `IntentBasedRouter` 的位置需要在 NLU 组件之后、命令生成器之前。

根据选择的其他组件，配置文件可能如下所示。

```yaml hl_lines="11"
recipe: default.v1
language: en
pipeline:
- name: WhitespaceTokenizer
- name: CountVectorsFeaturizer
- name: CountVectorsFeaturizer
  analyzer: char_wb
  min_ngram: 1
  max_ngram: 4
- name: LogisticRegressionClassifier
- name: IntentBasedRouter
  # additional configuration parameters
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

### IntentBasedRouter 的配置 {#configuration-of-the-intentbasedrouter}

需要配置以下强制配置参数：

- `nlu_entry`:
    - `sticky`：应以粘性方式路由到基于 NLU 的系统的意图列表。
    - `non_sticky`：应以非粘性方式路由到基于 NLU 的系统的意图列表。
- `calm_entry`:
    - `sticky`：应以粘性方式路由到 CALM 的意图列表。

`IntentBasedRouter` 的完整配置可能如下所示。

```yaml
pipeline:
# ...
- name: IntentBasedRouter
  nlu_entry:
    sticky:
      - transfer_money
      - check_balance
      - search_transactions
    non_sticky:
      - chitchat
  calm_entry:
    sticky:
      - book_hotel
      - cancel_hotel
      - list_hotel_bookings
# ...
```

!!! info "注意"

    一旦 `IntentBasedRouter` 将会话分配给基于 NLU 的系统，`LLMCommandGenerator` 将被跳过，这样就不会产生不必要的成本。

### 处理缺失的意图 {#handling-missing-intents}

如果 NLU 组件预测了意图，但该意图不属于 `IntentBasedRouter` 中列出的任何意图，并且当前未设置路由会话，则消息将根据以下规则路由：

1. 如果给定意图，则激活任何流的 NLU 触发器（请参阅 [NLU 触发器文档](../starting-flows.md#nlu-trigger)），将路由到 CALM。
2. 否则，将路由到基于 NLU 的系统。

## LLMBasedRouter {#llmbasedrouter}

`LLMBasedRouter` 使用 LLM（默认为 `gpt-3.5-turbo`）来决定是否应将消息路由到基于 NLU 的系统或 CALM。

!!! important "重要"

    为了将此组件用于共存解决方案，你需要将其作为配置文件中管道的第一个组件添加。

根据选择的其他组件，配置文件可能如下所示。

```yaml hl_lines="4"
recipe: default.v1
language: en
pipeline:
- name: LLMBasedRouter
  calm_entry:
    sticky: handles everything around hotel bookings
  # additional configuration parameters
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

### 配置 LLMBasedRouter 组件 {#configuring-the-llmbasedrouter-component}

`LLMBasedRouter` 组件具有以下配置参数：

- `nlu_entry`:
    - `sticky`：描述基于 NLU 的一般系统功能。默认情况下，该值为 `"handles everything else"`。
    - `non_sticky`：描述基于 NLU 的系统的功能，该功能不应导致基于 NLU 的系统的粘性路由。默认情况下，该值为 `"handles chitchat"`。
- `calm_entry`:
    - `sticky`（必需）：描述对话机器人的 CALM 系统中实现的功能。
- `llm`：llm 的配置（请参阅[此部分](#configuring-the-llm-of-the-llmbasedrouter)）。
- `prompt`：要使用的提示模板（jinja2 模板）的文件路径（请参阅[此部分](#configuring-the-prompt-of-the-llmbasedrouter)）

如果你想使用 Azure OpenAI 服务，可以按照 [Azure OpenAI 服务](llm-configuration.md#azure-openai-service)部分所述配置必要的参数。

#### 配置 LLMBasedRouter 的提示 {#configuring-the-prompt-of-the-llmbasedrouter}

默认情况下，配置的描述会组合成一个提示描述一个到 LLM 的路由任务：

```txt
You have to forward the user message to the right assistant.

The following assistants are available:

Assistant A: {{ calm_entry_sticky }}
Assistant B: {{ nlu_entry_non_sticky }}
Assistant C: {{ nlu_entry_sticky }}

The user said: """{{ user_message }}"""

Answer which assistant needs to get this message:
The message is for the assistant with the letter
```

配置参数 `nlu_entry.sticky` 进入 `{{ nlu_entry_sticky }}`，`nlu_entry.non_sticky` 进入 `{{ nlu_entry_non_sticky }}`，`calm_entry.sticky` 进入 `{{ calm_entry_sticky }}`。

此提示比 [`LLMCommandGenerator`](../dialogue-understanding.md#using-nlucommandadapter-and-llm-based-command-generators) 的提示简单得多，并且短 10 倍。利用 `gpt-3.5-turbo`，它比使用 `gpt-4` 的 `LLMCommandGenerator` 便宜约 200 倍。

你可以通过将自己的提示编写为 jinja2 模板并将其作为文件提供给组件来修改提示：

```yaml
pipeline:
# ...
- name: LLMBasedRouter
  prompt: prompts/llm-based-router-prompt.jinja2
# ...
```

!!! info "信息"

    一旦 `LLMBasedRouter` 将会话分配给基于 NLU 的系统，`LLMCommandGenerator` 将被跳过，这样就不会产生不必要的成本。

#### 配置 LLMBasedRouter 的 LLM {#configuring-the-llm-of-the-llmbasedrouter}

默认情况下，使用以下 LLM 配置：

```yaml
pipeline:
# ...
- name: LLMBasedRouter
  llm:
    _type: "openai"
    model_name: "gpt-3.5-turbo"
    request_timeout: 7
    temperature: 0.0
    max_tokens: 1
    logit_bias:
      "362": 100
      "426": 100
      "356": 100
# ...
```

这里有趣的设置是：

- `max_tokens: 1`，让 LLM 仅预测单个 token。
- `logit_bias` 分别提高 LLM 预测 token `" A"`，`" B"` 或 `" C"` 的概率。
    - 这些是三个大写字母的 token，前面有一个空格，用于预测路由器向 LLM 呈现的三种情况。
    - 362、426、356 是 chatgpt 和 gpt-4 模型中这三个 token 的 token id。

!!! info "信息"

    如果更改模型，也应该调整这些逻辑偏差或至少将其删除，以免提高其他模型中不相关标记的性能！

### 处理 LLM 的故障和停机时间 {#handling-failures-and-downtime-of-the-llm}

如果 LLM 预测的答案无效，例如 `A`、`B` 或 `C` 以外的其他字符，或者 LLM 的 API 已关闭且无法访问 LLM，则消息将以粘性方式路由到基于 NLU 的系统。
