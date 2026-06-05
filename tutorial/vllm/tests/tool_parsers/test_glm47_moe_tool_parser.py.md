# test_glm47_moe_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/tool_parsers/test_glm47_moe_tool_parser.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Tests for the GLM-4.7 tool call parser. / 该文件主要围绕 Glm47 MoE Tool Parser 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-19)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# ruff: noqa: E501
"""Tests for the GLM-4.7 tool call parser."""

import json
from unittest.mock import Mock

import pytest

from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionRequest,
    ChatCompletionToolsParam,
    FunctionDefinition,
)
from vllm.tokenizers import get_tokenizer
from vllm.tool_parsers.glm47_moe_tool_parser import Glm47MoeModelToolParser

MODEL = "zai-org/GLM-4.5"
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `json`, `unittest.mock`, `pytest`, `vllm.entrypoints.openai.chat_completion.protocol`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Fixture: glm47_tokenizer (lines 22-24)
```python
@pytest.fixture(scope="module")
def glm47_tokenizer():
    return get_tokenizer(tokenizer_name=MODEL)
```
**EN:** Provides a pytest fixture for Glm47 Tokenizer. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `pytest.fixture`, `get_tokenizer`.
**CN:** 该代码块定义 pytest 夹具 `glm47_tokenizer`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `get_tokenizer` 构造或返回测试所需的值。

### Fixture: sample_tools (lines 27-45)
```python
@pytest.fixture
def sample_tools():
    return [
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
```
**EN:** Provides a pytest fixture for Sample Tools. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `ChatCompletionToolsParam`, `FunctionDefinition`.
**CN:** 该代码块定义 pytest 夹具 `sample_tools`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `ChatCompletionToolsParam`, `FunctionDefinition` 构造或返回测试所需的值。

### Fixture: glm47_tool_parser (lines 48-50)
```python
@pytest.fixture
def glm47_tool_parser(glm47_tokenizer, sample_tools):
    return Glm47MoeModelToolParser(glm47_tokenizer, tools=sample_tools)
```
**EN:** Provides a pytest fixture for Glm47 Tool Parser. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `Glm47MoeModelToolParser`.
**CN:** 该代码块定义 pytest 夹具 `glm47_tool_parser`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `Glm47MoeModelToolParser` 构造或返回测试所需的值。

### Fixture: mock_request (lines 53-58)
```python
@pytest.fixture
def mock_request(sample_tools) -> ChatCompletionRequest:
    request = Mock(spec=ChatCompletionRequest)
    request.tools = sample_tools
    request.tool_choice = "auto"
    return request
```
**EN:** Provides a pytest fixture for Mock Request. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `Mock`.
**CN:** 该代码块定义 pytest 夹具 `mock_request`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `Mock` 构造或返回测试所需的值。

### Class: TestGlm47ExtractToolCalls (lines 61-122)
```python
class TestGlm47ExtractToolCalls:
    def test_no_tool_call(self, glm47_tool_parser, mock_request):
        out = "This is a plain response."
        r = glm47_tool_parser.extract_tool_calls(out, request=mock_request)
        assert not r.tools_called
        assert r.content == out

    def test_zero_arg_inline(self, glm47_tool_parser, mock_request):
        out = "<tool_call>get_current_date</tool_call>"
        r = glm47_tool_parser.extract_tool_calls(out, request=mock_request)
        assert r.tools_called
        assert r.tool_calls[0].function.name == "get_current_date"
        assert json.loads(r.tool_calls[0].function.arguments) == {}
        assert r.content is None

    def test_zero_arg_newline(self, glm47_tool_parser, mock_request):
        out = "<tool_call>get_current_date\n</tool_call>"
        r = glm47_tool_parser.extract_tool_calls(out, request=mock_request)
        assert r.tools_called
# ... omitted for brevity ...
            "<tool_call>get_weather<arg_key>city</arg_key><arg_value>Shanghai</arg_value></tool_call>"
        )
        r = glm47_tool_parser.extract_tool_calls(out, request=mock_request)
        assert len(r.tool_calls) == 2

    def test_empty_content_none(self, glm47_tool_parser, mock_request):
        out = "<tool_call>get_current_date</tool_call>"
        r = glm47_tool_parser.extract_tool_calls(out, request=mock_request)
        assert r.content is None

    def test_whitespace_content_none(self, glm47_tool_parser, mock_request):
        out = "  \n  <tool_call>get_current_date</tool_call>"
        r = glm47_tool_parser.extract_tool_calls(out, request=mock_request)
        assert r.content is None
```
**EN:** Groups related scenarios for Testglm47extracttoolcalls. The class contains 10 test method(s).
**CN:** 该类把与 Testglm47extracttoolcalls 相关的场景组织在一起。 其中包含 10 个测试方法。

### Helper: _reset (lines 125-131)
```python
def _reset(parser):
    parser.current_tool_name_sent = False
    parser.prev_tool_call_arr = []
    parser.current_tool_id = -1
    parser.streamed_args_for_tool = []
    parser._tool_call_ids = []
    parser._sent_content_idx = 0
```
**EN:** Implements a reusable helper for Reset, reducing duplication across related tests.
**CN:** 该辅助函数为 Reset 提供可复用逻辑，用于减少相关测试之间的重复代码。

### Class: TestGlm47Streaming (lines 134-176)
```python
class TestGlm47Streaming:
    def test_no_args(self, glm47_tool_parser, mock_request):
        _reset(glm47_tool_parser)
        chunks = ["<tool_call>", "get_current_date", "</tool_call>"]
        current_text = ""
        for chunk in chunks:
            current_text += chunk
            glm47_tool_parser.extract_tool_calls_streaming(
                previous_text="",
                current_text=current_text,
                delta_text=chunk,
                previous_token_ids=[],
                current_token_ids=[],
                delta_token_ids=[],
                request=mock_request,
            )
        assert len(glm47_tool_parser.prev_tool_call_arr) >= 1

    def test_with_args(self, glm47_tool_parser, mock_request):
# ... omitted for brevity ...
        current_text = ""
        for chunk in chunks:
            current_text += chunk
            glm47_tool_parser.extract_tool_calls_streaming(
                previous_text="",
                current_text=current_text,
                delta_text=chunk,
                previous_token_ids=[],
                current_token_ids=[],
                delta_token_ids=[],
                request=mock_request,
            )
        args = json.loads(glm47_tool_parser.prev_tool_call_arr[0]["arguments"])
        assert args["city"] == "Beijing"
```
**EN:** Groups related scenarios for Testglm47streaming. The class contains 2 test method(s).
**CN:** 该类把与 Testglm47streaming 相关的场景组织在一起。 其中包含 2 个测试方法。

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
- **vLLM internal / vLLM 内部依赖**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.tokenizers`, `vllm.tool_parsers.glm47_moe_tool_parser`
