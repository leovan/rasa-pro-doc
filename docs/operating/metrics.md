# 指标

!!! info "3.8 版本新特性"

    你现在可以使用基于 OpenTelemetry 的指标来衡量 CALM 对话机器人的性能。

指标是在运行时捕获的服务测量值，可作为可用性和性能的指标。指标可用于监控服务的运行状况、发出中断警报以及了解服务更改的影响。与[追踪](tracing.md)不同，指标旨在提供跨多条消息和对话的聚合统计信息，例如平均响应时间或吞吐量。

## 配置指标 {#configuring-metrics}

要在 Rasa Pro 中启用指标收集，你必须使用 [OTEL 收集器（OpenTelemetry Collector）](https://opentelemetry.io/docs/collector/)收集指标，然后将其发送到你选择的后端。

要配置指标 OTEL 收集器，请将 `metrics` 条目添加到你的端点，即在 `endpoints.yml` 文件中，或在部署中 Helm 值的相关部分。

要配置 OTEL 收集器，请将 `type` 指定为 `otlp`。

```yaml
metrics:
  type: otlp
  endpoint: my-otlp-host:4318
  insecure: false
  service_name: rasa
  root_certificates: ./tests/unit/tracing/fixtures/ca.pem
```

请注意，指标必须与[追踪](tracing.md)一起使用才能提供系统的完整视图。

## 记录的指标 {#recorded-metrics}

支持的自定义指标包括：

- 进行 LLM 调用时基于 LLM 的命令生成器 (`LLMCommandGenerator`、`SingleStepLLMCommandGenerator` 和 `MultiStepLLMCommandGenerator`) 的 CPU 和内存使用情况。
- 基于 LLM 的命令生成器的提示令牌使用情况（前提是启用了 [`trace_prompt_tokens` 配置属性](tracing.md#tracing-prompt-token-usage)）。
- 组件中 LLM 特定调用方法的调用持续时间测量，例如 `IntentlessPolicy`、`EnterpriseSearchPolicy`、`ContextualResponseRephraser`、`LLMCommandGenerator`、`SingleStepLLMCommandGenerator`、`MultiStepLLMCommandGenerator`。
- rasa 客户端 http 请求持续时间（例如到动作服务器或 NLG 服务器）。
- rasa 客户端 http 请求大小（以字节为单位）。

!!! info "弃用警告"

    之前的 `LLMCommandGenerator` 在版本 `3.9.0` 中已重命名为 `SingleStepLLMCommandGenerator`，同时保留了其功能。Rasa `4.0.0` 中将不再支持以前的名称 `LLMCommandGenerator`。请修改你的对话机器人的配置以改用 `SingleStepLLMCommandGenerator`。
