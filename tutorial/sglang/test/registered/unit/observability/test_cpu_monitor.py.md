# test_cpu_monitor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/observability/test_cpu_monitor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates cpu monitor behavior in SGLang's unit / observability area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / observability 领域中与 cpu monitor 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: module imports and dependencies / 模块导入与依赖
```python
import threading
import time
import unittest
from collections import namedtuple
from unittest.mock import MagicMock, patch

from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `threading`, `time`, `unittest`, `collections`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `threading`, `time`, `unittest`, `collections`。

### Lines 9-9: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=60, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 12-12: class TestCpuMonitor declaration / 类 TestCpuMonitor 声明
```python
class TestCpuMonitor(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 13-37: test case cpu monitor / 测试用例 cpu monitor
```python
    def test_cpu_monitor(self):
        from prometheus_client import REGISTRY

        from sglang.srt.observability.cpu_monitor import start_cpu_monitor_thread

        thread = start_cpu_monitor_thread("test", interval=0.1)
        self.assertTrue(thread.is_alive())
        self.assertTrue(thread.daemon)

        end_time = time.monotonic() + 0.3
        while time.monotonic() < end_time:
            _ = sum(i * i for i in range(1000))
        time.sleep(0.2)

        value = None
        for metric in REGISTRY.collect():
            for sample in metric.samples:
                if (
                    sample.name == "sglang:process_cpu_seconds_total"
                    and sample.labels.get("component") == "test"
                ):
                    value = sample.value
        print(f"sglang:process_cpu_seconds_total = {value}")
        self.assertIsNotNone(value)
        self.assertGreater(value, 0)
```
**EN:** This test exercises `test_cpu_monitor` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_cpu_monitor`。

### Lines 40-40: class TestCpuMonitorMocked declaration / 类 TestCpuMonitorMocked 声明
```python
class TestCpuMonitorMocked(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 41-41: supporting statements / 辅助语句
```python
    """Fast, deterministic tests for start_cpu_monitor_thread using mocks."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 43-96: test case delta calculation over two iterations / 测试用例 delta calculation over two iterations
```python
    @patch("prometheus_client.Counter")
    @patch("sglang.srt.observability.cpu_monitor.psutil.Process")
    @patch("sglang.srt.observability.cpu_monitor.time.sleep")
    def test_delta_calculation_over_two_iterations(
        self, mock_sleep, MockProcess, MockCounter
    ):
        """Verify delta=(user_diff+system_diff) and last_times update across iterations."""
        from sglang.srt.observability.cpu_monitor import start_cpu_monitor_thread

        CpuTimes = namedtuple("CpuTimes", ["user", "system"])
        mock_process = MockProcess.return_value
        mock_process.cpu_times.side_effect = [
            CpuTimes(user=1.0, system=0.5),  # initial (L18)
            CpuTimes(user=2.5, system=1.0),  # iteration 1 (L22)
            CpuTimes(user=4.0, system=2.0),  # iteration 2 (L22)
        ]

        # Allow 2 loop iterations, then stop the thread.
        # Override threading.excepthook to suppress the pytest warning from
        # the intentional exception used to terminate the monitor loop.
        remaining = [2]
        orig_hook = threading.excepthook

        def controlled_sleep(seconds):
            if remaining[0] <= 0:
                raise SystemExit
            remaining[0] -= 1

        mock_sleep.side_effect = controlled_sleep
        threading.excepthook = lambda args: None

        mock_labeled = MagicMock()
        MockCounter.return_value.labels.return_value = mock_labeled

        thread = start_cpu_monitor_thread("my_component", interval=3.0)
        thread.join(timeout=1.0)
        threading.excepthook = orig_hook

        # Thread is daemon (L29)
        self.assertTrue(thread.daemon)

        # Sleep called with correct interval (L21)
        mock_sleep.assert_called_with(3.0)

        # Counter labeled with component (L26)
        MockCounter.return_value.labels.assert_called_with(component="my_component")

        # Delta calculation (L23-24) and counter increment (L26)
        inc_calls = mock_labeled.inc.call_args_list
        self.assertEqual(len(inc_calls), 2)
        # Iteration 1: (2.5 - 1.0) + (1.0 - 0.5) = 2.0
        self.assertAlmostEqual(inc_calls[0].args[0], 2.0)
        # Iteration 2: (4.0 - 2.5) + (2.0 - 1.0) = 2.5 (proves last_times updated)
        self.assertAlmostEqual(inc_calls[1].args[0], 2.5)
```
**EN:** Verify delta=(user_diff+system_diff) and last_times update across iterations. This test exercises `test_delta_calculation_over_two_iterations` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Verify delta=(user_diff+system_diff) and last_times update across iterations. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_delta_calculation_over_two_iterations`。

### Lines 99-100: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestCpuMonitor`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestCpuMonitorMocked`: Fast, deterministic tests for start_cpu_monitor_thread using mocks. / 用于组织相关测试、夹具或辅助方法。
- `TestCpuMonitor.test_cpu_monitor`: This test exercises `test_cpu_monitor` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_cpu_monitor`。
- `TestCpuMonitorMocked.test_delta_calculation_over_two_iterations`: Verify delta=(user_diff+system_diff) and last_times update across iterations. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_delta_calculation_over_two_iterations`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `threading`, `time`, `unittest`, `collections`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 100
