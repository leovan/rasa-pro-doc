# Sanic 扩展

!!! info "3.6 版本新特性"

    你现在可以扩展 Sanic 功能，例如中间件、监听器、后台任务和附加路由。

现在，你可以通过访问动作服务器创建的应用程序对象来创建其他 Sanic 扩展。插件包中实现的钩子使你可以在启动动作服务器时访问由 `rasa-sdk` 创建的 Sanic 应用程序对象。

## 在 rasa_sdk 中创建自己的 Sanic 扩展的分步指南 {#step-by-step-guide-on-creating-your-own-sanic-extension-in-rasa_sdk}

此示例将向你展示如何使用插件创建 Sanic 侦听器。

### 创建 rasa_sdk_plugins 包 {#create-the-rasa_sdk_plugins-package}

在你的动作服务器项目中创建一个包，必须将其命名为 `rasa_sdk_plugins`。Rasa SDK 将尝试在你的项目中实例化此包以启动插件。如果未找到插件，它将打印一条调试日志，表明你的项目中没有插件。

### 注册包含钩子的模块 {#register-modules-containing-the-hooks}

创建包 `rasa_sdk_plugins` 并通过创建 `__init__.py` 文件来初始化钩子，插件管理器将在其中查找实现钩子的模块：

```python
def init_hooks(manager: pluggy.PluginManager) -> None:
    """Initialise hooks into rasa sdk."""
    import sys
    import rasa_sdk_plugins.your_module

    logger.info("Finding hooks")
    manager.register(sys.modules["rasa_sdk_plugins.your_module"])
```

### 实现你的钩子 {#implement-your-hook}

实现钩子 `attach_sanic_app_extensions`。此钩子转发由 Sanic 在 `rasa_sdk` 中创建的应用程序对象，并允许你创建其他路由、中间件、侦听器和后台任务。以下是创建侦听器的此实现的示例。

在你的 `rasa_sdk_plugins.your_module.py` 中：

```python
from __future__ import annotations

import logging
import pluggy

from asyncio import AbstractEventLoop
from functools import partial


logger = logging.getLogger(__name__)
hookimpl = pluggy.HookimplMarker("rasa_sdk")


@hookimpl  # type: ignore[misc]
def attach_sanic_app_extensions(app: Sanic) -> None:
    logger.info("hook called")
    app.register_listener(
        partial(before_server_start),
        "before_server_start",
    )


async def before_server_start(app: Sanic, loop: AbstractEventLoop):
    logger.info("BEFORE SERVER START")
```
