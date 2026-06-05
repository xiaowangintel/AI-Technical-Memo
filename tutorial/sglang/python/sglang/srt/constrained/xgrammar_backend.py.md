# xgrammar_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/constrained/xgrammar_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the constrained decoding and grammar control part of the SRT runtime and implements logic centered on `xgrammar_backend`. The module docstring frames it as: "Constrained decoding with xgrammar backend." / 该模块属于 SRT 运行时的约束解码与语法控制部分，主要实现围绕 `xgrammar_backend` 的逻辑。 它对外提供的主要入口包括 `XGrammarGrammar`, `TokenizerNotSupportedError`, `XGrammarGrammarBackend`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60: Module imports, constants, and setup
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
"""Constrained decoding with xgrammar backend."""

import dataclasses
import json
import logging
from typing import Dict, List, Optional, Tuple, Union

import torch
from xgrammar import (
    CompiledGrammar,
    GrammarCompiler,
    GrammarMatcher,
    StructuralTag,
    StructuralTagItem,
    TokenizerInfo,
    allocate_token_bitmask,
)

from sglang.srt.constrained.base_grammar_backend import (
    BaseGrammarBackend,
    BaseGrammarObject,
    GrammarStats,
    InvalidGrammarObject,
)
from sglang.srt.constrained.torch_ops.bitmask_ops import (
    apply_token_bitmask_inplace_torch,
)
from sglang.srt.constrained.utils import is_legacy_structural_tag
from sglang.srt.utils import is_hip

_is_hip = is_hip()
if _is_hip:
    from sgl_kernel import apply_token_bitmask_inplace_cuda
else:
    from sglang.srt.constrained.triton_ops.bitmask_ops import (
        apply_token_bitmask_inplace_triton,
    )

from sglang.srt.constrained.torch_ops.token_filter_torch_ops import (
    set_token_filter_torch,
)
from sglang.srt.constrained.triton_ops.token_filter_ops import set_token_filter_triton

logger = logging.getLogger(__name__)
MAX_ROLLBACK_TOKENS = 200


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; handles grammar or regular-expression constraints.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；处理语法或正则约束。

### Lines 61-62: Class XGrammarGrammar
```python
class XGrammarGrammar(BaseGrammarObject):

```
**EN:** This range introduces `XGrammarGrammar` and defines the structure or metadata that its methods rely on. In this range it handles grammar or regular-expression constraints.
**CN:** 这一段引入 `XGrammarGrammar`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理语法或正则约束。

### Lines 63-79: Method XGrammarGrammar.__init__
```python
    def __init__(
        self,
        matcher: GrammarMatcher,
        vocab_size: int,
        ctx: CompiledGrammar,
        override_stop_tokens: Optional[Union[List[int], int]],
        key_string: Optional[str] = None,
        grammar_stats: Optional[GrammarStats] = GrammarStats(),
    ) -> None:
        super().__init__()
        self.matcher = matcher
        self.vocab_size = vocab_size
        self.ctx = ctx
        self.override_stop_tokens = override_stop_tokens
        self.accepted_tokens = []
        self.key_string = key_string
        self.grammar_stats = grammar_stats
```
**EN:** This callable implements `XGrammarGrammar.__init__`. It takes `matcher`, `vocab_size`, `ctx`, `override_stop_tokens` and mainly initializes instance state and defaults. In this range it handles grammar or regular-expression constraints; prepares compilation-related behavior.
**CN:** 这一可调用对象实现了 `XGrammarGrammar.__init__`。它接收 `matcher`, `vocab_size`, `ctx`, `override_stop_tokens`，主要用于初始化实例状态与默认值。 在这一范围内，它会处理语法或正则约束；处理与编译相关的行为。

### Lines 81-93: Method XGrammarGrammar.accept_token
```python
    def accept_token(self, token: int):
        if not self.is_terminated():
            self.current_token = token
            accepted = self.matcher.accept_token(token)
            if not accepted:
                # log for debugging
                raise ValueError(
                    f"Tokens not accepted: {token}\n"
                    f"Accepted tokens: {self.accepted_tokens}\n"
                    f"Key string: {self.key_string}"
                )
            else:
                self.accepted_tokens.append(token)
```
**EN:** This callable implements `XGrammarGrammar.accept_token`. It takes `token` and mainly converts data into another representation. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `XGrammarGrammar.accept_token`。它接收 `token`，主要用于将数据转换为另一种表示。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 95-97: Method XGrammarGrammar.rollback
```python
    def rollback(self, k: int):
        self.matcher.rollback(k)
        self.accepted_tokens = self.accepted_tokens[:-k]
```
**EN:** This callable implements `XGrammarGrammar.rollback`. It takes `k` and mainly implements rollback.
**CN:** 这一可调用对象实现了 `XGrammarGrammar.rollback`。它接收 `k`，主要用于实现 rollback 相关逻辑。

### Lines 99-100: Method XGrammarGrammar.is_terminated
```python
    def is_terminated(self):
        return self.matcher.is_terminated()
```
**EN:** This callable implements `XGrammarGrammar.is_terminated` and mainly implements is terminated.
**CN:** 这一可调用对象实现了 `XGrammarGrammar.is_terminated`，主要用于实现 is terminated 相关逻辑。

### Lines 102-105: Method XGrammarGrammar.allocate_vocab_mask
```python
    def allocate_vocab_mask(
        self, vocab_size: int, batch_size: int, device
    ) -> torch.Tensor:
        return allocate_token_bitmask(batch_size, vocab_size)
```
**EN:** This callable implements `XGrammarGrammar.allocate_vocab_mask`. It takes `vocab_size`, `batch_size`, `device` and mainly implements allocate vocab mask.
**CN:** 这一可调用对象实现了 `XGrammarGrammar.allocate_vocab_mask`。它接收 `vocab_size`, `batch_size`, `device`，主要用于实现 allocate vocab mask 相关逻辑。

### Lines 107-108: Method XGrammarGrammar.fill_vocab_mask
```python
    def fill_vocab_mask(self, vocab_mask: torch.Tensor, idx: int) -> None:
        self.matcher.fill_next_token_bitmask(vocab_mask, idx)
```
**EN:** This callable implements `XGrammarGrammar.fill_vocab_mask`. It takes `vocab_mask`, `idx` and mainly implements fill vocab mask.
**CN:** 这一可调用对象实现了 `XGrammarGrammar.fill_vocab_mask`。它接收 `vocab_mask`, `idx`，主要用于实现 fill vocab mask 相关逻辑。

### Lines 110-112: Method XGrammarGrammar.move_vocab_mask
```python
    @staticmethod
    def move_vocab_mask(vocab_mask: torch.Tensor, device) -> torch.Tensor:
        return vocab_mask.to(device, non_blocking=True)
```
**EN:** This callable implements `XGrammarGrammar.move_vocab_mask`. It takes `vocab_mask`, `device` and mainly implements move vocab mask.
**CN:** 这一可调用对象实现了 `XGrammarGrammar.move_vocab_mask`。它接收 `vocab_mask`, `device`，主要用于实现 move vocab mask 相关逻辑。

### Lines 114-123: Method XGrammarGrammar.apply_vocab_mask
```python
    def apply_vocab_mask(self, logits: torch.Tensor, vocab_mask: torch.Tensor) -> None:
        if logits.device.type in {"cuda", "xpu", "musa"}:
            if _is_hip:
                apply_token_bitmask_inplace_cuda(logits, vocab_mask)
            else:
                apply_token_bitmask_inplace_triton(logits, vocab_mask)
        elif logits.device.type == "npu":
            apply_token_bitmask_inplace_torch(logits, vocab_mask)
        else:
            raise RuntimeError(f"Unsupported device: {logits.device.type}")
```
**EN:** This callable implements `XGrammarGrammar.apply_vocab_mask`. It takes `logits`, `vocab_mask` and mainly implements apply vocab mask. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `XGrammarGrammar.apply_vocab_mask`。它接收 `logits`, `vocab_mask`，主要用于实现 apply vocab mask 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 125-142: Method XGrammarGrammar.copy
```python
    def copy(self):
        matcher = GrammarMatcher(
            self.ctx,
            max_rollback_tokens=MAX_ROLLBACK_TOKENS,
            override_stop_tokens=self.override_stop_tokens,
        )
        if grammar_stats := self.grammar_stats:
            grammar_stats = dataclasses.replace(
                grammar_stats, is_cache_hit=True, tree_traversal_time=[]
            )
        return XGrammarGrammar(
            matcher,
            self.vocab_size,
            self.ctx,
            self.override_stop_tokens,
            self.key_string,
            grammar_stats,
        )
```
**EN:** This callable implements `XGrammarGrammar.copy` and mainly implements copy. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `XGrammarGrammar.copy`，主要用于实现 copy 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 144-148: Method XGrammarGrammar.try_jump_forward
```python
    def try_jump_forward(self, tokenizer) -> Optional[Tuple[List[int], str]]:
        s = self.matcher.find_jump_forward_string()
        if s:
            return [], s
        return None
```
**EN:** This callable implements `XGrammarGrammar.try_jump_forward`. It takes `tokenizer` and mainly implements try jump forward.
**CN:** 这一可调用对象实现了 `XGrammarGrammar.try_jump_forward`。它接收 `tokenizer`，主要用于实现 try jump forward 相关逻辑。

### Lines 150-152: Method XGrammarGrammar.jump_forward_str_state
```python
    def jump_forward_str_state(self, helper: Tuple[List[int], str]) -> Tuple[str, int]:
        _, data = helper
        return data, -1
```
**EN:** This callable implements `XGrammarGrammar.jump_forward_str_state`. It takes `helper` and mainly implements jump forward str state.
**CN:** 这一可调用对象实现了 `XGrammarGrammar.jump_forward_str_state`。它接收 `helper`，主要用于实现 jump forward str state 相关逻辑。

### Lines 154-176: Method XGrammarGrammar.jump_and_retokenize
```python
    def jump_and_retokenize(
        self, old_output_ids: List[int], new_output_ids: List[int], next_state: int
    ):
        k = 0
        for i, old_id in enumerate(old_output_ids):
            if old_id == new_output_ids[i]:
                k = i + 1
            else:
                break

        # rollback to the last token that is the same
        if k < len(old_output_ids):
            self.matcher.rollback(len(old_output_ids) - k)

        for i in range(k, len(new_output_ids)):
            if not self.matcher.accept_token(new_output_ids[i]):
                raise ValueError(
                    f"Token not accepted during retokenization: {new_output_ids[i]} "
                    f"at position {i}\n"
                    f"Old output IDs: {old_output_ids}\n"
                    f"New output IDs: {new_output_ids}\n"
                    f"Key string: {self.key_string}"
                )
```
**EN:** This callable implements `XGrammarGrammar.jump_and_retokenize`. It takes `old_output_ids`, `new_output_ids`, `next_state` and mainly converts data into another representation. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `XGrammarGrammar.jump_and_retokenize`。它接收 `old_output_ids`, `new_output_ids`, `next_state`，主要用于将数据转换为另一种表示。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 178-179: Method XGrammarGrammar.__repr__
```python
    def __repr__(self):
        return f"XGrammarGrammar({self.key_string=}, {self.accepted_tokens=}, {self.current_token=})"
```
**EN:** This callable implements `XGrammarGrammar.__repr__` and mainly implements repr. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `XGrammarGrammar.__repr__`，主要用于实现 repr 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 182-185: Class TokenizerNotSupportedError
```python
class TokenizerNotSupportedError(Exception):
    """Raised when tokenizer is not supported by XGrammar backend."""

    pass
```
**EN:** This range introduces `TokenizerNotSupportedError` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Raised when tokenizer is not supported by XGrammar backend." In this range it handles grammar or regular-expression constraints.
**CN:** 这一段引入 `TokenizerNotSupportedError`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理语法或正则约束。

### Lines 188-188: Class XGrammarGrammarBackend
```python
class XGrammarGrammarBackend(BaseGrammarBackend):
```
**EN:** This range introduces `XGrammarGrammarBackend` and defines the structure or metadata that its methods rely on. In this range it handles grammar or regular-expression constraints.
**CN:** 这一段引入 `XGrammarGrammarBackend`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理语法或正则约束。

### Lines 189-223: Method XGrammarGrammarBackend.__init__
```python
    def __init__(
        self,
        tokenizer,
        vocab_size: int,
        model_eos_token_ids: Optional[List[int]] = None,
        any_whitespace: bool = True,
    ):
        super().__init__()

        if hasattr(tokenizer, "init_xgrammar"):
            # For special tokenizer
            tokenizer_info, override_stop_tokens = tokenizer.init_xgrammar()

            if tokenizer_info is None:
                # Not supported tokenizer
                raise TokenizerNotSupportedError(
                    f"Tokenizer type {type(tokenizer).__name__} is not supported by XGrammar"
                )
        else:
            # Create TokenizerInfo with model's EOS tokens as the authoritative stop tokens
            # This ensures consistency between what the model considers EOS and what XGrammar uses
            try:
                tokenizer_info = TokenizerInfo.from_huggingface(
                    tokenizer, vocab_size=vocab_size, stop_token_ids=model_eos_token_ids
                )
                override_stop_tokens = None
            except Exception as e:
                raise TokenizerNotSupportedError(
                    f"Failed to create XGrammar TokenizerInfo from tokenizer: {e}"
                )

        self.grammar_compiler = GrammarCompiler(tokenizer_info=tokenizer_info)
        self.vocab_size = vocab_size
        self.override_stop_tokens = override_stop_tokens
        self.any_whitespace = any_whitespace
```
**EN:** This callable implements `XGrammarGrammarBackend.__init__`. It takes `tokenizer`, `vocab_size`, `model_eos_token_ids`, `any_whitespace` and mainly initializes instance state and defaults. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `XGrammarGrammarBackend.__init__`。它接收 `tokenizer`, `vocab_size`, `model_eos_token_ids`, `any_whitespace`，主要用于初始化实例状态与默认值。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；处理语法或正则约束。

### Lines 225-227: Method XGrammarGrammarBackend.is_support_token_filter
```python
    @property
    def is_support_token_filter(self):
        return True
```
**EN:** This callable implements `XGrammarGrammarBackend.is_support_token_filter` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `XGrammarGrammarBackend.is_support_token_filter`，主要用于将数据转换为另一种表示。

### Lines 229-231: Method XGrammarGrammarBackend.allocate_vocab_mask
```python
    @staticmethod
    def allocate_vocab_mask(vocab_size: int, batch_size: int, device) -> torch.Tensor:
        return allocate_token_bitmask(batch_size, vocab_size)
```
**EN:** This callable implements `XGrammarGrammarBackend.allocate_vocab_mask`. It takes `vocab_size`, `batch_size`, `device` and mainly implements allocate vocab mask.
**CN:** 这一可调用对象实现了 `XGrammarGrammarBackend.allocate_vocab_mask`。它接收 `vocab_size`, `batch_size`, `device`，主要用于实现 allocate vocab mask 相关逻辑。

### Lines 233-235: Method XGrammarGrammarBackend.move_vocab_mask
```python
    @staticmethod
    def move_vocab_mask(vocab_mask: torch.Tensor, device) -> torch.Tensor:
        return vocab_mask.to(device, non_blocking=True)
```
**EN:** This callable implements `XGrammarGrammarBackend.move_vocab_mask`. It takes `vocab_mask`, `device` and mainly implements move vocab mask.
**CN:** 这一可调用对象实现了 `XGrammarGrammarBackend.move_vocab_mask`。它接收 `vocab_mask`, `device`，主要用于实现 move vocab mask 相关逻辑。

### Lines 237-245: Method XGrammarGrammarBackend.apply_vocab_mask
```python
    @staticmethod
    def apply_vocab_mask(logits: torch.Tensor, vocab_mask: torch.Tensor) -> None:
        if logits.device.type in {"cuda", "npu", "xpu", "musa"}:
            if _is_hip:
                apply_token_bitmask_inplace_cuda(logits, vocab_mask)
            else:
                apply_token_bitmask_inplace_triton(logits, vocab_mask)
        else:
            raise RuntimeError(f"Unsupported device: {logits.device.type}")
```
**EN:** This callable implements `XGrammarGrammarBackend.apply_vocab_mask`. It takes `logits`, `vocab_mask` and mainly implements apply vocab mask. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `XGrammarGrammarBackend.apply_vocab_mask`。它接收 `logits`, `vocab_mask`，主要用于实现 apply vocab mask 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 247-270: Method XGrammarGrammarBackend.set_token_filter
```python
    @staticmethod
    def set_token_filter(
        vocab_mask: torch.Tensor,
        token_ids: List[int],
        batch_idx: int,
        is_allowed: bool = True,
        reset_vocab_mask: bool = True,
    ):
        if _is_hip or (vocab_mask.device.type != "cuda"):
            set_token_filter_torch(
                vocab_mask,
                token_ids,
                batch_idx,
                is_allowed=is_allowed,
                reset_vocab_mask=reset_vocab_mask,
            )
        else:
            set_token_filter_triton(
                vocab_mask,
                token_ids,
                batch_idx,
                is_allowed=is_allowed,
                reset_vocab_mask=reset_vocab_mask,
            )
```
**EN:** This callable implements `XGrammarGrammarBackend.set_token_filter`. It takes `vocab_mask`, `token_ids`, `batch_idx`, `is_allowed` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `XGrammarGrammarBackend.set_token_filter`。它接收 `vocab_mask`, `token_ids`, `batch_idx`, `is_allowed`，主要用于将数据转换为另一种表示。

### Lines 272-292: Method XGrammarGrammarBackend._sanitize_structural_format
```python
    @staticmethod
    def _sanitize_structural_format(structural_format):
        """Recursively replace missing json_schema fields with an empty schema."""
        if not isinstance(structural_format, dict):
            return

        fmt_type = structural_format.get("type")
        if fmt_type in {"json_schema", "qwen_xml_parameter"}:
            if structural_format.get("json_schema") is None:
                structural_format["json_schema"] = {}

        if fmt_type == "tag":
            XGrammarGrammarBackend._sanitize_structural_format(
                structural_format.get("content")
            )
        elif fmt_type in {"sequence", "or"}:
            for element in structural_format.get("elements", []):
                XGrammarGrammarBackend._sanitize_structural_format(element)
        elif fmt_type in {"triggered_tags", "tags_with_separator"}:
            for tag in structural_format.get("tags", []):
                XGrammarGrammarBackend._sanitize_structural_format(tag)
```
**EN:** This callable implements `XGrammarGrammarBackend._sanitize_structural_format`. It takes `structural_format` and mainly implements sanitize structural format. The docstring states: "Recursively replace missing json_schema fields with an empty schema." In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `XGrammarGrammarBackend._sanitize_structural_format`。它接收 `structural_format`，主要用于实现 sanitize structural format 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 294-298: Method XGrammarGrammarBackend._sanitize_structural_tag_structures
```python
    @staticmethod
    def _sanitize_structural_tag_structures(structural_tag: Dict) -> None:
        for structure in structural_tag.get("structures", []):
            if structure.get("schema") is None:
                structure["schema"] = {}
```
**EN:** This callable implements `XGrammarGrammarBackend._sanitize_structural_tag_structures`. It takes `structural_tag` and mainly implements sanitize structural tag structures.
**CN:** 这一可调用对象实现了 `XGrammarGrammarBackend._sanitize_structural_tag_structures`。它接收 `structural_tag`，主要用于实现 sanitize structural tag structures 相关逻辑。

### Lines 300-315: Method XGrammarGrammarBackend._from_context
```python
    def _from_context(
        self, ctx: CompiledGrammar, key_string: str, grammar_stats: GrammarStats
    ) -> XGrammarGrammar:
        matcher = GrammarMatcher(
            ctx,
            max_rollback_tokens=MAX_ROLLBACK_TOKENS,
            override_stop_tokens=self.override_stop_tokens,
        )
        return XGrammarGrammar(
            matcher,
            self.vocab_size,
            ctx,
            self.override_stop_tokens,
            key_string,
            grammar_stats,
        )
```
**EN:** This callable implements `XGrammarGrammarBackend._from_context`. It takes `ctx`, `key_string`, `grammar_stats` and mainly constructs data from an external representation. In this range it handles grammar or regular-expression constraints; prepares compilation-related behavior.
**CN:** 这一可调用对象实现了 `XGrammarGrammarBackend._from_context`。它接收 `ctx`, `key_string`, `grammar_stats`，主要用于从外部表示构造数据。 在这一范围内，它会处理语法或正则约束；处理与编译相关的行为。

### Lines 317-330: Method XGrammarGrammarBackend.dispatch_json
```python
    def dispatch_json(self, key_string: str) -> BaseGrammarObject:
        try:
            if key_string == "$$ANY$$":
                # Note: This builtin JSON grammar includes *all* valid JSON (including, for example, arrays at the root)
                ctx = self.grammar_compiler.compile_builtin_json_grammar()
            else:
                ctx = self.grammar_compiler.compile_json_schema(
                    schema=key_string, any_whitespace=self.any_whitespace
                )

        except (RuntimeError, json.decoder.JSONDecodeError, UnicodeDecodeError) as e:
            logger.error(f"Hit invalid json_schema: {key_string=}, {e=}")
            return InvalidGrammarObject(str(e))
        return self._from_context(ctx, key_string, GrammarStats(dispatch_type="json"))
```
**EN:** This callable implements `XGrammarGrammarBackend.dispatch_json`. It takes `key_string` and mainly implements dispatch json. In this range it emits logs for diagnostics; serializes or parses JSON payloads; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `XGrammarGrammarBackend.dispatch_json`。它接收 `key_string`，主要用于实现 dispatch json 相关逻辑。 在这一范围内，它会输出日志以便诊断；序列化或解析 JSON 数据；处理语法或正则约束。

### Lines 332-338: Method XGrammarGrammarBackend.dispatch_ebnf
```python
    def dispatch_ebnf(self, key_string: str) -> BaseGrammarObject:
        try:
            ctx = self.grammar_compiler.compile_grammar(key_string)
        except RuntimeError as e:
            logger.error(f"Hit invalid ebnf: {key_string=}, {e=}")
            return InvalidGrammarObject(str(e))
        return self._from_context(ctx, key_string, GrammarStats(dispatch_type="ebnf"))
```
**EN:** This callable implements `XGrammarGrammarBackend.dispatch_ebnf`. It takes `key_string` and mainly implements dispatch ebnf. In this range it emits logs for diagnostics; handles grammar or regular-expression constraints; prepares compilation-related behavior.
**CN:** 这一可调用对象实现了 `XGrammarGrammarBackend.dispatch_ebnf`。它接收 `key_string`，主要用于实现 dispatch ebnf 相关逻辑。 在这一范围内，它会输出日志以便诊断；处理语法或正则约束；处理与编译相关的行为。

### Lines 340-346: Method XGrammarGrammarBackend.dispatch_regex
```python
    def dispatch_regex(self, key_string: str) -> BaseGrammarObject:
        try:
            ctx = self.grammar_compiler.compile_regex(key_string)
        except RuntimeError as e:
            logger.error(f"Hit invalid regex: {key_string=}, {e=}")
            return InvalidGrammarObject(str(e))
        return self._from_context(ctx, key_string, GrammarStats(dispatch_type="regex"))
```
**EN:** This callable implements `XGrammarGrammarBackend.dispatch_regex`. It takes `key_string` and mainly implements dispatch regex. In this range it emits logs for diagnostics; handles grammar or regular-expression constraints; prepares compilation-related behavior.
**CN:** 这一可调用对象实现了 `XGrammarGrammarBackend.dispatch_regex`。它接收 `key_string`，主要用于实现 dispatch regex 相关逻辑。 在这一范围内，它会输出日志以便诊断；处理语法或正则约束；处理与编译相关的行为。

### Lines 348-379: Method XGrammarGrammarBackend.dispatch_structural_tag
```python
    def dispatch_structural_tag(self, key_string: str) -> BaseGrammarObject:
        try:
            # TODO(dark): it's REALLY stupid to construct object from string and decode it again
            structural_tag = json.loads(key_string)
            if is_legacy_structural_tag(structural_tag):
                self._sanitize_structural_tag_structures(structural_tag)
                tags = [
                    StructuralTagItem(
                        begin=structure["begin"],
                        schema=json.dumps(structure["schema"]),
                        end=structure["end"],
                    )
                    for structure in structural_tag["structures"]
                ]
                new_tag = StructuralTag.from_legacy_structural_tag(
                    tags, structural_tag["triggers"]
                )
                new_tag.format.at_least_one = structural_tag.get("at_least_one", False)
                ctx = self.grammar_compiler.compile_structural_tag(new_tag)
            else:
                format_dict = structural_tag.get("format")
                if isinstance(format_dict, dict):
                    self._sanitize_structural_format(format_dict)
                    structural_tag["format"] = format_dict
                    key_string = json.dumps(structural_tag)
                ctx = self.grammar_compiler.compile_structural_tag(key_string)
        except (RuntimeError, json.decoder.JSONDecodeError) as e:
            logger.error(f"Hit invalid structural_tag: {key_string=}, {e=}")
            return InvalidGrammarObject(str(e))
        return self._from_context(
            ctx, key_string, GrammarStats(dispatch_type="structural_tag")
        )
```
**EN:** This callable implements `XGrammarGrammarBackend.dispatch_structural_tag`. It takes `key_string` and mainly implements dispatch structural tag. In this range it sets up imports and shared symbols; emits logs for diagnostics; serializes or parses JSON payloads.
**CN:** 这一可调用对象实现了 `XGrammarGrammarBackend.dispatch_structural_tag`。它接收 `key_string`，主要用于实现 dispatch structural tag 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；序列化或解析 JSON 数据。

### Lines 381-383: Method XGrammarGrammarBackend.reset
```python
    def reset(self):
        super().reset()
        self.grammar_compiler.clear_cache()
```
**EN:** This callable implements `XGrammarGrammarBackend.reset` and mainly applies configuration to mutable state. In this range it handles grammar or regular-expression constraints; prepares compilation-related behavior.
**CN:** 这一可调用对象实现了 `XGrammarGrammarBackend.reset`，主要用于将配置写入可变状态。 在这一范围内，它会处理语法或正则约束；处理与编译相关的行为。

### Lines 386-415: Function demo_test
```python
def demo_test():
    from transformers import AutoConfig, AutoTokenizer

    from sglang.test.test_utils import DEFAULT_MODEL_NAME_FOR_TEST

    tokenizer = AutoTokenizer.from_pretrained(DEFAULT_MODEL_NAME_FOR_TEST)
    hf_config = AutoConfig.from_pretrained(DEFAULT_MODEL_NAME_FOR_TEST)

    # Should use vocab size from model config
    vocab_size = hf_config.vocab_size
    eos_token_id = tokenizer.eos_token_id

    backend = XGrammarGrammarBackend(
        tokenizer, vocab_size=vocab_size, model_eos_token_ids=[eos_token_id]
    )
    regex = r"hello (world|there)"
    grammar = backend.dispatch_regex(regex)
    tokens = [
        tokenizer.encode(t, add_special_tokens=False)[0] for t in ["hello", " world"]
    ]

    # Test termination
    grammar.accept_token(tokens[0])  # accept "hello"
    grammar.accept_token(tokens[1])  # accept " world"
    grammar.accept_token(eos_token_id)  # accept EOS
    assert grammar.is_terminated()

    # Test rollback the terminated state
    grammar.rollback(1)
    assert not grammar.is_terminated()
```
**EN:** This callable implements `demo_test` and mainly implements demo test. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `demo_test`，主要用于实现 demo test 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；处理语法或正则约束。

### Lines 416-419: Module entry point
```python


if __name__ == "__main__":
    demo_test()
```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

## Key Concepts / 关键概念
- `MAX_ROLLBACK_TOKENS`: module constant or capability flag / 模块常量或能力标记
- `XGrammarGrammar`: core class or state container / 核心类或状态容器
- `TokenizerNotSupportedError`: core class or state container / 核心类或状态容器
- `XGrammarGrammarBackend`: core class or state container / 核心类或状态容器
- `demo_test`: implements demo test / 实现 demo test 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `json`, `logging`, `typing`
- **Third-party / 第三方**: `torch`, `xgrammar`, `sgl_kernel`, `transformers`
- **Internal modules / 内部模块**: `sglang.srt.constrained.base_grammar_backend`, `sglang.srt.constrained.torch_ops.bitmask_ops`, `sglang.srt.constrained.utils`, `sglang.srt.utils`, `sglang.srt.constrained.torch_ops.token_filter_torch_ops`, `sglang.srt.constrained.triton_ops.token_filter_ops`, `sglang.srt.constrained.triton_ops.bitmask_ops`, `sglang.test.test_utils`
