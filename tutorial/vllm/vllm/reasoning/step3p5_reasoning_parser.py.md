# step3p5_reasoning_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/reasoning/step3p5_reasoning_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the step3p5 reasoning parser that separates model reasoning traces from user-facing answer content in full and streaming responses. / 实现 step3p5 推理解析器，在完整响应与流式响应中将模型推理轨迹与面向用户的答案内容分离。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-9)
```python
from collections.abc import Iterable, Sequence
from typing import TYPE_CHECKING

from vllm.entrypoints.openai.engine.protocol import DeltaMessage
from vllm.reasoning.basic_parsers import BaseThinkingReasoningParser
from vllm.tokenizers import TokenizerLike
```
**EN:** Imports tokenizer interfaces, protocol/message types, and shared helpers required to detect reasoning spans and split final content correctly.
**CN:** 导入分词器接口、协议/消息类型以及共享辅助工具，用于检测推理片段并正确切分最终内容。

### Class `Step3p5ReasoningParser` (lines 16-184)
```python
class Step3p5ReasoningParser(BaseThinkingReasoningParser):
    """
    Reasoning parser for Step3p5 model.

    Step3p5 uses the <think>...</think> format, but it tends to emit an extra
    newline immediately before and/or after the </think> token. This parser trims:
      - the newline right before </think>
      - the newline right after </think>
    """
```
**EN:** Reasoning parser for Step3p5 model.
**CN:** 定义 `Step3p5ReasoningParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `Step3p5ReasoningParser.start_token` (lines 27-28)
```python
    def start_token(self) -> str:
        return "<think>"
```
**EN:** Provides the `start_token` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Step3p5ReasoningParser.start_token` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Step3p5ReasoningParser.end_token` (lines 31-32)
```python
    def end_token(self) -> str:
        return "</think>"
```
**EN:** Provides the `end_token` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Step3p5ReasoningParser.end_token` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Step3p5ReasoningParser.__init__` (lines 34-43)
```python
    def __init__(self, tokenizer: TokenizerLike, *args, **kwargs):
        super().__init__(tokenizer, *args, **kwargs)

        # Used to hold a trailing "\n" from reasoning content so we can decide
        # whether it is immediately before </think>.
        self._pending_reasoning_newline = False

        # Tracks whether we've seen </think> but are still waiting for one more
        # token to confirm the end.
        self._end_token_pending = False
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `Step3p5ReasoningParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `Step3p5ReasoningParser.is_reasoning_end` (lines 45-46)
```python
    def is_reasoning_end(self, input_ids: Sequence[int]) -> bool:
        return self._is_reasoning_end_from_ids(input_ids)
```
**EN:** This predicate checks whether a delimiter, completion condition, or format-specific state transition has been reached.
**CN:** `Step3p5ReasoningParser.is_reasoning_end` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Method `Step3p5ReasoningParser.is_reasoning_end_streaming` (lines 48-52)
```python
    def is_reasoning_end_streaming(
        self, input_ids: Sequence[int], delta_ids: Iterable[int]
    ) -> bool:
        # Only examine newly generated tokens; they may contain multiple ids.
        return self._is_reasoning_end_from_ids(tuple(delta_ids))
```
**EN:** This predicate checks whether a delimiter, completion condition, or format-specific state transition has been reached.
**CN:** `Step3p5ReasoningParser.is_reasoning_end_streaming` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Method `Step3p5ReasoningParser._is_reasoning_end_from_ids` (lines 54-93)
```python
    def _is_reasoning_end_from_ids(self, input_ids: Sequence[int]) -> bool:
        # Scan backwards to find the last special token, <think> or </think>.
        last_special = None
        last_idx = -1
        for i in range(len(input_ids) - 1, -1, -1):
            token_id = input_ids[i]
            if token_id == self.start_token_id:
                last_special = "start"
                last_idx = i
                break
            if token_id == self.end_token_id:
                last_special = "end"
                last_idx = i
                break

        if last_special == "start":
            # If we're already waiting for one token after </think>, do not
            # clear the pending state just because the prompt contains <think>.
            # Streaming deltas should not include <think> for this model.
            if self._end_token_pending:
                return False
            # A start token after any end token means reasoning is ongoing.
            self._end_token_pending = False
            return False

        if last_special == "end":
            # Require at least one token after </think> before ending.
            if last_idx < len(input_ids) - 1:
                self._end_token_pending = False
                return True
            self._end_token_pending = True
            return False

        # No special tokens in this input. If we were waiting for one token
        # after </think>, any new token completes the end.
        if self._end_token_pending and input_ids:
            self._end_token_pending = False
            return True

        return False
```
**EN:** Provides the `_is_reasoning_end_from_ids` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Step3p5ReasoningParser._is_reasoning_end_from_ids` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Step3p5ReasoningParser.extract_reasoning` (lines 95-105)
```python
    def extract_reasoning(
        self,
        model_output: str,
        request: "ChatCompletionRequest | ResponsesRequest",
    ) -> tuple[str | None, str | None]:
        reasoning, content = super().extract_reasoning(model_output, request)
        if reasoning is not None:
            reasoning = reasoning.removesuffix("\n")
        if content is not None:
            content = content.removeprefix("\n")
        return reasoning or None, content or None
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `Step3p5ReasoningParser.extract_reasoning` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `Step3p5ReasoningParser.extract_reasoning_streaming` (lines 107-184)
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
        # Drop the immediate newline that models often emit after </think>.
        if previous_text.endswith(self.end_token) and delta_text:
            if delta_text == "\n":
                return None
            elif delta_text.startswith("\n"):
                remaining = delta_text.removeprefix("\n")
                return DeltaMessage(content=remaining) if remaining else None

        ret = super().extract_reasoning_streaming(
            previous_text,
            current_text,
            delta_text,
            previous_token_ids,
            current_token_ids,
            delta_token_ids,
        )

        if ret is None:
            return None

        # Compatibility path for models that don't generate the start token:
        # treat everything before </think> as reasoning and everything after
        # as content.
        if (
            self.start_token_id not in previous_token_ids
            and self.start_token_id not in delta_token_ids
        ):
            if self.end_token_id in delta_token_ids:
                end_index = delta_text.find(self.end_token)
                reasoning = delta_text[:end_index]
                content = delta_text[end_index + len(self.end_token) :]
                ret = DeltaMessage(reasoning=reasoning, content=content or None)
            elif self.end_token_id in previous_token_ids:
                ret = DeltaMessage(content=delta_text)
            else:
                ret = DeltaMessage(reasoning=delta_text)

        reasoning_to_output = ret.reasoning
        content_to_output = ret.content

        # Reasoning: handle the newline immediately before </think>.
        if reasoning_to_output is not None:
            if self._pending_reasoning_newline:
                reasoning_to_output = "\n" + reasoning_to_output
                self._pending_reasoning_newline = False

            if reasoning_to_output.endswith("\n"):
                reasoning_to_output = reasoning_to_output.removesuffix("\n")
                if self.end_token in delta_text:
                    # Trailing "\n" is right before </think>, drop it.
                    self._pending_reasoning_newline = False
                else:
                    # Hold the trailing "\n" until we know whether </think> follows.
                    self._pending_reasoning_newline = True

        # Content: handle the newline immediately after </think>.
        if content_to_output is not None:
            # If we have content, reasoning must have ended.
            self._pending_reasoning_newline = False

            if self.end_token in delta_text and content_to_output.startswith("\n"):
                content_to_output = content_to_output.removeprefix("\n")

        reasoning_to_output = reasoning_to_output or None
        content_to_output = content_to_output or None
        if reasoning_to_output is None and content_to_output is None:
            return None

        return DeltaMessage(reasoning=reasoning_to_output, content=content_to_output)
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `Step3p5ReasoningParser.extract_reasoning_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Reasoning boundary detection**: Reasoning boundary detection. / 使用显式标记、token ID 或文本模式检测推理片段的边界。
- **Streaming extraction**: Streaming extraction. / 在流式解码时把新产生的推理内容与最终答案内容分开输出。
- **Parser specialization**: Parser specialization. / 通过子类覆盖或模型特定规则适配不同模型的推理格式。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `vllm.entrypoints.openai.engine.protocol`, `vllm.reasoning.basic_parsers`, `vllm.tokenizers`
