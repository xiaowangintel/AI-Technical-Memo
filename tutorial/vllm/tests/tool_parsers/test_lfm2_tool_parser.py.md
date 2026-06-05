# test_lfm2_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/tool_parsers/test_lfm2_tool_parser.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Lfm2 Tool Parser behavior in the Tool Parsers test area through focused pytest scenarios. It focuses on scenarios such as Lfm2 Tokenizer, Wrap, No Tool Call. / 该文件在 Tool Parsers 测试域中，通过有针对性的 pytest 场景验证 Lfm2 Tool Parser 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-76)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

from unittest.mock import MagicMock, patch

import pytest
from transformers import AutoTokenizer

from tests.tool_parsers.utils import (
    run_tool_extraction,
    run_tool_extraction_streaming,
)
from vllm.entrypoints.openai.engine.protocol import FunctionCall
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers import ToolParser, ToolParserManager

TOOL_CALL_START = "<|tool_call_start|>"
TOOL_CALL_END = "<|tool_call_end|>"

# ... omitted for brevity ...
)
DOTTED_NAME_FUNCTION_OUTPUT = (
    "grocery.orderIngredients("
    "ingredientList=[{'name': 'Lasagna noodles', 'amount': 250, 'unit': 'g'}], "
    "deliveryAddress='845 Willow Lane, Springfield, IL 62704')"
)
DOTTED_NAME_FUNCTION_CALL = FunctionCall(
    name="grocery.orderIngredients",
    arguments=(
        '{"ingredientList": ['
        '{"name": "Lasagna noodles", "amount": 250, "unit": "g"}], '
        '"deliveryAddress": "845 Willow Lane, Springfield, IL 62704"}'
    ),
)
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `unittest.mock`, `json`, `pytest`, `transformers`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Fixture: lfm2_tokenizer (lines 79-81)
```python
@pytest.fixture(scope="module")
def lfm2_tokenizer() -> TokenizerLike:
    return AutoTokenizer.from_pretrained("LiquidAI/LFM2.5-1.2B-Instruct")
```
**EN:** Provides a pytest fixture for Lfm2 Tokenizer. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `pytest.fixture`, `AutoTokenizer.from_pretrained`.
**CN:** 该代码块定义 pytest 夹具 `lfm2_tokenizer`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `AutoTokenizer.from_pretrained` 构造或返回测试所需的值。

### Helper: _wrap (lines 84-89)
```python
def _wrap(tool_text: str, content_after: str = "") -> str:
    """Wrap pythonic tool call in LFM2.5 sentinel tokens."""
    result = f"{TOOL_CALL_START}[{tool_text}]{TOOL_CALL_END}"
    if content_after:
        result += f"\n{content_after}"
    return result
```
**EN:** Wrap pythonic tool call in LFM2.5 sentinel tokens.
**CN:** 该辅助函数为 Wrap 提供可复用逻辑，用于减少相关测试之间的重复代码。

### Test: test_no_tool_call (lines 92-102)
```python
@pytest.mark.parametrize("streaming", [True, False])
def test_no_tool_call(streaming: bool, lfm2_tokenizer: TokenizerLike):
    tool_parser: ToolParser = ToolParserManager.get_tool_parser("lfm2")(lfm2_tokenizer)
    model_output = "How can I help you today?"

    content, tool_calls = run_tool_extraction(
        tool_parser, model_output, streaming=streaming
    )

    assert content == model_output
    assert len(tool_calls) == 0
```
**EN:** Checks No Tool Call under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `ToolParserManager.get_tool_parser('lfm2')`, `run_tool_extraction` before asserting the expected outcome.
**CN:** 该测试用例验证 No Tool Call 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `ToolParserManager.get_tool_parser('lfm2')`, `run_tool_extraction` 驱动目标逻辑，再断言预期结果。

### Constants / assignments (lines 105-230)
```python
TEST_CASES = [
    pytest.param(
        True,
        _wrap(SIMPLE_FUNCTION_OUTPUT),
        [SIMPLE_FUNCTION_CALL],
        None,
        id="simple_streaming",
    ),
    pytest.param(
        False,
        _wrap(SIMPLE_FUNCTION_OUTPUT),
        [SIMPLE_FUNCTION_CALL],
        None,
        id="simple_nonstreaming",
    ),
    pytest.param(
        True,
        _wrap(MORE_TYPES_FUNCTION_OUTPUT),
        [MORE_TYPES_FUNCTION_CALL],
# ... omitted for brevity ...
        True,
        _wrap(DOTTED_NAME_FUNCTION_OUTPUT),
        [DOTTED_NAME_FUNCTION_CALL],
        None,
        id="dotted_name_streaming",
    ),
    pytest.param(
        False,
        _wrap(DOTTED_NAME_FUNCTION_OUTPUT),
        [DOTTED_NAME_FUNCTION_CALL],
        None,
        id="dotted_name_nonstreaming",
    ),
]
```
**EN:** Defines shared constants or configuration objects like `TEST_CASES`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `TEST_CASES`），供后续测试重复使用。

### Test: test_tool_call (lines 233-255)
```python
@pytest.mark.parametrize(
    "streaming, model_output, expected_tool_calls, expected_content",
    TEST_CASES,
)
def test_tool_call(
    streaming: bool,
    model_output: str,
    expected_tool_calls: list[FunctionCall],
    expected_content: str | None,
    lfm2_tokenizer: TokenizerLike,
):
    tool_parser: ToolParser = ToolParserManager.get_tool_parser("lfm2")(lfm2_tokenizer)

    content, tool_calls = run_tool_extraction(
        tool_parser, model_output, streaming=streaming
    )

    if expected_content and not streaming:
        assert content == expected_content
    assert len(tool_calls) == len(expected_tool_calls)
    for actual, expected in zip(tool_calls, expected_tool_calls):
        assert actual.type == "function"
        assert actual.function == expected
```
**EN:** Checks Tool Call under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `ToolParserManager.get_tool_parser('lfm2')`, `run_tool_extraction` before asserting the expected outcome.
**CN:** 该测试用例验证 Tool Call 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `ToolParserManager.get_tool_parser('lfm2')`, `run_tool_extraction` 驱动目标逻辑，再断言预期结果。

### Test: test_streaming_tool_call_with_large_steps (lines 258-273)
```python
def test_streaming_tool_call_with_large_steps(lfm2_tokenizer: TokenizerLike):
    tool_parser: ToolParser = ToolParserManager.get_tool_parser("lfm2")(lfm2_tokenizer)
    model_output_deltas = [
        f"{TOOL_CALL_START}[get_candidate_status(candidate_id='12345'), "
        f"{PARAMETERLESS_FUNCTION_OUTPUT}, "
        f"{EMPTY_LIST_FUNCTION_OUTPUT}]{TOOL_CALL_END}",
    ]

    reconstructor = run_tool_extraction_streaming(
        tool_parser, model_output_deltas, assert_one_tool_per_delta=False
    )

    assert len(reconstructor.tool_calls) == 3
    assert reconstructor.tool_calls[0].function == SIMPLE_FUNCTION_CALL
    assert reconstructor.tool_calls[1].function == PARAMETERLESS_FUNCTION_CALL
    assert reconstructor.tool_calls[2].function == EMPTY_LIST_FUNCTION_CALL
```
**EN:** Checks Streaming Tool Call With Large Steps under a focused test scenario. The body exercises logic via `ToolParserManager.get_tool_parser('lfm2')`, `run_tool_extraction_streaming`, `ToolParserManager.get_tool_parser` before asserting the expected outcome.
**CN:** 该测试用例验证 Streaming Tool Call With Large Steps 在特定场景下的行为。 函数体会先通过 `ToolParserManager.get_tool_parser('lfm2')`, `run_tool_extraction_streaming`, `ToolParserManager.get_tool_parser` 驱动目标逻辑，再断言预期结果。

### Test: test_streaming_full_block_and_trailing_in_single_delta (lines 276-289)
```python
def test_streaming_full_block_and_trailing_in_single_delta(
    lfm2_tokenizer: TokenizerLike,
):
    """The entire <|tool_call_start|>[...]<|tool_call_end|> block plus
    trailing assistant text arrive in one delta. Trailing content must
    still be emitted — not silently dropped."""
    tool_parser: ToolParser = ToolParserManager.get_tool_parser("lfm2")(lfm2_tokenizer)
    full_text = f"{TOOL_CALL_START}[{SIMPLE_FUNCTION_OUTPUT}]{TOOL_CALL_END}\nDone."

    reconstructor = run_tool_extraction_streaming(tool_parser, [full_text])

    assert len(reconstructor.tool_calls) == 1
    assert reconstructor.tool_calls[0].function == SIMPLE_FUNCTION_CALL
    assert "Done." in reconstructor.other_content
```
**EN:** The entire <|tool_call_start|>[...]<|tool_call_end|> block plus trailing assistant text arrive in one delta. The body exercises logic via `ToolParserManager.get_tool_parser('lfm2')`, `run_tool_extraction_streaming`, `ToolParserManager.get_tool_parser` before asserting the expected outcome.
**CN:** 该测试用例验证 Streaming Full Block And Trailing In Single Delta 在特定场景下的行为。 函数体会先通过 `ToolParserManager.get_tool_parser('lfm2')`, `run_tool_extraction_streaming`, `ToolParserManager.get_tool_parser` 驱动目标逻辑，再断言预期结果。

### Test: test_streaming_leading_content_and_full_block_in_single_delta (lines 292-306)
```python
def test_streaming_leading_content_and_full_block_in_single_delta(
    lfm2_tokenizer: TokenizerLike,
):
    """Leading assistant text plus the entire tool block arrive in one
    delta. Leading content must be emitted — not silently dropped."""
    tool_parser: ToolParser = ToolParserManager.get_tool_parser("lfm2")(lfm2_tokenizer)
    full_text = (
        f"Let me check. {TOOL_CALL_START}[{SIMPLE_FUNCTION_OUTPUT}]{TOOL_CALL_END}"
    )

    reconstructor = run_tool_extraction_streaming(tool_parser, [full_text])

    assert len(reconstructor.tool_calls) == 1
    assert reconstructor.tool_calls[0].function == SIMPLE_FUNCTION_CALL
    assert "Let me check." in reconstructor.other_content
```
**EN:** Leading assistant text plus the entire tool block arrive in one delta. The body exercises logic via `ToolParserManager.get_tool_parser('lfm2')`, `run_tool_extraction_streaming`, `ToolParserManager.get_tool_parser` before asserting the expected outcome.
**CN:** 该测试用例验证 Streaming Leading Content And Full Block In Single Delta 在特定场景下的行为。 函数体会先通过 `ToolParserManager.get_tool_parser('lfm2')`, `run_tool_extraction_streaming`, `ToolParserManager.get_tool_parser` 驱动目标逻辑，再断言预期结果。

### Test: test_streaming_leading_block_and_trailing_in_single_delta (lines 309-325)
```python
def test_streaming_leading_block_and_trailing_in_single_delta(
    lfm2_tokenizer: TokenizerLike,
):
    """Leading text + complete tool block + trailing text in one delta.
    Both leading and trailing content must be preserved."""
    tool_parser: ToolParser = ToolParserManager.get_tool_parser("lfm2")(lfm2_tokenizer)
    full_text = (
        "Let me check. "
        f"{TOOL_CALL_START}[{SIMPLE_FUNCTION_OUTPUT}]{TOOL_CALL_END}\nDone."
    )

    reconstructor = run_tool_extraction_streaming(tool_parser, [full_text])

    assert len(reconstructor.tool_calls) == 1
    assert reconstructor.tool_calls[0].function == SIMPLE_FUNCTION_CALL
    assert "Let me check." in reconstructor.other_content
    assert "Done." in reconstructor.other_content
```
**EN:** Leading text + complete tool block + trailing text in one delta. The body exercises logic via `ToolParserManager.get_tool_parser('lfm2')`, `run_tool_extraction_streaming`, `ToolParserManager.get_tool_parser` before asserting the expected outcome.
**CN:** 该测试用例验证 Streaming Leading Block And Trailing In Single Delta 在特定场景下的行为。 函数体会先通过 `ToolParserManager.get_tool_parser('lfm2')`, `run_tool_extraction_streaming`, `ToolParserManager.get_tool_parser` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
test_echoed_tool_call_body_not_leaked_to_content
test_streaming_char_by_char_multi_dict_list
test_streaming_dotted_name_in_single_delta
test_adjust_request_disables_skip_special_tokens
test_regex_timeout_handling
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
- **Standard library / 标准库**: `unittest.mock`, `json`
- **Third-party / 第三方依赖**: `pytest`, `transformers`
- **vLLM internal / vLLM 内部依赖**: `vllm.entrypoints.openai.engine.protocol`, `vllm.tokenizers`, `vllm.tool_parsers`, `vllm.entrypoints.openai.chat_completion.protocol`
- **Local test utilities / 本地测试辅助**: `tests.tool_parsers.utils`
