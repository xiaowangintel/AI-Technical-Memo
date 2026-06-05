# test_olmo3_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/tool_parsers/test_olmo3_tool_parser.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Olmo3 Tool Parser behavior in the Tool Parsers test area through focused pytest scenarios. It focuses on scenarios such as No Tool Call, Tool Call, Streaming Tool Call With Large Steps. / 该文件在 Tool Parsers 测试域中，通过有针对性的 pytest 场景验证 Olmo3 Tool Parser 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-68)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

from unittest.mock import MagicMock, patch

import pytest

from tests.tool_parsers.utils import (
    run_tool_extraction,
    run_tool_extraction_streaming,
)
from vllm.entrypoints.openai.engine.protocol import FunctionCall
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers import ToolParser, ToolParserManager

# https://github.com/meta-llama/llama-models/blob/main/models/llama3_2/text_prompt_format.md#model-response-format-1
SIMPLE_FUNCTION_OUTPUT = "get_weather(city='San Francisco', metric='celsius')"
SIMPLE_FUNCTION_CALL = FunctionCall(
    name="get_weather",
# ... omitted for brevity ...
    arguments='{"additional_data": {}}',
)
EMPTY_LIST_FUNCTION_OUTPUT = "do_something_cool(steps=[])"
EMPTY_LIST_FUNCTION_CALL = FunctionCall(
    name="do_something_cool",
    arguments='{"steps": []}',
)
ESCAPED_STRING_FUNCTION_OUTPUT = (
    r"get_weather(city='Martha\'s Vineyard', metric='\"cool units\"')"
)
ESCAPED_STRING_FUNCTION_CALL = FunctionCall(
    name="get_weather",
    arguments='{"city": "Martha\'s Vineyard", "metric": "\\"cool units\\""}',
)
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `unittest.mock`, `pytest`, `vllm.entrypoints.openai.engine.protocol`, `vllm.tokenizers`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Test: test_no_tool_call (lines 71-83)
```python
@pytest.mark.parametrize("streaming", [True, False])
def test_no_tool_call(streaming: bool, default_tokenizer: TokenizerLike):
    tool_parser: ToolParser = ToolParserManager.get_tool_parser("olmo3")(
        default_tokenizer
    )
    model_output = "How can I help you today?"

    content, tool_calls = run_tool_extraction(
        tool_parser, model_output, streaming=streaming
    )

    assert content == model_output
    assert len(tool_calls) == 0
```
**EN:** Checks No Tool Call under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `ToolParserManager.get_tool_parser('olmo3')`, `run_tool_extraction` before asserting the expected outcome.
**CN:** 该测试用例验证 No Tool Call 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `ToolParserManager.get_tool_parser('olmo3')`, `run_tool_extraction` 驱动目标逻辑，再断言预期结果。

### Constants / assignments (lines 86-183)
```python
TEST_CASES = [
    pytest.param(
        True,
        f"<function_calls>{SIMPLE_FUNCTION_OUTPUT}</function_calls>",
        [SIMPLE_FUNCTION_CALL],
        id="simple_streaming",
    ),
    pytest.param(
        False,
        f"<function_calls>{SIMPLE_FUNCTION_OUTPUT}</function_calls>",
        [SIMPLE_FUNCTION_CALL],
        id="simple_nonstreaming",
    ),
    pytest.param(
        True,
        f"<function_calls>{MORE_TYPES_FUNCTION_OUTPUT}</function_calls>",
        [MORE_TYPES_FUNCTION_CALL],
        id="more_types_streaming",
    ),
# ... omitted for brevity ...
    ),
    pytest.param(
        True,
        f"<function_calls>{SIMPLE_FUNCTION_OUTPUT}\n{MORE_TYPES_FUNCTION_OUTPUT}</function_calls>",
        [SIMPLE_FUNCTION_CALL, MORE_TYPES_FUNCTION_CALL],
        id="parallel_calls_streaming",
    ),
    pytest.param(
        False,
        f"<function_calls>{SIMPLE_FUNCTION_OUTPUT}\n{MORE_TYPES_FUNCTION_OUTPUT}</function_calls>",
        [SIMPLE_FUNCTION_CALL, MORE_TYPES_FUNCTION_CALL],
        id="parallel_calls_nonstreaming",
    ),
]
```
**EN:** Defines shared constants or configuration objects like `TEST_CASES`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `TEST_CASES`），供后续测试重复使用。

### Test: test_tool_call (lines 186-205)
```python
@pytest.mark.parametrize("streaming, model_output, expected_tool_calls", TEST_CASES)
def test_tool_call(
    streaming: bool,
    model_output: str,
    expected_tool_calls: list[FunctionCall],
    default_tokenizer: TokenizerLike,
):
    tool_parser: ToolParser = ToolParserManager.get_tool_parser("olmo3")(
        default_tokenizer
    )

    content, tool_calls = run_tool_extraction(
        tool_parser, model_output, streaming=streaming
    )

    assert content is None
    assert len(tool_calls) == len(expected_tool_calls)
    for actual, expected in zip(tool_calls, expected_tool_calls):
        assert actual.type == "function"
        assert actual.function == expected
```
**EN:** Checks Tool Call under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `ToolParserManager.get_tool_parser('olmo3')`, `run_tool_extraction` before asserting the expected outcome.
**CN:** 该测试用例验证 Tool Call 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `ToolParserManager.get_tool_parser('olmo3')`, `run_tool_extraction` 驱动目标逻辑，再断言预期结果。

### Test: test_streaming_tool_call_with_large_steps (lines 208-227)
```python
def test_streaming_tool_call_with_large_steps(default_tokenizer: TokenizerLike):
    tool_parser: ToolParser = ToolParserManager.get_tool_parser("olmo3")(
        default_tokenizer
    )
    model_output_deltas = [
        "<function_calls>get_weather(city='San",
        " Francisco', metric='celsius')\n"
        f"{PARAMETERLESS_FUNCTION_OUTPUT}\n"
        f"{EMPTY_LIST_FUNCTION_OUTPUT}</function_calls>",
    ]

    reconstructor = run_tool_extraction_streaming(
        tool_parser, model_output_deltas, assert_one_tool_per_delta=False
    )

    assert reconstructor.other_content == ""
    assert len(reconstructor.tool_calls) == 3
    assert reconstructor.tool_calls[0].function == SIMPLE_FUNCTION_CALL
    assert reconstructor.tool_calls[1].function == PARAMETERLESS_FUNCTION_CALL
    assert reconstructor.tool_calls[2].function == EMPTY_LIST_FUNCTION_CALL
```
**EN:** Checks Streaming Tool Call With Large Steps under a focused test scenario. The body exercises logic via `ToolParserManager.get_tool_parser('olmo3')`, `run_tool_extraction_streaming`, `ToolParserManager.get_tool_parser` before asserting the expected outcome.
**CN:** 该测试用例验证 Streaming Tool Call With Large Steps 在特定场景下的行为。 函数体会先通过 `ToolParserManager.get_tool_parser('olmo3')`, `run_tool_extraction_streaming`, `ToolParserManager.get_tool_parser` 驱动目标逻辑，再断言预期结果。

### Test: test_regex_timeout_handling (lines 230-251)
```python
@pytest.mark.parametrize("streaming", [False])
def test_regex_timeout_handling(streaming: bool, default_tokenizer: TokenizerLike):
    """test regex timeout is handled gracefully"""
    tool_parser: ToolParser = ToolParserManager.get_tool_parser("olmo3")(
        default_tokenizer
    )

    fake_problematic_input = "hello world[A(A=" + "\t)A(A=,\t" * 2

    # create a mock regex that raises TimeoutError
    mock_regex = MagicMock()
    mock_regex.match.side_effect = TimeoutError("Regex timeout")

    with patch.object(tool_parser, "TOOL_CALL_REGEX", mock_regex):
        content, tool_calls = run_tool_extraction(
            tool_parser, fake_problematic_input, streaming=streaming
        )

        # should treat as regular text when regex times out
        assert content == fake_problematic_input
        assert len(tool_calls) == 0
        mock_regex.match.assert_called_once()
```
**EN:** test regex timeout is handled gracefully Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `ToolParserManager.get_tool_parser('olmo3')`, `MagicMock` before asserting the expected outcome.
**CN:** 该测试用例验证 Regex Timeout Handling 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `ToolParserManager.get_tool_parser('olmo3')`, `MagicMock` 驱动目标逻辑，再断言预期结果。

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
- **Standard library / 标准库**: `unittest.mock`
- **Third-party / 第三方依赖**: `pytest`
- **vLLM internal / vLLM 内部依赖**: `vllm.entrypoints.openai.engine.protocol`, `vllm.tokenizers`, `vllm.tool_parsers`
- **Local test utilities / 本地测试辅助**: `tests.tool_parsers.utils`
