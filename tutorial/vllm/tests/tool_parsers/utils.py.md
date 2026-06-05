# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/tool_parsers/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Utils behavior in the Tool Parsers test area through focused pytest scenarios. It focuses on scenarios such as Streamingtoolreconstructor, Run Tool Extraction, Run Tool Extraction Nonstreaming. / 该文件在 Tool Parsers 测试域中，通过有针对性的 pytest 场景验证 Utils 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-14)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

from collections.abc import Iterable

from vllm.entrypoints.openai.chat_completion.protocol import ChatCompletionRequest
from vllm.entrypoints.openai.engine.protocol import (
    DeltaMessage,
    ExtractedToolCallInformation,
    FunctionCall,
    ToolCall,
)
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers import ToolParser
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `collections.abc`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Class: StreamingToolReconstructor (lines 17-81)
```python
class StreamingToolReconstructor:
    def __init__(self, assert_one_tool_per_delta: bool = True):
        self.tool_calls: list[ToolCall] = []
        self.other_content: str = ""
        self._assert_one_tool_per_delta = assert_one_tool_per_delta

    def append_delta(self, delta: DeltaMessage):
        if delta.content is not None:
            self.other_content += delta.content
        else:
            assert delta.tool_calls, (
                "Streaming results should have either content or tool calls (or both)"
            )
        if self._assert_one_tool_per_delta:
            # Note: This isn't strictly required by the API and may not be
            # possible to adhere to depending on the token space and number of
            # tokens per streamed response from the model, but it is required
            # by tool_use tests, so we enforce it here by default also.
            assert len(delta.tool_calls) < 2, (
# ... omitted for brevity ...
                )
                assert call_delta.index == len(self.tool_calls), (
                    f"Incorrect index for tool delta. Got {call_delta.index}, "
                    f"expected {len(self.tool_calls)}"
                )
                self.tool_calls.append(
                    ToolCall(
                        id=call_delta.id,
                        function=FunctionCall(
                            name=call_delta.function.name,
                            arguments=call_delta.function.arguments or "",
                        ),
                    )
                )
```
**EN:** Groups related scenarios for Streamingtoolreconstructor.
**CN:** 该类把与 Streamingtoolreconstructor 相关的场景组织在一起。

### Helper: run_tool_extraction (lines 84-102)
```python
def run_tool_extraction(
    tool_parser: ToolParser,
    model_output: str,
    request: ChatCompletionRequest | None = None,
    streaming: bool = False,
    assert_one_tool_per_delta: bool = True,
) -> tuple[str | None, list[ToolCall]]:
    if streaming:
        reconstructor = run_tool_extraction_streaming(
            tool_parser,
            model_output,
            request,
            assert_one_tool_per_delta=assert_one_tool_per_delta,
        )
        return reconstructor.other_content or None, reconstructor.tool_calls
    else:
        extracted = run_tool_extraction_nonstreaming(tool_parser, model_output, request)
        assert extracted.tools_called == bool(extracted.tool_calls)
        return extracted.content, extracted.tool_calls
```
**EN:** Implements a reusable helper for Run Tool Extraction, reducing duplication across related tests. It coordinates operations such as `run_tool_extraction_streaming`, `run_tool_extraction_nonstreaming`, `bool`.
**CN:** 该辅助函数为 Run Tool Extraction 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `run_tool_extraction_streaming`, `run_tool_extraction_nonstreaming`, `bool` 等操作。

### Helper: run_tool_extraction_nonstreaming (lines 105-111)
```python
def run_tool_extraction_nonstreaming(
    tool_parser: ToolParser,
    model_output: str,
    request: ChatCompletionRequest | None = None,
) -> ExtractedToolCallInformation:
    request = request or ChatCompletionRequest(messages=[], model="test-model")
    return tool_parser.extract_tool_calls(model_output, request)
```
**EN:** Implements a reusable helper for Run Tool Extraction Nonstreaming, reducing duplication across related tests. It coordinates operations such as `tool_parser.extract_tool_calls`, `ChatCompletionRequest`.
**CN:** 该辅助函数为 Run Tool Extraction Nonstreaming 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `tool_parser.extract_tool_calls`, `ChatCompletionRequest` 等操作。

### Helper: split_string_into_token_deltas (lines 114-126)
```python
def split_string_into_token_deltas(tokenizer: TokenizerLike, text: str) -> list[str]:
    # Split a string into a series of deltas using the provided tokenizer. Each
    # delta will be the string equivalent of a single token.
    token_ids = tokenizer.encode(text, add_special_tokens=False)
    previously_decoded_text = ""
    deltas = []
    for i in range(1, len(token_ids) + 1):
        current_tokens = token_ids[:i]
        current_text = tokenizer.decode(current_tokens)
        new_text = current_text[len(previously_decoded_text) :]
        previously_decoded_text = current_text
        deltas.append(new_text)
    return deltas
```
**EN:** Implements a reusable helper for Split String Into Token Deltas, reducing duplication across related tests. It coordinates operations such as `tokenizer.encode`, `range`, `tokenizer.decode`.
**CN:** 该辅助函数为 Split String Into Token Deltas 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `tokenizer.encode`, `range`, `tokenizer.decode` 等操作。

### Helper: run_tool_extraction_streaming (lines 129-167)
```python
def run_tool_extraction_streaming(
    tool_parser: ToolParser,
    model_deltas: Iterable[str],
    request: ChatCompletionRequest | None = None,
    assert_one_tool_per_delta: bool = True,
) -> StreamingToolReconstructor:
    if isinstance(model_deltas, str):
        model_deltas = split_string_into_token_deltas(
            tool_parser.model_tokenizer, model_deltas
        )

    request = request or ChatCompletionRequest(messages=[], model="test-model")
    reconstructor = StreamingToolReconstructor(
        assert_one_tool_per_delta=assert_one_tool_per_delta
    )
    previous_text = ""
    previous_tokens: list[int] = []
    for delta in model_deltas:
        token_delta = [
# ... omitted for brevity ...
        delta_message = tool_parser.extract_tool_calls_streaming(
            previous_text,
            current_text,
            delta,
            previous_tokens,
            current_tokens,
            token_delta,
            request,
        )
        if delta_message is not None:
            reconstructor.append_delta(delta_message)
        previous_text = current_text
        previous_tokens = current_tokens
    return reconstructor
```
**EN:** Implements a reusable helper for Run Tool Extraction Streaming, reducing duplication across related tests. It coordinates operations such as `isinstance`, `StreamingToolReconstructor`, `split_string_into_token_deltas`.
**CN:** 该辅助函数为 Run Tool Extraction Streaming 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `isinstance`, `StreamingToolReconstructor`, `split_string_into_token_deltas` 等操作。

## Key Concepts / 关键概念
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
- **Standard library / 标准库**: `collections.abc`
- **vLLM internal / vLLM 内部依赖**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.tokenizers`, `vllm.tool_parsers`
