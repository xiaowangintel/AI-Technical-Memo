# test_streaming.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/parser/test_streaming.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Streaming behavior in the Parser test area through focused pytest scenarios. It focuses on scenarios such as Thinkreasoningparser, Tokenizer, Request Obj. / 该文件在 Parser 测试域中，通过有针对性的 pytest 场景验证 Streaming 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-12)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import json

import pytest

from vllm.entrypoints.openai.chat_completion.protocol import ChatCompletionRequest
from vllm.entrypoints.openai.engine.protocol import DeltaMessage
from vllm.parser.abstract_parser import _WrappedParser
from vllm.reasoning.basic_parsers import BaseThinkingReasoningParser
from vllm.tool_parsers.hermes_tool_parser import Hermes2ProToolParser
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `json`, `pytest`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Class: ThinkReasoningParser (lines 15-22)
```python
class ThinkReasoningParser(BaseThinkingReasoningParser):
    @property
    def start_token(self) -> str:
        return "<think>"

    @property
    def end_token(self) -> str:
        return "</think>"
```
**EN:** Groups related scenarios for Thinkreasoningparser.
**CN:** 该类把与 Thinkreasoningparser 相关的场景组织在一起。

### Constants / assignments (lines 25-29)
```python
MODEL_OUTPUT = (
    "<think>let me think about this</think>"
    '<tool_call>\n{"name": "get_weather", '
    '"arguments": {"city": "Dallas"}}\n</tool_call>'
)
```
**EN:** Defines shared constants or configuration objects like `MODEL_OUTPUT`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `MODEL_OUTPUT`），供后续测试重复使用。

### Fixture: tokenizer (lines 32-36)
```python
@pytest.fixture(scope="module")
def tokenizer():
    from vllm.tokenizers import get_tokenizer

    return get_tokenizer("Qwen/Qwen3-32B")
```
**EN:** Provides a pytest fixture for Tokenizer. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `pytest.fixture`, `get_tokenizer`.
**CN:** 该代码块定义 pytest 夹具 `tokenizer`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `get_tokenizer` 构造或返回测试所需的值。

### Fixture: request_obj (lines 39-44)
```python
@pytest.fixture
def request_obj():
    return ChatCompletionRequest(
        model="test-model",
        messages=[{"role": "user", "content": "hi"}],
    )
```
**EN:** Provides a pytest fixture for Request Obj. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `ChatCompletionRequest`.
**CN:** 该代码块定义 pytest 夹具 `request_obj`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `ChatCompletionRequest` 构造或返回测试所需的值。

### Helper: make_parser (lines 47-50)
```python
def make_parser(tokenizer, reasoning=False, tool=False):
    _WrappedParser.reasoning_parser_cls = ThinkReasoningParser if reasoning else None
    _WrappedParser.tool_parser_cls = Hermes2ProToolParser if tool else None
    return _WrappedParser(tokenizer)
```
**EN:** Implements a reusable helper for Make Parser, reducing duplication across related tests. It coordinates operations such as `_WrappedParser`.
**CN:** 该辅助函数为 Make Parser 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `_WrappedParser` 等操作。

### Helper: stream_text (lines 53-63)
```python
def stream_text(parser, tokenizer, text, request, prompt_token_ids=None):
    token_ids = tokenizer.encode(text, add_special_tokens=False)
    results: list[DeltaMessage | None] = []
    for tid in token_ids:
        delta_text = tokenizer.decode([tid])
        result = parser.parse_delta(
            delta_text, [tid], request, prompt_token_ids=prompt_token_ids
        )
        prompt_token_ids = None
        results.append(result)
    return results
```
**EN:** Implements a reusable helper for Stream Text, reducing duplication across related tests. It coordinates operations such as `tokenizer.encode`, `tokenizer.decode`, `parser.parse_delta`.
**CN:** 该辅助函数为 Stream Text 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `tokenizer.encode`, `tokenizer.decode`, `parser.parse_delta` 等操作。

### Test: test_parse_delta_neither_parser (lines 73-85)
```python
def test_parse_delta_neither_parser(tokenizer, request_obj):
    parser = make_parser(tokenizer, reasoning=False, tool=False)
    results = stream_text(
        parser, tokenizer, MODEL_OUTPUT, request_obj, prompt_token_ids=[]
    )
    reasoning, content, tool_calls = collect_fields(results)

    assert reasoning == ""
    assert len(tool_calls) == 0
    assert "<think>" in content
    assert "let me think about this" in content
    assert "<tool_call>" in content
    assert "get_weather" in content
```
**EN:** Checks Parse Delta Neither Parser under a focused test scenario. The body exercises logic via `make_parser`, `stream_text`, `collect_fields` before asserting the expected outcome.
**CN:** 该测试用例验证 Parse Delta Neither Parser 在特定场景下的行为。 函数体会先通过 `make_parser`, `stream_text`, `collect_fields` 驱动目标逻辑，再断言预期结果。

### Test: test_parse_delta_tool_parser_only (lines 88-105)
```python
def test_parse_delta_tool_parser_only(tokenizer, request_obj):
    parser = make_parser(tokenizer, reasoning=False, tool=True)
    results = stream_text(
        parser, tokenizer, MODEL_OUTPUT, request_obj, prompt_token_ids=[]
    )
    reasoning, content, tool_calls = collect_fields(results)

    assert reasoning == ""
    assert "<think>" in content
    assert "let me think about this" in content
    assert "</think>" in content

    assert len(tool_calls) > 0
    assert tool_calls[0].function.name == "get_weather"
    tool_args = "".join(
        tc.function.arguments for tc in tool_calls if tc.function.arguments
    )
    assert json.loads(tool_args) == {"city": "Dallas"}
```
**EN:** Checks Parse Delta Tool Parser Only under a focused test scenario. The body exercises logic via `make_parser`, `stream_text`, `collect_fields` before asserting the expected outcome.
**CN:** 该测试用例验证 Parse Delta Tool Parser Only 在特定场景下的行为。 函数体会先通过 `make_parser`, `stream_text`, `collect_fields` 驱动目标逻辑，再断言预期结果。

### Test: test_parse_delta_reasoning_parser_only (lines 108-119)
```python
def test_parse_delta_reasoning_parser_only(tokenizer, request_obj):
    parser = make_parser(tokenizer, reasoning=True, tool=False)
    results = stream_text(
        parser, tokenizer, MODEL_OUTPUT, request_obj, prompt_token_ids=[]
    )
    reasoning, content, tool_calls = collect_fields(results)

    assert "let me think about this" in reasoning
    assert len(tool_calls) == 0
    assert "<tool_call>" in content
    assert "get_weather" in content
    assert "</tool_call>" in content
```
**EN:** Checks Parse Delta Reasoning Parser Only under a focused test scenario. The body exercises logic via `make_parser`, `stream_text`, `collect_fields` before asserting the expected outcome.
**CN:** 该测试用例验证 Parse Delta Reasoning Parser Only 在特定场景下的行为。 函数体会先通过 `make_parser`, `stream_text`, `collect_fields` 驱动目标逻辑，再断言预期结果。

### Test: test_parse_delta_both_parsers (lines 122-137)
```python
def test_parse_delta_both_parsers(tokenizer, request_obj):
    parser = make_parser(tokenizer, reasoning=True, tool=True)
    results = stream_text(
        parser, tokenizer, MODEL_OUTPUT, request_obj, prompt_token_ids=[]
    )
    reasoning, content, tool_calls = collect_fields(results)

    assert "let me think about this" in reasoning
    assert content == ""

    assert len(tool_calls) > 0
    assert tool_calls[0].function.name == "get_weather"
    tool_args = "".join(
        tc.function.arguments for tc in tool_calls if tc.function.arguments
    )
    assert json.loads(tool_args) == {"city": "Dallas"}
```
**EN:** Checks Parse Delta Both Parsers under a focused test scenario. The body exercises logic via `make_parser`, `stream_text`, `collect_fields` before asserting the expected outcome.
**CN:** 该测试用例验证 Parse Delta Both Parsers 在特定场景下的行为。 函数体会先通过 `make_parser`, `stream_text`, `collect_fields` 驱动目标逻辑，再断言预期结果。

### Test: test_parse_delta_reasoning_only_thinking_disabled (lines 140-165)
```python
def test_parse_delta_reasoning_only_thinking_disabled(tokenizer, request_obj):
    """Regression test for vllm-project/vllm#40466.

    When enable_thinking=False, the chat template places <think>\\n\\n</think>
    in the prompt. The model then generates pure content (no think tokens).
    All streaming output must go to delta.content, not delta.reasoning.
    """
    parser = make_parser(tokenizer, reasoning=True, tool=False)

    end_token_id = parser._reasoning_parser.end_token_id
    prompt_token_ids = [1, 2, end_token_id, 3]

    content_text = "Hello! How can I assist you today?"
    results = stream_text(
        parser,
        tokenizer,
        content_text,
        request_obj,
        prompt_token_ids=prompt_token_ids,
    )
    reasoning, content, tool_calls = collect_fields(results)

    assert reasoning == "", f"Expected no reasoning, got: {reasoning!r}"
    assert "Hello" in content
    assert "assist" in content
    assert len(tool_calls) == 0
```
**EN:** Regression test for vllm-project/vllm#40466. The body exercises logic via `make_parser`, `stream_text`, `collect_fields` before asserting the expected outcome.
**CN:** 该测试用例验证 Parse Delta Reasoning Only Thinking Disabled 在特定场景下的行为。 函数体会先通过 `make_parser`, `stream_text`, `collect_fields` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
collect_fields
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
- **Reasoning traces / 推理轨迹**
  - **EN:** The tests inspect reasoning-specific formats or parser behavior in intermediate outputs.
  - **CN:** 这些测试检查中间输出中的推理格式或解析器行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`
- **Third-party / 第三方依赖**: `pytest`
- **vLLM internal / vLLM 内部依赖**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.parser.abstract_parser`, `vllm.reasoning.basic_parsers`, `vllm.tool_parsers.hermes_tool_parser`, `vllm.tokenizers`
