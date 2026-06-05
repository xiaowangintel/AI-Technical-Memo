# serving.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/openai/responses/serving.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the serving layer that coordinates preprocessing, engine calls, and response shaping. Scope: OpenAI-compatible responses. / 实现服务层，负责预处理、引擎调用与响应整形。 作用域：OpenAI 兼容 / 响应。

## Line-by-Line Analysis / 逐行分析
### Lines 4-109 — Imports and shared dependencies
```python
import asyncio
import time
from collections import deque
from collections.abc import AsyncGenerator, AsyncIterator, Callable, Mapping, Sequence
from contextlib import AsyncExitStack
from copy import copy
from http import HTTPStatus
from typing import Any, Final

from fastapi import Request
from openai.types.responses import (
    ResponseFunctionToolCall,
    ResponseOutputItem,
    ResponseOutputMessage,
    ResponseOutputText,
    ResponseStatus,
    response_text_delta_event,
)
from openai.types.responses.response_output_text import Logprob, LogprobTopLogprob
from openai.types.responses.tool import Mcp, Tool
from openai_harmony import Message as OpenAIHarmonyMessage
from pydantic import TypeAdapter

from vllm import envs
from vllm.config.utils import replace
from vllm.engine.protocol import EngineClient
...
from vllm.lora.request import LoRARequest
from vllm.outputs import CompletionOutput
from vllm.parser import ParserManager
from vllm.sampling_params import SamplingParams, StructuredOutputsParams
from vllm.tokenizers import TokenizerLike
from vllm.tool_parsers import ToolParser
from vllm.utils import random_uuid
from vllm.utils.collection_utils import as_list
```
**EN:** This import block pulls in standard-library modules such as `asyncio`, `time`, `collections`, `contextlib`, `copy`, `http`, uses third-party packages like `fastapi`, `openai`, `openai_harmony`, `pydantic`, depends on internal helpers such as `vllm`, `vllm.config.utils`, `vllm.engine.protocol`, `vllm.entrypoints.chat_utils`, `vllm.entrypoints.logger`, `vllm.entrypoints.mcp.tool_server`.
**CN:** 该导入块引入 `asyncio`, `time`, `collections`, `contextlib`, `copy`, `http` 等标准库模块，使用 `fastapi`, `openai`, `openai_harmony`, `pydantic` 等第三方库，依赖 `vllm`, `vllm.config.utils`, `vllm.engine.protocol`, `vllm.entrypoints.chat_utils`, `vllm.entrypoints.logger`, `vllm.entrypoints.mcp.tool_server` 等 vLLM 内部模块。

### Lines 111-111 — Shared module state
```python
logger = init_logger(__name__)
```
**EN:** This block initializes `logger`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`，为后续逻辑准备模块级常量或共享状态。

### Lines 114-150 — Function `_extract_allowed_tools_from_mcp_requests`
```python
def _extract_allowed_tools_from_mcp_requests(
    tools: list[Tool],
) -> dict[str, list[str] | None]:
    """
    Extract allowed_tools mapping from MCP tool requests.

    Returns a dictionary mapping server_label to allowed_tools list.
    Handles both list format and McpAllowedToolsMcpToolFilter object format.

    Special handling:
    - If allowed_tools is None, returns None (allows all tools)
    - If allowed_tools contains "*", returns None (allows all tools)
    - Otherwise, returns the list of specific tool names

    This function can be reused for both harmony and non-harmony MCP calls.
    """
    allowed_tools_map: dict[str, list[str] | None] = {}
    for tool in tools:
        if not isinstance(tool, Mcp):
            continue

        # allowed_tools can be a list or an object with tool_names
        # Extract the actual list of tool names
        allowed_tools_val = None
        if tool.allowed_tools is not None:
            if isinstance(tool.allowed_tools, list):
                allowed_tools_val = tool.allowed_tools
            elif hasattr(tool.allowed_tools, "tool_names"):
                # It's an McpAllowedToolsMcpToolFilter object
                allowed_tools_val = tool.allowed_tools.tool_names

        # Normalize "*" to None (both mean "allow all tools")
        if allowed_tools_val is not None and "*" in allowed_tools_val:
            allowed_tools_val = None

        allowed_tools_map[tool.server_label] = allowed_tools_val
    return allowed_tools_map
```
**EN:** This function `_extract_allowed_tools_from_mcp_requests` is documented as: Extract allowed_tools mapping from MCP tool requests.
**CN:** 这里定义函数 `_extract_allowed_tools_from_mcp_requests`，其文档字符串说明了主要职责与调用约定。

### Lines 153-1576 — Class `OpenAIServingResponses`
```python
class OpenAIServingResponses(OpenAIServing):
    def __init__(
        self,
        engine_client: EngineClient,
        models: OpenAIServingModels,
        openai_serving_render: OpenAIServingRender,
        *,
        request_logger: RequestLogger | None,
        chat_template: str | None,
        chat_template_content_format: ChatTemplateContentFormatOption,
        return_tokens_as_token_ids: bool = False,
        reasoning_parser: str = "",
        enable_auto_tools: bool = False,
        tool_parser: str | None = None,
        tool_server: ToolServer | None = None,
        enable_prompt_tokens_details: bool = False,
        enable_force_include_usage: bool = False,
        enable_log_outputs: bool = False,
        default_chat_template_kwargs: dict[str, Any] | None = None,
    ) -> None:
        super().__init__(
            engine_client=engine_client,
            models=models,
            request_logger=request_logger,
            return_tokens_as_token_ids=return_tokens_as_token_ids,
        )

        self.openai_serving_render = openai_serving_render
...
            )
            yield _increment_sequence_number_and_return(
                ResponseCompletedEvent(
                    type="response.completed",
                    sequence_number=-1,
                    response=final_response,
                )
            )
```
**EN:** Class `OpenAIServingResponses` is defined here, extending `OpenAIServing`, and groups behavior through methods like `__init__`, `_effective_chat_template_kwargs`, `_validate_generator_input`, `_validate_create_responses_input`.
**CN:** 这里定义类 `OpenAIServingResponses`，其职责是封装相关状态与行为，并通过 `__init__`、`_effective_chat_template_kwargs`、`_validate_generator_input`、`_validate_create_responses_input` 等方法组织逻辑。

## Key Concepts / 关键概念
- FastAPI routing and application assembly / FastAPI 路由与应用组装
- Async request handling / 异步请求处理
- Streaming responses / 流式响应
- Incremental result emission / 增量结果输出
- Schema validation with Pydantic / 使用 Pydantic 进行模式校验
- Engine-backed serving orchestration / 基于引擎的服务编排
- Metrics and observability / 指标与可观测性
- Tokenization or token-level processing / 分词或 token 级处理

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `asyncio`, `time`, `collections`, `contextlib`, `copy`, `http`, `typing`
- **Third-party / 第三方**: `fastapi`, `openai`, `openai_harmony`, `pydantic`
- **vLLM Internal / vLLM 内部**: `vllm`, `vllm.config.utils`, `vllm.engine.protocol`, `vllm.entrypoints.chat_utils`, `vllm.entrypoints.logger`, `vllm.entrypoints.mcp.tool_server`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.openai.engine.serving`, `vllm.entrypoints.openai.models.serving`, `vllm.entrypoints.openai.parser.harmony_utils`, `vllm.entrypoints.openai.responses.context`, `vllm.entrypoints.openai.responses.harmony`
