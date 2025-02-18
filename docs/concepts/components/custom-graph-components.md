# 自定义图组件

你可以使用自定义 NLU 组件和策略扩展 Rasa。本页面提供了有关如何开发你自己的自定义图组件的指南。

Rasa 提供各种现成的 [NLU 组件](../../nlu-based-assistants/components.md)和[策略](../policies/policy-overview.md)。你可以使用自定义图组件自定义它们或从头开始创建自己的组件。

要将自定义图组件与 Rasa 一起使用，它必须满足以下要求：

- 它必须实现 [`GraphComponent` 接口](#the-graphcomponent-interface)。
- 它必须在使用的模型配置中[注册](#registering-graph-components-with-the-model-configuration)。
- 它必须在[配置文件](#using-custom-components-in-your-model-configuration)中使用。
- 它必须使用类型注释。Rasa 使用类型注释来验证模型配置。不允许[前向引用](https://www.python.org/dev/peps/pep-0484/#forward-references)。如果你使用的是 Python 3.7，则可以使用 [`from __future__ import comments`](https://www.python.org/dev/peps/pep-0563/#enabling-the-future-behavior-in-python-3-7) 来摆脱前向引用。

## 图组件 {#graph-components}

Rasa 使用传入的[模型配置](../../nlu-based-assistants/model-configuration.md)来构建[有向无环图](https://en.wikipedia.org/wiki/Directed_acyclic_graph)。此图描述了模型配置中各项目之间的依赖关系以及数据在它们之间的流动方式。这有两个主要好处：

- Rasa 可以使用计算图来优化模型的执行。例如，高效缓存训练步骤或并行执行独立步骤。
- Rasa 可以灵活地表示不同的模型架构。只要图保持无环，理论上 Rasa 就可以根据模型配置将任何数据传递给任何图组件，而无需将底层软件架构与使用的模型架构绑定在一起。

将模型配置转换为计算图时，[策略](../policies/policy-overview.md)和 [NLU 组件](../../nlu-based-assistants/components.md)将成为此图中的节点。虽然模型配置中的策略和 NLU 组件之间存在区别，但当它们放置在图中时，区别就被抽象掉了。此时，策略和 NLU 组件成为抽象图组件。实际上，这由 [`GraphComponent`](#the-graphcomponent-interface) 接口表示：策略和 NLU 组件都必须从此接口继承，才能与 Rasa 的图兼容并可执行。

<figure markdown>
  ![](../../images/concepts/components/custom-graph-components/graph_architecture.png)
</figure>

## 入门 {#getting-started}

在开始之前，你必须决定是否要实现自定义 [NLU 组件](../../nlu-based-assistants/components.md)或[策略](../policies/policy-overview.md)。如果你要实现自定义策略，那么我们建议扩展已实现 `GraphComponent` 接口的现有 `rasa.core.policies.policy.Policy` 类。

```python
from rasa.core.policies.policy import Policy
from rasa.engine.recipes.default_recipe import DefaultV1Recipe

# TODO: Correctly register your graph component
@DefaultV1Recipe.register(
    [DefaultV1Recipe.ComponentType.POLICY_WITHOUT_END_TO_END_SUPPORT], is_trainable=True
)
class MyPolicy(Policy):
    ...
```

如果你想实现自定义 NLU 组件，那么请从以下框架开始：

```python
from typing import Dict, Text, Any, List

from rasa.engine.graph import GraphComponent, ExecutionContext
from rasa.engine.recipes.default_recipe import DefaultV1Recipe
from rasa.engine.storage.resource import Resource
from rasa.engine.storage.storage import ModelStorage
from rasa.shared.nlu.training_data.message import Message
from rasa.shared.nlu.training_data.training_data import TrainingData

# TODO: Correctly register your component with its type
@DefaultV1Recipe.register(
    [DefaultV1Recipe.ComponentType.INTENT_CLASSIFIER], is_trainable=True
)
class CustomNLUComponent(GraphComponent):
    @classmethod
    def create(
        cls,
        config: Dict[Text, Any],
        model_storage: ModelStorage,
        resource: Resource,
        execution_context: ExecutionContext,
    ) -> GraphComponent:
        # TODO: Implement this
        ...

    def train(self, training_data: TrainingData) -> Resource:
        # TODO: Implement this if your component requires training
        ...

    def process_training_data(self, training_data: TrainingData) -> TrainingData:
        # TODO: Implement this if your component augments the training data with
        #       tokens or message features which are used by other components
        #       during training.
        ...

        return training_data

    def process(self, messages: List[Message]) -> List[Message]:
        # TODO: This is the method which Rasa Open Source will call during inference.
        ...
        return messages
```

阅读以下部分，了解如何解决上述示例中的 `TODO`，以及自定义组件中需要实现哪些其他方法。

!!! info "自定义分词器"

    如果创建自定义分词器，则应扩展 `rasa.nlu.tokenizers.tokenizer.Tokenizer` 类。`train` 和 `process` 方法已实现，因此你只需覆盖 `tokenize` 方法。

## `GraphComponent` 接口 {#the-graphcomponent-interface}

要使用 Rasa 运行自定义 NLU 组件或策略，它必须实现 `GraphComponent` 接口。

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from typing import List, Type, Dict, Text, Any, Optional

from rasa.engine.graph import ExecutionContext
from rasa.engine.storage.resource import Resource
from rasa.engine.storage.storage import ModelStorage


class GraphComponent(ABC):
    """Interface for any component which will run in a graph."""

    @classmethod
    def required_components(cls) -> List[Type]:
        """Components that should be included in the pipeline before this component."""
        return []

    @classmethod
    @abstractmethod
    def create(
        cls,
        config: Dict[Text, Any],
        model_storage: ModelStorage,
        resource: Resource,
        execution_context: ExecutionContext,
    ) -> GraphComponent:
        """Creates a new `GraphComponent`.

        Args:
            config: This config overrides the `default_config`.
            model_storage: Storage which graph components can use to persist and load
                themselves.
            resource: Resource locator for this component which can be used to persist
                and load itself from the `model_storage`.
            execution_context: Information about the current graph run.

        Returns: An instantiated `GraphComponent`.
        """
        ...

    @classmethod
    def load(
        cls,
        config: Dict[Text, Any],
        model_storage: ModelStorage,
        resource: Resource,
        execution_context: ExecutionContext,
        **kwargs: Any,
    ) -> GraphComponent:
        """Creates a component using a persisted version of itself.

        If not overridden this method merely calls `create`.

        Args:
            config: The config for this graph component. This is the default config of
                the component merged with config specified by the user.
            model_storage: Storage which graph components can use to persist and load
                themselves.
            resource: Resource locator for this component which can be used to persist
                and load itself from the `model_storage`.
            execution_context: Information about the current graph run.
            kwargs: Output values from previous nodes might be passed in as `kwargs`.

        Returns:
            An instantiated, loaded `GraphComponent`.
        """
        return cls.create(config, model_storage, resource, execution_context)

    @staticmethod
    def get_default_config() -> Dict[Text, Any]:
        """Returns the component's default config.

        Default config and user config are merged by the `GraphNode` before the
        config is passed to the `create` and `load` method of the component.

        Returns:
            The default config of the component.
        """
        return {}

    @staticmethod
    def supported_languages() -> Optional[List[Text]]:
        """Determines which languages this component can work with.

        Returns: A list of supported languages, or `None` to signify all are supported.
        """
        return None

    @staticmethod
    def not_supported_languages() -> Optional[List[Text]]:
        """Determines which languages this component cannot work with.

        Returns: A list of not supported languages, or
            `None` to signify all are supported.
        """
        return None

    @staticmethod
    def required_packages() -> List[Text]:
        """Any extra python dependencies required for this component to run."""
        return []

    @classmethod
    def fingerprint_addon(cls, config: Dict[str, Any]) -> Optional[str]:
        """Adds additional data to the fingerprint calculation.

        This is useful if a component uses external data that is not provided
        by the graph.
        """
        return None
```

### `create` {#create}

`create` 方法用于在训练期间实例化图组件，并且必须被覆盖。调用该方法时，Rasa 会传递以下参数：

- `config`：这是组件的默认配置，与模型配置文件中提供给图组件的配置合并。
- `model_storage`：可以使用它来持久化和加载图组件。有关其用法的更多详细信息，请参阅[模型持久化](#model-persistence)部分。
- `resource`：`model_storage` 中组件的唯一标识符。有关其用法的更多详细信息，请参阅[模型持久化](#model-persistence)部分。
- `execution_context`：这提供了有关当前执行模式的其他信息：
    - `model_id`：推理期间使用的模型的唯一标识符。此参数在训练期间为 `None`。
    - `should_add_diagnostic_data`：如果为 `True`，则应在实际预测的基础上将其他诊断元数据添加到图组件的预测中。
    - `is_finetuning`：如果为 `True`，则可以使用[微调](../../command-line-interface.md#incremental-training)来训练图组件。
    - `graph_schema`：`graph_schema` 描述用于训练对话机器人或用其进行预测的计算图。
    - `node_name`：`node_name` 是图模式中步骤的唯一标识符，由调用的图组件实现。

### `load` {#load}

`load` 方法用于在推理期间实例化图组件。此方法的默认实现会调用 `create` 方法。如果图组件将[数据作为训练的一部分保留](#model-persistence)，则建议覆盖此方法。请参阅 [`create`](#create) 以了解各个参数的描述。

### `get_default_config` {#get_default_config}

`get_default_config` 方法返回图组件的默认配置。其默认实现返回一个空字典，这意味着图组件没有任何配置。Rasa 将在运行时使用配置文件中给出的内容更新默认配置。

### `supports_languages` {#supports_languages}

`supported_languages` 方法指定图组件支持哪些[语言](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes)。Rasa 将使用模型配置文件中的 `language` 键来验证图组件是否适用于指定语言。如果图组件返回 `None`（这是默认实现），则表示图组件支持不属于 `not_supported_languages` 的所有语言。

示例：

- `[]`：图组件不支持任何语言。
- `None`：除 `not_supported_languages` 中定义的语言外，所有语言均受支持。
- `["en"]`：图组件只能用于英语对话。

### `not_supported_languages` {#not_supported_languages}

`not_supported_languages` 方法指定图组件不支持哪些[语言](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes)。Rasa 将使用模型配置文件中的 `language` 键来验证图组件是否适用于指定语言。如果图组件返回 `None`（这是默认实现），则表明它支持在 `supported_languages` 中指定的所有语言。

示例：

- `None` 或 `[]`：`supported_languages` 中指定的所有语言均受支持。
- `["en"]`：图组件可以与除英语以外的任何语言一起使用。

### `required_pa​​ckages` {#required_pa​​ckages}

`required_pa​​ckages` 方法指示需要安装哪些额外的 Python 包才能使用此图组件。如果在运行时未找到所需的库，Rasa 将在执行期间引发错误。默认情况下，此方法返回一个空列表，这意味着图组件没有任何额外的依赖项。

示例：

- `[]`：使用此图组件不需要额外的包。
- `["spacy"]`：需要安装 Python 包 `spacy` 才能使用此图组件。

## 模型持久化 {#model-persistence}

某些图组件需要在训练期间持久保存数据，这些数据应在推理时可供图组件使用。一个典型的用例是存储模型权重。Rasa 为此目的向图组件的创建和加载方法提供了 `model_storage` 和 `resource` 参数，如下面的代码片段所示。`model_storage` 提供对所有图组件数据的访问。该 `resource` 允许唯一地标识图组件在模型存储中的位置。

```python hl_lines="14 15 24 25"
from __future__ import annotations

from typing import Any, Dict, Text

from rasa.engine.graph import GraphComponent, ExecutionContext
from rasa.engine.storage.resource import Resource
from rasa.engine.storage.storage import ModelStorage

class MyComponent(GraphComponent):
    @classmethod
    def create(
        cls,
        config: Dict[Text, Any],
        model_storage: ModelStorage,
        resource: Resource,
        execution_context: ExecutionContext,
    ) -> MyComponent:
        ...

    @classmethod
    def load(
        cls,
        config: Dict[Text, Any],
        model_storage: ModelStorage,
        resource: Resource,
        execution_context: ExecutionContext,
        **kwargs: Any
    ) -> MyComponent:
        ...
```

### 写入模型存储 {#writing-to-the-model-storage}

下面的代码片段说明了如何将图组件的数据写入模型存储。为了在训练后保留图组件，训练方法需要访问 `model_storage` 和 `resource` 的值。因此，你应该在初始化时存储 `model_storage` 和 `resource` 的值。

图组件的训练方法必须返回 `resource` 的值，以便 Rasa 可以在训练之间缓存训练结果。`self._model_storage.write_to(self._resource)` 上下文管理器提供了一个目录路径，你可以在其中保留图组件所需的任何数据。

```python
from __future__ import annotations
import json
from typing import Optional, Dict, Any, Text

from rasa.engine.graph import GraphComponent, ExecutionContext
from rasa.engine.storage.resource import Resource
from rasa.engine.storage.storage import ModelStorage
from rasa.shared.nlu.training_data.training_data import TrainingData

class MyComponent(GraphComponent):

    def __init__(
        self,
        model_storage: ModelStorage,
        resource: Resource,
        training_artifact: Optional[Dict],
    ) -> None:
        # Store both `model_storage` and `resource` as object attributes to be able
        # to utilize them at the end of the training
        self._model_storage = model_storage
        self._resource = resource

    @classmethod
    def create(
        cls,
        config: Dict[Text, Any],
        model_storage: ModelStorage,
        resource: Resource,
        execution_context: ExecutionContext,
    ) -> MyComponent:
        return cls(model_storage, resource, training_artifact=None)

    def train(self, training_data: TrainingData) -> Resource:
        # Train your graph component
        ...

        # Persist your graph component
        with self._model_storage.write_to(self._resource) as directory_path:
            with open(directory_path / "artifact.json", "w") as file:
                json.dump({"my": "training artifact"}, file)

        # Return resource to make sure the training artifacts
        # can be cached.
        return self._resource
```

### 从模型存储中读取 {#reading-from-the-model-storage}

Rasa 将调用图组件的 `load` 方法来实例化它以进行推理。你可以使用上下文管理器 `self._model_storage.read_from(resource)` 获取图组件数据持久化的目录路径。然后，你可以使用提供的路径加载持久化数据并使用它初始化图组件。请注意，如果未找到给定 `resource` 的持久化数据，`model_storage` 将抛出 `ValueError`。

```python
from __future__ import annotations
import json
from typing import Optional, Dict, Any, Text

from rasa.engine.graph import GraphComponent, ExecutionContext
from rasa.engine.storage.resource import Resource
from rasa.engine.storage.storage import ModelStorage

class MyComponent(GraphComponent):

    def __init__(
        self,
        model_storage: ModelStorage,
        resource: Resource,
        training_artifact: Optional[Dict],
    ) -> None:
        self._model_storage = model_storage
        self._resource = resource

    @classmethod
    def load(
        cls,
        config: Dict[Text, Any],
        model_storage: ModelStorage,
        resource: Resource,
        execution_context: ExecutionContext,
        **kwargs: Any,
    ) -> MyComponent:
        try:
            with model_storage.read_from(resource) as directory_path:
                with open(directory_path / "artifact.json", "r") as file:
                    training_artifact = json.load(file)
                    return cls(
                        model_storage, resource, training_artifact=training_artifact
                    )
        except ValueError:
            # This allows you to handle the case if there was no
            # persisted data for your component
            ...
```

## 使用模型配置注册图组件 {#registering-graph-components-with-the-model-configuration}

要使图组件可供 Rasa 使用，你可能必须使用配方注册图组件。Rasa 使用配方将模型配置的内容转换为可执行[图](#graph-components)。目前，Rasa 支持 `default.v1` 和实验性 `graph.v1` 配方。对于 `default.v1` 配方，你需要使用 `DefaultV1Recipe.register` 装饰器注册图组件：

```python
from rasa.engine.graph import GraphComponent
from rasa.engine.recipes.default_recipe import DefaultV1Recipe


@DefaultV1Recipe.register(
    component_types=[DefaultV1Recipe.ComponentType.INTENT_CLASSIFIER],
    is_trainable=True,
    model_from="SpacyNLP",
)
class MyComponent(GraphComponent):
    ...
```

Rasa 使用 `register` 装饰器中提供的信息以及图组件在配置文件中的位置来安排图组件及其所需数据的执行。`DefaultV1Recipe.register` 装饰器允许你指定以下详细信息：

- `component_types`：这指定了图组件在对话机器人中实现的用途。可以指定多种类型（例如，如果图组件既是意图分类器又是实体提取器）：
    - `ComponentType.MODEL_LOADER`：[语言模型](../../nlu-based-assistants/components.md#language-models)的组件类型。如果其他图组件指定了 `model_from=<model loader name>`，则此类型的图组件会为它们提供预训练模型。此图组件在训练和推理期间运行。Rasa 将使用图组件的 `provide` 方法来检索应提供给依赖图组件的模型。
    - `ComponentType.MESSAGE_TOKENIZER`：[分词器](../../nlu-based-assistants/components.md#tokenizers)的组件类型。此图组件在训练和推理期间运行。如果指定了 `is_trainable=True`，Rasa 将使用图组件的 `train` 方法。Rasa 将使用 `process_training_data` 对训练数据示例进行分词，并使用 `process` 在推理期间对消息进行分词。
    - `ComponentType.MESSAGE_FEATURIZER`：[特征化器](../../nlu-based-assistants/components.md#featurizers)的组件类型。此图组件在训练和推理期间运行。如果指定了 `is_trainable=True`，Rasa 将使用图组件的 `train` 方法。Rasa 将使用 `process_training_data` 来特征化训练数据示例，并使用 `process` 在推理期间特征化消息。
    - `ComponentType.INTENT_CLASSIFIER`：[意图分类器](../../nlu-based-assistants/components.md#intent-classifiers)的组件类型。如果 `is_trainable=True`，则此图组件仅在训练期间运行。图组件始终在推理期间运行。如果指定了 `is_trainable=True`，Rasa 将使用图组件的 `train` 方法。在推理期间，Rasa 将使用图组件的 `process` 方法对消息的意图进行分类。
    - `ComponentType.ENTITY_EXTRACTOR`：[实体提取器](../../nlu-based-assistants/components.md#entity-extractors)的组件类型。如果 `is_trainable=True`，则此图组件仅在训练期间运行。图组件始终在推理期间运行。如果指定了 `is_trainable=True`，则 Rasa 将使用图组件的 `train` 方法。在推理期间，Rasa 将使用图组件的 `process` 方法提取实体。
    - `ComponentType.POLICY_WITHOUT_END_TO_END_SUPPORT`：不需要额外端到端功能的[策略](../policies/policy-overview.md)的组件类型（有关详细信息，请参阅[端到端训练](../../nlu-based-assistants/stories.md#end-to-end-training)）。如果 `is_trainable=True`，则此图组件仅在训练期间运行。图组件始终在推理期间运行。如果指定了 `is_trainable=True`，Rasa 将使用图组件的 `train` 方法。Rasa 将使用图组件的 `predict_action_probabilities` 来预测对话中应运行的下一个动作。
    - `ComponentType.POLICY_WITH_END_TO_END_SUPPORT`：需要额外端到端功能的[策略](../policies/policy-overview.md)的组件类型（有关更多信息，请参阅[端到端训练](../../nlu-based-assistants/stories.md#end-to-end-training)）。端到端功能作为 `precomputations` 参数传递到图组件的 `train` 和 `predict_action_probabilities` 中。如果 `is_trainable=True`，则此图组件仅在训练期间运行。图组件始终在推理期间运行。如果指定了 `is_trainable=True`，Rasa 将使用图组件的 `train` 方法。Rasa 将使用图组件的 `predict_action_probabilities` 来预测对话中应运行的下一个动作。

- `is_trainable`：指定图组件是否需要先进行自我训练，然后才能处理其他依赖图组件的训练数据或进行预测。
- `model_from`：指定是否需要向图组件的 `train`、`process_training_data` 和 `process` 方法提供预训练[语言模型](../../nlu-based-assistants/components.md#language-models)。这些方法必须支持参数 `model` 才能接收语言模型。请注意，你仍需确保提供此模型的图组件是模型配置的一部分。一个常见的用例是，如果你想将 [SpacyNLP](https://rasa.com/docs/rasa-pro/nlu-based-assistants/components#spacynlp) 语言模型公开给其他 NLU 组件。

## 在模型配置中使用自定义组件 {#using-custom-components-in-your-model-configuration}

你可以在[模型配置](../../nlu-based-assistants/model-configuration.md)中像使用任何其他 NLU 组件或策略一样使用自定义图形组件。唯一的变化是你必须指定完整的模块名称，而不是仅指定类名称。完整的模块名称取决于模块相对于指定 [PYTHONPATH](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONPATH) 的位置。默认情况下，Rasa 会将你运行 CLI 的目录添加到 `PYTHONPATH`。例如，如果你从 `/Users/<user>/my-rasa-project` 运行 CLI，并且模块 `MyComponent` 位于 `/Users/<user>/my-rasa-project/custom_components/my_component.py` 中，则模块路径为 `custom_components.my_component.MyComponent`。除名称条目之外的所有内容都将作为配置传递给组件。

```yaml title="config.yml" hl_lines="5 6 10"
recipe: default.v1
language: en
pipeline:
# other NLU components
- name: your.custom.NLUComponent
  setting_a: 0.01
  setting_b: string_value

policies:
# other dialogue policies
- name: your.custom.Policy
```

## 实现提示 {#implementation-hints}

### 消息元数据 {#message-metadata}

当你[在训练数据中为意图示例定义元数据](../../nlu-based-assistants/training-data-format.md#training-examples)时，你的 NLU 组件可以在处理过程中访问意图元数据和意图示例元数据：

```python
# in your component class

    def process(self, message: Message, **kwargs: Any) -> None:
        metadata = message.get("metadata")
        print(metadata.get("intent"))
        print(metadata.get("example"))
```

### 稀疏和稠密消息特征 {#sparse-and-dense-message-features}

如果你创建自定义消息特征化器，可以返回两种不同类型的特征：序列特征和句子特征。序列特征是一个大小为 `(number-of-tokens x feature-dimension)` 的矩阵，即矩阵包含序列中每个 token 的特征向量。句子特征由一个大小为 `(1 x feature-dimension)` 的矩阵表示。

## 自定义组件示例 {#examples-of-custom-components}

### 稠密消息特征化器 {#dense-message-featurizer}

以下是使用预训练模型的稠密[消息特征化器](../../nlu-based-assistants/components.md#featurizers)示例：

```python
import numpy as np
import logging
from bpemb import BPEmb
from typing import Any, Text, Dict, List, Type

from rasa.engine.recipes.default_recipe import DefaultV1Recipe
from rasa.engine.graph import ExecutionContext, GraphComponent
from rasa.engine.storage.resource import Resource
from rasa.engine.storage.storage import ModelStorage
from rasa.nlu.featurizers.dense_featurizer.dense_featurizer import DenseFeaturizer
from rasa.nlu.tokenizers.tokenizer import Tokenizer
from rasa.shared.nlu.training_data.training_data import TrainingData
from rasa.shared.nlu.training_data.features import Features
from rasa.shared.nlu.training_data.message import Message
from rasa.nlu.constants import (
    DENSE_FEATURIZABLE_ATTRIBUTES,
    FEATURIZER_CLASS_ALIAS,
)
from rasa.shared.nlu.constants import (
    TEXT,
    TEXT_TOKENS,
    FEATURE_TYPE_SENTENCE,
    FEATURE_TYPE_SEQUENCE,
)


logger = logging.getLogger(__name__)


@DefaultV1Recipe.register(
    DefaultV1Recipe.ComponentType.MESSAGE_FEATURIZER, is_trainable=False
)
class BytePairFeaturizer(DenseFeaturizer, GraphComponent):
    @classmethod
    def required_components(cls) -> List[Type]:
        """Components that should be included in the pipeline before this component."""
        return [Tokenizer]

    @staticmethod
    def required_packages() -> List[Text]:
        """Any extra python dependencies required for this component to run."""
        return ["bpemb"]

    @staticmethod
    def get_default_config() -> Dict[Text, Any]:
        """Returns the component's default config."""
        return {
            **DenseFeaturizer.get_default_config(),
            # specifies the language of the subword segmentation model
            "lang": None,
            # specifies the dimension of the subword embeddings
            "dim": None,
            # specifies the vocabulary size of the segmentation model
            "vs": None,
            # if set to True and the given vocabulary size can't be loaded for the given
            # model, the closest size is chosen
            "vs_fallback": True,
        }

    def __init__(
        self,
        config: Dict[Text, Any],
        name: Text,
    ) -> None:
        """Constructs a new byte pair vectorizer."""
        super().__init__(name, config)
        # The configuration dictionary is saved in `self._config` for reference.
        self.model = BPEmb(
            lang=self._config["lang"],
            dim=self._config["dim"],
            vs=self._config["vs"],
            vs_fallback=self._config["vs_fallback"],
        )

    @classmethod
    def create(
        cls,
        config: Dict[Text, Any],
        model_storage: ModelStorage,
        resource: Resource,
        execution_context: ExecutionContext,
    ) -> GraphComponent:
        """Creates a new component (see parent class for full docstring)."""
        return cls(config, execution_context.node_name)

    def process(self, messages: List[Message]) -> List[Message]:
        """Processes incoming messages and computes and sets features."""
        for message in messages:
            for attribute in DENSE_FEATURIZABLE_ATTRIBUTES:
                self._set_features(message, attribute)
        return messages

    def process_training_data(self, training_data: TrainingData) -> TrainingData:
        """Processes the training examples in the given training data in-place."""
        self.process(training_data.training_examples)
        return training_data

    def _create_word_vector(self, document: Text) -> np.ndarray:
        """Creates a word vector from a text. Utility method."""
        encoded_ids = self.model.encode_ids(document)
        if encoded_ids:
            return self.model.vectors[encoded_ids[0]]

        return np.zeros((self.component_config["dim"],), dtype=np.float32)

    def _set_features(self, message: Message, attribute: Text = TEXT) -> None:
        """Sets the features on a single message. Utility method."""
        tokens = message.get(TEXT_TOKENS)

        # If the message doesn't have tokens, we can't create features.
        if not tokens:
            return None

        # We need to reshape here such that the shape is equivalent to that of sparsely
        # generated features. Without it, it'd be a 1D tensor. We need 2D (n_utterance, n_dim).
        text_vector = self._create_word_vector(document=message.get(TEXT)).reshape(
            1, -1
        )
        word_vectors = np.array(
            [self._create_word_vector(document=t.text) for t in tokens]
        )

        final_sequence_features = Features(
            word_vectors,
            FEATURE_TYPE_SEQUENCE,
            attribute,
            self._config[FEATURIZER_CLASS_ALIAS],
        )
        message.add_features(final_sequence_features)
        final_sentence_features = Features(
            text_vector,
            FEATURE_TYPE_SENTENCE,
            attribute,
            self._config[FEATURIZER_CLASS_ALIAS],
        )
        message.add_features(final_sentence_features)

    @classmethod
    def validate_config(cls, config: Dict[Text, Any]) -> None:
        """Validates that the component is configured properly."""
        if not config["lang"]:
            raise ValueError("BytePairFeaturizer needs language setting via `lang`.")
        if not config["dim"]:
            raise ValueError(
                "BytePairFeaturizer needs dimensionality setting via `dim`."
            )
        if not config["vs"]:
            raise ValueError("BytePairFeaturizer needs a vector size setting via `vs`.")
```

### 稀疏消息特征化器 {#sparse-message-featurizer}

以下是训练新模型的稀疏[消息特征化器](../../nlu-based-assistants/components.md#featurizers)示例：

```python
import logging
from typing import Any, Text, Dict, List, Type

from sklearn.feature_extraction.text import TfidfVectorizer
from rasa.engine.recipes.default_recipe import DefaultV1Recipe
from rasa.engine.graph import ExecutionContext, GraphComponent
from rasa.engine.storage.resource import Resource
from rasa.engine.storage.storage import ModelStorage
from rasa.nlu.featurizers.sparse_featurizer.sparse_featurizer import SparseFeaturizer
from rasa.nlu.tokenizers.tokenizer import Tokenizer
from rasa.shared.nlu.training_data.training_data import TrainingData
from rasa.shared.nlu.training_data.features import Features
from rasa.shared.nlu.training_data.message import Message
from rasa.nlu.constants import (
    DENSE_FEATURIZABLE_ATTRIBUTES,
    FEATURIZER_CLASS_ALIAS,
)
from joblib import dump, load
from rasa.shared.nlu.constants import (
    TEXT,
    TEXT_TOKENS,
    FEATURE_TYPE_SENTENCE,
    FEATURE_TYPE_SEQUENCE,
)

logger = logging.getLogger(__name__)


@DefaultV1Recipe.register(
    DefaultV1Recipe.ComponentType.MESSAGE_FEATURIZER, is_trainable=True
)
class TfIdfFeaturizer(SparseFeaturizer, GraphComponent):
    @classmethod
    def required_components(cls) -> List[Type]:
        """Components that should be included in the pipeline before this component."""
        return [Tokenizer]

    @staticmethod
    def required_packages() -> List[Text]:
        """Any extra python dependencies required for this component to run."""
        return ["sklearn"]

    @staticmethod
    def get_default_config() -> Dict[Text, Any]:
        """Returns the component's default config."""
        return {
            **SparseFeaturizer.get_default_config(),
            "analyzer": "word",
            "min_ngram": 1,
            "max_ngram": 1,
        }

    def __init__(
        self,
        config: Dict[Text, Any],
        name: Text,
        model_storage: ModelStorage,
        resource: Resource,
    ) -> None:
        """Constructs a new tf/idf vectorizer using the sklearn framework."""
        super().__init__(name, config)
        # Initialize the tfidf sklearn component
        self.tfm = TfidfVectorizer(
            analyzer=config["analyzer"],
            ngram_range=(config["min_ngram"], config["max_ngram"]),
        )

        # We need to use these later when saving the trained component.
        self._model_storage = model_storage
        self._resource = resource

    def train(self, training_data: TrainingData) -> Resource:
        """Trains the component from training data."""
        texts = [e.get(TEXT) for e in training_data.training_examples if e.get(TEXT)]
        self.tfm.fit(texts)
        self.persist()
        return self._resource

    @classmethod
    def create(
        cls,
        config: Dict[Text, Any],
        model_storage: ModelStorage,
        resource: Resource,
        execution_context: ExecutionContext,
    ) -> GraphComponent:
        """Creates a new untrained component (see parent class for full docstring)."""
        return cls(config, execution_context.node_name, model_storage, resource)

    def _set_features(self, message: Message, attribute: Text = TEXT) -> None:
        """Sets the features on a single message. Utility method."""
        tokens = message.get(TEXT_TOKENS)

        # If the message doesn't have tokens, we can't create features.
        if not tokens:
            return None

        # Make distinction between sentence and sequence features
        text_vector = self.tfm.transform([message.get(TEXT)])
        word_vectors = self.tfm.transform([t.text for t in tokens])

        final_sequence_features = Features(
            word_vectors,
            FEATURE_TYPE_SEQUENCE,
            attribute,
            self._config[FEATURIZER_CLASS_ALIAS],
        )
        message.add_features(final_sequence_features)
        final_sentence_features = Features(
            text_vector,
            FEATURE_TYPE_SENTENCE,
            attribute,
            self._config[FEATURIZER_CLASS_ALIAS],
        )
        message.add_features(final_sentence_features)

    def process(self, messages: List[Message]) -> List[Message]:
        """Processes incoming message and compute and set features."""
        for message in messages:
            for attribute in DENSE_FEATURIZABLE_ATTRIBUTES:
                self._set_features(message, attribute)
        return messages

    def process_training_data(self, training_data: TrainingData) -> TrainingData:
        """Processes the training examples in the given training data in-place."""
        self.process(training_data.training_examples)
        return training_data

    def persist(self) -> None:
        """
        Persist this model into the passed directory.

        Returns the metadata necessary to load the model again. In this case; `None`.
        """
        with self._model_storage.write_to(self._resource) as model_dir:
            dump(self.tfm, model_dir / "tfidfvectorizer.joblib")

    @classmethod
    def load(
        cls,
        config: Dict[Text, Any],
        model_storage: ModelStorage,
        resource: Resource,
        execution_context: ExecutionContext,
    ) -> GraphComponent:
        """Loads trained component from disk."""
        try:
            with model_storage.read_from(resource) as model_dir:
                tfidfvectorizer = load(model_dir / "tfidfvectorizer.joblib")
                component = cls(
                    config, execution_context.node_name, model_storage, resource
                )
                component.tfm = tfidfvectorizer
        except (ValueError, FileNotFoundError):
            logger.debug(
                f"Couldn't load metadata for component '{cls.__name__}' as the persisted "
                f"model data couldn't be loaded."
            )
        return component

    @classmethod
    def validate_config(cls, config: Dict[Text, Any]) -> None:
        """Validates that the component is configured properly."""
        pass
```

## NLU 元学习器 {#nlu-meta-learners}

!!! info "高级用例"

    NLU 元学习器是一种高级用例。以下部分仅适用于拥有基于先前分类器的输出来学习参数的组件的情况。对于手动设置参数或逻辑的组件，你可以创建一个具有 `is_trainable=False` 的组件，而不必担心前面的分类器。

NLU 元学习器是意图分类器或实体提取器，它们使用其他经过训练的意图分类器或实体提取器的预测并尝试改进其结果。元学习器的一个示例是，一个组件对两个先前的意图分类器的输出进行平均，或者一个回退分类器，它根据意图分类器对训练示例的置信度设置其阈值。

从概念上讲，要构建可训练的回退分类器，你首先需要将该回退分类器创建为自定义组件：

```python
from typing import Dict, Text, Any, List

from rasa.engine.graph import GraphComponent, ExecutionContext
from rasa.engine.recipes.default_recipe import DefaultV1Recipe
from rasa.engine.storage.resource import Resource
from rasa.engine.storage.storage import ModelStorage
from rasa.shared.nlu.training_data.message import Message
from rasa.shared.nlu.training_data.training_data import TrainingData
from rasa.nlu.classifiers.fallback_classifier import FallbackClassifier


@DefaultV1Recipe.register(
    [DefaultV1Recipe.ComponentType.INTENT_CLASSIFIER], is_trainable=True
)
class MetaFallback(FallbackClassifier):

    def __init__(
        self,
        config: Dict[Text, Any],
        model_storage: ModelStorage,
        resource: Resource,
        execution_context: ExecutionContext,
    ) -> None:
        super().__init__(config)

        self._model_storage = model_storage
        self._resource = resource

    @classmethod
    def create(
        cls,
        config: Dict[Text, Any],
        model_storage: ModelStorage,
        resource: Resource,
        execution_context: ExecutionContext,
    ) -> FallbackClassifier:
        """Creates a new untrained component (see parent class for full docstring)."""
        return cls(config, model_storage, resource, execution_context)

    def train(self, training_data: TrainingData) -> Resource:
        # Do something here with the messages
        return self._resource
```

接下来，你需要创建一个自定义意图分类器，该分类器也是一个特征生成器，因为分类器的输出需要由下游的另一个组件使用。对于自定义意图分类器组件，你还需要定义如何将其预测添加到指定 `process_training_data` 方法的消息数据中。确保不要覆盖意图的真实标签。以下模板展示了如何为此目的对 DIET 进行子类化：

```python
from rasa.engine.recipes.default_recipe import DefaultV1Recipe
from rasa.shared.nlu.training_data.training_data import TrainingData
from rasa.nlu.classifiers.diet_classifier import DIETClassifier


@DefaultV1Recipe.register(
    [DefaultV1Recipe.ComponentType.INTENT_CLASSIFIER,
     DefaultV1Recipe.ComponentType.ENTITY_EXTRACTOR,
     DefaultV1Recipe.ComponentType.MESSAGE_FEATURIZER], is_trainable=True
)
class DIETFeaturizer(DIETClassifier):

    def process_training_data(self, training_data: TrainingData) -> TrainingData:
        # classify and add the attributes to the messages on the training data
        return training_data
```
