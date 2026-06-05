# ernie45_reasoning_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/reasoning/ernie45_reasoning_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the ernie45 reasoning parser that separates model reasoning traces from user-facing answer content in full and streaming responses. / 实现 ernie45 推理解析器，在完整响应与流式响应中将模型推理轨迹与面向用户的答案内容分离。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-11)
```python
from collections.abc import Sequence
from typing import TYPE_CHECKING

from transformers import PreTrainedTokenizerBase

from vllm.entrypoints.openai.engine.protocol import DeltaMessage
from vllm.logger import init_logger
from vllm.reasoning.basic_parsers import BaseThinkingReasoningParser
```
**EN:** Imports tokenizer interfaces, protocol/message types, and shared helpers required to detect reasoning spans and split final content correctly.
**CN:** 导入分词器接口、协议/消息类型以及共享辅助工具，用于检测推理片段并正确切分最终内容。

### Module constants/configuration 1 (lines 17-17)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `Ernie45ReasoningParser` (lines 20-162)
```python
class Ernie45ReasoningParser(BaseThinkingReasoningParser):
    """
    Reasoning parser for Ernie45 thinking model.
    The Ernie45 thinking model output format is
        abc\n</think>\n\n<response>\ndef\n</response>\n
    or  abc\n</think>\ndef
    """
```
**EN:** Reasoning parser for Ernie45 thinking model.
**CN:** 定义 `Ernie45ReasoningParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `Ernie45ReasoningParser.start_token` (lines 33-35)
```python
    def start_token(self) -> str:
        """The token that starts reasoning content."""
        return "<think>"
```
**EN:** The token that starts reasoning content.
**CN:** `Ernie45ReasoningParser.start_token` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Ernie45ReasoningParser.end_token` (lines 38-40)
```python
    def end_token(self) -> str:
        """The token that ends reasoning content."""
        return "</think>"
```
**EN:** The token that ends reasoning content.
**CN:** `Ernie45ReasoningParser.end_token` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Ernie45ReasoningParser.__init__` (lines 42-55)
```python
    def __init__(self, tokenizer: PreTrainedTokenizerBase, *args, **kwargs):
        super().__init__(tokenizer, *args, **kwargs)

        if not self.model_tokenizer:
            raise ValueError(
                "The model tokenizer must be passed to the ReasoningParser "
                "constructor during construction."
            )

        self.response_start_token_id = self.vocab.get(self.response_start_token)
        self.response_end_token_id = self.vocab.get(self.response_end_token)
        self.newline_token_id = self.vocab.get(self.newline_token)

        self.parser_token_ids = [self.end_token_id, self.response_end_token_id]
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `Ernie45ReasoningParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `Ernie45ReasoningParser.extract_reasoning_streaming` (lines 57-138)
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
        The Ernie45 thinking model output format is
            abc\n</think>\n\n<response>\ndef\n</response>\n
        or  abc\n</think>\ndef
        - 'abc' goes to reasoning
        - 'def' goes to content
        """
        # Skip single special tokens
        if len(delta_token_ids) == 1 and (
            delta_token_ids[0]
            in [
                self.start_token_id,
                self.end_token_id,
                self.response_start_token_id,
                self.response_end_token_id,
            ]
        ):
            return None

        # No <think> in previous or delta, also need to check for </think>.
        # Because the model may have generated </think> without <think>
        if self.end_token_id in delta_token_ids:
            # </think> in delta with more tokens,
            # extract reasoning content and content
            think_end_index = delta_text.find(self.end_token)
            reasoning = delta_text[:think_end_index]
            content = delta_text[think_end_index + len(self.end_token) :]
            content = content.lstrip("\n")
            response_start_idx = content.find(self.response_start_token)
            response_end_idx = content.rfind(self.response_end_token)
            if response_start_idx != -1:
                content = content[response_start_idx + len(self.response_start_token) :]
            if response_end_idx != -1:
                content = content[:response_end_idx]
            return DeltaMessage(
                reasoning=reasoning,
                content=content if content else None,
            )
# ... omitted for brevity / 为简洁省略 ...
            if (
                len(previous_token_ids) > 1
                and previous_token_ids[-2] == self.end_token_id
            ) and (
                len(delta_token_ids) > 0 and delta_token_ids[0] == self.newline_token_id
            ):
                content = content.lstrip("\n")

            return DeltaMessage(content=content if content else None)
        else:
            # no </think> in previous or delta, reasoning content continues
            return DeltaMessage(reasoning=delta_text)
```
**EN:** Extract reasoning content from a delta message.
**CN:** `Ernie45ReasoningParser.extract_reasoning_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `Ernie45ReasoningParser.extract_reasoning` (lines 140-162)
```python
    def extract_reasoning(
        self, model_output: str, request: "ChatCompletionRequest | ResponsesRequest"
    ) -> tuple[str | None, str | None]:
        """
        Extract reasoning content from the model output.
        The Ernie45 thinking model output format is
            abc\n</think>\n\n\n<response>\ndef\n</response>\n
        or  abc\n</think>\ndef
        - 'abc' goes to reasoning
        - 'def' goes to content
        Returns:
            tuple[Optional[str], Optional[str]]: reasoning content and content
        """
        reasoning, content = super().extract_reasoning(model_output, request)
        if content:
            start_idx = content.find(self.response_start_token)
            end_idx = content.rfind(self.response_end_token)
            # Simultaneously existing and in the correct order
            if start_idx != -1 and end_idx != -1 and start_idx < end_idx:
                content = content[start_idx + len(self.response_start_token) : end_idx]
        final_content = content or None

        return reasoning, final_content
```
**EN:** Extract reasoning content from the model output.
**CN:** `Ernie45ReasoningParser.extract_reasoning` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Reasoning boundary detection**: Reasoning boundary detection. / 使用显式标记、token ID 或文本模式检测推理片段的边界。
- **Streaming extraction**: Streaming extraction. / 在流式解码时把新产生的推理内容与最终答案内容分开输出。
- **Parser specialization**: Parser specialization. / 通过子类覆盖或模型特定规则适配不同模型的推理格式。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc`, `typing`
- **Third-party / 第三方**: `transformers`
- **Internal / 内部**: `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.reasoning.basic_parsers`
