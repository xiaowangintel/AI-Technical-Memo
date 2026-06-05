# evict_policy.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/evict_policy.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the evict policy logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的evict策略相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Imports and setup / 导入与初始化
```python
from __future__ import annotations

from abc import ABC, abstractmethod
from typing import TYPE_CHECKING, Tuple, Union
```
**EN:** Imports `__future__`, `abc`, `typing` and other helpers used by the surrounding scope.
**CN:** 导入 `__future__`, `abc`, `typing` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 6-7: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    from sglang.srt.mem_cache.radix_cache import TreeNode
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 10-10: EvictionStrategy declaration / EvictionStrategy 声明
```python
class EvictionStrategy(ABC):
```
**EN:** Declares the `EvictionStrategy` class and connects it to `ABC`.
**CN:** 声明 `EvictionStrategy` 类，并将其关联到 `ABC`。

### Lines 11-13: get_priority implementation / get_priority 实现
```python
    @abstractmethod
    def get_priority(self, node: "TreeNode") -> Union[float, Tuple]:
        pass
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `EvictionStrategy`.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `EvictionStrategy`。

### Lines 16-16: LRUStrategy declaration / LRUStrategy 声明
```python
class LRUStrategy(EvictionStrategy):
```
**EN:** Declares the `LRUStrategy` class and connects it to `EvictionStrategy`.
**CN:** 声明 `LRUStrategy` 类，并将其关联到 `EvictionStrategy`。

### Lines 17-18: get_priority implementation / get_priority 实现
```python
    def get_priority(self, node: "TreeNode") -> float:
        return node.last_access_time
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `LRUStrategy`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `LRUStrategy`。它会向调用方返回计算结果。

### Lines 21-21: LFUStrategy declaration / LFUStrategy 声明
```python
class LFUStrategy(EvictionStrategy):
```
**EN:** Declares the `LFUStrategy` class and connects it to `EvictionStrategy`.
**CN:** 声明 `LFUStrategy` 类，并将其关联到 `EvictionStrategy`。

### Lines 22-23: get_priority implementation / get_priority 实现
```python
    def get_priority(self, node: "TreeNode") -> Tuple[int, float]:
        return (node.hit_count, node.last_access_time)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `LFUStrategy`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `LFUStrategy`。它会向调用方返回计算结果。

### Lines 26-26: FIFOStrategy declaration / FIFOStrategy 声明
```python
class FIFOStrategy(EvictionStrategy):
```
**EN:** Declares the `FIFOStrategy` class and connects it to `EvictionStrategy`.
**CN:** 声明 `FIFOStrategy` 类，并将其关联到 `EvictionStrategy`。

### Lines 27-28: get_priority implementation / get_priority 实现
```python
    def get_priority(self, node: "TreeNode") -> float:
        return node.creation_time
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `FIFOStrategy`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `FIFOStrategy`。它会向调用方返回计算结果。

### Lines 31-31: MRUStrategy declaration / MRUStrategy 声明
```python
class MRUStrategy(EvictionStrategy):
```
**EN:** Declares the `MRUStrategy` class and connects it to `EvictionStrategy`.
**CN:** 声明 `MRUStrategy` 类，并将其关联到 `EvictionStrategy`。

### Lines 32-33: get_priority implementation / get_priority 实现
```python
    def get_priority(self, node: "TreeNode") -> float:
        return -node.last_access_time
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `MRUStrategy`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MRUStrategy`。它会向调用方返回计算结果。

### Lines 36-36: FILOStrategy declaration / FILOStrategy 声明
```python
class FILOStrategy(EvictionStrategy):
```
**EN:** Declares the `FILOStrategy` class and connects it to `EvictionStrategy`.
**CN:** 声明 `FILOStrategy` 类，并将其关联到 `EvictionStrategy`。

### Lines 37-38: get_priority implementation / get_priority 实现
```python
    def get_priority(self, node: "TreeNode") -> float:
        return -node.creation_time
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `FILOStrategy`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `FILOStrategy`。它会向调用方返回计算结果。

### Lines 41-43: PriorityStrategy declaration / PriorityStrategy 声明
```python
class PriorityStrategy(EvictionStrategy):
    """Priority-aware eviction: lower priority values evicted first, then LRU within same priority."""

```
**EN:** Priority-aware eviction: lower priority values evicted first, then LRU within same priority. Declares the `PriorityStrategy` class and connects it to `EvictionStrategy`.
**CN:** 声明 `PriorityStrategy` 类，并将其关联到 `EvictionStrategy`。

### Lines 44-46: get_priority implementation / get_priority 实现
```python
    def get_priority(self, node: "TreeNode") -> Tuple[int, float]:
        # Return (priority, last_access_time) so lower priority nodes are evicted first
        return (node.priority, node.last_access_time)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `PriorityStrategy`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `PriorityStrategy`。它会向调用方返回计算结果。

### Lines 49-49: SLRUStrategy declaration / SLRUStrategy 声明
```python
class SLRUStrategy(EvictionStrategy):
```
**EN:** Declares the `SLRUStrategy` class and connects it to `EvictionStrategy`.
**CN:** 声明 `SLRUStrategy` 类，并将其关联到 `EvictionStrategy`。

### Lines 50-51: __init__ implementation / __init__ 实现
```python
    def __init__(self, protected_threshold: int = 2):
        self.protected_threshold = protected_threshold
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `SLRUStrategy`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `SLRUStrategy`。

### Lines 53-65: get_priority implementation / get_priority 实现
```python
    def get_priority(self, node: "TreeNode") -> Tuple[int, float]:
        # Priority Logic:
        # Smaller value = Evicted earlier.
        #
        # Segment 0 (Probationary): hit_count < threshold
        # Segment 1 (Protected): hit_count >= threshold
        #
        # Tuple comparison: (segment, last_access_time)
        # Nodes in segment 0 will always be evicted before segment 1.
        # Inside the same segment, older nodes (smaller time) are evicted first.

        is_protected = 1 if node.hit_count >= self.protected_threshold else 0
        return (is_protected, node.last_access_time)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `SLRUStrategy`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `SLRUStrategy`。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`EvictionStrategy`**: Defines the `EvictionStrategy` type and its core responsibilities. / 定义 `EvictionStrategy` 类型及其核心职责。
- **`LRUStrategy`**: Defines the `LRUStrategy` type and its core responsibilities. / 定义 `LRUStrategy` 类型及其核心职责。
- **`LFUStrategy`**: Defines the `LFUStrategy` type and its core responsibilities. / 定义 `LFUStrategy` 类型及其核心职责。
- **`FIFOStrategy`**: Defines the `FIFOStrategy` type and its core responsibilities. / 定义 `FIFOStrategy` 类型及其核心职责。
- **`MRUStrategy`**: Defines the `MRUStrategy` type and its core responsibilities. / 定义 `MRUStrategy` 类型及其核心职责。
- **`FILOStrategy`**: Defines the `FILOStrategy` type and its core responsibilities. / 定义 `FILOStrategy` 类型及其核心职责。
- **`PriorityStrategy`**: Defines the `PriorityStrategy` type and its core responsibilities. / 定义 `PriorityStrategy` 类型及其核心职责。
- **`SLRUStrategy`**: Defines the `SLRUStrategy` type and its core responsibilities. / 定义 `SLRUStrategy` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `abc`, `typing`
- **Internal / 内部**: `sglang.srt.mem_cache.radix_cache`
