# test_request_logger.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/utils/test_request_logger.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates request logger behavior in SGLang's utils area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 工具 领域中与 request logger 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: module imports and dependencies / 模块导入与依赖
```python
import io
import json
import os
import tempfile
import time
import unittest
from pathlib import Path

import requests

from sglang.srt.constants import HEALTH_CHECK_RID_PREFIX
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `io`, `json`, `os`, `tempfile`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `io`, `json`, `os`, `tempfile`。

### Lines 21-27: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=120, suite="nightly-1-gpu", nightly=True)
register_amd_ci(est_time=120, suite="nightly-amd-1-gpu", nightly=True)

TEST_ROUTING_KEY = "test-routing-key-12345"
TEST_CUSTOM_HEADER_NAME = "X-Test-Header"
TEST_CUSTOM_HEADER_VALUE = "test-header-value-67890"
TEST_MODEL_NAME = "Qwen/Qwen3-0.6B"
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 30-30: class BaseTestRequestLogger declaration / 类 BaseTestRequestLogger 声明
```python
class BaseTestRequestLogger:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 31-33: class-level constants and configuration for `BaseTestRequestLogger` / 类级常量与配置
```python
    log_requests_format = None
    env_vars: dict[str, str] = {}  # Env vars to set before server launch
    request_headers: dict[str, str] = {"X-SMG-Routing-Key": TEST_ROUTING_KEY}
```
**EN:** This block defines shared names such as `log_requests_format`, `env_vars`, `request_headers`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `log_requests_format`, `env_vars`, `request_headers` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 35-64: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls._temp_dir_obj = tempfile.TemporaryDirectory()
        cls.temp_dir = cls._temp_dir_obj.name
        cls.stdout = io.StringIO()
        cls.stderr = io.StringIO()
        other_args = [
            "--log-requests",
            "--log-requests-level",
            "2",
            "--log-requests-format",
            cls.log_requests_format,
            "--skip-server-warmup",
            "--log-requests-target",
            "stdout",
            cls.temp_dir,
        ]
        # Set env vars and save old values for restoration
        cls._old_env_vars = {}
        for key, value in cls.env_vars.items():
            cls._old_env_vars[key] = os.environ.get(key)
            os.environ[key] = value

        cls.process = popen_launch_server(
            TEST_MODEL_NAME,
            DEFAULT_URL_FOR_TEST,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=other_args,
            return_stdout_stderr=(cls.stdout, cls.stderr),
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 66-77: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
        cls.stdout.close()
        cls.stderr.close()
        cls._temp_dir_obj.cleanup()
        # Restore env vars
        for key, old_value in cls._old_env_vars.items():
            if old_value is None:
                os.environ.pop(key, None)
            else:
                os.environ[key] = old_value
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 79-80: method verify logs / 方法 verify logs
```python
    def _verify_logs(self, content: str, source_name: str):
        raise NotImplementedError
```
**EN:** This block implements `_verify_logs` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_verify_logs`，承担模块行为中的一个聚焦逻辑片段。

### Lines 82-83: method verify openai logs / 方法 verify openai logs
```python
    def _verify_openai_logs(self, content: str, source_name: str):
        raise NotImplementedError
```
**EN:** This block implements `_verify_openai_logs` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_verify_openai_logs`，承担模块行为中的一个聚焦逻辑片段。

### Lines 85-106: method wait until verified / 方法 wait until verified
```python
    def _wait_until_verified(
        self,
        verify_fn,
        get_content_fn,
        source_name: str,
        timeout: float = 10.0,
        interval: float = 0.1,
    ):
        deadline = time.time() + timeout
        last_error = None

        while time.time() < deadline:
            content = get_content_fn()
            try:
                verify_fn(content, source_name)
                return
            except AssertionError as err:
                last_error = err
                time.sleep(interval)

        if last_error is not None:
            raise last_error
```
**EN:** This block implements `_wait_until_verified` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_wait_until_verified`，承担模块行为中的一个聚焦逻辑片段。

### Lines 108-131: test case logging / 测试用例 logging
```python
    def test_logging(self):
        response = requests.post(
            DEFAULT_URL_FOR_TEST + "/generate",
            json={
                "text": "Hello",
                "sampling_params": {"max_new_tokens": 8, "temperature": 0},
            },
            headers=self.request_headers,
            timeout=30,
        )
        self.assertEqual(response.status_code, 200)
        self._wait_until_verified(
            self._verify_logs,
            lambda: self.stdout.getvalue() + self.stderr.getvalue(),
            "stdout",
        )
        self._wait_until_verified(
            self._verify_logs,
            lambda: "".join(f.read_text() for f in Path(self.temp_dir).glob("*.log")),
            "log files",
        )

        log_files = list(Path(self.temp_dir).glob("*.log"))
        self.assertGreater(len(log_files), 0, "No log files found in temp directory")
```
**EN:** This test exercises `test_logging` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_logging`。

### Lines 133-158: test case openai chat logging / 测试用例 openai chat logging
```python
    def test_openai_chat_logging(self):
        response = requests.post(
            DEFAULT_URL_FOR_TEST + "/v1/chat/completions",
            json={
                "model": TEST_MODEL_NAME,
                "messages": [{"role": "user", "content": "hello request logger"}],
                "max_tokens": 8,
                "temperature": 0,
            },
            headers=self.request_headers,
            timeout=30,
        )
        self.assertEqual(response.status_code, 200)
        self._wait_until_verified(
            self._verify_openai_logs,
            lambda: self.stdout.getvalue() + self.stderr.getvalue(),
            "stdout",
        )
        self._wait_until_verified(
            self._verify_openai_logs,
            lambda: "".join(f.read_text() for f in Path(self.temp_dir).glob("*.log")),
            "log files",
        )

        log_files = list(Path(self.temp_dir).glob("*.log"))
        self.assertGreater(len(log_files), 0, "No log files found in temp directory")
```
**EN:** This test exercises `test_openai_chat_logging` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_openai_chat_logging`。

### Lines 161-161: class TestRequestLoggerText declaration / 类 TestRequestLoggerText 声明
```python
class TestRequestLoggerText(BaseTestRequestLogger, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `BaseTestRequestLogger`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `BaseTestRequestLogger`, `CustomTestCase`。

### Lines 162-162: class-level constants and configuration for `TestRequestLoggerText` / 类级常量与配置
```python
    log_requests_format = "text"
```
**EN:** This block defines shared names such as `log_requests_format`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `log_requests_format` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 164-172: method verify logs / 方法 verify logs
```python
    def _verify_logs(self, content: str, source_name: str):
        self.assertIn("Receive:", content, f"'Receive:' not found in {source_name}")
        self.assertIn("Finish:", content, f"'Finish:' not found in {source_name}")
        self.assertIn(
            TEST_ROUTING_KEY, content, f"Routing key not found in {source_name}"
        )
        self.assertIn(
            "x-smg-routing-key", content, f"Header name not found in {source_name}"
        )
```
**EN:** This block implements `_verify_logs` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_verify_logs`，承担模块行为中的一个聚焦逻辑片段。

### Lines 174-183: method verify openai logs / 方法 verify openai logs
```python
    def _verify_openai_logs(self, content: str, source_name: str):
        self.assertIn(
            "Receive OpenAI:", content, f"OpenAI receive log not found in {source_name}"
        )
        self.assertIn("'messages':", content, f"Messages not found in {source_name}")
        self.assertIn(
            "hello request logger",
            content,
            f"OpenAI user prompt not found in {source_name}",
        )
```
**EN:** This block implements `_verify_openai_logs` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_verify_openai_logs`，承担模块行为中的一个聚焦逻辑片段。

### Lines 186-186: class TestRequestLoggerJson declaration / 类 TestRequestLoggerJson 声明
```python
class TestRequestLoggerJson(BaseTestRequestLogger, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `BaseTestRequestLogger`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `BaseTestRequestLogger`, `CustomTestCase`。

### Lines 187-187: class-level constants and configuration for `TestRequestLoggerJson` / 类级常量与配置
```python
    log_requests_format = "json"
```
**EN:** This block defines shared names such as `log_requests_format`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `log_requests_format` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 189-226: method verify logs / 方法 verify logs
```python
    def _verify_logs(self, content: str, source_name: str):
        received_found = False
        finished_found = False
        for line in content.splitlines():
            idx = line.find("{")
            if idx == -1:
                continue
            try:
                data = json.loads(line[idx:])
            except json.JSONDecodeError:
                continue

            rid = data.get("rid", "")
            if rid.startswith(HEALTH_CHECK_RID_PREFIX):
                continue

            if data.get("event") == "request.received":
                self.assertIn("rid", data)
                self.assertIn("obj", data)
                self.assertEqual(
                    data.get("headers", {}).get("x-smg-routing-key"), TEST_ROUTING_KEY
                )
                received_found = True
            elif data.get("event") == "request.finished":
                self.assertIn("rid", data)
                self.assertIn("obj", data)
                self.assertIn("out", data)
                self.assertEqual(
                    data.get("headers", {}).get("x-smg-routing-key"), TEST_ROUTING_KEY
                )
                finished_found = True

        self.assertTrue(
            received_found, f"request.received event not found in {source_name}"
        )
        self.assertTrue(
            finished_found, f"request.finished event not found in {source_name}"
        )
```
**EN:** This block implements `_verify_logs` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_verify_logs`，承担模块行为中的一个聚焦逻辑片段。

### Lines 228-255: method verify openai logs / 方法 verify openai logs
```python
    def _verify_openai_logs(self, content: str, source_name: str):
        openai_received_found = False
        for line in content.splitlines():
            idx = line.find("{")
            if idx == -1:
                continue
            try:
                data = json.loads(line[idx:])
            except json.JSONDecodeError:
                continue
            if data.get("event") != "request.received.openai":
                continue

            obj = data.get("obj", {})
            self.assertEqual(obj.get("model"), TEST_MODEL_NAME)
            self.assertIsInstance(obj.get("messages"), list)
            self.assertGreater(len(obj.get("messages")), 0)
            self.assertEqual(obj["messages"][0].get("content"), "hello request logger")
            self.assertEqual(
                data.get("headers", {}).get("x-smg-routing-key"), TEST_ROUTING_KEY
            )
            openai_received_found = True
            break

        self.assertTrue(
            openai_received_found,
            f"request.received.openai event not found in {source_name}",
        )
```
**EN:** This block implements `_verify_openai_logs` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_verify_openai_logs`，承担模块行为中的一个聚焦逻辑片段。

### Lines 258-258: class TestCustomHeaderViaEnvVar declaration / 类 TestCustomHeaderViaEnvVar 声明
```python
class TestCustomHeaderViaEnvVar(BaseTestRequestLogger, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `BaseTestRequestLogger`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `BaseTestRequestLogger`, `CustomTestCase`。

### Lines 259-266: class-level constants and configuration for `TestCustomHeaderViaEnvVar` / 类级常量与配置
```python
    """Test that custom headers can be added via SGLANG_LOG_REQUEST_HEADERS env var."""

    log_requests_format = "text"
    env_vars = {"SGLANG_LOG_REQUEST_HEADERS": TEST_CUSTOM_HEADER_NAME}
    request_headers = {
        "X-SMG-Routing-Key": TEST_ROUTING_KEY,
        TEST_CUSTOM_HEADER_NAME: TEST_CUSTOM_HEADER_VALUE,
    }
```
**EN:** This block defines shared names such as `log_requests_format`, `env_vars`, `request_headers`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `log_requests_format`, `env_vars`, `request_headers` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 268-290: method verify logs / 方法 verify logs
```python
    def _verify_logs(self, content: str, source_name: str):
        # Verify custom header is logged
        self.assertIn(
            TEST_CUSTOM_HEADER_NAME.lower(),
            content,
            f"Custom header name not found in {source_name}",
        )
        self.assertIn(
            TEST_CUSTOM_HEADER_VALUE,
            content,
            f"Custom header value not found in {source_name}",
        )
        # Verify default header is still logged (env var appends, not replaces)
        self.assertIn(
            "x-smg-routing-key",
            content,
            f"Default header should still be in whitelist in {source_name}",
        )
        self.assertIn(
            TEST_ROUTING_KEY,
            content,
            f"Default header value not found in {source_name}",
        )
```
**EN:** This block implements `_verify_logs` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_verify_logs`，承担模块行为中的一个聚焦逻辑片段。

### Lines 292-305: method verify openai logs / 方法 verify openai logs
```python
    def _verify_openai_logs(self, content: str, source_name: str):
        self.assertIn(
            "Receive OpenAI:", content, f"OpenAI receive log not found in {source_name}"
        )
        self.assertIn(
            TEST_CUSTOM_HEADER_NAME.lower(),
            content,
            f"Custom header name not found in {source_name}",
        )
        self.assertIn(
            TEST_CUSTOM_HEADER_VALUE,
            content,
            f"Custom header value not found in {source_name}",
        )
```
**EN:** This block implements `_verify_openai_logs` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_verify_openai_logs`，承担模块行为中的一个聚焦逻辑片段。

### Lines 308-309: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `BaseTestRequestLogger`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestRequestLoggerText`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestRequestLoggerJson`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestCustomHeaderViaEnvVar`: Test that custom headers can be added via SGLANG_LOG_REQUEST_HEADERS env var. / 用于组织相关测试、夹具或辅助方法。
- `BaseTestRequestLogger.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `BaseTestRequestLogger.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `BaseTestRequestLogger._verify_logs`: This block implements `_verify_logs` and captures one focused piece of the module's behavior. / 该代码块实现 `_verify_logs`，承担模块行为中的一个聚焦逻辑片段。
- `BaseTestRequestLogger._verify_openai_logs`: This block implements `_verify_openai_logs` and captures one focused piece of the module's behavior. / 该代码块实现 `_verify_openai_logs`，承担模块行为中的一个聚焦逻辑片段。
- `BaseTestRequestLogger._wait_until_verified`: This block implements `_wait_until_verified` and captures one focused piece of the module's behavior. / 该代码块实现 `_wait_until_verified`，承担模块行为中的一个聚焦逻辑片段。
- `BaseTestRequestLogger.test_logging`: This test exercises `test_logging` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_logging`。
- `BaseTestRequestLogger.test_openai_chat_logging`: This test exercises `test_openai_chat_logging` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_openai_chat_logging`。
- `TestRequestLoggerText._verify_logs`: This block implements `_verify_logs` and captures one focused piece of the module's behavior. / 该代码块实现 `_verify_logs`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `io`, `json`, `os`, `tempfile`, `time`, `unittest`, `pathlib`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.srt.constants`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 309
