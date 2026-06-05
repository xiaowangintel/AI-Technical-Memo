# usage_processor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/openai/usage_processor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements usage processor logic for the OpenAI-compatible serving layer. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 usage 处理器 相关的逻辑，并服务于 OpenAI 兼容服务层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-3: Import runtime dependencies / 导入运行时依赖
```python
from typing import Any, Dict, List, Mapping, Optional, final
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 5-12: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.entrypoints.openai.protocol import PromptTokensDetails, UsageInfo


@final
class UsageProcessor:
    """Stateless helpers that turn raw token counts into a UsageInfo."""

    @staticmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 13-15: Implement details if cached / 实现details if cached
```python
    def _details_if_cached(count: int) -> Optional[PromptTokensDetails]:
        """Return PromptTokensDetails only when count > 0 (keeps JSON slim)."""
        return PromptTokensDetails(cached_tokens=count) if count > 0 else None
```
**EN:** This block implements the method `_details_if_cached(count)` on `UsageProcessor`. It focuses on Return PromptTokensDetails only when count > 0 (keeps JSON slim)., so the class can advance the usage processor workflow in a self-contained way.
**CN:** 该代码块实现 `UsageProcessor` 上的方法 `_details_if_cached(count)`。它围绕 `_details_if_cached` 所承担的 usage 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 16-17: Provide supporting module logic / 提供辅助模块逻辑
```python

    @staticmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 18-49: Implement calculate response usage / 实现calculate 响应 usage
```python
    def calculate_response_usage(
        responses: List[Dict[str, Any]],
        n_choices: int = 1,
        enable_cache_report: bool = False,
    ) -> UsageInfo:
        completion_tokens = sum(
            r["meta_info"].get("completion_tokens", 0) for r in responses
        )
        prompt_tokens = sum(
            responses[i]["meta_info"].get("prompt_tokens", 0)
            for i in range(0, len(responses), n_choices)
        )

        # some API don't have reasoning_tokens semantics
        reasoning_tokens = sum(
            r["meta_info"].get("reasoning_tokens", 0) for r in responses
        )

        cached_details = None
        if enable_cache_report:
            cached_total = sum(
                responses[i]["meta_info"].get("cached_tokens", 0)
                for i in range(0, len(responses), n_choices)
            )
            cached_details = UsageProcessor._details_if_cached(cached_total)

        return UsageProcessor.calculate_token_usage(
            prompt_tokens=prompt_tokens,
            reasoning_tokens=reasoning_tokens,
            completion_tokens=completion_tokens,
            cached_tokens=cached_details,
        )
```
**EN:** This block implements the method `calculate_response_usage(responses, n_choices, enable_cache_report)` on `UsageProcessor`. It focuses on handling the usage processor responsibilities represented by `calculate_response_usage`, so the class can advance the usage processor workflow in a self-contained way.
**CN:** 该代码块实现 `UsageProcessor` 上的方法 `calculate_response_usage(responses, n_choices, enable_cache_report)`。它围绕 `calculate_response_usage` 所承担的 usage 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 50-51: Provide supporting module logic / 提供辅助模块逻辑
```python

    @staticmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 52-80: Implement calculate streaming usage / 实现calculate streaming usage
```python
    def calculate_streaming_usage(
        prompt_tokens: Mapping[int, int],
        reasoning_tokens: Mapping[int, int],
        completion_tokens: Mapping[int, int],
        cached_tokens: Mapping[int, int],
        n_choices: int,
        enable_cache_report: bool = False,
    ) -> UsageInfo:
        # index % n_choices == 0 marks the first choice of a prompt
        total_prompt_tokens = sum(
            tok for idx, tok in prompt_tokens.items() if idx % n_choices == 0
        )
        total_reasoning_tokens = sum(reasoning_tokens.values())
        total_completion_tokens = sum(completion_tokens.values())

        cached_details = (
            UsageProcessor._details_if_cached(
                sum(tok for idx, tok in cached_tokens.items() if idx % n_choices == 0)
            )
            if enable_cache_report
            else None
        )

        return UsageProcessor.calculate_token_usage(
            prompt_tokens=total_prompt_tokens,
            reasoning_tokens=total_reasoning_tokens,
            completion_tokens=total_completion_tokens,
            cached_tokens=cached_details,
        )
```
**EN:** This block implements the method `calculate_streaming_usage(prompt_tokens, reasoning_tokens, completion_tokens, cached_tokens, n_choices, ...)` on `UsageProcessor`. It focuses on handling the usage processor responsibilities represented by `calculate_streaming_usage`, so the class can advance the usage processor workflow in a self-contained way.
**CN:** 该代码块实现 `UsageProcessor` 上的方法 `calculate_streaming_usage(prompt_tokens, reasoning_tokens, completion_tokens, cached_tokens, n_choices, ...)`。它围绕 `calculate_streaming_usage` 所承担的 usage 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 81-82: Provide supporting module logic / 提供辅助模块逻辑
```python

    @staticmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 83-96: Implement calculate token usage / 实现calculate Token usage
```python
    def calculate_token_usage(
        prompt_tokens: int,
        completion_tokens: int,
        reasoning_tokens: Optional[int] = 0,
        cached_tokens: Optional[PromptTokensDetails] = None,
    ) -> UsageInfo:
        """Calculate token usage information"""
        return UsageInfo(
            prompt_tokens=prompt_tokens,
            completion_tokens=completion_tokens,
            total_tokens=prompt_tokens + completion_tokens,
            prompt_tokens_details=cached_tokens,
            reasoning_tokens=reasoning_tokens,
        )
```
**EN:** This block implements the method `calculate_token_usage(prompt_tokens, completion_tokens, reasoning_tokens, cached_tokens)` on `UsageProcessor`. It focuses on Calculate token usage information, so the class can advance the usage processor workflow in a self-contained way.
**CN:** 该代码块实现 `UsageProcessor` 上的方法 `calculate_token_usage(prompt_tokens, completion_tokens, reasoning_tokens, cached_tokens)`。它围绕 `calculate_token_usage` 所承担的 usage 处理器 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: UsageProcessor
- **Domain focus / 领域焦点**: usage processor / usage 处理器
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: typing
- **Third-party / 第三方库**: __future__
- **Local Modules / 本地模块**: sglang.srt.entrypoints.openai.protocol
