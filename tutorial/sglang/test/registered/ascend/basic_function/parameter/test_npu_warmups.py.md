# test_npu_warmups.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/basic_function/parameter/test_npu_warmups.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on parameter npu warmups in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 parameter npu warmups 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Import dependencies
```python
import os
import unittest

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ascend.test_ascend_utils import MINICPM_O_2_6_WEIGHTS_PATH
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.test_utils import (
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 15-20: Register CI metadata
```python
register_npu_ci(
    est_time=400,
    suite="nightly-4-npu-a3",
    nightly=True,
    disabled="run failed",
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 23-23: Define class TestAscendWarmups
```python
class TestAscendWarmups(CustomTestCase):
```
**EN:** This declaration introduces the `TestAscendWarmups` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestAscendWarmups` 测试类，并说明它通过继承承担的职责。

### Lines 24-28: Document the class `TestAscendWarmups`
```python
    """Testcase: Test that the warm-up task runs successfully when the --warmups voice_chat parameter is specified upon service startup.

    [Test Category] Parameter
    [Test Target] --warmups
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestAscendWarmups`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestAscendWarmups`的设计意图。

### Lines 30-31: Declare TestAscendWarmups configuration
```python
    model = MINICPM_O_2_6_WEIGHTS_PATH
    base_url = DEFAULT_URL_FOR_TEST
```
**EN:** This block defines class-level settings that are shared across the `TestAscendWarmups` test methods.
**CN:** 该代码块定义了 `TestAscendWarmups` 各测试方法共享的类级配置。

### Lines 33-55: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        other_args = [
            "--trust-remote-code",
            "--warmups",
            "voice_chat",
            "--tp-size",
            "4",
            "--mem-fraction-static",
            "0.8",
            "--attention-backend",
            "ascend",
            "--disable-cuda-graph",
        ]
        cls.out_log_file = open("./out_log.txt", "w+", encoding="utf-8")
        cls.err_log_file = open("./err_log.txt", "w+", encoding="utf-8")
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=3600,
            other_args=other_args,
            return_stdout_stderr=(cls.out_log_file, cls.err_log_file),
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 57-63: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
        cls.out_log_file.close()
        cls.err_log_file.close()
        os.remove("./out_log.txt")
        os.remove("./err_log.txt")
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 65-88: Run test: warmups with voice chat
```python
    def test_warmups_with_voice_chat(self):
        # Call the get_server_info API to verify that the warmups parameter configuration takes effect.
        response = requests.get(f"{DEFAULT_URL_FOR_TEST}/server_info")
        self.assertEqual(response.status_code, 200)
        self.assertEqual("voice_chat", response.json().get("warmups"))

        # Verify the actual execution of the warm-up task.
        self.err_log_file.seek(0)
        content = self.err_log_file.read()
        self.assertIn("Running warmup voice_chat", content)

        # Verify that the inference API functions properly.
        response = requests.post(
            f"{DEFAULT_URL_FOR_TEST}/generate",
            json={
                "text": "The capital of France is",
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 32,
                },
            },
        )
        self.assertEqual(response.status_code, 200)
        self.assertIn("Paris", response.text)
```
**EN:** This test method exercises warmups with voice chat and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and issues HTTP POST requests against the exposed endpoint.
**CN:** 该测试方法会执行 warmups with voice chat 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并向暴露的端点发起 HTTP POST 请求。

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
- Notable symbols / 关键符号: `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `requests.get`, `requests.post`, `unittest.main`
