# cpu_monitor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/observability/cpu_monitor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the metrics, tracing, and monitoring part of the SRT runtime and implements logic centered on `cpu_monitor`. It exposes primary entry points such as `start_cpu_monitor_thread`. / 该模块属于 SRT 运行时的指标、追踪与监控部分，主要实现围绕 `cpu_monitor` 的逻辑。 它对外提供的主要入口包括 `start_cpu_monitor_thread`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Module imports, constants, and setup
```python
import threading
import time

import psutil


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 7-31: Function start_cpu_monitor_thread
```python
def start_cpu_monitor_thread(component: str, interval: float = 5.0) -> threading.Thread:
    from prometheus_client import Counter

    cpu_seconds_total = Counter(
        name="sglang:process_cpu_seconds_total",
        documentation="Total CPU time consumed by this process (user + system)",
        labelnames=["component"],
    )

    def monitor():
        process = psutil.Process()
        last_times = process.cpu_times()

        while True:
            time.sleep(interval)
            curr_times = process.cpu_times()
            delta = (curr_times.user - last_times.user) + (
                curr_times.system - last_times.system
            )
            cpu_seconds_total.labels(component=component).inc(delta)
            last_times = curr_times

    t = threading.Thread(target=monitor, daemon=True)
    t.start()
    return t
```
**EN:** This callable implements `start_cpu_monitor_thread`. It takes `component`, `interval` and mainly converts data into another representation. In this range it sets up imports and shared symbols; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `start_cpu_monitor_thread`。它接收 `component`, `interval`，主要用于将数据转换为另一种表示。 在这一范围内，它会建立导入关系并准备共享符号；记录指标或追踪信号。

## Key Concepts / 关键概念
- `start_cpu_monitor_thread`: converts data into another representation / 将数据转换为另一种表示

## Dependencies / 依赖关系
- **Standard library / 标准库**: `threading`, `time`
- **Third-party / 第三方**: `psutil`, `prometheus_client`
