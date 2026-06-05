# preprocess.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/renderers/inputs/preprocess.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Schemas and utilities for preprocessing inputs / 该模块围绕 `preprocess` 相关逻辑组织实现，提供与源文件相匹配的核心功能。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-26)
```python
"""
Schemas and utilities for preprocessing inputs.
"""

# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
from collections.abc import Mapping, Sequence
from typing import TYPE_CHECKING, NamedTuple, TypeAlias, TypedDict, overload

from vllm.inputs import (
    EmbedsPrompt,
    EngineInput,
    ExplicitEncoderDecoderPrompt,
    PromptType,
    SingletonPrompt,
    TextPrompt,
    TokensPrompt,
)
from vllm.utils import length_from_prompt_token_ids_or_embeds
from vllm.utils.collection_utils import is_list_of

if TYPE_CHECKING:
    import torch

    from vllm.config import ModelConfig
    from vllm.entrypoints.chat_utils import ChatCompletionMessageParam
```
**EN:** Sets up the module with standard-library support such as `collections.abc`, `typing`, external packages such as `torch`, vLLM modules such as `vllm.inputs`, `vllm.utils`, `vllm.utils.collection_utils`. It prepares the symbols later used by `EncoderDecoderDictPrompt`, `PromptComponents`, `prompt_to_seq`, `prompt_to_seq`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.inputs`, `vllm.utils`, `vllm.utils.collection_utils` 等 vLLM 内部依赖。 这些准备工作为后续的 `EncoderDecoderDictPrompt`, `PromptComponents`, `prompt_to_seq`, `prompt_to_seq` 提供上下文。

### prompt_to_seq (lines 30-32)
```python
def prompt_to_seq(
    prompt_or_prompts: SingletonPrompt | bytes | Sequence[SingletonPrompt | bytes],
) -> Sequence[SingletonPrompt]: ...
```
**EN:** `prompt_to_seq` implements helper logic used by this module. It mainly works with `prompt_or_prompts`.
**CN:** `prompt_to_seq` 负责实现本模块使用的辅助逻辑。 它主要处理 `prompt_or_prompts` 等参数。

### prompt_to_seq (lines 36-39)
```python
def prompt_to_seq(  # type: ignore[misc]
    prompt_or_prompts: ExplicitEncoderDecoderPrompt
    | Sequence[ExplicitEncoderDecoderPrompt],
) -> Sequence[ExplicitEncoderDecoderPrompt]: ...
```
**EN:** `prompt_to_seq` implements helper logic used by this module. It mainly works with `prompt_or_prompts`.
**CN:** `prompt_to_seq` 负责实现本模块使用的辅助逻辑。 它主要处理 `prompt_or_prompts` 等参数。

### prompt_to_seq (lines 43-45)
```python
def prompt_to_seq(  # type: ignore[misc]
    prompt_or_prompts: PromptType | Sequence[PromptType],
) -> Sequence[PromptType]: ...
```
**EN:** `prompt_to_seq` implements helper logic used by this module. It mainly works with `prompt_or_prompts`.
**CN:** `prompt_to_seq` 负责实现本模块使用的辅助逻辑。 它主要处理 `prompt_or_prompts` 等参数。

### prompt_to_seq (lines 48-56)
```python
def prompt_to_seq(
    prompt_or_prompts: PromptType | bytes | Sequence[PromptType | bytes],
) -> Sequence[PromptType]:
    if isinstance(prompt_or_prompts, (dict, str, bytes)) or (
        len(prompt_or_prompts) > 0 and is_list_of(prompt_or_prompts, int)
    ):
        return [prompt_or_prompts]  # type: ignore[list-item]

    return prompt_or_prompts  # type: ignore[return-value]
```
**EN:** `prompt_to_seq` implements helper logic used by this module. It mainly works with `prompt_or_prompts`. Inside the body, it relies on `is_list_of` to complete the main steps.
**CN:** `prompt_to_seq` 负责实现本模块使用的辅助逻辑。 它主要处理 `prompt_or_prompts` 等参数。 实现过程中会调用 `is_list_of` 等函数完成关键步骤。

### conversation_to_seq (lines 59-68)
```python
def conversation_to_seq(
    conversation_or_conversations: list["ChatCompletionMessageParam"]
    | Sequence[list["ChatCompletionMessageParam"]],
) -> Sequence[list["ChatCompletionMessageParam"]]:
    if len(conversation_or_conversations) > 0 and is_list_of(
        conversation_or_conversations, dict
    ):
        return [conversation_or_conversations]  # type: ignore[list-item]

    return conversation_or_conversations  # type: ignore[return-value]
```
**EN:** `conversation_to_seq` implements helper logic used by this module. It mainly works with `conversation_or_conversations`. Inside the body, it relies on `is_list_of` to complete the main steps.
**CN:** `conversation_to_seq` 负责实现本模块使用的辅助逻辑。 它主要处理 `conversation_or_conversations` 等参数。 实现过程中会调用 `is_list_of` 等函数完成关键步骤。

### EncoderDecoderDictPrompt (lines 92-100)
```python
class EncoderDecoderDictPrompt(TypedDict):
    """
    A [`EncoderDecoderPrompt`][vllm.inputs.llm.EncoderDecoderPrompt]
    that has been standardized into a dictionary.
    """

    encoder_prompt: EncoderDictPrompt

    decoder_prompt: DecoderDictPrompt | None
```
**EN:** `EncoderDecoderDictPrompt`: A [`EncoderDecoderPrompt`][vllm.inputs.llm.EncoderDecoderPrompt] that has been standardized into a dictionary. It extends `TypedDict`.
**CN:** `EncoderDecoderDictPrompt` 是该文件中的核心类，用于封装与 `EncoderDecoderDictPrompt` 相关的状态和行为。 它继承自 `TypedDict`。

### _validate_prompt_dict (lines 119-129)
```python
def _validate_prompt_dict(prompt: Mapping[str, object]) -> None:
    """Reject malformed dict prompts before renderer tokenization."""
    if (
        "prompt" not in prompt
        or "prompt_token_ids" in prompt
        or "prompt_embeds" in prompt
    ):
        return

    if not isinstance(prompt["prompt"], str):
        raise TypeError("Prompt text should be a string")
```
**EN:** `_validate_prompt_dict`: Reject malformed dict prompts before renderer tokenization. It mainly works with `prompt`. Inside the body, it relies on `TypeError` to complete the main steps.
**CN:** `_validate_prompt_dict` 负责校验取值并在约束不满足时抛出清晰错误。 它主要处理 `prompt` 等参数。 实现过程中会调用 `TypeError` 等函数完成关键步骤。

### parse_dec_only_prompt (lines 132-160)
```python
def parse_dec_only_prompt(prompt: PromptType | object) -> DecoderOnlyDictPrompt:
    """
    Parse a prompt for a decoder-only model and normalize it to a dictionary.
    """
    if isinstance(prompt, str):
        return TextPrompt(prompt=prompt)

    if isinstance(prompt, list):
        if not is_list_of(prompt, int):
            raise TypeError("Token prompt should be a list of integers")

        return TokensPrompt(prompt_token_ids=prompt)

    if isinstance(prompt, dict):
        if "encoder_prompt" in prompt:
            raise TypeError("Cannot pass encoder-decoder prompt to decoder-only models")

        _validate_prompt_dict(prompt)

        if (
            "prompt" in prompt
            or "prompt_token_ids" in prompt
            or "prompt_embeds" in prompt
        ):
            return prompt  # type: ignore[return-value]

        raise TypeError("Prompt dictionary must contain text, tokens, or embeddings")

    raise TypeError("Prompt should be a string, list of tokens, or dictionary")
```
**EN:** `parse_dec_only_prompt`: Parse a prompt for a decoder-only model and normalize it to a dictionary. It mainly works with `prompt`. Inside the body, it relies on `TypeError`, `TextPrompt`, `TokensPrompt` to complete the main steps.
**CN:** `parse_dec_only_prompt` 负责把原始输入解析为结构化对象。 它主要处理 `prompt` 等参数。 实现过程中会调用 `TypeError`, `TextPrompt`, `TokensPrompt` 等函数完成关键步骤。

### _parse_enc_prompt (lines 163-184)
```python
def _parse_enc_prompt(prompt: PromptType | object) -> EncoderDictPrompt:
    if isinstance(prompt, str):
        return TextPrompt(prompt=prompt)

    if isinstance(prompt, list):
        if not is_list_of(prompt, int):
            raise TypeError("Token prompt should be a list of integers")

        return TokensPrompt(prompt_token_ids=prompt)

    if isinstance(prompt, dict):
        _validate_prompt_dict(prompt)

        if "prompt_embeds" in prompt:
            raise TypeError("Cannot pass embeddings prompt to encoder-decoder models")

        if "prompt" in prompt or "prompt_token_ids" in prompt:
            return prompt  # type: ignore[return-value]

        raise TypeError("Prompt dictionary must contain text or tokens")

    raise TypeError("Prompt should be a string, list of tokens, or dictionary")
```
**EN:** `_parse_enc_prompt` parses raw inputs into structured objects. It mainly works with `prompt`. Inside the body, it relies on `TypeError`, `TextPrompt`, `TokensPrompt` to complete the main steps.
**CN:** `_parse_enc_prompt` 负责把原始输入解析为结构化对象。 它主要处理 `prompt` 等参数。 实现过程中会调用 `TypeError`, `TextPrompt`, `TokensPrompt` 等函数完成关键步骤。

### _parse_dec_prompt (lines 187-215)
```python
def _parse_dec_prompt(prompt: PromptType | object) -> DecoderDictPrompt:
    if isinstance(prompt, str):
        return TextPrompt(prompt=prompt)

    if isinstance(prompt, list):
        if not is_list_of(prompt, int):
            raise TypeError("Token prompt should be a list of integers")

        return TokensPrompt(prompt_token_ids=prompt)

    if isinstance(prompt, dict):
        _validate_prompt_dict(prompt)

        if "prompt_embeds" in prompt:
            raise TypeError("Cannot pass embeddings prompt to encoder-decoder models")

        if (
            "multi_modal_data" in prompt
            or "mm_processor_kwargs" in prompt
            or "multi_modal_uuids" in prompt
        ):
            raise TypeError("Cannot pass multi-modal inputs to decoder prompt")

        if "prompt" in prompt or "prompt_token_ids" in prompt:
            return prompt  # type: ignore[return-value]

        raise TypeError("Prompt dictionary must contain text or tokens")

    raise TypeError("Prompt should be a string, list of tokens, or dictionary")
```
**EN:** `_parse_dec_prompt` parses raw inputs into structured objects. It mainly works with `prompt`. Inside the body, it relies on `TypeError`, `TextPrompt`, `TokensPrompt` to complete the main steps.
**CN:** `_parse_dec_prompt` 负责把原始输入解析为结构化对象。 它主要处理 `prompt` 等参数。 实现过程中会调用 `TypeError`, `TextPrompt`, `TokensPrompt` 等函数完成关键步骤。

### parse_enc_dec_prompt (lines 218-232)
```python
def parse_enc_dec_prompt(prompt: PromptType | object) -> EncoderDecoderDictPrompt:
    """
    Parse a prompt for an encoder-decoder model and normalize it to a dictionary.
    """
    if isinstance(prompt, dict) and "encoder_prompt" in prompt:
        enc_prompt = prompt["encoder_prompt"]  # type: ignore[typeddict-item]
        dec_prompt = prompt["decoder_prompt"]  # type: ignore[typeddict-item]
    else:
        enc_prompt = prompt
        dec_prompt = None

    return EncoderDecoderDictPrompt(
        encoder_prompt=_parse_enc_prompt(enc_prompt),
        decoder_prompt=None if dec_prompt is None else _parse_dec_prompt(dec_prompt),
    )
```
**EN:** `parse_enc_dec_prompt`: Parse a prompt for an encoder-decoder model and normalize it to a dictionary. It mainly works with `prompt`. Inside the body, it relies on `EncoderDecoderDictPrompt`, `_parse_enc_prompt`, `_parse_dec_prompt` to complete the main steps.
**CN:** `parse_enc_dec_prompt` 负责把原始输入解析为结构化对象。 它主要处理 `prompt` 等参数。 实现过程中会调用 `EncoderDecoderDictPrompt`, `_parse_enc_prompt`, `_parse_dec_prompt` 等函数完成关键步骤。

### parse_model_prompt (lines 235-239)
```python
def parse_model_prompt(model_config: "ModelConfig", prompt: object):
    if model_config.is_encoder_decoder:
        return parse_enc_dec_prompt(prompt)

    return parse_dec_only_prompt(prompt)
```
**EN:** `parse_model_prompt` parses raw inputs into structured objects. It mainly works with `model_config`, `prompt`. Inside the body, it relies on `parse_dec_only_prompt`, `parse_enc_dec_prompt` to complete the main steps.
**CN:** `parse_model_prompt` 负责把原始输入解析为结构化对象。 它主要处理 `model_config`, `prompt` 等参数。 实现过程中会调用 `parse_dec_only_prompt`, `parse_enc_dec_prompt` 等函数完成关键步骤。

### PromptComponents (lines 242-245)
```python
class PromptComponents(NamedTuple):
    text: str | None = None
    token_ids: list[int] | None = None
    embeds: "torch.Tensor | None" = None
```
**EN:** Defines the `PromptComponents` class used by this module. It extends `NamedTuple`.
**CN:** `PromptComponents` 是该文件中的核心类，用于封装与 `PromptComponents` 相关的状态和行为。 它继承自 `NamedTuple`。

### extract_target_prompt (lines 248-253)
```python
def extract_target_prompt(model_config: "ModelConfig", prompt: object):
    return (
        parse_enc_dec_prompt(prompt)["encoder_prompt"]
        if model_config.is_encoder_decoder
        else parse_dec_only_prompt(prompt)
    )
```
**EN:** `extract_target_prompt` implements helper logic used by this module. It mainly works with `model_config`, `prompt`. Inside the body, it relies on `parse_dec_only_prompt`, `parse_enc_dec_prompt` to complete the main steps.
**CN:** `extract_target_prompt` 负责实现本模块使用的辅助逻辑。 它主要处理 `model_config`, `prompt` 等参数。 实现过程中会调用 `parse_dec_only_prompt`, `parse_enc_dec_prompt` 等函数完成关键步骤。

### extract_prompt_components (lines 256-266)
```python
def extract_prompt_components(
    model_config: "ModelConfig",
    prompt: PromptType | EngineInput,
) -> PromptComponents:
    target_prompt = extract_target_prompt(model_config, prompt)

    return PromptComponents(
        text=target_prompt.get("prompt"),
        token_ids=target_prompt.get("prompt_token_ids"),
        embeds=target_prompt.get("prompt_embeds"),
    )
```
**EN:** `extract_prompt_components` implements helper logic used by this module. It mainly works with `model_config`, `prompt`. Inside the body, it relies on `extract_target_prompt`, `PromptComponents`, `target_prompt.get` to complete the main steps.
**CN:** `extract_prompt_components` 负责实现本模块使用的辅助逻辑。 它主要处理 `model_config`, `prompt` 等参数。 实现过程中会调用 `extract_target_prompt`, `PromptComponents`, `target_prompt.get` 等函数完成关键步骤。

### extract_prompt_len (lines 269-278)
```python
def extract_prompt_len(
    model_config: "ModelConfig",
    prompt: PromptType | EngineInput,
):
    target_prompt = extract_target_prompt(model_config, prompt)

    return length_from_prompt_token_ids_or_embeds(
        target_prompt.get("prompt_token_ids"),
        target_prompt.get("prompt_embeds"),
    )
```
**EN:** `extract_prompt_len` implements helper logic used by this module. It mainly works with `model_config`, `prompt`. Inside the body, it relies on `extract_target_prompt`, `length_from_prompt_token_ids_or_embeds`, `target_prompt.get` to complete the main steps.
**CN:** `extract_prompt_len` 负责实现本模块使用的辅助逻辑。 它主要处理 `model_config`, `prompt` 等参数。 实现过程中会调用 `extract_target_prompt`, `length_from_prompt_token_ids_or_embeds`, `target_prompt.get` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`EncoderDecoderDictPrompt`**: Core class that organizes module behavior. / **`EncoderDecoderDictPrompt`**：组织模块行为的核心类。
- **`PromptComponents`**: Core class that organizes module behavior. / **`PromptComponents`**：组织模块行为的核心类。
- **`prompt_to_seq`**: Key helper or entry point in this file. / **`prompt_to_seq`**：本文件中的关键辅助函数或入口。
- **`prompt_to_seq`**: Key helper or entry point in this file. / **`prompt_to_seq`**：本文件中的关键辅助函数或入口。
- **`prompt_to_seq`**: Key helper or entry point in this file. / **`prompt_to_seq`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: collections.abc, typing
- **Third-party / 第三方**: torch
- **Internal vLLM / vLLM 内部依赖**: vllm.inputs, vllm.utils, vllm.utils.collection_utils, vllm.config, vllm.entrypoints.chat_utils
