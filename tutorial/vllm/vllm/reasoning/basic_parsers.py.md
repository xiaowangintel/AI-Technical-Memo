# basic_parsers.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/reasoning/basic_parsers.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Provides a reusable base parser for models that wrap reasoning inside explicit start/end thinking tokens. / 为使用显式思考起止标记包裹推理内容的模型提供可复用的基础解析器。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-11)
```python
from abc import abstractmethod
from collections.abc import Iterable, Sequence
from itertools import islice
from typing import TYPE_CHECKING

from vllm.entrypoints.openai.engine.protocol import DeltaMessage
from vllm.reasoning.abs_reasoning_parsers import ReasoningParser
from vllm.tokenizers import TokenizerLike
```
**EN:** Imports tokenizer interfaces, protocol/message types, and shared helpers required to detect reasoning spans and split final content correctly.
**CN:** 导入分词器接口、协议/消息类型以及共享辅助工具，用于检测推理片段并正确切分最终内容。

### Class `BaseThinkingReasoningParser` (lines 18-201)
```python
class BaseThinkingReasoningParser(ReasoningParser):
    """
    Base class for reasoning parsers that use thinking tokens.

    This class provides common functionality for parsers that use start and end
    tokens to delimit reasoning content (
        e.g., <think>...</think>, <seed:think>...</seed:think>).

    Subclasses must implement the start and end tokens via abstract
    properties.
    """
```
**EN:** Base class for reasoning parsers that use thinking tokens.
**CN:** 定义 `BaseThinkingReasoningParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `BaseThinkingReasoningParser.start_token` (lines 32-34)
```python
    def start_token(self) -> str:
        """The token that starts reasoning content."""
        raise NotImplementedError
```
**EN:** The token that starts reasoning content.
**CN:** `BaseThinkingReasoningParser.start_token` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `BaseThinkingReasoningParser.end_token` (lines 38-40)
```python
    def end_token(self) -> str:
        """The token that ends reasoning content."""
        raise NotImplementedError
```
**EN:** The token that ends reasoning content.
**CN:** `BaseThinkingReasoningParser.end_token` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `BaseThinkingReasoningParser.reasoning_start_str` (lines 43-44)
```python
    def reasoning_start_str(self) -> str:
        return self.start_token
```
**EN:** Provides the `reasoning_start_str` helper used by this module to keep parsing logic modular and reusable.
**CN:** `BaseThinkingReasoningParser.reasoning_start_str` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `BaseThinkingReasoningParser.__init__` (lines 50-70)
```python
    def __init__(self, tokenizer: TokenizerLike, *args, **kwargs):
        super().__init__(tokenizer, *args, **kwargs)

        if not self.model_tokenizer:
            raise ValueError(
                "The model tokenizer must be passed to the ReasoningParser "
                "constructor during construction."
            )

        if not self.start_token or not self.end_token:
            raise ValueError("start_token and end_token must be defined in subclasses")

        start_token_id = self.vocab.get(self.start_token)
        end_token_id = self.vocab.get(self.end_token)
        if start_token_id is None or end_token_id is None:
            raise RuntimeError(
                f"{self.__class__.__name__} reasoning parser could not locate "
                "think start/end tokens in the tokenizer!"
            )
        self.start_token_id: int = start_token_id
        self.end_token_id: int = end_token_id
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `BaseThinkingReasoningParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `BaseThinkingReasoningParser.is_reasoning_end` (lines 72-81)
```python
    def is_reasoning_end(self, input_ids: Sequence[int]) -> bool:
        start_token_id = self.start_token_id
        end_token_id = self.end_token_id

        for i in range(len(input_ids) - 1, -1, -1):
            if input_ids[i] == start_token_id:
                return False
            if input_ids[i] == end_token_id:
                return True
        return False
```
**EN:** This predicate checks whether a delimiter, completion condition, or format-specific state transition has been reached.
**CN:** `BaseThinkingReasoningParser.is_reasoning_end` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Method `BaseThinkingReasoningParser.is_reasoning_end_streaming` (lines 83-87)
```python
    def is_reasoning_end_streaming(
        self, input_ids: Sequence[int], delta_ids: Iterable[int]
    ) -> bool:
        end_token_id = self.end_token_id
        return end_token_id in delta_ids
```
**EN:** This predicate checks whether a delimiter, completion condition, or format-specific state transition has been reached.
**CN:** `BaseThinkingReasoningParser.is_reasoning_end_streaming` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Method `BaseThinkingReasoningParser.extract_content_ids` (lines 89-96)
```python
    def extract_content_ids(self, input_ids: list[int]) -> list[int]:
        """
        Extract the content after the end tokens
        """
        if self.end_token_id not in islice(input_ids, 0, max(0, len(input_ids) - 1)):
            return []
        else:
            return input_ids[input_ids.index(self.end_token_id) + 1 :]
```
**EN:** Extract the content after the end tokens
**CN:** `BaseThinkingReasoningParser.extract_content_ids` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `BaseThinkingReasoningParser.extract_reasoning_streaming` (lines 98-155)
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
        Extract reasoning content from a delta message.
        Handles streaming output where previous + delta = current.
        Uses token IDs for faster processing.
        """
        # Skip single special tokens
        if len(delta_token_ids) == 1 and (
            delta_token_ids[0] in [self.start_token_id, self.end_token_id]
        ):
            return None

        # Check if start token is present in previous or delta.
        # Keep compatibility with models that don't generate start tokens.
        if self.start_token_id in previous_token_ids:
            if self.end_token_id in delta_token_ids:
                # start token in previous, end token in delta,
                # extract reasoning content
                end_index = delta_text.find(self.end_token)
                reasoning = delta_text[:end_index]
                content = delta_text[end_index + len(self.end_token) :]
                return DeltaMessage(
                    reasoning=reasoning, content=content if content else None
                )
            elif self.end_token_id in previous_token_ids:
                # start token in previous, end token in previous,
                # reasoning content continues
                return DeltaMessage(content=delta_text)
            else:
                # start token in previous, no end token in previous or delta,
                # reasoning content continues
                return DeltaMessage(reasoning=delta_text)
        elif self.start_token_id in delta_token_ids:
            if self.end_token_id in delta_token_ids:
                # start token in delta, end token in delta,
                # extract reasoning content
                start_index = delta_text.find(self.start_token)
                end_index = delta_text.find(self.end_token)
                reasoning = delta_text[start_index + len(self.start_token) : end_index]
                content = delta_text[end_index + len(self.end_token) :]
                return DeltaMessage(
                    reasoning=reasoning, content=content if content else None
                )
            else:
                # start token in delta, no end token in delta,
                # reasoning content continues
                return DeltaMessage(reasoning=delta_text)
        else:
            # not find thinking start token
            return DeltaMessage(content=delta_text)
```
**EN:** Extract reasoning content from a delta message.
**CN:** `BaseThinkingReasoningParser.extract_reasoning_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `BaseThinkingReasoningParser.extract_reasoning` (lines 157-181)
```python
    def extract_reasoning(
        self, model_output: str, request: "ChatCompletionRequest | ResponsesRequest"
    ) -> tuple[str | None, str | None]:
        """
        Extract reasoning content from the model output.

        This is the base implementation that works for most models.
        Subclasses can override this method for specific behavior.
        """
        # Check if the start token is present in the model output, remove it
        # if it is present.
        model_output_parts = model_output.partition(self.start_token)
        model_output = (
            model_output_parts[2] if model_output_parts[1] else model_output_parts[0]
        )

        # For models that may not generate start token,
        # assume the reasoning content is always at the start.
        if self.end_token not in model_output:
            return model_output, None
        else:
            reasoning, _, content = model_output.partition(self.end_token)
            # If generation stops right after end-of-think, return null content
            final_content = content or None
            return reasoning, final_content
```
**EN:** Extract reasoning content from the model output.
**CN:** `BaseThinkingReasoningParser.extract_reasoning` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `BaseThinkingReasoningParser.count_reasoning_tokens` (lines 183-201)
```python
    def count_reasoning_tokens(self, token_ids: Sequence[int]) -> int:
        """Count tokens that fall within start/end thinking markers.

        Uses a depth counter so nested spans are handled safely and stray end
        tokens do not drive the counter negative.
        """
        count = 0
        depth = 0
        for token_id in token_ids:
            if token_id == self.start_token_id:
                depth += 1
                continue
            if token_id == self.end_token_id:
                if depth > 0:
                    depth -= 1
                continue
            if depth > 0:
                count += 1
        return count
```
**EN:** Count tokens that fall within start/end thinking markers.
**CN:** `BaseThinkingReasoningParser.count_reasoning_tokens` 方法统计与解析相关的片段或 token 数量，便于上层服务进行计数和报告。

## Key Concepts / 关键概念
- **Reasoning boundary detection**: Reasoning boundary detection. / 使用显式标记、token ID 或文本模式检测推理片段的边界。
- **Streaming extraction**: Streaming extraction. / 在流式解码时把新产生的推理内容与最终答案内容分开输出。
- **Parser specialization**: Parser specialization. / 通过子类覆盖或模型特定规则适配不同模型的推理格式。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `abc`, `collections.abc`, `itertools`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `vllm.entrypoints.openai.engine.protocol`, `vllm.reasoning.abs_reasoning_parsers`, `vllm.tokenizers`
