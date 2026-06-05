# chat_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/chat_utils.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the module logic for this entrypoint component. / 实现该入口组件的模块逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-63 — Imports and shared dependencies
```python
import asyncio
import json
from abc import ABC, abstractmethod
from collections import Counter, defaultdict
from collections.abc import Awaitable, Callable, Iterable
from dataclasses import dataclass
from functools import cached_property, lru_cache, partial
from itertools import accumulate
from pathlib import Path
from typing import TYPE_CHECKING, Any, Final, Generic, Literal, TypeAlias, TypeVar, cast

from openai.types.chat import (
    ChatCompletionAssistantMessageParam,
    ChatCompletionContentPartImageParam,
    ChatCompletionContentPartInputAudioParam,
    ChatCompletionContentPartRefusalParam,
    ChatCompletionContentPartTextParam,
    ChatCompletionFunctionToolParam,
    ChatCompletionMessageToolCallParam,
    ChatCompletionToolMessageParam,
)
from openai.types.chat import (
    ChatCompletionContentPartParam as OpenAIChatCompletionContentPartParam,
)
from openai.types.chat import (
    ChatCompletionMessageParam as OpenAIChatCompletionMessageParam,
...
from vllm.multimodal.processing import BaseMultiModalProcessor
from vllm.renderers.embed_utils import (
    safe_load_prompt_embeds,
    safe_load_prompt_embeds_async,
)
from vllm.utils import random_uuid
from vllm.utils.collection_utils import is_list_of
from vllm.utils.import_utils import LazyLoader
```
**EN:** This import block pulls in standard-library modules such as `asyncio`, `json`, `abc`, `collections`, `dataclasses`, `functools`, uses third-party packages like `openai`, `openai_harmony`, `PIL`, `pydantic`, `typing_extensions`, depends on internal helpers such as `vllm`, `vllm.config`, `vllm.exceptions`, `vllm.inputs`, `vllm.logger`, `vllm.model_executor.models`.
**CN:** 该导入块引入 `asyncio`, `json`, `abc`, `collections`, `dataclasses`, `functools` 等标准库模块，使用 `openai`, `openai_harmony`, `PIL`, `pydantic`, `typing_extensions` 等第三方库，依赖 `vllm`, `vllm.config`, `vllm.exceptions`, `vllm.inputs`, `vllm.logger`, `vllm.model_executor.models` 等 vLLM 内部模块。

### Lines 65-70 — Conditional top-level flow
```python
if TYPE_CHECKING:
    import torch
    import transformers
else:
    transformers = LazyLoader("transformers", globals(), "transformers")
    torch = LazyLoader("torch", globals(), "torch")
```
**EN:** This top-level conditional toggles behavior based on runtime configuration or platform state.
**CN:** 该顶层条件分支会根据运行时配置或平台状态切换行为。

### Lines 72-72 — Shared module state
```python
logger = init_logger(__name__)
```
**EN:** This block initializes `logger`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`，为后续逻辑准备模块级常量或共享状态。

### Lines 75-80 — Class `ChatTemplateResolutionError`
```python
class ChatTemplateResolutionError(ValueError):
    """Raised when chat template resolution fails.

    This is a subclass of ValueError for backward compatibility with
    existing exception handlers.
    """
```
**EN:** Class `ChatTemplateResolutionError` is introduced here. Its docstring describes the intent as: Raised when chat template resolution fails.
**CN:** 这里定义类 `ChatTemplateResolutionError`。其文档字符串说明了该类的职责与使用方式。

### Lines 83-91 — Module constants
```python
MODALITY_PLACEHOLDERS_MAP = {
    "image": "<##IMAGE##>",
    "audio": "<##AUDIO##>",
    "video": "<##VIDEO##>",
    "prompt_embeds": "<##PROMPT_EMBEDS##>",
}


PROMPT_EMBEDS_PLACEHOLDER_TOKEN: Final[str] = "<prompt_embeds>"
```
**EN:** This block initializes `MODALITY_PLACEHOLDERS_MAP`, `PROMPT_EMBEDS_PLACEHOLDER_TOKEN`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `MODALITY_PLACEHOLDERS_MAP`, `PROMPT_EMBEDS_PLACEHOLDER_TOKEN`，为后续逻辑准备模块级常量或共享状态。

### Lines 92-97 — Top-level logic
```python
"""The special token used as a placeholder for each embedding
position during chat template rendering.

Registered as an additional special token when `--enable-prompt-embeds` is set.
See `_ensure_prompt_embeds_placeholder_token` in `vllm/renderers/hf.py`.
"""
```
**EN:** This block performs module-level orchestration that does not fit into a named function or class.
**CN:** 该代码块执行无法归入具名函数或类的模块级编排逻辑。

### Lines 100-118 — Module constants
```python
_REQUIRE_MM_PROCESSOR_ERROR: Final[str] = (
    "Resolving modality {modality!r} requires a multimodal processor "
    "but none is available."
)

_ENABLE_PROMPT_EMBEDS_ERROR: Final[str] = (
    "You must set `--enable-prompt-embeds` to input `prompt_embeds`"
)

_PROMPT_EMBEDS_MISSING_DATA_ERROR: Final[str] = (
    "prompt_embeds content part requires a non-empty `data` field "
    "with base64-encoded tensor bytes."
)

_RESERVED_PLACEHOLDER_IN_TEXT_ERROR: Final[str] = (
    "Text content may not contain the reserved placeholder {token!r}. "
    "This placeholder is used internally to mark `prompt_embeds` splice "
    "positions in the tokenized prompt."
)
```
**EN:** This block initializes `_REQUIRE_MM_PROCESSOR_ERROR`, `_ENABLE_PROMPT_EMBEDS_ERROR`, `_PROMPT_EMBEDS_MISSING_DATA_ERROR`, `_RESERVED_PLACEHOLDER_IN_TEXT_ERROR`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `_REQUIRE_MM_PROCESSOR_ERROR`, `_ENABLE_PROMPT_EMBEDS_ERROR`, `_PROMPT_EMBEDS_MISSING_DATA_ERROR`, `_RESERVED_PLACEHOLDER_IN_TEXT_ERROR`，为后续逻辑准备模块级常量或共享状态。

### Lines 121-125 — Class `AudioURL`
```python
class AudioURL(TypedDict, total=False):
    url: Required[str]
    """
    Either a URL of the audio or a data URL with base64 encoded audio data.
    """
```
**EN:** Class `AudioURL` is defined here, extending `TypedDict`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `AudioURL`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 128-132 — Class `ChatCompletionContentPartAudioParam`
```python
class ChatCompletionContentPartAudioParam(TypedDict, total=False):
    audio_url: Required[AudioURL]

    type: Required[Literal["audio_url"]]
    """The type of the content part."""
```
**EN:** Class `ChatCompletionContentPartAudioParam` is defined here, extending `TypedDict`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `ChatCompletionContentPartAudioParam`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 135-148 — Class `ChatCompletionContentPartImageEmbedsParam`
```python
class ChatCompletionContentPartImageEmbedsParam(TypedDict, total=False):
    image_embeds: str | dict[str, str] | None
    """
    The image embeddings. It can be either:
    - A single base64 string.
    - A dictionary where each value is a base64 string.
    """
    type: Required[Literal["image_embeds"]]
    """The type of the content part."""
    uuid: str | None
    """
    User-provided UUID of a media. User must guarantee that it is properly
    generated and unique for different medias.
    """
```
**EN:** Class `ChatCompletionContentPartImageEmbedsParam` is defined here, extending `TypedDict`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `ChatCompletionContentPartImageEmbedsParam`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 151-164 — Class `ChatCompletionContentPartAudioEmbedsParam`
```python
class ChatCompletionContentPartAudioEmbedsParam(TypedDict, total=False):
    audio_embeds: str | dict[str, str] | None
    """
    The audio embeddings. It can be either:
    - A single base64 string representing a serialized torch tensor.
    - A dictionary where each value is a base64 string.
    """
    type: Required[Literal["audio_embeds"]]
    """The type of the content part."""
    uuid: str | None
    """
    User-provided UUID of a media. User must guarantee that it is properly
    generated and unique for different medias.
    """
```
**EN:** Class `ChatCompletionContentPartAudioEmbedsParam` is defined here, extending `TypedDict`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `ChatCompletionContentPartAudioEmbedsParam`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 167-175 — Class `ChatCompletionContentPartPromptEmbedsParam`
```python
class ChatCompletionContentPartPromptEmbedsParam(TypedDict, total=False):
    data: Required[str]
    """
    Base64-encoded bytes of a serialized `torch.Tensor` of shape
    `(num_tokens, hidden_size)`. The tensor's `dtype` and `hidden_size` must
    match the model's input embedding layer.
    """
    type: Required[Literal["prompt_embeds"]]
    """The type of the content part."""
```
**EN:** Class `ChatCompletionContentPartPromptEmbedsParam` is defined here, extending `TypedDict`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `ChatCompletionContentPartPromptEmbedsParam`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 178-182 — Class `VideoURL`
```python
class VideoURL(TypedDict, total=False):
    url: Required[str]
    """
    Either a URL of the video or a data URL with base64 encoded video data.
    """
```
**EN:** Class `VideoURL` is defined here, extending `TypedDict`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `VideoURL`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 185-189 — Class `ChatCompletionContentPartVideoParam`
```python
class ChatCompletionContentPartVideoParam(TypedDict, total=False):
    video_url: Required[VideoURL]

    type: Required[Literal["video_url"]]
    """The type of the content part."""
```
**EN:** Class `ChatCompletionContentPartVideoParam` is defined here, extending `TypedDict`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `ChatCompletionContentPartVideoParam`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 192-198 — Class `PILImage`
```python
class PILImage(BaseModel):
    """
    A PIL.Image.Image object.
    """

    image_pil: Image.Image
    model_config = ConfigDict(arbitrary_types_allowed=True)
```
**EN:** Class `PILImage` is introduced here. Its docstring describes the intent as: A PIL.Image.Image object.
**CN:** 这里定义类 `PILImage`。其文档字符串说明了该类的职责与使用方式。

### Lines 201-215 — Class `CustomChatCompletionContentPILImageParam`
```python
class CustomChatCompletionContentPILImageParam(TypedDict, total=False):
    """A simpler version of the param that only accepts a PIL image.

    Example:
    {
        "image_pil": ImageAsset('cherry_blossom').pil_image
    }
    """

    image_pil: PILImage | None
    uuid: str | None
    """
    User-provided UUID of a media. User must guarantee that it is properly
    generated and unique for different medias.
    """
```
**EN:** Class `CustomChatCompletionContentPILImageParam` is introduced here. Its docstring describes the intent as: A simpler version of the param that only accepts a PIL image.
**CN:** 这里定义类 `CustomChatCompletionContentPILImageParam`。其文档字符串说明了该类的职责与使用方式。

### Lines 218-233 — Class `CustomChatCompletionContentSimpleImageParam`
```python
class CustomChatCompletionContentSimpleImageParam(TypedDict, total=False):
    """A simpler version of the param that only accepts a plain image_url.
    This is supported by OpenAI API, although it is not documented.

    Example:
    {
        "image_url": "https://example.com/image.jpg"
    }
    """

    image_url: str | None
    uuid: str | None
    """
    User-provided UUID of a media. User must guarantee that it is properly
    generated and unique for different medias.
    """
```
**EN:** Class `CustomChatCompletionContentSimpleImageParam` is introduced here. Its docstring describes the intent as: A simpler version of the param that only accepts a plain image_url.
**CN:** 这里定义类 `CustomChatCompletionContentSimpleImageParam`。其文档字符串说明了该类的职责与使用方式。

### Lines 236-245 — Class `CustomChatCompletionContentSimpleAudioParam`
```python
class CustomChatCompletionContentSimpleAudioParam(TypedDict, total=False):
    """A simpler version of the param that only accepts a plain audio_url.

    Example:
    {
        "audio_url": "https://example.com/audio.mp3"
    }
    """

    audio_url: str | None
```
**EN:** Class `CustomChatCompletionContentSimpleAudioParam` is introduced here. Its docstring describes the intent as: A simpler version of the param that only accepts a plain audio_url.
**CN:** 这里定义类 `CustomChatCompletionContentSimpleAudioParam`。其文档字符串说明了该类的职责与使用方式。

### Lines 248-262 — Class `CustomChatCompletionContentSimpleVideoParam`
```python
class CustomChatCompletionContentSimpleVideoParam(TypedDict, total=False):
    """A simpler version of the param that only accepts a plain audio_url.

    Example:
    {
        "video_url": "https://example.com/video.mp4"
    }
    """

    video_url: str | None
    uuid: str | None
    """
    User-provided UUID of a media. User must guarantee that it is properly
    generated and unique for different medias.
    """
```
**EN:** Class `CustomChatCompletionContentSimpleVideoParam` is introduced here. Its docstring describes the intent as: A simpler version of the param that only accepts a plain audio_url.
**CN:** 这里定义类 `CustomChatCompletionContentSimpleVideoParam`。其文档字符串说明了该类的职责与使用方式。

### Lines 265-283 — Class `CustomThinkCompletionContentParam`
```python
class CustomThinkCompletionContentParam(TypedDict, total=False):
    """A Think Completion Content Param that accepts a plain text and a boolean.

    Example:
    {
        "thinking": "I am thinking about the answer",
        "closed": True,
        "type": "thinking"
    }
    """

    thinking: Required[str]
    """The thinking content."""

    closed: bool
    """Whether the thinking is closed."""

    type: Required[Literal["thinking"]]
    """The thinking type."""
```
**EN:** Class `CustomThinkCompletionContentParam` is introduced here. Its docstring describes the intent as: A Think Completion Content Param that accepts a plain text and a boolean.
**CN:** 这里定义类 `CustomThinkCompletionContentParam`。其文档字符串说明了该类的职责与使用方式。

### Lines 286-300 — Class `CustomChatCompletionContentToolReferenceParam`
```python
class CustomChatCompletionContentToolReferenceParam(TypedDict, total=False):
    """A tool reference content param that only accepts a plain tool name.

    Example:
    {
        "name": "get_weather",
        "type": "tool_reference"
    }
    """

    name: str
    """The name of the tool being referenced."""

    type: Literal["tool_reference"]
    """The content type."""
```
**EN:** Class `CustomChatCompletionContentToolReferenceParam` is introduced here. Its docstring describes the intent as: A tool reference content param that only accepts a plain tool name.
**CN:** 这里定义类 `CustomChatCompletionContentToolReferenceParam`。其文档字符串说明了该类的职责与使用方式。

### Lines 303-319 — Module constants and state
```python
ChatCompletionContentPartParam: TypeAlias = (
    OpenAIChatCompletionContentPartParam
    | ChatCompletionContentPartAudioParam
    | ChatCompletionContentPartInputAudioParam
    | ChatCompletionContentPartVideoParam
    | ChatCompletionContentPartRefusalParam
    | CustomChatCompletionContentPILImageParam
    | CustomChatCompletionContentSimpleImageParam
    | ChatCompletionContentPartImageEmbedsParam
    | ChatCompletionContentPartAudioEmbedsParam
    | ChatCompletionContentPartPromptEmbedsParam
    | CustomChatCompletionContentSimpleAudioParam
    | CustomChatCompletionContentSimpleVideoParam
    | CustomChatCompletionContentToolReferenceParam
    | str
    | CustomThinkCompletionContentParam
)
```
**EN:** This block initializes `ChatCompletionContentPartParam`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `ChatCompletionContentPartParam`，为后续逻辑准备模块级常量或共享状态。

### Lines 322-351 — Class `CustomChatCompletionMessageParam`
```python
class CustomChatCompletionMessageParam(TypedDict, total=False):
    """Enables custom roles in the Chat Completion API."""

    role: Required[str]
    """The role of the message's author."""

    content: str | list[ChatCompletionContentPartParam]
    """The contents of the message."""

    name: str
    """An optional name for the participant.

    Provides the model information to differentiate between participants of the
    same role.
    """

    tool_call_id: str | None
    """Tool call that this message is responding to."""

    tool_calls: list[ChatCompletionMessageToolCallParam] | None
    """The tool calls generated by the model, such as function calls."""

    reasoning: str | None
    """The reasoning content for interleaved thinking."""

    tools: list[ChatCompletionFunctionToolParam] | None
    """The tools for developer role."""

    task: str | None
    """Model-specific task marker. Currently passed through for DeepSeek V4."""
```
**EN:** Class `CustomChatCompletionMessageParam` is introduced here. Its docstring describes the intent as: Enables custom roles in the Chat Completion API.
**CN:** 这里定义类 `CustomChatCompletionMessageParam`。其文档字符串说明了该类的职责与使用方式。

### Lines 354-358 — Module constants and state
```python
ChatCompletionMessageParam: TypeAlias = (
    OpenAIChatCompletionMessageParam
    | CustomChatCompletionMessageParam
    | OpenAIHarmonyMessage
)
```
**EN:** This block initializes `ChatCompletionMessageParam`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `ChatCompletionMessageParam`，为后续逻辑准备模块级常量或共享状态。

### Lines 362-388 — Class `ConversationMessage`
```python
class ConversationMessage(TypedDict, total=False):
    role: Required[str]
    """The role of the message's author."""

    content: str | None | list[dict[str, str]]
    """The contents of the message"""

    tool_call_id: str | None
    """Tool call that this message is responding to."""

    name: str | None
    """The name of the function to call"""

    tool_calls: list[ChatCompletionMessageToolCallParam] | None
    """The tool calls generated by the model, such as function calls."""

    reasoning: str | None
    """The reasoning content for interleaved thinking."""

    reasoning_content: str | None
    """Deprecated: The reasoning content for interleaved thinking."""

    tools: list[ChatCompletionFunctionToolParam] | None
    """The tools for developer role."""

    task: str | None
    """Model-specific task marker. Currently passed through for DeepSeek V4."""
```
**EN:** Class `ConversationMessage` acts as a protocol/data model with about 9 field declarations and helper methods such as its inherited interface.
**CN:** 类 `ConversationMessage` 充当协议/数据模型，包含约 9 个字段声明，并提供 继承接口 等辅助方法。

### Lines 392-407 — Module constants
```python
ChatTemplateContentFormatOption = Literal["auto", "string", "openai"]

# After resolving "auto"
ChatTemplateContentFormat = Literal["string", "openai"]


ModalityStr = Literal[
    "image",
    "audio",
    "video",
    "image_embeds",
    "audio_embeds",
    "vision_chunk",
    "prompt_embeds",
]
_T = TypeVar("_T")
```
**EN:** This block initializes `ChatTemplateContentFormatOption`, `ChatTemplateContentFormat`, `ModalityStr`, `_T`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `ChatTemplateContentFormatOption`, `ChatTemplateContentFormat`, `ModalityStr`, `_T`，为后续逻辑准备模块级常量或共享状态。

### Lines 411-412 — Class `_BatchedSingleItemField`
```python
class _BatchedSingleItemField(MultiModalSharedField):
    pass
```
**EN:** Class `_BatchedSingleItemField` is defined here, extending `MultiModalSharedField`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `_BatchedSingleItemField`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 415-443 — Function `_detect_field`
```python
def _detect_field(
    tensors: list[torch.Tensor],
    mm_processor: BaseMultiModalProcessor,
):
    first_item = tensors[0]
    hidden_size = mm_processor.info.ctx.model_config.get_inputs_embeds_size()

    if (
        len(tensors) == 1
        and first_item.ndim == 3
        and first_item.shape[0] == 1
        and first_item.shape[-1] == hidden_size
    ):
        logger.warning(
            "Batched multi-modal embedding inputs are deprecated for Chat API. "
            "Please pass a separate content part for each multi-modal item."
        )
        return _BatchedSingleItemField(batch_size=1)

    first_shape = first_item.shape
    if all(t.shape == first_shape for t in tensors):
        return MultiModalBatchedField()

    size_per_item = [len(tensor) for tensor in tensors]
    slice_idxs = [0, *accumulate(size_per_item)]
    slices = [
        (slice(slice_idxs[i], slice_idxs[i + 1]),) for i in range(len(size_per_item))
    ]
    return MultiModalFlatField(slices=slices)
```
**EN:** This function `_detect_field` implements the `detect field` step within the module flow.
**CN:** 该函数 `_detect_field` 实现了模块流程中的“detectfield”步骤。

### Lines 446-494 — Function `_merge_embeds`
```python
def _merge_embeds(
    data_items: list[dict[str, "torch.Tensor"]],
    mm_processor: BaseMultiModalProcessor,
):
    if not data_items:
        return {}

    first_keys = set(data_items[0].keys())
    if any(set(item.keys()) != first_keys for item in data_items[1:]):
        raise ValueError(
            "All dictionaries in the list of embeddings must have the same keys."
        )

    fields = {
        key: _detect_field([item[key] for item in data_items], mm_processor)
        for key in first_keys
    }
    data_merged = {
        key: field._reduce_data([item[key] for item in data_items], pin_memory=False)
        for key, field in fields.items()
    }

    try:
        # TODO: Support per-request mm_processor_kwargs
        parsed_configs = mm_processor._get_mm_fields_config(
            transformers.BatchFeature(data_merged),
            {},
        )
...
            )
    except Exception:
        logger.exception(
            "Error when parsing merged embeddings. "
            "Falling back to auto-detected fields."
        )

    return data_merged
```
**EN:** This function `_merge_embeds` implements the `merges embeds` step within the module flow.
**CN:** 该函数 `_merge_embeds` 实现了模块流程中的“合并embeds”步骤。

### Lines 497-516 — Function `_get_embeds_data`
```python
def _get_embeds_data(
    modality: str,
    data_items: list[Any],
    mm_processor: BaseMultiModalProcessor,
):
    if len(data_items) == 0:
        return data_items

    if all(item is None for item in data_items):
        return data_items

    if is_list_of(data_items, torch.Tensor):
        embeds_key = f"{modality}_embeds"
        dict_items = [{embeds_key: item} for item in data_items]
        return _merge_embeds(dict_items, mm_processor)[embeds_key]

    if is_list_of(data_items, dict):
        return _merge_embeds(data_items, mm_processor)

    raise NotImplementedError(type(data_items))
```
**EN:** This function `_get_embeds_data` implements the `gets embeds data` step within the module flow.
**CN:** 该函数 `_get_embeds_data` 实现了模块流程中的“获取embedsdata”步骤。

### Lines 519-641 — Class `BaseMultiModalItemTracker`
```python
class BaseMultiModalItemTracker(ABC, Generic[_T]):
    """
    Tracks multi-modal items in a given request and ensures that the number
    of multi-modal items in a given request does not exceed the configured
    maximum per prompt.
    """

    def __init__(
        self,
        model_config: ModelConfig,
        media_io_kwargs: dict[str, dict[str, Any]] | None = None,
    ):
        super().__init__()

        self._model_config = model_config
        self._media_io_kwargs = media_io_kwargs

        self._items_by_modality = defaultdict[str, list[_T]](list)
        # Track original modality for each vision_chunk item (image or video)
        self._modality_order = defaultdict[str, list[str]](list)

    @cached_property
    def use_unified_vision_chunk_modality(self) -> bool:
        """Check if model uses unified vision_chunk modality for images/videos."""
        return getattr(self._model_config.hf_config, "use_unified_vision_chunk", False)

    @property
    def model_config(self) -> ModelConfig:
...

        return self.model_cls.get_placeholder_str(modality, num_items)

    @abstractmethod
    def create_parser(
        self, mm_processor_kwargs: dict[str, Any] | None = None
    ) -> "BaseMultiModalContentParser":
        raise NotImplementedError
```
**EN:** Class `BaseMultiModalItemTracker` is introduced here. Its docstring describes the intent as: Tracks multi-modal items in a given request and ensures that the number of multi-modal items in a given request does not exceed the configured maximum per prompt.
**CN:** 这里定义类 `BaseMultiModalItemTracker`。其文档字符串说明了该类的职责与使用方式。

### Lines 644-703 — Function `_resolve_vision_chunk_items`
```python
def _resolve_vision_chunk_items(
    vision_chunk_items: list[tuple[object, str | None]],
    mm_processor: BaseMultiModalProcessor,
    vision_chunks_modality_order: list[str],
):
    # Process vision_chunk items - extract from (data, modality) tuples
    # and convert to VisionChunk types with proper UUID handling
    vision_chunks_uuids = [uuid for data, uuid in vision_chunk_items]

    assert len(vision_chunk_items) == len(vision_chunks_modality_order), (
        f"vision_chunk items ({len(vision_chunk_items)}) and "
        f"modality_order ({len(vision_chunks_modality_order)}) must have same length"
    )

    processed_chunks: list[VisionChunk] = []
    video_idx = 0
    for inner_modality, (data, uuid) in zip(
        vision_chunks_modality_order, vision_chunk_items
    ):
        if inner_modality == "image":
            # Cast data to proper type for image
            # Use .media (PIL.Image) directly to avoid redundant
            # bytes→PIL conversion in media_processor
            if hasattr(data, "media"):
                image_data = data.media  # type: ignore[union-attr]
                processed_chunks.append(
                    VisionChunkImage(type="image", image=image_data, uuid=uuid)
                )
...
                        )
                    video_idx += 1
                except Exception as e:
                    logger.warning("Failed to split video chunks: %s", e)
                    processed_chunks.append(data)  # type: ignore[arg-type]
            else:
                processed_chunks.append(data)  # type: ignore[arg-type]
    return processed_chunks, vision_chunks_uuids
```
**EN:** This function `_resolve_vision_chunk_items` implements the `resolves vision chunk items` step within the module flow.
**CN:** 该函数 `_resolve_vision_chunk_items` 实现了模块流程中的“解析visionchunkitems”步骤。

### Lines 706-773 — Function `_resolve_items`
```python
def _resolve_items(
    items_by_modality: dict[str, list[tuple[object, str | None]]],
    mm_processor: BaseMultiModalProcessor | None,
    modality_order: dict[str, list[str]],
) -> tuple[MultiModalDataDict, MultiModalUUIDDict]:
    """
    Materialize the tracker's per-modality items into `mm_data` / `mm_uuids`.

    Note:
        `mm_processor` is `None` for text-only models (no registered HF
        processor) whose only modality is `prompt_embeds`. Every other
        modality requires a processor, enforced by the guard below.
    """
    if "image" in items_by_modality and "image_embeds" in items_by_modality:
        raise ValueError("Mixing raw image and embedding inputs is not allowed")
    if "audio" in items_by_modality and "audio_embeds" in items_by_modality:
        raise ValueError("Mixing raw audio and embedding inputs is not allowed")
    # `prompt_embeds` bypasses HF MM processors. Every other modality requires one.
    processor_modalities = items_by_modality.keys() - {"prompt_embeds"}
    if processor_modalities and mm_processor is None:
        raise RuntimeError(
            _REQUIRE_MM_PROCESSOR_ERROR.format(modality=processor_modalities)
        )

    mm_data = {}
    mm_uuids = {}
    if "image_embeds" in items_by_modality:
        assert mm_processor is not None
...
        mm_data["vision_chunk"] = processed_chunks
        mm_uuids["vision_chunk"] = vision_chunk_uuids
    if "prompt_embeds" in items_by_modality:
        mm_data["prompt_embeds"] = [
            data for data, _uuid in items_by_modality["prompt_embeds"]
        ]

    return mm_data, mm_uuids
```
**EN:** This function `_resolve_items` is documented as: Materialize the tracker's per-modality items into `mm_data` / `mm_uuids`.
**CN:** 这里定义函数 `_resolve_items`，其文档字符串说明了主要职责与调用约定。

### Lines 776-798 — Class `MultiModalItemTracker`
```python
class MultiModalItemTracker(BaseMultiModalItemTracker[tuple[object, str | None]]):
    def resolve_items(
        self,
    ) -> tuple[MultiModalDataDict | None, MultiModalUUIDDict | None]:
        if not self._items_by_modality:
            return None, None

        # Text-only models (`is_multimodal_model=False`) with inputs of
        # modality `prompt_embeds` have no MM processor since `prompt_embeds` are
        # pre-computed and require no processing, so we pass `None`.
        mm_processor = (
            self.mm_processor if self._model_config.is_multimodal_model else None
        )
        return _resolve_items(
            dict(self._items_by_modality),
            mm_processor,
            self._modality_order,
        )

    def create_parser(
        self, mm_processor_kwargs: dict[str, Any] | None = None
    ) -> "BaseMultiModalContentParser":
        return MultiModalContentParser(self, mm_processor_kwargs=mm_processor_kwargs)
```
**EN:** Class `MultiModalItemTracker` is defined here, as a standalone type, and groups behavior through methods like `resolve_items`, `create_parser`.
**CN:** 这里定义类 `MultiModalItemTracker`，其职责是封装相关状态与行为，并通过 `resolve_items`、`create_parser` 等方法组织逻辑。

### Lines 801-829 — Class `AsyncMultiModalItemTracker`
```python
class AsyncMultiModalItemTracker(
    BaseMultiModalItemTracker[Awaitable[tuple[object, str | None]]]
):
    async def resolve_items(
        self,
    ) -> tuple[MultiModalDataDict | None, MultiModalUUIDDict | None]:
        if not self._items_by_modality:
            return None, None

        resolved_items_by_modality = {
            modality: await asyncio.gather(*coros)
            for modality, coros in self._items_by_modality.items()
        }

        mm_processor = (
            self.mm_processor if self._model_config.is_multimodal_model else None
        )
        return _resolve_items(
            resolved_items_by_modality,
            mm_processor,
            self._modality_order,
        )

    def create_parser(
        self, mm_processor_kwargs: dict[str, Any] | None = None
    ) -> "BaseMultiModalContentParser":
        return AsyncMultiModalContentParser(
            self, mm_processor_kwargs=mm_processor_kwargs
        )
```
**EN:** Class `AsyncMultiModalItemTracker` is defined here, as a standalone type, and groups behavior through methods like `resolve_items`, `create_parser`.
**CN:** 这里定义类 `AsyncMultiModalItemTracker`，其职责是封装相关状态与行为，并通过 `resolve_items`、`create_parser` 等方法组织逻辑。

### Lines 832-900 — Class `BaseMultiModalContentParser`
```python
class BaseMultiModalContentParser(ABC):
    def __init__(self) -> None:
        super().__init__()

        # stores model placeholders list with corresponding
        # general MM placeholder:
        # {
        #   "<##IMAGE##>": ["<image>", "<image>", "<image>"],
        #   "<##AUDIO##>": ["<audio>", "<audio>"],
        #   "<##PROMPT_EMBEDS##>": ["<prompt_embeds>", "<prompt_embeds>"]
        # }
        self._placeholder_storage: dict[str, list] = defaultdict(list)

    @property
    @abstractmethod
    def model_config(self) -> ModelConfig:
        raise NotImplementedError

    def _add_placeholder(self, modality: ModalityStr, placeholder: str | None):
        mod_placeholder = MODALITY_PLACEHOLDERS_MAP[modality]
        if placeholder:
            self._placeholder_storage[mod_placeholder].append(placeholder)

    def mm_placeholder_storage(self) -> dict[str, list]:
        return dict(self._placeholder_storage)

    @abstractmethod
    def parse_image(self, image_url: str | None, uuid: str | None = None) -> None:
...

    @abstractmethod
    def parse_prompt_embeds(self, data: str) -> None:
        raise NotImplementedError

    @abstractmethod
    def parse_video(self, video_url: str | None, uuid: str | None = None) -> None:
        raise NotImplementedError
```
**EN:** Class `BaseMultiModalContentParser` is defined here, extending `ABC`, and groups behavior through methods like `__init__`, `model_config`, `_add_placeholder`, `mm_placeholder_storage`.
**CN:** 这里定义类 `BaseMultiModalContentParser`，其职责是封装相关状态与行为，并通过 `__init__`、`model_config`、`_add_placeholder`、`mm_placeholder_storage` 等方法组织逻辑。

### Lines 903-1041 — Class `MultiModalContentParser`
```python
class MultiModalContentParser(BaseMultiModalContentParser):
    def __init__(
        self,
        tracker: MultiModalItemTracker,
        mm_processor_kwargs: dict[str, Any] | None = None,
    ) -> None:
        super().__init__()

        self._tracker = tracker

        self._connector: MediaConnector = MEDIA_CONNECTOR_REGISTRY.load(
            envs.VLLM_MEDIA_CONNECTOR,
            media_io_kwargs=tracker.media_io_kwargs,
            allowed_local_media_path=tracker.allowed_local_media_path,
            allowed_media_domains=tracker.allowed_media_domains,
        )

        self._mm_processor_kwargs = mm_processor_kwargs

    @property
    def model_config(self) -> ModelConfig:
        return self._tracker.model_config

    @override
    def parse_prompt_embeds(self, data: str) -> None:
        """Decode a base64 prompt embeds tensor and store it in the tracker.

        Emits a single `PROMPT_EMBEDS_PLACEHOLDER_TOKEN` sentinel per
...
        if (
            video_url
            and self._mm_processor_kwargs
            and self._mm_processor_kwargs.get("use_audio_in_video", False)
        ):
            audio = self._connector.fetch_audio(video_url) if video_url else None
            audio_placeholder = self._tracker.add("audio", (audio, uuid))
            self._add_placeholder("audio", audio_placeholder)
```
**EN:** Class `MultiModalContentParser` is defined here, extending `BaseMultiModalContentParser`, and groups behavior through methods like `__init__`, `model_config`, `parse_prompt_embeds`, `parse_image`.
**CN:** 这里定义类 `MultiModalContentParser`，其职责是封装相关状态与行为，并通过 `__init__`、`model_config`、`parse_prompt_embeds`、`parse_image` 等方法组织逻辑。

### Lines 1044-1226 — Class `AsyncMultiModalContentParser`
```python
class AsyncMultiModalContentParser(BaseMultiModalContentParser):
    def __init__(
        self,
        tracker: AsyncMultiModalItemTracker,
        mm_processor_kwargs: dict[str, Any] | None = None,
    ) -> None:
        super().__init__()

        self._tracker = tracker
        self._connector: MediaConnector = MEDIA_CONNECTOR_REGISTRY.load(
            envs.VLLM_MEDIA_CONNECTOR,
            media_io_kwargs=tracker.media_io_kwargs,
            allowed_local_media_path=tracker.allowed_local_media_path,
            allowed_media_domains=tracker.allowed_media_domains,
        )
        self._mm_processor_kwargs: dict[str, Any] | None = mm_processor_kwargs

    @property
    def model_config(self) -> ModelConfig:
        return self._tracker.model_config

    @override
    def parse_prompt_embeds(self, data: str) -> None:
        """Schedule async prompt embeds decode and store the coroutine in the tracker.

        Like the sync variant, emits a single sentinel `PROMPT_EMBEDS_PLACEHOLDER_TOKEN`
        per content part. Unlike the sync variant, the tensor decode is deferred to a
        thread-pool executor via `safe_load_prompt_embeds_async`.
...
        if (
            video_url
            and self._mm_processor_kwargs
            and self._mm_processor_kwargs.get("use_audio_in_video", False)
        ):
            audio_coro = self._audio_with_uuid_async(video_url, uuid)
            audio_placeholder = self._tracker.add("audio", audio_coro)
            self._add_placeholder("audio", audio_placeholder)
```
**EN:** Class `AsyncMultiModalContentParser` is defined here, extending `BaseMultiModalContentParser`, and groups behavior through methods like `__init__`, `model_config`, `parse_prompt_embeds`, `_load_prompt_embeds_async`.
**CN:** 这里定义类 `AsyncMultiModalContentParser`，其职责是封装相关状态与行为，并通过 `__init__`、`model_config`、`parse_prompt_embeds`、`_load_prompt_embeds_async` 等方法组织逻辑。

### Lines 1230-1233 — Class `ChatTemplateConfig`
```python
class ChatTemplateConfig:
    chat_template: str | None = None
    chat_template_content_format: ChatTemplateContentFormatOption = "auto"
    trust_request_chat_template: bool = False
```
**EN:** Class `ChatTemplateConfig` is defined here, as a standalone type, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `ChatTemplateConfig`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 1236-1264 — Function `validate_chat_template`
```python
def validate_chat_template(chat_template: Path | str | None):
    """Raises if the provided chat template appears invalid."""
    if chat_template is None:
        return

    elif isinstance(chat_template, Path) and not chat_template.exists():
        raise FileNotFoundError("the supplied chat template path doesn't exist")

    elif isinstance(chat_template, str):
        JINJA_CHARS = "{}\n"
        if (
            not any(c in chat_template for c in JINJA_CHARS)
            and not Path(chat_template).exists()
        ):
            # Try to find the template in the built-in templates directory
            from vllm.transformers_utils.chat_templates.registry import (
                CHAT_TEMPLATES_DIR,
            )

            builtin_template_path = CHAT_TEMPLATES_DIR / chat_template
            if not builtin_template_path.exists():
                raise ValueError(
                    f"The supplied chat template string ({chat_template}) "
                    f"appears path-like, but doesn't exist! "
                    f"Tried: {chat_template} and {builtin_template_path}"
                )

    else:
        raise TypeError(f"{type(chat_template)} is not a valid chat template type")
```
**EN:** This function `validate_chat_template` is documented as: Raises if the provided chat template appears invalid.
**CN:** 这里定义函数 `validate_chat_template`，其文档字符串说明了主要职责与调用约定。

### Lines 1267-1312 — Function `_load_chat_template`
```python
def _load_chat_template(
    chat_template: Path | str | None,
    *,
    is_literal: bool = False,
) -> str | None:
    if chat_template is None:
        return None

    if is_literal:
        if isinstance(chat_template, Path):
            raise TypeError(
                "chat_template is expected to be read directly from its value"
            )

        return chat_template

    try:
        with open(chat_template) as f:
            return f.read()
    except OSError as e:
        if isinstance(chat_template, Path):
            raise

        JINJA_CHARS = "{}\n"
        if not any(c in chat_template for c in JINJA_CHARS):
            # Try to load from the built-in templates directory
            from vllm.transformers_utils.chat_templates.registry import (
                CHAT_TEMPLATES_DIR,
...
                    f"Tried: {chat_template} and {builtin_template_path}. "
                    f"Reason: {e}"
                )
                raise ValueError(msg) from e

        # If opening a file fails, set chat template to be args to
        # ensure we decode so our escape are interpreted correctly
        return _load_chat_template(chat_template, is_literal=True)
```
**EN:** This function `_load_chat_template` implements the `loads chat template` step within the module flow.
**CN:** 该函数 `_load_chat_template` 实现了模块流程中的“加载对话template”步骤。

### Lines 1315-1315 — Module constants and state
```python
_cached_load_chat_template = lru_cache(_load_chat_template)
```
**EN:** This block initializes `_cached_load_chat_template`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `_cached_load_chat_template`，为后续逻辑准备模块级常量或共享状态。

### Lines 1318-1323 — Function `load_chat_template`
```python
def load_chat_template(
    chat_template: Path | str | None,
    *,
    is_literal: bool = False,
) -> str | None:
    return _cached_load_chat_template(chat_template, is_literal=is_literal)
```
**EN:** This function `load_chat_template` implements the `loads chat template` step within the module flow.
**CN:** 该函数 `load_chat_template` 实现了模块流程中的“加载对话template”步骤。

### Lines 1326-1333 — Function `_get_interleaved_text_prompt`
```python
def _get_interleaved_text_prompt(
    placeholder_storage: dict[str, list], texts: list[str]
) -> str:
    for idx, elem in enumerate(texts):
        if elem in placeholder_storage:
            texts[idx] = placeholder_storage[elem].pop(0)

    return "\n".join(texts)
```
**EN:** This function `_get_interleaved_text_prompt` implements the `gets interleaved text prompt` step within the module flow.
**CN:** 该函数 `_get_interleaved_text_prompt` 实现了模块流程中的“获取interleaved文本提示词”步骤。

### Lines 1338-1392 — Function `_get_full_multimodal_text_prompt`
```python
def _get_full_multimodal_text_prompt(
    placeholder_storage: dict[str, list],
    texts: list[str],
    interleave_strings: bool,
    multimodal_content_part_separator: str = "\n",
) -> str:
    """Combine multimodal prompts for a multimodal language model."""

    # flatten storage to make it looks like
    # {
    #   "<|image|>": 2,
    #   "<|audio|>": 1
    # }
    placeholder_counts = Counter(
        [v for elem in placeholder_storage.values() for v in elem]
    )

    if interleave_strings:
        text_prompt = _get_interleaved_text_prompt(placeholder_storage, texts)
    else:
        text_prompt = "\n".join(texts)

    # Pass interleaved text further in case the user used image placeholders
    # himself, but forgot to disable the 'interleave_strings' flag

    # Look through the text prompt to check for missing placeholders
    missing_placeholders: list[str] = []
    for placeholder in placeholder_counts:
...
    # NOTE: Default behaviour: we always add missing placeholders
    # at the front of the prompt, if interleave_strings=False
    if text_prompt:
        return multimodal_content_part_separator.join(
            missing_placeholders + [text_prompt]
        )
    else:
        return multimodal_content_part_separator.join(missing_placeholders)
```
**EN:** This function `_get_full_multimodal_text_prompt` is documented as: Combine multimodal prompts for a multimodal language model.
**CN:** 这里定义函数 `_get_full_multimodal_text_prompt`，其文档字符串说明了主要职责与调用约定。

### Lines 1396-1434 — Module constants
```python
_TextParser = partial(cast, ChatCompletionContentPartTextParam)
_ImageEmbedsParser = partial(cast, ChatCompletionContentPartImageEmbedsParam)
_AudioEmbedsParser = partial(cast, ChatCompletionContentPartAudioEmbedsParam)
_PromptEmbedsParser = partial(cast, ChatCompletionContentPartPromptEmbedsParam)
_InputAudioParser = partial(cast, ChatCompletionContentPartInputAudioParam)
_RefusalParser = partial(cast, ChatCompletionContentPartRefusalParam)
_PILImageParser = partial(cast, CustomChatCompletionContentPILImageParam)
_ThinkParser = partial(cast, CustomThinkCompletionContentParam)
# Need to validate url objects
_ImageParser = TypeAdapter(ChatCompletionContentPartImageParam).validate_python
_AudioParser = TypeAdapter(ChatCompletionContentPartAudioParam).validate_python
_VideoParser = TypeAdapter(ChatCompletionContentPartVideoParam).validate_python

_ResponsesInputImageParser = TypeAdapter(ResponseInputImageParam).validate_python
_ContentPart: TypeAlias = str | dict[str, str] | InputAudio | PILImage
...
    "audio_url": lambda part: _AudioParser(part).get("audio_url", {}).get("url", None),
    "input_audio": lambda part: _InputAudioParser(part).get("input_audio", None),
    "refusal": lambda part: _RefusalParser(part).get("refusal", None),
    "video_url": lambda part: _VideoParser(part).get("video_url", {}).get("url", None),
    "tool_reference": lambda part: cast(
        CustomChatCompletionContentToolReferenceParam, part
    ).get("name", None),
}
```
**EN:** This block initializes `_TextParser`, `_ImageEmbedsParser`, `_AudioEmbedsParser`, `_PromptEmbedsParser`, `_InputAudioParser`, `_RefusalParser`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `_TextParser`, `_ImageEmbedsParser`, `_AudioEmbedsParser`, `_PromptEmbedsParser`, `_InputAudioParser`, `_RefusalParser`，为后续逻辑准备模块级常量或共享状态。

### Lines 1437-1541 — Function `_parse_chat_message_content_mm_part`
```python
def _parse_chat_message_content_mm_part(
    part: ChatCompletionContentPartParam,
) -> tuple[str, _ContentPart]:
    """
    Parses a given multi-modal content part based on its type.

    Args:
        part: A dict containing the content part, with a potential 'type' field.

    Returns:
        A tuple (part_type, content) where:
        - part_type: Type of the part (e.g., 'text', 'image_url').
        - content: Parsed content (e.g., text, image URL).

    Raises:
        ValueError: If the 'type' field is missing and no direct URL is found.
    """
    assert isinstance(
        part, dict
    )  # This is needed to avoid mypy errors: part.get() from str
    part_type = part.get("type", None)
    uuid = part.get("uuid", None)

    if isinstance(part_type, str) and part_type in MM_PARSER_MAP and uuid is None:  # noqa: E501
        content = MM_PARSER_MAP[part_type](part)

        # Special case for 'image_url.detail'
        # We only support 'auto', which is the default
...
            tool_reference = tool_reference_params.get("name", None)
            return "tool_reference", tool_reference
        # Raise an error if no 'type' or direct URL is found.
        raise ValueError("Missing 'type' field in multimodal part.")

    if not isinstance(part_type, str):
        raise ValueError("Invalid 'type' field in multimodal part.")
    return part_type, "unknown part_type content"
```
**EN:** This function `_parse_chat_message_content_mm_part` is documented as: Parses a given multi-modal content part based on its type.
**CN:** 这里定义函数 `_parse_chat_message_content_mm_part`，其文档字符串说明了主要职责与调用约定。

### Lines 1544-1547 — Module constants
```python
PART_TYPES_TO_SKIP_NONE_CONTENT = (
    "text",
    "refusal",
)
```
**EN:** This block initializes `PART_TYPES_TO_SKIP_NONE_CONTENT`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `PART_TYPES_TO_SKIP_NONE_CONTENT`，为后续逻辑准备模块级常量或共享状态。

### Lines 1550-1589 — Function `_parse_chat_message_content_parts`
```python
def _parse_chat_message_content_parts(
    role: str,
    parts: Iterable[ChatCompletionContentPartParam],
    mm_tracker: BaseMultiModalItemTracker,
    *,
    wrap_dicts: bool,
    interleave_strings: bool,
    mm_processor_kwargs: dict[str, Any] | None = None,
    multimodal_content_part_separator="\n",
) -> list[ConversationMessage]:
    content = list[_ContentPart]()

    mm_parser = mm_tracker.create_parser(mm_processor_kwargs=mm_processor_kwargs)

    for part in parts:
        parse_res = _parse_chat_message_content_part(
            part,
            mm_parser,
            wrap_dicts=wrap_dicts,
            interleave_strings=interleave_strings,
        )
        if parse_res:
            content.append(parse_res)

    if wrap_dicts:
        # Parsing wraps images and texts as interleaved dictionaries
        return [ConversationMessage(role=role, content=content)]  # type: ignore
    texts = cast(list[str], content)
    mm_placeholder_storage = mm_parser.mm_placeholder_storage()
    if mm_placeholder_storage:
        text_prompt = _get_full_multimodal_text_prompt(
            mm_placeholder_storage,
            texts,
            interleave_strings,
            multimodal_content_part_separator=multimodal_content_part_separator,
        )
    else:
        text_prompt = "\n".join(texts)

    return [ConversationMessage(role=role, content=text_prompt)]
```
**EN:** This function `_parse_chat_message_content_parts` checks or normalizes inputs.
**CN:** 该函数 `_parse_chat_message_content_parts` 检查或规范化输入。

### Lines 1592-1607 — Function `_reject_reserved_placeholder_in_text`
```python
def _reject_reserved_placeholder_in_text(text: str, model_config: ModelConfig) -> None:
    """Reject user-supplied text parts that contains the reserved `prompt_embeds`
    placeholder sentinel.

    When the server accepts `prompt_embeds`, the placeholder token is
    registered as a single unsplittable special token on the tokenizer. Any
    user text that happens to contain the literal sequence would tokenize to
    the same ID and be mistaken for a splice point by the renderer, letting a
    caller move or inject splice positions via plain text content.
    """
    if model_config.enable_prompt_embeds and PROMPT_EMBEDS_PLACEHOLDER_TOKEN in text:
        raise ValueError(
            _RESERVED_PLACEHOLDER_IN_TEXT_ERROR.format(
                token=PROMPT_EMBEDS_PLACEHOLDER_TOKEN
            )
        )
```
**EN:** This function `_reject_reserved_placeholder_in_text` is documented as: Reject user-supplied text parts that contains the reserved `prompt_embeds` placeholder sentinel.
**CN:** 这里定义函数 `_reject_reserved_placeholder_in_text`，其文档字符串说明了主要职责与调用约定。

### Lines 1610-1718 — Function `_parse_chat_message_content_part`
```python
def _parse_chat_message_content_part(
    part: ChatCompletionContentPartParam,
    mm_parser: BaseMultiModalContentParser,
    *,
    wrap_dicts: bool,
    interleave_strings: bool,
) -> _ContentPart | None:
    """Parses a single part of a conversation. If wrap_dicts is True,
    structured dictionary pieces for texts and images will be
    wrapped in dictionaries, i.e., {"type": "text", "text", ...} and
    {"type": "image"}, respectively. Otherwise multimodal data will be
    handled by mm_parser, and texts will be returned as strings to be joined
    with multimodal placeholders.
    """
    if isinstance(part, str):  # Handle plain text parts
        _reject_reserved_placeholder_in_text(part, mm_parser.model_config)
        if wrap_dicts:
            return {"type": "text", "text": part}
        return part
    # Handle structured dictionary parts
    part_type, content = _parse_chat_message_content_mm_part(part)
    # if part_type is text/refusal/image_url/audio_url/video_url/input_audio but
    # content is None, log a warning and skip
    if part_type in PART_TYPES_TO_SKIP_NONE_CONTENT and content is None:
        logger.warning(
            "Skipping multimodal part '%s' (type: '%s') "
            "with empty / unparsable content.",
            part,
...
        return {"type": modality}
    if modality == "prompt_embeds":
        # Emit the renderer token inline regardless of `interleave_strings`,
        # prompt_embeds are spliced at the token offset so position matters.
        # Falling back to front-padding via `missing_placeholders` would
        # reorder them relative to surrounding text.
        return PROMPT_EMBEDS_PLACEHOLDER_TOKEN
    return MODALITY_PLACEHOLDERS_MAP[modality] if interleave_strings else None
```
**EN:** This function `_parse_chat_message_content_part` is documented as: Parses a single part of a conversation.
**CN:** 这里定义函数 `_parse_chat_message_content_part`，其文档字符串说明了主要职责与调用约定。

### Lines 1722-1723 — Module constants and state
```python
_AssistantParser = partial(cast, ChatCompletionAssistantMessageParam)
_ToolParser = partial(cast, ChatCompletionToolMessageParam)
```
**EN:** This block initializes `_AssistantParser`, `_ToolParser`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `_AssistantParser`, `_ToolParser`，为后续逻辑准备模块级常量或共享状态。

### Lines 1726-1800 — Function `_parse_chat_message_content`
```python
def _parse_chat_message_content(
    message: ChatCompletionMessageParam,
    mm_tracker: BaseMultiModalItemTracker,
    content_format: ChatTemplateContentFormat,
    interleave_strings: bool,
    mm_processor_kwargs: dict[str, Any] | None = None,
) -> list[ConversationMessage]:
    role = message["role"]
    content = message.get("content")
    reasoning = message.get("reasoning")

    if content is None:
        content = []
    elif isinstance(content, str):
        content = [ChatCompletionContentPartTextParam(type="text", text=content)]
    result = _parse_chat_message_content_parts(
        role,
        content,  # type: ignore
        mm_tracker,
        wrap_dicts=(content_format == "openai"),
        interleave_strings=interleave_strings,
        mm_processor_kwargs=mm_processor_kwargs,
    )

    for result_msg in result:
        if role == "assistant":
            parsed_msg = _AssistantParser(message)

...
            result_msg["name"] = message["name"]

        if "task" in message and isinstance(message["task"], str):
            result_msg["task"] = message["task"]

        if role == "developer":
            result_msg["tools"] = message.get("tools", None)
    return result
```
**EN:** This function `_parse_chat_message_content` checks or normalizes inputs.
**CN:** 该函数 `_parse_chat_message_content` 检查或规范化输入。

### Lines 1803-1826 — Function `_postprocess_messages`
```python
def _postprocess_messages(messages: list[ConversationMessage]) -> None:
    # per the Transformers docs & maintainers, tool call arguments in
    # assistant-role messages with tool_calls need to be dicts not JSON str -
    # this is how tool-use chat templates will expect them moving forwards
    # so, for messages that have tool_calls, parse the string (which we get
    # from openAI format) to dict
    for message in messages:
        if message["role"] == "assistant" and "tool_calls" in message:
            tool_calls = message.get("tool_calls")
            if not isinstance(tool_calls, list):
                continue

            if len(tool_calls) == 0:
                # Drop empty tool_calls to keep templates on the normal assistant path.
                message.pop("tool_calls", None)
                continue

            for item in tool_calls:
                # if arguments is None or empty string, set to {}
                if content := item["function"].get("arguments"):
                    if not isinstance(content, (dict, list)):
                        item["function"]["arguments"] = json.loads(content)
                else:
                    item["function"]["arguments"] = {}
```
**EN:** This function `_postprocess_messages` implements the `postprocess messages` step within the module flow.
**CN:** 该函数 `_postprocess_messages` 实现了模块流程中的“postprocessmessages”步骤。

### Lines 1829-1865 — Function `parse_chat_messages`
```python
def parse_chat_messages(
    messages: list[ChatCompletionMessageParam],
    model_config: ModelConfig,
    content_format: ChatTemplateContentFormat,
    media_io_kwargs: dict[str, dict[str, Any]] | None = None,
    mm_processor_kwargs: dict[str, Any] | None = None,
) -> tuple[
    list[ConversationMessage],
    MultiModalDataDict | None,
    MultiModalUUIDDict | None,
]:
    conversation: list[ConversationMessage] = []
    mm_tracker = MultiModalItemTracker(
        model_config,
        media_io_kwargs=media_io_kwargs,
    )

    for msg in messages:
        sub_messages = _parse_chat_message_content(
            msg,
            mm_tracker,
            content_format,
            interleave_strings=(
                content_format == "string"
                and model_config.multimodal_config is not None
                and model_config.multimodal_config.interleave_mm_strings
            ),
            mm_processor_kwargs=mm_processor_kwargs,
        )

        conversation.extend(sub_messages)

    _postprocess_messages(conversation)

    mm_data, mm_uuids = mm_tracker.resolve_items()

    return conversation, mm_data, mm_uuids
```
**EN:** This function `parse_chat_messages` checks or normalizes inputs.
**CN:** 该函数 `parse_chat_messages` 检查或规范化输入。

### Lines 1868-1904 — Function `parse_chat_messages_async`
```python
async def parse_chat_messages_async(
    messages: list[ChatCompletionMessageParam],
    model_config: ModelConfig,
    content_format: ChatTemplateContentFormat,
    media_io_kwargs: dict[str, dict[str, Any]] | None = None,
    mm_processor_kwargs: dict[str, Any] | None = None,
) -> tuple[
    list[ConversationMessage],
    MultiModalDataDict | None,
    MultiModalUUIDDict | None,
]:
    conversation: list[ConversationMessage] = []
    mm_tracker = AsyncMultiModalItemTracker(
        model_config,
        media_io_kwargs=media_io_kwargs,
    )

    for msg in messages:
        sub_messages = _parse_chat_message_content(
            msg,
            mm_tracker,
            content_format,
            interleave_strings=(
                content_format == "string"
                and model_config.multimodal_config is not None
                and model_config.multimodal_config.interleave_mm_strings
            ),
            mm_processor_kwargs=mm_processor_kwargs,
        )

        conversation.extend(sub_messages)

    _postprocess_messages(conversation)

    mm_data, mm_uuids = await mm_tracker.resolve_items()

    return conversation, mm_data, mm_uuids
```
**EN:** This async function `parse_chat_messages_async` checks or normalizes inputs.
**CN:** 该异步函数 `parse_chat_messages_async` 检查或规范化输入。

### Lines 1907-1913 — Function `get_history_tool_calls_cnt`
```python
def get_history_tool_calls_cnt(conversation: list[ConversationMessage]):
    idx = 0
    for msg in conversation:
        if msg["role"] == "assistant":
            tool_calls = msg.get("tool_calls")
            idx += len(list(tool_calls)) if tool_calls is not None else 0  # noqa
    return idx
```
**EN:** This function `get_history_tool_calls_cnt` implements the `gets history tool calls cnt` step within the module flow.
**CN:** 该函数 `get_history_tool_calls_cnt` 实现了模块流程中的“获取historytoolcallscnt”步骤。

### Lines 1916-1916 — Module constants
```python
_KIMI_MODEL_TYPES = ("kimi_k2", "kimi_k25")
```
**EN:** This block initializes `_KIMI_MODEL_TYPES`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `_KIMI_MODEL_TYPES`，为后续逻辑准备模块级常量或共享状态。

### Lines 1919-1927 — Function `get_tool_call_id_type`
```python
def get_tool_call_id_type(model_config: ModelConfig) -> str:
    """Return the tool-call ID type for a given model configuration."""
    hf_overrides = getattr(model_config, "hf_overrides", None)
    if model_config.hf_text_config.model_type in _KIMI_MODEL_TYPES or (
        isinstance(hf_overrides, dict)
        and hf_overrides.get("model_type") in _KIMI_MODEL_TYPES
    ):
        return "kimi_k2"
    return "random"
```
**EN:** This function `get_tool_call_id_type` is documented as: Return the tool-call ID type for a given model configuration.
**CN:** 这里定义函数 `get_tool_call_id_type`，其文档字符串说明了主要职责与调用约定。

### Lines 1930-1935 — Function `make_tool_call_id`
```python
def make_tool_call_id(id_type: str = "random", func_name=None, idx=None):
    if id_type == "kimi_k2":
        return f"functions.{func_name}:{idx}"
    else:
        # by default return random
        return f"chatcmpl-tool-{random_uuid()}"
```
**EN:** This function `make_tool_call_id` implements the `makes tool call id` step within the module flow.
**CN:** 该函数 `make_tool_call_id` 实现了模块流程中的“生成toolcallid”步骤。

## Key Concepts / 关键概念
- Async request handling / 异步请求处理
- Schema validation with Pydantic / 使用 Pydantic 进行模式校验
- Structured request/response models / 结构化请求/响应模型
- Tokenization or token-level processing / 分词或 token 级处理
- Prompt or multimodal rendering / 提示词或多模态渲染
- Batch workflow handling / 批处理工作流
- Pooling task support / 池化任务支持
- Embedding generation / 嵌入生成

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `asyncio`, `json`, `abc`, `collections`, `dataclasses`, `functools`, `itertools`, `pathlib`, `typing`
- **Third-party / 第三方**: `openai`, `openai_harmony`, `PIL`, `pydantic`, `typing_extensions`, `torch`, `transformers`
- **vLLM Internal / vLLM 内部**: `vllm`, `vllm.config`, `vllm.exceptions`, `vllm.inputs`, `vllm.logger`, `vllm.model_executor.models`, `vllm.multimodal`, `vllm.multimodal.inputs`, `vllm.multimodal.media`, `vllm.multimodal.processing`, `vllm.renderers.embed_utils`, `vllm.utils`
