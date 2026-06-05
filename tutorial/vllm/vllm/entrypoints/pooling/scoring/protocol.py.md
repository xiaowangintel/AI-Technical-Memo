# protocol.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/pooling/scoring/protocol.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Declares request/response schemas, protocol objects, and validation helpers. Scope: pooling scoring. / 声明请求/响应模式、协议对象与校验辅助逻辑。 作用域：池化 / 评分。

## Line-by-Line Analysis / 逐行分析
### Lines 3-16 — Imports and shared dependencies
```python
import time
from typing import Any, TypeAlias

from pydantic import BaseModel, Field, model_validator

from vllm import PoolingParams
from vllm.config import ModelConfig
from vllm.entrypoints.openai.engine.protocol import OpenAIBaseModel, UsageInfo
from vllm.renderers import TokenizeParams
from vllm.tasks import PoolingTask
from vllm.utils import random_uuid

from ..base.protocol import ClassifyRequestMixin, PoolingBasicRequestMixin
from .typing import ScoreContentPartParam, ScoreInput
```
**EN:** This import block pulls in standard-library modules such as `time`, `typing`, uses third-party packages like `pydantic`, depends on internal helpers such as `vllm`, `vllm.config`, `vllm.entrypoints.openai.engine.protocol`, `vllm.renderers`, `vllm.tasks`, `vllm.utils`.
**CN:** 该导入块引入 `time`, `typing` 等标准库模块，使用 `pydantic` 等第三方库，依赖 `vllm`, `vllm.config`, `vllm.entrypoints.openai.engine.protocol`, `vllm.renderers`, `vllm.tasks`, `vllm.utils` 等 vLLM 内部模块。

### Lines 19-81 — Class `ScoringRequestMixin`
```python
class ScoringRequestMixin(PoolingBasicRequestMixin, ClassifyRequestMixin):
    # --8<-- [start:scoring-common-params]
    max_tokens_per_query: int = Field(
        default=0,
        description=(
            "Maximum number of tokens per query. Queries longer than "
            "this will be truncated to this length. 0 means no "
            "query-level truncation is applied."
        ),
    )
    max_tokens_per_doc: int = Field(
        default=0,
        description=(
            "Maximum number of tokens per document. Documents longer than "
            "this will be truncated to this length. 0 means no "
            "document-level truncation is applied (only truncate_prompt_tokens "
            "applies to the combined query+document)."
        ),
    )
    instruction: str | None = Field(
        default=None,
        description=(
            "Task instruction prepended to each scored pair via the chat "
            "template. Equivalent to passing "
            "chat_template_kwargs={'instruction': ...}."
        ),
    )
    chat_template_kwargs: dict[str, Any] | None = Field(
...
            max_output_tokens=0,
        )

    def to_pooling_params(self, task: PoolingTask = "classify"):
        return PoolingParams(
            task=task,
            use_activation=self.use_activation,
        )
```
**EN:** Class `ScoringRequestMixin` is defined here, extending `PoolingBasicRequestMixin`, `ClassifyRequestMixin`, and groups behavior through methods like `_merge_instruction_into_kwargs`, `build_tok_params`, `to_pooling_params`.
**CN:** 这里定义类 `ScoringRequestMixin`，其职责是封装相关状态与行为，并通过 `_merge_instruction_into_kwargs`、`build_tok_params`、`to_pooling_params` 等方法组织逻辑。

### Lines 84-86 — Class `ScoreDataRequest`
```python
class ScoreDataRequest(ScoringRequestMixin):
    data_1: ScoreInput | list[ScoreInput]
    data_2: ScoreInput | list[ScoreInput]
```
**EN:** Class `ScoreDataRequest` acts as a protocol/data model with about 2 field declarations and helper methods such as its inherited interface.
**CN:** 类 `ScoreDataRequest` 充当协议/数据模型，包含约 2 个字段声明，并提供 继承接口 等辅助方法。

### Lines 89-101 — Class `ScoreQueriesDocumentsRequest`
```python
class ScoreQueriesDocumentsRequest(ScoringRequestMixin):
    # --8<-- [start:score-request-params]
    queries: ScoreInput | list[ScoreInput]
    documents: ScoreInput | list[ScoreInput]
    # --8<-- [end:score-request-params]

    @property
    def data_1(self):
        return self.queries

    @property
    def data_2(self):
        return self.documents
```
**EN:** Class `ScoreQueriesDocumentsRequest` acts as a protocol/data model with about 2 field declarations and helper methods such as `data_1`, `data_2`.
**CN:** 类 `ScoreQueriesDocumentsRequest` 充当协议/数据模型，包含约 2 个字段声明，并提供 `data_1`、`data_2` 等辅助方法。

### Lines 104-114 — Class `ScoreQueriesItemsRequest`
```python
class ScoreQueriesItemsRequest(ScoringRequestMixin):
    queries: ScoreInput | list[ScoreInput]
    items: ScoreInput | list[ScoreInput]

    @property
    def data_1(self):
        return self.queries

    @property
    def data_2(self):
        return self.items
```
**EN:** Class `ScoreQueriesItemsRequest` acts as a protocol/data model with about 2 field declarations and helper methods such as `data_1`, `data_2`.
**CN:** 类 `ScoreQueriesItemsRequest` 充当协议/数据模型，包含约 2 个字段声明，并提供 `data_1`、`data_2` 等辅助方法。

### Lines 117-127 — Class `ScoreTextRequest`
```python
class ScoreTextRequest(ScoringRequestMixin):
    text_1: ScoreInput | list[ScoreInput]
    text_2: ScoreInput | list[ScoreInput]

    @property
    def data_1(self):
        return self.text_1

    @property
    def data_2(self):
        return self.text_2
```
**EN:** Class `ScoreTextRequest` acts as a protocol/data model with about 2 field declarations and helper methods such as `data_1`, `data_2`.
**CN:** 类 `ScoreTextRequest` 充当协议/数据模型，包含约 2 个字段声明，并提供 `data_1`、`data_2` 等辅助方法。

### Lines 130-135 — Module constants and state
```python
ScoreRequest: TypeAlias = (
    ScoreQueriesDocumentsRequest
    | ScoreQueriesItemsRequest
    | ScoreDataRequest
    | ScoreTextRequest
)
```
**EN:** This block initializes `ScoreRequest`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `ScoreRequest`，为后续逻辑准备模块级常量或共享状态。

### Lines 138-142 — Class `RerankRequest`
```python
class RerankRequest(ScoringRequestMixin):
    # --8<-- [start:rerank-request-params]
    query: ScoreInput
    documents: ScoreInput | list[ScoreInput]
    top_n: int = Field(default_factory=lambda: 0)
```
**EN:** Class `RerankRequest` acts as a protocol/data model with about 3 field declarations and helper methods such as its inherited interface.
**CN:** 类 `RerankRequest` 充当协议/数据模型，包含约 3 个字段声明，并提供 继承接口 等辅助方法。

### Lines 146-146 — Module constants and state
```python
ScoringRequest: TypeAlias = ScoreRequest | RerankRequest
```
**EN:** This block initializes `ScoringRequest`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `ScoringRequest`，为后续逻辑准备模块级常量或共享状态。

### Lines 149-151 — Class `RerankDocument`
```python
class RerankDocument(BaseModel):
    text: str | None = None
    multi_modal: list[ScoreContentPartParam] | None = None
```
**EN:** Class `RerankDocument` is defined here, extending `BaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `RerankDocument`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 154-157 — Class `RerankResult`
```python
class RerankResult(BaseModel):
    index: int
    document: RerankDocument
    relevance_score: float
```
**EN:** Class `RerankResult` is defined here, extending `BaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `RerankResult`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 160-162 — Class `RerankUsage`
```python
class RerankUsage(BaseModel):
    prompt_tokens: int
    total_tokens: int
```
**EN:** Class `RerankUsage` is defined here, extending `BaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `RerankUsage`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 165-169 — Class `RerankResponse`
```python
class RerankResponse(OpenAIBaseModel):
    id: str
    model: str
    usage: RerankUsage
    results: list[RerankResult]
```
**EN:** Class `RerankResponse` acts as a protocol/data model with about 4 field declarations and helper methods such as its inherited interface.
**CN:** 类 `RerankResponse` 充当协议/数据模型，包含约 4 个字段声明，并提供 继承接口 等辅助方法。

### Lines 172-175 — Class `ScoreResponseData`
```python
class ScoreResponseData(OpenAIBaseModel):
    index: int
    object: str = "score"
    score: float
```
**EN:** Class `ScoreResponseData` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `ScoreResponseData`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 178-184 — Class `ScoreResponse`
```python
class ScoreResponse(OpenAIBaseModel):
    id: str = Field(default_factory=lambda: f"embd-{random_uuid()}")
    object: str = "list"
    created: int = Field(default_factory=lambda: int(time.time()))
    model: str
    data: list[ScoreResponseData]
    usage: UsageInfo
```
**EN:** Class `ScoreResponse` acts as a protocol/data model with about 6 field declarations and helper methods such as its inherited interface.
**CN:** 类 `ScoreResponse` 充当协议/数据模型，包含约 6 个字段声明，并提供 继承接口 等辅助方法。

### Lines 187-187 — Module constants and state
```python
ScoringResponse: TypeAlias = RerankResponse | ScoreResponse
```
**EN:** This block initializes `ScoringResponse`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `ScoringResponse`，为后续逻辑准备模块级常量或共享状态。

## Key Concepts / 关键概念
- Schema validation with Pydantic / 使用 Pydantic 进行模式校验
- Structured request/response models / 结构化请求/响应模型
- Tokenization or token-level processing / 分词或 token 级处理
- Prompt or multimodal rendering / 提示词或多模态渲染
- Pooling task support / 池化任务支持
- Scoring/ranking logic / 评分/排序逻辑
- Pooling task workflows / 池化任务工作流

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `time`, `typing`
- **Third-party / 第三方**: `pydantic`
- **vLLM Internal / vLLM 内部**: `vllm`, `vllm.config`, `vllm.entrypoints.openai.engine.protocol`, `vllm.renderers`, `vllm.tasks`, `vllm.utils`, `..base.protocol`, `.typing`
