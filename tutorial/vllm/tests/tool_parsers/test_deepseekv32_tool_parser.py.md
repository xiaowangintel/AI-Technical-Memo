# test_deepseekv32_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/tool_parsers/test_deepseekv32_tool_parser.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Unit tests for DeepSeekV32ToolParser. / 该文件主要围绕 Deepseekv32 Tool Parser 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-30)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

"""Unit tests for DeepSeekV32ToolParser.

These tests use a minimal mock tokenizer so no real model weights are required.
"""

import json
from unittest.mock import MagicMock

import pytest

from tests.tool_parsers.utils import run_tool_extraction_streaming
from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionToolsParam,
    FunctionDefinition,
)
from vllm.tokenizers import get_tokenizer
from vllm.tool_parsers.deepseekv32_tool_parser import DeepSeekV32ToolParser

# ---------------------------------------------------------------------------
# Helpers
# ---------------------------------------------------------------------------

# Token IDs are not used by the V32 parser logic, so we only need the
# tokenizer object to be truthy (the parser checks `if not self.model_tokenizer`).
MOCK_TOKENIZER = MagicMock()
MOCK_TOKENIZER.get_vocab.return_value = {}
MOCK_TOKENIZER.tokenize.return_value = []
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `json`, `unittest.mock`, `pytest`, `vllm.entrypoints.openai.chat_completion.protocol`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: make_parser (lines 33-34)
```python
def make_parser(tools=None) -> DeepSeekV32ToolParser:
    return DeepSeekV32ToolParser(MOCK_TOKENIZER, tools=tools)
```
**EN:** Implements a reusable helper for Make Parser, reducing duplication across related tests. It coordinates operations such as `DeepSeekV32ToolParser`.
**CN:** 该辅助函数为 Make Parser 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `DeepSeekV32ToolParser` 等操作。

### Helper: make_tool_param (lines 37-42)
```python
def make_tool_param(name: str, params: dict) -> MagicMock:
    """Build a mock tool matching the ChatCompletionToolsParam shape."""
    tool = MagicMock()
    tool.function.name = name
    tool.function.parameters = params
    return tool
```
**EN:** Build a mock tool matching the ChatCompletionToolsParam shape. It coordinates operations such as `MagicMock`.
**CN:** 该辅助函数为 Make Tool Param 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `MagicMock` 等操作。

### Constants / assignments (lines 52-52)
```python
FC_START = "<｜DSML｜function_calls>"
```
**EN:** Defines shared constants or configuration objects like `FC_START`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `FC_START`），供后续测试重复使用。

### Class: TestConvertParamValue (lines 73-117)
```python
class TestConvertParamValue:
    @pytest.fixture
    def parser(self):
        return make_parser()

    def test_null(self, parser):
        assert parser._convert_param_value("null", "string") is None
        assert parser._convert_param_value("NULL", "integer") is None

    def test_string(self, parser):
        assert parser._convert_param_value("hello", "string") == "hello"

    def test_integer_valid(self, parser):
        assert parser._convert_param_value("42", "integer") == 42

    def test_integer_invalid_falls_back_to_str(self, parser):
        assert parser._convert_param_value("abc", "int") == "abc"

    def test_number_float(self, parser):
# ... omitted for brevity ...
        assert parser._convert_param_value("False", "bool") is False

    def test_object_valid_json(self, parser):
        assert parser._convert_param_value('{"k": 1}', "object") == {"k": 1}

    def test_object_invalid_json_falls_back(self, parser):
        assert parser._convert_param_value("not-json", "object") == "not-json"

    def test_array_valid_json(self, parser):
        assert parser._convert_param_value("[1, 2]", "array") == [1, 2]

    def test_unknown_type_tries_json_then_string(self, parser):
        assert parser._convert_param_value("123", "unknown") == 123
        assert parser._convert_param_value("hello", "unknown") == "hello"
```
**EN:** Groups related scenarios for Testconvertparamvalue. The class contains 12 test method(s) and 1 helper/setup method(s).
**CN:** 该类把与 Testconvertparamvalue 相关的场景组织在一起。 其中包含 12 个测试方法，以及 1 个辅助或初始化方法。

### Class: TestExtractToolCalls (lines 125-332)
```python
class TestExtractToolCalls:
    @pytest.fixture
    def parser(self):
        return make_parser()

    def test_no_tool_call(self, parser):
        result = parser.extract_tool_calls("just some text", None)
        assert not result.tools_called
        assert result.tool_calls == []
        assert result.content == "just some text"

    def test_single_tool_no_params(self, parser):
        model_output = f'{FC_START}\n{INV_START}get_time">\n{INV_END}\n{FC_END}'
        result = parser.extract_tool_calls(model_output, None)
        assert result.tools_called
        assert len(result.tool_calls) == 1
        assert result.tool_calls[0].function.name == "get_time"
        assert json.loads(result.tool_calls[0].function.arguments) == {}

# ... omitted for brevity ...
        parser = make_parser(tools=[tool])
        model_output = (
            f"{FC_START}\n"
            f'{INV_START}get_weather">\n'
            f'{PARAM_START}input" string="true">'
            f'{{"location":"Beijing"}}'
            f"{PARAM_END}\n"
            f"{INV_END}\n"
            f"{FC_END}"
        )
        result = parser.extract_tool_calls(model_output, None)
        assert result.tools_called
        args = json.loads(result.tool_calls[0].function.arguments)
        assert args == {"location": "Beijing"}
```
**EN:** Groups related scenarios for Testextracttoolcalls. The class contains 11 test method(s) and 1 helper/setup method(s).
**CN:** 该类把与 Testextracttoolcalls 相关的场景组织在一起。 其中包含 11 个测试方法，以及 1 个辅助或初始化方法。

### Class: TestExtractToolCallsStreaming (lines 340-714)
```python
class TestExtractToolCallsStreaming:
    """Simulate character-by-character streaming and verify reconstructed args."""

    @pytest.fixture
    def parser(self):
        return make_parser()

    def _stream(self, parser, full_text: str, request=None):
        """Drive the parser line-by-line and collect non-None deltas.

        Real tokenizers emit multi-character chunks, not individual characters.
        Streaming character-by-character would never deliver the full sentinel
        token (e.g. '｜DSML｜') in a single delta, so we split on newlines to
        ensure each sentinel always lands in one chunk.
        """
        if request is None:
            request = make_request()
        # Split into lines, preserving the trailing newline in each chunk.
        chunks: list[str] = []
# ... omitted for brevity ...
            full_text = build_tool_call("fn", {"k": "v"})
            deltas = self._stream_chunked(p, full_text, chunk_size=chunk_size)
            content = "".join(d.content for d in deltas if d.content is not None)
            assert content == "", (
                f"Leaked content {content!r} at chunk_size={chunk_size}"
            )

    def test_false_partial_marker_emitted(self, parser):
        """Text ending with a prefix of the start token that turns out
        NOT to be a marker must still be emitted as content."""
        full_text = "<｜DSM some regular text"
        deltas = self._stream_chunked(parser, full_text, chunk_size=3)
        content = "".join(d.content for d in deltas if d.content is not None)
        assert content == full_text
```
**EN:** Groups related scenarios for Testextracttoolcallsstreaming. The class contains 21 test method(s) and 4 helper/setup method(s).
**CN:** 该类把与 Testextracttoolcallsstreaming 相关的场景组织在一起。 其中包含 21 个测试方法，以及 4 个辅助或初始化方法。

### Class: TestDelimiterPreservation (lines 717-796)
```python
class TestDelimiterPreservation:
    """Regression: fast detokenization skipping DSML delimiters (PR #33964)."""

    @pytest.fixture
    def parser(self):
        return make_parser()

    def test_delimiter_preserved_fast_detokenization(self, parser):
        """DSML delimiters as literal text must still be detected."""
        # Delimiters appear as regular text (fast detokenization scenario).
        model_output = (
            f"{FC_START}\n"
            f'{INV_START}get_weather">\n'
            f'{PARAM_START}location" string="true">Tokyo{PARAM_END}\n'
            f"{INV_END}\n"
            f"{FC_END}"
        )

        # Non-streaming: parser must detect the tool call
# ... omitted for brevity ...
            nl = remaining.find("\n")
            if nl == -1:
                chunks.append(remaining)
                break
            chunks.append(remaining[: nl + 1])
            remaining = remaining[nl + 1 :]

        reconstructor = run_tool_extraction_streaming(
            parser, chunks, request, assert_one_tool_per_delta=False
        )
        assert len(reconstructor.tool_calls) == 1
        assert reconstructor.tool_calls[0].function.name == "search"
        streamed_args = json.loads(reconstructor.tool_calls[0].function.arguments)
        assert streamed_args == ns_args
```
**EN:** Groups related scenarios for Testdelimiterpreservation. The class contains 2 test method(s) and 1 helper/setup method(s).
**CN:** 该类把与 Testdelimiterpreservation 相关的场景组织在一起。 其中包含 2 个测试方法，以及 1 个辅助或初始化方法。

### Fixture: deepseekv32_tokenizer (lines 799-801)
```python
@pytest.fixture(scope="module")
def deepseekv32_tokenizer():
    return get_tokenizer(tokenizer_name="deepseek-ai/DeepSeek-V3.2")
```
**EN:** Provides a pytest fixture for Deepseekv32 Tokenizer. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `pytest.fixture`, `get_tokenizer`.
**CN:** 该代码块定义 pytest 夹具 `deepseekv32_tokenizer`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `get_tokenizer` 构造或返回测试所需的值。

### Fixture: parser (lines 804-806)
```python
@pytest.fixture
def parser(deepseekv32_tokenizer):
    return DeepSeekV32ToolParser(deepseekv32_tokenizer)
```
**EN:** Provides a pytest fixture for Parser. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `DeepSeekV32ToolParser`.
**CN:** 该代码块定义 pytest 夹具 `parser`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `DeepSeekV32ToolParser` 构造或返回测试所需的值。

### Test: test_convert_param_value_single_types (lines 809-853)
```python
def test_convert_param_value_single_types(parser):
    """Test _convert_param_value with single type parameters."""
    # Test string type
    assert parser._convert_param_value("hello", "string") == "hello"
    assert parser._convert_param_value("123", "string") == "123"

    # Test integer type - valid integers
    assert parser._convert_param_value("123", "integer") == 123
    assert parser._convert_param_value("456", "int") == 456
    # Invalid integer should return original string (due to exception catch)
    assert parser._convert_param_value("abc", "integer") == "abc"

    # Test float/number type
    assert parser._convert_param_value("123.45", "float") == 123.45
    assert (
        parser._convert_param_value("123.0", "number") == 123
    )  # Should be int when whole number
    assert parser._convert_param_value("123.5", "number") == 123.5
    # Invalid float should return original string
# ... omitted for brevity ...
    assert parser._convert_param_value("null", "string") is None
    assert parser._convert_param_value("null", "integer") is None

    # Test object/array type (JSON)
    assert parser._convert_param_value('{"key": "value"}', "object") == {"key": "value"}
    assert parser._convert_param_value("[1, 2, 3]", "array") == [1, 2, 3]
    # Invalid JSON should return original string
    assert parser._convert_param_value("{invalid}", "object") == "{invalid}"

    # Test fallback for unknown type (tries json.loads, then returns original)
    assert parser._convert_param_value('{"key": "value"}', "unknown") == {
        "key": "value"
    }
    assert parser._convert_param_value("plain text", "unknown") == "plain text"
```
**EN:** Test _convert_param_value with single type parameters. The body exercises logic via `parser._convert_param_value` before asserting the expected outcome.
**CN:** 该测试用例验证 Convert Param Value Single Types 在特定场景下的行为。 函数体会先通过 `parser._convert_param_value` 驱动目标逻辑，再断言预期结果。

### Test: test_convert_param_value_multi_typed_values (lines 856-896)
```python
def test_convert_param_value_multi_typed_values(parser):
    """Test _convert_param_value with multi-typed values (list of types)."""
    # Test with list of types where first type succeeds
    assert parser._convert_param_value("123", ["integer", "string"]) == 123
    assert parser._convert_param_value("true", ["boolean", "string"]) is True
    assert parser._convert_param_value('{"x": 1}', ["object", "string"]) == {"x": 1}

    # Test with list of types where first type fails but second succeeds
    # "abc" is not a valid integer, so should try string next
    assert parser._convert_param_value("abc", ["integer", "string"]) == "abc"

    # Test with list of types where all fail - should return original value
    # "invalid json" is not valid JSON, last type is "object" which will fail JSON parse
    result = parser._convert_param_value("invalid json", ["integer", "object"])
    assert result == "invalid json"  # Returns original value after all types fail

    # Test with three types
    assert parser._convert_param_value("123.5", ["integer", "float", "string"]) == 123.5
    assert parser._convert_param_value("true", ["integer", "boolean", "string"]) is True
# ... omitted for brevity ...

    # Test that order matters
    assert (
        parser._convert_param_value("123", ["string", "integer"]) == "123"
    )  # String first
    assert (
        parser._convert_param_value("123", ["integer", "string"]) == 123
    )  # Integer first

    # Test with all types failing - returns original value
    assert (
        parser._convert_param_value("not_a_number", ["integer", "float", "boolean"])
        == "not_a_number"
    )
```
**EN:** Test _convert_param_value with multi-typed values (list of types). The body exercises logic via `parser._convert_param_value` before asserting the expected outcome.
**CN:** 该测试用例验证 Convert Param Value Multi Typed Values 在特定场景下的行为。 函数体会先通过 `parser._convert_param_value` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
make_request
Constants / assignments
Constants / assignments
Constants / assignments
Constants / assignments
Constants / assignments
build_tool_call
test_convert_param_value_stricter_type_checking
test_convert_param_value_edge_cases
test_convert_param_value_checked_helper
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
- **Distributed coordination / 分布式协同**
  - **EN:** The file validates multi-process or multi-worker coordination primitives such as collectives and rank-aware setup.
  - **CN:** 该文件验证多进程或多工作节点之间的协同原语，例如集合通信和基于 rank 的初始化。
- **Tokenization / 分词与反分词**
  - **EN:** The logic checks how text and token IDs are converted, streamed, or reconstructed.
  - **CN:** 逻辑重点检查文本与 token ID 的转换、流式处理和重建过程。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest.mock`
- **Third-party / 第三方依赖**: `pytest`
- **vLLM internal / vLLM 内部依赖**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.tokenizers`, `vllm.tool_parsers.deepseekv32_tool_parser`
- **Local test utilities / 本地测试辅助**: `tests.tool_parsers.utils`
