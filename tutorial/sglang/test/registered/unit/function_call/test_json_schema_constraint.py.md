# test_json_schema_constraint.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/function_call/test_json_schema_constraint.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates json schema constraint behavior in SGLang's unit / function call area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 函数调用 领域中与 json schema constraint 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: supporting statements / 辅助语句
```python
"""
Tests for JSON schema constraint functionality used by JsonArrayParser
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 5-19: module imports and dependencies / 模块导入与依赖
```python
import unittest

import jsonschema

from sglang.srt.entrypoints.openai.protocol import (
    Function,
    Tool,
    ToolChoice,
    ToolChoiceFuncName,
)
from sglang.srt.function_call.utils import (
    _get_tool_schema_defs,
    get_json_schema_constraint,
)
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `jsonschema`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `jsonschema`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.utils`。

### Lines 21-21: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(5, "base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 24-24: class TestJsonSchemaConstraint declaration / 类 TestJsonSchemaConstraint 声明
```python
class TestJsonSchemaConstraint(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 25-25: supporting statements / 辅助语句
```python
    """Test JSON schema constraint generation for tool choices"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 27-69: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Set up test tools"""
        self.tools = [
            Tool(
                type="function",
                function=Function(
                    name="get_weather",
                    description="Get weather information",
                    parameters={
                        "type": "object",
                        "properties": {
                            "location": {
                                "type": "string",
                                "description": "Location to get weather for",
                            },
                            "unit": {
                                "type": "string",
                                "enum": ["celsius", "fahrenheit"],
                                "description": "Temperature unit",
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
```
**EN:** Set up test tools This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Set up test tools 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 71-91: test case required tool choice schema / 测试用例 required tool choice schema
```python
    def test_required_tool_choice_schema(self):
        """Test schema generation for tool_choice='required'"""
        schema = get_json_schema_constraint(self.tools, "required")

        self.assertIsNotNone(schema)
        jsonschema.Draft202012Validator.check_schema(schema)

        self.assertEqual(schema["type"], "array")
        self.assertEqual(schema["minItems"], 1)
        self.assertIn("items", schema)
        self.assertIn("anyOf", schema["items"])

        # Should have schemas for both tools
        self.assertEqual(len(schema["items"]["anyOf"]), 2)

        # Check that each tool schema is present
        tool_names = [
            item["properties"]["name"]["enum"][0] for item in schema["items"]["anyOf"]
        ]
        self.assertIn("get_weather", tool_names)
        self.assertIn("search", tool_names)
```
**EN:** Test schema generation for tool_choice='required' This test exercises `test_required_tool_choice_schema` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test schema generation for tool_choice='required' 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_required_tool_choice_schema`。

### Lines 93-110: test case specific tool choice schema / 测试用例 specific tool choice schema
```python
    def test_specific_tool_choice_schema(self):
        """Test schema generation for specific tool choice"""
        tool_choice = ToolChoice(
            type="function", function=ToolChoiceFuncName(name="get_weather")
        )
        schema = get_json_schema_constraint(self.tools, tool_choice)

        self.assertIsNotNone(schema)
        jsonschema.Draft202012Validator.check_schema(schema)

        self.assertEqual(schema["type"], "array")
        self.assertEqual(schema["minItems"], 1)
        self.assertNotIn("maxItems", schema)

        # Should only have schema for the specific tool
        item_schema = schema["items"]
        self.assertEqual(item_schema["properties"]["name"]["enum"], ["get_weather"])
        self.assertIn("parameters", item_schema["properties"])
```
**EN:** Test schema generation for specific tool choice This test exercises `test_specific_tool_choice_schema` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test schema generation for specific tool choice 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_specific_tool_choice_schema`。

### Lines 112-129: test case specific tool choice dict schema / 测试用例 specific tool choice dict schema
```python
    def test_specific_tool_choice_dict_schema(self):
        """Test schema generation for specific tool choice as ToolChoice object"""
        tool_choice = ToolChoice(
            type="function", function=ToolChoiceFuncName(name="search")
        )
        schema = get_json_schema_constraint(self.tools, tool_choice)

        self.assertIsNotNone(schema)
        jsonschema.Draft202012Validator.check_schema(schema)

        self.assertEqual(schema["type"], "array")
        self.assertEqual(schema["minItems"], 1)
        self.assertNotIn("maxItems", schema)

        # Should only have schema for the specific tool
        item_schema = schema["items"]
        self.assertEqual(item_schema["properties"]["name"]["enum"], ["search"])
        self.assertIn("parameters", item_schema["properties"])
```
**EN:** Test schema generation for specific tool choice as ToolChoice object This test exercises `test_specific_tool_choice_dict_schema` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test schema generation for specific tool choice as ToolChoice object 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_specific_tool_choice_dict_schema`。

### Lines 131-154: test case specific tool choice allows multiple calls / 测试用例 specific tool choice allows multiple calls
```python
    def test_specific_tool_choice_allows_multiple_calls(self):
        """Test that specific tool choice schema allows multiple calls.

        Regression test for https://github.com/sgl-project/sglang/issues/17998:
        maxItems: 1 caused the model to stall on whitespace when the prompt
        implied multiple calls to the same function.
        """
        tool_choice = ToolChoice(
            type="function", function=ToolChoiceFuncName(name="get_weather")
        )
        schema = get_json_schema_constraint(self.tools, tool_choice)

        single_call = [
            {"name": "get_weather", "parameters": {"location": "NYC"}},
        ]
        multi_call = [
            {"name": "get_weather", "parameters": {"location": "NYC"}},
            {"name": "get_weather", "parameters": {"location": "LA"}},
            {"name": "get_weather", "parameters": {"location": "Chicago"}},
        ]

        validator = jsonschema.Draft202012Validator(schema)
        validator.validate(single_call)
        validator.validate(multi_call)
```
**EN:** Test that specific tool choice schema allows multiple calls. This test exercises `test_specific_tool_choice_allows_multiple_calls` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that specific tool choice schema allows multiple calls. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_specific_tool_choice_allows_multiple_calls`。

### Lines 156-179: test case specific tool choice no parallel / 测试用例 specific tool choice no parallel
```python
    def test_specific_tool_choice_no_parallel(self):
        """Test that parallel_tool_calls=False sets maxItems=1"""
        tool_choice = ToolChoice(
            type="function", function=ToolChoiceFuncName(name="get_weather")
        )
        schema = get_json_schema_constraint(
            self.tools, tool_choice, parallel_tool_calls=False
        )

        self.assertIsNotNone(schema)
        self.assertEqual(schema["maxItems"], 1)

        single_call = [
            {"name": "get_weather", "parameters": {"location": "NYC"}},
        ]
        multi_call = [
            {"name": "get_weather", "parameters": {"location": "NYC"}},
            {"name": "get_weather", "parameters": {"location": "LA"}},
        ]

        validator = jsonschema.Draft202012Validator(schema)
        validator.validate(single_call)
        with self.assertRaises(jsonschema.ValidationError):
            validator.validate(multi_call)
```
**EN:** Test that parallel_tool_calls=False sets maxItems=1 This test exercises `test_specific_tool_choice_no_parallel` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that parallel_tool_calls=False sets maxItems=1 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_specific_tool_choice_no_parallel`。

### Lines 181-188: test case required tool choice no parallel / 测试用例 required tool choice no parallel
```python
    def test_required_tool_choice_no_parallel(self):
        """Test that required + parallel_tool_calls=False sets maxItems=1"""
        schema = get_json_schema_constraint(
            self.tools, "required", parallel_tool_calls=False
        )

        self.assertIsNotNone(schema)
        self.assertEqual(schema["maxItems"], 1)
```
**EN:** Test that required + parallel_tool_calls=False sets maxItems=1 This test exercises `test_required_tool_choice_no_parallel` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that required + parallel_tool_calls=False sets maxItems=1 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_required_tool_choice_no_parallel`。

### Lines 190-197: test case nonexistent tool choice / 测试用例 nonexistent tool choice
```python
    def test_nonexistent_tool_choice(self):
        """Test schema generation for nonexistent tool"""
        tool_choice = ToolChoice(
            type="function", function=ToolChoiceFuncName(name="nonexistent")
        )
        schema = get_json_schema_constraint(self.tools, tool_choice)

        self.assertIsNone(schema)
```
**EN:** Test schema generation for nonexistent tool This test exercises `test_nonexistent_tool_choice` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test schema generation for nonexistent tool 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_nonexistent_tool_choice`。

### Lines 199-204: test case nonexistent tool choice dict / 测试用例 nonexistent tool choice dict
```python
    def test_nonexistent_tool_choice_dict(self):
        """Test schema generation for nonexistent tool as dict"""
        tool_choice = {"type": "function", "function": {"name": "nonexistent"}}
        schema = get_json_schema_constraint(self.tools, tool_choice)

        self.assertIsNone(schema)
```
**EN:** Test schema generation for nonexistent tool as dict This test exercises `test_nonexistent_tool_choice_dict` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test schema generation for nonexistent tool as dict 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_nonexistent_tool_choice_dict`。

### Lines 206-210: test case auto tool choice schema / 测试用例 auto tool choice schema
```python
    def test_auto_tool_choice_schema(self):
        """Test schema generation for tool_choice='auto'"""
        schema = get_json_schema_constraint(self.tools, "auto")

        self.assertIsNone(schema)
```
**EN:** Test schema generation for tool_choice='auto' This test exercises `test_auto_tool_choice_schema` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test schema generation for tool_choice='auto' 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_auto_tool_choice_schema`。

### Lines 212-216: test case none tool choice schema / 测试用例 none tool choice schema
```python
    def test_none_tool_choice_schema(self):
        """Test schema generation for tool_choice=None"""
        schema = get_json_schema_constraint(self.tools, None)

        self.assertIsNone(schema)
```
**EN:** Test schema generation for tool_choice=None This test exercises `test_none_tool_choice_schema` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test schema generation for tool_choice=None 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_none_tool_choice_schema`。

### Lines 218-260: test case tools with defs / 测试用例 tools with defs
```python
    def test_tools_with_defs(self):
        """Test schema generation with tools that have $defs"""
        tools_with_defs = [
            Tool(
                type="function",
                function=Function(
                    name="complex_tool",
                    description="Tool with complex schema",
                    parameters={
                        "type": "object",
                        "properties": {
                            "data": {
                                "type": "object",
                                "properties": {
                                    "nested": {"$ref": "#/$defs/NestedType"},
                                },
                            },
                        },
                        "$defs": {
                            "NestedType": {
                                "type": "object",
                                "properties": {
                                    "value": {"type": "string"},
                                },
                            },
                        },
                    },
                ),
            ),
        ]

        try:
            _get_tool_schema_defs(tools_with_defs)
        except ValueError as e:
            self.fail(f"Should not raise ValueError, but got: {e}")

        schema = get_json_schema_constraint(tools_with_defs, "required")

        self.assertIsNotNone(schema)
        jsonschema.Draft202012Validator.check_schema(schema)

        self.assertIn("$defs", schema)
        self.assertIn("NestedType", schema["$defs"])
```
**EN:** Test schema generation with tools that have $defs This test exercises `test_tools_with_defs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test schema generation with tools that have $defs 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tools_with_defs`。

### Lines 262-284: test case tools without parameters / 测试用例 tools without parameters
```python
    def test_tools_without_parameters(self):
        """Test schema generation with tools that have no parameters"""
        tools_without_params = [
            Tool(
                type="function",
                function=Function(
                    name="simple_tool",
                    description="Tool without parameters",
                    parameters=None,
                ),
            ),
        ]

        schema = get_json_schema_constraint(tools_without_params, "required")

        self.assertIsNotNone(schema)
        jsonschema.Draft202012Validator.check_schema(schema)

        item_schema = schema["items"]["anyOf"][0]
        self.assertEqual(
            item_schema["properties"]["parameters"],
            {"type": "object", "properties": {}},
        )
```
**EN:** Test schema generation with tools that have no parameters This test exercises `test_tools_without_parameters` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test schema generation with tools that have no parameters 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tools_without_parameters`。

### Lines 286-332: test case conflicting defs raises valueerror / 测试用例 conflicting defs raises valueerror
```python
    def test_conflicting_defs_raises_valueerror(self):
        """Test that conflicting tool definitions raise ValueError with proper message"""
        tools_with_conflicting_defs = [
            Tool(
                type="function",
                function=Function(
                    name="tool1",
                    description="Tool 1",
                    parameters={
                        "type": "object",
                        "properties": {},
                        "$defs": {
                            "ConflictingType": {
                                "type": "object",
                                "properties": {"value": {"type": "string"}},
                            },
                        },
                    },
                ),
            ),
            Tool(
                type="function",
                function=Function(
                    name="tool2",
                    description="Tool 2",
                    parameters={
                        "type": "object",
                        "properties": {},
                        "$defs": {
                            "ConflictingType": {
                                "type": "object",
                                "properties": {"value": {"type": "number"}},
                            },
                        },
                    },
                ),
            ),
        ]

        with self.assertRaises(ValueError) as context:
            _get_tool_schema_defs(tools_with_conflicting_defs)

        self.assertIn(
            "Tool definition 'ConflictingType' has multiple schemas",
            str(context.exception),
        )
        self.assertIn("which is not supported", str(context.exception))
```
**EN:** Test that conflicting tool definitions raise ValueError with proper message This test exercises `test_conflicting_defs_raises_valueerror` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that conflicting tool definitions raise ValueError with proper message 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_conflicting_defs_raises_valueerror`。

### Lines 334-364: test case tools with empty defs / 测试用例 tools with empty defs
```python
    def test_tools_with_empty_defs(self):
        """Test tools with empty $defs objects"""
        tools_with_empty_defs = [
            Tool(
                type="function",
                function=Function(
                    name="empty_defs_tool",
                    description="Tool with empty $defs",
                    parameters={
                        "type": "object",
                        "properties": {
                            "data": {"type": "string"},
                        },
                        "required": ["data"],
                        "$defs": {},
                    },
                ),
            ),
        ]

        try:
            _get_tool_schema_defs(tools_with_empty_defs)
        except ValueError as e:
            self.fail(f"Should not raise ValueError, but got: {e}")

        schema = get_json_schema_constraint(tools_with_empty_defs, "required")
        self.assertIsNotNone(schema)
        jsonschema.Draft202012Validator.check_schema(schema)

        # Should not have $defs section when empty
        self.assertNotIn("$defs", schema)
```
**EN:** Test tools with empty $defs objects This test exercises `test_tools_with_empty_defs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test tools with empty $defs objects 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tools_with_empty_defs`。

### Lines 366-440: test case tools with identical defs / 测试用例 tools with identical defs
```python
    def test_tools_with_identical_defs(self):
        """Test different tools with same $defs names but identical schemas (should not raise exception)"""
        tools_with_identical_defs = [
            Tool(
                type="function",
                function=Function(
                    name="weather_tool",
                    description="Get weather information",
                    parameters={
                        "type": "object",
                        "properties": {
                            "location": {"$ref": "#/$defs/Location"},
                        },
                        "required": ["location"],
                        "$defs": {
                            "Location": {
                                "type": "object",
                                "properties": {
                                    "lat": {"type": "number"},
                                    "lon": {"type": "number"},
                                },
                                "required": ["lat", "lon"],
                            },
                        },
                    },
                ),
            ),
            Tool(
                type="function",
                function=Function(
                    name="address_tool",
                    description="Get address information",
                    parameters={
                        "type": "object",
                        "properties": {
                            "address": {"$ref": "#/$defs/Location"},
                        },
                        "required": ["address"],
                        "$defs": {
                            "Location": {
                                "type": "object",
                                "properties": {
                                    "lat": {"type": "number"},
                                    "lon": {"type": "number"},
                                },
                                "required": ["lat", "lon"],
                            },
                        },
                    },
                ),
            ),
        ]

        try:
            _get_tool_schema_defs(tools_with_identical_defs)
        except ValueError as e:
            self.fail(
                f"Should not raise ValueError for identical schemas, but got: {e}"
            )

        # Also test that schema generation works
        schema = get_json_schema_constraint(tools_with_identical_defs, "required")
        self.assertIsNotNone(schema)
        jsonschema.Draft202012Validator.check_schema(schema)

        # Verify both tools are present
        tool_names = [
            item["properties"]["name"]["enum"][0] for item in schema["items"]["anyOf"]
        ]
        self.assertIn("weather_tool", tool_names)
        self.assertIn("address_tool", tool_names)

        # Should have $defs with Location
        self.assertIn("$defs", schema)
        self.assertIn("Location", schema["$defs"])
```
**EN:** Test different tools with same $defs names but identical schemas (should not raise exception) This test exercises `test_tools_with_identical_defs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test different tools with same $defs names but identical schemas (should not raise exception) 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tools_with_identical_defs`。

### Lines 442-503: test case tools with nested defs / 测试用例 tools with nested defs
```python
    def test_tools_with_nested_defs(self):
        """Test tools with nested $defs"""
        tools_with_nested_defs = [
            Tool(
                type="function",
                function=Function(
                    name="complex_tool",
                    description="Tool with nested $defs",
                    parameters={
                        "type": "object",
                        "properties": {
                            "user": {"$ref": "#/$defs/User"},
                            "settings": {"$ref": "#/$defs/Settings"},
                        },
                        "required": ["user"],
                        "$defs": {
                            "User": {
                                "type": "object",
                                "properties": {
                                    "id": {"type": "string"},
                                    "profile": {"$ref": "#/$defs/Profile"},
                                },
                                "required": ["id"],
                            },
                            "Profile": {
                                "type": "object",
                                "properties": {
                                    "name": {"type": "string"},
                                    "email": {"type": "string", "format": "email"},
                                },
                                "required": ["name"],
                            },
                            "Settings": {
                                "type": "object",
                                "properties": {
                                    "theme": {
                                        "type": "string",
                                        "enum": ["light", "dark"],
                                    },
                                    "notifications": {"type": "boolean"},
                                },
                            },
                        },
                    },
                ),
            ),
        ]

        try:
            _get_tool_schema_defs(tools_with_nested_defs)
        except ValueError as e:
            self.fail(f"Should not raise ValueError, but got: {e}")

        schema = get_json_schema_constraint(tools_with_nested_defs, "required")
        self.assertIsNotNone(schema)
        jsonschema.Draft202012Validator.check_schema(schema)

        # Verify all $defs are properly included
        self.assertIn("$defs", schema)
        self.assertIn("User", schema["$defs"])
        self.assertIn("Profile", schema["$defs"])
        self.assertIn("Settings", schema["$defs"])
```
**EN:** Test tools with nested $defs This test exercises `test_tools_with_nested_defs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test tools with nested $defs 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tools_with_nested_defs`。

### Lines 505-582: test case mixed tools with and without defs / 测试用例 mixed tools with and without defs
```python
    def test_mixed_tools_with_and_without_defs(self):
        """Test mixed tools with and without $defs"""
        mixed_tools = [
            Tool(
                type="function",
                function=Function(
                    name="simple_tool",
                    description="Simple tool without $defs",
                    parameters={
                        "type": "object",
                        "properties": {
                            "query": {"type": "string"},
                        },
                        "required": ["query"],
                    },
                ),
            ),
            Tool(
                type="function",
                function=Function(
                    name="complex_tool",
                    description="Complex tool with $defs",
                    parameters={
                        "type": "object",
                        "properties": {
                            "data": {"$ref": "#/$defs/DataType"},
                        },
                        "required": ["data"],
                        "$defs": {
                            "DataType": {
                                "type": "object",
                                "properties": {
                                    "value": {"type": "string"},
                                    "metadata": {"type": "object"},
                                },
                                "required": ["value"],
                            },
                        },
                    },
                ),
            ),
            Tool(
                type="function",
                function=Function(
                    name="another_simple_tool",
                    description="Another simple tool",
                    parameters={
                        "type": "object",
                        "properties": {
                            "id": {"type": "integer"},
                        },
                        "required": ["id"],
                    },
                ),
            ),
        ]

        try:
            _get_tool_schema_defs(mixed_tools)
        except ValueError as e:
            self.fail(f"Should not raise ValueError, but got: {e}")

        schema = get_json_schema_constraint(mixed_tools, "required")
        self.assertIsNotNone(schema)
        jsonschema.Draft202012Validator.check_schema(schema)

        # Should have $defs from the complex tool
        self.assertIn("$defs", schema)
        self.assertIn("DataType", schema["$defs"])

        # Should have all three tools
        tool_names = [
            item["properties"]["name"]["enum"][0] for item in schema["items"]["anyOf"]
        ]
        self.assertEqual(len(tool_names), 3)
        self.assertIn("simple_tool", tool_names)
        self.assertIn("complex_tool", tool_names)
        self.assertIn("another_simple_tool", tool_names)
```
**EN:** Test mixed tools with and without $defs This test exercises `test_mixed_tools_with_and_without_defs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test mixed tools with and without $defs 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mixed_tools_with_and_without_defs`。

### Lines 584-622: test case tools with defs but no refs / 测试用例 tools with defs but no refs
```python
    def test_tools_with_defs_but_no_refs(self):
        """Test tools with $defs but no $ref usage"""
        tools_with_unused_defs = [
            Tool(
                type="function",
                function=Function(
                    name="unused_defs_tool",
                    description="Tool with $defs but no $ref usage",
                    parameters={
                        "type": "object",
                        "properties": {
                            "data": {"type": "string"},
                        },
                        "required": ["data"],
                        "$defs": {
                            "UnusedType": {
                                "type": "object",
                                "properties": {
                                    "value": {"type": "string"},
                                },
                            },
                        },
                    },
                ),
            ),
        ]

        try:
            _get_tool_schema_defs(tools_with_unused_defs)
        except ValueError as e:
            self.fail(f"Should not raise ValueError, but got: {e}")

        schema = get_json_schema_constraint(tools_with_unused_defs, "required")
        self.assertIsNotNone(schema)
        jsonschema.Draft202012Validator.check_schema(schema)

        # Should still include $defs even if not referenced
        self.assertIn("$defs", schema)
        self.assertIn("UnusedType", schema["$defs"])
```
**EN:** Test tools with $defs but no $ref usage This test exercises `test_tools_with_defs_but_no_refs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test tools with $defs but no $ref usage 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tools_with_defs_but_no_refs`。

### Lines 625-626: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestJsonSchemaConstraint`: Test JSON schema constraint generation for tool choices / 用于组织相关测试、夹具或辅助方法。
- `TestJsonSchemaConstraint.setUp`: Set up test tools / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestJsonSchemaConstraint.test_required_tool_choice_schema`: Test schema generation for tool_choice='required' / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_required_tool_choice_schema`。
- `TestJsonSchemaConstraint.test_specific_tool_choice_schema`: Test schema generation for specific tool choice / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_specific_tool_choice_schema`。
- `TestJsonSchemaConstraint.test_specific_tool_choice_dict_schema`: Test schema generation for specific tool choice as ToolChoice object / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_specific_tool_choice_dict_schema`。
- `TestJsonSchemaConstraint.test_specific_tool_choice_allows_multiple_calls`: Test that specific tool choice schema allows multiple calls. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_specific_tool_choice_allows_multiple_calls`。
- `TestJsonSchemaConstraint.test_specific_tool_choice_no_parallel`: Test that parallel_tool_calls=False sets maxItems=1 / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_specific_tool_choice_no_parallel`。
- `TestJsonSchemaConstraint.test_required_tool_choice_no_parallel`: Test that required + parallel_tool_calls=False sets maxItems=1 / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_required_tool_choice_no_parallel`。
- `TestJsonSchemaConstraint.test_nonexistent_tool_choice`: Test schema generation for nonexistent tool / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_nonexistent_tool_choice`。
- `TestJsonSchemaConstraint.test_nonexistent_tool_choice_dict`: Test schema generation for nonexistent tool as dict / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_nonexistent_tool_choice_dict`。
- `TestJsonSchemaConstraint.test_auto_tool_choice_schema`: Test schema generation for tool_choice='auto' / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_auto_tool_choice_schema`。
- `TestJsonSchemaConstraint.test_none_tool_choice_schema`: Test schema generation for tool_choice=None / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_none_tool_choice_schema`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `jsonschema`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.function_call.utils`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 626
