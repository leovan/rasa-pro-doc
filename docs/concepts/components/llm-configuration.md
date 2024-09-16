# LLM 配置

本章节为有关如何设置和配置 OpenAI、Azure 和其他提供者提供的大型语言模型的说明。在这里，你将了解需要配置的内容以及如何自定义 LLM 以有效地处理特定用例。

## 概述 {#overview}

本页适用于以下使用 LLM 的组件：

- [SingleStepLLMCommandGenerator](llm-command-generators.md#singlestepllmcommandgenerator)
- [MultiStepLLMCommandGenerator](llm-command-generators.md#multistepllmcommandgenerator)
- [EnterpriseSearchPolicy](../policies/enterprise-search-policy.md)
- [IntentlessPolicy](../policies/intentless-policy.md)
- [ContextualResponseRephraser](../contextual-response-rephraser.md)
- [LLMBasedRouter](coexistence-routers.md#llmbasedrouter)

所有上述组件均可配置为更改：

- LLM 提供者
- 要使用的模型

从 Rasa Pro 3.10 版本开始，CALM 在后台使用 [LiteLLM](https://litellm.vercel.app/) 与不同的 LLM 提供者集成。因此，所有 [LiteLLM 集成的提供者](https://litellm.vercel.app/docs/providers)都支持 CALM。我们在以下部分中明确提到了最常用的设置所需的设置。

!!! warning "警告"

    如果你想尝试 OpenAI / Azure OpenAI 以外的提供者，建议安装 Rasa Pro 版本 >= 3.10。

## 推荐模型 {#recommended-models}

下表记录了我们推荐用于各种 Rasa 组件的每种模型的版本。随着新模型的发布，Rasa 将对其进行测试，并在适当的情况下将其添加为推荐模型。

| 组件                                                         | 提供平台         | 建议使用                                                     |
| :----------------------------------------------------------- | :--------------- | :----------------------------------------------------------- |
| `SingleStepLLMCommandGenerator`<br/>`EnterpriseSearchPolicy`<br/>`IntentlessPolicy` | OpenAI<br/>Azure | `gpt-4-0613`                                                 |
| `ContextualResponseRephraser`                                | OpenAI<br/>Azure | `gpt-4-0613`<br/>`gpt-3.5-turbo-0125`                        |
| `MultiStepLLMCommandGenerator`                               | OpenAI<br/>Azure | `gpt-4-turbo-2024-04-09`<br/>`gpt-3.5-turbo-0125`<br/>`gpt-3.5-turbo-1106`<br/>`gpt-4o-2024-08-06` |

## 聊天补全模型 {#chat-completion-models}

!!! info "默认提供者"

    CALM 与 LLM 无关，可以使用不同的 LLM 配置，但 OpenAI 是默认的模型提供者。我们的大多数实验都是使用 OpenAI 或 OpenAI Azure 服务上提供的模型进行的。使用其他 LLM 时，对话机器人的性能可能会有所不同，但可以通过调整流和收集步骤描述来进行改进。

要配置使用聊天补全模型作为 LLM 的组件，请在该组件配置的 `llm` 键下声明配置。例如：

```yaml title="config.yml" hl_lines="5"
recipe: default.v1
language: en
pipeline:
- name: SingleStepLLMCommandGenerator
  llm:
     ...
```

### 必需参数 {#required-parameters}

`llm` 键下有一些必需参数：

- `model`：指定 LLM 提供者文档中可用的模型标识符的名称，例如 `gpt-4`。
- `provider`：用于调用指定模型的提供者的唯一标识符。

```yaml title="config.yml" hl_lines="6-7"
   recipe: default.v1
   language: en
   pipeline:
   - name: SingleStepLLMCommandGenerator
     llm:
        model: gpt-4
        provider: openai
```

### 可选参数 {#optional-parameters}

`llm` 键还接受推理参数，如 `temperature` 等，这些参数是可选的，但可用于从正在使用的模型中提取最佳性能。请参阅[官方 LiteLLM 文档](https://litellm.vercel.app/docs/completion/input)以获取受支持的此类参数列表。

在配置特定提供者时，有一些特定于提供者的设置，这些设置在下面每个提供者的单独子部分中进行了说明。

!!! info "重要"

    如果你切换到其他 LLM 提供者，旧提供者的所有默认参数将被新提供者的默认参数覆盖。

    例如，如果提供者设置 `temperature=0.7` 作为默认值，而你切换到其他 LLM 提供者，则此默认值将被忽略，你需要自行设置新提供者的温度。

### OpenAI {#openai}

#### API 令牌 {#api-token}

API 令牌可验证对 OpenAI API 的请求。

要配置 API 令牌，请按照以下步骤操作：

1. 如果还没有请在 OpenAI 平台上注册一个帐户。
2. 导航到 [OpenAI 密钥管理页面](https://platform.openai.com/account/api-keys)，然后单击“Create New Secret Key”按钮以启动获取 API 密钥的过程。
3. 要将 API 密钥设置为环境变量，你可以在终端或命令提示符中使用以下命令：

    === "Linux/MacOS"

        ```shell
        export OPENAI_API_KEY=<your-api-key>
        ```

    === "Windows"

        ```powershell
        setx OPENAI_API_KEY <your-api-key>
        ```

将 `<your-api-key>` 替换为你从 OpenAI 平台获得的实际 API 密钥。

#### 配置 {#configuration}

没有其他 OpenAI 特定参数需要配置。但是，你可能想要修改 `temperature` 等模型特定参数。此类参数的名称可以在 [OpenAI 的 API 文档](https://platform.openai.com/docs/api-reference/chat)中找到，并在组件配置的 `llm` 键下定义。请参阅 [LiteLLM 的文档](https://litellm.vercel.app/docs/providers/openai#openai-chat-completion-models)以了解 OpenAI 平台支持的模型列表。

#### 模型弃用 {#model-deprecations}

OpenAI 定期发布其模型的弃用时间表。你可以在 [OpenAI 发布的文档](https://platform.openai.com/docs/deprecations)中访问此时间表。

### Azure OpenAI 服务 {#azure-openai-service}

#### API 令牌 {#api-token-1}

API 令牌可验证你对 Azure OpenAI 服务的请求。

将 API 令牌设置为环境变量。你可以在终端或命令提示符中使用以下命令：

=== "Linux/MacOS"

    ```shell
    export AZURE_API_KEY=<your-api-key>
    ```

=== "Windows"

    ```powershell
    setx AZURE_API_KEY <your-api-key>
    ```

将 `<your-api-key>` 替换为你从 Azure OpenAI 服务平台获取的实际 API 密钥。

#### 配置 {#configuration-1}

要访问 Azure OpenAI 服务提供的模型，需要配置一些其他参数：

- `provider`：设置为 `azure`。
- `api_type`：要使用的 API 类型。应将其设置为 `azure` 以指示使用 Azure OpenAI 服务。
- `api_base`：Azure OpenAI 实例的 URL。示例可能如下所示：`https://my-azure.openai.azure.com/`。
- `api_version`：此动作要使用的 API 版本。这遵循 `YYYY-MM-DD` 格式，并且值应括在单引号或双引号中。
- `engine` / `deployment_name`：`deployment` 参数的别名。Azure 上的部署名称。

还可以定义 `temperature` 等模型特定参数。有关可用参数名称的信息，请参阅 [OpenAI Azure 服务的 API 文档](https://learn.microsoft.com/en-us/azure/ai-services/openai/reference#request-body)。

使用 Azure OpenAI 服务的 `SingleStepLLMCommandGenerator` 的完整示例配置如下所示：

=== "Rasa Pro >= 3.10.x"

    ```yaml title="config.yml"
    - name: SingleStepLLMCommandGenerator
      llm:
        provider: azure
        deployment: rasa-gpt-4
        api_type: azure
        api_base: https://my-azure.openai.azure.com/
        api_version: "2024-02-15-preview"
        timeout: 7
    ```

=== "3.8.x <= Rasa Pro <= 3.9.x"

    ```yaml title="config.yml"
    - name: SingleStepLLMCommandGenerator
      llm:
        engine: rasa-gpt-4
        api_type: azure
        api_base: https://my-azure.openai.azure.com/
        api_version: "2024-02-15-preview"
        request_timeout: 7
    ```

=== "Rasa Pro <= 3.7.x"

    ```yaml title="config.yml"
    - name: LLMCommandGenerator
      llm:
        deployment: rasa-gpt-4
        api_type: azure
        api_base: https://my-azure.openai.azure.com/
        api_version: "2024-02-15-preview"
        request_timeout: 7
    ```

[此处](#azure)提供了在更多 CALM 组件中使用 Azure OpenAI 服务的更全面示例。

#### 模型弃用 {#model-deprecations-1}

Azure 定期发布其 OpenAI Azure 服务下的模型的弃用计划。你可以在 [Azure 发布的文档](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/model-retirements)中访问此计划。

#### 调试 {#debugging}

如果遇到超时错误，请将 `request_timeout` 参数配置为更大的值。确切的值取决于你的 Azure 实例的配置方式。

### Amazon Bedrock {#amazon-bedrock}

#### 要求 {#requirements}

1. 确保已安装 `rasa-pro >= 3.10.x`。
2. 安装 `boto3 >= 1.28.57`。
3. 设置以下环境变量：`AWS_ACCESS_KEY_ID`、`AWS_SECRET_ACCESS_KEY`、`AWS_REGION_NAME`。
4. （可选）如果你的组织要求使用[临时凭证](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html)来确保安全，则可能必须设置 `AWS_SESSION_TOKEN`。

完成上述步骤后，编辑 `config.yaml` 以使用适当的模型并将 `provider` 设置为 `bedrock`：

```yaml title="config.yml" hl_lines="3-4"
- name: SingleStepLLMCommandGenerator
  llm:
    provider: bedrock
    model: anthropic.claude-instant-v1
```

还可以定义模型特定参数（例如 `temperature`）。有关[可用参数名称](https://litellm.vercel.app/docs/completion/input#translated-openai-params)和[支持模型](https://litellm.vercel.app/docs/providers/bedrock#supported-aws-bedrock-models)的信息，请参阅 LiteLLM 的文档。

### Gemini - Google AI Studio {#gemini---google-ai-studio}

#### 要求 {#requirements-1}

1. 确保已安装 `rasa-pro >= 3.10.x`。
2. 安装 python 包 `google-generativeai`。
3. 在 <https://aistudio.google.com/> 获取 API 密钥。
4. 将 API 密钥设置为环境变量 `GEMINI_API_KEY`。

完成上述步骤后，编辑 `config.yaml` 以使用适当的模型并将 `provider` 设置为 `gemini`：

```yaml title="config.yml" hl_lines="3-4"
- name: SingleStepLLMCommandGenerator
  llm:
    provider: gemini
    model: gemini-pro
```

请参阅 LiteLLM 的文档以了解支持哪些[附加参数](https://litellm.vercel.app/docs/providers/gemini#supported-openai-params)和[模型](https://litellm.vercel.app/docs/providers/gemini#chat-models)。

### HuggingFace Inference Endpoints {#huggingface-inference-endpoints}

#### 要求 {#requirements-2}

1. 确保已安装 `rasa-pro >= 3.10.x`。
2. 将 API 密钥设置为环境变量 `HUGGINGFACE_API_KEY`。
3. 编辑 `config.yaml` 以使用适当的模型，将 `provider` 设置为 `huggingface`，将 `api_base` 设置为已部署端点的基础 URL：

```yaml title="config.yml" hl_lines="3-5"
- name: SingleStepLLMCommandGenerator
  llm:
    provider: huggingface
    model: meta-llama/CodeLlama-7b-Instruct-hf
    api_base: "https://my-endpoint.huggingface.cloud"
```

### 自托管模型服务器 {#self-hosted-model-server}

CALM 的组件也可以配置为与托管在开源模型服务器（如 [vLLM](https://github.com/vllm-project/vllm)（推荐）、[Ollama](https://ollama.com/) 或 [Llama.cpp Web 服务器](https://github.com/ggerganov/llama.cpp?tab=readme-ov-file#web-server)）上的开源 LLM 配合使用。唯一的要求是模型服务器应遵循 [OpenAI API 格式](https://platform.openai.com/docs/api-reference/introduction)。

一旦模型服务器运行，请配置 CALM 对话机器人的 `config.yaml`：

#### vLLM

```yaml title="config.yml" hl_lines="3-5"
- name: SingleStepLLMCommandGenerator
  llm:
    provider: self-hosted
    model: meta-llama/CodeLlama-7b-Instruct-hf
    api_base: "https://my-endpoint/v1"
```

重要提示：

1. 建议使用的 `vllm` 版本为 `0.6.0`。
2. `model` 应包含提供给 `vllm` 启动命令的模型名称，例如，如果你的模型服务器使用以下命令启动：

    ```
    vllm serve meta-llama/CodeLlama-7b-Instruct-hf
    ```

    `model` 应设置为 `meta-llama/CodeLlama-7b-Instruct-hf`。

4. `api_base` 应包含模型服务器的完整公开 URL，并将 `v1` 作为后缀附加到 URL。

#### Ollama

一旦 ollama 模型服务器运行，编辑 `config.yaml` 文件：

```yaml title="config.yml" hl_lines="3-5"
- name: SingleStepLLMCommandGenerator
  llm:
    provider: ollama
    model: llama3.1
    api_base: "https://my-endpoint"
```

### 其他提供者 {#other-providers}

!!! info "信息"

    如果你想尝试其中一个提供者，建议安装 Rasa Pro 版本 `>= 3.10`。

除了上述提供者之外，我们还测试了对以下提供者的支持：

| 平台        | `provider`    | API-KEY 变量         |
| :---------- | :------------ | :------------------- |
| Anthropic   | `anthropic`   | `ANTHROPIC_API_KEY`  |
| Cohere      | `cohere`      | `COHERE_API_KEY`     |
| Mistral     | `mistral`     | `MISTRAL_API_KEY`    |
| Together AI | `together_ai` | `TOGETHERAI_API_KEY` |
| Groq        | `groq`        | `GROQ_API_KEY`       |

对于上述每一个，请确保已将 API-KEY 变量列中的值命名为该平台的 API 密钥的环境变量，并将组件配置的 `llm` 密钥下的 `provider` 参数设置为 `provider` 列中的值。

## 嵌入模型 {#embedding-models}

要配置使用嵌入模型的组件，请在该组件配置的 `embeddings` 键下声明配置。例如：

```yaml title="config.yml" hl_lines="6"
pipeline:
  - name: "SingleStepLLMCommandGenerator"
    llm:
      model: gpt-4
    flow_retrieval:
      embeddings:
        ...
```

`embeddings` 属性需要两个强制参数：

1. `model`：指定 LLM 提供者文档中可用的模型标识符的名称，例如 `text-embedding-ada-002`。
2. `provider`：用于调用指定模型的提供者的唯一标识符，例如 `openai`。

=== "Rasa Pro >= 3.10.x"

    ```yaml title="config.yml" hl_lines="7-8"
    pipeline:
      - name: "SingleStepLLMCommandGenerator"
        llm:
          provider: openai
          model: gpt-4
        flow_retrieval:
          embeddings:
            provider: openai
            model: text-embedding-ada-002
    ```

=== "Rasa Pro <= 3.9.x"

    ```yaml title="config.yml" hl_lines="7-8"
    pipeline:
      - name: "SingleStepLLMCommandGenerator"
        llm:
          model: gpt-4
        flow_retrieval:
          embeddings:
            type: openai
            model: text-embedding-ada-002
    ```

配置特定提供者时，有一些提供者特定的设置，这些设置在下面每个提供者的单独子部分中进行了解释。

### OpenAI {#openai-1}

OpenAI 用作默认嵌入模型提供者。要开始使用，请确保你已配置 API 令牌，就像为 [OpenAI 平台的聊天补全模型](#api-token)所做的那样。

#### 配置 {#configuration-2}

=== "Rasa Pro >= 3.10.x"

    ```yaml title="config.yml" hl_lines="7-8"
    pipeline:
      - name: "SingleStepLLMCommandGenerator"
        llm:
          provider: openai
          model: gpt-4
        flow_retrieval:
          embeddings:
            provider: openai
            model: text-embedding-ada-002
    ```

=== "Rasa Pro <= 3.9.x"

    ```yaml title="config.yml" hl_lines="7-8"
    pipeline:
      - name: "SingleStepLLMCommandGenerator"
        llm:
          model: gpt-4
        flow_retrieval:
          embeddings:
            type: openai
            model: text-embedding-ada-002
    ```

### Azure OpenAI 服务 {#azure-openai-service-1}

确保你已配置 API 令牌，就像为 [Azure OpenAI 服务的聊天补全模型](#api-token-1)配置一样。

#### 配置 {#configuration-3}

从 Azure OpenAI 服务配置嵌入模型需要与从 [Azure OpenAI 服务配置聊天补全模型](#configuration-1)所需的相同参数集的值。

=== "Rasa Pro >= 3.10.x"

    ```yaml title="config.yml" hl_lines="7-12"
    pipeline:
      - name: "SingleStepLLMCommandGenerator"
        llm:
          model: gpt-4
        flow_retrieval:
          embeddings:
            provider: azure
            deployment: engine-embed
            api_type: azure
            api_base: https://my-azure.openai.azure.com/
            api_version: "2024-02-15-preview"
            timeout: 7
    ```

=== "Rasa Pro <= 3.9.x"

    ```yaml title="config.yml" hl_lines="7-12"
    pipeline:
      - name: "SingleStepLLMCommandGenerator"
        llm:
          model: gpt-4
        flow_retrieval:
          embeddings:
            type: azure
            engine: engine-embed
            api_type: azure
            api_base: https://my-azure.openai.azure.com/
            api_version: "2024-02-15-preview"
            request_timeout: 7
    ```

### Amazon Bedrock {#amazon-bedrock-1}

配置来自 amazon bedrock 的嵌入模型需要与[聊天补全模型](#requirements)相同的先决条件。请确保在继续操作之前已解决这些问题。

#### 配置 {#configuration-4}

```yaml title="config.yml" hl_lines="7-8"
pipeline:
  - name: "SingleStepLLMCommandGenerator"
    llm:
      provider: openai
      model: gpt-4
    flow_retrieval:
      embeddings:
        provider: bedrock
        model: amazon.titan-embed-text-v1
```

请参阅 LiteLLM 文档中有关 Amazon Bedrock 支持的[嵌入模型列表](https://litellm.vercel.app/docs/providers/bedrock#supported-aws-bedrock-embedding-models)。

### In-Memory {#configuration-4}

CALM 还提供了一个选项，可以在内存中加载轻量级嵌入模型，而无需通过 API 使用它们。它使用底层的句子转换器库来加载和运行推理。

#### 配置 {#configuration-5}

=== "Rasa Pro >= 3.10.x"

    ```yaml title="config.yml" hl_lines="7-12"
    pipeline:
      - name: "SingleStepLLMCommandGenerator"
        llm:
          provider: "openai"
          model: "gpt-4"
        flow_retrieval:
          embeddings:
            provider: "huggingface_local"
            model: "BAAI/bge-small-en-v1.5"
            model_kwargs: # used during instantiation
              device: "cpu"
            encode_kwargs: # used during inference
              normalize_embeddings: true
    ```

=== "Rasa Pro <= 3.9.x"

    ```yaml title="config.yml" hl_lines="7-12"
    pipeline:
      - name: "SingleStepLLMCommandGenerator"
        llm:
          model: gpt-4
        flow_retrieval:
          embeddings:
            type: "huggingface"
            model: "BAAI/bge-small-en-v1.5"
            model_kwargs: # used during instantiation
              device: 'cpu'
            encode_kwargs: # used during inference
              normalize_embeddings: True
    ```

- `model` 参数将 HuggingFace 中心上可用的任何嵌入模型存储库作为值。
- `model_kwargs` 参数用于为句子转换器库提供[加载时间参数](https://sbert.net/docs/package_reference/sentence_transformer/SentenceTransformer.html#id1)。
- `encode_kwargs` 参数用于为句子转换器库提供[推理时间参数](https://sbert.net/docs/package_reference/sentence_transformer/SentenceTransformer.html#sentence_transformers.SentenceTransformer.encode)。

### 其他提供者 {#other-providers-1}

除了上述提供者之外，我们还测试了对以下提供者的支持：

| 平台      | `provider` | API-KEY 变量      |
| :-------- | :--------- | :---------------- |
| Cohere    | `cohere`   | `COHERE_API_KEY`  |
| Mistral   | `mistral`  | `MISTRAL_API_KEY` |
| Voyage AI | `voyage`   | `VOYAGE_API_KEY`  |

对于上述每一个，请确保已将 API-KEY 变量列中的值命名为该平台的 API 密钥的环境变量，并将组件配置的 `llm` 密钥下的 `provider` 参数设置为 `provider` 列中的值。

## FAQ {#faq}

### OpenAI 会使用我的数据来训练他们的模型吗？ {#does-openai-use-my-data-to-train-their-models}

不会。OpenAI 不会使用你的数据来训练他们的模型。摘自他们的[网站](https://openai.com/security)：

> Data submitted through the OpenAI API is not used to train OpenAI models or improve OpenAI's service offering.

## 示例配置 {#example-configurations}

### Azure {#azure}

一个全面的示例，其中包括：

- `config.yml` 中组件的 `llm` 和 `embeddings`：
    - `IntentlessPolicy`
    - `EnterpriseSearchPolicy`
    - `SingleStepLLMCommandGenerator`
    - 3.8.x 中的 `flow_retrieval`
- `endpoints.yml` 中用于改写的 `llm `配置（`ContextualResponseRephraser`）

=== "Rasa Pro >= 3.10.x"

    ```yaml title="endpoints.yml"
    nlg:
      type: rephrase
      llm:
        provider: azure
        deployment: rasa-gpt-4
        api_type: azure
        api_version: "2024-02-15-preview"
        api_base: https://my-azure.openai.azure.com
        timeout: 7
    ```

=== "3.8.x <= Rasa Pro <= 3.9.x"

    ```yaml title="endpoints.yml"
    nlg:
      type: rephrase
      llm:
        engine: rasa-gpt-4
        api_type: azure
        api_version: "2024-02-15-preview"
        api_base: https://my-azure.openai.azure.com
        request_timeout: 7
    ```

=== "Rasa Pro <= 3.7.x"

    ```yaml title="endpoints.yml"
    nlg:
      type: rasa_plus.ml.ContextualResponseRephraser
      llm:
        engine: rasa-gpt-4
        api_type: azure
        api_version: "2024-02-15-preview"
        api_base: https://my-azure.openai.azure.com
        request_timeout: 7
    ```

```yaml title="config.yml"
recipe: default.v1
language: en
pipeline:
- name: LLMCommandGenerator
  llm:
    engine: rasa-gpt-4
    api_type: azure
    api_base: https://my-azure.openai.azure.com/
    api_version: "2024-02-15-preview"
    request_timeout: 7

policies:
- name: FlowPolicy
- name: rasa_plus.ml.IntentlessPolicy
  llm:
    engine: rasa-gpt-4
    api_type: azure
    api_base: https://my-azure.openai.azure.com/
    api_version: "2024-02-15-preview"
    request_timeout: 7
  embeddings:
    model: text-embedding-3-small
    engine: rasa-embedding-small
    api_type: azure
    api_base: https://my-azure.openai.azure.com/
    api_version: "2024-02-15-preview"
    request_timeout: 7
- name: rasa_plus.ml.EnterpriseSearchPolicy
  vector_store:
    type: "faiss"
    threshold: 0.0
  llm:
    model: gpt-4
    engine: rasa-gpt-4
    api_type: azure
    api_base: https://my-azure.openai.azure.com/
    api_version: "2024-02-15-preview"
    request_timeout: 7
  embeddings:
    model: text-embedding-3-small
    engine: rasa-embedding-small
    api_type: azure
    api_base: https://my-azure.openai.azure.com/
    api_version: "2024-02-15-preview"
    request_timeout: 7
```
