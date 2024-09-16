# 简介

## 部署选项 {#deployment-options}

### 开发者版 {#developer-edition}

如果你拥有 [Rasa Pro 开发者版](../developer-edition.md)的扩展许可证，则可以使用此 [docker compose 模板](https://github.com/RasaHQ/developer-edition-docker-compose)快速部署对话机器人。这通常需要不到 20 分钟的时间，包括配置虚拟机所需的时间。

### 可扩展部署 {#scalable-deployment}

如果你拥有商业（付费）Rasa 许可证，我们将帮助你设置对话机器人的可扩展集群部署。[完整的 Rasa Pro 部署](../production/arch-overview.md)包括以下服务：

- 运行多个 Rasa Pod 以应对高流量
- 分析
- 通过 S3 管理模型
- 密钥管理
- 连接到 Rasa Studio

继续阅读以了解有关使用 [kubernetes 进行可扩展部署](../production/cluster-requirements.md)的更多详细信息。
