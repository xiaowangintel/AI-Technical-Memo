# outlines_jump_forward.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/constrained/outlines_jump_forward.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the constrained decoding and grammar control part of the SRT runtime and implements logic centered on `outlines_jump_forward`. The module docstring frames it as: "Faster constrained decoding with jump forward decoding / compressed finite state machine." / 该模块属于 SRT 运行时的约束解码与语法控制部分，主要实现围绕 `outlines_jump_forward` 的逻辑。 它对外提供的主要入口包括 `JumpEdge`, `disk_cache`, `init_state_to_jump_forward`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-45: Module imports, constants, and setup
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
"""
Faster constrained decoding with jump forward decoding / compressed finite state machine.
Reference: https://lmsys.org/blog/2024-02-05-compressed-fsm/
"""

import dataclasses
import logging
from collections import defaultdict
from typing import Optional

import interegular
from interegular import InvalidSyntax
from outlines.caching import cache

from sglang.srt.utils import get_bool_env_var

try:
    # outlines >= 0.1.0
    from outlines_core.fsm.outlines_core_rs import FSMInfo
    from outlines_core.fsm.regex import make_byte_level_fsm, make_deterministic_fsm
except ImportError:
    # outlines <= 0.0.46
    from outlines.fsm.regex import FSMInfo, make_byte_level_fsm, make_deterministic_fsm

IP_REGEX = r"((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)"

# Env var was set in sglang.srt.server_args.ServerArgs.__post_init__
DISABLE_DISK_CACHE = get_bool_env_var("SGLANG_DISABLE_OUTLINES_DISK_CACHE", "true")

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; handles grammar or regular-expression constraints.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；处理语法或正则约束。

### Lines 46-51: Class JumpEdge
```python
@dataclasses.dataclass
class JumpEdge:
    symbol: str = None
    symbol_next_state: int = None
    byte: int = None
    byte_next_state: int = None
```
**EN:** This range introduces `JumpEdge` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `JumpEdge`，并定义其后续方法依赖的结构或元数据。

### Lines 54-58: Function disk_cache
```python
def disk_cache(expire: Optional[float] = None, typed=False, ignore=()):
    if not DISABLE_DISK_CACHE:
        return cache(expire, typed, ignore)
    else:
        return lambda fn: None
```
**EN:** This callable implements `disk_cache`. It takes `expire`, `typed`, `ignore` and mainly implements disk cache.
**CN:** 这一可调用对象实现了 `disk_cache`。它接收 `expire`, `typed`, `ignore`，主要用于实现 disk cache 相关逻辑。

### Lines 61-120: Function init_state_to_jump_forward (part 1/2)
```python
@disk_cache()
def init_state_to_jump_forward(regex_string):
    try:
        regex_pattern = interegular.parse_pattern(regex_string)
    except InvalidSyntax as e:
        logger.warning(f"skip invalid regex: {regex_string}, {e=}")
        return

    byte_fsm = make_byte_level_fsm(regex_pattern.to_fsm().reduce(), keep_utf8=True)
    regex_fsm, _ = make_deterministic_fsm(byte_fsm)

    fsm_info: FSMInfo = regex_fsm.fsm_info

    symbol_to_id = fsm_info.alphabet_symbol_mapping
    id_to_symbol = {}
    for symbol, id_ in symbol_to_id.items():
        id_to_symbol.setdefault(id_, []).append(symbol)

    transitions = fsm_info.transitions

    outgoings_ct = defaultdict(int)
    # NOTE(lsyin): Final states can lead to terminate, so they have one outgoing edge naturally
    for s in fsm_info.finals:
        outgoings_ct[s] = 1

    state_to_jump_forward = {}
    for (state, id_), next_state in transitions.items():
        if id_ == fsm_info.alphabet_anything_value:
            # Arbitrarily symbol cannot be recognized as jump forward
            continue

        symbols = id_to_symbol[id_]
        for c in symbols:
            if len(c) > 1:
                # Skip byte level transitions like c = "5E"
                continue

            outgoings_ct[state] += 1
            if outgoings_ct[state] > 1:
                if state in state_to_jump_forward:
                    del state_to_jump_forward[state]
                break

            state_to_jump_forward[state] = JumpEdge(
                symbol=c,
                symbol_next_state=next_state,
            )

    # Process the byte level jump forward
    outgoings_ct = defaultdict(int)
    for s in fsm_info.finals:
        outgoings_ct[s] = 1

    for (state, id_), next_state in transitions.items():
        if id_ == fsm_info.alphabet_anything_value:
            continue
        symbols = id_to_symbol[id_]
        for c in symbols:
            byte_ = None
            if len(c) == 1 and ord(c) < 0x80:
```
**EN:** This callable implements `init_state_to_jump_forward`. It takes `regex_string` and mainly converts data into another representation. This chunk is part 1 of 2 for the same logical block. In this range it emits logs for diagnostics; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `init_state_to_jump_forward`。它接收 `regex_string`，主要用于将数据转换为另一种表示。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会输出日志以便诊断；处理语法或正则约束。

### Lines 121-139: Function init_state_to_jump_forward (part 2/2)
```python
                # ASCII character
                byte_ = ord(c)
            elif len(c) > 1:
                # FIXME: This logic is due to the leading \x00
                # https://github.com/outlines-dev/outlines/pull/930
                byte_ = int(symbols[0][1:], 16)

            if byte_ is not None:
                outgoings_ct[state] += 1
                if outgoings_ct[state] > 1:
                    if state in state_to_jump_forward:
                        del state_to_jump_forward[state]
                    break
                e = state_to_jump_forward.get(state, JumpEdge())
                e.byte = byte_
                e.byte_next_state = next_state
                state_to_jump_forward[state] = e

    return state_to_jump_forward
```
**EN:** This callable implements `init_state_to_jump_forward`. It takes `regex_string` and mainly converts data into another representation. This chunk is part 2 of 2 for the same logical block.
**CN:** 这一可调用对象实现了 `init_state_to_jump_forward`。它接收 `regex_string`，主要用于将数据转换为另一种表示。 该片段是同一逻辑块的第 2/2 部分。

### Lines 142-142: Class OutlinesJumpForwardMap
```python
class OutlinesJumpForwardMap:
```
**EN:** This range introduces `OutlinesJumpForwardMap` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `OutlinesJumpForwardMap`，并定义其后续方法依赖的结构或元数据。

### Lines 143-144: Method OutlinesJumpForwardMap.__init__
```python
    def __init__(self, regex_string):
        self.state_to_jump_forward = init_state_to_jump_forward(regex_string)
```
**EN:** This callable implements `OutlinesJumpForwardMap.__init__`. It takes `regex_string` and mainly initializes instance state and defaults. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `OutlinesJumpForwardMap.__init__`。它接收 `regex_string`，主要用于初始化实例状态与默认值。 在这一范围内，它会处理语法或正则约束。

### Lines 146-157: Method OutlinesJumpForwardMap.jump_forward_symbol
```python
    def jump_forward_symbol(self, state):
        jump_forward_str = ""
        next_state = state
        while state in self.state_to_jump_forward:
            e = self.state_to_jump_forward[state]
            if e.symbol is None:
                break
            jump_forward_str += e.symbol
            next_state = e.symbol_next_state
            state = next_state

        return jump_forward_str, next_state
```
**EN:** This callable implements `OutlinesJumpForwardMap.jump_forward_symbol`. It takes `state` and mainly implements jump forward symbol.
**CN:** 这一可调用对象实现了 `OutlinesJumpForwardMap.jump_forward_symbol`。它接收 `state`，主要用于实现 jump forward symbol 相关逻辑。

### Lines 159-172: Method OutlinesJumpForwardMap.jump_forward_byte
```python
    def jump_forward_byte(self, state):
        if state not in self.state_to_jump_forward:
            return None

        jump_forward_bytes = []
        next_state = None
        while state in self.state_to_jump_forward:
            e = self.state_to_jump_forward[state]
            assert e.byte is not None and e.byte_next_state is not None
            jump_forward_bytes.append((e.byte, e.byte_next_state))
            next_state = e.byte_next_state
            state = next_state

        return jump_forward_bytes
```
**EN:** This callable implements `OutlinesJumpForwardMap.jump_forward_byte`. It takes `state` and mainly implements jump forward byte. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `OutlinesJumpForwardMap.jump_forward_byte`。它接收 `state`，主要用于实现 jump forward byte 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 174-178: Method OutlinesJumpForwardMap.is_jump_forward_symbol_state
```python
    def is_jump_forward_symbol_state(self, state):
        return (
            state in self.state_to_jump_forward
            and self.state_to_jump_forward[state].symbol is not None
        )
```
**EN:** This callable implements `OutlinesJumpForwardMap.is_jump_forward_symbol_state`. It takes `state` and mainly implements is jump forward symbol state.
**CN:** 这一可调用对象实现了 `OutlinesJumpForwardMap.is_jump_forward_symbol_state`。它接收 `state`，主要用于实现 is jump forward symbol state 相关逻辑。

### Lines 181-188: Function test_main
```python
def test_main(regex_string):
    jump_forward_map = OutlinesJumpForwardMap(regex_string)
    for state, e in jump_forward_map.state_to_jump_forward.items():
        if e.symbol is not None:
            jump_forward_str, next_state = jump_forward_map.jump_forward_symbol(state)
            print(f"{state} -> {next_state}", jump_forward_str)
        bytes_ = jump_forward_map.jump_forward_byte(state)
        print(f"{state} -> {bytes_[-1][1]}", [hex(b) for b, _ in bytes_])
```
**EN:** This callable implements `test_main`. It takes `regex_string` and mainly implements test main. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `test_main`。它接收 `regex_string`，主要用于实现 test main 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 189-200: Module imports, constants, and setup
```python


if __name__ == "__main__":
    import outlines

    outlines.caching.clear_cache()
    test_main(r"The google's DNS sever address is " + IP_REGEX)
    test_main(r"霍格沃茨特快列车|霍比特人比尔博")
    # 霍格: \xe9\x9c\x8d \xe6\xa0\xbc ...
    # 霍比: \xe9\x9c\x8d \xe6\xaf\x94 ...

    test_main(r"[-+]?[0-9]+[ ]*")
```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; handles grammar or regular-expression constraints.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；处理语法或正则约束。

## Key Concepts / 关键概念
- `IP_REGEX`: module constant or capability flag / 模块常量或能力标记
- `DISABLE_DISK_CACHE`: module constant or capability flag / 模块常量或能力标记
- `JumpEdge`: core class or state container / 核心类或状态容器
- `disk_cache`: implements disk cache / 实现 disk cache 相关逻辑
- `init_state_to_jump_forward`: converts data into another representation / 将数据转换为另一种表示
- `OutlinesJumpForwardMap`: core class or state container / 核心类或状态容器
- `test_main`: implements test main / 实现 test main 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `logging`, `collections`, `typing`
- **Third-party / 第三方**: `interegular`, `outlines.caching`, `outlines_core.fsm.outlines_core_rs`, `outlines_core.fsm.regex`, `outlines`, `outlines.fsm.regex`
- **Internal modules / 内部模块**: `sglang.srt.utils`
