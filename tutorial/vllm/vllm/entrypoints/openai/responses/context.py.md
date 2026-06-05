# context.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/openai/responses/context.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Maintains request/response context objects shared across handlers. Scope: OpenAI-compatible responses. / 维护处理器共享的请求/响应上下文对象。 作用域：OpenAI 兼容 / 响应。

## Line-by-Line Analysis / 逐行分析
### Lines 3-47 — Imports and shared dependencies
```python
import asyncio
import contextlib
import copy
import json
import logging
from abc import ABC, abstractmethod
from contextlib import AsyncExitStack
from dataclasses import replace
from typing import TYPE_CHECKING, Any, Final, Union

from openai.types.responses.response_function_tool_call_output_item import (
    ResponseFunctionToolCallOutputItem,
)
from openai.types.responses.tool import Mcp
from openai_harmony import Author, Message, Role, StreamState, TextContent

from vllm import envs
from vllm.entrypoints.chat_utils import (
    ChatTemplateContentFormatOption,
)
from vllm.entrypoints.constants import MCP_PREFIX
from vllm.entrypoints.mcp.tool import Tool
from vllm.entrypoints.mcp.tool_server import ToolServer
from vllm.entrypoints.openai.engine.protocol import (
    FunctionCall,
)
...
    ResponsesRequest,
)
from vllm.entrypoints.openai.responses.utils import construct_tool_dicts
from vllm.outputs import RequestOutput
from vllm.reasoning.abs_reasoning_parsers import ReasoningParser
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers.abstract_tool_parser import ToolParser
from vllm.utils import random_uuid
```
**EN:** This import block pulls in standard-library modules such as `asyncio`, `contextlib`, `copy`, `json`, `logging`, `abc`, uses third-party packages like `openai`, `openai_harmony`, depends on internal helpers such as `vllm`, `vllm.entrypoints.chat_utils`, `vllm.entrypoints.constants`, `vllm.entrypoints.mcp.tool`, `vllm.entrypoints.mcp.tool_server`, `vllm.entrypoints.openai.engine.protocol`.
**CN:** 该导入块引入 `asyncio`, `contextlib`, `copy`, `json`, `logging`, `abc` 等标准库模块，使用 `openai`, `openai_harmony` 等第三方库，依赖 `vllm`, `vllm.entrypoints.chat_utils`, `vllm.entrypoints.constants`, `vllm.entrypoints.mcp.tool`, `vllm.entrypoints.mcp.tool_server`, `vllm.entrypoints.openai.engine.protocol` 等 vLLM 内部模块。

### Lines 49-50 — Conditional top-level flow
```python
if TYPE_CHECKING:
    from mcp.client import ClientSession
```
**EN:** This top-level conditional toggles behavior based on runtime configuration or platform state.
**CN:** 该顶层条件分支会根据运行时配置或平台状态切换行为。

### Lines 52-61 — Module constants
```python
logger = logging.getLogger(__name__)

# This is currently needed as the tool type doesn't 1:1 match the
# tool namespace, which is what is used to look up the
# connection to the tool server
_TOOL_NAME_TO_TYPE_MAP = {
    "browser": "web_search_preview",
    "python": "code_interpreter",
    "container": "container",
}
```
**EN:** This block initializes `logger`, `_TOOL_NAME_TO_TYPE_MAP`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`, `_TOOL_NAME_TO_TYPE_MAP`，为后续逻辑准备模块级常量或共享状态。

### Lines 64-71 — Function `_map_tool_name_to_tool_type`
```python
def _map_tool_name_to_tool_type(tool_name: str) -> str:
    if tool_name not in _TOOL_NAME_TO_TYPE_MAP:
        available_tools = ", ".join(_TOOL_NAME_TO_TYPE_MAP.keys())
        raise ValueError(
            f"Built-in tool name '{tool_name}' not defined in mapping. "
            f"Available tools: {available_tools}"
        )
    return _TOOL_NAME_TO_TYPE_MAP[tool_name]
```
**EN:** This function `_map_tool_name_to_tool_type` implements the `map tool name converts to tool type` step within the module flow.
**CN:** 该函数 `_map_tool_name_to_tool_type` 实现了模块流程中的“maptoolname转换为tooltype”步骤。

### Lines 74-103 — Class `TurnMetrics`
```python
class TurnMetrics:
    """Tracks token and toolcall details for a single conversation turn."""

    def __init__(
        self,
        input_tokens: int = 0,
        output_tokens: int = 0,
        cached_input_tokens: int = 0,
        tool_output_tokens: int = 0,
    ) -> None:
        self.input_tokens = input_tokens
        self.output_tokens = output_tokens
        self.cached_input_tokens = cached_input_tokens
        self.tool_output_tokens = tool_output_tokens

    def reset(self) -> None:
        """Reset counters for a new turn."""
        self.input_tokens = 0
        self.output_tokens = 0
        self.cached_input_tokens = 0
        self.tool_output_tokens = 0

    def copy(self) -> "TurnMetrics":
        """Create a copy of this turn's token counts."""
        return TurnMetrics(
            self.input_tokens,
            self.output_tokens,
            self.cached_input_tokens,
            self.tool_output_tokens,
        )
```
**EN:** Class `TurnMetrics` is introduced here. Its docstring describes the intent as: Tracks token and toolcall details for a single conversation turn.
**CN:** 这里定义类 `TurnMetrics`。其文档字符串说明了该类的职责与使用方式。

### Lines 106-139 — Class `ConversationContext`
```python
class ConversationContext(ABC):
    @abstractmethod
    def append_output(self, output: RequestOutput) -> None:
        pass

    @abstractmethod
    def append_tool_output(self, output) -> None:
        pass

    @abstractmethod
    async def call_tool(self) -> list[Message]:
        pass

    @abstractmethod
    def need_builtin_tool_call(self) -> bool:
        pass

    @abstractmethod
    def render_for_completion(self) -> list[int]:
        pass

    @abstractmethod
    async def init_tool_sessions(
        self,
        tool_server: ToolServer | None,
        exit_stack: AsyncExitStack,
        request_id: str,
        mcp_tools: dict[str, Mcp],
    ) -> None:
        pass

    @abstractmethod
    async def cleanup_session(self) -> None:
        raise NotImplementedError("Should not be called.")
```
**EN:** Class `ConversationContext` is defined here, extending `ABC`, and groups behavior through methods like `append_output`, `append_tool_output`, `call_tool`, `need_builtin_tool_call`.
**CN:** 这里定义类 `ConversationContext`，其职责是封装相关状态与行为，并通过 `append_output`、`append_tool_output`、`call_tool`、`need_builtin_tool_call` 等方法组织逻辑。

### Lines 142-161 — Function `_create_json_parse_error_messages`
```python
def _create_json_parse_error_messages(
    last_msg: Message, e: json.JSONDecodeError
) -> list[Message]:
    """
    Creates an error message when json parse failed.
    """
    error_msg = (
        f"Error parsing tool arguments as JSON: {str(e)}. "
        "Please ensure the tool call arguments are valid JSON and try again."
    )
    content = TextContent(text=error_msg)
    author = Author(role=Role.TOOL, name=last_msg.recipient)
    return [
        Message(
            author=author,
            content=[content],
            recipient=Role.ASSISTANT,
            channel=last_msg.channel,
        )
    ]
```
**EN:** This function `_create_json_parse_error_messages` is documented as: Creates an error message when json parse failed.
**CN:** 这里定义函数 `_create_json_parse_error_messages`，其文档字符串说明了主要职责与调用约定。

### Lines 164-266 — Class `SimpleContext`
```python
class SimpleContext(ConversationContext):
    """This is a context that cannot handle MCP tool calls"""

    def __init__(self):
        self.last_output = None

        # Accumulated final output for streaming mode
        self._accumulated_text: str = ""
        self._accumulated_token_ids: list[int] = []
        self._accumulated_logprobs: list = []

        self.num_prompt_tokens = 0
        self.num_output_tokens = 0
        self.num_cached_tokens = 0
        # todo num_reasoning_tokens is not implemented yet.
        self.num_reasoning_tokens = 0
        # not implemented yet for SimpleContext
        self.all_turn_metrics = []

        self.input_messages: list[ResponseRawMessageAndToken] = []
        self.kv_transfer_params: dict[str, Any] | None = None

    def append_output(self, output) -> None:
        self.last_output = output
        if not isinstance(output, RequestOutput):
            raise ValueError("SimpleContext only supports RequestOutput.")
        self.num_prompt_tokens = len(output.prompt_token_ids or [])
        self.num_cached_tokens = output.num_cached_tokens or 0
...
        exit_stack: AsyncExitStack,
        request_id: str,
        mcp_tools: dict[str, Mcp],
    ) -> None:
        pass

    async def cleanup_session(self) -> None:
        raise NotImplementedError("Should not be called.")
```
**EN:** Class `SimpleContext` is introduced here. Its docstring describes the intent as: This is a context that cannot handle MCP tool calls
**CN:** 这里定义类 `SimpleContext`。其文档字符串说明了该类的职责与使用方式。

### Lines 269-519 — Class `ParsableContext`
```python
class ParsableContext(ConversationContext):
    def __init__(
        self,
        *,
        response_messages: list[ResponseInputOutputItem],
        tokenizer: TokenizerLike,
        reasoning_parser_cls: type[ReasoningParser] | None,
        request: ResponsesRequest,
        available_tools: list[str] | None,
        tool_parser_cls: type[ToolParser] | None,
        chat_template: str | None,
        chat_template_content_format: ChatTemplateContentFormatOption,
    ):
        self.num_prompt_tokens = 0
        self.num_output_tokens = 0
        self.num_cached_tokens = 0
        self.num_reasoning_tokens = 0
        # not implemented yet for ParsableContext
        self.all_turn_metrics: list[TurnMetrics] = []

        if reasoning_parser_cls is None:
            raise ValueError("reasoning_parser_cls must be provided.")

        self.parser = get_responses_parser_for_simple_context(
            tokenizer=tokenizer,
            reasoning_parser_cls=reasoning_parser_cls,
            response_messages=response_messages,
            request=request,
...
                    await tool_session.call_tool("cleanup_session", {})

        await asyncio.gather(
            *(
                cleanup_tool_session(self._tool_sessions[tool])
                for tool in self.called_tools
            )
        )
```
**EN:** Class `ParsableContext` is defined here, extending `ConversationContext`, and groups behavior through methods like `__init__`, `append_output`, `append_tool_output`, `need_builtin_tool_call`.
**CN:** 这里定义类 `ParsableContext`，其职责是封装相关状态与行为，并通过 `__init__`、`append_output`、`append_tool_output`、`need_builtin_tool_call` 等方法组织逻辑。

### Lines 522-847 — Class `HarmonyContext`
```python
class HarmonyContext(ConversationContext):
    def __init__(
        self,
        messages: list,
        available_tools: list[str],
        function_tool_names: frozenset[str] | None = None,
    ):
        self._messages = messages
        self.finish_reason: str | None = None
        self.available_tools = available_tools
        self.function_tool_names = function_tool_names
        self._tool_sessions: dict[str, ClientSession | Tool] = {}
        self.called_tools: set[str] = set()

        self.parser = get_streamable_parser_for_assistant()
        self.num_init_messages = len(messages)
        self.num_prompt_tokens = 0
        self.num_output_tokens = 0
        self.num_cached_tokens = 0
        self.num_reasoning_tokens = 0
        self.num_tool_output_tokens = 0

        # Turn tracking - replaces multiple individual tracking variables
        self.current_turn_metrics = TurnMetrics()
        # Track metrics for all turns
        self.all_turn_metrics: list[TurnMetrics] = []
        self.is_first_turn = True
        self.first_tok_of_message = True  # For streaming support
...
                    await tool_session.call_tool("cleanup_session", {})

        await asyncio.gather(
            *(
                cleanup_tool_session(self._tool_sessions[tool])
                for tool in self.called_tools
            )
        )
```
**EN:** Class `HarmonyContext` is defined here, extending `ConversationContext`, and groups behavior through methods like `__init__`, `_update_num_reasoning_tokens`, `append_output`, `append_tool_output`.
**CN:** 这里定义类 `HarmonyContext`，其职责是封装相关状态与行为，并通过 `__init__`、`_update_num_reasoning_tokens`、`append_output`、`append_tool_output` 等方法组织逻辑。

### Lines 850-932 — Class `StreamingHarmonyContext`
```python
class StreamingHarmonyContext(HarmonyContext):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.last_output = None

        self.parser = get_streamable_parser_for_assistant()
        self.encoding = get_encoding()
        self.last_tok = None
        self.first_tok_of_message = True
        self.last_content_delta = None

    @property
    def messages(self) -> list:
        return self._messages

    def append_output(self, output: RequestOutput) -> None:
        # append_output is called for each output token in streaming case,
        # so we only want to add the prompt tokens once for each message.
        self.last_content_delta = None
        if self.first_tok_of_message:
            self._update_prefill_token_usage(output)
        # Reset self.first_tok_of_message if needed:
        # if the current token is the last one of the current message
        # (finished=True), then the next token processed will mark the
        # beginning of a new message
        self.first_tok_of_message = output.finished
        last_delta_text = ""
        for tok in output.outputs[0].token_ids:
...
        to_process = []
        while rendered_tokens[last_n] != self.last_tok:
            to_process.append(rendered_tokens[last_n])
            last_n -= 1
        for tok in reversed(to_process):
            self.parser.process(tok)

        return rendered_tokens
```
**EN:** Class `StreamingHarmonyContext` is defined here, extending `HarmonyContext`, and groups behavior through methods like `__init__`, `messages`, `append_output`, `append_tool_output`.
**CN:** 这里定义类 `StreamingHarmonyContext`，其职责是封装相关状态与行为，并通过 `__init__`、`messages`、`append_output`、`append_tool_output` 等方法组织逻辑。

## Key Concepts / 关键概念
- Async request handling / 异步请求处理
- Metrics and observability / 指标与可观测性
- Tokenization or token-level processing / 分词或 token 级处理
- Prompt or multimodal rendering / 提示词或多模态渲染
- MCP tool integration / MCP 工具集成
- OpenAI-compatible API surface / OpenAI 兼容 API 接口面

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `asyncio`, `contextlib`, `copy`, `json`, `logging`, `abc`, `dataclasses`, `typing`
- **Third-party / 第三方**: `openai`, `openai_harmony`, `mcp`
- **vLLM Internal / vLLM 内部**: `vllm`, `vllm.entrypoints.chat_utils`, `vllm.entrypoints.constants`, `vllm.entrypoints.mcp.tool`, `vllm.entrypoints.mcp.tool_server`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.parser.harmony_utils`, `vllm.entrypoints.openai.parser.responses_parser`, `vllm.entrypoints.openai.responses.protocol`, `vllm.entrypoints.openai.responses.utils`, `vllm.outputs`, `vllm.reasoning.abs_reasoning_parsers`
