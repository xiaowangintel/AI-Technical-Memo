# test_gigachat3_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/tool_parsers/test_gigachat3_tool_parser.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Gigachat3 Tool Parser behavior in the Tool Parsers test area through focused pytest scenarios. It focuses on scenarios such as Default Tokenizer, Gigachat Tokenizer, No Tool Call. / 该文件在 Tool Parsers 测试域中，通过有针对性的 pytest 场景验证 Gigachat3 Tool Parser 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-15)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import json

import pytest
from transformers import AutoTokenizer

from tests.tool_parsers.utils import (
    run_tool_extraction,
    run_tool_extraction_streaming,
)
from vllm.entrypoints.openai.engine.protocol import FunctionCall
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers import ToolParser, ToolParserManager
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `json`, `pytest`, `transformers`, `vllm.entrypoints.openai.engine.protocol`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Fixture: default_tokenizer (lines 18-22)
```python
@pytest.fixture(scope="function")
def default_tokenizer() -> TokenizerLike:
    """Override module-scoped default_tokenizer because gigachat tests
    mutate the tokenizer via ``add_tokens``."""
    return AutoTokenizer.from_pretrained("gpt2")
```
**EN:** Override module-scoped default_tokenizer because gigachat tests mutate the tokenizer via ``add_tokens``. The fixture mainly builds or returns values through `pytest.fixture`, `AutoTokenizer.from_pretrained`.
**CN:** 该代码块定义 pytest 夹具 `default_tokenizer`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `AutoTokenizer.from_pretrained` 构造或返回测试所需的值。

### Constants / assignments (lines 25-25)
```python
MSG_SEP_TOKEN = "<|message_sep|>\n\n"
```
**EN:** Defines shared constants or configuration objects like `MSG_SEP_TOKEN`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `MSG_SEP_TOKEN`），供后续测试重复使用。

### Fixture: fixture_gigachat_tokenizer (lines 104-114)
```python
@pytest.fixture(name="gigachat_tokenizer")
def fixture_gigachat_tokenizer(default_tokenizer: TokenizerLike):
    default_tokenizer.add_tokens(
        [
            MSG_SEP_TOKEN,
            ROLE_SEP_TOKEN,
            TOOL_HEADER_GIGACHAT31,
            EOS_TOKEN,
        ]
    )
    return default_tokenizer
```
**EN:** Provides a pytest fixture for Gigachat Tokenizer. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `pytest.fixture`, `default_tokenizer.add_tokens`.
**CN:** 该代码块定义 pytest 夹具 `fixture_gigachat_tokenizer`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `default_tokenizer.add_tokens` 构造或返回测试所需的值。

### Test: test_no_tool_call (lines 117-127)
```python
@pytest.mark.parametrize("streaming", [True, False])
def test_no_tool_call(streaming: bool, gigachat_tokenizer: TokenizerLike):
    tool_parser: ToolParser = ToolParserManager.get_tool_parser("gigachat3")(
        gigachat_tokenizer
    )
    model_output = "How can I help you today?"
    content, tool_calls = run_tool_extraction(
        tool_parser, model_output, streaming=streaming
    )
    assert content == model_output
    assert len(tool_calls) == 0
```
**EN:** Checks No Tool Call under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `ToolParserManager.get_tool_parser('gigachat3')`, `run_tool_extraction` before asserting the expected outcome.
**CN:** 该测试用例验证 No Tool Call 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `ToolParserManager.get_tool_parser('gigachat3')`, `run_tool_extraction` 驱动目标逻辑，再断言预期结果。

### Test: test_tool_call (lines 274-299)
```python
@pytest.mark.parametrize(
    "streaming, model_output, expected_tool_calls, expected_content", TEST_CASES
)
def test_tool_call(
    streaming: bool,
    model_output: str,
    expected_tool_calls: list[FunctionCall],
    expected_content: str | None,
    gigachat_tokenizer: TokenizerLike,
):
    tool_parser: ToolParser = ToolParserManager.get_tool_parser("gigachat3")(
        gigachat_tokenizer
    )
    content, tool_calls = run_tool_extraction(
        tool_parser, model_output, streaming=streaming
    )
    if content == "":
        content = None
    assert content == expected_content
    assert len(tool_calls) == len(expected_tool_calls)
    for actual, expected in zip(tool_calls, expected_tool_calls):
        assert actual.type == "function"
        assert actual.function.name == expected.name
        actual_args = json.loads(actual.function.arguments)
        expected_args = json.loads(expected.arguments)
        assert actual_args == expected_args
```
**EN:** Checks Tool Call under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `ToolParserManager.get_tool_parser('gigachat3')`, `run_tool_extraction` before asserting the expected outcome.
**CN:** 该测试用例验证 Tool Call 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `ToolParserManager.get_tool_parser('gigachat3')`, `run_tool_extraction` 驱动目标逻辑，再断言预期结果。

### Test: test_streaming_tool_call_with_large_steps (lines 302-352)
```python
@pytest.mark.parametrize(
    "model_output_deltas",
    [
        pytest.param(
            [
                CONTENT_TEXT[:3],
                CONTENT_TEXT[3:5],
                CONTENT_TEXT[5:],
                MSG_SEP_TOKEN,
                TOOL_HEADER_GIGACHAT3,
                COMPLEX_FUNCTION_JSON[:40],
                COMPLEX_FUNCTION_JSON[40:-1],
                COMPLEX_FUNCTION_JSON[-1],
            ],
            id="gigachat3",
        ),
        pytest.param(
            [
                CONTENT_TEXT[:3],
# ... omitted for brevity ...
    tool_parser: ToolParser = ToolParserManager.get_tool_parser("gigachat3")(
        gigachat_tokenizer
    )
    reconstructor = run_tool_extraction_streaming(
        tool_parser,
        model_output_deltas,
        assert_one_tool_per_delta=False,
    )
    assert len(reconstructor.tool_calls) == 1
    call = reconstructor.tool_calls[0]
    assert call.type == "function"
    assert call.function.name == "manage_user_memory"
    args_dict = json.loads(call.function.arguments)
    assert args_dict == COMPLEX_ARGS_DICT
```
**EN:** Test that the closing braces are streamed correctly. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `ToolParserManager.get_tool_parser('gigachat3')`, `run_tool_extraction_streaming` before asserting the expected outcome.
**CN:** 该测试用例验证 Streaming Tool Call With Large Steps 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `ToolParserManager.get_tool_parser('gigachat3')`, `run_tool_extraction_streaming` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
Constants / assignments
Constants / assignments
Constants / assignments
Constants / assignments
Constants / assignments
Constants / assignments
Constants / assignments
Constants / assignments
Constants / assignments
Constants / assignments
Constants / assignments
Constants / assignments
Constants / assignments
Constants / assignments
Constants / assignments
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
- **Assertions / 断言**
  - **EN:** The file relies on explicit assertions to pin down expected state transitions and outputs.
  - **CN:** 该文件通过显式断言固定预期的状态变化与输出结果。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`
- **Third-party / 第三方依赖**: `pytest`, `transformers`
- **vLLM internal / vLLM 内部依赖**: `vllm.entrypoints.openai.engine.protocol`, `vllm.tokenizers`, `vllm.tool_parsers`
- **Local test utilities / 本地测试辅助**: `tests.tool_parsers.utils`
