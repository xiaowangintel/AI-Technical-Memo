# engine.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/inputs/engine.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Schema and utilities for inputs to the engine client (`LLMEngine`/`AsyncLLM`). / 该文件的核心目的为：Schema and utilities for inputs to the engine client (`LLMEngine`/`AsyncLLM`).

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 1-1)
```python
"""Schema and utilities for inputs to the engine client (`LLMEngine`/`AsyncLLM`)."""
```
**EN:** This opening docstring states the module's intent, expected usage, and any important assumptions before the implementation details begin.
**CN:** 开头的文档字符串先说明模块意图、预期用法以及实现展开前需要了解的重要假设。

### Imports (lines 5-8)
```python
from collections.abc import Mapping, Sequence

from typing import TYPE_CHECKING, Literal, TypeAlias

from typing_extensions import NotRequired, TypedDict, assert_never
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 114-264)
```python
MultiModalHashes: TypeAlias = Mapping[str, list[str]]

MultiModalPlaceholders: TypeAlias = Mapping[str, Sequence["PlaceholderRange"]]

DecoderOnlyEngineInput: TypeAlias = TokensInput | EmbedsInput | MultiModalInput

EncoderInput: TypeAlias = TokensInput | MultiModalEncDecInput

DecoderEngineInput: TypeAlias = TokensInput | MultiModalInput

SingletonInput: TypeAlias = DecoderOnlyEngineInput | MultiModalEncDecInput

EngineInput: TypeAlias = DecoderOnlyEngineInput | EncoderDecoderInput
```
**EN:** This constant/configuration block defines `MultiModalHashes`, `MultiModalPlaceholders`, `DecoderOnlyEngineInput`, `EncoderInput`, `DecoderEngineInput`, `SingletonInput`, `EngineInput`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `MultiModalHashes`, `MultiModalPlaceholders`, `DecoderOnlyEngineInput`, `EncoderInput`, `DecoderEngineInput`, `SingletonInput`, `EngineInput`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Large-file note
```python
# Focused on representative top-level definitions for a large module.
# The full file contains additional helpers following the same patterns.
```
**EN:** Because the file is large, the analysis below focuses on the most important top-level definitions and leaves repetitive helpers summarized at a higher level.
**CN:** 由于文件较大，下面的分析聚焦最重要的顶层定义；其余重复性辅助逻辑将以更高层次方式概括。

### Class `_InputOptions` (lines 16-26)
```python
class _InputOptions(TypedDict):
    """
    Additional options available to all
    [`SingletonInput`][vllm.inputs.engine.SingletonInput] types.
    """

    arrival_time: NotRequired[float]
    """The time when the input was received (before rendering)."""

    cache_salt: NotRequired[str]
    """Optional cache salt to be used for prefix caching."""
```
**EN:** Class `_InputOptions` is a structured building block in this module. It inherits from `TypedDict`. The class docstring says: Additional options available to all [`SingletonInput`][vllm.inputs.engine.SingletonInput] types.
**CN:** 类 `_InputOptions` 是该模块中的结构化构件，继承自 `TypedDict`。 类文档说明：Additional options available to all [`SingletonInput`][vllm.inputs.engine.SingletonInput] types.

### Class `TokensInput` (lines 29-39)
```python
class TokensInput(_InputOptions):
    """Represents token-based input to the engine."""

    type: Literal["token"]
    """The type of input."""

    prompt_token_ids: list[int]
    """The token IDs of the prompt."""

    prompt: NotRequired[str]
    """The prompt text corresponding to the token IDs, if available."""
```
**EN:** Class `TokensInput` is a structured building block in this module. It inherits from `_InputOptions`. The class docstring says: Represents token-based input to the engine.
**CN:** 类 `TokensInput` 是该模块中的结构化构件，继承自 `_InputOptions`。 类文档说明：Represents token-based input to the engine.

### Function `tokens_input` (lines 42-59)
```python
def tokens_input(
    prompt_token_ids: list[int],
    *,
    prompt: str | None = None,
    cache_salt: str | None = None,
) -> TokensInput:
    """
    Construct [`TokensInput`][vllm.inputs.engine.TokensInput]
    from optional values.
    """
    inputs = TokensInput(type="token", prompt_token_ids=prompt_token_ids)

    if prompt is not None:
        inputs["prompt"] = prompt
    if cache_salt is not None:
        inputs["cache_salt"] = cache_salt

    return inputs
```
**EN:** Function `tokens_input` implements an encoding/decoding or token-transformation step. The docstring highlights: Construct [`TokensInput`][vllm.inputs.engine.TokensInput] from optional values. Key calls such as `TokensInput` show the concrete execution path.
**CN:** Function `tokens_input` 实现编码/解码或 Token 变换步骤。 文档字符串强调：Construct [`TokensInput`][vllm.inputs.engine.TokensInput] from optional values. 像 `TokensInput` 这样的关键调用展示了该代码块的具体执行路径。

### Class `EmbedsInput` (lines 62-85)
```python
class EmbedsInput(_InputOptions):
    """Represents embeddings-based input to the engine."""

    type: Literal["embeds"]
    """The type of input."""

    prompt_embeds: "torch.Tensor"
    """The embeddings of the prompt."""

    prompt: NotRequired[str]
    """The prompt text corresponding to the token IDs, if available."""

    prompt_token_ids: NotRequired[list[int]]
    """Token IDs of the rendered prompt. Only set for mixed-mode inputs
    (chat completion with `prompt_embeds` content parts). When present,
    `is_token_ids` MUST also be present and have the same length. 
    For pure-embeds inputs this field is absent."""

    is_token_ids: NotRequired[list[bool]]
    """Per-position mask for mixed-mode inputs. `True` means the position
    is a real token ID (use the model's embedding layer); `False` means
    the position uses a pre-computed embedding row from `prompt_embeds`.
    Length MUST equal `len(prompt_token_ids)`.
    For pure-embeds inputs this field is absent."""
```
**EN:** Class `EmbedsInput` is a structured building block in this module. It inherits from `_InputOptions`. The class docstring says: Represents embeddings-based input to the engine.
**CN:** 类 `EmbedsInput` 是该模块中的结构化构件，继承自 `_InputOptions`。 类文档说明：Represents embeddings-based input to the engine.

### Function `embeds_input` (lines 88-111)
```python
def embeds_input(
    prompt_embeds: "torch.Tensor",
    *,
    prompt: str | None = None,
    cache_salt: str | None = None,
    prompt_token_ids: list[int] | None = None,
    is_token_ids: list[bool] | None = None,
) -> EmbedsInput:
    """
    Construct [`EmbedsInput`][vllm.inputs.engine.EmbedsInput]
    from optional values.
    """
    inputs = EmbedsInput(type="embeds", prompt_embeds=prompt_embeds)

    if prompt is not None:
        inputs["prompt"] = prompt
    if cache_salt is not None:
        inputs["cache_salt"] = cache_salt
    if prompt_token_ids is not None:
        inputs["prompt_token_ids"] = prompt_token_ids
    if is_token_ids is not None:
        inputs["is_token_ids"] = is_token_ids

    return inputs
```
**EN:** Function `embeds_input` provides a reusable helper around the module's main workflow. The docstring highlights: Construct [`EmbedsInput`][vllm.inputs.engine.EmbedsInput] from optional values. Key calls such as `EmbedsInput` show the concrete execution path.
**CN:** Function `embeds_input` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Construct [`EmbedsInput`][vllm.inputs.engine.EmbedsInput] from optional values. 像 `EmbedsInput` 这样的关键调用展示了该代码块的具体执行路径。

### Class `MultiModalInput` (lines 126-148)
```python
class MultiModalInput(_InputOptions):
    """Represents multi-modal input to the engine."""

    type: Literal["multimodal"]
    """The type of input."""

    prompt_token_ids: list[int]
    """The processed token IDs which includes placeholder tokens."""

    prompt: NotRequired[str]
    """The prompt text corresponding to the token IDs, if available."""

    mm_kwargs: "MultiModalKwargsOptionalItems"
    """Keyword arguments to be directly passed to the model after batching."""

    mm_hashes: MultiModalHashes
    """The hashes of the multi-modal data."""

    mm_placeholders: MultiModalPlaceholders
    """
    For each modality, information about the placeholder tokens in
    `prompt_token_ids`.
    """
```
**EN:** Class `MultiModalInput` is a structured building block in this module. It inherits from `_InputOptions`. The class docstring says: Represents multi-modal input to the engine.
**CN:** 类 `MultiModalInput` 是该模块中的结构化构件，继承自 `_InputOptions`。 类文档说明：Represents multi-modal input to the engine.

### Function `mm_input` (lines 151-173)
```python
def mm_input(
    prompt_token_ids: list[int],
    mm_kwargs: "MultiModalKwargsOptionalItems",
    mm_hashes: MultiModalHashes,
    mm_placeholders: MultiModalPlaceholders,
    *,
    prompt: str | None = None,
    cache_salt: str | None = None,
) -> MultiModalInput:
    inputs = MultiModalInput(
        type="multimodal",
        prompt_token_ids=prompt_token_ids,
        mm_kwargs=mm_kwargs,
        mm_hashes=mm_hashes,
        mm_placeholders=mm_placeholders,
    )

    if prompt is not None:
        inputs["prompt"] = prompt
    if cache_salt is not None:
        inputs["cache_salt"] = cache_salt

    return inputs
```
**EN:** Function `mm_input` provides a reusable helper around the module's main workflow. Key calls such as `MultiModalInput` show the concrete execution path.
**CN:** Function `mm_input` 为模块主流程提供可复用的辅助逻辑。 像 `MultiModalInput` 这样的关键调用展示了该代码块的具体执行路径。

### Class `MultiModalEncDecInput` (lines 176-190)
```python
class MultiModalEncDecInput(MultiModalInput):
    """
    Represents multi-modal input to the engine for encoder-decoder models.

    Note:
        Even text-only encoder-decoder models are currently implemented
        as multi-modal models for convenience.
        (Example: https://github.com/vllm-project/bart-plugin)
    """

    encoder_prompt_token_ids: list[int]
    """The processed token IDs of the encoder prompt."""

    encoder_prompt: NotRequired[str]
    """The prompt text corresponding to the encoder token IDs, if available."""
```
**EN:** Class `MultiModalEncDecInput` is a structured building block in this module. It inherits from `MultiModalInput`. The class docstring says: Represents multi-modal input to the engine for encoder-decoder models.
**CN:** 类 `MultiModalEncDecInput` 是该模块中的结构化构件，继承自 `MultiModalInput`。 类文档说明：Represents multi-modal input to the engine for encoder-decoder models.

### Function `mm_enc_dec_input` (lines 193-215)
```python
def mm_enc_dec_input(
    encoder_inputs: MultiModalInput,
    decoder_prompt_token_ids: list[int],
    *,
    decoder_prompt: str | None = None,
) -> MultiModalEncDecInput:
    inputs = MultiModalEncDecInput(
        type="multimodal",
        prompt_token_ids=decoder_prompt_token_ids,
        encoder_prompt_token_ids=encoder_inputs["prompt_token_ids"],
        mm_kwargs=encoder_inputs["mm_kwargs"],
        mm_hashes=encoder_inputs["mm_hashes"],
        mm_placeholders=encoder_inputs["mm_placeholders"],
    )

    if decoder_prompt is not None:
        inputs["prompt"] = decoder_prompt
    if "prompt" in encoder_inputs:
        inputs["encoder_prompt"] = encoder_inputs["prompt"]
    if "cache_salt" in encoder_inputs:
        inputs["cache_salt"] = encoder_inputs["cache_salt"]

    return inputs
```
**EN:** Function `mm_enc_dec_input` provides a reusable helper around the module's main workflow. Key calls such as `MultiModalEncDecInput` show the concrete execution path.
**CN:** Function `mm_enc_dec_input` 为模块主流程提供可复用的辅助逻辑。 像 `MultiModalEncDecInput` 这样的关键调用展示了该代码块的具体执行路径。

### Class `EncoderDecoderInput` (lines 239-254)
```python
class EncoderDecoderInput(TypedDict):
    """
    A rendered [`EncoderDecoderPrompt`][vllm.inputs.llm.EncoderDecoderPrompt]
    which can be passed to `LLMEngine.add_request` or `AsyncLLM.add_request`.
    """

    type: Literal["enc_dec"]

    encoder_prompt: EncoderInput
    """The inputs for the encoder portion."""

    decoder_prompt: DecoderEngineInput
    """The inputs for the decoder portion."""

    arrival_time: NotRequired[float]
    """The time when the input was received (before rendering)."""
```
**EN:** Class `EncoderDecoderInput` is a structured building block in this module. It inherits from `TypedDict`. The class docstring says: A rendered [`EncoderDecoderPrompt`][vllm.inputs.llm.EncoderDecoderPrompt] which can be passed to `LLMEngine.add_request` or `AsyncLLM.add_request`.
**CN:** 类 `EncoderDecoderInput` 是该模块中的结构化构件，继承自 `TypedDict`。 类文档说明：A rendered [`EncoderDecoderPrompt`][vllm.inputs.llm.EncoderDecoderPrompt] which can be passed to `LLMEngine.add_request` or `AsyncLLM.add_request`.

### Function `_validate_enc_input` (lines 271-286)
```python
def _validate_enc_input(enc_input: SingletonInput) -> EncoderInput:
    if enc_input["type"] == "embeds":
        raise ValueError(
            "Embedding inputs are not supported for encoder-decoder models"
        )

    if (
        enc_input["type"] == "multimodal"
        and "encoder_prompt_token_ids" not in enc_input
    ):
        raise RuntimeError(
            "You should register an encoder-decoder multi-modal processor "
            "for encoder-decoder models."
        )

    return enc_input  # type: ignore[return-value]
```
**EN:** Function `_validate_enc_input` validates assumptions and guards module invariants. Key calls such as `ValueError`, `RuntimeError` show the concrete execution path.
**CN:** Function `_validate_enc_input` 负责校验前置条件并保护模块不变量。 像 `ValueError`, `RuntimeError` 这样的关键调用展示了该代码块的具体执行路径。

### Function `build_enc_dec_input` (lines 315-362)
```python
def build_enc_dec_input(
    encoder_input: SingletonInput,
    decoder_input: SingletonInput | None,
    decoder_start_token_id: int,
    skip_decoder_start_token: bool = False,
) -> EncoderDecoderInput:
    enc_input = _validate_enc_input(encoder_input)

    if decoder_input is None:
        dec_input: DecoderEngineInput = enc_input
    else:
        dec_input = _validate_dec_input(decoder_input)

    enc_input_new: EncoderInput
    dec_input_new: DecoderEngineInput

    if enc_input["type"] == "multimodal":
        enc_input_new = tokens_input(
            enc_input["encoder_prompt_token_ids"],
            prompt=enc_input.get("encoder_prompt"),
        )
        dec_input_new = mm_input(
            prompt_token_ids=dec_input["prompt_token_ids"],
            prompt=dec_input.get("prompt"),
            mm_kwargs=enc_input["mm_kwargs"],
    # ... omitted for brevity ...
        decoder_prompt=dec_input_new,
    )
```
**EN:** Function `build_enc_dec_input` constructs derived objects, runtime state, or helper structures. Key calls such as `_validate_enc_input`, `_validate_dec_input`, `tokens_input`, `enc_input.get`, `mm_input` show the concrete execution path.
**CN:** Function `build_enc_dec_input` 负责构造派生对象、运行时状态或辅助结构。 像 `_validate_enc_input`, `_validate_dec_input`, `tokens_input`, `enc_input.get`, `mm_input` 这样的关键调用展示了该代码块的具体执行路径。

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
- **Standard library / 标准库**: `from collections.abc import Mapping, Sequence`, `from typing import TYPE_CHECKING, Literal, TypeAlias`
- **Third-party / 第三方**: `from typing_extensions import NotRequired, TypedDict, assert_never`
