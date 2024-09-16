# 对话修复

对话修复是指对话机器人处理偏离预期路径的对话的能力。Rasa 使用一组可自定义的模式自动处理对话修复。

!!! info "3.7 版本新特性"

    对话修复是 Rasa 的新[语言模型 (CALM) 对话式 AI 方法](../calm.md)的一部分，从 `3.7.0` 版本开始可用。

## 概述 {#overview}

[流](flows.md)描述了对话机器人的业务逻辑。在[教程](../tutorial.md)示例中，`transfer_money` 流指定对话机器人需要向用户询问 `recipient` 和 `amount_of_money`。

“预期流经”是指每次对话机器人向用户询问信息时，用户都会提供一个成功填充请求槽的答案。对话修复可以处理所有与预期路径不同的对话。

例如：

- 对话机器人询问金额，但用户说了其他话。
- 用户中断当前流并将上下文切换到另一个主题。
- 用户改变了之前所说的话的想法。

Rasa 有处理每种情况的默认模式。每种模式都是预构建的[流](flows.md)。可以通过向对话机器人添加同名的流来自定义。

## 对话修复案例 {#conversation-repair-cases}

### 1. 离题 {#1-digressions}

当用户从一个流转到另一个流时，就会出现离题。

**示例**：在转账过程中，用户可能会询问其当前余额。

<div class="md-chat"><div class="chat-container"><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">I want to send some money to Sudarshana</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">How much do you want to send to Sudarshana?</span></p></div><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">Ah wait, how much money do I have?</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">You currently have 4021.20$ in your account.</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">Let's continue with sending money to Sudarshana.</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">How much money do you want to send to Sudarshana then?</span></p></div><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">...</span></p></div></div><figcaption>用户在转账过程中离题</figcaption></div>

### 2. 更正 {#2-corrections}

更正发生在用户修改输入数据或纠正错误时。

**示例**：用户可能会改变转账接收人。

<div class="md-chat"><div class="chat-container"><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">I want to send some money to Joe</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">How much money do you want to send?</span></p></div><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">50$</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">Do you want to send 50$ to Joe? (Yes/No)</span></p></div><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">Oh wait!! I meant to say to John, not Joe!</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">Updated recipient to John</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">Do you want to send 50$ to John? (Yes/No)</span></p></div><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">...</span></p></div></div><figcaption>用户更正收款人姓名</figcaption></div>

!!! info "重要"

    更正后，流将重新追溯以与更正后的数据保持一致。用户可能会看到基于更正的替代流。

### 3. 取消 {#3-cancellations}

当用户中途停止流时，就会发生取消。

**示例**：用户在启动流后选择不汇款。

<div class="md-chat"><div class="chat-container"><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">I want to send some money to Dimitri</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">How much money do you want to send?</span></p></div><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">Ah, nevermind. I see I have already sent it earlier.</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">Okay, I am cancelling the transfer.</span></p></div><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">...</span></p></div></div><figcaption>用户取消转账</figcaption></div>

### 4. 跳过收集步骤 {#4-skipping-collect-steps}

当用户试图绕过 [`collect` 步骤](flows.md#collect)时，通过拒绝提供所请求的信息或请求跳过当前步骤。

**示例**：用户拒绝回答机器人问题。

<div class="md-chat"><div class="chat-container"><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">I want to send some money to Dimitri</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">How much money do you want to send?</span></p></div><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">Go to the next question.</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">I'm here to provide you with the best assistance, and in order to do so, I kindly request that we complete this step together. Your input is essential for a seamless experience!</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">How much money do you want to send?</span></p></div><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">...</span></p></div></div><figcaption>用户拒绝提供所请求的信息</figcaption></div>

### 5. 闲聊 {#5-chitchat}

参与不影响流的离题互动。

**示例**：用户与对话机器人进行随意交谈，对话机器人以自由格式的回复进行响应。

<div class="md-chat"><div class="chat-container"><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">Hi</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">Hi! I'm your Financial Assistant!</span></p></div><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">are you a bot?</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">I'm a virtual assistant made with Rasa.</span></p></div></div><figcaption>与对话机器人闲聊</figcaption></div>

**默认行为** 依赖于领域内定义的不属于任何流的响应，由[无意图策略](policies/intentless-policy.md)确定适当的响应。如果未配置 `IntentlessPolicy`，对话机器人将激活[无法处理](#9-cannot-handle)模式，从而通知用户无法处理他们的请求。这实际上禁用了闲聊。

此外，你还可以选择明确禁用闲聊。有关如何执行此动作的说明，请参阅[本节](#free-form-generation-for-chitchat)。

你可以自定义默认行为以启用 **自由格式的响应**。请参阅[本节](#free-form-generation-for-chitchat)了解如何执行此动作。

### 6. 完成 {#6-completion}

流以达成用户目标或用户放弃而结束。

**示例**：用户查询账户余额。

<div class="md-chat"><div class="chat-container"><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">Hey, how much money do I have?</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">You currently have 4021.20$ in your account.</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">Is there anything else I can help you with?</span></p></div></div><figcaption>流完成</figcaption></div>

### 7. 澄清 {#7-clarification}

当无法明确识别用户请求并可能匹配多个流时，需要澄清。

**示例**：用户请求可以匹配两个程。

<div class="md-chat"><div class="chat-container"><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">cash</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">I'm not sure what you'd like to achieve. Do you want to check your balance or transfer money?</span></p></div></div><figcaption>澄清</figcaption></div>

### 8. 内部错误 {#8-internal-errors}

错误源自意外的系统或流问题：

1. 不可用的动作、无响应的模块或来自依赖服务的错误（例如 LLM 的超时响应）。
2. 用户输入超出预定义限制（如果设置了限制）。
3. 用户发送空消息。
4. 来自依赖服务的错误，例如来自 LLM 的超时响应。
5. （使用企业搜索策略）在连接到向量存储、文档检索期间或从 LLM 时发生错误。单击[此处](policies/enterprise-search-policy.md#error-handling)了解更多详细信息。

以下是展示不同内部错误场景的示例：

**示例**：由于 LLM 的动作不可用或超时而引发内部错误。

<div class="md-chat"><div class="chat-container"><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">Hey, how much money do I have?</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">Sorry, I am having trouble with that. Please try again later.</span></p></div></div><figcaption>由于 LLM 的动作不可用或超时而引发内部错误</figcaption></div>

**示例**：由于超出输入限制而引发内部错误。

<div class="md-chat"><div class="chat-container"><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">[User sends a message exceeding the set input limit]</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">I'm sorry, but your message is too long for me to process. Please keep your message concise and within a reasonable length.</span></p></div></div><figcaption>由于超出输入限制而引发内部错误</figcaption></div>

**示例**：由于用户消息为空而引发内部错误。

<div class="md-chat"><div class="chat-container"><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content"></span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">I see an empty message. What can I assist you with?</span></p></div></div><figcaption>由于用户消息为空而引发内部错误</figcaption></div>

**上下文属性**

`error_type` 和 `info` 用作管理 [`pattern_internal_error`](#reference-default-pattern-configuration) 中错误的上下文属性。`info` 属性是一个字典，用于包含 `max_characters` 等附加信息。`error_type` 用作 switch-case 选择器。它根据遇到的场景确定适当的错误消息。然后将消息发送给用户。

- `rasa_internal_error_user_input_too_long`，当用户输入超过 `LLMCommandGenerator` 的 `user_input.max_character` 限制时设置。
- `rasa_internal_error_user_input_empty`，由空白输入设置。

这些上下文属性在 `pattern_internal_error` 之外不可用。

### 9. 无法处理 {#9-cannot-handle}

触发此模式是为了妥善处理以下情况：

- 当[基于 LLM 的命令生成器](dialogue-understanding.md)遇到无法预测有效命令的情况（例如，LLM 产生幻觉试图启动不存在的流）时，此机制会提示用户重新措辞其请求。此外，当用户消息的范围超出启动、取消或澄清流时，`MultiStepLLMCommandGenerator` 可以直接预测命令。
- 当[企业搜索策略](policies/enterprise-search-policy.md)无法从向量存储中检索任何相关文档时，会触发此机制。在这种情况下，对话机器人可能需要提示用户重新措辞其请求或通知用户无法找到相关信息。
- 用户沉迷于离题的对话（闲聊），对话机器人设置为使用预定义的响应进行回复，但它是在没有管道中的 [`IntentlessPolicy`](policies/intentless-policy.md) 的情况下进行训练的。

**上下文属性**

模式 [`pattern_cannot_handle`](#reference-default-pattern-configuration) 具有 `reason` 上下文属性。`reason` 设置为：

- `cannot_handle_chitchat`，当 [`pattern_chitchat`](#5-chitchat) 尝试在未定义 `IntentlessPolicy` 的情况下调用 `action_trigger_chitchat` 时。

上下文属性在 `pattern_cannot_handle` 之外不可用。

### 10. 人工处理 {#10-human-handoff}

当用户请求连接到人工或对话机器人无法处理用户请求时，对话机器人可以转接对话。

**示例**：用户请求连接到人工。

<div class="md-chat"><div class="chat-container"><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">I want to be connected to a human agent.</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">I understand you want to be connected to a human agent, but that's something I cannot help you with at the moment. Is there something else I can help you with?</span></p></div></div><figcaption>人工处理</figcaption></div>

## 配置 {#configurations}

### 默认行为 {#default-behavior}

Rasa 为每个开箱即用的对话修复案例提供了一个默认行为。每个案例都通过一个模式来处理，该模式是专门为处理该案例而设计的特殊流：

- `pattern_continue_interrupted` 用于离题。
- `pattern_correction` 用于更正。
- `pattern_cancel_flow` 用于取消。
- `pattern_skip_question` 用于跳过收集步骤。
- `pattern_chitchat` 用于闲聊。
- `pattern_completed` 用于完成。
- `pattern_clarification` 用于澄清。
- `pattern_internal_error` 用于内部错误。
- `pattern_cannot_handle` 用于无法处理。
- `pattern_human_handoff` 用于人工处理。

这些流的语法与其他流相同。

!!! info "信息"

    对话修复案例应开箱即用。这意味着，如果[默认行为](#reference-default-pattern-configuration)对于对话机器人的用例来说已经足够好，那么对话机器人的项目目录中就不需要处理修复案例的模式所对应的流。

!!! info "信息"

    [上下文响应改写器](contextual-response-rephraser.md)可帮助模式中的默认响应自然地适应对话的上下文。

### 修改默认行为 {#modifying-default-behaviour}

可以通过创建与用于处理相应案例的模式同名的流（如 `pattern_correction`）来覆盖每个对话修复案例的默认行为。如果模式使用需要修改的默认动作，你可以通过实施新的自定义动作来覆盖默认动作的实施，并在流中使用该自定义动作。

!!! info "信息"

    确保修改完成后对话机器人得到重新训练。

!!! info "信息"

    由于大多数这些模式会中断另一个流，因此它们应该保持简短和简单。

### 示例配置 {#sample-configuration}

修改流结束时 Rasa 的响应：

```yaml title="flows.yml"
flows:
  pattern_completed:
    description: Completion of a user's flow
    steps:
      - action: utter_can_do_something_else
```

```yaml title="domain.yml"
responses:
  utter_can_do_something_else:
    - text: "Is there anything else I can assist you with?"
```

## 常见修改 {#common-modifications}

以下是对默认行为的一些常见修改。

### 要求确认 {#requiring-confirmation}

你可以修改更正的默认实现，并在更新槽之前要求用户确认，例如，这将导致如下对话：

<div class="md-chat"><div class="chat-container"><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">I want to send some money to Joe</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">How much money do you want to send?</span></p></div><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">50$</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">Do you want to send 50$ to Joe? (Yes/No)</span></p></div><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">Oh wait!! I meant to say to John, not Joe!</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">Do you want to update the recipient to John? (Yes/No)</span></p></div><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">Yes!</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">Updated recipient to John</span></p></div><div class="chat-output chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">Bot: </span><span class="content">Do you want to send 50$ to John? (Yes/No)</span></p></div><div class="chat-input chat-item stack-xs"><p class="chat-bubble"><span class="sr-only">User: </span><span class="content">...</span></p></div></div><figcaption>常见的纠正场景</figcaption></div>

为了实现上述确认，创建一个名为 `pattern_correction` 的流，其定义如下：

```yaml title="flows.yml"
flows:
  pattern_correction:
    description: Confirm a previous correction of a slot value.
    steps:
      - noop: true
        next:
          - if: context.is_reset_only
            then:
              - action: action_correct_flow_slot
                next: END
          - else: confirm_first
      - id: confirm_first
        collect: confirm_slot_correction
        next:
          - if: not slots.confirm_slot_correction
            then:
              - action: utter_not_corrected_previous_input
                next: END
          - else:
              - action: action_correct_flow_slot
              - action: utter_corrected_previous_input
                next: END
```

还要确保将使用的响应和槽添加到领域文件中：

```yaml title="domain.yml"
slots:
  confirm_slot_correction:
    type: bool

responses:
  utter_ask_confirm_slot_correction:
    - text: "Do you want to update the {{ context.corrected_slots.keys()|join(', ') }}?"
      buttons:
        - payload: "yes"
          title: "Yes"
        - payload: "no"
          title: "No, please keep the previous information"
      metadata:
        rephrase: True
        template: jinja

  utter_not_corrected_previous_input:
    - text: "Ok, I did not correct the previous input."
      metadata:
        rephrase: True
```

### 实现人工处理 {#implementing-a-human-handoff}

目前，人工处理的默认行为是通知用户对话机器人无法处理请求。然而，在有客户服务可用的情况下，实现人工处理变得很重要。你可以通过编写自定义动作并覆盖名为 `pattern_human_handoff` 的流来实现人工处理：

```yaml title="flows.yml"
flows:
  pattern_human_handoff:
    description: Human handoff implementation
    steps:
      - collect: confirm_human_handoff
        next:
          - if: slots.confirm_human_handoff
            then:
            - action: action_human_handoff
              next: END
          - else:
            - action: utter_human_handoff_cancelled
              next: END
```

还要确保将使用的动作、响应和槽添加到领域文件中：

```yaml title="domain.yml"
slots:
  confirm_human_handoff:
    type: bool
    mappings:
      - type: custom

actions:
  - action: action_human_handoff

responses:
  utter_ask_confirm_human_handoff:
    - text: "Do you want to be connected to a human agent?"
      buttons:
        - payload: "yes"
          title: "Yes"
        - payload: "no"
          title: "No"
  utter_human_handoff_cancelled:
    - text: "Ok, I understand you don't want to be connected to a human agent. Is there something else I can help you with?"
      metadata:
        rephrase: True
```

### 根据当前流做出回应 {#react-dependent-on-the-current-flow}

你可以根据被中断的流更改模式的行为。这可以通过在模式的 `if` 条件中使用 `context` 对象来实现

```yaml title="flows.yml"
flows:
  pattern_cancel_flow:
    description: A meta flow that's started when a flow is cancelled.

    steps:
      - id: decide_cancel_step
        noop:
          - if: context.canceled_name = "transfer money"
            then: inform_user
          - else: cancel_flow # skips the inform step
      - id: inform_user
        action: utter_flow_cancelled_rasa
        next: cancel_flow
      - id: cancel_flow
        action: action_cancel_flow
```

在上面的例子中，仅当中断的流名为 `transfer_money` 时，才使用 `inform_user` 步骤。

### 闲聊自由格式生成 {#free-form-generation-for-chitchat}

默认情况下，闲聊通过 `action_trigger_chitchat` 运行，该动作调用 [`IntentlessPolicy`](policies/intentless-policy.md) 来提供相关的预定义响应。

要切换到自由格式生成的响应，需通过创建名为 `pattern_chitchat` 的流来覆盖 `pattern_chitchat` 的默认行为，该流定义如下：

```yaml title="flows.yml" hl_lines="6"
flows:
  pattern_chitchat:
    description: handle interactions with the user that are not task-oriented
    name: pattern chitchat
    steps:
      - action: utter_free_chitchat_response
```

!!! warning "警告"

    将使用 LLM 生成自由格式的响应。对话机器人可能会回答超出预期领域的查询。

### 禁用闲聊 {#disabling-chitchat}

默认情况下，如果未配置 [Intentless 策略](policies/intentless-policy.md)，对话机器人将默认采用[无法处理](#9-cannot-handle)模式，通过通知用户无法处理请求来有效地限制闲聊。

要完全限制随意对话，需通过创建名为 `pattern_chitchat` 的流来覆盖 `pattern_chitchat` 的默认行为。与其使用触发 `action_trigger_chitchat` 的通常行为，不如将其配置为使用预定义响应：

```yaml title="flows.yml" hl_lines="8"
flows:
  pattern_chitchat:
    description: |
      Handle interactions with the user that
      are not task-oriented using a predefined response
    name: pattern chitchat
    steps:
      - action: utter_cannot_handle  # or any other response template
```

## 参考：默认模式配置 {#reference-default-pattern-configuration}

作为参考，以下是对话修复的完整默认配置：

```yaml title="默认模式"
version: "3.1"
responses:

  utter_ask_rephrase:
    - text: I’m sorry I am unable to understand you, could you please rephrase?

  utter_boolean_slot_rejection:
    - text: "Sorry, the value you provided, `{{value}}`, is not valid. Please respond with a valid value."
      metadata:
        rephrase: True
        template: jinja

  utter_can_do_something_else:
    - text: "What else can I help you with?"
      metadata:
        rephrase: True

  utter_cannot_handle:
    - text: I'm sorry, I'm not trained to help with that.

  utter_categorical_slot_rejection:
    - text: "Sorry, you responded with an invalid value - `{{value}}`. Please select one of the available options."
      metadata:
        rephrase: True
        template: jinja

  utter_clarification_options_rasa:
    - text: "I can help, but I need more information. Which of these would you like to do: {{context.clarification_options}}?"
      metadata:
        rephrase: True
        template: jinja

  utter_corrected_previous_input:
    - text: "Ok, I am updating {{ context.corrected_slots.keys()|join(', ') }} to {{ context.corrected_slots.values()|join(', ') }} respectively."
      metadata:
        rephrase: True
        template: jinja

  utter_float_slot_rejection:
    - text: "Sorry, it seems the value you provided `{{value}}` is not a valid number. Please provide a valid number in your response."
      metadata:
        rephrase: True
        template: jinja

  utter_flow_cancelled_rasa:
    - text: "Okay, stopping {{ context.canceled_name }}."
      metadata:
        rephrase: True
        template: jinja

  utter_flow_continue_interrupted:
    - text: "Let's continue with {{ context.previous_flow_name }}."
      metadata:
        rephrase: True
        template: jinja

  utter_free_chitchat_response:
    - text: "placeholder_this_utterance_needs_the_rephraser"
      metadata:
        rephrase: True
        rephrase_prompt: |
          You are an incredibly friendly assistant. Generate a short
          response to the user's comment in simple english.

          User: {{current_input}}
          Response:

  utter_human_handoff_not_available:
    - text: I understand you want to be connected to a human agent, but that's something I cannot help you with at the moment. Is there something else I can help you with?
      metadata:
        rephrase: True

  utter_inform_code_change:
    - text: There has been an update to my code. I need to wrap up our running dialogue and start from scratch.
      metadata:
        rephrase: True

  utter_internal_error_rasa:
    - text: Sorry, I am having trouble with that. Please try again in a few minutes.

  utter_no_knowledge_base:
    - text: I am afraid, I don't know the answer. At this point, I don't have access to a knowledge base.
      metadata:
        rephrase: True

  utter_skip_question_answer:
    - text: I'm here to provide you with the best assistance, and in order to do so, I kindly request that we complete this step together. Your input is essential for a seamless experience!
      metadata:
        rephrase: True

  utter_user_input_empty_error_rasa:
    - text: I see an empty message. What can I assist you with?

  utter_user_input_too_long_error_rasa:
    - text: I'm sorry, but your message is too long for me to process. Please keep your message concise and within {% if context.info.max_characters %}{{context.info.max_characters}} characters.{% else %}a reasonable length.{% endif %}
      metadata:
        template: jinja

slots:
  confirm_correction:
    type: bool

flows:
  pattern_cancel_flow:
    description: Conversation repair flow that starts when a flow is cancelled
    name: pattern_cancel_flow
    steps:
      - action: action_cancel_flow
      - action: utter_flow_cancelled_rasa

  pattern_cannot_handle:
    description: |
      Conversation repair flow for addressing failed command generation scenarios
    name: pattern cannot handle
    steps:
      - noop: true
        next:
          # chitchat fallback
          - if: "'{{context.reason}}' = 'cannot_handle_chitchat'"
            then:
              - action: utter_cannot_handle
                next: END
          # fallback for things that are not supported
          - if: "'{{context.reason}}' = 'cannot_handle_not_supported'"
            then:
              - action: utter_cannot_handle
                next: "END"
          # default
          - else:
              - action: utter_ask_rephrase
                next: "END"

  pattern_chitchat:
    description: Conversation repair flow for off-topic interactions that won't disrupt the main conversation
    name: pattern chitchat
    steps:
      - action: action_trigger_chitchat

  pattern_clarification:
    description: Conversation repair flow for handling ambiguous requests that could match multiple flows
    name: pattern clarification
    steps:
      - action: action_clarify_flows
      - action: utter_clarification_options_rasa

  pattern_code_change:
    description: Conversation repair flow for cleaning the stack after an assistant update
    name: pattern code change
    steps:
      - action: utter_inform_code_change
      - action: action_clean_stack

  pattern_collect_information:
    description: Flow for collecting information from users
    name: pattern collect information
    steps:
      - id: "start"
        action: action_run_slot_rejections
      - action: validate_{{context.collect}}
        next:
        - if: "slots.{{context.collect}} is not null"
          then: END
        - else: ask_collect
      - id: ask_collect
        action: "{{context.utter}}"
      - action: "{{context.collect_action}}"
      - action: action_listen
        next: start

  pattern_completed:
    description: Flow that asks if the user needs more help after completing their initiated use cases
    name: pattern completed
    steps:
      - action: utter_can_do_something_else

  pattern_continue_interrupted:
    description: Conversation repair flow for managing when users switch between different flows
    name: pattern continue interrupted
    steps:
      - action: utter_flow_continue_interrupted

  pattern_correction:
    description: Conversation repair flow for managing user input changes or error corrections
    name: pattern correction
    steps:
      - action: action_correct_flow_slot
        next:
          - if: not context.is_reset_only
            then:
              - action: utter_corrected_previous_input
                next: END
          - else: END

  pattern_human_handoff:
    description: Conversation repair flow for switching users to a human agent if their request can't be handled
    name: pattern human handoff
    steps:
      - action: utter_human_handoff_not_available

  pattern_internal_error:
    description: Conversation repair flow for informing users about internal errors
    name: pattern internal error
    steps:
      - noop: true
        next:
        - if: "'{{context.error_type}}' = 'rasa_internal_error_user_input_too_long'"
          then:
          - action: utter_user_input_too_long_error_rasa
            next: "END"
        - if: "'{{context.error_type}}' = 'rasa_internal_error_user_input_empty'"
          then:
            - action: utter_user_input_empty_error_rasa
              next: END
        - else:
            - action: utter_internal_error_rasa
              next: END


  pattern_restart:
    description: Flow for restarting the conversation
    name: pattern restart
    nlu_trigger:
      - intent: restart
    steps:
      - action: action_restart

  pattern_search:
    description: Flow for handling knowledge-based questions
    name: pattern search
    steps:
      - action: utter_no_knowledge_base
      # - action: action_trigger_search to use doc search policy if present

  pattern_session_start:
    description: Flow for starting the conversation
    name: pattern session start
    nlu_trigger:
      - intent: session_start
    steps:
      - action: action_session_start

  pattern_skip_question:
    description: Conversation repair flow for managing user intents to skip questions (steps)
    name: pattern skip question
    steps:
      - action: utter_skip_question_answer
```
