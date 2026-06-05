# comment_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/dims_spec/comment_parser.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on tensor dimension specification handling. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于张量维度规格处理。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

import re
from typing import NamedTuple, Optional

from sglang.srt.debug_utils.comparator.dims_spec.types import (
    _AXIS_LOOKUP,
    ParallelAxis,
)
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 11-12: Declare module-level symbols such as `_DP_ALIAS_PATTERN`, `_REPLICATED_PATTERN` / 声明模块级符号，例如 `_DP_ALIAS_PATTERN`, `_REPLICATED_PATTERN`
```python
_DP_ALIAS_PATTERN = re.compile(r"^dp:=(\w+)$")
_REPLICATED_PATTERN = re.compile(r"^(\w+):replicated$")
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 15-15: Define class `_CommentSuffix` and class context / 定义类 `_CommentSuffix`及类上下文
```python
class _CommentSuffix(NamedTuple):
```
**EN:** This section introduces `_CommentSuffix`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_CommentSuffix`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 16-17: Declare fields for `_CommentSuffix` such as `dp_group_alias`, `replicated_axes` / 为 `_CommentSuffix` 声明字段，例如 `dp_group_alias`, `replicated_axes`
```python
    dp_group_alias: Optional[str] = None
    replicated_axes: frozenset[ParallelAxis] = frozenset()
```
**EN:** These lines declare the state carried by `_CommentSuffix`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `_CommentSuffix` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 20-59: Implement helper `_parse_comment_suffix` / 实现辅助函数 `_parse_comment_suffix`
```python
def _parse_comment_suffix(declaration_part: str) -> _CommentSuffix:
    """Parse the ``#`` comment section for dp alias and replicated declarations."""
    dp_group_alias: Optional[str] = None
    replicated_axes: set[ParallelAxis] = set()

    for token in declaration_part.strip().split():
        dp_match = _DP_ALIAS_PATTERN.match(token)
        if dp_match is not None:
            if dp_group_alias is not None:
                raise ValueError(
                    f"Duplicate dp alias declaration: already have {dp_group_alias!r}, "
                    f"got {dp_match.group(1)!r}"
                )
            dp_group_alias = dp_match.group(1)
            continue

        repl_match = _REPLICATED_PATTERN.match(token)
        if repl_match is not None:
            axis_str: str = repl_match.group(1)
            axis: Optional[ParallelAxis] = _AXIS_LOOKUP.get(axis_str)
            if axis is None:
                raise ValueError(
                    f"Unknown axis {axis_str!r} in replicated declaration: {token!r}"
                )
            if axis in replicated_axes:
                raise ValueError(
                    f"Duplicate replicated declaration for axis {axis_str!r}"
                )
            replicated_axes.add(axis)
            continue

        raise ValueError(
            f"Unrecognized token {token!r} in # comment section. "
            f"Expected 'dp:=<group>' or '<axis>:replicated'."
        )

    return _CommentSuffix(
        dp_group_alias=dp_group_alias,
        replicated_axes=frozenset(replicated_axes),
    )
```
**EN:** Function `_parse_comment_suffix` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_parse_comment_suffix` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `_CommentSuffix`, `_parse_comment_suffix`
- **Module role / 模块角色**: Tensor dimension specification handling / 张量维度规格处理
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `re`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.dims_spec.types`
