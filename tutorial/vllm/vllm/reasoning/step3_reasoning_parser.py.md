# step3_reasoning_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/reasoning/step3_reasoning_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the step3 reasoning parser that separates model reasoning traces from user-facing answer content in full and streaming responses. / 实现 step3 推理解析器，在完整响应与流式响应中将模型推理轨迹与面向用户的答案内容分离。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-13)
```python
from collections.abc import Iterable, Sequence
from itertools import islice
from typing import TYPE_CHECKING

import regex as re
from transformers import PreTrainedTokenizerBase

from vllm.entrypoints.openai.engine.protocol import DeltaMessage
from vllm.logger import init_logger
from vllm.reasoning import ReasoningParser
```
**EN:** Imports tokenizer interfaces, protocol/message types, and shared helpers required to detect reasoning spans and split final content correctly.
**CN:** 导入分词器接口、协议/消息类型以及共享辅助工具，用于检测推理片段并正确切分最终内容。

### Module constants/configuration 1 (lines 19-19)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `Step3ReasoningParser` (lines 22-128)
```python
class Step3ReasoningParser(ReasoningParser):
    """
    Reasoning parser for Step3 model.

    The Step3 model uses </think> token to denote the end of reasoning
    text. This parser extracts all content before </think> as reasoning content.
    """
```
**EN:** Reasoning parser for Step3 model.
**CN:** 定义 `Step3ReasoningParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `Step3ReasoningParser.__init__` (lines 30-49)
```python
    def __init__(self, tokenizer: PreTrainedTokenizerBase, *args, **kwargs):
        super().__init__(tokenizer, *args, **kwargs)
        self.think_start_token = "<think>"
        self.think_end_token = "</think>"

        self.reasoning_regex = re.compile(rf"(.*?){self.think_end_token}", re.DOTALL)

        if not self.model_tokenizer:
            raise ValueError(
                "The model tokenizer must be passed to the ReasoningParser "
                "constructor during construction."
            )

        think_end_token_id = self.vocab.get(self.think_end_token)
        if think_end_token_id is None:
            raise RuntimeError(
                "Step3 reasoning parser could not locate think end "
                "token in the tokenizer!"
            )
        self.think_end_token_id: int = think_end_token_id
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `Step3ReasoningParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `Step3ReasoningParser.reasoning_start_str` (lines 52-53)
```python
    def reasoning_start_str(self) -> str:
        return self.think_start_token
```
**EN:** Provides the `reasoning_start_str` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Step3ReasoningParser.reasoning_start_str` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Step3ReasoningParser.reasoning_end_str` (lines 56-57)
```python
    def reasoning_end_str(self) -> str:
        return self.think_end_token
```
**EN:** Provides the `reasoning_end_str` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Step3ReasoningParser.reasoning_end_str` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Step3ReasoningParser.extract_reasoning_streaming` (lines 59-94)
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
        For text "abc</think>xyz":
        - 'abc' goes to reasoning
        - 'xyz' goes to content
        """
        # Skip single special token
        if len(delta_token_ids) == 1 and delta_token_ids[0] == self.think_end_token_id:
            return None

        if self.think_end_token_id in delta_token_ids:
            # </think> in delta, extract reasoning content and remaining content
            end_index = delta_text.find(self.think_end_token)
            reasoning = delta_text[:end_index]
            content = delta_text[end_index + len(self.think_end_token) :]
            return DeltaMessage(
                reasoning=reasoning,
                content=content if content else None,
            )
        elif self.think_end_token_id in previous_token_ids:
            # </think> already seen in previous text, everything is content
            return DeltaMessage(content=delta_text)
        else:
            # No </think> seen yet, everything is reasoning
            return DeltaMessage(reasoning=delta_text)
```
**EN:** Extract reasoning content from a delta message.
**CN:** `Step3ReasoningParser.extract_reasoning_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `Step3ReasoningParser.extract_reasoning` (lines 96-111)
```python
    def extract_reasoning(
        self, model_output: str, request: "ChatCompletionRequest | ResponsesRequest"
    ) -> tuple[str | None, str | None]:
        # Check if the model output contains the </think> token
        if self.think_end_token not in model_output:
            # If no </think> token, everything is reasoning content
            return model_output, None
        else:
            # Find the first occurrence of </think>
            end_index = model_output.find(self.think_end_token)
            reasoning = model_output[:end_index]

            # Content after </think> token
            content = model_output[end_index + len(self.think_end_token) :] or None

            return reasoning, content
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `Step3ReasoningParser.extract_reasoning` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `Step3ReasoningParser.is_reasoning_end` (lines 113-114)
```python
    def is_reasoning_end(self, input_ids: Sequence[int]) -> bool:
        return self.think_end_token_id in input_ids
```
**EN:** This predicate checks whether a delimiter, completion condition, or format-specific state transition has been reached.
**CN:** `Step3ReasoningParser.is_reasoning_end` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Method `Step3ReasoningParser.is_reasoning_end_streaming` (lines 116-120)
```python
    def is_reasoning_end_streaming(
        self, input_ids: Sequence[int], delta_ids: Iterable[int]
    ) -> bool:
        end_token_id = self.think_end_token_id
        return end_token_id in delta_ids
```
**EN:** This predicate checks whether a delimiter, completion condition, or format-specific state transition has been reached.
**CN:** `Step3ReasoningParser.is_reasoning_end_streaming` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Method `Step3ReasoningParser.extract_content_ids` (lines 122-128)
```python
    def extract_content_ids(self, input_ids: list[int]) -> list[int]:
        if self.think_end_token_id not in islice(
            input_ids, 0, max(0, len(input_ids) - 1)
        ):
            return []
        else:
            return input_ids[input_ids.index(self.think_end_token_id) + 1 :]
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `Step3ReasoningParser.extract_content_ids` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Reasoning boundary detection**: Reasoning boundary detection. / 使用显式标记、token ID 或文本模式检测推理片段的边界。
- **Streaming extraction**: Streaming extraction. / 在流式解码时把新产生的推理内容与最终答案内容分开输出。
- **Parser specialization**: Parser specialization. / 通过子类覆盖或模型特定规则适配不同模型的推理格式。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc`, `itertools`, `typing`
- **Third-party / 第三方**: `regex`, `transformers`
- **Internal / 内部**: `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.reasoning`
