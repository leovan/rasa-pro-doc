# 模型存储桶

本页介绍如何使用模型存储桶将模型加载到在 Kubernetes/Openshift 上运行的 Rasa Pro 容器中。

## 先决条件：模型存储桶 {#prerequisite-model-storage-bucket}

在云中，你首先必须设置一个模型存储桶：

| 类型     | AWS       | Azure              | Google               |
| :------- | :-------- | :----------------- | :------------------- |
| 模型存储 | Amazon S3 | Azure Blob Storage | Google Cloud Storage |

## 上传训练过的模型 {#upload-your-trained-model}

建议你使用 [CI/CD 管道](../production/setting-up-ci-cd.md)来训练、测试 Rasa Pro 模型并将其上传到此模型存储桶。

### 配置 {#configuration}

Rasa 需要能够访问模型存储桶，这在每个云平台上的工作方式不同，并且每个平台都有多个选项。我们在这里描述了在 Google Cloud Platform 上配置它的方法之一。

=== "Google"

    Google Cloud 中推荐的方法是使用 [Cloud Storage FUSE CSI 驱动程序](https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/cloud-storage-fuse-csi-driver)。

    这样你就可以将 Cloud Storage 存储桶挂载为文件系统，避免使用特定于云的 API。

    - 按照[启用 Cloud Storage FUSE CSI 驱动程序](https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/cloud-storage-fuse-csi-driver#enable)的步骤操作。
    - 按照[创建 IAM 服务帐户密钥](https://cloud.google.com/iam/docs/keys-create-delete#creating)的步骤操作，并确保服务帐户对你的 Cloud Storage 存储桶具有读/写/创建访问权限。
    - 为 rasa-pro-sa Kubernetes 服务帐户设置 IAM 策略绑定。

        ```shell
        gcloud iam service-accounts add-iam-policy-binding \
            <YOUR SERVICE ACCOUNT EMAIL> \
            --role roles/iam.workloadIdentityUser \
            --member "serviceAccount:<YOUR PROJECT_ID>.svc.id.goog[<YOUR NAMESPACE>/rasa-pro-sa]"
        ```

        此绑定将允许 Kubernetes 服务帐户充当 IAM 服务帐户。

        注意：你不需要先拥有 rasa-pro-sa Kubernetes 服务帐户。它将在你使用下一步中的值部署时创建。

    - 更新 `values.yml` 文件以将模型存储桶挂载到 Rasa：

        ```yaml
        rasa:
            endpoints:
                models:
                enabled: false
            
            volumes:
                - csi:
                    driver: gcsfuse.csi.storage.gke.io
                    readOnly: true
                    volumeAttributes:
                    bucketName: <YOUR BUCKET NAME>
                    mountOptions: implicit-dirs,only-dir=<YOUR DIR IN YOUR BUCKET>
                name: rasa-models

            volumeMounts:
                - name: rasa-models
                mountPath: /app/models
                readOnly: true

            serviceAccount:
                # serviceAccount.create specifies whether a service account should be created.
                # set to false if you have already created the kubernetes service account
                create: true
                annotations:
                iam.gke.io/gcp-service-account: <YOUR SERVICE ACCOUNT EMAIL>
                name: "rasa-pro-sa"

            podAnnotations:
                gke-gcsfuse/volumes: "true"
        ```

## 部署 {#deploy}

要更新 helm 部署，可以使用：

```shell
helm upgrade \
    --namespace <your namespace> \
    --values values.yml \
    <release name> \
    rasa-<version>.tgz
```

当 Rasa Pro Pod 重新启动时，它会从模型存储中加载模型。
