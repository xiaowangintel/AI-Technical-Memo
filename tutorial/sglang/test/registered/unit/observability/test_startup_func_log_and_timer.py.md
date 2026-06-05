# test_startup_func_log_and_timer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/observability/test_startup_func_log_and_timer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates startup func log and timer behavior in SGLang's unit / observability area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / observability 领域中与 startup func log and timer 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for startup_func_log_and_timer.py — no server, no model loading."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-3: module imports and dependencies / 模块导入与依赖
```python
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.test.ci.ci_register`。

### Lines 5-5: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=6, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 7-18: module imports and dependencies / 模块导入与依赖
```python
import unittest
from unittest.mock import MagicMock, patch

import sglang.srt.observability.startup_func_log_and_timer as mod
from sglang.srt.observability.startup_func_log_and_timer import (
    enable_startup_timer,
    get_max_duration,
    reset_startup_timers,
    set_startup_metric,
    startup_timer,
    time_startup_latency,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `unittest.mock`, `sglang.srt.observability.startup_func_log_and_timer`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `unittest.mock`, `sglang.srt.observability.startup_func_log_and_timer`。

### Lines 21-21: class TestStartupFuncLogAndTimer declaration / 类 TestStartupFuncLogAndTimer 声明
```python
class TestStartupFuncLogAndTimer(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 22-25: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.orig_enable = mod.enable_startup_metrics
        self.orig_gauge = mod.STARTUP_LATENCY_SECONDS
        mod._max_durations.clear()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 27-30: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        mod.enable_startup_metrics = self.orig_enable
        mod.STARTUP_LATENCY_SECONDS = self.orig_gauge
        mod._max_durations.clear()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 32-37: test case enable startup timer / 测试用例 enable startup timer
```python
    @patch("prometheus_client.Gauge")
    def test_enable_startup_timer(self, MockGauge):
        enable_startup_timer()
        self.assertTrue(mod.enable_startup_metrics)
        self.assertIs(mod.STARTUP_LATENCY_SECONDS, MockGauge.return_value)
        MockGauge.assert_called_once()
```
**EN:** This test exercises `test_enable_startup_timer` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_enable_startup_timer`。

### Lines 39-44: test case reset and get max duration / 测试用例 reset and get max duration
```python
    def test_reset_and_get_max_duration(self):
        mod._max_durations["ctx"] = 5.0
        self.assertAlmostEqual(get_max_duration("ctx"), 5.0)
        self.assertIsNone(get_max_duration("nonexistent"))
        reset_startup_timers()
        self.assertIsNone(get_max_duration("ctx"))
```
**EN:** This test exercises `test_reset_and_get_max_duration` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reset_and_get_max_duration`。

### Lines 46-50: test case set startup metric disabled / 测试用例 set startup metric disabled
```python
    def test_set_startup_metric_disabled(self):
        """When metrics disabled, returns early without tracking max."""
        mod.enable_startup_metrics = False
        set_startup_metric("ctx", 1.0)
        self.assertIsNone(get_max_duration("ctx"))
```
**EN:** When metrics disabled, returns early without tracking max. This test exercises `test_set_startup_metric_disabled` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When metrics disabled, returns early without tracking max. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_set_startup_metric_disabled`。

### Lines 52-66: test case set startup metric enabled / 测试用例 set startup metric enabled
```python
    def test_set_startup_metric_enabled(self):
        """Tracks max and updates gauge when enabled."""
        mock_gauge = MagicMock()
        mod.enable_startup_metrics = True
        mod.STARTUP_LATENCY_SECONDS = mock_gauge

        set_startup_metric("ctx", 1.0)
        self.assertAlmostEqual(get_max_duration("ctx"), 1.0)
        mock_gauge.labels.assert_called_with(context="ctx")

        # Lower value → not updated
        mock_gauge.reset_mock()
        set_startup_metric("ctx", 0.5)
        self.assertAlmostEqual(get_max_duration("ctx"), 1.0)
        mock_gauge.labels().set.assert_not_called()
```
**EN:** Tracks max and updates gauge when enabled. This test exercises `test_set_startup_metric_enabled` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Tracks max and updates gauge when enabled. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_set_startup_metric_enabled`。

### Lines 68-72: test case set startup metric no log / 测试用例 set startup metric no log
```python
    def test_set_startup_metric_no_log(self):
        mod.enable_startup_metrics = False
        with patch.object(mod.logger, "info") as mock_log:
            set_startup_metric("ctx", 1.0, should_log=False)
            mock_log.assert_not_called()
```
**EN:** This test exercises `test_set_startup_metric_no_log` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_set_startup_metric_no_log`。

### Lines 74-77: test case startup timer basic / 测试用例 startup timer basic
```python
    def test_startup_timer_basic(self):
        with startup_timer("block"):
            pass
        self.assertGreaterEqual(get_max_duration("block"), 0.0)
```
**EN:** This test exercises `test_startup_timer_basic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_startup_timer_basic`。

### Lines 79-88: test case startup timer with gauge / 测试用例 startup timer with gauge
```python
    def test_startup_timer_with_gauge(self):
        """Gauge updated when metrics enabled and log_only=False."""
        mock_gauge = MagicMock()
        mod.enable_startup_metrics = True
        mod.STARTUP_LATENCY_SECONDS = mock_gauge

        with startup_timer("block"):
            pass
        mock_gauge.labels.assert_called_with(context="block")
        mock_gauge.labels().set.assert_called_once()
```
**EN:** Gauge updated when metrics enabled and log_only=False. This test exercises `test_startup_timer_with_gauge` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Gauge updated when metrics enabled and log_only=False. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_startup_timer_with_gauge`。

### Lines 90-99: test case startup timer log only / 测试用例 startup timer log only
```python
    def test_startup_timer_log_only(self):
        """log_only=True skips gauge but still tracks max."""
        mock_gauge = MagicMock()
        mod.enable_startup_metrics = True
        mod.STARTUP_LATENCY_SECONDS = mock_gauge

        with startup_timer("block", log_only=True):
            pass
        mock_gauge.labels.assert_not_called()
        self.assertIsNotNone(get_max_duration("block"))
```
**EN:** log_only=True skips gauge but still tracks max. This test exercises `test_startup_timer_log_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** log_only=True skips gauge but still tracks max. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_startup_timer_log_only`。

### Lines 101-109: test case decorator direct / 测试用例 decorator direct
```python
    def test_decorator_direct(self):
        """Direct decorator @time_startup_latency preserves return value."""

        @time_startup_latency
        def add(a, b):
            return a + b

        self.assertEqual(add(2, 3), 5)
        self.assertIsNotNone(get_max_duration("add"))
```
**EN:** Direct decorator @time_startup_latency preserves return value. This test exercises `test_decorator_direct` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Direct decorator @time_startup_latency preserves return value. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decorator_direct`。

### Lines 111-122: test case decorator factory with gauge / 测试用例 decorator factory with gauge
```python
    def test_decorator_factory_with_gauge(self):
        """Factory decorator with custom name, gauge updated."""
        mock_gauge = MagicMock()
        mod.enable_startup_metrics = True
        mod.STARTUP_LATENCY_SECONDS = mock_gauge

        @time_startup_latency(name="custom_op")
        def add(a, b):
            return a + b

        self.assertEqual(add(2, 3), 5)
        mock_gauge.labels.assert_called_with(context="custom_op")
```
**EN:** Factory decorator with custom name, gauge updated. This test exercises `test_decorator_factory_with_gauge` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Factory decorator with custom name, gauge updated. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decorator_factory_with_gauge`。

### Lines 124-136: test case decorator log only / 测试用例 decorator log only
```python
    def test_decorator_log_only(self):
        """log_only=True skips gauge but still tracks max."""
        mock_gauge = MagicMock()
        mod.enable_startup_metrics = True
        mod.STARTUP_LATENCY_SECONDS = mock_gauge

        @time_startup_latency(log_only=True)
        def add(a, b):
            return a + b

        self.assertEqual(add(2, 3), 5)
        mock_gauge.labels.assert_not_called()
        self.assertIsNotNone(get_max_duration("add"))
```
**EN:** log_only=True skips gauge but still tracks max. This test exercises `test_decorator_log_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** log_only=True skips gauge but still tracks max. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decorator_log_only`。

### Lines 139-140: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestStartupFuncLogAndTimer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestStartupFuncLogAndTimer.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestStartupFuncLogAndTimer.tearDown`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestStartupFuncLogAndTimer.test_enable_startup_timer`: This test exercises `test_enable_startup_timer` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_enable_startup_timer`。
- `TestStartupFuncLogAndTimer.test_reset_and_get_max_duration`: This test exercises `test_reset_and_get_max_duration` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reset_and_get_max_duration`。
- `TestStartupFuncLogAndTimer.test_set_startup_metric_disabled`: When metrics disabled, returns early without tracking max. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_set_startup_metric_disabled`。
- `TestStartupFuncLogAndTimer.test_set_startup_metric_enabled`: Tracks max and updates gauge when enabled. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_set_startup_metric_enabled`。
- `TestStartupFuncLogAndTimer.test_set_startup_metric_no_log`: This test exercises `test_set_startup_metric_no_log` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_set_startup_metric_no_log`。
- `TestStartupFuncLogAndTimer.test_startup_timer_basic`: This test exercises `test_startup_timer_basic` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_startup_timer_basic`。
- `TestStartupFuncLogAndTimer.test_startup_timer_with_gauge`: Gauge updated when metrics enabled and log_only=False. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_startup_timer_with_gauge`。
- `TestStartupFuncLogAndTimer.test_startup_timer_log_only`: log_only=True skips gauge but still tracks max. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_startup_timer_log_only`。
- `TestStartupFuncLogAndTimer.test_decorator_direct`: Direct decorator @time_startup_latency preserves return value. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decorator_direct`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.srt.observability.startup_func_log_and_timer`

- **Total lines / 总行数**: 140
