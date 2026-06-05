# test_kimik2_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/function_call/test_kimik2_detector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates kimik2 detector behavior in SGLang's function call area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 函数调用 领域中与 kimik2 detector 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: module imports and dependencies / 模块导入与依赖
```python
import json
import unittest

from sglang.srt.entrypoints.openai.protocol import Function, Tool
from sglang.srt.function_call.kimik2_detector import (
    KimiK2Detector as KimiK2FuncDetector,
)
from sglang.srt.function_call.kimik2_detector import (
    _strip_special_tokens,
)
from sglang.srt.parser.reasoning_parser import KimiK2Detector as KimiK2ReasoningDetector
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `unittest`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.kimik2_detector`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `unittest`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.kimik2_detector`。

### Lines 14-14: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(5, "base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 17-33: function make tool / 函数 make tool
```python
def _make_tool(name, parameters=None):
    """Helper to create a Tool with less boilerplate."""
    return Tool(
        type="function",
        function=Function(
            name=name,
            description=f"{name} tool",
            parameters=parameters
            or {
                "type": "object",
                "properties": {
                    "path": {"type": "string", "description": "File path"},
                },
                "required": ["path"],
            },
        ),
    )
```
**EN:** Helper to create a Tool with less boilerplate. This block implements `_make_tool` and captures one focused piece of the module's behavior.
**CN:** Helper to create a Tool with less boilerplate. 该代码块实现 `_make_tool`，承担模块行为中的一个聚焦逻辑片段。

### Lines 36-52: function collect streaming tool calls / 函数 collect streaming tool calls
```python
def _collect_streaming_tool_calls(detector, chunks, tools):
    """Run streaming chunks through a detector and collect assembled tool calls."""
    tool_calls = []
    all_normal_text = ""
    for chunk in chunks:
        result = detector.parse_streaming_increment(chunk, tools)
        all_normal_text += result.normal_text
        for tc_chunk in result.calls:
            if tc_chunk.tool_index is not None:
                while len(tool_calls) <= tc_chunk.tool_index:
                    tool_calls.append({"name": "", "parameters": ""})
                tc = tool_calls[tc_chunk.tool_index]
                if tc_chunk.name:
                    tc["name"] = tc_chunk.name
                if tc_chunk.parameters:
                    tc["parameters"] += tc_chunk.parameters
    return tool_calls, all_normal_text
```
**EN:** Run streaming chunks through a detector and collect assembled tool calls. This block implements `_collect_streaming_tool_calls` and captures one focused piece of the module's behavior.
**CN:** Run streaming chunks through a detector and collect assembled tool calls. 该代码块实现 `_collect_streaming_tool_calls`，承担模块行为中的一个聚焦逻辑片段。

### Lines 53-59: supporting source context / 辅助源码上下文
```python


# ============================================================
# Part 1: KimiK2Detector (function call parsing) tests
# ============================================================


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 60-60: class TestKimiK2DetectorBasic declaration / 类 TestKimiK2DetectorBasic 声明
```python
class TestKimiK2DetectorBasic(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 61-61: supporting statements / 辅助语句
```python
    """Basic non-streaming parsing tests for KimiK2Detector."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 63-78: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.tools = [
            _make_tool("ReadFile"),
            _make_tool(
                "get_weather",
                {
                    "type": "object",
                    "properties": {
                        "city": {"type": "string"},
                        "unit": {"type": "string"},
                    },
                    "required": ["city"],
                },
            ),
        ]
        self.detector = KimiK2FuncDetector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 80-93: test case single tool call / 测试用例 single tool call
```python
    def test_single_tool_call(self):
        """Parse a single complete tool call."""
        text = (
            "<|tool_calls_section_begin|>"
            "<|tool_call_begin|>functions.ReadFile:0"
            '<|tool_call_argument_begin|>{"path": "/test.py"}'
            "<|tool_call_end|>"
            "<|tool_calls_section_end|>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "ReadFile")
        self.assertEqual(result.calls[0].parameters, '{"path": "/test.py"}')
        self.assertEqual(result.normal_text, "")
```
**EN:** Parse a single complete tool call. This test exercises `test_single_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Parse a single complete tool call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_tool_call`。

### Lines 95-111: test case multiple tool calls / 测试用例 multiple tool calls
```python
    def test_multiple_tool_calls(self):
        """Parse two consecutive tool calls."""
        text = (
            "<|tool_calls_section_begin|>"
            "<|tool_call_begin|>functions.ReadFile:0"
            '<|tool_call_argument_begin|>{"path": "/a.py"}'
            "<|tool_call_end|>"
            "<|tool_call_begin|>functions.get_weather:1"
            '<|tool_call_argument_begin|>{"city": "Tokyo"}'
            "<|tool_call_end|>"
            "<|tool_calls_section_end|>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 2)
        self.assertEqual(result.calls[0].name, "ReadFile")
        self.assertEqual(result.calls[1].name, "get_weather")
        self.assertEqual(result.calls[1].parameters, '{"city": "Tokyo"}')
```
**EN:** Parse two consecutive tool calls. This test exercises `test_multiple_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Parse two consecutive tool calls. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_tool_calls`。

### Lines 113-125: test case normal text before tool call / 测试用例 normal text before tool call
```python
    def test_normal_text_before_tool_call(self):
        """Normal text before tool call markers is preserved."""
        text = (
            "Let me check the file."
            "<|tool_calls_section_begin|>"
            "<|tool_call_begin|>functions.ReadFile:0"
            '<|tool_call_argument_begin|>{"path": "/test.py"}'
            "<|tool_call_end|>"
            "<|tool_calls_section_end|>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.normal_text, "Let me check the file.")
```
**EN:** Normal text before tool call markers is preserved. This test exercises `test_normal_text_before_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Normal text before tool call markers is preserved. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_normal_text_before_tool_call`。

### Lines 127-132: test case no tool call / 测试用例 no tool call
```python
    def test_no_tool_call(self):
        """Text without tool call markers returns as normal text."""
        text = "Just a normal response."
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 0)
        self.assertEqual(result.normal_text, text)
```
**EN:** Text without tool call markers returns as normal text. This test exercises `test_no_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Text without tool call markers returns as normal text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_tool_call`。

### Lines 134-139: test case has tool call / 测试用例 has tool call
```python
    def test_has_tool_call(self):
        """has_tool_call correctly detects the presence of tool call markers."""
        self.assertTrue(
            self.detector.has_tool_call("<|tool_calls_section_begin|>stuff")
        )
        self.assertFalse(self.detector.has_tool_call("no markers here"))
```
**EN:** has_tool_call correctly detects the presence of tool call markers. This test exercises `test_has_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** has_tool_call correctly detects the presence of tool call markers. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_has_tool_call`。

### Lines 142-142: class TestKimiK2DetectorHyphenatedNames declaration / 类 TestKimiK2DetectorHyphenatedNames 声明
```python
class TestKimiK2DetectorHyphenatedNames(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 143-143: supporting statements / 辅助语句
```python
    """Test support for hyphenated function names (common in MCP tools)."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 145-150: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.tools = [
            _make_tool("mcp__portal__search-documents"),
            _make_tool("list-files"),
        ]
        self.detector = KimiK2FuncDetector()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 152-163: test case hyphenated name non streaming / 测试用例 hyphenated name non streaming
```python
    def test_hyphenated_name_non_streaming(self):
        """Parse tool call with hyphenated function name."""
        text = (
            "<|tool_calls_section_begin|>"
            "<|tool_call_begin|>functions.mcp__portal__search-documents:0"
            '<|tool_call_argument_begin|>{"path": "/docs"}'
            "<|tool_call_end|>"
            "<|tool_calls_section_end|>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "mcp__portal__search-documents")
```
**EN:** Parse tool call with hyphenated function name. This test exercises `test_hyphenated_name_non_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Parse tool call with hyphenated function name. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hyphenated_name_non_streaming`。

### Lines 165-179: test case hyphenated name streaming / 测试用例 hyphenated name streaming
```python
    def test_hyphenated_name_streaming(self):
        """Stream tool call with hyphenated function name."""
        chunks = [
            "<|tool_calls_section_begin|>"
            "<|tool_call_begin|>functions.list-files:0"
            '<|tool_call_argument_begin|>{"path',
            '": "/home"}',
            "<|tool_call_end|>",
            "<|tool_calls_section_end|>",
        ]
        tool_calls, _ = _collect_streaming_tool_calls(self.detector, chunks, self.tools)
        self.assertEqual(len(tool_calls), 1)
        self.assertEqual(tool_calls[0]["name"], "list-files")
        params = json.loads(tool_calls[0]["parameters"])
        self.assertEqual(params["path"], "/home")
```
**EN:** Stream tool call with hyphenated function name. This test exercises `test_hyphenated_name_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Stream tool call with hyphenated function name. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hyphenated_name_streaming`。

### Lines 182-182: class TestKimiK2DetectorStreaming declaration / 类 TestKimiK2DetectorStreaming 声明
```python
class TestKimiK2DetectorStreaming(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 183-183: supporting statements / 辅助语句
```python
    """Streaming incremental parsing tests for KimiK2Detector."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 185-196: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.tools = [
            _make_tool("ReadFile"),
            _make_tool(
                "get_weather",
                {
                    "type": "object",
                    "properties": {"city": {"type": "string"}},
                    "required": ["city"],
                },
            ),
        ]
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 198-212: test case streaming single tool call / 测试用例 streaming single tool call
```python
    def test_streaming_single_tool_call(self):
        """Stream a single tool call across multiple chunks."""
        detector = KimiK2FuncDetector()
        chunks = [
            "<|tool_calls_section_begin|>"
            "<|tool_call_begin|>functions.ReadFile:0"
            "<|tool_call_argument_begin|>{",
            '"path": "/test.py"',
            "}",
            "<|tool_call_end|><|tool_calls_section_end|>",
        ]
        tool_calls, _ = _collect_streaming_tool_calls(detector, chunks, self.tools)
        self.assertEqual(len(tool_calls), 1)
        self.assertEqual(tool_calls[0]["name"], "ReadFile")
        self.assertEqual(tool_calls[0]["parameters"], '{"path": "/test.py"}')
```
**EN:** Stream a single tool call across multiple chunks. This test exercises `test_streaming_single_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Stream a single tool call across multiple chunks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_single_tool_call`。

### Lines 214-231: test case streaming multiple tool calls / 测试用例 streaming multiple tool calls
```python
    def test_streaming_multiple_tool_calls(self):
        """Stream two tool calls sequentially."""
        detector = KimiK2FuncDetector()
        chunks = [
            "<|tool_calls_section_begin|>"
            "<|tool_call_begin|>functions.ReadFile:0"
            '<|tool_call_argument_begin|>{"path": "/a.py"}',
            "<|tool_call_end|>",
            "<|tool_call_begin|>functions.get_weather:1"
            '<|tool_call_argument_begin|>{"city": "Paris"}',
            "<|tool_call_end|>",
            "<|tool_calls_section_end|>",
        ]
        tool_calls, _ = _collect_streaming_tool_calls(detector, chunks, self.tools)
        self.assertEqual(len(tool_calls), 2)
        self.assertEqual(tool_calls[0]["name"], "ReadFile")
        self.assertEqual(tool_calls[1]["name"], "get_weather")
        self.assertEqual(json.loads(tool_calls[1]["parameters"]), {"city": "Paris"})
```
**EN:** Stream two tool calls sequentially. This test exercises `test_streaming_multiple_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Stream two tool calls sequentially. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_multiple_tool_calls`。

### Lines 233-247: test case streaming state reset after completion / 测试用例 streaming state reset after completion
```python
    def test_streaming_state_reset_after_completion(self):
        """Buffer and state reset after tool call completes."""
        detector = KimiK2FuncDetector()
        chunks = [
            "<|tool_calls_section_begin|>"
            "<|tool_call_begin|>functions.ReadFile:0"
            '<|tool_call_argument_begin|>{"path": "/x"}',
            "<|tool_call_end|>",
            "<|tool_calls_section_end|>",
        ]
        for chunk in chunks:
            detector.parse_streaming_increment(chunk, self.tools)

        self.assertEqual(detector._buffer, "")
        self.assertEqual(detector.current_tool_id, 1)
```
**EN:** Buffer and state reset after tool call completes. This test exercises `test_streaming_state_reset_after_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Buffer and state reset after tool call completes. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_state_reset_after_completion`。

### Lines 250-250: class TestKimiK2DetectorSpecialTokenLeakage declaration / 类 TestKimiK2DetectorSpecialTokenLeakage 声明
```python
class TestKimiK2DetectorSpecialTokenLeakage(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 251-251: supporting statements / 辅助语句
```python
    """Verify special tokens are never leaked into normal_text output."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 253-254: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.tools = [_make_tool("ReadFile")]
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 256-263: test case no leak in non tool text / 测试用例 no leak in non tool text
```python
    def test_no_leak_in_non_tool_text(self):
        """End tokens appearing without start tokens are stripped from output."""
        detector = KimiK2FuncDetector()
        result = detector.parse_streaming_increment(
            "normal text<|tool_calls_section_end|>", self.tools
        )
        self.assertNotIn("<|tool_calls_section_end|>", result.normal_text)
        self.assertIn("normal text", result.normal_text)
```
**EN:** End tokens appearing without start tokens are stripped from output. This test exercises `test_no_leak_in_non_tool_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** End tokens appearing without start tokens are stripped from output. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_leak_in_non_tool_text`。

### Lines 265-271: test case no leak of argument begin token / 测试用例 no leak of argument begin token
```python
    def test_no_leak_of_argument_begin_token(self):
        """Argument begin token is stripped when leaked."""
        detector = KimiK2FuncDetector()
        result = detector.parse_streaming_increment(
            "text<|tool_call_argument_begin|>more", self.tools
        )
        self.assertNotIn("<|tool_call_argument_begin|>", result.normal_text)
```
**EN:** Argument begin token is stripped when leaked. This test exercises `test_no_leak_of_argument_begin_token` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Argument begin token is stripped when leaked. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_leak_of_argument_begin_token`。

### Lines 273-278: test case no leak on error fallback / 测试用例 no leak on error fallback
```python
    def test_no_leak_on_error_fallback(self):
        """On parse errors, normal_text fallback has tokens stripped."""
        cleaned = _strip_special_tokens(
            "leaked<|tool_calls_section_begin|>" "<|tool_call_end|>content"
        )
        self.assertEqual(cleaned, "leakedcontent")
```
**EN:** On parse errors, normal_text fallback has tokens stripped. This test exercises `test_no_leak_on_error_fallback` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** On parse errors, normal_text fallback has tokens stripped. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_leak_on_error_fallback`。

### Lines 280-289: test case strip special tokens all tokens / 测试用例 strip special tokens all tokens
```python
    def test_strip_special_tokens_all_tokens(self):
        """All 5 known special tokens are stripped."""
        dirty = (
            "<|tool_calls_section_begin|>"
            "<|tool_call_begin|>"
            "<|tool_call_argument_begin|>"
            "<|tool_call_end|>"
            "<|tool_calls_section_end|>"
        )
        self.assertEqual(_strip_special_tokens(dirty), "")
```
**EN:** All 5 known special tokens are stripped. This test exercises `test_strip_special_tokens_all_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** All 5 known special tokens are stripped. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_strip_special_tokens_all_tokens`。

### Lines 291-294: test case strip preserves normal text / 测试用例 strip preserves normal text
```python
    def test_strip_preserves_normal_text(self):
        """Stripping doesn't affect normal text content."""
        text = "Hello world, this is normal text."
        self.assertEqual(_strip_special_tokens(text), text)
```
**EN:** Stripping doesn't affect normal text content. This test exercises `test_strip_preserves_normal_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Stripping doesn't affect normal text content. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_strip_preserves_normal_text`。

### Lines 295-301: supporting source context / 辅助源码上下文
```python


# ============================================================
# Part 2: KimiK2ReasoningDetector tests
# ============================================================


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 302-302: class TestKimiK2ReasoningDetectorNonStreaming declaration / 类 TestKimiK2ReasoningDetectorNonStreaming 声明
```python
class TestKimiK2ReasoningDetectorNonStreaming(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 303-303: supporting statements / 辅助语句
```python
    """Non-streaming tests for KimiK2ReasoningDetector."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 305-318: test case normal reasoning with think end / 测试用例 normal reasoning with think end
```python
    def test_normal_reasoning_with_think_end(self):
        """Standard case: <think>...</think> followed by tool call markers."""
        det = KimiK2ReasoningDetector()
        text = (
            "<think>I need to check the file.</think>"
            "<|tool_calls_section_begin|>"
            "<|tool_call_begin|>functions.ReadFile:0"
            '<|tool_call_argument_begin|>{"path": "/test.py"}'
            "<|tool_call_end|>"
            "<|tool_calls_section_end|>"
        )
        result = det.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "I need to check the file.")
        self.assertIn("<|tool_calls_section_begin|>", result.normal_text)
```
**EN:** Standard case: <think>...</think> followed by tool call markers. This test exercises `test_normal_reasoning_with_think_end` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Standard case: <think>...</think> followed by tool call markers. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_normal_reasoning_with_think_end`。

### Lines 320-347: test case tool call inside think without close tag / 测试用例 tool call inside think without close tag
```python
    def test_tool_call_inside_think_without_close_tag(self):
        """
        BUG FIX: Model outputs tool call markers inside <think> without </think>.

        This is the primary scenario that caused special token leakage.
        The model decides to call a tool while reasoning and directly outputs
        <|tool_calls_section_begin|> without first closing with </think>.
        """
        det = KimiK2ReasoningDetector()
        text = (
            "<think>Let me read this file..."
            "<|tool_calls_section_begin|>"
            "<|tool_call_begin|>functions.ReadFile:0"
            '<|tool_call_argument_begin|>{"path": "/test.py"}'
            "<|tool_call_end|>"
            "<|tool_calls_section_end|>"
        )
        result = det.detect_and_parse(text)

        # Reasoning content must NOT contain tool call tokens
        self.assertNotIn("<|tool_calls_section_begin|>", result.reasoning_text)
        self.assertNotIn("<|tool_call_begin|>", result.reasoning_text)
        self.assertIn("Let me read this file...", result.reasoning_text)
        self.assertNotIn("<think>", result.reasoning_text)

        # Tool call markers must be in normal_text for downstream parsing
        self.assertIn("<|tool_calls_section_begin|>", result.normal_text)
        self.assertIn("<|tool_call_begin|>", result.normal_text)
```
**EN:** BUG FIX: Model outputs tool call markers inside <think> without </think>. This test exercises `test_tool_call_inside_think_without_close_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** BUG FIX: Model outputs tool call markers inside <think> without </think>. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_call_inside_think_without_close_tag`。

### Lines 349-361: test case no reasoning just tool call / 测试用例 no reasoning just tool call
```python
    def test_no_reasoning_just_tool_call(self):
        """No <think> block, just tool call markers — pass through as normal_text."""
        det = KimiK2ReasoningDetector()
        text = (
            "<|tool_calls_section_begin|>"
            "<|tool_call_begin|>functions.ReadFile:0"
            '<|tool_call_argument_begin|>{"path": "/x"}'
            "<|tool_call_end|>"
            "<|tool_calls_section_end|>"
        )
        result = det.detect_and_parse(text)
        self.assertEqual(result.reasoning_text, "")
        self.assertIn("<|tool_calls_section_begin|>", result.normal_text)
```
**EN:** No <think> block, just tool call markers — pass through as normal_text. This test exercises `test_no_reasoning_just_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** No <think> block, just tool call markers — pass through as normal_text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_reasoning_just_tool_call`。

### Lines 363-368: test case normal text without reasoning / 测试用例 normal text without reasoning
```python
    def test_normal_text_without_reasoning(self):
        """Plain text without reasoning or tool calls."""
        det = KimiK2ReasoningDetector()
        result = det.detect_and_parse("Hello, how can I help?")
        self.assertEqual(result.normal_text, "Hello, how can I help?")
        self.assertEqual(result.reasoning_text, "")
```
**EN:** Plain text without reasoning or tool calls. This test exercises `test_normal_text_without_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Plain text without reasoning or tool calls. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_normal_text_without_reasoning`。

### Lines 371-371: class TestKimiK2ReasoningDetectorStreaming declaration / 类 TestKimiK2ReasoningDetectorStreaming 声明
```python
class TestKimiK2ReasoningDetectorStreaming(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 372-372: supporting statements / 辅助语句
```python
    """Streaming tests for KimiK2ReasoningDetector."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 374-383: method run streaming / 方法 run streaming
```python
    def _run_streaming(self, chunks, **kwargs):
        """Helper: run chunks through streaming detector, collect reasoning and normal text."""
        det = KimiK2ReasoningDetector(**kwargs)
        all_reasoning = ""
        all_normal = ""
        for chunk in chunks:
            r = det.parse_streaming_increment(chunk)
            all_reasoning += r.reasoning_text
            all_normal += r.normal_text
        return all_reasoning, all_normal
```
**EN:** Helper: run chunks through streaming detector, collect reasoning and normal text. This block implements `_run_streaming` and captures one focused piece of the module's behavior.
**CN:** Helper: run chunks through streaming detector, collect reasoning and normal text. 该代码块实现 `_run_streaming`，承担模块行为中的一个聚焦逻辑片段。

### Lines 385-398: test case streaming normal think then tool call / 测试用例 streaming normal think then tool call
```python
    def test_streaming_normal_think_then_tool_call(self):
        """Standard streaming: <think>...</think> then tool call markers."""
        reasoning, normal = self._run_streaming(
            [
                "<think>",
                "Analyzing the request...",
                "</think>",
                "<|tool_calls_section_begin|>",
                "<|tool_call_begin|>functions.ReadFile:0",
            ]
        )
        self.assertIn("Analyzing the request...", reasoning)
        self.assertIn("<|tool_calls_section_begin|>", normal)
        self.assertNotIn("<|tool_calls_section_begin|>", reasoning)
```
**EN:** Standard streaming: <think>...</think> then tool call markers. This test exercises `test_streaming_normal_think_then_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Standard streaming: <think>...</think> then tool call markers. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_normal_think_then_tool_call`。

### Lines 400-429: test case streaming tool call inside think / 测试用例 streaming tool call inside think
```python
    def test_streaming_tool_call_inside_think(self):
        """
        BUG FIX (streaming): Tool call markers inside <think> without </think>.

        This is the streaming equivalent of the primary bug. The model streams
        reasoning content, then directly outputs tool call markers without </think>.
        """
        reasoning, normal = self._run_streaming(
            [
                "<think>",
                "I need to",
                " read the file.",
                "<|tool_calls_section_begin|>",
                "<|tool_call_begin|>functions.ReadFile:5",
                "<|tool_call_argument_begin|>",
                '{"path": "/Users/user/project/file.ts"}',
                "<|tool_call_end|>",
                "<|tool_calls_section_end|>",
            ]
        )

        # Reasoning is clean
        self.assertIn("I need to read the file.", reasoning)
        self.assertNotIn("<|tool_calls_section_begin|>", reasoning)
        self.assertNotIn("<|tool_call_begin|>", reasoning)
        self.assertNotIn("<think>", reasoning)

        # Tool call markers are in normal_text
        self.assertIn("<|tool_calls_section_begin|>", normal)
        self.assertIn("functions.ReadFile:5", normal)
```
**EN:** BUG FIX (streaming): Tool call markers inside <think> without </think>. This test exercises `test_streaming_tool_call_inside_think` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** BUG FIX (streaming): Tool call markers inside <think> without </think>. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_tool_call_inside_think`。

### Lines 431-440: test case streaming tool call marker in single chunk / 测试用例 streaming tool call marker in single chunk
```python
    def test_streaming_tool_call_marker_in_single_chunk(self):
        """Tool call marker arrives in a single chunk while in reasoning mode."""
        reasoning, normal = self._run_streaming(
            [
                "<think>thinking...",
                '<|tool_calls_section_begin|><|tool_call_begin|>functions.ReadFile:0<|tool_call_argument_begin|>{"path": "/x"}',
            ]
        )
        self.assertIn("thinking...", reasoning)
        self.assertIn("<|tool_calls_section_begin|>", normal)
```
**EN:** Tool call marker arrives in a single chunk while in reasoning mode. This test exercises `test_streaming_tool_call_marker_in_single_chunk` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Tool call marker arrives in a single chunk while in reasoning mode. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_tool_call_marker_in_single_chunk`。

### Lines 442-464: test case streaming partial marker buffering / 测试用例 streaming partial marker buffering
```python
    def test_streaming_partial_marker_buffering(self):
        """
        Partial tool call marker at end of chunk is buffered to prevent
        premature streaming of marker characters as reasoning content.
        """
        det = KimiK2ReasoningDetector(stream_reasoning=True)

        # First chunk: reasoning + partial marker "<|tool_calls"
        det._in_reasoning = True
        det.stripped_think_start = True

        r1 = det.parse_streaming_increment("some reasoning")
        self.assertEqual(r1.reasoning_text, "some reasoning")

        # Chunk that ends with start of marker
        r2 = det.parse_streaming_increment("<|tool")
        # Partial marker should be buffered, not streamed
        self.assertNotIn("<|tool", r2.reasoning_text)

        # Complete the marker
        r3 = det.parse_streaming_increment("_calls_section_begin|>rest")
        # Now it should force-exit reasoning
        self.assertIn("<|tool_calls_section_begin|>", r3.normal_text)
```
**EN:** Partial tool call marker at end of chunk is buffered to prevent premature streaming of marker characters as reasoning content. This test exercises `test_streaming_partial_marker_buffering` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Partial tool call marker at end of chunk is buffered to prevent premature streaming of marker characters as reasoning content. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_partial_marker_buffering`。

### Lines 466-476: test case streaming no reasoning mode / 测试用例 streaming no reasoning mode
```python
    def test_streaming_no_reasoning_mode(self):
        """Normal text without reasoning passes through as normal_text."""
        reasoning, normal = self._run_streaming(
            [
                "Hello, I can help with that.",
                " What do you need?",
            ]
        )
        self.assertEqual(reasoning, "")
        self.assertIn("Hello, I can help with that.", normal)
        self.assertIn(" What do you need?", normal)
```
**EN:** Normal text without reasoning passes through as normal_text. This test exercises `test_streaming_no_reasoning_mode` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Normal text without reasoning passes through as normal_text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_no_reasoning_mode`。

### Lines 478-489: test case streaming force reasoning / 测试用例 streaming force reasoning
```python
    def test_streaming_force_reasoning(self):
        """With force_reasoning, content before </think> is reasoning."""
        reasoning, normal = self._run_streaming(
            [
                "I should analyze this...",
                "</think>",
                "Here is the answer.",
            ],
            force_reasoning=True,
        )
        self.assertIn("I should analyze this...", reasoning)
        self.assertIn("Here is the answer.", normal)
```
**EN:** With force_reasoning, content before </think> is reasoning. This test exercises `test_streaming_force_reasoning` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** With force_reasoning, content before </think> is reasoning. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_force_reasoning`。

### Lines 490-496: supporting source context / 辅助源码上下文
```python


# ============================================================
# Part 3: End-to-end integration tests
# ============================================================


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 497-497: class TestKimiK2EndToEnd declaration / 类 TestKimiK2EndToEnd 声明
```python
class TestKimiK2EndToEnd(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 498-505: supporting statements / 辅助语句
```python
    """
    End-to-end tests simulating the full flow:
    reasoning parser -> tool call parser.

    These test the exact bug scenario from the issue: Kimi-K2.5 outputs
    tool call markers inside <think> blocks, which must be correctly
    split between reasoning and tool call parsers.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 507-518: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.tools = [
            _make_tool("ReadFile"),
            _make_tool(
                "get_weather",
                {
                    "type": "object",
                    "properties": {"city": {"type": "string"}},
                    "required": ["city"],
                },
            ),
        ]
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 520-568: test case e2e streaming reasoning to tool call / 测试用例 e2e streaming reasoning to tool call
```python
    def test_e2e_streaming_reasoning_to_tool_call(self):
        """
        Full pipeline: streaming reasoning parser feeds into streaming tool call parser.

        Simulates the exact path through serving_chat.py:
        1. Model outputs <think>reasoning...<|tool_calls_section_begin|>...
        2. ReasoningParser splits: reasoning_text + normal_text
        3. FunctionCallParser receives normal_text and extracts tool calls
        """
        reasoning_det = KimiK2ReasoningDetector(stream_reasoning=True)
        tc_det = KimiK2FuncDetector()

        streaming_chunks = [
            "<think>",
            "I need to read the file",
            " to understand the code.",
            "<|tool_calls_section_begin|>",
            "<|tool_call_begin|>functions.ReadFile:0",
            "<|tool_call_argument_begin|>",
            '{"path": "/Users/user/project/file.ts"}',
            "<|tool_call_end|>",
            "<|tool_calls_section_end|>",
        ]

        all_reasoning = ""
        all_tc_calls = []

        for chunk in streaming_chunks:
            # Step 1: reasoning parser
            r = reasoning_det.parse_streaming_increment(chunk)
            all_reasoning += r.reasoning_text

            # Step 2: feed normal_text into tool call parser (like serving_chat.py does)
            if r.normal_text:
                tc_result = tc_det.parse_streaming_increment(r.normal_text, self.tools)
                all_tc_calls.extend(tc_result.calls)

        # Verify reasoning content
        self.assertIn("I need to read the file to understand the code.", all_reasoning)
        self.assertNotIn("<|", all_reasoning)

        # Verify tool calls were extracted
        name_calls = [c for c in all_tc_calls if c.name]
        self.assertEqual(len(name_calls), 1)
        self.assertEqual(name_calls[0].name, "ReadFile")

        param_calls = [c for c in all_tc_calls if c.parameters]
        full_params = "".join(c.parameters for c in param_calls)
        self.assertIn("/Users/user/project/file.ts", full_params)
```
**EN:** Full pipeline: streaming reasoning parser feeds into streaming tool call parser. This test exercises `test_e2e_streaming_reasoning_to_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Full pipeline: streaming reasoning parser feeds into streaming tool call parser. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_e2e_streaming_reasoning_to_tool_call`。

### Lines 570-596: test case e2e non streaming reasoning to tool call / 测试用例 e2e non streaming reasoning to tool call
```python
    def test_e2e_non_streaming_reasoning_to_tool_call(self):
        """Non-streaming pipeline: reason parser then tool call parser."""
        reasoning_det = KimiK2ReasoningDetector()
        tc_det = KimiK2FuncDetector()

        text = (
            "<think>Let me check this file."
            "<|tool_calls_section_begin|>"
            "<|tool_call_begin|>functions.ReadFile:0"
            '<|tool_call_argument_begin|>{"path": "/src/main.py"}'
            "<|tool_call_end|>"
            "<|tool_calls_section_end|>"
        )

        # Step 1: reasoning parser
        r = reasoning_det.detect_and_parse(text)
        self.assertIn("Let me check this file.", r.reasoning_text)
        self.assertNotIn("<|", r.reasoning_text)

        # Step 2: tool call parser on normal_text
        tc_result = tc_det.detect_and_parse(r.normal_text, self.tools)
        self.assertEqual(len(tc_result.calls), 1)
        self.assertEqual(tc_result.calls[0].name, "ReadFile")
        self.assertEqual(
            json.loads(tc_result.calls[0].parameters),
            {"path": "/src/main.py"},
        )
```
**EN:** Non-streaming pipeline: reason parser then tool call parser. This test exercises `test_e2e_non_streaming_reasoning_to_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Non-streaming pipeline: reason parser then tool call parser. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_e2e_non_streaming_reasoning_to_tool_call`。

### Lines 598-627: test case e2e normal think close then tool call / 测试用例 e2e normal think close then tool call
```python
    def test_e2e_normal_think_close_then_tool_call(self):
        """Standard case with </think> — should also work correctly."""
        reasoning_det = KimiK2ReasoningDetector(stream_reasoning=True)
        tc_det = KimiK2FuncDetector()

        chunks = [
            "<think>",
            "Thinking about it...",
            "</think>",
            "<|tool_calls_section_begin|>",
            "<|tool_call_begin|>functions.get_weather:0",
            '<|tool_call_argument_begin|>{"city": "London"}',
            "<|tool_call_end|>",
            "<|tool_calls_section_end|>",
        ]

        all_reasoning = ""
        all_tc_calls = []

        for chunk in chunks:
            r = reasoning_det.parse_streaming_increment(chunk)
            all_reasoning += r.reasoning_text
            if r.normal_text:
                tc_result = tc_det.parse_streaming_increment(r.normal_text, self.tools)
                all_tc_calls.extend(tc_result.calls)

        self.assertIn("Thinking about it...", all_reasoning)
        name_calls = [c for c in all_tc_calls if c.name]
        self.assertEqual(len(name_calls), 1)
        self.assertEqual(name_calls[0].name, "get_weather")
```
**EN:** Standard case with </think> — should also work correctly. This test exercises `test_e2e_normal_think_close_then_tool_call` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Standard case with </think> — should also work correctly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_e2e_normal_think_close_then_tool_call`。

### Lines 629-663: test case e2e multiple tool calls without think close / 测试用例 e2e multiple tool calls without think close
```python
    def test_e2e_multiple_tool_calls_without_think_close(self):
        """Multiple tool calls inside <think> without </think>."""
        reasoning_det = KimiK2ReasoningDetector(stream_reasoning=True)
        tc_det = KimiK2FuncDetector()

        chunks = [
            "<think>",
            "Let me check both files.",
            "<|tool_calls_section_begin|>",
            "<|tool_call_begin|>functions.ReadFile:0"
            '<|tool_call_argument_begin|>{"path": "/a.py"}',
            "<|tool_call_end|>",
            "<|tool_call_begin|>functions.ReadFile:1"
            '<|tool_call_argument_begin|>{"path": "/b.py"}',
            "<|tool_call_end|>",
            "<|tool_calls_section_end|>",
        ]

        all_reasoning = ""
        all_tc_calls = []

        for chunk in chunks:
            r = reasoning_det.parse_streaming_increment(chunk)
            all_reasoning += r.reasoning_text
            if r.normal_text:
                tc_result = tc_det.parse_streaming_increment(r.normal_text, self.tools)
                all_tc_calls.extend(tc_result.calls)

        self.assertIn("Let me check both files.", all_reasoning)
        self.assertNotIn("<|", all_reasoning)

        name_calls = [c for c in all_tc_calls if c.name]
        self.assertEqual(len(name_calls), 2)
        self.assertEqual(name_calls[0].name, "ReadFile")
        self.assertEqual(name_calls[1].name, "ReadFile")
```
**EN:** Multiple tool calls inside <think> without </think>. This test exercises `test_e2e_multiple_tool_calls_without_think_close` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Multiple tool calls inside <think> without </think>. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_e2e_multiple_tool_calls_without_think_close`。

### Lines 664-670: supporting source context / 辅助源码上下文
```python


# ============================================================
# Part 3: Bare-counter tool call ID parsing
# ============================================================


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 671-671: class TestKimiK2BareCounterParsing declaration / 类 TestKimiK2BareCounterParsing 声明
```python
class TestKimiK2BareCounterParsing(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 672-672: supporting statements / 辅助语句
```python
    """Tests for bare numeric tool_call_id format (e.g., '3' instead of 'functions.ReadFile:0')."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 674-689: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.detector = KimiK2FuncDetector()
        self.tools = [
            _make_tool("ReadFile"),
            _make_tool(
                "get_weather",
                {
                    "type": "object",
                    "properties": {
                        "city": {"type": "string"},
                        "unit": {"type": "string"},
                    },
                    "required": ["city"],
                },
            ),
        ]
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 690-692: supporting source context / 辅助源码上下文
```python

    # --- _parse_tool_call_id ---

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 693-698: test case standard format with functions prefix / 测试用例 standard format with functions prefix
```python
    def test_standard_format_with_functions_prefix(self):
        name, idx = self.detector._parse_tool_call_id(
            "functions.ReadFile:0", self.tools
        )
        self.assertEqual(name, "ReadFile")
        self.assertEqual(idx, 0)
```
**EN:** This test exercises `test_standard_format_with_functions_prefix` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_standard_format_with_functions_prefix`。

### Lines 700-703: test case standard format without functions prefix / 测试用例 standard format without functions prefix
```python
    def test_standard_format_without_functions_prefix(self):
        name, idx = self.detector._parse_tool_call_id("ReadFile:1", self.tools)
        self.assertEqual(name, "ReadFile")
        self.assertEqual(idx, 1)
```
**EN:** This test exercises `test_standard_format_without_functions_prefix` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_standard_format_without_functions_prefix`。

### Lines 705-711: test case bare counter single tool / 测试用例 bare counter single tool
```python
    def test_bare_counter_single_tool(self):
        single_tool = [_make_tool("search")]
        name, idx = self.detector._parse_tool_call_id(
            "3", single_tool, '{"query": "test"}'
        )
        self.assertEqual(name, "search")
        self.assertEqual(idx, 3)
```
**EN:** This test exercises `test_bare_counter_single_tool` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bare_counter_single_tool`。

### Lines 713-718: test case bare counter infers by args / 测试用例 bare counter infers by args
```python
    def test_bare_counter_infers_by_args(self):
        name, idx = self.detector._parse_tool_call_id(
            "0", self.tools, '{"city": "Tokyo"}'
        )
        self.assertEqual(name, "get_weather")
        self.assertEqual(idx, 0)
```
**EN:** This test exercises `test_bare_counter_infers_by_args` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bare_counter_infers_by_args`。

### Lines 720-723: test case bare counter no tools returns none / 测试用例 bare counter no tools returns none
```python
    def test_bare_counter_no_tools_returns_none(self):
        name, idx = self.detector._parse_tool_call_id("5", [], '{"x": 1}')
        self.assertIsNone(name)
        self.assertEqual(idx, 5)
```
**EN:** This test exercises `test_bare_counter_no_tools_returns_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bare_counter_no_tools_returns_none`。

### Lines 725-728: test case bare counter no args multiple tools returns none / 测试用例 bare counter no args multiple tools returns none
```python
    def test_bare_counter_no_args_multiple_tools_returns_none(self):
        name, idx = self.detector._parse_tool_call_id("2", self.tools, None)
        self.assertIsNone(name)
        self.assertEqual(idx, 2)
```
**EN:** This test exercises `test_bare_counter_no_args_multiple_tools_returns_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bare_counter_no_args_multiple_tools_returns_none`。

### Lines 730-733: test case unexpected format returns none / 测试用例 unexpected format returns none
```python
    def test_unexpected_format_returns_none(self):
        name, idx = self.detector._parse_tool_call_id("some_garbage", self.tools)
        self.assertIsNone(name)
        self.assertEqual(idx, 0)
```
**EN:** This test exercises `test_unexpected_format_returns_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unexpected_format_returns_none`。

### Lines 734-736: supporting source context / 辅助源码上下文
```python

    # --- _infer_tool_name ---

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 737-738: test case infer no tools / 测试用例 infer no tools
```python
    def test_infer_no_tools(self):
        self.assertIsNone(self.detector._infer_tool_name([], '{"x": 1}'))
```
**EN:** This test exercises `test_infer_no_tools` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_infer_no_tools`。

### Lines 740-742: test case infer single tool / 测试用例 infer single tool
```python
    def test_infer_single_tool(self):
        result = self.detector._infer_tool_name([_make_tool("only_one")], '{"x": 1}')
        self.assertEqual(result, "only_one")
```
**EN:** This test exercises `test_infer_single_tool` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_infer_single_tool`。

### Lines 744-748: test case infer by argument overlap / 测试用例 infer by argument overlap
```python
    def test_infer_by_argument_overlap(self):
        result = self.detector._infer_tool_name(
            self.tools, '{"city": "Paris", "unit": "celsius"}'
        )
        self.assertEqual(result, "get_weather")
```
**EN:** This test exercises `test_infer_by_argument_overlap` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_infer_by_argument_overlap`。

### Lines 750-752: test case infer malformed json returns none / 测试用例 infer malformed json returns none
```python
    def test_infer_malformed_json_returns_none(self):
        result = self.detector._infer_tool_name(self.tools, '{"city": "Par')
        self.assertIsNone(result)
```
**EN:** This test exercises `test_infer_malformed_json_returns_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_infer_malformed_json_returns_none`。

### Lines 754-756: test case infer empty args returns none / 测试用例 infer empty args returns none
```python
    def test_infer_empty_args_returns_none(self):
        self.assertIsNone(self.detector._infer_tool_name(self.tools, None))
        self.assertIsNone(self.detector._infer_tool_name(self.tools, ""))
```
**EN:** This test exercises `test_infer_empty_args_returns_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_infer_empty_args_returns_none`。

### Lines 758-764: test case infer no matching props returns none / 测试用例 infer no matching props returns none
```python
    def test_infer_no_matching_props_returns_none(self):
        tools_no_props = [
            _make_tool("a", {"type": "object"}),
            _make_tool("b", {"type": "object"}),
        ]
        result = self.detector._infer_tool_name(tools_no_props, '{"x": 1}')
        self.assertIsNone(result)
```
**EN:** This test exercises `test_infer_no_matching_props_returns_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_infer_no_matching_props_returns_none`。

### Lines 765-767: supporting source context / 辅助源码上下文
```python

    # --- detect_and_parse with bare counter (end-to-end) ---

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 768-779: test case detect and parse bare counter / 测试用例 detect and parse bare counter
```python
    def test_detect_and_parse_bare_counter(self):
        text = (
            "<|tool_calls_section_begin|>"
            "<|tool_call_begin|>0"
            '<|tool_call_argument_begin|>{"city": "Tokyo"}'
            "<|tool_call_end|>"
            "<|tool_calls_section_end|>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        self.assertEqual(len(result.calls), 1)
        self.assertEqual(result.calls[0].name, "get_weather")
        self.assertEqual(result.calls[0].parameters, '{"city": "Tokyo"}')
```
**EN:** This test exercises `test_detect_and_parse_bare_counter` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_bare_counter`。

### Lines 781-791: test case detect and parse bare counter skips unknown / 测试用例 detect and parse bare counter skips unknown
```python
    def test_detect_and_parse_bare_counter_skips_unknown(self):
        text = (
            "<|tool_calls_section_begin|>"
            "<|tool_call_begin|>0"
            '<|tool_call_argument_begin|>{"unknown_key": "value"}'
            "<|tool_call_end|>"
            "<|tool_calls_section_end|>"
        )
        result = self.detector.detect_and_parse(text, self.tools)
        # No tool props match, _infer_tool_name returns None, call is skipped
        self.assertEqual(len(result.calls), 0)
```
**EN:** This test exercises `test_detect_and_parse_bare_counter_skips_unknown` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_detect_and_parse_bare_counter_skips_unknown`。

### Lines 793-806: test case streaming bare counter single tool / 测试用例 streaming bare counter single tool
```python
    def test_streaming_bare_counter_single_tool(self):
        detector = KimiK2FuncDetector()
        single_tool = [_make_tool("search")]
        chunks = [
            "<|tool_calls_section_begin|>"
            "<|tool_call_begin|>0"
            '<|tool_call_argument_begin|>{"path',
            '": "/test"}',
            "<|tool_call_end|>",
            "<|tool_calls_section_end|>",
        ]
        tool_calls, _ = _collect_streaming_tool_calls(detector, chunks, single_tool)
        self.assertEqual(len(tool_calls), 1)
        self.assertEqual(tool_calls[0]["name"], "search")
```
**EN:** This test exercises `test_streaming_bare_counter_single_tool` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_bare_counter_single_tool`。

### Lines 809-810: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_make_tool`: Helper to create a Tool with less boilerplate. / 该代码块实现 `_make_tool`，承担模块行为中的一个聚焦逻辑片段。
- `_collect_streaming_tool_calls`: Run streaming chunks through a detector and collect assembled tool calls. / 该代码块实现 `_collect_streaming_tool_calls`，承担模块行为中的一个聚焦逻辑片段。
- `TestKimiK2DetectorBasic`: Basic non-streaming parsing tests for KimiK2Detector. / 用于组织相关测试、夹具或辅助方法。
- `TestKimiK2DetectorHyphenatedNames`: Test support for hyphenated function names (common in MCP tools). / 用于组织相关测试、夹具或辅助方法。
- `TestKimiK2DetectorStreaming`: Streaming incremental parsing tests for KimiK2Detector. / 用于组织相关测试、夹具或辅助方法。
- `TestKimiK2DetectorSpecialTokenLeakage`: Verify special tokens are never leaked into normal_text output. / 用于组织相关测试、夹具或辅助方法。
- `TestKimiK2ReasoningDetectorNonStreaming`: Non-streaming tests for KimiK2ReasoningDetector. / 用于组织相关测试、夹具或辅助方法。
- `TestKimiK2ReasoningDetectorStreaming`: Streaming tests for KimiK2ReasoningDetector. / 用于组织相关测试、夹具或辅助方法。
- `TestKimiK2DetectorBasic.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestKimiK2DetectorBasic.test_single_tool_call`: Parse a single complete tool call. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_tool_call`。
- `TestKimiK2DetectorBasic.test_multiple_tool_calls`: Parse two consecutive tool calls. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_tool_calls`。
- `TestKimiK2DetectorBasic.test_normal_text_before_tool_call`: Normal text before tool call markers is preserved. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_normal_text_before_tool_call`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.kimik2_detector`, `sglang.srt.parser.reasoning_parser`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 810
