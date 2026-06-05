# llama_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/llama_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the llama tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 llama 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-34)
```python
import json
from collections.abc import Sequence

import partial_json_parser
import regex as re
from partial_json_parser.core.options import Allow
from transformers import PreTrainedTokenizerBase

import vllm.envs as envs
from vllm.entrypoints.chat_utils import make_tool_call_id
from vllm.entrypoints.openai.chat_completion.protocol import (
    ChatCompletionRequest,
)
from vllm.entrypoints.openai.engine.protocol import (
    DeltaFunctionCall,
    DeltaMessage,
    DeltaToolCall,
    ExtractedToolCallInformation,
    FunctionCall,
    ToolCall,
)
from vllm.logger import init_logger
from vllm.tool_parsers.abstract_tool_parser import (
    Tool,
    ToolParser,
)
from vllm.tool_parsers.utils import (
    find_common_prefix,
    is_complete_json,
    partial_json_loads,
)
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 36-36)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `Llama3JsonToolParser` (lines 39-335)
```python
class Llama3JsonToolParser(ToolParser):
    """
    Tool call parser for Llama 3.x and 4 models intended for use with the
    examples/tool_chat_template_llama.jinja template.

    Used when --enable-auto-tool-choice --tool-call-parser llama3_json or
    llama4_json are set.
    """
```
**EN:** Tool call parser for Llama 3.x and 4 models intended for use with the examples/tool_chat_template_llama.jinja template.
**CN:** 定义 `Llama3JsonToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `Llama3JsonToolParser.__init__` (lines 54-74)
```python
    def __init__(
        self,
        tokenizer: PreTrainedTokenizerBase,
        tools: list[Tool] | None = None,
    ):
        super().__init__(tokenizer, tools)

        # initialize properties used for state when parsing tool calls in
        # streaming mode
        self.prev_tool_call_arr: list[dict] = []
        self.current_tool_id: int = -1
        self.current_tool_name_sent: bool = False
        self.streamed_args_for_tool: list[
            str
        ] = []  # map what has been streamed for each tool so far to a list
        self.bot_token_id = self.vocab.get(self.bot_token)
        if self.bot_token_id is None:
            raise RuntimeError(
                "Llama3JsonToolParser could not locate the bot token "
                f"'{self.bot_token}' in the tokenizer."
            )
```
**EN:** Initializes parser state, caches reusable metadata, and validates the configuration needed by later parsing steps.
**CN:** `Llama3JsonToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `Llama3JsonToolParser.extract_tool_calls` (lines 76-167)
```python
    def extract_tool_calls(
        self, model_output: str, request: ChatCompletionRequest
    ) -> ExtractedToolCallInformation:
        """
        Extract the tool calls from a complete model response.
        Only extracts JSON content and ignores any surrounding plain text.
        Supports both single JSON and multiple JSONs separated by semicolons.
        """
        # Quick check before running regex
        if not (self.bot_token in model_output or "{" in model_output):
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )

        # Keep track of the end index of the last parsed JSON object
        # so we don't parse inner brackets
        end_index = -1
        tool_calls: list[ToolCall] = []

        try:
            for match in self.tool_call_start_regex.finditer(
                model_output, timeout=envs.VLLM_TOOL_PARSE_REGEX_TIMEOUT_SECONDS
            ):
                start_index = match.start()
                # Skip if this brace is inside a previously parsed JSON object
                if start_index <= end_index:
                    continue

                try:
                    obj, json_end_index = self.json_decoder.raw_decode(
                        model_output[start_index:]
                    )
                    end_index = start_index + json_end_index

                    # raise KeyError if missing
                    name = obj["name"]
                    arguments_or_params = (
                        obj["arguments"] if "arguments" in obj else obj["parameters"]
                    )

                    tool_calls.append(
                        ToolCall(
                            type="function",
                            function=FunctionCall(
                                name=name,
                                # function call args are JSON but as a string
                                arguments=json.dumps(
                                    arguments_or_params, ensure_ascii=False
                                ),
                            ),
# ... omitted for brevity / 为简洁省略 ...
            )

        # If we have valid tool calls, return them normally
        if tool_calls:
            return ExtractedToolCallInformation(
                tools_called=True, tool_calls=tool_calls, content=None
            )

        # No valid tool calls found
        return ExtractedToolCallInformation(
            tools_called=False, tool_calls=[], content=model_output
        )
```
**EN:** Extract the tool calls from a complete model response.
**CN:** `Llama3JsonToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `Llama3JsonToolParser.extract_tool_calls_streaming` (lines 169-335)
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
        if not (
            current_text.startswith(self.bot_token) or current_text.startswith("{")
        ):
            return DeltaMessage(content=delta_text)

        # bit mask flags for partial JSON parsing. If the name hasn't been
        # sent yet, don't allow sending
        # an incomplete string since OpenAI only ever (as far as I have
        # seen) allows sending the entire tool/ function name at once.
        flags = Allow.ALL if self.current_tool_name_sent else Allow.ALL & ~Allow.STR
        try:
            tool_call_arr = []
            is_complete = []
            try:
                # depending on the prompt format the Llama model may or may not
                # prefix the output with the <|python_tag|> token
                start_idx = (
                    len(self.bot_token)
                    if current_text.startswith(self.bot_token)
                    else 0
                )
                while start_idx < len(current_text):
                    (obj, end_idx) = partial_json_loads(current_text[start_idx:], flags)
                    is_complete.append(
                        is_complete_json(current_text[start_idx : start_idx + end_idx])
                    )
                    start_idx += end_idx + len("; ")
                    # depending on the prompt Llama can use
                    # either arguments or parameters
                    if "parameters" in obj:
                        assert "arguments" not in obj, (
                            "model generated both parameters and arguments"
                        )
                        obj["arguments"] = obj["parameters"]
                    tool_call_arr.append(obj)
            except partial_json_parser.core.exceptions.MalformedJSON:
                logger.debug("not enough tokens to parse into JSON yet")
                return None

            # select as the current tool call the one we're on the state at
# ... omitted for brevity / 为简洁省略 ...
                            argument_diff
                        )

            self.prev_tool_call_arr = tool_call_arr
            return delta

        except Exception:
            logger.exception("Error trying to handle streaming tool call.")
            logger.debug(
                "Skipping chunk as a result of tool streaming extraction error"
            )
            return None
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `Llama3JsonToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Partial JSON recovery**: Partial JSON recovery. / 在 JSON 尚未闭合时尽量提取稳定的中间结果。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `collections.abc`
- **Third-party / 第三方**: `partial_json_parser`, `regex`, `partial_json_parser.core.options`, `transformers`
- **Internal / 内部**: `vllm.envs`, `vllm.entrypoints.chat_utils`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.tool_parsers.abstract_tool_parser`, `vllm.tool_parsers.utils`
