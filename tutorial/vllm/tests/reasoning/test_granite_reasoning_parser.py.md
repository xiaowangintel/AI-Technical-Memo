# test_granite_reasoning_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/reasoning/test_granite_reasoning_parser.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Granite Reasoning Parser behavior in the Reasoning test area through focused pytest scenarios. It focuses on scenarios such as Reasoning, Streaming Subcases. / 该文件在 Reasoning 测试域中，通过有针对性的 pytest 场景验证 Granite Reasoning Parser 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-123)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import pytest
from transformers import AutoTokenizer

from tests.reasoning.utils import DeltaMessage, run_reasoning_extraction
from vllm.reasoning import ReasoningParser, ReasoningParserManager

parser_name = "granite"
START_REASONING = "Here is my thought process:"
START_RESPONSE = "Here is my response:"

SIMPLE_REASONING = {
    "output": f"{START_REASONING}This is a reasoning section{START_RESPONSE}This is the rest",  # noqa: E501
    "reasoning": "This is a reasoning section",
    "content": "This is the rest",
}
COMPLETE_REASONING = {
    "output": f"{START_REASONING}This is a reasoning section{START_RESPONSE}",
# ... omitted for brevity ...
    pytest.param(
        True,
        COMPLETE_REASONING_WITH_THINK,
        id="complete_reasoning_with_think_streaming",
    ),
    pytest.param(
        True,
        MULTIPLE_LINES_WITH_THINK,
        id="multiple_lines_with_think_streaming",
    ),
]

# Global tokenizer initialization to avoid repeated loading
tokenizer = AutoTokenizer.from_pretrained("facebook/opt-125m")
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `pytest`, `transformers`, `vllm.reasoning`, `tests.reasoning.utils`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Test: test_reasoning (lines 126-145)
```python
@pytest.mark.parametrize("streaming, param_dict", TEST_CASES)
def test_reasoning(
    streaming: bool,
    param_dict: dict,
):
    output = tokenizer.tokenize(param_dict["output"])
    # decode everything to tokens
    output_tokens: list[str] = [
        tokenizer.convert_tokens_to_string([token]) for token in output
    ]
    parser: ReasoningParser = ReasoningParserManager.get_reasoning_parser(parser_name)(
        tokenizer
    )

    reasoning, content = run_reasoning_extraction(
        parser, output_tokens, streaming=streaming
    )

    assert reasoning == param_dict["reasoning"]
    assert content == param_dict["content"]
```
**EN:** Checks Reasoning under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `tokenizer.tokenize`, `ReasoningParserManager.get_reasoning_parser(parser_name)` before asserting the expected outcome.
**CN:** 该测试用例验证 Reasoning 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `tokenizer.tokenize`, `ReasoningParserManager.get_reasoning_parser(parser_name)` 驱动目标逻辑，再断言预期结果。

### Constants / assignments (lines 154-160)
```python
STREAMING_1 = {
    "previous_text": None,
    "current_text": "Here",
    "delta_text": "Here",
    "reasoning": None,
    "content": None,
}
```
**EN:** Defines shared constants or configuration objects like `STREAMING_1`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `STREAMING_1`），供后续测试重复使用。

### Test: test_streaming_subcases (lines 314-344)
```python
@pytest.mark.parametrize("param_dict", STREAMING_SUBCASES)
def test_streaming_subcases(param_dict):
    # Get all of the token IDs
    previous_token_ids = (
        tokenizer.encode(param_dict["previous_text"])
        if param_dict["previous_text"] is not None
        else []
    )
    current_token_ids = tokenizer.encode(param_dict["current_text"])
    delta_token_ids = tokenizer.encode(param_dict["delta_text"])

    parser: ReasoningParser = ReasoningParserManager.get_reasoning_parser(parser_name)(
        tokenizer
    )

    response = parser.extract_reasoning_streaming(
        previous_text=param_dict["previous_text"],
        current_text=param_dict["current_text"],
        delta_text=param_dict["delta_text"],
        previous_token_ids=previous_token_ids,
        current_token_ids=current_token_ids,
        delta_token_ids=delta_token_ids,
    )
    # Streaming currently expects at least one of reasoning content / content,
    # so the response should return None in that case.
    if param_dict["reasoning"] is None and param_dict["content"] is None:
        assert response is None
    else:
        assert isinstance(response, DeltaMessage)
        assert param_dict["reasoning"] == response.reasoning
        assert param_dict["content"] == response.content
```
**EN:** Checks Streaming Subcases under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `tokenizer.encode`, `ReasoningParserManager.get_reasoning_parser(parser_name)` before asserting the expected outcome.
**CN:** 该测试用例验证 Streaming Subcases 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `tokenizer.encode`, `ReasoningParserManager.get_reasoning_parser(parser_name)` 驱动目标逻辑，再断言预期结果。

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
- **Reasoning traces / 推理轨迹**
  - **EN:** The tests inspect reasoning-specific formats or parser behavior in intermediate outputs.
  - **CN:** 这些测试检查中间输出中的推理格式或解析器行为。
- **Assertions / 断言**
  - **EN:** The file relies on explicit assertions to pin down expected state transitions and outputs.
  - **CN:** 该文件通过显式断言固定预期的状态变化与输出结果。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `pytest`, `transformers`
- **vLLM internal / vLLM 内部依赖**: `vllm.reasoning`
- **Local test utilities / 本地测试辅助**: `tests.reasoning.utils`
