# test_npu_sampling_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/basic_function/backends/test_npu_sampling_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on backends npu sampling backend in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 backends npu sampling backend 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Import dependencies
```python
import unittest
from types import SimpleNamespace

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.run_eval import run_eval
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
register_npu_ci(est_time=400, suite="stage-b-test-1-npu-a2", nightly=False)
register_npu_ci(est_time=400, suite="nightly-1-npu-a3", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 20-20: Define class TestAscendSamplingBackend
```python
class TestAscendSamplingBackend(CustomTestCase):
```
**EN:** This declaration introduces the `TestAscendSamplingBackend` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestAscendSamplingBackend` 测试类，并说明它通过继承承担的职责。

### Lines 21-37: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = "/root/.cache/modelscope/hub/models/Qwen/Qwen2.5-7B-Instruct"
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--sampling-backend",
                "ascend",
                "--disable-radix-cache",
                "--disable-cuda-graph",
                "--mem-fraction-static",
                0.85,
            ],
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 39-41: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 43-54: Run test: mmlu
```python
    def test_mmlu(self):
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="mmlu",
            num_examples=64,
            num_threads=32,
            temperature=0.1,
        )

        metrics = run_eval(args)
        self.assertGreaterEqual(metrics["score"], 0.65)
```
**EN:** This test method exercises mmlu and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 mmlu 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 56-96: Run test: greedy
```python
    def test_greedy(self):

        first_text = None

        # ensure the answer is identical across single response
        for _ in range(5):
            response_single = requests.post(
                self.base_url + "/generate",
                json={
                    "text": "The capital of Germany is",
                    "sampling_params": {
                        "temperature": 0,
                        "max_new_tokens": 32,
                    },
                },
            ).json()
            text = response_single["text"]
            if first_text is None:
                first_text = text

            self.assertEqual(text, first_text)

        first_text = None

        response_batch = requests.post(
            self.base_url + "/generate",
            json={
                "text": ["The capital of Germany is"] * 10,
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 32,
                },
            },
        ).json()

        # ensure the answer is identical among the batch
        for i in range(10):
            text = response_batch[i]["text"]
            if first_text is None:
                first_text = text
            self.assertEqual(text, first_text)
```
**EN:** This test method exercises greedy and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 greedy 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 99-100: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `requests`, `types`, `unittest`
- Notable symbols / 关键符号: `popen_launch_server`, `kill_process_tree`, `run_eval`, `CustomTestCase`, `SimpleNamespace`, `requests.post`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
