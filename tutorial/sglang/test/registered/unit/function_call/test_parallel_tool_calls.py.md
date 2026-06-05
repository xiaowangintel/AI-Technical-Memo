# test_parallel_tool_calls.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/function_call/test_parallel_tool_calls.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates parallel tool calls behavior in SGLang's unit / function call area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 函数调用 领域中与 parallel tool calls 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: module imports and dependencies / 模块导入与依赖
```python
import json
```
**EN:** This block imports the modules needed by the rest of the file, including `json`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`。

### Lines 3-18: supporting statements / 辅助语句
```python
"""
Test case for parallel tool call parsing.

This test verifies that the parser correctly handles parallel tool calls
with array parameters in JSON array format.

Scenario:
- Model outputs two parallel tool calls in JSON array format
- Both tools have array parameters (e.g., "title": ["7.8.9 H-9 ..."])
- First tool completes with closing braces
- Second tool starts with opening brace
- The parser must correctly handle the '[' characters in array parameters
  without confusing them with the JSON array start

Expected behavior: Both tools should be parsed correctly.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 20-24: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.srt.entrypoints.openai.protocol import Function, Tool
from sglang.srt.function_call.json_array_parser import JsonArrayParser
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.json_array_parser`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.json_array_parser`, `sglang.test.ci.ci_register`。

### Lines 26-26: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(5, "base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 29-29: class TestParallelToolCalls declaration / 类 TestParallelToolCalls 声明
```python
class TestParallelToolCalls(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 30-30: supporting statements / 辅助语句
```python
    """Test case for parallel tool call parsing with array parameters."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 32-54: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Set up test tools and detector."""
        self.tools = [
            Tool(
                type="function",
                function=Function(
                    name="search_docs",
                    description="Search documents",
                    parameters={
                        "type": "object",
                        "properties": {
                            "title": {
                                "type": "array",
                                "items": {"type": "string"},
                                "description": "Document title",
                            }
                        },
                        "required": ["title"],
                    },
                ),
            ),
        ]
        self.detector = JsonArrayParser()
```
**EN:** Set up test tools and detector. This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Set up test tools and detector. 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 56-68: method accumulate tool calls / 方法 accumulate tool calls
```python
    def _accumulate_tool_calls(self, tool_calls, result):
        """Helper method to accumulate tool call results from parsing output."""
        if not result.calls:
            return
        for call in result.calls:
            if call.tool_index is None:
                continue
            while len(tool_calls) <= call.tool_index:
                tool_calls.append({"name": "", "parameters": ""})
            if call.name:
                tool_calls[call.tool_index]["name"] = call.name
            if call.parameters:
                tool_calls[call.tool_index]["parameters"] += call.parameters
```
**EN:** Helper method to accumulate tool call results from parsing output. This block implements `_accumulate_tool_calls` and captures one focused piece of the module's behavior.
**CN:** Helper method to accumulate tool call results from parsing output. 该代码块实现 `_accumulate_tool_calls`，承担模块行为中的一个聚焦逻辑片段。

### Lines 70-134: test case parallel tool calls with array parameters / 测试用例 parallel tool calls with array parameters
```python
    def test_parallel_tool_calls_with_array_parameters(self):
        """
        Test parsing two parallel tool calls where both have array parameters.

        This test reproduces the specific scenario:
        - Two tool calls separated by comma
        - Both tools have array parameters containing '[' character
        - First tool completes with '}},'
        - Second tool starts with '{"name": ..., "parameters": {"title": ["'

        Expected: Both tools should be parsed correctly without errors.
        """
        # Simulate more realistic streaming chunks where
        # the key issue is the comma separator followed by second tool with array param
        chunks = [
            "[\n",
            '  {"name": "search_docs", "parameters": {"title": ["7.8.9"',
            '], "filename": "doc1"}},\n',
            '  {"name": "search_docs", "parameters": {"title": ',
            '["4.8"], "filename": "doc2"}}',
            "]",
        ]

        tool_calls = []
        errors = []

        for i, chunk in enumerate(chunks):
            try:
                result = self.detector.parse_streaming_increment(chunk, self.tools)
                # Collect tool calls
                self._accumulate_tool_calls(tool_calls, result)

            except Exception as e:
                errors.append(f"Chunk {i} ({repr(chunk)}): {type(e).__name__}: {e}")

        # Verify no errors occurred
        if errors:
            self.fail("Errors occurred during parsing:\n" + "\n".join(errors))

        # Verify both tool calls were parsed
        self.assertEqual(len(tool_calls), 2, "Should have parsed exactly 2 tool calls")

        # Verify first tool call
        self.assertEqual(
            tool_calls[0]["name"],
            "search_docs",
            "First tool name should be search_docs",
        )
        params1 = json.loads(tool_calls[0]["parameters"])
        self.assertEqual(params1["title"], ["7.8.9"], "First tool title should match")
        self.assertEqual(
            params1["filename"], "doc1", "First tool filename should be doc1"
        )

        # Verify second tool call
        self.assertEqual(
            tool_calls[1]["name"],
            "search_docs",
            "Second tool name should be search_docs",
        )
        params2 = json.loads(tool_calls[1]["parameters"])
        self.assertEqual(params2["title"], ["4.8"], "Second tool title should match")
        self.assertEqual(
            params2["filename"], "doc2", "Second tool filename should be doc2"
        )
```
**EN:** Test parsing two parallel tool calls where both have array parameters. This test exercises `test_parallel_tool_calls_with_array_parameters` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parsing two parallel tool calls where both have array parameters. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parallel_tool_calls_with_array_parameters`。

### Lines 136-159: test case simple parallel tool calls / 测试用例 simple parallel tool calls
```python
    def test_simple_parallel_tool_calls(self):
        """
        Test a simpler case of two parallel tool calls with array parameters.

        This is a minimal test case that still tests the core functionality.
        """
        chunks = [
            "[\n",
            '  {"name": "search_docs", "parameters": {"title": ["a"]}},',
            "\n",
            '  {"name": "search_docs", "parameters": {"title": ["b"]}}',
            "]",
        ]

        tool_calls = []

        for chunk in chunks:
            result = self.detector.parse_streaming_increment(chunk, self.tools)
            self._accumulate_tool_calls(tool_calls, result)

        # Should parse both tools successfully
        self.assertEqual(len(tool_calls), 2, "Should parse 2 tool calls")
        self.assertEqual(tool_calls[0]["name"], "search_docs")
        self.assertEqual(tool_calls[1]["name"], "search_docs")
```
**EN:** Test a simpler case of two parallel tool calls with array parameters. This test exercises `test_simple_parallel_tool_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test a simpler case of two parallel tool calls with array parameters. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_simple_parallel_tool_calls`。

### Lines 162-163: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestParallelToolCalls`: Test case for parallel tool call parsing with array parameters. / 用于组织相关测试、夹具或辅助方法。
- `TestParallelToolCalls.setUp`: Set up test tools and detector. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestParallelToolCalls._accumulate_tool_calls`: Helper method to accumulate tool call results from parsing output. / 该代码块实现 `_accumulate_tool_calls`，承担模块行为中的一个聚焦逻辑片段。
- `TestParallelToolCalls.test_parallel_tool_calls_with_array_parameters`: Test parsing two parallel tool calls where both have array parameters. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parallel_tool_calls_with_array_parameters`。
- `TestParallelToolCalls.test_simple_parallel_tool_calls`: Test a simpler case of two parallel tool calls with array parameters. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_simple_parallel_tool_calls`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.json_array_parser`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 163
