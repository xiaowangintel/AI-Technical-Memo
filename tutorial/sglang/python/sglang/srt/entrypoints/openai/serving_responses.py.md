# serving_responses.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/openai/serving_responses.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements serving responses logic for the OpenAI-compatible serving layer. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 serving responses 相关的逻辑，并服务于 OpenAI 兼容服务层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-3: Provide supporting module logic / 提供辅助模块逻辑
```python
# SPDX-License-Identifier: Apache-2.0
# Adapted from vLLM's OpenAIServingResponses
"""Handler for /v1/responses requests"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 5-5: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 7-14: Import runtime dependencies / 导入运行时依赖
```python
import asyncio
import copy
import json
import logging
import time
from contextlib import AsyncExitStack
from http import HTTPStatus
from typing import TYPE_CHECKING, Any, AsyncGenerator, AsyncIterator, Optional, Union
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 16-30: Provide supporting module logic / 提供辅助模块逻辑
```python
import jinja2
import openai.types.responses as openai_responses_types
import orjson
from fastapi import Request
from fastapi.responses import ORJSONResponse
from openai.types.responses import (
    ResponseOutputMessage,
    ResponseOutputText,
    ResponseReasoningItem,
)
from openai.types.responses.response_function_tool_call import ResponseFunctionToolCall
from openai.types.responses.response_reasoning_item import (
    Content as ResponseReasoningTextContent,
)
from openai_harmony import Message as OpenAIMessage
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 32-61: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.entrypoints.context import (
    ConversationContext,
    HarmonyContext,
    SimpleContext,
    StreamingHarmonyContext,
)
from sglang.srt.entrypoints.harmony_utils import (
    get_developer_message,
    get_stop_tokens_for_assistant_actions,
    get_system_message,
    get_user_message,
    parse_output_message,
    parse_remaining_state,
    parse_response_input,
    render_for_completion,
)
from sglang.srt.entrypoints.openai.protocol import (
    ChatCompletionMessageParam,
    ChatCompletionRequest,
    PromptTokenUsageInfo,
    RequestResponseMetadata,
    ResponsesRequest,
    ResponsesResponse,
    UsageInfo,
)
from sglang.srt.entrypoints.openai.serving_chat import OpenAIServingChat
from sglang.srt.entrypoints.openai.tool_server import MCPToolServer, ToolServer
from sglang.srt.managers.io_struct import GenerateReqInput
from sglang.srt.parser.reasoning_parser import ReasoningParser
from sglang.srt.utils import random_uuid
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 63-65: Provide supporting module logic / 提供辅助模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.managers.template_manager import TemplateManager
    from sglang.srt.managers.tokenizer_manager import TokenizerManager
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 67-67: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 70-71: Provide supporting module logic / 提供辅助模块逻辑
```python
class OpenAIServingResponses(OpenAIServingChat):
    """Handler for /v1/responses requests"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 73-108: Initialize OpenAIServingResponses / 初始化 OpenAIServingResponses
```python
    def __init__(
        self,
        tokenizer_manager: TokenizerManager,
        template_manager: TemplateManager,
        *,
        enable_prompt_tokens_details: bool = False,
        tool_server: Optional[ToolServer] = None,
    ) -> None:
        super().__init__(tokenizer_manager, template_manager)

        # template_manager is already set by parent class
        self.reasoning_parser = self.tokenizer_manager.server_args.reasoning_parser
        self.enable_prompt_tokens_details = enable_prompt_tokens_details

        # Get default sampling params from model config if available
        self.default_sampling_params = {}

        self.supports_browsing = (
            tool_server.has_tool("browser") if tool_server else False
        )
        self.supports_code_interpreter = (
            tool_server.has_tool("python") if tool_server else False
        )
        self.tool_server = tool_server
        # Get from model config
        self.use_harmony = (
            self.tokenizer_manager.model_config.hf_config.model_type == "gpt_oss"
        )

        if self.use_harmony:
            # OpenAI models have two EOS-like tokens: <|return|> and <|call|>.
            # We need to add them to the stop token ids.
            if "stop_token_ids" not in self.default_sampling_params:
                self.default_sampling_params["stop_token_ids"] = []
            self.default_sampling_params["stop_token_ids"].extend(
                get_stop_tokens_for_assistant_actions()
```
**EN:** This block implements the initializer `__init__(tokenizer_manager, template_manager, enable_prompt_tokens_details, tool_server)` for `OpenAIServingResponses`. It prepares the object state and connects the instance to the surrounding serving responses workflow.
**CN:** 该代码块实现 `OpenAIServingResponses` 的初始化方法 `__init__(tokenizer_manager, template_manager, enable_prompt_tokens_details, tool_server)`。它负责准备对象状态，并把实例接入 serving responses 相关的运行流程。

### Lines 109-124: Initialize OpenAIServingResponses (continued) / 初始化 OpenAIServingResponses（续）
```python
            )

        # Response storage for background and retrieval operations
        # Note: In production, this should use a proper storage backend (Redis, database)
        # with TTL/expiration to prevent memory leaks
        self.response_store: dict[str, ResponsesResponse] = {}
        self.response_store_lock = asyncio.Lock()

        # Message storage for conversation continuity
        # Note: In production, this should use a proper storage backend (Redis, database)
        # with TTL/expiration to prevent memory leaks
        self.msg_store: dict[
            str, Union[list[ChatCompletionMessageParam], list["OpenAIMessage"]]
        ] = {}

        self.background_tasks: dict[str, asyncio.Task] = {}
```
**EN:** This block implements the initializer `__init__(tokenizer_manager, template_manager, enable_prompt_tokens_details, tool_server)` for `OpenAIServingResponses`. It prepares the object state and connects the instance to the surrounding serving responses workflow.
**CN:** 该代码块实现 `OpenAIServingResponses` 的初始化方法 `__init__(tokenizer_manager, template_manager, enable_prompt_tokens_details, tool_server)`。它负责准备对象状态，并把实例接入 serving responses 相关的运行流程。

### Lines 125-126: Import runtime dependencies / 导入运行时依赖
```python

    # error helpers dedicated for v1/responses
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 127-140: Implement create error response / 实现create error 响应
```python
    def create_error_response(
        self,
        message: str,
        err_type: str = "invalid_request_error",
        status_code: int = 400,
        param: Optional[str] = None,
    ) -> ORJSONResponse:
        nested_error = {
            "message": message,
            "type": err_type,
            "param": param,
            "code": status_code,
        }
        return ORJSONResponse(content={"error": nested_error}, status_code=status_code)
```
**EN:** This block implements the method `create_error_response(message, err_type, status_code, param)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `create_error_response`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的方法 `create_error_response(message, err_type, status_code, param)`。它围绕 `create_error_response` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 142-157: Implement create streaming error response / 实现create streaming error 响应
```python
    def create_streaming_error_response(
        self,
        message: str,
        err_type: str = "BadRequestError",
        status_code: int = 400,
    ) -> str:
        return json.dumps(
            {
                "error": {
                    "message": message,
                    "type": err_type,
                    "param": None,
                    "code": status_code,
                }
            }
        )
```
**EN:** This block implements the method `create_streaming_error_response(message, err_type, status_code)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `create_streaming_error_response`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的方法 `create_streaming_error_response(message, err_type, status_code)`。它围绕 `create_streaming_error_response` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 159-160: Implement request id prefix / 实现请求 id prefix
```python
    def _request_id_prefix(self) -> str:
        return "resp_"
```
**EN:** This block implements the method `_request_id_prefix()` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `_request_id_prefix`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的方法 `_request_id_prefix()`。它围绕 `_request_id_prefix` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 162-197: Implement async create responses / 实现异步create responses
```python
    async def create_responses(
        self,
        request: ResponsesRequest,
        raw_request: Optional[Request] = None,
    ) -> Union[AsyncGenerator[str, None], ResponsesResponse, ORJSONResponse]:
        # Validate model
        if not self.tokenizer_manager:
            return self.create_error_response("Model not loaded")

        # FIXME: If the engine is dead, raise an error
        # This is required for the streaming case

        # Handle the previous response ID
        prev_response_id = request.previous_response_id
        if prev_response_id is not None:
            if not prev_response_id.startswith("resp_"):
                return self._make_invalid_id_error(prev_response_id)
            async with self.response_store_lock:
                prev_response = self.response_store.get(prev_response_id)
            if prev_response is None:
                return self._make_not_found_error(prev_response_id)
        else:
            prev_response = None

        try:
            model_name = request.model
            tokenizer = self.tokenizer_manager.tokenizer

            if self.use_harmony:
                messages, request_prompts, engine_prompts = (
                    self._make_request_with_harmony(request, prev_response)
                )
            else:
                messages, request_prompts, engine_prompts = await self._make_request(
                    request, prev_response, tokenizer
                )
```
**EN:** This block implements the async method `create_responses(request, raw_request)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `create_responses`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `create_responses(request, raw_request)`。它围绕 `create_responses` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 198-233: Continue async create responses / 继续说明异步create responses
```python

        except (ValueError, TypeError, RuntimeError, jinja2.TemplateError) as e:
            logger.exception("Error in preprocessing prompt inputs")
            return self.create_error_response(f"{e} {e.__cause__}")

        request_metadata = RequestResponseMetadata(request_id=request.request_id)
        if raw_request:
            raw_request.state.request_metadata = request_metadata

        if (
            self.tool_server is not None
            and isinstance(self.tool_server, MCPToolServer)
            and (request.background or request.stream)
            and request.tools
            and any(
                tool.type in ["web_search_preview", "code_interpreter"]
                for tool in request.tools
            )
        ):
            return self.create_error_response(
                "MCP tool server is not supported in background mode and "
                "streaming mode"
            )

        # Schedule the request and get the result generator
        generators: list[AsyncGenerator[Any, None]] = []
        tool_list = []
        if self.use_harmony:
            if self.supports_browsing:
                tool_list.append("browser")
            if self.supports_code_interpreter:
                tool_list.append("python")
        async with AsyncExitStack() as exit_stack:
            try:
                if self.tool_server is not None:
                    tool_session_ctxs: dict[str, Any] = {
```
**EN:** This block implements the async method `create_responses(request, raw_request)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `create_responses`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `create_responses(request, raw_request)`。它围绕 `create_responses` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 234-269: Continue async create responses / 继续说明异步create responses
```python
                        tool_name: exit_stack.enter_async_context(
                            self.tool_server.get_tool_session(tool_name)
                        )
                        for tool_name in tool_list
                    }
                    tool_sessions = {}
                    for tool_name in tool_list:
                        tool_sessions[tool_name] = await tool_session_ctxs[tool_name]
                else:
                    assert len(tool_list) == 0
                    tool_sessions = {}
                for i, engine_prompt in enumerate(engine_prompts):
                    # Calculate default max tokens from context length minus prompt length
                    if hasattr(engine_prompt, "__len__"):
                        prompt_length = len(engine_prompt)
                    elif isinstance(engine_prompt, list):
                        prompt_length = len(engine_prompt)
                    else:
                        prompt_length = 0

                    context_len = (
                        self.tokenizer_manager.model_config.context_len
                        if hasattr(self.tokenizer_manager.model_config, "context_len")
                        else 4096
                    )
                    # Account for reserved tokens (e.g., EAGLE speculative decoding slots)
                    # that the tokenizer_manager adds during validation
                    num_reserved_tokens = self.tokenizer_manager.num_reserved_tokens
                    default_max_tokens = max(
                        context_len - prompt_length - num_reserved_tokens, 512
                    )  # Ensure minimum 512 tokens
                    sampling_params = request.to_sampling_params(
                        default_max_tokens, self.default_sampling_params
                    )

                    context: ConversationContext
```
**EN:** This block implements the async method `create_responses(request, raw_request)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `create_responses`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `create_responses(request, raw_request)`。它围绕 `create_responses` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 270-305: Continue async create responses / 继续说明异步create responses
```python
                    if self.use_harmony:
                        if request.stream:
                            context = StreamingHarmonyContext(messages, tool_sessions)
                        else:
                            context = HarmonyContext(messages, tool_sessions)
                    else:
                        context = SimpleContext()

                    # Create GenerateReqInput for SGLang
                    adapted_request = GenerateReqInput(
                        input_ids=engine_prompt,
                        sampling_params=sampling_params,
                        stream=request.stream,
                        rid=request.request_id,
                        extra_key=self._compute_extra_key(request),
                        background=request.background,
                    )

                    generator = self._generate_with_builtin_tools(
                        request.request_id,
                        request_prompts[i],
                        adapted_request,
                        sampling_params,
                        context,
                        raw_request=raw_request,
                        priority=request.priority,
                    )
                    generators.append(generator)
            except ValueError as e:
                return self.create_error_response(str(e))

            assert len(generators) == 1
            (result_generator,) = generators

            # Store the input messages
            if request.store:
```
**EN:** This block implements the async method `create_responses(request, raw_request)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `create_responses`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `create_responses(request, raw_request)`。它围绕 `create_responses` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 306-341: Continue async create responses / 继续说明异步create responses
```python
                self.msg_store[request.request_id] = messages

            if request.background:
                created_time = int(time.time())
                response = ResponsesResponse.from_request(
                    request,
                    sampling_params,
                    model_name=model_name,
                    created_time=created_time,
                    output=[],
                    status="queued",
                    usage=None,
                )
                async with self.response_store_lock:
                    self.response_store[response.id] = response

                # Run the request in the background
                task = asyncio.create_task(
                    self._run_background_request(
                        request,
                        sampling_params,
                        result_generator,
                        context,
                        model_name,
                        tokenizer,
                        request_metadata,
                        created_time,
                    ),
                    name=f"create_{response.id}",
                )

                # For cleanup
                self.background_tasks[response.id] = task
                task.add_done_callback(
                    lambda _: self.background_tasks.pop(response.id, None)
                )
```
**EN:** This block implements the async method `create_responses(request, raw_request)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `create_responses`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `create_responses(request, raw_request)`。它围绕 `create_responses` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 342-369: Continue async create responses / 继续说明异步create responses
```python
                return response

            if request.stream:
                return self.responses_stream_generator(
                    request,
                    sampling_params,
                    result_generator,
                    context,
                    model_name,
                    tokenizer,
                    request_metadata,
                )
            try:
                result: Union[ORJSONResponse, ResponsesResponse] = (
                    await self.responses_full_generator(
                        request,
                        sampling_params,
                        result_generator,
                        context,
                        model_name,
                        tokenizer,
                        request_metadata,
                    )
                )
                return result
            except Exception as e:
                return self.create_error_response(str(e))
        return self.create_error_response("Unknown error")
```
**EN:** This block implements the async method `create_responses(request, raw_request)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `create_responses`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `create_responses(request, raw_request)`。它围绕 `create_responses` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 371-413: Implement async make request / 实现异步make 请求
```python
    async def _make_request(
        self,
        request: ResponsesRequest,
        prev_response: Optional[ResponsesResponse],
        tokenizer: Any,
    ):
        # Construct the input messages
        messages = self._construct_input_messages(request, prev_response)

        # Follow SGLang's pattern: create a ChatCompletionRequest and process messages
        try:
            # Convert ResponsesRequest to ChatCompletionRequest for processing
            chat_request = ChatCompletionRequest(
                model=request.model,
                messages=messages,
                stream=request.stream,
            )

            # Follow SGLang's _process_messages pattern
            is_multimodal = self.tokenizer_manager.model_config.is_multimodal
            processed_messages = self._process_messages(chat_request, is_multimodal)

            # Extract the results
            if is_multimodal:
                request_prompts = [processed_messages.prompt]
                engine_prompts = [processed_messages.prompt]
            else:
                request_prompts = [processed_messages.prompt_ids]
                engine_prompts = [processed_messages.prompt_ids]

        except Exception as e:
            logger.warning(f"Chat processing failed, using fallback: {e}")
            # Fallback to simple encoding
            prompt_text = ""
            for msg in messages:
                role = msg.get("role", "user")
                content = msg.get("content", "")
                prompt_text += f"{role}: {content}\n"
            prompt_ids = tokenizer.encode(prompt_text)
            request_prompts = [prompt_ids]
            engine_prompts = [prompt_ids]

        return messages, request_prompts, engine_prompts
```
**EN:** This block implements the async method `_make_request(request, prev_response, tokenizer)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `_make_request`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `_make_request(request, prev_response, tokenizer)`。它围绕 `_make_request` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 415-427: Implement make request with harmony / 实现make 请求 with harmony
```python
    def _make_request_with_harmony(
        self,
        request: ResponsesRequest,
        prev_response: Optional[ResponsesResponse],
    ):
        if request.tool_choice != "auto":
            raise NotImplementedError(
                "Only 'auto' tool_choice is supported in " "response API"
            )
        messages = self._construct_input_messages_with_harmony(request, prev_response)
        prompt_token_ids = render_for_completion(messages)
        engine_prompt = prompt_token_ids
        return messages, [prompt_token_ids], [engine_prompt]
```
**EN:** This block implements the method `_make_request_with_harmony(request, prev_response)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `_make_request_with_harmony`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的方法 `_make_request_with_harmony(request, prev_response)`。它围绕 `_make_request_with_harmony` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 429-464: Implement async responses full generator / 实现异步responses full generator
```python
    async def responses_full_generator(
        self,
        request: ResponsesRequest,
        sampling_params: Any,
        result_generator: AsyncIterator[Any],
        context: ConversationContext,
        model_name: str,
        tokenizer: Any,
        request_metadata: RequestResponseMetadata,
        created_time: Optional[int] = None,
    ) -> Union[ResponsesResponse, ORJSONResponse]:
        if created_time is None:
            created_time = int(time.time())

        try:
            async for _ in result_generator:
                pass
        except asyncio.CancelledError:
            return self.create_error_response("Client disconnected")
        except ValueError as e:
            return self.create_error_response(str(e))

        if self.use_harmony:
            assert isinstance(context, HarmonyContext)
            output = self._make_response_output_items_with_harmony(context)
            # TODO: these are all 0 for now!
            num_prompt_tokens = context.num_prompt_tokens
            num_generated_tokens = context.num_output_tokens
            num_cached_tokens = context.num_cached_tokens
            num_reasoning_tokens = context.num_reasoning_tokens
        else:
            assert isinstance(context, SimpleContext)
            final_res = context.last_output
            assert final_res is not None

            output = self._make_response_output_items(
```
**EN:** This block implements the async method `responses_full_generator(request, sampling_params, result_generator, context, model_name, ...)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `responses_full_generator`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `responses_full_generator(request, sampling_params, result_generator, context, model_name, ...)`。它围绕 `responses_full_generator` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 465-500: Continue async responses full generator / 继续说明异步responses full generator
```python
                request, final_res["text"], tokenizer
            )

            # Calculate usage from actual output
            if hasattr(final_res, "meta_info"):
                num_prompt_tokens = final_res.meta_info.get("prompt_tokens", 0)
                num_generated_tokens = final_res.meta_info.get("completion_tokens", 0)
                num_cached_tokens = final_res.meta_info.get("cached_tokens", 0)
            elif hasattr(final_res, "prompt_token_ids") and hasattr(
                final_res, "outputs"
            ):
                # Fallback calculation if meta_info not available
                num_prompt_tokens = (
                    len(final_res.prompt_token_ids) if final_res.prompt_token_ids else 0
                )
                num_generated_tokens = (
                    len(final_res.outputs[0].token_ids)
                    if final_res.outputs and final_res.outputs[0].token_ids
                    else 0
                )
                num_cached_tokens = getattr(final_res, "num_cached_tokens", 0)
                num_reasoning_tokens = 0
            else:
                # Final fallback
                num_prompt_tokens = 0
                num_generated_tokens = 0
                num_cached_tokens = 0
                num_reasoning_tokens = 0

        usage = UsageInfo(
            prompt_tokens=num_prompt_tokens,
            completion_tokens=num_generated_tokens,
            total_tokens=num_prompt_tokens + num_generated_tokens,
            reasoning_tokens=num_reasoning_tokens,
        )
        if self.enable_prompt_tokens_details and num_cached_tokens:
```
**EN:** This block implements the async method `responses_full_generator(request, sampling_params, result_generator, context, model_name, ...)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `responses_full_generator`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `responses_full_generator(request, sampling_params, result_generator, context, model_name, ...)`。它围绕 `responses_full_generator` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 501-523: Continue async responses full generator / 继续说明异步responses full generator
```python
            usage.prompt_tokens_details = PromptTokenUsageInfo(
                cached_tokens=num_cached_tokens
            )
        request_metadata.final_usage_info = usage

        response = ResponsesResponse.from_request(
            request,
            sampling_params,
            model_name=model_name,
            created_time=created_time,
            output=output,
            status="completed",
            usage=usage,
        )

        if request.store:
            async with self.response_store_lock:
                stored_response = self.response_store.get(response.id)
                # If the response is already cancelled, don't update it
                if stored_response is None or stored_response.status != "cancelled":
                    self.response_store[response.id] = response

        return response
```
**EN:** This block implements the async method `responses_full_generator(request, sampling_params, result_generator, context, model_name, ...)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `responses_full_generator`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `responses_full_generator(request, sampling_params, result_generator, context, model_name, ...)`。它围绕 `responses_full_generator` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 525-560: Implement make response output items / 实现make 响应 output items
```python
    def _make_response_output_items(
        self,
        request: ResponsesRequest,
        final_output: Any,
        tokenizer: Any,
    ):
        # Handle reasoning parsing if enabled
        if self.reasoning_parser:
            # Use standard reasoning parser (openai maps to T4Detector internally)
            reasoning_parser = ReasoningParser(
                model_type=self.reasoning_parser,
                stream_reasoning=False,
                request=request,
            )
            reasoning_content, content = reasoning_parser.parse_non_stream(final_output)
        else:
            reasoning_content = None
            content = final_output

        output_items = []
        if reasoning_content:
            reasoning_item = ResponseReasoningItem(
                id=f"rs_{random_uuid()}",
                type="reasoning",
                summary=[],
                content=[
                    ResponseReasoningTextContent(
                        type="reasoning_text", text=reasoning_content
                    ),
                ],
                status=None,
            )
            output_items.append(reasoning_item)
        if content:
            output_text = ResponseOutputText(
                text=content,
```
**EN:** This block implements the method `_make_response_output_items(request, final_output, tokenizer)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `_make_response_output_items`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的方法 `_make_response_output_items(request, final_output, tokenizer)`。它围绕 `_make_response_output_items` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 561-573: Continue make response output items / 继续说明make 响应 output items
```python
                annotations=[],  # TODO
                type="output_text",
                logprobs=None,  # TODO
            )
            message = ResponseOutputMessage(
                id=f"msg_{random_uuid()}",
                content=[output_text],
                role="assistant",
                status="completed",
                type="message",
            )
            output_items.append(message)
        return output_items
```
**EN:** This block implements the method `_make_response_output_items(request, final_output, tokenizer)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `_make_response_output_items`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的方法 `_make_response_output_items(request, final_output, tokenizer)`。它围绕 `_make_response_output_items` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 575-587: Implement make response output items with harmony / 实现make 响应 output items with harmony
```python
    def _make_response_output_items_with_harmony(
        self,
        context: HarmonyContext,
    ):
        output_items = []
        num_init_messages = context.num_init_messages
        for msg in context.messages[num_init_messages:]:
            output_items.extend(parse_output_message(msg))
        # Handle the generation stopped in the middle (if any).
        last_items = parse_remaining_state(context.parser)
        if last_items:
            output_items.extend(last_items)
        return output_items
```
**EN:** This block implements the method `_make_response_output_items_with_harmony(context)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `_make_response_output_items_with_harmony`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的方法 `_make_response_output_items_with_harmony(context)`。它围绕 `_make_response_output_items_with_harmony` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 589-628: Implement construct input messages / 实现construct input messages
```python
    def _construct_input_messages(
        self,
        request: ResponsesRequest,
        prev_response: Optional[ResponsesResponse] = None,
    ) -> list[ChatCompletionMessageParam]:
        messages: list[ChatCompletionMessageParam] = []
        if request.instructions:
            messages.append(
                {
                    "role": "system",
                    "content": request.instructions,
                }
            )

        # Prepend the conversation history
        if prev_response is not None:
            # Add the previous messages
            prev_msg = self.msg_store[prev_response.id]
            messages.extend(prev_msg)

            # Add the previous output
            for output_item in prev_response.output:
                # NOTE: We skip the reasoning output of the previous response
                if isinstance(output_item, ResponseReasoningItem):
                    continue
                for content in output_item.content:
                    messages.append(
                        {
                            "role": "system",
                            "content": request.instructions,
                        }
                    )

        # Append the new input
        # Responses API supports simple text inputs without chat format
        if isinstance(request.input, str):
            messages.append({"role": "user", "content": request.input})
        else:
            messages.extend(request.input)  # type: ignore
        return messages
```
**EN:** This block implements the method `_construct_input_messages(request, prev_response)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `_construct_input_messages`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的方法 `_construct_input_messages(request, prev_response)`。它围绕 `_construct_input_messages` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 630-665: Implement construct input messages with harmony / 实现construct input messages with harmony
```python
    def _construct_input_messages_with_harmony(
        self,
        request: ResponsesRequest,
        prev_response: Optional[ResponsesResponse],
    ) -> list["OpenAIMessage"]:
        messages: list["OpenAIMessage"] = []
        if prev_response is None:
            # New conversation.
            reasoning_effort = request.reasoning.effort if request.reasoning else None
            tool_types = [tool.type for tool in request.tools]
            enable_browser = (
                "web_search_preview" in tool_types and self.tool_server is not None
            )
            enable_code_interpreter = (
                "code_interpreter" in tool_types and self.tool_server is not None
            )
            sys_msg = get_system_message(
                reasoning_effort=reasoning_effort,
                browser_description=(
                    self.tool_server.get_tool_description("browser")
                    if self.tool_server and enable_browser
                    else None
                ),
                python_description=(
                    self.tool_server.get_tool_description("python")
                    if self.tool_server and enable_code_interpreter
                    else None
                ),
            )
            messages.append(sys_msg)
            dev_msg = get_developer_message(request.instructions, request.tools)
            messages.append(dev_msg)
        else:
            # Continue the previous conversation.
            # FIXME: Currently, request params like reasoning and
            # instructions are ignored.
```
**EN:** This block implements the method `_construct_input_messages_with_harmony(request, prev_response)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `_construct_input_messages_with_harmony`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的方法 `_construct_input_messages_with_harmony(request, prev_response)`。它围绕 `_construct_input_messages_with_harmony` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 666-701: Continue construct input messages with harmony / 继续说明construct input messages with harmony
```python
            prev_msgs = self.msg_store[prev_response.id]
            # Remove the previous chain-of-thoughts if there is a new "final"
            # message.
            if (
                len(prev_msgs) > 0
                and hasattr(prev_msgs[-1], "channel")
                and prev_msgs[-1].channel == "final"
            ):  # type: ignore[union-attr]
                prev_final_msg_idx = -1
                for i in range(len(prev_msgs) - 2, -1, -1):
                    if (
                        hasattr(prev_msgs[i], "channel")
                        and prev_msgs[i].channel == "final"
                    ):  # type: ignore[union-attr]
                        prev_final_msg_idx = i
                        break
                recent_turn_msgs = prev_msgs[prev_final_msg_idx + 1 :]
                del prev_msgs[prev_final_msg_idx + 1 :]
                for msg in recent_turn_msgs:
                    if (
                        hasattr(msg, "channel") and msg.channel != "analysis"
                    ):  # type: ignore[union-attr]
                        prev_msgs.append(msg)
            messages.extend(prev_msgs)
        # Append the new input.
        # Responses API supports simple text inputs without chat format.
        if isinstance(request.input, str):
            messages.append(get_user_message(request.input))
        else:
            if prev_response is not None:
                prev_outputs = copy(prev_response.output)
            else:
                prev_outputs = []
            for response_msg in request.input:
                messages.append(parse_response_input(response_msg, prev_outputs))
                if isinstance(response_msg, ResponseFunctionToolCall):
```
**EN:** This block implements the method `_construct_input_messages_with_harmony(request, prev_response)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `_construct_input_messages_with_harmony`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的方法 `_construct_input_messages_with_harmony(request, prev_response)`。它围绕 `_construct_input_messages_with_harmony` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 702-703: Continue construct input messages with harmony / 继续说明construct input messages with harmony
```python
                    prev_outputs.append(response_msg)
        return messages
```
**EN:** This block implements the method `_construct_input_messages_with_harmony(request, prev_response)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `_construct_input_messages_with_harmony`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的方法 `_construct_input_messages_with_harmony(request, prev_response)`。它围绕 `_construct_input_messages_with_harmony` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 705-748: Implement async run background request / 实现异步run background 请求
```python
    async def _run_background_request(
        self,
        request: ResponsesRequest,
        sampling_params: Any,
        result_generator: AsyncIterator[Any],
        context: ConversationContext,
        model_name: str,
        tokenizer: Any,
        request_metadata: RequestResponseMetadata,
        created_time: Optional[int] = None,
        *args,
        **kwargs,
    ):
        try:
            # Update the status to "in_progress"
            async with self.response_store_lock:
                stored_response = self.response_store.get(request.request_id)
                assert stored_response is not None
                stored_response.status = "in_progress"

            response = await self.responses_full_generator(
                request,
                sampling_params,
                result_generator,
                context,
                model_name,
                tokenizer,
                request_metadata,
                created_time,
                *args,
                **kwargs,
            )
        except Exception as e:
            logger.exception("Background request failed for %s", request.request_id)
            response = self.create_error_response(str(e))

        if isinstance(response, ORJSONResponse):
            # If the request has failed, update the status to "failed"
            response_id = request.request_id
            async with self.response_store_lock:
                stored_response = self.response_store.get(response_id)
                assert stored_response is not None
                if stored_response.status not in ("completed", "cancelled"):
                    stored_response.status = "failed"
```
**EN:** This block implements the async method `_run_background_request(request, sampling_params, result_generator, context, model_name, ...)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `_run_background_request`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `_run_background_request(request, sampling_params, result_generator, context, model_name, ...)`。它围绕 `_run_background_request` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 750-762: Implement async retrieve responses / 实现异步retrieve responses
```python
    async def retrieve_responses(
        self,
        response_id: str,
    ) -> Union[ResponsesResponse, ORJSONResponse]:
        if not response_id.startswith("resp_"):
            return self._make_invalid_id_error(response_id)

        async with self.response_store_lock:
            response = self.response_store.get(response_id)

        if response is None:
            return self._make_not_found_error(response_id)
        return response
```
**EN:** This block implements the async method `retrieve_responses(response_id)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `retrieve_responses`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `retrieve_responses(response_id)`。它围绕 `retrieve_responses` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 764-795: Implement async cancel responses / 实现异步cancel responses
```python
    async def cancel_responses(
        self,
        response_id: str,
    ) -> Union[ResponsesResponse, ORJSONResponse]:
        if not response_id.startswith("resp_"):
            return self._make_invalid_id_error(response_id)

        async with self.response_store_lock:
            response = self.response_store.get(response_id)
            if response is None:
                return self._make_not_found_error(response_id)

            prev_status = response.status
            if prev_status not in ("queued", "in_progress"):
                return self.create_error_response(
                    err_type="invalid_request_error",
                    message="Cannot cancel a synchronous response.",
                )

            # Update the status to "cancelled"
            response.status = "cancelled"

        # The response_id is the same as the rid used when submitting the request
        self.tokenizer_manager.abort_request(rid=response_id)

        if task := self.background_tasks.get(response_id):
            task.cancel()
            try:
                await task
            except asyncio.CancelledError:
                logger.exception("Background task for %s was cancelled", response_id)
        return response
```
**EN:** This block implements the async method `cancel_responses(response_id)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `cancel_responses`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `cancel_responses(response_id)`。它围绕 `cancel_responses` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 797-805: Implement make invalid id error / 实现make invalid id error
```python
    def _make_invalid_id_error(self, response_id: str):
        return self.create_error_response(
            message=(
                f"Invalid 'response_id': '{response_id}'. "
                "Expected an ID that begins with 'resp'."
            ),
            err_type="invalid_request_error",
            param="response_id",
        )
```
**EN:** This block implements the method `_make_invalid_id_error(response_id)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `_make_invalid_id_error`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的方法 `_make_invalid_id_error(response_id)`。它围绕 `_make_invalid_id_error` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 807-813: Implement make not found error / 实现make not found error
```python
    def _make_not_found_error(self, response_id: str):
        return self.create_error_response(
            message=f"Response with id '{response_id}' not found.",
            err_type="invalid_request_error",
            status_code=HTTPStatus.NOT_FOUND,
            param="response_id",
        )
```
**EN:** This block implements the method `_make_not_found_error(response_id)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `_make_not_found_error`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的方法 `_make_not_found_error(response_id)`。它围绕 `_make_not_found_error` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 815-850: Implement async responses stream generator / 实现异步responses 流式 generator
```python
    async def responses_stream_generator(
        self,
        request: ResponsesRequest,
        sampling_params: Any,
        result_generator: AsyncIterator[StreamingHarmonyContext],
        context: StreamingHarmonyContext,
        model_name: str,
        tokenizer: Any,
        request_metadata: RequestResponseMetadata,
        created_time: Optional[int] = None,
    ) -> AsyncGenerator[str, None]:
        # TODO:
        # 1. Handle disconnect

        created_time = created_time or int(time.time())

        sequence_number = 0

        def _send_event(event):
            nonlocal sequence_number
            # Set sequence_number if the event has this attribute
            if hasattr(event, "sequence_number"):
                event.sequence_number = sequence_number
            sequence_number += 1
            # Get event type from the event's type field if it exists
            event_type = getattr(event, "type", "unknown")
            return (
                f"event: {event_type}\n"
                f"data: {event.model_dump_json(indent=None)}\n\n"
            )

        current_content_index = 0
        current_output_index = 0
        current_item_id = f"item_{random_uuid()}"
        sent_output_item_added = False
```
**EN:** This block implements the async method `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `responses_stream_generator`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)`。它围绕 `responses_stream_generator` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 851-886: Continue async responses stream generator / 继续说明异步responses 流式 generator
```python
        initial_response = ResponsesResponse.from_request(
            request,
            sampling_params,
            model_name=model_name,
            created_time=created_time,
            output=[],
            status="in_progress",
            usage=None,
        ).model_dump()
        yield _send_event(
            openai_responses_types.ResponseCreatedEvent(
                type="response.created",
                sequence_number=-1,
                response=initial_response,
            )
        )
        yield _send_event(
            openai_responses_types.ResponseInProgressEvent(
                type="response.in_progress",
                sequence_number=-1,
                response=initial_response,
            )
        )

        async for ctx in result_generator:

            # Only process context objects that implement the `is_expecting_start()` method,
            # which indicates they support per-turn streaming (e.g., StreamingHarmonyContext).
            # Contexts without this method are skipped, as they do not represent a new turn
            # or are not compatible with per-turn handling in the /v1/responses endpoint.
            if not hasattr(ctx, "is_expecting_start"):
                continue

            if ctx.is_expecting_start():
                current_output_index += 1
                sent_output_item_added = False
```
**EN:** This block implements the async method `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `responses_stream_generator`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)`。它围绕 `responses_stream_generator` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 887-922: Continue async responses stream generator / 继续说明异步responses 流式 generator
```python

                if len(ctx.parser.messages) > 0:
                    previous_item = ctx.parser.messages[-1]
                    if previous_item.recipient is not None:
                        # Deal with tool call here
                        pass
                    elif previous_item.channel == "analysis":
                        reasoning_item = ResponseReasoningItem(
                            id=f"rs_{random_uuid()}",
                            type="reasoning",
                            summary=[],
                            content=[
                                ResponseReasoningTextContent(
                                    text=previous_item.content[0].text,
                                    type="reasoning_text",
                                ),
                            ],
                            status="completed",
                        )
                        yield _send_event(
                            openai_responses_types.ResponseReasoningTextDoneEvent(
                                type="response.reasoning_text.done",
                                item_id=current_item_id,
                                sequence_number=-1,
                                output_index=current_output_index,
                                content_index=current_content_index,
                                text=previous_item.content[0].text,
                            )
                        )
                        yield _send_event(
                            openai_responses_types.ResponseOutputItemDoneEvent(
                                type="response.output_item.done",
                                sequence_number=-1,
                                output_index=current_output_index,
                                item=reasoning_item,
                            )
```
**EN:** This block implements the async method `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `responses_stream_generator`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)`。它围绕 `responses_stream_generator` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 923-958: Continue async responses stream generator / 继续说明异步responses 流式 generator
```python
                        )
                    elif previous_item.channel == "final":
                        text_content = openai_responses_types.ResponseOutputText(
                            type="output_text",
                            text=previous_item.content[0].text,
                            annotations=[],
                        )
                        yield _send_event(
                            openai_responses_types.ResponseTextDoneEvent(
                                type="response.output_text.done",
                                sequence_number=-1,
                                output_index=current_output_index,
                                content_index=current_content_index,
                                text=previous_item.content[0].text,
                                logprobs=[],
                                item_id=current_item_id,
                            )
                        )
                        yield _send_event(
                            openai_responses_types.ResponseContentPartDoneEvent(
                                type="response.content_part.done",
                                sequence_number=-1,
                                item_id=current_item_id,
                                output_index=current_output_index,
                                content_index=current_content_index,
                                part=text_content,
                            )
                        )
                        yield _send_event(
                            openai_responses_types.ResponseOutputItemDoneEvent(
                                type="response.output_item.done",
                                sequence_number=-1,
                                output_index=current_output_index,
                                item=openai_responses_types.ResponseOutputMessage(
                                    id=current_item_id,
                                    type="message",
```
**EN:** This block implements the async method `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `responses_stream_generator`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)`。它围绕 `responses_stream_generator` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 959-994: Continue async responses stream generator / 继续说明异步responses 流式 generator
```python
                                    role="assistant",
                                    content=[text_content],
                                    status="completed",
                                ),
                            )
                        )

            if ctx.parser.last_content_delta:
                if (
                    ctx.parser.current_channel == "final"
                    and ctx.parser.current_recipient is None
                ):
                    if not sent_output_item_added:
                        sent_output_item_added = True
                        yield _send_event(
                            openai_responses_types.ResponseOutputItemAddedEvent(
                                type="response.output_item.added",
                                sequence_number=-1,
                                output_index=current_output_index,
                                item=openai_responses_types.ResponseOutputMessage(
                                    id=current_item_id,
                                    type="message",
                                    role="assistant",
                                    content=[],
                                    status="in_progress",
                                ),
                            )
                        )
                        yield _send_event(
                            openai_responses_types.ResponseContentPartAddedEvent(
                                type="response.content_part.added",
                                sequence_number=-1,
                                output_index=current_output_index,
                                item_id=current_item_id,
                                content_index=current_content_index,
                                part=openai_responses_types.ResponseOutputText(
```
**EN:** This block implements the async method `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `responses_stream_generator`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)`。它围绕 `responses_stream_generator` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 995-1030: Continue async responses stream generator / 继续说明异步responses 流式 generator
```python
                                    type="output_text",
                                    text="",
                                    annotations=[],
                                    logprobs=None,
                                ),
                            )
                        )
                    yield _send_event(
                        openai_responses_types.ResponseTextDeltaEvent(
                            type="response.output_text.delta",
                            sequence_number=-1,
                            content_index=current_content_index,
                            output_index=current_output_index,
                            item_id=current_item_id,
                            delta=ctx.parser.last_content_delta,
                            # TODO, use logprobs from ctx.last_request_output
                            logprobs=[],
                        )
                    )
                elif (
                    ctx.parser.current_channel == "analysis"
                    and ctx.parser.current_recipient is None
                ):
                    if not sent_output_item_added:
                        sent_output_item_added = True
                        yield _send_event(
                            openai_responses_types.ResponseOutputItemAddedEvent(
                                type="response.output_item.added",
                                sequence_number=-1,
                                output_index=current_output_index,
                                item=openai_responses_types.ResponseReasoningItem(
                                    type="reasoning",
                                    id=current_item_id,
                                    summary=[],
                                    status="in_progress",
                                ),
```
**EN:** This block implements the async method `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `responses_stream_generator`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)`。它围绕 `responses_stream_generator` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 1031-1066: Continue async responses stream generator / 继续说明异步responses 流式 generator
```python
                            )
                        )
                        yield _send_event(
                            openai_responses_types.ResponseContentPartAddedEvent(
                                type="response.content_part.added",
                                sequence_number=-1,
                                output_index=current_output_index,
                                item_id=current_item_id,
                                content_index=current_content_index,
                                # TODO: migrate this to
                                # ResponseReasoningTextContent for now
                                part=openai_responses_types.ResponseOutputText(
                                    type="output_text",
                                    text="",
                                    annotations=[],
                                    logprobs=None,
                                ),
                            )
                        )
                    # TODO: migrate to OpenAI types once updated.
                    yield _send_event(
                        openai_responses_types.ResponseReasoningTextDeltaEvent(
                            type="response.reasoning_text.delta",
                            item_id=current_item_id,
                            output_index=current_output_index,
                            content_index=current_content_index,
                            delta=ctx.parser.last_content_delta,
                            sequence_number=-1,
                        )
                    )

            if ctx.is_assistant_action_turn() and len(ctx.parser.messages) > 0:
                previous_item = ctx.parser.messages[-1]
                if (
                    self.supports_browsing
                    and previous_item.recipient is not None
```
**EN:** This block implements the async method `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `responses_stream_generator`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)`。它围绕 `responses_stream_generator` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 1067-1102: Continue async responses stream generator / 继续说明异步responses 流式 generator
```python
                    and previous_item.recipient.startswith("browser.")
                ):
                    function_name = previous_item.recipient[len("browser.") :]
                    action = None
                    parsed_args = orjson.loads(previous_item.content[0].text)
                    if function_name == "search":
                        action = openai_responses_types.response_function_web_search.ActionSearch(
                            type="search",
                            query=parsed_args["query"],
                        )
                    elif function_name == "open":
                        action = openai_responses_types.response_function_web_search.ActionOpenPage(
                            type="open_page",
                            # TODO: translate to url
                            url=f"cursor:{parsed_args.get('cursor', '')}",
                        )
                    elif function_name == "find":
                        action = openai_responses_types.response_function_web_search.ActionFind(
                            type="find",
                            pattern=parsed_args["pattern"],
                            # TODO: translate to url
                            url=f"cursor:{parsed_args.get('cursor', '')}",
                        )
                    else:
                        raise ValueError(f"Unknown function name: {function_name}")

                    yield _send_event(
                        openai_responses_types.ResponseOutputItemAddedEvent(
                            type="response.output_item.added",
                            sequence_number=-1,
                            output_index=current_output_index,
                            item=openai_responses_types.response_function_web_search.ResponseFunctionWebSearch(
                                # TODO: generate a unique id for web search call
                                type="web_search_call",
                                id=current_item_id,
                                action=action,
```
**EN:** This block implements the async method `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `responses_stream_generator`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)`。它围绕 `responses_stream_generator` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 1103-1138: Continue async responses stream generator / 继续说明异步responses 流式 generator
```python
                                status="in_progress",
                            ),
                        )
                    )
                    yield _send_event(
                        openai_responses_types.ResponseWebSearchCallInProgressEvent(
                            type="response.web_search_call.in_progress",
                            sequence_number=-1,
                            output_index=current_output_index,
                            item_id=current_item_id,
                        )
                    )
                    yield _send_event(
                        openai_responses_types.ResponseWebSearchCallSearchingEvent(
                            type="response.web_search_call.searching",
                            sequence_number=-1,
                            output_index=current_output_index,
                            item_id=current_item_id,
                        )
                    )

                    # enqueue
                    yield _send_event(
                        openai_responses_types.ResponseWebSearchCallCompletedEvent(
                            type="response.web_search_call.completed",
                            sequence_number=-1,
                            output_index=current_output_index,
                            item_id=current_item_id,
                        )
                    )
                    yield _send_event(
                        openai_responses_types.ResponseOutputItemDoneEvent(
                            type="response.output_item.done",
                            sequence_number=-1,
                            output_index=current_output_index,
                            item=openai_responses_types.ResponseFunctionWebSearch(
```
**EN:** This block implements the async method `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `responses_stream_generator`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)`。它围绕 `responses_stream_generator` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 1139-1174: Continue async responses stream generator / 继续说明异步responses 流式 generator
```python
                                type="web_search_call",
                                id=current_item_id,
                                action=action,
                                status="completed",
                            ),
                        )
                    )

                if (
                    self.supports_code_interpreter
                    and previous_item.recipient is not None
                    and previous_item.recipient.startswith("python")
                ):
                    yield _send_event(
                        openai_responses_types.ResponseOutputItemAddedEvent(
                            type="response.output_item.added",
                            sequence_number=-1,
                            output_index=current_output_index,
                            item=openai_responses_types.ResponseCodeInterpreterToolCallParam(
                                type="code_interpreter_call",
                                id=current_item_id,
                                code="",
                                container_id="auto",
                                outputs=[],
                                status="in_progress",
                            ),
                        )
                    )
                    yield _send_event(
                        openai_responses_types.ResponseCodeInterpreterCallInProgressEvent(
                            type="response.code_interpreter_call.in_progress",
                            sequence_number=-1,
                            output_index=current_output_index,
                            item_id=current_item_id,
                        )
                    )
```
**EN:** This block implements the async method `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `responses_stream_generator`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)`。它围绕 `responses_stream_generator` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 1175-1210: Continue async responses stream generator / 继续说明异步responses 流式 generator
```python
                    # TODO: do we need to add delta event here?
                    yield _send_event(
                        openai_responses_types.ResponseCodeInterpreterCallCodeDoneEvent(
                            type="response.code_interpreter_call_code.done",
                            sequence_number=-1,
                            output_index=current_output_index,
                            item_id=current_item_id,
                            code=previous_item.content[0].text,
                        )
                    )
                    yield _send_event(
                        openai_responses_types.ResponseCodeInterpreterCallInterpretingEvent(
                            type="response.code_interpreter_call.interpreting",
                            sequence_number=-1,
                            output_index=current_output_index,
                            item_id=current_item_id,
                        )
                    )
                    yield _send_event(
                        openai_responses_types.ResponseCodeInterpreterCallCompletedEvent(
                            type="response.code_interpreter_call.completed",
                            sequence_number=-1,
                            output_index=current_output_index,
                            item_id=current_item_id,
                        )
                    )
                    yield _send_event(
                        openai_responses_types.ResponseOutputItemDoneEvent(
                            type="response.output_item.done",
                            sequence_number=-1,
                            output_index=current_output_index,
                            item=openai_responses_types.ResponseCodeInterpreterToolCallParam(
                                type="code_interpreter_call",
                                id=current_item_id,
                                code=previous_item.content[0].text,
                                container_id="auto",
```
**EN:** This block implements the async method `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `responses_stream_generator`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)`。它围绕 `responses_stream_generator` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 1211-1246: Continue async responses stream generator / 继续说明异步responses 流式 generator
```python
                                # TODO: add outputs here
                                outputs=[],
                                status="completed",
                            ),
                        )
                    )

        async def empty_async_generator():
            if False:
                yield

        final_response = await self.responses_full_generator(
            request,
            sampling_params,
            empty_async_generator(),
            context,
            model_name,
            tokenizer,
            request_metadata,
            created_time=created_time,
        )
        # Convert final_response to the format expected by ResponseCompletedEvent
        response_dict = final_response.model_dump()

        # Convert UsageInfo to ResponseUsage format
        if response_dict.get("usage"):
            usage_info = response_dict["usage"]
            response_dict["usage"] = {
                "input_tokens": usage_info.get("prompt_tokens", 0),
                "input_tokens_details": {
                    "cached_tokens": usage_info.get("cached_tokens", 0)
                },
                "output_tokens": usage_info.get("completion_tokens", 0),
                "output_tokens_details": {
                    "reasoning_tokens": usage_info.get("reasoning_tokens", 0)
                },
```
**EN:** This block implements the async method `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `responses_stream_generator`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)`。它围绕 `responses_stream_generator` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 1247-1256: Continue async responses stream generator / 继续说明异步responses 流式 generator
```python
                "total_tokens": usage_info.get("total_tokens", 0),
            }

        yield _send_event(
            openai_responses_types.ResponseCompletedEvent(
                type="response.completed",
                sequence_number=-1,
                response=response_dict,
            )
        )
```
**EN:** This block implements the async method `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)` on `OpenAIServingResponses`. It focuses on handling the serving responses responsibilities represented by `responses_stream_generator`, so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `responses_stream_generator(request, sampling_params, result_generator, context, model_name, ...)`。它围绕 `responses_stream_generator` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 1258-1293: Implement async generate with builtin tools / 实现异步generate with builtin 工具
```python
    async def _generate_with_builtin_tools(
        self,
        request_id: str,
        request_prompt: Any,
        adapted_request: GenerateReqInput,
        sampling_params: Any,
        context: ConversationContext,
        raw_request: Optional[Request] = None,
        priority: Optional[int] = None,
        **kwargs,
    ) -> AsyncGenerator[Any, None]:
        """Generate with builtin tool support for harmony-based models."""
        orig_priority = priority or 0

        while True:
            # Generate using SGLang's tokenizer manager
            generator = self.tokenizer_manager.generate_request(
                adapted_request, raw_request
            )

            async for res in generator:
                context.append_output(res)
                # NOTE(woosuk): The stop condition is handled by the engine.
                yield context

            if not context.need_builtin_tool_call():
                # The model did not ask for a tool call, so we're done.
                break

            # Call the tool and update the context with the result.
            tool_output = await context.call_tool()
            context.append_output(tool_output)

            # Prepare for the next generation turn
            # Render the updated conversation for the next completion
            prompt_token_ids = context.render_for_completion()
```
**EN:** This block implements the async method `_generate_with_builtin_tools(request_id, request_prompt, adapted_request, sampling_params, context, ...)` on `OpenAIServingResponses`. It focuses on Generate with builtin tool support for harmony-based models., so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `_generate_with_builtin_tools(request_id, request_prompt, adapted_request, sampling_params, context, ...)`。它围绕 `_generate_with_builtin_tools` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

### Lines 1294-1328: Continue async generate with builtin tools / 继续说明异步generate with builtin 工具
```python

            # Update the adapted request with new prompt
            adapted_request = GenerateReqInput(
                input_ids=prompt_token_ids,
                sampling_params=sampling_params,
                stream=adapted_request.stream,
                rid=request_id,
                extra_key=adapted_request.extra_key,
                return_logprob=adapted_request.return_logprob,
                logprob_start_len=adapted_request.logprob_start_len,
                top_logprobs_num=adapted_request.top_logprobs_num,
                return_text_in_logprobs=adapted_request.return_text_in_logprobs,
                return_hidden_states=adapted_request.return_hidden_states,
                background=adapted_request.background,
            )

            # Update sampling params with reduced max_tokens
            if hasattr(sampling_params, "max_new_tokens") or isinstance(
                sampling_params, dict
            ):
                context_len = getattr(
                    self.tokenizer_manager.model_config, "context_len", 4096
                )
                num_reserved_tokens = self.tokenizer_manager.num_reserved_tokens
                remaining_tokens = (
                    context_len - len(prompt_token_ids) - num_reserved_tokens
                )

                if isinstance(sampling_params, dict):
                    sampling_params["max_new_tokens"] = max(remaining_tokens, 1)
                else:
                    sampling_params.max_new_tokens = max(remaining_tokens, 1)

            # Slightly reduce priority for subsequent tool calls
            priority = orig_priority - 1
```
**EN:** This block implements the async method `_generate_with_builtin_tools(request_id, request_prompt, adapted_request, sampling_params, context, ...)` on `OpenAIServingResponses`. It focuses on Generate with builtin tool support for harmony-based models., so the class can advance the serving responses workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingResponses` 上的异步方法 `_generate_with_builtin_tools(request_id, request_prompt, adapted_request, sampling_params, context, ...)`。它围绕 `_generate_with_builtin_tools` 所承担的 serving responses 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: OpenAIServingResponses
- **Domain focus / 领域焦点**: serving responses / serving responses
- **Concurrency / 并发特征**: async/await appears throughout the module / 模块中多处使用 async/await 协调并发流程

## Dependencies / 依赖关系
- **Standard Library / 标准库**: asyncio, contextlib, copy, json, logging, time, typing
- **Third-party / 第三方库**: __future__, fastapi, fastapi.responses, http, jinja2, openai.types.responses, openai.types.responses.response_function_tool_call, openai.types.responses.response_reasoning_item, openai_harmony, orjson
- **Local Modules / 本地模块**: sglang.srt.entrypoints.context, sglang.srt.entrypoints.harmony_utils, sglang.srt.entrypoints.openai.protocol, sglang.srt.entrypoints.openai.serving_chat, sglang.srt.entrypoints.openai.tool_server, sglang.srt.managers.io_struct, sglang.srt.managers.template_manager, sglang.srt.managers.tokenizer_manager, sglang.srt.parser.reasoning_parser, sglang.srt.utils
