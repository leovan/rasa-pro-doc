# 故障排除

## M1/M2（苹果芯片）限制 {#m1--m2-apple-silicon-limitations}

默认情况下，苹果芯片上的 Rasa 安装不使用 [Apple Metal](https://developer.apple.com/metal/)。我们发现在苹果芯片上使用 GPU 会显著增加 [DIETClassifier](../nlu-based-assistants/components.md#dietclassifier) 和 [TEDPolicy](../nlu-based-assistants/policies.md#ted-policy) 的训练时间。你可以安装可选依赖来自行测试或使用其他组件进行尝试：`pip3 install 'rasa[metal]'`。

目前，并非所有 Rasa 的依赖性都原生支持 Apple Silicon。这会导致：

- 你不能在 Apple Silicon 上将 Duckling 作为 Docker 容器运行。如果使用 [duckling 实体提取器](../nlu-based-assistants/components.md#ducklingentityextractor)，建议在云端部署 duckling。相关进展请参见 [Duckling 项目](https://github.com/facebook/duckling/issues/695)。
- Apple Silicon 上的 Rasa 不支持 [ConveRTFeaturizer 组件](../nlu-based-assistants/components.md#convertfeaturizer)或包含它的管道。该组件依赖 Apple Silicon 目前还不可用的 `tensorflow-text`。相关进展请参见 [Tensorflow Text 项目](https://github.com/tensorflow/text/issues/823)。

## Python 3.10 依赖 {#python-310-requirements}

如果使用的是 Linux，安装 `rasa-pro` 可能会在安装 `tokenizer` 和 `cryptography` 时失败。

为了解决这个问题，必须按照以下步骤安装 Rust 编译器：

```bash
apt install rustc && apt install cargo
```

初始化 Rust 编译器后，应该重新启动控制台并检查其安装：

```bash
rustc --version
```

如果 `PATH` 变量未自动设置，请运行：

```bash
export PATH="$HOME/.cargo/bin:$PATH"
```

如果你使用的是 macOS，安装 `rasa-pro` 可能会在安装 `tokenizer` 时失败（[此处](https://github.com/huggingface/tokenizers/issues/1050)详细描述了该问题）。

为了解决这个问题，必须按照以下步骤安装 Rust 编译器：

```bash
brew install rustup
rustup-init
```

初始化 Rust 编译器后，应该重新启动控制台并检查其安装：

```bash
rustc --version
```

如果 `PATH` 变量未自动设置，请运行：

```bash
export PATH="$HOME/.cargo/bin:$PATH"
```

## OSError: [Errno 40] Message too long {#oserror-errno-40-message-too-long}

如果在本地开发环境中使用 Jaeger 作为后端启用追踪后运行 `rasa train` 或 `rasa run`，可能会遇到 `OSError: [Errno 40] Message too long` 错误。

这可能是由于本地开发环境的操作系统限制了 UDP 数据包大小。可以在 macOS 上运行 `sysctl net.inet.udp.maxdgram` 查看当前的 UDP 数据包大小。可以通过运行 `sudo sysctl -w net.inet.udp.maxdgram=65535` 来增加 UDP 数据包大小。

## Rasa Pro 3.8.x Warnings {#rasa-pro-38x-warnings}

如果使用的是 Rasa Pro 3.8.x，则在运行某些 CLI 命令时可能会遇到一些与 `pydantic` 和 `pkg_resources` 相关的警告。这些警告是由于依赖项已被弃用或移动所致。

升级到 Python 3.9.x 或更高版本即可解决这些警告。升级 Rasa Pro 版本会提供更流畅的 Rasa 体验。
