# 密钥管理

你可以将对话机器人的密钥存储在外部凭证管理器中。Rasa Pro 目前支持凭证管理器用于追踪器存储。

!!! info "Rasa Pro 3.5.0 版本后可用"

Rasa Pro 支持以下密钥管理器：

- [HashiCorp Vault](https://www.hashicorp.com/products/vault)

目前，Rasa Pro 支持保护以下服务的凭证：

- [追踪器存储](tracker-stores.md)

## HashiCorp Vault 密钥管理 {#hashicorp-vault-secrets-manager}

使用 Vault Secrets Manager 存储用于验证对外部服务的访问的凭据。凭据存储在 Vault 实例中，可以静态加密。要将凭据存储在 Vault 实例中，你可以阅读官方 Vault 文档将[密钥存储在 Vault 中](https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-first-secret)。

你还可以使用 [Vault Transit Engine](https://developer.hashicorp.com/vault/docs/secrets/transit) 静态加密凭据。

!!! note "注意"

    过期的令牌需要定期更新，更新过程通过网络完成，Rasa Pro 将在令牌过期前 15 秒尝试更新令牌。如果令牌的生存时间 (TTL) 小于 15 秒，我们将尝试在 1 秒后更新令牌，但可能会因网络延迟而失败。

    Rasa Pro 具有用于更新令牌的内置重试机制。

    如果令牌未成功更新，则将被视为已过期，Rasa Pro 将无法访问密钥。你需要创建一个新的可更新令牌并使用新令牌重新启动 Rasa Pro。

### 身份验证 {#authentication}

Rasa Pro 可以通过[令牌身份验证](https://developer.hashicorp.com/vault/docs/auth/token)向 Vault 进行身份验证。

支持 `expiring` 和 `non-expiring`（所谓的根令牌）令牌。如果令牌即将过期，Rasa Pro 将自动更新令牌。

### 如何配置对 Vault 的访问 {#how-to-configure-access-to-vault}

可以使用环境变量和通过 `endpoints.yml` 配置文件配置对 Vault 密钥管理器的访问。

环境变量和 `endpoints.yml` 配置文件合并在一起，环境变量的值优先。

!!! info "3.7 版本新特性"

    Vault 命名空间可用于隔离密钥。你可以使用 `VAULT_NAMESPACE` 环境变量或 `endpoints.yml` 文件的 `secrets_manager` 部分中的 `namespace` 键配置命名空间。要了解有关命名空间的更多信息，请查看 [Vault 命名空间文档](https://developer.hashicorp.com/vault/docs/enterprise/namespaces)。

以下环境变量可用：

| 环境变量                    | 描述                                                 | 默认值         |
| :-------------------------- | :--------------------------------------------------- | :------------- |
| `SECRET_MANAGER`            | **必需**。要使用的密钥管理器。目前仅支持 `vault`     | `vault`        |
| `VAULT_HOST`                | **必需**。Vault 服务器的地址                         |                |
| `VAULT_TOKEN`               | **必需**。用于向 Vault 服务器进行身份验证的令牌      |                |
| `VAULT_RASA_SECRETS_PATH`   | Vault 服务器中密钥的路径                             | `rasa-secrets` |
| `VAULT_TRANSIT_MOUNT_POINT` | 如果启用了传输密钥引擎，则将其设置为传输引擎的挂载点 |                |
| `VAULT_NAMESPACE`           | 如果使用命名空间，则将其设置为命名空间的路径         |                |

要配置 Vault 密钥管理器，你可以在 `endpoints.yml` 文件中填写以下部分：

```yaml
secrets_manager:
    type: vault # required - the secrets manager to use
    token: <token> # required - token to authenticate to the vault server
    url: "http://localhost:1234" # required - the address of the vault server
    secrets_path: rasa-secrets  # path to the secrets in the vault server if not set it defaults to `rasa-secrets`
    transit_mount_point: transit # if transit secrets engine is enabled, set this to mount point of the transit engine
    namespace: my-namespace # if namespaces are used, set this to the path of the namespace
```

#### 将访问凭据存储在环境变量中 {#store-access-credentials-in-environment-variables}

关于如何结合环境变量和 `endpoints.yml` 配置文件的一个简单示例是将访问令牌存储在环境变量中，将其余配置存储在 `endpoints.yml` 文件中。

```shell
# environment variables
VAULT_TOKEN=<token used to authenticate to Vault>
```

```yaml
secrets_manager:
    type: vault
    url: "http://localhost:1234"
    secrets_path: rasa-secrets  # if not set it defaults to `rasa-secrets`
    transit_mount_point: transit # if you have enabled transit secrets engine, and you want to use it
    namespace: my-namespace # if namespaces are used, set this to the path of the namespace
```

### 如何使用 Vault Secrets Manager 配置追踪器存储 {#how-to-configure-tracker-store-with-vault-secrets-manager}

1. 配置 Rasa 以访问 Vault 实例。查看[如何配置对 Vault 的访问](#how-to-configure-access-to-vault)部分以了解更多详细信息。
2. 配置 Rasa 以使用 Vault 密钥管理获取追踪器存储的凭据。

    ```yaml
    tracker_store:
      type: SQL
      url: localhost:5432
      username:
        source: secrets_manager.vault
        secret_key: sql_store_username
      password:
        source: secrets_manager.vault
        secret_key: sql_store_password
    ```
