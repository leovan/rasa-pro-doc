# 自定义动作

自定义动作可以运行你需要的任何代码，包括 API 调用、数据库查询等。例如可以打开灯、将事件添加到日历、检查用户的银行余额或你能想到的任何其他动作。

有关如何实现自定义动作的详细信息，请参阅 [SDK 文档](../action-server/running-action-server.md)。想要在故事中使用的任何自定义动作都应添加到[领域](domain.md)的 `actions` 部分中。

当对话引擎预测要执行的自定义动作时，它将调用动作服务器，并提供以下信息：

```json
{
  "next_action": "string",
  "tracker": {
    "conversation_id": "default",
    "sender_id": "string",
    "slots": {},
    "latest_message": {},
    "latest_event_time": 1537645578.314389,
    "followup_action": "string",
    "paused": false,
    "events": [],
    "latest_input_channel": "rest",
    "active_loop": {},
    "latest_action": {}
  },
  "domain": {
    "config": {},
    "session_config": {},
    "intents": [],
    "entities": [],
    "slots": {},
    "responses": {},
    "actions": [],
    "forms": {},
    "e2e_actions": []
  },
  "version": "version"
}
```

动作服务器应该以事件和响应列表进行响应：

```json
{
  "events": [{}],
  "responses": [{}]
}
```

## 由对话机器人直接运行自定义动作

!!! info "3.10 版本新特性"

    你现在可以直接在 Rasa 对话机器人上运行 Python 自定义动作，而无需单独的动作服务器。

构建对话机器人通常涉及执行自定义逻辑，如 API 调用或数据库查询。传统上，这需要一个动作服务器。但是，为了获得更集成、更高效的方法，你可以直接在 Rasa 对话机器人上运行用 Python 编写的自定义动作。这简化了部署并提高了对话机器人的整体效率。

### 直接自定义动作执行的好处 {#benefits-of-direct-custom-action-execution}

- **简化架构**：无需单独的动作服务器，从而降低了架构和部署过程的复杂性。
- **更低的延迟**：通过消除与外部动作服务器通信所需的网络往返来缩短响应时间。
- **统一环境**：在同一环境中执行对话机器人的所有组件，使调试和本地开发更加顺畅。
- **成本效率**：降低基础设施成本，因为无需维护额外的服务器来处理自定义动作。

### 直接自定义动作执行的缺点 {#disadvantages-of-direct-custom-action-execution}

- **付出更对来保护 Rasa 环境**：Rasa 对话机器人需要访问自定义动作所需的相同敏感资源来访问远程服务（即令牌、凭据），这可能会带来安全风险。因此，应该通过在更受保护的环境中运行来妥善保护 Rasa 实例。

## 如何配置该功能 {#how-to-configure-the-feature}

要使用此功能，你需要更新 `endpoints.yml` 文件的 `action_endpoint` 部分。添加 `actions_module` 字段并指定自定义动作 Python 包的路径。此包将被导入并由 Rasa 对话机器人直接用于运行动作。

例如，假设如下项目结构：

```txt
my_project/
├── actions/
│   ├── __init__.py
│   ├── actions.py
├── data/
├── models/
├── tests/
├── domain.yml
├── config.yml
├── credentials.yml
├── endpoints.yml
```

你可以在 `endpoints.yml` 文件中配置 `actions_module` 字段，如下所示：

```yaml title="endpoints.yml"
action_endpoint:
    actions_module: "actions"
```

此配置与 `url` 字段互斥，你通常使用 `url` 字段指向外部动作服务器。如果同时指定了 `url` 和 `action_module`，则将优先考虑 `action_module`。

!!! info "信息"

    每次更新自定义动作代码时，都必须重新启动 Rasa 对话机器人以通过 `rasa run` 或 `rasa inspect` 命令反映更改。
