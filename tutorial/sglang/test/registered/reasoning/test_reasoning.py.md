# test_reasoning.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/reasoning/test_reasoning.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates reasoning behavior in SGLang's reasoning area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 推理 领域中与 reasoning 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: module imports and dependencies / 模块导入与依赖
```python
import json
import unittest

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.kits.reasoning_kit import (
    ReasoningTokenUsageMixin,
    SeparateReasoningMixin,
)
from sglang.test.test_utils import (
    DEFAULT_ENABLE_THINKING_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `unittest`, `requests`, `sglang.srt.utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `unittest`, `requests`, `sglang.srt.utils`。

### Lines 20-21: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=129, stage="base-b", runner_config="1-gpu-large")
register_amd_ci(est_time=200, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 24-26: class TestEnableThinking declaration / 类 TestEnableThinking 声明
```python
class TestEnableThinking(
    ReasoningTokenUsageMixin, SeparateReasoningMixin, CustomTestCase
):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `ReasoningTokenUsageMixin`, `SeparateReasoningMixin`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `ReasoningTokenUsageMixin`, `SeparateReasoningMixin`, `CustomTestCase`。

### Lines 27-27: class-level constants and configuration for `TestEnableThinking` / 类级常量与配置
```python
    reasoning_parser_name = "qwen3"
```
**EN:** This block defines shared names such as `reasoning_parser_name`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `reasoning_parser_name` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 29-45: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_ENABLE_THINKING_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-1234"
        cls.init_reasoning_token_verifier()
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
            other_args=[
                "--reasoning-parser",
                "qwen3",
            ],
        )
        cls.additional_chat_kwargs = {}
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 47-49: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 51-73: test case chat completion with reasoning / 测试用例 chat completion with reasoning
```python
    def test_chat_completion_with_reasoning(self):
        # Test non-streaming with "enable_thinking": True, reasoning_content should not be empty
        client = requests.post(
            f"{self.base_url}/v1/chat/completions",
            headers={"Authorization": f"Bearer {self.api_key}"},
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
**EN:** This test exercises `test_chat_completion_with_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion_with_reasoning`。

### Lines 75-98: test case chat completion without reasoning / 测试用例 chat completion without reasoning
```python
    def test_chat_completion_without_reasoning(self):
        # Test non-streaming with "enable_thinking": False, reasoning_content should be empty
        client = requests.post(
            f"{self.base_url}/v1/chat/completions",
            headers={"Authorization": f"Bearer {self.api_key}"},
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
**EN:** This test exercises `test_chat_completion_without_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion_without_reasoning`。

### Lines 100-142: test case stream chat completion with reasoning / 测试用例 stream chat completion with reasoning
```python
    def test_stream_chat_completion_with_reasoning(self):
        # Test streaming with "enable_thinking": True, reasoning_content should not be empty
        response = requests.post(
            f"{self.base_url}/v1/chat/completions",
            headers={"Authorization": f"Bearer {self.api_key}"},
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
**EN:** This test exercises `test_stream_chat_completion_with_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stream_chat_completion_with_reasoning`。

### Lines 144-186: test case stream chat completion without reasoning / 测试用例 stream chat completion without reasoning
```python
    def test_stream_chat_completion_without_reasoning(self):
        # Test streaming with "enable_thinking": False, reasoning_content should be empty
        response = requests.post(
            f"{self.base_url}/v1/chat/completions",
            headers={"Authorization": f"Bearer {self.api_key}"},
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
**EN:** This test exercises `test_stream_chat_completion_without_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stream_chat_completion_without_reasoning`。

### Lines 189-190: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestEnableThinking`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestEnableThinking.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestEnableThinking.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestEnableThinking.test_chat_completion_with_reasoning`: This test exercises `test_chat_completion_with_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion_with_reasoning`。
- `TestEnableThinking.test_chat_completion_without_reasoning`: This test exercises `test_chat_completion_without_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion_without_reasoning`。
- `TestEnableThinking.test_stream_chat_completion_with_reasoning`: This test exercises `test_stream_chat_completion_with_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stream_chat_completion_with_reasoning`。
- `TestEnableThinking.test_stream_chat_completion_without_reasoning`: This test exercises `test_stream_chat_completion_without_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stream_chat_completion_without_reasoning`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kits.reasoning_kit`, `sglang.test.test_utils`

- **Total lines / 总行数**: 190
