# test_mistral_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/function_call/test_mistral_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates mistral detector behavior in SGLang's unit / function call area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 函数调用 领域中与 mistral detector 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for MistralDetector — no server, no model loading."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-8: module imports and dependencies / 模块导入与依赖
```python
import json

from sglang.srt.entrypoints.openai.protocol import Function, Tool
from sglang.srt.function_call.mistral_detector import MistralDetector
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.mistral_detector`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.mistral_detector`, `sglang.test.ci.ci_register`。

### Lines 10-10: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(1.0, "base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 13-13: class TestMistralDetector declaration / 类 TestMistralDetector 声明
```python
class TestMistralDetector(CustomTestCase):
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
        self.detector = MistralDetector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 53-55: supporting source context / 辅助源码上下文
```python

    # ==================== has_tool_call Tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 56-60: test case has tool call json array format / 测试用例 has tool call json array format
```python
    def test_has_tool_call_json_array_format(self):
        text = (
            '[TOOL_CALLS] [{"name": "get_weather", "arguments": {"city": "Beijing"}}]'
        )
        self.assertTrue(self.detector.has_tool_call(text))
```
**EN:** This test exercises `test_has_tool_call_json_array_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_json_array_format`。

### Lines 62-64: test case has tool call compact format / 测试用例 has tool call compact format
```python
    def test_has_tool_call_compact_format(self):
        text = '[TOOL_CALLS]get_weather[ARGS]{"city": "Beijing"}'
        self.assertTrue(self.detector.has_tool_call(text))
```
**EN:** This test exercises `test_has_tool_call_compact_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_compact_format`。

### Lines 66-68: test case has tool call false / 测试用例 has tool call false
```python
    def test_has_tool_call_false(self):
        text = "The weather in Beijing is sunny today."
        self.assertFalse(self.detector.has_tool_call(text))
```
**EN:** This test exercises `test_has_tool_call_false` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_false`。

### Lines 69-71: supporting source context / 辅助源码上下文
```python

    # ==================== JSON Array Format Tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 72-81: test case json array single tool call / 测试用例 json array single tool call
```python
    def test_json_array_single_tool_call(self):
        text = (
            '[TOOL_CALLS] [{"name": "get_weather", "arguments": {"city": "Beijing"}}]'
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
        args = json.loads(result.calls[0].parameters)
        self.assertEqual(args["city"], "Beijing")
        self.assertEqual(result.normal_text, "")
```
**EN:** This test exercises `test_json_array_single_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_json_array_single_tool_call`。

### Lines 83-88: test case json array multiple tool calls / 测试用例 json array multiple tool calls
```python
    def test_json_array_multiple_tool_calls(self):
        text = '[TOOL_CALLS] [{"name": "get_weather", "arguments": {"city": "Beijing"}}, {"name": "search", "arguments": {"query": "restaurants"}}]'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 2)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(result.calls[1].name, "search")
```
**EN:** This test exercises `test_json_array_multiple_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_json_array_multiple_tool_calls`。

### Lines 90-94: test case json array with leading text / 测试用例 json array with leading text
```python
    def test_json_array_with_leading_text(self):
        text = 'I will check. [TOOL_CALLS] [{"name": "get_weather", "arguments": {"city": "Tokyo"}}]'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.normal_text, "I will check.")
```
**EN:** This test exercises `test_json_array_with_leading_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_json_array_with_leading_text`。

### Lines 95-97: supporting source context / 辅助源码上下文
```python

    # ==================== Compact Format Tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 98-104: test case compact format single tool call / 测试用例 compact format single tool call
```python
    def test_compact_format_single_tool_call(self):
        text = '[TOOL_CALLS]get_weather[ARGS]{"city": "Beijing"}'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
        args = json.loads(result.calls[0].parameters)
        self.assertEqual(args["city"], "Beijing")
```
**EN:** This test exercises `test_compact_format_single_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_compact_format_single_tool_call`。

### Lines 106-110: test case compact format with leading text / 测试用例 compact format with leading text
```python
    def test_compact_format_with_leading_text(self):
        text = 'Let me help. [TOOL_CALLS]get_weather[ARGS]{"city": "Tokyo"}'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.normal_text, "Let me help.")
```
**EN:** This test exercises `test_compact_format_with_leading_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_compact_format_with_leading_text`。

### Lines 111-113: supporting source context / 辅助源码上下文
```python

    # ==================== No Tool Call Tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 114-118: test case no tool call / 测试用例 no tool call
```python
    def test_no_tool_call(self):
        text = "The weather is nice today."
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 0)
        self.assertEqual(result.normal_text, "The weather is nice today.")
```
**EN:** This test exercises `test_no_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_tool_call`。

### Lines 119-121: supporting source context / 辅助源码上下文
```python

    # ==================== Edge Cases ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 122-127: test case tool call with nested json / 测试用例 tool call with nested json
```python
    def test_tool_call_with_nested_json(self):
        text = '[TOOL_CALLS] [{"name": "get_weather", "arguments": {"city": "Beijing", "options": {"detailed": true}}}]'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        args = json.loads(result.calls[0].parameters)
        self.assertEqual(args["options"]["detailed"], True)
```
**EN:** This test exercises `test_tool_call_with_nested_json` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_call_with_nested_json`。

### Lines 129-133: test case json array with invalid json / 测试用例 json array with invalid json
```python
    def test_json_array_with_invalid_json(self):
        text = "[TOOL_CALLS] [not valid json]"
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 0)
        self.assertEqual(result.normal_text, "")
```
**EN:** This test exercises `test_json_array_with_invalid_json` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_json_array_with_invalid_json`。

### Lines 134-136: supporting source context / 辅助源码上下文
```python

    # ==================== Internal Methods Tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 137-145: test case extract json array / 测试用例 extract json array
```python
    def test_extract_json_array(self):
        text = (
            '[TOOL_CALLS] [{"name": "get_weather", "arguments": {"city": "Beijing"}}]'
        )
        result = self.detector._extract_json_array(text)
        self.assertIsNotNone(result)
        parsed = json.loads(result)
        self.assertEqual(len(parsed), 1)
        self.assertEqual(parsed[0]["name"], "get_weather")
```
**EN:** This test exercises `test_extract_json_array` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_extract_json_array`。

### Lines 147-154: test case extract json array nested brackets / 测试用例 extract json array nested brackets
```python
    def test_extract_json_array_nested_brackets(self):
        text = (
            '[TOOL_CALLS] [{"name": "get_weather", "arguments": {"tags": ["a", "b"]}}]'
        )
        result = self.detector._extract_json_array(text)
        self.assertIsNotNone(result)
        parsed = json.loads(result)
        self.assertEqual(parsed[0]["arguments"]["tags"], ["a", "b"])
```
**EN:** This test exercises `test_extract_json_array_nested_brackets` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_extract_json_array_nested_brackets`。

### Lines 156-159: test case extract json array no marker / 测试用例 extract json array no marker
```python
    def test_extract_json_array_no_marker(self):
        text = "no tool calls here"
        result = self.detector._extract_json_array(text)
        self.assertIsNone(result)
```
**EN:** This test exercises `test_extract_json_array_no_marker` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_extract_json_array_no_marker`。

### Lines 160-162: supporting source context / 辅助源码上下文
```python

    # ==================== structure_info Tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 163-168: test case structure info / 测试用例 structure info
```python
    def test_structure_info(self):
        info_func = self.detector.structure_info()
        info = info_func("get_weather")
        self.assertIn("get_weather", info.begin)
        self.assertIn("[TOOL_CALLS]", info.trigger)
        self.assertEqual(info.end, "}]")
```
**EN:** This test exercises `test_structure_info` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_structure_info`。

### Lines 169-171: supporting source context / 辅助源码上下文
```python

    # ==================== Streaming Tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 172-190: test case streaming compact format / 测试用例 streaming compact format
```python
    def test_streaming_compact_format(self):
        detector = MistralDetector()
        chunks = [
            "[TOOL_",
            "CALLS]get_weather",
            '[ARGS]{"city": "Beijing"}',
        ]
        all_calls = []
        for chunk in chunks:
            result = detector.parse_streaming_increment(chunk, self.tools)
            all_calls.extend(result.calls)

        func_calls = [c for c in all_calls if c.name]
        self.assertEqual(len(func_calls), 1)
        self.assertEqual(func_calls[0].name, "get_weather")

        full_params = "".join(c.parameters for c in all_calls if c.parameters)
        params = json.loads(full_params)
        self.assertEqual(params["city"], "Beijing")
```
**EN:** This test exercises `test_streaming_compact_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_compact_format`。

### Lines 192-196: test case streaming normal text before tool / 测试用例 streaming normal text before tool
```python
    def test_streaming_normal_text_before_tool(self):
        detector = MistralDetector()
        result = detector.parse_streaming_increment("Let me check. ", self.tools)
        self.assertEqual(result.normal_text, "Let me check. ")
        self.assertEqual(len(result.calls), 0)
```
**EN:** This test exercises `test_streaming_normal_text_before_tool` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_normal_text_before_tool`。

### Lines 198-218: test case streaming text then tool call / 测试用例 streaming text then tool call
```python
    def test_streaming_text_then_tool_call(self):
        detector = MistralDetector()
        chunks = [
            "Sure! ",
            "[TOOL_CALLS]get_weather",
            '[ARGS]{"city": "Tokyo"}',
        ]
        all_calls = []
        all_normal_text = ""
        for chunk in chunks:
            result = detector.parse_streaming_increment(chunk, self.tools)
            all_calls.extend(result.calls)
            all_normal_text += result.normal_text

        self.assertEqual(all_normal_text, "Sure! ")
        func_calls = [c for c in all_calls if c.name]
        self.assertEqual(len(func_calls), 1)
        self.assertEqual(func_calls[0].name, "get_weather")
        full_params = "".join(c.parameters for c in all_calls if c.parameters)
        params = json.loads(full_params)
        self.assertEqual(params["city"], "Tokyo")
```
**EN:** This test exercises `test_streaming_text_then_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_text_then_tool_call`。

### Lines 221-224: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    import unittest

    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestMistralDetector`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestMistralDetector.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestMistralDetector.test_has_tool_call_json_array_format`: This test exercises `test_has_tool_call_json_array_format` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_json_array_format`。
- `TestMistralDetector.test_has_tool_call_compact_format`: This test exercises `test_has_tool_call_compact_format` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_compact_format`。
- `TestMistralDetector.test_has_tool_call_false`: This test exercises `test_has_tool_call_false` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_false`。
- `TestMistralDetector.test_json_array_single_tool_call`: This test exercises `test_json_array_single_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_json_array_single_tool_call`。
- `TestMistralDetector.test_json_array_multiple_tool_calls`: This test exercises `test_json_array_multiple_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_json_array_multiple_tool_calls`。
- `TestMistralDetector.test_json_array_with_leading_text`: This test exercises `test_json_array_with_leading_text` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_json_array_with_leading_text`。
- `TestMistralDetector.test_compact_format_single_tool_call`: This test exercises `test_compact_format_single_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_compact_format_single_tool_call`。
- `TestMistralDetector.test_compact_format_with_leading_text`: This test exercises `test_compact_format_with_leading_text` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_compact_format_with_leading_text`。
- `TestMistralDetector.test_no_tool_call`: This test exercises `test_no_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_tool_call`。
- `TestMistralDetector.test_tool_call_with_nested_json`: This test exercises `test_tool_call_with_nested_json` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_call_with_nested_json`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.mistral_detector`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 224
