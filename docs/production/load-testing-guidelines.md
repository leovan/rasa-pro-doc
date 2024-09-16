# 负载测试指南

## 概述 {#overview}

为了收集有关我们系统处理增加的负载和用户的能力的指标，我们进行了测试以评估 Rasa 对话机器人在某些机器配置下可以处理的最大并发用户数。在每个测试案例中，我们在峰值并发时以每秒 1000 个用户的[生成速率](https://docs.locust.io/en/1.5.0/configuration.html#all-available-configuration-options)生成以下数量的并发用户。在我们的测试中，我们使用了 Rasa [HTTP API](https://rasa.com/docs/rasa/pages/http-api) 和 [Locust](https://locust.io/) 开源负载测试工具。

| 用户        | CPU                       | 内存  |
| :---------- | :------------------------ | :---- |
| 最多 50,000 | 6vCPU                     | 16 GB |
| 最多 80,000 | 6vCPU，CPU 使用率接近 90% | 16 GB |

### 在扩展时调试与机器人相关的问题 {#debugging-bot-related-issues-while-scaling-up}

为了测试 Rasa [HTTP API](https://rasa.com/docs/rasa/pages/http-api) 处理大量并发用户活动的能力，我们使用了 Rasa Pro [追踪](../operating/tracing.md)功能以及追踪后端或收集器（例如 Jaeger）来收集被测对话机器人的追踪信息。

!!! note "注意"

    我们的团队目前正在进行额外的性能相关测试。随着测试的进展，我们将在此处添加更多信息。
