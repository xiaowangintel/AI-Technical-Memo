# dispatch_policy.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/disaggregation/dispatch_policy.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the disaggregation runtime layer. It centers on `DispatchPolicy`, `RoundRobin`, and `MaxFreeSlotsFirst`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Dispatch policies for multi-instance disaggregated diffusion pipelines. / 该文件属于解耦运行时层。它围绕 `DispatchPolicy`、`RoundRobin` 和 `MaxFreeSlotsFirst` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-6: module setup and imports / 模块初始化与导入
```python
"""Dispatch policies for multi-instance disaggregated diffusion pipelines."""

import abc
import logging
import threading
```
**EN:** This block establishes the module context and imports `abc`, `logging`, and `threading`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `abc`、`logging` 和 `threading`。这些依赖为后续实现提供所需符号。

### Lines 8-8: supporting statements / 辅助语句
```python
logger = logging.getLogger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `logging.getLogger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `logging.getLogger` 协同工作。

### Lines 11-11: `DispatchPolicy` class overview / `DispatchPolicy` 类概览
```python
class DispatchPolicy(abc.ABC):
```
**EN:** This block defines class `DispatchPolicy`. It encapsulates dispatch policy behavior. It inherits from `abc.ABC`.
**CN:** 该代码块定义了类 `DispatchPolicy`。 它用于封装 dispatch policy 相关行为。 它继承自 `abc.ABC`。

### Lines 12-15: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, num_instances: int):
        if num_instances < 1:
            raise ValueError(f"num_instances must be >= 1, got {num_instances}")
        self._num_instances = num_instances
```
**EN:** This block defines method `__init__` on `DispatchPolicy`. It initializes the instance state. Key calls include `ValueError`. The implementation branches on conditions. Parameters such as `num_instances` drive the behavior in this section.
**CN:** 该代码块定义了 `DispatchPolicy` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `num_instances` 等参数驱动。

### Lines 17-19: `num_instances` implementation / `num_instances` 实现
```python
    @property
    def num_instances(self) -> int:
        return self._num_instances
```
**EN:** This block defines method `num_instances` on `DispatchPolicy`. It handles num instances logic.
**CN:** 该代码块定义了 `DispatchPolicy` 的方法 `num_instances`。 它用于处理 num instances 相关逻辑。

### Lines 21-22: `select` implementation / `select` 实现
```python
    @abc.abstractmethod
    def select(self, active_counts: list[int] | None = None) -> int: ...
```
**EN:** This block defines method `select` on `DispatchPolicy`. It selects function. Parameters such as `active_counts` drive the behavior in this section.
**CN:** 该代码块定义了 `DispatchPolicy` 的方法 `select`。 它用于选择函数。 本段逻辑主要由 `active_counts` 等参数驱动。

### Lines 24-28: `select_with_capacity` implementation / `select_with_capacity` 实现
```python
    def select_with_capacity(self, free_slots: list[int]) -> int | None:
        """Select an instance that has free capacity, or None if all full."""
        if not any(s > 0 for s in free_slots):
            return None
        return self.select(active_counts=None)
```
**EN:** This block defines method `select_with_capacity` on `DispatchPolicy`. Select an instance that has free capacity, or None if all full. Key calls include `self.select`, and `any`. The implementation branches on conditions. Parameters such as `free_slots` drive the behavior in this section.
**CN:** 该代码块定义了 `DispatchPolicy` 的方法 `select_with_capacity`。 它用于选择with capacity。 关键调用包括 `self.select` 和 `any`。 实现中包含条件分支。 本段逻辑主要由 `free_slots` 等参数驱动。

### Lines 30-31: `record_completion` implementation / `record_completion` 实现
```python
    def record_completion(self, instance_id: int) -> None:
        pass
```
**EN:** This block defines method `record_completion` on `DispatchPolicy`. It handles record completion logic. Parameters such as `instance_id` drive the behavior in this section.
**CN:** 该代码块定义了 `DispatchPolicy` 的方法 `record_completion`。 它用于处理 record completion 相关逻辑。 本段逻辑主要由 `instance_id` 等参数驱动。

### Lines 34-34: `RoundRobin` class overview / `RoundRobin` 类概览
```python
class RoundRobin(DispatchPolicy):
```
**EN:** This block defines class `RoundRobin`. It encapsulates round robin behavior. It inherits from `DispatchPolicy`.
**CN:** 该代码块定义了类 `RoundRobin`。 它用于封装 round robin 相关行为。 它继承自 `DispatchPolicy`。

### Lines 35-38: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, num_instances: int):
        super().__init__(num_instances)
        self._lock = threading.Lock()
        self._next = 0
```
**EN:** This block defines method `__init__` on `RoundRobin`. It initializes the instance state. Key calls include `super.__init__`, `threading.Lock`, and `super`. Parameters such as `num_instances` drive the behavior in this section.
**CN:** 该代码块定义了 `RoundRobin` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`threading.Lock` 和 `super`。 本段逻辑主要由 `num_instances` 等参数驱动。

### Lines 40-44: `select` implementation / `select` 实现
```python
    def select(self, active_counts: list[int] | None = None) -> int:
        with self._lock:
            chosen = self._next
            self._next = (self._next + 1) % self._num_instances
        return chosen
```
**EN:** This block defines method `select` on `RoundRobin`. It selects function. The implementation uses context-managed resources. Parameters such as `active_counts` drive the behavior in this section.
**CN:** 该代码块定义了 `RoundRobin` 的方法 `select`。 它用于选择函数。 实现中使用上下文管理资源。 本段逻辑主要由 `active_counts` 等参数驱动。

### Lines 46-53: `select_with_capacity` implementation / `select_with_capacity` 实现
```python
    def select_with_capacity(self, free_slots: list[int]) -> int | None:
        with self._lock:
            for _ in range(self._num_instances):
                idx = self._next
                self._next = (self._next + 1) % self._num_instances
                if free_slots[idx] > 0:
                    return idx
            return None
```
**EN:** This block defines method `select_with_capacity` on `RoundRobin`. It selects with capacity. Key calls include `range`. The implementation branches on conditions, iterates over collections or steps, uses context-managed resources. Parameters such as `free_slots` drive the behavior in this section.
**CN:** 该代码块定义了 `RoundRobin` 的方法 `select_with_capacity`。 它用于选择with capacity。 关键调用包括 `range`。 实现中包含条件分支，会遍历集合或步骤，使用上下文管理资源。 本段逻辑主要由 `free_slots` 等参数驱动。

### Lines 56-58: `MaxFreeSlotsFirst` class overview / `MaxFreeSlotsFirst` 类概览
```python
class MaxFreeSlotsFirst(DispatchPolicy):
    """Dispatch to the instance with the most free slots."""
```
**EN:** This block defines class `MaxFreeSlotsFirst`. Dispatch to the instance with the most free slots. It inherits from `DispatchPolicy`.
**CN:** 该代码块定义了类 `MaxFreeSlotsFirst`。 它用于封装 max free slots first 相关行为。 它继承自 `DispatchPolicy`。

### Lines 59-63: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, num_instances: int, max_slots_per_instance: int = 1):
        super().__init__(num_instances)
        self._max_slots = max_slots_per_instance
        self._lock = threading.Lock()
        self._tiebreak = 0
```
**EN:** This block defines method `__init__` on `MaxFreeSlotsFirst`. It initializes the instance state. Key calls include `super.__init__`, `threading.Lock`, and `super`. Parameters such as `num_instances`, and `max_slots_per_instance` drive the behavior in this section.
**CN:** 该代码块定义了 `MaxFreeSlotsFirst` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`threading.Lock` 和 `super`。 本段逻辑主要由 `num_instances` 和 `max_slots_per_instance` 等参数驱动。

### Lines 65-94: `select` implementation / `select` 实现
```python
    def select(self, active_counts: list[int] | None = None) -> int:
        with self._lock:
            if active_counts is None or len(active_counts) != self._num_instances:
                chosen = self._tiebreak % self._num_instances
                self._tiebreak += 1
                return chosen

            best_id = 0
            best_free = self._max_slots - active_counts[0]
            for i in range(1, self._num_instances):
                free = self._max_slots - active_counts[i]
                if free > best_free:
                    best_free = free
                    best_id = i
                elif free == best_free:
                    if i == (self._tiebreak % self._num_instances):
                        best_id = i

            self._tiebreak += 1

            if best_free <= 0:
                logger.warning(
                    "All %d instances are at capacity (%d slots each), "
                    "dispatching to instance %d anyway",
                    self._num_instances,
                    self._max_slots,
                    best_id,
                )

            return best_id
```
**EN:** This block defines method `select` on `MaxFreeSlotsFirst`. It selects function. Key calls include `range`, `logger.warning`, and `len`. The implementation branches on conditions, iterates over collections or steps, uses context-managed resources. Parameters such as `active_counts` drive the behavior in this section.
**CN:** 该代码块定义了 `MaxFreeSlotsFirst` 的方法 `select`。 它用于选择函数。 关键调用包括 `range`、`logger.warning` 和 `len`。 实现中包含条件分支，会遍历集合或步骤，使用上下文管理资源。 本段逻辑主要由 `active_counts` 等参数驱动。

### Lines 96-112: `select_with_capacity` implementation / `select_with_capacity` 实现
```python
    def select_with_capacity(self, free_slots: list[int]) -> int | None:
        with self._lock:
            best_id = -1
            best_free = 0
            for i in range(self._num_instances):
                if free_slots[i] > best_free:
                    best_free = free_slots[i]
                    best_id = i
                elif free_slots[i] == best_free and best_free > 0:
                    if i == (self._tiebreak % self._num_instances):
                        best_id = i

            self._tiebreak += 1

            if best_id < 0:
                return None
            return best_id
```
**EN:** This block defines method `select_with_capacity` on `MaxFreeSlotsFirst`. It selects with capacity. Key calls include `range`. The implementation branches on conditions, iterates over collections or steps, uses context-managed resources. Parameters such as `free_slots` drive the behavior in this section.
**CN:** 该代码块定义了 `MaxFreeSlotsFirst` 的方法 `select_with_capacity`。 它用于选择with capacity。 关键调用包括 `range`。 实现中包含条件分支，会遍历集合或步骤，使用上下文管理资源。 本段逻辑主要由 `free_slots` 等参数驱动。

### Lines 115-117: `PoolDispatcher` class overview / `PoolDispatcher` 类概览
```python
class PoolDispatcher:
    """Wraps three independent dispatch policies for encoder/denoiser/decoder pools."""
```
**EN:** This block defines class `PoolDispatcher`. Wraps three independent dispatch policies for encoder/denoiser/decoder pools.
**CN:** 该代码块定义了类 `PoolDispatcher`。 它用于封装 pool dispatcher 相关行为。

### Lines 118-134: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        num_encoders: int,
        num_denoisers: int,
        num_decoders: int,
        policy_name: str = "round_robin",
        **kwargs,
    ):
        self.encoder_policy = create_dispatch_policy(
            policy_name, num_encoders, **kwargs
        )
        self.denoiser_policy = create_dispatch_policy(
            policy_name, num_denoisers, **kwargs
        )
        self.decoder_policy = create_dispatch_policy(
            policy_name, num_decoders, **kwargs
        )
```
**EN:** This block defines method `__init__` on `PoolDispatcher`. It initializes the instance state. Key calls include `create_dispatch_policy`. Parameters such as `num_encoders`, `num_denoisers`, `num_decoders`, and `policy_name` drive the behavior in this section.
**CN:** 该代码块定义了 `PoolDispatcher` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `create_dispatch_policy`。 本段逻辑主要由 `num_encoders`、`num_denoisers`、`num_decoders` 和 `policy_name` 等参数驱动。

### Lines 136-137: `select_encoder` implementation / `select_encoder` 实现
```python
    def select_encoder(self, active_counts: list[int] | None = None) -> int:
        return self.encoder_policy.select(active_counts)
```
**EN:** This block defines method `select_encoder` on `PoolDispatcher`. It selects encoder. Key calls include `self.encoder_policy.select`. Parameters such as `active_counts` drive the behavior in this section.
**CN:** 该代码块定义了 `PoolDispatcher` 的方法 `select_encoder`。 它用于选择encoder。 关键调用包括 `self.encoder_policy.select`。 本段逻辑主要由 `active_counts` 等参数驱动。

### Lines 139-140: `select_denoiser` implementation / `select_denoiser` 实现
```python
    def select_denoiser(self, active_counts: list[int] | None = None) -> int:
        return self.denoiser_policy.select(active_counts)
```
**EN:** This block defines method `select_denoiser` on `PoolDispatcher`. It selects denoiser. Key calls include `self.denoiser_policy.select`. Parameters such as `active_counts` drive the behavior in this section.
**CN:** 该代码块定义了 `PoolDispatcher` 的方法 `select_denoiser`。 它用于选择denoiser。 关键调用包括 `self.denoiser_policy.select`。 本段逻辑主要由 `active_counts` 等参数驱动。

### Lines 142-143: `select_decoder` implementation / `select_decoder` 实现
```python
    def select_decoder(self, active_counts: list[int] | None = None) -> int:
        return self.decoder_policy.select(active_counts)
```
**EN:** This block defines method `select_decoder` on `PoolDispatcher`. It selects decoder. Key calls include `self.decoder_policy.select`. Parameters such as `active_counts` drive the behavior in this section.
**CN:** 该代码块定义了 `PoolDispatcher` 的方法 `select_decoder`。 它用于选择decoder。 关键调用包括 `self.decoder_policy.select`。 本段逻辑主要由 `active_counts` 等参数驱动。

### Lines 145-146: `select_encoder_with_capacity` implementation / `select_encoder_with_capacity` 实现
```python
    def select_encoder_with_capacity(self, free_slots: list[int]) -> int | None:
        return self.encoder_policy.select_with_capacity(free_slots)
```
**EN:** This block defines method `select_encoder_with_capacity` on `PoolDispatcher`. It selects encoder with capacity. Key calls include `self.encoder_policy.select_with_capacity`. Parameters such as `free_slots` drive the behavior in this section.
**CN:** 该代码块定义了 `PoolDispatcher` 的方法 `select_encoder_with_capacity`。 它用于选择encoder with capacity。 关键调用包括 `self.encoder_policy.select_with_capacity`。 本段逻辑主要由 `free_slots` 等参数驱动。

### Lines 148-149: `select_denoiser_with_capacity` implementation / `select_denoiser_with_capacity` 实现
```python
    def select_denoiser_with_capacity(self, free_slots: list[int]) -> int | None:
        return self.denoiser_policy.select_with_capacity(free_slots)
```
**EN:** This block defines method `select_denoiser_with_capacity` on `PoolDispatcher`. It selects denoiser with capacity. Key calls include `self.denoiser_policy.select_with_capacity`. Parameters such as `free_slots` drive the behavior in this section.
**CN:** 该代码块定义了 `PoolDispatcher` 的方法 `select_denoiser_with_capacity`。 它用于选择denoiser with capacity。 关键调用包括 `self.denoiser_policy.select_with_capacity`。 本段逻辑主要由 `free_slots` 等参数驱动。

### Lines 151-152: `select_decoder_with_capacity` implementation / `select_decoder_with_capacity` 实现
```python
    def select_decoder_with_capacity(self, free_slots: list[int]) -> int | None:
        return self.decoder_policy.select_with_capacity(free_slots)
```
**EN:** This block defines method `select_decoder_with_capacity` on `PoolDispatcher`. It selects decoder with capacity. Key calls include `self.decoder_policy.select_with_capacity`. Parameters such as `free_slots` drive the behavior in this section.
**CN:** 该代码块定义了 `PoolDispatcher` 的方法 `select_decoder_with_capacity`。 它用于选择decoder with capacity。 关键调用包括 `self.decoder_policy.select_with_capacity`。 本段逻辑主要由 `free_slots` 等参数驱动。

### Lines 155-165: `create_dispatch_policy` implementation / `create_dispatch_policy` 实现
```python
def create_dispatch_policy(name: str, num_instances: int, **kwargs) -> DispatchPolicy:
    policies = {
        "round_robin": RoundRobin,
        "max_free_slots": MaxFreeSlotsFirst,
    }
    cls = policies.get(name)
    if cls is None:
        raise ValueError(
            f"Unknown dispatch policy '{name}'. Available: {list(policies.keys())}"
        )
    return cls(num_instances=num_instances, **kwargs)
```
**EN:** This block defines function `create_dispatch_policy`. It creates dispatch policy. Key calls include `policies.get`, `cls`, `ValueError`, `list`, and `policies.keys`. The implementation branches on conditions. Parameters such as `name`, and `num_instances` drive the behavior in this section.
**CN:** 该代码块定义了函数 `create_dispatch_policy`。 它用于创建dispatch policy。 关键调用包括 `policies.get`、`cls`、`ValueError`、`list` 和 `policies.keys`。 实现中包含条件分支。 本段逻辑主要由 `name` 和 `num_instances` 等参数驱动。

## Key Concepts / 关键概念
- `DispatchPolicy`: Primary class that encapsulates dispatch policy behavior. / 核心类，用于封装 dispatch policy 相关行为。
- `RoundRobin`: Primary class that encapsulates round robin behavior. / 核心类，用于封装 round robin 相关行为。
- `MaxFreeSlotsFirst`: Dispatch to the instance with the most free slots. / 核心类，用于封装 max free slots first 相关行为。
- `PoolDispatcher`: Wraps three independent dispatch policies for encoder/denoiser/decoder pools. / 核心类，用于封装 pool dispatcher 相关行为。
- `create_dispatch_policy`: Top-level function that creates dispatch policy. / 顶层函数，用于创建dispatch policy。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `abc`, `logging`, `threading`

- **Total lines / 总行数**: 165
