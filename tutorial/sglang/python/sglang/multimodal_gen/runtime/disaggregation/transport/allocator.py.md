# allocator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/disaggregation/transport/allocator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the disaggregation runtime layer. It centers on `Block`, and `BuddyAllocator`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Buddy-system memory allocator for TransferTensorBuffer. / 该文件属于解耦运行时层。它围绕 `Block` 和 `BuddyAllocator` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-9: module setup and imports / 模块初始化与导入
```python
"""Buddy-system memory allocator for TransferTensorBuffer."""

from __future__ import annotations

import logging
import threading
from dataclasses import dataclass
from functools import lru_cache
```
**EN:** This block establishes the module context and imports `__future__`, `logging`, `threading`, `dataclasses`, and `functools`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`logging`、`threading`、`dataclasses` 和 `functools`。这些依赖为后续实现提供所需符号。

### Lines 11-11: supporting statements / 辅助语句
```python
logger = logging.getLogger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `logging.getLogger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `logging.getLogger` 协同工作。

### Lines 15-15: `Block` class overview / `Block` 类概览
```python
class Block:
```
**EN:** This block defines class `Block`. It encapsulates block behavior.
**CN:** 该代码块定义了类 `Block`。 它用于封装 block 相关行为。

### Lines 16-19: supporting statements / 辅助语句
```python
    offset: int  # byte offset from pool start
    size: int
    allocated: bool = False
    request_id: str | None = None
```
**EN:** This block gathers supporting statements inside `Block`. It updates names such as `offset`, `size`, `allocated`, and `request_id`.
**CN:** 该代码块汇集了位于 `Block` 内部的辅助语句。 它会更新 `offset`、`size`、`allocated` 和 `request_id` 等名称。

### Lines 22-24: `BuddyAllocator` class overview / `BuddyAllocator` 类概览
```python
class BuddyAllocator:
    """Power-of-2 buddy-system allocator for pinned memory."""
```
**EN:** This block defines class `BuddyAllocator`. Power-of-2 buddy-system allocator for pinned memory.
**CN:** 该代码块定义了类 `BuddyAllocator`。 它用于封装 buddy allocator 相关行为。

### Lines 25-46: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, pool_size: int, min_block_size: int = 1 << 20):
        if min_block_size <= 0 or (min_block_size & (min_block_size - 1)) != 0:
            raise ValueError(
                f"min_block_size must be a power of 2, got {min_block_size}"
            )

        self._min_block_size = min_block_size
        self._pool_size = self._next_power_of_2(max(pool_size, min_block_size))
        self._lock = threading.Lock()

        # Free lists indexed by order: order 0 = min_block_size, order 1 = 2*min_block_size, ...
        self._max_order = self._size_to_order(self._pool_size)
        self._free_lists: list[list[int]] = [[] for _ in range(self._max_order + 1)]

        self._blocks: dict[int, Block] = {}

        root = Block(offset=0, size=self._pool_size)
        self._blocks[0] = root
        self._free_lists[self._max_order].append(0)

        self._allocated_bytes = 0
        self._num_allocations = 0
```
**EN:** This block defines method `__init__` on `BuddyAllocator`. It initializes the instance state. Key calls include `self._next_power_of_2`, `threading.Lock`, `self._size_to_order`, `Block`, and `self._free_lists.append`. The implementation branches on conditions. Parameters such as `pool_size`, and `min_block_size` drive the behavior in this section.
**CN:** 该代码块定义了 `BuddyAllocator` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `self._next_power_of_2`、`threading.Lock`、`self._size_to_order`、`Block` 和 `self._free_lists.append`。 实现中包含条件分支。 本段逻辑主要由 `pool_size` 和 `min_block_size` 等参数驱动。

### Lines 48-50: `pool_size` implementation / `pool_size` 实现
```python
    @property
    def pool_size(self) -> int:
        return self._pool_size
```
**EN:** This block defines method `pool_size` on `BuddyAllocator`. It handles pool size logic.
**CN:** 该代码块定义了 `BuddyAllocator` 的方法 `pool_size`。 它用于处理 pool size 相关逻辑。

### Lines 52-67: `allocate` implementation / `allocate` 实现
```python
    def allocate(self, size: int, request_id: str | None = None) -> int | None:
        """Allocate a block of at least `size` bytes. Returns offset or None."""
        if size <= 0:
            raise ValueError(f"Allocation size must be positive, got {size}")

        alloc_size = max(self._next_power_of_2(size), self._min_block_size)
        target_order = self._size_to_order(alloc_size)

        if target_order > self._max_order:
            logger.warning(
                "Requested size %d exceeds pool size %d", size, self._pool_size
            )
            return None

        with self._lock:
            return self._allocate_locked(target_order, request_id)
```
**EN:** This block defines method `allocate` on `BuddyAllocator`. Allocate a block of at least `size` bytes. Returns offset or None. Key calls include `max`, `self._size_to_order`, `ValueError`, `self._next_power_of_2`, and `logger.warning`. The implementation branches on conditions, uses context-managed resources. Parameters such as `size`, and `request_id` drive the behavior in this section.
**CN:** 该代码块定义了 `BuddyAllocator` 的方法 `allocate`。 它用于处理 allocate 相关逻辑。 关键调用包括 `max`、`self._size_to_order`、`ValueError`、`self._next_power_of_2` 和 `logger.warning`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `size` 和 `request_id` 等参数驱动。

### Lines 69-72: `free` implementation / `free` 实现
```python
    def free(self, offset: int) -> bool:
        """Free the block at the given offset and coalesce with buddy if possible."""
        with self._lock:
            return self._free_locked(offset)
```
**EN:** This block defines method `free` on `BuddyAllocator`. Free the block at the given offset and coalesce with buddy if possible. Key calls include `self._free_locked`. The implementation uses context-managed resources. Parameters such as `offset` drive the behavior in this section.
**CN:** 该代码块定义了 `BuddyAllocator` 的方法 `free`。 它用于处理 free 相关逻辑。 关键调用包括 `self._free_locked`。 实现中使用上下文管理资源。 本段逻辑主要由 `offset` 等参数驱动。

### Lines 74-76: `get_block_info` implementation / `get_block_info` 实现
```python
    def get_block_info(self, offset: int) -> Block | None:
        with self._lock:
            return self._blocks.get(offset)
```
**EN:** This block defines method `get_block_info` on `BuddyAllocator`. It retrieves block info. Key calls include `self._blocks.get`. The implementation uses context-managed resources. Parameters such as `offset` drive the behavior in this section.
**CN:** 该代码块定义了 `BuddyAllocator` 的方法 `get_block_info`。 它用于获取block info。 关键调用包括 `self._blocks.get`。 实现中使用上下文管理资源。 本段逻辑主要由 `offset` 等参数驱动。

### Lines 78-94: `get_stats` implementation / `get_stats` 实现
```python
    def get_stats(self) -> dict:
        with self._lock:
            free_blocks_by_order = {}
            for order, offsets in enumerate(self._free_lists):
                if offsets:
                    block_size = self._min_block_size << order
                    free_blocks_by_order[block_size] = len(offsets)

            return {
                "pool_size": self._pool_size,
                "min_block_size": self._min_block_size,
                "allocated_bytes": self._allocated_bytes,
                "free_bytes": self._pool_size - self._allocated_bytes,
                "num_allocations": self._num_allocations,
                "num_blocks": len(self._blocks),
                "free_blocks_by_size": free_blocks_by_order,
            }
```
**EN:** This block defines method `get_stats` on `BuddyAllocator`. It retrieves stats. Key calls include `enumerate`, and `len`. The implementation branches on conditions, iterates over collections or steps, uses context-managed resources.
**CN:** 该代码块定义了 `BuddyAllocator` 的方法 `get_stats`。 它用于获取stats。 关键调用包括 `enumerate` 和 `len`。 实现中包含条件分支，会遍历集合或步骤，使用上下文管理资源。

### Lines 96-108: `count_free_slots` implementation / `count_free_slots` 实现
```python
    def count_free_slots(self, slot_size: int) -> int:
        """Count how many allocations of the given size can fit."""
        if slot_size <= 0:
            return 0
        alloc_size = max(self._next_power_of_2(slot_size), self._min_block_size)

        with self._lock:
            count = 0
            for order in range(self._size_to_order(alloc_size), self._max_order + 1):
                for _ in self._free_lists[order]:
                    block_size = self._min_block_size << order
                    count += block_size // alloc_size
            return count
```
**EN:** This block defines method `count_free_slots` on `BuddyAllocator`. Count how many allocations of the given size can fit. Key calls include `max`, `self._next_power_of_2`, `range`, and `self._size_to_order`. The implementation branches on conditions, iterates over collections or steps, uses context-managed resources. Parameters such as `slot_size` drive the behavior in this section.
**CN:** 该代码块定义了 `BuddyAllocator` 的方法 `count_free_slots`。 它用于处理 count free slots 相关逻辑。 关键调用包括 `max`、`self._next_power_of_2`、`range` 和 `self._size_to_order`。 实现中包含条件分支，会遍历集合或步骤，使用上下文管理资源。 本段逻辑主要由 `slot_size` 等参数驱动。

### Lines 112-142: `_allocate_locked` implementation / `_allocate_locked` 实现
```python
    def _allocate_locked(self, target_order: int, request_id: str | None) -> int | None:
        found_order = -1
        for order in range(target_order, self._max_order + 1):
            if self._free_lists[order]:
                found_order = order
                break

        if found_order < 0:
            return None

        offset = self._free_lists[found_order].pop(0)
        block = self._blocks[offset]

        # Split down to target_order
        while found_order > target_order:
            found_order -= 1
            buddy_size = self._min_block_size << found_order
            buddy_offset = offset + buddy_size

            buddy = Block(offset=buddy_offset, size=buddy_size)
            self._blocks[buddy_offset] = buddy
            self._free_lists[found_order].append(buddy_offset)

            block.size = buddy_size

        block.allocated = True
        block.request_id = request_id
        self._allocated_bytes += block.size
        self._num_allocations += 1

        return offset
```
**EN:** This block defines method `_allocate_locked` on `BuddyAllocator`. It handles allocate locked logic. Key calls include `range`, `self._free_lists.pop`, `Block`, and `self._free_lists.append`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `target_order`, and `request_id` drive the behavior in this section.
**CN:** 该代码块定义了 `BuddyAllocator` 的方法 `_allocate_locked`。 它用于处理 allocate locked 相关逻辑。 关键调用包括 `range`、`self._free_lists.pop`、`Block` 和 `self._free_lists.append`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `target_order` 和 `request_id` 等参数驱动。

### Lines 144-155: `_free_locked` implementation / `_free_locked` 实现
```python
    def _free_locked(self, offset: int) -> bool:
        block = self._blocks.get(offset)
        if block is None or not block.allocated:
            return False

        block.allocated = False
        block.request_id = None
        self._allocated_bytes -= block.size
        self._num_allocations -= 1

        self._coalesce(block)
        return True
```
**EN:** This block defines method `_free_locked` on `BuddyAllocator`. It handles free locked logic. Key calls include `self._blocks.get`, and `self._coalesce`. The implementation branches on conditions. Parameters such as `offset` drive the behavior in this section.
**CN:** 该代码块定义了 `BuddyAllocator` 的方法 `_free_locked`。 它用于处理 free locked 相关逻辑。 关键调用包括 `self._blocks.get` 和 `self._coalesce`。 实现中包含条件分支。 本段逻辑主要由 `offset` 等参数驱动。

### Lines 157-178: `_coalesce` implementation / `_coalesce` 实现
```python
    def _coalesce(self, block: Block) -> None:
        """Recursively merge with buddy if both are free."""
        while block.size < self._pool_size:
            buddy_offset = block.offset ^ block.size
            buddy = self._blocks.get(buddy_offset)

            if buddy is None or buddy.allocated or buddy.size != block.size:
                break

            order = self._size_to_order(buddy.size)
            self._free_lists[order].remove(buddy_offset)

            if buddy_offset < block.offset:
                del self._blocks[block.offset]
                buddy.size *= 2
                block = buddy
            else:
                del self._blocks[buddy_offset]
                block.size *= 2

        order = self._size_to_order(block.size)
        self._free_lists[order].append(block.offset)
```
**EN:** This block defines method `_coalesce` on `BuddyAllocator`. Recursively merge with buddy if both are free. Key calls include `self._size_to_order`, `self._free_lists.append`, `self._blocks.get`, and `self._free_lists.remove`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `block` drive the behavior in this section.
**CN:** 该代码块定义了 `BuddyAllocator` 的方法 `_coalesce`。 它用于处理 coalesce 相关逻辑。 关键调用包括 `self._size_to_order`、`self._free_lists.append`、`self._blocks.get` 和 `self._free_lists.remove`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `block` 等参数驱动。

### Lines 180-186: `_size_to_order` implementation / `_size_to_order` 实现
```python
    def _size_to_order(self, size: int) -> int:
        order = 0
        s = self._min_block_size
        while s < size:
            s <<= 1
            order += 1
        return order
```
**EN:** This block defines method `_size_to_order` on `BuddyAllocator`. It handles size to order logic. The implementation iterates over collections or steps. Parameters such as `size` drive the behavior in this section.
**CN:** 该代码块定义了 `BuddyAllocator` 的方法 `_size_to_order`。 它用于处理 size to order 相关逻辑。 实现中会遍历集合或步骤。 本段逻辑主要由 `size` 等参数驱动。

### Lines 188-200: `_next_power_of_2` implementation / `_next_power_of_2` 实现
```python
    @staticmethod
    @lru_cache(maxsize=256)
    def _next_power_of_2(n: int) -> int:
        if n <= 0:
            return 1
        n -= 1
        n |= n >> 1
        n |= n >> 2
        n |= n >> 4
        n |= n >> 8
        n |= n >> 16
        n |= n >> 32
        return n + 1
```
**EN:** This block defines method `_next_power_of_2` on `BuddyAllocator`. It handles next power of 2 logic. Key calls include `lru_cache`. The implementation branches on conditions. Parameters such as `n` drive the behavior in this section.
**CN:** 该代码块定义了 `BuddyAllocator` 的方法 `_next_power_of_2`。 它用于处理 next power of 2 相关逻辑。 关键调用包括 `lru_cache`。 实现中包含条件分支。 本段逻辑主要由 `n` 等参数驱动。

## Key Concepts / 关键概念
- `Block`: Primary class that encapsulates block behavior. / 核心类，用于封装 block 相关行为。
- `BuddyAllocator`: Power-of-2 buddy-system allocator for pinned memory. / 核心类，用于封装 buddy allocator 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `threading`, `dataclasses`, `functools`

- **Total lines / 总行数**: 200
