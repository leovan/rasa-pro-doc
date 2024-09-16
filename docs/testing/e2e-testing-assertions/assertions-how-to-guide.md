# 如何使用断言进行端到端测试

了解如何使用带有断言的端到端测试来测试对话机器人的行为。

## 测试格式概述 {#test-format-overview}

断言可以作为用户步骤的一部分在现有测试用例格式中定义。你可以为每个用户步骤定义多个断言。以下是带有断言的测试用例示例：

```yaml
test_cases:
- test_case: flight_booking
  steps:
    - user: "I want to book a flight"
      assertions:
        - bot_uttered:
            utter_name: "utter_ask_destination"
    - user: "New York"
      assertions:
        - slot_was_set:
            - name: "destination"
              value: "New York"
        - bot_uttered:
            utter_name: "utter_ask_origin"
    - user: "San Francisco"
      assertions:
        - slot_was_set:
            - name: "origin"
              value: "San Francisco"
        - bot_uttered:
            text_matches: "When would you like to travel?"
```

!!! warning "警告"

    请注意，你只能使用[预先存在的步骤类型](../../production/testing-your-assistant.md#how-to-write-test-cases)（例如 `bot` 或 `utter`）或使用断言来运行测试用例。

    一旦测试用例包含断言，测试运行器就会忽略预先存在的步骤类型。

默认情况下，只有每个用户回合生成的实际 Rasa 事件子集将用于验证断言。如果你还想验证断言的顺序是否正确，则可以在用户步骤中设置 `assertion_order_enabled: true` 键。这将确保按照测试用例中定义的顺序验证断言。例如：

```yaml
test_cases:
- test_case: flight_booking
  steps:
    - user: "I want to book a flight"
      assertions:
        - bot_uttered:
            utter_name: "utter_ask_destination"
    - user: "New York"
      assertion_order_enabled: true
      assertions:
        - slot_was_set:
            - name: "destination"
              value: "New York"
        - bot_uttered:
            utter_name: "utter_ask_origin"
```

## 如何处理同一测试用例中的重复用户文本消息 {#how-to-handle-duplicate-user-text-messages-in-the-same-test-case}

在某些情况下，你可能希望在同一测试用例中多次发送相同的用户文本消息。为了检索正确用户回合的实际事件，你必须定义并为每个用户步骤提供元数据。即使用户文本消息相同，每个用户步骤的元数据也必须是唯一的。

例如：

```yaml
metadata:
- duplicate_message_1:
    turn_idx: 1
- duplicate_message_2:
    turn_idx: 2

test_cases:
- test_case: flight_booking
  steps:
    ... # other user steps
    - user: "yes"
      metadata: duplicate_message_1
      assertions:
        - bot_uttered:
            utter_name: "utter_ask_confirmation_booking"
    - user: "yes"
      metadata: duplicate_message_2
      assertions:
        ... # other assertions
```

## 测试结果分析 {#test-results-analysis}

在运行带有断言的测试用例时，测试运行器将提供该特定测试运行中每种断言类型的准确度汇总统计信息。准确度计算为成功断言的数量除以成功和失败断言的总和。请注意，测试用例中先前断言失败后无法运行的断言不包括在准确度计算中。

```txt
========== Accuracy By Assertion Type ==========
┌---------------------------------┬----------┐
│         Assertion Type          │ Accuracy │
├---------------------------------┼----------┤
│          flow_started           │ 100.00%  │
│         flow_completed          │ 100.00%  │
│         flow_cancelled          │ 100.00%  │
│ pattern_clarification_contains  │ 100.00%  │
│         actino_executed         │ 100.00%  │
│          slot_was_set           │  98.75%  │
│        slot_was_not_set         │ 100.00%  │
│           bot_uttered           │ 100.00%  │
│ generative_response_is_relevant │ 100.00%  │
│ generative_response_is_grounded │ 100.00%  │
└---------------------------------┴----------┘
```

此外，测试运行器将提供失败断言的详细报告，包括断言本身、错误消息、测试用例文件中的行号以及在断言失败之前在对话中记录的实际事件的字符串表示的记录。
