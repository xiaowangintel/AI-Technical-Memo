# llm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/inputs/llm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Schema and utilities for input prompts to the LLM API. / 该文件的核心目的为：Schema and utilities for input prompts to the LLM API.

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 1-1)
```python
"""Schema and utilities for input prompts to the LLM API."""
```
**EN:** This opening docstring states the module's intent, expected usage, and any important assumptions before the implementation details begin.
**CN:** 开头的文档字符串先说明模块意图、预期用法以及实现展开前需要了解的重要假设。

### Imports (lines 5-8)
```python
from collections.abc import Mapping, Sequence

from typing import TYPE_CHECKING, Any, TypeAlias, TypeVar, final

from typing_extensions import NotRequired, TypedDict
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 16-215)
```python
_T = TypeVar("_T")

ModalityData: TypeAlias = _T | list[_T | None] | None

MultiModalDataDict: TypeAlias = Mapping[str, ModalityData[Any]]

MultiModalUUIDDict: TypeAlias = Mapping[str, Sequence[str | None] | str]

DecoderOnlyPrompt: TypeAlias = (
    str | TextPrompt | list[int] | TokensPrompt | EmbedsPrompt
)

EncoderPrompt: TypeAlias = str | TextPrompt | list[int] | TokensPrompt

DecoderPrompt: TypeAlias = str | TextPrompt | list[int] | TokensPrompt

EncoderDecoderPrompt: TypeAlias = EncoderPrompt | ExplicitEncoderDecoderPrompt

SingletonPrompt: TypeAlias = DecoderOnlyPrompt | EncoderPrompt | DecoderPrompt

PromptType: TypeAlias = DecoderOnlyPrompt | EncoderDecoderPrompt
```
**EN:** This constant/configuration block defines `_T`, `ModalityData`, `MultiModalDataDict`, `MultiModalUUIDDict`, `DecoderOnlyPrompt`, `EncoderPrompt`, `DecoderPrompt`, `EncoderDecoderPrompt`, `SingletonPrompt`, `PromptType`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `_T`, `ModalityData`, `MultiModalDataDict`, `MultiModalUUIDDict`, `DecoderOnlyPrompt`, `EncoderPrompt`, `DecoderPrompt`, `EncoderDecoderPrompt`, `SingletonPrompt`, `PromptType`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Class `MultiModalDataBuiltins` (lines 29-42)
```python
class MultiModalDataBuiltins(TypedDict, total=False):
    """Type annotations for modality types predefined by vLLM."""

    image: ModalityData["ImageItem"]
    """The input image(s)."""

    video: ModalityData["VideoItem"]
    """The input video(s)."""

    audio: ModalityData["AudioItem"]
    """The input audio(s)."""

    vision_chunk: ModalityData["VisionChunk"]
    """The input visual atom(s) - unified modality for images and video chunks."""
```
**EN:** Class `MultiModalDataBuiltins` is a structured building block in this module. It inherits from `TypedDict`. The class docstring says: Type annotations for modality types predefined by vLLM.
**CN:** 类 `MultiModalDataBuiltins` 是该模块中的结构化构件，继承自 `TypedDict`。 类文档说明：Type annotations for modality types predefined by vLLM.

### Class `_PromptOptions` (lines 64-96)
```python
class _PromptOptions(TypedDict):
    """
    Additional options available to all
    [`SingletonPrompt`][vllm.inputs.llm.SingletonPrompt] types.
    """

    multi_modal_data: NotRequired[MultiModalDataDict | None]
    """
    Optional multi-modal data to pass to the model,
    if the model supports it.
    """

    mm_processor_kwargs: NotRequired[dict[str, Any] | None]
    """
    Optional multi-modal processor kwargs to be forwarded to the
    multimodal input mapper & processor. Note that if multiple modalities
    have registered mappers etc for the model being considered, we attempt
    to pass the mm_processor_kwargs to each of them.
    """

    multi_modal_uuids: NotRequired[MultiModalUUIDDict]
    """
    Optional user-specified UUIDs for multimodal items, mapped by modality.
    # ... omitted for brevity ...
    Optional cache salt to be used for prefix caching.
    """
```
**EN:** Class `_PromptOptions` is a structured building block in this module. It inherits from `TypedDict`. The class docstring says: Additional options available to all [`SingletonPrompt`][vllm.inputs.llm.SingletonPrompt] types.
**CN:** 类 `_PromptOptions` 是该模块中的结构化构件，继承自 `TypedDict`。 类文档说明：Additional options available to all [`SingletonPrompt`][vllm.inputs.llm.SingletonPrompt] types.

### Class `TextPrompt` (lines 99-103)
```python
class TextPrompt(_PromptOptions):
    """Schema for a text prompt."""

    prompt: str
    """The input text to be tokenized before passing to the model."""
```
**EN:** Class `TextPrompt` is a structured building block in this module. It inherits from `_PromptOptions`. The class docstring says: Schema for a text prompt.
**CN:** 类 `TextPrompt` 是该模块中的结构化构件，继承自 `_PromptOptions`。 类文档说明：Schema for a text prompt.

### Class `TokensPrompt` (lines 106-116)
```python
class TokensPrompt(_PromptOptions):
    """Schema for a tokenized prompt."""

    prompt_token_ids: list[int]
    """A list of token IDs to pass to the model."""

    prompt: NotRequired[str]
    """The prompt text corresponding to the token IDs, if available."""

    token_type_ids: NotRequired[list[int]]
    """A list of token type IDs to pass to the cross encoder model."""
```
**EN:** Class `TokensPrompt` is a structured building block in this module. It inherits from `_PromptOptions`. The class docstring says: Schema for a tokenized prompt.
**CN:** 类 `TokensPrompt` 是该模块中的结构化构件，继承自 `_PromptOptions`。 类文档说明：Schema for a tokenized prompt.

### Class `EmbedsPrompt` (lines 119-137)
```python
class EmbedsPrompt(_PromptOptions):
    """Schema for a prompt provided via token embeddings."""

    prompt_embeds: "torch.Tensor"
    """The embeddings of the prompt."""

    prompt: NotRequired[str]
    """The prompt text corresponding to the token embeddings, if available."""

    prompt_token_ids: NotRequired[list[int]]
    """Token IDs for mixed-mode inputs (chat completion with
    `prompt_embeds` content parts). The tokens at positions where 
    `prompt_is_token_ids` is `False` are placeholder tokens that 
    get replaced by entries from `prompt_embeds` in the forward pass."""

    prompt_is_token_ids: NotRequired[list[bool]]
    """Per-position mask, `True` uses the real token ID, `False` uses
    the corresponding entry from `prompt_embeds`. 
    Must be the same length as `prompt_token_ids` when both are set."""
```
**EN:** Class `EmbedsPrompt` is a structured building block in this module. It inherits from `_PromptOptions`. The class docstring says: Schema for a prompt provided via token embeddings.
**CN:** 类 `EmbedsPrompt` 是该模块中的结构化构件，继承自 `_PromptOptions`。 类文档说明：Schema for a prompt provided via token embeddings.

### Class `ExplicitEncoderDecoderPrompt` (lines 179-195)
```python
class ExplicitEncoderDecoderPrompt(TypedDict):
    """
    Schema for a pair of encoder and decoder singleton prompts.

    Note:
        This schema is not valid for decoder-only models.
    """

    encoder_prompt: EncoderPrompt
    """The prompt for the encoder part of the model."""

    decoder_prompt: DecoderPrompt | None
    """
    The prompt for the decoder part of the model.

    Passing `None` will cause the prompt to be inferred automatically.
    """
```
**EN:** Class `ExplicitEncoderDecoderPrompt` is a structured building block in this module. It inherits from `TypedDict`. The class docstring says: Schema for a pair of encoder and decoder singleton prompts.
**CN:** 类 `ExplicitEncoderDecoderPrompt` 是该模块中的结构化构件，继承自 `TypedDict`。 类文档说明：Schema for a pair of encoder and decoder singleton prompts.

### Class `DataPrompt` (lines 223-233)
```python
class DataPrompt(_PromptOptions):
    """
    Represents generic inputs that are converted to
    [`PromptType`][vllm.inputs.llm.PromptType] by IO processor plugins.
    """

    data: Any
    """The input data."""

    data_format: str
    """The input data format."""
```
**EN:** Class `DataPrompt` is a structured building block in this module. It inherits from `_PromptOptions`. The class docstring says: Represents generic inputs that are converted to [`PromptType`][vllm.inputs.llm.PromptType] by IO processor plugins.
**CN:** 类 `DataPrompt` 是该模块中的结构化构件，继承自 `_PromptOptions`。 类文档说明：Represents generic inputs that are converted to [`PromptType`][vllm.inputs.llm.PromptType] by IO processor plugins.

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。
- **Multimodality / 多模态**
  - **EN:** The code contains modality-specific paths for text, image, audio, or video data.
  - **CN:** 代码包含面向文本、图像、音频或视频数据的模态专用路径。
- **Structured types / 结构化类型**
  - **EN:** Dataclasses, typed dictionaries, or aliases are used to make runtime contracts explicit.
  - **CN:** 通过 dataclass、类型化字典或别名来显式表达运行时契约。
- **Caching / 缓存**
  - **EN:** Some definitions are designed to reuse computed state and avoid repeated work.
  - **CN:** 部分定义旨在复用已计算状态，避免重复工作。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from collections.abc import Mapping, Sequence`, `from typing import TYPE_CHECKING, Any, TypeAlias, TypeVar, final`
- **Third-party / 第三方**: `from typing_extensions import NotRequired, TypedDict`
