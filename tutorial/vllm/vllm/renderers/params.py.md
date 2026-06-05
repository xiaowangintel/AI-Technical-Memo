# params.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/renderers/params.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements rendering adapters used to turn structured inputs into prompts or outputs. / 实现把结构化输入转换为提示词或输出内容的渲染适配层。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-25)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
from dataclasses import dataclass, field
from typing import TYPE_CHECKING, Any, Literal, TypeVar

from vllm.exceptions import VLLMValidationError
from vllm.inputs import EmbedsPrompt, TextPrompt, TokensPrompt
from vllm.logger import init_logger
from vllm.multimodal.media.connector import merge_media_io_kwargs
from vllm.tokenizers import TokenizerLike
from vllm.utils.import_utils import LazyLoader

if TYPE_CHECKING:
    import torch

    from vllm.entrypoints.chat_utils import ChatTemplateContentFormatOption
else:
    torch = LazyLoader("torch", globals(), "torch")

    ChatTemplateContentFormatOption = object

logger = init_logger(__name__)


_S = TypeVar("_S", list[int], "torch.Tensor")
```
**EN:** Sets up the module with standard-library support such as `dataclasses`, `typing`, external packages such as `torch`, vLLM modules such as `vllm.exceptions`, `vllm.inputs`, `vllm.logger`. It prepares the symbols later used by `ChatParams`, `TokenizeParams`, `merge_kwargs`, `recursively_merge_kwargs`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.exceptions`, `vllm.inputs`, `vllm.logger` 等 vLLM 内部依赖。 这些准备工作为后续的 `ChatParams`, `TokenizeParams`, `merge_kwargs`, `recursively_merge_kwargs` 提供上下文。

### merge_kwargs (lines 28-40)
```python
def merge_kwargs(
    defaults: dict[str, Any] | None,
    overrides: dict[str, Any] | None,
    /,
    *,
    unset_values: tuple[object, ...] = (None, "auto"),
) -> dict[str, Any]:
    if defaults is None:
        defaults = {}
    if overrides is None:
        overrides = {}

    return defaults | {k: v for k, v in overrides.items() if v not in unset_values}
```
**EN:** `merge_kwargs` merges related settings or data into a normalized result. It mainly works with `defaults`, `overrides`, `unset_values`. Inside the body, it relies on `overrides.items` to complete the main steps.
**CN:** `merge_kwargs` 负责合并相关配置或数据并生成规范化结果。 它主要处理 `defaults`, `overrides`, `unset_values` 等参数。 实现过程中会调用 `overrides.items` 等函数完成关键步骤。

### recursively_merge_kwargs (lines 43-68)
```python
def recursively_merge_kwargs(
    defaults: dict[str, Any] | None,
    overrides: dict[str, Any] | None,
    /,
    *,
    unset_values: tuple[object, ...] = (None, "auto"),
) -> dict[str, Any]:
    if defaults is None:
        defaults = {}
    if overrides is None:
        overrides = {}

    merged = dict(defaults)

    for k, v in overrides.items():
        if v in unset_values:
            continue

        if k in merged and isinstance(merged[k], dict) and isinstance(v, dict):
            merged[k] = recursively_merge_kwargs(
                merged[k], v, unset_values=unset_values
            )
        else:
            merged[k] = v

    return merged
```
**EN:** `recursively_merge_kwargs` implements helper logic used by this module. It mainly works with `defaults`, `overrides`, `unset_values`. Inside the body, it relies on `overrides.items`, `recursively_merge_kwargs` to complete the main steps.
**CN:** `recursively_merge_kwargs` 负责实现本模块使用的辅助逻辑。 它主要处理 `defaults`, `overrides`, `unset_values` 等参数。 实现过程中会调用 `overrides.items`, `recursively_merge_kwargs` 等函数完成关键步骤。

### ChatParams (lines 72-125)
```python
class ChatParams:
    """Configuration to control how to parse chat messages."""

    chat_template: str | None = None
    """The chat template to apply."""

    chat_template_content_format: "ChatTemplateContentFormatOption" = "auto"
    """The format of the chat template."""

    chat_template_kwargs: dict[str, Any] = field(default_factory=dict)
    """The kwargs to pass to the chat template."""

    media_io_kwargs: dict[str, dict[str, Any]] | None = None
    """Per-modality kwargs for media I/O (loading/decoding images, videos, etc.)."""

    mm_processor_kwargs: dict[str, Any] | None = None
    """The kwargs to pass to the multi-modal processor."""

    def with_defaults(
        self,
        default_chat_template_kwargs: dict[str, Any] | None = None,
        default_media_io_kwargs: dict[str, dict[str, Any]] | None = None,
        default_mm_processor_kwargs: dict[str, Any] | None = None,
    ):
        if (
            not default_chat_template_kwargs
            and not default_media_io_kwargs
            and not default_mm_processor_kwargs
    # ...
    def get_apply_chat_template_kwargs(self) -> dict[str, Any]:
        """The arguments to pass to `tokenizer.apply_chat_template`."""
        return merge_kwargs(
            self.chat_template_kwargs,
            dict(chat_template=self.chat_template, return_dict=False),
        )
```
**EN:** `ChatParams`: Configuration to control how to parse chat messages. Key methods include `with_defaults`, `get_apply_chat_template_kwargs`.
**CN:** `ChatParams` 是该文件中的核心类，用于封装与 `ChatParams` 相关的状态和行为。 关键方法包括 `with_defaults`, `get_apply_chat_template_kwargs`。

### TokenizeParams overview (lines 129-465)
```python
class TokenizeParams:
    """Configuration to control how prompts are tokenized."""

    max_total_tokens: int | None
    """
    Maximum allowed number of input + output tokens.
    
    Usually, this refers to the model's context length.
    """

    max_output_tokens: int = 0
    """Maximum requested number of output tokens."""

    pad_prompt_tokens: int | None = None
    """
    Number of tokens to pad to:
    - `None` means no padding.
    - `-1` maps to `max_input_tokens`.
    """

    truncate_prompt_tokens: int | None = None
    """
    Number of tokens to keep:
    - `None` means no truncation.
    - `-1` maps to `max_input_tokens`.
    """

    # ...
```
**EN:** `TokenizeParams`: Configuration to control how prompts are tokenized. Key methods include `max_input_tokens`, `with_kwargs`, `get_encode_kwargs`, `apply_pre_tokenization`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `TokenizeParams` 是该文件中的核心类，用于封装与 `TokenizeParams` 相关的状态和行为。 关键方法包括 `max_input_tokens`, `with_kwargs`, `get_encode_kwargs`, `apply_pre_tokenization`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### TokenizeParams.get_encode_kwargs (lines 288-312)
```python
    def get_encode_kwargs(self) -> dict[str, Any]:
        """The arguments to pass to `tokenizer.encode`."""
        max_length = self.truncate_prompt_tokens
        if max_length is not None and max_length < 0:
            max_length = self.max_input_tokens
        elif max_length is None and self.max_input_tokens is not None:
            # This prevents tokenization from taking up more resources than necessary
            # while still failing `self._token_len_check` as expected by users
            max_length = self.max_input_tokens + 1

        # Left-side truncation requires the full token sequence so we can
        # slice from the end in _token_truncation.  Disable HF-level
        # truncation (which would incorrectly truncate from the right for
        # pooling models) and let _token_truncation handle it.
        if self.truncation_side == "left":
            return dict(
                truncation=False,
                add_special_tokens=self.add_special_tokens,
            )

        return dict(
            truncation=max_length is not None,
            max_length=max_length,
            add_special_tokens=self.add_special_tokens,
        )
```
**EN:** `get_encode_kwargs`: The arguments to pass to `tokenizer.encode`.
**CN:** `get_encode_kwargs` 负责获取流水线所需的数据或状态。

### TokenizeParams.with_kwargs (lines 229-286)
```python
    def with_kwargs(self, **tokenization_kwargs: Any):
        max_length = tokenization_kwargs.pop("max_length", self.max_input_tokens)
        pad_prompt_tokens = tokenization_kwargs.pop(
            "pad_prompt_tokens", self.pad_prompt_tokens
        )
        truncate_prompt_tokens = tokenization_kwargs.pop(
            "truncate_prompt_tokens", self.truncate_prompt_tokens
        )
        do_lower_case = tokenization_kwargs.pop("do_lower_case", self.do_lower_case)
        add_special_tokens = tokenization_kwargs.pop(
            "add_special_tokens", self.add_special_tokens
        )
        needs_detokenization = tokenization_kwargs.pop(
            "needs_detokenization", self.needs_detokenization
        )

        # https://huggingface.co/docs/transformers/en/pad_truncation
        if padding := tokenization_kwargs.pop("padding", None):
            if padding == "max_length":
                pad_prompt_tokens = max_length
    # ...
            truncate_prompt_tokens=truncate_prompt_tokens,
            truncation_side=self.truncation_side,
            do_lower_case=do_lower_case,
            add_special_tokens=add_special_tokens,
            needs_detokenization=needs_detokenization,
        )
```
**EN:** `with_kwargs` implements helper logic used by this module. It mainly works with `**tokenization_kwargs`. Inside the body, it relies on `tokenization_kwargs.pop`, `TokenizeParams`, `logger.warning` to complete the main steps.
**CN:** `with_kwargs` 负责实现本模块使用的辅助逻辑。 它主要处理 `**tokenization_kwargs` 等参数。 实现过程中会调用 `tokenization_kwargs.pop`, `TokenizeParams`, `logger.warning` 等函数完成关键步骤。

### TokenizeParams.apply_post_tokenization (lines 443-465)
```python
    def apply_post_tokenization(
        self,
        tokenizer: TokenizerLike | None,
        prompt: TokensPrompt | EmbedsPrompt,
    ) -> TokensPrompt | EmbedsPrompt:
        """
        Ensure that the prompt meets the requirements set out by this config.
        If that is not possible, raise a `VLLMValidationError`.

        This method is run after tokenization occurs.
        """
        if "prompt_token_ids" in prompt:
            prompt["prompt_token_ids"] = self._validate_tokens(  # type: ignore[typeddict-unknown-key]
                tokenizer,
                prompt["prompt_token_ids"],  # type: ignore[typeddict-item]
            )
        if "prompt_embeds" in prompt:
            prompt["prompt_embeds"] = self._validate_tokens(  # type: ignore[typeddict-unknown-key]
                tokenizer,
                prompt["prompt_embeds"],  # type: ignore[typeddict-item]
            )

        return prompt
```
**EN:** `apply_post_tokenization`: Ensure that the prompt meets the requirements set out by this config. It mainly works with `tokenizer`, `prompt`. Inside the body, it relies on `self._validate_tokens` to complete the main steps.
**CN:** `apply_post_tokenization` 负责实现本模块使用的辅助逻辑。 它主要处理 `tokenizer`, `prompt` 等参数。 实现过程中会调用 `self._validate_tokens` 等函数完成关键步骤。

### TokenizeParams.apply_pre_tokenization (lines 355-368)
```python
    def apply_pre_tokenization(
        self,
        tokenizer: TokenizerLike | None,
        prompt: TextPrompt,
    ) -> TextPrompt:
        """
        Ensure that the prompt meets the requirements set out by this config.
        If that is not possible, raise a `VLLMValidationError`.

        This method is run before tokenization occurs.
        """
        prompt["prompt"] = self._validate_text(tokenizer, prompt["prompt"])

        return prompt
```
**EN:** `apply_pre_tokenization`: Ensure that the prompt meets the requirements set out by this config. It mainly works with `tokenizer`, `prompt`. Inside the body, it relies on `self._validate_text` to complete the main steps.
**CN:** `apply_pre_tokenization` 负责实现本模块使用的辅助逻辑。 它主要处理 `tokenizer`, `prompt` 等参数。 实现过程中会调用 `self._validate_text` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`ChatParams`**: Core class that organizes module behavior. / **`ChatParams`**：组织模块行为的核心类。
- **`TokenizeParams`**: Core class that organizes module behavior. / **`TokenizeParams`**：组织模块行为的核心类。
- **`merge_kwargs`**: Key helper or entry point in this file. / **`merge_kwargs`**：本文件中的关键辅助函数或入口。
- **`recursively_merge_kwargs`**: Key helper or entry point in this file. / **`recursively_merge_kwargs`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: dataclasses, typing
- **Third-party / 第三方**: torch
- **Internal vLLM / vLLM 内部依赖**: vllm.exceptions, vllm.inputs, vllm.logger, vllm.multimodal.media.connector, vllm.tokenizers, vllm.utils.import_utils, vllm.entrypoints.chat_utils
