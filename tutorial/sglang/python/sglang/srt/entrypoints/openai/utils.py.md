# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/openai/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements utils logic for the OpenAI-compatible serving layer. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 工具 相关的逻辑，并服务于 OpenAI 兼容服务层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2: Import runtime dependencies / 导入运行时依赖
```python
import logging
from typing import Any, Dict, List, Optional, Union
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 4-4: Import runtime dependencies / 导入运行时依赖
```python
import torch
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 6-12: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.entrypoints.openai.protocol import (
    CachedTokensDetails,
    ChatCompletionRequest,
    CompletionRequest,
    LogProbs,
    StreamOptions,
)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 14-14: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 17-51: Implement to openai style logprobs / 实现to OpenAI style logprobs
```python
def to_openai_style_logprobs(
    input_token_logprobs=None,
    output_token_logprobs=None,
    input_top_logprobs=None,
    output_top_logprobs=None,
):
    ret_logprobs = LogProbs()

    def append_token_logprobs(token_logprobs):
        for logprob, _, token_text in token_logprobs:
            ret_logprobs.tokens.append(token_text)
            ret_logprobs.token_logprobs.append(logprob)

            # Not supported yet
            ret_logprobs.text_offset.append(-1)

    def append_top_logprobs(top_logprobs):
        for tokens in top_logprobs:
            if tokens is not None:
                ret_logprobs.top_logprobs.append(
                    {token[2]: token[0] for token in tokens}
                )
            else:
                ret_logprobs.top_logprobs.append(None)

    if input_token_logprobs is not None:
        append_token_logprobs(input_token_logprobs)
    if output_token_logprobs is not None:
        append_token_logprobs(output_token_logprobs)
    if input_top_logprobs is not None:
        append_top_logprobs(input_top_logprobs)
    if output_top_logprobs is not None:
        append_top_logprobs(output_top_logprobs)

    return ret_logprobs
```
**EN:** This block implements the function `to_openai_style_logprobs(input_token_logprobs, output_token_logprobs, input_top_logprobs, output_top_logprobs)`. It focuses on handling the utils responsibilities represented by `to_openai_style_logprobs`, providing reusable behavior for the utils pipeline.
**CN:** 该代码块实现函数 `to_openai_style_logprobs(input_token_logprobs, output_token_logprobs, input_top_logprobs, output_top_logprobs)`。它围绕 `to_openai_style_logprobs` 所承担的 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 54-76: Implement process hidden states from ret / 实现process hidden states from ret
```python
def process_hidden_states_from_ret(
    ret_item: Dict[str, Any],
    request: Union[
        ChatCompletionRequest,
        CompletionRequest,
    ],
) -> Optional[List]:
    """Process hidden states from a ret item in non-streaming response.

    Args:
        ret_item: Response item containing meta_info
        request: The original request object

    Returns:
        Processed hidden states for the last token, or None
    """
    if not request.return_hidden_states:
        return None

    hidden_states = ret_item["meta_info"].get("hidden_states", None)
    if hidden_states is not None:
        hidden_states = hidden_states[-1] if len(hidden_states) > 1 else []
    return hidden_states
```
**EN:** This block implements the function `process_hidden_states_from_ret(ret_item, request)`. It focuses on Process hidden states from a ret item in non-streaming response., providing reusable behavior for the utils pipeline.
**CN:** 该代码块实现函数 `process_hidden_states_from_ret(ret_item, request)`。它围绕 `process_hidden_states_from_ret` 所承担的 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 79-93: Implement should include usage / 实现should include usage
```python
def should_include_usage(
    stream_options: StreamOptions | None, stream_response_default_include_usage: bool
) -> tuple[bool, bool]:
    # When stream_options are specified in the request
    if stream_options:
        include_usage = (
            stream_options.include_usage or stream_response_default_include_usage
        )
        continuous_usage_stats = bool(stream_options.continuous_usage_stats)
    else:
        include_usage, continuous_usage_stats = (
            stream_response_default_include_usage,
            False,
        )
    return include_usage, continuous_usage_stats
```
**EN:** This block implements the function `should_include_usage(stream_options, stream_response_default_include_usage)`. It focuses on handling the utils responsibilities represented by `should_include_usage`, providing reusable behavior for the utils pipeline.
**CN:** 该代码块实现函数 `should_include_usage(stream_options, stream_response_default_include_usage)`。它围绕 `should_include_usage` 所承担的 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 96-106: Implement process routed experts from ret / 实现process routed experts from ret
```python
def process_routed_experts_from_ret(
    ret_item: Dict[str, Any],
    request: Union[
        ChatCompletionRequest,
        CompletionRequest,
    ],
) -> Optional[str]:
    """Process routed experts from a ret item in non-streaming response."""
    if not getattr(request, "return_routed_experts", False):
        return None
    return ret_item["meta_info"].get("routed_experts", None)
```
**EN:** This block implements the function `process_routed_experts_from_ret(ret_item, request)`. It focuses on Process routed experts from a ret item in non-streaming response., providing reusable behavior for the utils pipeline.
**CN:** 该代码块实现函数 `process_routed_experts_from_ret(ret_item, request)`。它围绕 `process_routed_experts_from_ret` 所承担的 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 109-124: Implement cached tokens details from dict / 实现cached tokens details from dict
```python
def cached_tokens_details_from_dict(
    details: Dict[str, Any],
) -> CachedTokensDetails:
    """Convert a raw cached_tokens_details dict to a CachedTokensDetails object."""
    if "storage" in details:
        return CachedTokensDetails(
            device=details.get("device", 0),
            host=details.get("host", 0),
            storage=details.get("storage", 0),
            storage_backend=details.get("storage_backend"),
        )
    else:
        return CachedTokensDetails(
            device=details.get("device", 0),
            host=details.get("host", 0),
        )
```
**EN:** This block implements the function `cached_tokens_details_from_dict(details)`. It focuses on Convert a raw cached_tokens_details dict to a CachedTokensDetails object., providing reusable behavior for the utils pipeline.
**CN:** 该代码块实现函数 `cached_tokens_details_from_dict(details)`。它围绕 `cached_tokens_details_from_dict` 所承担的 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 127-142: Implement process cached tokens details from ret / 实现process cached tokens details from ret
```python
def process_cached_tokens_details_from_ret(
    ret_item: Dict[str, Any],
    request: Union[
        ChatCompletionRequest,
        CompletionRequest,
    ],
) -> Optional[CachedTokensDetails]:
    """Process cached tokens details from a ret item in non-streaming response."""
    if not request.return_cached_tokens_details:
        return None

    details = ret_item["meta_info"].get("cached_tokens_details", None)
    if details is None:
        return None

    return cached_tokens_details_from_dict(details)
```
**EN:** This block implements the function `process_cached_tokens_details_from_ret(ret_item, request)`. It focuses on Process cached tokens details from a ret item in non-streaming response., providing reusable behavior for the utils pipeline.
**CN:** 该代码块实现函数 `process_cached_tokens_details_from_ret(ret_item, request)`。它围绕 `process_cached_tokens_details_from_ret` 所承担的 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 145-180: Implement convert embeds to tensors / 实现convert embeds to tensors
```python
def convert_embeds_to_tensors(
    embeds: Optional[Union[List[Optional[List[List[float]]]], List[List[float]]]],
) -> Optional[List[Optional[List[torch.Tensor]]]]:
    """Convert nested float lists from the HTTP API to lists of tensors.

    Accepts either:
      - None -> returns None
      - List[List[float]] (single input) -> [[tensor, ...]]
      - List[Optional[List[List[float]]]] (batch) -> [Optional[List[tensor]], ...]
    Each innermost List[float] becomes a 1-D torch.Tensor.
    Per-input None entries are preserved (no overrides for that input).
    """
    if embeds is None:
        return None
    if len(embeds) == 0:
        return []
    # Find first non-None entry to detect nesting depth
    first_non_none = next((e for e in embeds if e is not None), None)
    if first_non_none is None:
        # All entries are None
        return [None] * len(embeds)
    # Detect nesting depth by checking the first non-None entry:
    # - Single input [num_replacements][hidden_size]: first element is List[float]
    # - Batch [num_inputs][num_replacements][hidden_size]: first element is List[List[float]]
    if not first_non_none or not isinstance(first_non_none[0], list):
        # Single input: each entry is a float vector
        return [[torch.tensor(vec, dtype=torch.float32) for vec in embeds]]
    # Otherwise it's batch: [num_inputs][num_replacements][hidden_size]
    return [
        (
            [torch.tensor(vec, dtype=torch.float32) for vec in per_input]
            if per_input is not None
            else None
        )
        for per_input in embeds
    ]
```
**EN:** This block implements the function `convert_embeds_to_tensors(embeds)`. It focuses on Convert nested float lists from the HTTP API to lists of tensors., providing reusable behavior for the utils pipeline.
**CN:** 该代码块实现函数 `convert_embeds_to_tensors(embeds)`。它围绕 `convert_embeds_to_tensors` 所承担的 工具 相关职责展开，为对应处理链路提供可复用能力。

## Key Concepts / 关键概念
- **Main callables / 主要可调用对象**: to_openai_style_logprobs, process_hidden_states_from_ret, should_include_usage, process_routed_experts_from_ret, cached_tokens_details_from_dict, process_cached_tokens_details_from_ret, convert_embeds_to_tensors
- **Domain focus / 领域焦点**: utils / 工具
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: logging, typing
- **Third-party / 第三方库**: torch
- **Local Modules / 本地模块**: sglang.srt.entrypoints.openai.protocol
