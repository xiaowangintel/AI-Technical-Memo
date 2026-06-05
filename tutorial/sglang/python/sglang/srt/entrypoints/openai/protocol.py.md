# protocol.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/openai/protocol.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements protocol logic for the OpenAI-compatible serving layer. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 protocol 相关的逻辑，并服务于 OpenAI 兼容服务层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Provide supporting module logic / 提供辅助模块逻辑
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
"""Pydantic models for OpenAI API protocol"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 16-30: Provide supporting module logic / 提供辅助模块逻辑
```python
import logging
import time
import uuid
from dataclasses import dataclass
from typing import (
    Any,
    Dict,
    List,
    NamedTuple,
    Optional,
    Tuple,
    TypeAlias,
    Union,
    get_args,
)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 32-50: Provide supporting module logic / 提供辅助模块逻辑
```python
from openai.types.responses import (
    ResponseFunctionToolCall,
    ResponseInputItemParam,
    ResponseOutputItem,
    ResponseOutputMessage,
    ResponseOutputText,
    ResponseReasoningItem,
)
from openai.types.responses.response import ToolChoice
from openai.types.responses.tool import Tool
from pydantic import (
    BaseModel,
    ConfigDict,
    Field,
    field_validator,
    model_serializer,
    model_validator,
)
from typing_extensions import Literal
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 52-55: Provide supporting module logic / 提供辅助模块逻辑
```python
try:
    from xgrammar import StructuralTag
except:
    StructuralTag = Any
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 57-57: Import runtime dependencies / 导入运行时依赖
```python
from sglang.utils import convert_json_schema_to_str
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 59-59: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 61-61: Provide supporting module logic / 提供辅助模块逻辑
```python
DEFAULT_MODEL_NAME = "default"
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 64-73: Define class ModelCard / 定义类 ModelCard
```python
class ModelCard(BaseModel):
    """Model cards."""

    id: str
    object: str = "model"
    created: int = Field(default_factory=lambda: int(time.time()))
    owned_by: str = "sglang"
    root: Optional[str] = None
    parent: Optional[str] = None
    max_model_len: Optional[int] = None
```
**EN:** This block declares the class `ModelCard`. It centers on Model cards..
**CN:** 该代码块声明类 `ModelCard`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 76-80: Define class ModelList / 定义类 ModelList
```python
class ModelList(BaseModel):
    """Model list consists of model cards."""

    object: str = "list"
    data: List[ModelCard] = Field(default_factory=list)
```
**EN:** This block declares the class `ModelList`. It centers on Model list consists of model cards..
**CN:** 该代码块声明类 `ModelList`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 83-88: Define class ErrorResponse / 定义类 ErrorResponse
```python
class ErrorResponse(BaseModel):
    object: str = "error"
    message: str
    type: str
    param: Optional[str] = None
    code: int
```
**EN:** This block declares the class `ErrorResponse`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ErrorResponse`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 91-95: Define class LogProbs / 定义类 LogProbs
```python
class LogProbs(BaseModel):
    text_offset: List[int] = Field(default_factory=list)
    token_logprobs: List[Optional[float]] = Field(default_factory=list)
    tokens: List[str] = Field(default_factory=list)
    top_logprobs: List[Optional[Dict[str, float]]] = Field(default_factory=list)
```
**EN:** This block declares the class `LogProbs`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `LogProbs`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 98-101: Define class TopLogprob / 定义类 TopLogprob
```python
class TopLogprob(BaseModel):
    token: str
    bytes: List[int]
    logprob: float
```
**EN:** This block declares the class `TopLogprob`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `TopLogprob`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 104-108: Define class ChatCompletionTokenLogprob / 定义类 ChatCompletionTokenLogprob
```python
class ChatCompletionTokenLogprob(BaseModel):
    token: str
    bytes: List[int]
    logprob: float
    top_logprobs: List[TopLogprob]
```
**EN:** This block declares the class `ChatCompletionTokenLogprob`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ChatCompletionTokenLogprob`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 111-113: Define class ChoiceLogprobs / 定义类 ChoiceLogprobs
```python
class ChoiceLogprobs(BaseModel):
    # build for v1/chat/completions response
    content: List[ChatCompletionTokenLogprob]
```
**EN:** This block declares the class `ChoiceLogprobs`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ChoiceLogprobs`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 116-133: Define class CachedTokensDetails / 定义类 CachedTokensDetails
```python
class CachedTokensDetails(BaseModel):
    """Detailed breakdown of cached tokens by cache source."""

    device: int = 0  # Tokens from device cache (GPU)
    host: int = 0  # Tokens from host cache (CPU memory)
    # L3 storage fields are only present when storage backend is enabled
    storage: Optional[int] = None  # Tokens from L3 storage backend
    storage_backend: Optional[str] = None  # Type of storage backend used

    @model_serializer(mode="wrap")
    def _serialize(self, handler):
        data = handler(self)
        # Remove None fields so they don't appear in response when L3 is disabled
        if self.storage is None:
            data.pop("storage", None)
        if self.storage_backend is None:
            data.pop("storage_backend", None)
        return data
```
**EN:** This block declares the class `CachedTokensDetails`. It centers on Detailed breakdown of cached tokens by cache source., with methods such as _serialize.
**CN:** 该代码块声明类 `CachedTokensDetails`。它负责承载与 protocol 相关的核心状态与行为，并通过 _serialize 等方法组织实现。

### Lines 136-139: Define class PromptTokensDetails / 定义类 PromptTokensDetails
```python
class PromptTokensDetails(BaseModel):
    """Details about prompt tokens."""

    cached_tokens: int = 0
```
**EN:** This block declares the class `PromptTokensDetails`. It centers on Details about prompt tokens..
**CN:** 该代码块声明类 `PromptTokensDetails`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 142-148: Define class UsageInfo / 定义类 UsageInfo
```python
class UsageInfo(BaseModel):
    prompt_tokens: int = 0
    total_tokens: int = 0
    completion_tokens: Optional[int] = 0
    # Used to return cached tokens info when --enable-cache-report is set
    prompt_tokens_details: Optional[PromptTokensDetails] = None
    reasoning_tokens: Optional[int] = 0
```
**EN:** This block declares the class `UsageInfo`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `UsageInfo`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 151-153: Define class StreamOptions / 定义类 StreamOptions
```python
class StreamOptions(BaseModel):
    include_usage: Optional[bool] = False
    continuous_usage_stats: Optional[bool] = False
```
**EN:** This block declares the class `StreamOptions`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `StreamOptions`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 156-161: Define class JsonSchemaResponseFormat / 定义类 JsonSchemaResponseFormat
```python
class JsonSchemaResponseFormat(BaseModel):
    name: str
    description: Optional[str] = None
    # use alias to workaround pydantic conflict
    schema_: Optional[Dict[str, object]] = Field(alias="schema", default=None)
    strict: Optional[bool] = False
```
**EN:** This block declares the class `JsonSchemaResponseFormat`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `JsonSchemaResponseFormat`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 164-166: Define class ResponseFormat / 定义类 ResponseFormat
```python
class ResponseFormat(BaseModel):
    type: Literal["text", "json_object", "json_schema"]
    json_schema: Optional[JsonSchemaResponseFormat] = None
```
**EN:** This block declares the class `ResponseFormat`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ResponseFormat`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 169-172: Define class StructuresResponseFormat / 定义类 StructuresResponseFormat
```python
class StructuresResponseFormat(BaseModel):
    begin: str
    schema_: Optional[Dict[str, object]] = Field(alias="schema", default=None)
    end: str
```
**EN:** This block declares the class `StructuresResponseFormat`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `StructuresResponseFormat`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 173-175: Import runtime dependencies / 导入运行时依赖
```python


# NOTE(dark): keep this for backward compatibility
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 176-180: Define class LegacyStructuralTagResponseFormat / 定义类 LegacyStructuralTagResponseFormat
```python
class LegacyStructuralTagResponseFormat(BaseModel):
    type: Literal["structural_tag"]
    structures: List[StructuresResponseFormat]
    triggers: List[str]
    at_least_one: bool = False
```
**EN:** This block declares the class `LegacyStructuralTagResponseFormat`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `LegacyStructuralTagResponseFormat`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 183-185: Provide supporting module logic / 提供辅助模块逻辑
```python
StructuralTagResponseFormat: TypeAlias = Union[
    LegacyStructuralTagResponseFormat, StructuralTag
]
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 187-190: Provide supporting module logic / 提供辅助模块逻辑
```python
ToolCallConstraint: TypeAlias = Union[
    Tuple[Literal["structural_tag"], StructuralTagResponseFormat],
    Tuple[Literal["json_schema"], Any],  # json_schema can be dict/str/None
]
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 193-198: Define class FileRequest / 定义类 FileRequest
```python
class FileRequest(BaseModel):
    # https://platform.openai.com/docs/api-reference/files/create
    file: bytes  # The File object (not file name) to be uploaded
    purpose: str = (
        "batch"  # The intended purpose of the uploaded file, default is "batch"
    )
```
**EN:** This block declares the class `FileRequest`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `FileRequest`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 201-207: Define class FileResponse / 定义类 FileResponse
```python
class FileResponse(BaseModel):
    id: str
    object: str = "file"
    bytes: int
    created_at: int
    filename: str
    purpose: str
```
**EN:** This block declares the class `FileResponse`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `FileResponse`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 210-213: Define class FileDeleteResponse / 定义类 FileDeleteResponse
```python
class FileDeleteResponse(BaseModel):
    id: str
    object: str = "file"
    deleted: bool
```
**EN:** This block declares the class `FileDeleteResponse`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `FileDeleteResponse`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 216-222: Define class BatchRequest / 定义类 BatchRequest
```python
class BatchRequest(BaseModel):
    input_file_id: (
        str  # The ID of an uploaded file that contains requests for the new batch
    )
    endpoint: str  # The endpoint to be used for all requests in the batch
    completion_window: str  # The time frame within which the batch should be processed
    metadata: Optional[dict] = None  # Optional custom metadata for the batch
```
**EN:** This block declares the class `BatchRequest`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `BatchRequest`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 225-245: Define class BatchResponse / 定义类 BatchResponse
```python
class BatchResponse(BaseModel):
    id: str
    object: str = "batch"
    endpoint: str
    errors: Optional[dict] = None
    input_file_id: str
    completion_window: str
    status: str = "validating"
    output_file_id: Optional[str] = None
    error_file_id: Optional[str] = None
    created_at: int
    in_progress_at: Optional[int] = None
    expires_at: Optional[int] = None
    finalizing_at: Optional[int] = None
    completed_at: Optional[int] = None
    failed_at: Optional[int] = None
    expired_at: Optional[int] = None
    cancelling_at: Optional[int] = None
    cancelled_at: Optional[int] = None
    request_counts: Optional[dict] = None
    metadata: Optional[dict] = None
```
**EN:** This block declares the class `BatchResponse`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `BatchResponse`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 248-259: Implement migrate deprecated dp rank / 实现migrate deprecated dp rank
```python
def _migrate_deprecated_dp_rank(values: dict) -> dict:
    if isinstance(values, dict) and values.get("data_parallel_rank") is not None:
        import warnings

        warnings.warn(
            "'data_parallel_rank' is deprecated, use 'routed_dp_rank' instead.",
            DeprecationWarning,
            stacklevel=2,
        )
        if values.get("routed_dp_rank") is None:
            values["routed_dp_rank"] = values["data_parallel_rank"]
    return values
```
**EN:** This block implements the function `_migrate_deprecated_dp_rank(values)`. It focuses on handling the protocol responsibilities represented by `_migrate_deprecated_dp_rank`, providing reusable behavior for the protocol pipeline.
**CN:** 该代码块实现函数 `_migrate_deprecated_dp_rank(values)`。它围绕 `_migrate_deprecated_dp_rank` 所承担的 protocol 相关职责展开，为对应处理链路提供可复用能力。

### Lines 262-344: Define class CompletionRequest / 定义类 CompletionRequest
```python
class CompletionRequest(BaseModel):
    # Ordered by official OpenAI API documentation
    # https://platform.openai.com/docs/api-reference/completions/create
    model: str = Field(
        default=DEFAULT_MODEL_NAME,
        description="Model name. Supports LoRA adapters via 'base-model:adapter-name' syntax.",
    )
    prompt: Union[List[int], List[List[int]], str, List[str]]
    best_of: Optional[int] = None
    echo: bool = False
    frequency_penalty: float = 0.0
    logit_bias: Optional[Dict[str, float]] = None
    logprobs: Optional[int] = None
    max_tokens: int = 16
    n: int = 1
    presence_penalty: float = 0.0
    seed: Optional[int] = None
    stop: Optional[Union[str, List[str]]] = None
    stream: bool = False
    stream_options: Optional[StreamOptions] = None
    suffix: Optional[str] = None
    temperature: float = 1.0
    top_p: float = 1.0
    user: Optional[str] = None
    return_hidden_states: bool = False
    return_routed_experts: bool = False
    routed_experts_start_len: int = 0
    return_cached_tokens_details: bool = False

    # Extra parameters for SRT backend only and will be ignored by OpenAI models.
    top_k: int = -1
    min_p: float = 0.0
    min_tokens: int = 0
    json_schema: Optional[str] = None
    regex: Optional[str] = None
    ebnf: Optional[str] = None
    repetition_penalty: float = 1.0
    stop_token_ids: Optional[List[int]] = None
    stop_regex: Optional[Union[str, List[str]]] = None
    no_stop_trim: bool = False
    ignore_eos: bool = False
    skip_special_tokens: bool = True
    lora_path: Optional[Union[List[Optional[str]], Optional[str]]] = None
    session_params: Optional[Dict] = None
    response_format: Optional[Union[ResponseFormat, StructuralTagResponseFormat]] = None
    custom_params: Optional[Dict] = None
    custom_logit_processor: Optional[str] = None

    # For PD disaggregation
    bootstrap_host: Optional[Union[List[str], str]] = None
    bootstrap_port: Optional[Union[List[Optional[int]], int]] = None
    bootstrap_room: Optional[Union[List[int], int]] = None

    # For DP routing — external router assigns a specific DP worker
    routed_dp_rank: Optional[int] = None
    # For PD disagg — hint telling decode which prefill DP worker has the KV cache
    disagg_prefill_dp_rank: Optional[int] = None
    # Deprecated: use routed_dp_rank instead
    data_parallel_rank: Optional[int] = None

    # For request id
    rid: Optional[Union[List[str], str]] = None
    # Extra key for classifying the request (e.g. cache_salt)
    extra_key: Optional[Union[List[str], str]] = None
    # Cache salt for request caching
    cache_salt: Optional[Union[List[str], str]] = None
    # Priority for the request
    priority: Optional[int] = None

    # For custom metric labels
    custom_labels: Optional[Dict[str, str]] = None

    @model_validator(mode="before")
    @classmethod
    def _handle_deprecated_dp_rank(cls, values):
        return _migrate_deprecated_dp_rank(values)

    @field_validator("max_tokens")
    @classmethod
    def validate_max_tokens_positive(cls, v):
        if v is not None and v <= 0:
            raise ValueError("max_tokens must be positive")
        return v
```
**EN:** This block declares the class `CompletionRequest`. It centers on coordinating protocol behavior, with methods such as _handle_deprecated_dp_rank, validate_max_tokens_positive.
**CN:** 该代码块声明类 `CompletionRequest`。它负责承载与 protocol 相关的核心状态与行为，并通过 _handle_deprecated_dp_rank, validate_max_tokens_positive 等方法组织实现。

### Lines 347-361: Define class SglExt / 定义类 SglExt
```python
class SglExt(BaseModel):
    """SGLang extension fields for OpenAI-compatible responses.

    Future SGLang-specific extensions to OpenAI-compatible response objects
    should be added as fields here rather than directly on the choice object.
    """

    routed_experts: Optional[str] = None
    cached_tokens_details: Optional[CachedTokensDetails] = None

    @model_serializer(mode="wrap")
    def _serialize(self, handler):
        data = handler(self)
        # Remove None fields to keep response clean
        return {k: v for k, v in data.items() if v is not None}
```
**EN:** This block declares the class `SglExt`. It centers on SGLang extension fields for OpenAI-compatible responses., with methods such as _serialize.
**CN:** 该代码块声明类 `SglExt`。它负责承载与 protocol 相关的核心状态与行为，并通过 _serialize 等方法组织实现。

### Lines 364-377: Define class CompletionResponseChoice / 定义类 CompletionResponseChoice
```python
class CompletionResponseChoice(BaseModel):
    index: int
    text: str
    logprobs: Optional[LogProbs] = None
    finish_reason: Optional[Literal["stop", "length", "content_filter", "abort"]] = None
    matched_stop: Union[None, int, str] = None
    hidden_states: Optional[object] = None

    @model_serializer(mode="wrap")
    def _serialize(self, handler):
        data = handler(self)
        if self.hidden_states is None:
            data.pop("hidden_states", None)
        return data
```
**EN:** This block declares the class `CompletionResponseChoice`. It centers on coordinating protocol behavior, with methods such as _serialize.
**CN:** 该代码块声明类 `CompletionResponseChoice`。它负责承载与 protocol 相关的核心状态与行为，并通过 _serialize 等方法组织实现。

### Lines 380-395: Define class CompletionResponse / 定义类 CompletionResponse
```python
class CompletionResponse(BaseModel):
    id: str
    object: str = "text_completion"
    created: int = Field(default_factory=lambda: int(time.time()))
    model: str
    choices: List[CompletionResponseChoice]
    usage: UsageInfo
    metadata: Optional[Dict[str, Any]] = None
    sglext: Optional[SglExt] = None

    @model_serializer(mode="wrap")
    def _serialize(self, handler):
        data = handler(self)
        if self.sglext is None:
            data.pop("sglext", None)
        return data
```
**EN:** This block declares the class `CompletionResponse`. It centers on coordinating protocol behavior, with methods such as _serialize.
**CN:** 该代码块声明类 `CompletionResponse`。它负责承载与 protocol 相关的核心状态与行为，并通过 _serialize 等方法组织实现。

### Lines 398-411: Define class CompletionResponseStreamChoice / 定义类 CompletionResponseStreamChoice
```python
class CompletionResponseStreamChoice(BaseModel):
    index: int
    text: str
    logprobs: Optional[LogProbs] = None
    finish_reason: Optional[Literal["stop", "length", "content_filter", "abort"]] = None
    matched_stop: Union[None, int, str] = None
    hidden_states: Optional[object] = None

    @model_serializer(mode="wrap")
    def _serialize(self, handler):
        data = handler(self)
        if self.hidden_states is None:
            data.pop("hidden_states", None)
        return data
```
**EN:** This block declares the class `CompletionResponseStreamChoice`. It centers on coordinating protocol behavior, with methods such as _serialize.
**CN:** 该代码块声明类 `CompletionResponseStreamChoice`。它负责承载与 protocol 相关的核心状态与行为，并通过 _serialize 等方法组织实现。

### Lines 414-428: Define class CompletionStreamResponse / 定义类 CompletionStreamResponse
```python
class CompletionStreamResponse(BaseModel):
    id: str
    object: str = "text_completion"
    created: int = Field(default_factory=lambda: int(time.time()))
    model: str
    choices: List[CompletionResponseStreamChoice]
    usage: Optional[UsageInfo] = None
    sglext: Optional[SglExt] = None

    @model_serializer(mode="wrap")
    def _serialize(self, handler):
        data = handler(self)
        if self.sglext is None:
            data.pop("sglext", None)
        return data
```
**EN:** This block declares the class `CompletionStreamResponse`. It centers on coordinating protocol behavior, with methods such as _serialize.
**CN:** 该代码块声明类 `CompletionStreamResponse`。它负责承载与 protocol 相关的核心状态与行为，并通过 _serialize 等方法组织实现。

### Lines 431-433: Define class ChatCompletionMessageContentTextPart / 定义类 ChatCompletionMessageContentTextPart
```python
class ChatCompletionMessageContentTextPart(BaseModel):
    type: Literal["text"]
    text: str
```
**EN:** This block declares the class `ChatCompletionMessageContentTextPart`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ChatCompletionMessageContentTextPart`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 436-440: Define class ChatCompletionMessageContentImageURL / 定义类 ChatCompletionMessageContentImageURL
```python
class ChatCompletionMessageContentImageURL(BaseModel):
    url: str
    detail: Optional[Literal["auto", "low", "high"]] = "auto"
    max_dynamic_patch: Optional[int] = None
    min_dynamic_patch: Optional[int] = None
```
**EN:** This block declares the class `ChatCompletionMessageContentImageURL`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ChatCompletionMessageContentImageURL`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 443-446: Define class ChatCompletionMessageContentVideoURL / 定义类 ChatCompletionMessageContentVideoURL
```python
class ChatCompletionMessageContentVideoURL(BaseModel):
    url: str
    max_dynamic_patch: Optional[int] = None
    min_dynamic_patch: Optional[int] = None
```
**EN:** This block declares the class `ChatCompletionMessageContentVideoURL`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ChatCompletionMessageContentVideoURL`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 449-450: Define class ChatCompletionMessageContentAudioURL / 定义类 ChatCompletionMessageContentAudioURL
```python
class ChatCompletionMessageContentAudioURL(BaseModel):
    url: str
```
**EN:** This block declares the class `ChatCompletionMessageContentAudioURL`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ChatCompletionMessageContentAudioURL`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 453-456: Define class ChatCompletionMessageContentImagePart / 定义类 ChatCompletionMessageContentImagePart
```python
class ChatCompletionMessageContentImagePart(BaseModel):
    type: Literal["image_url"]
    image_url: ChatCompletionMessageContentImageURL
    modalities: Optional[Literal["image", "multi-images", "video"]] = "image"
```
**EN:** This block declares the class `ChatCompletionMessageContentImagePart`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ChatCompletionMessageContentImagePart`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 459-461: Define class ChatCompletionMessageContentVideoPart / 定义类 ChatCompletionMessageContentVideoPart
```python
class ChatCompletionMessageContentVideoPart(BaseModel):
    type: Literal["video_url"]
    video_url: ChatCompletionMessageContentVideoURL
```
**EN:** This block declares the class `ChatCompletionMessageContentVideoPart`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ChatCompletionMessageContentVideoPart`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 464-466: Define class ChatCompletionMessageContentAudioPart / 定义类 ChatCompletionMessageContentAudioPart
```python
class ChatCompletionMessageContentAudioPart(BaseModel):
    type: Literal["audio_url"]
    audio_url: ChatCompletionMessageContentAudioURL
```
**EN:** This block declares the class `ChatCompletionMessageContentAudioPart`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ChatCompletionMessageContentAudioPart`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 469-477: Define class ChatCompletionMessageContentToolReferenceBlock / 定义类 ChatCompletionMessageContentToolReferenceBlock
```python
class ChatCompletionMessageContentToolReferenceBlock(BaseModel):
    # GLM-specific extension used alongside `defer_loading` tools. The chat
    # template looks up `tools[*].function.name == tr.name` and renders the
    # referenced tool schemas inline for the current turn. Not part of any
    # OpenAI API; included here so Pydantic accepts the content through the
    # Chat Completions path (the Anthropic endpoint translates its
    # `tool_name` field to `name` before forwarding).
    type: Literal["tool_reference"]
    name: str
```
**EN:** This block declares the class `ChatCompletionMessageContentToolReferenceBlock`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ChatCompletionMessageContentToolReferenceBlock`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 480-495: Provide supporting module logic / 提供辅助模块逻辑
```python
ChatCompletionMessageContentPart = Union[
    ChatCompletionMessageContentTextPart,
    ChatCompletionMessageContentImagePart,
    ChatCompletionMessageContentVideoPart,
    ChatCompletionMessageContentAudioPart,
    ChatCompletionMessageContentToolReferenceBlock,
]

# Rerank content types for multimodal reranking (e.g., Qwen3-VL-Reranker)
# Can be a simple string (text-only) or a list of multimodal content parts
RerankContentPart = Union[
    ChatCompletionMessageContentTextPart,
    ChatCompletionMessageContentImagePart,
    ChatCompletionMessageContentVideoPart,
]
RerankContent = Union[str, List[RerankContentPart]]
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 498-502: Define class FunctionResponse / 定义类 FunctionResponse
```python
class FunctionResponse(BaseModel):
    """Function response."""

    name: Optional[str] = None
    arguments: Optional[str | Dict[str, Any]] = None
```
**EN:** This block declares the class `FunctionResponse`. It centers on Function response..
**CN:** 该代码块声明类 `FunctionResponse`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 505-511: Define class ToolCall / 定义类 ToolCall
```python
class ToolCall(BaseModel):
    """Tool call response."""

    id: Optional[str] = None
    index: Optional[int] = None
    type: Literal["function"] = "function"
    function: FunctionResponse
```
**EN:** This block declares the class `ToolCall`. It centers on Tool call response..
**CN:** 该代码块声明类 `ToolCall`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 514-517: Provide supporting module logic / 提供辅助模块逻辑
```python
_GenericMessageRole = Literal[
    "system", "assistant", "tool", "function", "developer", "latest_reminder"
]
_GENERIC_MESSAGE_ROLES: Tuple[str, ...] = get_args(_GenericMessageRole)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 520-540: Define class ChatCompletionMessageGenericParam / 定义类 ChatCompletionMessageGenericParam
```python
class ChatCompletionMessageGenericParam(BaseModel):
    role: _GenericMessageRole
    content: Union[str, List[ChatCompletionMessageContentPart], None] = Field(
        default=None
    )
    tool_call_id: Optional[str] = None
    name: Optional[str] = None
    reasoning_content: Optional[str] = None
    tool_calls: Optional[List[ToolCall]] = Field(default=None, examples=[None])
    tools: Optional[List[Tool]] = Field(default=None, examples=[None])

    @field_validator("role", mode="before")
    @classmethod
    def _normalize_role(cls, v):
        if isinstance(v, str):
            v_lower = v.lower()
            if v_lower not in _GENERIC_MESSAGE_ROLES:
                allowed = ", ".join(repr(r) for r in _GENERIC_MESSAGE_ROLES)
                raise ValueError(f"'role' must be one of {allowed} (case-insensitive).")
            return v_lower
        raise ValueError("'role' must be a string")
```
**EN:** This block declares the class `ChatCompletionMessageGenericParam`. It centers on coordinating protocol behavior, with methods such as _normalize_role.
**CN:** 该代码块声明类 `ChatCompletionMessageGenericParam`。它负责承载与 protocol 相关的核心状态与行为，并通过 _normalize_role 等方法组织实现。

### Lines 543-545: Define class ChatCompletionMessageUserParam / 定义类 ChatCompletionMessageUserParam
```python
class ChatCompletionMessageUserParam(BaseModel):
    role: Literal["user"]
    content: Union[str, List[ChatCompletionMessageContentPart]]
```
**EN:** This block declares the class `ChatCompletionMessageUserParam`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ChatCompletionMessageUserParam`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 548-550: Provide supporting module logic / 提供辅助模块逻辑
```python
ChatCompletionMessageParam = Union[
    ChatCompletionMessageGenericParam, ChatCompletionMessageUserParam
]
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 553-567: Define class Function / 定义类 Function
```python
class Function(BaseModel):
    """Function descriptions."""

    description: Optional[str] = Field(default=None, examples=[None])
    name: str
    parameters: Optional[object] = None
    strict: bool = False
    defer_loading: Optional[bool] = None

    @model_serializer(mode="wrap")
    def _serialize(self, handler):
        data = handler(self)
        if self.defer_loading is None:
            data.pop("defer_loading", None)
        return data
```
**EN:** This block declares the class `Function`. It centers on Function descriptions., with methods such as _serialize.
**CN:** 该代码块声明类 `Function`。它负责承载与 protocol 相关的核心状态与行为，并通过 _serialize 等方法组织实现。

### Lines 570-581: Define class Tool / 定义类 Tool
```python
class Tool(BaseModel):
    """Function wrapper."""

    type: str = Field(default="function", examples=["function"])
    function: Function
    defer_loading: Optional[bool] = None

    @model_validator(mode="after")
    def _propagate_defer_loading(self) -> "Tool":
        if self.defer_loading is not None and self.function.defer_loading is None:
            self.function.defer_loading = self.defer_loading
        return self
```
**EN:** This block declares the class `Tool`. It centers on Function wrapper., with methods such as _propagate_defer_loading.
**CN:** 该代码块声明类 `Tool`。它负责承载与 protocol 相关的核心状态与行为，并通过 _propagate_defer_loading 等方法组织实现。

### Lines 584-587: Define class ToolChoiceFuncName / 定义类 ToolChoiceFuncName
```python
class ToolChoiceFuncName(BaseModel):
    """The name of tool choice function."""

    name: Optional[str] = None
```
**EN:** This block declares the class `ToolChoiceFuncName`. It centers on The name of tool choice function..
**CN:** 该代码块声明类 `ToolChoiceFuncName`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 590-594: Define class ToolChoice / 定义类 ToolChoice
```python
class ToolChoice(BaseModel):
    """The tool choice definition."""

    function: ToolChoiceFuncName
    type: Literal["function"] = Field(default="function", examples=["function"])
```
**EN:** This block declares the class `ToolChoice`. It centers on The tool choice definition..
**CN:** 该代码块声明类 `ToolChoice`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 597-648: Provide supporting module logic / 提供辅助模块逻辑
```python
class ChatCompletionRequest(BaseModel):
    # Ordered by official OpenAI API documentation
    # https://platform.openai.com/docs/api-reference/chat/create
    messages: List[ChatCompletionMessageParam]
    model: str = Field(
        default=DEFAULT_MODEL_NAME,
        description="Model name. Supports LoRA adapters via 'base-model:adapter-name' syntax.",
    )
    frequency_penalty: float = 0.0
    logit_bias: Optional[Dict[str, float]] = None
    logprobs: bool = False
    top_logprobs: Optional[int] = None
    max_tokens: Optional[int] = Field(
        default=None,
        deprecated="max_tokens is deprecated in favor of the max_completion_tokens field",
        description="The maximum number of tokens that can be generated in the chat completion. ",
    )
    max_completion_tokens: Optional[int] = Field(
        default=None,
        description="The maximum number of completion tokens for a chat completion request, "
        "including visible output tokens and reasoning tokens. Input tokens are not included. ",
    )
    n: int = 1
    presence_penalty: float = 0.0
    response_format: Optional[Union[ResponseFormat, StructuralTagResponseFormat]] = None
    seed: Optional[int] = None
    stop: Optional[Union[str, List[str]]] = None
    stream: bool = False
    stream_options: Optional[StreamOptions] = None
    temperature: Optional[float] = None
    top_p: Optional[float] = None
    user: Optional[str] = None
    tools: Optional[List[Tool]] = Field(default=None, examples=[None])
    tool_choice: Union[ToolChoice, Literal["auto", "required", "none"]] = Field(
        default="auto", examples=["none"]
    )  # noqa
    parallel_tool_calls: bool = True
    return_hidden_states: bool = False
    return_routed_experts: bool = False
    routed_experts_start_len: int = 0
    return_cached_tokens_details: bool = False
    reasoning_effort: Optional[Literal["none", "low", "medium", "high", "max"]] = Field(
        default=None,
        description="Constrains effort on reasoning for reasoning models. "
        "'none' disables reasoning entirely, 'low' is the least effort, 'high' is the most effort. "
        "Reducing reasoning effort can result in faster responses and fewer tokens used on reasoning "
        "in a response. 'none' defaults thinking and enable_thinking to false in "
        "chat_template_kwargs (unless explicitly overridden). Not supported in the harmony path."
        "'max' is an sglang extension to the OpenAI schema for "
        "models that expose a maximum-effort tier above 'high'; models that don't "
        "support it treat it the same as 'high'.",
    )
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 649-708: Provide supporting module logic / 提供辅助模块逻辑
```python
    task: Optional[
        Literal["action", "query", "authority", "domain", "title", "read_url"]
    ] = Field(
        default=None,
        description="DeepSeek-V4 quick instruction task. When set, the last "
        "user/developer message is treated as a single-shot classification prompt "
        "and the corresponding task special token (e.g. `<｜domain｜>`) is appended "
        "before generation. Only honored by the dsv4 chat encoder; ignored otherwise.",
    )

    # Extra parameters for SRT backend only and will be ignored by OpenAI models.
    top_k: Optional[int] = None
    min_p: Optional[float] = None
    min_tokens: int = 0
    regex: Optional[str] = None
    ebnf: Optional[str] = None
    repetition_penalty: Optional[float] = None
    stop_token_ids: Optional[List[int]] = None
    stop_regex: Optional[Union[str, List[str]]] = None
    no_stop_trim: bool = False
    ignore_eos: bool = False
    continue_final_message: bool = False
    skip_special_tokens: bool = True
    lora_path: Optional[Union[List[Optional[str]], Optional[str]]] = None
    session_params: Optional[Dict] = None
    separate_reasoning: bool = True
    stream_reasoning: bool = True
    chat_template_kwargs: Optional[Dict] = None

    # SGLang multimodal controls (extensions)
    max_dynamic_patch: Optional[int] = None
    min_dynamic_patch: Optional[int] = None
    use_audio_in_video: bool = False

    # Custom logit processor for advanced sampling control
    custom_logit_processor: Optional[Union[List[Optional[str]], str]] = None
    custom_params: Optional[Dict] = None

    # For request id
    rid: Optional[Union[List[str], str]] = None
    # Extra key for classifying the request (e.g. cache_salt)
    extra_key: Optional[Union[List[str], str]] = None
    # Cache salt for request caching
    cache_salt: Optional[Union[List[str], str]] = None
    # Priority for the request
    priority: Optional[int] = None

    # For PD disaggregation
    bootstrap_host: Optional[Union[List[str], str]] = None
    bootstrap_port: Optional[Union[List[Optional[int]], int]] = None
    bootstrap_room: Optional[Union[List[int], int]] = None

    # For DP routing — external router assigns a specific DP worker
    routed_dp_rank: Optional[int] = None
    # For PD disagg — hint telling decode which prefill DP worker has the KV cache
    disagg_prefill_dp_rank: Optional[int] = None
    # Deprecated: use routed_dp_rank instead
    data_parallel_rank: Optional[int] = None

    # OpenAI/SGLang default sampling parameters
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 709-718: Provide supporting module logic / 提供辅助模块逻辑
```python
    _DEFAULT_SAMPLING_PARAMS = {
        "temperature": 1.0,
        "top_p": 1.0,
        "top_k": -1,
        "min_p": 0.0,
        "repetition_penalty": 1.0,
    }

    @model_validator(mode="before")
    @classmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 719-720: Implement handle deprecated dp rank / 实现handle deprecated dp rank
```python
    def _handle_deprecated_dp_rank(cls, values):
        return _migrate_deprecated_dp_rank(values)
```
**EN:** This block implements the method `_handle_deprecated_dp_rank(values)` on `ChatCompletionRequest`. It focuses on handling the protocol responsibilities represented by `_handle_deprecated_dp_rank`, so the class can advance the protocol workflow in a self-contained way.
**CN:** 该代码块实现 `ChatCompletionRequest` 上的方法 `_handle_deprecated_dp_rank(values)`。它围绕 `_handle_deprecated_dp_rank` 所承担的 protocol 相关职责展开，使该类能够独立推进相应流程。

### Lines 721-723: Provide supporting module logic / 提供辅助模块逻辑
```python

    @model_validator(mode="before")
    @classmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 724-730: Implement set tool choice default / 实现set 工具 choice default
```python
    def set_tool_choice_default(cls, values):
        if values.get("tool_choice") is None:
            if values.get("tools") is None:
                values["tool_choice"] = "none"
            else:
                values["tool_choice"] = "auto"
        return values
```
**EN:** This block implements the method `set_tool_choice_default(values)` on `ChatCompletionRequest`. It focuses on handling the protocol responsibilities represented by `set_tool_choice_default`, so the class can advance the protocol workflow in a self-contained way.
**CN:** 该代码块实现 `ChatCompletionRequest` 上的方法 `set_tool_choice_default(values)`。它围绕 `set_tool_choice_default` 所承担的 protocol 相关职责展开，使该类能够独立推进相应流程。

### Lines 731-733: Provide supporting module logic / 提供辅助模块逻辑
```python

    @model_validator(mode="before")
    @classmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 734-771: Implement normalize reasoning inputs / 实现normalize reasoning inputs
```python
    def normalize_reasoning_inputs(cls, values: Dict):
        r = values.get("reasoning")

        if r is not None and isinstance(r, dict):
            effort = r.get("effort") or r.get("reasoning_effort")
            if effort in {"none", "low", "medium", "high"}:
                values["reasoning_effort"] = effort

            enabled = (
                r.get("enabled")
                if r.get("enabled") is not None
                else r.get("enable", False)
            )
            if isinstance(enabled, str):
                enabled = enabled.strip().lower() in {"1", "true", "yes", "y", "on"}
            if enabled:
                ctk = values.get("chat_template_kwargs")
                if not isinstance(ctk, dict):
                    ctk = {}
                # different models check different keys:
                # - "thinking" for deepseek-v3, kimi_k2
                # - "enable_thinking" for qwen3, glm45, nemotron_3, interns1, mimo
                ctk.setdefault("thinking", True)
                ctk.setdefault("enable_thinking", True)
                values["chat_template_kwargs"] = ctk

        if values.get("reasoning_effort") == "none":
            ctk = values.get("chat_template_kwargs")
            if not isinstance(ctk, dict):
                ctk = {}
            # different models check different keys:
            # - "thinking" for deepseek-v3, kimi_k2
            # - "enable_thinking" for qwen3, glm45, nemotron_3, interns1
            ctk.setdefault("thinking", False)
            ctk.setdefault("enable_thinking", False)
            values["chat_template_kwargs"] = ctk

        return values
```
**EN:** This block implements the method `normalize_reasoning_inputs(values)` on `ChatCompletionRequest`. It focuses on handling the protocol responsibilities represented by `normalize_reasoning_inputs`, so the class can advance the protocol workflow in a self-contained way.
**CN:** 该代码块实现 `ChatCompletionRequest` 上的方法 `normalize_reasoning_inputs(values)`。它围绕 `normalize_reasoning_inputs` 所承担的 protocol 相关职责展开，使该类能够独立推进相应流程。

### Lines 772-774: Provide supporting module logic / 提供辅助模块逻辑
```python

    @model_validator(mode="before")
    @classmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 775-803: Implement set json schema / 实现set JSON schema
```python
    def set_json_schema(cls, values):
        response_format = values.get("response_format")
        if not response_format:
            return values

        if response_format.get("type") != "json_schema":
            return values

        schema = response_format.pop("schema", None)
        json_schema = response_format.get("json_schema")

        if json_schema:
            return values

        if schema:
            name_ = schema.get("title", "Schema")
            strict_ = False
            if "properties" in schema and "strict" in schema["properties"]:
                item = schema["properties"].pop("strict", None)
                if item and item.get("default", False):
                    strict_ = True

            response_format["json_schema"] = {
                "name": name_,
                "schema": schema,
                "strict": strict_,
            }

        return values
```
**EN:** This block implements the method `set_json_schema(values)` on `ChatCompletionRequest`. It focuses on handling the protocol responsibilities represented by `set_json_schema`, so the class can advance the protocol workflow in a self-contained way.
**CN:** 该代码块实现 `ChatCompletionRequest` 上的方法 `set_json_schema(values)`。它围绕 `set_json_schema` 所承担的 protocol 相关职责展开，使该类能够独立推进相应流程。

### Lines 805-840: Implement to sampling params / 实现to sampling params
```python
    def to_sampling_params(
        self,
        stop: List[str],
        model_generation_config: Dict[str, Any],
        tool_call_constraint: Optional[ToolCallConstraint] = None,
    ) -> Dict[str, Any]:
        """
        Convert request to sampling parameters.
        Priority: user value > model generation_config > OpenAI defaults
        """

        def get_param(param_name: str):
            value = getattr(self, param_name)
            if value is None:
                return model_generation_config.get(
                    param_name, self._DEFAULT_SAMPLING_PARAMS[param_name]
                )
            return value

        # add per user request
        spaces_between_special_tokens = (
            True
            if self.chat_template_kwargs is None
            else self.chat_template_kwargs.get("spaces_between_special_tokens", True)
        )

        sampling_params = {
            "temperature": get_param("temperature"),
            "max_new_tokens": self.max_completion_tokens or self.max_tokens,
            "min_new_tokens": self.min_tokens,
            "stop": stop,
            "stop_token_ids": self.stop_token_ids,
            "stop_regex": self.stop_regex,
            "top_p": get_param("top_p"),
            "top_k": get_param("top_k"),
            "min_p": get_param("min_p"),
```
**EN:** This block implements the method `to_sampling_params(stop, model_generation_config, tool_call_constraint)` on `ChatCompletionRequest`. It focuses on Convert request to sampling parameters., so the class can advance the protocol workflow in a self-contained way.
**CN:** 该代码块实现 `ChatCompletionRequest` 上的方法 `to_sampling_params(stop, model_generation_config, tool_call_constraint)`。它围绕 `to_sampling_params` 所承担的 protocol 相关职责展开，使该类能够独立推进相应流程。

### Lines 841-876: Continue to sampling params / 继续说明to sampling params
```python
            "presence_penalty": self.presence_penalty,
            "frequency_penalty": self.frequency_penalty,
            "repetition_penalty": get_param("repetition_penalty"),
            "regex": self.regex,
            "ebnf": self.ebnf,
            "n": self.n,
            "no_stop_trim": self.no_stop_trim,
            "ignore_eos": self.ignore_eos,
            "skip_special_tokens": self.skip_special_tokens,
            "logit_bias": self.logit_bias,
            "custom_params": self.custom_params,
            "sampling_seed": self.seed,
            "spaces_between_special_tokens": spaces_between_special_tokens,
        }

        if self.response_format and self.response_format.type == "json_schema":
            sampling_params["json_schema"] = convert_json_schema_to_str(
                self.response_format.json_schema.schema_
            )
        elif self.response_format and self.response_format.type == "json_object":
            sampling_params["json_schema"] = '{"type": "object"}'
        elif self.response_format and self.response_format.type == "structural_tag":
            sampling_params["structural_tag"] = convert_json_schema_to_str(
                self.response_format.model_dump(by_alias=True)
            )

        # Check if there are already existing output constraints
        has_existing_constraints = (
            sampling_params.get("regex")
            or sampling_params.get("ebnf")
            or sampling_params.get("structural_tag")
            or sampling_params.get("json_schema")
        )

        if tool_call_constraint and has_existing_constraints:
            logger.warning("Constrained decoding is not compatible with tool calls.")
```
**EN:** This block implements the method `to_sampling_params(stop, model_generation_config, tool_call_constraint)` on `ChatCompletionRequest`. It focuses on Convert request to sampling parameters., so the class can advance the protocol workflow in a self-contained way.
**CN:** 该代码块实现 `ChatCompletionRequest` 上的方法 `to_sampling_params(stop, model_generation_config, tool_call_constraint)`。它围绕 `to_sampling_params` 所承担的 protocol 相关职责展开，使该类能够独立推进相应流程。

### Lines 877-890: Continue to sampling params / 继续说明to sampling params
```python
        elif tool_call_constraint:
            constraint_type, constraint_value = tool_call_constraint
            if constraint_type == "structural_tag":
                sampling_params[constraint_type] = convert_json_schema_to_str(
                    constraint_value.model_dump(by_alias=True)
                )
            elif constraint_type == "json_schema":
                sampling_params[constraint_type] = convert_json_schema_to_str(
                    constraint_value  # type: ignore
                )
            else:
                sampling_params[constraint_type] = constraint_value

        return sampling_params
```
**EN:** This block implements the method `to_sampling_params(stop, model_generation_config, tool_call_constraint)` on `ChatCompletionRequest`. It focuses on Convert request to sampling parameters., so the class can advance the protocol workflow in a self-contained way.
**CN:** 该代码块实现 `ChatCompletionRequest` 上的方法 `to_sampling_params(stop, model_generation_config, tool_call_constraint)`。它围绕 `to_sampling_params` 所承担的 protocol 相关职责展开，使该类能够独立推进相应流程。

### Lines 893-897: Define class ChatMessage / 定义类 ChatMessage
```python
class ChatMessage(BaseModel):
    role: Optional[str] = None
    content: Optional[str] = None
    reasoning_content: Optional[str] = None
    tool_calls: Optional[List[ToolCall]] = Field(default=None, examples=[None])
```
**EN:** This block declares the class `ChatMessage`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ChatMessage`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 900-917: Define class ChatCompletionResponseChoice / 定义类 ChatCompletionResponseChoice
```python
class ChatCompletionResponseChoice(BaseModel):
    index: int
    message: ChatMessage
    logprobs: Optional[Union[LogProbs, ChoiceLogprobs]] = None
    finish_reason: Optional[
        Literal[
            "stop", "length", "tool_calls", "content_filter", "function_call", "abort"
        ]
    ] = None
    matched_stop: Union[None, int, str] = None
    hidden_states: Optional[object] = None

    @model_serializer(mode="wrap")
    def _serialize(self, handler):
        data = handler(self)
        if self.hidden_states is None:
            data.pop("hidden_states", None)
        return data
```
**EN:** This block declares the class `ChatCompletionResponseChoice`. It centers on coordinating protocol behavior, with methods such as _serialize.
**CN:** 该代码块声明类 `ChatCompletionResponseChoice`。它负责承载与 protocol 相关的核心状态与行为，并通过 _serialize 等方法组织实现。

### Lines 920-935: Define class ChatCompletionResponse / 定义类 ChatCompletionResponse
```python
class ChatCompletionResponse(BaseModel):
    id: str
    object: str = "chat.completion"
    created: int = Field(default_factory=lambda: int(time.time()))
    model: str
    choices: List[ChatCompletionResponseChoice]
    usage: UsageInfo
    metadata: Optional[Dict[str, Any]] = None
    sglext: Optional[SglExt] = None

    @model_serializer(mode="wrap")
    def _serialize(self, handler):
        data = handler(self)
        if self.sglext is None:
            data.pop("sglext", None)
        return data
```
**EN:** This block declares the class `ChatCompletionResponse`. It centers on coordinating protocol behavior, with methods such as _serialize.
**CN:** 该代码块声明类 `ChatCompletionResponse`。它负责承载与 protocol 相关的核心状态与行为，并通过 _serialize 等方法组织实现。

### Lines 938-950: Define class DeltaMessage / 定义类 DeltaMessage
```python
class DeltaMessage(BaseModel):
    role: Optional[str] = None
    content: Optional[str] = None
    reasoning_content: Optional[str] = None
    tool_calls: Optional[List[ToolCall]] = Field(default=None, examples=[None])
    hidden_states: Optional[object] = None

    @model_serializer(mode="wrap")
    def _serialize(self, handler):
        data = handler(self)
        if self.hidden_states is None:
            data.pop("hidden_states", None)
        return data
```
**EN:** This block declares the class `DeltaMessage`. It centers on coordinating protocol behavior, with methods such as _serialize.
**CN:** 该代码块声明类 `DeltaMessage`。它负责承载与 protocol 相关的核心状态与行为，并通过 _serialize 等方法组织实现。

### Lines 953-962: Define class ChatCompletionResponseStreamChoice / 定义类 ChatCompletionResponseStreamChoice
```python
class ChatCompletionResponseStreamChoice(BaseModel):
    index: int
    delta: DeltaMessage
    logprobs: Optional[Union[LogProbs, ChoiceLogprobs]] = None
    finish_reason: Optional[
        Literal[
            "stop", "length", "tool_calls", "content_filter", "function_call", "abort"
        ]
    ] = None
    matched_stop: Union[None, int, str] = None
```
**EN:** This block declares the class `ChatCompletionResponseStreamChoice`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ChatCompletionResponseStreamChoice`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 965-979: Define class ChatCompletionStreamResponse / 定义类 ChatCompletionStreamResponse
```python
class ChatCompletionStreamResponse(BaseModel):
    id: str
    object: str = "chat.completion.chunk"
    created: int = Field(default_factory=lambda: int(time.time()))
    model: str
    choices: List[ChatCompletionResponseStreamChoice]
    usage: Optional[UsageInfo] = None
    sglext: Optional[SglExt] = None

    @model_serializer(mode="wrap")
    def _serialize(self, handler):
        data = handler(self)
        if self.sglext is None:
            data.pop("sglext", None)
        return data
```
**EN:** This block declares the class `ChatCompletionStreamResponse`. It centers on coordinating protocol behavior, with methods such as _serialize.
**CN:** 该代码块声明类 `ChatCompletionStreamResponse`。它负责承载与 protocol 相关的核心状态与行为，并通过 _serialize 等方法组织实现。

### Lines 982-985: Define class MultimodalEmbeddingInput / 定义类 MultimodalEmbeddingInput
```python
class MultimodalEmbeddingInput(BaseModel):
    text: Optional[str] = None
    image: Optional[str] = None
    video: Optional[str] = None
```
**EN:** This block declares the class `MultimodalEmbeddingInput`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `MultimodalEmbeddingInput`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 988-990: Provide supporting module logic / 提供辅助模块逻辑
```python
EmbeddingInput = Union[
    List[int], List[List[int]], str, List[str], List[MultimodalEmbeddingInput]
]
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 993-1012: Define class EmbeddingRequest / 定义类 EmbeddingRequest
```python
class EmbeddingRequest(BaseModel):
    # Ordered by official OpenAI API documentation
    # https://platform.openai.com/docs/api-reference/embeddings/create
    input: EmbeddingInput
    model: str = DEFAULT_MODEL_NAME
    encoding_format: str = "float"
    dimensions: Optional[int] = None
    user: Optional[str] = None

    # The request id.
    rid: Optional[Union[List[str], str]] = None
    # Priority for the request
    priority: Optional[int] = None
    # LoRA adapter path(s)
    lora_path: Optional[Union[List[Optional[str]], Optional[str]]] = None
    # Placeholder token id used to locate embedding override positions in input token IDs.
    embed_override_token_id: Optional[int] = None
    # Per-input embedding overrides (null entries skip that input).
    # Shape: [num_inputs][num_replacements][hidden_size]
    embed_overrides: Optional[List[Optional[List[List[float]]]]] = None
```
**EN:** This block declares the class `EmbeddingRequest`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `EmbeddingRequest`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1015-1018: Define class EmbeddingObject / 定义类 EmbeddingObject
```python
class EmbeddingObject(BaseModel):
    embedding: List[float]
    index: int
    object: str = "embedding"
```
**EN:** This block declares the class `EmbeddingObject`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `EmbeddingObject`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1021-1021: Provide supporting module logic / 提供辅助模块逻辑
```python
ClassifyInput = Union[str, List[str], List[int]]
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1024-1033: Define class ClassifyRequest / 定义类 ClassifyRequest
```python
class ClassifyRequest(BaseModel):
    # OpenAI-compatible classification request
    model: str = DEFAULT_MODEL_NAME
    input: ClassifyInput
    user: Optional[str] = None

    # The request id.
    rid: Optional[Union[List[str], str]] = None
    # Priority for the request
    priority: Optional[int] = None
```
**EN:** This block declares the class `ClassifyRequest`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ClassifyRequest`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1036-1040: Define class ClassifyData / 定义类 ClassifyData
```python
class ClassifyData(BaseModel):
    index: int
    label: str
    probs: List[float]
    num_classes: int
```
**EN:** This block declares the class `ClassifyData`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ClassifyData`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1043-1049: Define class ClassifyResponse / 定义类 ClassifyResponse
```python
class ClassifyResponse(BaseModel):
    id: str
    object: str = "list"
    created: int
    model: str
    data: List[ClassifyData]
    usage: UsageInfo
```
**EN:** This block declares the class `ClassifyResponse`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ClassifyResponse`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1052-1056: Define class EmbeddingResponse / 定义类 EmbeddingResponse
```python
class EmbeddingResponse(BaseModel):
    data: List[EmbeddingObject]
    model: str
    object: str = "list"
    usage: Optional[UsageInfo] = None
```
**EN:** This block declares the class `EmbeddingResponse`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `EmbeddingResponse`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1059-1082: Define class ScoringRequest / 定义类 ScoringRequest
```python
class ScoringRequest(BaseModel):
    query: Optional[Union[str, List[int]]] = (
        None  # Query text or pre-tokenized token IDs
    )
    items: Optional[Union[str, List[str], List[List[int]]]] = (
        None  # Item text(s) or pre-tokenized token IDs
    )
    # Placeholder token id used to locate embedding override positions in query/items.
    embed_override_token_id: Optional[int] = None
    # Query embedding overrides.
    query_embed_overrides: Optional[List[List[float]]] = (
        None  # [num_query_embed_overrides][hidden_size]
    )
    # Per-item embedding overrides (null entries skip that item).
    item_embed_overrides: Optional[List[Optional[List[List[float]]]]] = (
        None  # [num_items][num_item_embed_overrides][hidden_size]
    )
    label_token_ids: Optional[List[int]] = (
        None  # Token IDs to compute probabilities for
    )
    apply_softmax: bool = False
    item_first: bool = False
    return_pooled_hidden_states: bool = False
    model: str = DEFAULT_MODEL_NAME
```
**EN:** This block declares the class `ScoringRequest`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ScoringRequest`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1085-1092: Define class ScoringResponse / 定义类 ScoringResponse
```python
class ScoringResponse(BaseModel):
    scores: List[
        List[float]
    ]  # List of lists of probabilities, each in the order of label_token_ids
    pooled_hidden_states: Optional[List[Optional[List[float]]]] = None
    model: str
    usage: Optional[UsageInfo] = None
    object: str = "scoring"
```
**EN:** This block declares the class `ScoringResponse`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ScoringResponse`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1095-1134: Define class V1RerankReqInput / 定义类 V1RerankReqInput
```python
class V1RerankReqInput(BaseModel):
    query: RerankContent = Field(
        ...,
        description="The query to match against documents. Can be a string (text-only) "
        "or a list of content parts for multimodal queries (text, image_url, video_url).",
    )
    documents: List[RerankContent] = Field(
        ...,
        description="List of documents to rank. Each document can be a string (text-only) "
        "or a list of content parts for multimodal documents (text, image_url, video_url).",
    )
    instruct: Optional[str] = Field(
        default=None,
        description="The instruct to the reranker model.",
    )
    top_n: Optional[int] = Field(
        default=None,
        description="Maximum number of documents to return. Defaults to returning all documents. "
        "If specified value is greater than the total number of documents, all documents will be returned.",
    )
    return_documents: bool = Field(
        default=True,
        description="Whether to return documents in the response. Only included when set to true.",
    )

    @field_validator("top_n")
    @classmethod
    def validate_top_n(cls, v):
        if v is not None and v < 1:
            raise ValueError("Value error, parameter top_n should be larger than 0.")
        return v

    def is_multimodal(self) -> bool:
        """Check if the request contains any multimodal content."""
        if isinstance(self.query, list):
            return True
        for doc in self.documents:
            if isinstance(doc, list):
                return True
        return False
```
**EN:** This block declares the class `V1RerankReqInput`. It centers on coordinating protocol behavior, with methods such as validate_top_n, is_multimodal.
**CN:** 该代码块声明类 `V1RerankReqInput`。它负责承载与 protocol 相关的核心状态与行为，并通过 validate_top_n, is_multimodal 等方法组织实现。

### Lines 1137-1149: Define class RerankResponse / 定义类 RerankResponse
```python
class RerankResponse(BaseModel):
    score: float
    document: Optional[str] = None
    index: int
    meta_info: Optional[dict] = None

    @model_serializer(mode="wrap")
    def _serialize(self, handler):
        data = handler(self)
        # Exclude document field if it's None
        if self.document is None:
            data.pop("document", None)
        return data
```
**EN:** This block declares the class `RerankResponse`. It centers on coordinating protocol behavior, with methods such as _serialize.
**CN:** 该代码块声明类 `RerankResponse`。它负责承载与 protocol 相关的核心状态与行为，并通过 _serialize 等方法组织实现。

### Lines 1152-1186: Define class TokenizeRequest / 定义类 TokenizeRequest
```python
class TokenizeRequest(BaseModel):
    """Request schema for the /tokenize endpoint."""

    model_config = ConfigDict(extra="allow")

    model: str = DEFAULT_MODEL_NAME
    prompt: Optional[Union[str, List[str]]] = None
    messages: Optional[List[ChatCompletionMessageParam]] = None
    tools: Optional[List[Tool]] = Field(default=None, examples=[None])
    tool_choice: Optional[Union[ToolChoice, Literal["auto", "required", "none"]]] = (
        Field(default=None, examples=["auto"])
    )
    reasoning_effort: Optional[Literal["none", "low", "medium", "high"]] = None
    continue_final_message: bool = False
    chat_template_kwargs: Optional[Dict] = None
    add_special_tokens: bool = Field(
        default=True,
        description="whether to add model-specific special tokens (e.g. BOS/EOS) during encoding.",
    )

    @model_validator(mode="after")
    def validate_tokenize_input(self) -> "TokenizeRequest":
        if (self.prompt is None) == (self.messages is None):
            raise ValueError("Exactly one of 'prompt' or 'messages' must be provided.")
        return self

    def to_chat_completion_request(self) -> ChatCompletionRequest:
        data = self.model_dump(
            exclude={"prompt", "add_special_tokens"},
            exclude_none=True,
        )
        extra = getattr(self, "__pydantic_extra__", None)
        if extra:
            data.update(extra)
        return ChatCompletionRequest.model_validate(data)
```
**EN:** This block declares the class `TokenizeRequest`. It centers on Request schema for the /tokenize endpoint., with methods such as validate_tokenize_input, to_chat_completion_request.
**CN:** 该代码块声明类 `TokenizeRequest`。它负责承载与 protocol 相关的核心状态与行为，并通过 validate_tokenize_input, to_chat_completion_request 等方法组织实现。

### Lines 1189-1194: Define class TokenizeResponse / 定义类 TokenizeResponse
```python
class TokenizeResponse(BaseModel):
    """Response schema for the /tokenize endpoint."""

    tokens: Union[List[int], List[List[int]]]
    count: Union[int, List[int]]
    max_model_len: int
```
**EN:** This block declares the class `TokenizeResponse`. It centers on Response schema for the /tokenize endpoint..
**CN:** 该代码块声明类 `TokenizeResponse`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1197-1205: Define class DetokenizeRequest / 定义类 DetokenizeRequest
```python
class DetokenizeRequest(BaseModel):
    """Request schema for the /detokenize endpoint."""

    model: str = DEFAULT_MODEL_NAME
    tokens: Union[List[int], List[List[int]]]
    skip_special_tokens: bool = Field(
        default=True,
        description="whether to exclude special tokens (e.g. padding or EOS) during decoding.",
    )
```
**EN:** This block declares the class `DetokenizeRequest`. It centers on Request schema for the /detokenize endpoint..
**CN:** 该代码块声明类 `DetokenizeRequest`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1208-1211: Define class DetokenizeResponse / 定义类 DetokenizeResponse
```python
class DetokenizeResponse(BaseModel):
    """Response schema for the /detokenize endpoint."""

    text: Union[str, List[str]]
```
**EN:** This block declares the class `DetokenizeResponse`. It centers on Response schema for the /detokenize endpoint..
**CN:** 该代码块声明类 `DetokenizeResponse`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1214-1226: Provide supporting module logic / 提供辅助模块逻辑
```python
OpenAIServingRequest = Union[
    ChatCompletionRequest,
    CompletionRequest,
    EmbeddingRequest,
    ClassifyRequest,
    ScoringRequest,
    V1RerankReqInput,
    TokenizeRequest,
    DetokenizeRequest,
]


# Response API protocol definitions
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1227-1233: Define class ResponseReasoningParam / 定义类 ResponseReasoningParam
```python
class ResponseReasoningParam(BaseModel):
    """Reasoning parameters for responses."""

    effort: Optional[Literal["low", "medium", "high"]] = Field(
        default="medium",
        description="Constrains effort on reasoning for reasoning models.",
    )
```
**EN:** This block declares the class `ResponseReasoningParam`. It centers on Reasoning parameters for responses..
**CN:** 该代码块声明类 `ResponseReasoningParam`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1236-1241: Define class ResponseTool / 定义类 ResponseTool
```python
class ResponseTool(BaseModel):
    """Tool definition for responses."""

    type: Literal["web_search_preview", "code_interpreter"] = Field(
        description="Type of tool to enable"
    )
```
**EN:** This block declares the class `ResponseTool`. It centers on Tool definition for responses..
**CN:** 该代码块声明类 `ResponseTool`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1244-1248: Provide supporting module logic / 提供辅助模块逻辑
```python
ResponseInputOutputItem: TypeAlias = Union[
    ResponseInputItemParam,
    "ResponseReasoningItem",
    ResponseFunctionToolCall,
]
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1251-1363: Define class ResponsesRequest / 定义类 ResponsesRequest
```python
class ResponsesRequest(BaseModel):
    """Request body for v1/responses endpoint."""

    # Core OpenAI API fields (ordered by official documentation)
    background: Optional[bool] = False
    include: Optional[
        List[
            Literal[
                "code_interpreter_call.outputs",
                "computer_call_output.output.image_url",
                "file_search_call.results",
                "message.input_image.image_url",
                "message.output_text.logprobs",
                "reasoning.encrypted_content",
            ]
        ]
    ] = None
    input: Union[str, List[ResponseInputOutputItem]]
    instructions: Optional[str] = None
    max_output_tokens: Optional[int] = None
    max_tool_calls: Optional[int] = None
    metadata: Optional[Dict[str, Any]] = None
    model: Optional[str] = None  # Made optional to match vLLM
    parallel_tool_calls: Optional[bool] = True
    previous_response_id: Optional[str] = None
    reasoning: Optional[ResponseReasoningParam] = None
    service_tier: Literal["auto", "default", "flex", "scale", "priority"] = "auto"
    store: Optional[bool] = True
    stream: Optional[bool] = False
    temperature: Optional[float] = None
    tool_choice: Literal["auto", "required", "none"] = "auto"
    tools: List[ResponseTool] = Field(default_factory=list)
    top_logprobs: Optional[int] = 0
    top_p: Optional[float] = None
    truncation: Optional[Literal["auto", "disabled"]] = "disabled"
    user: Optional[str] = None

    # Extra SGLang parameters
    request_id: str = Field(
        default_factory=lambda: f"resp_{uuid.uuid4().hex}",
        description="The request_id related to this request. If the caller does not set it, a random uuid will be generated.",
    )
    priority: int = Field(default=0, description="Request priority")
    extra_key: Optional[str] = Field(
        default=None,
        description="Extra key for classifying the request (e.g. cache_salt)",
    )
    cache_salt: Optional[str] = Field(
        default=None, description="Cache salt for request caching"
    )

    # SGLang-specific sampling parameters
    frequency_penalty: float = 0.0
    presence_penalty: float = 0.0
    stop: Optional[Union[str, List[str]]] = None
    top_k: int = -1
    min_p: float = 0.0
    repetition_penalty: float = 1.0

    # Default sampling parameters
    _DEFAULT_SAMPLING_PARAMS = {
        "temperature": 0.7,
        "top_p": 1.0,
        "top_k": -1,
        "min_p": 0.0,
        "repetition_penalty": 1.0,
    }

    def to_sampling_params(
        self, default_max_tokens: int, default_params: Optional[Dict] = None
    ) -> Dict[str, Any]:
        """Convert to sampling parameters for generation."""
        if default_params is None:
            default_params = {}

        # Use max_output_tokens if available, otherwise use max_tokens for backwards compatibility
        if self.max_output_tokens is not None:
            max_tokens = min(self.max_output_tokens, default_max_tokens)
        else:
            max_tokens = default_max_tokens

        # Avoid exceed the context length by minus 2 token
        max_tokens -= 2

        # Get parameters with defaults
        temperature = self.temperature
        if temperature is None:
            temperature = default_params.get(
                "temperature", self._DEFAULT_SAMPLING_PARAMS["temperature"]
            )

        top_p = self.top_p
        if top_p is None:
            top_p = default_params.get("top_p", self._DEFAULT_SAMPLING_PARAMS["top_p"])

        params = {
            "max_new_tokens": max_tokens,
            "temperature": temperature,
            "top_p": top_p,
            "frequency_penalty": self.frequency_penalty,
            "presence_penalty": self.presence_penalty,
            "stop": self.stop,
            "top_k": self.top_k,
            "min_p": self.min_p,
            "repetition_penalty": self.repetition_penalty,
        }

        # Apply any additional default parameters
        for key, value in default_params.items():
            if key not in params or params[key] is None:
                params[key] = value

        return params
```
**EN:** This block declares the class `ResponsesRequest`. It centers on Request body for v1/responses endpoint., with methods such as to_sampling_params.
**CN:** 该代码块声明类 `ResponsesRequest`。它负责承载与 protocol 相关的核心状态与行为，并通过 to_sampling_params 等方法组织实现。

### Lines 1366-1369: Define class PromptTokenUsageInfo / 定义类 PromptTokenUsageInfo
```python
class PromptTokenUsageInfo(BaseModel):
    """Prompt token usage details."""

    cached_tokens: int = 0
```
**EN:** This block declares the class `PromptTokenUsageInfo`. It centers on Prompt token usage details..
**CN:** 该代码块声明类 `PromptTokenUsageInfo`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1372-1485: Define class ResponsesResponse / 定义类 ResponsesResponse
```python
class ResponsesResponse(BaseModel):
    """Response body for v1/responses endpoint."""

    id: str = Field(default_factory=lambda: f"resp_{time.time()}")
    object: Literal["response"] = "response"
    created_at: int = Field(default_factory=lambda: int(time.time()))
    model: str

    output: List[
        Union[ResponseOutputItem, ResponseReasoningItem, ResponseFunctionToolCall]
    ] = Field(default_factory=list)
    status: Literal["queued", "in_progress", "completed", "failed", "cancelled"]
    usage: Optional[UsageInfo] = None
    parallel_tool_calls: bool = True
    tool_choice: str = "auto"
    tools: List[ResponseTool] = Field(default_factory=list)

    # OpenAI compatibility fields. not all are used at the moment.
    # Recommend checking https://platform.openai.com/docs/api-reference/responses
    error: Optional[dict] = None
    incomplete_details: Optional[dict] = None  # TODO(v) support this input
    instructions: Optional[str] = None
    max_output_tokens: Optional[int] = None
    previous_response_id: Optional[str] = None
    reasoning: Optional[dict] = (
        # Unused. No model supports this. For GPT-oss, system prompt sets
        # the field, not server args.
        None  # {"effort": Optional[str], "summary": Optional[str]}
    )
    store: Optional[bool] = None
    temperature: Optional[float] = None
    text: Optional[dict] = None  # e.g. {"format": {"type": "text"}}
    top_p: Optional[float] = None
    truncation: Optional[str] = None
    user: Optional[str] = None
    metadata: Optional[Dict[str, Any]] = None

    @classmethod
    def from_request(
        cls,
        request: ResponsesRequest,
        sampling_params: Any,
        model_name: str,
        created_time: int,
        output: List[
            Union[ResponseOutputItem, ResponseReasoningItem, ResponseFunctionToolCall]
        ],
        status: str,
        usage: Optional[UsageInfo],
    ) -> "ResponsesResponse":
        """Create a response from a request."""

        # Determine if the output is plain text only to set text.format
        def _is_text_only(
            items: List[
                Union[
                    ResponseOutputItem, ResponseReasoningItem, ResponseFunctionToolCall
                ]
            ],
        ) -> bool:
            if not items:
                return False
            for it in items:
                # tool call -> not pure text.
                if isinstance(it, ResponseReasoningItem) or isinstance(
                    it, ResponseFunctionToolCall
                ):
                    return False
                try:
                    if isinstance(it, ResponseOutputText):
                        continue
                    elif isinstance(it, ResponseOutputMessage):
                        if not it.content:
                            continue
                        for c in it.content:
                            if not isinstance(c, ResponseOutputText):
                                return False
                    else:
                        # Unknown type, not considered text-only
                        return False
                except AttributeError:
                    return False
            return True

        text_format = {"format": {"type": "text"}} if _is_text_only(output) else None

        return cls(
            id=request.request_id,
            created_at=created_time,
            model=model_name,
            output=output,
            status=status,
            usage=usage,
            parallel_tool_calls=request.parallel_tool_calls or True,
            tool_choice=request.tool_choice,
            tools=request.tools,
            # fields for parity with v1/responses
            error=None,
            incomplete_details=None,
            instructions=request.instructions,
            max_output_tokens=request.max_output_tokens,
            previous_response_id=request.previous_response_id,  # TODO(v): ensure this is propagated if retrieved from store
            reasoning={
                "effort": request.reasoning.effort if request.reasoning else None,
                "summary": None,  # unused
            },
            store=request.store,
            temperature=request.temperature,
            text=text_format,  # TODO(v): Expand coverage per https://platform.openai.com/docs/api-reference/responses/list
            top_p=request.top_p,
            truncation=request.truncation,
            user=request.user,
            metadata=request.metadata or {},
        )
```
**EN:** This block declares the class `ResponsesResponse`. It centers on Response body for v1/responses endpoint., with methods such as from_request.
**CN:** 该代码块声明类 `ResponsesResponse`。它负责承载与 protocol 相关的核心状态与行为，并通过 from_request 等方法组织实现。

### Lines 1488-1492: Define class RequestResponseMetadata / 定义类 RequestResponseMetadata
```python
class RequestResponseMetadata(BaseModel):
    """Metadata for request/response tracking."""

    request_id: str
    final_usage_info: Optional[UsageInfo] = None
```
**EN:** This block declares the class `RequestResponseMetadata`. It centers on Metadata for request/response tracking..
**CN:** 该代码块声明类 `RequestResponseMetadata`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1493-1495: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1496-1520: Define class MessageProcessingResult / 定义类 MessageProcessingResult
```python
class MessageProcessingResult:
    """Result of processing chat messages and applying templates.

    This dataclass encapsulates all the outputs from message processing including
    prompt generation, multimodal data extraction, and constraint preparation.
    Used internally by OpenAIServingChat to pass processed data between methods.

    Args:
        prompt: The final text prompt after applying chat template
        prompt_ids: Either the text prompt (str) or tokenized IDs (List[int])
        image_data: Extracted image data from messages, if any
        audio_data: Extracted audio data from messages, if any
        modalities: List of modality types present in the messages
        stop: Combined stop strings from template and request
        tool_call_constraint: Optional constraint for structured tool calls
    """

    prompt: str
    prompt_ids: Union[str, List[int]]
    image_data: Optional[Any]
    audio_data: Optional[Any]
    video_data: Optional[Any]
    modalities: List[str]
    stop: List[str]
    tool_call_constraint: Optional[ToolCallConstraint] = None
```
**EN:** This block declares the class `MessageProcessingResult`. It centers on Result of processing chat messages and applying templates..
**CN:** 该代码块声明类 `MessageProcessingResult`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1523-1530: Define class ToolCallProcessingResult / 定义类 ToolCallProcessingResult
```python
class ToolCallProcessingResult(NamedTuple):
    """Result of processing tool calls in a response."""

    tool_calls: Optional[
        List[Any]
    ]  # List of ToolCall objects or None if parsing failed
    remaining_text: str  # Text remaining after parsing tool calls
    finish_reason: Dict[str, Any]  # Updated finish reason dictionary
```
**EN:** This block declares the class `ToolCallProcessingResult`. It centers on Result of processing tool calls in a response..
**CN:** 该代码块声明类 `ToolCallProcessingResult`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1533-1535: Define class ResponseReasoningTextContent / 定义类 ResponseReasoningTextContent
```python
class ResponseReasoningTextContent(BaseModel):
    text: str
    type: Literal["reasoning_text"] = "reasoning_text"
```
**EN:** This block declares the class `ResponseReasoningTextContent`. It centers on coordinating protocol behavior.
**CN:** 该代码块声明类 `ResponseReasoningTextContent`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1538-1545: Provide supporting module logic / 提供辅助模块逻辑
```python
ResponseInputOutputItem: TypeAlias = Union[
    ResponseInputItemParam, "ResponseReasoningItem", ResponseFunctionToolCall
]


# ================== Transcription API Protocol Definitions ==================
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1546-1557: Define class TranscriptionRequest / 定义类 TranscriptionRequest
```python
class TranscriptionRequest(BaseModel):
    """Request model for audio transcription (OpenAI-compatible)."""

    model: str = DEFAULT_MODEL_NAME
    language: Optional[str] = None
    response_format: str = "json"
    temperature: float = 0.0
    timestamp_granularities: Optional[List[str]] = None
    stream: bool = False
    # Internal fields (not from API)
    audio_data: Optional[bytes] = None
    audio_duration_s: float = 0.0
```
**EN:** This block declares the class `TranscriptionRequest`. It centers on Request model for audio transcription (OpenAI-compatible)..
**CN:** 该代码块声明类 `TranscriptionRequest`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1560-1564: Define class TranscriptionUsage / 定义类 TranscriptionUsage
```python
class TranscriptionUsage(BaseModel):
    """Usage info for transcription response (duration-based)."""

    type: Literal["duration"] = "duration"
    seconds: int  # Audio duration in seconds (rounded up)
```
**EN:** This block declares the class `TranscriptionUsage`. It centers on Usage info for transcription response (duration-based)..
**CN:** 该代码块声明类 `TranscriptionUsage`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1567-1571: Define class TranscriptionResponse / 定义类 TranscriptionResponse
```python
class TranscriptionResponse(BaseModel):
    """Non-streaming transcription response (OpenAI-compatible)."""

    text: str
    usage: Optional[TranscriptionUsage] = None
```
**EN:** This block declares the class `TranscriptionResponse`. It centers on Non-streaming transcription response (OpenAI-compatible)..
**CN:** 该代码块声明类 `TranscriptionResponse`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1574-1580: Define class TranscriptionSegment / 定义类 TranscriptionSegment
```python
class TranscriptionSegment(BaseModel):
    """A segment with timestamp information."""

    id: int
    start: float
    end: float
    text: str
```
**EN:** This block declares the class `TranscriptionSegment`. It centers on A segment with timestamp information..
**CN:** 该代码块声明类 `TranscriptionSegment`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1583-1591: Define class TranscriptionVerboseResponse / 定义类 TranscriptionVerboseResponse
```python
class TranscriptionVerboseResponse(BaseModel):
    """Verbose transcription response with timestamps (OpenAI-compatible)."""

    task: str = "transcribe"
    language: Optional[str] = None
    duration: Optional[float] = None
    text: str
    segments: List[TranscriptionSegment] = []
    usage: Optional[TranscriptionUsage] = None
```
**EN:** This block declares the class `TranscriptionVerboseResponse`. It centers on Verbose transcription response with timestamps (OpenAI-compatible)..
**CN:** 该代码块声明类 `TranscriptionVerboseResponse`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1594-1598: Define class TranscriptionStreamChoice / 定义类 TranscriptionStreamChoice
```python
class TranscriptionStreamChoice(BaseModel):
    """Delta content for streaming transcription."""

    delta: DeltaMessage
    finish_reason: Optional[str] = None
```
**EN:** This block declares the class `TranscriptionStreamChoice`. It centers on Delta content for streaming transcription..
**CN:** 该代码块声明类 `TranscriptionStreamChoice`。它负责承载与 protocol 相关的核心状态与行为。

### Lines 1601-1609: Define class TranscriptionStreamResponse / 定义类 TranscriptionStreamResponse
```python
class TranscriptionStreamResponse(BaseModel):
    """Streaming transcription chunk (OpenAI-compatible)."""

    id: str = Field(default_factory=lambda: f"trsc-{uuid.uuid4().hex}")
    object: Literal["transcription.chunk"] = "transcription.chunk"
    created: int = Field(default_factory=lambda: int(time.time()))
    model: str
    choices: List[TranscriptionStreamChoice]
    usage: Optional[UsageInfo] = None
```
**EN:** This block declares the class `TranscriptionStreamResponse`. It centers on Streaming transcription chunk (OpenAI-compatible)..
**CN:** 该代码块声明类 `TranscriptionStreamResponse`。它负责承载与 protocol 相关的核心状态与行为。

## Key Concepts / 关键概念
- **Core types / 核心类型**: ModelCard, ModelList, ErrorResponse, LogProbs, TopLogprob, ChatCompletionTokenLogprob, ...
- **Main callables / 主要可调用对象**: _migrate_deprecated_dp_rank
- **Domain focus / 领域焦点**: protocol / protocol
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: dataclasses, logging, time, typing, typing_extensions, uuid, warnings
- **Third-party / 第三方库**: openai.types.responses, openai.types.responses.response, openai.types.responses.tool, pydantic, xgrammar
- **Local Modules / 本地模块**: sglang.utils
