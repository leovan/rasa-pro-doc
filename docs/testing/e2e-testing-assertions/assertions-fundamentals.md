# 断言基础知识

了解 Rasa Pro 中使用断言进行端到端测试的基础知识。

## 断言类型 {#assertion-types}

软件工程中的断言是为了确保系统或单元按预期运行而进行的检查。本节提供了可在 Rasa Pro 端到端测试中使用的每种断言类型的全面示例。

### 流已启动断言 {#flow-started-assertion}

`flow_started` 断言检查具有所提供 ID 的流是否已启动。

```yaml
test_cases:
- test_case: flight_booking
  steps:
    - user: "I want to book a flight"
      assertions:
        - flow_started: "flight_booking"
```

### 流已完成断言 {#flow-completed-assertion}

`flow_completed` 断言检查具有所提供 ID 的流是否已完成。此外，你可以在断言中指定预期的最终步骤 ID。请注意，应自定义流步骤 ID 并在流定义中提供，以便使用此断言属性。

```yaml
test_cases:
- test_case: flight_booking
  steps:
    - user: "What is the average cost of a flight from New York to San Francisco?"
      assertions:
        - flow_completed:
              flow_id: "pattern_search"
              flow_step_id: "action_trigger_search"
```

### 流已取消断言 {#flow-cancelled-assertion}

`flow_cancelled` 断言检查具有所提供 ID 的流是否已取消。此外，你可以在断言中指定预期的最终步骤 ID。请注意，应自定义流步骤 ID 并在流定义中提供，以便使用此断言属性。

```yaml
test_cases:
- test_case: flight_booking
  steps:
    ... # other user steps
    - user: "Wait, I changed my mind, I don't want to book a flight."
      assertions:
        - flow_cancelled:
              flow_id: "flight_booking"
              flow_step_id: "make_payment"
```

### 模式澄清断言 {#pattern-clarification-contains-assertion}

`pattern_clarification_contains` 断言检查澄清修复模式是否已触发并返回预期的流名称。此断言必须列出预期作为澄清修复模式的一部分返回的所有流名称。

```yaml
test_cases:
- test_case: flight_booking
  steps:
    - user: "make booking"
      assertions:
        - pattern_clarification_contains:
            - "flight booking"
-           - "hotel booking"
```

### 槽已设置断言 {#slot-was-set-assertion}

`slot_was_set` 断言检查具有所提供名称的槽是否已用所提供值填充。

```yaml
test_cases:
- test_case: flight_booking
  steps:
    - user: "I want to book a flight from New York to San Francisco"
      assertions:
        - slot_was_set:
            - name: "origin"
              value: "New York"
            - name: "destination"
              value: "San Francisco"
```

### 槽未设置断言 {#slot-was-not-set-assertion}

`slot_was_not_set` 断言检查具有所提供名称的槽是否未填充。如果提供了值，则断言检查槽是否未用该特定值填充。

```yaml
test_cases:

- test_case: flight_booking
  steps:
    - user: "I want to book a flight to San Francisco."
      assertions:
        - slot_was_not_set:
             - name: "origin"
        - slot_was_not_set:
             - name: "destination"
               value: "New York"
```

请注意，当仅提供名称时，断言会检查槽是否未填充除 `None` 之外的任何值，并假设对于大多数槽，`None` 是默认初始值。

### 动作已执行断言 {#action-executed-assertion}

`action_executed` 断言检查是否执行了具有所提供名称的动作。

```yaml
test_cases:
- test_case: flight_booking
  steps:
    - user: "Book me a flight from New York to San Francisco tomorrow first thing in the morning."
      assertions:
        - action_executed: "action_book_flight"
```

### 对话机器人发出的断言 {#bot-uttered-assertion}

`bot_uttered` 断言检查对话机器人发出的话语是否与提供的模式、[按钮](../../concepts/responses.md#buttons)和/或[领域响应名称](../../concepts/responses.md#defining-responses)匹配。`text_matches` 键用于检查对话机器人发出的话语是否与提供的模式匹配，该模式可以是字符串或正则表达式。

```yaml
test_cases:
- test_case: flight_booking
  steps:
    - user: "I want to book a flight"
      assertions:
        - bot_uttered:
            utter_name: utter_ask_destination
            text_matches: "Where would you like to fly to?"
            buttons:
              - title: "New York"
                payload: "/SetSlots(destination=New York)"
              - title: "San Francisco"
                payload: "/SetSlots(destination=San Francisco)"
```

请注意，在断言按钮时，必须按照领域文件或自定义动作代码中定义的顺序列出它们。

### 生成响应相关断言 {#generative-response-is-relevant-assertion}

`generative_response_is_relevant` 断言检查生成响应是否与提供的用户输入相关。需要设置 `0` 到 `1` 之间的阈值来确定生成响应的相关性。LLM Judge 模型将按 `1` 到 `5` 的等级对生成响应输出进行评分，其中 `1` 表示最不相关，`5` 表示最相关。然后将分数映射到 `0` 到 `1` 之间的浮点值，可以将其与阈值进行比较。映射如下：`1 -> 0.2`、`2 -> 0.4`、`3 -> 0.6`、`4 -> 0.8`、`5 -> 1.0`。

```yaml
test_cases:
- test_case: flight_booking
  steps:
    - user: "What times are the flights from New York to San Francisco tomorrow?"
      assertions:
        - generative_response_is_relevant:
            threshold: 0.90
```

此外，如果你想检查重新措辞的响应是否与提供的用户输入相关，还可以向 `utter_name` 键提供[领域响应名称](../../concepts/responses.md#defining-responses)。

```yaml
test_cases:
- test_case: flight_booking
  steps:
    ... # other user steps
    - user: "Actually, I want to amend flight date to next week."
      assertions:
        - generative_response_is_relevant:
            threshold: 0.90
            utter_name: utter_ask_correction_confirmation
```

### 生成响应准确断言 {#generative-response-is-grounded-assertion}

`generative_response_is_grounded` 断言检查生成响应相对于基本事实是否准确。需要设置 `0` 到 `1` 之间的阈值来确定生成响应的事实准确性。LLM Judge 模型将按 `1` 到 `5` 的等级对生成响应输出进行评分，其中 `1` 表示事实准确性最低，`5` 表示事实准确性最高。然后将分数映射到 `0` 到 `1 `之间的浮点值，可以将其与阈值进行比较。映射如下：`1 -> 0.2`、`2 -> 0.4`、`3 -> 0.6`、`4 -> 0.8`、`5 -> 1.0`。

基本事实输入可以直接在测试中提供，也可以由测试运行器从实际的机器人话语事件元数据中提取，其中向量存储储存搜索结果（在企业搜索的情况下）或初始领域响应（在重新措辞的答案的情况下）。

```yaml
test_cases:
- test_case: flight_booking
  steps:
    - user: "What is the average cost of a flight from New York to San Francisco?"
      assertions:
        - generative_response_is_grounded:
            threshold: 0.90
            ground_truth: "The average cost of a flight from New York to San Francisco is $500."
```

此外，你还可以向 `utter_name` 键提供领域响应名称，用于过滤正确的对话机器人话语事件。
