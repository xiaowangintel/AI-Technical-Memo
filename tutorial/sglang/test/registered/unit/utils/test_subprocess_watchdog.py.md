# test_subprocess_watchdog.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/utils/test_subprocess_watchdog.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates subprocess watchdog behavior in SGLang's unit / utils area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 工具 领域中与 subprocess watchdog 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: supporting source context / 辅助源码上下文
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
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 14-14: supporting statements / 辅助语句
```python
"""Tests for SubprocessWatchdog in watchdog.py"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 16-25: module imports and dependencies / 模块导入与依赖
```python
import multiprocessing as mp
import os
import signal
import threading
import time
import unittest.mock

from sglang.srt.utils.watchdog import SubprocessWatchdog
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `multiprocessing`, `os`, `signal`, `threading`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `multiprocessing`, `os`, `signal`, `threading`。

### Lines 27-27: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=9, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 30-31: function healthy worker / 函数 healthy worker
```python
def healthy_worker():
    time.sleep(10)
```
**EN:** This block implements `healthy_worker` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `healthy_worker`，承担模块行为中的一个聚焦逻辑片段。

### Lines 34-35: function crashing worker / 函数 crashing worker
```python
def crashing_worker():
    os._exit(1)
```
**EN:** This block implements `crashing_worker` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `crashing_worker`，承担模块行为中的一个聚焦逻辑片段。

### Lines 38-40: function slow crash worker / 函数 slow crash worker
```python
def slow_crash_worker(delay: float = 0.5):
    time.sleep(delay)
    os._exit(42)
```
**EN:** This block implements `slow_crash_worker` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `slow_crash_worker`，承担模块行为中的一个聚焦逻辑片段。

### Lines 43-44: function noop worker / 函数 noop worker
```python
def noop_worker():
    pass
```
**EN:** This block implements `noop_worker` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `noop_worker`，承担模块行为中的一个聚焦逻辑片段。

### Lines 47-47: class TestSubprocessWatchdog declaration / 类 TestSubprocessWatchdog 声明
```python
class TestSubprocessWatchdog(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 48-62: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.sigquit_triggered = threading.Event()
        self._procs = []
        self._monitor = None

        original_kill = os.kill

        def mock_kill(pid, sig):
            if sig == signal.SIGQUIT:
                self.sigquit_triggered.set()
            else:
                original_kill(pid, sig)

        self._patcher = unittest.mock.patch("os.kill", side_effect=mock_kill)
        self._patcher.start()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 64-71: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        if self._monitor is not None:
            self._monitor.stop()
        self._patcher.stop()
        for p in self._procs:
            if p.is_alive():
                p.terminate()
                p.join(timeout=1)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 73-77: method spawn / 方法 spawn
```python
    def _spawn(self, target, args=()):
        proc = mp.Process(target=target, args=args)
        proc.start()
        self._procs.append(proc)
        return proc
```
**EN:** This block implements `_spawn` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_spawn`，承担模块行为中的一个聚焦逻辑片段。

### Lines 79-88: method watch / 方法 watch
```python
    def _watch(self, procs, names=None, interval=0.1):
        if not isinstance(procs, list):
            procs = [procs]
        self._monitor = SubprocessWatchdog(
            processes=procs,
            process_names=names,
            interval=interval,
        )
        self._monitor.start()
        return self._monitor
```
**EN:** This block implements `_watch` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_watch`，承担模块行为中的一个聚焦逻辑片段。

### Lines 90-94: test case healthy processes no sigquit / 测试用例 healthy processes no sigquit
```python
    def test_healthy_processes_no_sigquit(self):
        proc = self._spawn(healthy_worker)
        self._watch(proc)
        time.sleep(0.5)
        self.assertFalse(self.sigquit_triggered.is_set())
```
**EN:** This test exercises `test_healthy_processes_no_sigquit` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_healthy_processes_no_sigquit`。

### Lines 96-102: test case crashed process triggers sigquit / 测试用例 crashed process triggers sigquit
```python
    def test_crashed_process_triggers_sigquit(self):
        proc = self._spawn(slow_crash_worker, args=(0.2,))
        self._watch(proc)
        self.assertTrue(
            self.sigquit_triggered.wait(timeout=5.0),
            "SIGQUIT was not triggered within timeout",
        )
```
**EN:** This test exercises `test_crashed_process_triggers_sigquit` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_crashed_process_triggers_sigquit`。

### Lines 104-110: test case immediate crash detection / 测试用例 immediate crash detection
```python
    def test_immediate_crash_detection(self):
        proc = self._spawn(crashing_worker)
        self._watch(proc, interval=0.05)
        self.assertTrue(
            self.sigquit_triggered.wait(timeout=5.0),
            "Immediate crash was not detected",
        )
```
**EN:** This test exercises `test_immediate_crash_detection` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_immediate_crash_detection`。

### Lines 112-119: test case multiple processes one crashes / 测试用例 multiple processes one crashes
```python
    def test_multiple_processes_one_crashes(self):
        healthy = self._spawn(healthy_worker)
        crashing = self._spawn(slow_crash_worker, args=(0.2,))
        self._watch([healthy, crashing], names=["healthy", "crashing"])
        self.assertTrue(
            self.sigquit_triggered.wait(timeout=5.0),
            "Crash was not detected when one of multiple processes crashed",
        )
```
**EN:** This test exercises `test_multiple_processes_one_crashes` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_processes_one_crashes`。

### Lines 121-124: test case empty processes list / 测试用例 empty processes list
```python
    def test_empty_processes_list(self):
        self._watch([], interval=0.1)
        time.sleep(0.3)
        self.assertFalse(self.sigquit_triggered.is_set())
```
**EN:** This test exercises `test_empty_processes_list` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_processes_list`。

### Lines 126-134: test case normal exit no sigquit / 测试用例 normal exit no sigquit
```python
    def test_normal_exit_no_sigquit(self):
        proc = self._spawn(noop_worker)
        proc.join(timeout=2)
        self._watch(proc)
        time.sleep(0.3)
        self.assertFalse(
            self.sigquit_triggered.is_set(),
            "SIGQUIT should not be triggered for normal exit (exitcode=0)",
        )
```
**EN:** This test exercises `test_normal_exit_no_sigquit` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_normal_exit_no_sigquit`。

### Lines 137-140: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    import unittest

    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `healthy_worker`: This block implements `healthy_worker` and captures one focused piece of the module's behavior. / 该代码块实现 `healthy_worker`，承担模块行为中的一个聚焦逻辑片段。
- `crashing_worker`: This block implements `crashing_worker` and captures one focused piece of the module's behavior. / 该代码块实现 `crashing_worker`，承担模块行为中的一个聚焦逻辑片段。
- `slow_crash_worker`: This block implements `slow_crash_worker` and captures one focused piece of the module's behavior. / 该代码块实现 `slow_crash_worker`，承担模块行为中的一个聚焦逻辑片段。
- `noop_worker`: This block implements `noop_worker` and captures one focused piece of the module's behavior. / 该代码块实现 `noop_worker`，承担模块行为中的一个聚焦逻辑片段。
- `TestSubprocessWatchdog`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSubprocessWatchdog.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestSubprocessWatchdog.tearDown`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestSubprocessWatchdog._spawn`: This block implements `_spawn` and captures one focused piece of the module's behavior. / 该代码块实现 `_spawn`，承担模块行为中的一个聚焦逻辑片段。
- `TestSubprocessWatchdog._watch`: This block implements `_watch` and captures one focused piece of the module's behavior. / 该代码块实现 `_watch`，承担模块行为中的一个聚焦逻辑片段。
- `TestSubprocessWatchdog.test_healthy_processes_no_sigquit`: This test exercises `test_healthy_processes_no_sigquit` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_healthy_processes_no_sigquit`。
- `TestSubprocessWatchdog.test_crashed_process_triggers_sigquit`: This test exercises `test_crashed_process_triggers_sigquit` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_crashed_process_triggers_sigquit`。
- `TestSubprocessWatchdog.test_immediate_crash_detection`: This test exercises `test_immediate_crash_detection` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_immediate_crash_detection`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `multiprocessing`, `os`, `signal`, `threading`, `time`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.srt.utils.watchdog`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 140
