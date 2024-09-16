# 端到端测试转换

Rasa Pro 提供了一种将示例对话转换为 E2E 测试用例的方法。

## 端到端测试转换 {#end-to-end-test-conversion}

!!! danger "3.10 版本测试新特性"

    为了促进对话驱动的开发方法并确保 Rasa Pro 对话机器人的可靠性，Rasa Pro 3.10 引入了自动端到端测试用例转换。此功能允许你将示例对话转换为使用同一版本中引入的新断言格式的端到端测试用例。此功能目前处于测试阶段（实验性），可能会在未来的 Rasa Pro 版本中发生变化。

我们引入了一项新功能，可将 CSV 或 Excel 格式提供的示例对话转换为端到端 YAML 测试用例。这样你就可以从现有的对话数据中自动生成测试用例，确保对话机器人从开发过程一开始就按照预期行事。

我们建议针对对话机器人应处理的每个技能或用例运行 30-50 个示例对话 [CLI 命令](#cli-command)。这些示例对话应该是对话机器人应该能够处理的代表性对话。

审查生成的测试用例并根据需要用其他断言进行扩充至关重要。这种人工审查可确保测试用例准确、全面且量身定制以满足特定要求，从而进一步增强测试框架的稳健性。

新功能以测试版发布，我们很乐意听到你的反馈，尤其是关于可用性及其为你的测试工作流程带来的价值。我们还有一份希望获得反馈的[问题](#beta-feedback-questions)清单。请通过客户办公室团队与我们联系以分享你的反馈。

要启用该功能，你需要在测试环境中将环境变量 `RASA_PRO_BETA_E2E_CONVERSION` 设置为 `true`。

### 端到端测试转换的好处 {#benefits-of-end-to-end-test-conversion}

- **对话驱动开发 (CDD) 和测试驱动开发 (TDD) 工作流**：此功能通过采用 TDD 原则实现 CDD，该原则基于真实对话而非预定义流指导对话机器人的开发。这种方法确保它从一开始就处理实际的用户交互。
- **效率和减少手动工作量**：自动从对话数据生成全面的测试用例，大大减少手动工作量并加快开发速度。
- **迭代改进和定制**：支持对测试用例进行持续改进，测试用例应由人工审查并根据需要添加额外的断言。

### 工作原理 {#how-it-works}

你可以使用 `rasa data convert e2e <path>` CLI 命令执行此转换。以下是该命令及其功能的详细细分。

#### CLI 命令 {#cli-command}

```shell
rasa data convert e2e <path>
```

#### 参数 {#arguments}

- `path`（必需）：包含示例对话数据的输入 CSV 或 XLS/XLSX 文件的路径。
- `-o` 或 `--output`（可选）：用于存储生成的 YAML 测试用例的输出目录。这必须是指向对话机器人项目内部位置的相对路径。默认值为 `e2e_tests`。
- `--sheet-name`（如果使用 XLS/XLSX，则为必需）：XLS/XLSX 文件中包含相关数据的工作表的名称。

#### 输入解析 {#input-parsing}

该命令读取你的输入数据文件（CSV 或 XLS/XLSX）并准备进行处理。对话可以存储在一行中或多行中。你必须在每个对话之间添加一个空行，这样才能从数据中可靠地收集对话。

每个对话消息都应与某个参与者、用户或对话机器人相关联。例如，在同一行中的单独列中，包含以下消息：

```txt
user, "I want to book a flight"
bot, "Where would you like to fly to?"
,
user, "I am hungry, I want to order the food."
bot, "What would you like to eat?"
```

作为消息的前缀：

```txt
"user: I want to book a flight"
"bot: Where would you like to fly to?"
,
"user: I am hungry, I want to order the food."
"bot: "What would you like to eat?"
```

或者，可以使用适合用例的任何其他方法。

#### 数据转换 {#data-conversion}

每个示例对话都会同时发送到大型语言模型 (LLM)，从而使执行时间相对较短。LLM 根据提供的对话数据生成 YAML 测试用例。请参阅我们使用此功能进行的 LLM 成本和性能分析。

#### 成本和性能 {#costs-and-performance}

`GPT-4o-mini` 通常表现良好，可以为大多数对话生成准确的测试用例。但是，它有时可能会遇到更复杂的测试用例。另一方面，`GPT-4o` 在所有对话中始终如一地提供正确的测试用例，表现出很高的准确性和可靠性。在成本方面，典型的对话使用大约 500 个输入 Tokens 和 300 个输出 Tokens。这意味着使用 `GPT-40-mini` 时每个对话的成本约为 0.0003 美元，使用 `GPT-4o` 时每个对话的成本约为 0.0070 美元。

#### 测试用例存储 {#storage-of-test-cases}

生成的测试用例写入 YAML 文件中。如果未使用 CLI 参数指定输出路径，则将使用默认目录 `e2e_tests`。需要注意的是，如果指定了输出路径，则它必须是指向对话机器人项目内部位置的相对路径。

#### LLM 配置 {#llm-configuration}

默认情况下，LLM 模型配置为使用 OpenAI `gpt-4o-mini` 模型以利用长上下文窗口。如果你想使用不同的模型，可以在 `conftest.yml` 文件中配置 LLM 模型，只要它位于对话机器人项目的根目录中，Rasa Pro 就可以发现它。

可以根据首选提供者配置 LLM：

- OpenAI

    ```yaml
    llm_e2e_test_conversion:
      provider: openai
      model: ...
    ```

- Azure

    ```yaml
    llm_e2e_test_conversion:
      provider: azure
      deployment: ...
      api_base: ...
    ```

- 其他 LLM 提供者

    ```yaml
    llm_e2e_test_conversion:
      model: ...
      provider: ...
    ```

### 示例用法 {#example-usage}

要将示例对话数据转换为 E2E 测试用例，请运行以下命令：

```shell
rasa data convert e2e /path/to/your/conversations.csv -o /path/to/output
```

或者对于具有特定工作表的 XLSX 文件：

```shell
rasa data convert e2e /path/to/your/conversations.xlsx --sheet-name "Sheet1" -o /path/to/output
```

### 示例测试用例生成 {#example-test-case-generation}

给出一个包含示例对话的 CSV 文件：

```txt
user, "I want to book a flight"
bot, "Where would you like to fly to?"
user, "New York"
bot, "When would you like to travel?"
user, "Tomorrow"
bot, "Please wait while I check the available flights."
...
```

运行转换命令后，示例 YAML 测试用例将如下所示：

```yaml
test_cases:
- test_case: flight_booking
  steps:
    - user: "I want to book a flight"
      assertions:
        - bot_uttered:
            text_matches: "Where would you like to fly to?"
    - user: "New York"
      assertions:
        - bot_uttered:
            text_matches: "When would you like to travel?"
    - user: "Tomorrow"
      assertions:
        - bot_uttered:
            text_matches: "Please wait while I check the available flights."
```

### Beta 反馈问题 {#beta-feedback-questions}

我们希望获得反馈的问题包括：

- 你认为将示例对话转换为 E2E 测试用例对你的开发工作流程有多大价值？
- 示例对话的输入格式 (CSV/XLS/XLSX) 是否适合你的需求？
- 在准备对话数据进行转换时，你是否遇到任何困难？
- 我们如何提高 CLI 命令及其参数的可用性？
- 你对 LLM 生成的 YAML 测试用例的性能和准确性有多满意？
- 你认为默认的 `gpt-4o-mini` 模型足以满足你的需求吗？
- 你认为在 `conftest.yml` 文件中配置 LLM 模型的过程是否简单明了？
- 生成的 YAML 测试用例在确保 Rasa 对话机器人的可靠性方面有多大用处？
- 你在使用 E2E 测试转换功能时面临哪些挑战？
- 哪些附加功能或改进会使 E2E 测试转换对你更有效？
