# kv_cache_metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/core/kv_cache_metrics.py`
- **Repository**: vllm-project/vllm
- **Purpose**: KV cache metrics tracking. / 该模块位于 `core` 子系统，主要围绕 `BlockMetricsState`, `KVCacheMetricsCollector` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""KV cache metrics tracking."""

import random
import time
from collections import deque
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from vllm.v1.core.kv_cache_utils import KVCacheBlock

from vllm.v1.metrics.stats import KVCacheEvictionEvent
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

### `BlockMetricsState` class / `BlockMetricsState` 类
```python
class BlockMetricsState:
    """Tracks lifecycle metrics for a single KV cache block."""
```
**EN:** Introduces the `BlockMetricsState` class. Core methods include `__init__`, `record_access`, `get_lifetime_seconds`, `get_idle_time_seconds`, `get_reuse_gaps_seconds`. Docstring signal: Tracks lifecycle metrics for a single KV cache block.
**CN:** 这里定义 `BlockMetricsState` 类。核心方法包括 `__init__`, `record_access`, `get_lifetime_seconds`, `get_idle_time_seconds`, `get_reuse_gaps_seconds`。

### `BlockMetricsState.__init__` method / `BlockMetricsState.__init__` 方法
```python
    def __init__(self):
        now_ns = time.monotonic_ns()
        self.birth_time_ns = now_ns
        self.last_access_ns = now_ns
        # Bounded to prevent unbounded growth if a block is accessed many times.
        self.access_history: deque[int] = deque(maxlen=4)
```
**EN:** This method initializes the object state within `BlockMetricsState`. Key calls include `monotonic_ns`, `deque`. It touches state such as `birth_time_ns`, `last_access_ns`, `access_history`.
**CN:** 该方法会初始化对象状态，其作用域位于`BlockMetricsState`。 关键调用包括 `monotonic_ns`, `deque`。 它会读写 `birth_time_ns`, `last_access_ns`, `access_history` 等状态。

### `BlockMetricsState.record_access` method / `BlockMetricsState.record_access` 方法
```python
    def record_access(self) -> None:
        now_ns = time.monotonic_ns()
        self.last_access_ns = now_ns
        self.access_history.append(now_ns)
```
**EN:** This method implements `record_access` within `BlockMetricsState`. Key calls include `monotonic_ns`, `append`. It touches state such as `last_access_ns`.
**CN:** 该方法会实现 `record_access`，其作用域位于`BlockMetricsState`。 关键调用包括 `monotonic_ns`, `append`。 它会读写 `last_access_ns` 等状态。

### `BlockMetricsState.get_lifetime_seconds` method / `BlockMetricsState.get_lifetime_seconds` 方法
```python
    def get_lifetime_seconds(self) -> float:
        now_ns = time.monotonic_ns()
        return (now_ns - self.birth_time_ns) / 1e9
```
**EN:** This method returns or derives a value within `BlockMetricsState`. Key calls include `monotonic_ns`.
**CN:** 该方法会返回或推导一个值，其作用域位于`BlockMetricsState`。 关键调用包括 `monotonic_ns`。

### `BlockMetricsState.get_idle_time_seconds` method / `BlockMetricsState.get_idle_time_seconds` 方法
```python
    def get_idle_time_seconds(self) -> float:
        now_ns = time.monotonic_ns()
        return (now_ns - self.last_access_ns) / 1e9
```
**EN:** This method returns or derives a value within `BlockMetricsState`. Key calls include `monotonic_ns`.
**CN:** 该方法会返回或推导一个值，其作用域位于`BlockMetricsState`。 关键调用包括 `monotonic_ns`。

### `BlockMetricsState.get_reuse_gaps_seconds` method / `BlockMetricsState.get_reuse_gaps_seconds` 方法
```python
    def get_reuse_gaps_seconds(self) -> list[float]:
        if len(self.access_history) < 2:
            return []
        history = list(self.access_history)
        return [(history[i] - history[i - 1]) / 1e9 for i in range(1, len(history))]
```
**EN:** This method returns or derives a value within `BlockMetricsState`. Key calls include `list`, `len`, `range`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`BlockMetricsState`。 关键调用包括 `list`, `len`, `range`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `KVCacheMetricsCollector` class / `KVCacheMetricsCollector` 类
```python
class KVCacheMetricsCollector:
    """Collects KV cache residency metrics with sampling."""
```
**EN:** Introduces the `KVCacheMetricsCollector` class. Core methods include `__init__`, `should_sample_block`, `on_block_allocated`, `on_block_accessed`, `on_block_evicted`, `reset`. Docstring signal: Collects KV cache residency metrics with sampling.
**CN:** 这里定义 `KVCacheMetricsCollector` 类。核心方法包括 `__init__`, `should_sample_block`, `on_block_allocated`, `on_block_accessed`, `on_block_evicted`, `reset`。

### `KVCacheMetricsCollector.__init__` method / `KVCacheMetricsCollector.__init__` 方法
```python
    def __init__(self, sample_rate: float = 0.01):
        assert 0 < sample_rate <= 1.0, (
            f"sample_rate must be in (0, 1.0], got {sample_rate}"
        )
        self.sample_rate = sample_rate

        self.block_metrics: dict[int, BlockMetricsState] = {}

        self._eviction_events: list[KVCacheEvictionEvent] = []
```
**EN:** This method initializes the object state within `KVCacheMetricsCollector`. It touches state such as `sample_rate`, `block_metrics`, `_eviction_events`.
**CN:** 该方法会初始化对象状态，其作用域位于`KVCacheMetricsCollector`。 它会读写 `sample_rate`, `block_metrics`, `_eviction_events` 等状态。

### `KVCacheMetricsCollector.should_sample_block` method / `KVCacheMetricsCollector.should_sample_block` 方法
```python
    def should_sample_block(self) -> bool:
        return random.random() < self.sample_rate
```
**EN:** This method implements `should_sample_block` within `KVCacheMetricsCollector`. Key calls include `random`.
**CN:** 该方法会实现 `should_sample_block`，其作用域位于`KVCacheMetricsCollector`。 关键调用包括 `random`。

### `KVCacheMetricsCollector.on_block_allocated` method / `KVCacheMetricsCollector.on_block_allocated` 方法
```python
    def on_block_allocated(self, block: "KVCacheBlock") -> None:
        if self.should_sample_block():
            self.block_metrics[block.block_id] = BlockMetricsState()
```
**EN:** This method implements `on_block_allocated` within `KVCacheMetricsCollector`. Key calls include `should_sample_block`, `BlockMetricsState`. It touches state such as `block_metrics`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `on_block_allocated`，其作用域位于`KVCacheMetricsCollector`。 关键调用包括 `should_sample_block`, `BlockMetricsState`。 它会读写 `block_metrics` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `KVCacheMetricsCollector.on_block_accessed` method / `KVCacheMetricsCollector.on_block_accessed` 方法
```python
    def on_block_accessed(self, block: "KVCacheBlock") -> None:
        metrics = self.block_metrics.get(block.block_id)
        if metrics:
            metrics.record_access()
```
**EN:** This method implements `on_block_accessed` within `KVCacheMetricsCollector`. Key calls include `get`, `record_access`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `on_block_accessed`，其作用域位于`KVCacheMetricsCollector`。 关键调用包括 `get`, `record_access`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `KVCacheMetricsCollector.on_block_evicted` method / `KVCacheMetricsCollector.on_block_evicted` 方法
```python
    def on_block_evicted(self, block: "KVCacheBlock") -> None:
        metrics = self.block_metrics.pop(block.block_id, None)
        if not metrics:
            return

        lifetime = metrics.get_lifetime_seconds()
        idle_time = metrics.get_idle_time_seconds()
        reuse_gaps = tuple(metrics.get_reuse_gaps_seconds())

        self._eviction_events.append(
            KVCacheEvictionEvent(
                lifetime_seconds=lifetime,
                idle_seconds=idle_time,
                reuse_gaps_seconds=reuse_gaps,
            )
        )
```
**EN:** This method implements `on_block_evicted` within `KVCacheMetricsCollector`. Key calls include `pop`, `get_lifetime_seconds`, `get_idle_time_seconds`, `tuple`, `append`, `get_reuse_gaps_seconds`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `on_block_evicted`，其作用域位于`KVCacheMetricsCollector`。 关键调用包括 `pop`, `get_lifetime_seconds`, `get_idle_time_seconds`, `tuple`, `append`, `get_reuse_gaps_seconds`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `KVCacheMetricsCollector.reset` method / `KVCacheMetricsCollector.reset` 方法
```python
    def reset(self) -> None:
        """Clear all state on cache reset."""
        self.block_metrics.clear()
        self._eviction_events.clear()
```
**EN:** This method implements `reset` within `KVCacheMetricsCollector`. The docstring frames it as: Clear all state on cache reset. Key calls include `clear`.
**CN:** 该方法会实现 `reset`，其作用域位于`KVCacheMetricsCollector`。 关键调用包括 `clear`。

### `KVCacheMetricsCollector.drain_events` method / `KVCacheMetricsCollector.drain_events` 方法
```python
    def drain_events(self) -> list[KVCacheEvictionEvent]:
        events = self._eviction_events
        self._eviction_events = []
        return events
```
**EN:** This method implements `drain_events` within `KVCacheMetricsCollector`. It touches state such as `_eviction_events`.
**CN:** 该方法会实现 `drain_events`，其作用域位于`KVCacheMetricsCollector`。 它会读写 `_eviction_events` 等状态。

## Key Concepts / 关键概念
- `BlockMetricsState`: central class or interface in this module. / `BlockMetricsState`：本模块中的核心类或接口。
- `KVCacheMetricsCollector`: central class or interface in this module. / `KVCacheMetricsCollector`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `random`, `time`, `collections`, `typing`
- Internal vLLM / 内部依赖: `vllm.v1.core.kv_cache_utils`, `vllm.v1.metrics.stats`
