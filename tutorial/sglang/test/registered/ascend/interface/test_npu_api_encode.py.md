# test_npu_api_encode.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/interface/test_npu_api_encode.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on interface npu api encode in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 interface npu api encode 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Import dependencies
```python
import logging
import unittest

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ascend.test_ascend_utils import QWEN3_VL_4B_INSTRUCT_WEIGHTS_PATH
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

### Lines 16-20: Implement expr logic
```python
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s",
    handlers=[logging.StreamHandler()],
)
```
**EN:** This block contains supporting statements that connect the surrounding setup and assertions.
**CN:** 该代码块包含连接周边初始化与断言的辅助语句。

### Lines 21-21: Define module constants
```python
logger = logging.getLogger(__name__)
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 23-23: Register CI metadata
```python
register_npu_ci(est_time=400, suite="nightly-1-npu-a3", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 26-26: Define class TestNpuApi
```python
class TestNpuApi(CustomTestCase):
```
**EN:** This declaration introduces the `TestNpuApi` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNpuApi` 测试类，并说明它通过继承承担的职责。

### Lines 27-31: Document the class `TestNpuApi`
```python
    """Testcase: Verify the availability and correctness of the /encode API on Ascend backend with GME_QWEN2_VL_2B_INSTRUCT model.

    [Test Category] Interface
    [Test Target] /encode
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestNpuApi`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestNpuApi`的设计意图。

### Lines 33-49: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = QWEN3_VL_4B_INSTRUCT_WEIGHTS_PATH
        other_args = [
            "--attention-backend",
            "ascend",
            "--disable-cuda-graph",
            "--tp-size",
            2,
            "--is-embedding",
        ]
        cls.process = popen_launch_server(
            cls.model,
            DEFAULT_URL_FOR_TEST,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=other_args,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 51-53: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 55-71: Run test: api encode 01
```python
    def test_api_encode_01(self):
        # Test Scenario 1: Call /encode API with plain text parameter
        response = requests.post(
            f"{DEFAULT_URL_FOR_TEST}/encode",
            json={
                "rid": "2",
                "text": "what is the capital of France",
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 200,
                    "top_p": 1,
                },
            },
        )
        logger.info("Test 01 response keys: %s", response.json().keys())
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.json()["meta_info"]["id"], "2")
```
**EN:** This test method exercises api encode 01 and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 api encode 01 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 73-84: Run test: api encode 02
```python
    def test_api_encode_02(self):
        # Test Scenario 2: Call /encode API with input_ids parameter
        response = requests.post(
            f"{DEFAULT_URL_FOR_TEST}/encode",
            json={
                "rid": "3",
                "input_ids": [101, 7592, 2088, 102],
                "sampling_params": {"temperature": 0, "max_new_tokens": 200},
            },
        )
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.json()["meta_info"]["id"], "3")
```
**EN:** This test method exercises api encode 02 and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 api encode 02 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 86-99: Run test: api encode 03
```python
    def test_api_encode_03(self):
        # Test Scenario 3: Call /encode API with text and image parameters (multimodal capability verification)
        response = requests.post(
            f"{DEFAULT_URL_FOR_TEST}/encode",
            json={
                "rid": "4",
                "text": "show me the words",
                "image_data": "https://miaobi-lite.bj.bcebos.com/miaobi/5mao/b%27b2Ny6K%2BG5Yir5Luj56CBXzE3MzQ2MzcyNjAuMzgxNDk5NQ%3D%3D%27/0.png",
                "sampling_params": {"temperature": 0, "max_new_tokens": 200},
            },
        )
        logger.info("Test 03 response keys: %s", response.json().keys())
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.json()["meta_info"]["id"], "4")
```
**EN:** This test method exercises api encode 03 and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 api encode 03 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 101-130: Run test: api encode 04
```python
    def test_api_encode_04(self):
        # Test Scenario 4: Call /encode API with list of rids (multiple requests) - text input
        request_rids = ["5", "6", "7"]
        response = requests.post(
            f"{DEFAULT_URL_FOR_TEST}/encode",
            json={
                "rid": request_rids,
                "text": [
                    "what is the capital of UK",
                    "what is the capital of Germany",
                    "what is the capital of Japan",
                ],
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 200,
                    "top_p": 1,
                },
            },
        )
        response_json = response.json()
        logger.info(
            "Test 04 response type: %s, first item meta_info: %s",
            type(response_json),
            response_json[0].get("meta_info", {}),
        )

        self.assertEqual(response.status_code, 200)
        self.assertEqual(len(response_json), len(request_rids))
        for idx, result in enumerate(response_json):
            self.assertEqual(result["meta_info"]["id"], request_rids[idx])
```
**EN:** This test method exercises api encode 04 and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 api encode 04 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 133-134: Expose unittest entrypoint
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
- External and stdlib modules / 外部与标准库模块: `logging`, `requests`, `unittest`
- Notable symbols / 关键符号: `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `requests.post`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
