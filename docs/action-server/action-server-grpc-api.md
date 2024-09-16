# Rasa 动作服务器 gRPC API

## 输入和输出 {#input-and-output}

以下是 Rasa 动作服务器的 gRPC API 定义。该 API 在 Rasa SDK 存储库中的 [`proto/action_webhook.proto`](https://github.com/RasaHQ/rasa-sdk/blob/main/proto/action_webhook.proto) 文件中定义。

```proto title="action_webhook.proto"
syntax = "proto3";

package action_server_webhook;
import "google/protobuf/struct.proto";

service ActionService {
    rpc Webhook (WebhookRequest) returns (WebhookResponse);
    rpc Actions (ActionsRequest) returns (ActionsResponse);
}

message ActionsRequest {}

message ActionsResponse {
    repeated google.protobuf.Struct actions = 1;
}

message Tracker {
    string sender_id = 1;
    google.protobuf.Struct slots = 2;
    google.protobuf.Struct latest_message = 3;
    repeated google.protobuf.Struct events = 4;
    bool paused = 5;
    optional string followup_action = 6;
    map<string, string> active_loop = 7;
    optional string latest_action_name = 8;
    repeated google.protobuf.Struct stack = 9;
}

message Intent {
    string string_value = 1;
    google.protobuf.Struct dict_value = 2;
}

message Entity {
    string string_value = 1;
    google.protobuf.Struct dict_value = 2;
}

message Action {
    string string_value = 1;
    google.protobuf.Struct dict_value = 2;
}

message Domain {
    google.protobuf.Struct config = 1;
    google.protobuf.Struct session_config = 2;
    repeated Intent intents = 3;
    repeated Entity entities = 4;
    google.protobuf.Struct slots = 5;
    google.protobuf.Struct responses = 6;
    repeated Action actions = 7;
    google.protobuf.Struct forms = 8;
    repeated google.protobuf.Struct e2e_actions = 9;
}

message WebhookRequest {
    string next_action = 1;
    string sender_id = 2;
    Tracker tracker = 3;
    Domain domain = 4;
    string version = 5;
    optional string domain_digest = 6;
}

message WebhookResponse {
    repeated google.protobuf.Struct events = 1;
    repeated google.protobuf.Struct responses = 2;
}
```

## 错误处理 {#error-handling}

当 Rasa 服务器通过 gRPC 协议与动作服务器通信时发生错误，动作服务器应返回包含正确 gRPC 状态代码和详细信息的适当错误消息。

以下是可能的错误场景以及动作服务器的预期错误响应。

!!! info "Rasa Python 动作服务器错误处理支持"

    Rasa SDK 中提供的 Rasa Python 动作服务器已经处理了这些错误情况并向 Rasa 服务器返回了适当的错误消息。如果你使用自定义动作服务器，则需要相应地处理这些错误情况。

### 未找到自定义动作 {#custom-action-is-not-found}

如果动作服务器未找到自定义动作，则应返回：

- gRPC 状态代码为 `NOT_FOUND` 的错误
- 错误消息中的详细信息格式如下：

    ```json
    {
        "action_name" : { "type":  "string" },
        "message" : { "type": "string" },
        "resource_type" : "ACTION"
    }
    ```

    其中 `action_name` 是未找到的动作的名称，`message` 是可读的错误消息，`resource_type` 是未找到的资源的类型。

### 自定义动作在执行过程中失败 {#custom-action-failed-during-execution}

如果自定义动作在执行过程中失败，它应该返回：

- gRPC 状态代码为 `INTERNAL` 的错误
- 错误消息中的详细信息格式如下：

    ```json
    {
        "action_name" : { "type":  "string" },
        "message" : { "type": "string" }
    }
    ```

    其中 `action_name` 是失败的动作的名称，`message` 是可读的错误消息。

### 未找到领域 {#domain-is-not-found}

如果动作服务器未找到与对话机器人的 `domain_digest` 对应的领域，则应返回：

- gRPC 状态代码为 `NOT_FOUND` 的错误
- 错误消息中的详细信息格式如下：

    ```json
    {
        "action_name" : { "type":  "string" },
        "message" : { "type": "string" },
        "resource_type" : "DOMAIN"
    }
    ```

    其中 `action_name` 是要调用的动作的名称，`message` 是可读的错误消息，`resource_type` 是未找到的资源的类型。
