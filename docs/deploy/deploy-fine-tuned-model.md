# 使用 vLLM 部署微调模型以生成命令

!!! info "3.10 版本新特性"

    本页与[微调配方](../building-assistants/fine-tuning-recipe.md)有关，这是从 3.10.0 版本开始提供的测试版功能。

在为命令生成任务微调了基础语言模型后，你可以将其部署以供开发或生产 CALM 对话机器人使用。

对话机器人必须能够通过 [OpenAI 兼容 API](https://platform.openai.com/docs/api-reference/introduction) 访问部署的微调模型。

强烈建议你使用 [vLLM](https://docs.vllm.ai/en/latest/index.html) 模型服务库（版本 0.6.0），因为它提供了 [OpenAI 兼容服务器](https://docs.vllm.ai/en/latest/serving/openai_compatible_server.html)。

本页提供了有关如何在不同环境中使用 vLLM 部署微调模型的建议。可以使用相同的说明来部署 [Hugging Face](https://huggingface.co/models) 中的任何语言模型，不一定是你微调的模型。

!!! info "信息"

    部署模型后，你必须[更新对话机器人的配置](../concepts/components/llm-configuration.md#self-hosted-model-server)，以便它使用 vLLM 模型服务器后面的模型进行命令生成。

## 系统要求 {#system-requirements}

部署语言模型需要 GPU 加速器才能实现快速推理延迟。

如果你使用的是具有数十亿个参数的语言模型（例如 Llama 3.1），则强烈建议你使用具有相对较大内存的 GPU，例如 NVIDIA A100。

你应该使用具有至少与微调模型所用资源量相同的资源量的系统。

## 将微调模型部署到开发环境 {#deploy-the-fine-tuned-model-to-a-development-environment}

假设你已经[安装了 `vLLM==0.6.0`](https://docs.vllm.ai/en/latest/getting_started/installation.html)，并且微调模型文件位于名为 `finetuned_model` 的目录中，你可以将其部署到本地作为 vLLM 服务器进行开发：

```shell
vllm serve finetuned_model
```

如果你也在本地运行对话机器人，它将能够访问 `localhost` 上的 vLLM 服务器端口。除非使用标志覆盖，否则 API 调用中使用的模型名称将为 `finetuned_model`。

请参阅[官方文档](https://docs.vllm.ai/en/latest/serving/openai_compatible_server.html#command-line-arguments-for-the-server)了解 `vllm serve` 命令可用的所有标志，因为它们可能会影响预测延迟。

## 将微调模型部署到生产环境 {#deploy-the-fine-tuned-model-to-a-production-environment}

### VM 实例 {#vm-instance}

如果你想在 VM 实例上运行微调模型以供生产对话机器人使用，建议你使用[官方 vLLM docker 映像](https://docs.vllm.ai/en/latest/serving/deploying_with_docker.html)。推荐的 `vllm` 版本为 `0.6.0`。

假设：

- 已在具有 [NVIDIA GPU](https://docs.docker.com/desktop/gpu/) 的 VM 实例中安装了 [Docker](https://docs.docker.com/engine/install/) 和 [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)。
- 实例上名为 `finetuned_model` 的目录中提供了微调模型文件。你可以使用下面的 `docker` 命令在该实例上启动 vLLM 服务器。请确保首先更新 `--served-model-name` 标志的值，该标志将覆盖 API 调用中使用的模型名称。

    ```shell
    docker run --runtime nvidia --gpus all \
        -v "./finetuned_model:/mnt/models" \
        -p 8000:8000 --ipc="host" \
        vllm/vllm-openai:latest \
        --model "/mnt/models"  \
        --served-model-name "CHANGEME"
    ```

然后，你需要将 vLLM 服务器端口公开到外部，以便对话机器人可以通过公共 IP 地址访问它。

### Kubernetes 集群 {#kubernetes-cluster}

由于[集成了 vLLM 运行时](https://kserve.github.io/website/latest/modelserving/v1beta1/llm/huggingface/)，建议你在将经过微调的模型部署到 [Kubernetes](https://kubernetes.io/) 集群以供生产使用时使用 [KServe](https://github.com/kserve/kserve) 模型推理平台。

还建议你将模型文件放在云对象存储中，因为 KServe 可以[自动从存储桶下载模型](https://kserve.github.io/website/latest/modelserving/storage/storagecontainers/)。你必须首先使用云存储提供商的凭据配置 KServe，例如使用 [Amazon S3](https://kserve.github.io/website/0.10/modelserving/storage/s3/s3/) 或 [Google Cloud Storage](https://kserve.github.io/website/latest/modelserving/storage/gcs/gcs/)。

假设你的集群已经[安装了 KServe](https://kserve.github.io/website/latest/admin/serverless/serverless/)，并且至少有一个[带有适当驱动程序的 NVIDIA GPU 节点](https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/)，你可以使用以下清单从存储桶部署经过微调的模型。首先，确保更新以下值：

- `metadata.name` 字段，其中包含模型推理服务的唯一名称。
- `STORAGE_URI` 环境变量，其中包含模型的云存储 URI。
- `--served-model-name` 标志，其中包含调用 OpenAI API 时要使用的模型名称。

```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: "CHANGEME"
spec:
  predictor:
    containers:
    - name: "main"
      image: "kserve/vllmserver:latest"
      command:
      - "python3"
      - "-m"
      - "vllm.entrypoints.openai.api_server"
      args:
      - "--port"
      - "8000"
      - "--model"
      - "/mnt/models"
      - "--served-model-name"
      - "CHANGEME"
      env:
      - name: "STORAGE_URI"
        value: "CHANGEME"
      resources:
        limits:
          nvidia.com/gpu: "1"
```

如果你的 CALM 对话机器人与微调模型部署在同一个集群中，你可以利用 [Kubernetes DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) 并在对话机器人配置中使用推理服务的内部 URI。否则，你必须设置自己的[入口](https://kubernetes.io/docs/concepts/services-networking/ingress/)并使用服务的外部 IP。
