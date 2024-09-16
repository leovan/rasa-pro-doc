# 流中的条件

条件是流中的逻辑语句，用于确定哪些流可以执行、导航逻辑分支以及验证槽值。

## 条件 {#conditions}

条件在流中的三个不同位置使用：

- 在[流守卫](starting-flows.md#flow-guards)中确定是否可以启动流。
- 在流步骤的 [`next`](flows.md#steps) 字段中用于在业务逻辑的分支之间进行选择。
- 在 `collect` 步骤的 `rejections` 字段中[验证槽值](flows.md#slot-validation)。

## 语法 {#syntax}

流中的条件使用自然语言编写，可以包含逻辑运算符、条件运算符和其他构造。它们使用 [pypred](https://github.com/armon/pypred) 库运行。

这些条件支持以下运算符：

- `not`：否定条件。
- `and`：使用逻辑 AND 组合两个条件。
- `or`：使用逻辑 OR 组合两个条件。
- `>`：大于。
- `>=`：大于或等于。
- `<`：小于。
- `<=`：小于或等于。
- `=`：等于。
- `!=`：不等于。
- `is`：检查相同。
- `is not`：检查不相同。
- `contains`：检查集合是否包含值的子集运算符
- `matches`：使用正则表达式匹配字符串。

### 括号 {#parentheses}

使用括号对表达式进行分组并控制求值的顺序。例如：

```yaml
- collect: age
  next:
    - if: slots.age < 18
      then: under_18_step
    - if: (slots.age > 18 and (consent = "yes" or consent = "y"))
      then: consent_accept
    - else: consent_decline
```

### 子集运算符 {#subset-operator}

子集运算符 `contains` 可用于 `SET contains VALUE` 格式，其中 `SET` 是可能值的集合，`VALUE` 是槽的名称。例如：

```yaml
- collect: emergency
  next:
    - if: "{'WARN' 'ERR' 'CRIT'} contains slots.error_level"
      then: handoff
    - else: everything_okay
```

当用作 `slots.product contains "Rasa"` 时，`contains` 运算符也可用于识别子字符串，但此检查区分大小写。

### 常量 {#constants}

- 字符串文字：用单引号或双引号括起来（`'example'` 或 `"example"`）。
- 数字文字：不带引号的数字（`42`）。
- 常量：`true`、`false`、`undefined`、`null`、`empty`。

### 正则表达式 {#regular-expressions}

使用 `matches` 运算符时，你可以包含正则表达式修饰符。正则表达式修饰符应括在引号中。例如，此流步骤检查邮政编码槽是否包含美国邮政编码：

```yaml
- collect: zipcode
  description: ask zipcode and check if its a zip code
  next:
    - if: slots.zipcode matches "\d{5}(-\d{4})?"
      then: ask_payment
    - else: wrong_zipcode
```

可以进行不区分大小写的子字符串比较，条件 `product matches "(?i).*rasa.*"` 可以检查 `product` 变量中的子字符串 `rasa`。

### 空值 {#empty-values}

如果想检查布尔槽是否未设置，则需要使用语法 `<boolean-slot> is null`。

要检查文本槽是否未设置或为空，则使用语法 `not <text-slot>`。

### 示例 {#examples}

以下是一些条件示例，展示了如何使用不同的构造。

```python
# Simple conditions
age > 18
name is "Alice"
name is empty
status = "active"
status is not null

# Combining Conditions
age > 21 and gender = "female"
category = "electronics" or category = "computers"
status = "active" and (priority = 1 or priority = 2)
status = empty or status is null
description matches "/error \d{3}/i" and (severity = "high" or source contains "server")
```

## 命名空间 {#namespaces}

命名空间用于访问[分支条件](flows.md#next-property)、[槽验证](flows.md#slot-validation)和[流守卫](starting-flows.md#flow-guards)中使用的谓词中的不同类型的数据。有两个可用的命名空间：`slots` 和 `context`。`slots` 命名空间用于访问槽值，而 `context` 命名空间用于访问当前对话框架属性。

### 槽 {#slots}

`slots` 命名空间用于访问槽值。槽名称必须以 `slots.` 为前缀，才能在条件中访问。例如：

```yaml
- id: some_question
  collect: age
  next:
    - if: slots.age < 18
      then: under_18_step
    - else: over_18_step
```

确保在[领域](domain.md)中定义了槽。如果槽未在领域中定义，或者槽未以 `slots.` 命名空间作为前缀，则训练期间运行的验证将失败并出现相应错误。

### 上下文 {#context}

`context` 命名空间用于访问当前[对话框架](conditions.md#dialogue-frames)的属性。该属性必须以 `context.` 为前缀才能在谓词中访问。例如：

```yaml
  pattern_completed:
    description:  a flow has been completed and there is nothing else to be done
    steps:
      - noop: true
        next:
          - if: context.previous_flow_name != "greeting"
            then:
              - action: utter_what_can_help_with
                next: END
          - else: stop
      - id: stop
        action: action_stop
```

你还可以使用 jinja 模板来访问 `context` 命名空间。例如：

```yaml
  pattern_completed:
    description:  a flow has been completed and there is nothing else to be done
    steps:
      - noop: true
        next:
          - if: "{{context.previous_flow_name}}" != "greeting"
            then:
              - action: utter_what_can_help_with
                next: END
          - else: stop
      - id: stop
        action: action_stop
```

#### 对话框架 {#dialogue-frames}

对话管理器在对话框架堆栈中组织（用户定义和内置的）流的进展。对话框架堆栈表示对话框架的 LIFO（后进先出）堆栈。不同类型的对话框架被映射到内置对话模式，从而实现[对话修复](conversation-repair.md)。

每个对话框架都有一个 `flow_id` 和 `step_id` 属性。`flow_id` 是当前流的 ID，`step_id` 是流中当前步骤的 ID。

可用的对话框架类型如下：

1. [取消](#cancel)：处理流取消。
2. [闲聊](#chitchat)：处理闲聊。
3. [澄清](#clarify)：处理澄清。
4. [收集信息](#collect-information)：处理信息收集。
5. [完成](#completion)：处理流完成。
6. [继续中断](#continue-interrupted)：处理中断流的继续。
7. [更正](#correction)：处理更正。
8. [内部错误](#internal-error)：处理内部错误。
9. [搜索](#knowledge-search)：处理知识搜索。
10. [跳过问题](#skip-question)：处理跳过信息收集。
11. [代码更改](#can-not-handle)：在对话机器人更新后清理堆栈。
12. [无法处理](#can-not-handle)：处理对话机器人无法处理的情况。
13. [人工处理](#human-handoff)：处理交接给人工。

##### 取消 {#cancel}

`flow_id` 为 `pattern_cancel_flow`。`step_id` 为 `START`。此外，还有以下属性可用：

- `canceled_name`：应取消的流的名称。
- `canceled_frames`：应取消的堆栈框架列表。

##### 闲聊 {#chitchat}

`flow_id` 为 `pattern_chitchat`。`step_id` 为 `START`。

##### 澄清 {#clarify}

`flow_id` 为 `pattern_clarify`。`step_id` 为 `START`。此外，还有以下属性可用：

- `names`：用户可以从中选择的流名称列表。
- `clarification_options`：用户可以从中选择的字符串选项。

##### 收集信息 {#collect-information}

`flow_id` 为 `pattern_collect_information`。`step_id` 为 `START`。此外，还有以下属性可用：

- `collect`：应填充的槽的名称。
- `utter`：向用户询问信息应执行的响应。
- `collect_action`：向用户询问信息应执行的动作。
- `rejections`：如果用户提供无效信息，应执行的可选验证检查列表。

请注意，如果在流中使用 `context.collect` 属性，则必须以 `jinja` 模板样式编写此属性，并在其前面加上 `slots.` 命名空间。这是因为 `context.collect` 对应于槽名称，并且每个槽都必须以 `slots.` 命名空间开头。

例如：

```yaml
  pattern_collect_information:
    name: "pattern_collect_information"
    description:  the assistant is collecting information from the user
    steps:
      - id: check
        next:
          - if: "slots.{{context.collect}} is not null"
            then:
              - action: utter_ask_age
                next: listen
          - else: END
      - id: listen
        action: action_listen
```

##### 完成 {#completion}

`flow_id` 为 `pattern_completed`。`step_id` 为 `START`。此外，以下属性可用：

- `previous_flow_name`：已完成的流的名称。

##### 继续中断 {#continue-interrupted}

`flow_id` 为 `pattern_continue_interrupted`。`step_id` 为 `START`。此外，以下属性可用：

- `previous_flow_name`：已中断的流的名称。

##### 更正 {#correction}

`flow_id` 为 `pattern_correction`。`step_id` 为 `START`。此外，以下属性可用：

- `is_reset_only`：表示更正是否仅为流重置。
- `corrected_slots`：应更正的槽键值对。
- `reset_flow_id`：要重置为的流的 ID，默认为 `None`。
- `reset_step_id`：要重置为的步骤的 ID，默认为 `None`。

##### 内部错误 {#internal-error}

`flow_id` 为 `pattern_internal_error`。`step_id` 为 `START`。此外，以下属性可用：

- `error_type`：字符串表示错误类型。
- `info`：要提供给用户的附加信息。

##### 知识搜索 {#knowledge-search}

`flow_id` 为 `pattern_search`。`step_id` 为 `START`。

##### 跳过问题 {#skip-question}

`flow_id` 为 `pattern_skip_question`。`step_id` 为 `START`。

##### 代码更改 {#can-not-handle}

`flow_id` 为 `pattern_code_change`。`step_id` 为 `START`。

##### 无法处理 {#can-not-handle}

`flow_id` 为 `pattern_cannot_handle`。`step_id` 为 `START`。此外，还有以下属性可用：

- `reason`：字符串表示对话机器人无法处理的原因。

##### 人工处理 {#human-handoff}

`flow_id` 为 `pattern_human_handoff`。`step_id` 为 `START`。
