# _order.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `functorch/dim/_order.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements dimension-aware tensor wrappers and tracing helpers for experimental named-dimension style transforms.
- **Purpose (CN)**: 实现带维度语义的张量包装器与跟踪辅助逻辑，用于实验性的命名维度变换。
## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```python
from __future__ import annotations

from typing import Any, TYPE_CHECKING


if TYPE_CHECKING:
    from collections.abc import Sequence

import torch  # noqa: TC002

from ._dim_entry import _match_levels, DimEntry, ndim_of_levels


def _wrap_dim(arg: Any, orig_ndim: int, allow_none: bool = True) -> DimEntry:
    """
    Convert various dimension representations to DimEntry.
```
- **EN**: The import section wires together PyTorch-local modules such as torch, ._dim_entry; standard-library modules such as __future__, typing, collections.abc for the logic below. This chunk defines `_wrap_dim`, which wraps lower-level behavior in a transform-friendly interface. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 导入区把PyTorch 本地模块，如 torch、._dim_entry；标准库模块，如 __future__、typing、collections.abc组织在一起，供下方逻辑使用。 这一段定义了 `_wrap_dim`，其作用是把底层行为包装为更适合变换组合的接口。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 18-35
```python
    Args:
        arg: The argument to convert (Dim, int, or other)
        orig_ndim: Original number of dimensions
        allow_none: Whether to allow None values

    Returns:
        DimEntry representation of the dimension
    """
    from . import Dim

    if arg is None and allow_none:
        return DimEntry()  # None entry
    elif isinstance(arg, Dim):
        return DimEntry(arg)
    elif isinstance(arg, int):
        if arg < 0:
            pos = arg
        else:
```
- **EN**: The import section wires together PyTorch-local modules such as . for the logic below. This chunk continues `_wrap_dim` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 导入区把PyTorch 本地模块，如 .组织在一起，供下方逻辑使用。 这一段延续了 `_wrap_dim`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 36-49
```python
            pos = arg - orig_ndim
        return DimEntry(pos)
    else:
        return DimEntry()


def order(
    tensor_or_dim: torch.Tensor | Any, *dims: Any | Sequence[Any]
) -> torch.Tensor:
    """
    Reorder the dimensions of a tensor or create a tensor from a dimension.

    It allows reordering tensor dimensions using first-class dimensions and
    positional indices.
```
- **EN**: It introduces or extends dimensions, which hold the primary data model or public surface for this slice of the file. This chunk defines `order`, which reorders tensor structure or metadata according to explicit rules. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 它引入或扩展了 dimensions，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `order`，其作用是按照显式规则重排张量结构或元数据。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 51-66
```python
    Args:
        tensor_or_dim: Input tensor with first-class dimensions, or a Dim object
        *dims: Dimensions or sequences of dimensions specifying the new order

    Returns:
        Tensor with reordered dimensions

    Examples:
        >>> import torch
        >>> from functorch.dim import dims
        >>> batch, channel, height, width = dims(4)
        >>> x = torch.randn(2, 3, 4, 5)[batch, channel, height, width]
        >>> # Reorder to [height, width, batch, channel]
        >>> y = order(x, height, width, batch, channel)
    """
    from . import Dim, DimList, Tensor
```
- **EN**: The import section wires together PyTorch-local modules such as . for the logic below. It introduces or extends dimensions, which hold the primary data model or public surface for this slice of the file. This chunk continues `dimensions` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 导入区把PyTorch 本地模块，如 .组织在一起，供下方逻辑使用。 它引入或扩展了 dimensions，这些类型承载了本段的核心数据模型或对外接口。 这一段延续了 `dimensions`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 68-84
```python
    # Handle first argument - tensor or dimension
    if isinstance(tensor_or_dim, Tensor):
        # First-class tensor
        orig_levels = tensor_or_dim._levels[:]
        data = tensor_or_dim._tensor
        has_device = tensor_or_dim._has_device
    elif isinstance(tensor_or_dim, Dim):
        # Single dimension - create range tensor
        orig_levels = [DimEntry(tensor_or_dim)]
        data = tensor_or_dim._get_range()
        has_device = False
    else:
        raise ValueError("First argument must be a Tensor or Dim object")

    flat_positional_dims = []
    to_flatten = []  # List of (start_index, length) pairs for flattening
    levels = orig_levels[:]
```
- **EN**: It introduces or extends tensor, which hold the primary data model or public surface for this slice of the file. This chunk continues `tensor` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 它引入或扩展了 tensor，这些类型承载了本段的核心数据模型或对外接口。 这一段延续了 `tensor`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 86-103
```python
    orig_ndim = ndim_of_levels(levels)

    def append_dim(d: DimEntry) -> None:
        """Add a dimension to the reordering, removing it from available levels."""
        try:
            idx = levels.index(d)
        except ValueError:
            idx = None
        if idx is None:
            if d.is_positional():
                raise ValueError(
                    f"tensor has {orig_ndim} positional dimensions, but {d.position() + orig_ndim} specified, "
                    f"or it was specified twice"
                )
            else:
                raise ValueError(
                    f"tensor does not contain dim {d.dim()} or it was specified twice"
                )
```
- **EN**: This chunk defines `append_dim`, which implements one step in a functional transform, example, or package export flow. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Exception-handling structure makes cleanup or fallback behavior explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `append_dim`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 异常处理结构使清理逻辑或回退行为保持明确。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 105-122
```python
        levels[idx] = DimEntry()
        flat_positional_dims.append(d)

    n_new_positional = 0

    # Process each dimension argument
    for arg in dims:
        entry = _wrap_dim(arg, orig_ndim, False)
        if not entry.is_none():
            append_dim(entry)
            n_new_positional += 1
        elif isinstance(arg, DimList):
            # Handle DimList
            for dim in arg._dims:
                append_dim(DimEntry(dim))
                n_new_positional += 1
        else:
            # Handle sequences of dimensions for flattening
```
- **EN**: This chunk continues `append_dim` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `append_dim`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 123-139
```python
            n_new_positional += 1
            if not hasattr(arg, "__iter__"):
                raise ValueError("expected a Dim, List[Dim], or Sequence[Dim]")

            # Convert to list to get length
            seq = list(arg)
            to_flatten.append((len(flat_positional_dims), len(seq)))

            for item in seq:
                entry = _wrap_dim(item, orig_ndim, False)
                if entry.is_none():
                    raise ValueError("expected a Dim or int")
                append_dim(entry)

    # Build new level ordering
    insert_point = -1
    new_levels: list[DimEntry] = []
```
- **EN**: This chunk continues `append_dim` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `append_dim`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 141-154
```python
    # Add remaining (non-reordered) levels, finding insertion point for new dimensions
    for level in levels:
        if level.is_none():
            continue
        if level.is_positional():
            if insert_point == -1:
                insert_point = len(new_levels)
                new_levels.extend(flat_positional_dims)
        new_levels.append(level)

    # If no positional dimensions found, append new dims at the end
    if insert_point == -1:
        insert_point = len(new_levels)
        new_levels.extend(flat_positional_dims)
```
- **EN**: This chunk continues `append_dim` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `append_dim`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 156-169
```python
    # Match tensor to new level structure
    if data is None:
        raise AssertionError("Cannot reorder None tensor")
    ndata = _match_levels(data, orig_levels, new_levels)

    # Handle dimension flattening if requested
    if to_flatten:
        # Now build the reshape target
        view_shape = []
        sizes = ndata.size()

        # Add dimensions before the reordered ones
        for i in range(insert_point):
            view_shape.append(sizes[i])
```
- **EN**: This chunk continues `append_dim` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `append_dim`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 171-184
```python
        # Process flattening groups
        i = 0
        for start_idx, length in to_flatten:
            # Add individual dims before this flattening group
            while i < start_idx:
                view_shape.append(sizes[insert_point + i])
                i += 1

            # Flatten the group
            new_size = 1
            for j in range(length):
                new_size *= sizes[insert_point + i + j]
            view_shape.append(new_size)
            i += length
```
- **EN**: This chunk continues `append_dim` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `append_dim`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 186-201
```python
        # Add remaining individual dims
        while i < len(flat_positional_dims):
            view_shape.append(sizes[insert_point + i])
            i += 1

        # Add dimensions after the reordered ones
        for i in range(insert_point + len(flat_positional_dims), len(levels)):
            view_shape.append(sizes[i])

        # Update levels by removing flattened dimensions
        n_to_remove = len(flat_positional_dims) - n_new_positional
        if n_to_remove > 0:
            # Remove flattened levels
            new_levels = (
                new_levels[:insert_point] + new_levels[insert_point + n_to_remove :]
            )
```
- **EN**: This chunk continues `append_dim` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `append_dim`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 203-215
```python
        ndata = ndata.reshape(view_shape)

    # Renumber positional dimensions (negative indexing from the right)
    seen = 0
    for i in range(len(new_levels) - 1, -1, -1):
        if new_levels[i].is_positional() or (
            i >= insert_point and i < insert_point + n_new_positional
        ):
            seen -= 1
            new_levels[i] = DimEntry(seen)

    result = Tensor.from_positional(ndata, new_levels, has_device)
    return result  # type: ignore[return-value]
```
- **EN**: This chunk continues `append_dim` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `append_dim`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

## Key Concepts / 关键概念

- **Functional tensor transforms**
  - EN: Applies composable transforms such as batching, gradients, and tracing.
  - CN: 应用可组合的变换，例如 batching、梯度和 tracing。
- **_wrap_dim**
  - EN: `_wrap_dim` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `_wrap_dim` 是本文件声明、导出或驱动的显著符号之一。
- **order**
  - EN: `order` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `order` 是本文件声明、导出或驱动的显著符号之一。
## Dependencies / 依赖关系

- **Internal modules / 内部模块**: `torch`, `._dim_entry`, `.`
- **Standard library / 标准库**: `__future__`, `typing`, `collections.abc`
- **Primary symbols / 核心符号**: `_wrap_dim`, `order`, `append_dim`
