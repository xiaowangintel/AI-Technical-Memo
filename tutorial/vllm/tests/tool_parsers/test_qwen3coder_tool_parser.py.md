# test_qwen3coder_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/tool_parsers/test_qwen3coder_tool_parser.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Qwen3coder Tool Parser behavior in the Tool Parsers test area through focused pytest scenarios. It focuses on scenarios such as Qwen3 Tokenizer, Qwen3 Tool Parser, Qwen3 Xml Tool Parser. / 该文件在 Tool Parsers 测试域中，通过有针对性的 pytest 场景验证 Qwen3coder Tool Parser 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-29)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import json
from collections.abc import Generator

import pytest
from openai.types.responses.function_tool import FunctionTool
from xgrammar import StructuralTag

from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionNamedFunction,
    ChatCompletionNamedToolChoiceParam,
    ChatCompletionRequest,
    ChatCompletionToolsParam,
)
from vllm.entrypoints.openai.engine.protocol import (
    DeltaMessage,
    FunctionCall,
    ToolCall,
)
from vllm.tokenizers import TokenizerLike, get_tokenizer
from vllm.tokenizers.detokenizer_utils import detokenize_incrementally
from vllm.tool_parsers.qwen3coder_tool_parser import (
    Qwen3CoderToolParser,
)
from vllm.tool_parsers.qwen3xml_tool_parser import Qwen3XMLToolParser

MODEL = "Qwen/Qwen3-Coder-30B-A3B-Instruct-FP8"
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `json`, `collections.abc`, `pytest`, `openai.types.responses.function_tool`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Fixture: qwen3_tokenizer (lines 32-34)
```python
@pytest.fixture(scope="module")
def qwen3_tokenizer():
    return get_tokenizer(tokenizer_name=MODEL)
```
**EN:** Provides a pytest fixture for Qwen3 Tokenizer. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `pytest.fixture`, `get_tokenizer`.
**CN:** 该代码块定义 pytest 夹具 `qwen3_tokenizer`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `get_tokenizer` 构造或返回测试所需的值。

### Fixture: qwen3_tool_parser (lines 37-39)
```python
@pytest.fixture
def qwen3_tool_parser(qwen3_tokenizer, sample_tools):
    return Qwen3CoderToolParser(qwen3_tokenizer, tools=sample_tools)
```
**EN:** Provides a pytest fixture for Qwen3 Tool Parser. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `Qwen3CoderToolParser`.
**CN:** 该代码块定义 pytest 夹具 `qwen3_tool_parser`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `Qwen3CoderToolParser` 构造或返回测试所需的值。

### Constants / assignments (lines 56-64)
```python
WEATHER_PARAMS = {
    "type": "object",
    "properties": {
        "city": {"type": "string", "description": "The city name"},
        "state": {"type": "string", "description": "The state code"},
        "unit": {"type": "string", "enum": ["fahrenheit", "celsius"]},
    },
    "required": ["city", "state"],
}
```
**EN:** Defines shared constants or configuration objects like `WEATHER_PARAMS`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `WEATHER_PARAMS`），供后续测试重复使用。

### Helper: _as_chat_completion_tools (lines 114-132)
```python
def _as_chat_completion_tools(
    tools: list[ChatCompletionToolsParam | FunctionTool],
) -> list[ChatCompletionToolsParam]:
    normalized: list[ChatCompletionToolsParam] = []
    for tool in tools:
        if isinstance(tool, ChatCompletionToolsParam):
            normalized.append(tool)
        else:
            normalized.append(
                ChatCompletionToolsParam(
                    type="function",
                    function={
                        "name": tool.name,
                        "description": tool.description,
                        "parameters": tool.parameters,
                    },
                )
            )
    return normalized
```
**EN:** Implements a reusable helper for As Chat Completion Tools, reducing duplication across related tests. It coordinates operations such as `isinstance`, `normalized.append`, `ChatCompletionToolsParam`.
**CN:** 该辅助函数为 As Chat Completion Tools 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `isinstance`, `normalized.append`, `ChatCompletionToolsParam` 等操作。

### Helper: assert_tool_calls (lines 135-148)
```python
def assert_tool_calls(
    actual_tool_calls: list[ToolCall], expected_tool_calls: list[ToolCall]
):
    assert len(actual_tool_calls) == len(expected_tool_calls)

    for actual_tool_call, expected_tool_call in zip(
        actual_tool_calls, expected_tool_calls
    ):
        # Qwen3 parser doesn't generate IDs during extraction
        assert actual_tool_call.type == "function"
        assert actual_tool_call.function.name == expected_tool_call.function.name
        assert json.loads(actual_tool_call.function.arguments) == json.loads(
            expected_tool_call.function.arguments
        )
```
**EN:** Implements a reusable helper for Tool Calls, reducing duplication across related tests. It coordinates operations such as `zip`, `len`, `json.loads`.
**CN:** 该辅助函数为 Tool Calls 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `zip`, `len`, `json.loads` 等操作。

### Test: test_extract_tool_calls_no_tools (lines 202-209)
```python
def test_extract_tool_calls_no_tools(qwen3_tool_parser_parametrized):
    model_output = "This is a test response without any tool calls"
    extracted_tool_calls = qwen3_tool_parser_parametrized.extract_tool_calls(
        model_output, request=None
    )  # type: ignore[arg-type]
    assert not extracted_tool_calls.tools_called
    assert extracted_tool_calls.tool_calls == []
    assert extracted_tool_calls.content == model_output
```
**EN:** Checks Extract Tool Calls No Tools under a focused test scenario. The body exercises logic via `qwen3_tool_parser_parametrized.extract_tool_calls` before asserting the expected outcome.
**CN:** 该测试用例验证 Extract Tool Calls No Tools 在特定场景下的行为。 函数体会先通过 `qwen3_tool_parser_parametrized.extract_tool_calls` 驱动目标逻辑，再断言预期结果。

### Test: test_extract_tool_calls (lines 212-398)
```python
@pytest.mark.parametrize(
    ids=[
        "single_tool",
        "single_tool_with_content",
        "single_tool_multiline_param",
        "parallel_tools",
        "tool_with_typed_params",
    ],
    argnames=["model_output", "expected_tool_calls", "expected_content"],
    argvalues=[
        (
            """<tool_call>
<function=get_current_weather>
<parameter=city>
Dallas
</parameter>
<parameter=state>
TX
</parameter>
# ... omitted for brevity ...
    qwen3_tool_parser_parametrized,
    model_output,
    expected_tool_calls,
    expected_content,
):
    request = ChatCompletionRequest(model=MODEL, messages=[])
    extracted_tool_calls = qwen3_tool_parser_parametrized.extract_tool_calls(
        model_output, request=request
    )
    assert extracted_tool_calls.tools_called

    assert_tool_calls(extracted_tool_calls.tool_calls, expected_tool_calls)

    assert extracted_tool_calls.content == expected_content
```
**EN:** Checks Extract Tool Calls under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `ChatCompletionRequest`, `qwen3_tool_parser_parametrized.extract_tool_calls` before asserting the expected outcome.
**CN:** 该测试用例验证 Extract Tool Calls 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `ChatCompletionRequest`, `qwen3_tool_parser_parametrized.extract_tool_calls` 驱动目标逻辑，再断言预期结果。

### Test: test_extract_tool_calls_fallback_no_tags (lines 401-421)
```python
def test_extract_tool_calls_fallback_no_tags(
    qwen3_tool_parser_parametrized,
):
    """Test fallback parsing when XML tags are missing"""
    model_output = """<function=get_current_weather>
<parameter=city>
Dallas
</parameter>
<parameter=state>
TX
</parameter>
</function>"""

    request = ChatCompletionRequest(model=MODEL, messages=[])
    extracted_tool_calls = qwen3_tool_parser_parametrized.extract_tool_calls(
        model_output, request=request
    )

    assert extracted_tool_calls.tools_called
    assert len(extracted_tool_calls.tool_calls) == 1
    assert extracted_tool_calls.tool_calls[0].function.name == "get_current_weather"
```
**EN:** Test fallback parsing when XML tags are missing The body exercises logic via `ChatCompletionRequest`, `qwen3_tool_parser_parametrized.extract_tool_calls`, `len` before asserting the expected outcome.
**CN:** 该测试用例验证 Extract Tool Calls Fallback No Tags 在特定场景下的行为。 函数体会先通过 `ChatCompletionRequest`, `qwen3_tool_parser_parametrized.extract_tool_calls`, `len` 驱动目标逻辑，再断言预期结果。

### Test: test_extract_tool_calls_type_conversion (lines 424-474)
```python
def test_extract_tool_calls_type_conversion(qwen3_tokenizer):
    """Test parameter type conversion based on tool schema"""
    tools = [
        ChatCompletionToolsParam(
            type="function",
            function={
                "name": "test_types",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "int_param": {"type": "integer"},
                        "float_param": {"type": "float"},
                        "bool_param": {"type": "boolean"},
                        "str_param": {"type": "string"},
                        "obj_param": {"type": "object"},
                    },
                },
            },
        )
# ... omitted for brevity ...
</parameter>
</function>
</tool_call>"""

    parser = Qwen3XMLToolParser(qwen3_tokenizer, tools=tools)
    request = ChatCompletionRequest(model=MODEL, messages=[], tools=tools)
    extracted_tool_calls = parser.extract_tool_calls(model_output, request=request)

    args = json.loads(extracted_tool_calls.tool_calls[0].function.arguments)
    assert args["int_param"] == 42
    assert args["float_param"] == 3.14
    assert args["bool_param"] is True
    assert args["str_param"] == "hello world"
    assert args["obj_param"] == {"key": "value"}
```
**EN:** Test parameter type conversion based on tool schema The body exercises logic via `Qwen3XMLToolParser`, `ChatCompletionRequest`, `parser.extract_tool_calls` before asserting the expected outcome.
**CN:** 该测试用例验证 Extract Tool Calls Type Conversion 在特定场景下的行为。 函数体会先通过 `Qwen3XMLToolParser`, `ChatCompletionRequest`, `parser.extract_tool_calls` 驱动目标逻辑，再断言预期结果。

### Test: test_extract_tool_calls_streaming (lines 477-726)
```python
@pytest.mark.parametrize(
    ids=[
        "no_tools",
        "single_tool",
        "single_tool_with_content",
        "single_tool_multiline_param",
        "parallel_tools",
        "tool_with_typed_params",  # Added this test case
    ],
    argnames=["model_output", "expected_tool_calls", "expected_content"],
    argvalues=[
        ("This is a test without tools", [], "This is a test without tools"),
        (
            """<tool_call>
<function=get_current_weather>
<parameter=city>
Dallas
</parameter>
<parameter=state>
# ... omitted for brevity ...

    # Verify each tool call
    for idx, expected_tool in enumerate(expected_tool_calls):
        state = tool_states[idx]
        assert state["id"] is not None
        assert state["type"] == "function"
        assert state["name"] == expected_tool.function.name

        # Parse accumulated arguments
        arguments_str = state["arguments"]
        assert arguments_str is not None
        actual_args = json.loads(arguments_str)
        expected_args = json.loads(expected_tool.function.arguments)
        assert actual_args == expected_args
```
**EN:** Test incremental streaming behavior including typed parameters Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `ChatCompletionRequest`, `stream_delta_message_generator` before asserting the expected outcome.
**CN:** 该测试用例验证 Extract Tool Calls Streaming 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `ChatCompletionRequest`, `stream_delta_message_generator` 驱动目标逻辑，再断言预期结果。

### Test: test_extract_tool_calls_missing_closing_parameter_tag (lines 729-768)
```python
def test_extract_tool_calls_missing_closing_parameter_tag(
    qwen3_tool_parser_parametrized,
):
    """Test handling of missing closing </parameter> tag"""
    # Using get_current_weather from sample_tools but with malformed XML
    model_output = """Let me check the weather for you:
<tool_call>
<function=get_current_weather>
<parameter=city>
Dallas
<parameter=state>
TX
</parameter>
<parameter=unit>
fahrenheit
</parameter>
</function>
</tool_call>"""

# ... omitted for brevity ...
    assert len(extracted_tool_calls.tool_calls) == 1

    # Verify the function name is correct
    assert extracted_tool_calls.tool_calls[0].function.name == "get_current_weather"

    # Verify the arguments are parsed despite the missing closing tag
    args = json.loads(extracted_tool_calls.tool_calls[0].function.arguments)
    assert "city" in args
    assert args["city"] == "Dallas"
    assert args["state"] == "TX"
    assert args["unit"] == "fahrenheit"

    # Check that content before the tool call is preserved
    assert "Let me check the weather for you:" in extracted_tool_calls.content
```
**EN:** Test handling of missing closing </parameter> tag The body exercises logic via `ChatCompletionRequest`, `qwen3_tool_parser_parametrized.extract_tool_calls`, `json.loads` before asserting the expected outcome.
**CN:** 该测试用例验证 Extract Tool Calls Missing Closing Parameter Tag 在特定场景下的行为。 函数体会先通过 `ChatCompletionRequest`, `qwen3_tool_parser_parametrized.extract_tool_calls`, `json.loads` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
qwen3_xml_tool_parser
qwen3_tool_parser_parametrized
Constants / assignments
sample_tools
stream_delta_message_generator
test_extract_tool_calls_streaming_missing_closing_tag
test_extract_tool_calls_streaming_incremental
test_extract_tool_calls_complex_type_with_single_quote
test_extract_tool_calls_streaming_missing_opening_tag
test_malformed_xml_no_gt_delimiter
test_none_tool_calls_filtered
test_anyof_parameter_not_double_encoded
test_streaming_multi_param_single_chunk
test_no_double_serialization_string_args
test_get_vllm_registry_structural_tag_returns_structural_tag
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
- **Standard library / 标准库**: `json`, `collections.abc`
- **Third-party / 第三方依赖**: `pytest`, `openai.types.responses.function_tool`, `xgrammar`
- **vLLM internal / vLLM 内部依赖**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.tokenizers`, `vllm.tokenizers.detokenizer_utils`, `vllm.tool_parsers.qwen3coder_tool_parser`, `vllm.tool_parsers.qwen3xml_tool_parser`
- **Local test utilities / 本地测试辅助**: `tests.tool_parsers.utils`
