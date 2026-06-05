# protocol.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/pooling/embed/protocol.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Declares request/response schemas, protocol objects, and validation helpers. Scope: pooling embedding. / 声明请求/响应模式、协议对象与校验辅助逻辑。 作用域：池化 / 嵌入。

## Line-by-Line Analysis / 逐行分析
### Lines 3-7 — Module overview
```python
"""Embedding API protocol models for OpenAI and Cohere formats.

OpenAI: https://platform.openai.com/docs/api-reference/embeddings
Cohere: https://docs.cohere.com/reference/embed
"""
```
**EN:** The module header documents the main contract: Embedding API protocol models for OpenAI and Cohere formats.
**CN:** 模块开头的文档字符串说明了该文件的主要职责、使用边界或运行约束。

### Lines 9-28 — Imports and shared dependencies
```python
import builtins
import struct
import time
from collections.abc import Sequence
from typing import Literal, TypeAlias

import pybase64 as base64
from pydantic import BaseModel, Field

from vllm import PoolingParams
from vllm.entrypoints.openai.engine.protocol import OpenAIBaseModel, UsageInfo
from vllm.utils import random_uuid

from ..base.protocol import (
    ChatRequestMixin,
    CompletionRequestMixin,
    EmbeddingTokenizeParamsMixin,
    EmbedRequestMixin,
    PoolingBasicRequestMixin,
)
```
**EN:** This import block pulls in standard-library modules such as `builtins`, `struct`, `time`, `collections`, `typing`, uses third-party packages like `pybase64`, `pydantic`, depends on internal helpers such as `vllm`, `vllm.entrypoints.openai.engine.protocol`, `vllm.utils`, `..base.protocol`.
**CN:** 该导入块引入 `builtins`, `struct`, `time`, `collections`, `typing` 等标准库模块，使用 `pybase64`, `pydantic` 等第三方库，依赖 `vllm`, `vllm.entrypoints.openai.engine.protocol`, `vllm.utils`, `..base.protocol` 等 vLLM 内部模块。

### Lines 31-42 — Class `EmbeddingCompletionRequest`
```python
class EmbeddingCompletionRequest(
    PoolingBasicRequestMixin,
    CompletionRequestMixin,
    EmbedRequestMixin,
    EmbeddingTokenizeParamsMixin,
):
    def to_pooling_params(self):
        return PoolingParams(
            task="embed",
            dimensions=self.dimensions,
            use_activation=self.use_activation,
        )
```
**EN:** Class `EmbeddingCompletionRequest` acts as a protocol/data model with about 0 field declarations and helper methods such as `to_pooling_params`.
**CN:** 类 `EmbeddingCompletionRequest` 充当协议/数据模型，包含约 0 个字段声明，并提供 `to_pooling_params` 等辅助方法。

### Lines 45-56 — Class `EmbeddingChatRequest`
```python
class EmbeddingChatRequest(
    PoolingBasicRequestMixin,
    ChatRequestMixin,
    EmbedRequestMixin,
    EmbeddingTokenizeParamsMixin,
):
    def to_pooling_params(self):
        return PoolingParams(
            task="embed",
            dimensions=self.dimensions,
            use_activation=self.use_activation,
        )
```
**EN:** Class `EmbeddingChatRequest` acts as a protocol/data model with about 0 field declarations and helper methods such as `to_pooling_params`.
**CN:** 类 `EmbeddingChatRequest` 充当协议/数据模型，包含约 0 个字段声明，并提供 `to_pooling_params` 等辅助方法。

### Lines 59-59 — Module constants and state
```python
EmbeddingRequest: TypeAlias = EmbeddingCompletionRequest | EmbeddingChatRequest
```
**EN:** This block initializes `EmbeddingRequest`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `EmbeddingRequest`，为后续逻辑准备模块级常量或共享状态。

### Lines 67-70 — Class `EmbeddingResponseData`
```python
class EmbeddingResponseData(OpenAIBaseModel):
    index: int
    object: str = "embedding"
    embedding: list[float] | str
```
**EN:** Class `EmbeddingResponseData` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `EmbeddingResponseData`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 73-79 — Class `EmbeddingResponse`
```python
class EmbeddingResponse(OpenAIBaseModel):
    id: str = Field(default_factory=lambda: f"embd-{random_uuid()}")
    object: str = "list"
    created: int = Field(default_factory=lambda: int(time.time()))
    model: str | None = None
    data: list[EmbeddingResponseData]
    usage: UsageInfo
```
**EN:** Class `EmbeddingResponse` acts as a protocol/data model with about 6 field declarations and helper methods such as its inherited interface.
**CN:** 类 `EmbeddingResponse` 充当协议/数据模型，包含约 6 个字段声明，并提供 继承接口 等辅助方法。

### Lines 82-85 — Class `EmbeddingBytesResponse`
```python
class EmbeddingBytesResponse(OpenAIBaseModel):
    content: list[bytes]
    headers: dict[str, str] | None = None
    media_type: str = "application/octet-stream"
```
**EN:** Class `EmbeddingBytesResponse` acts as a protocol/data model with about 3 field declarations and helper methods such as its inherited interface.
**CN:** 类 `EmbeddingBytesResponse` 充当协议/数据模型，包含约 3 个字段声明，并提供 继承接口 等辅助方法。

### Lines 92-98 — Module constants and state
```python
CohereEmbeddingType = Literal[
    "float",
    "binary",
    "ubinary",
    "base64",
]
CohereTruncate = Literal["NONE", "START", "END"]
```
**EN:** This block initializes `CohereEmbeddingType`, `CohereTruncate`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `CohereEmbeddingType`, `CohereTruncate`，为后续逻辑准备模块级常量或共享状态。

### Lines 101-104 — Class `CohereEmbedContent`
```python
class CohereEmbedContent(BaseModel):
    type: Literal["text", "image_url"]
    text: str | None = None
    image_url: dict[str, str] | None = None
```
**EN:** Class `CohereEmbedContent` is defined here, extending `BaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `CohereEmbedContent`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 107-108 — Class `CohereEmbedInput`
```python
class CohereEmbedInput(BaseModel):
    content: list[CohereEmbedContent]
```
**EN:** Class `CohereEmbedInput` is defined here, extending `BaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `CohereEmbedInput`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 111-121 — Class `CohereEmbedRequest`
```python
class CohereEmbedRequest(BaseModel):
    model: str | None = None
    input_type: str | None = None
    texts: list[str] | None = None
    images: list[str] | None = None
    inputs: list[CohereEmbedInput] | None = None
    output_dimension: int | None = None
    embedding_types: list[CohereEmbeddingType] | None = None
    truncate: CohereTruncate = "END"
    max_tokens: int | None = None
    priority: int = 0
```
**EN:** Class `CohereEmbedRequest` acts as a protocol/data model with about 10 field declarations and helper methods such as its inherited interface.
**CN:** 类 `CohereEmbedRequest` 充当协议/数据模型，包含约 10 个字段声明，并提供 继承接口 等辅助方法。

### Lines 129-130 — Class `CohereApiVersion`
```python
class CohereApiVersion(BaseModel):
    version: str = "2"
```
**EN:** Class `CohereApiVersion` is defined here, extending `BaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `CohereApiVersion`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 133-135 — Class `CohereBilledUnits`
```python
class CohereBilledUnits(BaseModel):
    input_tokens: int | None = None
    image_tokens: int | None = None
```
**EN:** Class `CohereBilledUnits` is defined here, extending `BaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `CohereBilledUnits`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 138-140 — Class `CohereMeta`
```python
class CohereMeta(BaseModel):
    api_version: CohereApiVersion = Field(default_factory=CohereApiVersion)
    billed_units: CohereBilledUnits | None = None
```
**EN:** Class `CohereMeta` is defined here, extending `BaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `CohereMeta`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 143-149 — Class `CohereEmbedByTypeEmbeddings`
```python
class CohereEmbedByTypeEmbeddings(BaseModel):
    # The field name ``float`` shadows the builtin type, so the annotation
    # must use ``builtins.float`` to avoid a self-referential type error.
    float: list[list[builtins.float]] | None = None
    binary: list[list[int]] | None = None
    ubinary: list[list[int]] | None = None
    base64: list[str] | None = None
```
**EN:** Class `CohereEmbedByTypeEmbeddings` is defined here, extending `BaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `CohereEmbedByTypeEmbeddings`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 152-157 — Class `CohereEmbedResponse`
```python
class CohereEmbedResponse(BaseModel):
    id: str = Field(default_factory=lambda: f"embd-{random_uuid()}")
    embeddings: CohereEmbedByTypeEmbeddings
    texts: list[str] | None = None
    meta: CohereMeta | None = None
    response_type: Literal["embeddings_by_type"] = "embeddings_by_type"
```
**EN:** Class `CohereEmbedResponse` acts as a protocol/data model with about 5 field declarations and helper methods such as its inherited interface.
**CN:** 类 `CohereEmbedResponse` 充当协议/数据模型，包含约 5 个字段声明，并提供 继承接口 等辅助方法。

### Lines 164-164 — Module constants
```python
_UNSIGNED_TO_SIGNED_DIFF = 1 << 7  # 128
```
**EN:** This block initializes `_UNSIGNED_TO_SIGNED_DIFF`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `_UNSIGNED_TO_SIGNED_DIFF`，为后续逻辑准备模块级常量或共享状态。

### Lines 167-197 — Function `_pack_binary_embeddings`
```python
def _pack_binary_embeddings(
    float_embeddings: list[list[float]],
    signed: bool,
) -> list[list[int]]:
    """Bit-pack float embeddings: positive -> 1, negative -> 0.

    Each bit is shifted left by ``7 - idx%8``, and every 8 bits are packed
    into one byte.
    """
    result: list[list[int]] = []
    for embedding in float_embeddings:
        dim = len(embedding)
        if dim % 8 != 0:
            raise ValueError(
                "Embedding dimension must be a multiple of 8 for binary "
                f"embedding types, but got {dim}."
            )
        packed_len = dim // 8
        packed: list[int] = []
        byte_val = 0
        for idx, value in enumerate(embedding):
            bit = 1 if value >= 0 else 0
            byte_val += bit << (7 - idx % 8)
            if (idx + 1) % 8 == 0:
                if signed:
                    byte_val -= _UNSIGNED_TO_SIGNED_DIFF
                packed.append(byte_val)
                byte_val = 0
        assert len(packed) == packed_len
        result.append(packed)
    return result
```
**EN:** This function `_pack_binary_embeddings` is documented as: Bit-pack float embeddings: positive -> 1, negative -> 0.
**CN:** 这里定义函数 `_pack_binary_embeddings`，其文档字符串说明了主要职责与调用约定。

### Lines 200-208 — Function `_encode_base64_embeddings`
```python
def _encode_base64_embeddings(
    float_embeddings: list[list[float]],
) -> list[str]:
    """Encode float embeddings as base64 (little-endian float32)."""
    result: list[str] = []
    for embedding in float_embeddings:
        buf = struct.pack(f"<{len(embedding)}f", *embedding)
        result.append(base64.b64encode(buf).decode("utf-8"))
    return result
```
**EN:** This function `_encode_base64_embeddings` is documented as: Encode float embeddings as base64 (little-endian float32).
**CN:** 这里定义函数 `_encode_base64_embeddings`，其文档字符串说明了主要职责与调用约定。

### Lines 211-228 — Function `build_typed_embeddings`
```python
def build_typed_embeddings(
    float_embeddings: list[list[float]],
    embedding_types: Sequence[str],
) -> CohereEmbedByTypeEmbeddings:
    """Convert float embeddings to all requested Cohere embedding types."""
    result = CohereEmbedByTypeEmbeddings()

    for emb_type in embedding_types:
        if emb_type == "float":
            result.float = float_embeddings
        elif emb_type == "binary":
            result.binary = _pack_binary_embeddings(float_embeddings, signed=True)
        elif emb_type == "ubinary":
            result.ubinary = _pack_binary_embeddings(float_embeddings, signed=False)
        elif emb_type == "base64":
            result.base64 = _encode_base64_embeddings(float_embeddings)

    return result
```
**EN:** This function `build_typed_embeddings` is documented as: Convert float embeddings to all requested Cohere embedding types.
**CN:** 这里定义函数 `build_typed_embeddings`，其文档字符串说明了主要职责与调用约定。

## Key Concepts / 关键概念
- Schema validation with Pydantic / 使用 Pydantic 进行模式校验
- Structured request/response models / 结构化请求/响应模型
- Tokenization or token-level processing / 分词或 token 级处理
- Pooling task support / 池化任务支持
- Embedding generation / 嵌入生成
- Pooling task workflows / 池化任务工作流

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `builtins`, `struct`, `time`, `collections`, `typing`
- **Third-party / 第三方**: `pybase64`, `pydantic`
- **vLLM Internal / vLLM 内部**: `vllm`, `vllm.entrypoints.openai.engine.protocol`, `vllm.utils`, `..base.protocol`
