# 命令行界面

命令行界面（CLI）为你提供了易于记忆的常用任务命令。本页面描述了命令的行为以及需要传递的参数。

## Cheat Sheet {#cheat-sheet}

以下命令与使用 Rasa 构建的所有对话机器人相关。

| 命令                  | 效果                                                       |
| --------------------- | ---------------------------------------------------------- |
| `rasa init`           | 使用示例训练数据、动作和配置文件创建一个新项目。           |
| `rasa train`          | 根据你的 NLU 数据和故事训练一个模型，并保存至 `./models`。 |
| `rasa shell`          | 加载训练好的模型并让你可以通过命令行与对话机器人交谈。     |
| `rasa run`            | 使用训练好的模型启动服务。                                 |
| `rasa run actions`    | 使用 Rasa SDK 启动一个动作服务。                           |
| `rasa test e2e`       | 运行与充当验收测试的动作服务器完全集成的端到端测试。       |
| `rasa inspect`        | 开启 Rasa 检查。                                           |
| `rasa data split nlu` | 对 NLU 训练数据进行 80/20 切分。                           |
| `rasa data validate`  | 检查领域，NLU 和对话数据是否存在不一致。                   |
| `rasa export`         | 将对话从一个追踪器存储导出至一个事件代理。                 |
| `rasa marker upload`  | 将标记配置上传至分析数据流水线。                           |
| `rasa license`        | 显示许可证信息。                                           |
| `rasa -h`             | 显示所有可用的命令。                                       |

以下命令仅用于构建[基于 NLU 的对话机器人](calm.md#calm-compared-to-nlu-based-assistants)。

| 命令                      | 效果                                                         |
| ------------------------- | ------------------------------------------------------------ |
| `rasa interactive`        | 通过与你的对话机器人聊天来启动交互式学习会话以创建新的训练数据。 |
| `rasa visualize`          | 生成故事的可视化表示。                                       |
| `rasa test`               | 在任意以 `test_` 开头的文件上测试训练好的 Rasa 模型。        |
| `rasa data split stories` | 对故事数据进行 80/20 切分。                                  |
| `rasa data convert`       | 转换不同测试的训练数据。                                     |

以下命令仅用于 Rasa Studio。

| 命令                   | 效果                                        |
| ---------------------- | ------------------------------------------- |
| `rasa studio download` | 从 Rasa Studio 下载最新数据。               |
| `rasa studio train`    | 使用来自 Rasa Studio 的数据训练对话机器人。 |
| `rasa studio upload`   | 将你的意图和实体上传到 Rasa Studio。        |
| `rasa studio config`   | 使用  Studio 配置更新 `global.yml` 文件。   |
| `rasa studio login`    | 从 Rasa Studio 获取凭证。                   |

!!! note "注意"

    如果在 Windows 上遇到字符编码问题，例如：`UnicodeEncodeError: 'charmap' codec can't encode character ...` 或者终端无法正确显示彩色消息，请在要运行的命令前添加 `winpty`。例如，`winpty rasa init` 而不是 `rasa init`。

## 日志级别 {#log-level}

Rasa 可以生成不同级别的日志信息（例如：警告、信息、错误等）。使用 `--verbose`（与 `-v` 相同）或 `--debug`（与 `-vv` 相同）作为可选命令行参数来控制期望的日志级别。有关这些参数含义的更多解释，可以参见如下每个命令。

除了命令行参数外，还有若干环境变量可以用于更精细化地控制日志输出。通过这些环境变量，可以设置例如 Matplotlib、Pika 和 Kafka 外部库产生的日志。这些变量遵循 [Python 标准日志级别](https://docs.python.org/3/library/logging.html#logging-levels)。目前支持如下环境变量：

1. `LOG_LEVEL_LIBRARIES`：这是一个用于配置 Rasa 所用主要库日志级别的通用环境变量。包括 Tensorflow，`asyncio`，APScheduler，SocketIO，Matplotlib，RabbitMQ，Kafka。
2. `LOG_LEVEL_MATPLOTLIB`：这是一个专门用于配置 Matplotlib 日志级别的环境变量。
3. `LOG_LEVEL_RABBITMQ`：这是一个专门用于配置 AMQP 库日志级别的环境变量，目前可以设置 `aio_pika` 和 `aiormq` 的日志级别。
4. `LOG_LEVEL_KAFKA`：这是一个专门用于配置 kafka 日志级别的环境变量。
5. `LOG_LEVEL_PRESIDIO`：这是一个专门用于配置 Presidio 日志级别的环境变量，目前可以设置 `presidio_analyzer` 和 `presidio_anonymizer` 的日志级别。
6. `LOG_LEVEL_FAKER`：这是一个专门用于配置 Faker 日志级别的环境变量。

通用配置（`LOG_LEVEL_LIBRARIES`）的优先级相比于专用配置（`LOG_LEVEL_MATPLOTLIB`，`LOG_LEVEL_RABBITMQ` 等）更低。命令行参数设置了最低级别的日志。这意味着可以与这些变量一同使用，例如：

```shell
LOG_LEVEL_LIBRARIES=ERROR LOG_LEVEL_MATPLOTLIB=WARNING LOG_LEVEL_KAFKA=DEBUG rasa shell --debug
```

上述命令会产生如下效果：

- 默认处理比 `DEBUG` 级别更高的消息（由于 `--debug`）
- 对于 Matplotlib 库，处理比 `WARNING` 级别更高的消息
- 对于 kafka，处理比 `DEBUG` 级别更高的消息
- 对于其他未配置的库，处理比 `ERROR` 级别更高的消息

注意，命令行设置需要处理的最低级别日志消息，如下命令会将日志级别设置为 `INFO`（由于 `--verbose`），因此不会有任何调试信息展示（对库级别配置不会产生任何影响）：

```shell
LOG_LEVEL_LIBRARIES=DEBUG LOG_LEVEL_MATPLOTLIB=DEBUG rasa shell --verbose
```

命令行日志级别设置了根日志的级别（它包含了重要的 `coloredlogs` 处理器）。这意味着即使环境变量设置一个库日志较低的日志级别，根日志也会拒绝来自该库的消息。如果没有指定，命令行日志级别将被设置为 `INFO`。

## LLM 组件日志级别 {#log-level-llm-components}

Rasa 通过环境变量指定的细粒度、可自定义的日志记录增强了对 LLM 组件调试过程的控制。

例如，设置 `LOG_LEVEL_LLM` 环境变量可以在所需级别为所有 LLM 组件启用详细日志记录，或者通过设置 `LOG_LEVEL_LLM_ENTERPRISE_SEARCH` 环境变量来指定正在调试的组件：

```bash
export LOG_LEVEL_LLM=INFO
export LOG_LEVEL_LLM_COMMAND_GENERATOR=INFO
export LOG_LEVEL_LLM_ENTERPRISE_SEARCH=DEBUG
export LOG_LEVEL_LLM_INTENTLESS_POLICY=INFO
export LOG_LEVEL_LLM_REPHRASER=INFO
```

这些设置会覆盖指定组件的日志记录级别。

`LOG_LEVEL_LLM_COMMAND_GENERATOR` 变量适用于所有类型的[基于 LLM 的命令生成器](concepts/components/llm-command-generators.md)。

## 自定义日志配置

!!! info "3.4 版本新特性"

    Rasa 命令行现在包含一个新参数 `--logging-config-file`，其接受一个 YAML 文件。

你可以在单独的 YAML 文件中配置任意日志格式化器和处理器。日志配置 YAML 文件必须遵循 [Python 内置字典结构](https://docs.python.org/3/library/logging.config.html#dictionary-schema-details)，否则将会验证失败。你可以将此文件作为参数传递给命令行的 `--logging-config-file` 参数，将其同任意 Rasa 命令一起使用。

### 自定义日志配置示例 {#custom-logging-configuration-example}

以下示例说明如何使用 YAML 文件自定义日志配置。这里我们定义了一个自定义格式化器、一个用于根日志的流处理器和一个用于 rasa 日志的文件处理器。

```yaml
version: 1
disable_existing_loggers: false
formatters:
    customFormatter:
        format: "{\"time\": \"%(asctime)s\", \"name\": \"[%(name)s]\", \"levelname\": \"%(levelname)s\", \"message\": \"%(message)s\"}"

handlers:
  console:
    class: logging.StreamHandler
    level: INFO
    formatter: customFormatter
    stream: ext://sys.stdout
  file:
    class: logging.FileHandler
    filename: "rasa_debug.log"
    level: DEBUG
    formatter: customFormatter

loggers:
  root:
    handlers: [console]
  rasa:
    handlers: [file]
    propagate: 0
```

!!! info "信息"

    在 Rasa Pro 3.9 中，使用默认日志配置时，在调试模式下运行 `rasa shell` 或 `rasa interactive` 可能会导致 `BlockingIOError`。此问题可以通过使用自定义日志配置文件来解决。如果遇到此问题，你可以使用上述示例创建自定义日志配置文件并将其传递给 `--logging-config-file` 参数。

## rasa init

该命令将会利用一些示例训练数据来配置一个完整的对话机器人：

```shell
rasa init
```

这将创建如下文件：

```
.
├── actions
│   ├── __init__.py
│   └── actions.py
├── config.yml
├── credentials.yml
├── data
│   ├── nlu.yml
│   └── stories.yml
├── domain.yml
├── endpoints.yml
├── models
│   └── <timestamp>.tar.gz
└── tests
   └── test_stories.yml
```

会询问是否使用此数据训练初始模型，如果回答为否，则 `models` 目录将为空。

任何模型的命令行参数均需要这个项目设置，因此这是开始的最佳方式。无需任何额外的配置即可运行 `rasa train`，`rasa shell` 和 `rasa test` 命令。

除了上述默认 NLU 模板外，Rasa 还提供了另外两个模板。这两个模板都是开始构建你自己的 CALM 对话机器人的好方法：

- `rasa init --template calm`：生成一个带有[流](concepts/flows.md)和[自定义动作](concepts/custom-actions.md)的 CALM 对话机器人来管理一个简单的联系人列表。
- `rasa init --template tutorial`：生成 [CALM 教程](tutorial.md)中使用的代码库。

## rasa train

如下命令将会训练一个 Rasa 模型：

```shell
rasa train
```

如果在目录中已经存在模型（默认在 `models/` 下），只有修改过的模型才会被重新训练。例如，如果你只修改了 NLU 训练数据，则只有 NLU 部分会被训练。

如果希望单独训练 NLU 或对话模型，需要运行 `rasa train nlu` 或 `rasa train core`。如果只为其中之一提供了训练数据，则 `rasa train` 将会默认回退到这些命令之一。

`rasa train` 会将模型存储在 `--out` 定义的目录中，默认为 `models/`。模型的默认名称为 `<timestamp>.tar.gz`。如果想以不同的方式命名模型，可以通过 `--fixed-model-name` 参数来指定名称。

默认情况下，在训练模型之前需要运行验证。如果想跳过验证，可以使用 `--skip-validation`。如果想在收到验证警告时失败，可以使用 `--fail-on-validation-warnings`。`--validation-max-history` 类似于 `rasa data validate` 中的 `--max-history` 参数。

运行 `rasa train --help` 以查看完整的参数列表。

有关数据增强如何工作以及如果为参数设置值可以参阅[数据增强](nlu-based-assistants/policies.md#data-augmentation)部分。注意 `TEDPilicy` 是唯一受到数据增强影响的策略。

有关 `--epoch-fraction` 参数的更多信息，请参见如下有关[增量训练](#incremental-training)的部分。

### 增量训练 {#incremental-training}

!!! info "2.2 版本新特性"

    此功能是实验性的。我们通过社区反馈引入了这一实验性功能，因此鼓励大家进行尝试。但是，相关功在未来可能会发生变更或删除。如果有任何正面或负面反馈，可以在[Rasa 论坛](https://forum.rasa.com/)上进行分享。

为了提高对话机器人的性能，践行 CDD 和根据用户与机器人交谈的方式增加新的训练样本会很有帮助。通过 `rasa train --finetune` 可以使用已训练的模型来初始化整个流，之后通过包含额外训练样本的新训练数据进行微调。这有助于减少训练新模型的时间。

默认情况下，命令会选择 `models/` 目录中的最新模型。如果你希望改进特定的模型，则需要通过执行 `rasa train --finetune <path to model to finetune>` 来指定路径。微调模型通常在训练机器学习组件例如：`DIETClassifier`，`ResponseSelector` 和 `TEDPolicy` 时相比从头开始需要更少的时间。要么使用定义更少批次的模型微调配置，要么使用 `--epoch-fraction` 参数。在模型配置文件中，`--epoch-fraction` 会为每个机器学习组件指定部分批次。例如，训练 `DIETClassifier` 配置使用 100 个批次，指定 `--epoch-fraction 0.5` 将仅使用 50 个批次进行微调。

通过 `rasa train nlu --finetune` 和 `rasa train core --finetune` 可以分别仅对 NLU 或对话管理模型进行微调。

为了能够对一个模型进行微调，必须满足如下条件：

1. 用于训练模型的配置应该与用于微调模型的配置完全相同。唯一可以修改的参数就是用于各个机器学习组件和策略的 `epochs`。
2. 用于基础模型训练的标签集（意图、动作、实体和槽）应该与用于模型微调训练数据中的标签集完全相同。这意味着在进行增量训练的过程中不能添加新的意图、动作、实体或槽标签。可以为每个现有标签添加新的训练样本。如果在训练数据中添加或删除标签，则需要从头开始训练。
3. 要微调的模型会使用当前安装版本 Rasa 的 `MINIMUM_COMPATIBLE_VERSION` 版本进行训练。

## rasa interactive

通过如下命令可以开启一个交互式学习会话：

```shell
rasa interactive
```

这将首先训练一个模型，然后启动一个交互式 Shell 会话。之后随着同对话机器人的交谈可以对其预测进行不断修正。如果 [UnexpecTEDIntentPolicy](nlu-based-assistants/policies.md#unexpected-intent-policy) 包含在流中，则可以在任意对话抡次中触发 [`action_unlikely_intent`](nlu-based-assistants/default-actions.md#action_unlikely_intent)，之后会显示：

```
The bot wants to run 'action_unlikely_intent' to indicate that the last user message was unexpected
at this point in the conversation. Check out UnexpecTEDIntentPolicy docs to learn more.
```

正如消息所述，这表示你尝试了一个不在当前的训练故事集预期的一个对话路径，因此建议将该路径添加到训练故事中。同其他对话机器人响应一样，你可以选择接受或拒绝此动作。

如果使用 `--model` 参数提供一个训练好的模型，则会跳过训练过程直接加载这个模型。

在交互式学习过程中，Rasa 将会可视化当前对话和训练数据中的一些相似对话，来帮助你追踪当前的进度。会话开始后，可以在 <http://localhost:5005/visualization.html> 中查看相关可视化。图表生成需要一些时间。运行 `rasa interactive --skip-visualization` 可以跳过可视化。

!!! info "添加 3.5 版本中引入的 `ASSISTANT_ID` 健"

    在利用元数据不包含 `assistant_id` 的预训练模型运行交互式学习会出现错误并退出。如果发生这种情况，请在 `config.yml` 中添加具有唯一标示值的 `assistant_id` 并重新训练。

运行 `rasa interactive --help` 查看完整的参数列表。

## rasa shell

运行如下命令可以启动一个聊天会话：

```shell
rasa shell
```

默认情况下会加载最新的训练模型。你也可以通过 `--model` 参数指定要加载的其他模型。

如果仅使用 NLU 模型启动 shell，`rasa shell` 将输出输入消息的预测意图和实体。

如果已经训练了一个组合 Rasa 模型，但你只想看模型从文本中提取的意图和实体，可以使用 `rasa shell nlu`。

要提高调试的日志记录级别，可以运行：

```shell
rasa shell --debug
```

!!! info "注意"

    为了在外部频道中查看问候语和会话起始行为，你可以通过显式发送 `/session_start` 作为第一条消息。否则，会话起始行为将按照[会话配置](concepts/domain.md#session-configuration)中的描述开始。

如下参数可以用于配置此命令。大部分参数同 `rasa run` 相同。有关这些参数的更多信息，请参见[如下部分](#rasa-run)。

注意，在运行 `rasa shell` 时，`--connector` 参数将会始终被设置为 `cmdline`。这意味着 credentials 文件中的所有 credentials 都会被忽略，如果你为 `--connector` 参数指定了值，也将被忽略。

运行 `rasa shell --help` 查看完整的参数列表。

## rasa run

运行如下命令可以为训练好的模型启动一个服务：

```shell
rasa run
```

默认情况下，Rasa 服务使用 HTTP 进行通信。要使用 SSL 加密通讯并在 HTTPS 上运行服务，你需要提供一个有效的证书和对应的私钥文件。可以在 `rasa run` 命令中指定这些文件。如果在创建密钥文件过程中使用密码进行了加密，则还需要添加 `--ssl-passowrd` 参数。

```shell
rasa run --ssl-certificate myssl.crt --ssl-keyfile myssl.key --ssl-password mypassword
```

Rasa 模型监听每个可用的网络接口。通过 `-i` 命令行参数可以将其限制为特定的网络接口。

```shell
rasa run -i 192.168.69.150
```

Rasa 默认会连接到 credentials 文件中指定的所有频道。在 `--connector` 参数中指定频道的名称，可以仅连接一个频道并忽略 credentials 文件中的所有频道。

```shell
rasa run --connector rest
```

频道的名称需要同 credentials 文件中指定的名称相匹配。关于支持的频道请参见[消息和语音频道](connectors/messaging-and-voice-channels.md)。

运行 `rasa run --help` 查看完整的参数列表。

有关重要附加参数的更多信息，请参阅[模型存储](production/model-storage.md)。

请参阅 Rasa [REST API](production/rest-api.md) 页面以获取所有 API 的详细文档。

## rasa run actions

运行如下命令可以使用 Rasa SDK 启动动作服务：

```shell
rasa run actions
```

运行 `rasa run actions --help` 查看完整的参数列表。

## rasa visualize

运行如下命令可以在浏览器中可视化故事：

```shell
rasa visualize
```

如果故事存储于默认位置 `data/` 以外的位置，可以通过 `--stories` 指定它们的位置。

运行 `rasa visualize --help` 查看完整的参数列表。

## rasa test

运行如下命令可以利用测试数据评估模型：

```shell
rasa test
```

这会在以 `text_` 开头的文件中定义的故事上端到端的测试最新训练的模型。通过指定 `--model` 参数可以选择一个不同的模型。

如果想要分开评估对话和 NLU 模型，可以运行如下命令：

```shell
rasa test core
```

和

```shell
rasa rest nlu
```

你可以在[评估 NLU 模型](nlu-based-assistants/testing-your-assistant.md#evaluating-an-nlu-model)和[评估对话管理模型](nlu-based-assistants/testing-your-assistant.md#evaluating-a-dialogue-model)中找到有关每种测试类型参数的更多信息。

运行 `rasa test --help` 查看完整的参数列表。

## rasa test e2e

!!! info "3.5 版本新特性"

    你现在可以使用端到端测试来整体测试对话机器人，包括对话管理和自定义动作。

要对训练好的模型运行端到端测试，请运行：

```shell
rasa test e2e
```

这将在任何端到端测试用例上测试最新训练的模型。如果想使用其他模型，可以使用 `--model` 参数指定它。

!!! info "3.10 版本新特性"

    通过添加 `--coverage-report` 标志，你可以获得一份报告，该报告描述了端到端测试如何很好地覆盖对话机器人的流（就每个流测试的步骤共享而言）。该报告包含测试命令的直方图，并允许你使用 `--coverage-output-path` 标志指定输出路径。

    此功能目前以测试版发布。该功能将来可能会发生变化。如果你想启用此测试版功能，请设置环境变量 `RASA_PRO_BETA_FINE_TUNING_RECIPE=true`。

运行 `rasa test e2e --help` 查看完整的参数列表。

## rasa llm finetune prepare-data

!!! info "3.10 版本新特性"

    此命令是从版本 `3.10.0` 开始提供的[微调](operating/fine-tuning-recipe.md)的一部分。由于此功能是测试版功能，请将环境变量 `RASA_PRO_BETA_FINETUNING_RECIPE` 设置为 `true` 以启用它。

此命令创建来自 E2E 测试的提示到命令对的数据集，可用于微调命令生成任务的基础模型。要执行命令，请运行：

```shell
rasa llm finetune prepare-data <path-to-e2e-test-cases>
```

以下是一些可用参数：

```txt
positional arguments:
  path-to-e2e-test-cases
                        Input file or folder containing end-to-end test cases. (default: e2e_tests)

options:
  -o OUT, --out OUT     The output folder to store the data to. (default: output)
  -m MODEL, --model MODEL
                        Path to a trained Rasa model. If a directory is specified, it will use the latest model in this directory. (default: models)

Rephrasing Module:

  --num-rephrases  {0, ..., 49}
                        Number of rephrases to be generated per user utterance. (default: 10)
  --rephrase-config REPHRASE_CONFIG
                        Path to config file that contains the configuration of the rephrasing module. (default: None)

Train/Test Split Module:
  --train-frac TRAIN_FRAC
                        The amount of data that should go into the training dataset. The value should be >0.0 and <=1.0. (default: 0.8)
  --output-format [{instruction,conversational}]
                        Format of the output file. (default: instruction)
```

运行 `rasa finetune prepare-data --help` 查看完整的参数列表。

## rasa inspect

!!! info "3.7 版本新特性"

    此命令是 Rasa 的新语言模型（CALM）对话式 AI 的一部分，从 `3.7.0` 版本开始可用。

打开 [Rasa Inspector](production/inspect-assistant.md)，这是一个调试工具，可让开发人员深入了解 Rasa 对话机器人的对话机制。

运行 `rasa inspect --help` 查看完整的参数列表。

## rasa data split

运行如下命令可以对 NLU 训练数据进行拆分：

```shell
rasa data split nlu
```

默认情况下，这将对数据进行 80/20 拆分为训练集和测试集。运行 `rasa data split nlu --help` 查看完整的参数列表。

如果有用于检索动作的 NLG 数据，则会将其保存到单独的文件中：

```shell
ls train_test_split

      nlg_test_data.yml     test_data.yml
      nlg_training_data.yml training_data.yml
```

运行如下命令可以对故事数据进行拆分：

```shell
rasa data split stories
```

其与 `split nlu` 命令具有相同的参数，但会加载故事的 yaml 文件并进行随机拆分。目录 `train_test_split` 将处理后的前缀位 `train_` 的训练部分或前缀为 `test_` 的测试部分的所有 yaml 文件。

## rasa data convert nlu

你可以将 NLU 数据从：

- LUIS 数据格式
- WIT 数据格式
- Dialogflow 数据格式
- JSON

转换为：

- YAML
- JSON

通过如下命令可以执行转换：

```shell
rasa data convert nlu
```

如下参数可以指定输入文件或目录、输出文件或目录和输出格式。运行 `rasa data convert nlu --help` 查看完整的参数列表。

## rasa data migrate

领域是唯一在 2.0 和 3.0 版本之间发生格式改变的数据。你可以自动将 2.0 版本的领域迁移到 3.0 版本格式。

运行如下命令可以执行迁移：

```shell
rasa data migrate
```

如下参数可以指定输入文件或目录、输出文件或目录：

```
rasa data migrate -d DOMAIN --out OUT_PATH
```

如果没有指定参数，则默认领域路径（`domain.yml`）将用于输入和输出文件。

此命令还会将 2.0 版本的领域文件备份到不同的 `original_domain.yml` 文件或标有 `original_domain` 的目录中。

请注意，如果这些槽是表单 `required_slots` 的一部分，则迁移领域中的槽将包含[映射条件](concepts/domain.md#mapping-conditions)。

!!! warning "警告"

    当领域文件无效或已经为 3.0 版本格式时，当槽或表单分布在多个领域文件中且原始文件中缺少槽或表单时，迁移过程会终止并触发异常。这样做是为了避免在领域文件中出现重复的迁移部分。请确保所有槽或表单的定义都被整合到一个文件中。

运行如下命令可以了解该命令的更多信息：

```shell
rasa data migrate --help
```

## rasa data validate

你可以检查领域、NLU 和故事数据是否存在错误或不一致。运行如下命令可以验证数据：

```shell
rasa data validate
```

校验器会在数据中搜寻错误，例如：两个意图具有一些相同的训练样本。校验器也会检查是否存不同的对话响应来自相同的对话历史的故事。故事之间的冲突会阻碍模型学习正确的对话模式。要了解有关验证器对流执行的检查的更多信息，请继续阅读[下一部分](#validate-flows)。

!!! info "搜索 3.5 版本中引入的 `ASSISTANT_ID` 健"

    验证器会检查配置文件中是否存在 `assistant_id` 健，如果不存在或未修改默认值，则会发出警告。

如果在 `config.yml` 文件中对一个或多个策略提供了 `max_history` 的值，可以使用 `--max-history <max_history>` 参数为验证命令的提供对应的最小值。

### 验证流 {#validate-flows}

验证器将对流执行以下检查：

- 确定流名称或描述在删除标点符号后是否唯一。
- 验证[条件](concepts/conditions.md)或 `collect` 步骤 [`rejections`](concepts/flows.md#slot-validation) 中的逻辑表达式是否为有效的 `pypred` 表达式
- 确定流中使用的槽是否在领域中定义。
- 禁止在流 `collect` 步骤中使用 `list` 槽：CALM 仅支持在流中使用 `int`、`string` 或 `bool` 类型的值填充槽。
- 禁止在流中使用 `dialogue_stack` 内部槽。

对于每次失败，验证器都会记录错误并以退出代码 1 退出命令。

你只能通过运行以下命令来验证流：

```shell
rasa data validate flows
```

### 验证故事结构 {#validate-story-structure}

你还可以通过运行以下命令仅验证故事结构：

```shell
rasa data validate stories
```

!!! info "注意"

    运行 `asa data validate` 不会测试[规则](nlu-based-assistants/rules.md)是否和故事一致。但是在训练期间，`RulePolicy` 会检查规则和故事之间的冲突。任何此类冲突都会终止训练。

    此外，如果你使用端到端的故事，那么可能无法捕获所有冲突。例如，如果两个用户的输入有不同的分词但有相同特征，那么这些输入之后可能存在冲突动作且不会被工具报告。

使用 `--fail-on-warnings` 参数可以在诸如未使用的意图或动作之类的小问题时也中断验证。

!!! warning "检查故事名称"

    `rasa data validate stories` 命令假定所有故事的名称都是唯一的。

你可以使用如下附加参数运行 `rasa data validate`，例如：指定数据和领域文件的位置。运行 `rasa data validate --help` 查看完整的参数列表。

## rasa export

运行如下命令可以使用事件代理从追踪存储中导出事件：

```shell
rasa export
```

你可以指定环境文件的位置、发布时间时间戳的最小值和最大值、以及发布的对话 ID。运行 `rasa export  --help` 查看完整的参数列表。

!!! tip "将对话导入企业版 Rasa"

    此命令常用于将旧的对话导入企业版 Rasa 中并对其进行标注。可以在[导入对话至企业版 Rasa](https://rasa.com/docs/rasa-enterprise/installation-and-setup/deploy#1-import-existing-conversations-from-rasa-open-source){:target="_blank"}中获取更多信息。

## rasa markers upload

!!! info "3.6 版本新特性"

    此命令从 Rasa Pro `3.6.0` 开始可用，并且需要 [Rasa Analytics Data Pipeline](operating/analytics/getting-started-with-analytics.md)。

此命令适用于[标记](production/markers.md)及其[实时处理](production/markers.md)。运行此命令将根据域文件验证标记配置文件，并将配置上传到 Analytics Data Pipeline。

运行 `rasa markers upload --help` 查看完整的参数列表。

## rasa license

!!! info "3.3 版本新特性"

    引入此命令。

使用 `rasa license` 显示有关 Rasa Pro 中的许可信息，尤其是有关第三方依赖许可的信息。

运行 `rasa license --help` 查看完整的参数列表。

## rasa studio download

!!! info "3.7 版本新特性"

    此命令从 Rasa Pro `3.7.0` 开始可用，并且需要 Rasa Studio。

此命令从 Rasa Studio 下载数据并将其保存到 `data` 文件夹内的文件中。如果本地文件使用单个领域文件，则会相应地更新它。如果有领域文件夹，则领域更改将写入 `<domain_folder>/studio_domain.yml`。

该命令下载 Studio 中可用但在本地文件中不可用的 Studio 数据。支持以下数据：

- 配置
- 端点
- 流（用于 CALM 对话机器人）
- 响应
- 槽
- 自定义动作声明
- 意图
- 实体

当原语具有与从 Rasa Studio 下载的原语相同的 ID 时，可以使用 `--overwrite` 参数覆盖现有文件中的现有数据。意图会得到特殊处理。特殊情况：

- 如果本地文件中存在意图，但 Studio 本地缺少示例，则会下载这些示例。
- 如果在下载 CALM 对话机器人期间存在本地 `config` 和 `endpoints` 文件，则用户需要确认其覆盖意图，即使提供了 `--overwrite` 标志。

示例：

```shell
rasa studio download my_awesome_assistant -d my_domain_folder
```

运行 `rasa studio download --help` 查看完整的参数列表。

## rasa studio train

!!! info "3.7 版本新特性"

    此命令从 Rasa Pro `3.7.0` 开始可用，并且需要 Rasa Studio。

此命令类似于 `rasa train`。此命令结合本地文件和 Rasa Studio 的数据来训练模型。如果 Studio 和本地文件都具有相同 ID 的[原语](nlu-based-assistants/glossary.md#rasa-primitive)，则使用本地数据进行训练。

运行 `rasa studio train --help` 查看完整的参数列表。

## rasa studio upload

!!! info "3.7 版本新特性"

    此命令从 Rasa Pro `3.7.0` 开始可用，并且需要 Rasa Studio。

将对话机器人从本地文件上传到 Rasa Studio。

### 导入基于 NLU 的对话机器人 {#import-of-nlu-based-assistants}

对于基于 NLU 的对话机器人，它将把意图和实体定义上传到 Rasa Studio 中 Rasa Studio 中的现有对话机器人。当未提供指定要上传哪些意图或实体的参数时，将上传所有意图和实体。上传意图时，也会上传该意图对话示例注释中使用的所有实体。

!!! tip "提示"

    目前，只有部分意图和实体可以上传到 Studio。以下实体无法上传：

    - 检索意图
    - 具有 `entity_group` 的实体
    - 具有 `use_entities` 和 `ignore_entities` 的意图
    - 具有 `influence_conversation` 的实体

示例：

```shell
rasa studio upload my_awesome_assistant
```

运行 `rasa studio upload --help` 查看完整的参数列表。

### 导入 CALM 对话机器人 {#import-of-nlu-based-assistants}

要将 CALM 对话机器人上传到 Rasa Studio，请使用 `--calm` 参数运行此命令。

!!! info "注意"

    上传 CALM 对话机器人时，将创建一个具有指定名称的新 Rasa Studio 对话机器人。这与基于 NLU 的对话机器人上传不同，后者将重用现有的 Rasa Studio 对话机器人。

示例：

```shell
rasa studio upload my_awesome_assistant --calm
```

可能的错误

**对话机器人名称错误**

包括以下内容：

- `Assistant name cannot be empty`
- `Assistant with name <assistant_name> already exists`
- `<assistant_name> is not a valid name`

有效对话机器人名称的长度不得超过 128 个字符，并且不包含空格。

**无效 YAML 错误**

如果 YAML 文件结构出现问题，则会记录特定错误。例如，当动作、槽、响应、配置或流缺少必填字段时，你将看到这些错误。

示例：

```txt
Invalid domain: responses.utter_greeting.0.text: Required
Invalid flows: flows.transfer_money.description: Required
```

**引用错误**

如果流引用响应、槽、动作或其他流（带有链接步骤），则会记录以下错误：

```txt
Can't find <response/slot/action> utter_ask_add_contact_handle in domain
Can't find flow <flow_name> in flows
```

**不支持的功能错误**

Rasa Pro 中提供的功能并非全部都受 Rasa Studio 支持。尝试导入具有不支持功能的对话机器人将导致错误。要了解哪些版本的 Studio 支持你正在使用的 Rasa Pro 版本，请查看 [Studio 兼容性矩阵](https://rasa.com/docs/studio/change-log/compatibility-matrix)。

示例：

```txt
Flows with cycles are not supported, flow: <flow_name>
Comparing two slots is not supported. Condition: slots.recurrent_payment_end_date < slots.recurrent_payment_start_date
Having multiple rejections on one slot is not supported, collect: <slot_name>
```

**身份验证错误**

用户需要在上传前登录 Rasa Studio。使用 [`rasa studio login`](#rasa-studio-login) 命令。

## rasa studio config

!!! info "3.7 版本新特性"

    此命令从 Rasa Pro `3.7.0` 开始可用，并且需要 Rasa Studio。

此命令提示输入 Rasa Studio 安装的参数，并在执行 `rasa studio` 命令时针对该 Rasa Studio 实例配置 rasa。配置保存到：`$HOME/.config/rasa/global.yml`。

该命令将使用默认参数来配置身份验证服务器（领域名称、客户端 ID 和身份验证 URL）。如果要使用不同的配置，可以通过运行 `rasa studio config --advanced` 来指定参数。

该命令将使用新配置覆盖现有配置文件。

示例：

```shell
rasa studio config
```

运行 `rasa studio config --help` 查看完整的参数列表。

## rasa studio login

!!! info "3.7 版本新特性"

    此命令从 Rasa Pro `3.7.0` 开始可用，并且需要 Rasa Studio。

此命令用于从 Rasa Studio 检索访问令牌。所有其他 Studio 命令都使用此令牌与 Rasa Studio 进行身份验证。令牌保存到：`$HOME/.config/rasa/studio_token.yaml`。

示例：

```shell
rasa studio login --username my_user_name --password my_password
```

运行 `rasa studio login --help` 查看完整的参数列表。

## 已弃用的命令

以下命令已弃用并将在未来版本中删除。

### rasa evaluate markers

!!! warning "警告"

    此功能目前是实验性的，未来可能发生更改或删除。你可以在论坛中进行反馈，来帮助其变为生产可用。

如下命令将你在标记配置文件中定义的[标记](production/markers.md)应用于存储在[追踪存储](production/tracker-stores.md)中预先存在的对话，同时生成包含提取的标记和统计概要信息的 `.csv` 文件：

```shell
rasa evaluate markers all extracted_markers.csv
```

运行 `rasa evaluate markers --help` 查看完整的参数列表。
