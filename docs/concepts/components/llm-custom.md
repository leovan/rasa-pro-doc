# 自定义基于 LLM 的组件

<button data-md-color-primary="amber">Rasa Labs</button>

!!! info "3.7.0b1 新特性"

    Rasa Labs 的功能是实验性的。我们引入实验性功能是为了与客户共同创造。要了解有关如何参与我们的实验室计划的更多信息，请访问我们的 [Rasa Labs 页面](https://rasa.com/rasa-labs/)。

    我们根据客户反馈不断改进 Rasa Labs 功能。要从最新的错误修复和功能改进中受益，请使用以下方式安装最新的预发布版本：

    ```shell
    pip install 'rasa-pro>3.8' --pre --upgrade
    ```

LLM 组件可以通过自定义版本进行扩展和修改。这样你就可以根据需要自定义 LLM 组件的行为，并尝试不同的算法。

## 自定义组件 {#customizing-a-component}

LLM 组件被实现为一组可以扩展和修改的类。以下示例展示了如何扩展 `ContextualResponseRephraser` 组件以添加自定义行为。

例如，我们可以更改改写逻辑以使用一些更简单的改写策略，而不是在特定情况下调用 LLM（例如，当响应中存在某个关键字时）。这可以通过扩展 `ContextualResponseRephraser` 类并重写改写方法来实现：

```python
from rasa.core import ContextualResponseRephraser

class CustomContextualResponseRephraser(ContextualResponseRephraser):
    async def rephrase(self, response: Dict[str, Any], tracker: DialogueStateTracker) -> Dict[str, Any]:
        original_text = response.get("text", "")
        if "example_keyword" in original_text:
            response["text"] = original_text.replace("hello", "bonjour")
            return response
        else:
            # Fallback to the default rephrase method
            return await super().rephrase(response, tracker)
```

然后可以在 Rasa 配置文件中使用自定义组件：

```yaml title="config.yml"
pipeline:
  - name: CustomContextualResponseRephraser
    # ...
```

要在 Rasa 配置文件中引用组件，你需要使用组件类的全名。组件类的全名是 `<module>.<class>`。

所有组件的源代码都有很好的注释。代码可以在本地安装的 Rasa Pro python 包中找到。

## 需要覆盖的常见函数 {#common-functions-to-be-overridden}

以下是可以覆盖以自定义 LLM 组件的函数列表：

### ContextualResponseRephraser {#contextualresponserephraser}

#### rephrase {#rephrase}

重新措辞 LLM 生成的响应。默认实现通过提示 LLM 根据传入消息和原始生成的响应来重新措辞响应。然后用生成的响应替换原始生成的响应。

```python
    def rephrase(
        self,
        response: Dict[str, Any],
        tracker: DialogueStateTracker,
    ) -> Dict[str, Any]:
        """Predicts a variation of the response.

        Args:
            response: The response to rephrase.
            tracker: The tracker to use for the prediction.
            model_name: The name of the model to use for the prediction.

        Returns:
            The response with the rephrased text.
        """
```

### IntentlessPolicy {#intentlesspolicy}

#### select_response_examples {#select_response_examples}

对适合当前对话的响应进行抽样。默认实现从适合当前对话的领域中抽样响应。选择基于对话历史记录，将历史记录嵌入并选择最相似的响应。

```python
    def select_response_examples(
        self,
        history: str,
        number_of_samples: int,
        max_number_of_tokens: int,
    ) -> List[str]:
        """Samples responses that fit the current conversation.

        Args:
            history: The conversation history.
            policy_model: The policy model.
            number_of_samples: The number of samples to return.
            max_number_of_tokens: Maximum number of tokens for responses.

        Returns:
            The sampled conversation in order of score decrease.
        """
```

#### select_few_shot_conversations {#select_few_shot_conversations}

从训练数据中抽取对话样本。默认实现从适合当前对话的训练数据中抽取对话样本。选择基于对话历史，将历史记录嵌入并选择最相似的对话。

```python
    def select_few_shot_conversations(
        self,
        history: str,
        number_of_samples: int,
        max_number_of_tokens: int,
    ) -> List[str]:
        """Samples conversations from the given conversation samples.

        Excludes conversations without AI replies

        Args:
            history: The conversation history.
            number_of_samples: The number of samples to return.
            max_number_of_tokens: Maximum number of tokens for conversations.

        Returns:
            The sampled conversation ordered by similarity decrease.
        """
```
