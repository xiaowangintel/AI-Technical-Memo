# sticky_router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/schedule_simulator/routers/sticky_router.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on scheduler routing strategy simulation. It mainly encodes request-routing decisions used by the scheduling simulator. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于调度路由策略模拟。它主要用于编码调度模拟器中的请求路由决策。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Import dependencies and shared types / 导入依赖与共享类型
```python
import random
from collections import defaultdict

from sglang.srt.debug_utils.schedule_simulator.request import SimRequest
from sglang.srt.debug_utils.schedule_simulator.routers.base import RouterPolicy
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 8-8: Define class `StickyRouter` and class context / 定义类 `StickyRouter`及类上下文
```python
class StickyRouter(RouterPolicy):
```
**EN:** This section introduces `StickyRouter`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `StickyRouter`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 9-11: Implement method `__init__` for `StickyRouter` / 为 `StickyRouter` 实现方法 `__init__`
```python
    def __init__(self, num_gpus: int):
        self._num_gpus = num_gpus
        self._group_to_gpu = defaultdict(self._assign_gpu)
```
**EN:** Method `__init__` implements behavior on `StickyRouter`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__init__` 为 `StickyRouter` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 13-14: Implement method `_assign_gpu` for `StickyRouter` / 为 `StickyRouter` 实现方法 `_assign_gpu`
```python
    def _assign_gpu(self) -> int:
        return random.randint(0, self._num_gpus - 1)
```
**EN:** Method `_assign_gpu` implements behavior on `StickyRouter`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_assign_gpu` 为 `StickyRouter` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 16-20: Implement method `route` for `StickyRouter` / 为 `StickyRouter` 实现方法 `route`
```python
    def route(self, incoming_request: SimRequest) -> int:
        group_id = incoming_request.group_id
        if group_id is None:
            return random.randint(0, self._num_gpus - 1)
        return self._group_to_gpu[group_id]
```
**EN:** Method `route` implements behavior on `StickyRouter`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `route` 为 `StickyRouter` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `StickyRouter`
- **Module role / 模块角色**: Scheduler routing strategy simulation / 调度路由策略模拟
- **Implementation focus / 实现重点**: Encodes request-routing decisions used by the scheduling simulator / 编码调度模拟器中的请求路由决策

## Dependencies / 依赖关系
- **Standard library / 标准库**: `random`, `collections`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `sglang.srt.debug_utils.schedule_simulator.request`, `sglang.srt.debug_utils.schedule_simulator.routers.base`
