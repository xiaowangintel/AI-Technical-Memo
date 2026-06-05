# device_timer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/device_timer.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable runtime utilities centered on `device_timer` and the surrounding SGLang serving stack. / 提供围绕 `device_timer` 及其所处 SGLang 服务栈的可复用运行时工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Module setup and shared state / 模块设置与共享状态
```python
from collections import deque
from contextlib import contextmanager
from dataclasses import dataclass
from typing import Callable, Deque, Dict, List, Optional

import torch
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `collections`, `contextlib`, `dataclasses`, `typing`, `torch`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `collections`, `contextlib`, `dataclasses`, `typing`, `torch`。

### Lines 9-9: Class `DeviceTimer` declaration / 类 `DeviceTimer` 声明
```python
class DeviceTimer:
```
**EN:** This class establishes `DeviceTimer` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `add_reporter`, `wrap`, `_report`.
**CN:** 该类将 `DeviceTimer` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `add_reporter`, `wrap`, `_report` 等方法。

### Lines 10-12: Method `DeviceTimer.__init__` / 方法 `DeviceTimer.__init__`
```python
    def __init__(self, reporter: Callable):
        self._intervals: Deque[_TimingInterval] = deque()
        self._reporters: List[Callable] = [reporter]
```
**EN:** This method implements `__init__` on `DeviceTimer`. It primarily calls `deque` to complete its work. State updates are written into `self._intervals`, `self._reporters`.
**CN:** 该方法（属于 `DeviceTimer`）实现了 `__init__`。 它主要通过调用 `deque` 来完成任务。 状态更新主要写入 `self._intervals`, `self._reporters`。

### Lines 14-15: Method `DeviceTimer.add_reporter` / 方法 `DeviceTimer.add_reporter`
```python
    def add_reporter(self, reporter: Callable):
        self._reporters.append(reporter)
```
**EN:** This method implements `add_reporter` on `DeviceTimer`. It primarily calls `self._reporters.append` to complete its work.
**CN:** 该方法（属于 `DeviceTimer`）实现了 `add_reporter`。 它主要通过调用 `self._reporters.append` 来完成任务。

### Lines 17-24: Method `DeviceTimer.wrap` / 方法 `DeviceTimer.wrap`
```python
    @contextmanager
    def wrap(self, metadata: Dict):
        self._intervals.append(_TimingInterval.create())
        try:
            yield
        finally:
            self._intervals[-1].end(metadata=metadata)
            self._report()
```
**EN:** This method implements `wrap` on `DeviceTimer`. It primarily calls `self._intervals.append`, `_TimingInterval.create`, `self._intervals.end`, `self._report` to complete its work. The implementation relies on error handling.
**CN:** 该方法（属于 `DeviceTimer`）实现了 `wrap`。 它主要通过调用 `self._intervals.append`, `_TimingInterval.create`, `self._intervals.end`, `self._report` 来完成任务。 实现中使用了错误处理。

### Lines 26-35: Method `DeviceTimer._report` / 方法 `DeviceTimer._report`
```python
    def _report(self):
        while len(self._intervals) > 0:
            interval = self._intervals[0]
            if not interval.end_event.query():
                break

            self._intervals.popleft()
            elapsed = interval.elapsed_time() / 1000.0
            for reporter in self._reporters:
                reporter(t=elapsed, **interval.metadata)
```
**EN:** This method implements `_report` on `DeviceTimer`. It primarily calls `len`, `self._intervals.popleft`, `interval.end_event.query`, `interval.elapsed_time`, `reporter` to complete its work. State updates are written into `interval`, `elapsed`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `DeviceTimer`）实现了 `_report`。 它主要通过调用 `len`, `self._intervals.popleft`, `interval.end_event.query`, `interval.elapsed_time`, `reporter` 来完成任务。 状态更新主要写入 `interval`, `elapsed`。 实现中使用了条件分支、迭代逻辑。

### Lines 38-44: Class `GapTimer` declaration / 类 `GapTimer` 声明
```python
class GapTimer(DeviceTimer):
    """Measures GPU idle gaps between consecutive uses of a stream.

    Where DeviceTimer.wrap() measures the duration *inside* a block,
    GapTimer.wrap() measures the time *between* consecutive blocks
    (gap = next_block_start - last_block_end).
    """
```
**EN:** This class establishes `GapTimer` as the main container/coordinator for the surrounding logic. It inherits from `DeviceTimer`. Its core interface includes methods such as `__init__`, `wrap`, `cancel`.
**CN:** 该类将 `GapTimer` 定义为周边逻辑的主要封装体或协调者。 它继承自 `DeviceTimer`。 其核心接口包括 `__init__`, `wrap`, `cancel` 等方法。

### Lines 46-48: Method `GapTimer.__init__` / 方法 `GapTimer.__init__`
```python
    def __init__(self, reporter: Callable):
        super().__init__(reporter)
        self._pending: Optional[_TimingInterval] = None
```
**EN:** This method implements `__init__` on `GapTimer`. It primarily calls `super.__init__`, `super` to complete its work. State updates are written into `self._pending`.
**CN:** 该方法（属于 `GapTimer`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `super` 来完成任务。 状态更新主要写入 `self._pending`。

### Lines 50-60: Method `GapTimer.wrap` / 方法 `GapTimer.wrap`
```python
    @contextmanager
    def wrap(self, metadata: Dict):
        if self._pending is not None:
            self._pending.end(metadata=metadata)
            self._intervals.append(self._pending)
            self._pending = None
            self._report()
        try:
            yield
        finally:
            self._pending = _TimingInterval.create()
```
**EN:** This method implements `wrap` on `GapTimer`. It primarily calls `self._pending.end`, `self._intervals.append`, `self._report`, `_TimingInterval.create` to complete its work. State updates are written into `self._pending`. The implementation relies on conditional branches, error handling.
**CN:** 该方法（属于 `GapTimer`）实现了 `wrap`。 它主要通过调用 `self._pending.end`, `self._intervals.append`, `self._report`, `_TimingInterval.create` 来完成任务。 状态更新主要写入 `self._pending`。 实现中使用了条件分支、错误处理。

### Lines 62-64: Method `GapTimer.cancel` / 方法 `GapTimer.cancel`
```python
    def cancel(self):
        """Discard a pending gap (e.g. server went idle)."""
        self._pending = None
```
**EN:** This method implements `cancel` on `GapTimer`. State updates are written into `self._pending`.
**CN:** 该方法（属于 `GapTimer`）实现了 `cancel`。 状态更新主要写入 `self._pending`。

### Lines 67-71: Class `_TimingInterval` declaration / 类 `_TimingInterval` 声明
```python
@dataclass
class _TimingInterval:
    start_event: torch.cuda.Event
    end_event: Optional[torch.cuda.Event] = None
    metadata: Optional[Dict] = None
```
**EN:** This class establishes `_TimingInterval` as a compact data container for the surrounding logic. Its core interface includes methods such as `create`, `end`, `elapsed_time`.
**CN:** 该类将 `_TimingInterval` 定义为周边逻辑的紧凑的数据容器。 其核心接口包括 `create`, `end`, `elapsed_time` 等方法。

### Lines 73-77: Method `_TimingInterval.create` / 方法 `_TimingInterval.create`
```python
    @staticmethod
    def create():
        start_event = torch.cuda.Event(enable_timing=True)
        start_event.record()
        return _TimingInterval(start_event=start_event)
```
**EN:** This method implements `create` on `_TimingInterval`. It primarily calls `torch.cuda.Event`, `start_event.record`, `_TimingInterval` to complete its work. State updates are written into `start_event`.
**CN:** 该方法（属于 `_TimingInterval`）实现了 `create`。 它主要通过调用 `torch.cuda.Event`, `start_event.record`, `_TimingInterval` 来完成任务。 状态更新主要写入 `start_event`。

### Lines 79-85: Method `_TimingInterval.end` / 方法 `_TimingInterval.end`
```python
    def end(self, metadata: Dict):
        end_event = torch.cuda.Event(enable_timing=True)
        end_event.record()

        assert self.end_event is None
        self.end_event = end_event
        self.metadata = metadata
```
**EN:** This method implements `end` on `_TimingInterval`. It primarily calls `torch.cuda.Event`, `end_event.record` to complete its work. State updates are written into `end_event`, `self.end_event`, `self.metadata`.
**CN:** 该方法（属于 `_TimingInterval`）实现了 `end`。 它主要通过调用 `torch.cuda.Event`, `end_event.record` 来完成任务。 状态更新主要写入 `end_event`, `self.end_event`, `self.metadata`。

### Lines 87-88: Method `_TimingInterval.elapsed_time` / 方法 `_TimingInterval.elapsed_time`
```python
    def elapsed_time(self) -> float:
        return self.start_event.elapsed_time(self.end_event)
```
**EN:** This method implements `elapsed_time` on `_TimingInterval`. It primarily calls `self.start_event.elapsed_time` to complete its work.
**CN:** 该方法（属于 `_TimingInterval`）实现了 `elapsed_time`。 它主要通过调用 `self.start_event.elapsed_time` 来完成任务。

## Key Concepts / 关键概念
- **Classes / 类**: `DeviceTimer`, `GapTimer`, `_TimingInterval`
- **Functions / 函数**: `__init__`, `add_reporter`, `wrap`, `_report`, `__init__`, `wrap`, `cancel`, `create`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: None / 无
- **External / 外部依赖**: `torch`
- **Standard library / 标准库**: `collections`, `contextlib`, `dataclasses`, `typing`
