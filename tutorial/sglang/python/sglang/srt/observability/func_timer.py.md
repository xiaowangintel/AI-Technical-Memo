# func_timer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/observability/func_timer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the metrics, tracing, and monitoring part of the SRT runtime and implements logic centered on `func_timer`. The module docstring frames it as: "Records the latency of some functions" / 该模块属于 SRT 运行时的指标、追踪与监控部分，主要实现围绕 `func_timer` 的逻辑。 它对外提供的主要入口包括 `enable_func_timer`, `time_func_latency`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27: Module imports, constants, and setup
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
Records the latency of some functions
"""

import asyncio
import time
from functools import wraps
from typing import Any, Callable, Optional

from sglang.srt.observability.utils import exponential_buckets

enable_metrics = False


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; records metrics or tracing signals.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；记录指标或追踪信号。

### Lines 28-41: Function enable_func_timer
```python
def enable_func_timer():
    # We need to import prometheus_client after setting the env variable `PROMETHEUS_MULTIPROC_DIR`
    from prometheus_client import Histogram

    global enable_metrics, FUNC_LATENCY
    enable_metrics = True

    FUNC_LATENCY = Histogram(
        "sglang:func_latency_seconds",
        "Function latency in seconds",
        # captures latency in range [50ms - ~50s]
        buckets=exponential_buckets(start=0.05, width=1.5, length=18),
        labelnames=["name"],
    )
```
**EN:** This callable implements `enable_func_timer` and mainly implements enable func timer. In this range it sets up imports and shared symbols; records metrics or tracing signals; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `enable_func_timer`，主要用于实现 enable func timer 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；记录指标或追踪信号；管理图捕获或回放逻辑。

### Lines 42-46: Module-level constants and helpers
```python


FUNC_LATENCY = None


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 47-101: Function time_func_latency
```python
def time_func_latency(
    func: Callable = None, name: Optional[str] = None
) -> Callable[..., Any]:
    """
    A decorator to observe the latency of a function's execution. Supports both sync and async functions.

    NOTE: We use our own implementation of a timer decorator since prometheus_client does not support async
    context manager yet.

    Overhead: The overhead introduced here in case of an async function could likely be because of `await` introduced
    which will return in another coroutine object creation and under heavy load could see longer wall time
    (scheduling delays due to introduction of another awaitable).
    """

    def measure(func: Callable[..., Any]) -> Callable[..., Any]:
        nonlocal name

        name = name or func.__name__

        @wraps(func)
        async def async_wrapper(*args, **kwargs):
            if not enable_metrics:
                return await func(*args, **kwargs)

            metric = FUNC_LATENCY
            start = time.monotonic()
            ret = func(*args, **kwargs)
            if isinstance(ret, asyncio.Future) or asyncio.iscoroutine(ret):
                try:
                    ret = await ret
                finally:
                    metric.labels(name=name).observe(time.monotonic() - start)
            return ret

        @wraps(func)
        def sync_wrapper(*args, **kwargs):
            if not enable_metrics:
                return func(*args, **kwargs)

            metric = FUNC_LATENCY
            start = time.monotonic()
            try:
                ret = func(*args, **kwargs)
            finally:
                metric.labels(name=name).observe(time.monotonic() - start)
            return ret

        if asyncio.iscoroutinefunction(func):
            return async_wrapper
        return sync_wrapper

    if func:
        return measure(func)
    else:
        return measure
```
**EN:** This callable implements `time_func_latency`. It takes `func`, `name` and mainly implements time func latency. The docstring states: "A decorator to observe the latency of a function's execution." In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `time_func_latency`。它接收 `func`, `name`，主要用于实现 time func latency 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

## Key Concepts / 关键概念
- `enable_func_timer`: implements enable func timer / 实现 enable func timer 相关逻辑
- `FUNC_LATENCY`: module constant or capability flag / 模块常量或能力标记
- `time_func_latency`: implements time func latency / 实现 time func latency 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `time`, `functools`, `typing`
- **Third-party / 第三方**: `prometheus_client`
- **Internal modules / 内部模块**: `sglang.srt.observability.utils`
