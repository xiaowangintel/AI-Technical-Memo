# dp_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/dp_utils.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on distributed-parallel debug helpers. It mainly provides reusable helpers that reduce duplication across nearby modules. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于分布式并行调试辅助逻辑。它主要用于提供可复用的辅助函数，减少相邻模块中的重复代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Document the module intent / 说明模块意图
```python
"""DP filtering: keep only the non-empty dp_rank items."""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 3-11: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from collections import defaultdict
from typing import Optional

import torch

from sglang.srt.debug_utils.comparator.dims_spec import ParallelAxis
from sglang.srt.debug_utils.dump_loader import ValueWithMeta
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 13-13: Declare module-level symbols such as `_PARALLEL_INFO_KEYS` / 声明模块级符号，例如 `_PARALLEL_INFO_KEYS`
```python
_PARALLEL_INFO_KEYS = ("sglang_parallel_info", "megatron_parallel_info")
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 16-65: Implement function `filter_to_non_empty_dp_rank` / 实现函数 `filter_to_non_empty_dp_rank`
```python
def filter_to_non_empty_dp_rank(
    items: list[ValueWithMeta],
    *,
    dp_axis: ParallelAxis,
) -> list[ValueWithMeta]:
    """Filter items to the single non-empty dp_rank.

    - dp_size <= 1: return items unchanged.
    - dp_size > 1: group by dp_rank, assert exactly one group has non-empty
      tensors, return that group.

    *dp_axis* determines which rank/size fields to look up (e.g.
    ``ParallelAxis.MOE_DP`` → ``moe_dp_rank`` / ``moe_dp_size``).
    If the fields are absent the filter is a noop (items returned unchanged).
    """
    if not items:
        return items

    dp_info: Optional[tuple[int, int]] = _extract_dp_info(
        items[0].meta, dp_axis=dp_axis
    )
    if dp_info is None:
        return items

    _dp_rank, dp_size = dp_info
    if dp_size <= 1:
        return items

    has_any_tensor: bool = any(isinstance(item.value, torch.Tensor) for item in items)
    if not has_any_tensor:
        return items

    groups: dict[int, list[ValueWithMeta]] = defaultdict(list)
    for item in items:
        item_dp: Optional[tuple[int, int]] = _extract_dp_info(
            item.meta, dp_axis=dp_axis
        )
        rank: int = item_dp[0] if item_dp is not None else 0
        groups[rank].append(item)

    non_empty_ranks: list[int] = [
        rank for rank, group in groups.items() if _group_has_data(group)
    ]

    assert len(non_empty_ranks) == 1, (
        f"Expected exactly 1 non-empty dp_rank, got {len(non_empty_ranks)}: "
        f"ranks={non_empty_ranks}"
    )

    return groups[non_empty_ranks[0]]
```
**EN:** Function `filter_to_non_empty_dp_rank` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `filter_to_non_empty_dp_rank` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 68-92: Implement helper `_extract_dp_info` / 实现辅助函数 `_extract_dp_info`
```python
def _extract_dp_info(
    meta: dict,
    *,
    dp_axis: ParallelAxis,
) -> Optional[tuple[int, int]]:
    """Extract (dp_rank, dp_size) from meta's parallel_info block.

    *dp_axis* determines which fields to look up: e.g.
    ``ParallelAxis.DP`` → ``dp_rank``/``dp_size``,
    ``ParallelAxis.MOE_DP`` → ``moe_dp_rank``/``moe_dp_size``.
    """
    rank_field: str = f"{dp_axis.value}_rank"
    size_field: str = f"{dp_axis.value}_size"

    for key in _PARALLEL_INFO_KEYS:
        info = meta.get(key)
        if not isinstance(info, dict) or not info:
            continue

        dp_rank = info.get(rank_field)
        dp_size = info.get(size_field)
        if dp_rank is not None and dp_size is not None:
            return (int(dp_rank), int(dp_size))

    return None
```
**EN:** Function `_extract_dp_info` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_extract_dp_info` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 95-100: Implement helper `_group_has_data` / 实现辅助函数 `_group_has_data`
```python
def _group_has_data(group: list[ValueWithMeta]) -> bool:
    """Check if any tensor in the group is non-empty (numel > 0)."""
    return any(
        isinstance(item.value, torch.Tensor) and item.value.numel() > 0
        for item in group
    )
```
**EN:** Function `_group_has_data` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_group_has_data` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `filter_to_non_empty_dp_rank`, `_extract_dp_info`, `_group_has_data`
- **Module role / 模块角色**: Distributed-parallel debug helpers / 分布式并行调试辅助逻辑
- **Implementation focus / 实现重点**: Provides reusable helpers that reduce duplication across nearby modules / 提供可复用的辅助函数，减少相邻模块中的重复代码

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `collections`, `typing`
- **Third-party / 第三方**: `torch`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.srt.debug_utils.dump_loader`
