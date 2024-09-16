# 使用流的业务逻辑

Rasa 中的流提供了一种设计对话驱动的业务逻辑的结构化方法。本页详细介绍了在 Rasa 中有效使用和管理流的规则。

!!! info "3.7 版本新特性"

    流是 Rasa 的新[语言模型 (CALM) 对话式 AI 方法](../calm.md)的一部分，从 `3.7.0` 版本开始可用。

## 概述 {#overview}

在 [CALM](../calm.md) 中，AI 对话机器人的业务逻辑被通过一组流实现。每个流都描述了 AI 对话机器人完成任务所使用的逻辑步骤。它描述了你需要从用户那里获取的信息、从 API 或数据库中检索的数据以及基于收集的信息的分支逻辑。

Rasa 中的流仅描述对话机器人遵循的逻辑，而不是对话可能采取的所有潜在路径。如果习惯通过创建对话流程来设计 AI 对话机器人，你会发现 Rasa 中的流要简单得多。查看 [Rasa Studio 的 Flow Builder](https://rasa.com/docs/studio/user-guide/flow-builder/introduction) 来使用 Web 界面构建流。

要熟悉流的工作原理，请按照[教程](../tutorial.md)进行。此页面提供了流格式和属性的参考。

## Hello World {#hello-world}

流使用 YAML 语法定义。以下是一个简单流的示例：

```yaml title="flows.yml"
flows:
  hello_world:
    description: A simple flow that greets the user
    steps:
      - action: utter_greet
```

## 流属性 {#flow-properties}

流使用以下属性定义：

```yaml title="flows.yml"
flows:
  a_flow: # required id
    name: "A flow" # optional name
    description: "required description of what the flow does"
    always_include_in_prompt: false # optional boolean, defaults to false
    if: "condition" # optional flow guard
    nlu_trigger: # optional list of intents that can start a flow
     - intent: "starting_flow_intent"
    steps: [] # required list of steps
```

### 流 ID {#flow-id}

ID 是必需的，用于在所有流中流的唯一标识。它只允许使用字母、数字、下划线和连字符，但第一个字符不能是连字符。

### 名称 {#name}

`name` 字段是可选的人类可读的名称。

### 描述 {#description}

`description` 字段是流的摘要。它是必需的，应该描述流为用户做了什么。编写清晰的描述很重要，因为[对话理解](dialogue-understanding.md)组件会使用它来决定何时启动此流。有关更多详细信息，请参阅[启动流](starting-flows.md)。此外，有关如何编写简洁明了的描述的指南，请参阅[此处](components/llm-command-generators.md#customizing-the-prompt)提供的部分。

### 总是包含在提示中 {#always-include-in-prompt}

如果 `always_include_in_prompt` 字段设置为 `true`，并且 `if` 字段中定义的[流守卫](starting-flows.md#flow-guards)计算结果为 `true`，则该流将[始终包含在提示中](components/llm-command-generators.md#retrieving-relevant-flows)。

### NLU 触发器属性 {#nlu-trigger-property}

`nlu_trigger` 字段用于添加[可以启动流的意图](starting-flows.md#nlu-trigger)。

### If 属性 {#if-property}

If 属性用于添加[流守卫](starting-flows.md#flow-guards)。

### 步骤 {#steps}

`steps` 字段是必需的，并列出了流的步骤。有六种类型的步骤：

- [动作步骤](flows.md#action)：流运行的自定义动作或话语动作。
- [收集步骤](flows.md#collect)：向用户提出问题来填充槽。
- [调用步骤](flows.md#call)：调用另一个流的步骤。
- [链接步骤](flows.md#link)：在此流完成后链接另一个流的步骤。
- [设置槽步骤](flows.md#set-slots)：设置槽的步骤。
- [无动作步骤](flows.md#noop)：创建条件的步骤，而不必在该步骤本身中运行动作、发出响应、收集槽、链接或调用另一个流。

所有步骤都具有以下共同属性：

```yaml
- id: "an_optional_unique_id"
  description: "an optional description, used to help dialogue understanding extract slots correctly"
  next: "an id or conditions to determine which step should be executed next"
```

#### ID 属性 {#id-property}

`id` 字段是每个步骤的可选唯一标识符。它用于在其他步骤的 `next` 字段中引用该步骤。

#### Next 属性 {#next-property}

`next` 字段指定在此步骤之后执行哪个步骤。如果省略，则列表中的下一个步骤则为执行的下一个步骤。

要链接特定步骤，你可以使用下一步的 `id`：

```yaml
- collect: name
  next: the_next_step
- id: the_next_step
  collect: age
```

还可以通过在 `next` 字段下列出步骤来创建嵌套结构：

```yaml
- collect: name
  next:
    - collect: age
```

这有时更容易阅读，特别是在使用条件创建分支逻辑时。例如：

```yaml
- collect: age
  next:
    - if: slots.age < 18
      then:
        - action: utter_not_old_enough
          next: END
    - if: slots.age >= 18 and slots.age < 65
      then: 18_to_65_step
    - else: over_65_step
```

根据用户输入的年龄值，此示例显示流执行到不同的步骤。请注意，`age` 槽必须以 `slots` 为前缀。命名空间才能在条件中访问。对于命名空间，请导航到[命名空间](conditions.md#namespaces)部分。

当满足导致 `next: END` 的条件时，流将完成，无需执行进一步的步骤。

## 步骤类型 {#step-types}

流中的每个步骤都有一个类型。类型决定了步骤的作用以及它支持的属性。步骤必须具有以下键之一：

- `action`
- `collect`
- `link`
- `call`
- `set_slots`
- `noop`

### Action {#action}

带有 `action: action_do_something` 键值的步骤指示对话机器人执行 `action_do_something`，然后继续下一步。它不会等待用户输入。

`action` 键的值可以是自定义动作的名称：

```yaml
- action: action_check_sufficient_funds
```

或者在领域中定义的响应的名称：

```yaml
- action: utter_insufficient_funds
```

### Collect {#collect}

带有 `collect` 键的步骤指示对话机器人向用户请求信息以填充槽。例如：

```yaml
- collect: account_type # slot name
```

在填写完 `account_type` 槽之前，对话机器人不会继续执行流中的下一步。有关槽填充和在领域文件中定义槽的更多信息，请参阅[教程](../tutorial.md#collecting-information-in-slots)。

除了响应之外，还可以定义一个名为 `action_ask_recipient` 的[自定义动作](custom-actions.md)来向用户提出问题。例如，向用户提供一些按钮。确保将自定义动作添加到领域文件中。

```yaml title="domain.yml"
actions:
  - action_ask_recipient
```

!!! info "信息"

    你可以为 `collect` 步骤定义 `response` 或 `custom action`。无法同时定义两者。如果同时定义了两者，Rasa 将抛出验证错误。

#### 始终提问 {#always-asking-questions}

默认情况下，如果相应的位置已填满，则跳过 `collect` 步骤。如果无论潜在槽是否已填满，都应始终询问 `collect` 步骤，你可以在 `collect` 步骤上设置 `ask_before_filling: true`：

```yaml
- collect: final_confirmation # slot name
  ask_before_filling: true
```

如果“final_confirmation”槽已填满，对话机器人将清除该槽并继续询问 `collect` 步骤，然后再次填充槽。这确保始终会询问确认 `collect` 步骤。

#### 在流结束时重置槽 {#resetting-slots-at-the-end-of-a-flow}

默认情况下，当流完成时，所有通过 `collect` 步骤填充的槽都会被重置。流结束后，槽的值要么被重置为 null，要么被重置为槽的[初始值](domain.md#initial-slot-values)（如果在 `slots` 下的领域文件中有提供）。

如果想在流完成后保留槽的值，请将 `reset_after_flow_ends` 属性设置为 `false`。例如：

```yaml
- collect: user_name
  reset_after_flow_ends: false
```

在[共存模式](../building-assistants/coexistence.md)下，必须通过使用属性 [`shared_for_coexistence: True`](domain.md#persistence-of-slots-during-coexistence) 注释槽定义来强制执行此行为。

#### 使用不同的响应键执行 `collect` 步骤 {#using-a-different-response-key-for-the-collect-step}

默认情况下，Rasa 将查找名为 `utter_ask_{slot_name}` 的响应来执行 `collect` 步骤。可以通过向步骤添加 `utter` 属性来使用具有不同键的[响应](responses.md)。

例如：

```yaml title="flows.yml"
flows:
  replace_supplementary_card:
    description: This flow helps the user to replace a supplementary card for a family member.
    steps:
      - collect: account_type
        utter: utter_ask_secondary_account_type
        next: ask_account_number
```

#### 使用动作在 `collect` 步骤中询问信息 {#using-an-action-to-ask-for-information-in-collect-step}

除了使用模板响应进行 `collect` 步骤之外，还可以使用[自定义动作](custom-actions.md)。例如，如果你想要将从数据库中提取的槽的可用值显示为按钮，这将非常有用。Rasa 将寻找名为 `action_ask_{slot_name}` 的动作来执行 `collect` 步骤。自定义动作需要严格遵循指定的命名约定。确保将自定义动作 `action_ask_{slot_name}` 添加到领域文件中。

```yaml title="domain.yml"
actions:
  - action_ask_{slot_name}
```

无需更新 `collect` 步骤本身即可使用自定义动作。流内的 `collect` 步骤保持不变，例如：

```yaml
- collect: {slot_name}
```

!!! info "信息"

    你可以为 `collect` 步骤定义 `response` 或 `custom action`。无法同时定义两者。如果同时定义了两者，Rasa 将抛出验证错误。

#### 插槽验证 {#slot-validation}

可以通过向任何 `collect` 步骤添加 `rejections` 属性，直接在流 yaml 文件中定义槽验证规则。`rejections` 部分是一个映射列表。每个映射必须具有 `if` 和 `utter` 强制属性：

- `if` 属性是用自然语言编写并使用 [pypred](https://github.com/armon/pypred) 库进行评估的条件。在条件中，只能使用此步骤中收集的槽名称。目前不支持使用其他槽。
- `utter` 属性是如果条件评估为 `True`，对话机器人将发送的[响应](responses.md)的名称。

以下是一个例子：

```yaml title="flows.yml"
flows:
  verify_eligibility:
    description: This flow verifies if the user is eligible for creating an account.
    steps:
      - collect: age
        rejections:
          - if: slots.age < 1
            utter: utter_invalid_age
          - if: slots.age < 18
            utter: utter_age_at_least_18
        next: ask_email
```

当验证失败时，对话机器人将自动尝试再次收集信息。对话机器人将反复询问槽，直到值不被拒绝。

如果 `if` 属性中定义的谓词无效，对话机器人将记录错误并使用 `utter_internal_error_rasa` 进行响应，默认情况下为：Sorry, I'm having trouble understanding you right now. Please try again later。

你可以通过在领域文件中将此响应与自定义文本一起添加来覆盖 `utter_internal_error_rasa` 的文本消息。

!!! note "注意"

    对于更复杂的验证逻辑，你还可以在[自定义动作](custom-actions.md)中定义槽验证。请注意，此自定义动作必须遵循以下命名约定：`validate_{slot_name}`。

### Link {#link}

`link` 步骤用于链接流。链接只能用作流中的最后一步。到达 `link` 步骤时，当前流结束并启动目标流。你可以将 `link` 步骤定义为：

```yaml
- link: "id_of_another_flow"
```

无法在 `link` 步骤中添加任何其他属性（例如 `action` 或 `next`）。这样做会导致流验证期间出错。

### Call {#call}

!!! info "3.8 版本新特性"

    `call` 步骤从 Rasa Pro `3.8.0` 开始可用。

你可以在流（父流）中使用 `call` 步骤来嵌入另一个流（子流）。当执行到达 `call` 步骤时，CALM 会启动子流。子流完成后，将继续执行父流。

调用其他流有助于拆分和复用功能。它允许你定义一次流并在多个其他流中使用它。长流可以拆分成较小的流，使其更易于理解和维护。

你可以将 `call` 步骤定义为：

```yaml title="flows.yml" hl_lines="4"
flows:
  parent_flow:
    steps:
      - call: child_flow

  child_flow:
    steps:
      - action: utter_child_flow
```

`call` 步骤必须引用现有流，并且子流可以在与包含父流的同一文件中定义，也可以在不同的文件中定义。

#### `call` 步骤的行为 {#behavior-of-the-call-step}

`call` 步骤的设计旨在使子流表现得像父流的一部分。因此，以下属性是现成的：

1. 如果子流在任何步骤中具有 `next: END`，则控制权将传递回父流，并执行下一个逻辑步骤。
2. 子流的槽可以预先填充，甚至在启动子流之前。子流中的收集 `collect` 表现得就像它们是父流的直接一部分一样。
3. 子流的槽一旦被填充，并且控制权仍在子流或父流内，就可以更正。因此，即使子流完成后，只要父流仍处于活动状态，就可以更正槽。
4. 激活子流时，不会重置父流的槽。子流可以访问和修改父流的槽。
5. 控制权返回父流时，不会重置子流的槽。相反，一旦父流结束，它们将与父流的槽一起重置（除非任何槽的 `reset_after_flow_ends` 设置为 `False`）。
6. 如果在子流内触发 `CancelFlow()` 命令，则父流也会被取消。

!!! info "注意"

    为了防止子流被用户消息直接触发，可以向子流添加[流守卫](starting-flows.md#flow-guards)：

    ```yaml title="flows.yml" hl_lines="7"
    flows:
      parent_flow:
        steps:
          - call: child_flow
      
      child_flow:
        if: False
        steps:
          - action: utter_child_flow
    ```

    在上面的例子中，`child_flow` 永远不会被用户消息直接触发，而只有当父流调用它时才会触发。

#### `call` 步骤的限制 {#constraints-on-the-call-step}

调用其他流在设计上有以下限制：

- 子流不能使用 `link` 步骤，因为 `link` 步骤旨在始终终止前一个流，这与 `call` 步骤的行为相矛盾，在 `call` 步骤中，控制权应始终传递给父流。
- 模式不能使用 `call` 步骤。

!!! info "关于使用 `link` 与 `call` 的建议"

    如果用例要求将一个流作为另一个流的后续流来启动，那么 `link` 步骤更适合完成两个流之间的连接。但是，如果一个流需要表现得像另一个更大流的一部分，并且子流完成后需要在更大流内执行更多步骤，则应使用 `call` 步骤。

### Set Slots {#set-slots}

`set_slots` 步骤用于设置一个或多个槽。例如：

```yaml
- set_slots:
    - account_type: "savings"
    - account_number: "123456789"
```

通常，槽要么通过用户输入设置（使用 `collect` 步骤），要么通过从另一个系统获取信息设置（使用自定义动作）。

`set_slot` 在取消设置槽值时很有用。例如，如果用户要求转账的金额超过其账户中的可用金额，你可能需要通知他们。只需重置 `amount` 槽，而不是结束流。

```yaml title="flows.yml"
flows:
  transfer_money:
    description: This flow lets users send money to friends and family.
    steps:
      - collect: recipient
      - id: ask_amount
        collect: amount
        description: the number of US dollars to send
      - action: action_check_sufficient_funds
        next:
          - if: not has_sufficient_funds
            then:
              - action: utter_insufficient_funds
              - set_slots:
                  - amount: null
                next: ask_amount
          - else: final_confirmation
```

### Noop {#noop}

`noop` 步骤可以与 `next` 属性相结合，在流中创建[条件](conditions.md#conditions)分支，而不必在该步骤本身中运行动作、发出响应、填充槽、链接或调用另一个流。

例如：

```yaml hl_lines="5"
flows:
  change_address:
    description: Allow a user to change their address.
    steps:
      - noop: true
        next:
          - if: not slots.authenticated
            then:
              - call: authenticate_user
                next: ask_new_address
          - else: ask_new_address
      - id: ask_new_address
        collect: address
        ...
```

始终需要将 `next` 属性添加到 `noop` 步骤。否则，在训练期间流验证将失败。

## 示例 {#examples}

### 带分支的基本流 {#a-basic-flow-with-branching}

```yaml
flows:
  basic_flow_with_branching:
    description: a flow with a branch
    steps:
      - action: utter_greet
      - collect: age
        next:
          - if: age < 18
            then: too_young_step
          - else: old_enough_step
      - id: too_young_step
        action: utter_too_young
      - id: old_enough_step
        action: utter_old_enough
```

该示例显示了根据从用户消息收集的 `age` 槽分支的流。

## 链接多个流 {#linking-multiple-flows}

```yaml hl_lines="9"
flows:
  transfer_money:
    description: Send money to another individual
    steps:
      - collect: recipient_name
      - collect: amount_of_money
      - action: execute_transfer
      - action: utter_transfer_complete
      - link: collect_feedback

  collect_feedback:
    description: Collect feedback from user on their experience talking to the assistant
    steps:
      - collect: ask_rating
      - action: utter_thankyou
```

此示例演示了流中的 `link` 步骤如何使另一个流作为后续流启动。具体而言，在 `link` 步骤中终止 `transfer_money` 后，将启动 `collect_feedback` 流作为后续流。

## 将一个流嵌入另一个流 {#embedding-a-flow-inside-another-flow}

假设财务对话机器人需要满足两个用例：

- 向另一个人转账。
- 添加新的转账收款人。

与对话机器人交谈的最终用户可能想要向现有收款人发起汇款，因此不需要第二个用例。但是，用户也可能想要向新收款人汇款，在这种情况下，需要将这两个用例结合起来。这正是 `call` 步骤让你实现这两种可能性而无需创建冗余流的地方。要实现上述用例，你可以在流中利用 `call` 步骤，如下所示：

```yaml title="flows.yml" hl_lines="16 27 30"
flows:
  collect_recipient_details:
    description: Details of a money transfer recipient should be collected here
    if: False
    steps:
      - collect: recipient_name
        description: Name of a recipient who should be sent money
      - collect: recipient_iban
        description: IBAN of a recipient who should be sent money.
      - collect: recipient_phone_number
        description: Phone number of a recipient who should be sent money.

  add_recipient:
    description: User wants to add a new recipient for transferring money
    steps:
      - call: collect_recipient_details
      - action: add_new_recipient

  transfer_money:
    description: User wants to transfer money to a new or existing recipient
    steps:
      - action: show_existing_recipients
      - collect: need_new_recipient
        next:
          - if: slots.need_new_recipient
            then:
              - call: add_recipient
                next: get_confirmation
          - else:
              - call: collect_recipient_details
                next: get_confirmation
      - id: get_confirmation
        collect: transfer_confirmation
        ask_before_filling: true
      - action: execute_transfer
```

对于上述流结构，LLM 的提示将包含以下流定义：

```txt title="prompt.txt"
add_recipient: User wants to add a new recipient to transfer money
    slot: recipient_name (Name of a recipient who should be sent money)
    slot: recipient_iban (IBAN of a recipient who should be sent money)
    slot: recipient_phone_number (Phone number of a recipient who should be sent money)

transfer_money: User wants to transfer money to a new or existing recipient
    slot: need_new_recipient ((True/False))
    slot: recipient_name (Name of a recipient who should be sent money)
    slot: recipient_iban (IBAN of a recipient who should be sent money)
    slot: recipient_phone_number (Phone number of a recipient who should be sent money)
    slot: transfer_confirmation ((True/False))
```

上述每个流都非常独立，可以实现单一目的，并可有效重复使用以在单个对话中完成多个用例的组合。这增强了流的模块化和可维护性。另外，请注意，就捕获的信息而言，子流的槽不会与其父流的槽重叠。这对于命令生成器的 LLM 在填充此类槽时不会出现困惑非常重要。

### 选择性地询问信息 {#optionally-ask-for-information}

想象一下，一个服装购物 AI 对话机器人被创建来帮助用户寻找和购买服装。这个 AI 对话机器人包括两种类型的特征：主要特征（即服装类型、尺寸）和可选特征（即颜色、材质、价格范围等）。对话机器人应该询问主要特征，避免询问可选特征，除非用户明确要求。这种方法是避免询问太多问题和让用户不知所措的关键。

你可以通过在 `collect` 步骤中将 `ask_before_filling` 属性设置为 `false` 并为领域文件中的槽设置一个 `initial_value` 来实现这一点。

```yaml hl_lines="9 11"
flows:
  purchase_dress:
    name: purchase dress
    description: present options to the user and purchase the dress
    steps:
      - collect: dress_type
      - collect: dress_size
      - collect: dress_color
        ask_before_filling: false
      - collect: dress_material
        ask_before_filling: false
        next:
          - if: slots.dress_material = "cotton"
            then:
              - action: action_present_cotton_dresses
                next: END
          - if: slots.dress_material = "silk"
            then:
              - action: action_present_silk_dresses
                next: END
          - else:
              - action: action_present_all_dresses
                next: END
```

```yaml title="domain.yaml" hl_lines="20 25"
slots:
  dress_type:
    type: categorical
    values:
      - "shirt"
      - "pants"
      - "jacket"
    mappings:
      - type: custom
  dress_size:
    type: categorical
    values:
      - "small"
      - "medium"
      - "large"
    mappings:
      - type: custom
  dress_color:
    type: text
    initial_value: "unspecified"
    mappings:
      - type: custom
  dress_material:
    type: categorical
    initial_value: "any"
    values:
      - "cotton"
      - "silk"
      - "polyester"
      - "any"
    mappings:
      - type: custom
```

如果 `ask_before_filling` 设置为 `false`，并且假设槽（`dress_color` 和 `dress_material`）已经有初始值，则不会询问相应的 `collect` 步骤。相反，对话机器人将转到下一步。如果用户明确为槽提供了一个值，则这个新值将覆盖初始值。
