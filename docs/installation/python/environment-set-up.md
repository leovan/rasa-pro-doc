# Python 环境

# Python {#python}

检查 Python 环境是否已配置：

```bash
python --version
pip --version
```

!!! info "Python 3 版本支持"

    目前 Rasa 支持 Python 3.8、3.9 和 3.10。请注意，仅 3.4.x 或更高版本支持 Python 3.10。此外，苹果芯片机器的 Python 3.10 环境仅支持 3.5.x 或更高版本。

如果这些已经安装，命令应该显示相关版本号，则可以直接跳到下一步。

否则，请按照如下说明进行安装。

=== "Ubuntu"

    使用 `apt` 获取相关包，并使用 `pip` 安装 virtualenv。

    ```shell
    sudo apt update
    sudo apt install python3-dev python3-pip
    ```

=== "macOS"

    如果未安装 [Homebrew](https://brew.sh/) 包管理器请先安装。

    完成后即可安装 Python 3。

    ```shell
    brew update
    brew install python
    ```

=== "Windows"

    确保安装了 Microsoft VC++ 编译器，以便 Python 可以编译相关依赖项。你可以从 [Visual Studio](https://visualstudio.microsoft.com/visual-cpp-build-tools/) 获取编译器。下载安装程序并在列表中选择 VC++ 构建工具。安装适用于 Windows 的 64 位版本 [Python 3](https://www.python.org/downloads/windows/https://www.python.org/downloads/windows/)。

    ```powershell
    C:\> pip3 install -U pip
    ```

## 虚拟环境配置 {#virtual-environment-setup}

我们强烈建议使用虚拟环境隔离 Python 项目。Python 自带 [venv](https://docs.python.org/3/library/venv.html) 模块，其用法如下：

=== "Ubuntu"

    选择 Python 解释器并创建一个 `./venv` 目录来保存并创建一个新的虚拟环境：

    ```shell
    python -m venv ./venv
    ```

    激活虚拟环境：

    ```shell
    source ./venv/bin/activate
    ```

=== "macOS"

    选择 Python 解释器并创建一个 `./venv` 目录来保存并创建一个新的虚拟环境：

    ```shell
    python -m venv ./venv
    ```

    激活虚拟环境：

    ```shell
    source ./venv/bin/activate
    ```

=== "Windows"

    选择 Python 解释器并创建一个 `.\\venv` 目录来保存并创建一个新的虚拟环境：

    ```powershell
    C:\> python -m venv ./venv
    ```

    激活虚拟环境：

    ```powershell
    C:\> .\venv\Scripts\activate
    ```

`venv` 的替代品有 [virtualenv](https://virtualenv.pypa.io/en/latest/)、[virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/) 和 [Miniconda](https://docs.conda.io/projects/miniconda/en/latest/)。
