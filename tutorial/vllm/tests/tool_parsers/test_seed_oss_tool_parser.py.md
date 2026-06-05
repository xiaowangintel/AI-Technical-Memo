# test_seed_oss_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/tool_parsers/test_seed_oss_tool_parser.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Seed Oss Tool Parser behavior in the Tool Parsers test area through focused pytest scenarios. It focuses on scenarios such as Seed Oss Tokenizer, Seed Oss Tool Parser, Sample Tools. / 该文件在 Tool Parsers 测试域中，通过有针对性的 pytest 场景验证 Seed Oss Tool Parser 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-24)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# ruff: noqa: E501

import json
from collections.abc import Generator

import pytest

from vllm.entrypoints.openai.chat_completion.protocol import (
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
from vllm.tool_parsers.seed_oss_tool_parser import SeedOssToolParser

# Use a common model that is likely to be available
MODEL = "ByteDance-Seed/Seed-OSS-36B-Instruct"
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `json`, `collections.abc`, `pytest`, `vllm.entrypoints.openai.chat_completion.protocol`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Fixture: seed_oss_tokenizer (lines 27-29)
```python
@pytest.fixture(scope="module")
def seed_oss_tokenizer():
    return get_tokenizer(tokenizer_name=MODEL, trust_remote_code=True)
```
**EN:** Provides a pytest fixture for Seed Oss Tokenizer. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `pytest.fixture`, `get_tokenizer`.
**CN:** 该代码块定义 pytest 夹具 `seed_oss_tokenizer`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `get_tokenizer` 构造或返回测试所需的值。

### Fixture: seed_oss_tool_parser (lines 32-34)
```python
@pytest.fixture
def seed_oss_tool_parser(seed_oss_tokenizer, sample_tools):
    return SeedOssToolParser(seed_oss_tokenizer, tools=sample_tools)
```
**EN:** Provides a pytest fixture for Seed Oss Tool Parser. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `SeedOssToolParser`.
**CN:** 该代码块定义 pytest 夹具 `seed_oss_tool_parser`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `SeedOssToolParser` 构造或返回测试所需的值。

### Fixture: sample_tools (lines 37-74)
```python
@pytest.fixture
def sample_tools():
    return [
        ChatCompletionToolsParam(
            type="function",
            function={
                "name": "get_weather",
                "description": "Get current temperature for a given location.",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "location": {
                            "type": "string",
                            "description": "City and country e.g. Bogotá, Colombia",
                        },
                        "unit": {
                            "type": "string",
                            "description": "this is the unit of temperature",
                        },
# ... omitted for brevity ...
                    "type": "object",
                    "properties": {
                        "temperature": {
                            "type": "number",
                            "description": "temperature in celsius",
                        }
                    },
                    "required": ["temperature"],
                    "additionalProperties": False,
                },
                "strict": True,
            },
        ),
    ]
```
**EN:** Provides a pytest fixture for Sample Tools. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `ChatCompletionToolsParam`.
**CN:** 该代码块定义 pytest 夹具 `sample_tools`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `ChatCompletionToolsParam` 构造或返回测试所需的值。

### Helper: assert_tool_calls (lines 77-92)
```python
def assert_tool_calls(
    actual_tool_calls: list[ToolCall], expected_tool_calls: list[ToolCall]
):
    assert len(actual_tool_calls) == len(expected_tool_calls)

    for actual_tool_call, expected_tool_call in zip(
        actual_tool_calls, expected_tool_calls
    ):
        # Seed-OSS tool call will not generate id
        assert actual_tool_call.type == "function"
        assert actual_tool_call.function == expected_tool_call.function

        assert actual_tool_call.function.name == expected_tool_call.function.name
        assert (
            actual_tool_call.function.arguments == expected_tool_call.function.arguments
        )
```
**EN:** Implements a reusable helper for Tool Calls, reducing duplication across related tests. It coordinates operations such as `zip`, `len`.
**CN:** 该辅助函数为 Tool Calls 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `zip`, `len` 等操作。

### Test: test_extract_tool_calls_no_tools (lines 95-103)
```python
def test_extract_tool_calls_no_tools(seed_oss_tool_parser):
    model_output = "This is a test response without any tool calls"
    extracted_tool_calls = seed_oss_tool_parser.extract_tool_calls(
        model_output, request=None
    )  # type: ignore[arg-type]

    assert not extracted_tool_calls.tools_called
    assert extracted_tool_calls.tool_calls == []
    assert extracted_tool_calls.content == model_output
```
**EN:** Checks Extract Tool Calls No Tools under a focused test scenario. The body exercises logic via `seed_oss_tool_parser.extract_tool_calls` before asserting the expected outcome.
**CN:** 该测试用例验证 Extract Tool Calls No Tools 在特定场景下的行为。 函数体会先通过 `seed_oss_tool_parser.extract_tool_calls` 驱动目标逻辑，再断言预期结果。

### Test: test_extract_tool_calls (lines 106-235)
```python
@pytest.mark.parametrize(
    ids=[
        "tool_call_0_thinking_budget",
        "tool_call_512_thinking_budget",
        "tool_call_unlimited_thinking_budget",
    ],
    argnames=["model_output", "expected_tool_calls", "expected_content"],
    argvalues=[
        (
            """<seed:tool_call>\n<function=get_weather>\n"""
            """<parameter=location>Barcelona, Spain</parameter>\n</function>\n</seed:tool_call>""",
            [
                ToolCall(
                    function=FunctionCall(
                        name="get_weather",
                        arguments=json.dumps(
                            {
                                "location": "Barcelona, Spain",
                            },
# ... omitted for brevity ...
    sample_tools,
    model_output,
    expected_tool_calls,
    expected_content,
):
    request = ChatCompletionRequest(model=MODEL, messages=[], tools=sample_tools)
    extracted_tool_calls = seed_oss_tool_parser.extract_tool_calls(
        model_output, request=request
    )  # type: ignore[arg-type]
    assert extracted_tool_calls.tools_called

    assert_tool_calls(extracted_tool_calls.tool_calls, expected_tool_calls)

    assert extracted_tool_calls.content == expected_content
```
**EN:** Checks Extract Tool Calls under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `ChatCompletionRequest`, `seed_oss_tool_parser.extract_tool_calls` before asserting the expected outcome.
**CN:** 该测试用例验证 Extract Tool Calls 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `ChatCompletionRequest`, `seed_oss_tool_parser.extract_tool_calls` 驱动目标逻辑，再断言预期结果。

### Test: test_streaming_tool_calls_no_tools (lines 238-254)
```python
def test_streaming_tool_calls_no_tools(seed_oss_tool_parser):
    model_output = "This is a test response without any tool calls"

    result = seed_oss_tool_parser.extract_tool_calls_streaming(
        previous_text="his is a test response",
        current_text=model_output,
        delta_text=" without any tool calls.",
        previous_token_ids=[],
        current_token_ids=[],
        delta_token_ids=[],
        request=None,
    )

    # Should return the delta text as content
    assert result is not None
    assert hasattr(result, "content")
    assert result.content == " without any tool calls."
```
**EN:** Checks Streaming Tool Calls No Tools under a focused test scenario. The body exercises logic via `seed_oss_tool_parser.extract_tool_calls_streaming`, `hasattr` before asserting the expected outcome.
**CN:** 该测试用例验证 Streaming Tool Calls No Tools 在特定场景下的行为。 函数体会先通过 `seed_oss_tool_parser.extract_tool_calls_streaming`, `hasattr` 驱动目标逻辑，再断言预期结果。

### Helper: stream_delta_message_generator (lines 257-305)
```python
def stream_delta_message_generator(
    seed_oss_tool_parser: SeedOssToolParser,
    seed_oss_tokenizer: TokenizerLike,
    model_output: str,
    request: ChatCompletionRequest | None = None,
) -> Generator[DeltaMessage, None, None]:
    all_token_ids = seed_oss_tokenizer.encode(model_output, add_special_tokens=False)

    previous_text = ""
    previous_tokens = None
    prefix_offset = 0
    read_offset = 0
    for i, delta_token in enumerate(all_token_ids):
        delta_token_ids = [delta_token]
        previous_token_ids = all_token_ids[:i]
        current_token_ids = all_token_ids[: i + 1]

        (new_tokens, delta_text, new_prefix_offset, new_read_offset) = (
            detokenize_incrementally(
# ... omitted for brevity ...
            previous_token_ids,
            current_token_ids,
            delta_token_ids,
            request=request,
        )
        if delta_message:
            yield delta_message

        previous_text = current_text
        previous_tokens = (
            previous_tokens + new_tokens if previous_tokens else new_tokens
        )
        prefix_offset = new_prefix_offset
        read_offset = new_read_offset
```
**EN:** Implements a reusable helper for Stream Delta Message Generator, reducing duplication across related tests. It coordinates operations such as `seed_oss_tokenizer.encode`, `enumerate`, `detokenize_incrementally`.
**CN:** 该辅助函数为 Stream Delta Message Generator 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `seed_oss_tokenizer.encode`, `enumerate`, `detokenize_incrementally` 等操作。

### Test: test_streaming_tool_calls (lines 308-497)
```python
@pytest.mark.parametrize(
    ids=[
        "tool_call_0_thinking_budget",
        "tool_call_512_thinking_budget",
        "tool_call_unlimited_thinking_budget",
    ],
    argnames=["model_output", "expected_tool_calls", "expected_content"],
    argvalues=[
        (
            """<seed:think>\n</seed:cot_budget_reflect>\n</seed:cot_budget_reflect>\n"""
            """The current thinking budget is 0, so I will directly start answering the question.\n</seed:think>\n"""
            """<seed:tool_call>\n<function=get_weather>\n"""
            """<parameter=location>Barcelona, Spain</parameter>\n</function>\n</seed:tool_call>""",
            [
                ToolCall(
                    function=FunctionCall(
                        name="get_weather",
                        arguments=json.dumps(
                            {
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
**EN:** Test incremental streaming behavior Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `ChatCompletionRequest`, `stream_delta_message_generator` before asserting the expected outcome.
**CN:** 该测试用例验证 Streaming Tool Calls 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `ChatCompletionRequest`, `stream_delta_message_generator` 驱动目标逻辑，再断言预期结果。

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
- **Sampling behavior / 采样行为**
  - **EN:** The assertions target decoding controls and the shape of generated outputs.
  - **CN:** 这些断言关注解码控制参数以及生成结果的形态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `collections.abc`
- **Third-party / 第三方依赖**: `pytest`
- **vLLM internal / vLLM 内部依赖**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.tokenizers`, `vllm.tokenizers.detokenizer_utils`, `vllm.tool_parsers.seed_oss_tool_parser`
