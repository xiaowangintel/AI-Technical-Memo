# test_deepseek_chat_templates.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_deepseek_chat_templates.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `deepseek chat templates` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual` 中的 `deepseek chat templates` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Scenario logic / 场景逻辑
```python
"""
Unit tests for DeepSeek chat template tool call handling.

Tests verify that the DeepSeek chat templates (v3, v3.1, v3.2) correctly handle
both dict and string types for tool['function']['arguments'] without double-escaping,
addressing issue #11700.
"""

import os
import unittest

from jinja2 import Template
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `templates`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 15-18: Class definition for TestDeepSeekChatTemplateToolCalls / 类定义
```python
class TestDeepSeekChatTemplateToolCalls(unittest.TestCase):
    """Test DeepSeek chat templates handle tool calls correctly."""

    @classmethod
```
**EN:** This range declares `TestDeepSeekChatTemplateToolCalls`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 19-36: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        """Load all DeepSeek chat templates."""
        base_path = os.path.join(
            os.path.dirname(__file__), "..", "..", "examples", "chat_template"
        )

        cls.templates = {}
        template_files = {
            "v3": "tool_chat_template_deepseekv3.jinja",
            "v3.1": "tool_chat_template_deepseekv31.jinja",
            "v3.2": "tool_chat_template_deepseekv32.jinja",
        }

        for version, filename in template_files.items():
            template_path = os.path.join(base_path, filename)
            with open(template_path, "r") as f:
                template_content = f.read()
            cls.templates[version] = Template(template_content)
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `join`, `dirname`, `items` and `read`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 38-39: Helper routines around _render_template / 辅助例程
```python
    def _render_template(
        self, version, messages, tools=None, add_generation_prompt=True
```
**EN:** This range implements helper routine(s) `_render_template` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 40-54: Scenario logic / 场景逻辑
```python
    ):
        """Helper method to render a template with given messages and tools."""
        template = self.templates[version]

        # Common template variables
        context = {
            "messages": messages,
            "add_generation_prompt": add_generation_prompt,
            "bos_token": "<｜begin▁of▁sentence｜>",
        }

        if tools is not None:
            context["tools"] = tools

        return template.render(**context)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `render`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 56-64: Test routines around test_tool_arguments_as_dict / 测试例程
```python
    def test_tool_arguments_as_dict(self):
        """Test that tool arguments as dict are properly JSON-encoded (normal case)."""
        # This tests the normal case where arguments come from OpenAI API as dict

        for version in ["v3", "v3.1", "v3.2"]:
            with self.subTest(version=version):
                messages = [
                    {"role": "user", "content": "What's the weather in NYC?"},
                    {
```
**EN:** This range defines concrete test routine(s) `test_tool_arguments_as_dict`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Representative call sites include `encoded` and `subTest`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 65-80: Scenario logic / 场景逻辑
```python
                        "role": "assistant",
                        "content": None,
                        "tool_calls": [
                            {
                                "type": "function",
                                "function": {
                                    "name": "get_weather",
                                    "arguments": {
                                        "city": "New York",
                                        "unit": "celsius",
                                    },  # Dict
                                },
                            }
                        ],
                    },
                ]
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 81-89: Scenario logic / 场景逻辑
```python

                tools = [
                    {
                        "type": "function",
                        "function": {
                            "name": "get_weather",
                            "description": "Get weather information",
                            "parameters": {
                                "type": "object",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 90-105: Assertions and result checks / 断言与结果检查
```python
                                "properties": {
                                    "city": {"type": "string"},
                                    "unit": {"type": "string"},
                                },
                            },
                        },
                    }
                ]

                output = self._render_template(version, messages, tools)

                # Should contain properly formatted JSON (not double-escaped)
                self.assertIn('"city"', output, f"{version}: Should contain city key")
                self.assertIn(
                    '"New York"', output, f"{version}: Should contain city value"
                )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `_render_template`, `JSON` and `assertIn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 106-113: Assertions and result checks / 断言与结果检查
```python

                # Should NOT contain double-escaped quotes
                self.assertNotIn(
                    '\\"city\\"', output, f"{version}: Should not double-escape"
                )
                self.assertNotIn(
                    '\\\\"', output, f"{version}: Should not have escaped backslashes"
                )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertNotIn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 115-139: Test routines around test_tool_arguments_as_string / 测试例程
```python
    def test_tool_arguments_as_string(self):
        """Test that tool arguments as string are used as-is (multi-round case)."""
        # This tests the multi-round function calling case from issue #11700
        # where arguments might already be JSON strings from previous model output

        for version in ["v3", "v3.1", "v3.2"]:
            with self.subTest(version=version):
                messages = [
                    {"role": "user", "content": "What's the stock price of NVDA?"},
                    {
                        "role": "assistant",
                        "content": None,
                        "tool_calls": [
                            {
                                "type": "function",
                                "function": {
                                    "name": "get_stock_info",
                                    "arguments": '{"symbol": "NVDA"}',  # Already a JSON string
                                },
                            }
                        ],
                    },
                ]

                tools = [
```
**EN:** This range defines concrete test routine(s) `test_tool_arguments_as_string`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `is` and `subTest`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 140-164: Assertions and result checks / 断言与结果检查
```python
                    {
                        "type": "function",
                        "function": {
                            "name": "get_stock_info",
                            "description": "Get stock information",
                            "parameters": {
                                "type": "object",
                                "properties": {
                                    "symbol": {"type": "string"},
                                },
                            },
                        },
                    }
                ]

                output = self._render_template(version, messages, tools)

                # Should contain the JSON string as-is
                self.assertIn(
                    '{"symbol": "NVDA"}',
                    output,
                    f"{version}: Should contain JSON as-is",
                )

                # Should NOT double-escape (the bug from issue #11700)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `_render_template`, `assertIn` and `escape`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 165-176: Assertions and result checks / 断言与结果检查
```python
                # Bad output would look like: "{\"symbol\": \"NVDA\"}" or "{\\"symbol\\": \\"NVDA\\"}"
                self.assertNotIn(
                    '{\\"symbol\\"', output, f"{version}: Should not double-escape"
                )
                self.assertNotIn(
                    '"{\\"symbol', output, f"{version}: Should not wrap and escape"
                )

                # Verify it's not triple-quoted or escaped
                self.assertNotIn(
                    '""{"', output, f"{version}: Should not have extra quotes"
                )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertNotIn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 178-189: Test routines around test_multiple_tool_calls_mixed_types / 测试例程
```python
    def test_multiple_tool_calls_mixed_types(self):
        """Test multiple tool calls with mixed dict and string argument types."""
        # This tests a complex scenario with multiple tools, some with dict args, some with string

        for version in ["v3", "v3.1", "v3.2"]:
            with self.subTest(version=version):
                messages = [
                    {"role": "user", "content": "Get weather and stock info"},
                    {
                        "role": "assistant",
                        "content": None,
                        "tool_calls": [
```
**EN:** This range defines concrete test routine(s) `test_multiple_tool_calls_mixed_types`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `subTest`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 190-202: Scenario logic / 场景逻辑
```python
                            {
                                "type": "function",
                                "function": {
                                    "name": "get_weather",
                                    "arguments": {"city": "Boston"},  # Dict
                                },
                            },
                            {
                                "type": "function",
                                "function": {
                                    "name": "get_stock_info",
                                    "arguments": '{"symbol": "TSLA"}',  # String
                                },
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 203-214: Scenario logic / 场景逻辑
```python
                            },
                        ],
                    },
                ]

                tools = [
                    {
                        "type": "function",
                        "function": {
                            "name": "get_weather",
                            "description": "Get weather",
                            "parameters": {
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 215-227: Scenario logic / 场景逻辑
```python
                                "type": "object",
                                "properties": {"city": {"type": "string"}},
                            },
                        },
                    },
                    {
                        "type": "function",
                        "function": {
                            "name": "get_stock_info",
                            "description": "Get stock info",
                            "parameters": {
                                "type": "object",
                                "properties": {"symbol": {"type": "string"}},
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 228-239: Assertions and result checks / 断言与结果检查
```python
                            },
                        },
                    },
                ]

                output = self._render_template(version, messages, tools)

                # First tool (dict) should be properly JSON-encoded
                self.assertIn(
                    '"city"', output, f"{version}: First tool should have city key"
                )
                self.assertIn(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `_render_template`, `tool` and `assertIn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 240-252: Assertions and result checks / 断言与结果检查
```python
                    '"Boston"',
                    output,
                    f"{version}: First tool should have Boston value",
                )

                # Second tool (string) should be used as-is
                self.assertIn(
                    '{"symbol": "TSLA"}',
                    output,
                    f"{version}: Second tool should use string as-is",
                )

                # Neither should be double-escaped
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `tool` and `assertIn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 253-262: Assertions and result checks / 断言与结果检查
```python
                self.assertNotIn(
                    '\\"city\\"',
                    output,
                    f"{version}: First tool should not double-escape",
                )
                self.assertNotIn(
                    '\\"symbol\\"',
                    output,
                    f"{version}: Second tool should not double-escape",
                )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertNotIn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 264-264: Test routines around test_tool_call_with_content / 测试例程
```python
    def test_tool_call_with_content(self):
```
**EN:** This range defines concrete test routine(s) `test_tool_call_with_content`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 265-288: Scenario logic / 场景逻辑
```python
        """Test tool calls that also include content text."""
        # Some models include explanatory text along with tool calls

        for version in ["v3", "v3.1", "v3.2"]:
            with self.subTest(version=version):
                messages = [
                    {"role": "user", "content": "What's the weather?"},
                    {
                        "role": "assistant",
                        "content": "Let me check the weather for you.",
                        "tool_calls": [
                            {
                                "type": "function",
                                "function": {
                                    "name": "get_weather",
                                    "arguments": {"city": "Seattle"},
                                },
                            }
                        ],
                    },
                ]

                tools = [
                    {
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `subTest`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 289-289: Scenario logic / 场景逻辑
```python
                        "type": "function",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 290-313: Assertions and result checks / 断言与结果检查
```python
                        "function": {
                            "name": "get_weather",
                            "description": "Get weather",
                            "parameters": {
                                "type": "object",
                                "properties": {"city": {"type": "string"}},
                            },
                        },
                    }
                ]

                output = self._render_template(version, messages, tools)

                # Should contain both the content and the tool call
                self.assertIn(
                    "Let me check the weather",
                    output,
                    f"{version}: Should include content",
                )
                self.assertIn(
                    '"city"', output, f"{version}: Should include tool arguments"
                )
                self.assertNotIn(
                    '\\"city\\"', output, f"{version}: Should not double-escape"
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `_render_template`, `assertIn` and `assertNotIn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 314-314: Scenario logic / 场景逻辑
```python
                )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 315-318: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- HTTP/API interaction / HTTP/API 交互
- Token-level inspection / Token 级分析
- Tool calling workflow / 工具调用流程
- Structured outputs / 结构化输出

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `os`, `unittest`
- **Third-party / 第三方库**: `jinja2`
- **Project Modules / 项目模块**: None / 无
