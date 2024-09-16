# 运行 Rasa SDK 动作服务器

使用 Python 构建的 Python 动作服务器可以使用 rasa 命令或直接作为 Python 模块运行。动作服务器支持两种协议来调用自定义动作：HTTP 和 gRPC。默认情况下，动作服务器在 HTTP 协议上运行。要在 gRPC 协议上运行动作服务器，你需要指定 `--grpc` 标志。

## 运行动作服务器 {#running-the-action-server}

### 通过 HTTP(S) 协议运行动作服务器 {#running-action-server-over-https-protocol}

动作服务器支持安全 (HTTPS) 和不安全的 HTTP 连接。

#### HTTP 协议 {#http-protocol}

要通过 HTTP 协议运行动作服务器，请使用以下命令：

=== "使用 Rasa 命令"

    ```shell
    rasa run actions
    ```

=== "直接作为 Python 模块"

    ```shell
    python -m rasa_sdk
    ```

通过 HTTP 协议运行动作服务器时，请确保 Rasa 也配置为[使用 HTTP 协议](actions.md#http-protocol)。

#### HTTPS 协议 {#https-protocol}

要通过 HTTPS 协议运行动作服务器，请使用以下命令：

=== "使用 Rasa 命令"

    ```shell
    rasa run actions --ssl-certificate /path/to/ssl_server_certificate --ssl-keyfile /path/to/ssl_server_key
    ```

=== "直接作为 Python 模块"

    ```shell
    python -m rasa_sdk --ssl-certificate /path/to/ssl_server_certificate --ssl-keyfile /path/to/ssl_server_key
    ```

通过 HTTPS 协议运行动作服务器时，请确保 Rasa 服务器也配置为[使用 HTTPS 协议](actions.md#https-protocol)。

#### 监听特定地址 {#listen-on-specific-address}

你可以使用 `SANIC_HOST` 环境变量让动作服务器监听特定地址：

=== "使用 Rasa 命令"

    ```shell
    SANIC_HOST=192.168.69.150 rasa run actions
    ```

=== "直接作为 Python 模块"

    ```shell
    SANIC_HOST=192.168.69.150 python -m rasa_sdk
    ```

### 在 gRPC 协议上运行动作服务器 {#running-the-action-server-on-grpc-protocol}

动作服务器支持安全和不安全的 gRPC 连接。默认情况下，动作服务器在不安全的 gRPC 连接上运行。

#### 不安全 gRPC 连接 {#insecure-grpc-connection}

要运行动作服务器以接受不安全的 gRPC 连接，请使用以下命令：

=== "使用 Rasa 命令"

    ```shell
    rasa run actions --grpc
    ```

=== "直接作为 Python 模块"

    ```shell
    python -m rasa_sdk --grpc
    ```

运行动作服务器以接受不安全的 gRPC 连接时，请确保 Rasa 服务器也配置为[使用不安全的 gRPC 连接](actions.md#insecure-grpc-connection)。

#### 安全 gRPC 连接 {#secure-grpc-connection}

要运行动作服务器以接受安全 gRPC 连接，你需要指定 `--grpc`，以及 `--ssl-certificate` 和 `--ssl-keyfile` 标志：

=== "使用 Rasa 命令"

    ```shell
    rasa run actions --grpc --ssl-certificate /path/to/ssl_server_certificate --ssl-keyfile /path/to/ssl_server_key
    ```

=== "直接作为 Python 模块"

    ```shell
    python -m rasa_sdk --grpc --ssl-certificate /path/to/ssl_server_certificate --ssl-keyfile /path/to/ssl_server_key
    ```

运行动作服务器以接受安全的 gRPC 连接时，请确保 Rasa 服务器也配置为[使用安全的 gRPC 连接](actions.md#secure-tls-grpc-connection)。

## 指定动作模块或包 {#specifying-the-actions-module-or-package}

默认情况下，动作服务器将在名为 `actions.py` 的文件或名为 `actions` 的包目录中查找你的动作。

你可以使用 `--actions` 标志指定不同的动作模块或包。

=== "使用 Rasa 命令"

    ```shell
    rasa run actions --actions my_actions
    ```

=== "直接作为 Python 模块"

    ```shell
    python -m rasa_sdk --actions my_actions
    ```

使用上述命令，动作服务器将期望在名为 `my_actions.py` 的文件或名为 `my_actions` 的包目录中找到你的动作。

## 其他选项 {#other-options}

要查看运行动作服务器的完整选项列表，请运行：

=== "使用 Rasa 命令"

    ```shell
    rasa run actions --help
    ```

=== "直接作为 Python 模块"

    ```shell
    python -m rasa_sdk --help
    ```
