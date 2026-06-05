# test_llama32_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/function_call/test_llama32_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates llama32 detector behavior in SGLang's unit / function call area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 函数调用 领域中与 llama32 detector 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for Llama32Detector — no server, no model loading."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-8: module imports and dependencies / 模块导入与依赖
```python
import json

from sglang.srt.entrypoints.openai.protocol import Function, Tool
from sglang.srt.function_call.llama32_detector import Llama32Detector
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.llama32_detector`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.llama32_detector`, `sglang.test.ci.ci_register`。

### Lines 10-10: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(1.0, "base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 13-13: class TestLlama32Detector declaration / 类 TestLlama32Detector 声明
```python
class TestLlama32Detector(CustomTestCase):
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
        self.detector = Llama32Detector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 53-55: supporting source context / 辅助源码上下文
```python

    # ==================== has_tool_call Tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 56-58: test case has tool call with python tag / 测试用例 has tool call with python tag
```python
    def test_has_tool_call_with_python_tag(self):
        text = '<|python_tag|>{"name": "get_weather", "arguments": {"city": "Beijing"}}'
        self.assertTrue(self.detector.has_tool_call(text))
```
**EN:** This test exercises `test_has_tool_call_with_python_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_with_python_tag`。

### Lines 60-62: test case has tool call with json start / 测试用例 has tool call with json start
```python
    def test_has_tool_call_with_json_start(self):
        text = '{"name": "get_weather", "arguments": {"city": "Beijing"}}'
        self.assertTrue(self.detector.has_tool_call(text))
```
**EN:** This test exercises `test_has_tool_call_with_json_start` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_with_json_start`。

### Lines 64-66: test case has tool call false / 测试用例 has tool call false
```python
    def test_has_tool_call_false(self):
        text = "The weather in Beijing is sunny today."
        self.assertFalse(self.detector.has_tool_call(text))
```
**EN:** This test exercises `test_has_tool_call_false` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_false`。

### Lines 67-69: supporting source context / 辅助源码上下文
```python

    # ==================== detect_and_parse Tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 70-76: test case single tool call with python tag / 测试用例 single tool call with python tag
```python
    def test_single_tool_call_with_python_tag(self):
        text = '<|python_tag|>{"name": "get_weather", "arguments": {"city": "Beijing"}}'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
        args = json.loads(result.calls[0].parameters)
        self.assertEqual(args["city"], "Beijing")
```
**EN:** This test exercises `test_single_tool_call_with_python_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_tool_call_with_python_tag`。

### Lines 78-82: test case single tool call without python tag / 测试用例 single tool call without python tag
```python
    def test_single_tool_call_without_python_tag(self):
        text = '{"name": "get_weather", "arguments": {"city": "Beijing"}}'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
```
**EN:** This test exercises `test_single_tool_call_without_python_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_tool_call_without_python_tag`。

### Lines 84-88: test case normal text before python tag / 测试用例 normal text before python tag
```python
    def test_normal_text_before_python_tag(self):
        text = 'Let me check. <|python_tag|>{"name": "get_weather", "arguments": {"city": "Tokyo"}}'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.normal_text, "Let me check. ")
```
**EN:** This test exercises `test_normal_text_before_python_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_normal_text_before_python_tag`。

### Lines 90-94: test case no tool call / 测试用例 no tool call
```python
    def test_no_tool_call(self):
        text = "The weather is nice today."
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 0)
        self.assertEqual(result.normal_text, "The weather is nice today.")
```
**EN:** This test exercises `test_no_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_tool_call`。

### Lines 96-101: test case multiple json objects / 测试用例 multiple json objects
```python
    def test_multiple_json_objects(self):
        text = '<|python_tag|>{"name": "get_weather", "arguments": {"city": "Beijing"}};{"name": "search", "arguments": {"query": "restaurants"}}'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 2)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(result.calls[1].name, "search")
```
**EN:** This test exercises `test_multiple_json_objects` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_json_objects`。

### Lines 103-109: test case tool call with multiple arguments / 测试用例 tool call with multiple arguments
```python
    def test_tool_call_with_multiple_arguments(self):
        text = '<|python_tag|>{"name": "get_weather", "arguments": {"city": "London", "unit": "celsius"}}'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        args = json.loads(result.calls[0].parameters)
        self.assertEqual(args["city"], "London")
        self.assertEqual(args["unit"], "celsius")
```
**EN:** This test exercises `test_tool_call_with_multiple_arguments` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_call_with_multiple_arguments`。

### Lines 110-112: supporting source context / 辅助源码上下文
```python

    # ==================== Python dict conversion Tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 113-117: test case convert python dict to json / 测试用例 convert python dict to json
```python
    def test_convert_python_dict_to_json(self):
        python_dict = "{'name': 'get_weather', 'arguments': {'city': 'Beijing'}}"
        result = self.detector._convert_python_dict_to_json(python_dict)
        parsed = json.loads(result)
        self.assertEqual(parsed["name"], "get_weather")
```
**EN:** This test exercises `test_convert_python_dict_to_json` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_convert_python_dict_to_json`。

### Lines 119-122: test case convert invalid string returns original / 测试用例 convert invalid string returns original
```python
    def test_convert_invalid_string_returns_original(self):
        invalid = "not a dict at all"
        result = self.detector._convert_python_dict_to_json(invalid)
        self.assertEqual(result, invalid)
```
**EN:** This test exercises `test_convert_invalid_string_returns_original` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_convert_invalid_string_returns_original`。

### Lines 123-125: supporting source context / 辅助源码上下文
```python

    # ==================== structure_info Tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 126-131: test case structure info / 测试用例 structure info
```python
    def test_structure_info(self):
        info_func = self.detector.structure_info()
        info = info_func("get_weather")
        self.assertIn("get_weather", info.begin)
        self.assertIn("<|python_tag|>", info.begin)
        self.assertEqual(info.trigger, "<|python_tag|>")
```
**EN:** This test exercises `test_structure_info` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_structure_info`。

### Lines 132-134: supporting source context / 辅助源码上下文
```python

    # ==================== Streaming Tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 135-154: test case streaming single tool call / 测试用例 streaming single tool call
```python
    def test_streaming_single_tool_call(self):
        detector = Llama32Detector()
        chunks = [
            "<|python_",
            'tag|>{"name": "get_weather",',
            ' "arguments": {"city": "Beijing"',
            "}}",
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
**EN:** This test exercises `test_streaming_single_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_single_tool_call`。

### Lines 156-162: test case streaming normal text before tool / 测试用例 streaming normal text before tool
```python
    def test_streaming_normal_text_before_tool(self):
        detector = Llama32Detector()
        result = detector.parse_streaming_increment(
            "Let me check the weather. ", self.tools
        )
        self.assertEqual(result.normal_text, "Let me check the weather. ")
        self.assertEqual(len(result.calls), 0)
```
**EN:** This test exercises `test_streaming_normal_text_before_tool` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_normal_text_before_tool`。

### Lines 164-186: test case streaming text then tool call / 测试用例 streaming text then tool call
```python
    def test_streaming_text_then_tool_call(self):
        detector = Llama32Detector()
        chunks = [
            "I'll look that up. ",
            '<|python_tag|>{"name": "get_weather",',
            ' "arguments": {"city": "Tokyo", "unit": "celsius"',
            "}}",
        ]
        all_calls = []
        all_normal_text = ""
        for chunk in chunks:
            result = detector.parse_streaming_increment(chunk, self.tools)
            all_calls.extend(result.calls)
            all_normal_text += result.normal_text

        self.assertEqual(all_normal_text, "I'll look that up. ")
        func_calls = [c for c in all_calls if c.name]
        self.assertEqual(len(func_calls), 1)
        self.assertEqual(func_calls[0].name, "get_weather")
        full_params = "".join(c.parameters for c in all_calls if c.parameters)
        params = json.loads(full_params)
        self.assertEqual(params["city"], "Tokyo")
        self.assertEqual(params["unit"], "celsius")
```
**EN:** This test exercises `test_streaming_text_then_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_text_then_tool_call`。

### Lines 189-192: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    import unittest

    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestLlama32Detector`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestLlama32Detector.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestLlama32Detector.test_has_tool_call_with_python_tag`: This test exercises `test_has_tool_call_with_python_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_with_python_tag`。
- `TestLlama32Detector.test_has_tool_call_with_json_start`: This test exercises `test_has_tool_call_with_json_start` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_with_json_start`。
- `TestLlama32Detector.test_has_tool_call_false`: This test exercises `test_has_tool_call_false` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_false`。
- `TestLlama32Detector.test_single_tool_call_with_python_tag`: This test exercises `test_single_tool_call_with_python_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_tool_call_with_python_tag`。
- `TestLlama32Detector.test_single_tool_call_without_python_tag`: This test exercises `test_single_tool_call_without_python_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_tool_call_without_python_tag`。
- `TestLlama32Detector.test_normal_text_before_python_tag`: This test exercises `test_normal_text_before_python_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_normal_text_before_python_tag`。
- `TestLlama32Detector.test_no_tool_call`: This test exercises `test_no_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_tool_call`。
- `TestLlama32Detector.test_multiple_json_objects`: This test exercises `test_multiple_json_objects` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_json_objects`。
- `TestLlama32Detector.test_tool_call_with_multiple_arguments`: This test exercises `test_tool_call_with_multiple_arguments` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_call_with_multiple_arguments`。
- `TestLlama32Detector.test_convert_python_dict_to_json`: This test exercises `test_convert_python_dict_to_json` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_convert_python_dict_to_json`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.llama32_detector`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 192
