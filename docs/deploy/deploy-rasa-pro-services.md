# 部署 Rasa Pro 服务

本页介绍如何使用 Rasa Pro Helm Chart 部署 Rasa Pro 服务容器。

## Rasa Pro 服务设置 {#rasa-pro-services-setup}

安装 Rasa Pro 服务需要部署 docker 容器。容器可以部署在与 Rasa Pro 不同的环境中。两个部署都需要能够连接到同一个 Kafka 集群才能进行通信。

### 先决条件 {#prerequisites-1}

Rasa Pro 服务部署需要连接到

- Kafka 事件代理。
- 数据仓库（例如 PostgreSQL）。

Kafka 部署应为生产部署。我们建议使用托管部署，例如使用 [Amazon Managed Streaming for Apache Kafka](https://aws.amazon.com/msk/)。

#### 系统要求 {#system-requirements}

最低硬件要求包括有关安装和使用 Rasa Pro 服务所需要求的信息。硬件要求取决于平均对话次数和预期工作量。你的确切需求可能更多，具体取决于你的工作量。

以下是建议的最低硬件配置：

- CPU：2 vCPU
- 内存：4 GB
- 磁盘空间：10 GB

这些要求对应于 AWS 上的 `t3.medium` 实例类型。

#### 许可证 {#license}

运行 Rasa Pro 服务需要有效的许可证。你需要将许可证作为环境变量提供给服务。

## 使用 Rasa Pro Helm Chart 进行部署 {#deploy-using-rasa-pro-helm-chart}

### 先决条件 {#prerequisites-2}

需要配置 Rasa Pro 以将数据流式传输到 Kafka。

Kafka 的配置应位于用于部署 Rasa Pro 的 `values.yml` 的 `rasa.settings.endpoints.eventBroker` 部分。它看起来像下面的示例：

```yaml
rasa:
  settings:
    endpoints:
      eventBroker:
        enabled: true
        type: kafka
        security_protocol: SASL_SSL
        sasl_mechanism: SCRAM-SHA-512
        topic: rasa-assistant
        url: <KAFKA-BROKER-URL>
        sasl_username: <KAFKA-USERNAME>
        sasl_password: <KAFKA-PASSWORD>
        ssl_check_hostname: true
        ssl_cafile: /app/CAroot.pem
```

可以在 [Kafka Event Broker](../production/event-brokers.md) 文档中找到不同安全协议和其他参数的配置示例。

### 更新 values.yml {#update-valuesyml}

#### 指定镜像拉取密钥和许可证 {#specify-image-pull-secret-and-license}

在 `values.yml` 中指定镜像拉取密钥和 Rasa Pro 许可证，如下所示：

```yaml
# -- rasaProLicense is license key for Rasa Pro Services.
rasaProLicense:
  # name of kubernetes secret
  secretName: rasapro-license
  secretKey: licensekey
```

#### 启用 Rasa Pro 服务 {#enable-rasa-pro-services}

通过更新 `values.yml` 使用 Rasa Pro Helm Chart 启用 Rasa Pro 服务，如下所示：

```yaml
rasaProServices:
  enabled: true
  image:
    repository: "europe-west3-docker.pkg.dev/rasa-releases/rasa-pro/rasa-pro-services"
    # specify version in image tag
    tag: "3.2.0-latest"
```

#### 通过环境变量进行配置 {#configuration-through-environment-variables}

使用[环境变量](environment-variables.md#rasa-pro-services)，配置 Rasa Pro 服务以从 Rasa Pro 发布事件的 Kafka 主题中使用。配置应添加到 `values.yml` 中的 `rasaProServices.environmentVariables` 中。

!!! info "连接到安全的 Kafka 实例"

    可以通过在 Rasa Pro docker 容器上设置以下环境变量来配置与安全 Kafka 实例的连接：`KAFKA_SASL_MECHANISM`、`KAFKA_SASL_USERNAME`、`KAFKA_SASL_PASSWORD` 和 `KAFKA_SECURITY_PROTOCOL`。目前不支持使用 Kafka `truststores` 和 `keystores`。

#### 使用 Kubernetes 密钥进行配置 {#configure-using-a-kubernetes-secret}

要将 Kubernetes 密钥用于环境变量，可以像以下示例一样使用 `secret` 指定它：

```yaml
RASA_ANALYTICS_DB_URL:
# this uses a kubernetes secret
  secret:
    name: analytics-db-url
    key: dburl
```

你的 `values.yml` 中的 Rasa Pro 服务部分将如下例所示：

```yaml
rasaProServices:
  enabled: true
  image:
    repository: "europe-west3-docker.pkg.dev/rasa-releases/rasa-pro/rasa-pro-services"
    # specify version in image tag
    tag: "3.2.0-latest"
  environmentVariables:
    # Address of the Kafka broker.
    KAFKA_BROKER_ADDRESS:
      value: <KAFKA-BROKER-URL>
    # Topic Rasa Pro publishes events to and Rasa Pro consumes from
    KAFKA_TOPIC:
      value: rasa-assistant
    # URL of the data lake to store analytics data in
    RASA_ANALYTICS_DB_URL:
    # this uses a kubernetes secret
      secret:
        name: analytics-db-url
        key: dburl
    # Security protocol to use for communication with Kafka
    KAFKA_SECURITY_PROTOCOL:
      value: SASL_SSL
    # SASL mechanism to use for authentication.
    KAFKA_SASL_MECHANISM:
      value: SCRAM-SHA-512
    # Username for SASL authentication.
    KAFKA_SASL_USERNAME:
      secret:
        name: kafka-secrets
        key: username
    # Password for SASL authentication
    KAFKA_SASL_PASSWORD:
      secret:
        name: kafka-secrets
        key: password
    # Filepath for SSL CA Certificate that will be used to connect with Kafka
    KAFKA_SSL_CA_LOCATION:
      value: /app/CAroot.pem
```

### 使用 Helm 部署 {#deploy-with-helm}

要更新 Helm 部署，可以使用：

```shell
helm upgrade \
    --namespace <your namespace> \
    --values values.yml \
    <release name> \
    rasa-<version>.tgz
```

### 健康检查端点 {#healthcheck-endpoint}

Rasa Pro 服务在 `/healthcheck` 处启动健康检查端点。如果服务健康，端点将返回 `200` 状态代码。如果返回任何其他状态代码或端点无法访问，则服务不健康。

`/healthcheck` 的示例响应：

```json
{
  "details": {
    "analytics-consumer": {
      "alive": 1,
      "isHealthy": true
    }
  },
  "isHealthy": true
}
```

### 升级版本 {#upgrading-versions}

要升级到最新版本的 Rasa Pro 服务，必须遵循以下步骤：

- 阅读有关重大更改的[变更日志文档](../rasa-pro-changelog.md)。
- 下载新的 docker 容器并运行它。

!!! info "容器启动"

    请注意，容器可能需要一些时间才能启动，因为它在启动过程中运行数据库架构迁移。

    如果迁移失败，容器将被关闭。

## 环境变量 {#environment-variables}

有关可与 Rasa Pro 服务一起使用的环境变量的完整列表，请参阅[环境变量](environment-variables.md#rasa-pro-services)文档。
