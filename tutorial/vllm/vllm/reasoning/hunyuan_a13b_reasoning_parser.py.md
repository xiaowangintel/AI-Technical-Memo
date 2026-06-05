# hunyuan_a13b_reasoning_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/reasoning/hunyuan_a13b_reasoning_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the hunyuan a13b reasoning parser that separates model reasoning traces from user-facing answer content in full and streaming responses. / 实现 hunyuan a13b 推理解析器，在完整响应与流式响应中将模型推理轨迹与面向用户的答案内容分离。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-12)
```python
from collections.abc import Sequence
from typing import TYPE_CHECKING

import regex as re
from transformers import PreTrainedTokenizerBase

from vllm.entrypoints.openai.engine.protocol import DeltaMessage
from vllm.logger import init_logger
from vllm.reasoning import ReasoningParser
```
**EN:** Imports tokenizer interfaces, protocol/message types, and shared helpers required to detect reasoning spans and split final content correctly.
**CN:** 导入分词器接口、协议/消息类型以及共享辅助工具，用于检测推理片段并正确切分最终内容。

### Module constants/configuration 1 (lines 18-18)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `HunyuanA13BReasoningParser` (lines 21-242)
```python
class HunyuanA13BReasoningParser(ReasoningParser):
    """
    Reasoning parser for Hunyuan A13B Model

    HunyuanReasoningParser

    This class implements a reasoning parser specifically designed
    for the Hunyuan A13B Model. It is responsible for parsing and
    extracting structured reasoning and answer segments from model
    outputs that follow a specific pattern.

    Key Features:
        - For non-stream output , Recognizes and extracts reasoning ("think")
         and answer ("answer") sections from text using regular expressions.
        - For stream process, it requires a token id sequences to change the
```
**EN:** Reasoning parser for Hunyuan A13B Model HunyuanReasoningParser This class implements a reasoning parser specifically designed for the Hunyuan A13B Model.
**CN:** 定义 `HunyuanA13BReasoningParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `HunyuanA13BReasoningParser.__init__` (lines 45-82)
```python
    def __init__(self, tokenizer: PreTrainedTokenizerBase, *args, **kwargs):
        super().__init__(tokenizer, *args, **kwargs)
        self.think_start_expr = r"<think>\n"
        self.think_end_expr = r"\n</think>\n"

        self.response_start_expr = r"\n</think>\n<answer>\n"
        self.response_end_expr = r"\n</answer>"

        self.full_match_reasoning_regex = re.compile(
            rf"(?:{self.think_start_expr}(.*?){self.response_start_expr})?(.*?){self.response_end_expr}",
            re.DOTALL,
        )

        self.half_match_reasoning_regex = re.compile(
            rf"{self.think_start_expr}(.*?){self.response_start_expr}(.*)", re.DOTALL
        )

        self.think_start_ids = [14023, 771, 397]
        self.think_start_ids_fast = [14023, 771, 1363]
        self.response_start_ids = [198, 524, 27963, 397, 27, 9399, 397]
        self.response_start_ids_fast = [524, 27963, 397, 27, 9399, 397]
        self.response_end_ids = [198, 524, 9399, 29]
        self.fast_think_ids = [14023, 771, 1363, 524, 27963, 397, 27, 9399, 397]

        # when state change, send out all the buffered text in last state
        self.buffered_text: list[str] = []
        self.buffered_ids: list[int] = []

        self.current_state = "reasoning"
        self.all_states = ["reasoning", "response"]

        self.current_state = "idle"
        self.expected_sequence = self.think_start_ids
        # this sequence only for the think start, it has two way to start.
        self.expected_sequence_side = self.think_start_ids_fast
        self.sequence_index = 0
        self.token_buffer: list[int] = []
        self.text_buffer = ""
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `HunyuanA13BReasoningParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `HunyuanA13BReasoningParser.is_reasoning_end` (lines 84-85)
```python
    def is_reasoning_end(self, input_ids: Sequence[int]) -> bool:
        return self.current_state == "response"
```
**EN:** This predicate checks whether a delimiter, completion condition, or format-specific state transition has been reached.
**CN:** `HunyuanA13BReasoningParser.is_reasoning_end` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Method `HunyuanA13BReasoningParser.extract_content_ids` (lines 87-92)
```python
    def extract_content_ids(self, input_ids: list[int]) -> list[int]:
        # for hunyuan streaming reason parsing, the stream parse
        # will call first, and the same token will be called in
        # is_reasoning_end and extract_content_ids
        # this id is not part of content, so just return [] here.
        return []
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `HunyuanA13BReasoningParser.extract_content_ids` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `HunyuanA13BReasoningParser.extract_reasoning` (lines 94-134)
```python
    def extract_reasoning(
        self, model_output: str, request: "ChatCompletionRequest | ResponsesRequest"
    ) -> tuple[str | None, str | None]:
        """Extract the reasoning content & content sections, respectively.
        If the sequence doesn't match what we expect, i.e., the model generates
        something else, all content is considered non-reasoning content.

        Args:
            model_output (str): Output of the model to be parsed.
            request (ChatCompletionRequest): Request being processed.

        Returns:
            tuple[Optional[str], Optional[str]]: Tuple pair containing the
            reasoning content and non-reasoning content.
        """

        re_match = self.full_match_reasoning_regex.findall(model_output)
        if re_match:
            reasoning, response_content = re_match[0]
            if len(reasoning) == 0:
                reasoning = None
            if len(response_content) == 0:
                response_content = None
            return reasoning, response_content

        fallback_regex = self.half_match_reasoning_regex
        fallback_match = fallback_regex.findall(model_output)
        if fallback_match:
            reasoning, response_content = fallback_match[0]

            if response_content.endswith(self.response_end_expr):
                response_content = response_content[: -len(self.response_end_expr)]

            if len(reasoning) == 0:
                reasoning = None
            if len(response_content) == 0:
                response_content = None

            return reasoning, response_content

        return None, model_output
```
**EN:** Extract the reasoning content & content sections, respectively.
**CN:** `HunyuanA13BReasoningParser.extract_reasoning` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `HunyuanA13BReasoningParser._is_strict_increasing_subsequence` (lines 136-146)
```python
    def _is_strict_increasing_subsequence(
        self, subsequence: Sequence[int], sequence: Sequence[int]
    ) -> bool:
        if not subsequence:
            return False

        sub_idx = 0
        for num in sequence:
            if sub_idx < len(subsequence) and num == subsequence[sub_idx]:
                sub_idx += 1
        return sub_idx == len(subsequence)
```
**EN:** Provides the `_is_strict_increasing_subsequence` helper used by this module to keep parsing logic modular and reusable.
**CN:** `HunyuanA13BReasoningParser._is_strict_increasing_subsequence` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `HunyuanA13BReasoningParser.extract_reasoning_streaming` (lines 148-242)
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
        # Define sequences
        think_start_sequence = self.think_start_ids
        response_start_sequence = self.response_start_ids
        response_end_sequence = self.response_end_ids

        assert len(delta_token_ids) == 1
        # Process each token in the delta
        token = delta_token_ids[0]

        def check_token_with_sequence(token):
            if self.current_state == "idle" or self.current_state == "think":
                return (
                    token == self.expected_sequence[self.sequence_index]
                    or token == self.expected_sequence_side[self.sequence_index]
                )
            else:
                return token == self.expected_sequence[self.sequence_index]

        def check_last_token(token):
            if self.current_state == "idle" or self.current_state == "think":
                # only return true if it's judge using a side sequence.
                if (
                    self.sequence_index - 1 < len(self.expected_sequence_side)
                    and token == self.expected_sequence_side[self.sequence_index - 1]
                ):
                    return self.sequence_index == len(self.expected_sequence_side)
                else:
                    return self.sequence_index == len(self.expected_sequence)
            else:
                return self.sequence_index == len(self.expected_sequence)

        # Check if token matches expected sequence
        token_in_state_seq = check_token_with_sequence(token)

        if token_in_state_seq:
            # Store matching token
            self.token_buffer.append(token)
            self.text_buffer += delta_text
            self.sequence_index += 1
            ## state change from idle->think->response->idle
# ... omitted for brevity / 为简洁省略 ...
                    return DeltaMessage(reasoning=buffered_content, content=None)
                else:
                    return DeltaMessage(reasoning=None, content=buffered_content)
            else:
                # No buffered content, send normally
                if self.current_state == "think":
                    return DeltaMessage(reasoning=delta_text, content=None)
                else:
                    return DeltaMessage(reasoning=None, content=delta_text)

        # If no content to send in this delta
        return None
```
**EN:** Extract content using token ID sequence state machine
**CN:** `HunyuanA13BReasoningParser.extract_reasoning_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Reasoning boundary detection**: Reasoning boundary detection. / 使用显式标记、token ID 或文本模式检测推理片段的边界。
- **Streaming extraction**: Streaming extraction. / 在流式解码时把新产生的推理内容与最终答案内容分开输出。
- **Parser specialization**: Parser specialization. / 通过子类覆盖或模型特定规则适配不同模型的推理格式。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc`, `typing`
- **Third-party / 第三方**: `regex`, `transformers`
- **Internal / 内部**: `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.reasoning`
