# watchdog.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/watchdog.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable runtime utilities centered on `watchdog` and the surrounding SGLang serving stack. / 提供围绕 `watchdog` 及其所处 SGLang 服务栈的可复用运行时工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Module setup and shared state / 模块设置与共享状态
```python
from __future__ import annotations

import logging
import os
import signal
import sys
import threading
import time
from contextlib import contextmanager
from multiprocessing import Process
from typing import Callable, List, Optional

import psutil

from sglang.srt.utils.common import pyspy_dump_schedulers

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `logging`, `os`, `signal`, `sys`, `threading`. It also defines symbols such as `logger` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `logging`, `os`, `signal`, `sys`, `threading`。 同时定义了 `logger` 等符号，供后续逻辑使用。

### Lines 20-20: Class `Watchdog` declaration / 类 `Watchdog` 声明
```python
class Watchdog:
```
**EN:** This class establishes `Watchdog` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `create`, `feed`, `disable`.
**CN:** 该类将 `Watchdog` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `create`, `feed`, `disable` 等方法。

### Lines 21-38: Method `Watchdog.create` / 方法 `Watchdog.create`
```python
    @staticmethod
    def create(
        debug_name: str,
        watchdog_timeout: Optional[float],
        soft: bool = False,
        test_stuck_time: float = 0,
    ) -> Watchdog:
        if watchdog_timeout is None:
            assert (
                test_stuck_time == 0
            ), f"stuck tester can be enabled only if soft watchdog is enabled."
            return _WatchdogNoop()
        return _WatchdogReal(
            debug_name=debug_name,
            watchdog_timeout=watchdog_timeout,
            soft=soft,
            test_stuck_time=test_stuck_time,
        )
```
**EN:** This method implements `create` on `Watchdog`. It primarily calls `_WatchdogReal`, `_WatchdogNoop` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `Watchdog`）实现了 `create`。 它主要通过调用 `_WatchdogReal`, `_WatchdogNoop` 来完成任务。 实现中使用了条件分支。

### Lines 40-41: Method `Watchdog.feed` / 方法 `Watchdog.feed`
```python
    def feed(self):
        pass
```
**EN:** This method implements `feed` on `Watchdog`.
**CN:** 该方法（属于 `Watchdog`）实现了 `feed`。

### Lines 43-45: Method `Watchdog.disable` / 方法 `Watchdog.disable`
```python
    @contextmanager
    def disable(self):
        yield
```
**EN:** This method implements `disable` on `Watchdog`.
**CN:** 该方法（属于 `Watchdog`）实现了 `disable`。

### Lines 48-48: Class `_WatchdogReal` declaration / 类 `_WatchdogReal` 声明
```python
class _WatchdogReal(Watchdog):
```
**EN:** This class establishes `_WatchdogReal` as the main container/coordinator for the surrounding logic. It inherits from `Watchdog`. Its core interface includes methods such as `__init__`, `feed`, `disable`.
**CN:** 该类将 `_WatchdogReal` 定义为周边逻辑的主要封装体或协调者。 它继承自 `Watchdog`。 其核心接口包括 `__init__`, `feed`, `disable` 等方法。

### Lines 49-71: Method `_WatchdogReal.__init__` / 方法 `_WatchdogReal.__init__`
```python
    def __init__(
        self,
        debug_name: str,
        watchdog_timeout: float,
        soft: bool = False,
        test_stuck_time: float = 0,
    ):
        self._counter = 0
        self._active = True
        self._test_stuck_time = test_stuck_time
        self._test_stuck_triggered = False
        self._raw = WatchdogRaw(
            debug_name=debug_name,
            get_counter=lambda: self._counter,
            is_active=lambda: self._active,
            watchdog_timeout=watchdog_timeout,
            soft=soft,
        )
        logger.info(f"Watchdog {self._raw.debug_name} initialized.")
        if self._test_stuck_time > 0:
            logger.info(
                f"Watchdog {self._raw.debug_name} is configured to use {test_stuck_time=}."
            )
```
**EN:** This method implements `__init__` on `_WatchdogReal`. It primarily calls `WatchdogRaw`, `logger.info` to complete its work. State updates are written into `self._counter`, `self._active`, `self._test_stuck_time`, `self._test_stuck_triggered`, `self._raw`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `_WatchdogReal`）实现了 `__init__`。 它主要通过调用 `WatchdogRaw`, `logger.info` 来完成任务。 状态更新主要写入 `self._counter`, `self._active`, `self._test_stuck_time`, `self._test_stuck_triggered`, `self._raw`。 实现中使用了条件分支。

### Lines 73-86: Method `_WatchdogReal.feed` / 方法 `_WatchdogReal.feed`
```python
    def feed(self):
        # Only trigger the test stuck behavior once to avoid blocking server
        # startup health checks while still testing watchdog timeout detection
        if self._test_stuck_time > 0 and not self._test_stuck_triggered:
            self._test_stuck_triggered = True
            logger.info(
                f"Watchdog {self._raw.debug_name} start deliberately stuck for {self._test_stuck_time}s"
            )
            time.sleep(self._test_stuck_time)
            logger.info(
                f"Watchdog {self._raw.debug_name} end deliberately stuck for {self._test_stuck_time}s"
            )

        self._counter += 1
```
**EN:** This method implements `feed` on `_WatchdogReal`. It primarily calls `logger.info`, `time.sleep` to complete its work. State updates are written into `self._counter`, `self._test_stuck_triggered`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `_WatchdogReal`）实现了 `feed`。 它主要通过调用 `logger.info`, `time.sleep` 来完成任务。 状态更新主要写入 `self._counter`, `self._test_stuck_triggered`。 实现中使用了条件分支。

### Lines 88-96: Method `_WatchdogReal.disable` / 方法 `_WatchdogReal.disable`
```python
    @contextmanager
    def disable(self):
        assert self._active
        self._active = False
        try:
            yield
        finally:
            assert not self._active
            self._active = True
```
**EN:** This method implements `disable` on `_WatchdogReal`. State updates are written into `self._active`. The implementation relies on error handling.
**CN:** 该方法（属于 `_WatchdogReal`）实现了 `disable`。 状态更新主要写入 `self._active`。 实现中使用了错误处理。

### Lines 99-100: Class `_WatchdogNoop` declaration / 类 `_WatchdogNoop` 声明
```python
class _WatchdogNoop(Watchdog):
    pass
```
**EN:** This class establishes `_WatchdogNoop` as the main container/coordinator for the surrounding logic. It inherits from `Watchdog`.
**CN:** 该类将 `_WatchdogNoop` 定义为周边逻辑的主要封装体或协调者。 它继承自 `Watchdog`。

### Lines 103-103: Class `WatchdogRaw` declaration / 类 `WatchdogRaw` 声明
```python
class WatchdogRaw:
```
**EN:** This class establishes `WatchdogRaw` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `_watchdog_thread`, `_watchdog_once`.
**CN:** 该类将 `WatchdogRaw` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `_watchdog_thread`, `_watchdog_once` 等方法。

### Lines 104-122: Method `WatchdogRaw.__init__` / 方法 `WatchdogRaw.__init__`
```python
    def __init__(
        self,
        debug_name: str,
        get_counter: Callable[[], int],
        is_active: Callable[[], bool],
        watchdog_timeout: float,
        soft: bool = False,
        dump_info: Optional[Callable[[], str]] = None,
    ):
        self.debug_name = debug_name
        self.get_counter = get_counter
        self.is_active = is_active
        self.watchdog_timeout = watchdog_timeout
        self.soft = soft
        self.dump_info = dump_info

        self.parent_process = psutil.Process().parent()
        t = threading.Thread(target=self._watchdog_thread, daemon=True)
        t.start()
```
**EN:** This method implements `__init__` on `WatchdogRaw`. It primarily calls `psutil.Process.parent`, `threading.Thread`, `t.start`, `psutil.Process` to complete its work. State updates are written into `self.debug_name`, `self.get_counter`, `self.is_active`, `self.watchdog_timeout`, `self.soft`, `self.dump_info`.
**CN:** 该方法（属于 `WatchdogRaw`）实现了 `__init__`。 它主要通过调用 `psutil.Process.parent`, `threading.Thread`, `t.start`, `psutil.Process` 来完成任务。 状态更新主要写入 `self.debug_name`, `self.get_counter`, `self.is_active`, `self.watchdog_timeout`, `self.soft`, `self.dump_info`。

### Lines 124-131: Method `WatchdogRaw._watchdog_thread` / 方法 `WatchdogRaw._watchdog_thread`
```python
    def _watchdog_thread(self):
        try:
            while True:
                self._watchdog_once()
        except Exception as e:
            logger.error(
                f"{self.debug_name} watchdog thread crashed: {e}", exc_info=True
            )
```
**EN:** This method implements `_watchdog_thread` on `WatchdogRaw`. It primarily calls `self._watchdog_once`, `logger.error` to complete its work. The implementation relies on iteration, error handling.
**CN:** 该方法（属于 `WatchdogRaw`）实现了 `_watchdog_thread`。 它主要通过调用 `self._watchdog_once`, `logger.error` 来完成任务。 实现中使用了迭代逻辑、错误处理。

### Lines 133-163: Method `WatchdogRaw._watchdog_once` / 方法 `WatchdogRaw._watchdog_once`
```python
    def _watchdog_once(self):
        watchdog_last_counter = 0
        watchdog_last_time = time.perf_counter()

        while True:
            current = time.perf_counter()
            if self.is_active():
                current_counter = self.get_counter()
                if watchdog_last_counter == current_counter:
                    if current > watchdog_last_time + self.watchdog_timeout:
                        break
                else:
                    watchdog_last_counter = current_counter
                    watchdog_last_time = current
            time.sleep(self.watchdog_timeout / 2)

        if self.dump_info is not None and (info_msg := self.dump_info()):
            logger.error(f"{self.debug_name} debug info:\n{info_msg}")

        pyspy_dump_schedulers()
        logger.error(
            f"{self.debug_name} watchdog timeout "
            f"({self.watchdog_timeout=}, {self.soft=})"
        )
        print(file=sys.stderr, flush=True)
        print(file=sys.stdout, flush=True)

        if not self.soft:
            # Wait for some time so that the parent process can print the error.
            time.sleep(5)
            self.parent_process.send_signal(signal.SIGQUIT)
```
**EN:** This method implements `_watchdog_once` on `WatchdogRaw`. It primarily calls `time.perf_counter`, `pyspy_dump_schedulers`, `logger.error`, `print`, `self.is_active`, `time.sleep` to complete its work. State updates are written into `watchdog_last_counter`, `watchdog_last_time`, `current`, `current_counter`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `WatchdogRaw`）实现了 `_watchdog_once`。 它主要通过调用 `time.perf_counter`, `pyspy_dump_schedulers`, `logger.error`, `print`, `self.is_active`, `time.sleep` 来完成任务。 状态更新主要写入 `watchdog_last_counter`, `watchdog_last_time`, `current`, `current_counter`。 实现中使用了条件分支、迭代逻辑。

### Lines 166-175: Class `SubprocessWatchdog` declaration / 类 `SubprocessWatchdog` 声明
```python
class SubprocessWatchdog:
    """Monitors subprocess liveness and triggers SIGQUIT when a crash is detected.

    When a subprocess crashes (e.g., NCCL timeout causing C++ std::terminate()),
    Python exception handlers never run, leaving the main process as a zombie
    service. This watchdog polls subprocess liveness in a daemon thread and
    sends SIGQUIT to trigger proper cleanup.

    See: https://github.com/sgl-project/sglang/issues/18421
    """
```
**EN:** This class establishes `SubprocessWatchdog` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `start`, `stop`, `_monitor_loop`, `_check_processes`.
**CN:** 该类将 `SubprocessWatchdog` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `start`, `stop`, `_monitor_loop`, `_check_processes` 等方法。

### Lines 177-187: Method `SubprocessWatchdog.__init__` / 方法 `SubprocessWatchdog.__init__`
```python
    def __init__(
        self,
        processes: List[Process],
        process_names: Optional[List[str]] = None,
        interval: float = 1.0,
    ):
        self._processes = processes
        self._names = process_names or [f"process_{i}" for i in range(len(processes))]
        self._interval = interval
        self._stop_event = threading.Event()
        self._thread: Optional[threading.Thread] = None
```
**EN:** This method implements `__init__` on `SubprocessWatchdog`. It primarily calls `threading.Event`, `range`, `len` to complete its work. State updates are written into `self._processes`, `self._names`, `self._interval`, `self._stop_event`, `self._thread`.
**CN:** 该方法（属于 `SubprocessWatchdog`）实现了 `__init__`。 它主要通过调用 `threading.Event`, `range`, `len` 来完成任务。 状态更新主要写入 `self._processes`, `self._names`, `self._interval`, `self._stop_event`, `self._thread`。

### Lines 189-195: Method `SubprocessWatchdog.start` / 方法 `SubprocessWatchdog.start`
```python
    def start(self) -> None:
        if self._thread is not None or not self._processes:
            return
        self._thread = threading.Thread(
            target=self._monitor_loop, daemon=True, name="subprocess-watchdog"
        )
        self._thread.start()
```
**EN:** This method implements `start` on `SubprocessWatchdog`. It primarily calls `threading.Thread`, `self._thread.start` to complete its work. State updates are written into `self._thread`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `SubprocessWatchdog`）实现了 `start`。 它主要通过调用 `threading.Thread`, `self._thread.start` 来完成任务。 状态更新主要写入 `self._thread`。 实现中使用了条件分支。

### Lines 197-201: Method `SubprocessWatchdog.stop` / 方法 `SubprocessWatchdog.stop`
```python
    def stop(self) -> None:
        self._stop_event.set()
        if self._thread is not None:
            self._thread.join(timeout=self._interval * 2)
            self._thread = None
```
**EN:** This method implements `stop` on `SubprocessWatchdog`. It primarily calls `self._stop_event.set`, `self._thread.join` to complete its work. State updates are written into `self._thread`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `SubprocessWatchdog`）实现了 `stop`。 它主要通过调用 `self._stop_event.set`, `self._thread.join` 来完成任务。 状态更新主要写入 `self._thread`。 实现中使用了条件分支。

### Lines 203-209: Method `SubprocessWatchdog._monitor_loop` / 方法 `SubprocessWatchdog._monitor_loop`
```python
    def _monitor_loop(self) -> None:
        try:
            while not self._stop_event.wait(self._interval):
                if self._check_processes():
                    return
        except Exception as e:
            logger.error(f"SubprocessWatchdog thread crashed: {e}", exc_info=True)
```
**EN:** This method implements `_monitor_loop` on `SubprocessWatchdog`. It primarily calls `self._stop_event.wait`, `self._check_processes`, `logger.error` to complete its work. The implementation relies on conditional branches, iteration, error handling.
**CN:** 该方法（属于 `SubprocessWatchdog`）实现了 `_monitor_loop`。 它主要通过调用 `self._stop_event.wait`, `self._check_processes`, `logger.error` 来完成任务。 实现中使用了条件分支、迭代逻辑、错误处理。

### Lines 211-223: Method `SubprocessWatchdog._check_processes` / 方法 `SubprocessWatchdog._check_processes`
```python
    def _check_processes(self) -> bool:
        for proc, name in zip(self._processes, self._names):
            if proc.is_alive() or proc.exitcode == 0:
                continue

            logger.error(
                f"Subprocess {name} (pid={proc.pid}) crashed "
                f"with exit code {proc.exitcode}. "
                f"Triggering SIGQUIT for cleanup..."
            )
            os.kill(os.getpid(), signal.SIGQUIT)
            return True
        return False
```
**EN:** This method implements `_check_processes` on `SubprocessWatchdog`. It primarily calls `zip`, `logger.error`, `os.kill`, `proc.is_alive`, `os.getpid` to complete its work. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `SubprocessWatchdog`）实现了 `_check_processes`。 它主要通过调用 `zip`, `logger.error`, `os.kill`, `proc.is_alive`, `os.getpid` 来完成任务。 实现中使用了条件分支、迭代逻辑。

## Key Concepts / 关键概念
- **Classes / 类**: `Watchdog`, `_WatchdogReal`, `_WatchdogNoop`, `WatchdogRaw`, `SubprocessWatchdog`
- **Functions / 函数**: `create`, `feed`, `disable`, `__init__`, `feed`, `disable`, `__init__`, `_watchdog_thread`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.utils.common`
- **External / 外部依赖**: `psutil`
- **Standard library / 标准库**: `__future__`, `logging`, `os`, `signal`, `sys`, `threading`, `time`, `contextlib`, `multiprocessing`, `typing`
