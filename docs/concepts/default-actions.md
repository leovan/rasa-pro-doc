# 默认动作

默认动作是对话管理器默认内置的动作。其中大部分动作都是根据特定对话情况自动预测的。你可以自定义这些动作来个性化对话机器人。

这些动作中的每一个都有默认行为，如以下部分所述。为了覆盖此默认行为，请编写一个[自定义动作](custom-actions.md)，其 `name()` 方法返回与默认动作相同的名称：

```python
class ActionRestart(Action):

  def name(self) -> Text:
      return "action_restart"

  async def run(
      self, dispatcher, tracker: Tracker, domain: Dict[Text, Any]
  ) -> List[Dict[Text, Any]]:

      # custom behavior

      return [...]
```

将此动作添加到领域文件的 `actions` 部分，以便对话机器人知道使用自定义定义而不是默认定义：

```json
actions:
  - action_restart
```

!!! warning "警告"

    将此动作添加到领域文件后，使用 `rasa train --force` 重新训练模型。否则，Rasa 将不知道已更改的任何内容，并且可能会跳过重新训练对话模型。

## action_listen {#action_listen}

此动作表示对话机器人不应执行任何动作并等待下一个用户输入。

## action_restart {#action_restart}

此动作将重置整个对话历史记录，包括对话期间设置的任何槽。

如果模型配置中包含 [RulePolicy](../nlu-based-assistants/rules.md)，则用户可以在对话中通过发送 `/restart` 消息来触发此动作。如果在领域中定义了 `utter_restart` 响应，此响应也会发送给用户。

## action_session_start {#action_session_start}

此动作将启动新的对话会话，其在以下情况下执行：

- 每次新对话开始时。
- 用户在领域的[会话配置](domain.md#session-configuration)中 `session_expiration_time` 参数定义的时间段内处于不活动状态后。
- 当用户在对话期间发送 `/session_start` 消息时。

此动作将重置对话追踪器，但默认情况下不会清除已设置的任何插槽。

### 自定义 {#customization}

会话启动动作的默认行为是获取所有现有槽并将其延续到下一个会话。假如你不想延续所有槽，而只想延续用户的姓名和电话号码。为此，你可以使用自定义动作覆盖 `action_session_start`，该动作可能如下所示：

```python
from typing import Any, Text, Dict, List
from rasa_sdk import Action, Tracker
from rasa_sdk.events import SlotSet, SessionStarted, ActionExecuted, EventType


class ActionSessionStart(Action):
    def name(self) -> Text:
        return "action_session_start"

    @staticmethod
    def fetch_slots(tracker: Tracker) -> List[EventType]:
        """Collect slots that contain the user's name and phone number."""

        slots = []
        for key in ("name", "phone_number"):
            value = tracker.get_slot(key)
            if value is not None:
                slots.append(SlotSet(key=key, value=value))
        return slots

    async def run(
      self, dispatcher, tracker: Tracker, domain: Dict[Text, Any]
    ) -> List[Dict[Text, Any]]:

        # the session should begin with a `session_started` event
        events = [SessionStarted()]

        # any slots that should be carried over should come after the
        # `session_started` event
        events.extend(self.fetch_slots(tracker))

        # an `action_listen` should be added at the end as a user message follows
        events.append(ActionExecuted("action_listen"))

        return events
```

如果你想要访问随触发会话启动的用户消息一起发送的元数据，你可以访问特殊槽 `session_started_metadata`：

```python
from typing import Any, Text, Dict, List
from rasa_sdk import Action, Tracker
from rasa_sdk.events import SessionStarted, ActionExecuted


class ActionSessionStart(Action):
    def name(self) -> Text:
        return "action_session_start"

    async def run(
      self, dispatcher, tracker: Tracker, domain: Dict[Text, Any]
    ) -> List[Dict[Text, Any]]:
        metadata = tracker.get_slot("session_started_metadata")

        # Do something with the metadata
        print(metadata)

        # the session should begin with a `session_started` event and an `action_listen`
        # as a user message follows
        return [SessionStarted(), ActionExecuted("action_listen")]
```

##action_default_fallback {#action_default_fallback}

此动作撤消最后一次用户与对话机器人的交互，并发送 `utter_default` 响应（如果已定义）。如果你启用了此[回退机制](../nlu-based-assistants/fallback-handoff.md)，则它会因动作预测置信度低而触发。

!!! note "注意"

    如果 `action_default_fallback` 是对话机器人预测并执行的下一个动作，这将导致 `UserUtteranceReverted` 事件，该事件将取消设置上一个用户回合中填充的槽。

## action_run_slot_rejections {#action_run_slot_rejections}

此动作运行直接在 `flows.yaml` 文件中编写的[槽验证](flows.md#slot-validation)规则。当对话机器人在流中向用户询问信息时，`action_run_slot_rejections` 将作为默认流 `pattern_collect_information` 中的一个步骤执行。如果动作将其中一个规则评估为 `True`，则它将重置最初请求的槽，并通过分派 `utter` 属性中指示的响应再次向用户询问槽。如果没有规则被评估为 `True`，则动作将保留填充槽的原始值，对话机器人将继续执行流中的下一步。

## action_trigger_search {#action_trigger_search}

!!! note "注意"

    此动作需要[企业搜索策略](policies/enterprise-search-policy.md)。

此动作可用于从任何[流](flows.md)、规则或故事触发企业搜索策略。它通过操纵对话堆栈框架来工作。此动作的结果是 LLM 生成的对用户的响应。企业搜索策略通过使用相关知识库文档、槽上下文和对话记录提示 LLM 来生成响应。

如果你有 [`out_of_scope` 意图](https://rasa.leovan.tech/fallback-handoff/#handling-out-of-scope-messages)，可以按照以下方式使用此动作：

```yaml
rules:
- rule: Out of scope
  steps:
  - intent: out_of_scope
  - action: action_trigger_search
```

如果你使用 [`FallbackClassifier`](../nlu-based-assistants/fallback-handoff.md)，可以按照以下方式使用此动作：

```yaml
rules:
- rule: Respond with a knowledge base search if user sends a message with low NLU confidence
  steps:
  - intent: nlu_fallback
  - action: action_trigger_search
```

## action_reset_routing {#action_reset_routing}

此动作是 [NLU 系统和 CALM 系统共存](../building-assistants/coexistence.md)所必需的。它充当默认动作 `action_restart` 的软版本：

- 它会重置所有未标记为持久性的槽（请参阅[此章节](domain.md#persistence-of-slots-during-coexistence)）。
- 它不会完全重置追踪器，而是隐藏基于 NLU 的系统策略的所有先前追踪器事件。这样，在 CALM 中发生的事情将不会显示在基于 NLU 的系统策略的追踪器中，但你仍然可以在 [`rasa inspect`](../command-line-interface.md#rasa-inspect) 等工具中查看完整的追踪器历史记录。如果事件没有被隐藏，则在推理过程中，基于 NLU 的系统策略的追踪器看起来与训练期间的样子不同。这会导致错误的预测。隐藏事件的一个例外是持久化槽的 `SlotSet` 事件（请参阅[此章节](domain.md#persistence-of-slots-during-coexistence)）。

此动作还会重置槽 [`route_session_to_calm`](../building-assistants/coexistence.md#adding-the-routing-slot-to-your-domain)，确保共存路由器在下一个传入用户消息时再次启用。这样，用户可以在一次会话中掌握多种技能。

## action_clean_stack {#action_clean_stack}

此动作在对话机器人更新后维护堆栈完整性方面起着至关重要的作用。

对话机器人更新是指对对话机器人代码库进行的修改或增强，通常是为了引入新功能、修复错误或提高性能。在这种情况下，对话机器人版本从 A 过渡到 B 表示正在向对话机器人部署更新，这可能包括对其行为、响应或底层功能的更改。当对话机器人更新发生在正在进行的对话期间时，它需要特殊处理以确保对话保持一致且不受更新的影响。

此动作通过将堆栈中的所有帧设置为结束步骤来实现这一点。目前，它在 `pattern_code_change` 流中使用。
