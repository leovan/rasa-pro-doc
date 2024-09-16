# 为命令生成微调和自托管 LLM

[微调配方](../operating/fine-tuning-recipe.md)允许你微调基本 LLM 以执行命令生成任务。此微调模型可用作 CALM 中[基于 LLM 的命令生成器](../concepts/components/llm-command-generators.md)的一部分。这可以帮助缓解延迟、可靠性问题，并允许控制 LLM。

当前页面是一个教程，涵盖微调基本 LLM 并自行部署以执行命令生成任务所需的所有步骤。此[页面](../operating/fine-tuning-recipe.md)提供了配方的更概念性的概述。

使用微调配方的先决条件是你已经构建了一个对话机器人并编写了一些 [E2E 测试](../production/testing-your-assistant.md#end-to-end-testing)。为了微调基础模型并在 CALM 中使用它，你需要执行以下步骤：

- [步骤 1：确保全面覆盖系统以进行微调](#step-1-ensure-comprehensive-coverage-of-your-system-for-fine-tuning)
- [步骤 2：准备微调数据集](#step-2-prepare-the-fine-tuning-dataset)
- [步骤 3：微调基础模型](#step-3-fine-tune-a-base-model)
- [步骤 4：评估微调模型](#step-4-evaluate-your-fine-tuned-model)
- [步骤 5：创建可用于生产的微调模型](#step-5-create-a-production-ready-fine-tuned-model)
- [步骤 6：将微调模型部署到生产环境](#step-6-deploy-the-fine-tuned-model-to-production)
- [步骤 7：将对话机器人连接到生产环境中的微调模型](#step-7-connect-the-assistant-to-the-fine-tuned-model-in-production)

## 步骤 1：确保全面覆盖你的系统以进行微调 {#step-1-ensure-comprehensive-coverage-of-your-system-for-fine-tuning}

首先，通过将 `RASA_PRO_BETA_FINE_TUNING_RECIPE` 功能标志设置为 `True` 来启用该功能。

```shell
export RASA_PRO_BETA_FINE_TUNING_RECIPE=true
```

如上所述，微调配方需要样本对话来训练模型，这些样本对话以 E2E 测试格式编写。为了评估现有 E2E 测试的广度，让我们运行 [E2E 测试](../production/testing-your-assistant.md#e2e-test-coverage-report)，但要打开覆盖率标志。

```shell
rasa test e2e <path-to-test-cases> --coverage-report
```

上述命令记录了哪些流程步骤和命令被成功覆盖。然后它会生成：

- E2E 测试对流程步骤的覆盖[报告](../production/testing-your-assistant.md#flow-coverage)。
- 覆盖命令的[直方图](../production/testing-your-assistant.md#command-coverage)。
- 所有通过和失败的测试的单独文件。

!!! important "重要"

    检查流程覆盖率报告，确保 E2E 测试在流程中的总覆盖率至少达到 90%，并且没有单个流程的覆盖率低于 80%。还要检查命令覆盖率直方图，以确保每个命令都得到很好的表示。上述两项检查对于生成用于微调 LLM 的高质量数据都很重要。

## 步骤 2：准备微调数据集 {#step-2-prepare-the-fine-tuning-dataset}

!!! warning "警告"

    此步骤需要使用经过 `SingleStepLLMCommandGenerator` 训练的 Rasa 模型和为命令生成器配置的强 LLM（例如 `gpt-4`）来执行。

!!! info "信息"

    由于进行了多次 LLM 调用，此步骤需要一些时间才能执行。

要从通过的 E2E 测试中创建用于微调的训练和验证数据集，请执行以下[命令](../command-line-interface.md#rasa-llm-finetune-prepare-data)：

```shell
rasa llm finetune prepare-data <path-to-e2e-test-cases>
```

默认情况下，[步骤 1](#step-1-ensure-comprehensive-coverage-of-your-system-for-fine-tuning) 中通过的 E2E 测试将写入 `e2e_coverage_results/passed.yml` 文件。你可以将其用作此步骤的 E2E 测试，即：

```shell
rasa llm finetune prepare-data e2e_coverage_results/passed.yml
```

该命令将使用 `models` 文件夹中最新训练的 Rasa 模型来准备微调数据。如果要使用其他模型，可以设置标志 `--model <path-to-model>`。

数据准备脚本执行多项任务：

- 命令注释
- 合成数据生成
- 训练和验证数据集生成

有关这些任务如何工作的详细信息，请查看[微调配方的概念概述](../operating/fine-tuning-recipe.md#conceptual-overview)。

默认情况下，数据准备命令的输出将写入 `output` 文件夹。你可以通过设置标志 `--out <folder-name>` 来修改输出路径。输出文件夹具有以下结构：

```txt
.
├── output
│   ├── 1_command_annotations
│   └── 2_rephrasings
│   └── 3_llm_finetune_data
│   └── 4_train_test_split
│   │   └── e2e_tests
│   │   │   └── train.yml
│   │   │   └── validation.yaml
│   │   └── ft_splits
│   │   │   └── train.jsonl
│   │   │   └── val.jsonl
│   └── result_summary.yaml
│   └── params.yaml
```

最重要的文件位于 `output/4_train_test_split` 下。其余文件仅用于调试目的。如果你对这些数据的样子感兴趣，请查看[微调配方的概念概述](../operating/fine-tuning-recipe.md#conceptual-overview)。

[步骤 3](#step-3-fine-tune-a-base-model) 所需的训练和验证数据集位于 `output/4_train_test_split/ft_splits/train.jsonl` 和 `output/4_train_test_split/ft_splits/val.jsonl` 下。

要评估你的微调模型（参见[步骤 4](#step-4-evaluate-your-fine-tuned-model)），你将需要位于 `output/4_train_test_split/e2e_tests` 下的 E2E 测试。

## 步骤 3：微调基础模型 {#step-3-fine-tune-a-base-model}

### 环境配置 {#environment-configuration}

推荐硬件：

- NVIDIA A100 GPU（40GB VRAM）
- 12 核 CPU，64GB RAM
- 256GB 磁盘

推荐软件：

- Python 3.10
- CUDA Toolkit 12.1
- PyTorch 2.2

!!! warning "警告"

    尽管这些指令可以在性能相对较弱的 GPU（例如 NVIDIA T4）上运行，但实际的微调和推理会非常缓慢。

    强烈建议你使用 NVIDIA A100 或其他类似的 GPU 类型。

配置上述硬件并安装相关软件后，部署此 Python Notebook 并运行单元以完成其余的模型微调步骤。Notebook 的前几个单元也有助于设置环境。

下面，我们重点介绍了几个重要步骤，并针对你可以尝试的不同设置提出了建议。

!!! info "信息"

    如果用于微调模型的 Python Notebook 中提供的代码有任何问题，请在 [Github 上的源代码存储库](https://github.com/RasaHQ/notebooks)中打开一个问题。

### 基础模型 {#base-model}

微调的基础模型从 Hugging Face 模型中心下载。Python Notebook 会安装[相应的软件包](https://nbviewer.org/github/RasaHQ/notebooks/blob/main/cmd_gen_finetuning.ipynb#2.-Install-Python-requirements)以执行此操作。

接下来，确保在[此单元格](https://nbviewer.org/github/RasaHQ/notebooks/blob/main/cmd_gen_finetuning.ipynb#3.-Download-base-model)中添加 `HUGGINGFACE_TOKEN` 和 `BASE_MODEL` 变量，并使用你自己的值。

我们建议从 [Llama 3.1 8B Instruct 模型](https://huggingface.co/meta-llama/Meta-Llama-3.1-8B-Instruct)开始作为微调的基础模型。如果你想尝试不同的模型，我们建议使用在预训练期间专门调整指令的模型。这将确保模型经过预训练，可以忠实地遵循提示中的指令。一个很好的替代模型是 [CodeLlama 13b Instruct 模型](https://huggingface.co/codellama/CodeLlama-13b-Instruct-hf)。

### 训练和验证数据集 {#training-and-validation-datasets}

确保在[步骤 2](#step-2-prepare-the-fine-tuning-dataset) 中创建的训练和验证数据集（即 `train.jsonl` 和 `val.jsonl`）在运行微调的磁盘上可用。提供的代码应该能够加载数据集并自动格式化它们。

由于文件使用 TRL 指令格式，稍后使用的 TRL 训练器将能够[自动解析](https://huggingface.co/docs/trl/en/sft_trainer#dataset-format-support)数据集并从分词器中配置的模板[生成提示](https://huggingface.co/docs/transformers/en/chat_templating)。

提示模板因模型而异，TRL 将从你的基础模型推断出正确的模板。如果你的基础模型没有此功能，或者你希望更改它，可以设置自己的[模板字符串](https://huggingface.co/docs/transformers/en/chat_templating#advanced-adding-and-editing-chat-templates)。

你还可以[定义自己的提示格式化函数](https://docs.unsloth.ai/basics/chat-templates)，以便完全控制提示的构造方式。

### 训练超参数 {#training-hyper-parameters}

[此部分提供的代码](https://nbviewer.org/github/RasaHQ/notebooks/blob/main/cmd_gen_finetuning.ipynb#6.-Configure-trainer)使用 TRL 库中的 [SFTConfig](https://github.com/huggingface/trl/blob/main/trl/trainer/sft_config.py) 和 [SFTTrainer](https://huggingface.co/docs/trl/main/en/sft_trainer#trl.SFTTrainer)。在我们进行的一些内部实验中，超参数的默认值效果最好，但是你可以使用一些参数：

1. 如果在运行微调时出现 OOM 错误，可以减少 `per_device_train_batch_size` 以减少内存占用。但是，如果你的 GPU 有足够的内存，可以尝试增加它以减少总训练步骤数。
2. 考虑调整 `max_steps`，因为可能不需要执行所有 `epoch` 来实现最佳模型精度。相反，通过增加 `num_train_epochs`，可能会看到更好的模型精度。
3. 如果模型训练花费的时间太长，可以增加 `eval_steps` 以减少执行验证的频率。

### 保存训练好的模型 {#saving-the-trained-model}

[提供的代码](https://nbviewer.org/github/RasaHQ/notebooks/blob/main/cmd_gen_finetuning.ipynb#7.-Perform-supervised-fine-tuning)使用 16 位精度保存训练好的模型。它还将 LoRA 适配器与基础模型的权重合并，并将所有参数作为微调模型的一部分保存。如果你使用的是相对较小的 GPU，例如 NVIDIA T4，则可能必须以 4 位保存模型（例如 `save_method = "merged_4bit_forced"`）。

### 可视化训练指标 {#visualizing-training-metrics}

模型训练完成后，[绘制](https://nbviewer.org/github/RasaHQ/notebooks/blob/main/cmd_gen_finetuning.ipynb#8.-Visualize-fine-tuning-metrics)训练和验证损失图。观察以下图表：

1. 理想情况下，随着微调步骤的增加，训练和验证损失应该会减少并收敛。
2. 如果两个损失曲线不收敛，则可能值得执行更多微调步骤或批次。这种情况称为[欠拟合](https://www.ibm.com/topics/underfitting)。
3. 如果验证损失突然开始增加，而训练损失继续减少或收敛，则应减少总步骤数或批次。这称为[过拟合](https://www.ibm.com/topics/overfitting)。

### 导出微调模型 {#exporting-fine-tuned-model}

将微调后的模型目录[导出](https://nbviewer.org/github/RasaHQ/notebooks/blob/main/cmd_gen_finetuning.ipynb#10.-Export-fine-tuned-model)到适当的存储位置，以便以后轻松访问以进行[部署](../building-assistants/fine-tuning-recipe.md#step-6-deploy-the-fine-tuned-model-to-production)。

建议使用云对象存储，例如 [Amazon S3](https://aws.amazon.com/s3/) 或 [Google Cloud Storage](https://cloud.google.com/storage)。

## 步骤 4：评估你的微调模型 {#step-4-evaluate-your-fine-tuned-model}

一旦拥有微调模型，就必须对其进行评估并交叉检查其性能。请按照以下步骤执行此操作：

1. 确保已将微调模型下载到至少可以访问 A100 GPU 的云实例上。
2. 使用 `pip install vllm==0.6.0` [安装](https://docs.vllm.ai/en/latest/getting_started/installation.html) `vllm`。有关更多信息，请点击[此处](../deploy/deploy-fine-tuned-model.md#deploy-the-fine-tuned-model-to-a-development-environment)。
3. 使用 `vllm serve finetuned_model` 运行模型服务器，其中 `finetuned_model` 是包含模型工件的目录的名称。这应该在 `localhost` 网络接口上运行模型服务器。如果对话机器人位于不同的实例/机器上，则应在公共 IP 上公开模型服务器，以便可以从外部机器对其进行 ping。
4. 配置对话机器人以使用模型服务器：

    ```yaml title="config.yml" hl_lines="3-5"
    - name: SingleStepLLMCommandGenerator
      llm:
        model: finetuned_model
        provider: self-hosted
        api_base: <URL of the model server> # `localhost` if assistant and model server on same machine
    ```

5. 使用 `rasa train` 训练对话机器人。
6. 运行：

    ```shell
    rasa test e2e <path to validation E2E tests>
    ```

    `<path to validation E2E tests>` 是[步骤 2](#step-2-prepare-the-fine-tuning-dataset) 中生成的输出目录中 `4_train_test_split/e2e_tests/validation.yaml` 的路径。E2E 测试的输出应显示通过和失败的 E2E 测试对话的数量。这些 E2E 测试对话在基础模型的微调期间没有使用，因此它们很好地说明了微调模型在不同自然语言变体的用户消息和训练数据中不存在的流领域中的泛化能力。

7. 作为健全性检查，检查微调模型在用于生成用于微调 LLM 的数据的 E2E 测试上的性能也是很好的。为此，请运行：

    ```shell
    rasa test e2e <path to training E2E tests>
    ```

    `<path to training E2E tests>` 是[步骤 2](#step-2-prepare-the-fine-tuning-dataset) 中生成的输出目录内的 `4_train_test_split/e2e_tests/train.yaml` 的路径。这样做可确保微调没有完全脱轨。

为了评估微调模型是否表现良好，可以遵循以下几点：

1. 当使用微调模型作为命令生成器时，对话机器人能够通过来自 `validation.yaml` 的至少 80% 的 E2E 测试对话和来自 `train.yaml` 的 95% 的 E2E 测试对话。
2. 当对上述 `train.yml` 和 `val.yml` 进行评估时，使用微调的 LLM 作为命令生成器的d对话机器人的性能与使用强大的 LLM（例如 `gpt-4`）作为命令生成器的对话机器人的性能相当。

如果上述条件都不成立，你应该投入精力创建更多示例对话，用现有对话进行扩充，生成微调数据并微调新模型进行评估。

## 步骤 5：创建可用于生产的微调模型 {#step-5-create-a-production-ready-fine-tuned-model}

此步骤涉及重新训练模型，但这次将训练集 (`train.jsonl`) 和验证集 (`val.jsonl`) 合并​​为单个训练数据集。此步骤是可选的，应在训练数据有限的情况下使用。训练的有效性和超参数的选择应在[步骤 4](#step-4-evaluate-your-fine-tuned-model) 期间确定，此时有可用的验证集。完成后，通过使用相同的超参数和稍大的训练数据集（包括验证集）重新训练基础模型，可以获得性能的小幅提升，如下所示：

1. 合并训练和验证集例如：

    ```shell
    cat output/4_train_test_split/ft_splits/*.jsonl > output/4_train_test_split/ft_splits/combined_train_val.jsonl
    ```

2. 使用 `combined_train_val.jsonl` 按照[步骤 3：微调基础模型](#step-3-fine-tune-a-base-model) 重新训练模型。
3. 按照[步骤 4](#step-4-evaluate-your-fine-tuned-model) 评估此微调模型的性能，进行一些手动测试也可能有助于确保训练成功。

## 步骤 6：将微调模型部署到生产环境 {#step-6-deploy-the-fine-tuned-model-to-production}

假设你已：

- 已在具有 [NVIDIA GPU](https://docs.docker.com/desktop/gpu/) 的 VM 实例中安装 [Docker](https://docs.docker.com/engine/install/) 和 [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)。
- 实例上的 `finetuned_model` 目录中提供了微调模型文件。

你可以使用下面的 `docker` 命令在该实例上启动 vLLM 服务器。请确保首先更新 `--served-model-name` 标志的值，该标志将覆盖 API 调用中使用的模型名称。

```shell
docker run --runtime nvidia --gpus all \
    -v "./finetuned_model:/mnt/models" \
    -p 8000:8000 --ipc="host" \
    vllm/vllm-openai:latest \
    --model "/mnt/models"  \
    --served-model-name "llama-fine-tuned"
```

然后，需要将 vLLM 服务器端口公开给外界，以便对话机器人可以通过公共 IP 地址访问它。

如果你希望使用 Kubernetes 进行部署，请参阅[此页面上的详细信息](../deploy/deploy-fine-tuned-model.md#kubernetes-cluster)。

## 步骤 7：将对话机器人连接到生产中的微调模型 {#step-7-connect-the-assistant-to-the-fine-tuned-model-in-production}

更改对话机器人的配置以连接到生产中部署的微调模型，作为[步骤 6](#step-6-deploy-the-fine-tuned-model-to-production) 的一部分：

```yaml title="config.yml" hl_lines="3-5"
- name: SingleStepLLMCommandGenerator
  llm:
    model: llama-fine-tuned
    provider: self-hosted
    api_base: <URL of the model server from step 6>
```

使用 `rasa train` 重新训练对话机器人。
