# qwen3_reasoning_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/reasoning/qwen3_reasoning_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the qwen3 reasoning parser that separates model reasoning traces from user-facing answer content in full and streaming responses. / 实现 qwen3 推理解析器，在完整响应与流式响应中将模型推理轨迹与面向用户的答案内容分离。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-8)
```python
from collections.abc import Iterable, Sequence
from typing import TYPE_CHECKING

from vllm.entrypoints.openai.engine.protocol import DeltaMessage
from vllm.reasoning.basic_parsers import BaseThinkingReasoningParser
```
**EN:** Imports tokenizer interfaces, protocol/message types, and shared helpers required to detect reasoning spans and split final content correctly.
**CN:** 导入分词器接口、协议/消息类型以及共享辅助工具，用于检测推理片段并正确切分最终内容。

### Class `Qwen3ReasoningParser` (lines 16-231)
```python
class Qwen3ReasoningParser(BaseThinkingReasoningParser):
    """
    Reasoning parser for the Qwen3/Qwen3.5 model family.

    The Qwen3 model family uses <think>...</think> tokens to denote reasoning
    text. Starting with Qwen3.5, the chat template places <think> in the
    prompt so only </think> appears in the generated output. The model
    provides a strict switch to disable reasoning output via the
    'enable_thinking=False' parameter.

    When thinking is disabled, the template places <think>\\n\\n</think>\\n\\n
    in the prompt. The serving layer detects this via prompt_is_reasoning_end
    and routes deltas as content without calling the streaming parser.

    NOTE: Models up to the 2507 release (e.g., Qwen/Qwen3-235B-A22B-Instruct-2507)
```
**EN:** Reasoning parser for the Qwen3/Qwen3.5 model family.
**CN:** 定义 `Qwen3ReasoningParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `Qwen3ReasoningParser.__init__` (lines 40-51)
```python
    def __init__(self, tokenizer: "TokenizerLike", *args, **kwargs):
        super().__init__(tokenizer, *args, **kwargs)

        chat_kwargs = kwargs.get("chat_template_kwargs", {}) or {}
        # Qwen3 defaults to thinking enabled; only treat output as
        # pure content when the user explicitly disables it.
        self.thinking_enabled = chat_kwargs.get("enable_thinking", True)

        self._tool_call_tag = "<tool_call>"
        self._tool_call_token_id = self.vocab.get(self._tool_call_tag)
        self._tool_call_end_tag = "</tool_call>"
        self._tool_call_end_token_id = self.vocab.get(self._tool_call_end_tag)
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `Qwen3ReasoningParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `Qwen3ReasoningParser.start_token` (lines 54-56)
```python
    def start_token(self) -> str:
        """The token that starts reasoning content."""
        return "<think>"
```
**EN:** The token that starts reasoning content.
**CN:** `Qwen3ReasoningParser.start_token` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Qwen3ReasoningParser.end_token` (lines 59-61)
```python
    def end_token(self) -> str:
        """The token that ends reasoning content."""
        return "</think>"
```
**EN:** The token that ends reasoning content.
**CN:** `Qwen3ReasoningParser.end_token` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Qwen3ReasoningParser.is_reasoning_end` (lines 63-86)
```python
    def is_reasoning_end(self, input_ids: Sequence[int]) -> bool:
        start_token_id = self.start_token_id
        end_token_id = self.end_token_id
        tool_call_token_id = self._tool_call_token_id
        tool_call_end_token_id = self._tool_call_end_token_id

        for i in range(len(input_ids) - 1, -1, -1):
            token_id = input_ids[i]
            if token_id == start_token_id:
                # Found <think> before </think> or <tool_call>
                return False
            if token_id == end_token_id:
                return True
            if tool_call_token_id is not None and token_id == tool_call_token_id:
                # Only treat as implicit reasoning end if this <tool_call>
                # is NOT followed by </tool_call>.  Paired occurrences are
                # template examples in the prompt, not model output.
                if tool_call_end_token_id is not None and any(
                    input_ids[j] == tool_call_end_token_id
                    for j in range(i + 1, len(input_ids))
                ):
                    continue
                return True
        return False
```
**EN:** This predicate checks whether a delimiter, completion condition, or format-specific state transition has been reached.
**CN:** `Qwen3ReasoningParser.is_reasoning_end` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Method `Qwen3ReasoningParser.is_reasoning_end_streaming` (lines 88-95)
```python
    def is_reasoning_end_streaming(
        self, input_ids: Sequence[int], delta_ids: Iterable[int]
    ) -> bool:
        if super().is_reasoning_end_streaming(input_ids, delta_ids):
            return True
        if self._tool_call_token_id is not None:
            return self._tool_call_token_id in delta_ids
        return False
```
**EN:** This predicate checks whether a delimiter, completion condition, or format-specific state transition has been reached.
**CN:** `Qwen3ReasoningParser.is_reasoning_end_streaming` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Method `Qwen3ReasoningParser.extract_content_ids` (lines 97-113)
```python
    def extract_content_ids(self, input_ids: list[int]) -> list[int]:
        """
        Extract content token ids from the input_ids.
        """
        result = super().extract_content_ids(input_ids)
        if result:
            return result
        # Fall back: content starts at <tool_call> (implicit reasoning end).
        if (
            self._tool_call_token_id is not None
            and self._tool_call_token_id in input_ids
        ):
            tool_call_index = (
                len(input_ids) - 1 - input_ids[::-1].index(self._tool_call_token_id)
            )
            return input_ids[tool_call_index:]
        return []
```
**EN:** Extract content token ids from the input_ids.
**CN:** `Qwen3ReasoningParser.extract_content_ids` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `Qwen3ReasoningParser.extract_reasoning` (lines 115-158)
```python
    def extract_reasoning(
        self, model_output: str, request: "ChatCompletionRequest | ResponsesRequest"
    ) -> tuple[str | None, str | None]:
        """
        Extract reasoning content from the model output.

        The <think> token is placed in the prompt by the chat template,
        so typically only </think> appears in the generated output.
        If <think> is present (e.g. from a different template), it is
        stripped before extraction.

        When thinking is explicitly disabled and no </think> appears,
        returns (None, model_output) — all output is content.
        Otherwise (thinking enabled, default), a missing </think> means
        the output was truncated and everything is reasoning:
        returns (model_output, None).

        Returns:
            tuple[Optional[str], Optional[str]]: reasoning content and content
        """

        # Strip <think> if present in the generated output.
        model_output_parts = model_output.partition(self.start_token)
        model_output = (
            model_output_parts[2] if model_output_parts[1] else model_output_parts[0]
        )

        if self.end_token in model_output:
            reasoning, _, content = model_output.partition(self.end_token)
            return reasoning, content or None

        if not self.thinking_enabled:
            # Thinking explicitly disabled — treat everything as content.
            return None, model_output

        # No </think> — check for implicit reasoning end via <tool_call>.
        tool_call_index = model_output.find(self._tool_call_tag)
        if tool_call_index != -1:
            reasoning = model_output[:tool_call_index]
            content = model_output[tool_call_index:]
            return reasoning or None, content or None
        # Thinking enabled but no </think>: output was truncated.
        # Everything generated so far is reasoning.
        return model_output, None
```
**EN:** Extract reasoning content from the model output.
**CN:** `Qwen3ReasoningParser.extract_reasoning` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `Qwen3ReasoningParser.extract_reasoning_streaming` (lines 160-231)
```python
    def extract_reasoning_streaming(
        self,
        previous_text: str,
        current_text: str,
        delta_text: str,
        previous_token_ids: Sequence[int],
        current_token_ids: Sequence[int],
        delta_token_ids: Sequence[int],
    ) -> DeltaMessage | None:
        """
        Extract reasoning content from a streaming delta.

        Since <think> is placed in the prompt by the chat template, all
        generated tokens before </think> are reasoning and tokens after
        are content.

        NOTE: When thinking is disabled, no think tokens appear in the
        generated output. The serving layer detects this via
        prompt_is_reasoning_end and routes deltas as content without
        calling this method.
        """
        # Strip <think> from delta if present (old template / edge case
        # where the model generates <think> itself).
        if self.start_token_id in delta_token_ids:
            start_idx = delta_text.find(self.start_token)
            if start_idx >= 0:
                delta_text = delta_text[start_idx + len(self.start_token) :]

        if self.end_token_id in delta_token_ids:
            # End token in this delta: split reasoning from content.
            end_index = delta_text.find(self.end_token)
            if end_index >= 0:
                reasoning = delta_text[:end_index]
                content = delta_text[end_index + len(self.end_token) :]
                if not reasoning and not content:
                    return None
                return DeltaMessage(
                    reasoning=reasoning if reasoning else None,
                    content=content if content else None,
                )
            # end_token_id in IDs but not in text (already stripped)
            return None

        # Implicit reasoning end via <tool_call>.
        if (
            self._tool_call_token_id is not None
            and self._tool_call_token_id in delta_token_ids
        ):
            tool_index = delta_text.find(self._tool_call_tag)
            if tool_index >= 0:
                reasoning = delta_text[:tool_index]
                content = delta_text[tool_index:]
                return DeltaMessage(
                    reasoning=reasoning if reasoning else None,
                    content=content if content else None,
                )

        # No end token in this delta.
        if not delta_text:
            # Nothing left after stripping start token.
            return None
        elif self.end_token_id in previous_token_ids:
            # End token already passed: everything is content now.
            return DeltaMessage(content=delta_text)
        elif (
            self._tool_call_token_id is not None
            and self._tool_call_token_id in previous_token_ids
        ):
            return DeltaMessage(content=delta_text)
        else:
            # No end token yet: still in reasoning phase.
            return DeltaMessage(reasoning=delta_text)
```
**EN:** Extract reasoning content from a streaming delta.
**CN:** `Qwen3ReasoningParser.extract_reasoning_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Reasoning boundary detection**: Reasoning boundary detection. / 使用显式标记、token ID 或文本模式检测推理片段的边界。
- **Streaming extraction**: Streaming extraction. / 在流式解码时把新产生的推理内容与最终答案内容分开输出。
- **Parser specialization**: Parser specialization. / 通过子类覆盖或模型特定规则适配不同模型的推理格式。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `vllm.entrypoints.openai.engine.protocol`, `vllm.reasoning.basic_parsers`
