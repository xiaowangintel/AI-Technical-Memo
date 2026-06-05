# test_minimax_m2_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/tool_parsers/test_minimax_m2_tool_parser.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Minimax M2 Tool Parser behavior in the Tool Parsers test area through focused pytest scenarios. It focuses on scenarios such as Faketokenizer, Parser, Feed. / 该文件在 Tool Parsers 测试域中，通过有针对性的 pytest 场景验证 Minimax M2 Tool Parser 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-21)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import json

import pytest

from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionToolsParam,
    FunctionDefinition,
)
from vllm.tool_parsers.minimax_m2_tool_parser import (
    MinimaxM2ToolParser,
)

pytestmark = pytest.mark.cpu_test

# Token IDs matching FakeTokenizer.vocab
TC_START_ID = 1
TC_END_ID = 2
EOS_ID = 99
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `json`, `pytest`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.tool_parsers.minimax_m2_tool_parser`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Class: FakeTokenizer (lines 24-35)
```python
class FakeTokenizer:
    """Minimal fake tokenizer for unit tests."""

    def __init__(self):
        self.model_tokenizer = True
        self.vocab = {
            "<minimax:tool_call>": TC_START_ID,
            "</minimax:tool_call>": TC_END_ID,
        }

    def get_vocab(self):
        return self.vocab
```
**EN:** Groups related scenarios for Faketokenizer.
**CN:** 该类把与 Faketokenizer 相关的场景组织在一起。

### Fixture: parser (lines 38-40)
```python
@pytest.fixture
def parser():
    return MinimaxM2ToolParser(FakeTokenizer())
```
**EN:** Provides a pytest fixture for Parser. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `MinimaxM2ToolParser`, `FakeTokenizer`.
**CN:** 该代码块定义 pytest 夹具 `parser`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `MinimaxM2ToolParser`, `FakeTokenizer` 构造或返回测试所需的值。

### Helper: _feed (lines 48-80)
```python
def _feed(parser, chunks, request=None):
    """Feed chunks through the streaming parser and collect results.

    Each element in *chunks* is either:
    - a ``str``: used as delta_text (current_text accumulates automatically)
    - a ``(delta_text, delta_token_ids)`` tuple for special-token scenarios

    Returns a list of non-None DeltaMessage objects.
    """
    previous = ""
    results = []
    for chunk in chunks:
        if isinstance(chunk, tuple):
            delta, delta_ids = chunk
        else:
            delta = chunk
            delta_ids = []

        current = previous + delta
        result = parser.extract_tool_calls_streaming(
            previous_text=previous,
            current_text=current,
            delta_text=delta,
            previous_token_ids=[],
            current_token_ids=[],
            delta_token_ids=delta_ids,
            request=request,
        )
        if result is not None:
            results.append(result)
        previous = current

    return results
```
**EN:** Feed chunks through the streaming parser and collect results. It coordinates operations such as `isinstance`, `parser.extract_tool_calls_streaming`, `results.append`.
**CN:** 该辅助函数为 Feed 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `isinstance`, `parser.extract_tool_calls_streaming`, `results.append` 等操作。

### Helper: _collect_content (lines 83-85)
```python
def _collect_content(results):
    """Join all content strings from a list of DeltaMessages."""
    return "".join(r.content for r in results if r.content)
```
**EN:** Join all content strings from a list of DeltaMessages. It coordinates operations such as `''.join`.
**CN:** 该辅助函数为 Collect Content 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `''.join` 等操作。

### Class: TestContentStreaming (lines 117-143)
```python
class TestContentStreaming:
    """Tests for plain content (no tool calls)."""

    def test_plain_content(self, parser):
        """No tool call tokens — all text is streamed as content."""
        results = _feed(parser, ["Hello ", "world"])
        assert _collect_content(results) == "Hello world"
        assert not parser.prev_tool_call_arr

    def test_content_before_tool_call(self, parser):
        """Text before <minimax:tool_call> is streamed as content."""
        results = _feed(
            parser,
            [
                "Let me check. ",
                '<minimax:tool_call><invoke name="get_weather">'
                '<parameter name="city">Seattle</parameter>'
                "</invoke></minimax:tool_call>",
            ],
        )
        assert _collect_content(results) == "Let me check. "
        assert len(parser.prev_tool_call_arr) == 1

    def test_empty_delta_no_crash(self, parser):
        """Empty delta_text with no token IDs returns None."""
        results = _feed(parser, [("", [])])
        assert results == []
```
**EN:** Groups related scenarios for Testcontentstreaming. The class contains 3 test method(s).
**CN:** 该类把与 Testcontentstreaming 相关的场景组织在一起。 其中包含 3 个测试方法。

### Class: TestSingleInvoke (lines 151-201)
```python
class TestSingleInvoke:
    """Tests for a single <invoke> block."""

    def test_incremental_chunks(self, parser):
        """Each XML element arrives in a separate chunk."""
        results = _feed(
            parser,
            [
                "<minimax:tool_call>",
                '<invoke name="get_weather">',
                '<parameter name="city">Seattle</parameter>',
                "</invoke></minimax:tool_call>",
            ],
        )
        tc = _collect_tool_calls(results)
        assert len(tc) == 1
        assert tc[0]["name"] == "get_weather"
        assert json.loads(tc[0]["arguments"]) == {"city": "Seattle"}
        assert tc[0]["id"] is not None
# ... omitted for brevity ...
            parser,
            [
                "<minimax:tool_call>",
                '<invoke name="get_weather">',
                '<parameter name="city">Seattle</parameter>',
                '<parameter name="days">5</parameter>',
                "</invoke></minimax:tool_call>",
            ],
        )
        tc = _collect_tool_calls(results)
        assert json.loads(tc[0]["arguments"]) == {
            "city": "Seattle",
            "days": "5",
        }
```
**EN:** Groups related scenarios for Testsingleinvoke. The class contains 3 test method(s).
**CN:** 该类把与 Testsingleinvoke 相关的场景组织在一起。 其中包含 3 个测试方法。

### Class: TestMultipleInvokes (lines 204-262)
```python
class TestMultipleInvokes:
    """Tests for multiple <invoke> blocks in one tool call."""

    def test_two_invokes_incremental(self, parser):
        """Two invokes arriving one chunk at a time."""
        results = _feed(
            parser,
            [
                "<minimax:tool_call>",
                '<invoke name="search_web">'
                '<parameter name="query">OpenAI</parameter>'
                "</invoke>",
                '<invoke name="search_web">'
                '<parameter name="query">Gemini</parameter>'
                "</invoke>",
                "</minimax:tool_call>",
            ],
        )
        tc = _collect_tool_calls(results)
# ... omitted for brevity ...
            [
                "<minimax:tool_call>",
                '<invoke name="get_weather">'
                '<parameter name="city">NYC</parameter>'
                "</invoke>",
                '<invoke name="get_stock">'
                '<parameter name="ticker">AAPL</parameter>'
                "</invoke>",
                "</minimax:tool_call>",
            ],
        )
        tc = _collect_tool_calls(results)
        assert tc[0]["name"] == "get_weather"
        assert tc[1]["name"] == "get_stock"
```
**EN:** Groups related scenarios for Testmultipleinvokes. The class contains 3 test method(s).
**CN:** 该类把与 Testmultipleinvokes 相关的场景组织在一起。 其中包含 3 个测试方法。

### Class: TestInternalState (lines 270-301)
```python
class TestInternalState:
    """Verify prev_tool_call_arr is correct."""

    def test_prev_tool_call_arr_single(self, parser):
        _feed(
            parser,
            [
                '<minimax:tool_call><invoke name="fn">'
                '<parameter name="a">1</parameter>'
                "</invoke></minimax:tool_call>",
            ],
        )
        assert len(parser.prev_tool_call_arr) == 1
        assert parser.prev_tool_call_arr[0]["name"] == "fn"
        assert parser.prev_tool_call_arr[0]["arguments"] == {"a": "1"}

    def test_prev_tool_call_arr_multiple(self, parser):
        """prev_tool_call_arr records each invoke with correct arguments."""
        _feed(
            parser,
            [
                "<minimax:tool_call>",
                '<invoke name="search"><parameter name="q">hello</parameter></invoke>',
                '<invoke name="search"><parameter name="q">world</parameter></invoke>',
                "</minimax:tool_call>",
            ],
        )
        assert len(parser.prev_tool_call_arr) == 2
        assert parser.prev_tool_call_arr[0]["name"] == "search"
        assert parser.prev_tool_call_arr[0]["arguments"] == {"q": "hello"}
        assert parser.prev_tool_call_arr[1]["name"] == "search"
        assert parser.prev_tool_call_arr[1]["arguments"] == {"q": "world"}
```
**EN:** Groups related scenarios for Testinternalstate. The class contains 2 test method(s).
**CN:** 该类把与 Testinternalstate 相关的场景组织在一起。 其中包含 2 个测试方法。

### Class: TestDeltaMessageFormat (lines 309-344)
```python
class TestDeltaMessageFormat:
    """Verify the shape of emitted DeltaMessage / DeltaToolCall."""

    def test_tool_call_fields(self, parser):
        """Each emitted tool call has id, name, arguments, type, index."""
        results = _feed(
            parser,
            [
                '<minimax:tool_call><invoke name="fn">'
                '<parameter name="k">v</parameter>'
                "</invoke></minimax:tool_call>",
            ],
        )
        tc_deltas = [tc for r in results for tc in (r.tool_calls or [])]
        assert len(tc_deltas) == 1
        tc = tc_deltas[0]
        assert tc.index == 0
        assert tc.type == "function"
        assert tc.id is not None and tc.id.startswith("call_")
# ... omitted for brevity ...
    def test_multi_invoke_indices(self, parser):
        """Multiple invokes get sequential indices."""
        results = _feed(
            parser,
            [
                "<minimax:tool_call>",
                '<invoke name="a"><parameter name="x">1</parameter></invoke>',
                '<invoke name="b"><parameter name="x">2</parameter></invoke>',
                "</minimax:tool_call>",
            ],
        )
        tc_deltas = [tc for r in results for tc in (r.tool_calls or [])]
        indices = [tc.index for tc in tc_deltas]
        assert indices == [0, 1]
```
**EN:** Groups related scenarios for Testdeltamessageformat. The class contains 2 test method(s).
**CN:** 该类把与 Testdeltamessageformat 相关的场景组织在一起。 其中包含 2 个测试方法。

### Additional scenarios (summary)
```python
_collect_tool_calls
TestEOSHandling
TestSpecialTokenDetection
TestLargeChunks
TestAnyOfNullableParam
TestNoneStringPreservation
```
**EN:** The file contains more helper or test blocks than can be shown compactly here. This summary lists additional scenarios that continue the same validation strategy.
**CN:** 该文件包含的辅助逻辑或测试场景较多，无法在此完整展开；这里列出其余场景名称，便于快速把握覆盖范围。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **Tokenization / 分词与反分词**
  - **EN:** The logic checks how text and token IDs are converted, streamed, or reconstructed.
  - **CN:** 逻辑重点检查文本与 token ID 的转换、流式处理和重建过程。
- **Tool-call parsing / 工具调用解析**
  - **EN:** The scenarios verify extraction of structured tool calls from model outputs and streaming deltas.
  - **CN:** 这些场景验证如何从模型输出和流式增量中提取结构化工具调用。
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`
- **Third-party / 第三方依赖**: `pytest`
- **vLLM internal / vLLM 内部依赖**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.tool_parsers.minimax_m2_tool_parser`
