# partitioning.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/partitioning.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates partitioning with focused assertions and fixtures. Key symbols include `PartitionItem`, `partition_items_by_lpt`. / 该测试模块通过有针对性的断言与夹具，验证 partitioning 的实现。 关键符号包括 `PartitionItem`, `partition_items_by_lpt`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: Imports and module setup / 导入与模块初始化
```python
from __future__ import annotations

from dataclasses import dataclass
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 6-11: Class `PartitionItem` / 类 `PartitionItem`
```python
@dataclass(frozen=True)
class PartitionItem:
    kind: str
    item_id: str
    est_time: float
    used_fallback_estimate: bool = False
```
**EN:** This class models `PartitionItem`.
**CN:** 该类实现 `PartitionItem`。

### Lines 14-32: Function `partition_items_by_lpt` / 函数 `partition_items_by_lpt`
```python
def partition_items_by_lpt(
    items: list[PartitionItem], num_partitions: int
) -> list[list[PartitionItem]]:
    if not items or num_partitions <= 0:
        return []

    sorted_items = sorted(
        items,
        key=lambda item: (-item.est_time, item.kind, item.item_id),
    )
    partitions: list[list[PartitionItem]] = [[] for _ in range(num_partitions)]
    partition_sums = [0.0] * num_partitions

    for item in sorted_items:
        min_idx = partition_sums.index(min(partition_sums))
        partitions[min_idx].append(item)
        partition_sums[min_idx] += item.est_time

    return partitions
```
**EN:** This function drives `partition_items_by_lpt` with inputs such as `items`, `num_partitions`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `partition_items_by_lpt`，主要处理 `items`, `num_partitions` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- Automated verification / 自动化验证
- Symbol `PartitionItem` anchors the module API / 符号 `PartitionItem` 构成该模块的核心 API
- Symbol `partition_items_by_lpt` anchors the module API / 符号 `partition_items_by_lpt` 构成该模块的核心 API

## Dependencies / 依赖关系
- **External / 外部**: `__future__`
- **Stdlib / 标准库**: `dataclasses`
