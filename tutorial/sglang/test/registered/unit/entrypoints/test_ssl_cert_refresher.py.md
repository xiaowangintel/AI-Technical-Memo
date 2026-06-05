# test_ssl_cert_refresher.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/entrypoints/test_ssl_cert_refresher.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates ssl cert refresher behavior in SGLang's unit / entrypoints area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 入口 领域中与 ssl cert refresher 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: module imports and dependencies / 模块导入与依赖
```python
import asyncio
import os
import tempfile
import unittest
from unittest.mock import MagicMock

from sglang.srt.entrypoints.ssl_utils import SSLCertRefresher
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `asyncio`, `os`, `tempfile`, `unittest`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `asyncio`, `os`, `tempfile`, `unittest`。

### Lines 11-11: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=14, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 14-20: function make temp pem / 函数 make temp pem
```python
def _make_temp_pem(content: bytes) -> str:
    """Create a temporary PEM file and return its path."""
    f = tempfile.NamedTemporaryFile(suffix=".pem", delete=False)
    f.write(content)
    f.flush()
    f.close()
    return f.name
```
**EN:** Create a temporary PEM file and return its path. This block implements `_make_temp_pem` and captures one focused piece of the module's behavior.
**CN:** Create a temporary PEM file and return its path. 该代码块实现 `_make_temp_pem`，承担模块行为中的一个聚焦逻辑片段。

### Lines 23-23: class TestSSLCertRefresher declaration / 类 TestSSLCertRefresher 声明
```python
class TestSSLCertRefresher(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 24-24: supporting statements / 辅助语句
```python
    """Tests for the SSLCertRefresher class."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 26-28: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        super().setUp()
        self._temp_files: list[str] = []
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 30-36: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        for path in self._temp_files:
            try:
                os.unlink(path)
            except OSError:
                pass
        super().tearDown()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 38-41: method track / 方法 track
```python
    def _track(self, path: str) -> str:
        """Register a temp file for cleanup."""
        self._temp_files.append(path)
        return path
```
**EN:** Register a temp file for cleanup. This block implements `_track` and captures one focused piece of the module's behavior.
**CN:** Register a temp file for cleanup. 该代码块实现 `_track`，承担模块行为中的一个聚焦逻辑片段。

### Lines 43-49: method run async / 方法 run async
```python
    def _run_async(self, coro):
        """Helper to run an async coroutine in tests."""
        loop = asyncio.new_event_loop()
        try:
            return loop.run_until_complete(coro)
        finally:
            loop.close()
```
**EN:** Helper to run an async coroutine in tests. This block implements `_run_async` and captures one focused piece of the module's behavior.
**CN:** Helper to run an async coroutine in tests. 该代码块实现 `_run_async`，承担模块行为中的一个聚焦逻辑片段。

### Lines 51-69: test case reload cert key on file change / 测试用例 reload cert key on file change
```python
    def test_reload_cert_key_on_file_change(self):
        """SSLCertRefresher calls load_cert_chain when cert/key files change."""
        mock_ctx = MagicMock()
        cert_path = self._track(_make_temp_pem(b"CERT_V1"))
        key_path = self._track(_make_temp_pem(b"KEY_V1"))

        async def _test():
            refresher = SSLCertRefresher(mock_ctx, key_path, cert_path)
            await asyncio.sleep(0.3)

            with open(cert_path, "w") as f:
                f.write("CERT_V2")

            await asyncio.sleep(1.5)
            refresher.stop()
            return mock_ctx

        result_ctx = self._run_async(_test())
        result_ctx.load_cert_chain.assert_called_with(cert_path, key_path)
```
**EN:** SSLCertRefresher calls load_cert_chain when cert/key files change. This test exercises `test_reload_cert_key_on_file_change` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** SSLCertRefresher calls load_cert_chain when cert/key files change. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reload_cert_key_on_file_change`。

### Lines 71-90: test case reload ca on file change / 测试用例 reload ca on file change
```python
    def test_reload_ca_on_file_change(self):
        """SSLCertRefresher calls load_verify_locations when CA file changes."""
        mock_ctx = MagicMock()
        cert_path = self._track(_make_temp_pem(b"CERT"))
        key_path = self._track(_make_temp_pem(b"KEY"))
        ca_path = self._track(_make_temp_pem(b"CA_V1"))

        async def _test():
            refresher = SSLCertRefresher(mock_ctx, key_path, cert_path, ca_path)
            await asyncio.sleep(0.3)

            with open(ca_path, "w") as f:
                f.write("CA_V2")

            await asyncio.sleep(1.5)
            refresher.stop()
            return mock_ctx

        result_ctx = self._run_async(_test())
        result_ctx.load_verify_locations.assert_called_with(ca_path)
```
**EN:** SSLCertRefresher calls load_verify_locations when CA file changes. This test exercises `test_reload_ca_on_file_change` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** SSLCertRefresher calls load_verify_locations when CA file changes. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reload_ca_on_file_change`。

### Lines 92-111: test case stop cancels tasks / 测试用例 stop cancels tasks
```python
    def test_stop_cancels_tasks(self):
        """Calling stop() prevents further reloads."""
        mock_ctx = MagicMock()
        cert_path = self._track(_make_temp_pem(b"CERT"))
        key_path = self._track(_make_temp_pem(b"KEY"))

        async def _test():
            refresher = SSLCertRefresher(mock_ctx, key_path, cert_path)
            await asyncio.sleep(0.2)

            refresher.stop()

            with open(cert_path, "w") as f:
                f.write("CERT_AFTER_STOP")

            await asyncio.sleep(1.0)
            return mock_ctx

        result_ctx = self._run_async(_test())
        result_ctx.load_cert_chain.assert_not_called()
```
**EN:** Calling stop() prevents further reloads. This test exercises `test_stop_cancels_tasks` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Calling stop() prevents further reloads. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stop_cancels_tasks`。

### Lines 113-124: test case no ca watcher when ca not provided / 测试用例 no ca watcher when ca not provided
```python
    def test_no_ca_watcher_when_ca_not_provided(self):
        """No CA watcher task is created when ca_path is None."""
        mock_ctx = MagicMock()
        cert_path = self._track(_make_temp_pem(b"CERT"))
        key_path = self._track(_make_temp_pem(b"KEY"))

        async def _test():
            refresher = SSLCertRefresher(mock_ctx, key_path, cert_path)
            self.assertEqual(len(refresher._tasks), 1)
            refresher.stop()

        self._run_async(_test())
```
**EN:** No CA watcher task is created when ca_path is None. This test exercises `test_no_ca_watcher_when_ca_not_provided` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** No CA watcher task is created when ca_path is None. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_ca_watcher_when_ca_not_provided`。

### Lines 126-138: test case ca watcher created when ca provided / 测试用例 ca watcher created when ca provided
```python
    def test_ca_watcher_created_when_ca_provided(self):
        """A CA watcher task is created when ca_path is provided."""
        mock_ctx = MagicMock()
        cert_path = self._track(_make_temp_pem(b"CERT"))
        key_path = self._track(_make_temp_pem(b"KEY"))
        ca_path = self._track(_make_temp_pem(b"CA"))

        async def _test():
            refresher = SSLCertRefresher(mock_ctx, key_path, cert_path, ca_path)
            self.assertEqual(len(refresher._tasks), 2)
            refresher.stop()

        self._run_async(_test())
```
**EN:** A CA watcher task is created when ca_path is provided. This test exercises `test_ca_watcher_created_when_ca_provided` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** A CA watcher task is created when ca_path is provided. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ca_watcher_created_when_ca_provided`。

### Lines 140-161: test case reload error does not crash / 测试用例 reload error does not crash
```python
    def test_reload_error_does_not_crash(self):
        """A reload error is logged but doesn't crash the watcher."""
        mock_ctx = MagicMock()
        mock_ctx.load_cert_chain.side_effect = Exception("bad cert")
        cert_path = self._track(_make_temp_pem(b"CERT"))
        key_path = self._track(_make_temp_pem(b"KEY"))

        async def _test():
            refresher = SSLCertRefresher(mock_ctx, key_path, cert_path)
            await asyncio.sleep(0.3)

            with open(cert_path, "w") as f:
                f.write("BAD_CERT")

            await asyncio.sleep(1.5)

            for task in refresher._tasks:
                self.assertFalse(task.done())

            refresher.stop()

        self._run_async(_test())
```
**EN:** A reload error is logged but doesn't crash the watcher. This test exercises `test_reload_error_does_not_crash` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** A reload error is logged but doesn't crash the watcher. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reload_error_does_not_crash`。

### Lines 164-165: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_make_temp_pem`: Create a temporary PEM file and return its path. / 该代码块实现 `_make_temp_pem`，承担模块行为中的一个聚焦逻辑片段。
- `TestSSLCertRefresher`: Tests for the SSLCertRefresher class. / 用于组织相关测试、夹具或辅助方法。
- `TestSSLCertRefresher.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestSSLCertRefresher.tearDown`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestSSLCertRefresher._track`: Register a temp file for cleanup. / 该代码块实现 `_track`，承担模块行为中的一个聚焦逻辑片段。
- `TestSSLCertRefresher._run_async`: Helper to run an async coroutine in tests. / 该代码块实现 `_run_async`，承担模块行为中的一个聚焦逻辑片段。
- `TestSSLCertRefresher.test_reload_cert_key_on_file_change`: SSLCertRefresher calls load_cert_chain when cert/key files change. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reload_cert_key_on_file_change`。
- `TestSSLCertRefresher.test_reload_ca_on_file_change`: SSLCertRefresher calls load_verify_locations when CA file changes. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reload_ca_on_file_change`。
- `TestSSLCertRefresher.test_stop_cancels_tasks`: Calling stop() prevents further reloads. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stop_cancels_tasks`。
- `TestSSLCertRefresher.test_no_ca_watcher_when_ca_not_provided`: No CA watcher task is created when ca_path is None. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_ca_watcher_when_ca_not_provided`。
- `TestSSLCertRefresher.test_ca_watcher_created_when_ca_provided`: A CA watcher task is created when ca_path is provided. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ca_watcher_created_when_ca_provided`。
- `TestSSLCertRefresher.test_reload_error_does_not_crash`: A reload error is logged but doesn't crash the watcher. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reload_error_does_not_crash`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `os`, `tempfile`, `unittest`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.ssl_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 165
