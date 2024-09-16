# 验证槽值

本指南介绍了验证[流](../concepts/flows.md)中 `collect` 步骤后返回的槽值​​的不同方法。简单的验证可以在流步骤本身中定义，而需要外部数据的验证则由自定义动作执行。

## 验证电话号码 {#validating-a-phone-number}

假设对话机器人有一个流，允许用户更改与其帐户关联的电话号码。请注意，`collect` 步骤的 `description` 字段已经向命令生成器提供了有关所需格式的信息。

```yaml title="flows.yml"
flows:
  update_number:
    description: allows users to update the phone
      number associated with their account.
    steps:
      - collect: phone_number
        description: 'an entire US phone number, 
          including the area code. 
          For example: (415) 555-1234 . 
          When setting as a slot use this format, 
          with brackets and a hyphen, 
          even if the user didnt type it that way.'
```

```yaml title="domain.yml"
slots:
  phone_number:
    type: text

responses:
  utter_ask_phone_number:
    - text: "What is the new phone number?"
```

## 使用条件验证槽值的格式 {#validating-a-slot-values-format-with-a-condition}

拒绝是一种对提取的槽值进行简单验证的轻量级方法。在 `collect` 步骤中添加 `rejections` 字段，以验证 `LLMCommandGenerator` 提取的电话号码，并在其不匹配时拒绝该值。此示例使用正则表达式，你可以使用任何[条件](../concepts/conditions.md)。

```yaml title="flows.yml" hl_lines="13-15"
flows:
  update_number:
    description: allows users to update the phone
      number associated with their account.
    steps:
      - collect: phone_number
        description: 'an entire US phone number, 
          including the area code. 
          For example: (415) 555-1234 . 
          When setting as a slot use this format, 
          with brackets and a hyphen, 
          even if the user didnt type it that way.'
        rejections:
          - if: not ( slots.phone_number matches "^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$")
            utter: utter_invalid_phone_number
```

记得在领域中添加相应的 `response`，以通知用户其电话号码未成功收集。之后将再次要求用户输入其电话号码。

```yaml title="domain.yml"
responses:
  ...
  utter_invalid_phone_number:
    - text: "Sorry, I didn't get that. Please use the format (xxx) xxx-xxxx"
```

## 根据数据库或 API 验证槽值 {#validating-a-slot-value-against-a-database-or-api}

如果你需要更高级的逻辑来决定是否接受槽值，则可以使用自定义动作而不是拒绝。例如，如果你需要外部数据来了解槽值是否有效，则可以使用自定义动作。此示例检查是否已有另一个帐户与此电话号码关联。

为此，请将 `action` 步骤添加到流中，并使用以下条件来处理分支逻辑。首先，创建一个自定义动作来调用 API：

```python title="actions.py"
from typing import Any, Text, Dict, List
from rasa_sdk import Action, Tracker
from rasa_sdk.events import SlotSet
from rasa_sdk.executor import CollectingDispatcher

class ActionCheckPhoneNumberHasAccount(Action):
    def name(self) -> Text:
        return "action_check_phone_number_has_account"

    def run(self, dispatcher: CollectingDispatcher,
            tracker: Tracker,
            domain: Dict[Text, Any]) -> List[Dict[Text, Any]]:
        new_number = tracker.get_slot("phone_number")
        # Mock logic for deciding if a number is available.
        # This is where you would call an API.
        has_account = hash(new_number) % 2 == 0
        return [SlotSet("phone_number_has_account", has_account)]
```

此示例动作使用号码的哈希值来确定结果，以便你可以使用[固定物](../production/testing-your-assistant.md#fixtures-for-pre-filled-slots)编写确定性的端到端测试。

并将一个名为 `phone_number_has_account` 的布尔槽添加到领域中，以及在电话号码已与另一个帐户关联的情况下发送的响应。将自定义动作的名称添加到领域中：

```yaml title="domain.yml"
slots:
  ...
  phone_number_has_account:
    type: bool

actions:
  - action_check_phone_number_has_account

responses:
  ...
  utter_inform_phone_number_has_account:
    - text: "Unfortunately that number is already associated with an account."
```

现在，向流添加一个新步骤来调用此自定义动作。在此步骤的 `next` 字段中创建一个条件，以根据 `phone_number_has_account` 槽进行分支。如果该号码已经有帐户，对话机器人将通知用户，取消设置 `phone_number` 槽，并返回到表单的开头。为了跳回到表单的开头，请在流的第一步中添加一个“id”字段。流现在应该如下所示：

```yaml title="flows.yml" hl_lines="6 17-24"
flows:
  update_number:
    description: allows users to update the phone
      number associated with their account.
    steps:
      - id: "collect_phone_number"
        collect: phone_number
        description: 'an entire US phone number, 
          including the area code. 
          For example: (415) 555-1234 . 
          When setting as a slot use this format, 
          with brackets and a hyphen, 
          even if the user didnt type it that way.'
        rejections:
          - if: not ( slots.phone_number matches "^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$")
            utter: utter_invalid_phone_number
      - action: action_check_phone_number_has_account
        next:
          - if: slots.phone_number_has_account
            then:
              - action: utter_inform_phone_number_has_account
              - set_slots:
                  - phone_number: null
                next: "collect_phone_number"          
```
