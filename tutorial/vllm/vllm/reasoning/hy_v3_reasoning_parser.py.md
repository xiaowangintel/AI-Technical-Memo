# hy_v3_reasoning_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/reasoning/hy_v3_reasoning_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the hy V3 reasoning parser that separates model reasoning traces from user-facing answer content in full and streaming responses. / 实现 hy V3 推理解析器，在完整响应与流式响应中将模型推理轨迹与面向用户的答案内容分离。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-14)
```python
from collections.abc import Iterable, Sequence

from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionRequest,
)
from vllm.entrypoints.openai.engine.protocol import DeltaMessage
from vllm.entrypoints.openai.responses.protocol import ResponsesRequest
from vllm.logger import init_logger
from vllm.reasoning.basic_parsers import BaseThinkingReasoningParser
from vllm.reasoning.identity_reasoning_parser import IdentityReasoningParser
from vllm.tokenizers import TokenizerLike
```
**EN:** Imports tokenizer interfaces, protocol/message types, and shared helpers required to detect reasoning spans and split final content correctly.
**CN:** 导入分词器接口、协议/消息类型以及共享辅助工具，用于检测推理片段并正确切分最终内容。

### Module constants/configuration 1 (lines 16-16)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `HYV3ReasoningParser` (lines 19-141)
```python
class HYV3ReasoningParser(BaseThinkingReasoningParser):
    """
    HYV3 parser that delegates to either HYV3ReasoningParser or
    IdentityReasoningParser based on `reasoning_effort`.

    The HYV3 model uses <think>...</think> tokens to denote reasoning text.
    This parser extracts the reasoning content from the model output.
    """
```
**EN:** HYV3 parser that delegates to either HYV3ReasoningParser or IdentityReasoningParser based on `reasoning_effort`.
**CN:** 定义 `HYV3ReasoningParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `HYV3ReasoningParser.__init__` (lines 28-50)
```python
    def __init__(self, tokenizer: TokenizerLike, *args, **kwargs):
        super().__init__(tokenizer, *args, **kwargs)

        # First, If there is reasoning_effort in chat_kwargs,
        # prioritize using chat_kwargs.reasoning_effort.
        # If it's not present, use the "reasoning_effort" field
        # at the outer level of the chat message.
        # Otherwise, If both are empty, assign "no_think".

        chat_kwargs = kwargs.get("chat_template_kwargs", {}) or {}
        reasoning_effort = (
            chat_kwargs.get("reasoning_effort")
            or kwargs.get("reasoning_effort")
            or "no_think"
        )

        logger.debug("reasoning_effort for choosing parser: %s", reasoning_effort)

        self._identity_parser: IdentityReasoningParser | None
        if reasoning_effort == "no_think":
            self._identity_parser = IdentityReasoningParser(tokenizer, *args, **kwargs)
        else:
            self._identity_parser = None
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `HYV3ReasoningParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `HYV3ReasoningParser.start_token` (lines 53-55)
```python
    def start_token(self) -> str:
        """The token that starts reasoning content."""
        return "<think>"
```
**EN:** The token that starts reasoning content.
**CN:** `HYV3ReasoningParser.start_token` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `HYV3ReasoningParser.end_token` (lines 58-60)
```python
    def end_token(self) -> str:
        """The token that ends reasoning content."""
        return "</think>"
```
**EN:** The token that ends reasoning content.
**CN:** `HYV3ReasoningParser.end_token` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `HYV3ReasoningParser.is_reasoning_end` (lines 62-66)
```python
    def is_reasoning_end(self, input_ids: Sequence[int]) -> bool:
        if self._identity_parser is not None:
            return self._identity_parser.is_reasoning_end(input_ids)

        return super().is_reasoning_end(input_ids)
```
**EN:** This predicate checks whether a delimiter, completion condition, or format-specific state transition has been reached.
**CN:** `HYV3ReasoningParser.is_reasoning_end` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Method `HYV3ReasoningParser.is_reasoning_end_streaming` (lines 68-76)
```python
    def is_reasoning_end_streaming(
        self, input_ids: Sequence[int], delta_ids: Iterable[int]
    ) -> bool:
        if self._identity_parser is not None:
            return self._identity_parser.is_reasoning_end_streaming(
                input_ids, delta_ids
            )

        return super().is_reasoning_end_streaming(input_ids, delta_ids)
```
**EN:** This predicate checks whether a delimiter, completion condition, or format-specific state transition has been reached.
**CN:** `HYV3ReasoningParser.is_reasoning_end_streaming` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Method `HYV3ReasoningParser.extract_content_ids` (lines 78-82)
```python
    def extract_content_ids(self, input_ids: list[int]) -> list[int]:
        if self._identity_parser is not None:
            return self._identity_parser.extract_content_ids(input_ids)

        return super().extract_content_ids(input_ids)
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `HYV3ReasoningParser.extract_content_ids` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `HYV3ReasoningParser.extract_reasoning` (lines 84-90)
```python
    def extract_reasoning(
        self, model_output: str, request: "ChatCompletionRequest | ResponsesRequest"
    ) -> tuple[str | None, str | None]:
        if self._identity_parser is not None:
            return self._identity_parser.extract_reasoning(model_output, request)

        return super().extract_reasoning(model_output, request)
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `HYV3ReasoningParser.extract_reasoning` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `HYV3ReasoningParser.extract_reasoning_streaming` (lines 92-141)
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
        if self._identity_parser is not None:
            return self._identity_parser.extract_reasoning_streaming(
                previous_text,
                current_text,
                delta_text,
                previous_token_ids,
                current_token_ids,
                delta_token_ids,
            )

        ret = super().extract_reasoning_streaming(
            previous_text,
            current_text,
            delta_text,
            previous_token_ids,
            current_token_ids,
            delta_token_ids,
        )
        if (
            ret is not None
            and self.start_token_id not in previous_token_ids
            and self.start_token_id not in delta_token_ids
        ):
            if self.end_token_id in delta_token_ids:
                # end token in delta with more tokens,
                # extract reasoning content and content
                end_index = delta_text.find(self.end_token)
                reasoning = delta_text[:end_index]
                content = delta_text[end_index + len(self.end_token) :]
                return DeltaMessage(
                    reasoning=reasoning,
                    content=content if content else None,
                )
            elif self.end_token_id in previous_token_ids:
                # end token in previous, thinking content ends
                return DeltaMessage(content=delta_text)
            else:
                # no end token in previous or delta, reasoning content continues
                return DeltaMessage(reasoning=delta_text)

        return ret
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `HYV3ReasoningParser.extract_reasoning_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Reasoning boundary detection**: Reasoning boundary detection. / 使用显式标记、token ID 或文本模式检测推理片段的边界。
- **Streaming extraction**: Streaming extraction. / 在流式解码时把新产生的推理内容与最终答案内容分开输出。
- **Parser specialization**: Parser specialization. / 通过子类覆盖或模型特定规则适配不同模型的推理格式。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.logger`, `vllm.reasoning.basic_parsers`, `vllm.reasoning.identity_reasoning_parser`, `vllm.tokenizers`
