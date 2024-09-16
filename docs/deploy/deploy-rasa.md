# 部署 Rasa Pro

本页介绍如何使用 Rasa Pro Helm Chart 将 Rasa Pro 部署到 Kubernetes 或 OpenShift 集群。

## 安装要求 {#installation-requirements}

!!! important "重要"

    如果使用的 Helm 版本 `<3.5`，请更新至版本 `>=3.5`。

1. 检查你是否已安装 Kubernetes 或 OpenShift 命令行界面（CLI）。你可以使用如下命令进行检查：

    === "Kubernets"

        ```shell
        kubectl version

        # The output should be similar to this
        # Client Version: v1.28.2
        ```

    === "OpenShift"

        ```shell
        oc version --client

        # The output should be similar to this
        # Client Version: 4.7.13
        ```

    如果命令报错，请根据你使用的集群安装 [Kubernets CLI](https://kubernetes.io/docs/tasks/tools/install-kubectl/) 或 [OpenShift CLI](https://docs.openshift.com/container-platform/4.7/cli_reference/openshift_cli/getting-started-cli.html#installing-openshift-cli)。

2. 确保 Kubernetes / OpenShift CLI 可以正确连接到你的集群。可以使用如下命令执行此操作：

    === "Kubernets"

        ```shell
        kubectl version

        # The output should be similar to this
        # Client Version: v1.28.2
        # Server Version: v1.27.4-gke.900
        ```

    === "OpenShift"

        ```shell
        oc version

        # The output should be similar to this
        # Client Version: 4.7.13
        # Kubernetes Version: v1.20.0+df9c838
        ```

    如果执行命令报错，则说明你未连接到集群。要获取连接到集群的命令，请咨询你的集群管理员或参见云服务提供者的文档。

3. 确保你已安装 [Helm CLI](https://helm.sh/docs/intro/install/)。要检查这一点，请运行：

    ```shell
    helm version --short

    # The output should be similar to this
    # v3.9.0+g7ceeda6
    ```

    如果命令报错，请安装 [Helm CLI](https://helm.sh/docs/intro/install/)。

## 命名空间和密钥 {#namespace--secrets}

### 创建命名空间 {#create-namespace}

我们建议将 Rasa 安装到单独的命名空间中，以避免干扰现有的集群部署。要创建新的[命名空间](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)，请运行如下命令：

=== "Kubernets"

    ```shell
    kubectl create namespace <your namespace>
    ```

    将其设为默认命名空间，以方便使用：

    ```shell
    kubectl config set-context --current --namespace=<your namespace>
    ```

=== "OpenShift"

    ```shell
    oc create namespace <your namespace>
    ```

### 运行时密钥 {#runtime-secrets}

准备一个名为 `secrets.yml` 的空文件，其中包含所有密钥值。

对于最基本的部署，请将以下内容添加到 `secrets.yml`，并将 `BASE64ENCODEDVALUE` 替换为实际密钥值（base64 编码）：

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: rasa-secrets
type: Opaque
data:
  # The Rasa Pro License provided to you by Rasa
  rasaProLicense: BASE64ENCODEDVALUE
  # A token Rasa accepts as authentication token from other Rasa services
  authToken: BASE64ENCODEDVALUE
  # A JWT token Rasa accepts as authentication token from other Rasa services
  jwtSecret: BASE64ENCODEDVALUE
```

!!! note "注意"

    要在 Linux 或 macOS 中对值进行 base64 编码，可以使用：

    ```shell
    echo -n <your-secret-value> | base64
    ```

使用以下命令将这些密钥应用到集群中的命名空间：

=== "Kubernets"

    ```shell
    kubectl apply -f secrets.yml
    ```

=== "OpenShift"

    ```shell
    oc apply -f secrets.yml
    ```

## 部署 {#deploy}

### 创建 values.yml {#create-valuesyml}

准备一个名为 `values.yml` 的空文件，该文件将包含使用 Helm 安装的所有自定义配置。

对于最基本的部署，请将以下内容添加到 `values.yml` 中：

=== "Rasa Pro >= 3.8.x"

    ```yaml
    # -- rasaProServices.enabled enables Rasa Pro Services deployment
    rasaProServices:
    enabled: false

    # -- rasa-pro image & configurations
    rasa:
    # rasa.image defines image settings
    image:
        repository: "europe-west3-docker.pkg.dev/rasa-releases/rasa-pro/rasa-pro"
        # -- image.tag specifies image tag
        tag: "3.8.0-latest"
    ```

=== "Rasa Pro <=3.7.x"

    ```yaml
    # -- rasaProServices.enabled enables Rasa Pro Services deployment
    rasaProServices:
    enabled: false

    # -- rasa-plus image & configurations
    rasa:
    # rasa.image defines image settings
    image:
        repository: "europe-west3-docker.pkg.dev/rasa-releases/rasa-plus/rasa-plus"
        # -- image.tag specifies image tag
        tag: "3.7.0-latest"
    ```

### 使用 Helm 部署 {#deploy-with-helm}

你需要先下载 [Rasa Pro Helm Chart](../production/rasa-pro-helm-chart.md)。

现在你可以使用以下方式部署 Rasa Pro 容器：

```shell
helm install \
    --namespace <your namespace> \
    --values values.yml \
    <release name> \
    rasa-<version>.tgz
```

这会将获得许可的 rasa-pro 容器部署到 kubernetes pod。

!!! note "注意"

    要更新 helm 部署，可以使用：

    ```shell
    helm upgrade \
        --namespace <your namespace> \
        --values values.yml \
        <release name> \
        rasa-<version>.tgz
    ```

!!! note "注意"

    要删除 helm 部署，可以使用：

    ```shell
    helm delete <release name>
    ```

## 环境变量 {#environment-variables}

有关可与 Rasa Pro 一起使用的环境变量的完整列表，请参阅[环境变量](environment-variables.md#rasa-pro)文档。
