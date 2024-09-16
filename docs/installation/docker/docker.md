# 使用 Docker

Rasa Pro Docker 映像名为：

- `rasa-pro`（适用于 `>=3.8.x` 版本）
- `rasa-plus`（适用于 `<=3.7.x` 版本）

Docker 映像托管在我们的 GCP Artifact Registry 上。要提取映像，你可以运行以下命令：

=== "Rasa Pro >= 3.8.x"

    !!! note "注意"

        在此命令中，将 `RASAVERSION` 替换为所需的 Rasa Pro 版本。要查找 Rasa Pro 的最新版本，请参阅[变更日志](../../rasa-pro-changelog.md)。

    ```bash
    docker pull europe-west3-docker.pkg.devrasa-releases/rasa-pro/rasa-pro:RASAVERSION
    ```

=== "Rasa Pro <=3.7.x"

    ```bash
    docker pull europe-west3-docker.pkg.dev/rasa-releases/rasa-plus/rasa-plus
    ```

## 许可 {#licensing}

如[许可](../python/licensing.md)中所述，在运行时，必须在环境变量 `RASA_PRO_LICENSE` 中提供许可证密钥。

在系统上定义环境变量后，可以通过 `-e` 参数将其传递到 docker 容器中，例如：

=== "Rasa Pro >= 3.8.x"

    !!! note "注意"

        在此命令中，将 `RASAVERSION` 替换为所需的 Rasa Pro 版本。要查找 Rasa Pro 的最新版本，请参阅[变更日志](../../rasa-pro-changelog.md)。

    ```bash
    # execute `rasa init` via docker
    docker run -v ./:/app \
                -e RASA_PRO_LICENSE=${RASA_PRO_LICENSE} \
                europe-west3-docker.pkg.dev/rasa-releases/rasa-pro/rasa-pro:RASAVERSION \
                init --no-prompt
    ```

=== "Rasa Pro <=3.7.x"

    ```bash
    # execute `rasa init` via docker
    docker run -v ./:/app \
                -e RASA_PRO_LICENSE=${RASA_PRO_LICENSE} \
                europe-west3-docker.pkg.dev/rasa-releases/rasa-plus/rasa-plus \
                init --no-prompt
    ```
