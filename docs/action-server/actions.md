# 动作

当 Rasa 对话机器人调用自定义动作时，它会向动作服务器发送请求。Rasa 只知道请求响应中返回的任何事件和响应，由动作服务器根据 Rasa 提供的动作名称调用正确的代码。

为了更好地理解 Rasa 调用自定义动作时会发生什么，请考虑如下示例：

你已将天气机器人部署到 Facebook 和 Slack。用户可以使用 `ask_weather` 意图来询问天气。如果用户指定了 `location`，则会有一个槽位置。`action_tell_weather` 动作将使用 API 获取天气预报，如果用户未指定，则使用默认位置。该动作会将 `temperature` 设置为天气预报的最高温度。返回的消息将根据他们使用的频道有所不同。

## 动作服务器支持的协议 {#protocols-supported-by-the-action-server}

动作服务器支持两种与 Rasa 服务器通信的协议：

- [HTTP API](#http-protocol)
- [gRPC](#grpc-protocol)

### HTTP(S) 协议 {#https-protocol}

HTTP 协议的 API 规范可在[此处]()找到。

!!! info "在 HTTP 请求中压缩请求体"

    Rasa 可以对自定义动作的 HTTP 请求体进行压缩。默认情况下此选项处于关闭状态，以保证能向后兼容旧版本不能接受压缩的 HTTP 请求体的自定义动作。要启用此选项，请将环境变量 `COMPRESS_ACTION_SERVER_REQUEST` 设置为 `True`。

    Rasa SDK 的 3.2.2，3.3.1，3.4.1 和更高版本的自定义动作服务器支持压缩和非压缩的 HTTP 请求体。无需额外进行设置。

#### HTTP 协议 {#http-protocol}

要使用 HTTP 协议连接到动作服务器，你需要在 `endpoints.yml` 文件中的 `action_endpoint` 的 `url` 中将协议模式设置为 `http`：

```yaml
action_endpoint:
  url: "http://localhost:5055/webhook"
```

使用 HTTP 协议时，请确保[动作服务器在启用 HTTP 的情况下启动](running-action-server.md#http-protocol)。

#### HTTPS 协议 {#https-protocol-1}

要使用 HTTPS 协议连接到动作服务器，请在 `action_endpoint` 的 `url` 中将协议架构设置为 `https`，并在 `endpoints.yml` 文件中的 `cafile` 字段中提供 CA 证书：

```yaml
action_endpoint:
  url: "https://localhost:5055/webhook"
  cafile: "/path/to/ssl_ca_certificate"
```

使用 HTTPS 协议时，请确保：

- [动作服务器在启用 HTTPS 的情况下启动](running-action-server.md#https-protocol)
- 证书由受信任的 CA 签名
- 服务器证书具有正确设置的主机列表

### gRPC 协议 {#grpc-protocol}

!!! info "3.9 版本新特性"

gRPC 协议的 API 规范可在[此处](action-server-grpc-api.md)找到。gRPC 协议的数据模式与 HTTP 协议紧密相关。

我们在 gRPC 协议中默认使用压缩。它无法被禁用。

我们支持安全 (TLS) 和不安全的 gRPC 连接。

### 不安全 gRPC 连接 {#insecure-grpc-connection}

要通过不安全的 gRPC 连接连接到动作服务器，你需要在 `endpoints.yml` 文件中的`action_endpoint` 的 `url` 字段中将协议模式设置为 `grpc`：

```yaml
action_endpoint:
  url: "grpc://localhost:5055"
```

使用不安全的 gRPC 连接时，请确保[动作服务器在启用不安全连接的情况下启动](running-action-server.md#insecure-grpc-connection)。

### 安全 (TLS) gRPC 连接 {#secure-tls-grpc-connection}

要通过安全的 gRPC 连接连接到动作服务器，你需要在 `endpoints.yml` 文件的 `cafile` 字段中提供 SSL CA 证书的路径：

```yaml
action_endpoint:
  url: "grpc://localhost:5055"
  cafile: "/path/to/ssl_ca_certificate"
```

使用安全 gRPC 连接时，请确保：

- [动作服务器在启用安全连接的情况下启动](running-action-server.md#secure-grpc-connection)
- 证书由受信任的 CA 签名
- 服务器证书已正确设置主机列表

## 自定义动作输入 {#custom-action-input}

注意：下面的示例使用了 HTTP 协议。gRPC 协议的输入格式遵循类似的数据格式。例如，下面的 json 有效负载可以序列化为 gRPC 请求对象并通过 gRPC 协议将其发送到d动作服务器。

=== "输入格式"

    动作服务器从 Rasa 服务器接收以下有效负载：

    有效负载中的某些字段是可选的，可能并非存在于所有请求中。标记为 `object` 的字段具有非常复杂的结构，此处未显示。

    ```json
    # pseudo code
    request {
      next_action: { type: string }
      sender_id: { type: string }
      tracker: { type: object }
      domain: { type: optional[object] }
      domain_digest: { type: optonal[string] }
      version: { type: string }
    }

    domain {
      config: { type: object }
      session_config: { type: object }
      intents: { type: object }
      entities: { type: object }
      slots: { type: object }
      responses: { type: object }
      actions: { type: object }
      forms: { type: object }
      e2e_actions: { type: object }
    }

    tracker {
      sender_id: { type: string }
      slots: { type: object }
      latest_message: { type: object }
      events: { type: array_of_objects }
      paused: { type: bool }
      followup_action: { type: optional[string] }
      active_loop: { type: map }
      latest_action_name: { type: string }
      stack: { type: array_of_objects }
    }
    ```

=== "JSON 示例"

    ```json
    {
      "next_action": "action_tell_weather",
      "sender_id": "2687378567977106",
      "tracker": {
        "sender_id": "2687378567977106",
        "slots": {
          "location": null,
          "temperature": null
        },
        "latest_message": {
          "text": "/ask_weather",
          "intent": {
            "name": "ask_weather",
            "confidence": 1
          },
          "intent_ranking": [
            {
              "name": "ask_weather",
              "confidence": 1
            }
          ],
          "entities": []
        },
        "followup_action": null,
        "paused": false,
        "events": [
          {
            "event": "action",
            "timestamp": 1599850576.654908,
            "name": "action_session_start",
            "policy": null,
            "confidence": null
          },
          {
            "event": "session_started",
            "timestamp": 1599850576.654916
          },
          {
            "event": "action",
            "timestamp": 1599850576.654928,
            "name": "action_listen",
            "policy": null,
            "confidence": null
          },
          {
            "event": "user",
            "timestamp": 1599850576.655345,
            "text": "/ask_weather",
            "parse_data": {
              "text": "/ask_weather",
              "intent": {
                "name": "ask_weather",
                "confidence": 1
              },
              "intent_ranking": [
                {
                  "name": "ask_weather",
                  "confidence": 1
                }
              ],
              "entities": []
            },
            "input_channel": "facebook",
            "message_id": "3f2f2317dada4908b7a841fd3eab6bf9",
            "metadata": {}
          }
        ],
        "active_form": {},
        "latest_action_name": "action_listen"
      },
      "domain": {
        "config": {
          "store_entities_as_slots": true
        },
        "session_config": {
          "session_expiration_time": 60,
          "carry_over_slots_to_new_session": true
        },
        "intents": [
          {
            "greet": {
              "use_entities": true
            }
          },
          {
            "ask_weather": {
              "use_entities": true
            }
          }
        ],
        "entities": [],
        "slots": {
          "location": {
            "type": "rasa.core.slots.UnfeaturizedSlot",
            "initial_value": null,
            "auto_fill": true
          },
          "temperature": {
            "type": "rasa.core.slots.UnfeaturizedSlot",
            "initial_value": null,
            "auto_fill": true
          }
        },
        "responses": {
          "utter_greet": [
            {
              "text": "Hey! How are you?"
            }
          ]
        },
        "actions": ["action_tell_weather", "utter_greet"],
        "forms": []
      },
      "version": "2.0.0"
    }
    ```

## 有效载荷中的字段 {#fields-in-the-payload}

以下是有效载荷中字段的描述。

### `next_action` {#next_action}

`next_action` 字段告诉动作服务器要运行什么动作。动作不必作为类的实现，但必须可以按名称调用。

在示例中，动作服务器应运行 `action_tell_weather` 动作。

### `sender_id` {#sender_id}

`sender_id` 告诉你进行对话的用户的唯一 ID。其格式因输入频道而异。它告诉你有关用户的内容，也取决于输入频道以及频道如何识别用户。

在示例中，`sender_id` 不用于任何事情。

### `tracker` {#tracker}

`tracker` 包含有关对话的信息，包括历史记录和所有槽的记录：

- `sender_id`：与有效负载顶层可用的相同 `sender_id`
- `slots`：对话机器人领域中的每个槽及其当前时间的值
- `latest_message`：最新消息的属性
- `latest_event_time`：最后一个事件添加到追踪器的时间戳
- `followup_action`：调用的动作是强制的后续动作
- `paused`：对话当前是否暂停
- `events`：所有先前[事件](events.md)的列表
- `active_loop`：当前活动表单的名称，如果有
- `active_form`：（废弃，由 `activa_loop` 替代）当前活动表单的名称，如果有
- `latest_action_name`：对话机器人执行的最后一个动作的名称
- `stack`：当前对话栈

在示例中，自定义动作使用 `location` 槽的值（如果已设置）来获取天气预报。

### `domain` {#domain}

`domain` 是 `domain.yml` 文件的 JSON 表示形式。自定义动作不太可能引用其内容，因为他们是静态的并且不指示对话的状态。

你可以控制一个动作是否接受领域，参见[选择性领域](../concepts/domain.md#select-which-actions-should-receive-domain)。

!!! info "缓存领域对象以提高性能"

    从 `3.9.0` 版开始，Rasa 动作服务器将领域对象缓存在动作服务器中。这样做是为了避免每次请求时都通过网络发送领域对象。检查 `domain_digest` 字段以了解有关用于生成领域对象的领域文件的更多信息。

### `domain_digest` {#domain_digest}

用于检查动作服务器中缓存的领域对象是否是最新的。包含由 Rasa 加载的训练模型文件的名称。

从版本 `3.9.0` 开始，Rasa SDK 将领域对象缓存在动作服务器中，因此无需在每次请求时都发送该领域对象。为了确保动作服务器了解最新的域更改，你可以在请求中发送 `domain_digest` 字段。除了 `domain_digest`，你还可以在请求中发送领域对象。动作服务器将比较摘要与其缓存的摘要。如果摘要（动作服务器中保留的摘要和通过网络发送的摘要）不匹配，动作服务器将采取以下措施：

- 如果请求中未发送 `domain`，则如果使用 HTTP 协议，它将使用 `RETRY (449)` 响应代码进行响应；如果使用 gRPC 协议，它将使用 `NOT_FOUND` 错误代码和 `DOMAIN_NOT_FOUND` 错误详细信息进行响应。在下一个请求中，Rasa 服务器将发送领域对象。
- 如果在请求中发送了 `domain`，则动作服务器将使用请求中发送的新领域对象更新缓存的领域对象。

### `version` {#version}

这是 Rasa 服务器的版本。自定义动作也不太可能引用此内容，但如果动作服务器仅与某些 Rasa 版本兼容，你可能会在验证步骤中使用它。

## 自定义动作输出 {#custom-action-output}

Rasa 服务器需要一个 `events` 和 `responses` 字典作为对自定义动作调用的响应。

### `events` {#events}

[事件](events.md)表示动作服务器如何影响对话。在示例中，自定义动作应将最高温度存储在 `temperature` 槽中，因此它需要返回一个 [`slot` 事件](events.md#slot)。要设置槽并不执行任何其他动作，响应负载应如下所示：

```json
{
    "events": [
        {
            "event": "slot",
            "timestamp": null,
            "name": "temperature",
            "value": "30"
        }
    ],
    "responses": []
}
```

请注意，事件将按照列出的顺序应用于追踪器。对于 `slot` 事件，顺序无关紧要，但对于其他事件类型不是。

### `responses` {#responses}

响应可以是[富响应文档](../concepts/responses.md#rich-responses)中描述的任何响应类型。有关预期格式，请参阅 [API 规范](https://rasa.com/docs/rasa/pages/action-server-api/){:target="_blank"}的响应示例。

在示例案例中，你希望向用户发送包含天气预报的消息。要发送常规文本消息，响应负载将如下所示：

```json
{
    "events": [
        {
            "event": "slot",
            "timestamp": null,
            "name": "temperature",
            "value": "30"
        }
    ],
    "responses": [
        {
            "text": "This is your weather forecast!"
        }
    ]
}
```

当这个响应被发送回 Rasa 服务器时，Rasa 会将 `slot` 事件和两个响应应用到追踪器，并将两个消息都返回给用户。

## 特殊动作类型 {#special-action-types}

在某些情况下会自动触发一些特殊的动作类型，即[默认动作](../concepts/default-actions.md)和[槽验证动作](../nlu-based-assistants/slot-validation-actions.md)。这些特殊动作类型具有预定义的命名约定，必须遵循这些约定以保持自动触发行为。

可以通过实现具有完全相同名称的自定义动作来自定义默认动作。请参阅有关[默认动作的文档](../concepts/default-actions.md)来了解每个动作的预期行为。

槽验证动作在每个用户轮次运行，具体取决于表单是否处于活动状态。当表单不活动时应该运行的槽验证动作必须名为 `action_validate_slot_mappings`。当表单处于活动状态时应该运行的槽验证动作必须名为 `validate_<form name>`。这些动作应该只返回 `SlotSet` 事件，并且分别表现得像 Rasa SDK [`ValidationAction` 类](validation-action.md#validationaction-class-implementation)和 [`FormValidationAction` 类](validation-action.md#formvalidationaction-class-implementation)。
