# test_npu_log_level.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/basic_function/parameter/test_npu_log_level.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on parameter npu log level in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 parameter npu log level 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Import dependencies
```python
import os
import unittest

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ascend.test_ascend_utils import LLAMA_3_2_1B_INSTRUCT_WEIGHTS_PATH
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 16-16: Register CI metadata
```python
register_npu_ci(est_time=400, suite="nightly-1-npu-a3", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 19-19: Define class TestLogLevel
```python
class TestLogLevel(CustomTestCase):
```
**EN:** This declaration introduces the `TestLogLevel` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestLogLevel` 测试类，并说明它通过继承承担的职责。

### Lines 20-24: Document the class `TestLogLevel`
```python
    """Testcase：Verify set log-level parameter, the printed log level is the same as the configured log level and the inference request is successfully processed.

    [Test Category] Parameter
    [Test Target] --log-level
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestLogLevel`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestLogLevel`的设计意图。

### Lines 26-28: Declare TestLogLevel configuration
```python
    model = LLAMA_3_2_1B_INSTRUCT_WEIGHTS_PATH
    OUT_LOG_PATH = "./out_log.txt"
    ERR_LOG_PATH = "./err_log.txt"
```
**EN:** This block defines class-level settings that are shared across the `TestLogLevel` test methods.
**CN:** 该代码块定义了 `TestLogLevel` 各测试方法共享的类级配置。

### Lines 30-62: Define helper: launch server and run infer
```python
    def _launch_server_and_run_infer(self, other_args):
        out_log_file = None
        err_log_file = None
        process = None
        try:
            out_log_file = open(self.OUT_LOG_PATH, "w+", encoding="utf-8")
            err_log_file = open(self.ERR_LOG_PATH, "w+", encoding="utf-8")
            process = popen_launch_server(
                self.model,
                DEFAULT_URL_FOR_TEST,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=other_args,
                return_stdout_stderr=(out_log_file, err_log_file),
            )
            health_resp = requests.get(f"{DEFAULT_URL_FOR_TEST}/health_generate")
            self.assertEqual(health_resp.status_code, 200)
            gen_resp = requests.post(
                f"{DEFAULT_URL_FOR_TEST}/generate",
                json={
                    "text": "The capital of France is",
                    "sampling_params": {"temperature": 0, "max_new_tokens": 32},
                },
            )
            self.assertEqual(gen_resp.status_code, 200)
            self.assertIn("Paris", gen_resp.text)
            out_log_file.seek(0)
            return out_log_file.read()
        finally:
            kill_process_tree(process.pid)
            out_log_file.close()
            err_log_file.close()
            os.remove(self.OUT_LOG_PATH)
            os.remove(self.ERR_LOG_PATH)
```
**EN:** This helper function encapsulates reusable logic inside `TestLogLevel` so the scenario stays organized. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestLogLevel` 内部调用，从而让场景结构更清晰。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 64-74: Run test: log level
```python
    def test_log_level(self):
        # Verify set --log-level=warning and not set --log-level-http, logs print only warning level (no HTTP info)
        other_args = [
            "--log-level",
            "warning",
            "--attention-backend",
            "ascend",
            "--disable-cuda-graph",
        ]
        log_content = self._launch_server_and_run_infer(other_args)
        self.assertNotIn("POST /generate HTTP/1.1", log_content)
```
**EN:** This test method exercises log level and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 log level 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 76-88: Run test: log http level
```python
    def test_log_http_level(self):
        # Verify set --log-level=warning and set --log-level-http=info, log level print http info
        other_args = [
            "--log-level",
            "warning",
            "--log-level-http",
            "info",
            "--attention-backend",
            "ascend",
            "--disable-cuda-graph",
        ]
        log_content = self._launch_server_and_run_infer(other_args)
        self.assertIn("POST /generate HTTP/1.1", log_content)
```
**EN:** This test method exercises log http level and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 log http level 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 91-92: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ascend.test_ascend_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `requests`, `unittest`
- Notable symbols / 关键符号: `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `requests.get`, `requests.post`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
