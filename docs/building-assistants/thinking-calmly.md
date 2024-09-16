# CALM 思维

CALM 专为创建可扩展的企业级对话机器人而设计。如果你之前构建过基于 LLM 的对话机器人，那么此页面尤其有助于挑战你重新思考如何解决问题。这将帮助你以惯用方式创建对话机器人，并因此构建更可靠的对话式 AI。

## 避免将 LLM 用于确定性任务 {#avoid-using-an-llm-for-deterministic-tasks}

始终尝试思考 LLM 的任务是否可以以确定性的方式完成，从而保证对话机器人的行为。

例如，让我们考虑一个电子商务商店的对话机器人。我们有用户最近的交易：

| 商品名称                     | 订单 ID | 订单日期   |
| :--------------------------- | :------ | :--------- |
| Vintage Leather Backpack     | 98765   | 2024-02-23 |
| Smart Home Speaker           | 98766   | 2024-02-20 |
| Limited Edition Vinyl Record | 98767   | 2024-02-18 |

我们知道用户会以多种不同方式提及这些产品（例如“the leather bag”、“my speaker”），业务逻辑需要先识别所有交易细节，然后才能继续。

识别用户所指内容的一种方法是依靠命令生成器为交易的每个属性设置一个槽，即订单 ID、商品名称、订单日期。因此，如果用户说的是“the leather bag”，则预期命令生成器会发出 3 个 `SetSlot` 命令：`SetSlot("Product Name", "Vintage Leather Backpack")`、`SetSlot("Order ID", 98765)` 和 `SetSlot("Order Date", "2024-02-23")`。这需要 LLM 非常强大，以确保交易属性之间的对应关系完全准确。

但是，几乎总是要引用的数据会为每个条目设置一个唯一标识符，并且让 LLM 预测唯一标识符的值应该足以找出其他细节。

因此，上述示例中首选的 CALM 解决方案是期望 LLM 输出 `SetSlot("OrderID", 98765)`，然后编写自定义动作来获取交易的其他属性。

## 使用 LLM 生成结构化查询 {#use-llms-to-generate-structured-queries}

如果你想根据结构化数据回答问题，请生成运行结构化查询所需的参数，而不是使用 LLM 推理数据。

例如，如果你有一个机场数据库，并且希望对话机器人回答以下问题：“How many Terminals does JFK have?”

| 名称                | 城市     | 国家          | 航站楼数量 | IATA 编码 | 纬度      | 经度       | 连接数量 | ObjectID |
| :------------------ | :------- | :------------ | :--------- | :-------- | :-------- | :--------- | :------- | :------- |
| La Guardia          | New York | United States | 3          | LGA       | 40.777245 | -73.872608 | 316      | 3697     |
| John F Kennedy Intl | New York | United States | 5          | JFK       | 40.639751 | -73.778925 | 911      | 3797     |

不要将数据提供给 LLM 并要求其生成答案，而是为要运行的查询定义一个带有槽值的流：

```yaml
flows:
 airport_info:
  description: answer users' questions about airports
  steps:
   - collect: iata_code
   - collect: airport_name
   - collect: attribute_to_query
   - action: utter_query
```

这样你的命令生成器就会输出 `StartFlow("airport_info")`、`SetSlot("iata_code", "JFK")`、`SetSlot("attribute_to_query", "num_terminals")`。当你希望将来支持更复杂的问题时，这种方法会更加强大，例如：“Which New York Airports have more than 4 Terminals?”。

## 将逻辑排除在自定义动作之外和流之内 {#keep-logic-out-of-custom-actions-and-inside-flows}

你团队中的任何人都应该能够通过查看流（直接检查 YAML 文件或通过 [Studio UI](https://rasa.com/docs/studio/user-guide/flow-builder/manage-flows)）来了解对话机器人的工作原理。

将逻辑隐藏在自定义动作中会使对话机器人更难理解。因此，请避免编写如下自定义动作：

```python
class CheckRestaurantAvailability(Action):
    def name():
        return "check_restaurant_availability"

    def run():
        has_availability = True # (fetched from an API)
        if has_availability:
            dispatcher.utter("Yes we have availability today.")
        else:
            dispatcher.utter("Unfortunately we are fully booked")
```

而是编写一个自定义动作，将相关数据返回到流，然后在 `next` 中指定条件：

```python
class CheckRestaurantAvailability(Action):
    def name():
        return "check_restaurant_availability"

    def run():
        has_availability = True # (fetched from an API)
        return SlotSet("has_availability", has_availability)
```

```yaml
flows:
  restaurant_booking:
    description: reserve a table at a restaurant 
    steps:
      - action: check_restaurant_availability
        next:
          - if: has_availability
              - action: utter_has_availability
          - else:
              - action: utter_no_availability
```

## 使用流中的逻辑运算符来定义业务逻辑 {#use-logical-operators-inside-flows-to-define-business-logic}

在 CALM 中，我们使用 LLM 来了解用户，而不是猜测业务逻辑。如果你有一个流应该根据用户是否是未成年人来执行不同的业务逻辑，请避免依赖 LLM 来选择适当的逻辑。相反，请在流中使用带有结构化运算符的条件来分支逻辑。

```yaml
flows:
  change_address:
    description: Allow a user to change their address.
    steps:
      - noop: true
        next:
          - if: slots.age < 18
            then:
              - action: utter_contact_helpline
          - else: ask_new_address
      - id: ask_new_address
        collect: address
        ...
```

## 使用确定性逻辑来限制访问 {#use-deterministic-logic-to-restrict-access}

如果你构建了某些功能，而这些功能只能由特定类别的用户访问，那么你应该使用[流守卫](../concepts/starting-flows.md#flow-guards)来隐藏其他用户的该功能，而不是依赖 LLM 来推理其访问权限。

例如，如果对话机器人需要只允许高级类别的客户与真实代理交谈，那么请避免在流描述中添加此信息：

```yaml
flows:
  talk_to_agent:
  description: Allows only premium users to talk to a human agent.
  steps:
    - action: trigger_human_handoff
```

相反，CALM 方法是使用流的流守卫属性中定义的确定性逻辑：

```yaml
flows:
  talk_to_agent:
    description: Allows user to talk to a human agent
    if: slots.is_premium_user
    steps:
      - action: trigger_human_handoff
```

添加此流守卫后，如果非高级用户正在与对话机器人交谈，则命令生成器将永远看不到与真实代理交谈的能力，因此用户将永远无法访问它。
