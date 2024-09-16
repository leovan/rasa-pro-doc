# 领域

领域定义了对话机器人运行的领域。

在 Rasa 中，`domain` 定义了对话机器人运行的领域。具体来说，它列出了：

- 可用作模板消息发送给用户的 `responses`。
- 可以通过对话策略预测的自定义 `actions`。
- 在整个对话过程中充当对话机器人记忆的 `slots`。
- 会话配置参数，包括不活动超时。

如果你正在构建基于 NLU 的对话机器人，请参阅[领域](../nlu-based-assistants/domain.md)文档以了解如何在领域中配置意图、实体、槽映射和槽特征化。

## 多领域文件 {#multiple-domain-files}

领域可以定义为单个 YAML 文件，也可以拆分为目录中的多个文件。拆分为多个文件时，领域内容将被读取并自动合并在一起。你还可以在 Rasa Studio 中管理响应、槽和自定义动作。

使用命令行界面，可以通过运行以下命令来训练具有拆分领域文件的模型：

```shell
rasa train --domain path_to_domain_directory
```

## 响应 {#responses}

响应是对话机器人可以发送给用户的模板消息。响应可以包含丰富的内容，如按钮、图像和自定义 json 负载。每个响应也是一个[动作](#actions)，这意味着它可以直接在流中的 `action` 步骤中使用。响应可以直接在领域文件中的 `responses` 键下定义。有关响应及其定义方法的更多信息，请参阅[响应](responses.md)。

## 动作 {#actions}

[动作](actions.md)是对话机器人可以做的事情。例如，动作可以：

- 响应用户。
- 进行外部 API 调用。
- 查询数据库。
- 几乎任何事情！

所有[自定义动作](custom-actions.md)都应列在领域中。

Rasa 还具有[默认动作](default-actions.md)，你无需将其列在领域中。

## 槽 {#slots}

槽是对话机器人的记忆。它们充当键值存储，可用于存储用户提供的信息（例如他们的家乡）以及收集的有关外部世界的信息（例如数据库查询的结果）。

槽在领域的 `slots` 部分中定义，包括其名称、[类型](#slot-types)和默认值。存在不同的槽类型来限制槽可以采用的可能值。

!!! info "注意"

    如果你决定通过[响应按钮](responses.md#buttons)填充槽，其中[有效载荷语法](responses.md#payload-syntax)发出 `SetSlot` 命令，请注意槽名称不能包含某些字符，例如 `(`、`)`、`=` 或 `,`。

### 槽类型 {#slot-types}

#### 文本类型槽 {#text-slot}

文本类型槽可以使用任何字符串值。

- 示例

    ```yaml
    slots:
      cuisine:
        type: text
    ```

- 允许值

    任意字符串。

#### 布尔类型槽 {#boolean-slot}

布尔类型槽只能取 `true` 或 `false` 值。当你想要存储二进制值时，这很有用。

- 示例

    ```yaml
    slots:
      confirmation:
        type: bool
    ```

- 允许值

    `true` 或 `false`。

#### 分类槽 {#categorical-slot}

分类槽只能从预定义集合中取值。当你想要限制槽可以取的可能值时，这很有用。

如果用户提供的值的大小写与领域中定义的值的大小写不匹配，则该值将被强制转换为正确的大小写。例如，如果用户为具有 `low`、`medium`、`high` 的值的槽提供值 `LOW`，则该值将转换为 `low` 并存储在槽中。

如果你使用值列表定义分类槽，其中多个值为同一值，则会发出警告，你应该从领域中的集合中删除其中一个值。例如，如果你使用值 `low`、`medium`、`high` 和 `Low` 定义分类槽，则值 `Low` 将被强制转换为 `low` 并发出警告。

- 示例

    ```yaml
    slots:
      risk_level:
        type: categorical
        values:
          - low
          - medium
          - high
    ```

#### 浮点类型槽 {#float-slot}

浮点类型槽只接受浮点值。当你想要存储带有小数点的数字时，这很有用。

- 示例

    ```yaml
    slots:
      temperature:
        type: float
    ```

#### 任意类型槽 {#any-slot}

此类型槽可以接受任何值。当你想要存储任何类型的信息（包括字典等结构化数据）时，这很有用。

- 示例

    ```yaml
    slots:
      shopping_items:
        type: any
    ```

#### 列表类型槽 {#list-slot}

列表类型槽可以接受列表值。请注意，在使用 [CALM](../calm.md) 构建对话机器人时，列表类型槽仅在[自定义动作](custom-actions.md)中受支持。列表类型槽不能用流中的 [`collect`](flows.md#collect) 或 [`set_slots`](flows.md#set-slots) 流步骤类型填充。

### CALM 槽映射 {#calm-slot-mappings}

!!! info "3.9.0 版本新特性"

    使用 [CALM](../calm.md) 构建对话机器人时，你可以配置槽填充以使用[基于 NLU 的预定义](#nlu-based-predefined-slot-mappings)槽映射或新引入的 [`from_llm`](#from_llm) 槽映射类型。

#### 基于 NLU 的预定义槽映射 {#nlu-based-predefined-slot-mappings}

在使用 CALM 构建对话机器人时，你可以继续使用[基于 NLU 的预定义](#nlu-based-predefined-slot-mappings)槽映射，例如 [`from_entity`](../nlu-based-assistants/domain.md#from_entity) 或 [`from_intent`](../nlu-based-assistants/domain.md#from_intent)。除了将分词器、特征化器、意图分类器和实体提取器添加到管道之外，还必须将 [`NLUCommandAdapter`](components/nlu-command-adapter.md) 添加到 `config.yml` 文件中。`NLUCommandAdapter` 会将 NLU 管道的输出（意图和实体）与领域文件中定义的槽映射进行匹配。如果槽映射得到满足，`NLUCommandAdapter` 将发出 [`set slot` 命令](dialogue-understanding.md#set-slot)来填充槽。

如果在消息处理期间，`NLUCommandAdapter` 发出命令，则管道中的以下命令生成器（例如[基于 LLM 的命令生成器](components/llm-command-generators.md)）将被完全绕过。因此，基于 LLM 的命令生成器将无法通过在对话流中的任何点发出 [`set slot` 命令](dialogue-understanding.md#set-slot)来填充槽。如果基于 LLM 的命令生成器发出命令以使用基于 NLU 的预定义映射填充槽，则这些来自基于 LLM 的命令生成器的 `set slot` 命令将被忽略。如果在同一回合没有预测到其他命令，则对话机器人将触发 `cannot_handle` [对话修复模式](conversation-repair.md#9-cannot-handle)。

有时，用户消息可能包含超出设置槽范围的意图。例如，用户消息可能包含填充槽的实体，但也包含流必须处理的离题。在这种情况下，我们建议使用 [NLU 触发器](starting-flows.md#nlu-trigger)来处理流中的这些特定意图。有关更多详细信息，请参阅[不同场景中槽映射的影响](#impact-of-slot-mappings-in-different-scenarios)部分。

!!! note "注意"

    在使用流构建并使用 NLU 组件处理消息的 CALM 对话机器人中，默认动作 [`action_extract_slots`](../nlu-based-assistants/default-actions.md#action_extract_slots) 将不会运行，因为在命令执行期间，槽填充事件会应用于对话跟踪器。这可确保此默认动作不会覆盖 CALM [`set slot`](dialogue-understanding.md#set-slot) 命令，也不会重复已应用于对话跟踪器的 `SlotSet` 事件。

    在[共存](business-logic/coexistence.md)的情况下，仅当基于 NLU 的系统处于活动状态时，才会执行 `action_extract_slots` 动作。

#### `from_llm` {#from_llm}

可以使用 `from_llm` 槽映射类型，用[基于 LLM 的命令生成器](components/llm-command-generators.md)生成的值填充槽。如果领域文件中未明确定义映射，则这是默认槽映射类型。

以下是示例：

```yaml
slots:
  user_name:
    type: text
    mappings:
      - type: from_llm
```

在此示例中，`user_name` 槽将由基于 LLM 的命令生成器生成的值填充。基于 LLM 的命令生成器可以在对话流中的任何点填充此槽位，而不仅仅是在此槽位的相应 `collect` 步骤中。

如果你在 `config.yml` 管道中定义了其他基于 NLU 的组件，这些组件将继续处理用户消息，但它们将无法填充槽。`NLUCommandAdapter` 将跳过任何具有 `from_llm` 映射的槽，并且不会发出 [`set slot` 命令](dialogue-understanding.md#set-slot)来填充这些槽。有关更多详细信息，请参阅[不同场景中槽映射的影响](#impact-of-slot-mappings-in-different-scenarios)部分。

请注意，槽不能同时具有 `from_llm` 和基于 NLU 的预定义映射或[自定义槽映射](../nlu-based-assistants/domain.md#custom-slot-mappings)。如果使用 `from_llm` 映射定义槽，则不能为该槽定义任何其他映射类型。

#### 映射条件 {#mapping-conditions}

你可以定义在填充槽之前要满足的槽映射条件。条件定义为 `conditions` 键下的条件列表。每个条件都可以将必须​​处于活动状态的流 ID 指定为 `active_flow` 属性。

如果你定义了映射到同一实体的多个槽，但不想在提取实体时填充所有槽，则这特别有用。

例如：

```yaml
entities:
- person

slots:
  first_name:
    type: text
    mappings:
      - type: from_entity
        entity: person
        conditions:
          - active_flow: greet_user

  last_name:
    type: text
    mappings:
      - type: from_entity
        entity: person
        conditions:
          - active_flow: issue_invoice
```

#### 自定义插槽映射 {#custom-slot-mappings}

你可以使用 `custom` 映射类型来定义应由[自定义动作](custom-actions.md)填充的槽的自定义槽映射。必须在槽映射的 `action` 属性中指定自定义动作。你还必须在领域文件中的 `operations` 键下列出该动作。

例如：

```yaml title="domain.yml"
actions:
- action_fill_user_name

slots:
  user_name:
    type: text
    mappings:
      - type: custom
        action: action_fill_user_name
```

在此示例中，`user_name` 槽将由 `action_fill_user_name` 自定义动作填充。自定义动作必须返回带有槽名称和值的 `SlotSet` 事件才能填充槽。

请注意，如果你使用 [`action_ask_<slot_name>` 命名约定](flows.md#using-an-action-to-ask-for-information-in-collect-step)通过自定义动作请求用户输入，但该槽由基于 LLM 的命令生成器生成的值填充，则不应为该槽定义自定义槽映射。相反，请使用 `from_llm` 映射类型，因为自定义映射类型是为由返回 `SlotSet` 事件的自定义动作设置的槽保留的（例如，由外部源设置的槽）。你可以继续使用 `action_ask_<slot_name>` 约定来请求由基于 LLM 的命令生成器填充的槽的用户输入。

如果你使用自定义验证动作（使用 `validate_<slot_name>` 命名约定）来验证由基于 LLM 的生成器从最终用户的输入中提取的槽值，则也不应该为这些槽定义自定义槽映射。相反，请为这些槽使用 `from_llm` 映射类型来。

!!! warning "警告"

    如果你使用 `--skip-validation` 标志进行训练，并且已定义具有自定义槽映射的槽，这些槽未在领域文件中指定 `action` 属性，也没有相应的 `action_ask_<slot_name>` 自定义动作来请求这些槽，则将不会在训练时收到错误。但是，在运行时，[`FlowPolicy`](policies/flow-policy.md) 将首先取消正在进行的用户流，然后触发 [`pattern_internal_error`](conversation-repair.md#8-internal-errors)。

    你也可以通过 [`rasa data verify`](../command-line-interface.md#rasa-data-validate) 命令运行此检查。

#### 不同场景中槽映射的影响 {#impact-of-slot-mappings-in-different-scenarios}

本节阐明了当流处于 `name` 槽的 `collect` 步骤或任何其他步骤时，使用流和 NLU 管道构建的 CALM 对话机器人中的哪些组件在不同场景中填充槽。

假设 `name` 槽使用 `from_llm` 映射类型定义。

| 功能                                               | `name` 槽的收集步骤 | 任何其他不收集槽的步骤 |
| -------------------------------------------------- | ------------------- | ---------------------- |
| 基于 LLM 的生成器处于活动状态                      | ✅                   | ✅                      |
| NLU 组件（例如意图分类器、实体提取器）处于活动状态 | ✅                   | ✅                      |
| 基于 LLM 的生成器是否可以填充 `name` 槽            | ✅                   | ✅                      |
| `NLUCommandAdapter` 是否可以填充 `name` 槽         | ❌                   | ❌                      |

要点是 `NLUCommandAdapter` 无法在对话的任何阶段用 `from_llm` 映射填充槽。

假设 `name` 槽是使用基于 NLU 的预定义映射之一（例如 `from_entity`）定义的。

| 功能                                               | `name` 槽的收集步骤 | 任何其他不收集槽的步骤 |
| -------------------------------------------------- | ------------------- | ---------------------- |
| 基于 LLM 的生成器处于活动状态                      | ❌                   | ✅                      |
| NLU 组件（例如意图分类器、实体提取器）处于活动状态 | ✅                   | ✅                      |
| 基于 LLM 的生成器是否可以填充 `name` 槽            | ❌                   | ❌                      |
| `NLUCommandAdapter` 是否可以填充 `name` 槽         | ✅                   | ✅                      |

要点：

- 基于 LLM 的生成器无法在对话的任何阶段用基于 NLU 的预定义映射填充槽。
- 基于 LLM 的生成器在 `name` 槽的收集步骤中不会处于活动状态。如果预计用户话语包含离题或其他意图，超出了设置槽的信息，则应使用 [NLU 触发器](starting-flows.md#nlu-trigger)来处理流中的这些特定意图。
- 基于 LLM 的生成器可以在未收集 `name` 槽且具有 `from_llm` 映射类型的步骤中填充其他槽。

### 初始化槽值 {#initial-slot-values}

你可以为领域文件中的任何槽提供初始值：

```yaml
slots:
  num_fallbacks:
    type: float
    initial_value: 0
```

### 共存期间槽的持久性 {#persistence-of-slots-during-coexistence}

在[基于 NLU 的系统和 CALM 系统的共存](../building-assistants/coexistence.md)中，[`action_reset_routing`](default-actions.md#action_reset_routing) 动作会重置所有槽并隐藏基于 NLU 的系统策略的特征化事件，以防止它们看到 CALM 处于活动状态时发生的事件。但是，你可能希望共享一些 CALM 和基于 NLU 的系统都应该能够使用的槽。这些槽的一个用例是基本用户配置文件槽。基于 NLU 的系统和 CALM 都应该能够知道用户是否登录、他们的用户名是什么或他们正在使用哪个频道。如果你将此类数据存储在槽中，则可以使用选项 `shared_for_coexistence: True` 注释这些槽定义。

```yaml hl_lines="9 13"
version: "3.1"

slots:
  user_channel:
    type: categorical
    values:
      - web
      - teams
    shared_for_coexistence: True

  user_name:
    type: text
    shared_for_coexistence: True
```

在共存模式下，如果未将选项 `shared_for_coexistence` 设置为 `true`，则会使流定义中的 [`reset_after_flow_ends: False` 属性](flows.md#resetting-slots-at-the-end-of-a-flow)无效。为了在整个对话过程中保留槽值，必须将 `shared_for_coexistence` 设置为 `true`。

### 会话配置 {#session-configuration}

对话会话代表对话机器人与用户之间的对话。对话会话可以通过三种方式开始：

- 用户开始与对话机器人对话。
- 用户在可配置的不活动期后发送第一条消息。
- 使用 `/session_start` 意图消息触发会话手动启动。

你可以在领域中 `session_config` 键下定义不活动时间，在此时间之后将触发新的对话会话。

可用参数包括：

- `session_expiration_time` 定义不活动时间（以分钟为单位），在此时间之后将开始新的会话。
- `carry_over_slots_to_new_session` 确定是否应将现有的设置时隙转移到新会话。
默认会话配置如下所示：

```yaml
session_config:
  session_expiration_time: 60  # value in minutes, 0 means infinitely long
  carry_over_slots_to_new_session: true  # set to false to forget slots between sessions
```

这意味着，如果用户在 60 分钟不活动后发送第一条消息，则会触发新的对话会话，并且任何现有时段都会延续到新会话中。将 `session_expiration_time` 的值设置为 `0` 意味着会话不会结束（请注意，`action_session_start` 动作仍会在对话开始时触发）。

!!! note "注意"

    会话启动会触发默认动作 `action_session_start`。其默认实现会将所有现有槽移至新会话中。请注意，所有对话都以 `action_session_start` 开头。例如，覆盖此动作可用于使用来自外部 API 调用的槽初始化追踪器，或使用机器人消息启动对话。有关[自定义会话启动动作](default-actions.md#customization)的文档展示了如何执行此动作。

### 选择哪些动作应该接收领域 {#select-which-actions-should-receive-domain}

!!! info "3.4.3 新特性"

    你可以控制某个动作是否应该接收领域。

为此，你必须首先在 `endpoints.yml` 中为 `action_endpoint` 启用 API 配置中的选择性领域。

```yaml title="endpoints.yml"
action_endpoint:
  url: "http://localhost:5055/webhook" # URL to your action server
  enable_selective_domain: true
```

启用自定义动作的选择性领域后，领域将仅发送给明确声明需要的自定义动作。从 rasa-sdk [`FormValidationAction`](../action-server/validation-action.md#formvalidationaction-class) 父类继承的自定义动作是此规则的一个例外，因为它们始终会将领域发送给它们。要指定动作是否需要领域，请在 `domain.yml` 中的动作列表中将 `{send_domain: true}` 添加到自定义动作：

```yaml title="domain.yml"
actions:
  - action_hello_world: {send_domain: True} # will receive domain
  - action_calculate_mass_of_sun # will not receive domain
  - validate_my_form # will receive domain
```
