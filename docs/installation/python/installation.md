# 本地开发安装

Rasa Pro 包含一个 Python 包：

- `rasa-pro`（适用于 `>=3.8.x` 版本）
- `rasa-plus`（适用于 `<=3.7.x` 版本）

你可以使用 `pip` 或 `poetry` 在本地安装。

## 使用 `pip` 安装 {#installing-with-pip}

=== "Rasa Pro >= 3.8.x"

    使用以下命令安装 `rasa-pro`：

    ```shell
    pip install rasa-pro --extra-index-url=https://europe-west3-python.pkg.dev/rasa-releases/rasa-pro-python/simple/
    ```

    可以使用 uv 加快安装过程：

    ```shell
    pip install uv
    uv pip install rasa-pro --extra-index-url=https://europe-west3-python.pkg.dev/rasa-releases/rasa-pro-python/simple/
    ```

=== "Rasa Pro <= 3.7.x"

    使用以下命令安装 `rasa-plus`：

    ```shell
    pip install rasa-plus --extra-index-url=https://europe-west3-python.pkg.dev/rasa-releases/rasa-plus-py/simple/
    ```

    可以使用 uv 加快安装过程：

    ```shell
    pip install uv
    uv pip install rasa-plus --extra-index-url=https://europe-west3-python.pkg.dev/rasa-releases/rasa-plus-py/simple/
    ```

## 使用 `poetry` 安装 {#installing-with-poetry}

!!! note "注意"

    根据 [Poetry 文档](https://python-poetry.org/docs/)，Poetry 应始终安装在专用虚拟环境中，以将其与系统的其余部分隔离。在任何情况下都不应将其安装在要由 Poetry 管理的项目环境中。这可以确保 Poetry 自身的依赖项不会被意外升级或卸载。

### Rasa Pro >= 3.8.1 {#rasa-381}

Rasa Pro 版本 `>=3.8.1` 需要 `poetry` 版本 `1.8.2`。如果使用的是旧版本的 `poetry`，则需要升级到 `1.8.2`。

通过将以下部分添加到 `pyproject.toml`，将 Artifact Registry URL 与 `rasa-pro` 关联：

```toml title="pyproject.toml"
[[tool.poetry.source]]
name = "rasa-pro"
url = "https://europe-west3-python.pkg.dev/rasa-releases/rasa-pro-python/simple"
priority = "supplemental"
```

现在可以使用以下命令进行安装：

```bash
poetry install
```

### Rasa Pro == 3.8.0 {#rasa-380}

=== "Poetry 1.4"

    首先需要通过将此部分添加到 `pyproject.toml` 将 Artifact Registry URL 与 `rasa-pro` 关联起来：

    ```toml title="pyproject.toml"
    [[tool.poetry.source]]
    name = "rasa-pro"
    url = "https://europe-west3-python.pkg.dev/rasa-releases/rasa-pro-python/simple"
    default = false
    secondary = true
    ```

    现在可以使用以下命令进行安装：

    ```bash
    poetry install
    ```

=== "Poetry 1.6"

    首先需要通过将此部分添加到 `pyproject.toml` 将 Artifact Registry URL 与 `rasa-pro` 关联起来：

    ```toml title="pyproject.toml"
    [[tool.poetry.source]]
    name = "rasa-pro"
    url = "https://europe-west3-python.pkg.dev/rasa-releases/rasa-pro-python/simple"
    priority = "supplemental"
    ```

    现在可以使用以下命令进行安装：

    ```bash
    poetry install
    ```

### Rasa Pro >= 3.7.10, < 3.8.0 {#rasa-pro-3710-380}

Rasa Pro 版本 `>= 3.7.10, < 3.8.0` 需要 `poetry` 版本 `1.8.2`。如果你使用的是较旧版本的 `poetry`，则需要升级到 `1.8.2`。

通过将以下部分添加到 `pyproject.toml`，将 Artifact Registry URL 与 `rasa-pro` 关联：

```toml title="pyproject.toml"
[[tool.poetry.source]]
name = "rasa-pro"
url = "https://europe-west3-python.pkg.dev/rasa-releases/rasa-pro-python/simple"
priority = "supplemental"
```

现在可以使用以下命令进行安装：

```bash
poetry install
```

### Rasa Pro <= 3.7.9 {#rasa-pro-379}

=== "Poetry 1.4"

    首先需要通过将此部分添加到 `pyproject.toml` 将 Artifact Registry URL 与 `rasa-pro` 关联起来：

    ```toml title="pyproject.toml"
    [[tool.poetry.source]]
    name = "rasa-plus"
    url = "https://europe-west3-python.pkg.dev/rasa-releases/rasa-pro-py/simple"
    default = false
    secondary = true
    ```

    现在可以使用以下命令进行安装：

    ```bash
    poetry install
    ```

=== "Poetry 1.6"

    首先需要通过将此部分添加到 `pyproject.toml` 将 Artifact Registry URL 与 `rasa-pro` 关联起来：

    ```toml title="pyproject.toml"
    [[tool.poetry.source]]
    name = "rasa-plus"
    url = "https://europe-west3-python.pkg.dev/rasa-releases/rasa-plus-py/simple"
    priority = "supplemental"
    ```

    现在可以使用以下命令进行安装：

    ```bash
    poetry install
    ```
