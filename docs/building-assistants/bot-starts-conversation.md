# 对话机器人开始对话

本指南向你展示如何让对话机器人主动开始与用户的对话。这是通过代表用户发送触发会话启动模式的消息 `/session_start` 来实现的。

## 修改模式会话启动 {#modifying-the-pattern-session-start}

模式会话启动是一个由意图 `/session_start` 触发的预构建[流](../concepts/flows.md)。你可以按如下方式修改它：

```yaml title="patterns.xml"
flows:
  pattern_session_start:
    description: Flow for starting the conversation
    name: pattern session start
    nlu_trigger:
      - intent: session_start
    steps:
      - action: utter_hello
```

## 触发欢迎消息 {#triggering-the-welcome-message}

要触发欢迎流，你需要代表用户发送消息 `/session_start`。具体动作取决于用户与对话机器人交谈的频道。

### 使用聊天小部件 {#using-a-chat-widget}

某些聊天小部件（包括 [Rasa 聊天小部件](https://rasa.leovan.tech/connectors/your-own-website#chat-widget)）提供在用户打开聊天窗口时发送结构化消息的选项。配置此选项以发送消息 `/session_start`，用户将看到会话启动模式中定义的动作。

### 使用 API 调用 {#using-an-api-call}

如果你使用的是自定义频道或 [REST](https://rasa.leovan.tech/connectors/your-own-website#restinput) 频道，则可以发送带有有效负载的 POST 请求，如下所示：

```json
{
  "sender": "user1234",  // sender ID of the user sending the message
  "message": "/session_start"
}
```
