# 部署动作服务器

本页介绍如何构建动作服务器映像并使用 Rasa Pro Helm Chart 部署 Rasa 动作服务器。

## 构建动作服务器镜像 {#building-an-action-server-image}

如果你构建包含动作代码的镜像并将其存储在容器注册表中，则可以将其作为部署的一部分运行，而无需在服务器之间移动代码。此外，你可以添加任何额外的系统或 Python 库依赖项，这些依赖项是动作代码的一部分，但不包含在基本 `rasa/rasa-sdk` 镜像中。

### 手动构建动作服务器 {#manually-building-an-action-server}

要创建镜像：

1. 确保动作在 `action/actions.py` 中定义。`rasa/rasa-sdk` 镜像将自动查找此文件中的动作。
2. 如果动作有任何额外的依赖项，请在文件 `action/requirements-actions.txt` 中创建它们的列表。
3. 在项目目录中创建一个名为 `Dockerfile` 的文件，将在其中扩展官方 SDK 镜像、复制代码并添加任何自定义依赖项（如有必要）。例如：

    ```dockerfile
    # Extend the official Rasa SDK image
    FROM rasa/rasa-sdk:latest

    # Use subdirectory as working directory
    WORKDIR /app

    # Copy any additional custom requirements, if necessary (uncomment next line)
    # COPY actions/requirements-actions.txt ./

    # Change back to root user to install dependencies
    USER root

    # Install extra requirements for actions code, if necessary (uncomment next line)
    # RUN pip install -r requirements-actions.txt

    # Copy actions folder to working directory
    COPY ./actions /app/actions

    # By best practices, don't run the code with root user
    USER 1001
    ```

然后可以通过以下命令构建镜像：

```shell
docker build . -t <account_username>/<repository_name>:<custom_image_tag>
```

`<custom_image_tag>` 应指明此映像与其他映像的不同之处。例如，你可以对标签进行版本控制或日期标注，以及为生产和开发服务器创建具有不同代码的不同标签。每次更新代码并想要重新部署时，你都应创建一个新标签。

### 使用自定义动作服务器映像 {#using-your-custom-action-server-image}

如果你要构建此映像以使其可从其他服务器使用，则应将映像推送到云存储库。

本文档假设你将映像推送到 [DockerHub](https://hub.docker.com/)。DockerHub 允许你免费托管多个公共存储库和一个私有存储库。请务必先[创建一个帐户](https://hub.docker.com/signup/)并[创建一个存储库](https://hub.docker.com/signup/)来存储你的映像。你还可以将映像推送到不同的 Docker 注册表，例如 [Google Container Registry](https://cloud.google.com/container-registry)、[Amazon Elastic Container Registry](https://aws.amazon.com/ecr/) 或 [Azure Container Registry](https://azure.microsoft.com/en-us/services/container-registry/)。

你可以通过以下方式将映像推送到 DockerHub：

```shell
docker login --username <account_username> --password <account_password>
docker push <account_username>/<repository_name>:<custom_image_tag>
```

要验证并将映像推送到不同的容器注册表，请参阅所选容器注册表的文档。

## 使用 Rasa Pro Helm Chart 部署动作服务器 {#deploy-an-action-server-with-rasa-pro-helm-chart}

要运行自定义动作服务器，你需要为动作服务器准备一个 Docker 映像。请参阅[构建动作服务器映像](#building-an-action-server-image)了解更多信息。

### 更新 values.yml {#update-valuesyml}

用要使用的映像和标签更新 `values.yml` 文件：

```yaml
rasa:
  endpoints:
    actionEndpoint:
        url: http://action-server:5055/webhook

actionServer:
  enabled: true
  image:
    repository: <account_username>/<repository_name>
    tag: <custom_image_tag>
```

### 使用 Helm 部署 {#deploy-with-helm}

要更新 Helm 部署，可以使用：

```yaml
helm upgrade \
    --namespace <your namespace> \
    --values values.yml \
    <release name> \
    rasa-<version>.tgz
```

## 环境变量 {#environment-variables}

Rasa 动作服务器的所有可用环境变量均可在[此处](environment-variables.md#rasa-sdk)找到。
