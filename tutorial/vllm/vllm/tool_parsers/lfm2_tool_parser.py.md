# lfm2_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/lfm2_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the lfm2 tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 lfm2 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-29)
```python
import ast
from collections.abc import Sequence

import regex as re

import vllm.envs as envs
from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionRequest,
)
from vllm.entrypoints.openai.engine.protocol import (
    DeltaMessage,
    ExtractedToolCallInformation,
)
from vllm.entrypoints.openai.responses.protocol import ResponsesRequest
from vllm.logger import init_logger
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers.abstract_tool_parser import (
    Tool,
    ToolParser,
)
from vllm.tool_parsers.utils import (
    UnexpectedAstError,
    compute_tool_delta,
    handle_single_tool,
    make_valid_python,
)
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 31-34)
```python
logger = init_logger(__name__)

TOOL_CALL_START = "<|tool_call_start|>"
TOOL_CALL_END = "<|tool_call_end|>"
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `Lfm2ToolParser` (lines 37-343)
```python
class Lfm2ToolParser(ToolParser):
    """
    Tool call parser for LiquidAI LFM2/LFM2.5 models that produce pythonic
    tool calls wrapped in <|tool_call_start|> and <|tool_call_end|> tokens.

    Example model output:
        <|tool_call_start|>[get_weather(location="Paris")]<|tool_call_end|>
        The weather in Paris is sunny.

    Used when --enable-auto-tool-choice --tool-call-parser lfm2 are all set.
    """
```
**EN:** Tool call parser for LiquidAI LFM2/LFM2.5 models that produce pythonic tool calls wrapped in <|tool_call_start|> and <|tool_call_end|> tokens.
**CN:** 定义 `Lfm2ToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `Lfm2ToolParser.__init__` (lines 51-72)
```python
    def __init__(
        self,
        tokenizer: TokenizerLike,
        tools: list[Tool] | None = None,
    ):
        super().__init__(tokenizer, tools)

        self.tool_call_start_token_id = self.vocab.get(TOOL_CALL_START)
        self.tool_call_end_token_id = self.vocab.get(TOOL_CALL_END)

        if self.tool_call_start_token_id is None or self.tool_call_end_token_id is None:
            raise RuntimeError(
                "LFM2 tool parser could not locate "
                "<|tool_call_start|>/<|tool_call_end|> tokens in the "
                "tokenizer!"
            )

        # Trailing content already emitted to the client. Used by the
        # streaming path to suppress LFM2's frequent echo of the tool
        # call body after the first <|tool_call_end|> while still
        # allowing legitimate post-call prose through.
        self._trailing_emitted: str = ""
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `Lfm2ToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `Lfm2ToolParser.adjust_request` (lines 74-86)
```python
    def adjust_request(
        self, request: ChatCompletionRequest | ResponsesRequest
    ) -> ChatCompletionRequest | ResponsesRequest:
        request = super().adjust_request(request)
        if request.tools and request.tool_choice != "none":
            # The <|tool_call_start|>/<|tool_call_end|> sentinels are
            # registered as special tokens in the LFM2/LFM2.5 tokenizer.
            # With the default ``skip_special_tokens=True`` they are
            # stripped from the decoded text before reaching this parser,
            # so the tool block becomes invisible. Force the engine to
            # preserve them when tool calling is enabled.
            request.skip_special_tokens = False
        return request
```
**EN:** Provides the `adjust_request` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Lfm2ToolParser.adjust_request` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Lfm2ToolParser.current_tool_index` (lines 90-91)
```python
    def current_tool_index(self) -> int:
        return self.current_tool_id
```
**EN:** Provides the `current_tool_index` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Lfm2ToolParser.current_tool_index` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Lfm2ToolParser.current_tool_index` (lines 94-95)
```python
    def current_tool_index(self, value: int) -> None:
        self.current_tool_id = value
```
**EN:** Provides the `current_tool_index` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Lfm2ToolParser.current_tool_index` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Lfm2ToolParser._strip_echo` (lines 98-106)
```python
    def _strip_echo(raw_after: str) -> str:
        """Drop any orphan <|tool_call_end|> (and the preceding text) from
        trailing content. LFM2 occasionally echoes the call body after the
        first end token and caps it with a second end token; everything
        through the last such orphan is model garbage, not user content."""
        last_orphan = raw_after.rfind(TOOL_CALL_END)
        if last_orphan != -1:
            return raw_after[last_orphan + len(TOOL_CALL_END) :]
        return raw_after
```
**EN:** Drop any orphan <|tool_call_end|> (and the preceding text) from trailing content.
**CN:** `Lfm2ToolParser._strip_echo` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Lfm2ToolParser._extract_tool_call_text` (lines 109-142)
```python
    def _extract_tool_call_text(
        cls, model_output: str
    ) -> tuple[str | None, str | None]:
        """Extract the pythonic call text and surrounding content.

        Returns (tool_text, content) where tool_text is the text between
        the sentinel tokens and content is everything outside them.
        """
        start_idx = model_output.find(TOOL_CALL_START)
        if start_idx == -1:
            return None, model_output

        end_idx = model_output.find(TOOL_CALL_END, start_idx)
        if end_idx == -1:
            # Incomplete — treat entire text after start as tool call
            tool_text = model_output[start_idx + len(TOOL_CALL_START) :]
            content_before = model_output[:start_idx].strip()
            content = content_before or None
            return tool_text, content

        tool_text = model_output[start_idx + len(TOOL_CALL_START) : end_idx]
        content_before = model_output[:start_idx].strip()
        content_after = cls._strip_echo(
            model_output[end_idx + len(TOOL_CALL_END) :]
        ).strip()

        content_parts = []
        if content_before:
            content_parts.append(content_before)
        if content_after:
            content_parts.append(content_after)
        content = "\n".join(content_parts) if content_parts else None

        return tool_text, content
```
**EN:** Extract the pythonic call text and surrounding content.
**CN:** `Lfm2ToolParser._extract_tool_call_text` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Lfm2ToolParser.extract_tool_calls` (lines 144-193)
```python
    def extract_tool_calls(
        self, model_output: str, request: ChatCompletionRequest
    ) -> ExtractedToolCallInformation:
        tool_text, content = self._extract_tool_call_text(model_output)

        if tool_text is None:
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )

        tool_text = tool_text.strip()

        is_tool_call_pattern = False
        try:
            is_tool_call_pattern = (
                self.TOOL_CALL_REGEX.match(
                    tool_text,
                    timeout=envs.VLLM_TOOL_PARSE_REGEX_TIMEOUT_SECONDS,
                )
                is not None
            )
        except TimeoutError:
            logger.warning("Regex timeout occurred when matching tool call pattern.")

        if not is_tool_call_pattern:
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )

        try:
            module = ast.parse(tool_text)
            parsed = getattr(module.body[0], "value", None)
            if isinstance(parsed, ast.List) and all(
                isinstance(e, ast.Call) for e in parsed.elts
            ):
                return ExtractedToolCallInformation(
                    tools_called=True,
                    tool_calls=[
                        handle_single_tool(e)  # type: ignore
                        for e in parsed.elts
                    ],
                    content=content,
                )
            else:
                raise UnexpectedAstError("Tool output must be a list of function calls")
        except Exception:
            logger.exception("Error in extracting tool call from response.")
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `Lfm2ToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `Lfm2ToolParser.extract_tool_calls_streaming` (lines 195-343)
```python
    def extract_tool_calls_streaming(
        self,
        previous_text: str,
        current_text: str,
        delta_text: str,
        previous_token_ids: Sequence[int],
        current_token_ids: Sequence[int],
        delta_token_ids: Sequence[int],
        request: ChatCompletionRequest,
    ) -> DeltaMessage | None:
        # If the tool call start token hasn't appeared yet, stream as content.
        if TOOL_CALL_START not in current_text:
            return DeltaMessage(content=delta_text)

        # Compute leading content (before <|tool_call_start|>) that arrived
        # in this delta and hasn't been streamed yet. Without this, when the
        # prefix and the start token land in the same delta the prefix is
        # silently dropped — token-by-token streaming masked the bug because
        # the prefix tokens always arrived in earlier deltas.
        leading_content = ""
        if TOOL_CALL_START not in previous_text:
            start_idx = current_text.find(TOOL_CALL_START)
            # previous_text contained no start token, so it has already been
            # streamed via the no-start-token branch above.
            leading_content = current_text[len(previous_text) : start_idx]

        has_end_in_current = TOOL_CALL_END in current_text
        has_end_in_previous = TOOL_CALL_END in previous_text

        # Compute trailing content (after <|tool_call_end|>) not yet
        # streamed. LFM2 frequently echoes the tool call body again
        # after the first end token, capped with a second end token.
        # Suppress that echo:
        #   - If a second <|tool_call_end|> has appeared, treat
        #     everything through the last one as garbage.
        #   - If the trailing starts with `[` or `<` (potential echo
        #     body or another sentinel) and no second end token has
        #     arrived yet, buffer it instead of emitting.
        trailing_content = ""
        if has_end_in_current:
            end_idx = current_text.find(TOOL_CALL_END) + len(TOOL_CALL_END)
            full_trailing = current_text[end_idx:]
            stripped_trailing = self._strip_echo(full_trailing)
            if stripped_trailing == full_trailing:
                # No second end token yet — possibly mid-echo.
                lstripped = full_trailing.lstrip()
                if lstripped.startswith("[") or lstripped.startswith("<"):
                    # Suspect echo; hold off until resolved.
                    final_trailing = self._trailing_emitted
                else:
# ... omitted for brevity / 为简洁省略 ...
                    tool_calls=tool_deltas,
                )
            elif not added_text and self.current_tool_id > 0:
                return DeltaMessage(content="")
            else:
                return None
        except Exception:
            logger.exception("Error trying to handle streaming tool call.")
            logger.debug(
                "Skipping chunk as a result of tool streaming extraction error"
            )
            return _content_only_or_none()
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `Lfm2ToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `ast`, `collections.abc`
- **Third-party / 第三方**: `regex`
- **Internal / 内部**: `vllm.envs`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.responses.protocol`, `vllm.logger`, `vllm.tokenizers`, `vllm.tool_parsers.abstract_tool_parser`, `vllm.tool_parsers.utils`
