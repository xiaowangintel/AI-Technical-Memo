# test_hermes_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/tool_parsers/test_hermes_tool_parser.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Hermes Tool Parser behavior in the Tool Parsers test area through focused pytest scenarios. It focuses on scenarios such as Qwen Tokenizer, Hermes Parser, Any Chat Request. / 该文件在 Tool Parsers 测试域中，通过有针对性的 pytest 场景验证 Hermes Tool Parser 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-21)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import json

import pytest

from vllm.entrypoints.openai.chat_completion.protocol import ChatCompletionRequest
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers.abstract_tool_parser import ToolParser
from vllm.tool_parsers.granite4_tool_parser import Granite4ToolParser
from vllm.tool_parsers.hermes_tool_parser import Hermes2ProToolParser

CONFIGS = {
    "llama": {
        "tool_parser": Hermes2ProToolParser,
    },
    "granite4": {
        "tool_parser": Granite4ToolParser,
    },
}
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `json`, `pytest`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.tokenizers`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Fixture: qwen_tokenizer (lines 24-28)
```python
@pytest.fixture
def qwen_tokenizer() -> TokenizerLike:
    from vllm.tokenizers import get_tokenizer

    return get_tokenizer("Qwen/Qwen3-32B")
```
**EN:** Provides a pytest fixture for Qwen Tokenizer. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `get_tokenizer`.
**CN:** 该代码块定义 pytest 夹具 `qwen_tokenizer`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `get_tokenizer` 构造或返回测试所需的值。

### Fixture: hermes_parser (lines 31-34)
```python
@pytest.fixture(params=CONFIGS.keys())
def hermes_parser(request, qwen_tokenizer: TokenizerLike) -> ToolParser:
    config = CONFIGS[request.param]
    return config["tool_parser"](qwen_tokenizer)
```
**EN:** Provides a pytest fixture for Hermes Parser. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `pytest.fixture`, `config['tool_parser']`, `CONFIGS.keys`.
**CN:** 该代码块定义 pytest 夹具 `hermes_parser`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `config['tool_parser']`, `CONFIGS.keys` 构造或返回测试所需的值。

### Test: test_hermes_parser_streaming_just_forward_text (lines 46-75)
```python
def test_hermes_parser_streaming_just_forward_text(
    qwen_tokenizer: TokenizerLike,
    hermes_parser: ToolParser,
    any_chat_request: ChatCompletionRequest,
) -> None:
    text = """This is some prior text that has nothing to do with tool calling."""
    tokens = qwen_tokenizer.encode(text)
    previous_text = ""
    delta_messages = []
    for token in tokens:
        delta_text = qwen_tokenizer.decode([token])
        current_text = previous_text + delta_text
        delta = hermes_parser.extract_tool_calls_streaming(
            previous_text=previous_text,
            current_text=current_text,
            delta_text=delta_text,
            previous_token_ids=[],
            current_token_ids=[],
            delta_token_ids=[],
            request=any_chat_request,
        )
        previous_text = current_text
        delta_messages.append(delta)

    for delta in delta_messages:
        assert delta is not None
        assert not delta.tool_calls

    print(delta_messages)
    assert "".join([delta.content for delta in delta_messages]) == text
```
**EN:** Checks Hermes Parser Streaming Just Forward Text under a focused test scenario. The body exercises logic via `qwen_tokenizer.encode`, `print`, `qwen_tokenizer.decode` before asserting the expected outcome.
**CN:** 该测试用例验证 Hermes Parser Streaming Just Forward Text 在特定场景下的行为。 函数体会先通过 `qwen_tokenizer.encode`, `print`, `qwen_tokenizer.decode` 驱动目标逻辑，再断言预期结果。

### Test: test_hermes_parser_streaming_failure_case_bug_19056 (lines 78-109)
```python
def test_hermes_parser_streaming_failure_case_bug_19056(
    qwen_tokenizer: TokenizerLike,
    hermes_parser: ToolParser,
    any_chat_request: ChatCompletionRequest,
) -> None:
    text = """<tool_call>
{"name": "final_answer", "arguments": {"trigger": true}}
</tool_call>"""
    tokens = qwen_tokenizer.encode(text)
    previous_text = ""
    delta_messages = []
    for token in tokens:
        text = qwen_tokenizer.decode([token])
        current_text = previous_text + text
        delta = hermes_parser.extract_tool_calls_streaming(
            previous_text=previous_text,
            current_text=current_text,
            delta_text=text,
            previous_token_ids=[],
            current_token_ids=[],
            delta_token_ids=[],
            request=any_chat_request,
        )
        previous_text = current_text
        if delta is not None:
            delta_messages.append(delta)

    assert delta_messages[0].tool_calls[0].function.name == "final_answer"
    tool_call_args = "".join(
        delta.tool_calls[0].function.arguments or "" for delta in delta_messages
    )
    assert tool_call_args == '{"trigger": true}'
```
**EN:** Checks Hermes Parser Streaming Failure Case Bug 19056 under a focused test scenario. The body exercises logic via `qwen_tokenizer.encode`, `''.join`, `qwen_tokenizer.decode` before asserting the expected outcome.
**CN:** 该测试用例验证 Hermes Parser Streaming Failure Case Bug 19056 在特定场景下的行为。 函数体会先通过 `qwen_tokenizer.encode`, `''.join`, `qwen_tokenizer.decode` 驱动目标逻辑，再断言预期结果。

### Test: test_hermes_parser_streaming (lines 112-152)
```python
def test_hermes_parser_streaming(
    qwen_tokenizer: TokenizerLike,
    hermes_parser: ToolParser,
    any_chat_request: ChatCompletionRequest,
) -> None:
    text = '<tool_call>\
{"name": "get_current_temperature",\
"arguments": {"location":\
"San Francisco, California, United States", "unit": "celsius"}}\
</tool_call>'

    tokens = qwen_tokenizer.encode(text)
    previous_text = ""
    delta_messages = []
    for token in tokens:
        text = qwen_tokenizer.decode([token])
        current_text = previous_text + text
        delta = hermes_parser.extract_tool_calls_streaming(
            previous_text=previous_text,
# ... omitted for brevity ...
        if delta is not None:
            delta_messages.append(delta)
    print(delta_messages)
    assert delta_messages[0].tool_calls[0].function.name == "get_current_temperature"
    # load to normalize whitespace
    tool_call_args = json.loads(
        "".join(
            delta.tool_calls[0].function.arguments or "" for delta in delta_messages
        )
    )
    assert tool_call_args == {
        "location": "San Francisco, California, United States",
        "unit": "celsius",
    }
```
**EN:** Checks Hermes Parser Streaming under a focused test scenario. The body exercises logic via `qwen_tokenizer.encode`, `print`, `json.loads` before asserting the expected outcome.
**CN:** 该测试用例验证 Hermes Parser Streaming 在特定场景下的行为。 函数体会先通过 `qwen_tokenizer.encode`, `print`, `json.loads` 驱动目标逻辑，再断言预期结果。

### Helper: _simulate_streaming (lines 155-187)
```python
def _simulate_streaming(
    tokenizer: TokenizerLike,
    parser: ToolParser,
    request: ChatCompletionRequest,
    text: str,
    stream_interval: int = 1,
) -> list:
    """Simulate streaming with a given stream_interval.

    Tokens are batched into chunks of `stream_interval` tokens,
    mimicking how the output processor delivers them.
    Returns a list of non-None DeltaMessages.
    """
    tokens = tokenizer.encode(text)
    previous_text = ""
    delta_messages = []
    for i in range(0, len(tokens), stream_interval):
        chunk_ids = tokens[i : i + stream_interval]
        delta_text = tokenizer.decode(chunk_ids)
        current_text = previous_text + delta_text
        delta = parser.extract_tool_calls_streaming(
            previous_text=previous_text,
            current_text=current_text,
            delta_text=delta_text,
            previous_token_ids=[],
            current_token_ids=[],
            delta_token_ids=chunk_ids,
            request=request,
        )
        previous_text = current_text
        if delta is not None:
            delta_messages.append(delta)
    return delta_messages
```
**EN:** Simulate streaming with a given stream_interval. It coordinates operations such as `tokenizer.encode`, `range`, `len`.
**CN:** 该辅助函数为 Simulate Streaming 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `tokenizer.encode`, `range`, `len` 等操作。

### Test: test_hermes_streaming_tool_call_with_stream_interval (lines 190-217)
```python
@pytest.mark.parametrize("stream_interval", [2, 3, 5, 8])
def test_hermes_streaming_tool_call_with_stream_interval(
    qwen_tokenizer: TokenizerLike,
    any_chat_request: ChatCompletionRequest,
    stream_interval: int,
) -> None:
    """Tool call streaming must produce correct name + args at any interval."""
    text = (
        '<tool_call>{"name": "get_current_temperature", '
        '"arguments": {"location": "San Francisco", "unit": "celsius"}}'
        "</tool_call>"
    )
    parser = Hermes2ProToolParser(qwen_tokenizer)
    deltas = _simulate_streaming(
        qwen_tokenizer, parser, any_chat_request, text, stream_interval
    )

    # Flatten all DeltaToolCalls across all deltas.
    tool_deltas = [tc for d in deltas if d.tool_calls for tc in d.tool_calls]
    assert tool_deltas, "Expected at least one tool call delta"
    assert tool_deltas[0].function.name == "get_current_temperature"

    # Concatenated arguments must be valid JSON matching the original.
    args_str = "".join(tc.function.arguments or "" for tc in tool_deltas)
    assert json.loads(args_str) == {
        "location": "San Francisco",
        "unit": "celsius",
    }
```
**EN:** Tool call streaming must produce correct name + args at any interval. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `Hermes2ProToolParser`, `_simulate_streaming` before asserting the expected outcome.
**CN:** 该测试用例验证 Hermes Streaming Tool Call With Stream Interval 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `Hermes2ProToolParser`, `_simulate_streaming` 驱动目标逻辑，再断言预期结果。

### Test: test_hermes_streaming_content_then_tool_call_with_stream_interval (lines 220-248)
```python
@pytest.mark.parametrize("stream_interval", [2, 3, 5, 8])
def test_hermes_streaming_content_then_tool_call_with_stream_interval(
    qwen_tokenizer: TokenizerLike,
    any_chat_request: ChatCompletionRequest,
    stream_interval: int,
) -> None:
    """Content before a tool call must be fully streamed, then tool call."""
    text = (
        "Sure, let me check the weather."
        '<tool_call>{"name": "get_weather", '
        '"arguments": {"city": "NYC"}}</tool_call>'
    )
    parser = Hermes2ProToolParser(qwen_tokenizer)
    deltas = _simulate_streaming(
        qwen_tokenizer, parser, any_chat_request, text, stream_interval
    )

    content_deltas = [d for d in deltas if d.content]
    tool_deltas = [d for d in deltas if d.tool_calls]

    # Content must reconstruct the prefix.
    content_str = "".join(d.content for d in content_deltas)
    assert content_str == "Sure, let me check the weather."

    # Tool call must be correct.
    tool_calls = [tc for d in tool_deltas for tc in d.tool_calls]
    assert tool_calls[0].function.name == "get_weather"
    args_str = "".join(tc.function.arguments or "" for tc in tool_calls)
    assert json.loads(args_str) == {"city": "NYC"}
```
**EN:** Content before a tool call must be fully streamed, then tool call. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `Hermes2ProToolParser`, `_simulate_streaming` before asserting the expected outcome.
**CN:** 该测试用例验证 Hermes Streaming Content Then Tool Call With Stream Interval 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `Hermes2ProToolParser`, `_simulate_streaming` 驱动目标逻辑，再断言预期结果。

### Test: test_hermes_streaming_multiple_tool_calls_with_stream_interval (lines 251-280)
```python
@pytest.mark.parametrize("stream_interval", [1, 2, 4])
def test_hermes_streaming_multiple_tool_calls_with_stream_interval(
    qwen_tokenizer: TokenizerLike,
    any_chat_request: ChatCompletionRequest,
    stream_interval: int,
) -> None:
    """Multiple sequential tool calls must each be streamed correctly."""
    text = (
        '<tool_call>{"name": "search", "arguments": {"q": "cats"}}</tool_call>'
        '<tool_call>{"name": "search", "arguments": {"q": "dogs"}}</tool_call>'
    )
    parser = Hermes2ProToolParser(qwen_tokenizer)
    deltas = _simulate_streaming(
        qwen_tokenizer, parser, any_chat_request, text, stream_interval
    )

    # Flatten all DeltaToolCalls across all deltas.
    all_tool_calls = [tc for d in deltas if d.tool_calls for tc in d.tool_calls]

    # Separate by tool index.
    tool0 = [tc for tc in all_tool_calls if tc.index == 0]
    tool1 = [tc for tc in all_tool_calls if tc.index == 1]

    assert tool0[0].function.name == "search"
    args0 = "".join(tc.function.arguments or "" for tc in tool0)
    assert json.loads(args0) == {"q": "cats"}

    assert tool1[0].function.name == "search"
    args1 = "".join(tc.function.arguments or "" for tc in tool1)
    assert json.loads(args1) == {"q": "dogs"}
```
**EN:** Multiple sequential tool calls must each be streamed correctly. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `Hermes2ProToolParser`, `_simulate_streaming` before asserting the expected outcome.
**CN:** 该测试用例验证 Hermes Streaming Multiple Tool Calls With Stream Interval 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `Hermes2ProToolParser`, `_simulate_streaming` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
any_chat_request
test_hermes_streaming_boolean_args_with_stream_interval
test_hermes_streaming_just_forward_text_with_stream_interval
test_hermes_parser_non_streaming_no_tool_call
test_hermes_parser_non_streaming_tool_call_between_tags
test_hermes_parser_non_streaming_tool_call_until_eos
test_hermes_parser_non_streaming_tool_call_invalid_json
test_hermes_streaming_content_and_tool_call_in_single_chunk
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
- **vLLM internal / vLLM 内部依赖**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.tokenizers`, `vllm.tool_parsers.abstract_tool_parser`, `vllm.tool_parsers.granite4_tool_parser`, `vllm.tool_parsers.hermes_tool_parser`
