# granite_reasoning_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/reasoning/granite_reasoning_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the granite reasoning parser that separates model reasoning traces from user-facing answer content in full and streaming responses. / 实现 granite 推理解析器，在完整响应与流式响应中将模型推理轨迹与面向用户的答案内容分离。

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

### Class `GraniteReasoningParser` (lines 21-368)
```python
class GraniteReasoningParser(ReasoningParser):
    """
    Reasoning parser for IBM Granite.

    IBM granite models currently use "Here is my thought process:"
    and "Here is my response:" to separate its thinking / response outputs.
    """
```
**EN:** Reasoning parser for IBM Granite.
**CN:** 定义 `GraniteReasoningParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `GraniteReasoningParser.__init__` (lines 29-55)
```python
    def __init__(self, tokenizer: PreTrainedTokenizerBase, *args, **kwargs):
        super().__init__(tokenizer, *args, **kwargs)

        # NOTE: There have been some observed occurrences of quantized
        # instances of the current models using "Here's" instead of "Here is",
        # so to be safe, we match on both.
        self.think_start_expr = r"(?:Here's|Here is) my thought process:"
        self.response_start_expr = r"(?:Here's|Here is) my response:"

        self.reasoning_regex = re.compile(
            rf"{self.think_start_expr}(.*?){self.response_start_expr}(.*)", re.DOTALL
        )

        self.valid_think_starts = [
            "Here's my thought process:",
            "Here is my thought process:",
        ]
        self.valid_response_starts = ["Here's my response:", "Here is my response:"]

        # Substrings to match for sequence boundaries on raw text
        self.seq_boundary_end = ":"
        self.seq_boundary_start = "Here"

        # The longest any thinking / start of response message can be
        self.longest_think_start = max(
            len(think_start) for think_start in self.valid_think_starts
        )
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `GraniteReasoningParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `GraniteReasoningParser.extract_reasoning` (lines 57-78)
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
        re_match = self.reasoning_regex.findall(model_output)
        if not re_match:
            return None, model_output
        reasoning, response_content = re_match[0]
        if not response_content:
            return reasoning, None
        return reasoning, response_content
```
**EN:** Extract the reasoning content & content sections, respectively.
**CN:** `GraniteReasoningParser.extract_reasoning` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `GraniteReasoningParser.extract_reasoning_streaming` (lines 80-137)
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
        """Extract the reasoning content / content emitted by granite models;
        If the sequence doesn't match what we expect, i.e., the model generates
        something else, all content is considered non-reasoning content.

        NOTE: Granite models do not use a special token to start their reasoning
        and response sections; instead they have token sequences, e.g.,

                Here is my thought process: Foo Here is my response: Bar

        This increases the complexity of correctly handling streams, since we
        need to watch for specific sequences and correctly parse them without
        dropping content that is potentially overlapping & spanning multiple
        delta messages.

        Args:
            previous_text (str): Previous text outside of this delta message.
            current_text (str): Previous text + delta text.
            delta_text (str): Text to consider and parse content from.
            previous_token_ids (Sequence[int]): Token IDs of previous_text.
            current_token_ids (Sequence[int]): Token IDs of current_text.
            delta_token_ids (Sequence[int]): Token IDs of delta_text.

        Returns:
            Union[DeltaMessage, None]
                DeltaMessage with either reasoning content or content, or None.
        """
        reasoning, resp_seq_len, content = self._get_content_sections(current_text)
        # Either we haven't finished the start of the reasoning sequence,
        # or the model is generating something unexpected.
        if not reasoning:
            delta_message = self._get_delta_message_with_no_reasoning_bounds(
                current_text, delta_text
            )
        # We have a start of reasoning message, but have not yet finished
        # the start of response sequence.
        elif not content:
            delta_message = self._get_delta_message_with_no_response_bounds(
                current_text, reasoning, delta_text
            )
        # We've finished both the start of reasoning and start of response seq.
        else:
            # This should never happen since we matched on the response
            assert resp_seq_len is not None
            delta_message = self._get_delta_message_with_both_bounds(
                delta_text, reasoning, content, current_text, resp_seq_len
            )
        if not delta_message.content and not delta_message.reasoning:
            return None
        return delta_message
```
**EN:** Extract the reasoning content / content emitted by granite models; If the sequence doesn't match what we expect, i.e., the model generates something else, all content is considered non-reasoning content.
**CN:** `GraniteReasoningParser.extract_reasoning_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `GraniteReasoningParser._is_reasoning_start_substr` (lines 140-151)
```python
    def _is_reasoning_start_substr(self, text: str) -> bool:
        """Check if a text matches one of the possible start reasoning seqs.

        Args:
            text (str): Text to check for leading substr.

        Returns:
            bool: True if any of the possible reasoning start seqs match.
        """
        return any(
            think_start.startswith(text) for think_start in self.valid_think_starts
        )
```
**EN:** Check if a text matches one of the possible start reasoning seqs.
**CN:** `GraniteReasoningParser._is_reasoning_start_substr` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `GraniteReasoningParser._is_response_start_substr` (lines 153-165)
```python
    def _is_response_start_substr(self, text: str) -> bool:
        """Check if a text matches one of the possible start response seqs.

        Args:
            text (str): Text to check for leading substr.

        Returns:
            bool: True if any of the possible response start seqs match.
        """
        return any(
            response_start.startswith(text)
            for response_start in self.valid_response_starts
        )
```
**EN:** Check if a text matches one of the possible start response seqs.
**CN:** `GraniteReasoningParser._is_response_start_substr` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `GraniteReasoningParser._get_delta_message_with_no_reasoning_bounds` (lines 167-199)
```python
    def _get_delta_message_with_no_reasoning_bounds(
        self,
        current_text: str,
        delta_text: str,
    ) -> DeltaMessage:
        """Parse the delta message when the current text has not yet completed
        its start of reasoning sequence.

        Args:
            current_text (str): The full previous + delta text.
            delta_text (str): Text to consider and parse content from.

        Returns:
            DeltaMessage: Message containing the parsed content.
        """
        prev_longest_length = len(current_text) - len(delta_text)
        is_substr = self._is_reasoning_start_substr(current_text)
        was_substr = self._is_reasoning_start_substr(current_text[:prev_longest_length])

        # Check if we just generated something NOT in the special token seq;
        # if so, add everything that we previously skipped with this delta
        # message and append everything to content in the future.
        if was_substr and not is_substr:
            return DeltaMessage(
                reasoning=None,
                content=current_text,
            )
        if is_substr:
            # Might still be in the special token sequence; return nothing
            return DeltaMessage(reasoning=None, content=None)
        # Otherwise the sequence has already been broken and we already
        # corrected; just return the delta text as normal content.
        return DeltaMessage(reasoning=None, content=delta_text)
```
**EN:** Parse the delta message when the current text has not yet completed its start of reasoning sequence.
**CN:** `GraniteReasoningParser._get_delta_message_with_no_reasoning_bounds` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `GraniteReasoningParser._get_delta_message_with_no_response_bounds` (lines 201-277)
```python
    def _get_delta_message_with_no_response_bounds(
        self,
        current_text: str,
        reasoning: str,
        delta_text: str,
    ) -> DeltaMessage:
        """Parse the delta message when the current text has both reasoning
        content with no (response) content. NOTE that we may have overlapping
        tokens with the start of reasoning / start of response sequences on
        either side of the delta text.

        Args:
            current_text (str): The full previous + delta text.
            reasoning (str): reasoning content from current_text.
            delta_text (str): Text to consider and parse content from.

        Returns:
            DeltaMessage: Message containing the parsed content.
        """
        # If we have no reasoning content or explicitly end with the start of
        # response sequence, we are in transition to the response; need to be
        # careful here, since the final token (:) will match the reasoning
        # content and fully parse it out; we should not pass the : back.
        ends_with_start_response_seq = any(
            current_text.endswith(response_start)
            for response_start in self.valid_response_starts
        )
        if reasoning is None or ends_with_start_response_seq:
            return DeltaMessage(reasoning=None, content=None)

        # Consider previous / current text only within context of the reasoning
        previous_text = reasoning[: -len(delta_text)]
        current_text = reasoning

        # We need to be careful about adding unfinished response sequences;
        # Find the place at which we MIGHT be starting a response sequence
        prev_idx = previous_text.rfind(self.seq_boundary_start)
        delta_idx = delta_text.rfind(self.seq_boundary_start)

        # Check the state of potential start of response substring matches.
        prev_was_substr = (
            self._is_response_start_substr(previous_text[prev_idx:])
            if prev_idx >= 0
            else False
        )
        delta_continues_substr = (
            self._is_response_start_substr(current_text[prev_idx:])
            if prev_idx >= 0
            else False
        )
        delta_new_substr = (
            self._is_response_start_substr(delta_text[delta_idx:])
            if delta_idx >= 0
            else False
        )

        # Delta only contains potential continued response sequence text.
        if delta_continues_substr:
            return DeltaMessage(reasoning=None, content=None)

        if not prev_was_substr:
            # Delta may be starting a new response seq but has other text too.
            if delta_new_substr:
                return DeltaMessage(reasoning=delta_text[:delta_idx], content=None)
            # Normal case for most reasoning text (no potential special seqs).
            return DeltaMessage(reasoning=delta_text, content=None)
        # The substring that previously seemed to be a potential response
        # seq wasn't one; we need to add the content to the delta message,
        # and also slice off the potential response sequence
        elif delta_new_substr:
            reasoning = previous_text[prev_idx:] + delta_text[:delta_idx]
            return DeltaMessage(reasoning=reasoning, content=None)
        # No new substring yet, and we broke our old one; take the whole delta
        return DeltaMessage(
            reasoning=previous_text[prev_idx:] + delta_text,
            content=None,
        )
```
**EN:** Parse the delta message when the current text has both reasoning content with no (response) content.
**CN:** `GraniteReasoningParser._get_delta_message_with_no_response_bounds` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `GraniteReasoningParser._get_delta_message_with_both_bounds` (lines 279-320)
```python
    def _get_delta_message_with_both_bounds(
        self,
        delta_text: str,
        reasoning: str,
        response_content: str,
        current_text: str,
        response_seq_len: int,
    ) -> DeltaMessage:
        """Parse the delta message when the current text has both reasoning
        content and normal (response) content.

        Args:
            delta_text: Text to consider and parse content from.
            reasoning: reasoning content from current_text.
            response_content: response content from current_text.
            current_text: The full previous + delta text.
            response_seq_len: Len of the complete response sequence used.

        Returns:
            DeltaMessage: Message containing the parsed content.
        """
        # Always have content; take length to the end
        delta_content = delta_text[-len(response_content) :]
        reasoning_end_idx = len(delta_text) - (len(response_content) + response_seq_len)

        if reasoning_end_idx < 0:
            delta_reasoning = None
        else:
            # Get the starting offset
            start_reasoning_idx = (
                len(reasoning) + response_seq_len + len(response_content) - 1
            )
            delta_offset = len(current_text) - len(delta_text)
            start_offset = start_reasoning_idx - delta_offset
            if start_offset < 0:
                start_offset = 0
            delta_reasoning = delta_text[start_offset:reasoning_end_idx]

        return DeltaMessage(
            reasoning=delta_reasoning,
            content=delta_content,
        )
```
**EN:** Parse the delta message when the current text has both reasoning content and normal (response) content.
**CN:** `GraniteReasoningParser._get_delta_message_with_both_bounds` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `GraniteReasoningParser._get_content_sections` (lines 322-368)
```python
    def _get_content_sections(
        self, current_text: str
    ) -> tuple[str | None, int | None, str | None]:
        """Parse the text to extract the reasoning content / content
        if we have them.

        Args:
            current_text (str): The full previous + delta text.

        Returns:
            tuple[Optional[str], Optional[int], Optional[str]]: Tuple of len 3
            containing the reasoning content, the length of the response seq
            (if there is one) and the non-reasoning content.
        """
        current_chunk_start = 0
        start_reasoning = None
        parsed_content = False
        delimiter_idxs = [
            idx
            for idx, char in enumerate(current_text)
            if char == self.seq_boundary_end
        ]

        for current_chunk_end in delimiter_idxs:
            current_chunk = current_text[current_chunk_start:current_chunk_end]
            # Check to see if the start of reasoning seq if complete
            if start_reasoning is None:
                for think_start in self.valid_think_starts:
                    if current_chunk == think_start[:-1]:
                        start_reasoning = current_chunk_end + 1
                        current_chunk_start = current_chunk_end + 1
                        break

            # Check to see if the start of response seq if complete
            elif not parsed_content:
                for response_start in self.valid_response_starts:
                    if current_chunk[-len(response_start) + 1 :] == response_start[:-1]:
                        # Mark end of reasoning and start response content
                        # after the start of response sequence.
                        end_reasoning = current_chunk_end - len(response_start)
                        reasoning = current_text[start_reasoning:end_reasoning]
                        response_content = current_text[current_chunk_end + 1 :]
                        return reasoning, len(response_start), response_content

        if start_reasoning and not parsed_content:
            return current_text[start_reasoning:], None, None
        return None, None, None
```
**EN:** Parse the text to extract the reasoning content / content if we have them.
**CN:** `GraniteReasoningParser._get_content_sections` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

## Key Concepts / 关键概念
- **Reasoning boundary detection**: Reasoning boundary detection. / 使用显式标记、token ID 或文本模式检测推理片段的边界。
- **Streaming extraction**: Streaming extraction. / 在流式解码时把新产生的推理内容与最终答案内容分开输出。
- **Parser specialization**: Parser specialization. / 通过子类覆盖或模型特定规则适配不同模型的推理格式。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc`, `typing`
- **Third-party / 第三方**: `regex`, `transformers`
- **Internal / 内部**: `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.reasoning`
