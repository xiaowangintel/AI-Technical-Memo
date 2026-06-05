# hf.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/renderers/hf.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements rendering adapters used to turn structured inputs into prompts or outputs. / 实现把结构化输入转换为提示词或输出内容的渲染适配层。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-100)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
from __future__ import annotations

import copy
import inspect
import itertools
import weakref
from collections import defaultdict, deque
from collections.abc import Sequence
from functools import lru_cache
from typing import TYPE_CHECKING, Any, Final, Literal, cast, overload

import jinja2
import jinja2.ext
import jinja2.meta
import jinja2.nodes
import jinja2.parser
import jinja2.sandbox
import torch
from typing_extensions import override

from vllm.entrypoints.chat_utils import (
    PROMPT_EMBEDS_PLACEHOLDER_TOKEN,
    # ...
    "tokenize=False."
)
_TOKENIZE_OVERRIDE_WARNING: Final[str] = (
    "Overriding `tokenize=False` to `True` because `prompt_embeds` "
    "post-processing requires tokenized IDs."
)
```
**EN:** Sets up the module with standard-library support such as `__future__`, `copy`, `inspect`, external packages such as `jinja2`, `jinja2.ext`, `jinja2.meta`, vLLM modules such as `vllm.entrypoints.chat_utils`, `vllm.inputs`, `vllm.inputs.engine`. It prepares the symbols later used by `AssistantTracker`, `HfRenderer`, `_ensure_prompt_embeds_placeholder_token`, `_build_prompt_embeds_updates`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.entrypoints.chat_utils`, `vllm.inputs`, `vllm.inputs.engine` 等 vLLM 内部依赖。 这些准备工作为后续的 `AssistantTracker`, `HfRenderer`, `_ensure_prompt_embeds_placeholder_token`, `_build_prompt_embeds_updates` 提供上下文。

### _ensure_prompt_embeds_placeholder_token (lines 103-126)
```python
def _ensure_prompt_embeds_placeholder_token(tokenizer: HfTokenizer) -> int:
    """Register `PROMPT_EMBEDS_PLACEHOLDER_TOKEN` as a special token and return
    its token ID."""
    cached = _PROMPT_EMBEDS_PLACEHOLDER_TOKEN_ID_CACHE.get(tokenizer)
    if cached is not None:
        return cached

    tokenizer.add_special_tokens(
        {"additional_special_tokens": [PROMPT_EMBEDS_PLACEHOLDER_TOKEN]}
    )

    ids = tokenizer.encode(PROMPT_EMBEDS_PLACEHOLDER_TOKEN, add_special_tokens=False)
    if len(ids) != 1:
        raise RuntimeError(
            _PROMPT_EMBEDS_PLACEHOLDER_TOKEN_ID_ERROR.format(
                token=PROMPT_EMBEDS_PLACEHOLDER_TOKEN,
                num_ids=len(ids),
                ids=ids,
            )
        )

    token_id = ids[0]
    _PROMPT_EMBEDS_PLACEHOLDER_TOKEN_ID_CACHE[tokenizer] = token_id
    return token_id
```
**EN:** `_ensure_prompt_embeds_placeholder_token`: Register `PROMPT_EMBEDS_PLACEHOLDER_TOKEN` as a special token and return its token ID. It mainly works with `tokenizer`. Inside the body, it relies on `_PROMPT_EMBEDS_PLACEHOLDER_TOKEN_ID_CACHE.get`, `tokenizer.add_special_tokens`, `tokenizer.encode` to complete the main steps.
**CN:** `_ensure_prompt_embeds_placeholder_token` 负责实现本模块使用的辅助逻辑。 它主要处理 `tokenizer` 等参数。 实现过程中会调用 `_PROMPT_EMBEDS_PLACEHOLDER_TOKEN_ID_CACHE.get`, `tokenizer.add_special_tokens`, `tokenizer.encode` 等函数完成关键步骤。

### _build_prompt_embeds_updates (lines 129-147)
```python
def _build_prompt_embeds_updates(
    prompt_embeds_tensors: Sequence[torch.Tensor],
    placeholder_token_id: int,
) -> MultiModalPromptUpdates:
    """Build `MultiModalPromptUpdates` for `prompt_embeds` expansion.

    Each tensor produces a `PromptReplacement` that maps
    `[placeholder_token_id]` -> `[placeholder_token_id] x N`
    (where `N = tensor.shape[0]`).
    """
    updates: list[Sequence[ResolvedPromptUpdate]] = []
    for i, tensor in enumerate(prompt_embeds_tensors):
        update = PromptReplacement(
            modality="prompt_embeds",
            target=[placeholder_token_id],
            replacement=[placeholder_token_id] * tensor.shape[0],
        )
        updates.append([update.resolve(item_idx=i)])
    return {"prompt_embeds": updates}
```
**EN:** `_build_prompt_embeds_updates`: Build `MultiModalPromptUpdates` for `prompt_embeds` expansion. It mainly works with `prompt_embeds_tensors`, `placeholder_token_id`. Inside the body, it relies on `PromptReplacement`, `updates.append`, `update.resolve` to complete the main steps.
**CN:** `_build_prompt_embeds_updates` 负责为下游执行构造对象。 它主要处理 `prompt_embeds_tensors`, `placeholder_token_id` 等参数。 实现过程中会调用 `PromptReplacement`, `updates.append`, `update.resolve` 等函数完成关键步骤。

### _build_prompt_embeds_positions (lines 164-189)
```python
def _build_prompt_embeds_positions(
    token_ids: list[int],
    num_tensors: int,
    mm_prompt_updates: MultiModalPromptUpdates,
) -> list[tuple[int, int]]:
    """Locate each prompt_embeds placeholder span in `token_ids`.

    Expects `token_ids` to already contain expanded N-token spans.
    Returns `[(start_idx, length), ...]` aligned with the tensors.
    """
    placeholders = find_mm_placeholders(
        prompt=token_ids,
        mm_prompt_updates=mm_prompt_updates,
        tokenizer=None,
    )
    features = placeholders.get("prompt_embeds", [])

    if len(features) != num_tensors:
        raise ValueError(
            _PROMPT_EMBEDS_PLACEHOLDER_SPAN_MISMATCH_ERROR.format(
                expected=num_tensors,
                actual=len(features),
            )
        )

    return [(f.start_idx, f.length) for f in features]
```
**EN:** `_build_prompt_embeds_positions`: Locate each prompt_embeds placeholder span in `token_ids`. It mainly works with `token_ids`, `num_tensors`, `mm_prompt_updates`. Inside the body, it relies on `find_mm_placeholders`, `placeholders.get`, `ValueError` to complete the main steps.
**CN:** `_build_prompt_embeds_positions` 负责为下游执行构造对象。 它主要处理 `token_ids`, `num_tensors`, `mm_prompt_updates` 等参数。 实现过程中会调用 `find_mm_placeholders`, `placeholders.get`, `ValueError` 等函数完成关键步骤。

### _build_mixed_prompt_embeds (lines 192-210)
```python
def _build_mixed_prompt_embeds(
    token_ids: list[int],
    prompt_embeds_tensors: Sequence[torch.Tensor],
    positions: list[tuple[int, int]],
) -> tuple[torch.Tensor, list[bool]]:
    """Build the full-length `prompt_embeds` tensor and the `is_token_ids`
    mask aligned to `token_ids`."""
    total_len = len(token_ids)
    hidden_size = prompt_embeds_tensors[0].shape[1]
    dtype = prompt_embeds_tensors[0].dtype

    full_embeds = torch.zeros(total_len, hidden_size, dtype=dtype)
    is_token_ids = torch.ones(total_len, dtype=torch.bool)

    for (start, length), tensor in zip(positions, prompt_embeds_tensors, strict=True):
        full_embeds[start : start + length] = tensor
        is_token_ids[start : start + length] = False

    return full_embeds, is_token_ids.tolist()
```
**EN:** `_build_mixed_prompt_embeds`: Build the full-length `prompt_embeds` tensor and the `is_token_ids` mask aligned to `token_ids`. It mainly works with `token_ids`, `prompt_embeds_tensors`, `positions`. Inside the body, it relies on `torch.zeros`, `torch.ones`, `is_token_ids.tolist` to complete the main steps.
**CN:** `_build_mixed_prompt_embeds` 负责为下游执行构造对象。 它主要处理 `token_ids`, `prompt_embeds_tensors`, `positions` 等参数。 实现过程中会调用 `torch.zeros`, `torch.ones`, `is_token_ids.tolist` 等函数完成关键步骤。

### _try_get_processor_chat_template (lines 222-262)
```python
def _try_get_processor_chat_template(
    tokenizer: HfTokenizer,
    *,
    trust_remote_code: bool,
) -> str | None:
    cache_key = (tokenizer.name_or_path, trust_remote_code)
    if cache_key in _PROCESSOR_CHAT_TEMPLATES:
        return _PROCESSOR_CHAT_TEMPLATES[cache_key]

    from transformers import (
        PreTrainedTokenizer,
        PreTrainedTokenizerFast,
        ProcessorMixin,
    )

    try:
        processor = cached_get_processor(
            tokenizer.name_or_path,
            processor_cls=(
                PreTrainedTokenizer,
                PreTrainedTokenizerFast,
                ProcessorMixin,
            ),
            trust_remote_code=trust_remote_code,
    # ...
            tokenizer.name_or_path,
            exc_info=True,
        )

    _PROCESSOR_CHAT_TEMPLATES[cache_key] = None
    return None
```
**EN:** `_try_get_processor_chat_template` implements helper logic used by this module. It mainly works with `tokenizer`, `trust_remote_code`. Inside the body, it relies on `cached_get_processor`, `logger.debug` to complete the main steps.
**CN:** `_try_get_processor_chat_template` 负责实现本模块使用的辅助逻辑。 它主要处理 `tokenizer`, `trust_remote_code` 等参数。 实现过程中会调用 `cached_get_processor`, `logger.debug` 等函数完成关键步骤。

### resolve_chat_template (lines 265-314)
```python
def resolve_chat_template(
    tokenizer: HfTokenizer,
    chat_template: str | None,
    tools: list[dict[str, Any]] | None,
    *,
    model_config: ModelConfig,
) -> str | None:
    # 1st priority: The given chat template
    if chat_template is not None:
        # Resolve template names (e.g. "tool_use") to actual Jinja content
        # so that downstream kwargs detection can parse template variables.
        return tokenizer.get_chat_template(chat_template, tools=tools)

    # 2nd priority: AutoProcessor chat template, unless tool calling is enabled
    if tools is None:
        chat_template = _try_get_processor_chat_template(
            tokenizer,
            trust_remote_code=model_config.trust_remote_code,
        )
        if chat_template is not None:
            return chat_template

    # 3rd priority: AutoTokenizer chat template
    try:
    # ...
    else:
        logger.debug_once(
            "There is no chat template fallback for %s", tokenizer.name_or_path
        )

    return chat_template
```
**EN:** `resolve_chat_template` resolves symbolic settings into concrete runtime values. It mainly works with `tokenizer`, `chat_template`, `tools`, `model_config`. Inside the body, it relies on `get_chat_template_fallback_path`, `tokenizer.get_chat_template`, `_try_get_processor_chat_template` to complete the main steps.
**CN:** `resolve_chat_template` 负责把符号化配置解析为具体运行时取值。 它主要处理 `tokenizer`, `chat_template`, `tools`, `model_config` 等参数。 实现过程中会调用 `get_chat_template_fallback_path`, `tokenizer.get_chat_template`, `_try_get_processor_chat_template` 等函数完成关键步骤。

### AssistantTracker overview (lines 535-543)
```python
class AssistantTracker(jinja2.ext.Extension):
    tags = {"generation"}

    def parse(self, parser: jinja2.parser.Parser) -> jinja2.nodes.Node:
        lineno = next(parser.stream).lineno
        body = parser.parse_statements(("name:endgeneration",), drop_needle=True)
        call = self.call_method("_generation_support")
        call_block = jinja2.nodes.CallBlock(call, [], [], body)
        return call_block.set_lineno(lineno)
```
**EN:** Defines the `AssistantTracker` class used by this module. It extends `jinja2.ext.Extension`. Key methods include `parse`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `AssistantTracker` 是该文件中的核心类，用于封装与 `AssistantTracker` 相关的状态和行为。 它继承自 `jinja2.ext.Extension`。 关键方法包括 `parse`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### AssistantTracker.parse (lines 538-543)
```python
    def parse(self, parser: jinja2.parser.Parser) -> jinja2.nodes.Node:
        lineno = next(parser.stream).lineno
        body = parser.parse_statements(("name:endgeneration",), drop_needle=True)
        call = self.call_method("_generation_support")
        call_block = jinja2.nodes.CallBlock(call, [], [], body)
        return call_block.set_lineno(lineno)
```
**EN:** `parse` parses raw inputs into structured objects. It mainly works with `parser`. Inside the body, it relies on `parser.parse_statements`, `self.call_method`, `jinja2.nodes.CallBlock` to complete the main steps.
**CN:** `parse` 负责把原始输入解析为结构化对象。 它主要处理 `parser` 等参数。 实现过程中会调用 `parser.parse_statements`, `self.call_method`, `jinja2.nodes.CallBlock` 等函数完成关键步骤。

### resolve_chat_template_kwargs (lines 581-609)
```python
def resolve_chat_template_kwargs(
    tokenizer: HfTokenizer,
    chat_template: str,
    chat_template_kwargs: dict[str, Any],
    raise_on_unexpected: bool = True,
) -> dict[str, Any]:
    # We exclude chat_template from kwargs here, because
    # chat template has been already resolved at this stage
    unexpected_vars = {"chat_template", "tokenize"}
    if raise_on_unexpected and (
        unexpected_in_kwargs := unexpected_vars & chat_template_kwargs.keys()
    ):
        raise ValueError(
            "Found unexpected chat template kwargs from request: "
            f"{unexpected_in_kwargs}"
        )

    fn_kw = {
        k
        for k in chat_template_kwargs
        if supports_kw(tokenizer.apply_chat_template, k, allow_var_kwargs=False)
    }
    template_vars = _cached_resolve_chat_template_kwargs(chat_template)

    # Allow standard HF parameters even if tokenizer uses **kwargs to receive them
    hf_base_params = _get_hf_base_chat_template_params()

    accept_vars = (fn_kw | template_vars | hf_base_params) - unexpected_vars
    return {k: v for k, v in chat_template_kwargs.items() if k in accept_vars}
```
**EN:** `resolve_chat_template_kwargs` resolves symbolic settings into concrete runtime values. It mainly works with `tokenizer`, `chat_template`, `chat_template_kwargs`, `raise_on_unexpected`. Inside the body, it relies on `_cached_resolve_chat_template_kwargs`, `_get_hf_base_chat_template_params`, `ValueError` to complete the main steps.
**CN:** `resolve_chat_template_kwargs` 负责把符号化配置解析为具体运行时取值。 它主要处理 `tokenizer`, `chat_template`, `chat_template_kwargs`, `raise_on_unexpected` 等参数。 实现过程中会调用 `_cached_resolve_chat_template_kwargs`, `_get_hf_base_chat_template_params`, `ValueError` 等函数完成关键步骤。

### safe_apply_chat_template (lines 634-687)
```python
def safe_apply_chat_template(
    model_config: ModelConfig,
    tokenizer: HfTokenizer,
    conversation: list[ConversationMessage],
    *,
    tools: list[dict[str, Any]] | None = None,
    chat_template: str | None = None,
    tokenize: bool = True,
    **kwargs,
) -> str | list[int]:
    chat_template = resolve_chat_template(
        tokenizer,
        chat_template=chat_template,
        tools=tools,
        model_config=model_config,
    )
    if chat_template is None:
        raise ChatTemplateResolutionError(
            "As of transformers v4.44, default chat template is no longer "
            "allowed, so you must provide a chat template if the tokenizer "
            "does not define one."
        )

    resolved_kwargs = resolve_chat_template_kwargs(
    # ...
        # Log and report any library-related exceptions for further
        # investigation.
        logger.exception(
            "An error occurred in `transformers` while applying chat template"
        )
        raise ValueError(str(e)) from e
```
**EN:** `safe_apply_chat_template` implements helper logic used by this module. It mainly works with `model_config`, `tokenizer`, `conversation`, `tools`. Inside the body, it relies on `resolve_chat_template`, `resolve_chat_template_kwargs`, `ChatTemplateResolutionError` to complete the main steps.
**CN:** `safe_apply_chat_template` 负责实现本模块使用的辅助逻辑。 它主要处理 `model_config`, `tokenizer`, `conversation`, `tools` 等参数。 实现过程中会调用 `resolve_chat_template`, `resolve_chat_template_kwargs`, `ChatTemplateResolutionError` 等函数完成关键步骤。

### rebuild_mm_uuids_from_mm_data (lines 690-722)
```python
def rebuild_mm_uuids_from_mm_data(
    mm_uuids: MultiModalUUIDDict,
    mm_data: MultiModalDataDict,
) -> MultiModalUUIDDict:
    """Rebuild mm_uuids after vision_chunk processing.

    When videos are split into chunks, the original UUIDs need to be updated
    to reflect the new UUIDs generated for each chunk.

    Args:
        mm_uuids: Original UUIDs dictionary
        mm_data: Processed multimodal data with vision_chunk items

    Returns:
        Updated UUIDs dictionary with chunk UUIDs
    """
    vision_chunks = mm_data.get("vision_chunk")
    if vision_chunks is None:
        return mm_uuids

    assert all(isinstance(item, dict) for item in vision_chunks), (
        "Expected all vision_chunk items to be dicts"
    )
    vision_chunks = cast(list[dict[str, Any]], vision_chunks)
    # ...

    if vision_chunk_uuids:
        mm_uuids = dict(mm_uuids)
        mm_uuids["vision_chunk"] = vision_chunk_uuids

    return mm_uuids
```
**EN:** `rebuild_mm_uuids_from_mm_data`: Rebuild mm_uuids after vision_chunk processing. It mainly works with `mm_uuids`, `mm_data`. Inside the body, it relies on `mm_data.get`, `all`, `cast` to complete the main steps.
**CN:** `rebuild_mm_uuids_from_mm_data` 负责实现本模块使用的辅助逻辑。 它主要处理 `mm_uuids`, `mm_data` 等参数。 实现过程中会调用 `mm_data.get`, `all`, `cast` 等函数完成关键步骤。

### Additional repeated patterns / 其他重复模式
```python
# Large file omitted for brevity
# ...
```
**EN:** The remaining code mostly expands the same abstractions with more wrappers, constants, or schema variants. The analysis above focuses on the control points that shape overall behavior.
**CN:** 剩余代码主要在相同抽象之上继续展开，补充更多包装函数、常量或模式变体。上面的分析聚焦于决定整体行为的关键控制点。

## Key Concepts / 关键概念
- **`AssistantTracker`**: Core class that organizes module behavior. / **`AssistantTracker`**：组织模块行为的核心类。
- **`HfRenderer`**: Core class that organizes module behavior. / **`HfRenderer`**：组织模块行为的核心类。
- **`_ensure_prompt_embeds_placeholder_token`**: Key helper or entry point in this file. / **`_ensure_prompt_embeds_placeholder_token`**：本文件中的关键辅助函数或入口。
- **`_build_prompt_embeds_updates`**: Key helper or entry point in this file. / **`_build_prompt_embeds_updates`**：本文件中的关键辅助函数或入口。
- **`_expand_prompt_embeds_placeholders`**: Key helper or entry point in this file. / **`_expand_prompt_embeds_placeholders`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: __future__, copy, inspect, itertools, weakref, collections, collections.abc, functools, typing
- **Third-party / 第三方**: jinja2, jinja2.ext, jinja2.meta, jinja2.nodes, jinja2.parser, jinja2.sandbox, torch, typing_extensions, transformers, transformers.utils.chat_template_utils
- **Internal vLLM / vLLM 内部依赖**: vllm.entrypoints.chat_utils, vllm.inputs, vllm.inputs.engine, vllm.logger, vllm.multimodal.hasher, vllm.multimodal.inputs, vllm.multimodal.processing.processor, vllm.tokenizers.hf, vllm.transformers_utils.chat_templates, vllm.transformers_utils.processor, vllm.utils.async_utils, vllm.utils.func_utils, .base, .inputs.preprocess
