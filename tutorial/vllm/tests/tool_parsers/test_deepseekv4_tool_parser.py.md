# test_deepseekv4_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/tool_parsers/test_deepseekv4_tool_parser.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Unit tests for DeepSeekV4ToolParser. / 该文件主要围绕 Deepseekv4 Tool Parser 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-29)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

"""Unit tests for DeepSeekV4ToolParser."""

import json
from unittest.mock import MagicMock

import pytest
from xgrammar import StructuralTag

from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionNamedFunction,
    ChatCompletionNamedToolChoiceParam,
    ChatCompletionRequest,
    ChatCompletionToolsParam,
)
from vllm.tool_parsers import ToolParserManager
from vllm.tool_parsers.deepseekv4_tool_parser import DeepSeekV4ToolParser

MOCK_TOKENIZER = MagicMock()
MOCK_TOKENIZER.get_vocab.return_value = {}

TC_START = "<｜DSML｜tool_calls>"
TC_END = "</｜DSML｜tool_calls>"
INV_START = '<｜DSML｜invoke name="'
INV_END = "</｜DSML｜invoke>"
PARAM_START = '<｜DSML｜parameter name="'
PARAM_END = "</｜DSML｜parameter>"
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `json`, `unittest.mock`, `pytest`, `xgrammar`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Fixture: sample_tools (lines 32-66)
```python
@pytest.fixture
def sample_tools() -> list[ChatCompletionToolsParam]:
    return [
        ChatCompletionToolsParam(
            type="function",
            function={
                "name": "get_current_weather",
                "description": "Get the current weather",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "city": {"type": "string", "description": "The city name"},
                        "state": {"type": "string", "description": "The state code"},
                        "unit": {"type": "string", "enum": ["fahrenheit", "celsius"]},
                    },
                    "required": ["city", "state"],
                },
            },
        ),
# ... omitted for brevity ...
            function={
                "name": "calculate_area",
                "description": "Calculate area of a shape",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "shape": {"type": "string"},
                        "dimensions": {"type": "object"},
                        "precision": {"type": "integer"},
                    },
                },
            },
        ),
    ]
```
**EN:** Provides a pytest fixture for Sample Tools. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `ChatCompletionToolsParam`.
**CN:** 该代码块定义 pytest 夹具 `sample_tools`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `ChatCompletionToolsParam` 构造或返回测试所需的值。

### Helper: make_parser (lines 69-70)
```python
def make_parser(tools=None) -> DeepSeekV4ToolParser:
    return DeepSeekV4ToolParser(MOCK_TOKENIZER, tools=tools)
```
**EN:** Implements a reusable helper for Make Parser, reducing duplication across related tests. It coordinates operations such as `DeepSeekV4ToolParser`.
**CN:** 该辅助函数为 Make Parser 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `DeepSeekV4ToolParser` 等操作。

### Helper: make_request (lines 73-76)
```python
def make_request(tools=None) -> MagicMock:
    req = MagicMock()
    req.tools = tools
    return req
```
**EN:** Implements a reusable helper for Make Request, reducing duplication across related tests. It coordinates operations such as `MagicMock`.
**CN:** 该辅助函数为 Make Request 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `MagicMock` 等操作。

### Test: test_registered (lines 121-122)
```python
def test_registered():
    assert ToolParserManager.get_tool_parser("deepseek_v4") is DeepSeekV4ToolParser
```
**EN:** Checks Registered under a focused test scenario. The body exercises logic via `ToolParserManager.get_tool_parser` before asserting the expected outcome.
**CN:** 该测试用例验证 Registered 在特定场景下的行为。 函数体会先通过 `ToolParserManager.get_tool_parser` 驱动目标逻辑，再断言预期结果。

### Test: test_extract_tool_calls (lines 125-141)
```python
def test_extract_tool_calls():
    parser = make_parser()
    model_output = "Let me check. " + build_tool_call(
        "get_weather", {"location": "Beijing", "unit": "celsius"}
    )

    result = parser.extract_tool_calls(model_output, make_request())

    assert result.tools_called
    assert result.content == "Let me check. "
    assert len(result.tool_calls) == 1
    tool_call = result.tool_calls[0]
    assert tool_call.function.name == "get_weather"
    assert json.loads(tool_call.function.arguments) == {
        "location": "Beijing",
        "unit": "celsius",
    }
```
**EN:** Checks Extract Tool Calls under a focused test scenario. The body exercises logic via `make_parser`, `parser.extract_tool_calls`, `build_tool_call` before asserting the expected outcome.
**CN:** 该测试用例验证 Extract Tool Calls 在特定场景下的行为。 函数体会先通过 `make_parser`, `parser.extract_tool_calls`, `build_tool_call` 驱动目标逻辑，再断言预期结果。

### Test: test_function_calls_block_is_not_accepted (lines 144-153)
```python
def test_function_calls_block_is_not_accepted():
    parser = make_parser()
    model_output = build_tool_call("search", {"query": "vllm"}).replace(
        "tool_calls", "function_calls"
    )

    result = parser.extract_tool_calls(model_output, make_request())

    assert not result.tools_called
    assert result.content == model_output
```
**EN:** Checks Function Calls Block Is Not Accepted under a focused test scenario. The body exercises logic via `make_parser`, `build_tool_call('search', {'query': 'vllm'}).replace`, `parser.extract_tool_calls` before asserting the expected outcome.
**CN:** 该测试用例验证 Function Calls Block Is Not Accepted 在特定场景下的行为。 函数体会先通过 `make_parser`, `build_tool_call('search', {'query': 'vllm'}).replace`, `parser.extract_tool_calls` 驱动目标逻辑，再断言预期结果。

### Test: test_streaming_extracts_complete_invokes (lines 156-169)
```python
def test_streaming_extracts_complete_invokes():
    parser = make_parser()
    full_text = build_tool_call("search", {"query": "deepseek v4"})

    deltas = stream(parser, full_text, chunk_size=5)

    names = [
        tool_call.function.name
        for delta in deltas
        if delta.tool_calls
        for tool_call in delta.tool_calls
    ]
    assert names == ["search"]
    assert json.loads(reconstruct_args(deltas)) == {"query": "deepseek v4"}
```
**EN:** Checks Streaming Extracts Complete Invokes under a focused test scenario. The body exercises logic via `make_parser`, `build_tool_call`, `stream` before asserting the expected outcome.
**CN:** 该测试用例验证 Streaming Extracts Complete Invokes 在特定场景下的行为。 函数体会先通过 `make_parser`, `build_tool_call`, `stream` 驱动目标逻辑，再断言预期结果。

### Test: test_get_vllm_registry_structural_tag_returns_structural_tag (lines 172-205)
```python
def test_get_vllm_registry_structural_tag_returns_structural_tag(
    sample_tools: list[ChatCompletionToolsParam],
) -> None:
    parser = make_parser()
    req = ChatCompletionRequest(
        messages=[],
        model="m",
        tools=sample_tools,
        tool_choice="auto",
    )
    tag = parser.get_structural_tag(req)
    assert isinstance(tag, StructuralTag)

    req = ChatCompletionRequest(
        messages=[],
        model="m",
        tools=sample_tools,
        tool_choice="required",
    )
    tag = parser.get_structural_tag(req)
    assert isinstance(tag, StructuralTag)

    if sample_tools:
        tool = sample_tools[0]
        req = ChatCompletionRequest(
            messages=[],
            model="m",
            tools=sample_tools,
        )
        req.tool_choice = ChatCompletionNamedToolChoiceParam(
            function=ChatCompletionNamedFunction(name=tool.function.name)
        )
        tag = parser.get_structural_tag(req)
        assert isinstance(tag, StructuralTag)
```
**EN:** Checks Get vLLM Registry Structural Tag Returns Structural Tag under a focused test scenario. The body exercises logic via `make_parser`, `ChatCompletionRequest`, `parser.get_structural_tag` before asserting the expected outcome.
**CN:** 该测试用例验证 Get vLLM Registry Structural Tag Returns Structural Tag 在特定场景下的行为。 函数体会先通过 `make_parser`, `ChatCompletionRequest`, `parser.get_structural_tag` 驱动目标逻辑，再断言预期结果。

### Test: test_extract_tool_calls_arguments_wrapper (lines 208-238)
```python
def test_extract_tool_calls_arguments_wrapper():
    mock_tokenizer = MagicMock()
    mock_tokenizer.get_vocab.return_value = {}

    tool = ChatCompletionToolsParam(
        type="function",
        function={
            "name": "get_weather",
            "parameters": {
                "type": "object",
                "properties": {"location": {"type": "string"}},
            },
        },
    )

    parser = DeepSeekV4ToolParser(mock_tokenizer, tools=[tool])
    request = MagicMock()
    request.tools = [tool]

    model_output = (
        f"{TC_START}"
        f'{INV_START}get_weather">'
        f'{PARAM_START}arguments" string="false">{{"location":"Beijing"}}{PARAM_END}'
        f"{INV_END}"
        f"{TC_END}"
    )

    result = parser.extract_tool_calls(model_output, request)
    assert result.tools_called
    args = json.loads(result.tool_calls[0].function.arguments)
    assert args == {"location": "Beijing"}
```
**EN:** Checks Extract Tool Calls Arguments Wrapper under a focused test scenario. The body exercises logic via `MagicMock`, `ChatCompletionToolsParam`, `DeepSeekV4ToolParser` before asserting the expected outcome.
**CN:** 该测试用例验证 Extract Tool Calls Arguments Wrapper 在特定场景下的行为。 函数体会先通过 `MagicMock`, `ChatCompletionToolsParam`, `DeepSeekV4ToolParser` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
build_tool_call
stream
reconstruct_args
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
- **Third-party / 第三方依赖**: `pytest`, `xgrammar`
- **vLLM internal / vLLM 内部依赖**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.tool_parsers`, `vllm.tool_parsers.deepseekv4_tool_parser`
