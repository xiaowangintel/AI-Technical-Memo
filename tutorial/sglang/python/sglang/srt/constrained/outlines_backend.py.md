# outlines_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/constrained/outlines_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the constrained decoding and grammar control part of the SRT runtime and implements logic centered on `outlines_backend`. The module docstring frames it as: "Constrained decoding with outlines backend." / 该模块属于 SRT 运行时的约束解码与语法控制部分，主要实现围绕 `outlines_backend` 的逻辑。 它对外提供的主要入口包括 `OutlinesGrammar`, `OutlinesGrammarBackend`, `build_regex_from_object`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-41: Module imports, constants, and setup
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
"""Constrained decoding with outlines backend."""

import json
import logging
from typing import Dict, List, Optional, Tuple, Union

import interegular
import torch
from outlines.fsm.guide import RegexGuide
from outlines.models.transformers import TransformerTokenizer
from pydantic import BaseModel

from sglang.srt.constrained.base_grammar_backend import (
    BaseGrammarBackend,
    BaseGrammarObject,
    InvalidGrammarObject,
)
from sglang.srt.constrained.outlines_jump_forward import OutlinesJumpForwardMap

try:
    from outlines.fsm.json_schema import build_regex_from_schema
except ImportError:
    from outlines_core.fsm.json_schema import build_regex_from_schema


logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; handles grammar or regular-expression constraints.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；处理语法或正则约束。

### Lines 42-42: Class OutlinesGrammar
```python
class OutlinesGrammar(BaseGrammarObject):
```
**EN:** This range introduces `OutlinesGrammar` and defines the structure or metadata that its methods rely on. In this range it handles grammar or regular-expression constraints.
**CN:** 这一段引入 `OutlinesGrammar`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理语法或正则约束。

### Lines 43-51: Method OutlinesGrammar.__init__
```python
    def __init__(
        self,
        guide: RegexGuide,
        jump_forward_map: Union[OutlinesJumpForwardMap, None],
    ) -> None:
        super().__init__()
        self.guide = guide
        self.jump_forward_map = jump_forward_map
        self.state = 0
```
**EN:** This callable implements `OutlinesGrammar.__init__`. It takes `guide`, `jump_forward_map` and mainly initializes instance state and defaults. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `OutlinesGrammar.__init__`。它接收 `guide`, `jump_forward_map`，主要用于初始化实例状态与默认值。 在这一范围内，它会处理语法或正则约束。

### Lines 53-54: Method OutlinesGrammar.accept_token
```python
    def accept_token(self, token: int):
        self.state = self.guide.get_next_state(self.state, token)
```
**EN:** This callable implements `OutlinesGrammar.accept_token`. It takes `token` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `OutlinesGrammar.accept_token`。它接收 `token`，主要用于将数据转换为另一种表示。

### Lines 56-59: Method OutlinesGrammar.allocate_vocab_mask
```python
    def allocate_vocab_mask(
        self, vocab_size: int, batch_size: int, device
    ) -> torch.Tensor:
        return torch.zeros(batch_size, vocab_size, dtype=torch.bool, device=device)
```
**EN:** This callable implements `OutlinesGrammar.allocate_vocab_mask`. It takes `vocab_size`, `batch_size`, `device` and mainly implements allocate vocab mask.
**CN:** 这一可调用对象实现了 `OutlinesGrammar.allocate_vocab_mask`。它接收 `vocab_size`, `batch_size`, `device`，主要用于实现 allocate vocab mask 相关逻辑。

### Lines 61-63: Method OutlinesGrammar.move_vocab_mask
```python
    @staticmethod
    def move_vocab_mask(vocab_mask: torch.Tensor, device) -> torch.Tensor:
        return vocab_mask
```
**EN:** This callable implements `OutlinesGrammar.move_vocab_mask`. It takes `vocab_mask`, `device` and mainly implements move vocab mask.
**CN:** 这一可调用对象实现了 `OutlinesGrammar.move_vocab_mask`。它接收 `vocab_mask`, `device`，主要用于实现 move vocab mask 相关逻辑。

### Lines 65-71: Method OutlinesGrammar.fill_vocab_mask
```python
    def fill_vocab_mask(self, vocab_mask: torch.Tensor, idx: int) -> None:
        tokens = torch.tensor(
            self.guide.get_next_instruction(self.state).tokens, dtype=torch.int64
        ).to(vocab_mask.device, non_blocking=True)
        vocab_mask = vocab_mask[idx]
        vocab_mask.fill_(1)
        vocab_mask.scatter_(0, tokens, torch.zeros_like(tokens, dtype=torch.bool))
```
**EN:** This callable implements `OutlinesGrammar.fill_vocab_mask`. It takes `vocab_mask`, `idx` and mainly implements fill vocab mask.
**CN:** 这一可调用对象实现了 `OutlinesGrammar.fill_vocab_mask`。它接收 `vocab_mask`, `idx`，主要用于实现 fill vocab mask 相关逻辑。

### Lines 73-75: Method OutlinesGrammar.apply_vocab_mask
```python
    @staticmethod
    def apply_vocab_mask(logits: torch.Tensor, vocab_mask: torch.Tensor):
        logits.masked_fill_(vocab_mask, float("-inf"))
```
**EN:** This callable implements `OutlinesGrammar.apply_vocab_mask`. It takes `logits`, `vocab_mask` and mainly implements apply vocab mask.
**CN:** 这一可调用对象实现了 `OutlinesGrammar.apply_vocab_mask`。它接收 `logits`, `vocab_mask`，主要用于实现 apply vocab mask 相关逻辑。

### Lines 77-78: Method OutlinesGrammar.copy
```python
    def copy(self):
        return OutlinesGrammar(self.guide, self.jump_forward_map)
```
**EN:** This callable implements `OutlinesGrammar.copy` and mainly implements copy. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `OutlinesGrammar.copy`，主要用于实现 copy 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 80-102: Method OutlinesGrammar.try_jump_forward
```python
    def try_jump_forward(self, tokenizer) -> Optional[Tuple]:
        if not self.jump_forward_map:
            return None

        jump_forward_bytes = self.jump_forward_map.jump_forward_byte(self.state)
        if jump_forward_bytes is None or len(jump_forward_bytes) <= 1:
            return None

        # preprocess the jump forward string
        suffix_bytes = []
        continuation_range = range(0x80, 0xC0)
        cur_state = self.state
        while (
            len(jump_forward_bytes) and jump_forward_bytes[0][0] in continuation_range
        ):
            # continuation bytes
            byte_edge = jump_forward_bytes.pop(0)
            suffix_bytes.append(byte_edge[0])
            cur_state = byte_edge[1]

        suffix_tokens = [f"<0x{hex(b)[2:].upper()}>" for b in suffix_bytes]
        suffix_ids = tokenizer.convert_tokens_to_ids(suffix_tokens)
        return suffix_ids, cur_state
```
**EN:** This callable implements `OutlinesGrammar.try_jump_forward`. It takes `tokenizer` and mainly implements try jump forward.
**CN:** 这一可调用对象实现了 `OutlinesGrammar.try_jump_forward`。它接收 `tokenizer`，主要用于实现 try jump forward 相关逻辑。

### Lines 104-106: Method OutlinesGrammar.jump_forward_str_state
```python
    def jump_forward_str_state(self, helper: Tuple[List[int], str]) -> Tuple[str, int]:
        _, cur_state = helper
        return self.jump_forward_map.jump_forward_symbol(cur_state)
```
**EN:** This callable implements `OutlinesGrammar.jump_forward_str_state`. It takes `helper` and mainly implements jump forward str state.
**CN:** 这一可调用对象实现了 `OutlinesGrammar.jump_forward_str_state`。它接收 `helper`，主要用于实现 jump forward str state 相关逻辑。

### Lines 108-111: Method OutlinesGrammar.jump_and_retokenize
```python
    def jump_and_retokenize(
        self, old_output_ids: List[int], new_output_ids: List[int], next_state: int
    ):
        self.state = next_state
```
**EN:** This callable implements `OutlinesGrammar.jump_and_retokenize`. It takes `old_output_ids`, `new_output_ids`, `next_state` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `OutlinesGrammar.jump_and_retokenize`。它接收 `old_output_ids`, `new_output_ids`, `next_state`，主要用于将数据转换为另一种表示。

### Lines 114-114: Class OutlinesGrammarBackend
```python
class OutlinesGrammarBackend(BaseGrammarBackend):
```
**EN:** This range introduces `OutlinesGrammarBackend` and defines the structure or metadata that its methods rely on. In this range it handles grammar or regular-expression constraints.
**CN:** 这一段引入 `OutlinesGrammarBackend`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理语法或正则约束。

### Lines 115-143: Method OutlinesGrammarBackend.__init__
```python
    def __init__(
        self,
        tokenizer,
        whitespace_pattern: str | None,
    ):
        super().__init__()

        try:
            self.outlines_tokenizer = TransformerTokenizer(tokenizer)
        except AttributeError:
            # FIXME: tmp fix for chatglm2 & chatglm3 (pad_token_id=0)
            origin_pad_token_id = tokenizer.pad_token_id

            def fset(self, value):
                self._value = value

            type(tokenizer).pad_token_id = property(
                fget=type(tokenizer).pad_token_id.fget, fset=fset
            )
            self.outlines_tokenizer = TransformerTokenizer(tokenizer)
            self.outlines_tokenizer.tokenizer.pad_token_id = origin_pad_token_id
            self.outlines_tokenizer.pad_token_id = origin_pad_token_id
            self.outlines_tokenizer.pad_token = (
                self.outlines_tokenizer.tokenizer.pad_token
            )
            self.outlines_tokenizer.vocabulary = (
                self.outlines_tokenizer.tokenizer.get_vocab()
            )
        self.whitespace_pattern = whitespace_pattern
```
**EN:** This callable implements `OutlinesGrammarBackend.__init__`. It takes `tokenizer`, `whitespace_pattern` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `OutlinesGrammarBackend.__init__`。它接收 `tokenizer`, `whitespace_pattern`，主要用于初始化实例状态与默认值。

### Lines 145-158: Method OutlinesGrammarBackend._compile_regex
```python
    def _compile_regex(self, regex: str) -> BaseGrammarObject:
        try:
            if hasattr(RegexGuide, "from_regex"):
                # outlines >= 0.1.1
                guide = RegexGuide.from_regex(regex, self.outlines_tokenizer)
            else:
                # outlines <= 0.0.46
                guide = RegexGuide(regex, self.outlines_tokenizer)
        except interegular.patterns.InvalidSyntax as e:
            logger.error(f"Hit invalid regex schema: {regex=}, {e=}")
            return InvalidGrammarObject(str(e))

        jump_forward_map = None
        return OutlinesGrammar(guide, jump_forward_map)
```
**EN:** This callable implements `OutlinesGrammarBackend._compile_regex`. It takes `regex` and mainly implements compile regex. In this range it emits logs for diagnostics; handles grammar or regular-expression constraints; prepares compilation-related behavior.
**CN:** 这一可调用对象实现了 `OutlinesGrammarBackend._compile_regex`。它接收 `regex`，主要用于实现 compile regex 相关逻辑。 在这一范围内，它会输出日志以便诊断；处理语法或正则约束；处理与编译相关的行为。

### Lines 160-161: Method OutlinesGrammarBackend.dispatch_ebnf
```python
    def dispatch_ebnf(self, key_string: str):
        return super().dispatch_ebnf(key_string)
```
**EN:** This callable implements `OutlinesGrammarBackend.dispatch_ebnf`. It takes `key_string` and mainly implements dispatch ebnf.
**CN:** 这一可调用对象实现了 `OutlinesGrammarBackend.dispatch_ebnf`。它接收 `key_string`，主要用于实现 dispatch ebnf 相关逻辑。

### Lines 163-164: Method OutlinesGrammarBackend.dispatch_structural_tag
```python
    def dispatch_structural_tag(self, key_string: str):
        return super().dispatch_structural_tag(key_string)
```
**EN:** This callable implements `OutlinesGrammarBackend.dispatch_structural_tag`. It takes `key_string` and mainly implements dispatch structural tag.
**CN:** 这一可调用对象实现了 `OutlinesGrammarBackend.dispatch_structural_tag`。它接收 `key_string`，主要用于实现 dispatch structural tag 相关逻辑。

### Lines 166-175: Method OutlinesGrammarBackend.dispatch_json
```python
    def dispatch_json(self, key_string: str):
        try:
            regex = build_regex_from_object(
                key_string,
                whitespace_pattern=self.whitespace_pattern,
            )
        except (NotImplementedError, json.decoder.JSONDecodeError, ValueError) as e:
            logger.error(f"Hit invalid json_schema: {key_string=}, {e=}")
            return InvalidGrammarObject(str(e))
        return self._compile_regex(regex)
```
**EN:** This callable implements `OutlinesGrammarBackend.dispatch_json`. It takes `key_string` and mainly implements dispatch json. In this range it emits logs for diagnostics; serializes or parses JSON payloads; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `OutlinesGrammarBackend.dispatch_json`。它接收 `key_string`，主要用于实现 dispatch json 相关逻辑。 在这一范围内，它会输出日志以便诊断；序列化或解析 JSON 数据；处理语法或正则约束。

### Lines 177-178: Method OutlinesGrammarBackend.dispatch_regex
```python
    def dispatch_regex(self, key_string: str):
        return self._compile_regex(key_string)
```
**EN:** This callable implements `OutlinesGrammarBackend.dispatch_regex`. It takes `key_string` and mainly implements dispatch regex. In this range it handles grammar or regular-expression constraints; prepares compilation-related behavior.
**CN:** 这一可调用对象实现了 `OutlinesGrammarBackend.dispatch_regex`。它接收 `key_string`，主要用于实现 dispatch regex 相关逻辑。 在这一范围内，它会处理语法或正则约束；处理与编译相关的行为。

### Lines 181-190: Function build_regex_from_object
```python
def build_regex_from_object(
    object: Union[str, BaseModel, Dict], whitespace_pattern: Optional[str] = None
):
    if isinstance(object, type(BaseModel)):
        schema = json.dumps(object.model_json_schema())
    elif isinstance(object, Dict):
        schema = json.dumps(object)
    else:
        schema = object
    return build_regex_from_schema(schema, whitespace_pattern)
```
**EN:** This callable implements `build_regex_from_object`. It takes `object`, `whitespace_pattern` and mainly constructs data from an external representation. In this range it serializes or parses JSON payloads; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `build_regex_from_object`。它接收 `object`, `whitespace_pattern`，主要用于从外部表示构造数据。 在这一范围内，它会序列化或解析 JSON 数据；处理语法或正则约束。

## Key Concepts / 关键概念
- `OutlinesGrammar`: core class or state container / 核心类或状态容器
- `OutlinesGrammarBackend`: core class or state container / 核心类或状态容器
- `build_regex_from_object`: constructs data from an external representation / 从外部表示构造数据

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`, `typing`
- **Third-party / 第三方**: `interegular`, `torch`, `outlines.fsm.guide`, `outlines.models.transformers`, `pydantic`, `outlines.fsm.json_schema`, `outlines_core.fsm.json_schema`
- **Internal modules / 内部模块**: `sglang.srt.constrained.base_grammar_backend`, `sglang.srt.constrained.outlines_jump_forward`
