# llguidance_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/constrained/llguidance_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the constrained decoding and grammar control part of the SRT runtime and implements logic centered on `llguidance_backend`. The module docstring frames it as: "Constrained decoding with llguidance backend." / 该模块属于 SRT 运行时的约束解码与语法控制部分，主要实现围绕 `llguidance_backend` 的逻辑。 它对外提供的主要入口包括 `GuidanceGrammar`, `GuidanceBackend`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-39: Module imports, constants, and setup
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
"""Constrained decoding with llguidance backend."""

import json
import logging
import os
from typing import List, Optional, Tuple

import torch
from llguidance import LLMatcher, LLTokenizer, StructTag, grammar_from
from llguidance.hf import from_tokenizer
from llguidance.torch import (
    allocate_token_bitmask,
    apply_token_bitmask_inplace,
    fill_next_token_bitmask,
)

from sglang.srt.constrained.base_grammar_backend import (
    BaseGrammarBackend,
    BaseGrammarObject,
    InvalidGrammarObject,
)
from sglang.srt.constrained.utils import is_legacy_structural_tag

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; handles grammar or regular-expression constraints.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；处理语法或正则约束。

### Lines 40-41: Class GuidanceGrammar
```python
class GuidanceGrammar(BaseGrammarObject):

```
**EN:** This range introduces `GuidanceGrammar` and defines the structure or metadata that its methods rely on. In this range it handles grammar or regular-expression constraints.
**CN:** 这一段引入 `GuidanceGrammar`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理语法或正则约束。

### Lines 42-55: Method GuidanceGrammar.__init__
```python
    def __init__(self, llguidance_tokenizer: LLTokenizer, serialized_grammar: str):
        super().__init__()
        self.llguidance_tokenizer = llguidance_tokenizer
        self.serialized_grammar = serialized_grammar

        self.ll_matcher = LLMatcher(
            self.llguidance_tokenizer,
            self.serialized_grammar,
            log_level=int(os.environ.get("LLGUIDANCE_LOG_LEVEL", "1")),
        )
        self._check_err()

        self.bitmask = None
        self.eos_token = self.llguidance_tokenizer.eos_token
```
**EN:** This callable implements `GuidanceGrammar.__init__`. It takes `llguidance_tokenizer`, `serialized_grammar` and mainly initializes instance state and defaults. In this range it reads environment-driven configuration; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `GuidanceGrammar.__init__`。它接收 `llguidance_tokenizer`, `serialized_grammar`，主要用于初始化实例状态与默认值。 在这一范围内，它会读取环境变量驱动的配置；处理语法或正则约束。

### Lines 57-64: Method GuidanceGrammar.accept_token
```python
    def accept_token(self, token: int):
        if self.finished:
            return
        if self.ll_matcher.is_stopped() and token == self.eos_token:
            self.finished = True
            return
        self.ll_matcher.consume_token(token)
        self._check_err()
```
**EN:** This callable implements `GuidanceGrammar.accept_token`. It takes `token` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `GuidanceGrammar.accept_token`。它接收 `token`，主要用于将数据转换为另一种表示。

### Lines 66-74: Method GuidanceGrammar.rollback
```python
    def rollback(self, num_tokens: int) -> None:
        if num_tokens <= 0:
            return
        if self.finished:
            self.finished = False
            # EOS token after stop isn't tracked in ll_matcher
            num_tokens -= 1
        self.ll_matcher.rollback(num_tokens)
        self._check_err()
```
**EN:** This callable implements `GuidanceGrammar.rollback`. It takes `num_tokens` and mainly implements rollback.
**CN:** 这一可调用对象实现了 `GuidanceGrammar.rollback`。它接收 `num_tokens`，主要用于实现 rollback 相关逻辑。

### Lines 76-77: Method GuidanceGrammar.is_terminated
```python
    def is_terminated(self):
        return self.finished
```
**EN:** This callable implements `GuidanceGrammar.is_terminated` and mainly implements is terminated.
**CN:** 这一可调用对象实现了 `GuidanceGrammar.is_terminated`，主要用于实现 is terminated 相关逻辑。

### Lines 79-81: Method GuidanceGrammar.fill_vocab_mask
```python
    def fill_vocab_mask(self, vocab_mask: torch.Tensor, idx: int) -> None:
        fill_next_token_bitmask(self.ll_matcher, vocab_mask, idx)
        self._check_err()
```
**EN:** This callable implements `GuidanceGrammar.fill_vocab_mask`. It takes `vocab_mask`, `idx` and mainly implements fill vocab mask.
**CN:** 这一可调用对象实现了 `GuidanceGrammar.fill_vocab_mask`。它接收 `vocab_mask`, `idx`，主要用于实现 fill vocab mask 相关逻辑。

### Lines 83-95: Method GuidanceGrammar.allocate_vocab_mask
```python
    def allocate_vocab_mask(
        self, vocab_size: int, batch_size: int, device
    ) -> torch.Tensor:
        if self.bitmask is None or self.bitmask.shape[0] < batch_size:
            # only create bitmask when batch gets larger
            self.bitmask = allocate_token_bitmask(
                batch_size, self.llguidance_tokenizer.vocab_size
            )
            bitmask = self.bitmask
        else:
            bitmask = self.bitmask[:batch_size]

        return bitmask
```
**EN:** This callable implements `GuidanceGrammar.allocate_vocab_mask`. It takes `vocab_size`, `batch_size`, `device` and mainly implements allocate vocab mask.
**CN:** 这一可调用对象实现了 `GuidanceGrammar.allocate_vocab_mask`。它接收 `vocab_size`, `batch_size`, `device`，主要用于实现 allocate vocab mask 相关逻辑。

### Lines 97-99: Method GuidanceGrammar.move_vocab_mask
```python
    @staticmethod
    def move_vocab_mask(vocab_mask: torch.Tensor, device) -> torch.Tensor:
        return vocab_mask.to(device, non_blocking=True)
```
**EN:** This callable implements `GuidanceGrammar.move_vocab_mask`. It takes `vocab_mask`, `device` and mainly implements move vocab mask.
**CN:** 这一可调用对象实现了 `GuidanceGrammar.move_vocab_mask`。它接收 `vocab_mask`, `device`，主要用于实现 move vocab mask 相关逻辑。

### Lines 101-103: Method GuidanceGrammar.apply_vocab_mask
```python
    @staticmethod
    def apply_vocab_mask(logits: torch.Tensor, vocab_mask: torch.Tensor) -> None:
        apply_token_bitmask_inplace(logits, vocab_mask)
```
**EN:** This callable implements `GuidanceGrammar.apply_vocab_mask`. It takes `logits`, `vocab_mask` and mainly implements apply vocab mask.
**CN:** 这一可调用对象实现了 `GuidanceGrammar.apply_vocab_mask`。它接收 `logits`, `vocab_mask`，主要用于实现 apply vocab mask 相关逻辑。

### Lines 105-109: Method GuidanceGrammar.copy
```python
    def copy(self):
        return GuidanceGrammar(
            llguidance_tokenizer=self.llguidance_tokenizer,
            serialized_grammar=self.serialized_grammar,
        )
```
**EN:** This callable implements `GuidanceGrammar.copy` and mainly implements copy. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `GuidanceGrammar.copy`，主要用于实现 copy 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 111-116: Method GuidanceGrammar.try_jump_forward
```python
    def try_jump_forward(self, tokenizer) -> Optional[Tuple[List[int], str]]:
        ff_tokens = self.ll_matcher.compute_ff_tokens()
        if ff_tokens:
            return ff_tokens, ""
        else:
            return None
```
**EN:** This callable implements `GuidanceGrammar.try_jump_forward`. It takes `tokenizer` and mainly implements try jump forward.
**CN:** 这一可调用对象实现了 `GuidanceGrammar.try_jump_forward`。它接收 `tokenizer`，主要用于实现 try jump forward 相关逻辑。

### Lines 118-119: Method GuidanceGrammar.jump_forward_str_state
```python
    def jump_forward_str_state(self, helper: Tuple[List[int], str]) -> Tuple[str, int]:
        return "", -1
```
**EN:** This callable implements `GuidanceGrammar.jump_forward_str_state`. It takes `helper` and mainly implements jump forward str state.
**CN:** 这一可调用对象实现了 `GuidanceGrammar.jump_forward_str_state`。它接收 `helper`，主要用于实现 jump forward str state 相关逻辑。

### Lines 121-124: Method GuidanceGrammar.jump_and_retokenize
```python
    def jump_and_retokenize(
        self, old_output_ids: List[int], new_output_ids: List[int], next_state: int
    ):
        pass
```
**EN:** This callable implements `GuidanceGrammar.jump_and_retokenize`. It takes `old_output_ids`, `new_output_ids`, `next_state` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `GuidanceGrammar.jump_and_retokenize`。它接收 `old_output_ids`, `new_output_ids`, `next_state`，主要用于将数据转换为另一种表示。

### Lines 126-128: Method GuidanceGrammar._check_err
```python
    def _check_err(self) -> None:
        if self.ll_matcher.is_error():
            raise ValueError(self.ll_matcher.get_error())
```
**EN:** This callable implements `GuidanceGrammar._check_err` and mainly checks preconditions and compatibility. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `GuidanceGrammar._check_err`，主要用于检查前置条件与兼容性。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 131-132: Class GuidanceBackend
```python
class GuidanceBackend(BaseGrammarBackend):

```
**EN:** This range introduces `GuidanceBackend` and defines the structure or metadata that its methods rely on. In this range it handles grammar or regular-expression constraints.
**CN:** 这一段引入 `GuidanceBackend`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理语法或正则约束。

### Lines 133-145: Method GuidanceBackend.__init__
```python
    def __init__(
        self,
        tokenizer,
        any_whitespace: bool = True,
        whitespace_pattern: Optional[str] = None,
        n_vocab: Optional[int] = None,
    ):
        super().__init__()

        self.tokenizer = tokenizer
        self.any_whitespace = any_whitespace
        self.whitespace_pattern = whitespace_pattern
        self.llguidance_tokenizer = from_tokenizer(self.tokenizer, n_vocab)
```
**EN:** This callable implements `GuidanceBackend.__init__`. It takes `tokenizer`, `any_whitespace`, `whitespace_pattern`, `n_vocab` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `GuidanceBackend.__init__`。它接收 `tokenizer`, `any_whitespace`, `whitespace_pattern`, `n_vocab`，主要用于初始化实例状态与默认值。

### Lines 147-155: Method GuidanceBackend._from_serialized
```python
    def _from_serialized(self, serialized_grammar) -> BaseGrammarObject:
        try:
            return GuidanceGrammar(
                llguidance_tokenizer=self.llguidance_tokenizer,
                serialized_grammar=serialized_grammar,
            )
        except Exception as e:
            logger.error(f"Hit invalid grammar: {serialized_grammar=}, {e=}")
            return InvalidGrammarObject(str(e))
```
**EN:** This callable implements `GuidanceBackend._from_serialized`. It takes `serialized_grammar` and mainly constructs data from an external representation. In this range it emits logs for diagnostics; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `GuidanceBackend._from_serialized`。它接收 `serialized_grammar`，主要用于从外部表示构造数据。 在这一范围内，它会输出日志以便诊断；处理语法或正则约束。

### Lines 157-169: Method GuidanceBackend.dispatch_json
```python
    def dispatch_json(self, key_string: str) -> BaseGrammarObject:
        try:
            serialized_grammar = LLMatcher.grammar_from_json_schema(
                key_string,
                defaults={
                    "whitespace_flexible": self.any_whitespace,
                    "whitespace_pattern": self.whitespace_pattern,
                },
            )
        except Exception as e:
            logger.error(f"Hit invalid json_schema: {key_string=}, {e=}")
            return InvalidGrammarObject(str(e))
        return self._from_serialized(serialized_grammar)
```
**EN:** This callable implements `GuidanceBackend.dispatch_json`. It takes `key_string` and mainly implements dispatch json. In this range it emits logs for diagnostics; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `GuidanceBackend.dispatch_json`。它接收 `key_string`，主要用于实现 dispatch json 相关逻辑。 在这一范围内，它会输出日志以便诊断；处理语法或正则约束。

### Lines 171-173: Method GuidanceBackend.dispatch_regex
```python
    def dispatch_regex(self, key_string: str) -> BaseGrammarObject:
        serialized_grammar = grammar_from("regex", key_string)
        return self._from_serialized(serialized_grammar)
```
**EN:** This callable implements `GuidanceBackend.dispatch_regex`. It takes `key_string` and mainly implements dispatch regex. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `GuidanceBackend.dispatch_regex`。它接收 `key_string`，主要用于实现 dispatch regex 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 175-181: Method GuidanceBackend.dispatch_ebnf
```python
    def dispatch_ebnf(self, key_string: str) -> BaseGrammarObject:
        try:
            serialized_grammar = grammar_from("ebnf", key_string)
            return self._from_serialized(serialized_grammar)
        except ValueError as e:
            logger.error(f"Hit invalid ebnf: {key_string=}, {e=}")
            return InvalidGrammarObject(str(e))
```
**EN:** This callable implements `GuidanceBackend.dispatch_ebnf`. It takes `key_string` and mainly implements dispatch ebnf. In this range it emits logs for diagnostics; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `GuidanceBackend.dispatch_ebnf`。它接收 `key_string`，主要用于实现 dispatch ebnf 相关逻辑。 在这一范围内，它会输出日志以便诊断；处理语法或正则约束。

### Lines 183-200: Method GuidanceBackend.dispatch_structural_tag
```python
    def dispatch_structural_tag(self, key_string: str) -> BaseGrammarObject:
        try:
            structural_tag = json.loads(key_string)
            assert is_legacy_structural_tag(structural_tag)
            tags = [
                StructTag(
                    begin=structure["begin"],
                    grammar=structure["schema"],
                    end=structure["end"],
                    trigger=structural_tag["triggers"][0],  # TODO?
                )
                for structure in structural_tag["structures"]
            ]
            g = StructTag.to_grammar(tags)
            return self._from_serialized(g)
        except Exception as e:
            logger.error(f"Hit invalid structural_tag: {key_string=}, {e=}")
            return InvalidGrammarObject(str(e))
```
**EN:** This callable implements `GuidanceBackend.dispatch_structural_tag`. It takes `key_string` and mainly implements dispatch structural tag. In this range it performs defensive checks on invalid state; emits logs for diagnostics; serializes or parses JSON payloads.
**CN:** 这一可调用对象实现了 `GuidanceBackend.dispatch_structural_tag`。它接收 `key_string`，主要用于实现 dispatch structural tag 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；输出日志以便诊断；序列化或解析 JSON 数据。

## Key Concepts / 关键概念
- `GuidanceGrammar`: core class or state container / 核心类或状态容器
- `GuidanceBackend`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`, `os`, `typing`
- **Third-party / 第三方**: `torch`, `llguidance`, `llguidance.hf`, `llguidance.torch`
- **Internal modules / 内部模块**: `sglang.srt.constrained.base_grammar_backend`, `sglang.srt.constrained.utils`
