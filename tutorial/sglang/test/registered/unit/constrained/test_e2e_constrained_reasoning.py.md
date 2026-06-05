# test_e2e_constrained_reasoning.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/constrained/test_e2e_constrained_reasoning.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates e2e constrained reasoning behavior in SGLang's unit / constrained area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / constrained 领域中与 e2e constrained reasoning 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: supporting statements / 辅助语句
```python
"""
End-to-end tests for strict reasoning + constrained decoding.

Tests that the full pipeline works:
- AC-5.1: Strict reasoning + JSON schema constrained generation
- AC-5.2: Strict reasoning + tool call parsing (basic validation only)

These tests launch a real server with a small model and verify
the constrained decoding pipeline produces valid output.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 12-23: module imports and dependencies / 模块导入与依赖
```python
import json
import unittest

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `unittest`, `requests`, `sglang.srt.utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `unittest`, `requests`, `sglang.srt.utils`。

### Lines 25-29: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=120, stage="base-b", runner_config="1-gpu-small")

MODEL = "Qwen/Qwen3-0.6B"
BASE_URL = "http://127.0.0.1:39877"
API_KEY = "sk-test-1234"
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 32-32: class TestConstrainedReasoningE2E declaration / 类 TestConstrainedReasoningE2E 声明
```python
class TestConstrainedReasoningE2E(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 33-47: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = MODEL
        cls.base_url = BASE_URL
        cls.api_key = API_KEY
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
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 49-51: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 53-73: method chat / 方法 chat
```python
    def _chat(self, **kwargs):
        default = {
            "model": self.model,
            "messages": [
                {
                    "role": "user",
                    "content": "What is 2+2? Answer with just the number.",
                }
            ],
            "temperature": 0,
            "max_tokens": 256,
        }
        default.update(kwargs)
        resp = requests.post(
            f"{self.base_url}/v1/chat/completions",
            headers={"Authorization": f"Bearer {self.api_key}"},
            json=default,
            timeout=60,
        )
        self.assertEqual(resp.status_code, 200, f"Request failed: {resp.text}")
        return resp.json()
```
**EN:** This block implements `_chat` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_chat`，承担模块行为中的一个聚焦逻辑片段。

### Lines 75-115: test case reasoning with json schema / 测试用例 reasoning with json schema
```python
    def test_reasoning_with_json_schema(self):
        """AC-5.1: Reasoning + JSON schema produces valid JSON output."""
        schema = {
            "type": "object",
            "properties": {
                "answer": {"type": "integer"},
            },
            "required": ["answer"],
        }
        data = self._chat(
            response_format={
                "type": "json_schema",
                "json_schema": {
                    "name": "answer_schema",
                    "schema": schema,
                },
            },
            chat_template_kwargs={"enable_thinking": True},
            separate_reasoning=True,
        )

        choice = data["choices"][0]
        content = choice["message"]["content"] or ""

        # Content should be valid JSON conforming to schema when non-empty.
        # With small models + separate_reasoning, content may be empty if the
        # model puts everything in reasoning_content. That's acceptable.
        if content.strip():
            try:
                parsed = json.loads(content)
                self.assertIn("answer", parsed)
                self.assertIsInstance(parsed["answer"], int)
            except (json.JSONDecodeError, TypeError):
                # Small models may produce imperfect JSON
                self.assertTrue(
                    content.strip().startswith("{"),
                    f"Expected JSON-like output, got: {content!r}",
                )

        # Content should NOT contain <think> tags (those go to reasoning_content)
        self.assertNotIn("<think>", content)
```
**EN:** AC-5.1: Reasoning + JSON schema produces valid JSON output. This test exercises `test_reasoning_with_json_schema` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** AC-5.1: Reasoning + JSON schema produces valid JSON output. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reasoning_with_json_schema`。

### Lines 117-142: test case reasoning disabled with json schema / 测试用例 reasoning disabled with json schema
```python
    def test_reasoning_disabled_with_json_schema(self):
        """JSON schema still works when reasoning is explicitly disabled."""
        schema = {
            "type": "object",
            "properties": {
                "answer": {"type": "integer"},
            },
            "required": ["answer"],
        }
        data = self._chat(
            response_format={
                "type": "json_schema",
                "json_schema": {
                    "name": "answer_schema",
                    "schema": schema,
                },
            },
            chat_template_kwargs={"enable_thinking": False},
        )

        choice = data["choices"][0]
        content = choice["message"]["content"]

        # Should still produce valid JSON
        parsed = json.loads(content)
        self.assertIn("answer", parsed)
```
**EN:** JSON schema still works when reasoning is explicitly disabled. This test exercises `test_reasoning_disabled_with_json_schema` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** JSON schema still works when reasoning is explicitly disabled. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reasoning_disabled_with_json_schema`。

### Lines 144-157: test case reasoning with separate output / 测试用例 reasoning with separate output
```python
    def test_reasoning_with_separate_output(self):
        """Reasoning content is correctly separated from normal content."""
        data = self._chat(
            chat_template_kwargs={"enable_thinking": True},
            separate_reasoning=True,
        )

        choice = data["choices"][0]
        content = choice["message"]["content"]
        reasoning = choice["message"].get("reasoning_content")

        # Content should not contain think tags
        self.assertNotIn("<think>", content)
        self.assertNotIn("</think>", content)
```
**EN:** Reasoning content is correctly separated from normal content. This test exercises `test_reasoning_with_separate_output` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Reasoning content is correctly separated from normal content. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reasoning_with_separate_output`。

### Lines 159-195: test case tool call after reasoning / 测试用例 tool call after reasoning
```python
    def test_tool_call_after_reasoning(self):
        """AC-5.2: Tool call parsing works with reasoning enabled."""
        tools = [
            {
                "type": "function",
                "function": {
                    "name": "get_weather",
                    "description": "Get the current weather",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "location": {"type": "string"},
                        },
                        "required": ["location"],
                    },
                },
            }
        ]
        data = self._chat(
            messages=[
                {
                    "role": "user",
                    "content": "What's the weather in Paris?",
                }
            ],
            tools=tools,
            chat_template_kwargs={"enable_thinking": True},
            separate_reasoning=True,
        )

        choice = data["choices"][0]
        # The model may or may not produce tool calls (depends on model capability)
        # but the response should be well-formed (no crashes)
        self.assertIn("message", choice)
        self.assertIn("finish_reason", choice)
        # finish_reason should be either "stop" or "tool_calls"
        self.assertIn(choice["finish_reason"], ["stop", "tool_calls", "length"])
```
**EN:** AC-5.2: Tool call parsing works with reasoning enabled. This test exercises `test_tool_call_after_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** AC-5.2: Tool call parsing works with reasoning enabled. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_call_after_reasoning`。

### Lines 198-198: class TestStrictThinkingE2E declaration / 类 TestStrictThinkingE2E 声明
```python
class TestStrictThinkingE2E(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 199-204: supporting statements / 辅助语句
```python
    """E2E tests with --enable-strict-thinking flag.

    Validates that the strict thinking flag is correctly propagated through
    the full pipeline: server_args -> grammar_backend -> ReasonerGrammarBackend
    -> token filtering during thinking phase.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 206-221: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = MODEL
        cls.base_url = "http://127.0.0.1:39878"
        cls.api_key = API_KEY
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
            other_args=[
                "--reasoning-parser",
                "qwen3",
                "--enable-strict-thinking",
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 223-225: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 227-247: method chat / 方法 chat
```python
    def _chat(self, **kwargs):
        default = {
            "model": self.model,
            "messages": [
                {
                    "role": "user",
                    "content": "What is 2+2? Answer with just the number.",
                }
            ],
            "temperature": 0,
            "max_tokens": 256,
        }
        default.update(kwargs)
        resp = requests.post(
            f"{self.base_url}/v1/chat/completions",
            headers={"Authorization": f"Bearer {self.api_key}"},
            json=default,
            timeout=60,
        )
        self.assertEqual(resp.status_code, 200, f"Request failed: {resp.text}")
        return resp.json()
```
**EN:** This block implements `_chat` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_chat`，承担模块行为中的一个聚焦逻辑片段。

### Lines 249-284: test case strict thinking with json schema / 测试用例 strict thinking with json schema
```python
    def test_strict_thinking_with_json_schema(self):
        """Strict thinking + JSON schema: server starts and produces valid output."""
        schema = {
            "type": "object",
            "properties": {
                "answer": {"type": "integer"},
            },
            "required": ["answer"],
        }
        data = self._chat(
            response_format={
                "type": "json_schema",
                "json_schema": {
                    "name": "answer_schema",
                    "schema": schema,
                },
            },
            chat_template_kwargs={"enable_thinking": True},
            separate_reasoning=True,
        )

        choice = data["choices"][0]
        content = choice["message"]["content"] or ""

        if content.strip():
            try:
                parsed = json.loads(content)
                self.assertIn("answer", parsed)
            except (json.JSONDecodeError, TypeError):
                self.assertTrue(
                    content.strip().startswith("{"),
                    f"Expected JSON-like output, got: {content!r}",
                )

        # Think tags must not leak into content
        self.assertNotIn("<think>", content)
```
**EN:** Strict thinking + JSON schema: server starts and produces valid output. This test exercises `test_strict_thinking_with_json_schema` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Strict thinking + JSON schema: server starts and produces valid output. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_strict_thinking_with_json_schema`。

### Lines 286-296: test case strict thinking disabled per request / 测试用例 strict thinking disabled per request
```python
    def test_strict_thinking_disabled_per_request(self):
        """When thinking is disabled per-request, strict server still works."""
        data = self._chat(
            chat_template_kwargs={"enable_thinking": False},
        )

        choice = data["choices"][0]
        self.assertIn("message", choice)
        self.assertIn("finish_reason", choice)
        # Should complete normally without errors
        self.assertIn(choice["finish_reason"], ["stop", "length"])
```
**EN:** When thinking is disabled per-request, strict server still works. This test exercises `test_strict_thinking_disabled_per_request` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When thinking is disabled per-request, strict server still works. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_strict_thinking_disabled_per_request`。

### Lines 298-310: test case strict thinking separate reasoning / 测试用例 strict thinking separate reasoning
```python
    def test_strict_thinking_separate_reasoning(self):
        """Strict thinking with separate_reasoning produces well-formed output."""
        data = self._chat(
            chat_template_kwargs={"enable_thinking": True},
            separate_reasoning=True,
        )

        choice = data["choices"][0]
        content = choice["message"]["content"] or ""

        # Think tags must not leak into content
        self.assertNotIn("<think>", content)
        self.assertNotIn("</think>", content)
```
**EN:** Strict thinking with separate_reasoning produces well-formed output. This test exercises `test_strict_thinking_separate_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Strict thinking with separate_reasoning produces well-formed output. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_strict_thinking_separate_reasoning`。

### Lines 313-314: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestConstrainedReasoningE2E`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestStrictThinkingE2E`: E2E tests with --enable-strict-thinking flag. / 用于组织相关测试、夹具或辅助方法。
- `TestConstrainedReasoningE2E.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestConstrainedReasoningE2E.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestConstrainedReasoningE2E._chat`: This block implements `_chat` and captures one focused piece of the module's behavior. / 该代码块实现 `_chat`，承担模块行为中的一个聚焦逻辑片段。
- `TestConstrainedReasoningE2E.test_reasoning_with_json_schema`: AC-5.1: Reasoning + JSON schema produces valid JSON output. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reasoning_with_json_schema`。
- `TestConstrainedReasoningE2E.test_reasoning_disabled_with_json_schema`: JSON schema still works when reasoning is explicitly disabled. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reasoning_disabled_with_json_schema`。
- `TestConstrainedReasoningE2E.test_reasoning_with_separate_output`: Reasoning content is correctly separated from normal content. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reasoning_with_separate_output`。
- `TestConstrainedReasoningE2E.test_tool_call_after_reasoning`: AC-5.2: Tool call parsing works with reasoning enabled. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_call_after_reasoning`。
- `TestStrictThinkingE2E.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestStrictThinkingE2E.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestStrictThinkingE2E._chat`: This block implements `_chat` and captures one focused piece of the module's behavior. / 该代码块实现 `_chat`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 314
