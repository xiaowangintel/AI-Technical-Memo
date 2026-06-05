# test_npu_offload_modes.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/basic_function/offloading/test_npu_offload_modes.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on offloading npu offload modes in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 offloading npu offload modes 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Import dependencies
```python
import unittest
from urllib.parse import urlparse

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ascend.test_ascend_utils import DEEPSEEK_CODER_V2_LITE_WEIGHTS_PATH
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
register_npu_ci(est_time=800, suite="nightly-2-npu-a3", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 18-20: Define module constants
```python
TEST_MODEL_MATRIX = {
    DEEPSEEK_CODER_V2_LITE_WEIGHTS_PATH,
}
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 23-23: Define class TestAscendOffloadModes
```python
class TestAscendOffloadModes(CustomTestCase):
```
**EN:** This declaration introduces the `TestAscendOffloadModes` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestAscendOffloadModes` 测试类，并说明它通过继承承担的职责。

### Lines 25-45: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.models = TEST_MODEL_MATRIX
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.url = urlparse(DEFAULT_URL_FOR_TEST)
        cls.common_args = [
            "--trust-remote-code",
            "--disable-cuda-graph",
            "--mem-fraction-static",
            0.9,
            "--attention-backend",
            "ascend",
            "--offload-group-size",
            4,
            "--offload-num-in-group",
            1,
            "--offload-prefetch-step",
            1,
            "--dp-size",
            2,
        ]
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 47-92: Define helper: run a test
```python
    def run_a_test(self, offload_mode, additional_args=None):
        """Run test for a specific offload mode."""
        for model in self.models:
            with self.subTest(model=model, offload_mode=offload_mode):
                print(f"##=== Testing {offload_mode} offload: {model} ===##")

                args = [
                    *self.common_args,
                    "--offload-mode",
                    offload_mode,
                ]

                if additional_args:
                    args.extend(additional_args)

                process = popen_launch_server(
                    model,
                    self.base_url,
                    timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                    other_args=args,
                )

                try:
                    # Check if server is running (basic functionality test)
                    response = requests.post(
                        f"{DEFAULT_URL_FOR_TEST}/generate",
                        json={
                            "text": "Where is the capital of France?",
                            "sampling_params": {
                                "temperature": 0,
                                "max_new_tokens": 32,
                            },
                        },
                    )
                    self.assertEqual(
                        response.status_code,
                        200,
                        f"The request status code is not 200, server failed to respond for {offload_mode}",
                    )
                    self.assertIn(
                        "Paris",
                        response.text,
                        f"The inference result does not include Paris, server failed to respond for {offload_mode}",
                    )
                finally:
                    kill_process_tree(process.pid)
```
**EN:** This helper function encapsulates reusable logic inside `TestAscendOffloadModes` so the scenario stays organized. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestAscendOffloadModes` 内部调用，从而让场景结构更清晰。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 94-96: Run test: offload mode cpu
```python
    def test_offload_mode_cpu(self):
        """Test offload mode: cpu"""
        self.run_a_test("cpu")
```
**EN:** This test method exercises offload mode cpu and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 offload mode cpu 场景，并验证观测到的行为是否符合预期契约。

### Lines 98-100: Run test: offload mode sharded gpu
```python
    def test_offload_mode_sharded_gpu(self):
        """Test offload mode: sharded_gpu"""
        self.run_a_test("sharded_gpu")
```
**EN:** This test method exercises offload mode sharded gpu and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 offload mode sharded gpu 场景，并验证观测到的行为是否符合预期契约。

### Lines 103-104: Expose unittest entrypoint
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
- External and stdlib modules / 外部与标准库模块: `requests`, `unittest`, `urllib.parse`
- Notable symbols / 关键符号: `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `requests.post`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
