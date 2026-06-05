# test_hermes_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/function_call/test_hermes_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates hermes detector behavior in SGLang's unit / function call area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 函数调用 领域中与 hermes detector 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for HermesDetector — no server, no model loading."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-8: module imports and dependencies / 模块导入与依赖
```python
import json

from sglang.srt.entrypoints.openai.protocol import Function, Tool
from sglang.srt.function_call.hermes_detector import HermesDetector
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.hermes_detector`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.hermes_detector`, `sglang.test.ci.ci_register`。

### Lines 10-10: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(1.0, "base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 13-13: class TestHermesDetector declaration / 类 TestHermesDetector 声明
```python
class TestHermesDetector(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 14-52: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.tools = [
            Tool(
                type="function",
                function=Function(
                    name="get_weather",
                    description="Get weather information",
                    parameters={
                        "type": "object",
                        "properties": {
                            "city": {"type": "string", "description": "City name"},
                            "unit": {
                                "type": "string",
                                "enum": ["celsius", "fahrenheit"],
                            },
                        },
                        "required": ["city"],
                    },
                ),
            ),
            Tool(
                type="function",
                function=Function(
                    name="search",
                    description="Search the web",
                    parameters={
                        "type": "object",
                        "properties": {
                            "query": {
                                "type": "string",
                                "description": "Search query",
                            },
                        },
                        "required": ["query"],
                    },
                ),
            ),
        ]
        self.detector = HermesDetector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 53-55: supporting source context / 辅助源码上下文
```python

    # ==================== has_tool_call Tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 56-58: test case has tool call true / 测试用例 has tool call true
```python
    def test_has_tool_call_true(self):
        text = '<tool_call>{"name": "get_weather", "arguments": {"city": "Beijing"}}</tool_call>'
        self.assertTrue(self.detector.has_tool_call(text))
```
**EN:** This test exercises `test_has_tool_call_true` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_true`。

### Lines 60-62: test case has tool call false / 测试用例 has tool call false
```python
    def test_has_tool_call_false(self):
        text = "The weather in Beijing is sunny today."
        self.assertFalse(self.detector.has_tool_call(text))
```
**EN:** This test exercises `test_has_tool_call_false` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_false`。

### Lines 63-65: supporting source context / 辅助源码上下文
```python

    # ==================== detect_and_parse Tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 66-73: test case single tool call / 测试用例 single tool call
```python
    def test_single_tool_call(self):
        text = '<tool_call>{"name": "get_weather", "arguments": {"city": "Beijing"}}</tool_call>'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
        args = json.loads(result.calls[0].parameters)
        self.assertEqual(args["city"], "Beijing")
        self.assertEqual(result.normal_text, "")
```
**EN:** This test exercises `test_single_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_tool_call`。

### Lines 75-83: test case multiple tool calls / 测试用例 multiple tool calls
```python
    def test_multiple_tool_calls(self):
        text = (
            '<tool_call>{"name": "get_weather", "arguments": {"city": "Beijing"}}</tool_call>'
            '<tool_call>{"name": "search", "arguments": {"query": "restaurants"}}</tool_call>'
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 2)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(result.calls[1].name, "search")
```
**EN:** This test exercises `test_multiple_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_tool_calls`。

### Lines 85-90: test case tool call with leading text / 测试用例 tool call with leading text
```python
    def test_tool_call_with_leading_text(self):
        text = 'I will check the weather for you. <tool_call>{"name": "get_weather", "arguments": {"city": "Tokyo"}}</tool_call>'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(result.normal_text, "I will check the weather for you.")
```
**EN:** This test exercises `test_tool_call_with_leading_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_call_with_leading_text`。

### Lines 92-96: test case no tool call / 测试用例 no tool call
```python
    def test_no_tool_call(self):
        text = "The weather is nice today."
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 0)
        self.assertEqual(result.normal_text, "The weather is nice today.")
```
**EN:** This test exercises `test_no_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_tool_call`。

### Lines 98-104: test case tool call with multiple arguments / 测试用例 tool call with multiple arguments
```python
    def test_tool_call_with_multiple_arguments(self):
        text = '<tool_call>{"name": "get_weather", "arguments": {"city": "London", "unit": "celsius"}}</tool_call>'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        args = json.loads(result.calls[0].parameters)
        self.assertEqual(args["city"], "London")
        self.assertEqual(args["unit"], "celsius")
```
**EN:** This test exercises `test_tool_call_with_multiple_arguments` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_call_with_multiple_arguments`。

### Lines 106-110: test case malformed json returns original text / 测试用例 malformed json returns original text
```python
    def test_malformed_json_returns_original_text(self):
        text = "<tool_call>not valid json</tool_call>"
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 0)
        self.assertEqual(result.normal_text, text)
```
**EN:** This test exercises `test_malformed_json_returns_original_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_malformed_json_returns_original_text`。

### Lines 111-113: supporting source context / 辅助源码上下文
```python

    # ==================== structure_info Tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 114-119: test case structure info / 测试用例 structure info
```python
    def test_structure_info(self):
        info_func = self.detector.structure_info()
        info = info_func("get_weather")
        self.assertIn("get_weather", info.begin)
        self.assertEqual(info.trigger, "<tool_call>")
        self.assertEqual(info.end, "}</tool_call>")
```
**EN:** This test exercises `test_structure_info` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_structure_info`。

### Lines 120-122: supporting source context / 辅助源码上下文
```python

    # ==================== Streaming Tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 123-144: test case streaming single tool call / 测试用例 streaming single tool call
```python
    def test_streaming_single_tool_call(self):
        detector = HermesDetector()
        chunks = [
            "<tool_",
            'call>{"name": "get_weather",',
            ' "arguments": {"city": "Beijing"',
            "}}</tool_call>",
        ]
        all_calls = []
        for chunk in chunks:
            result = detector.parse_streaming_increment(chunk, self.tools)
            all_calls.extend(result.calls)

        # Verify tool name
        func_calls = [c for c in all_calls if c.name]
        self.assertEqual(len(func_calls), 1)
        self.assertEqual(func_calls[0].name, "get_weather")

        # Verify parameters
        full_params = "".join(c.parameters for c in all_calls if c.parameters)
        params = json.loads(full_params)
        self.assertEqual(params["city"], "Beijing")
```
**EN:** This test exercises `test_streaming_single_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_single_tool_call`。

### Lines 146-150: test case streaming normal text before tool / 测试用例 streaming normal text before tool
```python
    def test_streaming_normal_text_before_tool(self):
        detector = HermesDetector()
        result = detector.parse_streaming_increment("Hello! Let me help. ", self.tools)
        self.assertEqual(result.normal_text, "Hello! Let me help. ")
        self.assertEqual(len(result.calls), 0)
```
**EN:** This test exercises `test_streaming_normal_text_before_tool` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_normal_text_before_tool`。

### Lines 152-173: test case streaming text then tool call / 测试用例 streaming text then tool call
```python
    def test_streaming_text_then_tool_call(self):
        detector = HermesDetector()
        chunks = [
            "Sure, let me check. ",
            '<tool_call>{"name": "get_weather",',
            ' "arguments": {"city": "Tokyo"',
            "}}</tool_call>",
        ]
        all_calls = []
        all_normal_text = ""
        for chunk in chunks:
            result = detector.parse_streaming_increment(chunk, self.tools)
            all_calls.extend(result.calls)
            all_normal_text += result.normal_text

        self.assertEqual(all_normal_text, "Sure, let me check. ")
        func_calls = [c for c in all_calls if c.name]
        self.assertEqual(len(func_calls), 1)
        self.assertEqual(func_calls[0].name, "get_weather")
        full_params = "".join(c.parameters for c in all_calls if c.parameters)
        params = json.loads(full_params)
        self.assertEqual(params["city"], "Tokyo")
```
**EN:** This test exercises `test_streaming_text_then_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_text_then_tool_call`。

### Lines 176-179: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    import unittest

    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestHermesDetector`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestHermesDetector.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestHermesDetector.test_has_tool_call_true`: This test exercises `test_has_tool_call_true` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_true`。
- `TestHermesDetector.test_has_tool_call_false`: This test exercises `test_has_tool_call_false` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_false`。
- `TestHermesDetector.test_single_tool_call`: This test exercises `test_single_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_tool_call`。
- `TestHermesDetector.test_multiple_tool_calls`: This test exercises `test_multiple_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_tool_calls`。
- `TestHermesDetector.test_tool_call_with_leading_text`: This test exercises `test_tool_call_with_leading_text` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_call_with_leading_text`。
- `TestHermesDetector.test_no_tool_call`: This test exercises `test_no_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_tool_call`。
- `TestHermesDetector.test_tool_call_with_multiple_arguments`: This test exercises `test_tool_call_with_multiple_arguments` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_call_with_multiple_arguments`。
- `TestHermesDetector.test_malformed_json_returns_original_text`: This test exercises `test_malformed_json_returns_original_text` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_malformed_json_returns_original_text`。
- `TestHermesDetector.test_structure_info`: This test exercises `test_structure_info` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_structure_info`。
- `TestHermesDetector.test_streaming_single_tool_call`: This test exercises `test_streaming_single_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_single_tool_call`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.hermes_detector`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 179
