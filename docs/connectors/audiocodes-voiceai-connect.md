# Audiocodes VoiceAI Connect

使用此频道将你的 Rasa 连接到 [Audiocodes VoiceAI Connect](https://www.audiocodes.com/solutions-products/voiceai/voiceai-connect)。

## 获取凭据 {#getting-credentials}

要获取凭据，请在 [VoiceAI Connect 门户](https://voiceaiconnect.audiocodes.io/)上创建一个机器人。

1. 在左侧边栏中选择 Bots。
2. 单击 + 号以创建一个新机器人。
3. 选择 Rasa Pro 作为 Bot Framework
4. 设置机器人 URL 并选择令牌值。
5. 完成“Setting credentials”部分并且机器人正在运行时“Validate the bot configuration”。只有当机器人正在运行并且令牌设置正确时，验证才会成功。

!!! info "本地测试时使用隧道解决方案设置机器人 URL"

    访问[此部分](messaging-and-voice-channels.md#testing-channels-on-your-local-machine)以了解如何在本地机器上测试频道时生成所需的机器人 URL。

## 设置凭据 {#setting-credentials}

上面选择的令牌值将在 `credentials.yml` 中使用：

=== "Rasa Pro >= 3.8.x"

    ```yaml
    audiocodes:
      token: "token"
    ```

=== "Rasa Pro <= 3.7.x"

    ```yaml
    rasa_plus.channels.audiocodes.AudiocodesInput:
      token: "token"
    ```

你还可以指定可选参数：

| 参数            | 默认值 | 描述                                                         |
| :-------------- | :----- | :----------------------------------------------------------- |
| `token`         | 无     | 用于验证 Rasa 对话机器人和 VoiceAI 连接之间的通话的令牌      |
| `use_websocket` | `true` | 如果设置为 `true`，Rasa 将通过 Web 套接字发送消息。如果设置为 `false`，Rasa 将通过 http API 调用发送消息。 |
| `keep_alive`    | 120    | 单位：秒。对于每个正在进行的对话，VoiceAI Connect 将定期验证对话在 Rasa 端是否仍然活跃。 |

然后重新启动 Rasa 服务器以使新的频道端点可用。

## 用法 {#usage}

### 接收来自用户的消息 {#receiving-messages-from-a-user}

当用户在电话上讲话时，VoiceAI Connect 将像其他任何频道一样向对话机器人发送一条文本消息（经过语音转文本引擎处理后）。该消息将由 Rasa 解释并由对话引擎处理，响应将发送回 VoiceAI Connect 以转换为语音消息并传递给用户。

### 向用户发送消息 {#sending-messages-to-a-user}

对话机器人将像其他任何频道一样用文本消息进行响应。文本转语音引擎将转换文本并将其作为语音消息传递给用户。

以下是一个例子：

```yaml
utter_greet:
  - text: "Hello! isn’t every life and every work beautiful?"
```

!!! note "注意"

    仅允许发送文本消息。语音频道不能使用图片、附件和按钮。

### 处理对话事件 {#handling-conversation-events}

对话机器人还可以处理非语音事件。以下是一些示例：

| 事件    | 意图               | 描述                                                         |
| :------ | :----------------- | :----------------------------------------------------------- |
| `start` | `vaig_event_start` | VoiceAI 接听电话时会发送此意图。一般来说，对该意图的响应是欢迎或问候消息。[通话上下文](https://techdocs.audiocodes.com/voice-ai-connect/#VAIG_Combined/call-initiation.htm?TocPath=Bot%20integration%7CBasic%20behavior%7C_____1)将通过实体提供。 |
| `end`   | `vaig_event_end`   | 通话结束时，VoiceAI 会发送此意图。可以使用它来调用更新通话信息的动作。 |
| `DTMF`  | `vaig_event_DTMF`  | VoiceAI 将在收到 DTMF 音调时发送此意图（即用户按下手机键盘上的数字）。发送的数字将在 `value` 实体中传递。 |

一般模式是，对于发送的每个 `event`，对话机器人都会收到 `vaig_event_<event>` 意图，其中包含实体中的上下文信息。

如下是在发起对机器人的呼叫时发送问候消息的简单规则：

```yaml
- rule: New call
  steps:
    - intent: vaig_event_start
    - action: utter_greet
```

查看 [VoiceAI Connect 文档](https://techdocs.audiocodes.com/voice-ai-connect/#VAIG_Combined/voiceai_connect.htm?TocPath=VoiceAI%2520Connect%257C_____0)以获取详尽的事件列表。

### 配置呼叫 {#configuring-calls}

你可以将事件从 Rasa 发送到 VoiceAI Connect 以更改当前呼叫配置。例如，你可能希望在用户保持沉默超过 5 秒时收到通知，或者你可能需要自定义 VoiceAI Connect 发送 DTMF 数字的方式。

呼叫配置事件通过自定义消息发送，并且特定于当前对话（有时是消息）。这意味着它们必须是故事或规则的一部分，以便相同的行为适用于所有对话。

那些 Rasa 响应不会说出任何内容，它们只是配置语音网关。最好以不同的方式命名它们，例如在它们前面加上 `utter_config_<what_it_does>`。

所有支持的事件都在 [VoiceAI Connect 文档](https://techdocs.audiocodes.com/voice-ai-connect/#VAIG_Combined/voiceai_connect.htm?TocPath=VoiceAI%2520Connect%257C_____0)中详尽记录。我们将在这里看一个例子来说明自定义消息和事件的使用。

示例：更改密码

在此示例中，我们创建了一个流，允许用户更改密码。

```yaml
- rule: Set pin code
  steps:
    # User says "I want to change my pin code"
    - intent: set_pin_code
    # Send the noUserInput configuration event
    - action: utter_config_no_user_input
    # Send the DTMF format configuration event
    - action: utter_config_dtmf_pin_code
    # A standard Rasa form to collect the pin code from the user
    - action: pin_code_form
    - ...
```

在领域中，可以添加 `utter_config_<config_event>` 响应：

[noUserInput 事件](https://techdocs.audiocodes.com/voice-ai-connect/#VAIG_Combined/inactivity-detection.htm?TocPath=Bot%2520integration%257CReceiving%2520notifications%257C_____3)

```yaml
utter_config_no_user_input:
  - custom:
      type: event
      name: config
      sessionParams:
        # If user stays silent for 5 seconds or more, the notification will be sent
        userNoInputTimeoutMS: 5000
        # If you want to allow for more than one notification during a call
        userNoInputRetries: 2
        # Enable the noUserInput notification
        userNoInputSendEvent: true
```

[DTMF 事件](https://techdocs.audiocodes.com/voice-ai-connect/#VAIG_Combined/receive-dtmf.htm?TocPath=Bot%2520integration%257CReceiving%2520notifications%257C_____2)

```yaml
utter_config_dtmf_pin_code:
- custom:
   type: event
   name: config
   sessionParams:
     # Enable grouped collection (i.e will send all digits in a single payload)
     dtmfCollect: true
     # If more than 5 secs have passed since a digit was pressed,
     # the input is considered completed and will be sent to the bot
     dtmfCollectInterDigitTimeoutMS: 5000
     # If 6 digits are collected, VoiceAI will send those 6 digits
     # even if the user keeps pressing buttons
     dtmfCollectMaxDigits: 6
     # If the user presses '#' the input is considered complete
     dtmfCollectSubmitDigit: "#"
```

现在，你可以配置 `pin_code_form` 中的 `pin_code` 槽，以使用 `vaig_event_DTMF` 意图从 `value` 实体中提取邮政编码：

```yaml
pin_code:
  type: text
  influence_conversation: false
  mappings:
    - type: from_entity
      entity: value
      intent: vaig_event_DTMF
      not_intent: vaig_event_noUserInput
      conditions:
        - active_loop: pin_code_form
          requested_slot: pin_code
```

请注意 `vaig_event_noUserInput` 是如何在 `not_intent` 字段中声明的。

由于 `vaig_event_noUserInput` 意图是在用户按照我们的配置保持沉默时由 VoiceAI Connect 发送的，因此我们必须停用该表单，以便我们可以从规则或故事中接听对话并妥善处理失败。

在以下示例中，如果我们在 `pin_code_form` 循环处于活动状态时收到 `vaig_event_noUserInput` 意图（即用户保持沉默），我们只需取消当前流。

```yaml
- rule: Set pin code - happy path
  steps:
    - intent: set_pin_code
    - action: utter_config_no_user_input
    - action: utter_config_dtmf_pin_code
    - action: pin_code_form
    - active_loop: pin_code_form
    - active_loop: null
    - slot_was_set:
        - requested_slot: null
    - action: utter_pin_code_changed
    - action: action_pin_code_cleanup

- rule: Set pin code - no response - cancel.
  condition:
    - active_loop: pin_code_form
  steps:
    - intent: vaig_event_noUserInput
    - action: utter_cancel_set_pin_code
    - action: action_deactivate_loop
    - active_loop: null
```
