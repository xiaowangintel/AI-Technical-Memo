# dims_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/dims_spec/dims_parser.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on tensor dimension specification handling. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于张量维度规格处理。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from typing import Optional

from sglang.srt.debug_utils.comparator.dims_spec.comment_parser import (
    _CommentSuffix,
    _parse_comment_suffix,
)
from sglang.srt.debug_utils.comparator.dims_spec.dim_parser import parse_dim
from sglang.srt.debug_utils.comparator.dims_spec.types import (
    SQUEEZE_DIM_NAME,
    DimSpec,
    DimsSpec,
    ParallelAxis,
)
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 18-18: Define class `_SingletonDimUtil` and class context / 定义类 `_SingletonDimUtil`及类上下文
```python
class _SingletonDimUtil:
```
**EN:** This section introduces `_SingletonDimUtil`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_SingletonDimUtil`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 19-19: Document the module intent / 说明模块意图
```python
    """Utilities for squeeze dims (name="1") and their singleton tensor-name mapping."""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 21-21: Declare fields for `_SingletonDimUtil` such as `PREFIX` / 为 `_SingletonDimUtil` 声明字段，例如 `PREFIX`
```python
    PREFIX: str = "singleton"
```
**EN:** These lines declare the state carried by `_SingletonDimUtil`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `_SingletonDimUtil` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 24-25: Implement method `is_squeeze` for `_SingletonDimUtil` / 为 `_SingletonDimUtil` 实现方法 `is_squeeze`
```python
    def is_squeeze(spec: DimSpec) -> bool:
        return spec.name == SQUEEZE_DIM_NAME
```
**EN:** Method `is_squeeze` implements behavior on `_SingletonDimUtil`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `is_squeeze` 为 `_SingletonDimUtil` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 28-29: Implement method `filter_out` for `_SingletonDimUtil` / 为 `_SingletonDimUtil` 实现方法 `filter_out`
```python
    def filter_out(dim_specs: list[DimSpec]) -> list[DimSpec]:
        return [s for s in dim_specs if not _SingletonDimUtil.is_squeeze(s)]
```
**EN:** Method `filter_out` implements behavior on `_SingletonDimUtil`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `filter_out` 为 `_SingletonDimUtil` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 32-33: Implement method `make_name` for `_SingletonDimUtil` / 为 `_SingletonDimUtil` 实现方法 `make_name`
```python
    def make_name(index: int) -> str:
        return f"{_SingletonDimUtil.PREFIX}{index}"
```
**EN:** Method `make_name` implements behavior on `_SingletonDimUtil`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `make_name` 为 `_SingletonDimUtil` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 36-40: Implement method `is_singleton_name` for `_SingletonDimUtil` / 为 `_SingletonDimUtil` 实现方法 `is_singleton_name`
```python
    def is_singleton_name(name: str) -> bool:
        return (
            name.startswith(_SingletonDimUtil.PREFIX)
            and name[len(_SingletonDimUtil.PREFIX) :].isdigit()
        )
```
**EN:** Method `is_singleton_name` implements behavior on `_SingletonDimUtil`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `is_singleton_name` 为 `_SingletonDimUtil` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 43-55: Implement method `sanitize_names` for `_SingletonDimUtil` / 为 `_SingletonDimUtil` 实现方法 `sanitize_names`
```python
    def sanitize_names(names: list[str]) -> list[str]:
        """Replace '1' with 'singleton0', 'singleton1', ... for named tensor compatibility."""
        result: list[str] = []
        sq_idx: int = 0

        for name in names:
            if name == SQUEEZE_DIM_NAME:
                result.append(_SingletonDimUtil.make_name(sq_idx))
                sq_idx += 1
            else:
                result.append(name)

        return result
```
**EN:** Method `sanitize_names` implements behavior on `_SingletonDimUtil`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `sanitize_names` 为 `_SingletonDimUtil` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 58-106: Implement function `parse_dims` / 实现函数 `parse_dims`
```python
def parse_dims(dims_str: str) -> DimsSpec:
    """Parse ``"b s[cp:zigzag] h[tp] d # dp:=moe_dp ep:replicated"`` → :class:`DimsSpec`.

    The shape part (before ``#``) produces :pyattr:`DimsSpec.dims`.
    The declaration part (after ``#``) is scanned for:
    - ``dp:=<group>`` → :pyattr:`DimsSpec.dp_group_alias`
    - ``axis:replicated`` → :pyattr:`DimsSpec.replicated_axes`
    """
    parts: list[str] = dims_str.split("#", maxsplit=1)
    raw: str = parts[0]

    if not raw.strip():
        raise ValueError("dims string must not be empty")

    dims: list[DimSpec] = [parse_dim(token) for token in raw.strip().split()]

    # Collect all semantic names (expanding fused sub-dims) for duplicate detection
    semantic_names: list[str] = []
    for spec in dims:
        if _SingletonDimUtil.is_squeeze(spec):
            continue
        semantic_names.extend(spec.sub_dims)

    if len(semantic_names) != len(set(semantic_names)):
        duplicates = sorted({n for n in semantic_names if semantic_names.count(n) > 1})
        raise ValueError(f"Duplicate dim names: {duplicates}")

    comment_suffix: _CommentSuffix = (
        _parse_comment_suffix(parts[1]) if len(parts) > 1 else _CommentSuffix()
    )
    dp_group_alias: Optional[str] = comment_suffix.dp_group_alias
    replicated_axes: frozenset[ParallelAxis] = comment_suffix.replicated_axes

    sharded_axes: set[ParallelAxis] = {
        m.axis for spec in dims for m in spec.parallel_modifiers
    }
    conflict: frozenset[ParallelAxis] = replicated_axes & sharded_axes
    if conflict:
        conflict_names: str = ", ".join(sorted(a.value for a in conflict))
        raise ValueError(
            f"Axes declared as both sharded (in dim spec) and replicated "
            f"(in # declaration): {conflict_names}"
        )

    return DimsSpec(
        dims=dims,
        dp_group_alias=dp_group_alias,
        replicated_axes=replicated_axes,
    )
```
**EN:** Function `parse_dims` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `parse_dims` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 109-113: Implement function `resolve_dim_names` / 实现函数 `resolve_dim_names`
```python
def resolve_dim_names(dims_str: str) -> list[str]:
    """Parse dims string and return tensor-compatible names ('1' → 'singleton0', ...)."""
    specs: list[DimSpec] = parse_dims(dims_str).dims
    names: list[str] = [spec.sanitized_name for spec in specs]
    return _SingletonDimUtil.sanitize_names(names)
```
**EN:** Function `resolve_dim_names` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `resolve_dim_names` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `_SingletonDimUtil`, `parse_dims`, `resolve_dim_names`
- **Module role / 模块角色**: Tensor dimension specification handling / 张量维度规格处理
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.dims_spec.comment_parser`, `sglang.srt.debug_utils.comparator.dims_spec.dim_parser`, `sglang.srt.debug_utils.comparator.dims_spec.types`
