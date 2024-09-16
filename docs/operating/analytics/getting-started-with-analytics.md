# 开始使用分析

在所选工具中可视化和处理 Rasa 对话机器人指标。

Analytics Data Pipeline 可帮助你在所选工具（BI 工具、数据仓库）中可视化和处理 Rasa 对话机器人指标。通过对生产对话机器人及其对话的可视化和分析，你可以评估投资回报率并随着时间的推移提高对话机器人的性能。

<figure markdown>
  ![](../../images/operating/analytics/graph-number-sessions-channel.png){ width="45%" align=left }
  ![](../../images/operating/analytics/graph-escalation-rate.png){ width="45%" align=right }
</figure>

<figure markdown>
  ![](../../images/operating/analytics/graph-intent-distribution.png){ width="45%" align=left }
  ![](../../images/operating/analytics/graph-top-5-intents.png){ width="45%" align=right }
</figure>

衡量 Rasa 对话机器人的成功并进行战略投资将带来更好的业务成果。它还将帮助你了解最终用户的需求并更好地为他们服务。

<figure markdown>
  ![](../../images/operating/analytics/rasa-pro-analytics-overview.png)
</figure>

Rasa Pro 将使用 Kafka 连接到你的生产对话机器人。一旦 docker 容器成功部署并连接到你的数据仓库和 Kafka 实例，管道就会计算分析。

## 指标类型 {#types-of-metrics}

从你的对话机器人收集的指标大致可分为以下几类

- 用户分析：对话机器人的用户是谁，他们对此有何感受？示例：人口统计、频道、情绪分析。
- 使用情况分析：对话机器人的整体健康状况如何，以及有哪些类型的流量？示例：会话总数、每个会话的时间、错误和错误率。
- 对话分析：对话期间发生了什么？示例：发送的消息数、放弃深度、用户引入的主题数、前 N 个意图。
- 业务分析：对话机器人在业务目标方面的表现如何？示例：每个 LoB 的对话机器人投资回报率、对话机器人与代理的时间比较、遏制率。

在此版本的分析管道中，可以测量以下指标：

| 指标 | 类型 | 说明 |
| :---------------------- | :--------------------- | :-------------------------------------------------------------- |
| 对话数量 | 使用情况分析 | 对话总数 |
| 用户数量 | 使用情况分析 | 用户总数 |
| 会话数量 | 使用情况分析 | 对话机器人总流量 |
| 使用的频道 | 使用情况分析 | 按频道划分的会话 |
| 用户会话计数 | 用户分析 | 用户会话总数或每个用户的平均会话数 |
| 前 N 个意图 | 对话分析 | 所有用户的热门意图 |
| 平均响应时间 | 对话分析 | 对话机器人的平均响应时间 |
| 遏制率 | 业务分析 | 纯粹由对话机器人处理的对话百分比（未交给人工代理） |
| 放弃率 | 业务分析 | 放弃对话的百分比 |
| 升级率 | 业务分析 | 升级到人工代理的对话百分比 |

有关如何提取这些指标的示例，请参阅[示例查询](example-queries.md)。

## 先决条件 {#prerequisites}

- 设置 Rasa Pro 需要 Kafka 的生产部署。我们建议使用 [Amazon Managed Streaming for Apache Kafka](https://aws.amazon.com/msk/)。
- 数据湖的生产部署需要连接到数据管道。Rasa Pro 直接支持以下数据湖：
    - [PostgreSQL](https://aws.amazon.com/rds/postgresql/)（推荐。所有 PostgreSQL >= 11.0 均受支持）
    - [Amazon Redshift](https://aws.amazon.com/redshift/)

    几乎任何其他数据湖都可以配置为与你的 PostgreSQL 部署同步。你可以在[连接数据仓库步骤](#connect-a-data-warehouse)中找到有关如何将你的 PostgreSQL 部署连接到 [BigQuery](#bigquery) 或 [Snowflake](#snowflake) 的其他说明。

    我们建议对数据湖进行托管部署，以最大限度地减少维护工作。

### 连接对话机器人 {#connect-an-assistant}

要将对话机器人连接到 Rasa Pro 服务，你需要将对话机器人连接到事件代理。对话机器人会将所有事件流式传输到事件代理，然后由 Rasa Pro 服务使用。

对话机器人的配置是[安装和配置](../../deploy/deploy-rasa-pro-services.md)的第一步。无需额外配置即可将对话机器人连接到分析管道。部署对话机器人后，分析管道将从对话机器人接收数据并将其保存到你的数据仓库中，该仓库将在下一步中进行配置。

### 连接数据仓库 {#connect-a-data-warehouse}

你可以选择配置分析管道，将转换后的对话数据传输到 AWS 中的两种不同类型的数据仓库：

- [PostgreSQL](#postgresql)
- [Redshift](#redshift)

此外，如果任何数据仓库与你的 PostgreSQL 部署同步配置，则均可支持，如 [Redshift 所建议](#streaming-from-postgresql-to-redshift)的那样。

我们提供了有关将你的 PostgreSQL 部署与以下设备同步的简要说明：

- [BigQuery](#bigquery)
- [Snowflake](#snowflake)

#### PostgreSQL {#postgresql}

你可以使用 Amazon Relational Database Service (RDS) 创建 PostgreSQL DB 实例，这是运行 PostgreSQL 数据库的环境。

首先，你必须按照[此处](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_SettingUp.html)列出的说明设置 Amazon RDS。接下来，创建 PostgreSQL DB 实例。你可以按照以下说明集之一进行操作：

- [创建 PostgreSQL DB 实例部分](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.PostgreSQL.html#CHAP_GettingStarted.Creating.PostgreSQL)中列出的 AWS Easy 创建说明。
- AWS [Standard 创建说明](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateDBInstance.html)。

要以[连接对话机器人部分](#connect-an-assistant)中指定的 DBAPI 格式构建 DB URL，你必须输入数据库凭证。在创建 PostgreSQL DB 实例的过程中选择数据库身份验证选项后，你必须获取数据库用户名和密码：

- 密码身份验证仅使用数据库凭证，在这种情况下，你必须输入主用户名的用户名，并生成主密码。
- [密码和 IAM DB 身份验证](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.html)使用 IAM 用户和角色进行数据库用户的身份验证。
- [密码和 Kerberos 身份验证](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/postgresql-kerberos.html)。

最后，在运行 Analytics 管道 Docker 容器时，将[环境变量 `RASA_ANALYTICS_DB_URL`](../../deploy/deploy-rasa-pro-services.md) 设置为 PostgreSQL Amazon RDS DB 实例 URL。

#### Redshift {#redshift}

如果你更愿意将 Amazon Redshift 设置为数据湖，你可以选择将 [PostgreSQL 步骤](#postgresql)中创建的 Amazon RDS DB 实例中的 PostgreSQL 源数据库的数据流式传输到 Redshift 目标，或[直接连接](#direct-connection)到 Amazon Redshift。

##### 从 PostgreSQL 流式传输到 Redshift {#streaming-from-postgresql-to-redshift}

如果你满足使用 Amazon Redshift 数据库作为目标的[先决条件](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Target.Redshift.html#CHAP_Target.Redshift.Prerequisites)，则需要执行两个步骤：

1. 按照以下[说明](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Source.PostgreSQL.html)为 AWS 数据库迁移服务 (DMS) 配置 PostgreSQL 源。
2. 按照[此处](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Target.Redshift.html)的说明为 AWS DMS 配置 Redshift 目标。

##### 直接连接 {#direct-connection}

你可以按照以下关于创建 [Amazon Redshift 集群](https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-clusters.html)的资源开始启用与 Redshift 的直接连接。在执行此操作之前，你应该考虑到将历史数据直接流式传输到 Redshift 可能比直接流式传输到或通过 PostgreSQL RDS 实例花费的时间更长。

你必须将 `RASA_ANALYTICS_DB_URL` 更新为 Redshift 集群 DB URL，该 URL 必须遵循以下格式：

```
redshift://<USER>:<PASSWORD>@<AWS URL>:5439/<DB NAME>
```

例如：

```
redshift://awsuser:4324312adfaGQ@analytics.cp1yucixmagz.us-east-1.redshift.amazonaws.com:5439/analytics
```

!!! warning "Redshift 写入性能"

    出于性能方面的考虑，我们强烈建议不要选择直接连接到 Redshift。Redshift 是一个非常适合分析的数据湖，但缺乏直接向其传输数据所需的写入性能。

#### BigQuery {#bigquery}

要将数据从 PostgreSQL 流式传输到 BigQuery，你可以使用 [Datastream for BigQuery](https://cloud.google.com/datastream/docs)。Datastream for BigQuery 支持多种 [PostgreSQL 部署类型](https://cloud.google.com/datastream/docs/configure-your-source-postgresql-database#overview)，包括 [CloudSQL](https://cloud.google.com/sql/docs/postgres)。

在开始之前，请务必检查 Datastream [先决条件](https://cloud.google.com/datastream/docs/before-you-begin)以及其他 Datastream 网络连接[要求](https://cloud.google.com/datastream/docs/quickstart-replication-to-bigquery#requirements)。

你可以密切关注此[快速入门指南](https://cloud.google.com/datastream/docs/quickstart-replication-to-bigquery)，了解如何使用 Datastream 将数据从 PostgreSQL CloudSQL 复制到 BigQuery。

或者，你可以深入了解以下 Datastream 设置指南：

- [配置源 PostgreSQL 数据库](https://cloud.google.com/datastream/docs/configure-your-source-postgresql-database)
- 可选：使用[客户管理的加密密钥](https://cloud.google.com/datastream/docs/use-cmek)
- [为 PostgreSQL 数据库创建连接配置文件](https://cloud.google.com/datastream/docs/create-connection-profiles#cp4postgresdb)
- 创建[流](https://cloud.google.com/datastream/docs/create-a-stream)

#### Snowflake {#snowflake}

你可以手动或通过自动化[合作伙伴解决方案](https://docs.snowflake.com/en/user-guide/ecosystem-etl.html)同步你的 PostgreSQL 部署。

手动同步的说明包括以下步骤：

- 使用 `COPY INTO` [命令](https://docs.snowflake.com/en/sql-reference/sql/copy-into-location.html)将数据从 PostgreSQL 提取到文件。你还应该在提取之前探索 Snowflake [数据加载最佳实践](https://docs.snowflake.com/en/user-guide/data-load-considerations.html)。
- 将提取的数据文件暂存到内部或外部位置，例如 AWS S3、Google Cloud Storage 或 Microsoft Azure。
- 使用 `COPY INTO` [命令](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table.html)将暂存的文件复制到 Snowflake 表。你可以决定使用[批量数据加载](https://docs.snowflake.com/en/user-guide/data-load-bulk.html)到 Snowflake 或使用 [Snowpipe](https://docs.snowflake.com/en/user-guide/data-load-snowpipe.html) 连续加载。或者，你也可以利用[此插件](https://cdap.atlassian.net/wiki/spaces/DOCS/pages/694157985/Cloud+Storage+to+Snowflake+Action)将数据加载到现有的 Snowflake 表中。

### 提取过去的对话（可选） {#ingest-past-conversations-optional}

当 Analytics 连接到你的 Kafka 实例时，它将使用 Kafka 主题上的所有先前事件并将其提取到数据库中。Kafka 对主题上的事件有一个[默认为 7 天的保留策略](https://docs.confluent.io/platform/current/installation/configuration/topic-configs.html#topicconfigs_retention.ms)。

如果你想处理比为 Rasa 主题配置的保留策略更早的对话中的事件，你可以手动提取过去对话中的事件。

手动提取过去对话中的数据需要连接到追踪器存储。追踪器存储包含过去的对话和与 Kafka 集群的连接。使用 `rasa export` 命令将存储在追踪器存储中的事件导出到 Kafka：

```shell
rasa export --endpoints endpoints.yml
```

配置导出以从生产追踪器存储中读取并作为事件代理写入 Kafka，例如：

```yaml title="endpoints.yml"
tracker_store:
    type: SQL
    dialect: "postgresql"
    url: "localhost"
    db: "tracker"
    username: postgres
    password: password

event_broker:
    type: kafka
    topic: rasa-events
    url: localhost:29092
    partition_by_sender: true
```

!!! note "注意"

    多次手动提取过去的事件将导致重复事件。Analytics 目前未实现重复数据删除。每个提取的事件都将存储在数据库中，即使它之前已处理过。

### 连接 BI 解决方案 {connect-a-bi-solution}

将 BI 平台连接到数据仓库的方法因平台而异。我们提供了 Metabase 和 Tableau 的示例说明，但你可以使用任何支持 AWS Redshift 或 PostgreSQL 的 BI 平台。

#### 示例：Metabase {#example-metabase}

Metabase 是一个免费的开源商业智能平台。它提供了一个简单的界面来查询和可视化数据。Metabase 可以连接到 PostgreSQL 或 Redshift 数据库。

- [将 Metabase 连接到 PostgreSQL](https://www.metabase.com/data_sources/postgresql)
- [将 Metabase 连接到 Redshift](https://www.metabase.com/data_sources/amazon-redshift)

#### 示例：Tableau {#example-tableau}

Tableau 是一个商业智能平台。它提供了一个灵活的界面来构建商业智能仪表板。Tableau 可以连接到 PostgreSQL 或 Redshift 数据库。

- [将 Tableau 连接到 PostgreSQL](https://help.tableau.com/current/pro/desktop/en-us/examples_postgresql.htm)
- [将 Tableau 连接到 Redshift](https://help.tableau.com/current/pro/desktop/en-us/examples_amazonredshift.htm)
