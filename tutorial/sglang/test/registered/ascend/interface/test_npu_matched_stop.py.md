# test_npu_matched_stop.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/interface/test_npu_matched_stop.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on interface npu matched stop in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 interface npu matched stop 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Import dependencies
```python
import json
import unittest

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ascend.test_ascend_utils import LLAMA_3_1_8B_INSTRUCT_WEIGHTS_PATH
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.test_utils import (
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 15-20: Define module constants
```python
MANY_NEW_TOKENS_PROMPT = """
Please write an extremely detailed and vivid fantasy story, set in a world full of intricate magic systems, political intrigue, and complex characters.
Ensure that you thoroughly describe every scene, character's motivations, and the environment. Include long, engaging dialogues and elaborate on the inner thoughts of the characters.
Each section should be as comprehensive as possible to create a rich and immersive experience for the reader.
The story should span multiple events, challenges, and character developments over time. Aim to make the story at least 3,000 words long.
"""
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 22-22: Register CI metadata
```python
register_npu_ci(est_time=400, suite="nightly-1-npu-a3", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 25-25: Define class TestMatchedStop
```python
class TestMatchedStop(CustomTestCase):
```
**EN:** This declaration introduces the `TestMatchedStop` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMatchedStop` 测试类，并说明它通过继承承担的职责。

### Lines 26-31: Document the class `TestMatchedStop`
```python
    """Testcase: Test configuring 'matched_stop' to different values(string, EOS token, length) correctly identifies
                 it as a stop signal.

    [Test Category] Interface
    [Test Target] /v1/chat/completions; /v1/completions
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestMatchedStop`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestMatchedStop`的设计意图。

### Lines 33-48: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = LLAMA_3_1_8B_INSTRUCT_WEIGHTS_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.other_args = [
            "--max-running-requests",
            10,
            "--attention-backend",
            "ascend",
            "--disable-cuda-graph",
            "--mem-fraction-static",
            0.8,
        ]
        cls.process = popen_launch_server(
            cls.model, cls.base_url, timeout=300, other_args=cls.other_args
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 50-52: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 54-85: Define helper: run completions generation
```python
    def run_completions_generation(
        self,
        prompt=MANY_NEW_TOKENS_PROMPT,
        max_tokens=1,
        stop=None,
        finish_reason=None,
        matched_stop=None,
    ):
        # Configure matched_stop to None, and use the '/v1/completions' interface
        # verify that the actual termination reason matches the configured value.
        payload = {
            "prompt": prompt,
            "model": self.model,
            "temperature": 0,
            "top_p": 1,
            "max_tokens": max_tokens,
        }

        if stop is not None:
            payload["stop"] = stop

        response_completions = requests.post(
            self.base_url + "/v1/completions",
            json=payload,
        )
        print(json.dumps(response_completions.json()))
        print("=" * 100)

        assert (
            response_completions.json()["choices"][0]["finish_reason"] == finish_reason
        )
        assert response_completions.json()["choices"][0]["matched_stop"] == matched_stop
```
**EN:** This helper function encapsulates reusable logic inside `TestMatchedStop` so the scenario stays organized. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestMatchedStop` 内部调用，从而让场景结构更清晰。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 87-119: Define helper: run chat completions generation
```python
    def run_chat_completions_generation(
        self,
        prompt=MANY_NEW_TOKENS_PROMPT,
        max_tokens=1,
        stop=None,
        finish_reason=None,
        matched_stop=None,
    ):
        # Configure matched_stop to None, and use the '/v1/chat/completions' interface
        # verify that the actual termination reason matches the configured value.
        chat_payload = {
            "model": self.model,
            "messages": [
                {"role": "system", "content": "You are a helpful AI assistant"},
                {"role": "user", "content": prompt},
            ],
            "temperature": 0,
            "top_p": 1,
            "max_tokens": max_tokens,
        }

        if stop is not None:
            chat_payload["stop"] = stop

        response_chat = requests.post(
            self.base_url + "/v1/chat/completions",
            json=chat_payload,
        )
        print(json.dumps(response_chat.json()))
        print("=" * 100)

        assert response_chat.json()["choices"][0]["finish_reason"] == finish_reason
        assert response_chat.json()["choices"][0]["matched_stop"] == matched_stop
```
**EN:** This helper function encapsulates reusable logic inside `TestMatchedStop` so the scenario stays organized. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestMatchedStop` 内部调用，从而让场景结构更清晰。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 121-128: Run test: finish stop str
```python
    def test_finish_stop_str(self):
        # Setting finish_reason="stop",'matched_stop="\n"' allows for correct termination
        self.run_completions_generation(
            max_tokens=1000, stop="\n", finish_reason="stop", matched_stop="\n"
        )
        self.run_chat_completions_generation(
            max_tokens=1000, stop="\n", finish_reason="stop", matched_stop="\n"
        )
```
**EN:** This test method exercises finish stop str and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 finish stop str 场景，并验证观测到的行为是否符合预期契约。

### Lines 130-150: Run test: finish stop eos
```python
    def test_finish_stop_eos(self):
        # Setting matched_stop is a specific EOS end flagallows for correct identification and termination of signal
        llama_format_prompt = """
        <|begin_of_text|><|start_header_id|>system<|end_header_id|>
        You are a helpful assistant.<|eot_id|><|start_header_id|>user<|end_header_id|>

        What is 2 + 2?<|eot_id|><|start_header_id|>assistant<|end_header_id|>
        """
        eos_token_id = 128009
        self.run_completions_generation(
            prompt=llama_format_prompt,
            max_tokens=1000,
            finish_reason="stop",
            matched_stop=eos_token_id,
        )
        self.run_chat_completions_generation(
            prompt="What is 2 + 2?",
            max_tokens=1000,
            finish_reason="stop",
            matched_stop=eos_token_id,
        )
```
**EN:** This test method exercises finish stop eos and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 finish stop eos 场景，并验证观测到的行为是否符合预期契约。

### Lines 152-159: Run test: finish length
```python
    def test_finish_length(self):
        # Setting finish_reason="length",'matched_stop="\n"' allows for correct termination
        self.run_completions_generation(
            max_tokens=5, finish_reason="length", matched_stop=None
        )
        self.run_chat_completions_generation(
            max_tokens=5, finish_reason="length", matched_stop=None
        )
```
**EN:** This test method exercises finish length and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 finish length 场景，并验证观测到的行为是否符合预期契约。

### Lines 162-163: Expose unittest entrypoint
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
- External and stdlib modules / 外部与标准库模块: `json`, `requests`, `unittest`
- Notable symbols / 关键符号: `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `requests.post`, `unittest.main`
