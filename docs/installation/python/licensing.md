# 许可

Rasa Pro 将在环境变量 `RASA_PRO_LICENSE` 中查找许可证，该变量必须包含 rasa 提供的许可证密钥文件的内容。

!!! note "开发者版本"

    如果你想开始使用 [CALM](../../calm.md) 构建对话机器人，你可以获取 Rasa Pro 开发者版本 [^rasa-pro-developer-edition]。

    [在此处获取](https://rasa.com/rasa-pro-developer-edition-license-key-request/){ .md-button .md-button--primary }

你可以在终端中临时设置 `RASA_PRO_LICENSE` 环境变量，但建议永久设置它，这样就不必在每次运行 Rasa Pro 时都设置它。

Bash：

```bash
## Temporary
export RASA_PRO_LICENSE=<your-license-string>

## Persistent
echo "export RASA_PRO_LICENSE=<your-license-string>" >> ~/.bashrc
## If you're using a different flavor of bash e.g. Zsh, replace .bashrc with your shell's initialization script e.g. ~/.zshrc
```

Windows Powershell：

```powershell
## Temporary
$env: RASA_PRO_LICENSE=<your-license-string>

## Persistent for the current user
[System.Environment]::SetEnvironmentVariable('RASA_PRO_LICENSE','<your-license-string>','USER')
```

然后可以使用 [`rasa` CLI](../../command-line-interface.md)，例如：

```bash
rasa init
```

[^rasa-pro-developer-edition]: **Rasa Pro 开发者版本** 旨在帮助开发者使用 CALM 构建对话机器人。此许可证允许在笔记本电脑、台式机或集成开发环境中本地运行 Rasa Pro（带 CALM）。详细条款可在[此处](https://rasa.com/eula/)找到。
