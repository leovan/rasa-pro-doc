# 策略概述

策略决定对话中的下一步行动。

在 Rasa 中，策略是负责对话管理的组件。在基于 CALM 的对话机器人中，[FlowPolicy](flow-policy.md) 负责执行业务逻辑。如果你正在构建基于 NLU 的对话机器人，可以在[此处](../../nlu-based-assistants/policies.md)阅读相关策略。

!!! info "CALM 中的策略"

    由于 CALM 中的[对话理解](../dialogue-understanding.md)组件已经考虑了对话的上下文，因此对话管理器的角色比基于 NLU 的对话机器人更简单。

你可以通过在项目的 `config.yml` 中指定 `policies` 键来自定义对话机器人使用的策略。在大多数情况下，默认配置应该可以满足你的需求，更高级的用例则可以进行自定义。

有不同的策略可供选择，你可以在单个配置中包含多个策略。以下是策略列表的示例：

=== "Rasa Pro >= 3.8.x"

    ```yaml title="config.yml"
    # ...

    policies:
    - name: FlowPolicy
    - name: EnterpriseSearchPolicy
    ```

=== "Rasa Pro <=3.7.x"

    ```yaml title="config.yml"
    # ...

    policies:
    - name: FlowPolicy
    - name: rasa_plus.ml.EnterpriseSearchPolicy
    ```

## 动作选择 {#action-selection}

每次，配置中定义的每个策略都有机会以一定的置信度预测下一步[动作](../actions.md)。策略也可以决定不预测任何动作。预测置信度最高的策略决定对话机器人的下一步动作。

!!! info "最大预测数量"

    默认情况下，对话机器人最多可以在每条用户消息后预测 10 个后续动作。要更新此值，可以将环境变量 `MAX_NUMBER_OF_PREDICTIONS` 设置为所需的最大预测数量。
