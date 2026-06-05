# test_hy_v3_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/tool_parsers/test_hy_v3_tool_parser.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Tests for the HYV3 tool call parser. / 该文件主要围绕 Hy V3 Tool Parser 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-21)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# ruff: noqa: E501
"""Tests for the HYV3 tool call parser."""

import json
from unittest.mock import Mock

import pytest

from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionRequest,
    ChatCompletionToolsParam,
    FunctionDefinition,
)
from vllm.entrypoints.openai.engine.protocol import DeltaMessage
from vllm.tokenizers import get_tokenizer
from vllm.tool_parsers.hy_v3_tool_parser import HYV3ToolParser

parser_name = "hy_v3"
MODEL = "tencent/Hy3-preview"
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `json`, `unittest.mock`, `pytest`, `vllm.entrypoints.openai.chat_completion.protocol`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Fixture: hy_v3_tokenizer (lines 24-26)
```python
@pytest.fixture(scope="module")
def hy_v3_tokenizer():
    return get_tokenizer(tokenizer_name=MODEL)
```
**EN:** Provides a pytest fixture for Hy V3 Tokenizer. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `pytest.fixture`, `get_tokenizer`.
**CN:** 该代码块定义 pytest 夹具 `hy_v3_tokenizer`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `get_tokenizer` 构造或返回测试所需的值。

### Fixture: hy_v3_tool_parser (lines 29-31)
```python
@pytest.fixture
def hy_v3_tool_parser(hy_v3_tokenizer):
    return HYV3ToolParser(hy_v3_tokenizer)
```
**EN:** Provides a pytest fixture for Hy V3 Tool Parser. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `HYV3ToolParser`.
**CN:** 该代码块定义 pytest 夹具 `hy_v3_tool_parser`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `HYV3ToolParser` 构造或返回测试所需的值。

### Fixture: mock_request (lines 34-55)
```python
@pytest.fixture
def mock_request() -> ChatCompletionRequest:
    request = Mock(spec=ChatCompletionRequest)
    request.tools = [
        ChatCompletionToolsParam(
            function=FunctionDefinition(name="get_current_date", parameters={}),
        ),
        ChatCompletionToolsParam(
            function=FunctionDefinition(
                name="get_weather",
                parameters={
                    "type": "object",
                    "properties": {
                        "city": {"type": "string"},
                        "date": {"type": "string"},
                    },
                },
            ),
        ),
    ]
    request.tool_choice = "auto"
    return request
```
**EN:** Provides a pytest fixture for Mock Request. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `Mock`, `ChatCompletionToolsParam`, `FunctionDefinition`.
**CN:** 该代码块定义 pytest 夹具 `mock_request`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `Mock`, `ChatCompletionToolsParam`, `FunctionDefinition` 构造或返回测试所需的值。

### Class: TestHYV3ExtractToolCalls (lines 58-124)
```python
class TestHYV3ExtractToolCalls:
    def test_no_tool_call(self, hy_v3_tool_parser, mock_request):
        out = "This is a plain response."
        r = hy_v3_tool_parser.extract_tool_calls(out, request=mock_request)
        assert not r.tools_called
        assert r.content == out

    def test_zero_arg_inline(self, hy_v3_tool_parser, mock_request):
        out = (
            "<tool_calls><tool_call>get_current_date<tool_sep></tool_call></tool_calls>"
        )
        r = hy_v3_tool_parser.extract_tool_calls(out, request=mock_request)
        assert r.tools_called
        assert r.tool_calls[0].function.name == "get_current_date"
        assert json.loads(r.tool_calls[0].function.arguments) == {}
        assert r.content is None

    def test_zero_arg_newline(self, hy_v3_tool_parser, mock_request):
        out = "<tool_calls>\n<tool_call>get_current_date<tool_sep>\n</tool_call>\n</tool_calls>"
# ... omitted for brevity ...
    def test_multiple(self, hy_v3_tool_parser, mock_request):
        out = (
            "<tool_calls>\n<tool_call>get_weather<tool_sep>\n<arg_key>city</arg_key>\n<arg_value>Beijing"
            "</arg_value>\n<arg_key>date</arg_key>\n<arg_value>2026-03-30</arg_value>\n</tool_call>\n"
            "<tool_call>get_weather<tool_sep>\n<arg_key>city</arg_key>\n<arg_value>Hangzhou</arg_value>\n"
            "<arg_key>date</arg_key>\n<arg_value>2026-03-30</arg_value>\n</tool_call>\n</tool_calls>"
        )
        r = hy_v3_tool_parser.extract_tool_calls(out, request=mock_request)
        assert len(r.tool_calls) == 2

    def test_empty_content_none(self, hy_v3_tool_parser, mock_request):
        out = "<tool_calls>\n<tool_call>get_current_date<tool_sep>\n</tool_call>\n</tool_calls>"
        r = hy_v3_tool_parser.extract_tool_calls(out, request=mock_request)
        assert r.content is None
```
**EN:** Groups related scenarios for Testhyv3extracttoolcalls. The class contains 8 test method(s).
**CN:** 该类把与 Testhyv3extracttoolcalls 相关的场景组织在一起。 其中包含 8 个测试方法。

### Helper: _simulate_streaming (lines 127-152)
```python
def _simulate_streaming(
    parser: HYV3ToolParser,
    deltas: list[str],
    request: ChatCompletionRequest,
) -> list[DeltaMessage | None]:
    results: list[DeltaMessage | None] = []
    previous_text = ""
    previous_token_ids: list[int] = []
    vocab = parser.vocab
    for delta_text in deltas:
        current_text = previous_text + delta_text
        delta_token_ids = [tid for tok, tid in vocab.items() if tok in delta_text]
        current_token_ids = previous_token_ids + delta_token_ids
        result = parser.extract_tool_calls_streaming(
            previous_text=previous_text,
            current_text=current_text,
            delta_text=delta_text,
            previous_token_ids=previous_token_ids,
            current_token_ids=current_token_ids,
            delta_token_ids=delta_token_ids,
            request=request,
        )
        results.append(result)
        previous_text = current_text
        previous_token_ids = current_token_ids
    return results
```
**EN:** Implements a reusable helper for Simulate Streaming, reducing duplication across related tests. It coordinates operations such as `parser.extract_tool_calls_streaming`, `results.append`, `vocab.items`.
**CN:** 该辅助函数为 Simulate Streaming 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `parser.extract_tool_calls_streaming`, `results.append`, `vocab.items` 等操作。

### Helper: _collect_streaming_tool_calls (lines 155-172)
```python
def _collect_streaming_tool_calls(results: list[DeltaMessage | None]) -> list[dict]:
    tool_calls: dict[int, dict] = {}
    for result in results:
        if result is None or not result.tool_calls:
            continue
        for tc in result.tool_calls:
            idx = tc.index
            if idx not in tool_calls:
                tool_calls[idx] = {
                    "name": tc.function.name or "",
                    "arguments": tc.function.arguments or "",
                }
            else:
                if tc.function.name:
                    tool_calls[idx]["name"] += tc.function.name
                if tc.function.arguments:
                    tool_calls[idx]["arguments"] += tc.function.arguments
    return [tool_calls[i] for i in sorted(tool_calls.keys())]
```
**EN:** Implements a reusable helper for Collect Streaming Tool Calls, reducing duplication across related tests. It coordinates operations such as `sorted`, `tool_calls.keys`.
**CN:** 该辅助函数为 Collect Streaming Tool Calls 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `sorted`, `tool_calls.keys` 等操作。

### Helper: _collect_streaming_content (lines 175-180)
```python
def _collect_streaming_content(results: list[DeltaMessage | None]) -> str:
    parts = []
    for result in results:
        if result is not None and result.content:
            parts.append(result.content)
    return "".join(parts)
```
**EN:** Implements a reusable helper for Collect Streaming Content, reducing duplication across related tests. It coordinates operations such as `''.join`, `parts.append`.
**CN:** 该辅助函数为 Collect Streaming Content 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `''.join`, `parts.append` 等操作。

### Class: TestHYV3ExtractToolCallsStreaming (lines 183-274)
```python
class TestHYV3ExtractToolCallsStreaming:
    def test_no_tool_call_streaming(self, hy_v3_tool_parser, mock_request):
        deltas = ["This is ", "a plain ", "response."]
        results = _simulate_streaming(hy_v3_tool_parser, deltas, mock_request)
        content = _collect_streaming_content(results)
        assert content == "This is a plain response."
        assert len(_collect_streaming_tool_calls(results)) == 0

    def test_zero_arg_streaming(self, hy_v3_tool_parser, mock_request):
        deltas = [
            "<tool_calls>",
            "\n<tool_call>",
            "get_current_date",
            "<tool_sep>",
            "\n</tool_call>",
            "\n</tool_calls>",
        ]
        results = _simulate_streaming(hy_v3_tool_parser, deltas, mock_request)
        tc = _collect_streaming_tool_calls(results)
# ... omitted for brevity ...
            "\n</tool_calls>",
        ]
        results = _simulate_streaming(hy_v3_tool_parser, deltas, mock_request)
        tc = _collect_streaming_tool_calls(results)
        assert len(tc) == 2
        assert json.loads(tc[0]["arguments"])["city"] == "Beijing"
        assert json.loads(tc[1]["arguments"])["city"] == "Hangzhou"

    def test_all_in_one_delta_streaming(self, hy_v3_tool_parser, mock_request):
        out = "<tool_calls>\n<tool_call>get_current_date<tool_sep>\n</tool_call>\n</tool_calls>"
        results = _simulate_streaming(hy_v3_tool_parser, [out], mock_request)
        tc = _collect_streaming_tool_calls(results)
        assert len(tc) == 1 and tc[0]["name"] == "get_current_date"
        assert json.loads(tc[0]["arguments"]) == {}
```
**EN:** Groups related scenarios for Testhyv3extracttoolcallsstreaming. The class contains 6 test method(s).
**CN:** 该类把与 Testhyv3extracttoolcallsstreaming 相关的场景组织在一起。 其中包含 6 个测试方法。

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
- **vLLM internal / vLLM 内部依赖**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.tokenizers`, `vllm.tool_parsers.hy_v3_tool_parser`
