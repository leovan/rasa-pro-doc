# 使用企业搜索策略进行自定义信息检索

了解如何使用自定义信息检索组件和企业搜索策略集成自定义搜索系统或向量存储。

!!! info "3.9 版本新特性"

    Rasa 现在支持与 [`EnterpriseSearchPolicy`](enterprise-search-policy.md) 一起使用的自定义信息检索器。此功能允许你将自己的自定义搜索系统或向量存储与 Rasa Pro 集成。

## 简介 {#introduction}

Rasa 最初与 Qdrant 和 Milvus 等向量存储的集成为更高级的信息检索功能奠定了基础。我们认识到需要提供更灵活、更可扩展的接口，从而创建了 `InformationRetrieval` 接口。此接口是向量存储的超集，涵盖了更广泛的信息检索技术。

`InformationRetrieval` 接口使你不仅可以集成向量存储，还可以集成自定义搜索系统、数据库或任何其他用于检索相关信息的机制。这种灵活性使你能够根据特定的用例和要求自定义和优化信息检索策略。

## 创建自定义信息检索类 {#creating-a-custom-information-retrieval-class}

你可以将自己的自定义信息检索组件实现为 Python 类。自定义信息检索类必须继承自 `rasa.core.information_retrieval.InformationRetrieval` 并实现 `connect` 和 `search` 方法。

```python
from rasa.utils.endpoints import EndpointConfig
from rasa.core.information_retrieval import SearchResultList, InformationRetrieval

class MyVectorStore(InformationRetrieval):
    def connect(self, config: EndpointConfig) -> None:
        # Create a connection to the search system
        pass

    async def search(
        self, query: Text, tracker_state: dict[Text, Any], threshold: float = 0.0
    ) -> SearchResultList:
        # Implement the search functionality to retrieve relevant results based on the query and top_n parameter.
        pass
```

你可以使用 `self.embeddings` 作为 [`langchain.schema.embeddings.Embeddings`](https://github.com/langchain-ai/langchain/blob/v0.0.329/libs/langchain/langchain/schema/embeddings.py) 对象来访问 Rasa 配置中定义的嵌入模型。

### `connect` 方法 {#connect-method}

`connect` 方法建立与信息检索系统的连接。它需要一个参数。

- `config`：这是信息检索组件的端点配置。`config.kwargs` 是一个 Python 字典，可用于访问 `vector_store` 下在 `endpoints.yml` 中定义的键。

例如：

```yaml title="endpoints.yml"
vector_store:
  api_key: <api_key> # user defined key
  collection: rasa   # user defined key
```

### `search` 方法 {#search-method}

`search` 方法查询信息检索系统中的文档并返回 `SearchResultList` 对象，该对象是与查询匹配的文档列表。

该方法需要以下参数：

- `query`：查询字符串。通常是最后一条用户消息。
- `tracker_state`：当前追踪器状态的字典。
- `threshold`：将文档视为匹配的最小相似度得分。

### 追踪器状态 {#tracker-state}

追踪器状态 Python 字典是 Rasa 追踪器的快照，包含有关 rasa 对话的元数据。它可用于获取有关任何对话事件的信息。它具有以下架构：

```json
{
    "sender_id": "string",
    "slots": {
        "additionalProp1": "string",
        "additionalProp2": "string",
        "additionalProp3": "string"
    },
    "latest_message": {
        "intent": {
        "name": "string",
        "confidence": 0
        },
        "entities": [
        {}
        ],
        "text": "string",
        "message_id": "string",
        "metadata": {},
        "commands": [
        {
            "command": "string"
        }
        ],
        "flows_from_semantic_search": [
        [
            "string",
            0
        ]
        ],
        "flows_in_prompt": [
        "string"
        ]
    },
    "latest_event_time": 0,
    "followup_action": "string",
    "paused": true,
    "stack": [
        {}
    ],
    "events": [
        {
        "event": "string",
        "timestamp": 0,
        "metadata": {},
        "name": "string",
        "policy": "string",
        "confidence": 0,
        "action_text": "string",
        "hide_rule_turn": true,
        "value": {}
        }
    ],
    "latest_input_channel": "string",
    "active_loop": {},
    "latest_action": {
        "action_name": "string"
    },
    "latest_action_name": "string"
}
```

追踪器状态有助于搜索系统为对话机器人提供进一步的自定义。例如，此 Python 函数从 `tracker_state` 对象创建聊天历史记录：

```python
def get_chat_history(tracker_state: dict[str, Any]) -> dict[str, Any]:
    chat_history = []
    last_user_message = ""
    for event in tracker_state.get("events"):
        if event.get("event") == "user":
            last_user_message = sanitize_message_for_prompt(event.get("text"))
            chat_history.append({"role": "USER", "message": last_user_message})
        elif event.get("event") == "bot":
            chat_history.append({"role": "CHATBOT", "message": event.get("text")})

    return chat_history
```

对话中当前处于活动状态的槽可以通过 `tracker_state.get("slots", {}).get(SLOT_NAME)` 访问。

### `SearchResultList` 数据类 {#searchresultlist-dataclass}

`SearchResultList` 数据类使用 `SearchResult` 数据类定义。这两个数据类均定义为：

```python
@dataclass
class SearchResult:
    text: str
    metadata: dict
    score: Optional[float] = None

@dataclass
class SearchResultList:
    results: List[SearchResult]
    metadata: dict
```

你可以使用类方法 `SearchResultList.from_document_list()` 从 [Langchain Document 对象](https://python.langchain.com/v0.2/docs/integrations/document_loaders/copypaste/)类型进行转换。

## 使用自定义信息检索组件 {#using-the-custom-information-retrieval-component}

要配置 [`EnterpriseSearchPolicy`](enterprise-search-policy.md) 以使用自定义组件。请将 `config.yml` 文件中的 `vector_store.type` 参数设置为自定义信息检索类的模块路径。

例如，对于保存在文件 `addons/custom_information_retrieval.py` 中的名为 `MyVectorStore` 的自定义信息检索类，模块路径将是 `addons.custom_information_retrieval.MyVectorStore`，凭据可能如下所示：

```yaml title="config.yml"
policies:
  - ...
  - name: EnterpriseSearchPolicy
    vector_store:
      type: "addons.custom_information_retrieval.MyVectorStore"
```

## 使用理念 {#usage-ideas}

自定义信息检索功能为自定义和增强对话机器人提供了一系列可能性。以下是一些潜在用例：

### 传统搜索、向量搜索或重新排序器 {#traditional-search-vector-search-or-rerankers}

你可以灵活地连接到任何搜索系统，无论是传统的基于 BM25 的搜索引擎、开源 Elasticsearch 实例、最先进的向量存储，还是你喜欢的重新排序模型。这种自由使你能够始终站在信息检索创新的最前沿，并利用最新的进展来增强对话机器人。

### 基于槽的检索 {#slot-based-retrieval}

使用自定义信息检索，你可以利用 `tracker_state` 提供的对话上下文和槽。这将启用基于槽的检索，允许你根据特定槽的值过滤搜索结果。例如，你可以根据用户之前提到的偏好或兴趣检索产品推荐。

你还可以根据定义用户访问级别的槽向搜索系统添加过滤器。

### 自定义搜索查询 {#customizing-search-queries}

对话机器人开发人员可以根据可用的对话上下文自定义或重新表述搜索查询。企业搜索策略仅使用最后一条用户消息作为搜索查询。使用自定义信息检索，你可以灵活地合并其他上下文或修改查询以更好地匹配用户的意图。

重新表述对于提高搜索准确性或使查询适应自定义系统的特定要求非常有用。

### 支持其他嵌入模型 {#support-for-additional-embedding-models}

如果 Rasa 本身不支持你想要使用的特定嵌入模型，则自定义信息检索可以解决问题。你可以集成你选择的本地或微调嵌入模型来生成搜索查询和文档的嵌入。

### 库灵活性 {#library-flexibility}

你可以自由地使用你选择的其他库或工具来执行信息检索任务。这使你能够利用你的组织可能已经在使用的专用库或内部工具。

这些用例突出了自定义信息检索的多功能性，使你能够定制信息检索策略以满足你的特定需求并增强整体用户体验。
