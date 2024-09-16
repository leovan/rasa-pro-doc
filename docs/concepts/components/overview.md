# 配置

配置文件定义了模型将用来根据用户输入进行预测的组件和策略。

你可以通过修改 `config.yml` 文件来自定义 Rasa 工作方式的许多方面。

[CALM](../../calm.md) 对话机器人的最小配置如下所示：

```yaml title="config.yml"
recipe: default.v1
language: en
assistant_id: 20230405-114328-tranquil-mustard

pipeline:
  - name: SingleStepLLMCommandGenerator

policies:
  - name: rasa.core.policies.flow_policy.FlowPolicy
```

!!! tip "默认配置"

    为了向后兼容，运行 `rasa init` 将创建一个基于 NLU 的对话机器人。要使用正确的 `config.yml` 创建 CALM 对话机器人，请添加额外的 --template 参数：

    ```shell
    rasa init --template calm
    ```

## `recipe`、`language` 和 `assistant_id` 键 {#the-recipe-language-and-assistant_id-keys}

仅当你要使用[自定义图配方](graph-recipe.md)时才需要修改 `recipe` 键。绝大多数项目应使用默认值 `"default.v1"`。

`language` 键是对话机器人支持的语言的 2 个字母 ISO 代码。

`assistant_id` 键应是唯一值，可让你区分多个部署的对话机器人。此 ID 与模型 ID 一起添加到每个事件的元数据中。有关更多信息，请参阅[事件代理](../../production/event-brokers.md)。请注意，如果配置文件不包含此必需键或未替换占位符默认值，则每次运行 `rasa train` 时都会生成一个随机对话机器人名称并将其添加到配置中。

## 管道 {#pipeline}

`pipeline` 键列出了将用于处理和理解最终用户发送给对话机器人的消息的组件。在 CALM 对话机器人中，组件管道的输出是[命令](../dialogue-understanding.md)列表。

管道中的主要组件是 `LLMCommandGenerator`。示例配置如下所示：

```yaml title="config.yml"
pipeline:
  - name: SingleStepLLMCommandGenerator
    llm:
      model_name: "gpt-4"
      request_timeout: 7
      temperature: 0.0
    user_input:
      max_characters: 420
```

[此处](../dialogue-understanding.md)列出了完整的可配置参数集。

所有使用 LLM 的组件都有通用的配置参数在[此处](llm-configuration.md)列出。

## 策略 {#policies}

`policies` 键列出了对话机器人将用来推进对话的[对话策略](../policies/policy-overview.md)。

```yaml title="config.yml"
policies:
  - name: rasa.core.policies.flow_policy.FlowPolicy
```

[FlowPolicy](../policies/flow-policy.md) 目前没有额外的配置参数。
