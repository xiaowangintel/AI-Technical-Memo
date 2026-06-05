# ernie45_tool_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tool_parsers/ernie45_tool_parser.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the ernie45 tool parser that converts model-specific tool-calling text into structured OpenAI-compatible tool call objects. / 实现 ernie45 工具解析器，把模型专用的工具调用文本转换为兼容 OpenAI 的结构化工具调用对象。

## Line-by-Line Analysis / 逐行分析
### Imports and setup (lines 4-25)
```python
import json
from collections.abc import Sequence

import regex as re

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
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers.abstract_tool_parser import (
    Tool,
    ToolParser,
)
```
**EN:** Imports OpenAI-compatible protocol types, tokenizer helpers, and JSON/streaming utilities that support model-specific tool-call parsing.
**CN:** 导入兼容 OpenAI 的协议类型、分词器辅助工具以及 JSON/流式处理能力，以支持模型专用的工具调用解析。

### Module constants/configuration 1 (lines 27-27)
```python
logger = init_logger(__name__)
```
**EN:** Defines top-level constants, aliases, registries, or configuration tables that the rest of the file consults during parsing.
**CN:** 定义顶层常量、类型别名、注册表或配置表，供本文件后续解析流程查阅。

### Class `Ernie45ToolParser` (lines 30-213)
```python
class Ernie45ToolParser(ToolParser):
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        """
        Ernie thinking model format:
        abc\n</think>\n\n\n<tool_call>\ndef\n</tool_call>\n
        """
        super().__init__(tokenizer, tools)
        self.current_tool_name_sent = False
        self.prev_tool_call_arr: list[dict] = []
        self.current_tool_id = -1
        self.streamed_args_for_tool: list[str] = []
        self.think_end_token = "</think>"
        self.response_start_token: str = "<response>"
        self.response_end_token: str = "</response>"
        self.tool_call_start_token = "<tool_call>"
```
**EN:** Declares `Ernie45ToolParser`, the main parser class in this module. It extends ToolParser. It centralizes state and exposes the core behaviors used by the surrounding serving pipeline.
**CN:** 定义 `Ernie45ToolParser` 类，作为本模块的核心解析器类型。它集中管理状态，并通过后续方法向服务流程暴露关键行为。

### Method `Ernie45ToolParser.__init__` (lines 31-71)
```python
    def __init__(self, tokenizer: TokenizerLike, tools: list[Tool] | None = None):
        """
        Ernie thinking model format:
        abc\n</think>\n\n\n<tool_call>\ndef\n</tool_call>\n
        """
        super().__init__(tokenizer, tools)
        self.current_tool_name_sent = False
        self.prev_tool_call_arr: list[dict] = []
        self.current_tool_id = -1
        self.streamed_args_for_tool: list[str] = []
        self.think_end_token = "</think>"
        self.response_start_token: str = "<response>"
        self.response_end_token: str = "</response>"
        self.tool_call_start_token = "<tool_call>"
        self.tool_call_end_token = "</tool_call>"
        self.tool_calls_start_token = self.tool_call_start_token
        self.newline_token: str = "<0x0A>"

        self.tool_call_regex = re.compile(
            r"<tool_call>\s*(?P<json>\{.*?\})\s*</tool_call>", re.DOTALL
        )

        if not self.model_tokenizer:
            raise ValueError(
                "The model tokenizer must be passed to the ToolParser "
                "constructor during construction."
            )

        self.think_end_token_id = self.vocab.get(self.think_end_token)
        self.response_start_token_id = self.vocab.get(self.response_start_token)
        self.response_end_token_id = self.vocab.get(self.response_end_token)
        self.tool_call_start_token_id = self.vocab.get(self.tool_call_start_token)
        self.tool_call_end_token_id = self.vocab.get(self.tool_call_end_token)
        self.newline_token_id = self.vocab.get(self.newline_token)
        self.parser_token_ids = [
            self.think_end_token_id,
            self.response_start_token_id,
            self.response_end_token_id,
        ]

        self._buffer = ""
```
**EN:** Ernie thinking model format: abc </think> <tool_call> def </tool_call>
**CN:** `Ernie45ToolParser.__init__` 方法负责初始化解析状态、缓存可复用元数据，并校验后续解析所需的配置。

### Method `Ernie45ToolParser.extract_tool_calls` (lines 73-117)
```python
    def extract_tool_calls(
        self,
        model_output: str,
        request: ChatCompletionRequest,
    ) -> ExtractedToolCallInformation:
        # sanity check; avoid unnecessary processing
        if self.tool_calls_start_token not in model_output:
            return ExtractedToolCallInformation(
                tools_called=False, tool_calls=[], content=model_output
            )

        else:
            try:
                tool_call_json_list = self.tool_call_regex.findall(model_output)

                tool_calls = []
                for tool_call_json in tool_call_json_list:
                    tool_call_dict = json.loads(tool_call_json)
                    args_str = json.dumps(
                        tool_call_dict.get("arguments", {}), ensure_ascii=False
                    )
                    tool_calls.append(
                        ToolCall(
                            type="function",
                            function=FunctionCall(
                                name=tool_call_dict.get("name", ""),
                                arguments=args_str,
                            ),
                        )
                    )

                content = model_output[
                    : model_output.find(self.tool_calls_start_token)
                ].rstrip("\n")
                return ExtractedToolCallInformation(
                    tools_called=True,
                    tool_calls=tool_calls,
                    content=content if content else None,
                )

            except Exception:
                logger.exception("Error in extracting tool call from response.")
                return ExtractedToolCallInformation(
                    tools_called=False, tool_calls=[], content=model_output
                )
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `Ernie45ToolParser.extract_tool_calls` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

### Method `Ernie45ToolParser.extract_tool_calls_streaming` (lines 119-213)
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
        self._buffer += delta_text
        cur_text = self._buffer
        start_idx = cur_text.find(self.tool_call_start_token)
        if start_idx == -1:
            self._buffer = ""
            # At least one toolcall has been completed
            if self.current_tool_id > 0:
                cur_text = ""
            if self.current_tool_id == -1 and all(
                token_id == self.newline_token_id for token_id in previous_token_ids
            ):
                cur_text = cur_text.strip("\n")

            # handle <response> </response> when tool_call is not triggered
            # cur_text === delta_text
            content = cur_text
            if self.response_start_token_id in delta_token_ids:
                content = content.lstrip("\n")
                response_start_idx = content.find(self.response_start_token)
                content = content[response_start_idx + len(self.response_start_token) :]
                # if have </response>, remove it
                response_end_idx = content.rfind(self.response_end_token)
                if response_end_idx != -1:
                    content = content[:response_end_idx]
            elif self.response_end_token_id in delta_token_ids:
                response_end_idx = content.rfind(self.response_end_token)
                content = content[:response_end_idx]
            # remove \n after </think> or <response> or </response>
            if (
                len(previous_token_ids) > 0
                and previous_token_ids[-1] in self.parser_token_ids
            ) and (
                len(delta_token_ids) > 0 and delta_token_ids[0] == self.newline_token_id
            ):
                content = content.lstrip("\n")

            return DeltaMessage(content=content if content else None)
        logger.debug("cur_text = %s", cur_text)
        end_idx = cur_text.find(self.tool_call_end_token)
        if end_idx != -1:
# ... omitted for brevity / 为简洁省略 ...
                            arguments=tool_call.function.arguments,
                        ),
                    )
                ],
            )
            self.current_tool_id += 1
            self._buffer = cur_text[end_idx + len(self.tool_call_end_token) :]
            return delta

        self._buffer = cur_text[start_idx:]
        content = cur_text[:start_idx].rstrip("\n")
        return DeltaMessage(content=content if content else None)
```
**EN:** This is a core extraction routine that turns raw text or token streams into structured parser output.
**CN:** `Ernie45ToolParser.extract_tool_calls_streaming` 方法是核心提取逻辑，用于把原始文本或 token 流转换为结构化解析结果。

## Key Concepts / 关键概念
- **Tool-call normalization**: Tool-call normalization. / 把模型生成的专有工具调用格式归一化为 OpenAI 兼容结构。
- **Streaming deltas**: Streaming deltas. / 在增量生成过程中只发送新增的工具名称、参数或调用状态。
- **Schema-aware parsing**: Schema-aware parsing. / 结合工具 schema、结构标签或 JSON 约束提升调用格式可靠性。
- **Pattern matching**: Pattern matching. / 借助正则或字符串模式匹配识别格式化片段。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `collections.abc`
- **Third-party / 第三方**: `regex`
- **Internal / 内部**: `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.tokenizers`, `vllm.tool_parsers.abstract_tool_parser`
