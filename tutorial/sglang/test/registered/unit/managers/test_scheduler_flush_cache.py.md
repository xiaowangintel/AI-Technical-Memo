# test_scheduler_flush_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/managers/test_scheduler_flush_cache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates scheduler flush cache behavior in SGLang's unit / managers area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / managers 领域中与 scheduler flush cache 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: module imports and dependencies / 模块导入与依赖
```python
import unittest
from unittest.mock import MagicMock, patch

from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import maybe_stub_sgl_kernel
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `unittest.mock`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `unittest.mock`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`。

### Lines 7-7: supporting statements / 辅助语句
```python
maybe_stub_sgl_kernel()
```
**EN:** This block performs supporting work through calls such as `maybe_stub_sgl_kernel`, preparing state for nearby definitions.
**CN:** 该代码块通过 `maybe_stub_sgl_kernel` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 9-10: module imports and dependencies / 模块导入与依赖
```python
from sglang.srt.managers.io_struct import FlushCacheReqInput
from sglang.srt.managers.scheduler import Scheduler
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.srt.managers.io_struct`, `sglang.srt.managers.scheduler`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.srt.managers.io_struct`, `sglang.srt.managers.scheduler`。

### Lines 12-12: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=14, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 15-15: class TestSchedulerFlushCache declaration / 类 TestSchedulerFlushCache 声明
```python
class TestSchedulerFlushCache(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 16-22: method new scheduler / 方法 new scheduler
```python
    def _new_scheduler(self) -> Scheduler:
        scheduler = Scheduler.__new__(Scheduler)
        scheduler._pending_flush = None
        scheduler.send_to_tokenizer = MagicMock()
        scheduler.flush_cache = MagicMock(return_value=True)
        scheduler.is_fully_idle = MagicMock(return_value=False)
        return scheduler
```
**EN:** This block implements `_new_scheduler` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_new_scheduler`，承担模块行为中的一个聚焦逻辑片段。

### Lines 24-34: test case immediate flush no timeout / 测试用例 immediate flush no timeout
```python
    def test_immediate_flush_no_timeout(self):
        """No timeout → flush immediately regardless of idle state."""
        scheduler = self._new_scheduler()
        scheduler.flush_cache.return_value = False

        output = Scheduler.flush_cache_wrapped(
            scheduler, FlushCacheReqInput(timeout_s=None)
        )

        self.assertFalse(output.success)
        scheduler.flush_cache.assert_called_once()
```
**EN:** No timeout → flush immediately regardless of idle state. This test exercises `test_immediate_flush_no_timeout` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** No timeout → flush immediately regardless of idle state. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_immediate_flush_no_timeout`。

### Lines 36-46: test case immediate flush when idle / 测试用例 immediate flush when idle
```python
    def test_immediate_flush_when_idle(self):
        """Positive timeout but already idle → flush immediately."""
        scheduler = self._new_scheduler()
        scheduler.is_fully_idle.return_value = True

        output = Scheduler.flush_cache_wrapped(
            scheduler, FlushCacheReqInput(timeout_s=5.0)
        )

        self.assertTrue(output.success)
        scheduler.flush_cache.assert_called_once()
```
**EN:** Positive timeout but already idle → flush immediately. This test exercises `test_immediate_flush_when_idle` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Positive timeout but already idle → flush immediately. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_immediate_flush_when_idle`。

### Lines 48-59: test case defers when busy / 测试用例 defers when busy
```python
    def test_defers_when_busy(self):
        """Positive timeout + busy → defers, returns None."""
        scheduler = self._new_scheduler()
        req = FlushCacheReqInput(timeout_s=3.0)

        with patch("sglang.srt.managers.scheduler.time.monotonic", return_value=10.0):
            output = Scheduler.flush_cache_wrapped(scheduler, req)

        self.assertIsNone(output)
        pending_req, deadline = scheduler._pending_flush
        self.assertIs(pending_req, req)
        self.assertEqual(deadline, 13.0)
```
**EN:** Positive timeout + busy → defers, returns None. This test exercises `test_defers_when_busy` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Positive timeout + busy → defers, returns None. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_defers_when_busy`。

### Lines 61-73: test case rejects when already pending / 测试用例 rejects when already pending
```python
    def test_rejects_when_already_pending(self):
        """Any new request is rejected while another is pending."""
        scheduler = self._new_scheduler()
        scheduler._pending_flush = (FlushCacheReqInput(timeout_s=10.0), 999.0)

        for timeout in [None, 5.0]:
            output = Scheduler.flush_cache_wrapped(
                scheduler, FlushCacheReqInput(timeout_s=timeout)
            )
            self.assertFalse(output.success)
            self.assertIn("already in progress", output.message)

        scheduler.flush_cache.assert_not_called()
```
**EN:** Any new request is rejected while another is pending. This test exercises `test_rejects_when_already_pending` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Any new request is rejected while another is pending. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rejects_when_already_pending`。

### Lines 75-86: test case pending flush completes on idle / 测试用例 pending flush completes on idle
```python
    def test_pending_flush_completes_on_idle(self):
        scheduler = self._new_scheduler()
        scheduler.is_fully_idle.return_value = True
        req = FlushCacheReqInput(timeout_s=1.0)
        scheduler._pending_flush = (req, 111.0)

        Scheduler._check_pending_flush(scheduler)

        self.assertIsNone(scheduler._pending_flush)
        scheduler.flush_cache.assert_called_once()
        out = scheduler.send_to_tokenizer.send_output.call_args.args[0]
        self.assertTrue(out.success)
```
**EN:** This test exercises `test_pending_flush_completes_on_idle` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pending_flush_completes_on_idle`。

### Lines 88-99: test case pending flush expires on timeout / 测试用例 pending flush expires on timeout
```python
    def test_pending_flush_expires_on_timeout(self):
        scheduler = self._new_scheduler()
        req = FlushCacheReqInput(timeout_s=1.0)
        scheduler._pending_flush = (req, 99.0)

        with patch("sglang.srt.managers.scheduler.time.monotonic", return_value=100.0):
            Scheduler._check_pending_flush(scheduler)

        self.assertIsNone(scheduler._pending_flush)
        scheduler.flush_cache.assert_not_called()
        out = scheduler.send_to_tokenizer.send_output.call_args.args[0]
        self.assertFalse(out.success)
```
**EN:** This test exercises `test_pending_flush_expires_on_timeout` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pending_flush_expires_on_timeout`。

### Lines 101-110: test case pending flush survives before deadline / 测试用例 pending flush survives before deadline
```python
    def test_pending_flush_survives_before_deadline(self):
        scheduler = self._new_scheduler()
        req = FlushCacheReqInput(timeout_s=5.0)
        scheduler._pending_flush = (req, 101.0)

        with patch("sglang.srt.managers.scheduler.time.monotonic", return_value=100.0):
            Scheduler._check_pending_flush(scheduler)

        self.assertIsNotNone(scheduler._pending_flush)
        scheduler.send_to_tokenizer.send_output.assert_not_called()
```
**EN:** This test exercises `test_pending_flush_survives_before_deadline` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pending_flush_survives_before_deadline`。

### Lines 113-114: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestSchedulerFlushCache`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSchedulerFlushCache._new_scheduler`: This block implements `_new_scheduler` and captures one focused piece of the module's behavior. / 该代码块实现 `_new_scheduler`，承担模块行为中的一个聚焦逻辑片段。
- `TestSchedulerFlushCache.test_immediate_flush_no_timeout`: No timeout → flush immediately regardless of idle state. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_immediate_flush_no_timeout`。
- `TestSchedulerFlushCache.test_immediate_flush_when_idle`: Positive timeout but already idle → flush immediately. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_immediate_flush_when_idle`。
- `TestSchedulerFlushCache.test_defers_when_busy`: Positive timeout + busy → defers, returns None. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_defers_when_busy`。
- `TestSchedulerFlushCache.test_rejects_when_already_pending`: Any new request is rejected while another is pending. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rejects_when_already_pending`。
- `TestSchedulerFlushCache.test_pending_flush_completes_on_idle`: This test exercises `test_pending_flush_completes_on_idle` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pending_flush_completes_on_idle`。
- `TestSchedulerFlushCache.test_pending_flush_expires_on_timeout`: This test exercises `test_pending_flush_expires_on_timeout` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pending_flush_expires_on_timeout`。
- `TestSchedulerFlushCache.test_pending_flush_survives_before_deadline`: This test exercises `test_pending_flush_survives_before_deadline` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pending_flush_survives_before_deadline`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.test_utils`, `sglang.srt.managers.io_struct`, `sglang.srt.managers.scheduler`

- **Total lines / 总行数**: 114
