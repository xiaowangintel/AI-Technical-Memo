# base_grammar_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/constrained/base_grammar_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the constrained decoding and grammar control part of the SRT runtime and implements logic centered on `base_grammar_backend`. The module docstring frames it as: "The baseclass of a backend for grammar-guided constrained decoding." / 该模块属于 SRT 运行时的约束解码与语法控制部分，主要实现围绕 `base_grammar_backend` 的逻辑。 它对外提供的主要入口包括 `GrammarStats`, `BaseGrammarObject`, `InvalidGrammarObject`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-29: Module imports, constants, and setup
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
"""The baseclass of a backend for grammar-guided constrained decoding."""

import logging
import time
from concurrent.futures import Future, ThreadPoolExecutor
from dataclasses import dataclass, field
from typing import Dict, List, Optional, Tuple

import torch

from sglang.srt.parser.reasoning_parser import ReasoningParser
from sglang.srt.server_args import ServerArgs

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; handles grammar or regular-expression constraints.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；处理语法或正则约束。

### Lines 30-39: Class GrammarStats
```python
@dataclass
class GrammarStats:
    compilation_time: Optional[float] = None
    schema_count: Optional[int] = None
    ebnf_size: Optional[int] = None
    is_cache_hit: bool = False
    is_grammar_aborted: bool = False
    tree_traversal_time: List[float] = field(default_factory=list)
    dispatch_type: Optional[str] = None
    num_timeout: int = 0
```
**EN:** This range introduces `GrammarStats` and defines the structure or metadata that its methods rely on. In this range it handles grammar or regular-expression constraints.
**CN:** 这一段引入 `GrammarStats`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理语法或正则约束。

### Lines 42-43: Class BaseGrammarObject
```python
class BaseGrammarObject:

```
**EN:** This range introduces `BaseGrammarObject` and defines the structure or metadata that its methods rely on. In this range it handles grammar or regular-expression constraints.
**CN:** 这一段引入 `BaseGrammarObject`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理语法或正则约束。

### Lines 44-47: Method BaseGrammarObject.__init__
```python
    def __init__(self):
        self._finished = False
        self.grammar_stats = None
        self.current_token = None
```
**EN:** This callable implements `BaseGrammarObject.__init__` and mainly initializes instance state and defaults. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `BaseGrammarObject.__init__`，主要用于初始化实例状态与默认值。 在这一范围内，它会处理语法或正则约束。

### Lines 49-50: Method BaseGrammarObject.maybe_init_reasoning
```python
    def maybe_init_reasoning(self, reasoning: bool):
        pass
```
**EN:** This callable implements `BaseGrammarObject.maybe_init_reasoning`. It takes `reasoning` and mainly implements maybe init reasoning.
**CN:** 这一可调用对象实现了 `BaseGrammarObject.maybe_init_reasoning`。它接收 `reasoning`，主要用于实现 maybe init reasoning 相关逻辑。

### Lines 52-56: Method BaseGrammarObject.accept_token
```python
    def accept_token(self, token: int) -> None:
        """
        Accept a token in the grammar.
        """
        raise NotImplementedError()
```
**EN:** This callable implements `BaseGrammarObject.accept_token`. It takes `token` and mainly converts data into another representation. The docstring states: "Accept a token in the grammar." In this range it performs defensive checks on invalid state; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `BaseGrammarObject.accept_token`。它接收 `token`，主要用于将数据转换为另一种表示。 在这一范围内，它会对非法状态执行防御性检查；处理语法或正则约束。

### Lines 58-59: Method BaseGrammarObject.rollback
```python
    def rollback(self, k: int):
        raise NotImplementedError()
```
**EN:** This callable implements `BaseGrammarObject.rollback`. It takes `k` and mainly implements rollback. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `BaseGrammarObject.rollback`。它接收 `k`，主要用于实现 rollback 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 61-62: Method BaseGrammarObject.is_terminated
```python
    def is_terminated(self):
        return False
```
**EN:** This callable implements `BaseGrammarObject.is_terminated` and mainly implements is terminated.
**CN:** 这一可调用对象实现了 `BaseGrammarObject.is_terminated`，主要用于实现 is terminated 相关逻辑。

### Lines 64-67: Method BaseGrammarObject.allocate_vocab_mask
```python
    def allocate_vocab_mask(
        self, vocab_size: int, batch_size: int, device
    ) -> torch.Tensor:
        raise NotImplementedError()
```
**EN:** This callable implements `BaseGrammarObject.allocate_vocab_mask`. It takes `vocab_size`, `batch_size`, `device` and mainly implements allocate vocab mask. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `BaseGrammarObject.allocate_vocab_mask`。它接收 `vocab_size`, `batch_size`, `device`，主要用于实现 allocate vocab mask 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 69-70: Method BaseGrammarObject.fill_vocab_mask
```python
    def fill_vocab_mask(self, vocab_mask: torch.Tensor, idx: int) -> None:
        raise NotImplementedError()
```
**EN:** This callable implements `BaseGrammarObject.fill_vocab_mask`. It takes `vocab_mask`, `idx` and mainly implements fill vocab mask. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `BaseGrammarObject.fill_vocab_mask`。它接收 `vocab_mask`, `idx`，主要用于实现 fill vocab mask 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 72-74: Method BaseGrammarObject.move_vocab_mask
```python
    @staticmethod
    def move_vocab_mask(vocab_mask: torch.Tensor, device) -> torch.Tensor:
        raise NotImplementedError()
```
**EN:** This callable implements `BaseGrammarObject.move_vocab_mask`. It takes `vocab_mask`, `device` and mainly implements move vocab mask. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `BaseGrammarObject.move_vocab_mask`。它接收 `vocab_mask`, `device`，主要用于实现 move vocab mask 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 76-78: Method BaseGrammarObject.apply_vocab_mask
```python
    @staticmethod
    def apply_vocab_mask(logits: torch.Tensor, vocab_mask: torch.Tensor) -> None:
        raise NotImplementedError()
```
**EN:** This callable implements `BaseGrammarObject.apply_vocab_mask`. It takes `logits`, `vocab_mask` and mainly implements apply vocab mask. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `BaseGrammarObject.apply_vocab_mask`。它接收 `logits`, `vocab_mask`，主要用于实现 apply vocab mask 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 80-81: Method BaseGrammarObject.copy
```python
    def copy(self) -> "BaseGrammarObject":
        return self
```
**EN:** This callable implements `BaseGrammarObject.copy` and mainly implements copy. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `BaseGrammarObject.copy`，主要用于实现 copy 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 83-85: Method BaseGrammarObject.finished
```python
    @property
    def finished(self):
        return self._finished
```
**EN:** This callable implements `BaseGrammarObject.finished` and mainly implements finished.
**CN:** 这一可调用对象实现了 `BaseGrammarObject.finished`，主要用于实现 finished 相关逻辑。

### Lines 87-89: Method BaseGrammarObject.finished
```python
    @finished.setter
    def finished(self, finished):
        self._finished = finished
```
**EN:** This callable implements `BaseGrammarObject.finished`. It takes `finished` and mainly implements finished.
**CN:** 这一可调用对象实现了 `BaseGrammarObject.finished`。它接收 `finished`，主要用于实现 finished 相关逻辑。

### Lines 91-99: Method BaseGrammarObject.try_jump_forward
```python
    def try_jump_forward(self, tokenizer) -> Optional[Tuple[List[int], str]]:
        """
        Try to jump forward in the grammar.

        Returns:
            A jump forward helper which may be used in `jump_forward_str_state`.
            None if the jump forward is not possible.
        """
        raise NotImplementedError()
```
**EN:** This callable implements `BaseGrammarObject.try_jump_forward`. It takes `tokenizer` and mainly implements try jump forward. The docstring states: "Try to jump forward in the grammar." In this range it performs defensive checks on invalid state; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `BaseGrammarObject.try_jump_forward`。它接收 `tokenizer`，主要用于实现 try jump forward 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；处理语法或正则约束。

### Lines 101-109: Method BaseGrammarObject.jump_forward_str_state
```python
    def jump_forward_str_state(self, helper: Tuple[List[int], str]) -> Tuple[str, int]:
        """
        Jump forward for the grammar.

        Returns:
            A tuple of the jump forward string and the next state of the grammar
            (which can be used in `jump_and_retokenize` if needed).
        """
        raise NotImplementedError()
```
**EN:** This callable implements `BaseGrammarObject.jump_forward_str_state`. It takes `helper` and mainly implements jump forward str state. The docstring states: "Jump forward for the grammar." In this range it performs defensive checks on invalid state; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `BaseGrammarObject.jump_forward_str_state`。它接收 `helper`，主要用于实现 jump forward str state 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；处理语法或正则约束。

### Lines 111-117: Method BaseGrammarObject.jump_and_retokenize
```python
    def jump_and_retokenize(
        self, old_output_ids: List[int], new_output_ids: List[int], next_state: int
    ) -> None:
        """
        Jump forward occurs, and update the grammar state if needed.
        """
        raise NotImplementedError()
```
**EN:** This callable implements `BaseGrammarObject.jump_and_retokenize`. It takes `old_output_ids`, `new_output_ids`, `next_state` and mainly converts data into another representation. The docstring states: "Jump forward occurs, and update the grammar state if needed." In this range it performs defensive checks on invalid state; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `BaseGrammarObject.jump_and_retokenize`。它接收 `old_output_ids`, `new_output_ids`, `next_state`，主要用于将数据转换为另一种表示。 在这一范围内，它会对非法状态执行防御性检查；处理语法或正则约束。

### Lines 120-122: Class InvalidGrammarObject
```python
class InvalidGrammarObject(BaseGrammarObject):
    """Represents a grammar that failed to compile, carrying the original error message."""

```
**EN:** This range introduces `InvalidGrammarObject` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Represents a grammar that failed to compile, carrying the original error message." In this range it handles grammar or regular-expression constraints; prepares compilation-related behavior.
**CN:** 这一段引入 `InvalidGrammarObject`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理语法或正则约束；处理与编译相关的行为。

### Lines 123-125: Method InvalidGrammarObject.__init__
```python
    def __init__(self, error_message: str = "Unknown grammar error"):
        super().__init__()
        self.error_message = error_message
```
**EN:** This callable implements `InvalidGrammarObject.__init__`. It takes `error_message` and mainly initializes instance state and defaults. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `InvalidGrammarObject.__init__`。它接收 `error_message`，主要用于初始化实例状态与默认值。 在这一范围内，它会处理语法或正则约束。

### Lines 127-128: Method InvalidGrammarObject.__repr__
```python
    def __repr__(self):
        return f"InvalidGrammarObject(error_message={self.error_message!r})"
```
**EN:** This callable implements `InvalidGrammarObject.__repr__` and mainly implements repr. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `InvalidGrammarObject.__repr__`，主要用于实现 repr 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 131-133: Class BaseGrammarBackend
```python
class BaseGrammarBackend:
    _enable_strict_thinking: bool = False

```
**EN:** This range introduces `BaseGrammarBackend` and defines the structure or metadata that its methods rely on. In this range it handles grammar or regular-expression constraints.
**CN:** 这一段引入 `BaseGrammarBackend`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理语法或正则约束。

### Lines 134-136: Method BaseGrammarBackend.__init__
```python
    def __init__(self):
        self.executor = ThreadPoolExecutor()
        self.cache: Dict[Tuple[str, str], BaseGrammarObject] = {}
```
**EN:** This callable implements `BaseGrammarBackend.__init__` and mainly initializes instance state and defaults. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `BaseGrammarBackend.__init__`，主要用于初始化实例状态与默认值。 在这一范围内，它会处理语法或正则约束。

### Lines 138-140: Method BaseGrammarBackend._not_supported
```python
    def _not_supported(self, key_type: str, key_string: str) -> BaseGrammarObject:
        logger.warning(f"Skip unsupported {key_type=}, {key_string=}")
        return InvalidGrammarObject()
```
**EN:** This callable implements `BaseGrammarBackend._not_supported`. It takes `key_type`, `key_string` and mainly implements not supported. In this range it emits logs for diagnostics; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `BaseGrammarBackend._not_supported`。它接收 `key_type`, `key_string`，主要用于实现 not supported 相关逻辑。 在这一范围内，它会输出日志以便诊断；处理语法或正则约束。

### Lines 142-144: Method BaseGrammarBackend.enable_strict_thinking
```python
    @property
    def enable_strict_thinking(self):
        return self._enable_strict_thinking
```
**EN:** This callable implements `BaseGrammarBackend.enable_strict_thinking` and mainly implements enable strict thinking.
**CN:** 这一可调用对象实现了 `BaseGrammarBackend.enable_strict_thinking`，主要用于实现 enable strict thinking 相关逻辑。

### Lines 146-148: Method BaseGrammarBackend.is_support_token_filter
```python
    @property
    def is_support_token_filter(self):
        return False
```
**EN:** This callable implements `BaseGrammarBackend.is_support_token_filter` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `BaseGrammarBackend.is_support_token_filter`，主要用于将数据转换为另一种表示。

### Lines 150-154: Method BaseGrammarBackend.set_token_filter
```python
    def set_token_filter(
        self, vocab_mask, token_ids, batch_idx, is_allowed=True, reset_vocab_mask=True
    ):
        """Set or clear specific tokens in the vocab mask. No-op by default."""
        pass
```
**EN:** This callable implements `BaseGrammarBackend.set_token_filter`. It takes `vocab_mask`, `token_ids`, `batch_idx`, `is_allowed` and mainly converts data into another representation. The docstring states: "Set or clear specific tokens in the vocab mask."
**CN:** 这一可调用对象实现了 `BaseGrammarBackend.set_token_filter`。它接收 `vocab_mask`, `token_ids`, `batch_idx`, `is_allowed`，主要用于将数据转换为另一种表示。

### Lines 156-158: Method BaseGrammarBackend.init_strict_reasoning_grammar
```python
    def init_strict_reasoning_grammar(self, reasoning: bool):
        """Create a grammar object for strict token filtering only. Returns None by default."""
        return None
```
**EN:** This callable implements `BaseGrammarBackend.init_strict_reasoning_grammar`. It takes `reasoning` and mainly implements init strict reasoning grammar. The docstring states: "Create a grammar object for strict token filtering only." In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `BaseGrammarBackend.init_strict_reasoning_grammar`。它接收 `reasoning`，主要用于实现 init strict reasoning grammar 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 160-164: Method BaseGrammarBackend.dispatch_fallback
```python
    def dispatch_fallback(self, key_type: str, key_string: str) -> BaseGrammarObject:
        """
        This function should not be reached in any case.
        """
        raise ValueError(f"Invalid key_type: {key_type}={key_string}")
```
**EN:** This callable implements `BaseGrammarBackend.dispatch_fallback`. It takes `key_type`, `key_string` and mainly implements dispatch fallback. The docstring states: "This function should not be reached in any case." In this range it performs defensive checks on invalid state; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `BaseGrammarBackend.dispatch_fallback`。它接收 `key_type`, `key_string`，主要用于实现 dispatch fallback 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；处理语法或正则约束。

### Lines 166-167: Method BaseGrammarBackend.dispatch_json
```python
    def dispatch_json(self, key_string: str) -> BaseGrammarObject:
        return self._not_supported("json", key_string)
```
**EN:** This callable implements `BaseGrammarBackend.dispatch_json`. It takes `key_string` and mainly implements dispatch json. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `BaseGrammarBackend.dispatch_json`。它接收 `key_string`，主要用于实现 dispatch json 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 169-170: Method BaseGrammarBackend.dispatch_regex
```python
    def dispatch_regex(self, key_string: str) -> BaseGrammarObject:
        return self._not_supported("regex", key_string)
```
**EN:** This callable implements `BaseGrammarBackend.dispatch_regex`. It takes `key_string` and mainly implements dispatch regex. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `BaseGrammarBackend.dispatch_regex`。它接收 `key_string`，主要用于实现 dispatch regex 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 172-173: Method BaseGrammarBackend.dispatch_ebnf
```python
    def dispatch_ebnf(self, key_string: str) -> BaseGrammarObject:
        return self._not_supported("ebnf", key_string)
```
**EN:** This callable implements `BaseGrammarBackend.dispatch_ebnf`. It takes `key_string` and mainly implements dispatch ebnf. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `BaseGrammarBackend.dispatch_ebnf`。它接收 `key_string`，主要用于实现 dispatch ebnf 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 175-176: Method BaseGrammarBackend.dispatch_structural_tag
```python
    def dispatch_structural_tag(self, key_string: str) -> BaseGrammarObject:
        return self._not_supported("structural_tag", key_string)
```
**EN:** This callable implements `BaseGrammarBackend.dispatch_structural_tag`. It takes `key_string` and mainly implements dispatch structural tag. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `BaseGrammarBackend.dispatch_structural_tag`。它接收 `key_string`，主要用于实现 dispatch structural tag 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 178-196: Method BaseGrammarBackend._init_value_dispatch
```python
    def _init_value_dispatch(
        self, key: Tuple[str, str], require_reasoning: bool
    ) -> BaseGrammarObject:
        s = time.perf_counter()
        key_type, key_string = key
        if key_type == "json":
            grammar = self.dispatch_json(key_string)
        elif key_type == "regex":
            grammar = self.dispatch_regex(key_string)
        elif key_type == "ebnf":
            grammar = self.dispatch_ebnf(key_string)
        elif key_type == "structural_tag":
            grammar = self.dispatch_structural_tag(key_string)
        else:
            grammar = self.dispatch_fallback(key_type, key_string)

        if grammar is not None and grammar.grammar_stats is not None:
            grammar.grammar_stats.compilation_time = time.perf_counter() - s
        return grammar
```
**EN:** This callable implements `BaseGrammarBackend._init_value_dispatch`. It takes `key`, `require_reasoning` and mainly implements init value dispatch. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `BaseGrammarBackend._init_value_dispatch`。它接收 `key`, `require_reasoning`，主要用于实现 init value dispatch 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 198-207: Method BaseGrammarBackend.get_cached_or_future_value
```python
    def get_cached_or_future_value(
        self, key: Tuple[str, str], require_reasoning: bool
    ) -> Tuple[BaseGrammarObject | Future[BaseGrammarObject], bool]:
        value = self.cache.get(key)
        if value:
            copied_value = value.copy()
            copied_value.maybe_init_reasoning(require_reasoning)
            return copied_value, True
        value = self.executor.submit(self._init_value_dispatch, key, require_reasoning)
        return value, False
```
**EN:** This callable implements `BaseGrammarBackend.get_cached_or_future_value`. It takes `key`, `require_reasoning` and mainly retrieves a value or derived view. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `BaseGrammarBackend.get_cached_or_future_value`。它接收 `key`, `require_reasoning`，主要用于获取某个值或派生视图。 在这一范围内，它会处理语法或正则约束。

### Lines 209-210: Method BaseGrammarBackend.set_cache
```python
    def set_cache(self, key: Tuple[str, str], value: BaseGrammarObject):
        self.cache[key] = value
```
**EN:** This callable implements `BaseGrammarBackend.set_cache`. It takes `key`, `value` and mainly applies configuration to mutable state. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `BaseGrammarBackend.set_cache`。它接收 `key`, `value`，主要用于将配置写入可变状态。 在这一范围内，它会处理语法或正则约束。

### Lines 212-213: Method BaseGrammarBackend.reset
```python
    def reset(self):
        self.cache.clear()
```
**EN:** This callable implements `BaseGrammarBackend.reset` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `BaseGrammarBackend.reset`，主要用于将配置写入可变状态。

### Lines 214-218: Module-level constants and helpers
```python


GRAMMAR_BACKEND_REGISTRY = {}


```
**EN:** This range organizes module-level state and shared setup. In this range it handles grammar or regular-expression constraints.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 219-220: Function register_grammar_backend
```python
def register_grammar_backend(name, init_func):
    GRAMMAR_BACKEND_REGISTRY[name] = init_func
```
**EN:** This callable implements `register_grammar_backend`. It takes `name`, `init_func` and mainly registers hooks, handlers, or operators. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `register_grammar_backend`。它接收 `name`, `init_func`，主要用于注册钩子、处理器或算子。 在这一范围内，它会处理语法或正则约束。

### Lines 223-282: Function create_grammar_backend (part 1/2)
```python
def create_grammar_backend(
    server_args: ServerArgs,
    tokenizer,
    vocab_size: int,
    eos_token_ids: Optional[set] = None,
    think_end_id: Optional[int] = None,
) -> Optional[BaseGrammarBackend]:
    name = server_args.grammar_backend

    # Custom grammar backend has the highest priority
    if name in GRAMMAR_BACKEND_REGISTRY:
        return GRAMMAR_BACKEND_REGISTRY[name](
            server_args, tokenizer, vocab_size, eos_token_ids
        )

    # Default grammar backends
    if name == "outlines":
        from sglang.srt.constrained.outlines_backend import OutlinesGrammarBackend

        grammar_backend = OutlinesGrammarBackend(
            tokenizer,
            whitespace_pattern=server_args.constrained_json_whitespace_pattern,
        )
    elif name == "xgrammar":
        from sglang.srt.constrained.xgrammar_backend import (
            TokenizerNotSupportedError,
            XGrammarGrammarBackend,
        )

        # Convert Set[int] to List[int] if needed
        eos_list = list(eos_token_ids) if eos_token_ids else None

        try:
            grammar_backend = XGrammarGrammarBackend(
                tokenizer,
                vocab_size=vocab_size,
                model_eos_token_ids=eos_list,
                any_whitespace=not server_args.constrained_json_disable_any_whitespace,
            )
        except TokenizerNotSupportedError as e:
            if server_args.enable_strict_thinking:
                raise ValueError(
                    f"--enable-strict-thinking requires a grammar backend with "
                    f"token filtering support, but XGrammar failed to initialize: "
                    f"{e}. Cannot fall back to grammar_backend='none' with strict "
                    f"thinking enabled."
                ) from e
            logger.warning(
                f"Grammar backend disabled because tokenizer is not supported by XGrammar: {e}. "
                "Falling back to grammar_backend='none'. "
                "Structured outputs (JSON schema, regex, EBNF) will not be available."
            )
            server_args.grammar_backend = "none"
            return None
    elif name == "llguidance":
        from sglang.srt.constrained.llguidance_backend import GuidanceBackend

        grammar_backend = GuidanceBackend(
            tokenizer=tokenizer,
            any_whitespace=not server_args.constrained_json_disable_any_whitespace,
```
**EN:** This callable implements `create_grammar_backend`. It takes `server_args`, `tokenizer`, `vocab_size`, `eos_token_ids` and mainly constructs new objects or contexts. This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `create_grammar_backend`。它接收 `server_args`, `tokenizer`, `vocab_size`, `eos_token_ids`，主要用于构造新的对象或上下文。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；输出日志以便诊断。

### Lines 283-313: Function create_grammar_backend (part 2/2)
```python
            whitespace_pattern=server_args.constrained_json_whitespace_pattern,
        )
    elif name == "none":
        if server_args.enable_strict_thinking:
            raise ValueError(
                "--enable-strict-thinking requires a grammar backend that supports "
                "token filtering, but grammar_backend='none' was specified. Use "
                "--grammar-backend xgrammar or another backend that supports token "
                "filtering."
            )
        return None
    else:
        raise ValueError(f"Invalid grammar backend: {name}")

    if server_args.reasoning_parser and think_end_id is not None:
        from sglang.srt.constrained.reasoner_grammar_backend import (
            ReasonerGrammarBackend,
        )

        reasoning_parser = ReasoningParser(
            model_type=server_args.reasoning_parser, stream_reasoning=False
        )

        grammar_backend = ReasonerGrammarBackend(
            grammar_backend,
            reasoning_parser,
            tokenizer,
            enable_strict_thinking=server_args.enable_strict_thinking,
        )

    return grammar_backend
```
**EN:** This callable implements `create_grammar_backend`. It takes `server_args`, `tokenizer`, `vocab_size`, `eos_token_ids` and mainly constructs new objects or contexts. This chunk is part 2 of 2 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `create_grammar_backend`。它接收 `server_args`, `tokenizer`, `vocab_size`, `eos_token_ids`，主要用于构造新的对象或上下文。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；处理语法或正则约束。

## Key Concepts / 关键概念
- `GrammarStats`: core class or state container / 核心类或状态容器
- `BaseGrammarObject`: core class or state container / 核心类或状态容器
- `InvalidGrammarObject`: core class or state container / 核心类或状态容器
- `BaseGrammarBackend`: core class or state container / 核心类或状态容器
- `GRAMMAR_BACKEND_REGISTRY`: module constant or capability flag / 模块常量或能力标记
- `register_grammar_backend`: registers hooks, handlers, or operators / 注册钩子、处理器或算子
- `create_grammar_backend`: constructs new objects or contexts / 构造新的对象或上下文

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `time`, `concurrent.futures`, `dataclasses`, `typing`
- **Third-party / 第三方**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.parser.reasoning_parser`, `sglang.srt.server_args`, `sglang.srt.constrained.outlines_backend`, `sglang.srt.constrained.reasoner_grammar_backend`, `sglang.srt.constrained.xgrammar_backend`, `sglang.srt.constrained.llguidance_backend`
