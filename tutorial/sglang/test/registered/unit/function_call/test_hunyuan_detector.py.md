# test_hunyuan_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/function_call/test_hunyuan_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates hunyuan detector behavior in SGLang's unit / function call area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 函数调用 领域中与 hunyuan detector 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for HunyuanDetector - no server, no model loading."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-9: module imports and dependencies / 模块导入与依赖
```python
import json
import unittest

from sglang.srt.entrypoints.openai.protocol import Function, Tool
from sglang.srt.function_call.hunyuan_detector import HunyuanDetector
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `unittest`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.hunyuan_detector`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `unittest`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.hunyuan_detector`。

### Lines 11-11: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=7, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 14-75: function make tools / 函数 make tools
```python
def _make_tools():
    return [
        Tool(
            type="function",
            function=Function(
                name="get_current_date",
                description="Get the current date",
                parameters={},
            ),
        ),
        Tool(
            type="function",
            function=Function(
                name="get_weather",
                description="Get weather information",
                parameters={
                    "type": "object",
                    "properties": {
                        "city": {"type": "string", "description": "City name"},
                        "date": {"type": "string", "description": "Date"},
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
                        "count": {
                            "type": "integer",
                            "description": "Number of results",
                        },
                    },
                    "required": ["query"],
                },
            ),
        ),
        Tool(
            type="function",
            function=Function(
                name="calculate",
                description="Calculate expression",
                parameters={
                    "type": "object",
                    "properties": {
                        "expression": {"type": "string"},
                        "precision": {"type": "number"},
                        "verbose": {"type": "boolean"},
                    },
                },
            ),
        ),
    ]
```
**EN:** This block implements `_make_tools` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_tools`，承担模块行为中的一个聚焦逻辑片段。

### Lines 78-78: class TestHunyuanDetectorHasToolCall declaration / 类 TestHunyuanDetectorHasToolCall 声明
```python
class TestHunyuanDetectorHasToolCall(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 79-80: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.detector = HunyuanDetector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 82-86: test case has tool call true / 测试用例 has tool call true
```python
    def test_has_tool_call_true(self):
        text = (
            "<tool_calls><tool_call>get_current_date<tool_sep></tool_call></tool_calls>"
        )
        self.assertTrue(self.detector.has_tool_call(text))
```
**EN:** This test exercises `test_has_tool_call_true` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_true`。

### Lines 88-91: test case has tool call false / 测试用例 has tool call false
```python
    def test_has_tool_call_false(self):
        self.assertFalse(
            self.detector.has_tool_call("The weather in Beijing is sunny today.")
        )
```
**EN:** This test exercises `test_has_tool_call_false` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_false`。

### Lines 93-95: test case has tool call partial tag / 测试用例 has tool call partial tag
```python
    def test_has_tool_call_partial_tag(self):
        self.assertFalse(self.detector.has_tool_call("<tool_call>"))
        self.assertFalse(self.detector.has_tool_call("<tool_call"))
```
**EN:** This test exercises `test_has_tool_call_partial_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_partial_tag`。

### Lines 97-100: test case has tool call with surrounding text / 测试用例 has tool call with surrounding text
```python
    def test_has_tool_call_with_surrounding_text(self):
        self.assertTrue(
            self.detector.has_tool_call("text before <tool_calls> text after")
        )
```
**EN:** This test exercises `test_has_tool_call_with_surrounding_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_with_surrounding_text`。

### Lines 103-103: class TestHunyuanDetectorDetectAndParse declaration / 类 TestHunyuanDetectorDetectAndParse 声明
```python
class TestHunyuanDetectorDetectAndParse(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 104-106: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.tools = _make_tools()
        self.detector = HunyuanDetector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 108-112: test case no tool call / 测试用例 no tool call
```python
    def test_no_tool_call(self):
        text = "This is a plain response."
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 0)
        self.assertEqual(result.normal_text, text)
```
**EN:** This test exercises `test_no_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_tool_call`。

### Lines 114-121: test case zero arg inline / 测试用例 zero arg inline
```python
    def test_zero_arg_inline(self):
        text = (
            "<tool_calls><tool_call>get_current_date<tool_sep></tool_call></tool_calls>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_current_date")
        self.assertEqual(json.loads(result.calls[0].parameters), {})
```
**EN:** This test exercises `test_zero_arg_inline` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_zero_arg_inline`。

### Lines 123-132: test case zero arg newline / 测试用例 zero arg newline
```python
    def test_zero_arg_newline(self):
        text = (
            "<tool_calls>\n"
            "<tool_call>get_current_date<tool_sep>\n"
            "</tool_call>\n"
            "</tool_calls>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_current_date")
```
**EN:** This test exercises `test_zero_arg_newline` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_zero_arg_newline`。

### Lines 134-143: test case single string arg / 测试用例 single string arg
```python
    def test_single_string_arg(self):
        text = (
            "<tool_calls><tool_call>get_weather<tool_sep>"
            "<arg_key>city</arg_key><arg_value>Beijing</arg_value>"
            "</tool_call></tool_calls>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        args = json.loads(result.calls[0].parameters)
        self.assertEqual(args, {"city": "Beijing"})
```
**EN:** This test exercises `test_single_string_arg` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_string_arg`。

### Lines 145-155: test case multiple args same line / 测试用例 multiple args same line
```python
    def test_multiple_args_same_line(self):
        text = (
            "<tool_calls><tool_call>get_weather<tool_sep>"
            "<arg_key>city</arg_key><arg_value>Beijing</arg_value>"
            "<arg_key>date</arg_key><arg_value>2026-03-30</arg_value>"
            "</tool_call></tool_calls>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        args = json.loads(result.calls[0].parameters)
        self.assertEqual(args["city"], "Beijing")
        self.assertEqual(args["date"], "2026-03-30")
```
**EN:** This test exercises `test_multiple_args_same_line` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_args_same_line`。

### Lines 157-171: test case args with newlines / 测试用例 args with newlines
```python
    def test_args_with_newlines(self):
        text = (
            "<tool_calls>\n"
            "<tool_call>get_weather<tool_sep>\n"
            "<arg_key>city</arg_key>\n"
            "<arg_value>Beijing</arg_value>\n"
            "<arg_key>date</arg_key>\n"
            "<arg_value>2026-03-30</arg_value>\n"
            "</tool_call>\n"
            "</tool_calls>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        args = json.loads(result.calls[0].parameters)
        self.assertEqual(args["city"], "Beijing")
        self.assertEqual(args["date"], "2026-03-30")
```
**EN:** This test exercises `test_args_with_newlines` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_args_with_newlines`。

### Lines 173-183: test case content before tool call / 测试用例 content before tool call
```python
    def test_content_before_tool_call(self):
        text = (
            "Checking."
            "<tool_calls>\n"
            "<tool_call>get_current_date<tool_sep>\n"
            "</tool_call>\n"
            "</tool_calls>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.normal_text, "Checking.")
```
**EN:** This test exercises `test_content_before_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_content_before_tool_call`。

### Lines 185-199: test case multiple tool calls / 测试用例 multiple tool calls
```python
    def test_multiple_tool_calls(self):
        text = (
            "<tool_calls>\n"
            "<tool_call>get_weather<tool_sep>\n"
            "<arg_key>city</arg_key>\n<arg_value>Beijing</arg_value>\n"
            "</tool_call>\n"
            "<tool_call>get_weather<tool_sep>\n"
            "<arg_key>city</arg_key>\n<arg_value>Hangzhou</arg_value>\n"
            "</tool_call>\n"
            "</tool_calls>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 2)
        self.assertEqual(json.loads(result.calls[0].parameters)["city"], "Beijing")
        self.assertEqual(json.loads(result.calls[1].parameters)["city"], "Hangzhou")
```
**EN:** This test exercises `test_multiple_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_tool_calls`。

### Lines 201-204: test case empty content returns empty normal text / 测试用例 empty content returns empty normal text
```python
    def test_empty_content_returns_empty_normal_text(self):
        text = "<tool_calls>\n<tool_call>get_current_date<tool_sep>\n</tool_call>\n</tool_calls>"
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(result.normal_text, "")
```
**EN:** This test exercises `test_empty_content_returns_empty_normal_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_content_returns_empty_normal_text`。

### Lines 206-213: test case unknown tool skipped / 测试用例 unknown tool skipped
```python
    def test_unknown_tool_skipped(self):
        text = (
            "<tool_calls><tool_call>nonexistent_func<tool_sep>"
            "<arg_key>x</arg_key><arg_value>1</arg_value>"
            "</tool_call></tool_calls>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 0)
```
**EN:** This test exercises `test_unknown_tool_skipped` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unknown_tool_skipped`。

### Lines 215-229: test case mixed known and unknown tools / 测试用例 mixed known and unknown tools
```python
    def test_mixed_known_and_unknown_tools(self):
        """Known tools should be parsed, unknown ones skipped."""
        text = (
            "<tool_calls>"
            "<tool_call>get_current_date<tool_sep></tool_call>"
            "<tool_call>nonexistent<tool_sep></tool_call>"
            "<tool_call>search<tool_sep>"
            "<arg_key>query</arg_key><arg_value>test</arg_value>"
            "</tool_call>"
            "</tool_calls>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 2)
        self.assertEqual(result.calls[0].name, "get_current_date")
        self.assertEqual(result.calls[1].name, "search")
```
**EN:** Known tools should be parsed, unknown ones skipped. This test exercises `test_mixed_known_and_unknown_tools` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Known tools should be parsed, unknown ones skipped. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mixed_known_and_unknown_tools`。

### Lines 231-250: test case three parallel tool calls / 测试用例 three parallel tool calls
```python
    def test_three_parallel_tool_calls(self):
        text = (
            "<tool_calls>"
            "<tool_call>get_weather<tool_sep>"
            "<arg_key>city</arg_key><arg_value>Beijing</arg_value>"
            "</tool_call>"
            "<tool_call>get_weather<tool_sep>"
            "<arg_key>city</arg_key><arg_value>Tokyo</arg_value>"
            "</tool_call>"
            "<tool_call>get_current_date<tool_sep></tool_call>"
            "</tool_calls>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 3)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(result.calls[1].name, "get_weather")
        self.assertEqual(result.calls[2].name, "get_current_date")
        # tool_index maps to position in tools list
        self.assertEqual(result.calls[0].tool_index, 1)  # get_weather is index 1
        self.assertEqual(result.calls[2].tool_index, 0)  # get_current_date is index 0
```
**EN:** This test exercises `test_three_parallel_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_three_parallel_tool_calls`。

### Lines 253-253: class TestHunyuanDetectorArgDeserialization declaration / 类 TestHunyuanDetectorArgDeserialization 声明
```python
class TestHunyuanDetectorArgDeserialization(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 254-254: supporting statements / 辅助语句
```python
    """Test type-aware argument deserialization."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 256-258: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.tools = _make_tools()
        self.detector = HunyuanDetector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 260-271: test case integer arg / 测试用例 integer arg
```python
    def test_integer_arg(self):
        text = (
            "<tool_calls><tool_call>search<tool_sep>"
            "<arg_key>query</arg_key><arg_value>restaurants</arg_value>"
            "<arg_key>count</arg_key><arg_value>5</arg_value>"
            "</tool_call></tool_calls>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        args = json.loads(result.calls[0].parameters)
        self.assertEqual(args["query"], "restaurants")
        self.assertEqual(args["count"], 5)
        self.assertIsInstance(args["count"], int)
```
**EN:** This test exercises `test_integer_arg` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_integer_arg`。

### Lines 273-283: test case float arg / 测试用例 float arg
```python
    def test_float_arg(self):
        text = (
            "<tool_calls><tool_call>calculate<tool_sep>"
            "<arg_key>expression</arg_key><arg_value>1+1</arg_value>"
            "<arg_key>precision</arg_key><arg_value>0.01</arg_value>"
            "</tool_call></tool_calls>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        args = json.loads(result.calls[0].parameters)
        self.assertEqual(args["expression"], "1+1")
        self.assertAlmostEqual(args["precision"], 0.01)
```
**EN:** This test exercises `test_float_arg` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_float_arg`。

### Lines 285-294: test case boolean arg / 测试用例 boolean arg
```python
    def test_boolean_arg(self):
        text = (
            "<tool_calls><tool_call>calculate<tool_sep>"
            "<arg_key>expression</arg_key><arg_value>2+2</arg_value>"
            "<arg_key>verbose</arg_key><arg_value>true</arg_value>"
            "</tool_call></tool_calls>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        args = json.loads(result.calls[0].parameters)
        self.assertIs(args["verbose"], True)
```
**EN:** This test exercises `test_boolean_arg` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_boolean_arg`。

### Lines 296-306: test case string arg not deserialized / 测试用例 string arg not deserialized
```python
    def test_string_arg_not_deserialized(self):
        """String-typed args should stay as strings even if they look like JSON."""
        text = (
            "<tool_calls><tool_call>search<tool_sep>"
            '<arg_key>query</arg_key><arg_value>{"key": "value"}</arg_value>'
            "</tool_call></tool_calls>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        args = json.loads(result.calls[0].parameters)
        self.assertEqual(args["query"], '{"key": "value"}')
        self.assertIsInstance(args["query"], str)
```
**EN:** String-typed args should stay as strings even if they look like JSON. This test exercises `test_string_arg_not_deserialized` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** String-typed args should stay as strings even if they look like JSON. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_string_arg_not_deserialized`。

### Lines 308-318: test case non json value stays string / 测试用例 non json value stays string
```python
    def test_non_json_value_stays_string(self):
        """Non-JSON-parseable values for non-string types should fall back to string."""
        text = (
            "<tool_calls><tool_call>search<tool_sep>"
            "<arg_key>query</arg_key><arg_value>hello world</arg_value>"
            "<arg_key>count</arg_key><arg_value>not a number</arg_value>"
            "</tool_call></tool_calls>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        args = json.loads(result.calls[0].parameters)
        self.assertEqual(args["count"], "not a number")
```
**EN:** Non-JSON-parseable values for non-string types should fall back to string. This test exercises `test_non_json_value_stays_string` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Non-JSON-parseable values for non-string types should fall back to string. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_non_json_value_stays_string`。

### Lines 321-333: function collect streamed tool calls / 函数 collect streamed tool calls
```python
def _collect_streamed_tool_calls(all_calls):
    """Accumulate streaming ToolCallItems (name + arg-JSON fragments) by tool_index."""
    tools = {}
    for c in all_calls:
        idx = c.tool_index
        if idx not in tools:
            tools[idx] = {"name": c.name or "", "parameters": c.parameters or ""}
        else:
            if c.name:
                tools[idx]["name"] += c.name
            if c.parameters:
                tools[idx]["parameters"] += c.parameters
    return [tools[i] for i in sorted(tools.keys())]
```
**EN:** Accumulate streaming ToolCallItems (name + arg-JSON fragments) by tool_index. This block implements `_collect_streamed_tool_calls` and captures one focused piece of the module's behavior.
**CN:** Accumulate streaming ToolCallItems (name + arg-JSON fragments) by tool_index. 该代码块实现 `_collect_streamed_tool_calls`，承担模块行为中的一个聚焦逻辑片段。

### Lines 336-336: class TestHunyuanDetectorStreaming declaration / 类 TestHunyuanDetectorStreaming 声明
```python
class TestHunyuanDetectorStreaming(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 337-338: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.tools = _make_tools()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 340-341: method new detector / 方法 new detector
```python
    def _new_detector(self):
        return HunyuanDetector()
```
**EN:** This block implements `_new_detector` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_new_detector`，承担模块行为中的一个聚焦逻辑片段。

### Lines 343-349: test case normal text only / 测试用例 normal text only
```python
    def test_normal_text_only(self):
        detector = self._new_detector()
        result = detector.parse_streaming_increment(
            "Hello, I can help you with that.", self.tools
        )
        self.assertEqual(result.normal_text, "Hello, I can help you with that.")
        self.assertEqual(len(result.calls), 0)
```
**EN:** This test exercises `test_normal_text_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_normal_text_only`。

### Lines 351-362: test case complete tool call single chunk / 测试用例 complete tool call single chunk
```python
    def test_complete_tool_call_single_chunk(self):
        detector = self._new_detector()
        text = (
            "<tool_calls>"
            "<tool_call>get_current_date<tool_sep></tool_call>"
            "</tool_calls>"
        )
        result = detector.parse_streaming_increment(text, self.tools)
        collected = _collect_streamed_tool_calls(result.calls)
        self.assertEqual(len(collected), 1)
        self.assertEqual(collected[0]["name"], "get_current_date")
        self.assertEqual(json.loads(collected[0]["parameters"]), {})
```
**EN:** This test exercises `test_complete_tool_call_single_chunk` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_complete_tool_call_single_chunk`。

### Lines 364-383: test case chunked tool call / 测试用例 chunked tool call
```python
    def test_chunked_tool_call(self):
        detector = self._new_detector()
        chunks = [
            "<tool_calls>",
            "<tool_call>get_weather<tool_sep>",
            "<arg_key>city</arg_key>",
            "<arg_value>Tokyo</arg_value>",
            "</tool_call>",
            "</tool_calls>",
        ]
        all_calls = []
        for chunk in chunks:
            result = detector.parse_streaming_increment(chunk, self.tools)
            all_calls.extend(result.calls)

        collected = _collect_streamed_tool_calls(all_calls)
        self.assertEqual(len(collected), 1)
        self.assertEqual(collected[0]["name"], "get_weather")
        args = json.loads(collected[0]["parameters"])
        self.assertEqual(args["city"], "Tokyo")
```
**EN:** This test exercises `test_chunked_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chunked_tool_call`。

### Lines 385-395: test case normal text before tool / 测试用例 normal text before tool
```python
    def test_normal_text_before_tool(self):
        detector = self._new_detector()
        r1 = detector.parse_streaming_increment("Let me check. ", self.tools)
        self.assertIn("Let me check.", r1.normal_text)

        r2 = detector.parse_streaming_increment(
            "<tool_calls><tool_call>get_current_date<tool_sep></tool_call></tool_calls>",
            self.tools,
        )
        collected = _collect_streamed_tool_calls(r2.calls)
        self.assertEqual([c["name"] for c in collected], ["get_current_date"])
```
**EN:** This test exercises `test_normal_text_before_tool` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_normal_text_before_tool`。

### Lines 397-417: test case multiple tool calls chunked / 测试用例 multiple tool calls chunked
```python
    def test_multiple_tool_calls_chunked(self):
        detector = self._new_detector()
        chunks = [
            "<tool_calls>\n",
            "<tool_call>get_weather<tool_sep>\n",
            "<arg_key>city</arg_key><arg_value>Beijing</arg_value>\n",
            "</tool_call>\n",
            "<tool_call>get_weather<tool_sep>\n",
            "<arg_key>city</arg_key><arg_value>Tokyo</arg_value>\n",
            "</tool_call>\n",
            "</tool_calls>",
        ]
        all_calls = []
        for chunk in chunks:
            result = detector.parse_streaming_increment(chunk, self.tools)
            all_calls.extend(result.calls)

        collected = _collect_streamed_tool_calls(all_calls)
        self.assertEqual(len(collected), 2)
        self.assertEqual(json.loads(collected[0]["parameters"])["city"], "Beijing")
        self.assertEqual(json.loads(collected[1]["parameters"])["city"], "Tokyo")
```
**EN:** This test exercises `test_multiple_tool_calls_chunked` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_tool_calls_chunked`。

### Lines 419-425: test case partial bot token buffered / 测试用例 partial bot token buffered
```python
    def test_partial_bot_token_buffered(self):
        """Partial <tool_calls> at end of chunk should be buffered, not emitted."""
        detector = self._new_detector()
        r1 = detector.parse_streaming_increment("Hello <tool_", self.tools)
        # "Hello " should be emitted, "<tool_" buffered
        self.assertIn("Hello", r1.normal_text)
        self.assertNotIn("<tool_", r1.normal_text)
```
**EN:** Partial <tool_calls> at end of chunk should be buffered, not emitted. This test exercises `test_partial_bot_token_buffered` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Partial <tool_calls> at end of chunk should be buffered, not emitted. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_partial_bot_token_buffered`。

### Lines 427-441: test case char by char streaming / 测试用例 char by char streaming
```python
    def test_char_by_char_streaming(self):
        """Simulate extreme character-by-character streaming."""
        detector = self._new_detector()
        full = (
            "<tool_calls><tool_call>get_current_date<tool_sep></tool_call></tool_calls>"
        )
        all_calls = []
        for ch in full:
            result = detector.parse_streaming_increment(ch, self.tools)
            all_calls.extend(result.calls)

        collected = _collect_streamed_tool_calls(all_calls)
        self.assertEqual(len(collected), 1)
        self.assertEqual(collected[0]["name"], "get_current_date")
        self.assertEqual(json.loads(collected[0]["parameters"]), {})
```
**EN:** Simulate extreme character-by-character streaming. This test exercises `test_char_by_char_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Simulate extreme character-by-character streaming. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_char_by_char_streaming`。

### Lines 443-458: test case streaming with args char by char / 测试用例 streaming with args char by char
```python
    def test_streaming_with_args_char_by_char(self):
        detector = self._new_detector()
        full = (
            "<tool_calls><tool_call>get_weather<tool_sep>"
            "<arg_key>city</arg_key><arg_value>NYC</arg_value>"
            "</tool_call></tool_calls>"
        )
        all_calls = []
        for ch in full:
            result = detector.parse_streaming_increment(ch, self.tools)
            all_calls.extend(result.calls)

        collected = _collect_streamed_tool_calls(all_calls)
        self.assertEqual(len(collected), 1)
        args = json.loads(collected[0]["parameters"])
        self.assertEqual(args["city"], "NYC")
```
**EN:** This test exercises `test_streaming_with_args_char_by_char` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_with_args_char_by_char`。

### Lines 460-481: test case streaming three tools sequential / 测试用例 streaming three tools sequential
```python
    def test_streaming_three_tools_sequential(self):
        """Three different tool calls arriving sequentially."""
        detector = self._new_detector()
        chunks = [
            "<tool_calls>",
            "<tool_call>get_current_date<tool_sep></tool_call>",
            "<tool_call>get_weather<tool_sep><arg_key>city</arg_key><arg_value>SF</arg_value></tool_call>",
            "<tool_call>search<tool_sep><arg_key>query</arg_key><arg_value>test</arg_value></tool_call>",
            "</tool_calls>",
        ]
        all_calls = []
        for chunk in chunks:
            result = detector.parse_streaming_increment(chunk, self.tools)
            all_calls.extend(result.calls)

        collected = _collect_streamed_tool_calls(all_calls)
        self.assertEqual(len(collected), 3)
        self.assertEqual(collected[0]["name"], "get_current_date")
        self.assertEqual(collected[1]["name"], "get_weather")
        self.assertEqual(collected[2]["name"], "search")
        # Streaming uses sequential tool_index (0, 1, 2)
        self.assertEqual(sorted({c.tool_index for c in all_calls}), [0, 1, 2])
```
**EN:** Three different tool calls arriving sequentially. This test exercises `test_streaming_three_tools_sequential` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Three different tool calls arriving sequentially. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_three_tools_sequential`。

### Lines 483-496: test case streaming normal text not lost / 测试用例 streaming normal text not lost
```python
    def test_streaming_normal_text_not_lost(self):
        """All normal text before tool_calls should be fully emitted."""
        detector = self._new_detector()
        all_normal = ""
        for chunk in ["I will ", "check the ", "date now. "]:
            result = detector.parse_streaming_increment(chunk, self.tools)
            all_normal += result.normal_text

        result = detector.parse_streaming_increment(
            "<tool_calls><tool_call>get_current_date<tool_sep></tool_call></tool_calls>",
            self.tools,
        )
        all_normal += result.normal_text
        self.assertIn("I will check the date now.", all_normal)
```
**EN:** All normal text before tool_calls should be fully emitted. This test exercises `test_streaming_normal_text_not_lost` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** All normal text before tool_calls should be fully emitted. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_normal_text_not_lost`。

### Lines 498-514: test case streaming name comes before args / 测试用例 streaming name comes before args
```python
    def test_streaming_name_comes_before_args(self):
        """The name delta must arrive before any arg deltas (two-phase contract)."""
        detector = self._new_detector()
        text = (
            "<tool_calls><tool_call>get_weather<tool_sep>"
            "<arg_key>city</arg_key><arg_value>Paris</arg_value>"
            "</tool_call></tool_calls>"
        )
        all_calls = []
        for ch in text:
            all_calls.extend(detector.parse_streaming_increment(ch, self.tools).calls)

        name_indices = [i for i, c in enumerate(all_calls) if c.name]
        param_indices = [i for i, c in enumerate(all_calls) if c.parameters]
        self.assertTrue(name_indices, "expected at least one name delta")
        self.assertTrue(param_indices, "expected at least one arg delta")
        self.assertLess(min(name_indices), min(param_indices))
```
**EN:** The name delta must arrive before any arg deltas (two-phase contract). This test exercises `test_streaming_name_comes_before_args` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** The name delta must arrive before any arg deltas (two-phase contract). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_name_comes_before_args`。

### Lines 516-535: test case streaming typed args coerced / 测试用例 streaming typed args coerced
```python
    def test_streaming_typed_args_coerced(self):
        """Streaming must apply schema-aware type coercion (int/float/bool)."""
        detector = self._new_detector()
        chunks = [
            "<tool_calls>",
            "<tool_call>search<tool_sep>",
            "<arg_key>query</arg_key><arg_value>pizza</arg_value>",
            "<arg_key>count</arg_key><arg_value>7</arg_value>",
            "</tool_call></tool_calls>",
        ]
        all_calls = []
        for chunk in chunks:
            all_calls.extend(
                detector.parse_streaming_increment(chunk, self.tools).calls
            )
        collected = _collect_streamed_tool_calls(all_calls)
        args = json.loads(collected[0]["parameters"])
        self.assertEqual(args["query"], "pizza")
        self.assertEqual(args["count"], 7)
        self.assertIsInstance(args["count"], int)
```
**EN:** Streaming must apply schema-aware type coercion (int/float/bool). This test exercises `test_streaming_typed_args_coerced` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Streaming must apply schema-aware type coercion (int/float/bool). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_typed_args_coerced`。

### Lines 537-551: test case streaming string arg holds back partial end tag / 测试用例 streaming string arg holds back partial end tag
```python
    def test_streaming_string_arg_holds_back_partial_end_tag(self):
        """Char-by-char string streaming must not leak `</arg_value>` into the value."""
        detector = self._new_detector()
        full = (
            "<tool_calls><tool_call>get_weather<tool_sep>"
            "<arg_key>city</arg_key><arg_value>San Francisco</arg_value>"
            "</tool_call></tool_calls>"
        )
        all_calls = []
        for ch in full:
            all_calls.extend(detector.parse_streaming_increment(ch, self.tools).calls)

        collected = _collect_streamed_tool_calls(all_calls)
        args = json.loads(collected[0]["parameters"])
        self.assertEqual(args["city"], "San Francisco")
```
**EN:** Char-by-char string streaming must not leak `</arg_value>` into the value. This test exercises `test_streaming_string_arg_holds_back_partial_end_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Char-by-char string streaming must not leak `</arg_value>` into the value. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_string_arg_holds_back_partial_end_tag`。

### Lines 553-564: test case streaming all in one delta / 测试用例 streaming all in one delta
```python
    def test_streaming_all_in_one_delta(self):
        """Entire tool call arriving in a single delta."""
        detector = self._new_detector()
        text = (
            "<tool_calls>\n<tool_call>get_current_date<tool_sep>\n"
            "</tool_call>\n</tool_calls>"
        )
        result = detector.parse_streaming_increment(text, self.tools)
        collected = _collect_streamed_tool_calls(result.calls)
        self.assertEqual(len(collected), 1)
        self.assertEqual(collected[0]["name"], "get_current_date")
        self.assertEqual(json.loads(collected[0]["parameters"]), {})
```
**EN:** Entire tool call arriving in a single delta. This test exercises `test_streaming_all_in_one_delta` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Entire tool call arriving in a single delta. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_all_in_one_delta`。

### Lines 566-587: test case streaming content before / 测试用例 streaming content before
```python
    def test_streaming_content_before(self):
        """Normal text preceding a tool call must be surfaced."""
        detector = self._new_detector()
        deltas = [
            "Checking.",
            "<tool_calls>",
            "\n<tool_call>",
            "get_current_date",
            "<tool_sep>",
            "\n</tool_call>",
            "\n</tool_calls>",
        ]
        all_calls = []
        all_normal = ""
        for d in deltas:
            r = detector.parse_streaming_increment(d, self.tools)
            all_calls.extend(r.calls)
            all_normal += r.normal_text
        self.assertIn("Checking.", all_normal)
        collected = _collect_streamed_tool_calls(all_calls)
        self.assertEqual(len(collected), 1)
        self.assertEqual(collected[0]["name"], "get_current_date")
```
**EN:** Normal text preceding a tool call must be surfaced. This test exercises `test_streaming_content_before` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Normal text preceding a tool call must be surfaced. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_content_before`。

### Lines 590-590: class TestHunyuanDetectorStructureInfo declaration / 类 TestHunyuanDetectorStructureInfo 声明
```python
class TestHunyuanDetectorStructureInfo(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 591-592: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.detector = HunyuanDetector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 594-601: test case structure info content / 测试用例 structure info content
```python
    def test_structure_info_content(self):
        info_fn = self.detector.structure_info()
        info = info_fn("get_weather")
        self.assertIn("get_weather", info.begin)
        self.assertIn("<tool_call>", info.begin)
        self.assertIn("<tool_sep>", info.begin)
        self.assertIn("</tool_call>", info.end)
        self.assertEqual(info.trigger, "<tool_calls>")
```
**EN:** This test exercises `test_structure_info_content` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_structure_info_content`。

### Lines 603-604: test case supports structural tag / 测试用例 supports structural tag
```python
    def test_supports_structural_tag(self):
        self.assertFalse(self.detector.supports_structural_tag())
```
**EN:** This test exercises `test_supports_structural_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_supports_structural_tag`。

### Lines 607-607: class TestHunyuanDetectorAccuracy declaration / 类 TestHunyuanDetectorAccuracy 声明
```python
class TestHunyuanDetectorAccuracy(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 608-608: supporting statements / 辅助语句
```python
    """Accuracy tests for realistic HYV3 output patterns."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 610-612: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.tools = _make_tools()
        self.detector = HunyuanDetector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 614-622: test case reference zero arg inline / 测试用例 reference zero arg inline
```python
    def test_reference_zero_arg_inline(self):
        out = (
            "<tool_calls><tool_call>get_current_date<tool_sep></tool_call></tool_calls>"
        )
        r = self.detector.detect_and_parse(out, self.tools)
        self.assertEqual(len(r.calls), 1)
        self.assertEqual(r.calls[0].name, "get_current_date")
        self.assertEqual(json.loads(r.calls[0].parameters), {})
        self.assertEqual(r.normal_text, "")
```
**EN:** This test exercises `test_reference_zero_arg_inline` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reference_zero_arg_inline`。

### Lines 624-628: test case reference zero arg newline / 测试用例 reference zero arg newline
```python
    def test_reference_zero_arg_newline(self):
        out = "<tool_calls>\n<tool_call>get_current_date<tool_sep>\n</tool_call>\n</tool_calls>"
        r = self.detector.detect_and_parse(out, self.tools)
        self.assertEqual(len(r.calls), 1)
        self.assertEqual(r.calls[0].name, "get_current_date")
```
**EN:** This test exercises `test_reference_zero_arg_newline` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reference_zero_arg_newline`。

### Lines 630-638: test case reference args same line / 测试用例 reference args same line
```python
    def test_reference_args_same_line(self):
        out = (
            "<tool_calls><tool_call>get_weather<tool_sep><arg_key>city</arg_key><arg_value>Beijing"
            "</arg_value><arg_key>date</arg_key><arg_value>2026-03-30</arg_value></tool_call></tool_calls>"
        )
        r = self.detector.detect_and_parse(out, self.tools)
        self.assertEqual(len(r.calls), 1)
        args = json.loads(r.calls[0].parameters)
        self.assertEqual(args, {"city": "Beijing", "date": "2026-03-30"})
```
**EN:** This test exercises `test_reference_args_same_line` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reference_args_same_line`。

### Lines 640-648: test case reference args with newlines / 测试用例 reference args with newlines
```python
    def test_reference_args_with_newlines(self):
        out = (
            "<tool_calls>\n<tool_call>get_weather<tool_sep>\n<arg_key>city</arg_key>\n<arg_value>Beijing"
            "</arg_value>\n<arg_key>date</arg_key>\n<arg_value>2026-03-30</arg_value>\n</tool_call>\n</tool_calls>"
        )
        r = self.detector.detect_and_parse(out, self.tools)
        self.assertEqual(len(r.calls), 1)
        args = json.loads(r.calls[0].parameters)
        self.assertEqual(args, {"city": "Beijing", "date": "2026-03-30"})
```
**EN:** This test exercises `test_reference_args_with_newlines` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reference_args_with_newlines`。

### Lines 650-654: test case reference content before / 测试用例 reference content before
```python
    def test_reference_content_before(self):
        out = "Checking.<tool_calls>\n<tool_call>get_current_date<tool_sep>\n</tool_call>\n</tool_calls>"
        r = self.detector.detect_and_parse(out, self.tools)
        self.assertEqual(len(r.calls), 1)
        self.assertEqual(r.normal_text, "Checking.")
```
**EN:** This test exercises `test_reference_content_before` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reference_content_before`。

### Lines 656-664: test case reference multiple / 测试用例 reference multiple
```python
    def test_reference_multiple(self):
        out = (
            "<tool_calls>\n<tool_call>get_weather<tool_sep>\n<arg_key>city</arg_key>\n<arg_value>Beijing"
            "</arg_value>\n<arg_key>date</arg_key>\n<arg_value>2026-03-30</arg_value>\n</tool_call>\n"
            "<tool_call>get_weather<tool_sep>\n<arg_key>city</arg_key>\n<arg_value>Hangzhou</arg_value>\n"
            "<arg_key>date</arg_key>\n<arg_value>2026-03-30</arg_value>\n</tool_call>\n</tool_calls>"
        )
        r = self.detector.detect_and_parse(out, self.tools)
        self.assertEqual(len(r.calls), 2)
```
**EN:** This test exercises `test_reference_multiple` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reference_multiple`。

### Lines 666-669: test case reference empty content none / 测试用例 reference empty content none
```python
    def test_reference_empty_content_none(self):
        out = "<tool_calls>\n<tool_call>get_current_date<tool_sep>\n</tool_call>\n</tool_calls>"
        r = self.detector.detect_and_parse(out, self.tools)
        self.assertEqual(r.normal_text, "")
```
**EN:** This test exercises `test_reference_empty_content_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reference_empty_content_none`。

### Lines 671-675: test case reference no tool call / 测试用例 reference no tool call
```python
    def test_reference_no_tool_call(self):
        out = "This is a plain response."
        r = self.detector.detect_and_parse(out, self.tools)
        self.assertEqual(len(r.calls), 0)
        self.assertEqual(r.normal_text, out)
```
**EN:** This test exercises `test_reference_no_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reference_no_tool_call`。

### Lines 678-678: class TestHunyuanDetectorFunctionCallParser declaration / 类 TestHunyuanDetectorFunctionCallParser 声明
```python
class TestHunyuanDetectorFunctionCallParser(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 679-679: supporting statements / 辅助语句
```python
    """Test through the FunctionCallParser interface."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 681-682: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.tools = _make_tools()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 684-688: test case parser registry / 测试用例 parser registry
```python
    def test_parser_registry(self):
        from sglang.srt.function_call.function_call_parser import FunctionCallParser

        parser = FunctionCallParser(self.tools, "hunyuan")
        self.assertIsInstance(parser.detector, HunyuanDetector)
```
**EN:** This test exercises `test_parser_registry` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parser_registry`。

### Lines 690-703: test case parse non stream / 测试用例 parse non stream
```python
    def test_parse_non_stream(self):
        from sglang.srt.function_call.function_call_parser import FunctionCallParser

        parser = FunctionCallParser(self.tools, "hunyuan")
        text = (
            "Checking.<tool_calls><tool_call>get_weather<tool_sep>"
            "<arg_key>city</arg_key><arg_value>Tokyo</arg_value>"
            "</tool_call></tool_calls>"
        )
        normal, calls = parser.parse_non_stream(text)
        self.assertEqual(normal, "Checking.")
        self.assertEqual(len(calls), 1)
        self.assertEqual(calls[0].name, "get_weather")
        self.assertEqual(json.loads(calls[0].parameters)["city"], "Tokyo")
```
**EN:** This test exercises `test_parse_non_stream` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_non_stream`。

### Lines 705-722: test case parse stream chunks / 测试用例 parse stream chunks
```python
    def test_parse_stream_chunks(self):
        from sglang.srt.function_call.function_call_parser import FunctionCallParser

        parser = FunctionCallParser(self.tools, "hunyuan")
        chunks = [
            "<tool_calls>",
            "<tool_call>get_current_date<tool_sep></tool_call>",
            "</tool_calls>",
        ]
        all_calls = []
        for chunk in chunks:
            normal, calls = parser.parse_stream_chunk(chunk)
            all_calls.extend(calls)

        collected = _collect_streamed_tool_calls(all_calls)
        self.assertEqual(len(collected), 1)
        self.assertEqual(collected[0]["name"], "get_current_date")
        self.assertEqual(json.loads(collected[0]["parameters"]), {})
```
**EN:** This test exercises `test_parse_stream_chunks` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_stream_chunks`。

### Lines 724-729: test case has tool call through parser / 测试用例 has tool call through parser
```python
    def test_has_tool_call_through_parser(self):
        from sglang.srt.function_call.function_call_parser import FunctionCallParser

        parser = FunctionCallParser(self.tools, "hunyuan")
        self.assertTrue(parser.has_tool_call("<tool_calls>foo</tool_calls>"))
        self.assertFalse(parser.has_tool_call("no tools here"))
```
**EN:** This test exercises `test_has_tool_call_through_parser` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_through_parser`。

### Lines 732-733: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_make_tools`: This block implements `_make_tools` and captures one focused piece of the module's behavior. / 该代码块实现 `_make_tools`，承担模块行为中的一个聚焦逻辑片段。
- `TestHunyuanDetectorHasToolCall`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestHunyuanDetectorDetectAndParse`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestHunyuanDetectorArgDeserialization`: Test type-aware argument deserialization. / 用于组织相关测试、夹具或辅助方法。
- `_collect_streamed_tool_calls`: Accumulate streaming ToolCallItems (name + arg-JSON fragments) by tool_index. / 该代码块实现 `_collect_streamed_tool_calls`，承担模块行为中的一个聚焦逻辑片段。
- `TestHunyuanDetectorStreaming`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestHunyuanDetectorStructureInfo`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestHunyuanDetectorAccuracy`: Accuracy tests for realistic HYV3 output patterns. / 用于组织相关测试、夹具或辅助方法。
- `TestHunyuanDetectorHasToolCall.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestHunyuanDetectorHasToolCall.test_has_tool_call_true`: This test exercises `test_has_tool_call_true` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_true`。
- `TestHunyuanDetectorHasToolCall.test_has_tool_call_false`: This test exercises `test_has_tool_call_false` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_false`。
- `TestHunyuanDetectorHasToolCall.test_has_tool_call_partial_tag`: This test exercises `test_has_tool_call_partial_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_partial_tag`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.hunyuan_detector`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 733
