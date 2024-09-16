# Rasa Pro 中的事件

本页概述了 Rasa Pro 在对话过程中发出的不同类型的事件。

Rasa Pro 中的每个对话都代表一系列事件。事件用于跟踪用户消息和对话机器人响应，以及 Rasa Pro 在对话期间采取的动作的副作用，例如推进流或设置槽。Rasa Pro 在对话的不同阶段发出事件，它们可用于通过查询 Rasa Pro [分析数据管道](../operating/analytics/getting-started-with-analytics.md)或运行[端到端测试](../testing/e2e-testing-assertions/assertions-introduction.md)来分析和改进对话机器人。

事件存储在 Rasa Pro [追踪器存储](../production/tracker-stores.md)中，可以通过 Rasa Pro [HTTP API](../pages/http-api.md) 进行访问。

## 事件类型 {#event-types}

Rasa Pro 在整个对话过程中发出不同类型的事件，其中包括：

- [`UserUttered`](#user-event)：用户发送的消息的事件。
- [`BotUttered`](#bot-event)：对话机器人发送的消息的事件。
- [`ActionExecuted`](#action-event)：对话机器人采取的行动的事件。
- [`SlotSet`](#slot-event)：正在设置的槽的事件。
- [`AllSlotsReset`](#reset-slots-event)：重置所有槽的事件。
- [`SessionStarted`](#reset-slots-event)：新对话会话开始的事件。
- [`Restarted`](#restarted-event)：对话会话重置的事件。
- [`FlowStarted`](#flow-started-event)：流开始的事件。
- [`FlowCompleted`](#flow-completed-event)：流完成的事件。
- [`FlowCancelled`](#flow-cancelled-event)：取消流的事件。
- [`FlowInterrupted`](#flow-interrupted-event)：流中断的事件。
- [`FlowResumed`](#flow-resumed-event)：恢复流的事件。
- [`RoutingSessionEnded`](#routing-session-ended-event)：标记共存路由会话结束的事件。
- [`DialogueStackUpdated`](#stack-event)：对话堆栈更​​新的事件。
- [`ReminderScheduled`](#reminder-event)：提醒安排的事件。
- [`ReminderCancelled`](#cancel-reminder-event)：取消提醒的事件。
- [`ConversationPaused`](#pause-event)：对话暂停的事件。
- [`ConversationResumed`](#resume-event)：对话恢复的事件。
- [`ActionExecutionRejected`](#action-execution-rejected-event)：拒绝执行动作的事件。
- [`EntitiesAdded`](#entities-event)：将实体添加到对话状态的事件。
- [`UserUtteranceReverted`](#rewind-event)：恢复最近用户消息之后发生的每个事件的事件。
- [`ActionReverted`](#undo-event)：恢复对话机器人机器人的最后一个动作的事件。
- [`StoryExported`](#export-story-event)：将训练数据故事导出到文件的事件。
- [`FollowupAction`](#followup-action-event)：后续动作排队的事件。
- [`ActiveLoop`](#active-loop-event)：激活表单的事件。
- [`LoopInterrupted`](#loop-interrupted-event)：表单中断的事件。

所有事件都具有以下属性：

- 类型名称：事件的类型。
- 时间戳：事件的时间戳。
- 元数据：有关事件的其他元数据。

### 用户事件 {#user-event}

`UserUttered` 事件表示用户发送的消息，其类型名称为 `user`。它包含以下附加属性：

- `text`：用户消息的文本。
- `intent`：用户消息的意图（如果适用）。
- `entities`：从用户消息中提取的实体（如果适用）。
- `parse_data`：用户消息的解析后的 NLU 数据，包括意图、实体和命令。
- `input_channel`：接收用户消息的频道。
- `message_id`：用户消息的唯一标识符。

### 对话机器人事件 {#bot-event}

`BotUttered` 事件表示对话机器人发送的消息，其类型名称为 `bot`。它包含以下附加属性：

- `text`：对话机器人消息的文本。
- `data`：更复杂的对话机器人话语的附加数据（例如按钮）

### 槽事件 {#slot-event}

`SlotSet` 事件表示填充槽，其类型名称为 `slot`。它包含以下附加属性：

- `name`：槽的名称。
- `value`：槽的值。

### 重置槽事件 {#reset-slots-event}

`AllSlotsReset` 事件表示重置所有槽，其类型名称为 `reset_slots`。它不包含任何附加属性。

### 实体事件 {#entities-event}

`EntitiesAdded` 事件表示向对话状态添加实体，其类型名称为 `entities`。它包含以下附加属性：

- `entities`：添加到对话状态的实体列表。

### 动作事件 {#action-event}

`ActionExecuted` 事件表示执行动作，其类型名称为 `action`。它包含以下附加属性：

- `name`：动作的名称。
- `policy`：用于预测动作的策略。
- `confidence`：动作预测的置信度。

### 后续动作事件 {#followup-action-event}

`FollowupAction` 事件表示将后续动作排入队列，其类型名称为 `followup`。它包含以下附加属性：

- `name`：要运行的后续动作的名称。

### 动作执行被拒绝事件 {#action-execution-rejected-event}

`ActionExecutionRejected` 事件表示动作执行被拒绝，其类型名称为 `action_execution_rejected`。它包含以下附加属性：

- `name`：被拒绝的动作的名称。
- `policy`：用于预测动作的策略。
- `confidence`：动作预测的置信度。

### 会话开始事件 {#session-started-event}

`SessionStarted` 事件表示新对话会话的开始，其类型名称为 `session_started`。它不包含任何其他属性。

### 重新启动事件 {#restarted-event}

`Restarted` 事件表示对话会话的重置，其类型名称为 `restart`。它不包含任何其他属性。

### 堆栈事件 {#stack-event}

`DialogueStackUpdated` 事件表示对话堆栈的更新，其类型名称为 `stack`。它包含以下附加属性：

- `update`：以字符串形式转储的 JsonPatch 对象。

### 路由会话结束事件 {#routing-session-ended-event}

`RoutingSessionEnded` 事件表示共存路由会话的结束，其类型名称为 `routing_session_ended`。它不包含任何其他属性。

### 流程开始事件 {#flow-started-event}

`FlowStarted` 事件表示流开始，其类型名称为 `flow_started`。它包含以下附加属性：

- `flow_id`：流的 ID。

### 流程中断事件 {#flow-interrupted-event}

`FlowInterrupted` 事件表示流中断，其类型名称为 `flow_interrupted`。它包含以下附加属性：

- `flow_id`：流的 ID。
- `step_id`：流中断步骤的 ID。

### 流程恢复事件 {#flow-resumed-event}

`FlowResumed` 事件表示流恢复，其类型名称为 `flow_resumed`。它包含以下附加属性：

- `flow_id`：流的 ID。
- `step_id`：流恢复步骤的 ID。

### 流程完成事件 {#flow-completed-event}

`FlowCompleted` 事件表示流完成，其类型名称为 `flow_completed`。它包含以下附加属性：

- `flow_id`：流的 ID。
- `step_id`：流完成步骤的 ID。

### 流程取消事件 {#flow-cancelled-event}

`FlowCancelled` 事件表示流取消，其类型名称为 `flow_cancelled`。它包含以下附加属性：

- `flow_id`：流的 ID。
- `step_id`：流取消步骤的 ID。

### 提醒事件 {#reminder-event}

`ReminderScheduled` 事件表示安排提醒，其类型名称为 `reminder`。它包含以下附加属性：

- `intent`：要触发的意图的名称。
- `entities`：触发意图时要使用的实体。
- `date_time`：应触发提醒的日期和时间。
- `name`：提醒的名称。
- `kill_on_user_message`：如果用户在触发日期之前发送消息，是否应取消提醒。

### 取消提醒事件 {#cancel-reminder-event}

`ReminderCancelled` 事件表示取消提醒，其类型名称为 `cancel_reminder`。它包含以下附加属性：

- `name`：要取消的提醒的名称。
- `intent`：用于识别要取消的提醒的意图的名称。
- `entities`：用于识别要取消的提醒的实体。

### 倒回事件 {#rewind-event}

`UserUtteranceReverted` 事件表示恢复最近用户消息之后发生的每个事件，其类型名称为 `rewind`。它不包含任何附加属性。

### 撤消事件 {#undo-event}

`ActionReverted` 事件表示恢复对话机器人的最后一个动作，其类型名称为 `undo`。它不包含任何附加属性。

### 导出故事事件 {#export-story-event}

`StoryExported` 事件表示将训练数据故事导出到文件，其类型名称为 `export`。它包含以下附加属性：

- `path`：导出的故事文件的路径。

### 暂停事件 {#pause-event}

`ConversationPaused` 事件表示对话暂停，其类型名称为 `pause`。它不包含任何附加属性。

### 恢复事件 {#resume-event}

`ConversationResumed` 事件表示对话恢复，其类型名称为 `resume`。它不包含任何附加属性。

### 活动循环事件 {#active-loop-event}

`ActiveLoop` 事件表示表单的激活，其类型名称为 `active_loop`。它包含以下附加属性：

- `name`：表单的名称。

### 循环中断事件 {#loop-interrupted-event}

`LoopInterrupted` 事件表示表单中断，其类型名称为 `loop_interrupted`。它包含以下附加属性：

- `is_interrupted`：布尔值，表示表单执行是否被中断。
