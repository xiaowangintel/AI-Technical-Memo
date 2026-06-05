# reasoner_grammar_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/constrained/reasoner_grammar_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the constrained decoding and grammar control part of the SRT runtime and implements logic centered on `reasoner_grammar_backend`. The module docstring frames it as: "The baseclass of a backend for reasoner grammar-guided constrained decoding." / 该模块属于 SRT 运行时的约束解码与语法控制部分，主要实现围绕 `reasoner_grammar_backend` 的逻辑。 它对外提供的主要入口包括 `ReasonerGrammarObject`, `ReasonerGrammarBackend`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: Module imports, constants, and setup
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
"""The baseclass of a backend for reasoner grammar-guided constrained decoding."""

import logging
from typing import List, Optional, Tuple, Union

import torch
from transformers import PreTrainedTokenizer, PreTrainedTokenizerFast

from sglang.srt.environ import envs
from sglang.srt.parser.reasoning_parser import ReasoningParser

from .base_grammar_backend import (
    BaseGrammarBackend,
    BaseGrammarObject,
    InvalidGrammarObject,
)

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; reads environment-driven configuration.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；读取环境变量驱动的配置。

### Lines 34-50: Class ReasonerGrammarObject
```python
class ReasonerGrammarObject(BaseGrammarObject):
    """Wraps a grammar object to handle reasoning (think/generation) phases.

    State machine (must call maybe_init_reasoning before use):
      THINKING (tokens_in_think >= 0, tokens_after_end == -1)
        -> grammar not consulted, optional token filtering
      GENERATION (tokens_after_end >= 0)
        -> grammar consulted for accept/fill/rollback

    When enable_token_filter=True (strict mode), fill_vocab_mask filters
    excluded tokens during THINKING and enforces max_think_tokens budget.
    When the budget is exhausted, only think_end_id is allowed, forcing the
    model to exit the thinking phase.
    When enable_token_filter=False (non-strict mode), fill_vocab_mask is
    a no-op during THINKING.
    """

```
**EN:** This range introduces `ReasonerGrammarObject` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Wraps a grammar object to handle reasoning (think/generation) phases." In this range it handles grammar or regular-expression constraints.
**CN:** 这一段引入 `ReasonerGrammarObject`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理语法或正则约束。

### Lines 51-76: Method ReasonerGrammarObject.__init__
```python
    def __init__(
        self,
        grammar: Optional[BaseGrammarObject],
        think_end_id: int,
        think_excluded_token_ids: Optional[List[int]] = None,
        max_think_tokens: int = -1,
        enable_token_filter: bool = False,
        token_filter_fn=None,
        allocate_vocab_mask_fn=None,
        move_vocab_mask_fn=None,
        apply_vocab_mask_fn=None,
    ):
        super().__init__()
        self.grammar = grammar
        self.think_end_id = think_end_id
        self.think_excluded_token_ids = think_excluded_token_ids
        self.max_think_tokens = max_think_tokens
        self.enable_token_filter = enable_token_filter
        self.token_filter_fn = token_filter_fn
        self.allocate_vocab_mask_fn = allocate_vocab_mask_fn
        self.move_vocab_mask_fn = move_vocab_mask_fn
        self.apply_vocab_mask_fn = apply_vocab_mask_fn
        self._think_end_id_list = [think_end_id]

        self.tokens_in_think = -1
        self.tokens_after_end = -1
```
**EN:** This callable implements `ReasonerGrammarObject.__init__`. It takes `grammar`, `think_end_id`, `think_excluded_token_ids`, `max_think_tokens` and mainly initializes instance state and defaults. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject.__init__`。它接收 `grammar`, `think_end_id`, `think_excluded_token_ids`, `max_think_tokens`，主要用于初始化实例状态与默认值。 在这一范围内，它会处理语法或正则约束。

### Lines 78-83: Method ReasonerGrammarObject.maybe_init_reasoning
```python
    def maybe_init_reasoning(self, reasoning: bool):
        if reasoning:
            self.tokens_in_think = 0
        else:
            self.tokens_in_think = -1
            self.tokens_after_end = 0
```
**EN:** This callable implements `ReasonerGrammarObject.maybe_init_reasoning`. It takes `reasoning` and mainly implements maybe init reasoning.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject.maybe_init_reasoning`。它接收 `reasoning`，主要用于实现 maybe init reasoning 相关逻辑。

### Lines 85-86: Method ReasonerGrammarObject._is_thinking
```python
    def _is_thinking(self):
        return self.tokens_in_think >= 0 and self.tokens_after_end == -1
```
**EN:** This callable implements `ReasonerGrammarObject._is_thinking` and mainly implements is thinking.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject._is_thinking`，主要用于实现 is thinking 相关逻辑。

### Lines 88-89: Method ReasonerGrammarObject._is_generation
```python
    def _is_generation(self):
        return self.tokens_after_end >= 0
```
**EN:** This callable implements `ReasonerGrammarObject._is_generation` and mainly implements is generation.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject._is_generation`，主要用于实现 is generation 相关逻辑。

### Lines 91-98: Method ReasonerGrammarObject.transfer_state
```python
    def transfer_state(self, token: int) -> None:
        if self._is_thinking():
            if token == self.think_end_id:
                self.tokens_after_end = 0
            else:
                self.tokens_in_think += 1
        elif self._is_generation():
            self.tokens_after_end += 1
```
**EN:** This callable implements `ReasonerGrammarObject.transfer_state`. It takes `token` and mainly implements transfer state.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject.transfer_state`。它接收 `token`，主要用于实现 transfer state 相关逻辑。

### Lines 100-108: Method ReasonerGrammarObject.rollback_state
```python
    def rollback_state(self):
        if self._is_thinking():
            if self.tokens_in_think > 0:
                self.tokens_in_think -= 1
        elif self._is_generation():
            if self.tokens_after_end == 0:
                self.tokens_after_end = -1
            elif self.tokens_after_end > 0:
                self.tokens_after_end -= 1
```
**EN:** This callable implements `ReasonerGrammarObject.rollback_state` and mainly implements rollback state.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject.rollback_state`，主要用于实现 rollback state 相关逻辑。

### Lines 110-113: Method ReasonerGrammarObject.accept_token
```python
    def accept_token(self, token: int):
        if self._is_generation() and self.grammar is not None:
            self.grammar.accept_token(token)
        self.transfer_state(token)
```
**EN:** This callable implements `ReasonerGrammarObject.accept_token`. It takes `token` and mainly converts data into another representation. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject.accept_token`。它接收 `token`，主要用于将数据转换为另一种表示。 在这一范围内，它会处理语法或正则约束。

### Lines 115-118: Method ReasonerGrammarObject.is_terminated
```python
    def is_terminated(self):
        if self.grammar is not None:
            return self.grammar.is_terminated()
        return False
```
**EN:** This callable implements `ReasonerGrammarObject.is_terminated` and mainly implements is terminated. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject.is_terminated`，主要用于实现 is terminated 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 120-126: Method ReasonerGrammarObject.rollback
```python
    def rollback(self, k):
        if self.grammar is not None:
            steps_after = min(k, max(0, self.tokens_after_end))
            if steps_after > 0:
                self.grammar.rollback(steps_after)
        for _ in range(k):
            self.rollback_state()
```
**EN:** This callable implements `ReasonerGrammarObject.rollback`. It takes `k` and mainly implements rollback. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject.rollback`。它接收 `k`，主要用于实现 rollback 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 128-129: Method ReasonerGrammarObject._can_think_more
```python
    def _can_think_more(self):
        return self.max_think_tokens < 0 or self.tokens_in_think < self.max_think_tokens
```
**EN:** This callable implements `ReasonerGrammarObject._can_think_more` and mainly implements can think more.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject._can_think_more`，主要用于实现 can think more 相关逻辑。

### Lines 131-133: Method ReasonerGrammarObject._do_token_filter
```python
    def _do_token_filter(self, vocab_mask, token_ids, idx, is_allowed=True):
        if self.token_filter_fn is not None:
            self.token_filter_fn(vocab_mask, token_ids, idx, is_allowed)
```
**EN:** This callable implements `ReasonerGrammarObject._do_token_filter`. It takes `vocab_mask`, `token_ids`, `idx`, `is_allowed` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject._do_token_filter`。它接收 `vocab_mask`, `token_ids`, `idx`, `is_allowed`，主要用于将数据转换为另一种表示。

### Lines 135-149: Method ReasonerGrammarObject.fill_vocab_mask
```python
    def fill_vocab_mask(self, vocab_mask: torch.Tensor, idx: int) -> None:
        if self._is_thinking():
            if not self.enable_token_filter:
                return
            if self._can_think_more():
                self._do_token_filter(
                    vocab_mask, self.think_excluded_token_ids, idx, is_allowed=False
                )
            else:
                self._do_token_filter(
                    vocab_mask, self._think_end_id_list, idx, is_allowed=True
                )
            return
        if self._is_generation() and self.grammar is not None:
            self.grammar.fill_vocab_mask(vocab_mask, idx)
```
**EN:** This callable implements `ReasonerGrammarObject.fill_vocab_mask`. It takes `vocab_mask`, `idx` and mainly implements fill vocab mask. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject.fill_vocab_mask`。它接收 `vocab_mask`, `idx`，主要用于实现 fill vocab mask 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 151-156: Method ReasonerGrammarObject.allocate_vocab_mask
```python
    def allocate_vocab_mask(self, vocab_size, batch_size, device):
        if self.grammar is not None:
            return self.grammar.allocate_vocab_mask(vocab_size, batch_size, device)
        if self.allocate_vocab_mask_fn is not None:
            return self.allocate_vocab_mask_fn(vocab_size, batch_size, device)
        return None
```
**EN:** This callable implements `ReasonerGrammarObject.allocate_vocab_mask`. It takes `vocab_size`, `batch_size`, `device` and mainly implements allocate vocab mask. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject.allocate_vocab_mask`。它接收 `vocab_size`, `batch_size`, `device`，主要用于实现 allocate vocab mask 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 158-163: Method ReasonerGrammarObject.move_vocab_mask
```python
    def move_vocab_mask(self, vocab_mask, device):
        if self.grammar is not None:
            return self.grammar.move_vocab_mask(vocab_mask, device)
        if self.move_vocab_mask_fn is not None:
            return self.move_vocab_mask_fn(vocab_mask, device)
        return vocab_mask
```
**EN:** This callable implements `ReasonerGrammarObject.move_vocab_mask`. It takes `vocab_mask`, `device` and mainly implements move vocab mask. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject.move_vocab_mask`。它接收 `vocab_mask`, `device`，主要用于实现 move vocab mask 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 165-169: Method ReasonerGrammarObject.apply_vocab_mask
```python
    @property
    def apply_vocab_mask(self):
        if self.grammar is not None:
            return self.grammar.apply_vocab_mask
        return self.apply_vocab_mask_fn
```
**EN:** This callable implements `ReasonerGrammarObject.apply_vocab_mask` and mainly implements apply vocab mask. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject.apply_vocab_mask`，主要用于实现 apply vocab mask 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 171-186: Method ReasonerGrammarObject.copy
```python
    def copy(self):
        new_obj = ReasonerGrammarObject(
            self.grammar.copy() if self.grammar is not None else None,
            self.think_end_id,
            self.think_excluded_token_ids,
            self.max_think_tokens,
            self.enable_token_filter,
            self.token_filter_fn,
            self.allocate_vocab_mask_fn,
            self.move_vocab_mask_fn,
            self.apply_vocab_mask_fn,
        )
        new_obj.tokens_in_think = self.tokens_in_think
        new_obj.tokens_after_end = self.tokens_after_end
        new_obj._finished = self._finished
        return new_obj
```
**EN:** This callable implements `ReasonerGrammarObject.copy` and mainly implements copy. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject.copy`，主要用于实现 copy 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 188-192: Method ReasonerGrammarObject.finished
```python
    @property
    def finished(self):
        if self.grammar is not None:
            return self.grammar.finished
        return self._finished
```
**EN:** This callable implements `ReasonerGrammarObject.finished` and mainly implements finished. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject.finished`，主要用于实现 finished 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 194-199: Method ReasonerGrammarObject.finished
```python
    @finished.setter
    def finished(self, finished):
        if self.grammar is not None:
            self.grammar.finished = finished
        else:
            self._finished = finished
```
**EN:** This callable implements `ReasonerGrammarObject.finished`. It takes `finished` and mainly implements finished. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject.finished`。它接收 `finished`，主要用于实现 finished 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 201-204: Method ReasonerGrammarObject.try_jump_forward
```python
    def try_jump_forward(self, tokenizer):
        if self.grammar is not None:
            return self.grammar.try_jump_forward(tokenizer)
        return None
```
**EN:** This callable implements `ReasonerGrammarObject.try_jump_forward`. It takes `tokenizer` and mainly implements try jump forward. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject.try_jump_forward`。它接收 `tokenizer`，主要用于实现 try jump forward 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 206-209: Method ReasonerGrammarObject.jump_forward_str_state
```python
    def jump_forward_str_state(self, helper):
        if self.grammar is not None:
            return self.grammar.jump_forward_str_state(helper)
        return None
```
**EN:** This callable implements `ReasonerGrammarObject.jump_forward_str_state`. It takes `helper` and mainly implements jump forward str state. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject.jump_forward_str_state`。它接收 `helper`，主要用于实现 jump forward str state 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 211-215: Method ReasonerGrammarObject.jump_and_retokenize
```python
    def jump_and_retokenize(self, old_output_ids, new_output_ids, next_state):
        if self.grammar is not None:
            return self.grammar.jump_and_retokenize(
                old_output_ids, new_output_ids, next_state
            )
```
**EN:** This callable implements `ReasonerGrammarObject.jump_and_retokenize`. It takes `old_output_ids`, `new_output_ids`, `next_state` and mainly converts data into another representation. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `ReasonerGrammarObject.jump_and_retokenize`。它接收 `old_output_ids`, `new_output_ids`, `next_state`，主要用于将数据转换为另一种表示。 在这一范围内，它会处理语法或正则约束。

### Lines 218-218: Class ReasonerGrammarBackend
```python
class ReasonerGrammarBackend(BaseGrammarBackend):
```
**EN:** This range introduces `ReasonerGrammarBackend` and defines the structure or metadata that its methods rely on. In this range it handles grammar or regular-expression constraints.
**CN:** 这一段引入 `ReasonerGrammarBackend`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理语法或正则约束。

### Lines 219-264: Method ReasonerGrammarBackend.__init__
```python
    def __init__(
        self,
        grammar_backend: BaseGrammarBackend,
        reasoning_parser: ReasoningParser,
        tokenizer: Union[PreTrainedTokenizer, PreTrainedTokenizerFast],
        enable_strict_thinking: bool = False,
    ):
        super().__init__()
        self.grammar_backend = grammar_backend
        think_end_ids = tokenizer.encode(
            reasoning_parser.detector.think_end_token, add_special_tokens=False
        )
        if not think_end_ids:
            raise ValueError(
                f"think_end_token '{reasoning_parser.detector.think_end_token}' "
                f"could not be encoded by the tokenizer."
            )
        if len(think_end_ids) != 1:
            raise ValueError(
                f"think_end_token '{reasoning_parser.detector.think_end_token}' "
                "must encode to exactly one token for constrained reasoning."
            )
        self.think_end_id = think_end_ids[0]
        self._enable_strict_thinking = enable_strict_thinking
        self.think_excluded_token_ids = self._get_think_excluded_token_ids(
            reasoning_parser, tokenizer
        )
        self.max_think_tokens = envs.SGLANG_MAX_THINK_TOKENS.get()
        if (
            self.enable_strict_thinking
            and self.think_excluded_token_ids is not None
            and not self.grammar_backend.is_support_token_filter
        ):
            raise ValueError(
                "Strict reasoning format requested but the grammar backend does not "
                "support token filtering. Use a grammar backend that supports token "
                "filtering (e.g., xgrammar) or disable strict reasoning mode."
            )
        self.enable_token_filter = (
            self.enable_strict_thinking
            and self.think_excluded_token_ids is not None
            and self.grammar_backend.is_support_token_filter
        )
        self._token_filter_fn = (
            self.grammar_backend.set_token_filter if self.enable_token_filter else None
        )
```
**EN:** This callable implements `ReasonerGrammarBackend.__init__`. It takes `grammar_backend`, `reasoning_parser`, `tokenizer`, `enable_strict_thinking` and mainly initializes instance state and defaults. In this range it performs defensive checks on invalid state; reads environment-driven configuration; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `ReasonerGrammarBackend.__init__`。它接收 `grammar_backend`, `reasoning_parser`, `tokenizer`, `enable_strict_thinking`，主要用于初始化实例状态与默认值。 在这一范围内，它会对非法状态执行防御性检查；读取环境变量驱动的配置；处理语法或正则约束。

### Lines 266-285: Method ReasonerGrammarBackend._get_think_excluded_token_ids
```python
    def _get_think_excluded_token_ids(
        self,
        reasoning_parser: ReasoningParser,
        tokenizer: Union[PreTrainedTokenizer, PreTrainedTokenizerFast],
    ) -> Optional[List[int]]:
        excluded_ids = []
        if (not self.enable_strict_thinking) or (
            not reasoning_parser.detector.think_excluded_tokens
        ):
            return None
        for token in reasoning_parser.detector.think_excluded_tokens:
            new_ids = tokenizer.encode(token, add_special_tokens=False)
            if not new_ids:
                raise ValueError(
                    f"think_excluded_token '{token}' could not be encoded by the "
                    f"tokenizer. All excluded tokens must be encodable for strict "
                    f"reasoning mode to function correctly."
                )
            excluded_ids += new_ids
        return excluded_ids
```
**EN:** This callable implements `ReasonerGrammarBackend._get_think_excluded_token_ids`. It takes `reasoning_parser`, `tokenizer` and mainly converts data into another representation. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ReasonerGrammarBackend._get_think_excluded_token_ids`。它接收 `reasoning_parser`, `tokenizer`，主要用于将数据转换为另一种表示。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 287-302: Method ReasonerGrammarBackend._make_grammar_object
```python
    def _make_grammar_object(
        self, grammar: Optional[BaseGrammarObject], reasoning: bool
    ) -> ReasonerGrammarObject:
        obj = ReasonerGrammarObject(
            grammar=grammar,
            think_end_id=self.think_end_id,
            think_excluded_token_ids=self.think_excluded_token_ids,
            max_think_tokens=self.max_think_tokens,
            enable_token_filter=self.enable_token_filter,
            token_filter_fn=self._token_filter_fn,
            allocate_vocab_mask_fn=self.grammar_backend.allocate_vocab_mask,
            move_vocab_mask_fn=self.grammar_backend.move_vocab_mask,
            apply_vocab_mask_fn=self.grammar_backend.apply_vocab_mask,
        )
        obj.maybe_init_reasoning(reasoning)
        return obj
```
**EN:** This callable implements `ReasonerGrammarBackend._make_grammar_object`. It takes `grammar`, `reasoning` and mainly implements make grammar object. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `ReasonerGrammarBackend._make_grammar_object`。它接收 `grammar`, `reasoning`，主要用于实现 make grammar object 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 304-310: Method ReasonerGrammarBackend.init_strict_reasoning_grammar
```python
    def init_strict_reasoning_grammar(
        self, reasoning: bool
    ) -> Optional[BaseGrammarObject]:
        """Create a grammar object for strict token filtering only (no inner grammar)."""
        if not self.enable_strict_thinking:
            return None
        return self._make_grammar_object(None, reasoning)
```
**EN:** This callable implements `ReasonerGrammarBackend.init_strict_reasoning_grammar`. It takes `reasoning` and mainly implements init strict reasoning grammar. The docstring states: "Create a grammar object for strict token filtering only (no inner grammar)." In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `ReasonerGrammarBackend.init_strict_reasoning_grammar`。它接收 `reasoning`，主要用于实现 init strict reasoning grammar 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 312-318: Method ReasonerGrammarBackend._init_value_dispatch
```python
    def _init_value_dispatch(
        self, key: Tuple[str, str], reasoning: bool
    ) -> Optional[BaseGrammarObject]:
        ret = self.grammar_backend._init_value_dispatch(key, reasoning)
        if ret is None or isinstance(ret, InvalidGrammarObject):
            return ret
        return self._make_grammar_object(ret, reasoning)
```
**EN:** This callable implements `ReasonerGrammarBackend._init_value_dispatch`. It takes `key`, `reasoning` and mainly implements init value dispatch. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `ReasonerGrammarBackend._init_value_dispatch`。它接收 `key`, `reasoning`，主要用于实现 init value dispatch 相关逻辑。 在这一范围内，它会处理语法或正则约束。

## Key Concepts / 关键概念
- `ReasonerGrammarObject`: core class or state container / 核心类或状态容器
- `ReasonerGrammarBackend`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `typing`
- **Third-party / 第三方**: `torch`, `transformers`
- **Internal modules / 内部模块**: `sglang.srt.environ`, `sglang.srt.parser.reasoning_parser`, `.base_grammar_backend`
