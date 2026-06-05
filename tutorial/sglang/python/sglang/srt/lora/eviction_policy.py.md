# eviction_policy.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/lora/eviction_policy.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Eviction policies for LoRA adapter memory management. / 该文件实现了核心 LoRA 运行时逻辑，例如适配器元数据、内存管理、层封装或整体编排。它属于 SGLang 支持动态适配器服务能力的基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-27: Module header, imports, and shared constants
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
Eviction policies for LoRA adapter memory management.
"""

import logging
import time
from abc import ABC, abstractmethod
from collections import OrderedDict
from typing import Optional, Set

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 28-30: EvictionPolicy class declaration
```python
class EvictionPolicy(ABC):
    """Abstract base class for LoRA adapter eviction policies."""
```
**EN:** This block declares the `EvictionPolicy` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `EvictionPolicy` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 31-34: EvictionPolicy.mark_used method
```python
    @abstractmethod
    def mark_used(self, uid: Optional[str]) -> None:
        """Marks an adapter as used."""
        pass
```
**EN:** This block uses `EvictionPolicy.mark_used` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `EvictionPolicy.mark_used` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 36-39: EvictionPolicy.select_victim method
```python
    @abstractmethod
    def select_victim(self, candidates: Set[Optional[str]]) -> Optional[str]:
        """Selects an adapter to evict from candidates."""
        pass
```
**EN:** This block uses `EvictionPolicy.select_victim` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `EvictionPolicy.select_victim` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 41-44: EvictionPolicy.remove method
```python
    @abstractmethod
    def remove(self, uid: Optional[str]) -> None:
        """Removes an adapter from the policy's tracking."""
        pass
```
**EN:** This block uses `EvictionPolicy.remove` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `EvictionPolicy.remove` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 47-49: LRUEvictionPolicy class declaration
```python
class LRUEvictionPolicy(EvictionPolicy):
    """LRU eviction policy - evicts the least recently used adapter."""
```
**EN:** This block declares the `LRUEvictionPolicy` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `LRUEvictionPolicy` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 50-53: LRUEvictionPolicy initializer
```python
    def __init__(self):
        self.access_order = OrderedDict()  # key=uid, value=last_access_time
        self.total_accesses = 0
        self.eviction_count = 0
```
**EN:** This block initializes the `LRUEvictionPolicy` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `LRUEvictionPolicy` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 55-62: LRUEvictionPolicy.mark_used method
```python
    def mark_used(self, uid: Optional[str]) -> None:
        if uid is not None:
            current_time = time.monotonic()
            # Remove and re-add to move to end (most recent)
            self.access_order.pop(uid, None)
            self.access_order[uid] = current_time
            self.total_accesses += 1
            logger.debug(f"LoRA {uid} marked as used at {current_time}")
```
**EN:** This block uses `LRUEvictionPolicy.mark_used` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LRUEvictionPolicy.mark_used` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 64-82: LRUEvictionPolicy.select_victim method
```python
    def select_victim(self, candidates: Set[Optional[str]]) -> Optional[str]:
        """Select the least recently used adapter from candidates."""
        # Iterate through access_order (oldest first) to find LRU victim
        for uid in list(self.access_order.keys()):
            if uid in candidates:
                logger.debug(f"Selected LoRA {uid} for eviction (LRU)")
                self.eviction_count += 1
                return uid

        # If no tracked UID found in candidates, check if None is available
        # This happens when the batch consists entirely of LoRA requests
        # and None (base model) is the only eviction candidate
        if None in candidates:
            logger.debug("Selected None (base model) for eviction")
            self.eviction_count += 1
            return None

        # Should never reach here if candidates is non-empty
        assert False, f"Failed to select LRU victim from candidates: {candidates}"
```
**EN:** This block uses `LRUEvictionPolicy.select_victim` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LRUEvictionPolicy.select_victim` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 84-87: LRUEvictionPolicy.remove method
```python
    def remove(self, uid: Optional[str]) -> None:
        if uid is not None:
            self.access_order.pop(uid, None)
            logger.debug(f"Removed LoRA {uid} from LRU tracking")
```
**EN:** This block uses `LRUEvictionPolicy.remove` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LRUEvictionPolicy.remove` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 90-92: FIFOEvictionPolicy class declaration
```python
class FIFOEvictionPolicy(EvictionPolicy):
    """FIFO eviction policy - for backward compatibility."""
```
**EN:** This block declares the `FIFOEvictionPolicy` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `FIFOEvictionPolicy` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 93-97: FIFOEvictionPolicy initializer
```python
    def __init__(self):
        self.insertion_order = (
            OrderedDict()
        )  # key=uid, OrderedDict maintains insertion order
        self.eviction_count = 0
```
**EN:** This block initializes the `FIFOEvictionPolicy` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `FIFOEvictionPolicy` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 99-104: FIFOEvictionPolicy.mark_used method
```python
    def mark_used(self, uid: Optional[str]) -> None:
        """For FIFO, we only track insertion order (not access time)."""
        if uid is not None and uid not in self.insertion_order:
            self.insertion_order[uid] = (
                True  # Value unused, OrderedDict tracks insertion order
            )
```
**EN:** This block uses `FIFOEvictionPolicy.mark_used` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FIFOEvictionPolicy.mark_used` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 106-124: FIFOEvictionPolicy.select_victim method
```python
    def select_victim(self, candidates: Set[Optional[str]]) -> Optional[str]:
        """Select the first inserted adapter from candidates."""
        # Iterate through insertion_order (oldest first) to find FIFO victim
        for uid in list(self.insertion_order.keys()):
            if uid in candidates:
                logger.debug(f"Selected LoRA {uid} for eviction (FIFO)")
                self.eviction_count += 1
                return uid

        # If no tracked UID found in candidates, check if None is available
        # This happens when the batch consists entirely of LoRA requests
        # and None (base model) is the only eviction candidate
        if None in candidates:
            logger.debug("Selected None (base model) for eviction")
            self.eviction_count += 1
            return None

        # Should never reach here if candidates is non-empty
        assert False, f"Failed to select FIFO victim from candidates: {candidates}"
```
**EN:** This block uses `FIFOEvictionPolicy.select_victim` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FIFOEvictionPolicy.select_victim` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 126-128: FIFOEvictionPolicy.remove method
```python
    def remove(self, uid: Optional[str]) -> None:
        if uid is not None:
            self.insertion_order.pop(uid, None)
```
**EN:** This block uses `FIFOEvictionPolicy.remove` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FIFOEvictionPolicy.remove` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 131-139: get eviction policy function
```python
def get_eviction_policy(policy_name: str) -> EvictionPolicy:
    """Factory function to create eviction policy instances."""
    policies = {
        "fifo": FIFOEvictionPolicy,
        "lru": LRUEvictionPolicy,
    }
    if policy_name not in policies:
        raise ValueError(f"Unknown eviction policy: {policy_name}")
    return policies[policy_name]()
```
**EN:** This block uses `get_eviction_policy` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `get_eviction_policy` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- LoRA adapter execution / LoRA 适配器执行

## Dependencies / 依赖关系
### Internal / 内部
- None / 无
### External / 外部
- `abc` (stdlib)
- `collections` (stdlib)
- `logging` (stdlib)
- `time` (stdlib)
- `typing` (stdlib)
