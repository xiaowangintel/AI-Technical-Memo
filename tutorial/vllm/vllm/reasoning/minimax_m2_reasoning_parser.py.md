# minimax_m2_reasoning_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/reasoning/minimax_m2_reasoning_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the minimax m2 reasoning parser that separates model reasoning traces from user-facing answer content in full and streaming responses. / 实现 minimax m2 推理解析器，在完整响应与流式响应中将模型推理轨迹与面向用户的答案内容分离。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-13)
```python
from collections.abc import Sequence
from typing import TYPE_CHECKING

from vllm.entrypoints.openai.engine.protocol import (
    DeltaMessage,
)
from vllm.logger import init_logger
from vllm.reasoning.abs_reasoning_parsers import ReasoningParser
from vllm.reasoning.basic_parsers import BaseThinkingReasoningParser
from vllm.tokenizers import TokenizerLike
```
**EN:** Imports tokenizer interfaces, protocol/message types, and shared helpers required to detect reasoning spans and split final content correctly.
**CN:** 导入分词器接口、协议/消息类型以及共享辅助工具，用于检测推理片段并正确切分最终内容。

### Module constants/configuration 1 (lines 19-19)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `MiniMaxM2ReasoningParser` (lines 22-78)
```python
class MiniMaxM2ReasoningParser(BaseThinkingReasoningParser):
    """
    Reasoning parser for MiniMax M2 model.

    MiniMax M2 models don't generate <think> start token, only </think> end
    token. All content before </think> is reasoning, content after is the
    actual response.
    """
```
**EN:** Reasoning parser for MiniMax M2 model.
**CN:** 定义 `MiniMaxM2ReasoningParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `MiniMaxM2ReasoningParser.start_token` (lines 32-34)
```python
    def start_token(self) -> str:
        """The token that starts reasoning content."""
        return "<think>"
```
**EN:** The token that starts reasoning content.
**CN:** `MiniMaxM2ReasoningParser.start_token` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `MiniMaxM2ReasoningParser.end_token` (lines 37-39)
```python
    def end_token(self) -> str:
        """The token that ends reasoning content."""
        return "</think>"
```
**EN:** The token that ends reasoning content.
**CN:** `MiniMaxM2ReasoningParser.end_token` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `MiniMaxM2ReasoningParser.extract_reasoning_streaming` (lines 41-78)
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
        Extract reasoning content from a delta message for streaming.

        MiniMax M2 models don't generate <think> start token, so we assume
        all content is reasoning until we encounter the </think> end token.
        """
        # Skip single end token
        if len(delta_token_ids) == 1 and delta_token_ids[0] == self.end_token_id:
            return None

        # Check if end token has already appeared in previous tokens
        # meaning we're past the reasoning phase
        if self.end_token_id in previous_token_ids:
            # We're past the reasoning phase, this is content
            return DeltaMessage(content=delta_text)

        # Check if end token is in delta tokens
        if self.end_token_id in delta_token_ids:
            # End token in delta, split reasoning and content
            end_index = delta_text.find(self.end_token)
            reasoning = delta_text[:end_index]
            content = delta_text[end_index + len(self.end_token) :]
            return DeltaMessage(
                reasoning=reasoning if reasoning else None,
                content=content if content else None,
            )

        # No end token yet, all content is reasoning
        return DeltaMessage(reasoning=delta_text)
```
**EN:** Extract reasoning content from a delta message for streaming.
**CN:** `MiniMaxM2ReasoningParser.extract_reasoning_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Class `MiniMaxM2AppendThinkReasoningParser` (lines 81-118)
```python
class MiniMaxM2AppendThinkReasoningParser(ReasoningParser):
    """
    Reasoning parser for MiniMax M2 model.
    """
```
**EN:** Reasoning parser for MiniMax M2 model.
**CN:** 定义 `MiniMaxM2AppendThinkReasoningParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `MiniMaxM2AppendThinkReasoningParser.__init__` (lines 86-89)
```python
    def __init__(self, tokenizer: TokenizerLike, *args, **kwargs):
        super().__init__(tokenizer, *args, **kwargs)
        self.end_token_id = self.vocab.get("</think>")
        self.start_token_id = self.vocab.get("<think>")
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `MiniMaxM2AppendThinkReasoningParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `MiniMaxM2AppendThinkReasoningParser.is_reasoning_end` (lines 91-97)
```python
    def is_reasoning_end(self, input_ids: Sequence[int]) -> bool:
        end_token_id = self.end_token_id
        start_token_id = self.start_token_id
        for input_id in reversed(input_ids):
            if input_id in (end_token_id, start_token_id):
                return input_id == end_token_id
        return False
```
**EN:** This predicate checks whether a delimiter, completion condition, or format-specific state transition has been reached.
**CN:** `MiniMaxM2AppendThinkReasoningParser.is_reasoning_end` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Method `MiniMaxM2AppendThinkReasoningParser.extract_content_ids` (lines 99-100)
```python
    def extract_content_ids(self, input_ids: list[int]) -> list[int]:
        return input_ids
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `MiniMaxM2AppendThinkReasoningParser.extract_content_ids` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `MiniMaxM2AppendThinkReasoningParser.extract_reasoning_streaming` (lines 102-113)
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
        if len(previous_token_ids) == 0:
            delta_text = "<think>" + delta_text
        return DeltaMessage(content=delta_text)
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `MiniMaxM2AppendThinkReasoningParser.extract_reasoning_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `MiniMaxM2AppendThinkReasoningParser.extract_reasoning` (lines 115-118)
```python
    def extract_reasoning(
        self, model_output: str, request: "ChatCompletionRequest | ResponsesRequest"
    ) -> tuple[str | None, str | None]:
        return None, "<think>" + model_output
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `MiniMaxM2AppendThinkReasoningParser.extract_reasoning` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Reasoning boundary detection**: Reasoning boundary detection. / 使用显式标记、token ID 或文本模式检测推理片段的边界。
- **Streaming extraction**: Streaming extraction. / 在流式解码时把新产生的推理内容与最终答案内容分开输出。
- **Parser specialization**: Parser specialization. / 通过子类覆盖或模型特定规则适配不同模型的推理格式。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.reasoning.abs_reasoning_parsers`, `vllm.reasoning.basic_parsers`, `vllm.tokenizers`
