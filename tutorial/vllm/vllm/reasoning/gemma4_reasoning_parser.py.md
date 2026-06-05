# gemma4_reasoning_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/reasoning/gemma4_reasoning_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the gemma4 reasoning parser that separates model reasoning traces from user-facing answer content in full and streaming responses. / 实现 gemma4 推理解析器，在完整响应与流式响应中将模型推理轨迹与面向用户的答案内容分离。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-9)
```python
from collections.abc import Sequence
from typing import TYPE_CHECKING

from vllm.entrypoints.openai.engine.protocol import DeltaMessage
from vllm.reasoning.basic_parsers import BaseThinkingReasoningParser
from vllm.tokenizers import TokenizerLike
```
**EN:** Imports tokenizer interfaces, protocol/message types, and shared helpers required to detect reasoning spans and split final content correctly.
**CN:** 导入分词器接口、协议/消息类型以及共享辅助工具，用于检测推理片段并正确切分最终内容。

### Module constants/configuration 1 (lines 20-20)
```python
_THOUGHT_PREFIX = "thought\n"
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `Gemma4ReasoningParser` (lines 23-214)
```python
class Gemma4ReasoningParser(BaseThinkingReasoningParser):
    """
    Reasoning parser for Google Gemma4 thinking models.

    Gemma4 uses <|channel>...<channel|> tokens to delimit reasoning/thinking
    content within its output. Thinking mode is activated by passing
    ``enable_thinking=True`` in the chat template kwargs, which injects a
    system turn containing <|think|> (token 98) to trigger chain-of-thought
    reasoning.

    Output pattern when thinking is enabled::

        <|channel>thought
        ...chain of thought reasoning...<channel|>
        Final answer text here.
```
**EN:** Reasoning parser for Google Gemma4 thinking models.
**CN:** 定义 `Gemma4ReasoningParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `Gemma4ReasoningParser.__init__` (lines 46-57)
```python
    def __init__(self, tokenizer: TokenizerLike, *args, **kwargs):
        super().__init__(tokenizer, *args, **kwargs)
        # Instance state for streaming prefix stripping.
        # Tracks only the reasoning text received from the base parser,
        # independent of current_text (which may contain pre-reasoning
        # content and lacks special token text due to
        # skip_special_tokens=True).
        self._reasoning_text: str = ""
        self._prefix_stripped: bool = False
        self.new_turn_token_id = self.vocab["<|turn>"]
        self.tool_call_token_id = self.vocab["<|tool_call>"]
        self.tool_response_token_id = self.vocab["<|tool_response>"]
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `Gemma4ReasoningParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `Gemma4ReasoningParser.adjust_request` (lines 59-64)
```python
    def adjust_request(
        self, request: "ChatCompletionRequest | ResponsesRequest"
    ) -> "ChatCompletionRequest | ResponsesRequest":
        """Disable special-token stripping to preserve boundary tokens."""
        request.skip_special_tokens = False
        return request
```
**EN:** Disable special-token stripping to preserve boundary tokens.
**CN:** `Gemma4ReasoningParser.adjust_request` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Gemma4ReasoningParser.start_token` (lines 67-69)
```python
    def start_token(self) -> str:
        """The token that starts reasoning content."""
        return "<|channel>"
```
**EN:** The token that starts reasoning content.
**CN:** `Gemma4ReasoningParser.start_token` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Gemma4ReasoningParser.end_token` (lines 72-74)
```python
    def end_token(self) -> str:
        """The token that ends reasoning content."""
        return "<channel|>"
```
**EN:** The token that ends reasoning content.
**CN:** `Gemma4ReasoningParser.end_token` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Gemma4ReasoningParser.is_reasoning_end` (lines 76-97)
```python
    def is_reasoning_end(self, input_ids: Sequence[int]) -> bool:
        start_token_id = self.start_token_id
        end_token_id = self.end_token_id
        new_turn_token_id = self.new_turn_token_id
        tool_call_token_id = self.tool_call_token_id
        tool_response_token_id = self.tool_response_token_id

        # Search from the end of input_ids to find the last match.
        for i in range(len(input_ids) - 1, -1, -1):
            if input_ids[i] == start_token_id:
                return False
            if input_ids[i] == tool_call_token_id:
                # We're generating a tool call, so reasoning must be ended.
                return True
            if input_ids[i] in (new_turn_token_id, tool_response_token_id):
                # We found a new turn or tool response token so don't consider
                # reasoning ended yet, since the model starts new reasoning
                # after these tokens.
                return False
            if input_ids[i] == end_token_id:
                return True
        return False
```
**EN:** This predicate checks whether a delimiter, completion condition, or format-specific state transition has been reached.
**CN:** `Gemma4ReasoningParser.is_reasoning_end` 方法用于判断是否到达分隔符、结束条件或特定格式的状态切换点。

### Method `Gemma4ReasoningParser.extract_reasoning` (lines 103-117)
```python
    def extract_reasoning(
        self,
        model_output: str,
        request: "ChatCompletionRequest | ResponsesRequest",
    ) -> tuple[str | None, str | None]:
        """Extract reasoning, stripping the ``thought\\n`` role label."""
        if self.start_token not in model_output and self.end_token not in model_output:
            # Default to content history if no tags are present
            # (or if they were stripped)
            return None, model_output

        reasoning, content = super().extract_reasoning(model_output, request)
        if reasoning is not None:
            reasoning = _strip_thought_label(reasoning)
        return reasoning, content
```
**EN:** Extract reasoning, stripping the ``thought\n`` role label.
**CN:** `Gemma4ReasoningParser.extract_reasoning` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `Gemma4ReasoningParser.extract_reasoning_streaming` (lines 123-214)
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
        """Extract streaming reasoning, stripping ``thought\\n`` from the
        first reasoning delta(s).

        The ``thought\\n`` prefix may arrive as a single delta or split
        across multiple deltas (e.g. ``"thought"`` then ``"\\n"``). We
        buffer early reasoning tokens until we can determine whether the
        prefix is present, then emit the buffered content minus the
        prefix.

        Unlike the previous implementation which reconstructed accumulated
        reasoning from ``current_text``, this uses instance state
        (``_reasoning_text``) to track only the reasoning content returned
        by the base parser. This is necessary because
        ``skip_special_tokens=True`` (the vLLM default) causes the
        ``<|channel>`` delimiter to be invisible in ``current_text``,
        making it impossible to separate pre-reasoning content from
        reasoning content via string matching.
        """
        result = super().extract_reasoning_streaming(
            previous_text,
            current_text,
            delta_text,
            previous_token_ids,
            current_token_ids,
            delta_token_ids,
        )
        if result is None:
            return None

        if result.reasoning is None:
            return result

        # Accumulate ONLY the reasoning text from base parser results.
        # This is immune to pre-reasoning content pollution.
        self._reasoning_text += result.reasoning

        # Once the prefix has been handled, all subsequent reasoning
        # deltas pass through unchanged.
        if self._prefix_stripped:
            return result

# ... omitted for brevity / 为简洁省略 ...
        # Buffer by suppressing — we can't yet tell if this will
        # become the full prefix or diverge.
        if _THOUGHT_PREFIX.startswith(self._reasoning_text):
            return None

        # Case 3: Accumulated text doesn't match the thought prefix
        # at all. This means prior deltas were buffered (suppressed
        # by Case 2) but the text diverged. Re-emit the full
        # accumulated text to avoid data loss.
        self._prefix_stripped = True
        result.reasoning = self._reasoning_text
        return result
```
**EN:** Extract streaming reasoning, stripping ``thought\n`` from the first reasoning delta(s).
**CN:** `Gemma4ReasoningParser.extract_reasoning_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Function `_strip_thought_label` (lines 217-225)
```python
def _strip_thought_label(text: str) -> str:
    """Remove the ``thought\\n`` role label from the beginning of text.

    Mirrors ``vllm.reasoning.gemma4_utils._strip_thought_label`` from the
    offline parser.
    """
    if text.startswith(_THOUGHT_PREFIX):
        return text[len(_THOUGHT_PREFIX) :]
    return text
```
**EN:** Remove the ``thought\n`` role label from the beginning of text.
**CN:** `_strip_thought_label` 函数提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

## Key Concepts / 关键概念
- **Reasoning boundary detection**: Reasoning boundary detection. / 使用显式标记、token ID 或文本模式检测推理片段的边界。
- **Streaming extraction**: Streaming extraction. / 在流式解码时把新产生的推理内容与最终答案内容分开输出。
- **Parser specialization**: Parser specialization. / 通过子类覆盖或模型特定规则适配不同模型的推理格式。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `vllm.entrypoints.openai.engine.protocol`, `vllm.reasoning.basic_parsers`, `vllm.tokenizers`
