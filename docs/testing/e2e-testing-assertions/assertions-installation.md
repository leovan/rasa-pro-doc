# 断言的安装先决条件

了解在端到端测试中使用断言的安装先决条件。

## 安装和配置先决条件 {#installation-and-configuration-prerequisites}

要在端到端测试中使用断言，请[安装 `rasa-pro` 包](../../installation/python/installation.md)并使用 Rasa Pro 的[有效许可证密钥](../../installation/python/licensing.md)。

### 可选依赖项 {#optional-dependency}

要在端到端测试中评估生成对话机器人响应的相关性和事实准确性，请安装可选依赖项 [`mlflow`](https://mlflow.org/docs/latest/model-evaluation/index.html) 以启用这些功能。此依赖项使用 [LLM（大型语言模型）评估](https://mlflow.org/docs/latest/llms/llm-evaluate/index.html)来评估 Rasa Pro 对话机器人生成响应的相关性和事实准确性。此 LLM 也称为“LLM-as-Judge”模型，因为它评估另一个模型的输出。在 Rasa Pro 的用例中，LLM-as-Judge 模型评估[生成响应是否与提供的输入相关](assertions-fundamentals.md#generative-response-is-relevant-assertion)，或者[生成响应相对于提供或提取的真实文本输入是否具有事实准确性](assertions-fundamentals.md#generative-response-is-grounded-assertion)。

你可以使用以下命令安装依赖项：

```shell
pip install rasa-pro[mlflow]
# or if you are using poetry
poetry add "rasa-pro[mlflow]"
poetry add rasa-pro -E mlflow
```

#### LLM Judge 生成响应配置 {#generative-response-llm-judge-configuration}

!!! info "信息"

    Rasa Pro 3.10 针对 LLM Judge 模型仅支持 OpenAI 模型。

默认情况下，LLM Judge 模型配置为使用 OpenAI `gpt-4o-mini` 模型以利用长上下文窗口。如果要使用其他模型，可以在 `conftest.yml` 文件中配置 LLM Judge 模型，该文件是 Rasa Pro 3.10 中添加的新测试配置文件。只要将其放在对话机器人项目的根目录中，Rasa Pro 就会自动发现它。

```yaml
llm_as_judge:
  api_type: openai
  model: "gpt-4"
```

### 环境变量 {#environment-variables}

要启用该功能，请在测试环境中将环境变量 `RASA_PRO_BETA_E2E_ASSERTIONS` 设置为 `true`。

```shell
export RASA_PRO_BETA_E2E_ASSERTIONS=true
```
