# 使用 GitHub Codespaces 快速入门

你可以立即在浏览器中开始使用 Rasa Pro，无需安装。

## 前置条件 {#prerequisites}

- 一个 Rasa Pro [许可证密钥](../developer-edition.md)和一个 OpenAI 或其他 LLM 提供者的 API 密钥。
- 一个 GitHub 帐户。

## 步骤 {#steps}

### 创建代码空间 {#create-a-codespace}

- 导航到 GitHub 上的[此存储库](https://github.com/RasaHQ/codespaces-quickstart/)。
- 单击“Code”，然后单击“Create codespace on main”。

<figure markdown>
  ![](../images/installation/create-a-codespace.png){ width="400" }
</figure>

### 设置环境 {#set-up-environment}

编辑名为 `.env` 的文件，内容如下：

```bash title=".env"
RASA_PRO_LICENSE='your_rasa_pro_license_key_here'
OPENAI_API_KEY='your_openai_api_key_here'
```

假设你正在使用 OpenAI，也可以使用[其他 LLM 提供者](../concepts/components/llm-configuration.md#other-providers)。通过运行以下命令从文件中加载这些环境变量：

```bash
source .env
```

然后运行以下命令激活 Python 环境：

```bash
source .venv/bin/activate
```

### 按照教程操作 {#follow-the-tutorial}

代码空间已设置好，现在可以运行 `rasa train` 和 `rasa inspect` 等命令。查看[教程](../tutorial.md)来构建你的第一个 CALM 对话机器人。
