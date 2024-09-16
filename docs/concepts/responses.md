# 响应

响应是对话机器人发送给用户的消息。响应通常只是文本，但也可以包含图像和按钮等内容。

## 定义响应 {#defining-responses}

响应位于领域文件中的 `responses` 键下或单独的 `responses.yml` 文件中。每个响应名称都应以 `utter_` 开头。例如，你可以在响应名称 `utter_greet` 和 `utter_bye` 下添加问候和道别的响应：

```yaml title="domain.yml" hl_lines="4-8"
intents:
  - greet

responses:
  utter_greet:
  - text: "Hi there!"
  utter_bye:
  - text: "See you!"
```

如果你在对话机器人中使用[检索意图](../nlu-based-assistants/glossary.md#retrieval-intent)，那么你还需要为对话机器人对这些意图的回复添加响应：

```yaml hl_lines="5 6 8 9"
intents:
  - chitchat

responses:
  utter_chitchat/ask_name:
  - text: Oh yeah, I am called the retrieval bot.

  utter_chitchat/ask_weather:
  - text: Oh, it does look sunny right now in Berlin.
```

!!! info "注意"

    请注意检索意图的响应名称的特殊格式。每个名称都以 `utter_` 开头，后跟检索意图的名称（此处为 `chitchat`），最后是指定不同响应键的后缀（此处为 `ask_name` 和 `ask_weather`）。请参阅 [NLU 训练示例文档](../nlu-based-assistants/training-data-format.md#training-examples)以了解更多信息。

### 在回复中使用变量 {#using-variables-in-responses}

你可以使用变量将信息插入回复中。在回复中，变量括在花括号中。例如，请参阅下面的变量 `name`：

```yaml title="domain.yml" hl_lines="3"
responses:
  utter_greet:
  - text: "Hey, {name}. How are you?"
```

当使用 `utter_greet` 响应时，Rasa 会自动使用在名为 `name` 的槽中找到的值填充变量。如果这样的槽不存在或为空，则变量将填充 `None`。

填充变量的另一种方法是在[自定义动作](custom-actions.md)中。在自定义动作代码中，你可以向响应提供值以填充特定变量。如果你将 Rasa SDK 用于动作服务器，则可以将变量的值作为关键字参数传递给 [`dispatcher.utter_message`](../action-server/sdk-dispatcher.md)：

```python hl_lines="3"
dispatcher.utter_message(
    template="utter_greet",
    name="Sara"
)
```

如果你使用[不同的自定义动作服务器](../action-server.md#other-action-servers)，请通过向服务器返回的响应添加额外的参数来提供值：

```json
{
  "events":[
    ...
  ],
  "responses":[
    {
      "template":"utter_greet",
      "name":"Sara"
    }
  ]
}
```

### 响应变体 {#response-variations}

如果为给定的响应名称提供多种响应变体以供选择，则可以让对话机器人的回复更有趣：

```yaml title="domain.yml" hl_lines="3-4"
responses:
  utter_greet:
  - text: "Hey, {name}. How are you?"
  - text: "Hey, {name}. How is your day going?"
```

在此示例中，当预测 `utter_greet` 为下一个动作时，Rasa 将随机选择两个响应变体之一来使用。

#### 响应的 ID {#ids-for-responses}

!!! info "3.6 版本新特性"

    你现在可以为任何响应设置 ID。当你想使用 [NLG 服务器](../production/nlg.md)生成响应时，这很有用。

    ID 的类型为字符串。

带 ID 的响应变体示例：

```yaml title="domain.yml" hl_lines="3-4"
responses:
  utter_greet:
  - id: "greet_1"
    text: "Hey, {name}. How are you?"
  - id: "greet_2"
    text: "Hey, {name}. How is your day going?"
```

### 特定于频道的响应变体 {#channel-specific-response-variations}

要根据用户所连接的频道指定不同的响应变体，请使用特定于频道的响应变体。

在以下示例中，`channel` 键使第一个响应变体特定于 `slack` 频道，而第二个变体不是特定于频道的：

```yaml title="domain.yml" hl_lines="4"
responses:
  utter_ask_game:
  - text: "Which game would you like to play on Slack?"
    channel: "slack"
  - text: "Which game would you like to play?"
```

!!! info "注意"

    确保 `channel` 键的值与输入频道的 `name()` 方法返回的值匹配。如果你使用的是内置频道，则此值也将与 `credentials.yml` 文件中使用的频道名称匹配。

当对话机器人在给定的响应名称下寻找合适的响应变体时，它会首先尝试从当前频道的特定频道变体中进行选择。如果没有这样的变体，对话机器人将从任何非特定频道的响应变体中进行选择。

在上面的例子中，第二个响应变体没有指定 `channel`，对话机器人可以将其用于除 `slack` 之外的所有频道。

!!! warning "注意"

    对于每个响应，请尝试至少有一个不带 `channel` 键的响应变体。这可让对话机器人在所有环境中（例如在新频道中、在 shell 中和在交互式学习中）正确响应。

### 条件响应变体 {#conditional-response-variations}

还可以使用条件响应变体根据一个或多个槽选择特定响应变体。条件响应变体在领域或响应 YAML 文件中的定义类似于标准响应变体，但带有附加`condition` 键。此键指定槽 `name` 和 `value` 约束的列表。

在对话期间触发响应时，将根据当前对话状态检查每个条件响应变体的约束。如果所有约束槽值都等于当前对话状态的相应槽值，则响应变体可供对话机器人使用。

!!! info "注意"

    对话状态槽值和约束槽值的比较由相等运算符 `==` 执行，该运算符也要求槽值的类型匹配。例如，如果约束指定为 `value: true`，则槽需要用布尔值 `true` 填充，而不是字符串 `"true"`。

在下面的例子中，我们将定义一个具有一个约束的条件响应变体，即将 `logged_in` 插槽设置为 `true`：

```yaml title="domain.yml"
slots:
  logged_in:
    type: bool
    influence_conversation: False
    mappings:
    - type: custom
  name:
    type: text
    influence_conversation: False
    mappings:
    - type: custom

responses:
  utter_greet:
    - condition:
        - type: slot
          name: logged_in
          value: true
      text: "Hey, {name}. Nice to see you again! How are you?"

    - text: "Welcome. How is your day going?"
```

```yaml title="stories.yml"
stories:
- story: greet
  steps:
  - action: action_log_in
  - slot_was_set:
    - logged_in: true
  - intent: greet
  - action: utter_greet
```

在上面的例子中，每当执行 `utter_greet` 动作并且将 `logged_in` 槽设置为 `true` 时，将使用第一个响应变体（`"Hey, {name}. Nice to see you again! How are you?"`）。第二个变体没有条件，将被视为默认变体，并在 `logged_in` 不等于 `true` 时使用。

!!! warning "注意"

    强烈建议始终提供没有条件的默认响应变体，以防止没有条件响应与已填充槽匹配的情况。

在对话过程中，Rasa 将从所有满足约束的条件响应变体中进行选择。如果有多个符合条件的条件响应变体，Rasa 将随机选择一种。例如，考虑以下响应：

```yaml title="domain.yml"
responses:
  utter_greet:
    - condition:
        - type: slot
          name: logged_in
          value: true
      text: "Hey, {name}. Nice to see you again! How are you?"

    - condition:
        - type: slot
          name: eligible_for_upgrade
          value: true
      text: "Welcome, {name}. Did you know you are eligible for a free upgrade?"

    - text: "Welcome. How is your day going?"
```

如果 `logged_in` 和 `eligible_for_upgrade` 都设置为 `true`，那么第一和第二个响应变体都可以使用，并且对话机器人将以相同的概率选择它们。

你可以继续使用特定于频道的响应变体以及条件响应变体，如下例所示。

```yaml title="domain.yml"
slots:
  logged_in:
    type: bool
    influence_conversation: False
    mappings:
    - type: custom
  name:
    type: text
    influence_conversation: False
    mappings:
    - type: custom

responses:
  utter_greet:
    - condition:
        - type: slot
          name: logged_in
          value: true
      text: "Hey, {name}. Nice to see you again on Slack! How are you?"
      channel: slack

    - text: "Welcome. How is your day going?"
```

Rasa 将按以下顺序优先选择响应：

1. 具有匹配频道的条件响应变体。
2. 具有匹配频道的默认响应。
3. 没有匹配频道的条件响应变体。
4. 没有匹配频道的默认响应。

## 富响应 {#rich-responses}

你可以通过添加视觉和交互元素来丰富响应。许多频道都支持多种类型的元素：

### 按钮 {#buttons}

你可以在回复中添加按钮，以允许用户从选项列表中进行选择。按钮在聊天窗口中显示为可点击元素。

`buttons` 列表中的每个按钮都应具有两个键：

- `title`：用户看到的按钮上显示的文本。
- `payload`：单击按钮时用户发送给对话机器人的消息。

按钮有效负载可用于：

- [触发意图并将实体传递](responses.md#triggering-intents-or-passing-entities)给对话机器人。
- 发出[命令以设置槽](responses.md#issuing-set-slot-commands)。
- 将预定义的自由格式字符串消息传递给对话机器人。请注意，如果上述选项都不可行，则应使用此选项。

此外，按钮的优势在于可以跳过 NLU 管道，直接使用有效负载中定义的意图、实体或设置槽命令标注用户消息。

!!! warning "检查频道"

    请记住，如何显示定义的按钮取决于输出频道的实现。例如，某些频道对可以提供的按钮数量有限制。请查看概念 > 频道连接器下的频道文档，了解任何特定于频道的限制。

#### 触发意图或传递实体 {#triggering-intents-or-passing-entities}

以下是使用按钮触发意图的响应示例：

```yaml title="domain.yml" hl_lines="4-9"
responses:
  utter_greet:
  - text: "Hey! How are you?"
    buttons:
    - title: "great"
      payload: "/mood_great"
    - title: "super sad"
      payload: "/mood_sad"
```

如果你希望按钮也将实体传递给对话机器人：

```yaml title="domain.yml" hl_lines="4-9"
responses:
  utter_greet:
  - text: "Hey! Would you like to purchase motor or home insurance?"
    buttons:
    - title: "Motor insurance"
      payload: '/inform{{"insurance":"motor"}}'
    - title: "Home insurance"
      payload: '/inform{{"insurance":"home"}}'
```

传递多个实体也是可能的：

```txt
'/intent_name{{"entity_type_1":"entity_value_1", "entity_type_2": "entity_value_2"}}'
```

!!! info "使用按钮覆盖 NLU"

    你可以使用按钮覆盖 NLU 预测并触发特定意图和实体。

    以 `/` 开头的消息由 `RegexInterpreter` 处理，它需要以缩短的 `/intent{entities}` 格式输入 NLU。在上面的示例中，如果用户单击按钮，则用户输入将被分类为 `mood_great` 或 `mood_sad` 意图。

    你可以使用以下格式将要传递给 `RegexInterpreter` 的意图实体包括在内：

    `/inform{"ORG":"Rasa", "GPE":"Germany"}`

    `RegexInterpreter` 将使用意图 `inform` 对上述消息进行分类，并提取分别属于 `ORG` 和 `GPE` 类型的实体 `Rasa` 和 `Germany`。

!!! info "在 domain.yml 中转义花括号"

    你需要在 `domain.yml` 中用双花括号编写 `/intent{entities}` 简写响应，以便对话机器人不会将其视为[响应中的变量](responses.md#using-variables-in-responses)并插入花括号内的内容。

##### 发出设置槽命令 {#issuing-set-slot-commands}

!!! info "3.9.0 版本新特性"

    从 Rasa Pro 3.9.0 开始，你可以使用按钮发出命令来设置槽。

##### 有效负载语法 {#payload-syntax}

要发出 [`set slot` 命令](dialogue-understanding.md#set-slot)，你可以在有效负载中使用以下格式：`/SetSlots(slot_name=slot_value)`。你可以在同一个命令中定义多个槽键值对。请注意，每个命令最多只能有 10 个槽键值对，以防止正则表达式拒绝服务 (ReDoS) 攻击。

以下是示例：

```yaml title="domain.yml" hl_lines="4-8"
responses:
  utter_contactless_limit:
  - text: "Which card would you like to set the maximum contactless limit for?"
    buttons:
    - title: "credit"
      payload: "/SetSlots(amount=100, card_type=credit)"
    - title: "debit"
      payload: "/SetSlots(amount=100, card_type=debit)"
```

请注意，`SetSlots` 命令区分大小写，应完全按照上面所示书写。用于从有效负载中提取槽名称和值的正则表达式不允许使用以下字符：

- 槽名称中：`=`、`,`、`(`、`)`。
- 槽值中：`,`、`(`、`)`。

你还可以使用此语法通过首先设置槽，然后在该槽上分支以执行 [`link`](flows.md#link) 或 [`call`](flows.md#call) 步骤来启动流。

!!! warning "注意"

    除 `list` 槽外，所有槽类型都支持通过按钮填充。

##### 动态按钮 {#dynamic-buttons}

你还可以通过自定义动作在回复中创建动态按钮列表。响应列表可能来自 API，或者按钮列表是根据另一个槽的值或对话状态确定的。

这可以通过收集步骤和名为 `action_ask_{slot_name}` 的自定义动作来完成。

例如，假设对话机器人需要询问用户需要帮助处理哪张信用卡。我们将创建一个不带按钮的响应，然后使用自定义动作获取与用户关联的卡列表。

还有一个 `cards` 槽，其中包含所有用户卡的列表。这是在用户首次通过 `action_session_start` 连接到对话机器人时加载的。还有一些带有当前卡名和卡号的插槽。

```yaml title="domain.yml"
slots:
  current_card_name:
    type: text
  current_card_number:
    type: text
  cards:
    type: list

responses:
  utter_select_card:
    - text: "Here are the your cards, select the one you are referring to?"
```

`select_card` 流执行 `collect: current_card_name` 来向用户请求当前卡。

```yaml title="flow.yml"
flows:
  select_card:
    description: This flow is called when the user has multiple cards and needs to select one.
    name: Select card
    # block this flow from th list of possible flows for the LLM, it should only be called from other flows
    if: False
    steps:
      - collect: current_card_name
        ask_before_filling: true
        next: END
```

创建一个名为 `action_ask_current_card_name` 的自定义动作，流 `collect` 将调用该动作。

```python title="actions.py"
class ActionShowSlots(Action):
    def name(self):
        return "action_ask_current_card_name"

    def run(self, dispatcher, tracker, domain):
        events = []
        cards = tracker.get_slot("cards")
        if not cards:
            dispatcher.utter_message(text="No cards found.")
        else:
            buttons = []
            for card in cards:
                buttons.append(
                    {
                        "title": card.get("name"),
                        "payload": f"/SetSlots(current_card_name={card.get('name')}, current_card_number={card.get('number')})"
                    }
                )
            dispatcher.utter_message(response="utter_select_card", buttons=buttons)
        return events
```

你可以在[此处](flows.md#using-an-action-to-ask-for-information-in-collect-step)阅读有关 `action_ask_{slot_name}` 的更多信息。

### 图片 {#images}

你可以通过在 `image` 键下提供图片的 URL 将图片添加到响应中：

```yaml title="domain.yml" hl_lines="3"
  utter_cheer_up:
  - text: "Here is something to cheer you up:"
    image: "https://i.imgur.com/nGF1K8f.jpg"
```

### 自定义输出负载 {#custom-output-payloads}

你可以使用 `custom` 键将任意输出发送到输出频道。输出频道将接收存储在 `custom` 键下的对象作为 JSON 负载。

以下是如何将[日期选择器](https://api.slack.com/reference/block-kit/block-elements#datepicker)发送到 [Slack 输出频道](../connectors/slack.md)的示例：

```yaml title="domain.yml" hl_lines="3-14"
responses:
  utter_take_bet:
  - custom:
      blocks:
      - type: section
        text:
          text: "Make a bet on when the world will end:"
          type: mrkdwn
        accessory:
          type: datepicker
          initial_date: '2019-05-21'
          placeholder:
            type: plain_text
            text: Select a date
```

## 在对话中使用响应 {#using-responses-in-conversations}

### 将响应调用为动作 {#calling-responses-as-actions}

如果响应的名称以 `utter_` 开头，则响应可以直接用作动作，而无需列在领域的 `actions` 部分中。你可以将响应添加到领域：

```yaml title="domain.yml"
responses:
  utter_greet:
  - text: "Hey! How are you?"
```

你可以在故事中使用相同的响应作为动作：

```yaml title="stories.yml" hl_lines="5"
stories:
- story: greet user
  steps:
  - intent: greet
  - action: utter_greet
```

当 `utter_greet` 动作运行时，它会将响应中的消息发送回用户。

!!! info "更改回复"

    如果你想更改文本或回复的任何其他部分，则需要重新训练对话机器人，然后这些更改才会被接受。

### 从自定义动作调用响应 {#calling-responses-from-custom-actions}

你可以使用响应从自定义动作生成响应消息。如果你使用 Rasa SDK 作为动作服务器，则可以使用调度程序生成响应消息，例如：

```python title="actions.py" hl_lines="8"
from rasa_sdk.interfaces import Action

class ActionGreet(Action):
    def name(self):
        return 'action_greet'

    def run(self, dispatcher, tracker, domain):
        dispatcher.utter_message(template="utter_greet")
        return []
```

如果你使用[不同的自定义动作服务器](../action-server.md#other-action-servers)，服务器应该返回以下 JSON 来调用 `utter_greet` 响应：

```json hl_lines="5"
{
  "events": [],
  "responses": [
    {
      "template": "utter_greet"
    }
  ]
}
```
