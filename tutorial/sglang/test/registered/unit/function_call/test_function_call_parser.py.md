# test_function_call_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/function_call/test_function_call_parser.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates function call parser behavior in SGLang's unit / function call area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 函数调用 领域中与 function call parser 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: module imports and dependencies / 模块导入与依赖
```python
import json
import unittest

from sglang.srt.entrypoints.openai.protocol import (
    Function,
    Tool,
    ToolChoice,
    ToolChoiceFuncName,
)
from sglang.srt.function_call.base_format_detector import BaseFormatDetector
from sglang.srt.function_call.core_types import StreamingParseResult
from sglang.srt.function_call.deepseekv3_detector import DeepSeekV3Detector
from sglang.srt.function_call.deepseekv4_detector import DeepSeekV4Detector
from sglang.srt.function_call.deepseekv32_detector import DeepSeekV32Detector
from sglang.srt.function_call.gemma4_detector import (
    Gemma4Detector,
    _parse_gemma4_args,
    _parse_gemma4_array,
    _parse_gemma4_value,
)
from sglang.srt.function_call.gigachat3_detector import GigaChat3Detector
from sglang.srt.function_call.glm4_moe_detector import Glm4MoeDetector
from sglang.srt.function_call.glm47_moe_detector import Glm47MoeDetector
from sglang.srt.function_call.gpt_oss_detector import GptOssDetector
from sglang.srt.function_call.json_array_parser import JsonArrayParser
from sglang.srt.function_call.kimik2_detector import KimiK2Detector
from sglang.srt.function_call.lfm2_detector import Lfm2Detector
from sglang.srt.function_call.llama32_detector import Llama32Detector
from sglang.srt.function_call.mistral_detector import MistralDetector
from sglang.srt.function_call.pythonic_detector import PythonicDetector
from sglang.srt.function_call.qwen3_coder_detector import Qwen3CoderDetector
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `unittest`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.base_format_detector`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `unittest`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.base_format_detector`。

### Lines 34-34: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=15, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 37-37: class TestPythonicDetector declaration / 类 TestPythonicDetector 声明
```python
class TestPythonicDetector(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 38-79: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        # Create sample tools for testing
        self.tools = [
            Tool(
                type="function",
                function=Function(
                    name="get_weather",
                    description="Get weather information",
                    parameters={
                        "properties": {
                            "location": {
                                "type": "string",
                                "description": "Location to get weather for",
                            },
                            "unit": {
                                "type": "string",
                                "description": "Temperature unit",
                                "enum": ["celsius", "fahrenheit"],
                            },
                        },
                        "required": ["location"],
                    },
                ),
            ),
            Tool(
                type="function",
                function=Function(
                    name="search",
                    description="Search for information",
                    parameters={
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
        self.detector = PythonicDetector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 81-88: test case parse streaming no brackets / 测试用例 parse streaming no brackets
```python
    def test_parse_streaming_no_brackets(self):
        """Test parsing text with no brackets (no tool calls)."""
        text = "This is just normal text without any tool calls."
        result = self.detector.parse_streaming_increment(text, self.tools)

        self.assertEqual(result.normal_text, text)
        self.assertEqual(result.calls, [])
        self.assertEqual(self.detector._buffer, "")  # Buffer should be cleared
```
**EN:** Test parsing text with no brackets (no tool calls). This test exercises `test_parse_streaming_no_brackets` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing text with no brackets (no tool calls). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_no_brackets`。

### Lines 90-105: test case parse streaming complete tool call / 测试用例 parse streaming complete tool call
```python
    def test_parse_streaming_complete_tool_call(self):
        """Test parsing a complete tool call."""
        text = "Here's a tool call: [get_weather(location='New York', unit='celsius')]"
        result = self.detector.parse_streaming_increment(text, self.tools)

        self.assertEqual(result.normal_text, "Here's a tool call: ")
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(
            self.detector._buffer, ""
        )  # Buffer should be cleared after processing

        # Check the parameters
        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["location"], "New York")
        self.assertEqual(params["unit"], "celsius")
```
**EN:** Test parsing a complete tool call. This test exercises `test_parse_streaming_complete_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing a complete tool call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_complete_tool_call`。

### Lines 107-118: test case parse streaming text before tool call / 测试用例 parse streaming text before tool call
```python
    def test_parse_streaming_text_before_tool_call(self):
        """Test parsing text that appears before a tool call."""
        text = "This is some text before [get_weather(location='London')]"
        result = self.detector.parse_streaming_increment(text, self.tools)

        self.assertEqual(result.normal_text, "This is some text before ")
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")

        # Check the parameters
        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["location"], "London")
```
**EN:** Test parsing text that appears before a tool call. This test exercises `test_parse_streaming_text_before_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing text that appears before a tool call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_text_before_tool_call`。

### Lines 120-145: test case parse streaming partial tool call / 测试用例 parse streaming partial tool call
```python
    def test_parse_streaming_partial_tool_call(self):
        """Test parsing a partial tool call that spans multiple chunks."""
        # First chunk with opening bracket but no closing bracket
        text1 = "Let me check the weather: [get_weather(location="
        result1 = self.detector.parse_streaming_increment(text1, self.tools)

        self.assertEqual(result1.normal_text, "Let me check the weather: ")
        self.assertEqual(result1.calls, [])
        self.assertEqual(
            self.detector._buffer, "[get_weather(location="
        )  # Partial tool call remains in buffer

        # Second chunk completing the tool call
        text2 = "'Paris')]"
        result2 = self.detector.parse_streaming_increment(text2, self.tools)

        self.assertEqual(result2.normal_text, "")
        self.assertEqual(len(result2.calls), 1)
        self.assertEqual(result2.calls[0].name, "get_weather")

        # Check the parameters
        params = json.loads(result2.calls[0].parameters)
        self.assertEqual(params["location"], "Paris")
        self.assertEqual(
            self.detector._buffer, ""
        )  # Buffer should be cleared after processing
```
**EN:** Test parsing a partial tool call that spans multiple chunks. This test exercises `test_parse_streaming_partial_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing a partial tool call that spans multiple chunks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_partial_tool_call`。

### Lines 147-158: test case parse streaming bracket without text before / 测试用例 parse streaming bracket without text before
```python
    def test_parse_streaming_bracket_without_text_before(self):
        """Test parsing a tool call that starts at the beginning of the text."""
        text = "[search(query='python programming')]"
        result = self.detector.parse_streaming_increment(text, self.tools)

        self.assertEqual(result.normal_text, "")
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "search")

        # Check the parameters
        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["query"], "python programming")
```
**EN:** Test parsing a tool call that starts at the beginning of the text. This test exercises `test_parse_streaming_bracket_without_text_before` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing a tool call that starts at the beginning of the text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_bracket_without_text_before`。

### Lines 160-177: test case parse streaming text after tool call / 测试用例 parse streaming text after tool call
```python
    def test_parse_streaming_text_after_tool_call(self):
        """Test parsing text that appears after a tool call."""
        # First chunk with complete tool call and some text after
        text = "[get_weather(location='Tokyo')] Here's the forecast:"
        result = self.detector.parse_streaming_increment(text, self.tools)

        self.assertEqual(result.normal_text, "")
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(
            self.detector._buffer, " Here's the forecast:"
        )  # Text after tool call remains in buffer

        # Process the remaining text in buffer
        result2 = self.detector.parse_streaming_increment("", self.tools)
        self.assertEqual(result2.normal_text, " Here's the forecast:")
        self.assertEqual(result2.calls, [])
        self.assertEqual(self.detector._buffer, "")  # Buffer should be cleared
```
**EN:** Test parsing text that appears after a tool call. This test exercises `test_parse_streaming_text_after_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing text that appears after a tool call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_text_after_tool_call`。

### Lines 179-194: test case parse streaming multiple tool calls / 测试用例 parse streaming multiple tool calls
```python
    def test_parse_streaming_multiple_tool_calls(self):
        """Test parsing multiple tool calls in sequence."""
        text = "[get_weather(location='Berlin')] and [search(query='restaurants')]"

        # First tool call
        result1 = self.detector.parse_streaming_increment(text, self.tools)
        self.assertEqual(len(result1.calls), 1)
        self.assertEqual(result1.calls[0].name, "get_weather")
        self.assertEqual(self.detector._buffer, " and [search(query='restaurants')]")

        # Second tool call
        result2 = self.detector.parse_streaming_increment("", self.tools)
        self.assertEqual(result2.normal_text, " and ")
        self.assertEqual(len(result2.calls), 1)
        self.assertEqual(result2.calls[0].name, "search")
        self.assertEqual(self.detector._buffer, "")
```
**EN:** Test parsing multiple tool calls in sequence. This test exercises `test_parse_streaming_multiple_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing multiple tool calls in sequence. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_multiple_tool_calls`。

### Lines 196-205: test case parse streaming opening bracket only / 测试用例 parse streaming opening bracket only
```python
    def test_parse_streaming_opening_bracket_only(self):
        """Test parsing text with only an opening bracket but no closing bracket."""
        text = "Let's try this: ["
        result = self.detector.parse_streaming_increment(text, self.tools)

        self.assertEqual(result.normal_text, "Let's try this: ")
        self.assertEqual(result.calls, [])
        self.assertEqual(
            self.detector._buffer, "["
        )  # Opening bracket remains in buffer
```
**EN:** Test parsing text with only an opening bracket but no closing bracket. This test exercises `test_parse_streaming_opening_bracket_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing text with only an opening bracket but no closing bracket. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_opening_bracket_only`。

### Lines 207-222: test case parse streaming nested brackets / 测试用例 parse streaming nested brackets
```python
    def test_parse_streaming_nested_brackets(self):
        """Test parsing tool calls with nested brackets in arguments."""
        # Test with list argument containing nested brackets
        text = "[get_weather(location='New York', unit='celsius', data=[1, 2, 3])]"
        result = self.detector.parse_streaming_increment(text, self.tools)

        self.assertEqual(result.normal_text, "")
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(self.detector._buffer, "")

        # Check the parameters
        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["location"], "New York")
        self.assertEqual(params["unit"], "celsius")
        self.assertEqual(params["data"], [1, 2, 3])
```
**EN:** Test parsing tool calls with nested brackets in arguments. This test exercises `test_parse_streaming_nested_brackets` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing tool calls with nested brackets in arguments. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_nested_brackets`。

### Lines 224-239: test case parse streaming nested brackets dict / 测试用例 parse streaming nested brackets dict
```python
    def test_parse_streaming_nested_brackets_dict(self):
        """Test parsing tool calls with nested dictionaries and lists."""
        # Test with nested dict and list arguments
        text = "[search(query='test', config={'options': [1, 2], 'nested': {'key': 'value'}})]"
        result = self.detector.parse_streaming_increment(text, self.tools)

        self.assertEqual(result.normal_text, "")
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "search")
        self.assertEqual(self.detector._buffer, "")

        # Check the parameters
        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["query"], "test")
        self.assertEqual(params["config"]["options"], [1, 2])
        self.assertEqual(params["config"]["nested"]["key"], "value")
```
**EN:** Test parsing tool calls with nested dictionaries and lists. This test exercises `test_parse_streaming_nested_brackets_dict` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing tool calls with nested dictionaries and lists. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_nested_brackets_dict`。

### Lines 241-260: test case parse streaming multiple tools with nested brackets / 测试用例 parse streaming multiple tools with nested brackets
```python
    def test_parse_streaming_multiple_tools_with_nested_brackets(self):
        """Test parsing multiple tool calls with nested brackets."""
        text = "[get_weather(location='Paris', data=[10, 20]), search(query='test', filters=['a', 'b'])]"
        result = self.detector.parse_streaming_increment(text, self.tools)

        self.assertEqual(result.normal_text, "")
        self.assertEqual(len(result.calls), 2)
        self.assertEqual(self.detector._buffer, "")

        # Check first tool call
        params1 = json.loads(result.calls[0].parameters)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(params1["location"], "Paris")
        self.assertEqual(params1["data"], [10, 20])

        # Check second tool call
        params2 = json.loads(result.calls[1].parameters)
        self.assertEqual(result.calls[1].name, "search")
        self.assertEqual(params2["query"], "test")
        self.assertEqual(params2["filters"], ["a", "b"])
```
**EN:** Test parsing multiple tool calls with nested brackets. This test exercises `test_parse_streaming_multiple_tools_with_nested_brackets` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing multiple tool calls with nested brackets. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_multiple_tools_with_nested_brackets`。

### Lines 262-286: test case parse streaming partial nested brackets / 测试用例 parse streaming partial nested brackets
```python
    def test_parse_streaming_partial_nested_brackets(self):
        """Test parsing partial tool calls with nested brackets across chunks."""
        # First chunk with nested brackets but incomplete
        text1 = "Here's a call: [get_weather(location='Tokyo', data=[1, 2"
        result1 = self.detector.parse_streaming_increment(text1, self.tools)

        self.assertEqual(result1.normal_text, "Here's a call: ")
        self.assertEqual(result1.calls, [])
        self.assertEqual(
            self.detector._buffer, "[get_weather(location='Tokyo', data=[1, 2"
        )

        # Second chunk completing the nested brackets
        text2 = ", 3])]"
        result2 = self.detector.parse_streaming_increment(text2, self.tools)

        self.assertEqual(result2.normal_text, "")
        self.assertEqual(len(result2.calls), 1)
        self.assertEqual(result2.calls[0].name, "get_weather")
        self.assertEqual(self.detector._buffer, "")

        # Check the parameters
        params = json.loads(result2.calls[0].parameters)
        self.assertEqual(params["location"], "Tokyo")
        self.assertEqual(params["data"], [1, 2, 3])
```
**EN:** Test parsing partial tool calls with nested brackets across chunks. This test exercises `test_parse_streaming_partial_nested_brackets` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing partial tool calls with nested brackets across chunks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_partial_nested_brackets`。

### Lines 288-343: test case parse streaming with python start and end token / 测试用例 parse streaming with python start and end token
```python
    def test_parse_streaming_with_python_start_and_end_token(self):
        """Test parsing a message that starts with <|python_start|> and <|python_end|> across chunks."""
        chunks = [
            "Here's a call: ",
            "<|python_",
            "start|>[get_weather(location=",
            "'Tokyo', data=[1, 2",
            ", 3])]<|python_end|>",
        ]

        normal_text = ""
        call_name = ""
        parameters = ""
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            if result.normal_text:
                normal_text += result.normal_text
            if result.calls:
                call_name += result.calls[0].name
                parameters += result.calls[0].parameters

        self.assertEqual(normal_text, "Here's a call: ")
        self.assertEqual(call_name, "get_weather")
        self.assertEqual(self.detector._buffer, "")
        self.assertEqual(
            result.normal_text, "", "Final result should have no normal text"
        )

        # Check the parameters
        params = json.loads(parameters)
        self.assertEqual(params["location"], "Tokyo")
        self.assertEqual(params["data"], [1, 2, 3])

        chunks = [
            "Here's a call: <|python_start|>[get_weather(location='Tokyo', data=[1, 2, 3])]<|python_end|>"
        ]

        normal_text = ""
        call_name = ""
        parameters = ""
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            if result.normal_text:
                normal_text += result.normal_text
            if result.calls:
                call_name += result.calls[0].name
                parameters += result.calls[0].parameters

        self.assertEqual(normal_text, "Here's a call: ")
        self.assertEqual(call_name, "get_weather")
        self.assertEqual(self.detector._buffer, "")

        # Check the parameters
        params = json.loads(parameters)
        self.assertEqual(params["location"], "Tokyo")
        self.assertEqual(params["data"], [1, 2, 3])
```
**EN:** Test parsing a message that starts with <|python_start|> and <|python_end|> across chunks. This test exercises `test_parse_streaming_with_python_start_and_end_token` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing a message that starts with <|python_start|> and <|python_end|> across chunks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_with_python_start_and_end_token`。

### Lines 345-361: test case detect and parse with python start and end token / 测试用例 detect and parse with python start and end token
```python
    def test_detect_and_parse_with_python_start_and_end_token(self):
        """Test parsing a message that starts with <|python_start|> and contains a valid tool call."""
        text = "User wants to get the weather in Mars. <|python_start|>[get_weather(location='Mars', unit='celsius')]<|python_end|> In this way we will get the weather in Mars."
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(
            result.normal_text,
            "User wants to get the weather in Mars.  In this way we will get the weather in Mars.",
        )
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(self.detector._buffer, "")

        # Check the parameters
        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["location"], "Mars")
        self.assertEqual(params["unit"], "celsius")
```
**EN:** Test parsing a message that starts with <|python_start|> and contains a valid tool call. This test exercises `test_detect_and_parse_with_python_start_and_end_token` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing a message that starts with <|python_start|> and contains a valid tool call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_with_python_start_and_end_token`。

### Lines 364-364: class TestMistralDetector declaration / 类 TestMistralDetector 声明
```python
class TestMistralDetector(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 365-390: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Set up test tools and detector for Mistral format testing."""
        self.tools = [
            Tool(
                type="function",
                function=Function(
                    name="make_next_step_decision",
                    description="Test function for decision making",
                    parameters={
                        "type": "object",
                        "properties": {
                            "decision": {
                                "type": "string",
                                "description": "The next step to take",
                            },
                            "content": {
                                "type": "string",
                                "description": "The content of the next step",
                            },
                        },
                        "required": ["decision", "content"],
                    },
                ),
            ),
        ]
        self.detector = MistralDetector()
```
**EN:** Set up test tools and detector for Mistral format testing. This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Set up test tools and detector for Mistral format testing. 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 392-430: test case detect and parse with nested brackets in content / 测试用例 detect and parse with nested brackets in content
```python
    def test_detect_and_parse_with_nested_brackets_in_content(self):
        """Test parsing Mistral format with nested brackets in JSON content.

        This test case specifically addresses the issue where the regex pattern
        was incorrectly truncating JSON when it contained nested brackets like [City Name].
        """
        # This is the exact problematic text from the original test failure
        test_text = '[TOOL_CALLS] [{"name":"make_next_step_decision", "arguments":{"decision":"","content":"```\\nTOOL: Access a weather API or service\\nOBSERVATION: Retrieve the current weather data for the top 5 populated cities in the US\\nANSWER: The weather in the top 5 populated cities in the US is as follows: [City Name] - [Weather Conditions] - [Temperature]\\n```"}}]'

        result = self.detector.detect_and_parse(test_text, self.tools)

        # Verify that the parsing was successful
        self.assertEqual(len(result.calls), 1, "Should detect exactly one tool call")

        call = result.calls[0]
        self.assertEqual(
            call.name,
            "make_next_step_decision",
            "Should detect the correct function name",
        )

        # Verify that the parameters are valid JSON and contain the expected content
        params = json.loads(call.parameters)
        self.assertEqual(
            params["decision"], "", "Decision parameter should be empty string"
        )

        # The content should contain the full text including the nested brackets [City Name]
        expected_content = "```\nTOOL: Access a weather API or service\nOBSERVATION: Retrieve the current weather data for the top 5 populated cities in the US\nANSWER: The weather in the top 5 populated cities in the US is as follows: [City Name] - [Weather Conditions] - [Temperature]\n```"
        self.assertEqual(
            params["content"],
            expected_content,
            "Content should include nested brackets without truncation",
        )

        # Verify that normal text is empty (since the entire input is a tool call)
        self.assertEqual(
            result.normal_text, "", "Normal text should be empty for pure tool call"
        )
```
**EN:** Test parsing Mistral format with nested brackets in JSON content. This test exercises `test_detect_and_parse_with_nested_brackets_in_content` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing Mistral format with nested brackets in JSON content. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_with_nested_brackets_in_content`。

### Lines 432-444: test case detect and parse simple case / 测试用例 detect and parse simple case
```python
    def test_detect_and_parse_simple_case(self):
        """Test parsing a simple Mistral format tool call without nested brackets."""
        test_text = '[TOOL_CALLS] [{"name":"make_next_step_decision", "arguments":{"decision":"TOOL", "content":"Use weather API"}}]'

        result = self.detector.detect_and_parse(test_text, self.tools)

        self.assertEqual(len(result.calls), 1)
        call = result.calls[0]
        self.assertEqual(call.name, "make_next_step_decision")

        params = json.loads(call.parameters)
        self.assertEqual(params["decision"], "TOOL")
        self.assertEqual(params["content"], "Use weather API")
```
**EN:** Test parsing a simple Mistral format tool call without nested brackets. This test exercises `test_detect_and_parse_simple_case` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing a simple Mistral format tool call without nested brackets. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_simple_case`。

### Lines 446-457: test case detect and parse no tool calls / 测试用例 detect and parse no tool calls
```python
    def test_detect_and_parse_no_tool_calls(self):
        """Test parsing text without any tool calls."""
        test_text = "This is just normal text without any tool calls."

        result = self.detector.detect_and_parse(test_text, self.tools)

        self.assertEqual(len(result.calls), 0, "Should detect no tool calls")
        self.assertEqual(
            result.normal_text,
            test_text,
            "Should return the original text as normal text",
        )
```
**EN:** Test parsing text without any tool calls. This test exercises `test_detect_and_parse_no_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing text without any tool calls. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_no_tool_calls`。

### Lines 459-473: test case detect and parse with text before tool call / 测试用例 detect and parse with text before tool call
```python
    def test_detect_and_parse_with_text_before_tool_call(self):
        """Test parsing text that has content before the tool call."""
        test_text = 'Here is some text before the tool call: [TOOL_CALLS] [{"name":"make_next_step_decision", "arguments":{"decision":"ANSWER", "content":"The answer is 42"}}]'

        result = self.detector.detect_and_parse(test_text, self.tools)

        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.normal_text, "Here is some text before the tool call:")

        call = result.calls[0]
        self.assertEqual(call.name, "make_next_step_decision")

        params = json.loads(call.parameters)
        self.assertEqual(params["decision"], "ANSWER")
        self.assertEqual(params["content"], "The answer is 42")
```
**EN:** Test parsing text that has content before the tool call. This test exercises `test_detect_and_parse_with_text_before_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing text that has content before the tool call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_with_text_before_tool_call`。

### Lines 475-484: test case detect and parse compact args format / 测试用例 detect and parse compact args format
```python
    def test_detect_and_parse_compact_args_format(self):
        """Test parsing compact format: [TOOL_CALLS]name[ARGS]{...}."""
        test_text = '[TOOL_CALLS]make_next_step_decision[ARGS]{"decision":"TOOL", "content":"Use weather API"}'

        result = self.detector.detect_and_parse(test_text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "make_next_step_decision")
        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["decision"], "TOOL")
        self.assertEqual(params["content"], "Use weather API")
```
**EN:** Test parsing compact format: [TOOL_CALLS]name[ARGS]{...}. This test exercises `test_detect_and_parse_compact_args_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing compact format: [TOOL_CALLS]name[ARGS]{...}. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_compact_args_format`。

### Lines 486-507: test case streaming compact args format emits tool calls / 测试用例 streaming compact args format emits tool calls
```python
    def test_streaming_compact_args_format_emits_tool_calls(self):
        """Test streaming chunks for compact format produce tool_calls items."""
        chunks = [
            "[TOOL_CALLS]make_next_step_decision[ARGS]",
            '{"decision":"TOOL", ',
            '"content":"Use weather API"}',
        ]

        emitted = []
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            if result.calls:
                emitted.extend(result.calls)

        # Expect two items: name chunk + full args chunk
        self.assertEqual(len(emitted), 2)
        self.assertEqual(emitted[0].name, "make_next_step_decision")
        self.assertEqual(emitted[0].parameters, "")
        self.assertIsNone(emitted[1].name)
        params = json.loads(emitted[1].parameters)
        self.assertEqual(params["decision"], "TOOL")
        self.assertEqual(params["content"], "Use weather API")
```
**EN:** Test streaming chunks for compact format produce tool_calls items. This test exercises `test_streaming_compact_args_format_emits_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming chunks for compact format produce tool_calls items. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_compact_args_format_emits_tool_calls`。

### Lines 510-510: class TestBaseFormatDetector declaration / 类 TestBaseFormatDetector 声明
```python
class TestBaseFormatDetector(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 511-511: supporting statements / 辅助语句
```python
    """Test buffer management and sequential tool index assignment in BaseFormatDetector."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 513-570: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Set up test detector and tools."""

        # Create a concrete implementation of BaseFormatDetector for testing
        class TestFormatDetector(BaseFormatDetector):
            def __init__(self):
                super().__init__()
                self.bot_token = "<tool_call>"
                self.eot_token = "</tool_call>"

            def detect_and_parse(self, text, tools):
                # Not used in streaming tests
                pass

            def has_tool_call(self, text):
                return "<tool_call>" in text

            def structure_info(self):
                # Not used in streaming tests
                pass

        self.detector = TestFormatDetector()
        self.tools = [
            Tool(
                type="function",
                function=Function(
                    name="get_weather",
                    description="Get weather information",
                    parameters={
                        "type": "object",
                        "properties": {
                            "city": {
                                "type": "string",
                                "description": "City name",
                            }
                        },
                        "required": ["city"],
                    },
                ),
            ),
            Tool(
                type="function",
                function=Function(
                    name="get_tourist_attractions",
                    description="Get tourist attractions",
                    parameters={
                        "type": "object",
                        "properties": {
                            "city": {
                                "type": "string",
                                "description": "City name",
                            }
                        },
                        "required": ["city"],
                    },
                ),
            ),
        ]
```
**EN:** Set up test detector and tools. This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Set up test detector and tools. 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 572-601: test case sequential tool index assignment / 测试用例 sequential tool index assignment
```python
    def test_sequential_tool_index_assignment(self):
        """Test that multiple tool calls get sequential tool_index values (0, 1, 2, ...)."""
        # Simulate streaming chunks for two consecutive tool calls
        chunks = [
            "<tool_call>",
            '{"name": "get_weather", ',
            '"arguments": {"city": "Paris"}}',
            ", ",
            '{"name": "get_tourist_attractions", ',
            '"arguments": {"city": "London"}}',
            "</tool_call>",
        ]

        tool_indices_seen = []

        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)

            if result.calls:
                for call in result.calls:
                    if call.tool_index is not None:
                        tool_indices_seen.append(call.tool_index)

        # Verify we got sequential tool indices
        unique_indices = sorted(set(tool_indices_seen))
        self.assertEqual(
            unique_indices,
            [0, 1],
            f"Expected sequential tool indices [0, 1], got {unique_indices}",
        )
```
**EN:** Test that multiple tool calls get sequential tool_index values (0, 1, 2, ...). This test exercises `test_sequential_tool_index_assignment` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that multiple tool calls get sequential tool_index values (0, 1, 2, ...). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_sequential_tool_index_assignment`。

### Lines 603-630: test case buffer content preservation / 测试用例 buffer content preservation
```python
    def test_buffer_content_preservation(self):
        """Test that buffer correctly preserves unprocessed content when tool completes."""
        # Test simpler scenario: tool completion followed by new tool start
        chunks = [
            "<tool_call>",
            '{"name": "get_weather", ',
            '"arguments": {"city": "Paris"}}',
            ", ",
            '{"name": "get_tourist_attractions", ',
            '"arguments": {"city": "London"}} </tool_call>',
        ]

        tool_calls_seen = []

        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            if result.calls:
                for call in result.calls:
                    if (
                        call.name
                    ):  # Only count calls with names (not just parameter updates)
                        tool_calls_seen.append(call.name)

        # Should see both tool names
        self.assertIn("get_weather", tool_calls_seen, "Should process first tool")
        self.assertIn(
            "get_tourist_attractions", tool_calls_seen, "Should process second tool"
        )
```
**EN:** Test that buffer correctly preserves unprocessed content when tool completes. This test exercises `test_buffer_content_preservation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that buffer correctly preserves unprocessed content when tool completes. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_buffer_content_preservation`。

### Lines 632-680: test case current tool id increment on completion / 测试用例 current tool id increment on completion
```python
    def test_current_tool_id_increment_on_completion(self):
        """Test that current_tool_id increments when a tool completes."""
        # Initial state
        self.assertEqual(
            self.detector.current_tool_id, -1, "Should start with current_tool_id=-1"
        )

        # Process first tool completely
        chunks = [
            "<tool_call>",
            '{"name": "get_weather", ',
        ]

        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)

        self.assertEqual(
            self.detector.current_tool_id, 0, "current_tool_id should be 0"
        )
        self.assertEqual(
            result.calls[0].name, "get_weather", "The first tool should be get_weather"
        )
        self.assertEqual(
            result.calls[0].tool_index, 0, "The first tool index should be 0"
        )

        # Complete second tool name - this should show that current_tool_id is now 1
        result = self.detector.parse_streaming_increment(
            '"arguments": {"city": "Paris"}}, {"name": "get_', self.tools
        )
        self.assertEqual(result.calls[0].parameters, '{"city": "Paris"}')

        self.assertEqual(
            self.detector.current_tool_id,
            1,
            "current_tool_id should be 1 after first tool completes and second tool starts",
        )

        result = self.detector.parse_streaming_increment(
            'tourist_attractions", ', self.tools
        )

        # Second tool should have tool_index=1
        tourist_calls = [
            call for call in result.calls if call.name == "get_tourist_attractions"
        ]
        self.assertEqual(
            tourist_calls[0].tool_index, 1, "Second tool should have tool_index=1"
        )
```
**EN:** Test that current_tool_id increments when a tool completes. This test exercises `test_current_tool_id_increment_on_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that current_tool_id increments when a tool completes. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_current_tool_id_increment_on_completion`。

### Lines 682-717: test case tool name streaming with correct index / 测试用例 tool name streaming with correct index
```python
    def test_tool_name_streaming_with_correct_index(self):
        """Test that tool names are streamed with correct tool_index values."""
        # Process first tool
        self.detector.parse_streaming_increment("<tool_call>", self.tools)
        result1 = self.detector.parse_streaming_increment(
            '{"name": "get_weather", ', self.tools
        )

        # First tool name should have tool_index=0
        weather_calls = [call for call in result1.calls if call.name == "get_weather"]
        self.assertEqual(len(weather_calls), 1, "Should have one weather call")
        self.assertEqual(
            weather_calls[0].tool_index, 0, "First tool should have tool_index=0"
        )

        # Complete first tool
        self.detector.parse_streaming_increment(
            '"arguments": {"city": "Paris"}}', self.tools
        )

        # Start second tool
        self.detector.parse_streaming_increment(", ", self.tools)
        result2 = self.detector.parse_streaming_increment(
            '{"name": "get_tourist_attractions", ', self.tools
        )

        # Second tool name should have tool_index=1
        tourist_calls = [
            call for call in result2.calls if call.name == "get_tourist_attractions"
        ]
        self.assertEqual(
            len(tourist_calls), 1, "Should have one tourist attractions call"
        )
        self.assertEqual(
            tourist_calls[0].tool_index, 1, "Second tool should have tool_index=1"
        )
```
**EN:** Test that tool names are streamed with correct tool_index values. This test exercises `test_tool_name_streaming_with_correct_index` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that tool names are streamed with correct tool_index values. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_name_streaming_with_correct_index`。

### Lines 719-735: test case buffer reset on invalid tool / 测试用例 buffer reset on invalid tool
```python
    def test_buffer_reset_on_invalid_tool(self):
        """Test that buffer and state are reset when an invalid tool name is encountered."""
        # Start fresh with an invalid tool name from the beginning
        result = self.detector.parse_streaming_increment(
            '<tool_call>{"name": "invalid_tool", ', self.tools
        )

        # Should return empty result and reset state
        self.assertEqual(result.calls, [], "Should return no calls for invalid tool")
        self.assertEqual(
            self.detector.current_tool_id,
            -1,
            "current_tool_id should remain -1 for invalid tool",
        )
        self.assertEqual(
            self.detector._buffer, "", "Buffer should be cleared for invalid tool"
        )
```
**EN:** Test that buffer and state are reset when an invalid tool name is encountered. This test exercises `test_buffer_reset_on_invalid_tool` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that buffer and state are reset when an invalid tool name is encountered. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_buffer_reset_on_invalid_tool`。

### Lines 737-779: test case chinese characters not double escaped / 测试用例 chinese characters not double escaped
```python
    def test_chinese_characters_not_double_escaped(self):
        """Test that Chinese characters in tool call parameters are not double-escaped."""
        # Test with Chinese city name "杭州" (Hangzhou)
        chunks = [
            "<tool_call>",
            '{"name": "get_weather", ',
            '"arguments": {"city": "杭州"}}',
            "</tool_call>",
        ]

        accumulated_parameters = {}
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            if result.calls:
                for call in result.calls:
                    if call.parameters:
                        tool_idx = call.tool_index if call.tool_index is not None else 0
                        if tool_idx not in accumulated_parameters:
                            accumulated_parameters[tool_idx] = ""
                        accumulated_parameters[tool_idx] += call.parameters

        # Verify that Chinese characters are preserved (not escaped as \uXXXX)
        self.assertGreater(
            len(accumulated_parameters), 0, "Should have parsed parameters"
        )
        final_params_str = accumulated_parameters[0]

        # The parameters string should contain the actual Chinese characters, not escaped Unicode
        self.assertIn(
            "杭州", final_params_str, "Should contain actual Chinese characters"
        )
        self.assertNotIn(
            "\\u676d", final_params_str, "Should not contain escaped Unicode sequences"
        )
        self.assertNotIn(
            "\\u5dde", final_params_str, "Should not contain escaped Unicode sequences"
        )

        # Verify the JSON can be parsed and contains the correct value
        params = json.loads(final_params_str)
        self.assertEqual(
            params["city"], "杭州", "Should correctly parse Chinese city name"
        )
```
**EN:** Test that Chinese characters in tool call parameters are not double-escaped. This test exercises `test_chinese_characters_not_double_escaped` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that Chinese characters in tool call parameters are not double-escaped. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chinese_characters_not_double_escaped`。

### Lines 781-819: test case chinese characters incremental streaming / 测试用例 chinese characters incremental streaming
```python
    def test_chinese_characters_incremental_streaming(self):
        """Test that Chinese characters work correctly with incremental streaming."""
        # Test incremental streaming with Chinese characters
        chunks = [
            "<tool_call>",
            '{"name": "get_weather", ',
            '"arguments": {"city": "',
            "杭州",
            '"}}',
            "</tool_call>",
        ]

        accumulated_parameters = {}
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            if result.calls:
                for call in result.calls:
                    if call.parameters:
                        tool_idx = call.tool_index if call.tool_index is not None else 0
                        if tool_idx not in accumulated_parameters:
                            accumulated_parameters[tool_idx] = ""
                        accumulated_parameters[tool_idx] += call.parameters

        # Verify Chinese characters are preserved throughout streaming
        self.assertGreater(
            len(accumulated_parameters), 0, "Should have parsed parameters"
        )
        final_params_str = accumulated_parameters[0]

        # Should contain actual Chinese characters, not escaped
        self.assertIn(
            "杭州", final_params_str, "Should contain actual Chinese characters"
        )

        # Parse and verify
        params = json.loads(final_params_str)
        self.assertEqual(
            params["city"], "杭州", "Should correctly parse Chinese city name"
        )
```
**EN:** Test that Chinese characters work correctly with incremental streaming. This test exercises `test_chinese_characters_incremental_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that Chinese characters work correctly with incremental streaming. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chinese_characters_incremental_streaming`。

### Lines 821-869: test case multiple chinese parameters / 测试用例 multiple chinese parameters
```python
    def test_multiple_chinese_parameters(self):
        """Test multiple tool calls with Chinese parameters."""
        # Test with multiple tool calls containing Chinese characters
        chunks = [
            "<tool_call>",
            '{"name": "get_weather", "arguments": {"city": "北京"}}, ',
            '{"name": "get_tourist_attractions", "arguments": {"city": "上海"}}',
            "</tool_call>",
        ]

        accumulated_parameters = {}
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            if result.calls:
                for call in result.calls:
                    if call.parameters:
                        tool_idx = call.tool_index if call.tool_index is not None else 0
                        if tool_idx not in accumulated_parameters:
                            accumulated_parameters[tool_idx] = ""
                        accumulated_parameters[tool_idx] += call.parameters

        # Verify both tool calls have correct Chinese characters
        self.assertGreaterEqual(
            len(accumulated_parameters), 1, "Should have parsed parameters"
        )

        # Check first tool call (北京 - Beijing)
        if 0 in accumulated_parameters:
            params0 = json.loads(accumulated_parameters[0])
            self.assertIn(
                "北京",
                accumulated_parameters[0],
                "Should contain actual Chinese characters",
            )
            self.assertEqual(
                params0["city"], "北京", "Should correctly parse first Chinese city"
            )

        # Check second tool call (上海 - Shanghai) if present
        if 1 in accumulated_parameters:
            params1 = json.loads(accumulated_parameters[1])
            self.assertIn(
                "上海",
                accumulated_parameters[1],
                "Should contain actual Chinese characters",
            )
            self.assertEqual(
                params1["city"], "上海", "Should correctly parse second Chinese city"
            )
```
**EN:** Test multiple tool calls with Chinese parameters. This test exercises `test_multiple_chinese_parameters` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test multiple tool calls with Chinese parameters. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_chinese_parameters`。

### Lines 872-872: class TestLlama32Detector declaration / 类 TestLlama32Detector 声明
```python
class TestLlama32Detector(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 873-911: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Set up test tools and detector for Mistral format testing."""
        self.tools = [
            Tool(
                type="function",
                function=Function(
                    name="get_weather",
                    description="Get weather information",
                    parameters={
                        "type": "object",
                        "properties": {
                            "city": {
                                "type": "string",
                                "description": "City name",
                            }
                        },
                        "required": ["city"],
                    },
                ),
            ),
            Tool(
                type="function",
                function=Function(
                    name="get_tourist_attractions",
                    description="Get tourist attractions",
                    parameters={
                        "type": "object",
                        "properties": {
                            "city": {
                                "type": "string",
                                "description": "City name",
                            }
                        },
                        "required": ["city"],
                    },
                ),
            ),
        ]
        self.detector = Llama32Detector()
```
**EN:** Set up test tools and detector for Mistral format testing. This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Set up test tools and detector for Mistral format testing. 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 913-918: test case single json / 测试用例 single json
```python
    def test_single_json(self):
        text = '{"name": "get_weather", "parameters": {"city": "Paris"}}'
        result = self.detector.detect_and_parse(text, self.tools)
        assert len(result.calls) == 1
        assert result.calls[0].name == "get_weather"
        assert result.normal_text == ""
```
**EN:** This test exercises `test_single_json` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_json`。

### Lines 920-928: test case multiple json with separator / 测试用例 multiple json with separator
```python
    def test_multiple_json_with_separator(self):
        text = (
            '<|python_tag|>{"name": "get_weather", "parameters": {"city": "Paris"}};'
            '{"name": "get_tourist_attractions", "parameters": {"city": "Paris"}}'
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 2)
        self.assertEqual(result.calls[1].name, "get_tourist_attractions")
        self.assertEqual(result.normal_text, "")
```
**EN:** This test exercises `test_multiple_json_with_separator` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_json_with_separator`。

### Lines 930-938: test case multiple json with separator customized / 测试用例 multiple json with separator customized
```python
    def test_multiple_json_with_separator_customized(self):
        text = (
            '<|python_tag|>{"name": "get_weather", "parameters": {}}'
            '<|python_tag|>{"name": "get_tourist_attractions", "parameters": {}}'
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 2)
        self.assertEqual(result.calls[1].name, "get_tourist_attractions")
        self.assertEqual(result.normal_text, "")
```
**EN:** This test exercises `test_multiple_json_with_separator_customized` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_json_with_separator_customized`。

### Lines 940-944: test case json with trailing text / 测试用例 json with trailing text
```python
    def test_json_with_trailing_text(self):
        text = '{"name": "get_weather", "parameters": {}} Some follow-up text'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertIn("follow-up", result.normal_text)
```
**EN:** This test exercises `test_json_with_trailing_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_json_with_trailing_text`。

### Lines 946-953: test case invalid then valid json / 测试用例 invalid then valid json
```python
    def test_invalid_then_valid_json(self):
        text = (
            '{"name": "get_weather", "parameters": {'  # malformed
            '{"name": "get_weather", "parameters": {}}'
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
```
**EN:** This test exercises `test_invalid_then_valid_json` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_invalid_then_valid_json`。

### Lines 955-959: test case plain text only / 测试用例 plain text only
```python
    def test_plain_text_only(self):
        text = "This is just plain explanation text."
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(result.calls, [])
        self.assertEqual(result.normal_text, text)
```
**EN:** This test exercises `test_plain_text_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_plain_text_only`。

### Lines 961-965: test case with python tag prefix / 测试用例 with python tag prefix
```python
    def test_with_python_tag_prefix(self):
        text = 'Some intro. <|python_tag|>{"name": "get_weather", "parameters": {}}'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertTrue(result.normal_text.strip().startswith("Some intro."))
```
**EN:** This test exercises `test_with_python_tag_prefix` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_with_python_tag_prefix`。

### Lines 968-969: class TestKimiK2Detector declaration / 类 TestKimiK2Detector 声明
```python
class TestKimiK2Detector(unittest.TestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 970-1008: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Set up test tools and detector."""
        self.tools = [
            Tool(
                type="function",
                function=Function(
                    name="get_weather",
                    description="Get weather information",
                    parameters={
                        "type": "object",
                        "properties": {
                            "city": {
                                "type": "string",
                                "description": "City name",
                            }
                        },
                        "required": ["city"],
                    },
                ),
            ),
            Tool(
                type="function",
                function=Function(
                    name="get_tourist_attractions",
                    description="Get tourist attractions",
                    parameters={
                        "type": "object",
                        "properties": {
                            "city": {
                                "type": "string",
                                "description": "City name",
                            }
                        },
                        "required": ["city"],
                    },
                ),
            ),
        ]
        self.detector = KimiK2Detector()
```
**EN:** Set up test tools and detector. This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Set up test tools and detector. 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 1010-1017: test case single tool call / 测试用例 single tool call
```python
    def test_single_tool_call(self):
        """Test parsing a single tool call in a complete text."""
        text = '<|tool_calls_section_begin|><|tool_call_begin|>functions.get_weather:0<|tool_call_argument_begin|>{"city": "Paris"}<|tool_call_end|><|tool_calls_section_end|>'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(result.calls[0].parameters, '{"city": "Paris"}')
        self.assertEqual(result.normal_text, "")
```
**EN:** Test parsing a single tool call in a complete text. This test exercises `test_single_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing a single tool call in a complete text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_tool_call`。

### Lines 1019-1028: test case multiple tool calls / 测试用例 multiple tool calls
```python
    def test_multiple_tool_calls(self):
        """Test parsing multiple tool calls in a complete text."""
        text = '<|tool_calls_section_begin|><|tool_call_begin|>functions.get_weather:0<|tool_call_argument_begin|>{"city": "Paris"}<|tool_call_end|><|tool_call_begin|>functions.get_tourist_attractions:1<|tool_call_argument_begin|>{"city": "London"}<|tool_call_end|><|tool_calls_section_end|>'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 2)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(result.calls[0].parameters, '{"city": "Paris"}')
        self.assertEqual(result.calls[1].name, "get_tourist_attractions")
        self.assertEqual(result.calls[1].parameters, '{"city": "London"}')
        self.assertEqual(result.normal_text, "")
```
**EN:** Test parsing multiple tool calls in a complete text. This test exercises `test_multiple_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing multiple tool calls in a complete text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_tool_calls`。

### Lines 1030-1057: test case streaming tool call / 测试用例 streaming tool call
```python
    def test_streaming_tool_call(self):
        """Test streaming incremental parsing of a tool call."""
        chunks = [
            "<|tool_calls_section_begin|><|tool_call_begin|>functions.get_weather:0<|tool_call_argument_begin|>{",
            '"city": "Paris"',
            "}",
            "<|tool_call_end|><|tool_calls_section_end|>",
        ]

        tool_calls = []
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            for tool_call_chunk in result.calls:
                if tool_call_chunk.tool_index is not None:

                    while len(tool_calls) <= tool_call_chunk.tool_index:
                        tool_calls.append({"name": "", "parameters": ""})

                    tc = tool_calls[tool_call_chunk.tool_index]

                    if tool_call_chunk.name:
                        tc["name"] += tool_call_chunk.name
                    if tool_call_chunk.parameters:
                        tc["parameters"] += tool_call_chunk.parameters

        self.assertEqual(len(tool_calls), 1)
        self.assertEqual(tool_calls[0]["name"], "get_weather")
        self.assertEqual(tool_calls[0]["parameters"], '{"city": "Paris"}')
```
**EN:** Test streaming incremental parsing of a tool call. This test exercises `test_streaming_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming incremental parsing of a tool call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_tool_call`。

### Lines 1059-1091: test case streaming multiple tool calls / 测试用例 streaming multiple tool calls
```python
    def test_streaming_multiple_tool_calls(self):
        """Test streaming incremental parsing of multiple tool calls."""
        chunks = [
            "<|tool_calls_section_begin|><|tool_call_begin|>functions.get_weather:0<|tool_call_argument_begin|>{",
            '"city": "Paris"',
            "}<|tool_call_end|>",
            "<|tool_call_begin|>functions.get_tourist_attractions:1<|tool_call_argument_begin|>{",
            '"city": "London"',
            "}<|tool_call_end|>",
            "<|tool_calls_section_end|>",
        ]

        tool_calls = []
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            for tool_call_chunk in result.calls:
                if tool_call_chunk.tool_index is not None:

                    while len(tool_calls) <= tool_call_chunk.tool_index:
                        tool_calls.append({"name": "", "parameters": ""})

                    tc = tool_calls[tool_call_chunk.tool_index]

                    if tool_call_chunk.name:
                        tc["name"] += tool_call_chunk.name
                    if tool_call_chunk.parameters:
                        tc["parameters"] += tool_call_chunk.parameters

        self.assertEqual(len(tool_calls), 2)
        self.assertEqual(tool_calls[0]["name"], "get_weather")
        self.assertEqual(tool_calls[0]["parameters"], '{"city": "Paris"}')
        self.assertEqual(tool_calls[1]["name"], "get_tourist_attractions")
        self.assertEqual(tool_calls[1]["parameters"], '{"city": "London"}')
```
**EN:** Test streaming incremental parsing of multiple tool calls. This test exercises `test_streaming_multiple_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming incremental parsing of multiple tool calls. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_multiple_tool_calls`。

### Lines 1093-1108: test case tool call completion / 测试用例 tool call completion
```python
    def test_tool_call_completion(self):
        """Test that the buffer and state are reset after a tool call is completed."""
        chunks = [
            "<|tool_calls_section_begin|><|tool_call_begin|>functions.get_weather:0<|tool_call_argument_begin|>{",
            '"city": "Paris"',
            "}",
            "<|tool_call_end|>",
            "<|tool_calls_section_end|>",
        ]

        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)

        # After processing all chunks, the buffer should be empty and current_tool_id should be reset
        self.assertEqual(self.detector._buffer, "")
        self.assertEqual(self.detector.current_tool_id, 1)
```
**EN:** Test that the buffer and state are reset after a tool call is completed. This test exercises `test_tool_call_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that the buffer and state are reset after a tool call is completed. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_call_completion`。

### Lines 1110-1139: test case tool name streaming / 测试用例 tool name streaming
```python
    def test_tool_name_streaming(self):
        """Test that tool names are streamed correctly with the right index."""
        chunks = [
            "<|tool_calls_section_begin|><|tool_call_begin|>functions.get_weather:0<|tool_call_argument_begin|>{",
            '"city": "Paris"',
            "}",
            "<|tool_call_end|>",
            "<|tool_call_begin|>functions.get_tourist_attractions:1<|tool_call_argument_begin|>{",
        ]

        tool_calls = []
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            for tool_call_chunk in result.calls:
                if tool_call_chunk.tool_index is not None:

                    while len(tool_calls) <= tool_call_chunk.tool_index:
                        tool_calls.append({"name": "", "parameters": ""})

                    tc = tool_calls[tool_call_chunk.tool_index]

                    if tool_call_chunk.name:
                        tc["name"] += tool_call_chunk.name
                    if tool_call_chunk.parameters:
                        tc["parameters"] += tool_call_chunk.parameters

        self.assertEqual(len(tool_calls), 2)
        self.assertEqual(tool_calls[0]["name"], "get_weather")
        self.assertEqual(tool_calls[0]["parameters"], '{"city": "Paris"}')
        self.assertEqual(tool_calls[1]["name"], "get_tourist_attractions")
```
**EN:** Test that tool names are streamed correctly with the right index. This test exercises `test_tool_name_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that tool names are streamed correctly with the right index. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_name_streaming`。

### Lines 1141-1146: test case invalid tool call / 测试用例 invalid tool call
```python
    def test_invalid_tool_call(self):
        """Test that invalid tool calls are handled correctly."""
        text = 'invalid_tool:0<|tool_call_argument_begin|>{"city": "Paris"}<|tool_call_end|><|tool_calls_section_end|>'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 0)
        self.assertEqual(result.normal_text, text)
```
**EN:** Test that invalid tool calls are handled correctly. This test exercises `test_invalid_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that invalid tool calls are handled correctly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_invalid_tool_call`。

### Lines 1148-1173: test case partial tool call / 测试用例 partial tool call
```python
    def test_partial_tool_call(self):
        """Test that partial tool calls are handled correctly in streaming mode."""
        chunks = [
            "<|tool_calls_section_begin|><|tool_call_begin|>functions.get_weather:0<|tool_call_argument_begin|>{",
            '"city": "Paris"',
        ]

        tool_calls = []
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            for tool_call_chunk in result.calls:
                if tool_call_chunk.tool_index is not None:

                    while len(tool_calls) <= tool_call_chunk.tool_index:
                        tool_calls.append({"name": "", "parameters": ""})

                    tc = tool_calls[tool_call_chunk.tool_index]

                    if tool_call_chunk.name:
                        tc["name"] += tool_call_chunk.name
                    if tool_call_chunk.parameters:
                        tc["parameters"] += tool_call_chunk.parameters

        self.assertEqual(len(tool_calls), 1)
        self.assertEqual(tool_calls[0]["name"], "get_weather")
        self.assertEqual(tool_calls[0]["parameters"], '{"city": "Paris"')
```
**EN:** Test that partial tool calls are handled correctly in streaming mode. This test exercises `test_partial_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that partial tool calls are handled correctly in streaming mode. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_partial_tool_call`。

### Lines 1176-1176: class TestDeepSeekV3Detector declaration / 类 TestDeepSeekV3Detector 声明
```python
class TestDeepSeekV3Detector(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 1177-1215: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Set up test tools and detector for DeepSeekV3 format testing."""
        self.tools = [
            Tool(
                type="function",
                function=Function(
                    name="get_weather",
                    description="Get weather information",
                    parameters={
                        "type": "object",
                        "properties": {
                            "city": {
                                "type": "string",
                                "description": "City name",
                            }
                        },
                        "required": ["city"],
                    },
                ),
            ),
            Tool(
                type="function",
                function=Function(
                    name="get_tourist_attractions",
                    description="Get tourist attractions",
                    parameters={
                        "type": "object",
                        "properties": {
                            "city": {
                                "type": "string",
                                "description": "City name",
                            }
                        },
                        "required": ["city"],
                    },
                ),
            ),
        ]
        self.detector = DeepSeekV3Detector()
```
**EN:** Set up test tools and detector for DeepSeekV3 format testing. This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Set up test tools and detector for DeepSeekV3 format testing. 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 1217-1262: test case parse streaming multiple tool calls with multi token chunk / 测试用例 parse streaming multiple tool calls with multi token chunk
```python
    def test_parse_streaming_multiple_tool_calls_with_multi_token_chunk(self):
        """Test parsing multiple tool calls when streaming chunks contains multi-tokens (e.g. DeepSeekV3 enable MTP)"""
        # Simulate streaming chunks with multi-tokens for two consecutive tool calls
        chunks = [
            "<｜tool▁calls▁begin｜>",
            "<｜tool▁call▁begin｜>function",
            "<｜tool▁sep｜>get",
            "_weather\n",
            "```json\n",
            '{"city":',
            '"Shanghai',
            '"}\n```<｜tool▁call▁end｜>',
            "\n<｜tool▁call▁begin｜>",
            "function<｜tool▁sep｜>",
            "get_tour",
            "ist_att",
            "ractions\n```" 'json\n{"',
            'city": "',
            'Beijing"}\n',
            "```<｜tool▁call▁end｜>",
            "<｜tool▁calls▁end｜>",
        ]

        tool_calls_seen = []
        tool_calls_parameters = []

        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            if result.calls:
                for call in result.calls:
                    if call.name:
                        tool_calls_seen.append(call.name)
                    if call.parameters:
                        tool_calls_parameters.append(call.parameters)

        # Should see both tool names
        self.assertIn("get_weather", tool_calls_seen, "Should process first tool")
        self.assertIn(
            "get_tourist_attractions", tool_calls_seen, "Should process second tool"
        )

        # Verify that the parameters are valid JSON and contain the expected content
        params1 = json.loads(tool_calls_parameters[0])
        params2 = json.loads(tool_calls_parameters[1])
        self.assertEqual(params1["city"], "Shanghai")
        self.assertEqual(params2["city"], "Beijing")
```
**EN:** Test parsing multiple tool calls when streaming chunks contains multi-tokens (e.g. This test exercises `test_parse_streaming_multiple_tool_calls_with_multi_token_chunk` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing multiple tool calls when streaming chunks contains multi-tokens (e.g. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_multiple_tool_calls_with_multi_token_chunk`。

### Lines 1265-1265: class TestDeepSeekV32Detector declaration / 类 TestDeepSeekV32Detector 声明
```python
class TestDeepSeekV32Detector(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 1266-1314: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Set up test tools and detector for DeepSeekV32 format testing."""
        self.tools = [
            Tool(
                type="function",
                function=Function(
                    name="search",
                    description="Searches for information related to query and displays topn results.",
                    parameters={
                        "type": "object",
                        "properties": {
                            "query": {
                                "type": "string",
                                "description": "The search query string",
                            },
                            "topn": {
                                "type": "integer",
                                "description": "Number of top results to display",
                                "default": 10,
                            },
                            "source": {
                                "type": "string",
                                "description": "Source to search within",
                                "enum": ["web", "news"],
                                "default": "web",
                            },
                        },
                        "required": ["query"],
                    },
                ),
            ),
            Tool(
                type="function",
                function=Function(
                    name="get_favorite_tourist_spot",
                    description="Return the favorite tourist spot for a given city.",
                    parameters={
                        "type": "object",
                        "properties": {"city": {"type": "string"}},
                        "required": ["city"],
                    },
                ),
            ),
        ]
        self.detector = DeepSeekV32Detector()
        from sglang.srt.utils.hf_transformers_utils import get_tokenizer

        self.tokenizer = get_tokenizer("deepseek-ai/DeepSeek-V3.2")
        self.interval = 1
```
**EN:** Set up test tools and detector for DeepSeekV32 format testing. This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Set up test tools and detector for DeepSeekV32 format testing. 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 1316-1347: test case detect and parse xml format / 测试用例 detect and parse xml format
```python
    def test_detect_and_parse_xml_format(self):
        """Test parsing standard XML format (DSML)"""
        text = """I'll help you with information about San Francisco and get its favorite tourist spot for you.\n\n
        <｜DSML｜function_calls>\n
            <｜DSML｜invoke name="get_favorite_tourist_spot">\n
                <｜DSML｜parameter name="city" string="true">San Francisco</｜DSML｜parameter>\n
            </｜DSML｜invoke>\n
            <｜DSML｜invoke name="search">
                <｜DSML｜parameter name="query" string="true">WebNav benchmark</｜DSML｜parameter>
                <｜DSML｜parameter name="topn" string="false">10</｜DSML｜parameter>
                <｜DSML｜parameter name="source" string="true">web</｜DSML｜parameter>
            </｜DSML｜invoke>
        </｜DSML｜function_calls>
        """
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertIn("I'll help you with information", result.normal_text)
        self.assertEqual(len(result.calls), 2)

        # Check first call
        call1 = result.calls[0]
        self.assertEqual(call1.name, "get_favorite_tourist_spot")
        params1 = json.loads(call1.parameters)
        self.assertEqual(params1["city"], "San Francisco")

        # Check second call
        call2 = result.calls[1]
        self.assertEqual(call2.name, "search")
        params2 = json.loads(call2.parameters)
        self.assertEqual(params2["query"], "WebNav benchmark")
        self.assertEqual(params2["topn"], 10)
        self.assertEqual(params2["source"], "web")
```
**EN:** Test parsing standard XML format (DSML) This test exercises `test_detect_and_parse_xml_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing standard XML format (DSML) 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_xml_format`。

### Lines 1349-1385: test case detect and parse json format / 测试用例 detect and parse json format
```python
    def test_detect_and_parse_json_format(self):
        """Test parsing JSON format inside invoke tags"""
        text = """I'll help you with information about San Francisco and get its favorite tourist spot for you.

        <｜DSML｜function_calls>
            <｜DSML｜invoke name="get_favorite_tourist_spot">
            {
                "city": "San Francisco"
            }
        </｜DSML｜invoke>
            <｜DSML｜invoke name="search">
            {
                "query": "WebNav benchmark",
                "topn": 10,
                "source": "web"
            }
        </｜DSML｜invoke>
        </｜DSML｜function_calls>
        """
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertIn("I'll help you with information", result.normal_text)
        self.assertEqual(len(result.calls), 2)

        # Check first call
        call1 = result.calls[0]
        self.assertEqual(call1.name, "get_favorite_tourist_spot")
        params1 = json.loads(call1.parameters)
        self.assertEqual(params1["city"], "San Francisco")

        # Check second call
        call2 = result.calls[1]
        self.assertEqual(call2.name, "search")
        params2 = json.loads(call2.parameters)
        self.assertEqual(params2["query"], "WebNav benchmark")
        self.assertEqual(params2["topn"], 10)
        self.assertEqual(params2["source"], "web")
```
**EN:** Test parsing JSON format inside invoke tags This test exercises `test_detect_and_parse_json_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing JSON format inside invoke tags 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_json_format`。

### Lines 1387-1435: test case streaming xml format / 测试用例 streaming xml format
```python
    def test_streaming_xml_format(self):
        """Test streaming parsing of XML format"""
        text = """<｜DSML｜function_calls>
            <｜DSML｜invoke name="get_favorite_tourist_spot">
                <｜DSML｜parameter name="city" string="true">San Francisco</｜DSML｜parameter>
                <｜DSML｜parameter name="another_city" string="true">London</｜DSML｜parameter>
                <｜DSML｜parameter name="topn" string="false">10</｜DSML｜parameter>
                <｜DSML｜parameter name="obj" string="false">{"name": "John", "age": 30}</｜DSML｜parameter>
            </｜DSML｜invoke>
        </｜DSML｜function_calls>"""

        input_ids = self.tokenizer.encode(text, add_special_tokens=False)
        chunk_ids = [
            input_ids[i : i + self.interval]
            for i in range(0, len(input_ids), self.interval)
        ]
        chunks = [self.tokenizer.decode(chunk_id) for chunk_id in chunk_ids]

        tool_calls_by_index = {}

        num_tool_call_chunks = 0
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            for call in result.calls:
                num_tool_call_chunks += 1
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }

                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters

        self.assertGreater(num_tool_call_chunks, 8)

        self.assertEqual(len(tool_calls_by_index), 1)
        self.assertEqual(tool_calls_by_index[0]["name"], "get_favorite_tourist_spot")
        params = json.loads(tool_calls_by_index[0]["parameters"])
        self.assertEqual(params["city"], "San Francisco")
        self.assertEqual(params["another_city"], "London")
        self.assertEqual(params["topn"], 10)
        self.assertEqual(params["obj"]["name"], "John")
        self.assertEqual(params["obj"]["age"], 30)
```
**EN:** Test streaming parsing of XML format This test exercises `test_streaming_xml_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parsing of XML format 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_xml_format`。

### Lines 1437-1488: test case streaming json format / 测试用例 streaming json format
```python
    def test_streaming_json_format(self):
        """Test streaming parsing of JSON format"""
        text = """<｜DSML｜function_calls>
            <｜DSML｜invoke name="get_favorite_tourist_spot">
            {
                "city": "San Francisco",
                "another_city": "London",
                "topn": 10,
                "obj": {
                    "name": "John",
                    "age": 30
                }
            }
            </｜DSML｜invoke>
        </｜DSML｜function_calls>"""

        input_ids = self.tokenizer.encode(text, add_special_tokens=False)
        chunk_ids = [
            input_ids[i : i + self.interval]
            for i in range(0, len(input_ids), self.interval)
        ]
        chunks = [self.tokenizer.decode(chunk_id) for chunk_id in chunk_ids]

        tool_calls_by_index = {}

        num_tool_call_chunks = 0
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            for call in result.calls:
                num_tool_call_chunks += 1
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }

                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters

        self.assertGreater(num_tool_call_chunks, 8)
        self.assertEqual(len(tool_calls_by_index), 1)
        self.assertEqual(tool_calls_by_index[0]["name"], "get_favorite_tourist_spot")

        # Clean up parameters string if needed (trim whitespace)
        params_str = tool_calls_by_index[0]["parameters"].strip()
        params = json.loads(params_str)
        self.assertEqual(params["city"], "San Francisco")
```
**EN:** Test streaming parsing of JSON format This test exercises `test_streaming_json_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parsing of JSON format 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_json_format`。

### Lines 1490-1519: test case detect and parse no parameters / 测试用例 detect and parse no parameters
```python
    def test_detect_and_parse_no_parameters(self):
        """Test parsing function calls with no parameters (non-streaming)"""
        # Add a no-parameter tool
        tools_with_no_param = self.tools + [
            Tool(
                type="function",
                function=Function(
                    name="get_date",
                    description="Get the current date.",
                    parameters={"type": "object", "properties": {}},
                ),
            ),
        ]

        text = """Let me get the current date for you.

<｜DSML｜function_calls>
<｜DSML｜invoke name="get_date">
</｜DSML｜invoke>
</｜DSML｜function_calls>"""

        result = self.detector.detect_and_parse(text, tools_with_no_param)

        self.assertIn("Let me get the current date", result.normal_text)
        self.assertEqual(len(result.calls), 1)

        call = result.calls[0]
        self.assertEqual(call.name, "get_date")
        params = json.loads(call.parameters)
        self.assertEqual(params, {})
```
**EN:** Test parsing function calls with no parameters (non-streaming) This test exercises `test_detect_and_parse_no_parameters` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing function calls with no parameters (non-streaming) 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_no_parameters`。

### Lines 1521-1583: test case streaming no parameters / 测试用例 streaming no parameters
```python
    def test_streaming_no_parameters(self):
        """Test streaming parsing of function calls with no parameters.

        This test verifies the fix for the bug where functions with no parameters
        were being silently skipped in streaming mode.
        """
        # Add a no-parameter tool
        tools_with_no_param = self.tools + [
            Tool(
                type="function",
                function=Function(
                    name="get_date",
                    description="Get the current date.",
                    parameters={"type": "object", "properties": {}},
                ),
            ),
        ]

        text = """<｜DSML｜function_calls>
<｜DSML｜invoke name="get_date">
</｜DSML｜invoke>
</｜DSML｜function_calls>"""

        # Reset detector state
        self.detector = DeepSeekV32Detector()

        # Simulate streaming by splitting into small chunks
        input_ids = self.tokenizer.encode(text, add_special_tokens=False)
        chunk_ids = [
            input_ids[i : i + self.interval]
            for i in range(0, len(input_ids), self.interval)
        ]
        chunks = [self.tokenizer.decode(chunk_id) for chunk_id in chunk_ids]

        tool_calls_by_index = {}

        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, tools_with_no_param)
            for call in result.calls:
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }

                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters

        # Verify that the no-parameter function was correctly parsed
        self.assertEqual(
            len(tool_calls_by_index), 1, "Should have exactly one tool call"
        )
        self.assertEqual(tool_calls_by_index[0]["name"], "get_date")

        # Parameters should be empty JSON object
        params_str = tool_calls_by_index[0]["parameters"].strip()
        params = json.loads(params_str)
        self.assertEqual(params, {})
```
**EN:** Test streaming parsing of function calls with no parameters. This test exercises `test_streaming_no_parameters` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parsing of function calls with no parameters. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_no_parameters`。

### Lines 1585-1640: test case streaming no parameters with whitespace / 测试用例 streaming no parameters with whitespace
```python
    def test_streaming_no_parameters_with_whitespace(self):
        """Test streaming parsing when invoke content has only whitespace (newlines)."""
        tools_with_no_param = self.tools + [
            Tool(
                type="function",
                function=Function(
                    name="get_date",
                    description="Get the current date.",
                    parameters={"type": "object", "properties": {}},
                ),
            ),
        ]

        # This format has newlines inside the invoke tag (common model output)
        text = """<｜DSML｜function_calls>
<｜DSML｜invoke name="get_date">

</｜DSML｜invoke>
</｜DSML｜function_calls>"""

        # Reset detector state
        self.detector = DeepSeekV32Detector()

        input_ids = self.tokenizer.encode(text, add_special_tokens=False)
        chunk_ids = [
            input_ids[i : i + self.interval]
            for i in range(0, len(input_ids), self.interval)
        ]
        chunks = [self.tokenizer.decode(chunk_id) for chunk_id in chunk_ids]

        tool_calls_by_index = {}

        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, tools_with_no_param)
            for call in result.calls:
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }

                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters

        # Should still parse correctly even with whitespace-only content
        self.assertEqual(
            len(tool_calls_by_index), 1, "Should have exactly one tool call"
        )
        self.assertEqual(tool_calls_by_index[0]["name"], "get_date")
        params = json.loads(tool_calls_by_index[0]["parameters"])
        self.assertEqual(params, {})
```
**EN:** Test streaming parsing when invoke content has only whitespace (newlines). This test exercises `test_streaming_no_parameters_with_whitespace` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parsing when invoke content has only whitespace (newlines). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_no_parameters_with_whitespace`。

### Lines 1642-1687: test case get model structural tag / 测试用例 get model structural tag
```python
    def test_get_model_structural_tag(self):
        import xgrammar as xgr

        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=True
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)

        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=False
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)

        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=True, tool_choice="required"
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)

        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=False, tool_choice="required"
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)

        tool_choice_name = ToolChoiceFuncName(name="search")
        tool_choice = ToolChoice(function=tool_choice_name)
        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=True, tool_choice=tool_choice
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)

        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=False, tool_choice=tool_choice
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)
```
**EN:** This test exercises `test_get_model_structural_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_model_structural_tag`。

### Lines 1689-1707: test case self closing zero arg invoke / 测试用例 self closing zero arg invoke
```python
    def test_self_closing_zero_arg_invoke(self):
        """V32 inherits the same regex; verify self-closing parses to empty
        params here too (V32 model rarely emits this shape, but the parser
        must agree with V4 since V4 inherits from V32)."""
        submit_tool = Tool(
            type="function",
            function=Function(
                name="submit",
                parameters={"type": "object", "properties": {}},
            ),
        )
        text = (
            '<｜DSML｜function_calls>\n<｜DSML｜invoke name="submit"/>\n'
            "</｜DSML｜function_calls>"
        )
        result = self.detector.detect_and_parse(text, [submit_tool])
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "submit")
        self.assertEqual(json.loads(result.calls[0].parameters), {})
```
**EN:** V32 inherits the same regex; verify self-closing parses to empty params here too (V32 model rarely emits this shape, but the parser must agree with V4 since V4 inherits from V32). This test exercises `test_self_closing_zero_arg_invoke` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** V32 inherits the same regex; verify self-closing parses to empty params here too (V32 model rarely emits this shape, but the parser must agree with V4 since V4 inherits from V32). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_self_closing_zero_arg_invoke`。

### Lines 1710-1710: class TestDeepSeekV4Detector declaration / 类 TestDeepSeekV4Detector 声明
```python
class TestDeepSeekV4Detector(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 1711-1759: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Set up test tools and detector for DeepSeekV4 format testing."""
        self.tools = [
            Tool(
                type="function",
                function=Function(
                    name="search",
                    description="Searches for information related to query and displays topn results.",
                    parameters={
                        "type": "object",
                        "properties": {
                            "query": {
                                "type": "string",
                                "description": "The search query string",
                            },
                            "topn": {
                                "type": "integer",
                                "description": "Number of top results to display",
                                "default": 10,
                            },
                            "source": {
                                "type": "string",
                                "description": "Source to search within",
                                "enum": ["web", "news"],
                                "default": "web",
                            },
                        },
                        "required": ["query"],
                    },
                ),
            ),
            Tool(
                type="function",
                function=Function(
                    name="get_favorite_tourist_spot",
                    description="Return the favorite tourist spot for a given city.",
                    parameters={
                        "type": "object",
                        "properties": {"city": {"type": "string"}},
                        "required": ["city"],
                    },
                ),
            ),
        ]
        self.detector = DeepSeekV4Detector()
        from sglang.srt.utils.hf_transformers_utils import get_tokenizer

        self.tokenizer = get_tokenizer("deepseek-ai/DeepSeek-V3.2")
        self.interval = 1
```
**EN:** Set up test tools and detector for DeepSeekV4 format testing. This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Set up test tools and detector for DeepSeekV4 format testing. 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 1761-1792: test case detect and parse xml format / 测试用例 detect and parse xml format
```python
    def test_detect_and_parse_xml_format(self):
        """Test parsing standard XML format (DSML)"""
        text = """I'll help you with information about San Francisco and get its favorite tourist spot for you.\n\n
        <｜DSML｜tool_calls>\n
            <｜DSML｜invoke name="get_favorite_tourist_spot">\n
                <｜DSML｜parameter name="city" string="true">San Francisco</｜DSML｜parameter>\n
            </｜DSML｜invoke>\n
            <｜DSML｜invoke name="search">
                <｜DSML｜parameter name="query" string="true">WebNav benchmark</｜DSML｜parameter>
                <｜DSML｜parameter name="topn" string="false">10</｜DSML｜parameter>
                <｜DSML｜parameter name="source" string="true">web</｜DSML｜parameter>
            </｜DSML｜invoke>
        </｜DSML｜tool_calls>
        """
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertIn("I'll help you with information", result.normal_text)
        self.assertEqual(len(result.calls), 2)

        # Check first call
        call1 = result.calls[0]
        self.assertEqual(call1.name, "get_favorite_tourist_spot")
        params1 = json.loads(call1.parameters)
        self.assertEqual(params1["city"], "San Francisco")

        # Check second call
        call2 = result.calls[1]
        self.assertEqual(call2.name, "search")
        params2 = json.loads(call2.parameters)
        self.assertEqual(params2["query"], "WebNav benchmark")
        self.assertEqual(params2["topn"], 10)
        self.assertEqual(params2["source"], "web")
```
**EN:** Test parsing standard XML format (DSML) This test exercises `test_detect_and_parse_xml_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing standard XML format (DSML) 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_xml_format`。

### Lines 1794-1830: test case detect and parse json format / 测试用例 detect and parse json format
```python
    def test_detect_and_parse_json_format(self):
        """Test parsing JSON format inside invoke tags"""
        text = """I'll help you with information about San Francisco and get its favorite tourist spot for you.

        <｜DSML｜tool_calls>
            <｜DSML｜invoke name="get_favorite_tourist_spot">
            {
                "city": "San Francisco"
            }
        </｜DSML｜invoke>
            <｜DSML｜invoke name="search">
            {
                "query": "WebNav benchmark",
                "topn": 10,
                "source": "web"
            }
        </｜DSML｜invoke>
        </｜DSML｜tool_calls>
        """
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertIn("I'll help you with information", result.normal_text)
        self.assertEqual(len(result.calls), 2)

        # Check first call
        call1 = result.calls[0]
        self.assertEqual(call1.name, "get_favorite_tourist_spot")
        params1 = json.loads(call1.parameters)
        self.assertEqual(params1["city"], "San Francisco")

        # Check second call
        call2 = result.calls[1]
        self.assertEqual(call2.name, "search")
        params2 = json.loads(call2.parameters)
        self.assertEqual(params2["query"], "WebNav benchmark")
        self.assertEqual(params2["topn"], 10)
        self.assertEqual(params2["source"], "web")
```
**EN:** Test parsing JSON format inside invoke tags This test exercises `test_detect_and_parse_json_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing JSON format inside invoke tags 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_json_format`。

### Lines 1832-1880: test case streaming xml format / 测试用例 streaming xml format
```python
    def test_streaming_xml_format(self):
        """Test streaming parsing of XML format"""
        text = """<｜DSML｜tool_calls>
            <｜DSML｜invoke name="get_favorite_tourist_spot">
                <｜DSML｜parameter name="city" string="true">San Francisco</｜DSML｜parameter>
                <｜DSML｜parameter name="another_city" string="true">London</｜DSML｜parameter>
                <｜DSML｜parameter name="topn" string="false">10</｜DSML｜parameter>
                <｜DSML｜parameter name="obj" string="false">{"name": "John", "age": 30}</｜DSML｜parameter>
            </｜DSML｜invoke>
        </｜DSML｜tool_calls>"""

        input_ids = self.tokenizer.encode(text, add_special_tokens=False)
        chunk_ids = [
            input_ids[i : i + self.interval]
            for i in range(0, len(input_ids), self.interval)
        ]
        chunks = [self.tokenizer.decode(chunk_id) for chunk_id in chunk_ids]

        tool_calls_by_index = {}

        num_tool_call_chunks = 0
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            for call in result.calls:
                num_tool_call_chunks += 1
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }

                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters

        self.assertGreater(num_tool_call_chunks, 8)

        self.assertEqual(len(tool_calls_by_index), 1)
        self.assertEqual(tool_calls_by_index[0]["name"], "get_favorite_tourist_spot")
        params = json.loads(tool_calls_by_index[0]["parameters"])
        self.assertEqual(params["city"], "San Francisco")
        self.assertEqual(params["another_city"], "London")
        self.assertEqual(params["topn"], 10)
        self.assertEqual(params["obj"]["name"], "John")
        self.assertEqual(params["obj"]["age"], 30)
```
**EN:** Test streaming parsing of XML format This test exercises `test_streaming_xml_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parsing of XML format 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_xml_format`。

### Lines 1882-1933: test case streaming json format / 测试用例 streaming json format
```python
    def test_streaming_json_format(self):
        """Test streaming parsing of JSON format"""
        text = """<｜DSML｜tool_calls>
            <｜DSML｜invoke name="get_favorite_tourist_spot">
            {
                "city": "San Francisco",
                "another_city": "London",
                "topn": 10,
                "obj": {
                    "name": "John",
                    "age": 30
                }
            }
            </｜DSML｜invoke>
        </｜DSML｜tool_calls>"""

        input_ids = self.tokenizer.encode(text, add_special_tokens=False)
        chunk_ids = [
            input_ids[i : i + self.interval]
            for i in range(0, len(input_ids), self.interval)
        ]
        chunks = [self.tokenizer.decode(chunk_id) for chunk_id in chunk_ids]

        tool_calls_by_index = {}

        num_tool_call_chunks = 0
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            for call in result.calls:
                num_tool_call_chunks += 1
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }

                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters

        self.assertGreater(num_tool_call_chunks, 8)
        self.assertEqual(len(tool_calls_by_index), 1)
        self.assertEqual(tool_calls_by_index[0]["name"], "get_favorite_tourist_spot")

        # Clean up parameters string if needed (trim whitespace)
        params_str = tool_calls_by_index[0]["parameters"].strip()
        params = json.loads(params_str)
        self.assertEqual(params["city"], "San Francisco")
```
**EN:** Test streaming parsing of JSON format This test exercises `test_streaming_json_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parsing of JSON format 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_json_format`。

### Lines 1935-1964: test case detect and parse no parameters / 测试用例 detect and parse no parameters
```python
    def test_detect_and_parse_no_parameters(self):
        """Test parsing function calls with no parameters (non-streaming)"""
        # Add a no-parameter tool
        tools_with_no_param = self.tools + [
            Tool(
                type="function",
                function=Function(
                    name="get_date",
                    description="Get the current date.",
                    parameters={"type": "object", "properties": {}},
                ),
            ),
        ]

        text = """Let me get the current date for you.

<｜DSML｜tool_calls>
<｜DSML｜invoke name="get_date">
</｜DSML｜invoke>
</｜DSML｜tool_calls>"""

        result = self.detector.detect_and_parse(text, tools_with_no_param)

        self.assertIn("Let me get the current date", result.normal_text)
        self.assertEqual(len(result.calls), 1)

        call = result.calls[0]
        self.assertEqual(call.name, "get_date")
        params = json.loads(call.parameters)
        self.assertEqual(params, {})
```
**EN:** Test parsing function calls with no parameters (non-streaming) This test exercises `test_detect_and_parse_no_parameters` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing function calls with no parameters (non-streaming) 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_no_parameters`。

### Lines 1966-2028: test case streaming no parameters / 测试用例 streaming no parameters
```python
    def test_streaming_no_parameters(self):
        """Test streaming parsing of function calls with no parameters.

        This test verifies the fix for the bug where functions with no parameters
        were being silently skipped in streaming mode.
        """
        # Add a no-parameter tool
        tools_with_no_param = self.tools + [
            Tool(
                type="function",
                function=Function(
                    name="get_date",
                    description="Get the current date.",
                    parameters={"type": "object", "properties": {}},
                ),
            ),
        ]

        text = """<｜DSML｜tool_calls>
<｜DSML｜invoke name="get_date">
</｜DSML｜invoke>
</｜DSML｜tool_calls>"""

        # Reset detector state
        self.detector = DeepSeekV4Detector()

        # Simulate streaming by splitting into small chunks
        input_ids = self.tokenizer.encode(text, add_special_tokens=False)
        chunk_ids = [
            input_ids[i : i + self.interval]
            for i in range(0, len(input_ids), self.interval)
        ]
        chunks = [self.tokenizer.decode(chunk_id) for chunk_id in chunk_ids]

        tool_calls_by_index = {}

        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, tools_with_no_param)
            for call in result.calls:
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }

                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters

        # Verify that the no-parameter function was correctly parsed
        self.assertEqual(
            len(tool_calls_by_index), 1, "Should have exactly one tool call"
        )
        self.assertEqual(tool_calls_by_index[0]["name"], "get_date")

        # Parameters should be empty JSON object
        params_str = tool_calls_by_index[0]["parameters"].strip()
        params = json.loads(params_str)
        self.assertEqual(params, {})
```
**EN:** Test streaming parsing of function calls with no parameters. This test exercises `test_streaming_no_parameters` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parsing of function calls with no parameters. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_no_parameters`。

### Lines 2030-2085: test case streaming no parameters with whitespace / 测试用例 streaming no parameters with whitespace
```python
    def test_streaming_no_parameters_with_whitespace(self):
        """Test streaming parsing when invoke content has only whitespace (newlines)."""
        tools_with_no_param = self.tools + [
            Tool(
                type="function",
                function=Function(
                    name="get_date",
                    description="Get the current date.",
                    parameters={"type": "object", "properties": {}},
                ),
            ),
        ]

        # This format has newlines inside the invoke tag (common model output)
        text = """<｜DSML｜tool_calls>
<｜DSML｜invoke name="get_date">

</｜DSML｜invoke>
</｜DSML｜tool_calls>"""

        # Reset detector state
        self.detector = DeepSeekV4Detector()

        input_ids = self.tokenizer.encode(text, add_special_tokens=False)
        chunk_ids = [
            input_ids[i : i + self.interval]
            for i in range(0, len(input_ids), self.interval)
        ]
        chunks = [self.tokenizer.decode(chunk_id) for chunk_id in chunk_ids]

        tool_calls_by_index = {}

        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, tools_with_no_param)
            for call in result.calls:
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }

                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters

        # Should still parse correctly even with whitespace-only content
        self.assertEqual(
            len(tool_calls_by_index), 1, "Should have exactly one tool call"
        )
        self.assertEqual(tool_calls_by_index[0]["name"], "get_date")
        params = json.loads(tool_calls_by_index[0]["parameters"])
        self.assertEqual(params, {})
```
**EN:** Test streaming parsing when invoke content has only whitespace (newlines). This test exercises `test_streaming_no_parameters_with_whitespace` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parsing when invoke content has only whitespace (newlines). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_no_parameters_with_whitespace`。

### Lines 2087-2132: test case get model structural tag / 测试用例 get model structural tag
```python
    def test_get_model_structural_tag(self):
        import xgrammar as xgr

        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=True
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)

        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=False
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)

        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=True, tool_choice="required"
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)

        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=False, tool_choice="required"
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)

        tool_choice_name = ToolChoiceFuncName(name="search")
        tool_choice = ToolChoice(function=tool_choice_name)
        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=True, tool_choice=tool_choice
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)

        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=False, tool_choice=tool_choice
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)
```
**EN:** This test exercises `test_get_model_structural_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_model_structural_tag`。

### Lines 2134-2156: test case self closing zero arg invoke / 测试用例 self closing zero arg invoke
```python
    def test_self_closing_zero_arg_invoke(self):
        """V4 emits `<｜DSML｜invoke name="x"/>` for zero-arg tools; the
        detector must parse it as a complete tool call with empty params
        instead of leaking the raw markup back into normal_text."""
        submit_tool = Tool(
            type="function",
            function=Function(
                name="submit",
                description="Submit the final answer.",
                parameters={"type": "object", "properties": {}},
            ),
        )

        text = (
            "Final answer.\n"
            '<｜DSML｜tool_calls>\n<｜DSML｜invoke name="submit"/>\n'
            "</｜DSML｜tool_calls>"
        )
        result = self.detector.detect_and_parse(text, [submit_tool])
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "submit")
        self.assertEqual(json.loads(result.calls[0].parameters), {})
        self.assertNotIn("DSML", result.normal_text)
```
**EN:** V4 emits `<｜DSML｜invoke name="x"/>` for zero-arg tools; the detector must parse it as a complete tool call with empty params instead of leaking the raw markup back into normal_text. This test exercises `test_self_closing_zero_arg_invoke` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** V4 emits `<｜DSML｜invoke name="x"/>` for zero-arg tools; the detector must parse it as a complete tool call with empty params instead of leaking the raw markup back into normal_text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_self_closing_zero_arg_invoke`。

### Lines 2158-2180: test case self closing mixed with long form / 测试用例 self closing mixed with long form
```python
    def test_self_closing_mixed_with_long_form(self):
        """Mix of long-form (with params) and self-closing tags in one block."""
        submit_tool = Tool(
            type="function",
            function=Function(
                name="submit",
                parameters={"type": "object", "properties": {}},
            ),
        )
        text = (
            "<｜DSML｜tool_calls>\n"
            '<｜DSML｜invoke name="get_favorite_tourist_spot">\n'
            '<｜DSML｜parameter name="city" string="true">SF</｜DSML｜parameter>\n'
            "</｜DSML｜invoke>\n"
            '<｜DSML｜invoke name="submit"/>\n'
            "</｜DSML｜tool_calls>"
        )
        result = self.detector.detect_and_parse(text, self.tools + [submit_tool])
        self.assertEqual(len(result.calls), 2)
        self.assertEqual(result.calls[0].name, "get_favorite_tourist_spot")
        self.assertEqual(json.loads(result.calls[0].parameters), {"city": "SF"})
        self.assertEqual(result.calls[1].name, "submit")
        self.assertEqual(json.loads(result.calls[1].parameters), {})
```
**EN:** Mix of long-form (with params) and self-closing tags in one block. This test exercises `test_self_closing_mixed_with_long_form` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Mix of long-form (with params) and self-closing tags in one block. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_self_closing_mixed_with_long_form`。

### Lines 2182-2222: test case streaming self closing invoke / 测试用例 streaming self closing invoke
```python
    def test_streaming_self_closing_invoke(self):
        """Self-closing invoke must terminate cleanly even when `/>` arrives
        after the `name=` attribute crosses chunk boundaries."""
        submit_tool = Tool(
            type="function",
            function=Function(
                name="submit",
                parameters={"type": "object", "properties": {}},
            ),
        )
        # Build the prompt and feed it through the tokenizer to exercise the
        # same chunk shapes the runtime sees.
        text = (
            "<｜DSML｜tool_calls>\n"
            '<｜DSML｜invoke name="submit"/>\n'
            "</｜DSML｜tool_calls>"
        )
        self.detector = DeepSeekV4Detector()
        input_ids = self.tokenizer.encode(text, add_special_tokens=False)
        chunks = [
            self.tokenizer.decode(input_ids[i : i + self.interval])
            for i in range(0, len(input_ids), self.interval)
        ]

        tool_calls_by_index = {}
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, [submit_tool])
            for call in result.calls:
                if call.tool_index is None:
                    continue
                slot = tool_calls_by_index.setdefault(
                    call.tool_index, {"name": "", "parameters": ""}
                )
                if call.name:
                    slot["name"] = call.name
                if call.parameters:
                    slot["parameters"] += call.parameters

        self.assertEqual(len(tool_calls_by_index), 1)
        self.assertEqual(tool_calls_by_index[0]["name"], "submit")
        self.assertEqual(json.loads(tool_calls_by_index[0]["parameters"]), {})
```
**EN:** Self-closing invoke must terminate cleanly even when `/>` arrives after the `name=` attribute crosses chunk boundaries. This test exercises `test_streaming_self_closing_invoke` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Self-closing invoke must terminate cleanly even when `/>` arrives after the `name=` attribute crosses chunk boundaries. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_self_closing_invoke`。

### Lines 2225-2225: class TestQwen3CoderDetector declaration / 类 TestQwen3CoderDetector 声明
```python
class TestQwen3CoderDetector(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 2226-2226: supporting statements / 辅助语句
```python
    """Test suite for Qwen3CoderDetector."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 2228-2285: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Initialize test fixtures before each test method."""
        self.tools = [
            Tool(
                type="function",
                function=Function(
                    name="get_current_weather",
                    parameters={
                        "type": "object",
                        "properties": {
                            "location": {"type": "string"},
                            "unit": {
                                "type": "string",
                                "enum": ["celsius", "fahrenheit"],
                            },
                            "days": {"type": "integer"},
                        },
                        "required": ["location"],
                    },
                ),
            ),
            Tool(
                type="function",
                function=Function(
                    name="sql_interpreter",
                    parameters={
                        "type": "object",
                        "properties": {
                            "query": {"type": "string"},
                            "dry_run": {"type": "boolean"},
                        },
                    },
                ),
            ),
            Tool(
                type="function",
                function=Function(
                    name="TodoWrite",
                    parameters={
                        "type": "object",
                        "properties": {
                            "todos": {
                                "type": "array",
                                "items": {
                                    "type": "object",
                                    "properties": {
                                        "content": {"type": "string"},
                                        "status": {"type": "string"},
                                    },
                                    "required": ["content", "status"],
                                },
                            },
                        },
                    },
                ),
            ),
        ]
        self.detector = Qwen3CoderDetector()
```
**EN:** Initialize test fixtures before each test method. This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Initialize test fixtures before each test method. 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 2286-2288: supporting source context / 辅助源码上下文
```python

    # ==================== Basic Functionality Tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 2289-2300: test case plain text only / 测试用例 plain text only
```python
    def test_plain_text_only(self):
        """
        Test parsing of plain text without any tool calls.

        Scenario: Input contains only plain text, no tool call markers.
        Purpose: Verify that plain text is correctly identified and no false tool calls are detected.
        """
        text = "This is plain text without any tool calls."
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(result.normal_text, text)
        self.assertEqual(len(result.calls), 0)
```
**EN:** Test parsing of plain text without any tool calls. This test exercises `test_plain_text_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing of plain text without any tool calls. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_plain_text_only`。

### Lines 2302-2324: test case single tool call / 测试用例 single tool call
```python
    def test_single_tool_call(self):
        """
        Test parsing of a single tool call.

        Scenario: Input contains one complete tool call with parameters.
        Purpose: Verify correct extraction of tool name and parameters.
        """
        text = """<tool_call>
<function=get_current_weather>
<parameter=location>Boston</parameter>
<parameter=unit>celsius</parameter>
<parameter=days>3</parameter>
</function>
</tool_call>"""
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_current_weather")

        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["location"], "Boston")
        self.assertEqual(params["unit"], "celsius")
        self.assertEqual(params["days"], 3)
```
**EN:** Test parsing of a single tool call. This test exercises `test_single_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing of a single tool call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_tool_call`。

### Lines 2326-2344: test case single tool call with text prefix / 测试用例 single tool call with text prefix
```python
    def test_single_tool_call_with_text_prefix(self):
        """
        Test parsing of tool call with preceding text.

        Scenario: Input has plain text followed by a tool call.
        Purpose: Verify correct separation of text and tool call.
        """
        text = """Let me check the weather for you.

<tool_call>
<function=get_current_weather>
<parameter=location>New York</parameter>
</function>
</tool_call>"""
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertTrue(result.normal_text.startswith("Let me check"))
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_current_weather")
```
**EN:** Test parsing of tool call with preceding text. This test exercises `test_single_tool_call_with_text_prefix` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing of tool call with preceding text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_tool_call_with_text_prefix`。

### Lines 2346-2375: test case multiple tool calls / 测试用例 multiple tool calls
```python
    def test_multiple_tool_calls(self):
        """
        Test parsing of multiple consecutive tool calls.

        Scenario: Input contains two tool calls one after another.
        Purpose: Verify that multiple tool calls are correctly identified and parsed.
        """
        text = """<tool_call>
<function=get_current_weather>
<parameter=location>New York</parameter>
</function>
</tool_call>
<tool_call>
<function=sql_interpreter>
<parameter=query>SELECT * FROM users</parameter>
<parameter=dry_run>True</parameter>
</function>
</tool_call>"""
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(len(result.calls), 2)
        self.assertEqual(result.calls[0].name, "get_current_weather")
        self.assertEqual(result.calls[1].name, "sql_interpreter")

        params1 = json.loads(result.calls[0].parameters)
        self.assertEqual(params1["location"], "New York")

        params2 = json.loads(result.calls[1].parameters)
        self.assertEqual(params2["query"], "SELECT * FROM users")
        self.assertEqual(params2["dry_run"], True)
```
**EN:** Test parsing of multiple consecutive tool calls. This test exercises `test_multiple_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing of multiple consecutive tool calls. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_tool_calls`。

### Lines 2376-2378: supporting source context / 辅助源码上下文
```python

    # ==================== Streaming Tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 2379-2415: test case streaming single tool call / 测试用例 streaming single tool call
```python
    def test_streaming_single_tool_call(self):
        """
        Test streaming parsing of a single tool call.

        Scenario: Tool call is fed incrementally in chunks.
        Purpose: Verify streaming parser correctly assembles tool call from chunks.
        """
        chunks = [
            "<tool_call>",
            "<function=get_current_weather>",
            "<parameter=location>",
            "Boston",
            "</parameter>",
            "<parameter=unit>celsius</parameter>",
            "</function>",
            "</tool_call>",
        ]

        detector = Qwen3CoderDetector()
        all_calls = []
        collected_params = ""

        for chunk in chunks:
            result = detector.parse_streaming_increment(chunk, self.tools)
            all_calls.extend(result.calls)
            for call in result.calls:
                if call.parameters:
                    collected_params += call.parameters

        # Verify we got the tool call
        self.assertGreater(len(all_calls), 0)

        # Verify parameters were collected
        if collected_params:
            params = json.loads(collected_params)
            self.assertEqual(params["location"], "Boston")
            self.assertEqual(params["unit"], "celsius")
```
**EN:** Test streaming parsing of a single tool call. This test exercises `test_streaming_single_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parsing of a single tool call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_single_tool_call`。

### Lines 2417-2445: test case streaming with text and tool / 测试用例 streaming with text and tool
```python
    def test_streaming_with_text_and_tool(self):
        """
        Test streaming parsing with mixed text and tool call.

        Scenario: Stream contains plain text followed by a tool call.
        Purpose: Verify correct separation in streaming mode.
        """
        chunks = [
            "Let me ",
            "help you.\n\n",
            "<tool_call>",
            "<function=get_current_weather>",
            "<parameter=location>Paris</parameter>",
            "</function>",
            "</tool_call>",
        ]

        detector = Qwen3CoderDetector()
        full_text = ""
        all_calls = []

        for chunk in chunks:
            result = detector.parse_streaming_increment(chunk, self.tools)
            if result.normal_text:
                full_text += result.normal_text
            all_calls.extend(result.calls)

        self.assertTrue(full_text.startswith("Let me"))
        self.assertGreater(len(all_calls), 0)
```
**EN:** Test streaming parsing with mixed text and tool call. This test exercises `test_streaming_with_text_and_tool` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parsing with mixed text and tool call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_with_text_and_tool`。

### Lines 2446-2448: supporting source context / 辅助源码上下文
```python

    # ==================== Parameter Type Tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 2449-2466: test case integer parameter conversion / 测试用例 integer parameter conversion
```python
    def test_integer_parameter_conversion(self):
        """
        Test correct type conversion for integer parameters.

        Scenario: Tool call with integer parameter.
        Purpose: Verify integer values are correctly parsed and typed.
        """
        text = """<tool_call>
<function=get_current_weather>
<parameter=location>Tokyo</parameter>
<parameter=days>5</parameter>
</function>
</tool_call>"""
        result = self.detector.detect_and_parse(text, self.tools)

        params = json.loads(result.calls[0].parameters)
        self.assertIsInstance(params["days"], int)
        self.assertEqual(params["days"], 5)
```
**EN:** Test correct type conversion for integer parameters. This test exercises `test_integer_parameter_conversion` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test correct type conversion for integer parameters. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_integer_parameter_conversion`。

### Lines 2468-2485: test case boolean parameter conversion / 测试用例 boolean parameter conversion
```python
    def test_boolean_parameter_conversion(self):
        """
        Test correct type conversion for boolean parameters.

        Scenario: Tool call with boolean parameter.
        Purpose: Verify boolean values are correctly parsed.
        """
        text = """<tool_call>
<function=sql_interpreter>
<parameter=query>SELECT 1</parameter>
<parameter=dry_run>True</parameter>
</function>
</tool_call>"""
        result = self.detector.detect_and_parse(text, self.tools)

        params = json.loads(result.calls[0].parameters)
        self.assertIsInstance(params["dry_run"], bool)
        self.assertEqual(params["dry_run"], True)
```
**EN:** Test correct type conversion for boolean parameters. This test exercises `test_boolean_parameter_conversion` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test correct type conversion for boolean parameters. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_boolean_parameter_conversion`。

### Lines 2487-2510: test case complex array parameter / 测试用例 complex array parameter
```python
    def test_complex_array_parameter(self):
        """
        Test parsing of complex array parameters.

        Scenario: Tool call with array of objects as parameter.
        Purpose: Verify complex nested structures are correctly parsed.
        """
        text = """<tool_call>
<function=TodoWrite>
<parameter=todos>
[
  {"content": "Buy groceries", "status": "pending"},
  {"content": "Finish report", "status": "completed"}
]
</parameter>
</function>
</tool_call>"""
        result = self.detector.detect_and_parse(text, self.tools)

        params = json.loads(result.calls[0].parameters)
        self.assertIsInstance(params["todos"], list)
        self.assertEqual(len(params["todos"]), 2)
        self.assertEqual(params["todos"][0]["content"], "Buy groceries")
        self.assertEqual(params["todos"][1]["status"], "completed")
```
**EN:** Test parsing of complex array parameters. This test exercises `test_complex_array_parameter` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing of complex array parameters. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_complex_array_parameter`。

### Lines 2511-2513: supporting source context / 辅助源码上下文
```python

    # ==================== Edge Cases ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 2514-2530: test case empty parameter value / 测试用例 empty parameter value
```python
    def test_empty_parameter_value(self):
        """
        Test handling of empty parameter values.

        Scenario: Tool call with empty parameter value.
        Purpose: Verify empty values are handled gracefully.
        """
        text = """<tool_call>
<function=get_current_weather>
<parameter=location></parameter>
</function>
</tool_call>"""
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(len(result.calls), 1)
        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["location"], "")
```
**EN:** Test handling of empty parameter values. This test exercises `test_empty_parameter_value` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test handling of empty parameter values. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_parameter_value`。

### Lines 2532-2548: test case parameter with special characters / 测试用例 parameter with special characters
```python
    def test_parameter_with_special_characters(self):
        """
        Test handling of parameters with special characters.

        Scenario: Parameter value contains special characters like quotes, newlines.
        Purpose: Verify special characters are correctly preserved.
        """
        text = """<tool_call>
<function=sql_interpreter>
<parameter=query>SELECT * FROM users WHERE name = 'John "Doe"'</parameter>
</function>
</tool_call>"""
        result = self.detector.detect_and_parse(text, self.tools)

        params = json.loads(result.calls[0].parameters)
        self.assertIn("John", params["query"])
        self.assertIn("Doe", params["query"])
```
**EN:** Test handling of parameters with special characters. This test exercises `test_parameter_with_special_characters` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test handling of parameters with special characters. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parameter_with_special_characters`。

### Lines 2550-2563: test case incomplete tool call / 测试用例 incomplete tool call
```python
    def test_incomplete_tool_call(self):
        """
        Test handling of incomplete tool call at end of stream.

        Scenario: Stream ends with an incomplete tool call (missing closing tag).
        Purpose: Verify detector handles incomplete input gracefully without crashing.
        """
        text = """<tool_call>
<function=get_current_weather>
<parameter=location>London"""

        # Should not crash
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertIsInstance(result, StreamingParseResult)
```
**EN:** Test handling of incomplete tool call at end of stream. This test exercises `test_incomplete_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test handling of incomplete tool call at end of stream. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_incomplete_tool_call`。

### Lines 2565-2575: test case has tool call detection / 测试用例 has tool call detection
```python
    def test_has_tool_call_detection(self):
        """
        Test the has_tool_call method for detecting tool call markers.

        Scenario: Various inputs with and without tool call markers.
        Purpose: Verify correct detection of tool call presence.
        """
        self.assertTrue(self.detector.has_tool_call("<tool_call>"))
        self.assertTrue(self.detector.has_tool_call("text <tool_call> more"))
        self.assertFalse(self.detector.has_tool_call("plain text only"))
        self.assertFalse(self.detector.has_tool_call(""))
```
**EN:** Test the has_tool_call method for detecting tool call markers. This test exercises `test_has_tool_call_detection` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test the has_tool_call method for detecting tool call markers. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_detection`。

### Lines 2576-2581: supporting source context / 辅助源码上下文
```python

    # ==================== Structural tag (xgrammar builtin) ====================
    # Qwen3 Coder uses the new builtin structural tag path. supports_structural_tag()
    # is True so required/named tool_choice routes through FunctionCallParser
    # instead of JsonArrayParser.

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 2582-2583: test case supports structural tag / 测试用例 supports structural tag
```python
    def test_supports_structural_tag(self):
        self.assertTrue(self.detector.supports_structural_tag())
```
**EN:** This test exercises `test_supports_structural_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_supports_structural_tag`。

### Lines 2585-2630: test case get model structural tag / 测试用例 get model structural tag
```python
    def test_get_model_structural_tag(self):
        import xgrammar as xgr

        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=True
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)

        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=False
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)

        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=True, tool_choice="required"
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)

        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=False, tool_choice="required"
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)

        tool_choice_name = ToolChoiceFuncName(name="get_current_weather")
        tool_choice = ToolChoice(function=tool_choice_name)
        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=True, tool_choice=tool_choice
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)

        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=False, tool_choice=tool_choice
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)
```
**EN:** This test exercises `test_get_model_structural_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_model_structural_tag`。

### Lines 2633-2633: class TestGptOssDetector declaration / 类 TestGptOssDetector 声明
```python
class TestGptOssDetector(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 2634-2670: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.tools = [
            Tool(
                type="function",
                function=Function(
                    name="search",
                    description="Searches for information.",
                    parameters={
                        "type": "object",
                        "properties": {
                            "query": {"type": "string"},
                            "topn": {"type": "integer"},
                        },
                        "required": ["query"],
                    },
                ),
            ),
            Tool(
                type="function",
                function=Function(
                    name="get_weather",
                    description="Get weather information for a city.",
                    parameters={
                        "type": "object",
                        "properties": {
                            "city": {"type": "string"},
                            "unit": {
                                "type": "string",
                                "enum": ["celsius", "fahrenheit"],
                            },
                        },
                        "required": ["city"],
                    },
                ),
            ),
        ]
        self.detector = GptOssDetector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 2672-2717: test case get model structural tag / 测试用例 get model structural tag
```python
    def test_get_model_structural_tag(self):
        import xgrammar as xgr

        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=True
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)

        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=False
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)

        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=True, tool_choice="required"
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)

        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=False, tool_choice="required"
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)

        tool_choice_name = ToolChoiceFuncName(name="search")
        tool_choice = ToolChoice(function=tool_choice_name)
        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=True, tool_choice=tool_choice
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)

        structural_tag = self.detector.get_structural_tag(
            self.tools, thinking_mode=False, tool_choice=tool_choice
        )
        self.assertIsInstance(structural_tag, xgr.StructuralTag)
        grammar = xgr.Grammar.from_structural_tag(structural_tag)
        self.assertIsInstance(grammar, xgr.Grammar)
```
**EN:** This test exercises `test_get_model_structural_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_model_structural_tag`。

### Lines 2720-2720: class TestGlm4MoeDetector declaration / 类 TestGlm4MoeDetector 声明
```python
class TestGlm4MoeDetector(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 2721-2739: setUp setup routine / setUp 初始化流程
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
                            "date": {"type": "string", "description": "Date"},
                        },
                        "required": ["city", "date"],
                    },
                ),
            ),
        ]
        self.detector = Glm4MoeDetector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 2741-2754: test case single tool call / 测试用例 single tool call
```python
    def test_single_tool_call(self):
        text = (
            "<tool_call>get_weather\n"
            "<arg_key>city</arg_key>\n<arg_value>Beijing</arg_value>\n"
            "<arg_key>date</arg_key>\n<arg_value>2024-06-27</arg_value>\n"
            "</tool_call>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(
            result.calls[0].parameters, '{"city": "Beijing", "date": "2024-06-27"}'
        )
        self.assertEqual(result.normal_text, "")
```
**EN:** This test exercises `test_single_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_tool_call`。

### Lines 2756-2777: test case multiple tool calls / 测试用例 multiple tool calls
```python
    def test_multiple_tool_calls(self):
        text = (
            "<tool_call>get_weather\n"
            "<arg_key>city</arg_key>\n<arg_value>Beijing</arg_value>\n"
            "<arg_key>date</arg_key>\n<arg_value>2024-06-27</arg_value>\n"
            "</tool_call>"
            "<tool_call>get_weather\n"
            "<arg_key>city</arg_key>\n<arg_value>Shanghai</arg_value>\n"
            "<arg_key>date</arg_key>\n<arg_value>2024-06-28</arg_value>\n"
            "</tool_call>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 2)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(
            result.calls[0].parameters, '{"city": "Beijing", "date": "2024-06-27"}'
        )
        self.assertEqual(result.calls[1].name, "get_weather")
        self.assertEqual(
            result.calls[1].parameters, '{"city": "Shanghai", "date": "2024-06-28"}'
        )
        self.assertEqual(result.normal_text, "")
```
**EN:** This test exercises `test_multiple_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_tool_calls`。

### Lines 2779-2806: test case streaming tool call / 测试用例 streaming tool call
```python
    def test_streaming_tool_call(self):
        """Test streaming incremental parsing of a tool call."""
        chunks = [
            "<tool_call>get_weather\n",
            "<arg_key>city</arg_key>\n<arg_value>Beijing</arg_value>\n",
            "<arg_key>date</arg_key>\n<arg_value>2024-06-27</arg_value>\n",
            "</tool_call>",
        ]
        tool_calls = []
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            for tool_call_chunk in result.calls:
                if (
                    hasattr(tool_call_chunk, "tool_index")
                    and tool_call_chunk.tool_index is not None
                ):
                    while len(tool_calls) <= tool_call_chunk.tool_index:
                        tool_calls.append({"name": "", "parameters": ""})
                    tc = tool_calls[tool_call_chunk.tool_index]
                    if tool_call_chunk.name:
                        tc["name"] = tool_call_chunk.name
                    if tool_call_chunk.parameters:
                        tc["parameters"] += tool_call_chunk.parameters
        self.assertEqual(len(tool_calls), 1)
        self.assertEqual(tool_calls[0]["name"], "get_weather")
        self.assertEqual(
            tool_calls[0]["parameters"], '{"city": "Beijing", "date": "2024-06-27"}'
        )
```
**EN:** Test streaming incremental parsing of a tool call. This test exercises `test_streaming_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming incremental parsing of a tool call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_tool_call`。

### Lines 2808-2842: test case streaming multiple tool calls / 测试用例 streaming multiple tool calls
```python
    def test_streaming_multiple_tool_calls(self):
        """Test streaming incremental parsing of multiple tool calls."""
        chunks = [
            "<tool_call>get_weather\n",
            "<arg_key>city</arg_key>\n<arg_value>Beijing</arg_value>\n",
            "<arg_key>date</arg_key>\n<arg_value>2024-06-27</arg_value>\n",
            "</tool_call><tool_call>get_weather\n",
            "<arg_key>city</arg_key>\n<arg_value>Shanghai</arg_value>\n",
            "<arg_key>date</arg_key>\n<arg_value>2024-06-28</arg_value>\n",
            "</tool_call>",
        ]
        tool_calls = []
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            for tool_call_chunk in result.calls:
                if (
                    hasattr(tool_call_chunk, "tool_index")
                    and tool_call_chunk.tool_index is not None
                ):
                    while len(tool_calls) <= tool_call_chunk.tool_index:
                        tool_calls.append({"name": "", "parameters": ""})
                    tc = tool_calls[tool_call_chunk.tool_index]
                    if tool_call_chunk.name:
                        tc["name"] = tool_call_chunk.name
                    if tool_call_chunk.parameters:
                        tc["parameters"] += tool_call_chunk.parameters
        self.assertEqual(len(tool_calls), 2)
        self.assertEqual(tool_calls[0]["name"], "get_weather")
        self.assertEqual(
            tool_calls[0]["parameters"], '{"city": "Beijing", "date": "2024-06-27"}'
        )
        self.assertEqual(tool_calls[1]["name"], "get_weather")
        self.assertEqual(
            tool_calls[1]["parameters"], '{"city": "Shanghai", "date": "2024-06-28"}'
        )
```
**EN:** Test streaming incremental parsing of multiple tool calls. This test exercises `test_streaming_multiple_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming incremental parsing of multiple tool calls. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_multiple_tool_calls`。

### Lines 2844-2854: test case tool call id / 测试用例 tool call id
```python
    def test_tool_call_id(self):
        """Test that the buffer and state are reset after a tool call is completed."""
        chunks = [
            "<tool_call>get_weather\n",
            "<arg_key>city</arg_key>\n<arg_value>Beijing</arg_value>\n",
            "<arg_key>date</arg_key>\n<arg_value>2024-06-27</arg_value>\n",
            "</tool_call>",
        ]
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
        self.assertEqual(self.detector.current_tool_id, 1)
```
**EN:** Test that the buffer and state are reset after a tool call is completed. This test exercises `test_tool_call_id` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that the buffer and state are reset after a tool call is completed. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_call_id`。

### Lines 2856-2860: test case invalid tool call / 测试用例 invalid tool call
```python
    def test_invalid_tool_call(self):
        """Test that invalid tool calls are handled correctly."""
        text = "<tool_call>invalid_func\n<arg_key>city</arg_key>\n<arg_value>Beijing</arg_value>\n</tool_call>"
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 0)
```
**EN:** Test that invalid tool calls are handled correctly. This test exercises `test_invalid_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that invalid tool calls are handled correctly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_invalid_tool_call`。

### Lines 2862-2890: test case partial tool call / 测试用例 partial tool call
```python
    def test_partial_tool_call(self):
        """Test parsing a partial tool call that spans multiple chunks."""
        chunks = [
            "<tool_call>get_weather\n",
            "<arg_key>city</arg_key>\n<arg_value>Beijing</arg_value>\n",
            "<arg_key>date</arg_key>\n<arg_value>2024-06-27</arg_value>\n</tool_call>",
        ]

        tool_calls = []
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            for tool_call_chunk in result.calls:
                if (
                    hasattr(tool_call_chunk, "tool_index")
                    and tool_call_chunk.tool_index is not None
                ):
                    while len(tool_calls) <= tool_call_chunk.tool_index:
                        tool_calls.append({"name": "", "parameters": ""})
                    tc = tool_calls[tool_call_chunk.tool_index]
                    if tool_call_chunk.name:
                        tc["name"] = tool_call_chunk.name
                    if tool_call_chunk.parameters:
                        tc["parameters"] += tool_call_chunk.parameters

        self.assertEqual(len(tool_calls), 1)
        self.assertEqual(tool_calls[0]["name"], "get_weather")
        self.assertEqual(
            tool_calls[0]["parameters"], '{"city": "Beijing", "date": "2024-06-27"}'
        )
```
**EN:** Test parsing a partial tool call that spans multiple chunks. This test exercises `test_partial_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing a partial tool call that spans multiple chunks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_partial_tool_call`。

### Lines 2892-2971: test case array argument with escaped json (part 1/2) / 测试用例 array argument with escaped json（第 1/2 部分）
```python
    def test_array_argument_with_escaped_json(self):
        """Test that array arguments with escaped JSON are properly handled without double-escaping."""
        # Add a tool with array parameter
        tools_with_array = [
            Tool(
                type="function",
                function=Function(
                    name="todo_write",
                    description="Write todos",
                    parameters={
                        "type": "object",
                        "properties": {
                            "todos": {
                                "type": "array",
                                "description": "The updated todo list",
                            }
                        },
                        "required": ["todos"],
                    },
                ),
            ),
        ]

        def check_params(result):
            self.assertEqual(1, len(result.calls))
            self.assertEqual("todo_write", result.calls[0].name)
            params = json.loads(result.calls[0].parameters)
            self.assertIsInstance(params["todos"], list)
            self.assertEqual(4, len(params["todos"]))
            self.assertEqual("1", params["todos"][0]["id"])
            self.assertEqual(
                "Check for hard-coded issues in the backend code",
                params["todos"][0]["task"],
            )
            self.assertEqual("in_progress", params["todos"][0]["status"])
            self.assertEqual("2", params["todos"][1]["id"])
            self.assertEqual(
                "Check for hard-coded issues in the frontend code",
                params["todos"][1]["task"],
            )
            self.assertEqual("pending", params["todos"][1]["status"])
            self.assertEqual("3", params["todos"][2]["id"])
            self.assertEqual(
                "Check for code violating the Single Responsibility Principle",
                params["todos"][2]["task"],
            )
            self.assertEqual("pending", params["todos"][2]["status"])
            self.assertEqual("4", params["todos"][3]["id"])
            self.assertEqual(
                "Generate a rectification proposal report", params["todos"][3]["task"]
            )
            self.assertEqual("pending", params["todos"][3]["status"])

        # Simulate the raw response from GLM-4.6 model with normal and escaped JSON in XML
        result = self.detector.detect_and_parse(
            """<tool_call>todo_write\n<arg_key>todos</arg_key>\n<arg_value>[{\"id\": \"1\", \"task\": \"Check for hard-coded issues in the backend code\", \"status\": \"in_progress\"}, {\"id\": \"2\", \"task\": \"Check for hard-coded issues in the frontend code\", \"status\": \"pending\"}, {\"id\": \"3\", \"task\": \"Check for code violating the Single Responsibility Principle\", \"status\": \"pending\"}, {\"id\": \"4\", \"task\": \"Generate a rectification proposal report\", \"status\": \"pending\"}]</arg_value>
</tool_call>""",
            tools_with_array,
        )
        check_params(result)
        result = self.detector.detect_and_parse(
            r"""<tool_call>todo_write\n<arg_key>todos</arg_key>\n<arg_value>[{\"id\": \"1\", \"task\": \"Check for hard-coded issues in the backend code\", \"status\": \"in_progress\"}, {\"id\": \"2\", \"task\": \"Check for hard-coded issues in the frontend code\", \"status\": \"pending\"}, {\"id\": \"3\", \"task\": \"Check for code violating the Single Responsibility Principle\", \"status\": \"pending\"}, {\"id\": \"4\", \"task\": \"Generate a rectification proposal report\", \"status\": \"pending\"}]</arg_value>
</tool_call>""",
            tools_with_array,
        )
        check_params(result)

        def check_single_todos(tool_result, expected):
            self.assertEqual(1, len(tool_result.calls))
            self.assertEqual("todo_write", tool_result.calls[0].name)
            params = json.loads(tool_result.calls[0].parameters)
            self.assertIsInstance(params["todos"], list)
            self.assertEqual(1, len(params["todos"]))
            self.assertEqual("1", params["todos"][0]["id"])
            self.assertEqual(expected, params["todos"][0]["task"])
            self.assertEqual("pending", params["todos"][0]["status"])

        # Test with escaped JSON containing backslashes in content (e.g., Windows paths)
        expected_path = r"Check file at C:\Users\test.txt"
        result = self.detector.detect_and_parse(
```
**EN:** Test that array arguments with escaped JSON are properly handled without double-escaping. This test exercises `test_array_argument_with_escaped_json` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** Test that array arguments with escaped JSON are properly handled without double-escaping. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_array_argument_with_escaped_json`。 这一段对应同一逻辑块的第 1 部分。

### Lines 2972-2993: test case array argument with escaped json (part 2/2) / 测试用例 array argument with escaped json（第 2/2 部分）
```python
            """<tool_call>todo_write\n<arg_key>todos</arg_key>\n<arg_value>[{\"id\": \"1\", \"task\": \"Check file at C:\\\\Users\\\\test.txt\", \"status\": \"pending\"}]</arg_value></tool_call>""",
            tools_with_array,
        )
        check_single_todos(result, expected_path)
        result = self.detector.detect_and_parse(
            r"""<tool_call>todo_write\n<arg_key>todos</arg_key>\n<arg_value>[{\"id\": \"1\", \"task\": \"Check file at C:\\\\Users\\\\test.txt\", \"status\": \"pending\"}]</arg_value></tool_call>""",
            tools_with_array,
        )
        check_single_todos(result, expected_path)

        # Should contain literal \n, not actual newline
        expected_output = r"Print \n to see newline"
        result = self.detector.detect_and_parse(
            """<tool_call>todo_write\n<arg_key>todos</arg_key>\n<arg_value>[{\"id\": \"1\", \"task\": \"Print \\\\n to see newline\",\"status\": \"pending\"}]</arg_value></tool_call>""",
            tools_with_array,
        )
        check_single_todos(result, expected_output)
        result = self.detector.detect_and_parse(
            r"""<tool_call>todo_write\n<arg_key>todos</arg_key>\n<arg_value>[{\"id\": \"1\", \"task\": \"Print \\\\n to see newline\",\"status\": \"pending\"}]</arg_value></tool_call>""",
            tools_with_array,
        )
        check_single_todos(result, expected_output)
```
**EN:** Test that array arguments with escaped JSON are properly handled without double-escaping. This test exercises `test_array_argument_with_escaped_json` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** Test that array arguments with escaped JSON are properly handled without double-escaping. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_array_argument_with_escaped_json`。 这一段对应同一逻辑块的第 2 部分。

### Lines 2995-3008: test case empty function name handling / 测试用例 empty function name handling
```python
    def test_empty_function_name_handling(self):
        """Test that empty function name is handled gracefully without assertion error."""
        # This test simulates the issue where the model outputs only the start token without a function name
        chunks = [
            "<tool_call>",  # Start token only, no function name yet
            "\n",  # More content without function name
        ]

        for chunk in chunks:
            # Should not raise AssertionError: func_name should not be empty
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            # Should return empty calls without error
            self.assertIsInstance(result, StreamingParseResult)
            self.assertEqual(result.calls, [])
```
**EN:** Test that empty function name is handled gracefully without assertion error. This test exercises `test_empty_function_name_handling` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that empty function name is handled gracefully without assertion error. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_function_name_handling`。

### Lines 3010-3041: test case whitespace preserved in arg values / 测试用例 whitespace preserved in arg values
```python
    def test_whitespace_preserved_in_arg_values(self):
        """Test that leading/trailing whitespace in arg values is not stripped."""
        tools_with_string = [
            Tool(
                type="function",
                function=Function(
                    name="apply_diff",
                    description="Apply a diff",
                    parameters={
                        "type": "object",
                        "properties": {
                            "old_string": {"type": "string"},
                            "new_string": {"type": "string"},
                        },
                        "required": ["old_string", "new_string"],
                    },
                ),
            )
        ]
        text = (
            "<tool_call>apply_diff\n"
            "<arg_key>old_string</arg_key>\n"
            "<arg_value>    indented code</arg_value>\n"
            "<arg_key>new_string</arg_key>\n"
            "<arg_value>        also indented</arg_value>\n"
            "</tool_call>"
        )
        result = self.detector.detect_and_parse(text, tools_with_string)
        self.assertEqual(len(result.calls), 1)
        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["old_string"], "    indented code")
        self.assertEqual(params["new_string"], "        also indented")
```
**EN:** Test that leading/trailing whitespace in arg values is not stripped. This test exercises `test_whitespace_preserved_in_arg_values` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that leading/trailing whitespace in arg values is not stripped. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_whitespace_preserved_in_arg_values`。

### Lines 3044-3044: class TestGlm47MoeDetector declaration / 类 TestGlm47MoeDetector 声明
```python
class TestGlm47MoeDetector(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 3045-3063: setUp setup routine / setUp 初始化流程
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
                            "date": {"type": "string", "description": "Date"},
                        },
                        "required": ["city", "date"],
                    },
                ),
            ),
        ]
        self.detector = Glm47MoeDetector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 3065-3078: test case single tool call / 测试用例 single tool call
```python
    def test_single_tool_call(self):
        text = (
            "<tool_call>get_weather"
            "<arg_key>city</arg_key><arg_value>Beijing</arg_value>"
            "<arg_key>date</arg_key><arg_value>2024-06-27</arg_value>"
            "</tool_call>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(
            result.calls[0].parameters, '{"city": "Beijing", "date": "2024-06-27"}'
        )
        self.assertEqual(result.normal_text, "")
```
**EN:** This test exercises `test_single_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_tool_call`。

### Lines 3080-3101: test case multiple tool calls / 测试用例 multiple tool calls
```python
    def test_multiple_tool_calls(self):
        text = (
            "<tool_call>get_weather"
            "<arg_key>city</arg_key><arg_value>Beijing</arg_value>"
            "<arg_key>date</arg_key><arg_value>2024-06-27</arg_value>"
            "</tool_call>"
            "<tool_call>get_weather"
            "<arg_key>city</arg_key><arg_value>Shanghai</arg_value>"
            "<arg_key>date</arg_key><arg_value>2024-06-28</arg_value>"
            "</tool_call>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 2)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(
            result.calls[0].parameters, '{"city": "Beijing", "date": "2024-06-27"}'
        )
        self.assertEqual(result.calls[1].name, "get_weather")
        self.assertEqual(
            result.calls[1].parameters, '{"city": "Shanghai", "date": "2024-06-28"}'
        )
        self.assertEqual(result.normal_text, "")
```
**EN:** This test exercises `test_multiple_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_tool_calls`。

### Lines 3103-3130: test case streaming tool call / 测试用例 streaming tool call
```python
    def test_streaming_tool_call(self):
        """Test streaming incremental parsing of a tool call."""
        chunks = [
            "<tool_call>get_weather",
            "<arg_key>city</arg_key><arg_value>Beijing</arg_value>",
            "<arg_key>date</arg_key><arg_value>2024-06-27</arg_value>",
            "</tool_call>",
        ]
        tool_calls = []
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            for tool_call_chunk in result.calls:
                if (
                    hasattr(tool_call_chunk, "tool_index")
                    and tool_call_chunk.tool_index is not None
                ):
                    while len(tool_calls) <= tool_call_chunk.tool_index:
                        tool_calls.append({"name": "", "parameters": ""})
                    tc = tool_calls[tool_call_chunk.tool_index]
                    if tool_call_chunk.name:
                        tc["name"] = tool_call_chunk.name
                    if tool_call_chunk.parameters:
                        tc["parameters"] += tool_call_chunk.parameters
        self.assertEqual(len(tool_calls), 1)
        self.assertEqual(tool_calls[0]["name"], "get_weather")
        self.assertEqual(
            tool_calls[0]["parameters"], '{"city": "Beijing", "date": "2024-06-27"}'
        )
```
**EN:** Test streaming incremental parsing of a tool call. This test exercises `test_streaming_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming incremental parsing of a tool call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_tool_call`。

### Lines 3132-3166: test case streaming multiple tool calls / 测试用例 streaming multiple tool calls
```python
    def test_streaming_multiple_tool_calls(self):
        """Test streaming incremental parsing of multiple tool calls."""
        chunks = [
            "<tool_call>get_weather",
            "<arg_key>city</arg_key><arg_value>Beijing</arg_value>",
            "<arg_key>date</arg_key><arg_value>2024-06-27</arg_value>",
            "</tool_call><tool_call>get_weather",
            "<arg_key>city</arg_key><arg_value>Shanghai</arg_value>",
            "<arg_key>date</arg_key><arg_value>2024-06-28</arg_value>",
            "</tool_call>",
        ]
        tool_calls = []
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            for tool_call_chunk in result.calls:
                if (
                    hasattr(tool_call_chunk, "tool_index")
                    and tool_call_chunk.tool_index is not None
                ):
                    while len(tool_calls) <= tool_call_chunk.tool_index:
                        tool_calls.append({"name": "", "parameters": ""})
                    tc = tool_calls[tool_call_chunk.tool_index]
                    if tool_call_chunk.name:
                        tc["name"] = tool_call_chunk.name
                    if tool_call_chunk.parameters:
                        tc["parameters"] += tool_call_chunk.parameters
        self.assertEqual(len(tool_calls), 2)
        self.assertEqual(tool_calls[0]["name"], "get_weather")
        self.assertEqual(
            tool_calls[0]["parameters"], '{"city": "Beijing", "date": "2024-06-27"}'
        )
        self.assertEqual(tool_calls[1]["name"], "get_weather")
        self.assertEqual(
            tool_calls[1]["parameters"], '{"city": "Shanghai", "date": "2024-06-28"}'
        )
```
**EN:** Test streaming incremental parsing of multiple tool calls. This test exercises `test_streaming_multiple_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming incremental parsing of multiple tool calls. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_multiple_tool_calls`。

### Lines 3168-3178: test case tool call id / 测试用例 tool call id
```python
    def test_tool_call_id(self):
        """Test that the buffer and state are reset after a tool call is completed."""
        chunks = [
            "<tool_call>get_weather",
            "<arg_key>city</arg_key><arg_value>Beijing</arg_value>",
            "<arg_key>date</arg_key><arg_value>2024-06-27</arg_value>",
            "</tool_call>",
        ]
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
        self.assertEqual(self.detector.current_tool_id, 1)
```
**EN:** Test that the buffer and state are reset after a tool call is completed. This test exercises `test_tool_call_id` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that the buffer and state are reset after a tool call is completed. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_call_id`。

### Lines 3180-3184: test case invalid tool call / 测试用例 invalid tool call
```python
    def test_invalid_tool_call(self):
        """Test that invalid tool calls are handled correctly."""
        text = "<tool_call>invalid_func<arg_key>city</arg_key><arg_value>Beijing</arg_value></tool_call>"
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 0)
```
**EN:** Test that invalid tool calls are handled correctly. This test exercises `test_invalid_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that invalid tool calls are handled correctly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_invalid_tool_call`。

### Lines 3186-3214: test case partial tool call / 测试用例 partial tool call
```python
    def test_partial_tool_call(self):
        """Test parsing a partial tool call that spans multiple chunks."""
        chunks = [
            "<tool_call>get_weather",
            "<arg_key>city</arg_key><arg_value>Beijing</arg_value>",
            "<arg_key>date</arg_key><arg_value>2024-06-27</arg_value></tool_call>",
        ]

        tool_calls = []
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            for tool_call_chunk in result.calls:
                if (
                    hasattr(tool_call_chunk, "tool_index")
                    and tool_call_chunk.tool_index is not None
                ):
                    while len(tool_calls) <= tool_call_chunk.tool_index:
                        tool_calls.append({"name": "", "parameters": ""})
                    tc = tool_calls[tool_call_chunk.tool_index]
                    if tool_call_chunk.name:
                        tc["name"] = tool_call_chunk.name
                    if tool_call_chunk.parameters:
                        tc["parameters"] += tool_call_chunk.parameters

        self.assertEqual(len(tool_calls), 1)
        self.assertEqual(tool_calls[0]["name"], "get_weather")
        self.assertEqual(
            tool_calls[0]["parameters"], '{"city": "Beijing", "date": "2024-06-27"}'
        )
```
**EN:** Test parsing a partial tool call that spans multiple chunks. This test exercises `test_partial_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing a partial tool call that spans multiple chunks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_partial_tool_call`。

### Lines 3216-3295: test case array argument with escaped json (part 1/2) / 测试用例 array argument with escaped json（第 1/2 部分）
```python
    def test_array_argument_with_escaped_json(self):
        """Test that array arguments with escaped JSON are properly handled without double-escaping."""
        # Add a tool with array parameter
        tools_with_array = [
            Tool(
                type="function",
                function=Function(
                    name="todo_write",
                    description="Write todos",
                    parameters={
                        "type": "object",
                        "properties": {
                            "todos": {
                                "type": "array",
                                "description": "The updated todo list",
                            }
                        },
                        "required": ["todos"],
                    },
                ),
            ),
        ]

        def check_params(result):
            self.assertEqual(1, len(result.calls))
            self.assertEqual("todo_write", result.calls[0].name)
            params = json.loads(result.calls[0].parameters)
            self.assertIsInstance(params["todos"], list)
            self.assertEqual(4, len(params["todos"]))
            self.assertEqual("1", params["todos"][0]["id"])
            self.assertEqual(
                "Check for hard-coded issues in the backend code",
                params["todos"][0]["task"],
            )
            self.assertEqual("in_progress", params["todos"][0]["status"])
            self.assertEqual("2", params["todos"][1]["id"])
            self.assertEqual(
                "Check for hard-coded issues in the frontend code",
                params["todos"][1]["task"],
            )
            self.assertEqual("pending", params["todos"][1]["status"])
            self.assertEqual("3", params["todos"][2]["id"])
            self.assertEqual(
                "Check for code violating the Single Responsibility Principle",
                params["todos"][2]["task"],
            )
            self.assertEqual("pending", params["todos"][2]["status"])
            self.assertEqual("4", params["todos"][3]["id"])
            self.assertEqual(
                "Generate a rectification proposal report", params["todos"][3]["task"]
            )
            self.assertEqual("pending", params["todos"][3]["status"])

        # Simulate the raw response from GLM-4.6 model with normal and escaped JSON in XML
        result = self.detector.detect_and_parse(
            """<tool_call>todo_write<arg_key>todos</arg_key><arg_value>[{\"id\": \"1\", \"task\": \"Check for hard-coded issues in the backend code\", \"status\": \"in_progress\"}, {\"id\": \"2\", \"task\": \"Check for hard-coded issues in the frontend code\", \"status\": \"pending\"}, {\"id\": \"3\", \"task\": \"Check for code violating the Single Responsibility Principle\", \"status\": \"pending\"}, {\"id\": \"4\", \"task\": \"Generate a rectification proposal report\", \"status\": \"pending\"}]</arg_value>
</tool_call>""",
            tools_with_array,
        )
        check_params(result)
        result = self.detector.detect_and_parse(
            r"""<tool_call>todo_write<arg_key>todos</arg_key><arg_value>[{\"id\": \"1\", \"task\": \"Check for hard-coded issues in the backend code\", \"status\": \"in_progress\"}, {\"id\": \"2\", \"task\": \"Check for hard-coded issues in the frontend code\", \"status\": \"pending\"}, {\"id\": \"3\", \"task\": \"Check for code violating the Single Responsibility Principle\", \"status\": \"pending\"}, {\"id\": \"4\", \"task\": \"Generate a rectification proposal report\", \"status\": \"pending\"}]</arg_value>
</tool_call>""",
            tools_with_array,
        )
        check_params(result)

        def check_single_todos(tool_result, expected):
            self.assertEqual(1, len(tool_result.calls))
            self.assertEqual("todo_write", tool_result.calls[0].name)
            params = json.loads(tool_result.calls[0].parameters)
            self.assertIsInstance(params["todos"], list)
            self.assertEqual(1, len(params["todos"]))
            self.assertEqual("1", params["todos"][0]["id"])
            self.assertEqual(expected, params["todos"][0]["task"])
            self.assertEqual("pending", params["todos"][0]["status"])

        # Test with escaped JSON containing backslashes in content (e.g., Windows paths)
        expected_path = r"Check file at C:\Users\test.txt"
        result = self.detector.detect_and_parse(
```
**EN:** Test that array arguments with escaped JSON are properly handled without double-escaping. This test exercises `test_array_argument_with_escaped_json` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** Test that array arguments with escaped JSON are properly handled without double-escaping. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_array_argument_with_escaped_json`。 这一段对应同一逻辑块的第 1 部分。

### Lines 3296-3317: test case array argument with escaped json (part 2/2) / 测试用例 array argument with escaped json（第 2/2 部分）
```python
            """<tool_call>todo_write<arg_key>todos</arg_key><arg_value>[{\"id\": \"1\", \"task\": \"Check file at C:\\\\Users\\\\test.txt\", \"status\": \"pending\"}]</arg_value></tool_call>""",
            tools_with_array,
        )
        check_single_todos(result, expected_path)
        result = self.detector.detect_and_parse(
            r"""<tool_call>todo_write<arg_key>todos</arg_key><arg_value>[{\"id\": \"1\", \"task\": \"Check file at C:\\\\Users\\\\test.txt\", \"status\": \"pending\"}]</arg_value></tool_call>""",
            tools_with_array,
        )
        check_single_todos(result, expected_path)

        # Should contain literal \n, not actual newline
        expected_output = r"Print \n to see newline"
        result = self.detector.detect_and_parse(
            """<tool_call>todo_write<arg_key>todos</arg_key><arg_value>[{\"id\": \"1\", \"task\": \"Print \\\\n to see newline\",\"status\": \"pending\"}]</arg_value></tool_call>""",
            tools_with_array,
        )
        check_single_todos(result, expected_output)
        result = self.detector.detect_and_parse(
            r"""<tool_call>todo_write<arg_key>todos</arg_key><arg_value>[{\"id\": \"1\", \"task\": \"Print \\\\n to see newline\",\"status\": \"pending\"}]</arg_value></tool_call>""",
            tools_with_array,
        )
        check_single_todos(result, expected_output)
```
**EN:** Test that array arguments with escaped JSON are properly handled without double-escaping. This test exercises `test_array_argument_with_escaped_json` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** Test that array arguments with escaped JSON are properly handled without double-escaping. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_array_argument_with_escaped_json`。 这一段对应同一逻辑块的第 2 部分。

### Lines 3319-3350: test case whitespace preserved in arg values / 测试用例 whitespace preserved in arg values
```python
    def test_whitespace_preserved_in_arg_values(self):
        """Test that leading/trailing whitespace in arg values is not stripped."""
        tools_with_string = [
            Tool(
                type="function",
                function=Function(
                    name="apply_diff",
                    description="Apply a diff",
                    parameters={
                        "type": "object",
                        "properties": {
                            "old_string": {"type": "string"},
                            "new_string": {"type": "string"},
                        },
                        "required": ["old_string", "new_string"],
                    },
                ),
            )
        ]
        text = (
            "<tool_call>apply_diff"
            "<arg_key>old_string</arg_key>"
            "<arg_value>    indented code</arg_value>"
            "<arg_key>new_string</arg_key>"
            "<arg_value>        also indented</arg_value>"
            "</tool_call>"
        )
        result = self.detector.detect_and_parse(text, tools_with_string)
        self.assertEqual(len(result.calls), 1)
        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["old_string"], "    indented code")
        self.assertEqual(params["new_string"], "        also indented")
```
**EN:** Test that leading/trailing whitespace in arg values is not stripped. This test exercises `test_whitespace_preserved_in_arg_values` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that leading/trailing whitespace in arg values is not stripped. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_whitespace_preserved_in_arg_values`。

### Lines 3353-3353: class TestJsonArrayParser declaration / 类 TestJsonArrayParser 声明
```python
class TestJsonArrayParser(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 3354-3395: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        # Create sample tools for testing
        self.tools = [
            Tool(
                type="function",
                function=Function(
                    name="get_weather",
                    description="Get weather information",
                    parameters={
                        "properties": {
                            "location": {
                                "type": "string",
                                "description": "Location to get weather for",
                            },
                            "unit": {
                                "type": "string",
                                "description": "Temperature unit",
                                "enum": ["celsius", "fahrenheit"],
                            },
                        },
                        "required": ["location"],
                    },
                ),
            ),
            Tool(
                type="function",
                function=Function(
                    name="search",
                    description="Search for information",
                    parameters={
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
        self.detector = JsonArrayParser()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 3397-3402: test case json detector has no ebnf / 测试用例 json detector has no ebnf
```python
    def test_json_detector_has_no_ebnf(self):
        """JsonArrayParser no longer exposes EBNF generation helpers."""
        self.assertFalse(
            hasattr(self.detector, "build_ebnf"),
            "JsonArrayParser should not expose EBNF helpers after cleanup",
        )
```
**EN:** JsonArrayParser no longer exposes EBNF generation helpers. This test exercises `test_json_detector_has_no_ebnf` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** JsonArrayParser no longer exposes EBNF generation helpers. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_json_detector_has_no_ebnf`。

### Lines 3404-3416: test case parse streaming increment malformed json / 测试用例 parse streaming increment malformed json
```python
    def test_parse_streaming_increment_malformed_json(self):
        """Test parsing with malformed JSON"""
        # Test with malformed JSON
        text = '[{"name": "get_weather", "parameters": {"location": "Tokyo"'
        result = self.detector.parse_streaming_increment(text, self.tools)

        # Should not crash and return a valid result
        self.assertIsInstance(result, StreamingParseResult)

        text = "[{}}}]"
        result = self.detector.parse_streaming_increment(text, self.tools)

        self.assertIsInstance(result, StreamingParseResult)
```
**EN:** Test parsing with malformed JSON This test exercises `test_parse_streaming_increment_malformed_json` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with malformed JSON 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_increment_malformed_json`。

### Lines 3418-3422: test case parse streaming increment empty input / 测试用例 parse streaming increment empty input
```python
    def test_parse_streaming_increment_empty_input(self):
        """Test parsing with empty input"""
        result = self.detector.parse_streaming_increment("", self.tools)
        self.assertEqual(len(result.calls), 0)
        self.assertEqual(result.normal_text, "")
```
**EN:** Test parsing with empty input This test exercises `test_parse_streaming_increment_empty_input` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with empty input 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_increment_empty_input`。

### Lines 3424-3434: test case parse streaming increment whitespace handling / 测试用例 parse streaming increment whitespace handling
```python
    def test_parse_streaming_increment_whitespace_handling(self):
        """Test parsing with various whitespace scenarios"""
        # Test with leading/trailing whitespace split across chunks
        chunk1 = '  [{"name": "get_weather", "parameters": '
        result1 = self.detector.parse_streaming_increment(chunk1, self.tools)
        self.assertIsInstance(result1, StreamingParseResult)
        chunk2 = '{"location": "Tokyo"}}]  '
        result2 = self.detector.parse_streaming_increment(chunk2, self.tools)

        # The base class should handle this
        self.assertIsInstance(result2, StreamingParseResult)
```
**EN:** Test parsing with various whitespace scenarios This test exercises `test_parse_streaming_increment_whitespace_handling` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with various whitespace scenarios 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_increment_whitespace_handling`。

### Lines 3436-3445: test case parse streaming increment nested objects / 测试用例 parse streaming increment nested objects
```python
    def test_parse_streaming_increment_nested_objects(self):
        """Test parsing with nested JSON objects"""
        chunk1 = '[{"name": "get_weather", "parameters": {"location": "Tokyo", '
        result1 = self.detector.parse_streaming_increment(chunk1, self.tools)
        self.assertIsInstance(result1, StreamingParseResult)
        chunk2 = '"nested": {"key": "value"}}}]'
        result2 = self.detector.parse_streaming_increment(chunk2, self.tools)

        # The base class should handle this
        self.assertIsInstance(result2, StreamingParseResult)
```
**EN:** Test parsing with nested JSON objects This test exercises `test_parse_streaming_increment_nested_objects` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with nested JSON objects 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_increment_nested_objects`。

### Lines 3447-3465: test case json parsing with commas / 测试用例 json parsing with commas
```python
    def test_json_parsing_with_commas(self):
        """Test that JSON parsing works correctly with comma separators"""
        # Stream two complete objects, at least 2 chunks per tool call
        chunk1 = '[{"name": "get_weather", "parameters": {"location": "Tok'
        result1 = self.detector.parse_streaming_increment(chunk1, self.tools)
        self.assertIsInstance(result1, StreamingParseResult)
        chunk2 = 'yo"}},'
        result2 = self.detector.parse_streaming_increment(chunk2, self.tools)
        self.assertIsInstance(result2, StreamingParseResult)

        chunk3 = '{"name": "get_weather", "parameters": {"location": "Par'
        result3 = self.detector.parse_streaming_increment(chunk3, self.tools)
        self.assertIsInstance(result3, StreamingParseResult)
        chunk4 = 'is"}}]'
        result4 = self.detector.parse_streaming_increment(chunk4, self.tools)
        self.assertIsInstance(result4, StreamingParseResult)
        self.assertGreater(
            len(result4.calls), 0, "Should parse tool calls from text with separators"
        )
```
**EN:** Test that JSON parsing works correctly with comma separators This test exercises `test_json_parsing_with_commas` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that JSON parsing works correctly with comma separators 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_json_parsing_with_commas`。

### Lines 3467-3494: test case braces in strings / 测试用例 braces in strings
```python
    def test_braces_in_strings(self):
        """Test that JSON with } characters inside strings works correctly"""
        # Test case: JSON array with } inside string values - streamed across chunks
        chunk1 = '[{"name": "get_weather", "parameters": {"location": "has } inside"'
        result1 = self.detector.parse_streaming_increment(chunk1, self.tools)
        self.assertIsInstance(result1, StreamingParseResult)
        chunk2 = "}}"
        result2 = self.detector.parse_streaming_increment(chunk2, self.tools)
        self.assertIsInstance(result2, StreamingParseResult)
        self.assertGreater(
            len(result2.calls), 0, "Should parse tool call with } in string"
        )

        # Test with separator (streaming in progress)
        chunk3 = '[{"name": "get_weather", "parameters": {"location": "has } inside"}'
        result3 = self.detector.parse_streaming_increment(chunk3, self.tools)
        self.assertIsInstance(result3, StreamingParseResult)
        chunk4 = "},"
        result4 = self.detector.parse_streaming_increment(chunk4, self.tools)
        self.assertIsInstance(result4, StreamingParseResult)
        chunk5 = '{"name": "get_weather"'
        result5 = self.detector.parse_streaming_increment(chunk5, self.tools)
        self.assertIsInstance(result5, StreamingParseResult)
        self.assertGreater(
            len(result5.calls),
            0,
            "Should parse tool calls with separator and } in string",
        )
```
**EN:** Test that JSON with } characters inside strings works correctly This test exercises `test_braces_in_strings` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that JSON with } characters inside strings works correctly 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_braces_in_strings`。

### Lines 3496-3509: test case separator in same chunk / 测试用例 separator in same chunk
```python
    def test_separator_in_same_chunk(self):
        """Test that separator already present in chunk works correctly"""
        # Test case: separator already in the chunk (streaming in progress) with 2+ chunks per tool call
        chunk1 = '[{"name": "get_weather", "parameters": {"location": "Tokyo"'
        result1 = self.detector.parse_streaming_increment(chunk1, self.tools)
        self.assertIsInstance(result1, StreamingParseResult)
        chunk2 = '}},{"name": "get_weather"'
        result2 = self.detector.parse_streaming_increment(chunk2, self.tools)
        self.assertIsInstance(result2, StreamingParseResult)
        self.assertGreater(
            len(result2.calls),
            0,
            "Should parse tool calls with separator in same chunk",
        )
```
**EN:** Test that separator already present in chunk works correctly This test exercises `test_separator_in_same_chunk` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that separator already present in chunk works correctly 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_separator_in_same_chunk`。

### Lines 3511-3528: test case separator in separate chunk / 测试用例 separator in separate chunk
```python
    def test_separator_in_separate_chunk(self):
        """Test that separator in separate chunk works correctly"""
        # Test case: separator in separate chunk - this tests streaming behavior
        chunk1 = '[{"name": "get_weather", "parameters": {"location": "Tokyo"}}'
        chunk2 = ","
        chunk3 = '{"name": "get_weather", "parameters": {"location": "Paris"}}'

        # Process first chunk
        result1 = self.detector.parse_streaming_increment(chunk1, self.tools)
        self.assertIsInstance(result1, StreamingParseResult)

        # Process separator chunk
        result2 = self.detector.parse_streaming_increment(chunk2, self.tools)
        self.assertIsInstance(result2, StreamingParseResult)

        # Process second chunk (streaming in progress)
        result3 = self.detector.parse_streaming_increment(chunk3, self.tools)
        self.assertIsInstance(result3, StreamingParseResult)
```
**EN:** Test that separator in separate chunk works correctly This test exercises `test_separator_in_separate_chunk` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that separator in separate chunk works correctly 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_separator_in_separate_chunk`。

### Lines 3530-3542: test case incomplete json across chunks / 测试用例 incomplete json across chunks
```python
    def test_incomplete_json_across_chunks(self):
        """Test that incomplete JSON across chunks works correctly"""
        # Test case: incomplete JSON across chunks - this tests streaming behavior
        chunk1 = '[{"name": "get_weather", "parameters": {"location": "Tokyo"'
        chunk2 = '}},{"name": "get_weather"'

        # Process first chunk (incomplete)
        result1 = self.detector.parse_streaming_increment(chunk1, self.tools)
        self.assertIsInstance(result1, StreamingParseResult)

        # Process second chunk (completes first object and starts second, streaming in progress)
        result2 = self.detector.parse_streaming_increment(chunk2, self.tools)
        self.assertIsInstance(result2, StreamingParseResult)
```
**EN:** Test that incomplete JSON across chunks works correctly This test exercises `test_incomplete_json_across_chunks` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that incomplete JSON across chunks works correctly 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_incomplete_json_across_chunks`。

### Lines 3544-3560: test case malformed json recovery / 测试用例 malformed json recovery
```python
    def test_malformed_json_recovery(self):
        """Test that malformed JSON recovers gracefully"""
        # Test with malformed JSON - should handle gracefully
        malformed_text = (
            '[{"name": "get_weather", "parameters": {"location": "unclosed string'
        )

        result1 = self.detector.parse_streaming_increment(malformed_text, self.tools)
        self.assertIsInstance(result1, StreamingParseResult)

        # Test valid JSON after malformed - streamed across 2 chunks (streaming in progress)
        valid_chunk1 = '[{"name": "get_weather", "parameters": {"location": "Tok'
        result2 = self.detector.parse_streaming_increment(valid_chunk1, self.tools)
        self.assertIsInstance(result2, StreamingParseResult)
        valid_chunk2 = 'yo"}}'
        result3 = self.detector.parse_streaming_increment(valid_chunk2, self.tools)
        self.assertIsInstance(result3, StreamingParseResult)
```
**EN:** Test that malformed JSON recovers gracefully This test exercises `test_malformed_json_recovery` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that malformed JSON recovers gracefully 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_malformed_json_recovery`。

### Lines 3562-3573: test case nested objects with commas / 测试用例 nested objects with commas
```python
    def test_nested_objects_with_commas(self):
        """Test that nested objects with commas inside work correctly"""
        # Test with nested objects that have commas - should work with json.loads()
        chunk1 = '[{"name": "get_weather", "parameters": {"location": "Tok'
        result1 = self.detector.parse_streaming_increment(chunk1, self.tools)
        self.assertIsInstance(result1, StreamingParseResult)
        chunk2 = 'yo", "unit": "celsius"}}'
        result2 = self.detector.parse_streaming_increment(chunk2, self.tools)
        self.assertIsInstance(result2, StreamingParseResult)
        self.assertGreater(
            len(result2.calls), 0, "Should parse tool call with nested objects"
        )
```
**EN:** Test that nested objects with commas inside work correctly This test exercises `test_nested_objects_with_commas` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that nested objects with commas inside work correctly 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_nested_objects_with_commas`。

### Lines 3575-3583: test case empty objects / 测试用例 empty objects
```python
    def test_empty_objects(self):
        """Test that empty objects work correctly"""
        # Test with empty objects - should work with json.loads()
        chunk1 = '[{"name": "get_weather", "parameters": '
        result1 = self.detector.parse_streaming_increment(chunk1, self.tools)
        self.assertIsInstance(result1, StreamingParseResult)
        chunk2 = "{}}"
        result2 = self.detector.parse_streaming_increment(chunk2, self.tools)
        self.assertIsInstance(result2, StreamingParseResult)
```
**EN:** Test that empty objects work correctly This test exercises `test_empty_objects` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that empty objects work correctly 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_objects`。

### Lines 3585-3593: test case whitespace handling / 测试用例 whitespace handling
```python
    def test_whitespace_handling(self):
        """Test that various whitespace scenarios work correctly"""
        # Test with various whitespace patterns - should work with json.loads()
        chunk1 = ' \n\n [{"name": "get_weather", "parameters": '
        result1 = self.detector.parse_streaming_increment(chunk1, self.tools)
        self.assertIsInstance(result1, StreamingParseResult)
        chunk2 = '{"location": "Tokyo"}}'
        result2 = self.detector.parse_streaming_increment(chunk2, self.tools)
        self.assertIsInstance(result2, StreamingParseResult)
```
**EN:** Test that various whitespace scenarios work correctly This test exercises `test_whitespace_handling` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that various whitespace scenarios work correctly 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_whitespace_handling`。

### Lines 3595-3617: test case multiple commas in chunk / 测试用例 multiple commas in chunk
```python
    def test_multiple_commas_in_chunk(self):
        """Test that multiple commas in a single chunk work correctly"""
        # Stream multiple tool calls ensuring at least 2 chunks per complete tool call
        chunk1 = '[{"name": "get_weather", "parameters": {"location": "To'
        result1 = self.detector.parse_streaming_increment(chunk1, self.tools)
        self.assertIsInstance(result1, StreamingParseResult)
        chunk2 = 'kyo"}},'
        result2 = self.detector.parse_streaming_increment(chunk2, self.tools)
        self.assertIsInstance(result2, StreamingParseResult)

        chunk3 = '{"name": "get_weather", "parameters": {"location": "Pa'
        result3 = self.detector.parse_streaming_increment(chunk3, self.tools)
        self.assertIsInstance(result3, StreamingParseResult)
        chunk4 = 'ris"}},'
        result4 = self.detector.parse_streaming_increment(chunk4, self.tools)
        self.assertIsInstance(result4, StreamingParseResult)

        chunk5 = '{"name": "get_weather"'
        result5 = self.detector.parse_streaming_increment(chunk5, self.tools)
        self.assertIsInstance(result5, StreamingParseResult)
        self.assertGreater(
            len(result5.calls), 0, "Should parse tool calls with multiple commas"
        )
```
**EN:** Test that multiple commas in a single chunk work correctly This test exercises `test_multiple_commas_in_chunk` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that multiple commas in a single chunk work correctly 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_commas_in_chunk`。

### Lines 3619-3636: test case complete tool call with trailing comma / 测试用例 complete tool call with trailing comma
```python
    def test_complete_tool_call_with_trailing_comma(self):
        """Test that complete tool call with trailing comma parses correctly"""
        # Test case: complete tool call followed by comma at end of chunk (split across 2 chunks)
        chunk1 = '[{"name": "get_weather", "parameters": {"location": "Tokyo"}'
        result1 = self.detector.parse_streaming_increment(chunk1, self.tools)
        self.assertIsInstance(result1, StreamingParseResult)
        chunk2 = "}, "
        result2 = self.detector.parse_streaming_increment(chunk2, self.tools)
        self.assertIsInstance(result2, StreamingParseResult)
        self.assertGreater(len(result2.calls), 0, "Should parse complete tool call")

        # Test that next chunk with opening brace gets the separator prepended
        next_chunk = '{"name": "get_weather", "parameters": {"location": "Paris"}}'
        result_next = self.detector.parse_streaming_increment(next_chunk, self.tools)
        self.assertIsInstance(result_next, StreamingParseResult)
        self.assertGreater(
            len(result_next.calls), 0, "Should parse subsequent tool call"
        )
```
**EN:** Test that complete tool call with trailing comma parses correctly This test exercises `test_complete_tool_call_with_trailing_comma` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that complete tool call with trailing comma parses correctly 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_complete_tool_call_with_trailing_comma`。

### Lines 3638-3665: test case three tool calls separate chunks with commas / 测试用例 three tool calls separate chunks with commas
```python
    def test_three_tool_calls_separate_chunks_with_commas(self):
        """Test parsing 3 tool calls in separate chunks with commas at the end"""
        # First tool call: 2 chunks
        chunk1_1 = '[{"name": "get_weather", "parameters": '
        result1_1 = self.detector.parse_streaming_increment(chunk1_1, self.tools)
        chunk1_2 = '{"location": "Tokyo"}},'
        result1_2 = self.detector.parse_streaming_increment(chunk1_2, self.tools)
        self.assertIsInstance(result1_2, StreamingParseResult)
        self.assertGreater(len(result1_2.calls), 0, "Should parse first tool call")

        # Second tool call: 2 chunks
        chunk2_1 = '{"name": "search", "parameters": '
        result2_1 = self.detector.parse_streaming_increment(chunk2_1, self.tools)
        chunk2_2 = '{"query": "restaurants"}},'
        result2_2 = self.detector.parse_streaming_increment(chunk2_2, self.tools)
        self.assertIsInstance(result2_2, StreamingParseResult)
        self.assertGreater(len(result2_2.calls), 0, "Should parse second tool call")

        # Third tool call: 2 chunks
        chunk3_1 = '{"name": "get_weather", "parameters": '
        result3_1 = self.detector.parse_streaming_increment(chunk3_1, self.tools)
        chunk3_2 = '{"location": "Paris"}}]'
        result3_2 = self.detector.parse_streaming_increment(chunk3_2, self.tools)
        self.assertIsInstance(result3_2, StreamingParseResult)
        self.assertGreater(len(result3_2.calls), 0, "Should parse third tool call")
        # Verify all tool calls were parsed correctly
        total_calls = len(result1_2.calls) + len(result2_2.calls) + len(result3_2.calls)
        self.assertEqual(total_calls, 3, "Should have parsed exactly 3 tool calls")
```
**EN:** Test parsing 3 tool calls in separate chunks with commas at the end This test exercises `test_three_tool_calls_separate_chunks_with_commas` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing 3 tool calls in separate chunks with commas at the end 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_three_tool_calls_separate_chunks_with_commas`。

### Lines 3668-3668: class TestLfm2Detector declaration / 类 TestLfm2Detector 声明
```python
class TestLfm2Detector(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 3669-3669: supporting statements / 辅助语句
```python
    """Tests for LFM2 (Liquid Foundation Model 2) function call detector."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3671-3731: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Set up test tools and detector."""
        self.tools = [
            Tool(
                type="function",
                function=Function(
                    name="get_weather",
                    description="Get weather information",
                    parameters={
                        "type": "object",
                        "properties": {
                            "city": {
                                "type": "string",
                                "description": "City name",
                            },
                            "unit": {
                                "type": "string",
                                "description": "Temperature unit",
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
                    description="Search for information",
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
            Tool(
                type="function",
                function=Function(
                    name="calculator",
                    description="Perform calculations",
                    parameters={
                        "type": "object",
                        "properties": {
                            "expression": {
                                "type": "string",
                                "description": "Math expression",
                            },
                        },
                        "required": ["expression"],
                    },
                ),
            ),
        ]
        self.detector = Lfm2Detector()
```
**EN:** Set up test tools and detector. This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Set up test tools and detector. 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 3732-3734: supporting source context / 辅助源码上下文
```python

    # ==================== has_tool_call tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 3735-3738: test case has tool call true / 测试用例 has tool call true
```python
    def test_has_tool_call_true(self):
        """Test detection of tool call markers."""
        text = '<|tool_call_start|>[get_weather(city="Paris")]<|tool_call_end|>'
        self.assertTrue(self.detector.has_tool_call(text))
```
**EN:** Test detection of tool call markers. This test exercises `test_has_tool_call_true` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test detection of tool call markers. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_true`。

### Lines 3740-3743: test case has tool call false / 测试用例 has tool call false
```python
    def test_has_tool_call_false(self):
        """Test no false positives for regular text."""
        text = "The weather in Paris is nice today."
        self.assertFalse(self.detector.has_tool_call(text))
```
**EN:** Test no false positives for regular text. This test exercises `test_has_tool_call_false` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test no false positives for regular text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_false`。

### Lines 3745-3748: test case has tool call partial marker / 测试用例 has tool call partial marker
```python
    def test_has_tool_call_partial_marker(self):
        """Test that partial markers are detected (start token present)."""
        text = '<|tool_call_start|>[get_weather(city="Paris")'
        self.assertTrue(self.detector.has_tool_call(text))
```
**EN:** Test that partial markers are detected (start token present). This test exercises `test_has_tool_call_partial_marker` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that partial markers are detected (start token present). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call_partial_marker`。

### Lines 3749-3751: supporting source context / 辅助源码上下文
```python

    # ==================== detect_and_parse tests (Pythonic format) ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 3752-3762: test case detect and parse pythonic simple / 测试用例 detect and parse pythonic simple
```python
    def test_detect_and_parse_pythonic_simple(self):
        """Test parsing a simple Pythonic format tool call."""
        text = '<|tool_call_start|>[get_weather(city="Paris")]<|tool_call_end|>'
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(result.calls[0].tool_index, 0)

        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["city"], "Paris")
```
**EN:** Test parsing a simple Pythonic format tool call. This test exercises `test_detect_and_parse_pythonic_simple` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing a simple Pythonic format tool call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_pythonic_simple`。

### Lines 3764-3774: test case detect and parse pythonic multiple args / 测试用例 detect and parse pythonic multiple args
```python
    def test_detect_and_parse_pythonic_multiple_args(self):
        """Test parsing with multiple arguments."""
        text = '<|tool_call_start|>[get_weather(city="London", unit="celsius")]<|tool_call_end|>'
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")

        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["city"], "London")
        self.assertEqual(params["unit"], "celsius")
```
**EN:** Test parsing with multiple arguments. This test exercises `test_detect_and_parse_pythonic_multiple_args` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with multiple arguments. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_pythonic_multiple_args`。

### Lines 3776-3793: test case detect and parse pythonic no args / 测试用例 detect and parse pythonic no args
```python
    def test_detect_and_parse_pythonic_no_args(self):
        """Test parsing function with no arguments."""
        # Add a no-arg tool for this test
        tools_with_noarg = self.tools + [
            Tool(
                type="function",
                function=Function(
                    name="get_time",
                    description="Get current time",
                    parameters={"type": "object", "properties": {}},
                ),
            ),
        ]
        text = "<|tool_call_start|>[get_time()]<|tool_call_end|>"
        result = self.detector.detect_and_parse(text, tools_with_noarg)

        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_time")
```
**EN:** Test parsing function with no arguments. This test exercises `test_detect_and_parse_pythonic_no_args` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing function with no arguments. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_pythonic_no_args`。

### Lines 3795-3807: test case detect and parse pythonic multiple calls / 测试用例 detect and parse pythonic multiple calls
```python
    def test_detect_and_parse_pythonic_multiple_calls(self):
        """Test parsing multiple tool calls in one block."""
        text = '<|tool_call_start|>[get_weather(city="Paris"), search(query="restaurants")]<|tool_call_end|>'
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(len(result.calls), 2)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(result.calls[1].name, "search")

        params1 = json.loads(result.calls[0].parameters)
        params2 = json.loads(result.calls[1].parameters)
        self.assertEqual(params1["city"], "Paris")
        self.assertEqual(params2["query"], "restaurants")
```
**EN:** Test parsing multiple tool calls in one block. This test exercises `test_detect_and_parse_pythonic_multiple_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing multiple tool calls in one block. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_pythonic_multiple_calls`。

### Lines 3809-3816: test case detect and parse with normal text before / 测试用例 detect and parse with normal text before
```python
    def test_detect_and_parse_with_normal_text_before(self):
        """Test parsing with normal text before the tool call."""
        text = 'Let me check the weather for you. <|tool_call_start|>[get_weather(city="Tokyo")]<|tool_call_end|>'
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(result.normal_text, "Let me check the weather for you.")
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
```
**EN:** Test parsing with normal text before the tool call. This test exercises `test_detect_and_parse_with_normal_text_before` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with normal text before the tool call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_with_normal_text_before`。

### Lines 3818-3827: test case detect and parse special characters in value / 测试用例 detect and parse special characters in value
```python
    def test_detect_and_parse_special_characters_in_value(self):
        """Test parsing with special characters in argument values."""
        text = (
            '<|tool_call_start|>[search(query="what\'s the weather?")]<|tool_call_end|>'
        )
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(len(result.calls), 1)
        params = json.loads(result.calls[0].parameters)
        self.assertIn("weather", params["query"])
```
**EN:** Test parsing with special characters in argument values. This test exercises `test_detect_and_parse_special_characters_in_value` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with special characters in argument values. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_special_characters_in_value`。

### Lines 3829-3835: test case detect and parse numeric values / 测试用例 detect and parse numeric values
```python
    def test_detect_and_parse_numeric_values(self):
        """Test parsing with numeric argument values."""
        text = '<|tool_call_start|>[calculator(expression="5 * 7")]<|tool_call_end|>'
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "calculator")
```
**EN:** Test parsing with numeric argument values. This test exercises `test_detect_and_parse_numeric_values` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with numeric argument values. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_numeric_values`。

### Lines 3836-3838: supporting source context / 辅助源码上下文
```python

    # ==================== detect_and_parse tests (JSON format) ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 3839-3848: test case detect and parse json simple / 测试用例 detect and parse json simple
```python
    def test_detect_and_parse_json_simple(self):
        """Test parsing JSON format tool call."""
        text = '<|tool_call_start|>[{"name": "get_weather", "arguments": {"city": "Berlin"}}]<|tool_call_end|>'
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")

        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["city"], "Berlin")
```
**EN:** Test parsing JSON format tool call. This test exercises `test_detect_and_parse_json_simple` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing JSON format tool call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_json_simple`。

### Lines 3850-3857: test case detect and parse json multiple calls / 测试用例 detect and parse json multiple calls
```python
    def test_detect_and_parse_json_multiple_calls(self):
        """Test parsing multiple JSON format tool calls."""
        text = '<|tool_call_start|>[{"name": "get_weather", "arguments": {"city": "Paris"}}, {"name": "search", "arguments": {"query": "hotels"}}]<|tool_call_end|>'
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(len(result.calls), 2)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(result.calls[1].name, "search")
```
**EN:** Test parsing multiple JSON format tool calls. This test exercises `test_detect_and_parse_json_multiple_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing multiple JSON format tool calls. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_json_multiple_calls`。

### Lines 3859-3866: test case detect and parse json with parameters key / 测试用例 detect and parse json with parameters key
```python
    def test_detect_and_parse_json_with_parameters_key(self):
        """Test parsing JSON format with 'parameters' key instead of 'arguments'."""
        text = '<|tool_call_start|>[{"name": "get_weather", "parameters": {"city": "Madrid"}}]<|tool_call_end|>'
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(len(result.calls), 1)
        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["city"], "Madrid")
```
**EN:** Test parsing JSON format with 'parameters' key instead of 'arguments'. This test exercises `test_detect_and_parse_json_with_parameters_key` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing JSON format with 'parameters' key instead of 'arguments'. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_json_with_parameters_key`。

### Lines 3867-3869: supporting source context / 辅助源码上下文
```python

    # ==================== Edge cases ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 3870-3876: test case detect and parse no tool call / 测试用例 detect and parse no tool call
```python
    def test_detect_and_parse_no_tool_call(self):
        """Test parsing text with no tool calls."""
        text = "This is just regular text without any tool calls."
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(result.normal_text, text)
        self.assertEqual(result.calls, [])
```
**EN:** Test parsing text with no tool calls. This test exercises `test_detect_and_parse_no_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing text with no tool calls. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_no_tool_call`。

### Lines 3878-3884: test case detect and parse unknown function / 测试用例 detect and parse unknown function
```python
    def test_detect_and_parse_unknown_function(self):
        """Test parsing with unknown function name - skipped by default (SGLANG_FORWARD_UNKNOWN_TOOLS=false)."""
        text = '<|tool_call_start|>[unknown_function(arg="value")]<|tool_call_end|>'
        result = self.detector.detect_and_parse(text, self.tools)

        # By default, unknown functions are skipped (consistent with other detectors)
        self.assertEqual(len(result.calls), 0)
```
**EN:** Test parsing with unknown function name - skipped by default (SGLANG_FORWARD_UNKNOWN_TOOLS=false). This test exercises `test_detect_and_parse_unknown_function` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with unknown function name - skipped by default (SGLANG_FORWARD_UNKNOWN_TOOLS=false). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_unknown_function`。

### Lines 3886-3891: test case detect and parse empty content / 测试用例 detect and parse empty content
```python
    def test_detect_and_parse_empty_content(self):
        """Test parsing with empty content between markers."""
        text = "<|tool_call_start|><|tool_call_end|>"
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(result.calls, [])
```
**EN:** Test parsing with empty content between markers. This test exercises `test_detect_and_parse_empty_content` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with empty content between markers. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_empty_content`。

### Lines 3893-3900: test case detect and parse multiple blocks / 测试用例 detect and parse multiple blocks
```python
    def test_detect_and_parse_multiple_blocks(self):
        """Test parsing multiple separate tool call blocks."""
        text = '<|tool_call_start|>[get_weather(city="Paris")]<|tool_call_end|> Some text <|tool_call_start|>[search(query="food")]<|tool_call_end|>'
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(len(result.calls), 2)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(result.calls[1].name, "search")
```
**EN:** Test parsing multiple separate tool call blocks. This test exercises `test_detect_and_parse_multiple_blocks` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing multiple separate tool call blocks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_multiple_blocks`。

### Lines 3901-3906: supporting source context / 辅助源码上下文
```python

    # ==================== Streaming tests ====================
    # The LFM2 detector buffers until it sees complete <|tool_call_start|>...<|tool_call_end|>
    # blocks, then parses the complete block. This allows proper handling of both
    # JSON and Pythonic formats.

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 3907-3913: test case streaming json complete in one chunk / 测试用例 streaming json complete in one chunk
```python
    def test_streaming_json_complete_in_one_chunk(self):
        """Test streaming with complete JSON tool call in one chunk."""
        text = '<|tool_call_start|>{"name": "get_weather", "arguments": {"city": "Rome"}}<|tool_call_end|>'
        result = self.detector.parse_streaming_increment(text, self.tools)

        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
```
**EN:** Test streaming with complete JSON tool call in one chunk. This test exercises `test_streaming_json_complete_in_one_chunk` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with complete JSON tool call in one chunk. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_json_complete_in_one_chunk`。

### Lines 3915-3934: test case streaming json split across chunks / 测试用例 streaming json split across chunks
```python
    def test_streaming_json_split_across_chunks(self):
        """Test streaming with JSON tool call split across multiple chunks - waits for complete block."""
        # Reset detector state
        self.detector = Lfm2Detector()

        # First chunk: start marker and partial JSON (no end token)
        chunk1 = '<|tool_call_start|>{"name": "get_weather", "arguments": {"city": '
        result1 = self.detector.parse_streaming_increment(chunk1, self.tools)

        # Should buffer and not emit calls yet (waiting for complete block)
        self.assertEqual(len(result1.calls), 0)
        self.assertEqual(result1.normal_text, "")

        # Second chunk: complete the JSON and end token
        chunk2 = '"Vienna"}}<|tool_call_end|>'
        result2 = self.detector.parse_streaming_increment(chunk2, self.tools)

        # Now should have the complete tool call
        self.assertEqual(len(result2.calls), 1)
        self.assertEqual(result2.calls[0].name, "get_weather")
```
**EN:** Test streaming with JSON tool call split across multiple chunks - waits for complete block. This test exercises `test_streaming_json_split_across_chunks` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with JSON tool call split across multiple chunks - waits for complete block. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_json_split_across_chunks`。

### Lines 3936-3950: test case streaming json normal text before tool call / 测试用例 streaming json normal text before tool call
```python
    def test_streaming_json_normal_text_before_tool_call(self):
        """Test streaming with normal text before JSON tool call."""
        # Reset detector state
        self.detector = Lfm2Detector()

        chunk1 = "I'll check the weather. "
        result1 = self.detector.parse_streaming_increment(chunk1, self.tools)

        # Normal text should be returned
        self.assertIn("check the weather", result1.normal_text)

        chunk2 = '<|tool_call_start|>{"name": "get_weather", "arguments": {"city": "Amsterdam"}}<|tool_call_end|>'
        result2 = self.detector.parse_streaming_increment(chunk2, self.tools)

        self.assertEqual(len(result2.calls), 1)
```
**EN:** Test streaming with normal text before JSON tool call. This test exercises `test_streaming_json_normal_text_before_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with normal text before JSON tool call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_json_normal_text_before_tool_call`。

### Lines 3952-3962: test case streaming eot token filtering / 测试用例 streaming eot token filtering
```python
    def test_streaming_eot_token_filtering(self):
        """Test that end-of-turn token is filtered from normal text."""
        # Reset detector state
        self.detector = Lfm2Detector()

        # Send text that ends with tool call end token (JSON format)
        text = '<|tool_call_start|>{"name": "get_weather", "arguments": {"city": "Oslo"}}<|tool_call_end|>'
        result = self.detector.parse_streaming_increment(text, self.tools)

        # The normal_text should not contain the eot_token
        self.assertNotIn("<|tool_call_end|>", result.normal_text)
```
**EN:** Test that end-of-turn token is filtered from normal text. This test exercises `test_streaming_eot_token_filtering` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that end-of-turn token is filtered from normal text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_eot_token_filtering`。

### Lines 3963-3965: supporting source context / 辅助源码上下文
```python

    # ==================== Pythonic streaming tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 3966-3974: test case streaming pythonic complete in one chunk / 测试用例 streaming pythonic complete in one chunk
```python
    def test_streaming_pythonic_complete_in_one_chunk(self):
        """Test streaming with complete Pythonic tool call in one chunk."""
        self.detector = Lfm2Detector()
        text = '<|tool_call_start|>[get_weather(city="Berlin")]<|tool_call_end|>'
        result = self.detector.parse_streaming_increment(text, self.tools)

        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(json.loads(result.calls[0].parameters), {"city": "Berlin"})
```
**EN:** Test streaming with complete Pythonic tool call in one chunk. This test exercises `test_streaming_pythonic_complete_in_one_chunk` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with complete Pythonic tool call in one chunk. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_pythonic_complete_in_one_chunk`。

### Lines 3976-3994: test case streaming pythonic split across chunks / 测试用例 streaming pythonic split across chunks
```python
    def test_streaming_pythonic_split_across_chunks(self):
        """Test streaming with Pythonic tool call split across multiple chunks."""
        self.detector = Lfm2Detector()

        # First chunk: start marker and partial call
        chunk1 = '<|tool_call_start|>[get_weather(city="'
        result1 = self.detector.parse_streaming_increment(chunk1, self.tools)

        # Should buffer and not emit calls yet
        self.assertEqual(len(result1.calls), 0)

        # Second chunk: complete the call
        chunk2 = 'Munich")]<|tool_call_end|>'
        result2 = self.detector.parse_streaming_increment(chunk2, self.tools)

        # Now should have the complete tool call
        self.assertEqual(len(result2.calls), 1)
        self.assertEqual(result2.calls[0].name, "get_weather")
        self.assertEqual(json.loads(result2.calls[0].parameters), {"city": "Munich"})
```
**EN:** Test streaming with Pythonic tool call split across multiple chunks. This test exercises `test_streaming_pythonic_split_across_chunks` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with Pythonic tool call split across multiple chunks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_pythonic_split_across_chunks`。

### Lines 3996-4005: test case streaming pythonic multiple calls / 测试用例 streaming pythonic multiple calls
```python
    def test_streaming_pythonic_multiple_calls(self):
        """Test streaming with multiple Pythonic tool calls."""
        self.detector = Lfm2Detector()

        text = '<|tool_call_start|>[get_weather(city="Paris"), search(query="hotels")]<|tool_call_end|>'
        result = self.detector.parse_streaming_increment(text, self.tools)

        self.assertEqual(len(result.calls), 2)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(result.calls[1].name, "search")
```
**EN:** Test streaming with multiple Pythonic tool calls. This test exercises `test_streaming_pythonic_multiple_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with multiple Pythonic tool calls. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_pythonic_multiple_calls`。

### Lines 4006-4008: supporting source context / 辅助源码上下文
```python

    # ==================== structure_info tests ====================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 4009-4013: test case supports structural tag / 测试用例 supports structural tag
```python
    def test_supports_structural_tag(self):
        """Test that LFM2 does not support structural tags (Pythonic format)."""
        # LFM2 uses Pythonic format which is not JSON-compatible,
        # so structural_tag constrained generation cannot be used
        self.assertFalse(self.detector.supports_structural_tag())
```
**EN:** Test that LFM2 does not support structural tags (Pythonic format). This test exercises `test_supports_structural_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that LFM2 does not support structural tags (Pythonic format). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_supports_structural_tag`。

### Lines 4015-4022: test case structure info / 测试用例 structure info
```python
    def test_structure_info(self):
        """Test structure info for constrained generation."""
        info_func = self.detector.structure_info()
        info = info_func("get_weather")

        self.assertEqual(info.begin, "<|tool_call_start|>[get_weather(")
        self.assertEqual(info.end, ")]<|tool_call_end|>")
        self.assertEqual(info.trigger, "<|tool_call_start|>")
```
**EN:** Test structure info for constrained generation. This test exercises `test_structure_info` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test structure info for constrained generation. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_structure_info`。

### Lines 4025-4025: class TestGigaChat3Detector declaration / 类 TestGigaChat3Detector 声明
```python
class TestGigaChat3Detector(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 4026-4075: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.tools = [
            Tool(
                type="function",
                function=Function(
                    name="manage_user_memory",
                    description="Create, update, or delete a user memory entry.",
                    parameters={
                        "type": "object",
                        "properties": {
                            "content": {
                                "anyOf": [{"type": "string"}, {"type": "null"}],
                                "default": None,
                            },
                            "action": {
                                "type": "string",
                                "enum": ["create", "update", "delete"],
                                "default": "create",
                            },
                            "id": {
                                "anyOf": [
                                    {"type": "string", "format": "uuid"},
                                    {"type": "null"},
                                ],
                                "default": None,
                            },
                        },
                    },
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
                            "unit": {
                                "type": "string",
                                "enum": ["celsius", "fahrenheit"],
                            },
                        },
                        "required": ["city"],
                    },
                ),
            ),
        ]
        self.detector = GigaChat3Detector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 4077-4081: test case has tool call / 测试用例 has tool call
```python
    def test_has_tool_call(self):
        """Test detection of tool call markers."""
        self.assertTrue(self.detector.has_tool_call("function call<|role_sep|>\n{}"))
        self.assertTrue(self.detector.has_tool_call("<|function_call|>{}"))
        self.assertFalse(self.detector.has_tool_call("No tool call here"))
```
**EN:** Test detection of tool call markers. This test exercises `test_has_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test detection of tool call markers. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call`。

### Lines 4083-4089: test case detect and parse no tool call / 测试用例 detect and parse no tool call
```python
    def test_detect_and_parse_no_tool_call(self):
        """Test parsing text without tool calls."""
        text = "How can I help you today?"
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(result.normal_text, text)
        self.assertEqual(len(result.calls), 0)
```
**EN:** Test parsing text without tool calls. This test exercises `test_detect_and_parse_no_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing text without tool calls. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_no_tool_call`。

### Lines 4091-4103: test case detect and parse simple tool call / 测试用例 detect and parse simple tool call
```python
    def test_detect_and_parse_simple_tool_call(self):
        """Test parsing a simple tool call without content."""
        text = '<|message_sep|>\n\nfunction call<|role_sep|>\n{"name": "manage_user_memory", "arguments": {"action": "create", "id": "preferences"}}'
        result = self.detector.detect_and_parse(text, self.tools)

        # No content before tool call
        self.assertEqual(result.normal_text, "")
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "manage_user_memory")

        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["action"], "create")
        self.assertEqual(params["id"], "preferences")
```
**EN:** Test parsing a simple tool call without content. This test exercises `test_detect_and_parse_simple_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing a simple tool call without content. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_simple_tool_call`。

### Lines 4105-4115: test case detect and parse parameterless tool call / 测试用例 detect and parse parameterless tool call
```python
    def test_detect_and_parse_parameterless_tool_call(self):
        """Test parsing a tool call with empty arguments."""
        text = '<|message_sep|>\n\nfunction call<|role_sep|>\n{"name": "manage_user_memory", "arguments": {}}'
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(result.normal_text, "")
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "manage_user_memory")

        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params, {})
```
**EN:** Test parsing a tool call with empty arguments. This test exercises `test_detect_and_parse_parameterless_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing a tool call with empty arguments. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_parameterless_tool_call`。

### Lines 4117-4135: test case detect and parse complex tool call / 测试用例 detect and parse complex tool call
```python
    def test_detect_and_parse_complex_tool_call(self):
        """Test parsing a tool call with nested objects."""
        text = """<|message_sep|>

function call<|role_sep|>
{"name": "manage_user_memory", "arguments": {"action": "create", "id": "preferences", "content": {"short_answers": true, "hate_emojis": true, "english_ui": false, "russian_math_explanations": true}}}"""

        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(result.normal_text, "")
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "manage_user_memory")

        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["action"], "create")
        self.assertEqual(params["id"], "preferences")
        self.assertIsInstance(params["content"], dict)
        self.assertEqual(params["content"]["short_answers"], True)
        self.assertEqual(params["content"]["hate_emojis"], True)
```
**EN:** Test parsing a tool call with nested objects. This test exercises `test_detect_and_parse_complex_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing a tool call with nested objects. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_complex_tool_call`。

### Lines 4137-4144: test case detect and parse with content before / 测试用例 detect and parse with content before
```python
    def test_detect_and_parse_with_content_before(self):
        """Test parsing tool call with text content before it."""
        text = 'I\'ll check that for you.<|message_sep|>\n\nfunction call<|role_sep|>\n{"name": "manage_user_memory", "arguments": {"action": "create", "id": "preferences"}}'
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(result.normal_text, "I'll check that for you.")
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "manage_user_memory")
```
**EN:** Test parsing tool call with text content before it. This test exercises `test_detect_and_parse_with_content_before` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing tool call with text content before it. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_with_content_before`。

### Lines 4146-4157: test case detect and parse with eos token / 测试用例 detect and parse with eos token
```python
    def test_detect_and_parse_with_eos_token(self):
        """Test parsing tool call with EOS token at the end."""
        text = '<|message_sep|>\n\nfunction call<|role_sep|>\n{"name": "manage_user_memory", "arguments": {"action": "create", "id": "preferences"}}</s>'
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(result.normal_text, "")
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "manage_user_memory")

        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["action"], "create")
        self.assertEqual(params["id"], "preferences")
```
**EN:** Test parsing tool call with EOS token at the end. This test exercises `test_detect_and_parse_with_eos_token` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing tool call with EOS token at the end. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_with_eos_token`。

### Lines 4159-4166: test case detect and parse with content and eos / 测试用例 detect and parse with content and eos
```python
    def test_detect_and_parse_with_content_and_eos(self):
        """Test parsing tool call with content and EOS token."""
        text = 'I\'ll remember that.<|message_sep|>\n\nfunction call<|role_sep|>\n{"name": "manage_user_memory", "arguments": {"action": "create", "id": "test"}}</s>'
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(result.normal_text, "I'll remember that.")
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "manage_user_memory")
```
**EN:** Test parsing tool call with content and EOS token. This test exercises `test_detect_and_parse_with_content_and_eos` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing tool call with content and EOS token. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_with_content_and_eos`。

### Lines 4168-4175: test case detect and parse invalid json / 测试用例 detect and parse invalid json
```python
    def test_detect_and_parse_invalid_json(self):
        """Test parsing with invalid JSON in function call."""
        text = '<|message_sep|>\n\nfunction call<|role_sep|>\n{"name": "manage_user_memory", "arguments": {invalid json}}'
        result = self.detector.detect_and_parse(text, self.tools)

        # Should return the full text as content when JSON parsing fails
        self.assertIn("function call", result.normal_text)
        self.assertEqual(len(result.calls), 0)
```
**EN:** Test parsing with invalid JSON in function call. This test exercises `test_detect_and_parse_invalid_json` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with invalid JSON in function call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_invalid_json`。

### Lines 4177-4183: test case detect and parse missing name / 测试用例 detect and parse missing name
```python
    def test_detect_and_parse_missing_name(self):
        """Test parsing with missing function name."""
        text = '<|message_sep|>\n\nfunction call<|role_sep|>\n{"arguments": {"action": "create"}}'
        result = self.detector.detect_and_parse(text, self.tools)

        # Should not extract tool call if name is missing
        self.assertEqual(len(result.calls), 0)
```
**EN:** Test parsing with missing function name. This test exercises `test_detect_and_parse_missing_name` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with missing function name. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_missing_name`。

### Lines 4185-4191: test case detect and parse missing arguments / 测试用例 detect and parse missing arguments
```python
    def test_detect_and_parse_missing_arguments(self):
        """Test parsing with missing arguments field."""
        text = '<|message_sep|>\n\nfunction call<|role_sep|>\n{"name": "manage_user_memory"}'
        result = self.detector.detect_and_parse(text, self.tools)

        # Should not extract tool call if arguments is missing
        self.assertEqual(len(result.calls), 0)
```
**EN:** Test parsing with missing arguments field. This test exercises `test_detect_and_parse_missing_arguments` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with missing arguments field. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_missing_arguments`。

### Lines 4193-4199: test case detect and parse arguments not dict / 测试用例 detect and parse arguments not dict
```python
    def test_detect_and_parse_arguments_not_dict(self):
        """Test parsing with arguments that is not a dict."""
        text = '<|message_sep|>\n\nfunction call<|role_sep|>\n{"name": "manage_user_memory", "arguments": "string_args"}'
        result = self.detector.detect_and_parse(text, self.tools)

        # Should not extract tool call if arguments is not a dict
        self.assertEqual(len(result.calls), 0)
```
**EN:** Test parsing with arguments that is not a dict. This test exercises `test_detect_and_parse_arguments_not_dict` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing with arguments that is not a dict. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_arguments_not_dict`。

### Lines 4201-4211: test case streaming no tool call / 测试用例 streaming no tool call
```python
    def test_streaming_no_tool_call(self):
        """Test streaming text without tool calls."""
        chunks = ["How ", "can ", "I ", "help ", "you?"]

        accumulated_text = ""
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            accumulated_text += result.normal_text

        self.assertEqual(accumulated_text, "How can I help you?")
        self.assertEqual(len(result.calls), 0)
```
**EN:** Test streaming text without tool calls. This test exercises `test_streaming_no_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming text without tool calls. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_no_tool_call`。

### Lines 4213-4248: test case streaming simple tool call / 测试用例 streaming simple tool call
```python
    def test_streaming_simple_tool_call(self):
        """Test streaming a simple tool call."""
        chunks = [
            "<|message_sep|>\n\n",
            "function call",
            "<|role_sep|>\n",
            '{"name": "manage_user_memory", ',
            '"arguments": {"action": "create"',
            ', "id": "preferences"}}',
        ]

        tool_calls_by_index = {}
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)

            for call in result.calls:
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }

                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters

        self.assertEqual(len(tool_calls_by_index), 1)
        self.assertEqual(tool_calls_by_index[0]["name"], "manage_user_memory")

        params = json.loads(tool_calls_by_index[0]["parameters"])
        self.assertEqual(params["action"], "create")
        self.assertEqual(params["id"], "preferences")
```
**EN:** Test streaming a simple tool call. This test exercises `test_streaming_simple_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming a simple tool call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_simple_tool_call`。

### Lines 4250-4290: test case streaming with content before / 测试用例 streaming with content before
```python
    def test_streaming_with_content_before(self):
        """Test streaming with content before tool call."""
        chunks = [
            "I'll ",
            "help ",
            "you.",
            "<|message_sep|>\n\n",
            "function call",
            "<|role_sep|>\n",
            '{"name": "get_weather", ',
            '"arguments": {"city": "Tokyo"}}',
        ]

        accumulated_text = ""
        tool_calls_by_index = {}

        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            accumulated_text += result.normal_text

            for call in result.calls:
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }

                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters

        self.assertEqual(accumulated_text, "I'll help you.")
        self.assertEqual(len(tool_calls_by_index), 1)
        self.assertEqual(tool_calls_by_index[0]["name"], "get_weather")

        params = json.loads(tool_calls_by_index[0]["parameters"])
        self.assertEqual(params["city"], "Tokyo")
```
**EN:** Test streaming with content before tool call. This test exercises `test_streaming_with_content_before` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with content before tool call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_with_content_before`。

### Lines 4292-4331: test case streaming complex arguments / 测试用例 streaming complex arguments
```python
    def test_streaming_complex_arguments(self):
        """Test streaming with complex nested arguments."""
        chunks = [
            "<|message_sep|>\n\n",
            "functi",
            "on call<|role_sep|>\n",
            '{"name": "manage_user_memory", "arguments": ',
            '{"action": "create", "id": "prefs", ',
            '"content": {"likes": ["short", "clear"], ',
            '"dislikes": ["emojis", "verbose"]}',
            "}}",
        ]

        tool_calls_by_index = {}

        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)

            for call in result.calls:
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }

                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters

        self.assertEqual(len(tool_calls_by_index), 1)
        self.assertEqual(tool_calls_by_index[0]["name"], "manage_user_memory")

        params = json.loads(tool_calls_by_index[0]["parameters"])
        self.assertEqual(params["action"], "create")
        self.assertEqual(params["content"]["likes"], ["short", "clear"])
        self.assertEqual(params["content"]["dislikes"], ["emojis", "verbose"])
```
**EN:** Test streaming with complex nested arguments. This test exercises `test_streaming_complex_arguments` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with complex nested arguments. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_complex_arguments`。

### Lines 4333-4368: test case streaming with eos token / 测试用例 streaming with eos token
```python
    def test_streaming_with_eos_token(self):
        """Test streaming with EOS token at the end."""
        chunks = [
            "<|message_sep|>\n\n",
            "function c",
            "all<|role_sep|>\n",
            '{"name": "get_weather", ',
            '"arguments": {"city": "Paris"}}',
            "</s>",
        ]

        tool_calls_by_index = {}

        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)

            for call in result.calls:
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }

                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters

        self.assertEqual(len(tool_calls_by_index), 1)
        self.assertEqual(tool_calls_by_index[0]["name"], "get_weather")

        params = json.loads(tool_calls_by_index[0]["parameters"])
        self.assertEqual(params["city"], "Paris")
```
**EN:** Test streaming with EOS token at the end. This test exercises `test_streaming_with_eos_token` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with EOS token at the end. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_with_eos_token`。

### Lines 4370-4404: test case streaming incomplete json / 测试用例 streaming incomplete json
```python
    def test_streaming_incomplete_json(self):
        """Test streaming with incomplete JSON (no closing brace)."""
        chunks = [
            "<|message_sep|>\n\n",
            "fun",
            "ction call<|role_sep|>\n",
            '{"name": "get_weather", ',
            '"arguments": {"city": "London"',
            # Missing closing braces
        ]

        tool_calls_by_index = {}

        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)

            for call in result.calls:
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }

                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters

        # Should have name but incomplete parameters
        self.assertEqual(len(tool_calls_by_index), 1)
        self.assertEqual(tool_calls_by_index[0]["name"], "get_weather")
        self.assertTrue(tool_calls_by_index[0]["parameters"].startswith('{"city":'))
```
**EN:** Test streaming with incomplete JSON (no closing brace). This test exercises `test_streaming_incomplete_json` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with incomplete JSON (no closing brace). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_incomplete_json`。

### Lines 4406-4446: test case streaming large steps / 测试用例 streaming large steps
```python
    def test_streaming_large_steps(self):
        """Test streaming with large chunks that complete in fewer steps."""
        chunks = [
            "I'll remember that.",
            "<|message_sep|>\n\nfuncti",
            "on call<|role_sep|>\n",
            '{"name": "manage_user_memory", "arguments": {"action": "create", "id": "preferences", "content": {"short_answers": true, "hate_emojis": true, ',
            '"english_ui": false, "russian_math_explanations": true}',
            "}}",
        ]

        accumulated_text = ""
        tool_calls_by_index = {}

        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            accumulated_text += result.normal_text

            for call in result.calls:
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }

                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters

        self.assertEqual(accumulated_text, "I'll remember that.")
        self.assertEqual(len(tool_calls_by_index), 1)
        self.assertEqual(tool_calls_by_index[0]["name"], "manage_user_memory")

        params = json.loads(tool_calls_by_index[0]["parameters"])
        self.assertEqual(params["action"], "create")
        self.assertEqual(params["content"]["short_answers"], True)
        self.assertEqual(params["content"]["russian_math_explanations"], True)
```
**EN:** Test streaming with large chunks that complete in fewer steps. This test exercises `test_streaming_large_steps` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with large chunks that complete in fewer steps. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_large_steps`。

### Lines 4448-4488: test case streaming very small chunks / 测试用例 streaming very small chunks
```python
    def test_streaming_very_small_chunks(self):
        """Test streaming with very small chunks (character by character)."""
        text = '{"name": "get_weather", "arguments": {"city": "NYC"}}'

        # Split into very small chunks (every 5 characters)
        chunk_size = 5
        chunked_text = [
            text[i : i + chunk_size] for i in range(0, len(text), chunk_size)
        ]
        chunks = [
            "<|message_sep|>\n\n",
            "func",
            "tion call",
            "<|role_sep|>\n",
            *chunked_text,
        ]
        tool_calls_by_index = {}

        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)

            for call in result.calls:
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }

                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters

        self.assertEqual(len(tool_calls_by_index), 1)
        self.assertEqual(tool_calls_by_index[0]["name"], "get_weather")

        params = json.loads(tool_calls_by_index[0]["parameters"])
        self.assertEqual(params["city"], "NYC")
```
**EN:** Test streaming with very small chunks (character by character). This test exercises `test_streaming_very_small_chunks` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with very small chunks (character by character). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_very_small_chunks`。

### Lines 4490-4528: test case streaming json split at quotes / 测试用例 streaming json split at quotes
```python
    def test_streaming_json_split_at_quotes(self):
        """Test streaming when JSON is split at quote boundaries."""
        chunks = [
            "<|message_sep|>\n\nfunction call<|role_sep|>\n",
            '{"name',
            '": "',
            "get_weather",
            '", "arguments',
            '": {"city',
            '": "',
            "Rome",
            '"}}',
        ]

        tool_calls_by_index = {}

        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)

            for call in result.calls:
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }

                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters

        self.assertEqual(len(tool_calls_by_index), 1)
        self.assertEqual(tool_calls_by_index[0]["name"], "get_weather")

        params = json.loads(tool_calls_by_index[0]["parameters"])
        self.assertEqual(params["city"], "Rome")
```
**EN:** Test streaming when JSON is split at quote boundaries. This test exercises `test_streaming_json_split_at_quotes` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming when JSON is split at quote boundaries. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_json_split_at_quotes`。

### Lines 4530-4541: test case detect and parse function call marker simple tool call / 测试用例 detect and parse function call marker simple tool call
```python
    def test_detect_and_parse_function_call_marker_simple_tool_call(self):
        """Test parsing a simple <|function_call|> tool call (GigaChat3.1-style)."""
        text = '<|function_call|>{"name": "manage_user_memory", "arguments": {"action": "create", "id": "preferences"}}'
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(result.normal_text, "")
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "manage_user_memory")

        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["action"], "create")
        self.assertEqual(params["id"], "preferences")
```
**EN:** Test parsing a simple <|function_call|> tool call (GigaChat3.1-style). This test exercises `test_detect_and_parse_function_call_marker_simple_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing a simple <|function_call|> tool call (GigaChat3.1-style). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_function_call_marker_simple_tool_call`。

### Lines 4543-4556: test case detect and parse function call marker with content before / 测试用例 detect and parse function call marker with content before
```python
    def test_detect_and_parse_function_call_marker_with_content_before(self):
        """Test parsing <|function_call|> tool call with prefix content."""
        text = (
            'I\'ll check that for you.<|function_call|>{"name": "get_weather", '
            '"arguments": {"city": "Tokyo"}}'
        )
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(result.normal_text, "I'll check that for you.")
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")

        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["city"], "Tokyo")
```
**EN:** Test parsing <|function_call|> tool call with prefix content. This test exercises `test_detect_and_parse_function_call_marker_with_content_before` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing <|function_call|> tool call with prefix content. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_function_call_marker_with_content_before`。

### Lines 4558-4569: test case detect and parse function call marker with eos token / 测试用例 detect and parse function call marker with eos token
```python
    def test_detect_and_parse_function_call_marker_with_eos_token(self):
        """Test parsing <|function_call|> tool call with EOS token at the end."""
        text = '<|function_call|>{"name": "manage_user_memory", "arguments": {"action": "create", "id": "preferences"}}</s>'
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(result.normal_text, "")
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "manage_user_memory")

        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["action"], "create")
        self.assertEqual(params["id"], "preferences")
```
**EN:** Test parsing <|function_call|> tool call with EOS token at the end. This test exercises `test_detect_and_parse_function_call_marker_with_eos_token` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing <|function_call|> tool call with EOS token at the end. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_function_call_marker_with_eos_token`。

### Lines 4571-4577: test case detect and parse function call marker invalid json / 测试用例 detect and parse function call marker invalid json
```python
    def test_detect_and_parse_function_call_marker_invalid_json(self):
        """Test parsing invalid JSON after <|function_call|> marker."""
        text = '<|function_call|>{"name": "manage_user_memory", "arguments": {invalid json}}'
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertIn("<|function_call|>", result.normal_text)
        self.assertEqual(len(result.calls), 0)
```
**EN:** Test parsing invalid JSON after <|function_call|> marker. This test exercises `test_detect_and_parse_function_call_marker_invalid_json` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing invalid JSON after <|function_call|> marker. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_function_call_marker_invalid_json`。

### Lines 4579-4616: test case streaming function call marker simple tool call / 测试用例 streaming function call marker simple tool call
```python
    def test_streaming_function_call_marker_simple_tool_call(self):
        """Test streaming parsing of the <|function_call|> marker form."""
        chunks = [
            "I'll help you.",
            "<|function_call|>",
            '{"name": "manage_user_memory", "arguments": ',
            '{"action": "create", "id": "prefs"}}',
        ]

        accumulated_text = ""
        tool_calls_by_index = {}

        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            accumulated_text += result.normal_text

            for call in result.calls:
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }

                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters

        self.assertEqual(accumulated_text, "I'll help you.")
        self.assertEqual(len(tool_calls_by_index), 1)
        self.assertEqual(tool_calls_by_index[0]["name"], "manage_user_memory")

        params = json.loads(tool_calls_by_index[0]["parameters"])
        self.assertEqual(params["action"], "create")
        self.assertEqual(params["id"], "prefs")
```
**EN:** Test streaming parsing of the <|function_call|> marker form. This test exercises `test_streaming_function_call_marker_simple_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming parsing of the <|function_call|> marker form. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_function_call_marker_simple_tool_call`。

### Lines 4618-4656: test case streaming function call marker json split at quotes / 测试用例 streaming function call marker json split at quotes
```python
    def test_streaming_function_call_marker_json_split_at_quotes(self):
        """Test streaming when JSON is split at quote boundaries (<|function_call|>)."""
        chunks = [
            "<|function_call|>",
            '{"name',
            '": "',
            "get_weather",
            '", "arguments',
            '": {"city',
            '": "',
            "Rome",
            '"}}',
        ]

        tool_calls_by_index = {}

        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)

            for call in result.calls:
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }

                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters

        self.assertEqual(len(tool_calls_by_index), 1)
        self.assertEqual(tool_calls_by_index[0]["name"], "get_weather")

        params = json.loads(tool_calls_by_index[0]["parameters"])
        self.assertEqual(params["city"], "Rome")
```
**EN:** Test streaming when JSON is split at quote boundaries (<|function_call|>). This test exercises `test_streaming_function_call_marker_json_split_at_quotes` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming when JSON is split at quote boundaries (<|function_call|>). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_function_call_marker_json_split_at_quotes`。

### Lines 4659-4659: class TestGetStructureConstraint declaration / 类 TestGetStructureConstraint 声明
```python
class TestGetStructureConstraint(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 4660-4664: supporting statements / 辅助语句
```python
    """Tests for FunctionCallParser.get_structure_constraint() logic.

    Verifies that detectors supporting structural_tag use it for required/named
    tool_choice, and that the generic json_schema fallback is used otherwise.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 4666-4681: method make tools / 方法 make tools
```python
    def _make_tools(self, strict=False):
        return [
            Tool(
                type="function",
                function=Function(
                    name="get_weather",
                    description="Get weather",
                    parameters={
                        "type": "object",
                        "properties": {"city": {"type": "string"}},
                        "required": ["city"],
                    },
                    strict=strict,
                ),
            ),
        ]
```
**EN:** This block implements `_make_tools` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_tools`，承担模块行为中的一个聚焦逻辑片段。

### Lines 4683-4686: method make parser / 方法 make parser
```python
    def _make_parser(self, parser_name, strict=False):
        from sglang.srt.function_call.function_call_parser import FunctionCallParser

        return FunctionCallParser(self._make_tools(strict=strict), parser_name)
```
**EN:** This block implements `_make_parser` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_parser`，承担模块行为中的一个聚焦逻辑片段。

### Lines 4687-4689: supporting source context / 辅助源码上下文
```python

    # --- structural_tag detectors (kimi_k2, deepseekv3, qwen25, etc.) ---

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 4690-4695: test case kimi required strict returns structural tag / 测试用例 kimi required strict returns structural tag
```python
    def test_kimi_required_strict_returns_structural_tag(self):
        parser = self._make_parser("kimi_k2", strict=True)
        result = parser.get_structure_constraint("required")
        self.assertIsNotNone(result)
        self.assertEqual(result[0], "structural_tag")
        self.assertTrue(result[1].at_least_one)
```
**EN:** This test exercises `test_kimi_required_strict_returns_structural_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_kimi_required_strict_returns_structural_tag`。

### Lines 4697-4703: test case kimi required no strict returns structural tag / 测试用例 kimi required no strict returns structural tag
```python
    def test_kimi_required_no_strict_returns_structural_tag(self):
        """required should use structural_tag even without strict, to preserve native format."""
        parser = self._make_parser("kimi_k2", strict=False)
        result = parser.get_structure_constraint("required")
        self.assertIsNotNone(result)
        self.assertEqual(result[0], "structural_tag")
        self.assertTrue(result[1].at_least_one)
```
**EN:** required should use structural_tag even without strict, to preserve native format. This test exercises `test_kimi_required_no_strict_returns_structural_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** required should use structural_tag even without strict, to preserve native format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_kimi_required_no_strict_returns_structural_tag`。

### Lines 4705-4710: test case kimi auto strict returns structural tag / 测试用例 kimi auto strict returns structural tag
```python
    def test_kimi_auto_strict_returns_structural_tag(self):
        parser = self._make_parser("kimi_k2", strict=True)
        result = parser.get_structure_constraint("auto")
        self.assertIsNotNone(result)
        self.assertEqual(result[0], "structural_tag")
        self.assertFalse(result[1].at_least_one)
```
**EN:** This test exercises `test_kimi_auto_strict_returns_structural_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_kimi_auto_strict_returns_structural_tag`。

### Lines 4712-4727: test case kimi routes through legacy with section markers / 测试用例 kimi routes through legacy with section markers
```python
    def test_kimi_routes_through_legacy_with_section_markers(self):
        """xgrammar 0.2.0's get_kimi_structural_tag(tool_choice='auto') emits
        a bare <|tool_call_begin|>...<|tool_call_end|> grammar without the
        section wrapper Kimi's chat template uses, so the parser would drop
        any generated tool calls. KimiK2Detector therefore stays on the
        legacy path; pin that here so a future tweak doesn't silently
        re-route Kimi through the broken builtin."""
        from sglang.srt.entrypoints.openai.protocol import (
            LegacyStructuralTagResponseFormat,
        )

        parser = self._make_parser("kimi_k2", strict=True)
        result = parser.get_structure_constraint("auto")
        self.assertIsInstance(result[1], LegacyStructuralTagResponseFormat)
        self.assertIn("<|tool_calls_section_begin|>", result[1].structures[0].begin)
        self.assertIn("<|tool_calls_section_end|>", result[1].structures[0].end)
```
**EN:** xgrammar 0.2.0's get_kimi_structural_tag(tool_choice='auto') emits a bare <|tool_call_begin|>...<|tool_call_end|> grammar without the section wrapper Kimi's chat template uses, so the parser would drop any generated tool calls. This test exercises `test_kimi_routes_through_legacy_with_section_markers` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** xgrammar 0.2.0's get_kimi_structural_tag(tool_choice='auto') emits a bare <|tool_call_begin|>...<|tool_call_end|> grammar without the section wrapper Kimi's chat template uses, so the parser would drop any generated tool calls. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_kimi_routes_through_legacy_with_section_markers`。

### Lines 4729-4733: test case kimi auto no strict returns none / 测试用例 kimi auto no strict returns none
```python
    def test_kimi_auto_no_strict_returns_none(self):
        """auto without strict should not constrain."""
        parser = self._make_parser("kimi_k2", strict=False)
        result = parser.get_structure_constraint("auto")
        self.assertIsNone(result)
```
**EN:** auto without strict should not constrain. This test exercises `test_kimi_auto_no_strict_returns_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** auto without strict should not constrain. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_kimi_auto_no_strict_returns_none`。

### Lines 4735-4745: test case kimi named tool choice returns structural tag / 测试用例 kimi named tool choice returns structural tag
```python
    def test_kimi_named_tool_choice_returns_structural_tag(self):
        from sglang.srt.entrypoints.openai.protocol import (
            ToolChoice,
            ToolChoiceFuncName,
        )

        parser = self._make_parser("kimi_k2", strict=False)
        tool_choice = ToolChoice(function=ToolChoiceFuncName(name="get_weather"))
        result = parser.get_structure_constraint(tool_choice)
        self.assertIsNotNone(result)
        self.assertEqual(result[0], "structural_tag")
```
**EN:** This test exercises `test_kimi_named_tool_choice_returns_structural_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_kimi_named_tool_choice_returns_structural_tag`。

### Lines 4747-4751: test case deepseekv3 required no strict returns structural tag / 测试用例 deepseekv3 required no strict returns structural tag
```python
    def test_deepseekv3_required_no_strict_returns_structural_tag(self):
        parser = self._make_parser("deepseekv3", strict=False)
        result = parser.get_structure_constraint("required")
        self.assertIsNotNone(result)
        self.assertEqual(result[0], "structural_tag")
```
**EN:** This test exercises `test_deepseekv3_required_no_strict_returns_structural_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deepseekv3_required_no_strict_returns_structural_tag`。

### Lines 4753-4757: test case qwen25 required no strict returns structural tag / 测试用例 qwen25 required no strict returns structural tag
```python
    def test_qwen25_required_no_strict_returns_structural_tag(self):
        parser = self._make_parser("qwen25", strict=False)
        result = parser.get_structure_constraint("required")
        self.assertIsNotNone(result)
        self.assertEqual(result[0], "structural_tag")
```
**EN:** This test exercises `test_qwen25_required_no_strict_returns_structural_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_qwen25_required_no_strict_returns_structural_tag`。

### Lines 4758-4760: supporting source context / 辅助源码上下文
```python

    # --- structural_tag content verification ---

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 4761-4768: test case kimi structural tag has kimi tokens / 测试用例 kimi structural tag has kimi tokens
```python
    def test_kimi_structural_tag_has_kimi_tokens(self):
        """Verify structural_tag contains kimi-specific special tokens."""
        parser = self._make_parser("kimi_k2", strict=True)
        result = parser.get_structure_constraint("required")
        tag = result[1]
        self.assertTrue(len(tag.structures) > 0)
        self.assertIn("<|tool_calls_section_begin|>", tag.structures[0].begin)
        self.assertIn("<|tool_call_end|>", tag.structures[0].end)
```
**EN:** Verify structural_tag contains kimi-specific special tokens. This test exercises `test_kimi_structural_tag_has_kimi_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Verify structural_tag contains kimi-specific special tokens. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_kimi_structural_tag_has_kimi_tokens`。

### Lines 4770-4775: test case kimi required no strict uses empty schema / 测试用例 kimi required no strict uses empty schema
```python
    def test_kimi_required_no_strict_uses_empty_schema(self):
        """Without strict, structural_tag should use empty schema per OpenAI
        protocol: strict=False means no parameter schema enforcement."""
        parser = self._make_parser("kimi_k2", strict=False)
        result = parser.get_structure_constraint("required")
        self.assertEqual(result[1].structures[0].schema_, {})
```
**EN:** Without strict, structural_tag should use empty schema per OpenAI protocol: strict=False means no parameter schema enforcement. This test exercises `test_kimi_required_no_strict_uses_empty_schema` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Without strict, structural_tag should use empty schema per OpenAI protocol: strict=False means no parameter schema enforcement. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_kimi_required_no_strict_uses_empty_schema`。

### Lines 4777-4783: test case kimi required strict uses tool schema / 测试用例 kimi required strict uses tool schema
```python
    def test_kimi_required_strict_uses_tool_schema(self):
        """With strict, structural_tag should include the tool's parameter schema."""
        parser = self._make_parser("kimi_k2", strict=True)
        result = parser.get_structure_constraint("required")
        schema = result[1].structures[0].schema_
        self.assertIn("properties", schema)
        self.assertIn("city", schema["properties"])
```
**EN:** With strict, structural_tag should include the tool's parameter schema. This test exercises `test_kimi_required_strict_uses_tool_schema` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** With strict, structural_tag should include the tool's parameter schema. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_kimi_required_strict_uses_tool_schema`。

### Lines 4784-4786: supporting source context / 辅助源码上下文
```python

    # --- reasoning-prefix ownership ---

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 4787-4796: test case default thinking mode is false / 测试用例 default thinking mode is false
```python
    def test_default_thinking_mode_is_false(self):
        """Default must be False so callers don't silently get a reasoning
        prefix added to their grammar (only relevant for detectors routed
        through the xgrammar builtin)."""
        import inspect

        from sglang.srt.function_call.function_call_parser import FunctionCallParser

        sig = inspect.signature(FunctionCallParser.get_structure_constraint)
        self.assertIs(sig.parameters["thinking_mode"].default, False)
```
**EN:** Default must be False so callers don't silently get a reasoning prefix added to their grammar (only relevant for detectors routed through the xgrammar builtin). This test exercises `test_default_thinking_mode_is_false` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Default must be False so callers don't silently get a reasoning prefix added to their grammar (only relevant for detectors routed through the xgrammar builtin). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_thinking_mode_is_false`。

### Lines 4799-4799: class TestQwen25Detector declaration / 类 TestQwen25Detector 声明
```python
class TestQwen25Detector(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 4800-4800: supporting statements / 辅助语句
```python
    """Test Qwen25Detector streaming and non-streaming multi-tool-call parsing."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 4802-4832: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        from sglang.srt.function_call.qwen25_detector import Qwen25Detector

        self.detector = Qwen25Detector()
        self.tools = [
            Tool(
                type="function",
                function=Function(
                    name="get_current_weather",
                    description="Get the current weather in a given location",
                    parameters={
                        "type": "object",
                        "properties": {
                            "city": {
                                "type": "string",
                                "description": "The city name",
                            },
                            "state": {
                                "type": "string",
                                "description": "Two-letter state abbreviation",
                            },
                            "unit": {
                                "type": "string",
                                "enum": ["celsius", "fahrenheit"],
                            },
                        },
                        "required": ["city", "state", "unit"],
                    },
                ),
            ),
        ]
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 4833-4835: supporting source context / 辅助源码上下文
```python

    # -- Non-streaming tests --

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 4836-4842: test case detect and parse single tool call / 测试用例 detect and parse single tool call
```python
    def test_detect_and_parse_single_tool_call(self):
        text = '<tool_call>\n{"name": "get_current_weather", "arguments": {"city": "NYC", "state": "NY", "unit": "fahrenheit"}}\n</tool_call>'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_current_weather")
        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["city"], "NYC")
```
**EN:** This test exercises `test_detect_and_parse_single_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_single_tool_call`。

### Lines 4844-4854: test case detect and parse multiple tool calls / 测试用例 detect and parse multiple tool calls
```python
    def test_detect_and_parse_multiple_tool_calls(self):
        text = (
            '<tool_call>\n{"name": "get_current_weather", "arguments": {"city": "NYC", "state": "NY", "unit": "fahrenheit"}}\n</tool_call>\n'
            '<tool_call>\n{"name": "get_current_weather", "arguments": {"city": "Baltimore", "state": "MD", "unit": "fahrenheit"}}\n</tool_call>\n'
            '<tool_call>\n{"name": "get_current_weather", "arguments": {"city": "Minneapolis", "state": "MN", "unit": "fahrenheit"}}\n</tool_call>\n'
            '<tool_call>\n{"name": "get_current_weather", "arguments": {"city": "Los Angeles", "state": "CA", "unit": "fahrenheit"}}\n</tool_call>'
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 4)
        cities = [json.loads(c.parameters)["city"] for c in result.calls]
        self.assertEqual(cities, ["NYC", "Baltimore", "Minneapolis", "Los Angeles"])
```
**EN:** This test exercises `test_detect_and_parse_multiple_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_multiple_tool_calls`。

### Lines 4856-4863: test case detect and parse with normal text prefix / 测试用例 detect and parse with normal text prefix
```python
    def test_detect_and_parse_with_normal_text_prefix(self):
        text = (
            "Sure, let me check the weather.\n"
            '<tool_call>\n{"name": "get_current_weather", "arguments": {"city": "NYC", "state": "NY", "unit": "celsius"}}\n</tool_call>'
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertIn("let me check", result.normal_text)
```
**EN:** This test exercises `test_detect_and_parse_with_normal_text_prefix` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_with_normal_text_prefix`。

### Lines 4864-4866: supporting source context / 辅助源码上下文
```python

    # -- Streaming tests --

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 4867-4885: method collect streaming tool calls / 方法 collect streaming tool calls
```python
    def _collect_streaming_tool_calls(self, chunks):
        """Helper: feed chunks through streaming parser and collect tool calls by index."""
        tool_calls_by_index = {}
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            for call in result.calls:
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }
                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters
        return tool_calls_by_index
```
**EN:** Helper: feed chunks through streaming parser and collect tool calls by index. This block implements `_collect_streaming_tool_calls` and captures one focused piece of the module's behavior.
**CN:** Helper: feed chunks through streaming parser and collect tool calls by index. 该代码块实现 `_collect_streaming_tool_calls`，承担模块行为中的一个聚焦逻辑片段。

### Lines 4887-4900: test case streaming single tool call / 测试用例 streaming single tool call
```python
    def test_streaming_single_tool_call(self):
        chunks = [
            "<tool_call>\n",
            '{"name": "get_current_weather",',
            ' "arguments": {"city": "NYC",',
            ' "state": "NY",',
            ' "unit": "fahrenheit"}}',
            "\n</tool_call>",
        ]
        result = self._collect_streaming_tool_calls(chunks)
        self.assertEqual(len(result), 1)
        self.assertEqual(result[0]["name"], "get_current_weather")
        params = json.loads(result[0]["parameters"])
        self.assertEqual(params["city"], "NYC")
```
**EN:** This test exercises `test_streaming_single_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_single_tool_call`。

### Lines 4902-4921: test case streaming multiple tool calls / 测试用例 streaming multiple tool calls
```python
    def test_streaming_multiple_tool_calls(self):
        """Core regression test: multiple tool calls must all be parsed in streaming mode."""
        chunks = [
            "<tool_call>\n",
            '{"name": "get_current_weather",',
            ' "arguments": {"city": "NYC", "state": "NY", "unit": "fahrenheit"}}',
            "\n</tool_call>\n",
            "<tool_call>\n",
            '{"name": "get_current_weather",',
            ' "arguments": {"city": "Baltimore", "state": "MD", "unit": "fahrenheit"}}',
            "\n</tool_call>\n",
            "<tool_call>\n",
            '{"name": "get_current_weather",',
            ' "arguments": {"city": "LA", "state": "CA", "unit": "fahrenheit"}}',
            "\n</tool_call>",
        ]
        result = self._collect_streaming_tool_calls(chunks)
        self.assertEqual(len(result), 3, f"Expected 3 tool calls, got {len(result)}")
        cities = [json.loads(result[i]["parameters"])["city"] for i in sorted(result)]
        self.assertEqual(cities, ["NYC", "Baltimore", "LA"])
```
**EN:** Core regression test: multiple tool calls must all be parsed in streaming mode. This test exercises `test_streaming_multiple_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Core regression test: multiple tool calls must all be parsed in streaming mode. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_multiple_tool_calls`。

### Lines 4923-4934: test case streaming multiple tool calls fused chunks / 测试用例 streaming multiple tool calls fused chunks
```python
    def test_streaming_multiple_tool_calls_fused_chunks(self):
        """Test when separator and next bot_token arrive in a single chunk."""
        chunks = [
            '<tool_call>\n{"name": "get_current_weather", "arguments": {"city": "NYC", "state": "NY", "unit": "fahrenheit"}}',
            '\n</tool_call>\n<tool_call>\n{"name": "get_current_weather",',
            ' "arguments": {"city": "LA", "state": "CA", "unit": "fahrenheit"}}',
            "\n</tool_call>",
        ]
        result = self._collect_streaming_tool_calls(chunks)
        self.assertEqual(len(result), 2, f"Expected 2 tool calls, got {len(result)}")
        cities = [json.loads(result[i]["parameters"])["city"] for i in sorted(result)]
        self.assertEqual(cities, ["NYC", "LA"])
```
**EN:** Test when separator and next bot_token arrive in a single chunk. This test exercises `test_streaming_multiple_tool_calls_fused_chunks` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test when separator and next bot_token arrive in a single chunk. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_multiple_tool_calls_fused_chunks`。

### Lines 4936-4951: test case streaming multiple tool calls char by char separator / 测试用例 streaming multiple tool calls char by char separator
```python
    def test_streaming_multiple_tool_calls_char_by_char_separator(self):
        """Test when the separator between tool calls arrives character by character."""
        call1 = '{"name": "get_current_weather", "arguments": {"city": "NYC", "state": "NY", "unit": "fahrenheit"}}'
        call2 = '{"name": "get_current_weather", "arguments": {"city": "LA", "state": "CA", "unit": "celsius"}}'
        separator = "\n</tool_call>\n<tool_call>\n"

        chunks = ["<tool_call>\n", call1]
        for ch in separator:
            chunks.append(ch)
        chunks.append(call2)
        chunks.append("\n</tool_call>")

        result = self._collect_streaming_tool_calls(chunks)
        self.assertEqual(len(result), 2, f"Expected 2 tool calls, got {len(result)}")
        cities = [json.loads(result[i]["parameters"])["city"] for i in sorted(result)]
        self.assertEqual(cities, ["NYC", "LA"])
```
**EN:** Test when the separator between tool calls arrives character by character. This test exercises `test_streaming_multiple_tool_calls_char_by_char_separator` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test when the separator between tool calls arrives character by character. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_multiple_tool_calls_char_by_char_separator`。

### Lines 4954-4954: class TestGemma4Detector declaration / 类 TestGemma4Detector 声明
```python
class TestGemma4Detector(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 4955-4976: setUp setup routine / setUp 初始化流程
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
                            "location": {"type": "string"},
                            "unit": {
                                "type": "string",
                                "enum": ["celsius", "fahrenheit"],
                            },
                        },
                        "required": ["location"],
                    },
                ),
            )
        ]
        self.detector = Gemma4Detector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 4978-4987: test case detect and parse / 测试用例 detect and parse
```python
    def test_detect_and_parse(self):
        text = 'Some text before <|tool_call>call:get_weather{location:<|"|>Tokyo<|"|>}<tool_call|>'
        result = self.detector.detect_and_parse(text, self.tools)

        self.assertEqual(result.normal_text, "Some text before ")
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")

        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["location"], "Tokyo")
```
**EN:** This test exercises `test_detect_and_parse` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse`。

### Lines 4989-5019: test case parse streaming increment / 测试用例 parse streaming increment
```python
    def test_parse_streaming_increment(self):
        chunks = [
            "Some text ",
            "before <|tool",
            "_call>call:get_we",
            "ather{location:<|",  # codespell:ignore
            '"|>Tokyo<|"|>}<tool_',
            "call|> after",
        ]

        all_results = []
        for chunk in chunks:
            res = self.detector.parse_streaming_increment(chunk, self.tools)
            all_results.append(res)

        combined_normal_text = "".join(r.normal_text for r in all_results)
        self.assertEqual(combined_normal_text, "Some text before  after")

        found_name = False
        found_params = False
        for res in all_results:
            for call in res.calls:
                if call.name == "get_weather":
                    found_name = True
                if call.parameters:
                    params = json.loads(call.parameters)
                    if params == {"location": "Tokyo"}:
                        found_params = True

        self.assertTrue(found_name)
        self.assertTrue(found_params)
```
**EN:** This test exercises `test_parse_streaming_increment` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_increment`。

### Lines 5021-5047: test case nested array streaming / 测试用例 nested array streaming
```python
    def test_nested_array_streaming(self):
        # Additional coverage for complex structure
        chunks = [
            '<|tool_call>call:get_weather{location:<|"',
            '|>New York<|"|>,nested:[1, 2, {inner:<|"|>',
            'val<|"|>}]}<tool_call|>',
        ]

        all_results = []
        for chunk in chunks:
            res = self.detector.parse_streaming_increment(chunk, self.tools)
            all_results.append(res)

        found_params = False
        for res in all_results:
            for call in res.calls:
                if call.parameters:
                    params = json.loads(call.parameters)
                    if "location" in params and params["location"] == "New York":
                        if "nested" in params and params["nested"] == [
                            1,
                            2,
                            {"inner": "val"},
                        ]:
                            found_params = True

        self.assertTrue(found_params)
```
**EN:** This test exercises `test_nested_array_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_nested_array_streaming`。

### Lines 5049-5055: test case has tool call / 测试用例 has tool call
```python
    def test_has_tool_call(self):
        self.assertTrue(
            self.detector.has_tool_call(
                '<|tool_call>call:get_weather{location:<|"|>Tokyo<|"|>}<tool_call|>'
            )
        )
        self.assertFalse(self.detector.has_tool_call("no tool call here"))
```
**EN:** This test exercises `test_has_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call`。

### Lines 5057-5061: test case detect and parse no tool call / 测试用例 detect and parse no tool call
```python
    def test_detect_and_parse_no_tool_call(self):
        text = "This is plain text without any tool calls."
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(result.normal_text, text)
        self.assertEqual(len(result.calls), 0)
```
**EN:** This test exercises `test_detect_and_parse_no_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_no_tool_call`。

### Lines 5063-5068: test case detect and parse tool index / 测试用例 detect and parse tool index
```python
    def test_detect_and_parse_tool_index(self):
        text = '<|tool_call>call:get_weather{location:<|"|>Tokyo<|"|>}<tool_call|>'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].tool_index, 0)
        self.assertEqual(result.calls[0].name, "get_weather")
```
**EN:** This test exercises `test_detect_and_parse_tool_index` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_tool_index`。

### Lines 5070-5074: test case detect and parse unknown tool index / 测试用例 detect and parse unknown tool index
```python
    def test_detect_and_parse_unknown_tool_index(self):
        text = '<|tool_call>call:unknown_func{arg:<|"|>val<|"|>}<tool_call|>'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].tool_index, -1)
```
**EN:** This test exercises `test_detect_and_parse_unknown_tool_index` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_unknown_tool_index`。

### Lines 5076-5084: test case detect and parse nested object / 测试用例 detect and parse nested object
```python
    def test_detect_and_parse_nested_object(self):
        text = '<|tool_call>call:get_weather{location:<|"|>Tokyo<|"|>,details:{temp:25,unit:<|"|>celsius<|"|>}}<tool_call|>'
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        params = json.loads(result.calls[0].parameters)
        self.assertEqual(params["location"], "Tokyo")
        self.assertIsInstance(params["details"], dict)
        self.assertEqual(params["details"]["temp"], 25)
        self.assertEqual(params["details"]["unit"], "celsius")
```
**EN:** This test exercises `test_detect_and_parse_nested_object` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_nested_object`。

### Lines 5086-5108: test case detect and parse multiple calls / 测试用例 detect and parse multiple calls
```python
    def test_detect_and_parse_multiple_calls(self):
        extra_tools = self.tools + [
            Tool(
                type="function",
                function=Function(
                    name="get_time",
                    description="Get current time",
                    parameters={
                        "type": "object",
                        "properties": {"timezone": {"type": "string"}},
                    },
                ),
            )
        ]
        text = (
            'Some text <|tool_call>call:get_weather{location:<|"|>Tokyo<|"|>}<tool_call|>'
            ' more text <|tool_call>call:get_time{timezone:<|"|>UTC<|"|>}<tool_call|>'
        )
        result = self.detector.detect_and_parse(text, extra_tools)
        self.assertEqual(len(result.calls), 2)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(result.calls[1].name, "get_time")
        self.assertEqual(result.normal_text, "Some text ")
```
**EN:** This test exercises `test_detect_and_parse_multiple_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_multiple_calls`。

### Lines 5110-5112: test case parse gemma4 args empty / 测试用例 parse gemma4 args empty
```python
    def test_parse_gemma4_args_empty(self):
        self.assertEqual(_parse_gemma4_args(""), {})
        self.assertEqual(_parse_gemma4_args("   "), {})
```
**EN:** This test exercises `test_parse_gemma4_args_empty` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_gemma4_args_empty`。

### Lines 5114-5117: test case parse gemma4 args booleans / 测试用例 parse gemma4 args booleans
```python
    def test_parse_gemma4_args_booleans(self):
        result = _parse_gemma4_args("flag:true,other:false")
        self.assertIs(result["flag"], True)
        self.assertIs(result["other"], False)
```
**EN:** This test exercises `test_parse_gemma4_args_booleans` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_gemma4_args_booleans`。

### Lines 5119-5122: test case parse gemma4 args numbers / 测试用例 parse gemma4 args numbers
```python
    def test_parse_gemma4_args_numbers(self):
        result = _parse_gemma4_args("count:42,ratio:3.14")
        self.assertEqual(result["count"], 42)
        self.assertAlmostEqual(result["ratio"], 3.14)
```
**EN:** This test exercises `test_parse_gemma4_args_numbers` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_gemma4_args_numbers`。

### Lines 5124-5126: test case parse gemma4 args string with colon / 测试用例 parse gemma4 args string with colon
```python
    def test_parse_gemma4_args_string_with_colon(self):
        result = _parse_gemma4_args('url:<|"|>http://example.com<|"|>')
        self.assertEqual(result["url"], "http://example.com")
```
**EN:** This test exercises `test_parse_gemma4_args_string_with_colon` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_gemma4_args_string_with_colon`。

### Lines 5128-5132: test case parse gemma4 args nested object / 测试用例 parse gemma4 args nested object
```python
    def test_parse_gemma4_args_nested_object(self):
        result = _parse_gemma4_args('outer:{inner:<|"|>val<|"|>,num:5}')
        self.assertIsInstance(result["outer"], dict)
        self.assertEqual(result["outer"]["inner"], "val")
        self.assertEqual(result["outer"]["num"], 5)
```
**EN:** This test exercises `test_parse_gemma4_args_nested_object` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_gemma4_args_nested_object`。

### Lines 5134-5140: test case parse gemma4 array mixed types / 测试用例 parse gemma4 array mixed types
```python
    def test_parse_gemma4_array_mixed_types(self):
        result = _parse_gemma4_array('<|"|>hello<|"|>, 42, true, {key:<|"|>val<|"|>}')
        self.assertEqual(result[0], "hello")
        self.assertEqual(result[1], 42)
        self.assertIs(result[2], True)
        self.assertIsInstance(result[3], dict)
        self.assertEqual(result[3]["key"], "val")
```
**EN:** This test exercises `test_parse_gemma4_array_mixed_types` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_gemma4_array_mixed_types`。

### Lines 5142-5148: test case parse gemma4 value types / 测试用例 parse gemma4 value types
```python
    def test_parse_gemma4_value_types(self):
        self.assertIs(_parse_gemma4_value("true"), True)
        self.assertIs(_parse_gemma4_value("false"), False)
        self.assertEqual(_parse_gemma4_value("42"), 42)
        self.assertAlmostEqual(_parse_gemma4_value("3.14"), 3.14)
        self.assertEqual(_parse_gemma4_value("hello"), "hello")
        self.assertEqual(_parse_gemma4_value(""), "")
```
**EN:** This test exercises `test_parse_gemma4_value_types` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_gemma4_value_types`。

### Lines 5150-5170: method collect streaming / 方法 collect streaming
```python
    def _collect_streaming(self, chunks):
        """Helper: feed chunks and collect normal text + tool calls by index."""
        normal_text = ""
        tool_calls_by_index = {}
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            normal_text += result.normal_text
            for call in result.calls:
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }
                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters
        return normal_text, tool_calls_by_index
```
**EN:** Helper: feed chunks and collect normal text + tool calls by index. This block implements `_collect_streaming` and captures one focused piece of the module's behavior.
**CN:** Helper: feed chunks and collect normal text + tool calls by index. 该代码块实现 `_collect_streaming`，承担模块行为中的一个聚焦逻辑片段。

### Lines 5172-5218: test case streaming multiple tool calls / 测试用例 streaming multiple tool calls
```python
    def test_streaming_multiple_tool_calls(self):
        """Test streaming with two consecutive tool calls."""
        extra_tools = self.tools + [
            Tool(
                type="function",
                function=Function(
                    name="get_time",
                    description="Get current time",
                    parameters={
                        "type": "object",
                        "properties": {"timezone": {"type": "string"}},
                    },
                ),
            )
        ]
        chunks = [
            '<|tool_call>call:get_weather{location:<|"|>',
            'Tokyo<|"|>}<tool_call|>',
            ' <|tool_call>call:get_time{timezone:<|"|>',
            'UTC<|"|>}<tool_call|>',
        ]
        normal_text = ""
        tool_calls_by_index = {}
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, extra_tools)
            normal_text += result.normal_text
            for call in result.calls:
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }
                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters

        self.assertEqual(len(tool_calls_by_index), 2)
        self.assertEqual(tool_calls_by_index[0]["name"], "get_weather")
        self.assertEqual(tool_calls_by_index[1]["name"], "get_time")
        params0 = json.loads(tool_calls_by_index[0]["parameters"])
        params1 = json.loads(tool_calls_by_index[1]["parameters"])
        self.assertEqual(params0["location"], "Tokyo")
        self.assertEqual(params1["timezone"], "UTC")
```
**EN:** Test streaming with two consecutive tool calls. This test exercises `test_streaming_multiple_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with two consecutive tool calls. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_multiple_tool_calls`。

### Lines 5220-5230: test case streaming very small chunks / 测试用例 streaming very small chunks
```python
    def test_streaming_very_small_chunks(self):
        """Test streaming with character-by-character chunks."""
        full_text = '<|tool_call>call:get_weather{location:<|"|>Rome<|"|>}<tool_call|>'
        chunks = list(full_text)

        normal_text, tool_calls = self._collect_streaming(chunks)

        self.assertEqual(len(tool_calls), 1)
        self.assertEqual(tool_calls[0]["name"], "get_weather")
        params = json.loads(tool_calls[0]["parameters"])
        self.assertEqual(params["location"], "Rome")
```
**EN:** Test streaming with character-by-character chunks. This test exercises `test_streaming_very_small_chunks` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with character-by-character chunks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_very_small_chunks`。

### Lines 5232-5237: test case streaming empty args / 测试用例 streaming empty args
```python
    def test_streaming_empty_args(self):
        """Test streaming a tool call with no arguments."""
        chunks = ["<|tool_call>call:get_weather{}", "<tool_call|>"]
        normal_text, tool_calls = self._collect_streaming(chunks)
        self.assertEqual(len(tool_calls), 1)
        self.assertEqual(tool_calls[0]["name"], "get_weather")
```
**EN:** Test streaming a tool call with no arguments. This test exercises `test_streaming_empty_args` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming a tool call with no arguments. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_empty_args`。

### Lines 5239-5286: test case streaming text between tool calls / 测试用例 streaming text between tool calls
```python
    def test_streaming_text_between_tool_calls(self):
        """Test streaming with normal text interleaved between two different tool calls."""
        extra_tools = self.tools + [
            Tool(
                type="function",
                function=Function(
                    name="get_time",
                    description="Get current time",
                    parameters={
                        "type": "object",
                        "properties": {"timezone": {"type": "string"}},
                    },
                ),
            )
        ]
        chunks = [
            "Hello! ",
            '<|tool_call>call:get_weather{location:<|"|>Paris<|"|>}<tool_call|>',
            " Let me also check ",
            '<|tool_call>call:get_time{timezone:<|"|>UTC<|"|>}<tool_call|>',
        ]
        normal_text = ""
        tool_calls_by_index = {}
        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, extra_tools)
            normal_text += result.normal_text
            for call in result.calls:
                if call.tool_index is not None:
                    if call.tool_index not in tool_calls_by_index:
                        tool_calls_by_index[call.tool_index] = {
                            "name": "",
                            "parameters": "",
                        }
                    if call.name:
                        tool_calls_by_index[call.tool_index]["name"] = call.name
                    if call.parameters:
                        tool_calls_by_index[call.tool_index][
                            "parameters"
                        ] += call.parameters
        self.assertIn("Hello!", normal_text)
        self.assertIn("Let me also check", normal_text)
        self.assertEqual(len(tool_calls_by_index), 2)
        self.assertEqual(tool_calls_by_index[0]["name"], "get_weather")
        self.assertEqual(tool_calls_by_index[1]["name"], "get_time")
        params0 = json.loads(tool_calls_by_index[0]["parameters"])
        params1 = json.loads(tool_calls_by_index[1]["parameters"])
        self.assertEqual(params0["location"], "Paris")
        self.assertEqual(params1["timezone"], "UTC")
```
**EN:** Test streaming with normal text interleaved between two different tool calls. This test exercises `test_streaming_text_between_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with normal text interleaved between two different tool calls. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_text_between_tool_calls`。

### Lines 5289-5290: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestPythonicDetector`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestMistralDetector`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestBaseFormatDetector`: Test buffer management and sequential tool index assignment in BaseFormatDetector. / 用于组织相关测试、夹具或辅助方法。
- `TestLlama32Detector`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestKimiK2Detector`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDeepSeekV3Detector`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDeepSeekV32Detector`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDeepSeekV4Detector`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestPythonicDetector.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestPythonicDetector.test_parse_streaming_no_brackets`: Test parsing text with no brackets (no tool calls). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_no_brackets`。
- `TestPythonicDetector.test_parse_streaming_complete_tool_call`: Test parsing a complete tool call. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_complete_tool_call`。
- `TestPythonicDetector.test_parse_streaming_text_before_tool_call`: Test parsing text that appears before a tool call. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_streaming_text_before_tool_call`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.base_format_detector`, `sglang.srt.function_call.core_types`, `sglang.srt.function_call.deepseekv3_detector`, `sglang.srt.function_call.deepseekv4_detector`, `sglang.srt.function_call.deepseekv32_detector`, `sglang.srt.function_call.gemma4_detector`, `sglang.srt.function_call.gigachat3_detector`, `sglang.srt.function_call.glm4_moe_detector`, `sglang.srt.function_call.glm47_moe_detector`, `sglang.srt.function_call.gpt_oss_detector`, `sglang.srt.function_call.json_array_parser`, `sglang.srt.function_call.kimik2_detector`, `sglang.srt.function_call.lfm2_detector`, `sglang.srt.function_call.llama32_detector`, `sglang.srt.function_call.mistral_detector`, `sglang.srt.function_call.pythonic_detector`, `sglang.srt.function_call.qwen3_coder_detector`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 5290
