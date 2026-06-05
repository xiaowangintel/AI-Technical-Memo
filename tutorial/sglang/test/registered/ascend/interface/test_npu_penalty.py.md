# test_npu_penalty.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/interface/test_npu_penalty.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on interface npu penalty in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 interface npu penalty 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Import dependencies
```python
import json
import random
import unittest
from concurrent.futures import ThreadPoolExecutor

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

### Lines 18-18: Register CI metadata
```python
register_npu_ci(est_time=400, suite="nightly-1-npu-a3", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 21-21: Define class TestPenalty
```python
class TestPenalty(CustomTestCase):
```
**EN:** This declaration introduces the `TestPenalty` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestPenalty` 测试类，并说明它通过继承承担的职责。

### Lines 22-26: Document the class `TestPenalty`
```python
    """Testcase：Verify successful processing of inference requests with three specific mechanisms(frequency_penalty, presence_penalty, min_new_tokens).

    [Test Category] Interface
    [Test Target] /generate
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestPenalty`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestPenalty`的设计意图。

### Lines 28-42: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = LLAMA_3_2_1B_INSTRUCT_WEIGHTS_PATH
        other_args = [
            "--attention-backend",
            "ascend",
            "--disable-cuda-graph",
        ]
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=other_args,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 44-46: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 48-74: Define helper: run decode
```python
    def run_decode(self, sampling_params):
        # Send inference request with specified sampling/penalty parameters.

        return_logprob = True
        top_logprobs_num = 5
        return_text = True
        n = 1

        response = requests.post(
            self.base_url + "/generate",
            json={
                # prompt that is supposed to generate < 32 tokens
                "text": "<|start_header_id|>user<|end_header_id|>\n\nWhat is the answer for 1 + 1 = ?<|eot_id|><|start_header_id|>assistant<|end_header_id|>\n\n",
                "sampling_params": {
                    "max_new_tokens": 48,
                    "n": n,
                    **sampling_params,
                },
                "return_logprob": return_logprob,
                "top_logprobs_num": top_logprobs_num,
                "return_text_in_logprobs": return_text,
                "logprob_start_len": 0,
            },
        )
        self.assertEqual(response.status_code, 200)
        print(json.dumps(response.json()))
        print("=" * 100)
```
**EN:** This helper function encapsulates reusable logic inside `TestPenalty` so the scenario stays organized. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestPenalty` 内部调用，从而让场景结构更清晰。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 76-77: Run test: default values
```python
    def test_default_values(self):
        self.run_decode({})
```
**EN:** This test method exercises default values and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 default values 场景，并验证观测到的行为是否符合预期契约。

### Lines 79-80: Run test: frequency penalty
```python
    def test_frequency_penalty(self):
        self.run_decode({"frequency_penalty": 2})
```
**EN:** This test method exercises frequency penalty and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 frequency penalty 场景，并验证观测到的行为是否符合预期契约。

### Lines 82-83: Run test: min new tokens
```python
    def test_min_new_tokens(self):
        self.run_decode({"min_new_tokens": 16})
```
**EN:** This test method exercises min new tokens and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 min new tokens 场景，并验证观测到的行为是否符合预期契约。

### Lines 85-86: Run test: presence penalty
```python
    def test_presence_penalty(self):
        self.run_decode({"presence_penalty": 2})
```
**EN:** This test method exercises presence penalty and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 presence penalty 场景，并验证观测到的行为是否符合预期契约。

### Lines 88-107: Run test: penalty mixed
```python
    def test_penalty_mixed(self):
        args = [
            {},
            {},
            {},
            {"frequency_penalty": 2},
            {"presence_penalty": 1},
            {"min_new_tokens": 16},
            {"frequency_penalty": 0.2},
            {"presence_penalty": 0.4},
            {"min_new_tokens": 8},
            {"frequency_penalty": 0.4, "presence_penalty": 0.8},
            {"frequency_penalty": 0.4, "min_new_tokens": 12},
            {"presence_penalty": 0.8, "min_new_tokens": 12},
            {"presence_penalty": -0.3, "frequency_penalty": 1.3, "min_new_tokens": 32},
            {"presence_penalty": 0.3, "frequency_penalty": -1.3, "min_new_tokens": 32},
        ]
        random.shuffle(args * 5)
        with ThreadPoolExecutor(8) as executor:
            list(executor.map(self.run_decode, args))
```
**EN:** This test method exercises penalty mixed and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 penalty mixed 场景，并验证观测到的行为是否符合预期契约。

### Lines 110-111: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main(verbosity=3)
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
- External and stdlib modules / 外部与标准库模块: `concurrent.futures`, `json`, `random`, `requests`, `unittest`
- Notable symbols / 关键符号: `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `requests.post`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
