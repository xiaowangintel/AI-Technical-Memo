# modifier_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/dims_spec/modifier_parser.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on tensor dimension specification handling. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于张量维度规格处理。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from typing import Optional

from sglang.srt.debug_utils.comparator.dims_spec.types import (
    _AXIS_LOOKUP,
    _QUALIFIER_LOOKUP,
    Ordering,
    ParallelAxis,
    ParallelModifier,
    Reduction,
)
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 15-63: Implement helper `_parse_modifier_token` / 实现辅助函数 `_parse_modifier_token`
```python
def _parse_modifier_token(modifier_token: str, dim_token: str) -> ParallelModifier:
    """Parse 'sp', 'cp:zigzag', 'tp:partial', or 'cp:zigzag+partial' → ParallelModifier.

    Format: ``axis`` or ``axis:qual`` or ``axis:qual+qual``.
    Colon separates axis from qualifiers; ``+`` separates multiple qualifiers.
    """
    axis_str: str
    qualifiers_str: str
    if ":" in modifier_token:
        axis_str, qualifiers_str = modifier_token.split(":", maxsplit=1)
    else:
        axis_str, qualifiers_str = modifier_token, ""

    axis_str = axis_str.strip()
    axis: Optional[ParallelAxis] = _AXIS_LOOKUP.get(axis_str)
    if axis is None:
        raise ValueError(
            f"Unknown axis {axis_str!r} in modifier {modifier_token!r} "
            f"of dim spec: {dim_token!r}"
        )

    ordering: Optional[Ordering] = None
    reduction: Optional[Reduction] = None

    for q_str in (q.strip() for q in qualifiers_str.split("+") if q.strip()):
        if q_str == "sharded":
            continue
        qualifier: Optional[Ordering | Reduction] = _QUALIFIER_LOOKUP.get(q_str)
        if qualifier is None:
            raise ValueError(
                f"Unknown qualifier {q_str!r} in modifier "
                f"{modifier_token!r} of dim spec: {dim_token!r}"
            )
        if isinstance(qualifier, Ordering):
            if ordering is not None:
                raise ValueError(
                    f"Multiple ordering values in modifier "
                    f"{modifier_token!r} of dim spec: {dim_token!r}"
                )
            ordering = qualifier
        else:
            if reduction is not None:
                raise ValueError(
                    f"Multiple reduction values in modifier "
                    f"{modifier_token!r} of dim spec: {dim_token!r}"
                )
            reduction = qualifier

    return ParallelModifier(axis=axis, ordering=ordering, reduction=reduction)
```
**EN:** Function `_parse_modifier_token` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_parse_modifier_token` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 66-84: Implement helper `_parse_modifiers` / 实现辅助函数 `_parse_modifiers`
```python
def _parse_modifiers(
    *, modifiers_str: Optional[str], dim_token: str
) -> list[ParallelModifier]:
    if modifiers_str is None:
        return []

    modifiers: list[ParallelModifier] = []
    seen_axes: set[ParallelAxis] = set()

    for modifier_token in (p.strip() for p in modifiers_str.split(",")):
        modifier: ParallelModifier = _parse_modifier_token(modifier_token, dim_token)
        if modifier.axis in seen_axes:
            raise ValueError(
                f"Duplicate axis {modifier.axis.value!r} in dim spec: {dim_token!r}"
            )
        seen_axes.add(modifier.axis)
        modifiers.append(modifier)

    return modifiers
```
**EN:** Function `_parse_modifiers` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_parse_modifiers` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `_parse_modifier_token`, `_parse_modifiers`
- **Module role / 模块角色**: Tensor dimension specification handling / 张量维度规格处理
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.dims_spec.types`
