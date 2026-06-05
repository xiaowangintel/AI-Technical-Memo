# test_soft_watchdog.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/test_soft_watchdog.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on debug utils soft watchdog in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 debug utils soft watchdog 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Import dependencies
```python
import io
import unittest

import requests

from sglang.srt.environ import envs
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 16-17: Register CI metadata
```python
register_cuda_ci(est_time=120, suite="nightly-1-gpu", nightly=True)
register_amd_ci(est_time=120, suite="nightly-amd-1-gpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 20-20: Define class BaseTestSoftWatchdog
```python
class BaseTestSoftWatchdog:
```
**EN:** This declaration introduces the `BaseTestSoftWatchdog` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `BaseTestSoftWatchdog` 测试类，并说明它通过继承承担的职责。

### Lines 21-22: Declare BaseTestSoftWatchdog configuration
```python
    env_override = None
    expected_message = None
```
**EN:** This block defines class-level settings that are shared across the `BaseTestSoftWatchdog` test methods.
**CN:** 该代码块定义了 `BaseTestSoftWatchdog` 各测试方法共享的类级配置。

### Lines 24-40: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.stdout = io.StringIO()
        cls.stderr = io.StringIO()

        with cls.env_override():
            cls.process = popen_launch_server(
                "Qwen/Qwen3-0.6B",
                DEFAULT_URL_FOR_TEST,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=[
                    "--soft-watchdog-timeout",
                    "20",
                    "--skip-server-warmup",
                ],
                return_stdout_stderr=(cls.stdout, cls.stderr),
            )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 42-46: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
        cls.stdout.close()
        cls.stderr.close()
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 48-64: Run test: watchdog triggers
```python
    def test_watchdog_triggers(self):
        print("Start call /generate API", flush=True)
        try:
            requests.post(
                DEFAULT_URL_FOR_TEST + "/generate",
                json={
                    "text": "Hello, please repeat this sentence for 1000 times.",
                    "sampling_params": {"max_new_tokens": 100, "temperature": 0},
                },
                timeout=30,
            )
        except requests.exceptions.ReadTimeout as e:
            print(f"requests.post timeout (but expected): {e}")
        print("End call /generate API", flush=True)

        combined_output = self.stdout.getvalue() + self.stderr.getvalue()
        self.assertIn(self.expected_message, combined_output)
```
**EN:** This test method exercises watchdog triggers and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 watchdog triggers 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 67-67: Define class TestSoftWatchdogDetokenizer
```python
class TestSoftWatchdogDetokenizer(BaseTestSoftWatchdog, CustomTestCase):
```
**EN:** This declaration introduces the `TestSoftWatchdogDetokenizer` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestSoftWatchdogDetokenizer` 测试类，并说明它通过继承承担的职责。

### Lines 68-69: Declare TestSoftWatchdogDetokenizer configuration
```python
    env_override = lambda: envs.SGLANG_TEST_STUCK_DETOKENIZER.override(30)
    expected_message = "DetokenizerManager watchdog timeout"
```
**EN:** This block defines class-level settings that are shared across the `TestSoftWatchdogDetokenizer` test methods.
**CN:** 该代码块定义了 `TestSoftWatchdogDetokenizer` 各测试方法共享的类级配置。

### Lines 72-72: Define class TestSoftWatchdogTokenizer
```python
class TestSoftWatchdogTokenizer(BaseTestSoftWatchdog, CustomTestCase):
```
**EN:** This declaration introduces the `TestSoftWatchdogTokenizer` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestSoftWatchdogTokenizer` 测试类，并说明它通过继承承担的职责。

### Lines 73-74: Declare TestSoftWatchdogTokenizer configuration
```python
    env_override = lambda: envs.SGLANG_TEST_STUCK_TOKENIZER.override(30)
    expected_message = "TokenizerManager watchdog timeout"
```
**EN:** This block defines class-level settings that are shared across the `TestSoftWatchdogTokenizer` test methods.
**CN:** 该代码块定义了 `TestSoftWatchdogTokenizer` 各测试方法共享的类级配置。

### Lines 77-77: Define class TestSoftWatchdogSchedulerInit
```python
class TestSoftWatchdogSchedulerInit(BaseTestSoftWatchdog, CustomTestCase):
```
**EN:** This declaration introduces the `TestSoftWatchdogSchedulerInit` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestSoftWatchdogSchedulerInit` 测试类，并说明它通过继承承担的职责。

### Lines 78-79: Declare TestSoftWatchdogSchedulerInit configuration
```python
    env_override = lambda: envs.SGLANG_TEST_STUCK_SCHEDULER_INIT.override(30)
    expected_message = "Scheduler watchdog timeout"
```
**EN:** This block defines class-level settings that are shared across the `TestSoftWatchdogSchedulerInit` test methods.
**CN:** 该代码块定义了 `TestSoftWatchdogSchedulerInit` 各测试方法共享的类级配置。

### Lines 82-83: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Server lifecycle helpers start the target runtime before assertions begin. / 服务生命周期辅助函数会在断言开始前启动目标运行时。
- Explicit teardown avoids leaked child processes between runs. / 显式清理可以避免多次运行之间遗留子进程。
- HTTP requests validate the externally visible API contract rather than only internal helpers. / HTTP 请求验证的是对外暴露的 API 契约，而不只是内部辅助函数。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `io`, `requests`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `requests.post`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
