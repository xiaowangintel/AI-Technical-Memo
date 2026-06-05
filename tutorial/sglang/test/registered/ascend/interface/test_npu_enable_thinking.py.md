# test_npu_enable_thinking.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/interface/test_npu_enable_thinking.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on interface npu enable thinking in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 interface npu enable thinking 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Import dependencies
```python
import json
import unittest

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ascend.test_ascend_utils import QWEN3_30B_A3B_WEIGHTS_PATH
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

### Lines 16-21: Register CI metadata
```python
register_npu_ci(
    est_time=400,
    suite="nightly-2-npu-a3",
    nightly=True,
    disabled="https://github.com/Ascend/sglang/issues/32",
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 24-24: Define class TestEnableThinking
```python
class TestEnableThinking(CustomTestCase):
```
**EN:** This declaration introduces the `TestEnableThinking` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestEnableThinking` 测试类，并说明它通过继承承担的职责。

### Lines 25-30: Document the class `TestEnableThinking`
```python
    """Testcase: Testing with the 'enable_thinking' feature enabled/disabled,
                 both streaming and non-streaming input requests successful

    [Test Category] Interface
    [Test Target] /v1/chat/completions
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestEnableThinking`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestEnableThinking`的设计意图。

### Lines 32-51: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = QWEN3_30B_A3B_WEIGHTS_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.other_args = [
            "--attention-backend",
            "ascend",
            "--disable-cuda-graph",
            "--mem-fraction-static",
            0.95,
            "--tp",
            16,
        ]
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=cls.other_args,
        )
        cls.additional_chat_kwargs = {}
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 53-55: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 57-78: Run test: chat completion with reasoning
```python
    def test_chat_completion_with_reasoning(self):
        # Test non-streaming with "enable_thinking": True, reasoning_content should not be empty
        client = requests.post(
            f"{self.base_url}/v1/chat/completions",
            json={
                "model": self.model,
                "messages": [{"role": "user", "content": "Hello"}],
                "temperature": 0,
                "separate_reasoning": True,
                "chat_template_kwargs": {"enable_thinking": True},
                **self.additional_chat_kwargs,
            },
        )

        self.assertEqual(client.status_code, 200, f"Failed with: {client.text}")
        data = client.json()

        self.assertIn("choices", data)
        self.assertTrue(len(data["choices"]) > 0)
        self.assertIn("message", data["choices"][0])
        self.assertIn("reasoning_content", data["choices"][0]["message"])
        self.assertIsNotNone(data["choices"][0]["message"]["reasoning_content"])
```
**EN:** This test method exercises chat completion with reasoning and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 chat completion with reasoning 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 80-102: Run test: chat completion without reasoning
```python
    def test_chat_completion_without_reasoning(self):
        # Test non-streaming with "enable_thinking": False, reasoning_content should be empty
        client = requests.post(
            f"{self.base_url}/v1/chat/completions",
            json={
                "model": self.model,
                "messages": [{"role": "user", "content": "Hello"}],
                "temperature": 0,
                "separate_reasoning": True,
                "chat_template_kwargs": {"enable_thinking": False},
                **self.additional_chat_kwargs,
            },
        )

        self.assertEqual(client.status_code, 200, f"Failed with: {client.text}")
        data = client.json()

        self.assertIn("choices", data)
        self.assertTrue(len(data["choices"]) > 0)
        self.assertIn("message", data["choices"][0])

        if "reasoning_content" in data["choices"][0]["message"]:
            self.assertIsNone(data["choices"][0]["message"]["reasoning_content"])
```
**EN:** This test method exercises chat completion without reasoning and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 chat completion without reasoning 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 104-146: Run test: stream chat completion with reasoning
```python
    def test_stream_chat_completion_with_reasoning(self):
        # Test streaming with "enable_thinking": True, reasoning_content should not be empty
        response = requests.post(
            f"{self.base_url}/v1/chat/completions",
            json={
                "model": self.model,
                "messages": [{"role": "user", "content": "Hello"}],
                "temperature": 0,
                "separate_reasoning": True,
                "stream": True,
                "chat_template_kwargs": {"enable_thinking": True},
                **self.additional_chat_kwargs,
            },
            stream=True,
        )

        self.assertEqual(response.status_code, 200, f"Failed with: {response.text}")

        has_reasoning = False
        has_content = False

        print("\n=== Stream With Reasoning ===")
        for line in response.iter_lines():
            if line:
                line = line.decode("utf-8")
                if line.startswith("data:") and not line.startswith("data: [DONE]"):
                    data = json.loads(line[6:])
                    if "choices" in data and len(data["choices"]) > 0:
                        delta = data["choices"][0].get("delta", {})

                        if "reasoning_content" in delta and delta["reasoning_content"]:
                            has_reasoning = True

                        if "content" in delta and delta["content"]:
                            has_content = True

        self.assertTrue(
            has_reasoning,
            "The reasoning content is not included in the stream response",
        )
        self.assertTrue(
            has_content, "The stream response does not contain normal content"
        )
```
**EN:** This test method exercises stream chat completion with reasoning and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 stream chat completion with reasoning 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 148-190: Run test: stream chat completion without reasoning
```python
    def test_stream_chat_completion_without_reasoning(self):
        # Test streaming with "enable_thinking": False, reasoning_content should  be empty
        response = requests.post(
            f"{self.base_url}/v1/chat/completions",
            json={
                "model": self.model,
                "messages": [{"role": "user", "content": "Hello"}],
                "temperature": 0,
                "separate_reasoning": True,
                "stream": True,
                "chat_template_kwargs": {"enable_thinking": False},
                **self.additional_chat_kwargs,
            },
            stream=True,
        )

        self.assertEqual(response.status_code, 200, f"Failed with: {response.text}")

        has_reasoning = False
        has_content = False

        print("\n=== Stream Without Reasoning ===")
        for line in response.iter_lines():
            if line:
                line = line.decode("utf-8")
                if line.startswith("data:") and not line.startswith("data: [DONE]"):
                    data = json.loads(line[6:])
                    if "choices" in data and len(data["choices"]) > 0:
                        delta = data["choices"][0].get("delta", {})

                        if "reasoning_content" in delta and delta["reasoning_content"]:
                            has_reasoning = True

                        if "content" in delta and delta["content"]:
                            has_content = True

        self.assertFalse(
            has_reasoning,
            "The reasoning content should not be included in the stream response",
        )
        self.assertTrue(
            has_content, "The stream response does not contain normal content"
        )
```
**EN:** This test method exercises stream chat completion without reasoning and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 stream chat completion without reasoning 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 193-194: Expose unittest entrypoint
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
- Notable symbols / 关键符号: `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `requests.post`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
