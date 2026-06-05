# custom_logit_processor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/sampling/custom_logit_processor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the sampling and penalty control part of the SRT runtime and implements logic centered on `custom_logit_processor`. It exposes primary entry points such as `_cache_from_str`, `CustomLogitProcessor`, `DisallowedTokensLogitsProcessor`. / 该模块属于 SRT 运行时的采样与惩罚控制部分，主要实现围绕 `custom_logit_processor` 的逻辑。 它对外提供的主要入口包括 `_cache_from_str`, `CustomLogitProcessor`, `DisallowedTokensLogitsProcessor`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Module imports, constants, and setup
```python
import json
from abc import ABC, abstractmethod
from functools import lru_cache
from typing import TYPE_CHECKING, Any, Dict, List, Optional, Set

import dill
import orjson
import torch

if TYPE_CHECKING:
    from sglang.srt.managers.schedule_batch import Req


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 14-20: Function _cache_from_str
```python
@lru_cache(maxsize=None)
def _cache_from_str(json_str: str):
    """Deserialize a json string to a Callable object.
    This function is cached to avoid redundant deserialization.
    """
    data = orjson.loads(json_str)
    return dill.loads(bytes.fromhex(data["callable"]))
```
**EN:** This callable implements `_cache_from_str`. It takes `json_str` and mainly constructs data from an external representation. The docstring states: "Deserialize a json string to a Callable object." In this range it serializes or parses JSON payloads.
**CN:** 这一可调用对象实现了 `_cache_from_str`。它接收 `json_str`，主要用于从外部表示构造数据。 在这一范围内，它会序列化或解析 JSON 数据。

### Lines 23-25: Class CustomLogitProcessor
```python
class CustomLogitProcessor(ABC):
    """Abstract base class for callable functions."""

```
**EN:** This range introduces `CustomLogitProcessor` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Abstract base class for callable functions."
**CN:** 这一段引入 `CustomLogitProcessor`，并定义其后续方法依赖的结构或元数据。

### Lines 26-33: Method CustomLogitProcessor.__call__
```python
    @abstractmethod
    def __call__(
        self,
        logits: torch.Tensor,
        custom_param_list: Optional[List[Dict[str, Any]]] = None,
    ) -> torch.Tensor:
        """Define the callable behavior."""
        raise NotImplementedError
```
**EN:** This callable implements `CustomLogitProcessor.__call__`. It takes `logits`, `custom_param_list` and mainly handles invocation-time behavior. The docstring states: "Define the callable behavior." In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `CustomLogitProcessor.__call__`。它接收 `logits`, `custom_param_list`，主要用于处理调用时的行为。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 35-38: Method CustomLogitProcessor.to_str
```python
    @classmethod
    def to_str(cls) -> str:
        """Serialize the callable function to a JSON-compatible string."""
        return json.dumps({"callable": dill.dumps(cls).hex()})
```
**EN:** This callable implements `CustomLogitProcessor.to_str`. It takes `cls` and mainly converts data into another representation. The docstring states: "Serialize the callable function to a JSON-compatible string." In this range it serializes or parses JSON payloads.
**CN:** 这一可调用对象实现了 `CustomLogitProcessor.to_str`。它接收 `cls`，主要用于将数据转换为另一种表示。 在这一范围内，它会序列化或解析 JSON 数据。

### Lines 40-43: Method CustomLogitProcessor.from_str
```python
    @classmethod
    def from_str(cls, json_str: str):
        """Deserialize a callable function from a JSON string."""
        return _cache_from_str(json_str)()
```
**EN:** This callable implements `CustomLogitProcessor.from_str`. It takes `cls`, `json_str` and mainly constructs data from an external representation. The docstring states: "Deserialize a callable function from a JSON string." In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `CustomLogitProcessor.from_str`。它接收 `cls`, `json_str`，主要用于从外部表示构造数据。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 46-46: Class DisallowedTokensLogitsProcessor
```python
class DisallowedTokensLogitsProcessor(CustomLogitProcessor):
```
**EN:** This range introduces `DisallowedTokensLogitsProcessor` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `DisallowedTokensLogitsProcessor`，并定义其后续方法依赖的结构或元数据。

### Lines 47-57: Method DisallowedTokensLogitsProcessor.__call__
```python
    def __call__(
        self,
        logits: torch.Tensor,
        custom_param_list: Optional[List[Dict[str, Any]]] = None,
    ) -> torch.Tensor:
        disallowed_token_ids = custom_param_list[0]["token_ids"]
        assert all(
            disallowed_token_ids == c["token_ids"] for c in custom_param_list
        ), f"{custom_param_list=}"
        logits[..., disallowed_token_ids] = -float("inf")
        return logits
```
**EN:** This callable implements `DisallowedTokensLogitsProcessor.__call__`. It takes `logits`, `custom_param_list` and mainly handles invocation-time behavior. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `DisallowedTokensLogitsProcessor.__call__`。它接收 `logits`, `custom_param_list`，主要用于处理调用时的行为。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 60-66: Class ThinkingBudgetLogitProcessor
```python
class ThinkingBudgetLogitProcessor(CustomLogitProcessor):
    """A logit processor that controls the length of thinking."""

    THINKING_START_TOKEN_ID: int
    THINKING_END_TOKEN_ID: int
    NEW_LINE_TOKEN_ID: int

```
**EN:** This range introduces `ThinkingBudgetLogitProcessor` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "A logit processor that controls the length of thinking."
**CN:** 这一段引入 `ThinkingBudgetLogitProcessor`，并定义其后续方法依赖的结构或元数据。

### Lines 67-112: Method ThinkingBudgetLogitProcessor.__call__
```python
    def __call__(self, logits, custom_param_list: list[dict[str, Any]]):
        if custom_param_list is None or not custom_param_list:
            return logits
        for i, param_dict in enumerate(custom_param_list):
            if param_dict is None:
                continue

            thinking_budget: int | None = param_dict.get("thinking_budget")

            # Skip if thinking_budget is unset, or not an integer, or negative
            if (
                thinking_budget is None
                or not isinstance(thinking_budget, int)
                or thinking_budget < 0
            ):
                continue
            req: Req = param_dict.get("__req__")
            cur_ids: list[int] = [*req.origin_input_ids, *req.output_ids]

            # Check if out of thinking stage
            if (
                self.THINKING_START_TOKEN_ID not in cur_ids
                or self.THINKING_END_TOKEN_ID in cur_ids
            ):
                continue

            # Find the index of the thinking start token
            start_index = cur_ids.index(self.THINKING_START_TOKEN_ID)

            # Count the number of tokens after the thinking start token
            num_tokens_after_start = len(cur_ids) - start_index - 1

            if num_tokens_after_start < thinking_budget:
                continue

            # Ensure new line token before thinking end token
            if not req.output_ids or req.output_ids[-1] != self.NEW_LINE_TOKEN_ID:
                logits[i, :] = -float("inf")
                logits[i, self.NEW_LINE_TOKEN_ID] = 0.0
                continue

            # Assign highest probability to the thinking end token
            logits[i, :] = -float("inf")
            logits[i, self.THINKING_END_TOKEN_ID] = 0.0

        return logits
```
**EN:** This callable implements `ThinkingBudgetLogitProcessor.__call__`. It takes `logits`, `custom_param_list` and mainly handles invocation-time behavior.
**CN:** 这一可调用对象实现了 `ThinkingBudgetLogitProcessor.__call__`。它接收 `logits`, `custom_param_list`，主要用于处理调用时的行为。

### Lines 115-120: Class Glm4MoeThinkingBudgetLogitProcessor
```python
class Glm4MoeThinkingBudgetLogitProcessor(ThinkingBudgetLogitProcessor):
    """A logit processor that controls the length of thinking for GLM-4.5 / GLM-4.6 / GLM-4.5V / GLM-4.6V models."""

    THINKING_START_TOKEN_ID: int = 151350
    THINKING_END_TOKEN_ID: int = 151351
    NEW_LINE_TOKEN_ID: int = 198
```
**EN:** This range introduces `Glm4MoeThinkingBudgetLogitProcessor` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "A logit processor that controls the length of thinking for GLM-4.5 / GLM-4.6 / GLM-4.5V / GLM-4.6V models."
**CN:** 这一段引入 `Glm4MoeThinkingBudgetLogitProcessor`，并定义其后续方法依赖的结构或元数据。

### Lines 123-128: Class Qwen3ThinkingBudgetLogitProcessor
```python
class Qwen3ThinkingBudgetLogitProcessor(ThinkingBudgetLogitProcessor):
    """A logit processor that controls the length of thinking for Qwen3 models."""

    THINKING_START_TOKEN_ID: int = 151667
    THINKING_END_TOKEN_ID: int = 151668
    NEW_LINE_TOKEN_ID: int = 198
```
**EN:** This range introduces `Qwen3ThinkingBudgetLogitProcessor` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "A logit processor that controls the length of thinking for Qwen3 models."
**CN:** 这一段引入 `Qwen3ThinkingBudgetLogitProcessor`，并定义其后续方法依赖的结构或元数据。

### Lines 131-136: Class DeepSeekR1ThinkingBudgetLogitProcessor
```python
class DeepSeekR1ThinkingBudgetLogitProcessor(ThinkingBudgetLogitProcessor):
    """A logit processor that controls the length of thinking for DeepSeek-R1 models."""

    THINKING_START_TOKEN_ID: int = 128798
    THINKING_END_TOKEN_ID: int = 128799
    NEW_LINE_TOKEN_ID: int = 201
```
**EN:** This range introduces `DeepSeekR1ThinkingBudgetLogitProcessor` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "A logit processor that controls the length of thinking for DeepSeek-R1 models."
**CN:** 这一段引入 `DeepSeekR1ThinkingBudgetLogitProcessor`，并定义其后续方法依赖的结构或元数据。

### Lines 137-139: Module imports, constants, and setup
```python


# Adapted from DeepSeek's implementation: https://github.com/deepseek-ai/DeepSeek-OCR/blob/main/DeepSeek-OCR-master/DeepSeek-OCR-vllm/process/ngram_norepeat.py
```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 140-142: Class DeepseekOCRNoRepeatNGramLogitProcessor
```python
class DeepseekOCRNoRepeatNGramLogitProcessor(CustomLogitProcessor):
    """Block n-gram repetitions within a sliding window for DeepSeek-OCR outputs."""

```
**EN:** This range introduces `DeepseekOCRNoRepeatNGramLogitProcessor` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Block n-gram repetitions within a sliding window for DeepSeek-OCR outputs."
**CN:** 这一段引入 `DeepseekOCRNoRepeatNGramLogitProcessor`，并定义其后续方法依赖的结构或元数据。

### Lines 143-202: Method DeepseekOCRNoRepeatNGramLogitProcessor.__call__
```python
    def __call__(
        self,
        logits: torch.Tensor,
        custom_param_list: Optional[List[Dict[str, Any]]] = None,
    ) -> torch.Tensor:
        if not custom_param_list:
            return logits

        for batch_idx, params in enumerate(custom_param_list):
            if not params:
                continue

            req = params.get("__req__")
            if req is None:
                continue

            try:
                ngram_size = int(params.get("ngram_size") or 0)
                window_size = int(params.get("window_size") or 0)
            except (TypeError, ValueError):
                continue

            if ngram_size <= 0 or window_size <= 0:
                continue

            sequence: List[int] = req.origin_input_ids + req.output_ids
            if len(sequence) < ngram_size:
                continue

            search_start = max(0, len(sequence) - window_size)
            search_end = len(sequence) - ngram_size + 1
            if search_end <= search_start:
                continue

            if ngram_size > 1:
                current_prefix = tuple(sequence[-(ngram_size - 1) :])
            else:
                current_prefix = tuple()

            banned_tokens: Set[int] = set()
            for idx in range(search_start, search_end):
                ngram = sequence[idx : idx + ngram_size]
                if ngram_size == 1 or tuple(ngram[:-1]) == current_prefix:
                    banned_tokens.add(ngram[-1])

            whitelist_ids = params.get("whitelist_token_ids") or []
            try:
                whitelist = {int(token_id) for token_id in whitelist_ids}
            except (TypeError, ValueError):
                whitelist = set()

            banned_tokens.difference_update(whitelist)

            if not banned_tokens:
                continue

            indices = list(banned_tokens)
            logits[batch_idx, indices] = -float("inf")

        return logits
```
**EN:** This callable implements `DeepseekOCRNoRepeatNGramLogitProcessor.__call__`. It takes `logits`, `custom_param_list` and mainly handles invocation-time behavior.
**CN:** 这一可调用对象实现了 `DeepseekOCRNoRepeatNGramLogitProcessor.__call__`。它接收 `logits`, `custom_param_list`，主要用于处理调用时的行为。

## Key Concepts / 关键概念
- `_cache_from_str`: constructs data from an external representation / 从外部表示构造数据
- `CustomLogitProcessor`: core class or state container / 核心类或状态容器
- `DisallowedTokensLogitsProcessor`: core class or state container / 核心类或状态容器
- `ThinkingBudgetLogitProcessor`: core class or state container / 核心类或状态容器
- `Glm4MoeThinkingBudgetLogitProcessor`: core class or state container / 核心类或状态容器
- `Qwen3ThinkingBudgetLogitProcessor`: core class or state container / 核心类或状态容器
- `DeepSeekR1ThinkingBudgetLogitProcessor`: core class or state container / 核心类或状态容器
- `DeepseekOCRNoRepeatNGramLogitProcessor`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `abc`, `functools`, `typing`
- **Third-party / 第三方**: `dill`, `orjson`, `torch`
- **Internal modules / 内部模块**: `sglang.srt.managers.schedule_batch`
