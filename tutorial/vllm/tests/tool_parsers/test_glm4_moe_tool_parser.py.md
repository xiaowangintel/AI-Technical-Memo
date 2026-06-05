# test_glm4_moe_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/tool_parsers/test_glm4_moe_tool_parser.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Glm4 MoE Tool Parser behavior in the Tool Parsers test area through focused pytest scenarios. It focuses on scenarios such as Glm4 MoE Tokenizer, Sample Tools, Glm4 MoE Tool Parser. / 该文件在 Tool Parsers 测试域中，通过有针对性的 pytest 场景验证 Glm4 MoE Tool Parser 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-21)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import json
from unittest.mock import Mock

import pytest

from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionRequest,
    ChatCompletionToolsParam,
    FunctionDefinition,
)
from vllm.entrypoints.openai.engine.protocol import FunctionCall, ToolCall
from vllm.tokenizers import get_tokenizer
from vllm.tool_parsers.glm4_moe_tool_parser import (
    Glm4MoeModelToolParser,
)

# Use a common model that is likely to be available
MODEL = "zai-org/GLM-4.5"
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `json`, `unittest.mock`, `pytest`, `vllm.entrypoints.openai.chat_completion.protocol`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Fixture: glm4_moe_tokenizer (lines 24-26)
```python
@pytest.fixture(scope="module")
def glm4_moe_tokenizer():
    return get_tokenizer(tokenizer_name=MODEL)
```
**EN:** Provides a pytest fixture for Glm4 MoE Tokenizer. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `pytest.fixture`, `get_tokenizer`.
**CN:** 该代码块定义 pytest 夹具 `glm4_moe_tokenizer`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `get_tokenizer` 构造或返回测试所需的值。

### Fixture: sample_tools (lines 29-38)
```python
@pytest.fixture
def sample_tools():
    return [
        ChatCompletionToolsParam(
            function=FunctionDefinition(
                name="get_weather",
                parameters={"city": {"type": "string"}},
            ),
        ),
    ]
```
**EN:** Provides a pytest fixture for Sample Tools. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `ChatCompletionToolsParam`, `FunctionDefinition`.
**CN:** 该代码块定义 pytest 夹具 `sample_tools`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `ChatCompletionToolsParam`, `FunctionDefinition` 构造或返回测试所需的值。

### Helper: assert_tool_calls (lines 53-69)
```python
def assert_tool_calls(
    actual_tool_calls: list[ToolCall], expected_tool_calls: list[ToolCall]
):
    assert len(actual_tool_calls) == len(expected_tool_calls)

    for actual_tool_call, expected_tool_call in zip(
        actual_tool_calls, expected_tool_calls
    ):
        assert isinstance(actual_tool_call.id, str)
        assert len(actual_tool_call.id) > 0

        assert actual_tool_call.type == "function"
        assert actual_tool_call.function.name == expected_tool_call.function.name
        # Compare arguments as JSON objects to handle formatting differences
        actual_args = json.loads(actual_tool_call.function.arguments)
        expected_args = json.loads(expected_tool_call.function.arguments)
        assert actual_args == expected_args
```
**EN:** Implements a reusable helper for Tool Calls, reducing duplication across related tests. It coordinates operations such as `zip`, `len`, `isinstance`.
**CN:** 该辅助函数为 Tool Calls 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `zip`, `len`, `isinstance` 等操作。

### Test: test_extract_tool_calls_no_tools (lines 72-79)
```python
def test_extract_tool_calls_no_tools(glm4_moe_tool_parser, mock_request):
    model_output = "This is a test"
    extracted_tool_calls = glm4_moe_tool_parser.extract_tool_calls(
        model_output, request=mock_request
    )  # type: ignore[arg-type]
    assert not extracted_tool_calls.tools_called
    assert extracted_tool_calls.tool_calls == []
    assert extracted_tool_calls.content == model_output
```
**EN:** Checks Extract Tool Calls No Tools under a focused test scenario. The body exercises logic via `glm4_moe_tool_parser.extract_tool_calls` before asserting the expected outcome.
**CN:** 该测试用例验证 Extract Tool Calls No Tools 在特定场景下的行为。 函数体会先通过 `glm4_moe_tool_parser.extract_tool_calls` 驱动目标逻辑，再断言预期结果。

### Test: test_extract_tool_calls (lines 82-249)
```python
@pytest.mark.parametrize(
    ids=[
        "single_tool_call",
        "multiple_tool_calls",
        "tool_call_with_content_before",
        "tool_call_with_mixed_args",
        "tool_call_with_chinese_content",
    ],
    argnames=["model_output", "expected_tool_calls", "expected_content"],
    argvalues=[
        (
            """<tool_call>get_current_weather
    <arg_key>city</arg_key>
    <arg_value>Dallas</arg_value>
    <arg_key>state</arg_key>
    <arg_value>TX</arg_value>
    <arg_key>unit</arg_key>
    <arg_value>fahrenheit</arg_value>
    </tool_call>""",
# ... omitted for brevity ...
def test_extract_tool_calls(
    glm4_moe_tool_parser,
    mock_request,
    model_output,
    expected_tool_calls,
    expected_content,
):
    extracted_tool_calls = glm4_moe_tool_parser.extract_tool_calls(
        model_output, request=mock_request
    )  # type: ignore[arg-type]
    assert extracted_tool_calls.tools_called
    assert_tool_calls(extracted_tool_calls.tool_calls, expected_tool_calls)

    assert extracted_tool_calls.content == expected_content
```
**EN:** Checks Extract Tool Calls under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `glm4_moe_tool_parser.extract_tool_calls`, `assert_tool_calls` before asserting the expected outcome.
**CN:** 该测试用例验证 Extract Tool Calls 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `glm4_moe_tool_parser.extract_tool_calls`, `assert_tool_calls` 驱动目标逻辑，再断言预期结果。

### Test: test_extract_tool_calls_with_thinking_tags (lines 252-276)
```python
def test_extract_tool_calls_with_thinking_tags(glm4_moe_tool_parser, mock_request):
    """Test tool extraction when thinking tags are present."""
    model_output = """<think>I want to get the weather.</think>

I will help you get the weather.
<tool_call>get_weather
<arg_key>city</arg_key>
<arg_value>Beijing</arg_value>
<arg_key>date</arg_key>
<arg_value>2025-08-01</arg_value>
</tool_call>"""

    extracted_tool_calls = glm4_moe_tool_parser.extract_tool_calls(
        model_output, request=mock_request
    )  # type: ignore[arg-type]

    assert extracted_tool_calls.tools_called
    assert len(extracted_tool_calls.tool_calls) == 1
    assert extracted_tool_calls.tool_calls[0].function.name == "get_weather"

    expected_content = """<think>I want to get the weather.</think>

I will help you get the weather.
"""
    assert extracted_tool_calls.content == expected_content
```
**EN:** Test tool extraction when thinking tags are present. The body exercises logic via `glm4_moe_tool_parser.extract_tool_calls`, `len` before asserting the expected outcome.
**CN:** 该测试用例验证 Extract Tool Calls With Thinking Tags 在特定场景下的行为。 函数体会先通过 `glm4_moe_tool_parser.extract_tool_calls`, `len` 驱动目标逻辑，再断言预期结果。

### Test: test_extract_tool_calls_malformed_xml (lines 279-296)
```python
def test_extract_tool_calls_malformed_xml(glm4_moe_tool_parser, mock_request):
    """Test that malformed XML is handled gracefully."""
    model_output = """<tool_call>get_weather
<arg_key>city</arg_key>
<arg_value>Seattle</arg_value>
<arg_key>incomplete_arg
<arg_value>value</arg_value>
</tool_call>"""

    extracted_tool_calls = glm4_moe_tool_parser.extract_tool_calls(
        model_output, request=mock_request
    )  # type: ignore[arg-type]

    # Should handle malformed XML gracefully
    # The parser should either extract what it can or return no tool calls
    # depending on how robust we want the parsing to be
    assert isinstance(extracted_tool_calls.tools_called, bool)
    assert isinstance(extracted_tool_calls.tool_calls, list)
```
**EN:** Test that malformed XML is handled gracefully. The body exercises logic via `glm4_moe_tool_parser.extract_tool_calls`, `isinstance` before asserting the expected outcome.
**CN:** 该测试用例验证 Extract Tool Calls Malformed Xml 在特定场景下的行为。 函数体会先通过 `glm4_moe_tool_parser.extract_tool_calls`, `isinstance` 驱动目标逻辑，再断言预期结果。

### Test: test_extract_tool_calls_empty_arguments (lines 299-312)
```python
def test_extract_tool_calls_empty_arguments(glm4_moe_tool_parser, mock_request):
    """Test tool calls with no arguments."""
    model_output = """<tool_call>get_current_time
</tool_call>"""

    extracted_tool_calls = glm4_moe_tool_parser.extract_tool_calls(
        model_output, request=mock_request
    )  # type: ignore[arg-type]

    assert extracted_tool_calls.tools_called
    assert len(extracted_tool_calls.tool_calls) == 1
    assert extracted_tool_calls.tool_calls[0].function.name == "get_current_time"
    # Empty arguments should result in empty JSON object
    assert extracted_tool_calls.tool_calls[0].function.arguments == "{}"
```
**EN:** Test tool calls with no arguments. The body exercises logic via `glm4_moe_tool_parser.extract_tool_calls`, `len` before asserting the expected outcome.
**CN:** 该测试用例验证 Extract Tool Calls Empty Arguments 在特定场景下的行为。 函数体会先通过 `glm4_moe_tool_parser.extract_tool_calls`, `len` 驱动目标逻辑，再断言预期结果。

### Test: test_extract_tool_calls_mixed_content (lines 315-355)
```python
def test_extract_tool_calls_mixed_content(glm4_moe_tool_parser, mock_request):
    """Test extraction with mixed content and multiple tool calls."""
    model_output = """I will help you get the weather info.

<tool_call>get_weather
<arg_key>city</arg_key>
<arg_value>Beijing</arg_value>
<arg_key>date</arg_key>
<arg_value>2025-08-01</arg_value>
</tool_call>

meaningwhile, I will also check the weather in Shanghai.

<tool_call>get_weather
<arg_key>city</arg_key>
<arg_value>Shanghai</arg_value>
<arg_key>date</arg_key>
<arg_value>2025-08-01</arg_value>
</tool_call>"""
# ... omitted for brevity ...
    # Check first tool call
    assert extracted_tool_calls.tool_calls[0].function.name == "get_weather"
    args1 = json.loads(extracted_tool_calls.tool_calls[0].function.arguments)
    assert args1["city"] == "Beijing"
    assert args1["date"] == "2025-08-01"

    # Check second tool call
    assert extracted_tool_calls.tool_calls[1].function.name == "get_weather"
    args2 = json.loads(extracted_tool_calls.tool_calls[1].function.arguments)
    assert args2["city"] == "Shanghai"
    assert args2["date"] == "2025-08-01"

    # Content should be everything before the first tool call
    assert extracted_tool_calls.content == "I will help you get the weather info.\n\n"
```
**EN:** Test extraction with mixed content and multiple tool calls. The body exercises logic via `glm4_moe_tool_parser.extract_tool_calls`, `json.loads`, `len` before asserting the expected outcome.
**CN:** 该测试用例验证 Extract Tool Calls Mixed Content 在特定场景下的行为。 函数体会先通过 `glm4_moe_tool_parser.extract_tool_calls`, `json.loads`, `len` 驱动目标逻辑，再断言预期结果。

### Helper: _reset_streaming_state (lines 468-475)
```python
def _reset_streaming_state(parser):
    """Helper to reset parser streaming state."""
    parser.current_tool_name_sent = False
    parser.prev_tool_call_arr = []
    parser.current_tool_id = -1
    parser.streamed_args_for_tool = []
    parser._tool_call_ids = []
    parser._sent_content_idx = 0
```
**EN:** Helper to reset parser streaming state.
**CN:** 该辅助函数为 Reset Streaming State 提供可复用逻辑，用于减少相关测试之间的重复代码。

### Additional scenarios (summary)
```python
glm4_moe_tool_parser
mock_request
test_streaming_basic_functionality
test_streaming_no_tool_calls
test_streaming_with_content_before_tool_calls
test_extract_tool_calls_special_characters
test_extract_tool_calls_incomplete_tool_call
test_streaming_incremental_string_value
test_streaming_empty_tool_call
test_streaming_prev_tool_call_arr_updates
test_streaming_multiple_tool_calls_sequential
test_streaming_json_escape_in_string
test_streaming_long_content_incremental
test_extract_tool_calls_numeric_deserialization
test_whitespace_preserved_in_arg_values
```
**EN:** The file contains more helper or test blocks than can be shown compactly here. This summary lists additional scenarios that continue the same validation strategy.
**CN:** 该文件包含的辅助逻辑或测试场景较多，无法在此完整展开；这里列出其余场景名称，便于快速把握覆盖范围。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **Mocking and patching / 模拟与补丁**
  - **EN:** Several scenarios replace dependencies or environment state so the tests can isolate one behavior at a time.
  - **CN:** 多个场景会替换依赖或环境状态，从而把验证范围限定在单一行为上。
- **Tokenization / 分词与反分词**
  - **EN:** The logic checks how text and token IDs are converted, streamed, or reconstructed.
  - **CN:** 逻辑重点检查文本与 token ID 的转换、流式处理和重建过程。
- **Tool-call parsing / 工具调用解析**
  - **EN:** The scenarios verify extraction of structured tool calls from model outputs and streaming deltas.
  - **CN:** 这些场景验证如何从模型输出和流式增量中提取结构化工具调用。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest.mock`
- **Third-party / 第三方依赖**: `pytest`
- **vLLM internal / vLLM 内部依赖**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.tokenizers`, `vllm.tool_parsers.glm4_moe_tool_parser`
