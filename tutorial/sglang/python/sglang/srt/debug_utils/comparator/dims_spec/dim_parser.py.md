# dim_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/dims_spec/dim_parser.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on tensor dimension specification handling. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于张量维度规格处理。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

import re
from typing import Optional

from sglang.srt.debug_utils.comparator.dims_spec.modifier_parser import (
    _parse_modifiers,
)
from sglang.srt.debug_utils.comparator.dims_spec.types import (
    SQUEEZE_DIM_NAME,
    DimSpec,
    ParallelModifier,
)
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 15-19: Declare module-level symbols such as `_DIM_PATTERN`, `_FUSED_DIM_PATTERN`, `_SUB_DIM_NAME_PATTERN` / 声明模块级符号，例如 `_DIM_PATTERN`, `_FUSED_DIM_PATTERN`, `_SUB_DIM_NAME_PATTERN`
```python
_DIM_PATTERN = re.compile(r"^(?P<name>[a-zA-Z_]\w*)(?:\[(?P<modifiers>[^\]]+)\])?$")

_FUSED_DIM_PATTERN = re.compile(r"^\((?P<inner>[^)]+)\)(?:\[(?P<modifiers>[^\]]+)\])?$")

_SUB_DIM_NAME_PATTERN = re.compile(r"^[a-zA-Z_]\w*$")
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 22-30: Implement function `parse_dim` / 实现函数 `parse_dim`
```python
def parse_dim(token: str) -> DimSpec:
    if token == SQUEEZE_DIM_NAME:
        return DimSpec(name=SQUEEZE_DIM_NAME)

    fused_match = _FUSED_DIM_PATTERN.match(token)
    if fused_match is not None:
        return _parse_fused_dim(token=token, fused_match=fused_match)

    return _parse_single_dim(token)
```
**EN:** Function `parse_dim` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `parse_dim` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 33-42: Implement helper `_parse_single_dim` / 实现辅助函数 `_parse_single_dim`
```python
def _parse_single_dim(token: str) -> DimSpec:
    match = _DIM_PATTERN.match(token)
    if match is None:
        raise ValueError(f"Invalid dim token: {token!r}")

    name: str = match.group("name")
    modifiers: list[ParallelModifier] = _parse_modifiers(
        modifiers_str=match.group("modifiers"), dim_token=token
    )
    return DimSpec(name=name, parallel_modifiers=modifiers)
```
**EN:** Function `_parse_single_dim` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_parse_single_dim` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 45-68: Implement helper `_parse_fused_dim` / 实现辅助函数 `_parse_fused_dim`
```python
def _parse_fused_dim(*, token: str, fused_match: re.Match[str]) -> DimSpec:
    inner: str = fused_match.group("inner")
    modifiers_str: Optional[str] = fused_match.group("modifiers")

    sub_names: list[str] = [s.strip() for s in inner.split("*")]
    for sub_name in sub_names:
        if not _SUB_DIM_NAME_PATTERN.match(sub_name):
            raise ValueError(
                f"Invalid sub-dim {sub_name!r} in fused dim token: {token!r}"
            )

    if len(sub_names) != len(set(sub_names)):
        raise ValueError(f"Duplicate sub-dim names in fused dim token: {token!r}")

    if len(sub_names) < 2:
        raise ValueError(
            f"Fused dim must have at least 2 sub-dims, got {len(sub_names)} in: {token!r}"
        )

    fused_name: str = "*".join(sub_names)
    modifiers: list[ParallelModifier] = _parse_modifiers(
        modifiers_str=modifiers_str, dim_token=token
    )
    return DimSpec(name=fused_name, parallel_modifiers=modifiers)
```
**EN:** Function `_parse_fused_dim` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_parse_fused_dim` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `parse_dim`, `_parse_single_dim`, `_parse_fused_dim`
- **Module role / 模块角色**: Tensor dimension specification handling / 张量维度规格处理
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `re`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.dims_spec.modifier_parser`, `sglang.srt.debug_utils.comparator.dims_spec.types`
