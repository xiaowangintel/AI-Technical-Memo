# olmo3_reasoning_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/reasoning/olmo3_reasoning_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the olmo3 reasoning parser that separates model reasoning traces from user-facing answer content in full and streaming responses. / 实现 olmo3 推理解析器，在完整响应与流式响应中将模型推理轨迹与面向用户的答案内容分离。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-13)
```python
import dataclasses as dt
import enum
from collections.abc import Sequence
from typing import TYPE_CHECKING

import regex as re

from vllm.entrypoints.openai.engine.protocol import DeltaMessage
from vllm.logger import init_logger
from vllm.reasoning import ReasoningParser
```
**EN:** Imports tokenizer interfaces, protocol/message types, and shared helpers required to detect reasoning spans and split final content correctly.
**CN:** 导入分词器接口、协议/消息类型以及共享辅助工具，用于检测推理片段并正确切分最终内容。

### Module constants/configuration 1 (lines 20-20)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `Olmo3ReasoningState` (lines 23-25)
```python
class Olmo3ReasoningState(enum.Enum):
    REASONING = 1
```
**EN:** Declares `Olmo3ReasoningState`, the main parser class in this module. It extends enum.Enum. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `Olmo3ReasoningState` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Class `Indices` (lines 29-34)
```python
class Indices:
    start: int
```
**EN:** Declares `Indices`, the main parser class in this module. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `Indices` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `Indices.__len__` (lines 33-34)
```python
    def __len__(self):
        return self.end - self.start
```
**EN:** Provides the `__len__` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Indices.__len__` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Function `string_overlap` (lines 37-76)
```python
def string_overlap(a: str, b: str) -> tuple[Indices | None, Indices | None]:
    """
    Find the longest overlap where the end of string a matches the start
    of string b.

    Args:
        a: First string
        b: Second string

    Returns:
        Tuple of IndicesTuples representing the overlapping portions in each
        string, or a tuple of None if no overlap exists
    """

    # swap so a is always the shorter string
    a, b, swap = (a, b, False) if len(a) < len(b) else (b, a, True)

    # first check: is a fully contained in b?
    if a in b:
        ind_a = Indices(0, len(a))
        ind_b = Indices(b.index(a), b.index(a) + len(a))
        return (ind_b, ind_a) if swap else (ind_a, ind_b)

    # second check: does the end of a overlap with the
    #               beginning of b?
    for i in range(len(a) - 1, 0, -1):
        if a[-i:] == b[:i]:
            ind_a = Indices(len(a) - i, len(a))
            ind_b = Indices(0, i)
            return (ind_b, ind_a) if swap else (ind_a, ind_b)

    # third check: does the beginning of a overlap with
    #              the end of b?
    for i in range(len(a) - 1, 0, -1):
        if b[-i:] == a[:i]:
            ind_a = Indices(0, i)
            ind_b = Indices(len(b) - i, len(b))
            return (ind_b, ind_a) if swap else (ind_a, ind_b)

    return None, None
```
**EN:** Find the longest overlap where the end of string a matches the start of string b.
**CN:** `string_overlap` 函数提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Class `Olmo3ReasoningBuffer` (lines 80-190)
```python
class Olmo3ReasoningBuffer:
    think_start: str = "<think>"
```
**EN:** Declares `Olmo3ReasoningBuffer`, the main parser class in this module. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `Olmo3ReasoningBuffer` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `Olmo3ReasoningBuffer.process_buffer` (lines 91-137)
```python
    def process_buffer(self) -> DeltaMessage | None:
        start_think_idx = self.buffer.find(self.think_start)

        if start_think_idx >= 0:
            self.state = Olmo3ReasoningState.REASONING
            pretext, self.buffer = (
                self.buffer[:start_think_idx],
                self.buffer[start_think_idx + len(self.think_start) :],
            )
            if start_think_idx > 0:
                # this covers the case there's content before
                # the start of the reasoning block
                return DeltaMessage(content=pretext)

        end_think_idx = self.buffer.rfind(self.think_end)

        if end_think_idx >= 0:
            self.state = Olmo3ReasoningState.CONTENT
            pretext, self.buffer = (
                self.buffer[:end_think_idx],
                self.buffer[end_think_idx + len(self.think_end) :],
            )
            if end_think_idx > 0:
                # this covers the case there's content before
                # the end of the reasoning block
                return DeltaMessage(reasoning=pretext)

        if self.state == Olmo3ReasoningState.REASONING:
            # we are inside reasoning block, return and empty
            # the text buffer
            (
                text_buffer,
                self.buffer,
            ) = self.buffer, ""
            return DeltaMessage(reasoning=text_buffer)

        if self.state == Olmo3ReasoningState.CONTENT:
            # we are outside reasoning block, return and empty
            # the text buffer
            (
                text_buffer,
                self.buffer,
            ) = self.buffer, ""
            return DeltaMessage(content=text_buffer)

        # nothing to return unless we are in reasoning or content state
        return None
```
**EN:** Provides the `process_buffer` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Olmo3ReasoningBuffer.process_buffer` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Olmo3ReasoningBuffer.__len__` (lines 139-141)
```python
    def __len__(self):
        # is the length of the text buffer
        return len(self.buffer)
```
**EN:** Provides the `__len__` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Olmo3ReasoningBuffer.__len__` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Olmo3ReasoningBuffer.add_text` (lines 143-190)
```python
    def add_text(self, delta_text: str) -> DeltaMessage | None:
        # we start by adding the delta text to the buffer
        self.buffer += delta_text

        # setting this to empty before starting
        delta_message: DeltaMessage | None = None

        # we start by computing the overlap between the delta_text
        # and start/end of think tokens.
        _, overlap_think_start = string_overlap(delta_text, self.think_start)
        _, overlap_think_end = string_overlap(delta_text, self.think_end)

        partial_overlap_start = overlap_think_start is not None and len(
            overlap_think_start
        ) < len(self.think_start)
        partial_overlap_end = overlap_think_end is not None and len(
            overlap_think_end
        ) < len(self.think_end)

        if (
            partial_overlap_start
            and self.think_start in self.buffer
            and not partial_overlap_end
        ):
            # we can only process the buffer if partial overlap
            # is the last part of think token (thus causing
            # text_buffer to contain the start of think token)
            # and there are no partial overlaps with end think
            delta_message = self.process_buffer()

        elif partial_overlap_end and self.think_end in self.buffer:
            # same as before (partial overlap only allowed)
            # if the buffer contains the end think token,
            # but we don't have to check for partial overlap
            # with start think token because they are handled
            # by the previous condition
            delta_message = self.process_buffer()

        elif partial_overlap_start or partial_overlap_end:
            # in general, if there are overlaps, we don't
            # process the buffer because we want to wait until
            # the think token is fully completed.
            return None
        else:
            # we process the buffer as normal
            delta_message = self.process_buffer()

        return delta_message
```
**EN:** Provides the `add_text` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Olmo3ReasoningBuffer.add_text` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Class `Olmo3ReasoningParser` (lines 193-323)
```python
class Olmo3ReasoningParser(ReasoningParser):
    """
    Reasoning parser for Olmo 3 model

    Olmo3ReasoningParser

    This class implements a reasoning parser specifically designed for the
    Olmo 3 family of models. Olmo 3 models do not use special tokens to
    indicate reasoning; rather, reasoning trace is wrapped in `<think>` and
    `</think>`, which are tokenized using standard vocabulary entries.
    Because of this, the parser operates in string space, accumulating the
    characters in a buffer until it sees `<think>` or `</think>`. tokens
    to switch modes.

    Key Features:
```
**EN:** Reasoning parser for Olmo 3 model Olmo3ReasoningParser This class implements a reasoning parser specifically designed for the Olmo 3 family of models.
**CN:** 定义 `Olmo3ReasoningParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `Olmo3ReasoningParser.__init__` (lines 236-246)
```python
    def __init__(self, tokenizer: "TokenizerLike", *args, **kwargs):
        super().__init__(tokenizer, *args, **kwargs)
        self.buffer = Olmo3ReasoningBuffer(
            think_start=self.think_start, think_end=self.think_end
        )
        self.think_end_first_token_ids: list[int] = [
            self.vocab[token] for token in self.think_end_first_split
        ]
        self.think_end_rest_token_ids: list[int] = [
            self.vocab[token] for token in self.think_end_rest_split
        ]
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `Olmo3ReasoningParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `Olmo3ReasoningParser.reasoning_start_str` (lines 249-250)
```python
    def reasoning_start_str(self) -> str:
        return self.think_start
```
**EN:** Provides the `reasoning_start_str` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Olmo3ReasoningParser.reasoning_start_str` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Olmo3ReasoningParser.reasoning_end_str` (lines 253-254)
```python
    def reasoning_end_str(self) -> str:
        return self.think_end
```
**EN:** Provides the `reasoning_end_str` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Olmo3ReasoningParser.reasoning_end_str` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Olmo3ReasoningParser.is_reasoning_end` (lines 256-265)
```python
    def is_reasoning_end(self, input_ids: Sequence[int]) -> bool:
        rest_ids = self.think_end_rest_token_ids
        rest_len = len(rest_ids)
        for i in range(len(input_ids) - rest_len, -1, -1):
            if (
                list(input_ids[i + 1 : i + 1 + rest_len]) == rest_ids
                and input_ids[i] in self.think_end_first_token_ids
            ):
                return True
        return False
```
**EN:** This predicate checks whether a delimiter, completion condition, or format-specific state transition has been reached.
**CN:** `Olmo3ReasoningParser.is_reasoning_end` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Method `Olmo3ReasoningParser.extract_content_ids` (lines 267-272)
```python
    def extract_content_ids(self, input_ids: list[int]) -> list[int]:
        # for Olmo 3 streaming reason parsing, the stream parse
        # will call first, and the same token will be called in
        # is_reasoning_end and extract_content_ids
        # this id is not part of content, so just return [] here.
        return []
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `Olmo3ReasoningParser.extract_content_ids` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `Olmo3ReasoningParser.extract_reasoning` (lines 274-300)
```python
    def extract_reasoning(
        self,
        model_output: str,
        request: "ChatCompletionRequest | ResponsesRequest",
    ) -> tuple[str | None, str | None]:
        """Extract the reasoning content & content sections, respectively.
        If the sequence doesn't match what we expect, i.e., the model generates
        something else, all content is considered non-reasoning content.

        Args:
            model_output: Output of the model to be parsed.
            request: Request being
                processed.

        Returns:
            tuple[Optional[str], Optional[str]]: Tuple pair containing the
            reasoning content and non-reasoning content.
        """

        re_match = self.reasoning_regex.match(model_output)
        if re_match:
            reasoning = re_match.group("reasoning") or None
            content = re_match.group("content") or None
            return reasoning, content

        # no reasoning content
        return None, model_output
```
**EN:** Extract the reasoning content & content sections, respectively.
**CN:** `Olmo3ReasoningParser.extract_reasoning` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `Olmo3ReasoningParser.extract_reasoning_streaming` (lines 302-323)
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
        """Extract content using token ID sequence state machine"""

        delta_message = self.buffer.add_text(delta_text)
        if delta_message is None and self.buffer.think_end in self.buffer.buffer:
            # this is a bit hacky, but, because of how the buffer is
            # constructed, if the last delta_text contains characters that
            # marks the end of thinking tokens, then messages in the buffer
            # would never be processed because we get no other turn. To get
            # around that, we check if the text buffer contains the end of
            # thinking tokens, and, if so, we reprocess the buffer again.
            delta_message = self.buffer.process_buffer()

        return delta_message
```
**EN:** Extract content using token ID sequence state machine
**CN:** `Olmo3ReasoningParser.extract_reasoning_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Reasoning boundary detection**: Reasoning boundary detection. / 使用显式标记、token ID 或文本模式检测推理片段的边界。
- **Streaming extraction**: Streaming extraction. / 在流式解码时把新产生的推理内容与最终答案内容分开输出。
- **Parser specialization**: Parser specialization. / 通过子类覆盖或模型特定规则适配不同模型的推理格式。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `dataclasses`, `enum`, `collections.abc`, `typing`
- **Third-party / 第三方**: `regex`
- **Internal / 内部**: `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.reasoning`
