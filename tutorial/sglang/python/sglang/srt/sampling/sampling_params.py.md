# sampling_params.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/sampling/sampling_params.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the sampling and penalty control part of the SRT runtime and implements logic centered on `sampling_params`. The module docstring frames it as: "Sampling parameters for text generation." / 该模块属于 SRT 运行时的采样与惩罚控制部分，主要实现围绕 `sampling_params` 的逻辑。 它对外提供的主要入口包括 `SamplingParams`, `get_max_seq_length`, `_max_length_from_subpattern`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: Module imports, constants, and setup
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
"""Sampling parameters for text generation."""

import logging
from typing import Any, Dict, List, Optional, Union

# sre_parse is deprecated in Python 3.11+, use re._parser instead
try:
    import re._parser as sre_parse
except ImportError:
    import sre_parse  # Python < 3.11

_SAMPLING_EPS = 1e-6
TOP_K_ALL = 1 << 30

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; handles grammar or regular-expression constraints.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；处理语法或正则约束。

### Lines 31-39: Class SamplingParams
```python
class SamplingParams:
    """
    The sampling parameters.

    See docs/backend/sampling_params.md or
    https://docs.sglang.io/backend/sampling_params.html
    for the documentation.
    """

```
**EN:** This range introduces `SamplingParams` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "The sampling parameters."
**CN:** 这一段引入 `SamplingParams`，并定义其后续方法依赖的结构或元数据。

### Lines 40-99: Method SamplingParams.__init__ (part 1/2)
```python
    def __init__(
        self,
        max_new_tokens: int = 128,
        stop: Optional[Union[str, List[str]]] = None,
        stop_token_ids: Optional[List[int]] = None,
        stop_regex: Optional[Union[str, List[str]]] = None,
        temperature: float = 1.0,
        top_p: float = 1.0,
        top_k: int = -1,
        min_p: float = 0.0,
        frequency_penalty: float = 0.0,
        presence_penalty: float = 0.0,
        repetition_penalty: float = 1.0,
        min_new_tokens: int = 0,
        n: int = 1,
        json_schema: Optional[str] = None,
        regex: Optional[str] = None,
        ebnf: Optional[str] = None,
        structural_tag: Optional[str] = None,
        ignore_eos: bool = False,
        skip_special_tokens: bool = True,
        spaces_between_special_tokens: bool = True,
        no_stop_trim: bool = False,
        custom_params: Optional[Dict[str, Any]] = None,
        stream_interval: Optional[int] = None,
        logit_bias: Optional[Dict[str, float]] = None,
        sampling_seed: Optional[int] = None,
    ) -> None:
        # For non-optional params, treat None as "use default" so that callers
        # (e.g. /generate) can pass null without crashing verify().
        self.max_new_tokens = max_new_tokens
        self.stop_strs = stop
        if stop_token_ids:
            filtered = {int(t) for t in stop_token_ids if t is not None}
            self.stop_token_ids = filtered or None
        else:
            self.stop_token_ids = None
        self.stop_regex_strs = stop_regex
        self.temperature = temperature if temperature is not None else 1.0
        self.top_p = top_p if top_p is not None else 1.0
        self.top_k = top_k if top_k is not None else -1
        self.min_p = min_p if min_p is not None else 0.0
        self.frequency_penalty = (
            frequency_penalty if frequency_penalty is not None else 0.0
        )
        self.presence_penalty = (
            presence_penalty if presence_penalty is not None else 0.0
        )
        self.repetition_penalty = (
            repetition_penalty if repetition_penalty is not None else 1.0
        )
        self.min_new_tokens = min_new_tokens if min_new_tokens is not None else 0
        self.regex = regex
        self.n = n if n is not None else 1
        self.json_schema = json_schema
        self.ebnf = ebnf
        self.structural_tag = structural_tag
        self.ignore_eos = ignore_eos if ignore_eos is not None else False
        self.skip_special_tokens = (
            skip_special_tokens if skip_special_tokens is not None else True
```
**EN:** This callable implements `SamplingParams.__init__`. It takes `max_new_tokens`, `stop`, `stop_token_ids`, `stop_regex` and mainly initializes instance state and defaults. This chunk is part 1 of 2 for the same logical block. In this range it handles grammar or regular-expression constraints; adjusts sampling behavior and decoding controls.
**CN:** 这一可调用对象实现了 `SamplingParams.__init__`。它接收 `max_new_tokens`, `stop`, `stop_token_ids`, `stop_regex`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会处理语法或正则约束；调整采样行为与解码控制。

### Lines 100-118: Method SamplingParams.__init__ (part 2/2)
```python
        )
        self.spaces_between_special_tokens = (
            spaces_between_special_tokens
            if spaces_between_special_tokens is not None
            else True
        )
        self.no_stop_trim = no_stop_trim if no_stop_trim is not None else False
        self.custom_params = custom_params
        self.stream_interval = stream_interval
        self.logit_bias = logit_bias
        self.sampling_seed = sampling_seed

        # Process some special cases
        if 0 <= self.temperature < _SAMPLING_EPS:
            # top_k = 1 means greedy sampling
            self.temperature = 1.0
            self.top_k = 1
        if self.top_k == -1:
            self.top_k = TOP_K_ALL  # whole vocabulary
```
**EN:** This callable implements `SamplingParams.__init__`. It takes `max_new_tokens`, `stop`, `stop_token_ids`, `stop_regex` and mainly initializes instance state and defaults. This chunk is part 2 of 2 for the same logical block. In this range it adjusts sampling behavior and decoding controls.
**CN:** 这一可调用对象实现了 `SamplingParams.__init__`。它接收 `max_new_tokens`, `stop`, `stop_token_ids`, `stop_regex`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会调整采样行为与解码控制。

### Lines 120-176: Method SamplingParams.verify
```python
    def verify(self, vocab_size):
        if self.temperature < 0.0:
            raise ValueError(
                f"temperature must be non-negative, got {self.temperature}."
            )
        if not 0.0 < self.top_p <= 1.0:
            raise ValueError(f"top_p must be in (0, 1], got {self.top_p}.")
        if not 0.0 <= self.min_p <= 1.0:
            raise ValueError(f"min_p must be in [0, 1], got {self.min_p}.")
        if self.top_k < 1 or self.top_k == -1:
            raise ValueError(
                f"top_k must be -1 (disable) or at least 1, got {self.top_k}."
            )
        if not -2.0 <= self.frequency_penalty <= 2.0:
            raise ValueError(
                "frequency_penalty must be in [-2, 2], got "
                f"{self.frequency_penalty}."
            )
        if not -2.0 <= self.presence_penalty <= 2.0:
            raise ValueError(
                "presence_penalty must be in [-2, 2], got " f"{self.presence_penalty}."
            )
        if not 0.0 < self.repetition_penalty <= 2.0:
            raise ValueError(
                "repetition_penalty must be in (0, 2] (1.0 = no penalty), "
                f"got {self.repetition_penalty}."
            )
        if not 0 <= self.min_new_tokens:
            raise ValueError(
                f"min_new_tokens must be in [0, max_new_tokens], got "
                f"{self.min_new_tokens}."
            )
        if self.max_new_tokens is not None:
            if self.max_new_tokens < 0:
                raise ValueError(
                    f"max_new_tokens must be at least 0, got {self.max_new_tokens}."
                )
            if not self.min_new_tokens <= self.max_new_tokens:
                raise ValueError(
                    f"min_new_tokens must be in [0, max_new_tokens({self.max_new_tokens})], got "
                    f"{self.min_new_tokens}."
                )
        if self.logit_bias is not None:
            for token_id in self.logit_bias:
                if not 0 <= int(token_id) < vocab_size:
                    raise ValueError(
                        f"logit_bias must has keys in [0, {vocab_size - 1}], got "
                        f"{token_id}."
                    )

        grammars = [
            self.json_schema,
            self.regex,
            self.ebnf,
        ]  # since mutually exclusive, only one can be set
        if sum(x is not None for x in grammars) > 1:
            raise ValueError("Only one of regex, json_schema, or ebnf can be set.")
```
**EN:** This callable implements `SamplingParams.verify`. It takes `vocab_size` and mainly validates inputs and invariants. In this range it performs defensive checks on invalid state; handles grammar or regular-expression constraints; adjusts sampling behavior and decoding controls.
**CN:** 这一可调用对象实现了 `SamplingParams.verify`。它接收 `vocab_size`，主要用于校验输入与不变量。 在这一范围内，它会对非法状态执行防御性检查；处理语法或正则约束；调整采样行为与解码控制。

### Lines 178-210: Method SamplingParams.normalize
```python
    def normalize(self, tokenizer):
        # Process stop strings
        if self.stop_strs is None:
            self.stop_strs = []
            self.stop_str_max_len = 0
        else:
            if isinstance(self.stop_strs, str):
                self.stop_strs = [self.stop_strs]

            stop_str_max_len = 0
            for stop_str in self.stop_strs:
                if tokenizer is not None:
                    stop_str_ids = tokenizer.encode(stop_str, add_special_tokens=False)
                    stop_str_max_len = max(stop_str_max_len, len(stop_str_ids))
                else:
                    stop_str_max_len = max(stop_str_max_len, len(stop_str))
            self.stop_str_max_len = stop_str_max_len

        # Process stop regex strings
        if self.stop_regex_strs is None:
            self.stop_regex_strs = []
            self.stop_regex_max_len = 0
        else:
            if isinstance(self.stop_regex_strs, str):
                self.stop_regex_strs = [self.stop_regex_strs]

            stop_regex_max_len = 0
            for stop_regex in self.stop_regex_strs:
                stop_regex_max_len = max(
                    stop_regex_max_len, get_max_seq_length(stop_regex)
                )

            self.stop_regex_max_len = stop_regex_max_len
```
**EN:** This callable implements `SamplingParams.normalize`. It takes `tokenizer` and mainly normalizes user-provided values. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `SamplingParams.normalize`。它接收 `tokenizer`，主要用于规范化用户提供的值。 在这一范围内，它会处理语法或正则约束。

### Lines 211-216: Module-level logic
```python


# This function gets a strict upperbound on the maximum number of tokens that would need
# to be buffered to match the input regex string
# NOTE: in the worst case, one character that needs to be buffered corresponds to one
# token
```
**EN:** This range organizes module-level state and shared setup. In this range it handles grammar or regular-expression constraints.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 217-218: Function get_max_seq_length
```python
def get_max_seq_length(regex_str: str):
    return _max_length_from_subpattern(sre_parse.parse(regex_str))
```
**EN:** This callable implements `get_max_seq_length`. It takes `regex_str` and mainly retrieves a value or derived view. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `get_max_seq_length`。它接收 `regex_str`，主要用于获取某个值或派生视图。 在这一范围内，它会处理语法或正则约束。

### Lines 219-223: Module-level constants and helpers
```python


MAX_LEN = 2**30


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 224-258: Function _max_length_from_subpattern
```python
def _max_length_from_subpattern(subpattern: sre_parse.SubPattern):
    total = 0
    for token, value in subpattern:
        if token in {
            sre_parse.LITERAL,  # `value` is any one character
            sre_parse.IN,  # Any character within `value`
            sre_parse.ANY,  # "."
        }:
            total += 1
        elif token == sre_parse.SUBPATTERN:
            # EG: (a\d+) ->
            # [(SUBPATTERN,
            #   (1, 0, 0, [(LITERAL, 97),
            #              (MAX_REPEAT, (1, MAXREPEAT, [(IN, [(CATEGORY, CATEGORY_DIGIT)])]))]))]
            _, _, _, inner_subpattern = value
            total += _max_length_from_subpattern(inner_subpattern)
        elif token == sre_parse.BRANCH:
            _, branches = value
            total += max(_max_length_from_subpattern(branch) for branch in branches)
        elif token in {sre_parse.MAX_REPEAT, sre_parse.MIN_REPEAT}:
            _, max_num_repeat, inner_subpattern = value
            if max_num_repeat == sre_parse.MAXREPEAT:
                total += MAX_LEN
            else:
                total += max_num_repeat * _max_length_from_subpattern(inner_subpattern)
        elif token == sre_parse.AT:
            # These are zero-width assertions like ^, $, and \b that don't add to the max
            # length
            total += 0
        else:
            logger.warning(f"Got unhandled regex token: {token}")

            total += MAX_LEN

    return total
```
**EN:** This callable implements `_max_length_from_subpattern`. It takes `subpattern` and mainly constructs data from an external representation. In this range it emits logs for diagnostics; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `_max_length_from_subpattern`。它接收 `subpattern`，主要用于从外部表示构造数据。 在这一范围内，它会输出日志以便诊断；处理语法或正则约束。

## Key Concepts / 关键概念
- `_SAMPLING_EPS`: module constant or capability flag / 模块常量或能力标记
- `TOP_K_ALL`: module constant or capability flag / 模块常量或能力标记
- `SamplingParams`: core class or state container / 核心类或状态容器
- `get_max_seq_length`: retrieves a value or derived view / 获取某个值或派生视图
- `MAX_LEN`: module constant or capability flag / 模块常量或能力标记
- `_max_length_from_subpattern`: constructs data from an external representation / 从外部表示构造数据

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `typing`, `re._parser`, `sre_parse`
