# Jambonz 作为语音网关

!!! info "3.10 版本新特性"

    Jambonz 连接器是 Rasa 3.10 中的新功能，已作为测试版功能发布。

使用此通道将 Rasa 对话机器人连接到 [Jambonz](https://www.jambonz.org/)。Jambonz 是一个语音网关，可以使用 SIP 与 Genesys、Avaya 和 Twilio 等各种客户支持平台交互。

## 基本 Rasa 配置 {#basic-rasa-configuration}

创建或编辑你的 `credentials.yml` 并添加新的频道配置：

```yaml title="credentials.yml"
jambonz:
  # currently no other parameters are needed
```

你可以使用 `rasa run` 运行该对话机器人。要配置 Jambonz 频道，你需要一个指向 Rasa 对话机器人的 URL。可以使用 [ngrok](https://ngrok.com/) 等隧道解决方案来公开对话机器人以供开发。

!!! info "用于开发的机器人 URL"

    访问[此部分](messaging-and-voice-channels.md#testing-channels-on-your-local-machine)以了解如何在本地机器上测试频道时生成所需的机器人 URL。

## 配置 Jambonz {#configuring-jambonz}

要将呼叫路由到你的 Rasa 对话机器人，需要有一个 Jambonz 部署、一个帐户和一个电话号码。

1. 注册 [Jambonz 云](https://jambonz.cloud/)或使用你自己的本地部署。
2. 登录后，创建一个将提供你的电话号码的“Carrier”（例如 twilio）。
3. 创建“Application”。要将呼叫路由到你的对话机器人，Jambonz 需要一个 webhook URL。你需要将对话机器人部署到服务器并将其公开给 Jambonz，或者在开发中使用像 [ngrok](https://ngrok.com/) 这样的隧道解决方案。

    webhook URL 的格式为：`wss://<your-server>/webhooks/jambonz/websocket`。

    使用 ngrok 的情况下如下所示：`wss://recently-communal-duckling.ngrok-free.app/webhooks/jambonz/websocket`。

4. 设置“Phone Number”。“Phone Number”的配置应该指向在上一步中创建的“Application”。

## 用法 {#usage}

### 接收来自用户的消息 {#receiving-messages-from-a-user}

当用户在电话中说话时，Jambonz 会像其他任何频道一样向对话机器人发送一条文本消息（经过语音转文本引擎处理后）。此消息将由 Rasa 解释，然后你可以使用流来推动对话。

### 向用户发送消息 {#sending-messages-to-a-user}

你的对话机器人将像其他任何频道一样用文本消息进行响应。文本转语音引擎将转换文本并将其作为语音消息传递给用户。

以下是一个例子：

```yaml
utter_greet:
  - text: "Hello! isn’t every life and every work beautiful?"
```

!!! note "注意"

    仅允许发送文本消息。语音频道不能使用图片、附件和按钮。

### 处理对话事件 {#handling-conversation-events}

对话机器人还可以处理非语音事件。以下是一些示例：

| 事件    | 意图            | 描述                                                         |
| :------ | :-------------- | :----------------------------------------------------------- |
| `start` | `session_start` | Jambonz 接听电话时会发送此意图。默认情况下，此意图会触发 `pattern_session_start`，可以自定义。 |
| `end`   | -               | 目前，Jambonz 不会在通话结束时发送任何信息。                 |
| `DTMF`  | -               | Jambonz 将以 1.0 的置信度将 DTMF（用户在电话上按下的数字）作为普通文本消息发送。 |

以下是对默认会话启动的一个简单修改，以便用话语向用户打招呼：

```yaml
flows:
  pattern_session_start:
    description: flow used to start the conversation
    name: pattern session start
    nlu_trigger:
    - intent: session_start
    steps:
    - action: utter_greet
```
