# idle_sleeper.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/scheduler_components/idle_sleeper.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements idle sleeper logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 idle sleeper 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
import zmq
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-5: Import runtime dependencies / 导入运行时依赖
```python
from sglang.srt.environ import envs
from sglang.srt.observability.req_time_stats import real_time
from sglang.srt.platforms import current_platform
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 8-35: Define class IdleSleeper / 定义类 IdleSleeper
```python
class IdleSleeper:
    """
    In setups which have long inactivity periods it is desirable to reduce
    system power consumption when sglang does nothing. This would lead not only
    to power savings, but also to more CPU thermal headroom when a request
    eventually comes. This is important in cases when multiple GPUs are connected
    as each GPU would otherwise pin one thread at 100% CPU usage.

    The simplest solution is to use zmq.Poller on all sockets that may receive
    data that needs handling immediately.
    """

    def __init__(self, sockets):
        self.poller = zmq.Poller()
        self.last_empty_time = real_time()
        for s in sockets:
            self.poller.register(s, zmq.POLLIN)

        self.empty_cache_interval = envs.SGLANG_EMPTY_CACHE_INTERVAL.get()

    def maybe_sleep(self):
        self.poller.poll(1000)
        if (
            self.empty_cache_interval > 0
            and real_time() - self.last_empty_time > self.empty_cache_interval
        ):
            self.last_empty_time = real_time()
            current_platform.empty_cache()
```
**EN:** This block declares the class `IdleSleeper`. It centers on In setups which have long inactivity periods it is desirable to reduce system power consumption when sglang does nothing., with methods such as __init__, maybe_sleep.
**CN:** 该代码块声明类 `IdleSleeper`。它负责承载与 idle sleeper 相关的核心状态与行为，并通过 __init__, maybe_sleep 等方法组织实现。

## Key Concepts / 关键概念
- **Core types / 核心类型**: IdleSleeper
- **Domain focus / 领域焦点**: idle sleeper / idle sleeper
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: None / 无
- **Third-party / 第三方库**: zmq
- **Local Modules / 本地模块**: sglang.srt.environ, sglang.srt.observability.req_time_stats, sglang.srt.platforms
