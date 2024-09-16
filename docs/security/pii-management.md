# 个人身份信息管理

!!! info "3.6 版本新特性"

    现在，你可以仅通过 Kafka 事件代理流式传输日志和事件中的个人身份信息 (PII) 数据。继续阅读以了解如何启用 PII 数据的匿名化。

!!! warning "功能尚未兼容 CALM"

    此功能目前与 [CALM](../calm.md) 不兼容。我们正在努力确保未来的集成和兼容性。

管理对话机器人收集的敏感客户数据是遵守法规、安全管理数据并使其可供分析的关键要求。Rasa Pro 3.6 引入了匿名化日志和通过 Kafka 事件代理传输的事件中的 PII 数据的功能。

请注意，追踪器存储会继续存储未匿名的对话数据。这是必需的，以便我们保留原始数据的真实来源，以实现不可否认的目的。了解如何使用 [Vault 密钥管理器](../production/secrets-managers.md)保护你的追踪器存储，以使用轮换凭据保护真实来源。

此外，由于追踪器存储数据在推理时对对话机器人的对话管理起着至关重要的作用，因此在传输过程中匿名化数据可能会对对话机器人的对话管理行动预测产生不良后果。

## 架构概述 {#architecture-overview}

Rasa 事件的匿名化在对话管理动作预测和执行结束时通过匿名化管道运行。请注意，匿名化管道完成的处理被安排为后台任务，不会影响对话机器人的响应时间。

匿名化步骤如下：

1. Rasa 代理在处理每个用户消息时调用匿名化管道。
2. 匿名化管道运行一系列应用于新事件的匿名化规则。
3. 管道将匿名化事件发布到映射到 `endpoints.yml` 中的匿名化规则列表的 Kafka 主题。
4. 追踪器存储保存未匿名化的原始事件。

### 支持的 Rasa 事件 {#supported-rasa-events}

匿名化的 Rasa 事件包括以下内容：

- `user`
- `bot`
- `slot`
- `entities`

### 支持的 PII 实体类型 {#supported-pii-entity-types}

匿名化管道使用 [Microsoft Presidio](https://microsoft.github.io/presidio/text_anonymization/) 作为实体识别器和匿名器。Presidio 是一个开源库，支持各种实体类型和匿名化方法。

你可以在匿名化规则中指定任何现成的 [Presidio 受支持的实体类型](https://microsoft.github.io/presidio/supported_entities/)。请注意，目前无法为 Rasa Pro 匿名化管道添加自定义实体类型。

## 如何编写匿名化规则 {#how-to-write-anonymization-rules}

你现在可以在 `endpoints.yml` 中编写匿名化规则，以明确声明哪些 Presidio 实体应该匿名化。匿名化管道可通过 `endpoints.yml` 文件中的 `anonymization` 部分进行配置。此部分必须具有以下示例中给出的结构：

```yaml
anonymization:
  metadata:
    language: en
    model_name: en_core_web_lg
    model_provider: spacy
  rule_lists:
    - id: rules_1
      rules:
        - entity: PERSON
          substitution: text
          value: John Doe
        - entity: LOCATION
          substitution: faker
        - entity: CREDIT_CARD
          substitution: faker
        - entity: IBAN_CODE
          substitution: faker
    - id: rules_2
      rules:
        - entity: CREDIT_CARD
          substitution: mask
        - entity: IBAN_CODE
          substitution: mask
```

### 如何填充元数据部分 {#how-to-populate-the-metadata-section}

元数据部分包含以下字段：`language`、`model_name` 和 `model_provider`。

`language` 字段指定要匿名化的文本的语言代码。请注意，每个匿名化管道只能指定一种语言，因此该功能目前无法处理最终用户的语言切换。

`model_name` 和 `model_provider` 字段指定用于匿名化的 Presidio 模型的名称和提供者。可用的模型提供者是 `spacy`、`stanza` 和 `transformers`。

如果你想使用 `spacy`，我们强烈建议使用可用的大型模型，例如 `en_core_web_lg` 或 `es_core_news_lg`。这是为了确保更准确的实体识别。

!!! warning "注意"

    当使用大型模型时，你必须确保 Rasa Pro 环境有足够的内存来加载模型。

如果你选择使用 `transformers` 模型提供程序，则必须在 `model_name` 字段中指定两个模型名称：HuggingFace 模型名称和 spaCy 模型名称，其中 spaCy 模型将包装 transformers NER 模型。例如：

```yaml
anonymization:
  metadata:
    language: en
    model_name:
      spacy: en_core_web_lg
      transformers: dslim/bert-base-NER
    model_provider: transformers
```

### 如何安装语言模型 {#how-to-install-the-language-mod}

你必须在 Rasa Pro 环境中安装你在 `model_name` 字段中声明的模型。例如，如果你声明 `model_name: en_core_web_lg`，则必须在 Rasa Pro 环境中安装 spaCy `en_core_web_lg` 模型。你可以按照 [Presidio 官方文档](https://microsoft.github.io/presidio/analyzer/nlp_engines/spacy_stanza/)中 spaCy 和 stanza 模型的模型安装说明进行操作。

对于 `transformers` 模型提供程序，你必须在 Rasa Pro 环境中安装你在 `model_name` 字段中声明的两个模型。例如，如果你在 `endpoints.yml` 中声明 `transformers: dslim/bert-base-NER`，则必须在 Rasa Pro 环境中安装 `dslim/bert-base-NER` 模型。你可以在 [Presidio 官方文档](https://microsoft.github.io/presidio/analyzer/nlp_engines/spacy_stanza/)中找到 `HuggingFace` 的模型下载说明。

!!! note "注意"

    并非所有语言都有预训练的语言模型。如果你想使用没有预训练语言模型的语言，你必须训练自己的 [spaCy](https://spacy.io/usage/training)、[stanza](https://stanfordnlp.github.io/stanza/training.html) 或 [huggingface](https://huggingface.co/docs/transformers/training) 模型并将其安装在你的 Rasa Pro 环境中。

### 如何填充 `rule_lists` 部分 {#how-to-populate-the-rule_lists-section}

`rule_lists` 部分包含匿名化规则列表列表。每个规则列表必须具有字符串类型的唯一 `id` 和 `rules` 列表。每个规则必须具有 `entity` 字段和 `substitution` 字段。`entity` 字段指定要匿名化的 Presidio 实体类型，并且必须大写。请注意，目前不支持使用正则表达式来识别实体。

`substitution` 字段指定要使用的匿名化方法。目前，支持以下匿名化方法：`text`、`mask` 和 `faker`：

- `text` 匿名化方法将原始实体值替换为 `value` 字段中指定的值。在以下示例中，`PERSON` 实体值将替换为 `John Doe`。

    ```yaml
    anonymization:
      metadata:
          language: en
          model_name: en_core_web_lg
          model_provider: spacy
      rule_lists:
          - id: rules_1
          rules:
              - entity: PERSON
              substitution: text
              value: John Doe
    ```

- `mask` 匿名化方法是将原始实体值替换为使用字符 `*` 的等长掩码，例如原始实体值为 `John Doe`，则匿名化后的值将为 `********`。

    ```yaml
    anonymization:
      metadata:
        language: en
        model_name: en_core_web_lg
        model_provider: spacy
      rule_lists:
        - id: rules_1
          rules:
            - entity: PERSON
              substitution: mask
    ```

- `faker` 匿名化方法将原始实体值替换为 [Faker](https://faker.readthedocs.io/en/stable/) 库生成的假值。例如，如果原始实体值为 `John Doe`，则匿名化后的值将被替换为 Faker 库生成的假名称。

    ```yaml
    anonymization:
      metadata:
        language: en
        model_name: en_core_web_lg
        model_provider: spacy
      rule_lists:
        - id: rules_1
          rules:
            - entity: PERSON
              substitution: faker
    ```

如果未指定替换方法，则默认替换方法为 `mask`。

`value` 字段仅对于文本匿名化方法才是必需的。它指定要用作匿名值的文本。如果未指定 `value` 字段，则要匿名化的原始实体值将替换为括号之间的实体类型名称。例如，如果未为 `PERSON` 实体类型指定 `value` 字段，则匿名值将为 `<PERSON>`。

`faker` 匿名化方法使用 [Faker](https://faker.readthedocs.io/en/stable/) 库来生成虚假数据。默认情况下，`faker` 匿名化方法将生成英文虚假数据，除非使用本地化的 Presidio 实体类型。例如，如果你对 `ES_NIF` 实体类型使用 `faker` 替换方法，则生成的虚假数据将与西班牙语 NIF 的格式匹配。

`faker` 替换方法不支持以下 Presidio 实体类型：

- `CRYPTO`、`NRP`、`MEDICAL_LICENSE`
- `US_BANK_NUMBER`、`US_DRIVER_LICENSE`
- `UK_NHS`
- `IT_FISCAL_CODE`、`IT_DRIVER_LICENSE`、`IT_PASSPORT`、`IT_IDENTITY_CARD`
- `SG_NRIC_FIN`
- `AU_ABN`、`AU_ACN`、`AU_TFN`、`AU_MEDICARE`

如果上述任何实体与 `faker` 替换方法一起使用，则匿名化管道将默认使用 `mask` 替换方法。

## 如何更新 Kafka 事件代理配置 {#how-to-update-the-kafka-event-broker-configuration}

匿名化管道使用 Kafka 将匿名化事件发布到映射到匿名化规则列表的 Kafka 主题。你可以在 `endpoints.yml` 文件中配置 Kafka 事件代理。Kafka 事件代理必须包含 `anonymization_topics` 部分，该部分必须具有以下结构：

```yaml
event_broker:
  type: kafka
  partition_by_sender: True
  security_protocol: PLAINTEXT
  url: localhost
  anonymization_topics:
    - name: topic_1
      anonymization_rules: rules_1
    - name: topic_2
      anonymization_rules: rules_2
  client_id: kafka-python-rasa
```

`anonymization_topics` 部分包含将发布匿名事件的 Kafka 主题列表。每个 Kafka 主题必须有一个 `name` 字段和一个 `anonymization_rules` 字段。`name` 字段指定 Kafka 主题的名称。`anonymization_rules` 字段指定要用于 Kafka 主题的匿名规则列表的 `id`。

### 使用 Kafka 将匿名事件流式传输到 Rasa X/Enterprise {#streaming-anonymized-events-to-rasa-xenterprise-with-kafka}

仅 Rasa X/Enterprise `1.3.0` 及以上版本支持将匿名事件流式传输到 Rasa X/Enterprise。此外，你必须使用 Kafka 事件代理，不支持其他事件代理类型。

你可以通过将 `rasa_x_consumer: true` 键值对添加到 `a`nonymization_topics` 部分，通过 Kafka 将匿名事件流式传输到 Rasa X/Enterprise：

```yaml
event_broker:
  type: kafka
  partition_by_sender: True
  url: localhost
  anonymization_topics:
    - name: topic_1
      anonymization_rules: rules_1
      rasa_x_consumer: true
    - name: topic_2
      anonymization_rules: rules_2
```

如果多个 Kafka 匿名化主题包含 `rasa_x_consume`r 键值对，则匿名化事件将流式传输到映射到 `anonymization_topics` 列表中第一个包含 `rasa_x_consumer` 键值对的主题的 Kafka 主题。

请注意，`rasa_x_consumer` 键值对是可选的。如果未指定，则匿名化事件将发布到 Kafka 主题，但不会流式传输到 Rasa X/Enterprise。

## 如何在日志中启用 PII 匿名化 {#how-to-enable-anonymization-of-pii-in-logs}

你可以通过填写 `endpoints.yml` 文件中的 `logger` 部分来启用日志中的 PII 匿名化。`logger` 部分必须具有以下结构：

```yaml
logger:
  formatter:
    anonymization_rules: rules_1
```

`anonymization_rules` 字段指定用于日志的匿名规则列表的 `id`。

!!! warning "注意"

    我们强烈建议在生产环境中以 INFO 日志级别运行。以 DEBUG 日志级别运行会因为处理延迟而增加对话机器人的响应延迟。

请注意，使用 Kafka 事件代理在调试模式下运行 `rasa shell` 可能会导致与事件发布相关的日志在机器人消息之后打印到控制台。此行为是预期的，因为事件匿名化和发布是作为后台任务异步完成的，因此它将在对话机器人已经预测并执行对话机器人响应后完成。
