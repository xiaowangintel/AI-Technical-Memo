# olmo3_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/olmo3_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the olmo3 tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 olmo3 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-28)
```python
import ast
from collections.abc import Sequence

import regex as re
from transformers import PreTrainedTokenizerBase

import vllm.envs as envs
from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionRequest,
)
from vllm.entrypoints.openai.engine.protocol import (
    DeltaMessage,
    ExtractedToolCallInformation,
)
from vllm.logger import init_logger
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

### Module constants/configuration 1 (lines 30-30)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `Olmo3PythonicToolParser` (lines 33-230)
```python
class Olmo3PythonicToolParser(ToolParser):
    """
    Tool call parser for Olmo 3 models that produce tool calls as
    newline-separated pythonic strings.
    Used when --enable-auto-tool-choice --tool-call-parser pythonic are all set
    Code copied from pythonic_tool_parser.py and updated to handle
    - newline separated pythonic tool calls.
    - argument values being null/true/false instead of Pythonic literals.
    """
```
**EN:** Tool call parser for Olmo 3 models that produce tool calls as newline-separated pythonic strings.
**CN:** 定义 `Olmo3PythonicToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `Olmo3PythonicToolParser.__init__` (lines 55-60)
```python
    def __init__(
        self,
        tokenizer: PreTrainedTokenizerBase,
        tools: list[Tool] | None = None,
    ):
        super().__init__(tokenizer, tools)
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `Olmo3PythonicToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `Olmo3PythonicToolParser.current_tool_index` (lines 64-65)
```python
    def current_tool_index(self) -> int:
        return self.current_tool_id
```
**EN:** Provides the `current_tool_index` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Olmo3PythonicToolParser.current_tool_index` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Olmo3PythonicToolParser.current_tool_index` (lines 68-69)
```python
    def current_tool_index(self, value: int) -> None:
        self.current_tool_id = value
```
**EN:** Provides the `current_tool_index` helper used by this module to keep parsing logic modular and reusable.
**CN:** `Olmo3PythonicToolParser.current_tool_index` 方法提供可复用的辅助能力，使本模块的解析逻辑更清晰、更模块化。

### Method `Olmo3PythonicToolParser.extract_tool_calls` (lines 71-130)
```python
    def extract_tool_calls(
        self, model_output: str, request: ChatCompletionRequest
    ) -> ExtractedToolCallInformation:
        """
        Extract the tool calls from a complete model response.
        """
        original_model_output = model_output
        # Remove xml tags.
        match = re.search(
            r"<function_calls>(.*?)</function_calls>", model_output, re.DOTALL
        )
        if match:
            model_output = match.group(1).strip()
        # Make the newline separated function calls into a list.
        model_output = ", ".join(
            [line.strip() for line in model_output.splitlines() if line.strip()]
        )
        model_output = f"[{model_output}]"

        is_tool_call_pattern = False
        try:
            is_tool_call_pattern = (
                self.TOOL_CALL_REGEX.match(
                    model_output, timeout=envs.VLLM_TOOL_PARSE_REGEX_TIMEOUT_SECONDS
                )
                is not None
            )
        except TimeoutError:
            logger.warning("Regex timeout occurred when matching tool call pattern.")
            logger.debug(
                "Regex timeout occurred when matching user input: %s", model_output
            )

        if not is_tool_call_pattern:
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=original_model_output
            )

        try:
            module = ast.parse(model_output)
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
                    content=None,
                )
            else:
                raise UnexpectedAstError("Tool output must be a list of function calls")
        except Exception:
            logger.exception("Error in extracting tool call from response.")
            # Treat as regular text
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=original_model_output
            )
```
**EN:** Extract the tool calls from a complete model response.
**CN:** `Olmo3PythonicToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `Olmo3PythonicToolParser.extract_tool_calls_streaming` (lines 132-230)
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
        # All function calls start with the <function_calls> tag.
        # But since this is streaming, we may have seen only part of the tag.
        if not current_text.startswith("<"):
            return DeltaMessage(content=delta_text)

        try:
            # Remove xml tags.
            if current_text.startswith("<function_calls>"):
                current_text = current_text[len("<function_calls>") :]
            if current_text.endswith("</function_calls>"):
                current_text = current_text[: -len("</function_calls>")]

            valid_and_added_text = make_valid_python(current_text)
            if valid_and_added_text is None:
                return None
            valid_text, added_text = valid_and_added_text

            # Make the newline separated function calls into a list.
            valid_text = ", ".join(
                [line.strip() for line in valid_text.splitlines() if line.strip()]
            )
            valid_text = f"[{valid_text}]"
            module = ast.parse(valid_text)
            parsed = getattr(module.body[0], "value", None)
            if not isinstance(parsed, ast.List) or not all(
                isinstance(e, ast.Call) for e in parsed.elts
            ):
                raise UnexpectedAstError(
                    "Tool output must be a sequence of newline-separated calls"
                )
            tool_calls = [
                handle_single_tool(e)  # type: ignore
                for e in parsed.elts
            ]

            tool_deltas = []
            for index, new_call in enumerate(tool_calls):
                if index < self.current_tool_index:
                    continue

# ... omitted for brevity / 为简洁省略 ...
            elif not added_text and self.current_tool_id > 0:
                # Return an empty DeltaMessage once the tool calls are all done
                # so that finish_reason gets set.
                return DeltaMessage(content="")
            else:
                return None
        except Exception:
            logger.exception("Error trying to handle streaming tool call.")
            logger.debug(
                "Skipping chunk as a result of tool streaming extraction error"
            )
            return None
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `Olmo3PythonicToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `ast`, `collections.abc`
- **Third-party / 第三方**: `regex`, `transformers`
- **Internal / 内部**: `vllm.envs`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.tool_parsers.abstract_tool_parser`, `vllm.tool_parsers.utils`
