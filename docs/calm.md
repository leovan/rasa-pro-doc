# 带语言模型的对话式 AI

带语言模型的对话式 AI (CALM) 是一种构建可靠对话式 AI 的 LLM 原生方法。它是在 Rasa 多年帮助企业团队构建面向客户的对话机器人的基础上开发的。CALM 旨在为你提供两全其美的解决方案：最新一代 LLM 的价值实现时间、通用性和流畅性，以及基于 NLU 的对话机器人的稳健性、可靠性、控制性和可调试性。

可以查看我们的 [YouTube 视频播放列表](https://www.youtube.com/playlist?list=PL75e0qA87dlHPWoD4c-NrYszndgq-NFz3)，以帮助你开始使用 Rasa 和 CALM。

如果你在研究中使用 CALM，请考虑引用[该论文](https://arxiv.org/abs/2402.12234)。

如果你熟悉在 Rasa 或其他平台中构建基于 NLU 的对话机器人，请转到 [CALM 与基于 NLU 的对话机器人的比较](#calm-compared-to-nlu-based-assistants)，以了解 CALM 的不同之处。

如果你熟悉构建 LLM 代理并想了解该方法与 CALM 的关系，请转到 [CALM 与 LLM 代理的比较](#calm-compared-to-llm-agents)。

## CALM 工作原理 {#how-calm-works}

CALM 方法有三个关键要素：业务逻辑、对话理解和自动对话修复。

[业务逻辑](concepts/flows.md)以一组流的形式实现。流描述了 AI 对话机器人可以处理的业务流程。它描述了需要从用户那里获取的信息、需要从 API 或数据库中检索的任何数据以及基于收集的信息的任何分支逻辑。流仅描述对话机器人要遵循的逻辑，它并不描述所有可能的对话路径。

[对话理解](concepts/dialogue-understanding.md)旨在解释终端用户与对话机器人交流的内容。此过程涉及生成反映用户意图的命令，与业务逻辑和正在进行的对话的上下文保持一致。有用于启动和停止流、填充槽等的命令。命令是 Rasa 用于导航对话的内部语法。请参阅[命令参考](concepts/dialogue-understanding.md#command-reference)中的完整列表。

[自动对话修复](concepts/conversation-repair.md)处理对话可能“偏离脚本”的所有情况。例如：

- 对话机器人要求提供电子邮件地址，但用户说了其他话。
- 终端用户中断当前流并将上下文切换到另一个主题。
- 终端用户改变了之前所说内容的想法。

这些情况以及其他情况均会自动处理，并且你可以完全自定义每个情况。

## CALM 与基于 NLU 的对话机器人的比较 {#calm-compared-to-nlu-based-assistants}

CALM 的一大转变是不再依赖“NLU”模型。在对话式 AI 中，NLU（自然语言理解）描述了处理用户消息并预测意图和实体以表示其含义。

CALM 使用一种称为对话理解 (DU) 的新方法，将用户所说的内容转化为对业务逻辑所表达的含义。这与传统的 NLU 方法在三个关键方面有所不同：

- NLU 单独解释一条消息，但 DU 会考虑更多的上下文：对话的来回和对话机器人的业务逻辑。
- DU 不会像 NLU 系统那样产生意图和实体，而是输出一系列命令，表示用户希望如何进行对话。
- NLU 系统仅限于固定的意图列表，而 DU 是生成式的，并根据内部语法生成一系列命令。这为我们提供了一种更丰富的语言来表示用户想要什么。

使用 CALM 比构建基于 NLU 的对话机器人要快得多。CALM 不依赖意图，这是[我们多年来在 Rasa 一直期待的举措](https://rasa.com/blog/its-about-time-we-get-rid-of-intents/)。CALM 还使用名为 [Flows](concepts/flows.md) 的原语来定义对话逻辑。与规则、表单和故事相比，Flows 的使用要简单得多，而且只需要涵盖预期路径。通过[本教程](tutorial.md)可以了解如何使用 CALM。

## CALM 与 LLM 代理的比较 {#calm-compared-to-llm-agents}

“LLM 代理”是指使用 LLM 作为推理引擎。请参阅 [Reasoning and Acting](https://arxiv.org/abs/2210.03629) (ReAct) 框架以获取示例。

CALM 使用 LLM 来确定用户希望如何进行对话。它不使用 LLM 来猜测完成该过程的正确步骤。这是两种方法之间的主要区别。

在 CALM 中，业务逻辑由 `Flow` 描述并使用 `FlowPolicy` 精确执行。

CALM 使用 LLM 来了解对话的用户端，而非系统端。在 ReAct 风格的代理中，LLM 用于两者。

那种何时合适：

| LLM 代理                                 | CALM                            |
| ---------------------------------------- | ------------------------------- |
| 允许用户以开放式方式使用工具/API         | 针对有限技能/用户目标的业务逻辑 |
| 拥有一组实际上无限的可能任务和开放式目标 | 业务逻辑需要严格执行            |
| 让机器人的终端用户完全自主               | 限制终端用户可以做什么          |

对于业务用例，CALM 比 LLM 代理具有优势：

1. 业务逻辑被明确地写下来，易于编辑，并且可以确保它被严格地遵循。
2. 业务逻辑已经为人所知，因此无需依赖 LLM 来猜测它。这避免了为了响应一条用户消息而连续执行多个 LLM 调用，这种方法对于大多数应用程序来说太慢了。
3. 可以根据需要使业务逻辑变得尽可能复杂，而不必担心 LLM 是否会“忘记”某个步骤。
4. 可以在对话过程中验证用户提供的所有信息（即每个槽值）。不必等到信息收集完毕，API 响应可以给到一个错误。
5. 终端用户无法使用提示注入来覆盖业务逻辑。语言模型作为推理引擎是一个从根本上不安全的提议，参见 [OWASP top 10](https://owasp.org/www-project-top-10-for-large-language-model-applications/assets/PDF/OWASP-Top-10-for-LLMs-2023-slides-v1_0.pdf)。
6. 如果没有 CALM 的可选生成组件，LLM 输出将仅限于一组固定命令，从而消除幻觉风险，同时降低延迟和 Token 生成成本。

!!! info "开发者版本"

    如果你想开始使用 CALM 构建对话机器人，可以获取 [Rasa Pro 开发者版](developer-edition.md)。
