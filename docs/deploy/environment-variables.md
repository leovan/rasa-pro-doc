# 添加环境变量

Rasa 通过环境变量提供高级配置。在此处查找完整参考。

要向容器添加额外的环境变量，请将环境变量添加到 `values.yml` 中服务上的 `extraEnvs` 数组中。这些值可以直接提供或引用[密钥](../production/secrets-managers.md)。`extraEnvs` 参数适用于所有服务。例如：

```yaml
# rasa: Settings common for all Rasa containers
rasa:
  # other configured parameters
  # ...
  extraEnvs:
    - name: "LOG_LEVEL"
      value: "warning"
```

## Rasa Pro {#rasa-pro}

运行 Rasa Pro 时，`RASA_PRO_LICENSE` 环境变量需要包含 Rasa Pro 许可证密钥。

### 支持服务 {#backing-services}

| 环境变量                          | 描述                                                         | 默认值           |
| :-------------------------------- | :----------------------------------------------------------- | :--------------- |
| `POSTGRESQL_SCHEMA`               | [`SQLTrackerStore`](../production/tracker-stores.md#sqltrackerstore) 的 postgres 模式。 | `"public"`       |
| `POSTGRESQL_POOL_SIZE`            | [`SQLTrackerStore`](../production/tracker-stores.md#sqltrackerstore) 打开连接数的限制。 | `50`             |
| `POSTGRESQL_MAX_OVERFLOW`         | [`SQLTrackerStore`](../production/tracker-stores.md#sqltrackerstore) 连接池的[最大溢出](https://docs.sqlalchemy.org/en/13/core/pooling.html.md#sqlalchemy.pool.QueuePool.params.max_overflow)。 | `100`            |
| `RABBITMQ_SSL_CLIENT_CERTIFICATE` | [`PikaEventBroker`](../production/event-brokers.md#pika-event-broker) 的 SSL 客户端证书路径。 |                  |
| `RABBITMQ_SSL_CLIENT_KEY`         | [`PikaEventBroker`](../production/event-brokers.md#pika-event-broker) 的 SSL 客户端密钥路径。 |                  |
| `RASA_ENVIRONMENT`                | [`PikaEventBroker`](../production/event-brokers.md#pika-event-broker) 和 [`KafkaEventBroker`](../production/event-brokers.md#kafka-event-broker) 的 Rasa 环境。 |                  |
| `SECRET_MANAGER`                  | [密钥管理器](../production/secrets-managers.md) 的类型。     | `"vault"`        |
| `TICKET_LOCK_LIFETIME`            | 票证锁的生命周期（以秒为单位）。它配置 [锁存储](../production/lock-stores.md)。 | `60`             |
| `VAULT_URL`                       | [HashiCorp Vault](../production/secrets-managers.md#hashicorp-vault-secrets-manager) 的 URl。 |                  |
| `VAULT_TOKEN`                     | 用于向 [HashiCorp Vault](../production/secrets-managers.md#hashicorp-vault-secrets-manager) 进行身份验证的令牌。 |                  |
| `VAULT_NAMESPACE`                 | [HashiCorp Vault](../production/secrets-managers.md#hashicorp-vault-secrets-manager) 的命名空间。 |                  |
| `VAULT_RASA_SECRETS_PATH`         | [HashiCorp Vault](../production/secrets-managers.md#hashicorp-vault-secrets-manager) 中 Rasa 密钥的路径。 | `"rasa-secrets"` |
| `VAULT_TRANSIT_MOUNT_POINT`       | [HashiCorp Vault](../production/secrets-managers.md#hashicorp-vault-secrets-manager) 的挂载点。 |                  |

### 模型存储 {#model-storage}

以下部分介绍了在云提供者上配置模型存储的环境变量：

- [Amazon S3 存储](../production/model-storage.md#amazon-s3-storage)
- [Google Cloud Storage](../production/model-storage.md#google-cloud-storage)
- [Azure 存储](../production/model-storage.md#azure-storage)

### 对话管理 {#dialogue-management}

| 环境变量                          | 描述                                                         | 默认值  |
| :-------------------------------- | :----------------------------------------------------------- | :------ |
| `RASA_DUCKLING_HTTP_URL`          | 为 [`DucklingEntityExtractor`](../nlu-based-assistants/components.md#ducklingentityextractor) 提供支持的 Duckling 服务的 URL。 |         |
| `MAX_NUMBER_OF_PREDICTIONS`       | 每条用户消息后的预测最大值。请参阅[动作选择](../concepts/policies/policy-overview.md#action-selection)。 | `10`    |
| `TF_GPU_MEMORY_ALLOC`             | TensorFlow 的 GPU 配置。请参阅 [配置 TensorFlow](../nlu-based-assistants/tuning-your-model.md#configuring-tensorflow)。 |         |
| `TF_INTER_OP_PARALLELISM_THREADS` | TensorFlow 中独立操作之间并行使用的线程数。请参阅[配置 TensorFlow](../nlu-based-assistants/tuning-your-model.md#configuring-tensorflow)。 |         |
| `TF_INTRA_OP_PARALLELISM_THREADS` | 单个 TensorFlow 操作中用于并行的线程数。请参阅[配置 TensorFlow](../nlu-based-assistants/tuning-your-model.md#configuring-tensorflow)。 |         |
| `TF_DETERMINISTIC_OPS`            | 配置 TensorFlow 操作以确定性运行。请参阅[配置 TensorFlow](../nlu-based-assistants/tuning-your-model.md#configuring-tensorflow)。 | `false` |

### 可观察性 {#observability}

| 环境变量                          | 描述                                                         | 默认值    |
| :-------------------------------- | :----------------------------------------------------------- | :-------- |
| `LOG_LEVEL`                       | Rasa 和 Rasa Pro 的日志级别。                                | `"INFO"`  |
| `LOG_LEVEL_LIBRARIES`             | 第三方库的日志级别。更多信息请参见[此处](../command-line-interface.md#log-level)。 | `"ERROR"` |
| `LOG_LEVEL_KAFKA`                 | `kafka` 库的日志级别。更多信息请见此处。                     | `"ERROR"` |
| `LOG_LEVEL_RABBITMQ`              | `rabbitmq` 库的日志级别。更多信息请见[此处](../command-line-interface.md#log-level)。 | `"ERROR"` |
| `LOG_LEVEL_FAKER`                 | `faker` 库的日志级别。更多信息请见[此处](../command-line-interface.md#log-level)。 | `"ERROR"` |
| `LOG_LEVEL_PRESIDIO`              | `presidio` 库的日志级别。更多信息请见[此处](../command-line-interface.md#log-level)。 | `"ERROR"` |
| `LOG_LEVEL_LLM`                   | 所有 `LLM` 组件的日志级别。更多信息请见[此处](../command-line-interface.md#log-level-llm-components)。 | `"DEBUG"` |
| `LOG_LEVEL_LLM_COMMAND_GENERATOR` | `LLMCommandGenerator` 提示的日志级别。更多信息请参见[此处](../command-line-interface.md#log-level-llm-components)。 | `"DEBUG"` |
| `LOG_LEVEL_LLM_ENTERPRISE_SEARCH` | `EnterpriseSearchPolicy` 提示的日志级别。更多信息请参见[此处](../command-line-interface.md#log-level-llm-components)。 | `"DEBUG"` |
| `LOG_LEVEL_LLM_INTENTLESS_POLICY` | `IntentlessPolicy` 提示的日志级别。更多信息请参见[此处](../command-line-interface.md#log-level-llm-components)。 | `"DEBUG"` |
| `LOG_LEVEL_LLM_REPHRASER`         | `ContextualResponseRephraser` 提示的日志级别。更多信息请参见[此处](../command-line-interface.md#log-level-llm-components)。 | `"DEBUG"` |
| `RASA_TELEMETRY_ENABLED`          | 切换遥测报告。更多信息请见[此处](../telemetry/telemetry.md)。 | `true`    |
| `RASA_TELEMETRY_DEBUG`            | 切换遥测报告的调试信息。                                     | `false`   |
| `RASA_PRO_TELEMETRY_ENABLED`      | 切换遥测报告。更多信息请见[此处](../telemetry/telemetry.md)。 | `true`    |
| `RASA_PRO_TELEMETRY_DEBUG`        | 切换遥测报告的调试信息。                                     | `false`   |
| `TRACING_SERVICE_NAME`            | 发送追踪时的上层服务名称。更多信息请参见[此处](../operating/tracing.md)。 | `"rasa"`  |

### 高级配置 {#advanced-configuration}

| 环境变量                                       | 描述                                                         | 默认值           |
| :--------------------------------------------- | :----------------------------------------------------------- | :--------------- |
| `COMPRESS_ACTION_SERVER_REQUEST`               | 切换发送到[动作服务器](../action-server/actions.md) 的 HTTP 请求的压缩。 | `false`          |
| `RASA_MAX_CACHE_SIZE`                          | 训练缓存的最大大小（以 MB 为单位）。                         | `1000`           |
| `RASA_CACHE_NAME`                              | 训练缓存的文件名。                                           | `"cache.db"`     |
| `RASA_CACHE_DIRECTORY`                         | 训练缓存的目录。                                             | `".rasa/cache/"` |
| `RASA_SHELL_STREAM_READING_TIMEOUT_IN_SECONDS` | [`rasa shell`](../command-line-interface.md#rasa-shell) 的流读取超时时间，以秒为单位。 | `10`             |
| `SANIC_BACKLOG`                                | [HTTP 服务器](../production/rest-api.md) 和 [NLG 服务器](../production/nlg.md) 在拒绝新连接之前将允许的未接受连接数。 | `100`            |
| `SANIC_WORKERS`                                | 启用 [HTTP 服务器](../production/rest-api.md)时的 HTTP 工作进程的数量。 | `1`              |
| `READ_YAML_FILE_CACHE_MAXSIZE`                 | 用于读取和解析 YAML 文件的 LRU（最近最少使用）缓存的最大大小。 | `256`            |

## Rasa Pro 服务 {#rasa-pro-services}

Rasa Pro 服务 docker 容器支持通过多个环境变量进行配置。下表列出了可用的环境变量：

| 环境变量                  | 描述                                                         | 默认值             |
| :------------------------ | :----------------------------------------------------------- | :----------------- |
| `RASA_PRO_LICENSE`        | **必需**。Rasa Pro 服务的许可证密钥。                        |                    |
| `KAFKA_BROKER_ADDRESS`    | **必需**。Kafka 代理的地址。                                 |                    |
| `KAFKA_TOPIC`             | **必需**。Rasa Pro 向该主题发布事件，并从该主题消费事件。    | `rasa_core_events` |
| `LOGGING_LEVEL`           | 设置应用程序的日志级别。有效级别为 DEBUG、INFO、WARNING、ERROR、CRITICAL。（从 3.0.2 开始可用） | `INFO`             |
| `RASA_ANALYTICS_DB_URL`   | 存储分析数据的数据湖的 URL。                                 |                    |
| `KAFKA_SASL_MECHANISM`    | 用于身份验证的 SASL 机制。                                   | `PLAIN`            |
| `KAFKA_SASL_USERNAME`     | 用于 SASL 认证的用户名。                                     |                    |
| `KAFKA_SASL_PASSWORD`     | 用于 SASL 认证的密码。                                       |                    |
| `KAFKA_SECURITY_PROTOCOL` | 与 Kafka 通信时使用的安全协议。支持的机制为 `PLAINTEXT` 和 `SASL_PLAINTEXT`。 | `PLAINTEXT`        |
| `KAFKA_SSL_CA_LOCATION`   | 用于连接 Kafka 的 SSL CA 证书的文件路径（从 `3.1.0b1` 开始可用） |                    |

## Rasa Studio {#rasa-studio}

这些环境变量优先于使用 `rasa studio config` 创建的 `global.yml` 文件中的值。

| 环境变量                         | 描述                                                         | 默认值 |
| :------------------------------- | :----------------------------------------------------------- | :----- |
| `RASA_STUDIO_AUTH_SERVER_URL`    | 身份验证服务器的 URL。                                       |        |
| `RASA_STUDIO_CLI_STUDIO_URL`     | Studio 数据端点的 URL。                                      |        |
| `RASA_STUDIO_CLI_REALM_NAME_KEY` | 保存有关客户端 ID 和客户端密钥的数据的 Keycloak realm 的名称。 |        |
| `RASA_STUDIO_CLI_CLIENT_ID_KEY`  | 使用 Keycloak 验证 `rasa studio` CLI 工具的客户端 ID         |        |

## Rasa SDK {#rasa-sdk}

Rasa SDK docker 容器支持通过多个环境变量进行配置。下表列出了可用的环境变量：

| 环境变量                      | 描述                                                         | 默认值  |
| :---------------------------- | :----------------------------------------------------------- | :------ |
| `ACTION_SERVER_SANIC_WORKERS` | 动作服务器中的 Sanic HTTP 工作程序的数量。                   | `1`     |
| `LOG_LEVEL_LIBRARIES`         | 第三方库的日志级别。请参阅[日志级别配置](../command-line-interface.md#log-level)。 | `ERROR` |
