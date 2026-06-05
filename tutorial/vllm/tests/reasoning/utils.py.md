# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/reasoning/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Utils behavior in the Reasoning test area through focused pytest scenarios. It focuses on scenarios such as Streamingreasoningreconstructor, Run Reasoning Extraction, Run Reasoning Extraction Mistral. / 该文件在 Reasoning 测试域中，通过有针对性的 pytest 场景验证 Utils 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-7)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

from vllm.entrypoints.openai.chat_completion.protocol import ChatCompletionRequest
from vllm.entrypoints.openai.engine.protocol import DeltaMessage
from vllm.reasoning import ReasoningParser
from vllm.utils.mistral import is_mistral_tokenizer
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Class: StreamingReasoningReconstructor (lines 10-30)
```python
class StreamingReasoningReconstructor:
    def __init__(self):
        self.reasoning = None
        self.other_content = None

    def append_delta(self, delta: DeltaMessage):
        # content and the reasoning content should not be present
        # at the same time
        assert delta.content is None or delta.reasoning is None, (
            "Both content and reasoning content are present in the delta message"
        )
        if delta.content is not None:
            if self.other_content is None:
                self.other_content = delta.content
            else:
                self.other_content += delta.content
        else:
            if self.reasoning is None:
                self.reasoning = delta.reasoning
            else:
                self.reasoning += delta.reasoning
```
**EN:** Groups related scenarios for Streamingreasoningreconstructor.
**CN:** 该类把与 Streamingreasoningreconstructor 相关的场景组织在一起。

### Helper: run_reasoning_extraction (lines 33-53)
```python
def run_reasoning_extraction(
    reasoning_parser: ReasoningParser,
    model_output: list[str],
    request: ChatCompletionRequest | None = None,
    streaming: bool = False,
) -> tuple[str | None, str | None]:
    if streaming:
        reconstructor = run_reasoning_extraction_streaming(
            reasoning_parser,
            model_output,
            request,
        )
        return (
            reconstructor.reasoning,
            reconstructor.other_content or None,
        )
    else:
        reasoning, content = run_reasoning_extraction_nonstreaming(
            reasoning_parser, model_output, request
        )
        return reasoning, content
```
**EN:** Implements a reusable helper for Run Reasoning Extraction, reducing duplication across related tests. It coordinates operations such as `run_reasoning_extraction_streaming`, `run_reasoning_extraction_nonstreaming`.
**CN:** 该辅助函数为 Run Reasoning Extraction 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `run_reasoning_extraction_streaming`, `run_reasoning_extraction_nonstreaming` 等操作。

### Helper: run_reasoning_extraction_mistral (lines 56-82)
```python
def run_reasoning_extraction_mistral(
    reasoning_parser: ReasoningParser,
    model_output: list[int],
    request: ChatCompletionRequest | None = None,
    streaming: bool = False,
) -> tuple[str | None, str | None]:
    assert is_mistral_tokenizer(reasoning_parser.model_tokenizer), type(
        reasoning_parser.model_tokenizer
    )
    if streaming:
        reconstructor = run_reasoning_extraction_streaming_mistral(
            reasoning_parser,
            model_output,
            request,
        )
        return (
            reconstructor.reasoning,
            reconstructor.other_content or None,
        )
    else:
        str_output = reasoning_parser.model_tokenizer.convert_ids_to_tokens(
            model_output
        )
        reasoning, content = run_reasoning_extraction_nonstreaming(
            reasoning_parser, str_output, request
        )
        return reasoning, content
```
**EN:** Implements a reusable helper for Run Reasoning Extraction Mistral, reducing duplication across related tests. It coordinates operations such as `is_mistral_tokenizer`, `type`, `run_reasoning_extraction_streaming_mistral`.
**CN:** 该辅助函数为 Run Reasoning Extraction Mistral 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `is_mistral_tokenizer`, `type`, `run_reasoning_extraction_streaming_mistral` 等操作。

### Helper: run_reasoning_extraction_nonstreaming (lines 85-93)
```python
def run_reasoning_extraction_nonstreaming(
    reasoning_parser: ReasoningParser,
    model_output: list[str],
    request: ChatCompletionRequest | None = None,
) -> tuple[str | None, str | None]:
    request = request or ChatCompletionRequest(messages=[], model="test-model")
    return reasoning_parser.extract_reasoning(
        model_output="".join(model_output), request=request
    )
```
**EN:** Implements a reusable helper for Run Reasoning Extraction Nonstreaming, reducing duplication across related tests. It coordinates operations such as `reasoning_parser.extract_reasoning`, `ChatCompletionRequest`, `''.join`.
**CN:** 该辅助函数为 Run Reasoning Extraction Nonstreaming 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `reasoning_parser.extract_reasoning`, `ChatCompletionRequest`, `''.join` 等操作。

### Helper: run_reasoning_extraction_streaming (lines 96-125)
```python
def run_reasoning_extraction_streaming(
    reasoning_parser: ReasoningParser,
    model_deltas: list[str],
    request: ChatCompletionRequest | None = None,
) -> StreamingReasoningReconstructor:
    request = request or ChatCompletionRequest(messages=[], model="test-model")
    reconstructor = StreamingReasoningReconstructor()
    previous_text = ""
    previous_tokens: list[int] = []
    for delta in model_deltas:
        token_delta = [
            reasoning_parser.vocab.get(token)
            for token in reasoning_parser.model_tokenizer.tokenize(delta)
            if token in reasoning_parser.vocab
        ]
        current_text = previous_text + delta
        current_tokens = previous_tokens + token_delta
        delta_message = reasoning_parser.extract_reasoning_streaming(
            previous_text,
            current_text,
            delta,
            previous_tokens,
            current_tokens,
            token_delta,
        )
        if delta_message is not None:
            reconstructor.append_delta(delta_message)
        previous_text = current_text
        previous_tokens = current_tokens
    return reconstructor
```
**EN:** Implements a reusable helper for Run Reasoning Extraction Streaming, reducing duplication across related tests. It coordinates operations such as `StreamingReasoningReconstructor`, `ChatCompletionRequest`, `reasoning_parser.extract_reasoning_streaming`.
**CN:** 该辅助函数为 Run Reasoning Extraction Streaming 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `StreamingReasoningReconstructor`, `ChatCompletionRequest`, `reasoning_parser.extract_reasoning_streaming` 等操作。

### Helper: run_reasoning_extraction_streaming_mistral (lines 128-157)
```python
def run_reasoning_extraction_streaming_mistral(
    reasoning_parser: ReasoningParser,
    model_deltas: list[int],
    request: ChatCompletionRequest | None = None,
) -> StreamingReasoningReconstructor:
    assert is_mistral_tokenizer(reasoning_parser.model_tokenizer), type(
        reasoning_parser.model_tokenizer
    )
    request = request or ChatCompletionRequest(messages=[], model="test-model")
    reconstructor = StreamingReasoningReconstructor()
    previous_text = ""
    previous_tokens: list[int] = []
    for model_delta in model_deltas:
        token_delta = [model_delta]
        delta = reasoning_parser.model_tokenizer.convert_ids_to_tokens([model_delta])[0]
        current_text = previous_text + delta
        current_tokens = previous_tokens + token_delta
        delta_message = reasoning_parser.extract_reasoning_streaming(
            previous_text,
            current_text,
            delta,
            previous_tokens,
            current_tokens,
            token_delta,
        )
        if delta_message is not None:
            reconstructor.append_delta(delta_message)
        previous_text = current_text
        previous_tokens = current_tokens
    return reconstructor
```
**EN:** Implements a reusable helper for Run Reasoning Extraction Streaming Mistral, reducing duplication across related tests. It coordinates operations such as `is_mistral_tokenizer`, `type`, `StreamingReasoningReconstructor`.
**CN:** 该辅助函数为 Run Reasoning Extraction Streaming Mistral 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `is_mistral_tokenizer`, `type`, `StreamingReasoningReconstructor` 等操作。

## Key Concepts / 关键概念
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
- **vLLM internal / vLLM 内部依赖**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.reasoning`, `vllm.utils.mistral`
