# 使用断言进行端到端测试

Rasa Pro 提供了一种使用断言端到端测试对话机器人的方法。

!!! danger "3.10 版本测试新特性"

    为了在将 Rasa Pro 对话机器人部署到生产环境之前对 [CALM](../../calm.md) 系统充满信心，Rasa Pro 3.10 引入了一种新方法，即在用户每次操作时使用多个断言对对话机器人进行端到端测试。此功能仅为测试版（实验性），可能会在未来的 Rasa Pro 版本中发生变化。

## 概述 {#overview}

端到端测试是任何对话式 AI 对话机器人开发过程中的关键部分。Rasa Pro 提供了一种使用断言端到端测试对话机器人的方法，以此来验证对话机器人在对话的每个步骤中的行为。

你可以使用的断言包括：

- [流已启动断言](assertions-fundamentals.md#flow-started-assertion)：检查具有所提供 ID 的流是否已启动。
- [流已完成断言](assertions-fundamentals.md#flow-completed-assertion)：检查具有所提供 ID 的流是否已完成。
- [流已取消断言](assertions-fundamentals.md#flow-cancelled-assertion)：检查具有所提供 ID 的流是否已取消。
- [模式澄清断言](assertions-fundamentals.md#pattern-clarification-contains-assertion)：检查是否使用提供的流名称触发了模式澄清。
- [槽已设置断言](assertions-fundamentals.md#slot-was-set-assertion)：检查具有所提供名称的槽是否已用所提供值填充。
- [槽未设置断言](assertions-fundamentals.md#slot-was-not-set-assertion)：检查具有所提供名称的槽是否未填充。
- [动作已执行断言](assertions-fundamentals.md#action-executed-assertion)：检查是否执行了具有所提供名称的动作。
- [对话机器人发出的断言](assertions-fundamentals.md#bot-uttered-assertion)：检查对话机器人的话语是否与提供的模式、按钮和/或领域响应名称匹配。
- [生成响应相关断言](assertions-fundamentals.md#generative-response-is-relevant-assertion)：检查生成响应是否与提供的输入相关。
- [生成响应准确断言](assertions-fundamentals.md#generative-response-is-grounded-assertion)：检查生成响应相对于基本事实是否准确。

这些断言与 Rasa Pro 在使用 Rasa Pro 对话机器人运行测试用例时发出的真实[事件](../../concepts/events.md)进行核对。

要开始使用断言进行端到端测试，请转到[安装先决条件](assertions-installation.md)和[如何使用断言进行端到端测试](assertions-how-to-guide.md)指南。

新功能以测试版发布，我们很乐意听到你的反馈，特别是关于可用性和它为你的测试工作流程带来的价值。我们还有一份[问题](#beta-feedback-questions)清单，希望得到反馈。请通过客户支持团队与我们联系以分享你的反馈。

## 使用断言进行端到端测试的好处 {#benefits-of-end-to-end-testing-with-assertions}

- **增强信心**：使用断言进行端到端测试可帮助你对 CALM 的行为充满信心，因为它可以处理 NLU 理解、流逻辑改进和企业搜索。
- **更快的反馈循环**：你可以快速识别对话机器人中的问题并在开发过程的早期修复它们。
- **提高质量**：通过使用断言对对话机器人进行端到端测试，你可以确保对话机器人在不同场景中的行为符合预期。
- **减少手动测试**：使用断言进行端到端测试可帮助你减少手动测试的需求，让你专注于更复杂的场景。

## Beta 反馈问题 {#beta-feedback-questions}

我们希望获得反馈的问题包括：

- 你认为断言在你的测试工作流程中有多大价值？
- 你是否认为在 2 种独占模式下运行测试存在任何风险：断言或先前的测试用例格式？
- 断言功能如何帮助你识别对话机器人中的问题？
- 我们如何提高断言功能的可用性？语法是否清晰易用？
- 你在使用断言功能时面临哪些挑战？
- 你希望将来看到哪些其他断言？
- 你正在测试的最常见的对话机器人场景是什么，目前断言尚未涵盖这些场景？
- 断言命令是否有助于你的测试工作流程？
- 与使用初始 e2e 测试格式相比，当对话机器人流程实现发生变化时，你需要多久更新一次断言？
- 断言顺序可配置性对你的测试工作流程有用吗？
- 如果你在领域文件中定义具有初始值的插槽，你是否发现 `slot_was_not_set` 初始值假设为 `None` 是测试工作流程的一个限制？
- 当前的生成性反应评估解决方案是否足以决定反应是否“通过”测试？
