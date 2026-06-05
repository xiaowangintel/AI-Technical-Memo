# test_json_mode.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/openai_server/features/test_json_mode.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates json mode behavior in SGLang's openai server / features area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 openai server / features 领域中与 json mode 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: module imports and dependencies / 模块导入与依赖
```python
import json
import unittest

import openai

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_amd_ci,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `unittest`, `openai`, `sglang.srt.utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `unittest`, `openai`, `sglang.srt.utils`。

### Lines 17-18: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=118, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=180, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 21-21: class JSONModeMixin declaration / 类 JSONModeMixin 声明
```python
class JSONModeMixin:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 22-22: supporting statements / 辅助语句
```python
    """Mixin class containing JSON mode test methods"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 24-51: test case json mode response / 测试用例 json mode response
```python
    def test_json_mode_response(self):
        """Test that response_format json_object (also known as "json mode") produces valid JSON, even without a system prompt that mentions JSON."""
        response = self.client.chat.completions.create(
            model=self.model,
            messages=[
                # We are deliberately omitting "That produces JSON" or similar phrases from the assistant prompt so that we don't have misleading test results
                {
                    "role": "system",
                    "content": "You are a helpful AI assistant that gives a short answer.",
                },
                {"role": "user", "content": "What is the capital of Bulgaria?"},
            ],
            temperature=0,
            max_tokens=128,
            response_format={"type": "json_object"},
        )
        text = response.choices[0].message.content

        print(f"Response ({len(text)} characters): {text}")

        # Verify the response is valid JSON
        try:
            js_obj = json.loads(text)
        except json.JSONDecodeError as e:
            self.fail(f"Response is not valid JSON. Error: {e}. Response: {text}")

        # Verify it's actually an object (dict)
        self.assertIsInstance(js_obj, dict, f"Response is not a JSON object: {text}")
```
**EN:** Test that response_format json_object (also known as "json mode") produces valid JSON, even without a system prompt that mentions JSON. This test exercises `test_json_mode_response` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that response_format json_object (also known as "json mode") produces valid JSON, even without a system prompt that mentions JSON. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_json_mode_response`。

### Lines 53-90: test case json mode with streaming / 测试用例 json mode with streaming
```python
    def test_json_mode_with_streaming(self):
        """Test that streaming with json_object response (also known as "json mode") format works correctly, even without a system prompt that mentions JSON."""
        stream = self.client.chat.completions.create(
            model=self.model,
            messages=[
                # We are deliberately omitting "That produces JSON" or similar phrases from the assistant prompt so that we don't have misleading test results
                {
                    "role": "system",
                    "content": "You are a helpful AI assistant that gives a short answer.",
                },
                {"role": "user", "content": "What is the capital of Bulgaria?"},
            ],
            temperature=0,
            max_tokens=128,
            response_format={"type": "json_object"},
            stream=True,
        )

        # Collect all chunks
        chunks = []
        for chunk in stream:
            if chunk.choices[0].delta.content is not None:
                chunks.append(chunk.choices[0].delta.content)
        full_response = "".join(chunks)

        print(
            f"Concatenated Response ({len(full_response)} characters): {full_response}"
        )

        # Verify the combined response is valid JSON
        try:
            js_obj = json.loads(full_response)
        except json.JSONDecodeError as e:
            self.fail(
                f"Streamed response is not valid JSON. Error: {e}. Response: {full_response}"
            )

        self.assertIsInstance(js_obj, dict)
```
**EN:** Test that streaming with json_object response (also known as "json mode") format works correctly, even without a system prompt that mentions JSON. This test exercises `test_json_mode_with_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that streaming with json_object response (also known as "json mode") format works correctly, even without a system prompt that mentions JSON. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_json_mode_with_streaming`。

### Lines 93-93: class ServerWithGrammarBackend declaration / 类 ServerWithGrammarBackend 声明
```python
class ServerWithGrammarBackend(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 94-96: class-level constants and configuration for `ServerWithGrammarBackend` / 类级常量与配置
```python
    """Base class for tests requiring a grammar backend server"""

    backend = "xgrammar"
```
**EN:** This block defines shared names such as `backend`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `backend` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 98-119: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST

        other_args = [
            "--max-running-requests",
            "10",
            "--grammar-backend",
            cls.backend,
        ]

        if is_in_amd_ci():
            other_args.append("--constrained-json-disable-any-whitespace")

        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=other_args,
        )
        cls.client = openai.Client(api_key="EMPTY", base_url=f"{cls.base_url}/v1")
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 121-123: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 126-126: class TestJSONModeXGrammar declaration / 类 TestJSONModeXGrammar 声明
```python
class TestJSONModeXGrammar(ServerWithGrammarBackend, JSONModeMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `ServerWithGrammarBackend`, `JSONModeMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `ServerWithGrammarBackend`, `JSONModeMixin`。

### Lines 127-127: class-level constants and configuration for `TestJSONModeXGrammar` / 类级常量与配置
```python
    backend = "xgrammar"
```
**EN:** This block defines shared names such as `backend`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `backend` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 130-130: class TestJSONModeOutlines declaration / 类 TestJSONModeOutlines 声明
```python
class TestJSONModeOutlines(ServerWithGrammarBackend, JSONModeMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `ServerWithGrammarBackend`, `JSONModeMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `ServerWithGrammarBackend`, `JSONModeMixin`。

### Lines 131-131: class-level constants and configuration for `TestJSONModeOutlines` / 类级常量与配置
```python
    backend = "outlines"
```
**EN:** This block defines shared names such as `backend`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `backend` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 134-134: class TestJSONModeLLGuidance declaration / 类 TestJSONModeLLGuidance 声明
```python
class TestJSONModeLLGuidance(ServerWithGrammarBackend, JSONModeMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `ServerWithGrammarBackend`, `JSONModeMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `ServerWithGrammarBackend`, `JSONModeMixin`。

### Lines 135-135: class-level constants and configuration for `TestJSONModeLLGuidance` / 类级常量与配置
```python
    backend = "llguidance"
```
**EN:** This block defines shared names such as `backend`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `backend` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 138-139: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `JSONModeMixin`: Mixin class containing JSON mode test methods / 用于组织相关测试、夹具或辅助方法。
- `ServerWithGrammarBackend`: Base class for tests requiring a grammar backend server / 用于组织相关测试、夹具或辅助方法。
- `TestJSONModeXGrammar`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestJSONModeOutlines`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestJSONModeLLGuidance`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `JSONModeMixin.test_json_mode_response`: Test that response_format json_object (also known as "json mode") produces valid JSON, even without a system prompt that mentions JSON. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_json_mode_response`。
- `JSONModeMixin.test_json_mode_with_streaming`: Test that streaming with json_object response (also known as "json mode") format works correctly, even without a system prompt that mentions JSON. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_json_mode_with_streaming`。
- `ServerWithGrammarBackend.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `ServerWithGrammarBackend.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest`
- **Third-party modules / 第三方模块**: `openai`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 139
