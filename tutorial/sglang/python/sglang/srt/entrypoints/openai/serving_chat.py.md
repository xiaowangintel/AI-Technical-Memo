# serving_chat.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/openai/serving_chat.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements serving chat logic for the OpenAI-compatible serving layer. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 serving 聊天 相关的逻辑，并服务于 OpenAI 兼容服务层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-9: Import runtime dependencies / 导入运行时依赖
```python
import copy
import json
import logging
import time
import uuid
from http import HTTPStatus
from typing import TYPE_CHECKING, Any, AsyncGenerator, Dict, List, Optional, Union
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 11-16: Import runtime dependencies / 导入运行时依赖
```python
import jinja2
import msgspec
import orjson
from fastapi import Request
from fastapi.responses import ORJSONResponse, StreamingResponse
from jsonschema import Draft202012Validator, SchemaError
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 18-57: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.entrypoints.openai import encoding_dsv4, encoding_dsv32
from sglang.srt.entrypoints.openai.protocol import (
    ChatCompletionRequest,
    ChatCompletionResponse,
    ChatCompletionResponseChoice,
    ChatCompletionResponseStreamChoice,
    ChatCompletionStreamResponse,
    ChatCompletionTokenLogprob,
    ChatMessage,
    ChoiceLogprobs,
    DeltaMessage,
    ErrorResponse,
    FunctionResponse,
    LogProbs,
    MessageProcessingResult,
    SglExt,
    ToolCall,
    ToolCallProcessingResult,
    ToolChoice,
    TopLogprob,
)
from sglang.srt.entrypoints.openai.serving_base import OpenAIServingBase
from sglang.srt.entrypoints.openai.usage_processor import UsageProcessor
from sglang.srt.entrypoints.openai.utils import (
    cached_tokens_details_from_dict,
    process_cached_tokens_details_from_ret,
    process_hidden_states_from_ret,
    process_routed_experts_from_ret,
    should_include_usage,
    to_openai_style_logprobs,
)
from sglang.srt.environ import envs
from sglang.srt.function_call.core_types import ToolCallItem
from sglang.srt.function_call.function_call_parser import FunctionCallParser
from sglang.srt.function_call.json_array_parser import JsonArrayParser
from sglang.srt.function_call.utils import get_json_schema_constraint
from sglang.srt.managers.io_struct import GenerateReqInput
from sglang.srt.parser.conversation import generate_chat_conv
from sglang.srt.parser.jinja_template_utils import process_content_for_template_format
from sglang.srt.parser.reasoning_parser import ReasoningParser
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 59-60: Provide supporting module logic / 提供辅助模块逻辑
```python
_SSE_DATA_B = b"data: "
_SSE_NL_B = b"\n\n"
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 63-71: Define class _StreamDelta / 定义类 _StreamDelta
```python
class _StreamDelta(msgspec.Struct, omit_defaults=True):
    # OpenAI Python SDK's ChoiceDelta does not declare reasoning_content; it is
    # surfaced via pydantic `extra`. With omit_defaults=True, defaulting to
    # None would drop the key entirely from the SSE payload, making
    # `data.reasoning_content` raise AttributeError on the client. Keep it
    # required (no default) so it is always serialized as null or a string.
    reasoning_content: Optional[str]
    role: Optional[str] = None
    content: Optional[str] = None
```
**EN:** This block declares the class `_StreamDelta`. It centers on coordinating serving chat behavior.
**CN:** 该代码块声明类 `_StreamDelta`。它负责承载与 serving 聊天 相关的核心状态与行为。

### Lines 74-79: Define class _StreamChoice / 定义类 _StreamChoice
```python
class _StreamChoice(msgspec.Struct):
    index: int
    delta: _StreamDelta
    logprobs: Optional[dict] = None
    finish_reason: Optional[str] = None
    matched_stop: Union[None, int, str] = None
```
**EN:** This block declares the class `_StreamChoice`. It centers on coordinating serving chat behavior.
**CN:** 该代码块声明类 `_StreamChoice`。它负责承载与 serving 聊天 相关的核心状态与行为。

### Lines 82-88: Define class _StreamChunk / 定义类 _StreamChunk
```python
class _StreamChunk(msgspec.Struct, omit_defaults=True):
    id: str
    object: str
    created: int
    model: str
    choices: List[_StreamChoice]
    usage: Optional[dict] = None
```
**EN:** This block declares the class `_StreamChunk`. It centers on coordinating serving chat behavior.
**CN:** 该代码块声明类 `_StreamChunk`。它负责承载与 serving 聊天 相关的核心状态与行为。

### Lines 91-91: Provide supporting module logic / 提供辅助模块逻辑
```python
_stream_encoder = msgspec.json.Encoder()
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 94-125: Implement fast sse content / 实现fast sse content
```python
def _fast_sse_content(
    chunk_id: str,
    created: int,
    model: str,
    index: int,
    role: Optional[str] = None,
    content: Optional[str] = None,
    reasoning_content: Optional[str] = None,
    finish_reason: Optional[str] = None,
    logprobs: Optional[dict] = None,
    matched_stop: Union[None, int, str] = None,
    usage: Optional[dict] = None,
) -> str:
    delta = _StreamDelta(
        role=role, content=content, reasoning_content=reasoning_content
    )
    choice = _StreamChoice(
        index=index,
        delta=delta,
        logprobs=logprobs,
        finish_reason=finish_reason,
        matched_stop=matched_stop,
    )
    chunk = _StreamChunk(
        id=chunk_id,
        object="chat.completion.chunk",
        created=created,
        model=model,
        choices=[choice],
        usage=usage,
    )
    return (_SSE_DATA_B + _stream_encoder.encode(chunk) + _SSE_NL_B).decode()
```
**EN:** This block implements the function `_fast_sse_content(chunk_id, created, model, index, role, ...)`. It focuses on handling the serving chat responsibilities represented by `_fast_sse_content`, providing reusable behavior for the serving chat pipeline.
**CN:** 该代码块实现函数 `_fast_sse_content(chunk_id, created, model, index, role, ...)`。它围绕 `_fast_sse_content` 所承担的 serving 聊天 相关职责展开，为对应处理链路提供可复用能力。

### Lines 128-130: Provide supporting module logic / 提供辅助模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.managers.template_manager import TemplateManager
    from sglang.srt.managers.tokenizer_manager import TokenizerManager
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 132-132: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 135-154: Implement normalize tool content / 实现normalize 工具 content
```python
def normalize_tool_content(role: str, content):
    """Normalize tool message content from OpenAI array format to plain string.

    OpenAI clients may send tool content as a list of content parts
    (e.g. [{"type":"text","text":"..."}]) but most chat templates expect
    a plain string for tool messages. Only flatten when ALL items are
    pure OpenAI text parts; preserve lists containing non-text-type items
    that some templates intentionally iterate over.
    """
    if role != "tool" or not isinstance(content, list):
        return content
    parts = content
    is_openai_text_parts = all(
        (isinstance(p, dict) and p.get("type") == "text") or isinstance(p, str)
        for p in parts
    )
    if is_openai_text_parts:
        text_parts = [p.get("text", "") if isinstance(p, dict) else p for p in parts]
        return " ".join(text_parts)
    return content
```
**EN:** This block implements the function `normalize_tool_content(role, content)`. It focuses on Normalize tool message content from OpenAI array format to plain string., providing reusable behavior for the serving chat pipeline.
**CN:** 该代码块实现函数 `normalize_tool_content(role, content)`。它围绕 `normalize_tool_content` 所承担的 serving 聊天 相关职责展开，为对应处理链路提供可复用能力。

### Lines 157-180: Implement extract max dynamic patch / 实现extract max dynamic patch
```python
def _extract_max_dynamic_patch(request: ChatCompletionRequest):
    img_vals = []
    vid_vals = []
    for msg in request.messages or []:
        content = getattr(msg, "content", None)
        if not isinstance(content, list):
            continue
        for part in content:
            # pydantic object or dict type
            if getattr(part, "type", None) == "image_url":
                iu = getattr(part, "image_url", None)
                mdp = getattr(iu, "max_dynamic_patch", None) if iu else None
                if mdp is not None:
                    img_vals.append(int(mdp))
            elif getattr(part, "type", None) == "video_url":
                vu = getattr(part, "video_url", None)
                mdp = getattr(vu, "max_dynamic_patch", None) if vu else None
                if mdp is not None:
                    vid_vals.append(int(mdp))

    # TODO(yuan-luo): per-item max_dynamic_patch for both image and video
    img_max_dynamic_patch = min(img_vals) if img_vals else None
    vid_max_dynamic_patch = min(vid_vals) if vid_vals else None
    return img_max_dynamic_patch, vid_max_dynamic_patch
```
**EN:** This block implements the function `_extract_max_dynamic_patch(request)`. It focuses on handling the serving chat responsibilities represented by `_extract_max_dynamic_patch`, providing reusable behavior for the serving chat pipeline.
**CN:** 该代码块实现函数 `_extract_max_dynamic_patch(request)`。它围绕 `_extract_max_dynamic_patch` 所承担的 serving 聊天 相关职责展开，为对应处理链路提供可复用能力。

### Lines 183-184: Provide supporting module logic / 提供辅助模块逻辑
```python
class OpenAIServingChat(OpenAIServingBase):
    """Handler for /v1/chat/completions requests"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 186-186: Provide supporting module logic / 提供辅助模块逻辑
```python
    _default_sampling_params_logged = False
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 188-223: Initialize OpenAIServingChat / 初始化 OpenAIServingChat
```python
    def __init__(
        self,
        tokenizer_manager: TokenizerManager,
        template_manager: TemplateManager,
    ):
        super().__init__(tokenizer_manager)
        self.template_manager = template_manager
        self.tool_call_parser = self.tokenizer_manager.server_args.tool_call_parser
        self.reasoning_parser = self.tokenizer_manager.server_args.reasoning_parser
        self._reasoning_detector = None
        if self.reasoning_parser:
            try:
                rp = ReasoningParser(
                    model_type=self.reasoning_parser, stream_reasoning=True
                )
                self._reasoning_detector = rp.detector
            except ValueError as e:
                logger.warning(
                    "Failed to initialize reasoning detector for parser '%s': %s",
                    self.reasoning_parser,
                    e,
                )

        # Get default sampling parameters from model's generation config
        self.default_sampling_params = (
            self.tokenizer_manager.model_config.get_default_sampling_params()
        )
        if (
            self.default_sampling_params
            and not OpenAIServingChat._default_sampling_params_logged
        ):
            logger.info(
                f"Using default chat sampling params from model generation config: {self.default_sampling_params}",
            )
            OpenAIServingChat._default_sampling_params_logged = True
```
**EN:** This block implements the initializer `__init__(tokenizer_manager, template_manager)` for `OpenAIServingChat`. It prepares the object state and connects the instance to the surrounding serving chat workflow.
**CN:** 该代码块实现 `OpenAIServingChat` 的初始化方法 `__init__(tokenizer_manager, template_manager)`。它负责准备对象状态，并把实例接入 serving 聊天 相关的运行流程。

### Lines 224-238: Initialize OpenAIServingChat (continued) / 初始化 OpenAIServingChat（续）
```python
        # Check if the model is a GPT-OSS model
        self.is_gpt_oss = (
            hasattr(self.tokenizer_manager.model_config, "hf_config")
            and hasattr(self.tokenizer_manager.model_config.hf_config, "model_type")
            and self.tokenizer_manager.model_config.hf_config.model_type == "gpt_oss"
        )
        self.is_gemma4 = (
            hasattr(self.tokenizer_manager.model_config, "hf_config")
            and hasattr(self.tokenizer_manager.model_config.hf_config, "model_type")
            and self.tokenizer_manager.model_config.hf_config.model_type == "gemma4"
        )

        # Which Python-based chat encoder (if any) bypasses apply_chat_template.
        # Values: "dsv32", "dsv4", or None.
        self.chat_encoding_spec = self._resolve_chat_encoding_spec()
```
**EN:** This block implements the initializer `__init__(tokenizer_manager, template_manager)` for `OpenAIServingChat`. It prepares the object state and connects the instance to the surrounding serving chat workflow.
**CN:** 该代码块实现 `OpenAIServingChat` 的初始化方法 `__init__(tokenizer_manager, template_manager)`。它负责准备对象状态，并把实例接入 serving 聊天 相关的运行流程。

### Lines 240-276: Implement handle last assistant message / 实现handle last assistant message
```python
    def _handle_last_assistant_message(
        self,
        messages: List[Dict[str, Any]],
        request: ChatCompletionRequest,
    ) -> tuple[List[Dict[str, Any]], Optional[str]]:
        """
        Handle continue_final_message feature: separate final assistant message.

        If continue_final_message is enabled and the last message is from assistant,
        extract its content and remove it from the message list.
        If continue_final_message is False and the last message is from assistant,
        convert it to a user message to ensure the last message is always from user.

        Only processes text-based content (strings), ignoring multimodal content (lists).

        Args:
            messages: List of message dictionaries
            request: ChatCompletionRequest with continue_final_message flag

        Returns:
            Tuple of (processed_messages, assistant_prefix)
            - processed_messages: Messages with last assistant message handled appropriately
            - assistant_prefix: Content of the last assistant message (string only), or None
        """
        assistant_prefix = None
        if messages and messages[-1].get("role") == "assistant":
            last_content = messages[-1].get("content")
            # Only process string content, ignore multimodal content (lists)
            if isinstance(last_content, str):
                if request.continue_final_message:
                    # Extract content and remove the assistant message
                    assistant_prefix = last_content
                    messages = messages[:-1]
                else:
                    # Convert the last assistant message to user message
                    messages[-1] = {"role": "user", "content": last_content}
        return messages, assistant_prefix
```
**EN:** This block implements the method `_handle_last_assistant_message(messages, request)` on `OpenAIServingChat`. It focuses on Handle continue_final_message feature: separate final assistant message., so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_handle_last_assistant_message(messages, request)`。它围绕 `_handle_last_assistant_message` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 278-294: Implement append assistant prefix to prompt ids / 实现append assistant prefix to prompt ids
```python
    def _append_assistant_prefix_to_prompt_ids(
        self, prompt_ids: List[int], assistant_prefix: str
    ) -> List[int]:
        """
        Append assistant prefix to prompt_ids.

        Args:
            prompt_ids: Current prompt token IDs
            assistant_prefix: Assistant message content to append

        Returns:
            Updated prompt_ids with assistant prefix appended
        """
        encoded = self.tokenizer_manager.tokenizer.encode(assistant_prefix)
        if encoded and encoded[0] == self.tokenizer_manager.tokenizer.bos_token_id:
            encoded = encoded[1:]
        return prompt_ids + encoded
```
**EN:** This block implements the method `_append_assistant_prefix_to_prompt_ids(prompt_ids, assistant_prefix)` on `OpenAIServingChat`. It focuses on Append assistant prefix to prompt_ids., so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_append_assistant_prefix_to_prompt_ids(prompt_ids, assistant_prefix)`。它围绕 `_append_assistant_prefix_to_prompt_ids` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 296-314: Implement resolve chat encoding spec / 实现resolve 聊天 encoding spec
```python
    def _resolve_chat_encoding_spec(self) -> Optional[str]:
        if self.tool_call_parser == "deepseekv4":
            return "dsv4"
        if self.tool_call_parser == "deepseekv32":
            return "dsv32"

        architectures = self.tokenizer_manager.model_config.hf_config.architectures
        arch = architectures[0] if architectures else ""

        if "DeepseekV4" in arch:
            return "dsv4"

        has_chat_template = (
            self.tokenizer_manager.tokenizer is not None
            and self.tokenizer_manager.tokenizer.chat_template is not None
        )
        if "DeepseekV3" in arch and not has_chat_template:
            return "dsv32"
        return None
```
**EN:** This block implements the method `_resolve_chat_encoding_spec()` on `OpenAIServingChat`. It focuses on handling the serving chat responsibilities represented by `_resolve_chat_encoding_spec`, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_resolve_chat_encoding_spec()`。它围绕 `_resolve_chat_encoding_spec` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 316-317: Implement request id prefix / 实现请求 id prefix
```python
    def _request_id_prefix(self) -> str:
        return "chatcmpl-"
```
**EN:** This block implements the method `_request_id_prefix()` on `OpenAIServingChat`. It focuses on handling the serving chat responsibilities represented by `_request_id_prefix`, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_request_id_prefix()`。它围绕 `_request_id_prefix` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 319-365: Implement validate request / 实现validate 请求
```python
    def _validate_request(self, request: ChatCompletionRequest) -> Optional[str]:
        """Validate that the input is valid."""
        if not request.messages:
            return "Messages cannot be empty."

        if (
            isinstance(request.tool_choice, str)
            and request.tool_choice.lower() == "required"
            and not request.tools
        ):
            return "Tools cannot be empty if tool choice is set to required."

        if request.tool_choice is not None and not isinstance(request.tool_choice, str):
            if not request.tools:
                return "Tools cannot be empty if tool choice is set to a specific tool."
            tool_name = request.tool_choice.function.name
            tool_exists = any(tool.function.name == tool_name for tool in request.tools)
            if not tool_exists:
                return f"Tool '{tool_name}' not found in tools list."

        # Validate tool definitions
        for i, tool in enumerate(request.tools or []):
            if tool.function.parameters is None:
                continue
            try:
                Draft202012Validator.check_schema(tool.function.parameters)
            except SchemaError as e:
                return f"Tool {i} function has invalid 'parameters' schema: {str(e)}"

        max_output_tokens = request.max_completion_tokens or request.max_tokens
        server_context_length = self.tokenizer_manager.server_args.context_length
        if (
            max_output_tokens
            and server_context_length
            and max_output_tokens > server_context_length
        ) and not self.tokenizer_manager.server_args.allow_auto_truncate:
            return (
                f"max_completion_tokens is too large: {max_output_tokens}."
                f"This model supports at most {server_context_length} completion tokens."
            )

        if request.response_format and request.response_format.type == "json_schema":
            schema = getattr(request.response_format.json_schema, "schema_", None)
            if schema is None:
                return "schema_ is required for json_schema response format request."

        return None
```
**EN:** This block implements the method `_validate_request(request)` on `OpenAIServingChat`. It focuses on Validate that the input is valid., so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_validate_request(request)`。它围绕 `_validate_request` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 367-402: Implement convert to internal request / 实现convert to internal 请求
```python
    def _convert_to_internal_request(
        self,
        request: ChatCompletionRequest,
        raw_request: Request = None,
    ) -> tuple[GenerateReqInput, ChatCompletionRequest]:
        reasoning_effort = (
            request.chat_template_kwargs.pop("reasoning_effort", None)
            if request.chat_template_kwargs
            else None
        )
        if self.is_gpt_oss and reasoning_effort == "none":
            raise ValueError(
                f"Harmony does not support reasoning effort {reasoning_effort}"
            )

        if reasoning_effort is not None:
            request.reasoning_effort = reasoning_effort

        """Convert OpenAI chat completion request to internal format"""
        is_multimodal = self.tokenizer_manager.model_config.is_multimodal

        # Process messages and apply chat template
        processed_messages = self._process_messages(request, is_multimodal)

        # Build sampling parameters
        sampling_params = request.to_sampling_params(
            stop=processed_messages.stop,
            model_generation_config=self.default_sampling_params,
            tool_call_constraint=processed_messages.tool_call_constraint,
        )

        # Handle single vs multiple requests
        if is_multimodal:
            prompt_kwargs = {"text": processed_messages.prompt}
        else:
            if isinstance(processed_messages.prompt_ids, str):
```
**EN:** This block implements the method `_convert_to_internal_request(request, raw_request)` on `OpenAIServingChat`. It focuses on handling the serving chat responsibilities represented by `_convert_to_internal_request`, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_convert_to_internal_request(request, raw_request)`。它围绕 `_convert_to_internal_request` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 403-438: Continue convert to internal request / 继续说明convert to internal 请求
```python
                prompt_kwargs = {"text": processed_messages.prompt_ids}
            else:
                prompt_kwargs = {"input_ids": processed_messages.prompt_ids}

        # Extract custom labels from raw request headers
        custom_labels = self.extract_custom_labels(raw_request)

        # Extract routed_dp_rank from header (has higher priority than body)
        effective_routed_dp_rank = self.extract_routed_dp_rank_from_header(
            raw_request, request.routed_dp_rank
        )

        # Resolve LoRA adapter from model parameter or explicit lora_path
        lora_path = self._resolve_lora_path(request.model, request.lora_path)
        img_max_dynamic_patch, vid_max_dynamic_patch = _extract_max_dynamic_patch(
            request
        )
        adapted_request = GenerateReqInput(
            **prompt_kwargs,
            image_data=processed_messages.image_data,
            video_data=processed_messages.video_data,
            audio_data=processed_messages.audio_data,
            sampling_params=sampling_params,
            return_logprob=request.logprobs,
            logprob_start_len=-1,
            top_logprobs_num=request.top_logprobs or 0,
            stream=request.stream,
            return_text_in_logprobs=True,
            modalities=processed_messages.modalities,
            lora_path=lora_path,
            bootstrap_host=request.bootstrap_host,
            bootstrap_port=request.bootstrap_port,
            bootstrap_room=request.bootstrap_room,
            routed_dp_rank=effective_routed_dp_rank,
            disagg_prefill_dp_rank=request.disagg_prefill_dp_rank,
            return_hidden_states=request.return_hidden_states,
```
**EN:** This block implements the method `_convert_to_internal_request(request, raw_request)` on `OpenAIServingChat`. It focuses on handling the serving chat responsibilities represented by `_convert_to_internal_request`, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_convert_to_internal_request(request, raw_request)`。它围绕 `_convert_to_internal_request` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 439-454: Continue convert to internal request / 继续说明convert to internal 请求
```python
            return_routed_experts=request.return_routed_experts,
            routed_experts_start_len=request.routed_experts_start_len,
            rid=request.rid,
            extra_key=self._compute_extra_key(request),
            require_reasoning=self._get_reasoning_from_request(request),
            priority=request.priority,
            routing_key=self.extract_routing_key(raw_request),
            custom_labels=custom_labels,
            custom_logit_processor=request.custom_logit_processor,
            image_max_dynamic_patch=img_max_dynamic_patch,
            video_max_dynamic_patch=vid_max_dynamic_patch,
            max_dynamic_patch=getattr(request, "max_dynamic_patch", None),
            use_audio_in_video=getattr(request, "use_audio_in_video", False),
        )

        return adapted_request, request
```
**EN:** This block implements the method `_convert_to_internal_request(request, raw_request)` on `OpenAIServingChat`. It focuses on handling the serving chat responsibilities represented by `_convert_to_internal_request`, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_convert_to_internal_request(request, raw_request)`。它围绕 `_convert_to_internal_request` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 456-491: Implement process messages / 实现process messages
```python
    def _process_messages(
        self, request: ChatCompletionRequest, is_multimodal: bool
    ) -> MessageProcessingResult:
        """Process chat messages and apply chat template"""
        # GptOss model needs to keep special tokens for harmony parsing
        if self.is_gpt_oss or self.is_gemma4:
            request.skip_special_tokens = False

        self._patch_mistral_skip_special_tokens(request)

        thinking_mode = self._get_reasoning_from_request(request)
        # SGLang's ReasonerGrammarBackend owns the reasoning prefix
        # when --reasoning-parser is configured, so builtin xgrammar
        # tags must describe only the post-reasoning tool-call suffix.
        xgrammar_reasoning = thinking_mode and (
            self.tokenizer_manager.server_args.reasoning_parser is not None
        )
        tool_call_constraint = None

        # Apply chat template and its stop strings
        tools = None
        if request.tools and request.tool_choice != "none":
            request.skip_special_tokens = False
            if not isinstance(request.tool_choice, str):
                tools = [
                    item.model_dump()
                    for item in request.tools
                    if item.function.name == request.tool_choice.function.name
                ]
            else:
                tools = [item.model_dump() for item in request.tools]
            if self.tool_call_parser:
                parser = FunctionCallParser(request.tools, self.tool_call_parser)
                tool_call_constraint = parser.get_structure_constraint(
                    request.tool_choice,
                    parallel_tool_calls=request.parallel_tool_calls,
```
**EN:** This block implements the method `_process_messages(request, is_multimodal)` on `OpenAIServingChat`. It focuses on Process chat messages and apply chat template, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_process_messages(request, is_multimodal)`。它围绕 `_process_messages` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 492-514: Continue process messages / 继续说明process messages
```python
                    thinking_mode=xgrammar_reasoning,
                )
            # Fallback: use generic JSON schema for required/named tool choice
            # only when no parser-specific constraint was set
            if tool_call_constraint is None and (
                request.tool_choice == "required"
                or isinstance(request.tool_choice, ToolChoice)
            ):
                json_schema = get_json_schema_constraint(
                    request.tools,
                    request.tool_choice,
                    parallel_tool_calls=request.parallel_tool_calls,
                )
                tool_call_constraint = ("json_schema", json_schema)

        # Use chat template
        if self.template_manager.chat_template_name is None:
            result = self._apply_jinja_template(request, tools, is_multimodal)
        else:
            result = self._apply_conversation_template(request, is_multimodal)

        result.tool_call_constraint = tool_call_constraint
        return result
```
**EN:** This block implements the method `_process_messages(request, is_multimodal)` on `OpenAIServingChat`. It focuses on Process chat messages and apply chat template, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_process_messages(request, is_multimodal)`。它围绕 `_process_messages` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 516-551: Implement apply jinja template / 实现apply jinja template
```python
    def _apply_jinja_template(
        self,
        request: ChatCompletionRequest,
        tools: Optional[List[Dict]],
        is_multimodal: bool,
    ) -> MessageProcessingResult:
        """Apply Jinja chat template"""
        prompt = ""
        prompt_ids = []
        openai_compatible_messages = []
        image_data = []
        video_data = []
        audio_data = []
        modalities = []

        template_content_format = self.template_manager.jinja_template_content_format

        if self.chat_encoding_spec is not None:
            # Per-request wins; env is fallback default for benchmark
            # workflows that can't pass per-request chat_template_kwargs.
            thinking_requested = (request.chat_template_kwargs or {}).get(
                "thinking", envs.SGLANG_DEFAULT_THINKING.get()
            )
            thinking_mode = "thinking" if thinking_requested else "chat"
            messages = [msg.model_dump() for msg in request.messages]

            # dsv4/dsv32 are text-only and consume string content; flatten
            # OpenAI parts-list content here so the encoder sees a plain string.
            for i, msg in enumerate(messages):
                if isinstance(msg.get("content"), list):
                    messages[i] = process_content_for_template_format(
                        msg, "string", [], [], [], []
                    )

            for msg in messages:
                if msg.get("content") is None:
```
**EN:** This block implements the method `_apply_jinja_template(request, tools, is_multimodal)` on `OpenAIServingChat`. It focuses on Apply Jinja chat template, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_apply_jinja_template(request, tools, is_multimodal)`。它围绕 `_apply_jinja_template` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 552-587: Continue apply jinja template / 继续说明apply jinja template
```python
                    msg["content"] = ""
                processed_msg = process_content_for_template_format(
                    msg,
                    template_content_format,
                    image_data,
                    video_data,
                    audio_data,
                    modalities,
                    use_dpsk_v32_encoding=self.chat_encoding_spec == "dsv32",
                )
                msg.update(processed_msg)

            # Handle continue_final_message: separate final assistant message
            messages, assistant_prefix = self._handle_last_assistant_message(
                messages, request
            )

            if messages[0]["role"] != "system":
                # insert an empty system prompt to help render tool system prompt
                messages.insert(0, {"role": "system", "content": ""})
            if request.tools:
                messages[0]["tools"] = [tool.model_dump() for tool in request.tools]

            if self.chat_encoding_spec == "dsv4":
                # V4 encoder only accepts "max" / "high" / None.
                # OpenAI protocol defaults to "medium" which V4 rejects; drop it.
                # Fallback: if request didn't set it, try env SGLANG_DSV4_REASONING_EFFORT.
                effort_source = request.reasoning_effort
                if effort_source is None:
                    env_val = envs.SGLANG_DSV4_REASONING_EFFORT.get()
                    if env_val:
                        effort_source = env_val
                v4_reasoning_effort = (
                    effort_source if effort_source in ("max", "high") else None
                )
                if request.task is not None:
```
**EN:** This block implements the method `_apply_jinja_template(request, tools, is_multimodal)` on `OpenAIServingChat`. It focuses on Apply Jinja chat template, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_apply_jinja_template(request, tools, is_multimodal)`。它围绕 `_apply_jinja_template` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 588-623: Continue apply jinja template / 继续说明apply jinja template
```python
                    encoding_dsv4.attach_task_to_last_user_message(
                        messages, request.task
                    )
                real_input = encoding_dsv4.encode_messages(
                    messages,
                    thinking_mode=thinking_mode,
                    reasoning_effort=v4_reasoning_effort,
                )
            else:
                real_input = encoding_dsv32.encode_messages(
                    messages, thinking_mode=thinking_mode
                )
            prompt_ids = self.tokenizer_manager.tokenizer.encode(real_input)

            # Append assistant prefix if continue_final_message is enabled
            if assistant_prefix:
                prompt_ids = self._append_assistant_prefix_to_prompt_ids(
                    prompt_ids, assistant_prefix
                )
        else:
            for message in request.messages:
                if message.content is None:
                    message.content = ""
                msg_dict = message.model_dump()

                # Process content based on detected template format
                processed_msg = process_content_for_template_format(
                    msg_dict,
                    template_content_format,
                    image_data,
                    video_data,
                    audio_data,
                    modalities,
                )

                processed_msg["content"] = normalize_tool_content(
```
**EN:** This block implements the method `_apply_jinja_template(request, tools, is_multimodal)` on `OpenAIServingChat`. It focuses on Apply Jinja chat template, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_apply_jinja_template(request, tools, is_multimodal)`。它围绕 `_apply_jinja_template` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 624-659: Continue apply jinja template / 继续说明apply jinja template
```python
                    processed_msg["role"], processed_msg.get("content")
                )

                # per the Transformers docs & maintainers, tool call arguments in
                # assistant-role messages with tool_calls need to be dicts not JSON str -
                # this is how tool-use chat templates will expect them moving forwards
                # so, for messages that have tool_calls, parse the string (which we get
                # from openAI format) to dict
                if (
                    processed_msg["role"] == "assistant"
                    and "tool_calls" in processed_msg
                    and isinstance(processed_msg["tool_calls"], list)
                ):
                    for item in processed_msg["tool_calls"]:
                        if "arguments" in item["function"] and isinstance(
                            item["function"]["arguments"], str
                        ):
                            item["function"]["arguments"] = orjson.loads(
                                item["function"]["arguments"]
                            )

                openai_compatible_messages.append(processed_msg)

            # Handle continue_final_message: separate final assistant message
            openai_compatible_messages, assistant_prefix = (
                self._handle_last_assistant_message(openai_compatible_messages, request)
            )

            extra_template_kwargs = {}
            if request.reasoning_effort is not None:
                extra_template_kwargs["reasoning_effort"] = request.reasoning_effort
            if request.chat_template_kwargs:
                extra_template_kwargs.update(request.chat_template_kwargs)

            try:
                prompt_ids = self.tokenizer_manager.tokenizer.apply_chat_template(
```
**EN:** This block implements the method `_apply_jinja_template(request, tools, is_multimodal)` on `OpenAIServingChat`. It focuses on Apply Jinja chat template, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_apply_jinja_template(request, tools, is_multimodal)`。它围绕 `_apply_jinja_template` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 660-695: Continue apply jinja template / 继续说明apply jinja template
```python
                    openai_compatible_messages,
                    tokenize=True,
                    add_generation_prompt=True,
                    tools=tools,
                    return_dict=False,
                    **extra_template_kwargs,
                )
            except Exception as e:
                # If the first attempt fails, try with flat function-only format.
                # Some templates (e.g. Mistral) expect tools without the OpenAI wrapper.
                tools = (
                    [t["function"] if "function" in t else t for t in tools]
                    if tools
                    else None
                )
                try:
                    prompt_ids = self.tokenizer_manager.tokenizer.apply_chat_template(
                        openai_compatible_messages,
                        tokenize=True,
                        add_generation_prompt=True,
                        tools=tools,
                        return_dict=False,
                        **extra_template_kwargs,
                    )
                except jinja2.TemplateError as template_error:
                    # Template errors (e.g., from raise_exception in Jinja templates)
                    # should be treated as client errors (400 BadRequest)
                    raise ValueError(str(template_error)) from template_error

            # Append assistant prefix if continue_final_message is enabled
            if assistant_prefix:
                prompt_ids = self._append_assistant_prefix_to_prompt_ids(
                    prompt_ids, assistant_prefix
                )

            if is_multimodal:
```
**EN:** This block implements the method `_apply_jinja_template(request, tools, is_multimodal)` on `OpenAIServingChat`. It focuses on Apply Jinja chat template, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_apply_jinja_template(request, tools, is_multimodal)`。它围绕 `_apply_jinja_template` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 696-711: Continue apply jinja template / 继续说明apply jinja template
```python
                prompt = self.tokenizer_manager.tokenizer.decode(prompt_ids)

        stop = request.stop
        image_data = image_data if image_data else None
        audio_data = audio_data if audio_data else None
        video_data = video_data if video_data else None
        modalities = modalities if modalities else []
        return MessageProcessingResult(
            prompt=prompt,
            prompt_ids=prompt_ids,
            image_data=image_data,
            video_data=video_data,
            audio_data=audio_data,
            modalities=modalities,
            stop=stop,
        )
```
**EN:** This block implements the method `_apply_jinja_template(request, tools, is_multimodal)` on `OpenAIServingChat`. It focuses on Apply Jinja chat template, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_apply_jinja_template(request, tools, is_multimodal)`。它围绕 `_apply_jinja_template` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 713-748: Implement apply conversation template / 实现apply conversation template
```python
    def _apply_conversation_template(
        self,
        request: ChatCompletionRequest,
        is_multimodal: bool,
    ) -> MessageProcessingResult:
        """Apply conversation template"""
        prompt = ""
        prompt_ids = []
        conv = generate_chat_conv(request, self.template_manager.chat_template_name)

        # If we should continue the final assistant message, adjust the conversation.
        if (
            request.continue_final_message
            and request.messages
            and request.messages[-1].role == "assistant"
        ):
            # Remove the auto-added blank assistant turn, if present.
            if conv.messages and conv.messages[-1][1] is None:
                conv.messages.pop()
            # Rebuild the prompt from the conversation.
            prompt = conv.get_prompt()
            # Strip trailing stop tokens or separators that indicate end-of-assistant.
            if isinstance(conv.stop_str, list):
                for stop_token in conv.stop_str:
                    if prompt.endswith(stop_token):
                        prompt = prompt[: -len(stop_token)]
            elif isinstance(conv.stop_str, str) and prompt.endswith(conv.stop_str):
                prompt = prompt[: -len(conv.stop_str)]
            if conv.sep and prompt.endswith(conv.sep):
                prompt = prompt[: -len(conv.sep)]
            if getattr(conv, "sep2", None) and prompt.endswith(conv.sep2):
                prompt = prompt[: -len(conv.sep2)]
        else:
            prompt = conv.get_prompt()
            if self._get_reasoning_from_request(request) and (
                self._reasoning_detector is None
```
**EN:** This block implements the method `_apply_conversation_template(request, is_multimodal)` on `OpenAIServingChat`. It focuses on Apply conversation template, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_apply_conversation_template(request, is_multimodal)`。它围绕 `_apply_conversation_template` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 749-777: Continue apply conversation template / 继续说明apply conversation template
```python
                or not self._reasoning_detector.thinks_internally
            ):
                # Models with thinks_internally=True think without a leading <think> token
                prompt += "<think>"  # Note(Xinyuan): hard code thinking token

        image_data = conv.image_data if conv.image_data else None
        video_data = conv.video_data if conv.video_data else None
        audio_data = conv.audio_data if conv.audio_data else None
        modalities = conv.modalities if conv.modalities else []
        stop = copy.copy(conv.stop_str or [] if not request.ignore_eos else [])

        if request.stop:
            if isinstance(request.stop, str):
                stop.append(request.stop)
            else:
                stop.extend(request.stop)

        if not is_multimodal:
            prompt_ids = self.tokenizer_manager.tokenizer.encode(prompt)

        return MessageProcessingResult(
            prompt=prompt,
            prompt_ids=prompt_ids,
            image_data=image_data,
            video_data=video_data,
            audio_data=audio_data,
            modalities=modalities,
            stop=stop,
        )
```
**EN:** This block implements the method `_apply_conversation_template(request, is_multimodal)` on `OpenAIServingChat`. It focuses on Apply conversation template, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_apply_conversation_template(request, is_multimodal)`。它围绕 `_apply_conversation_template` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 779-805: Implement async handle streaming request / 实现异步handle streaming 请求
```python
    async def _handle_streaming_request(
        self,
        adapted_request: GenerateReqInput,
        request: ChatCompletionRequest,
        raw_request: Request,
    ) -> Union[StreamingResponse, ErrorResponse]:
        """Handle streaming chat completion request"""
        generator = self._generate_chat_stream(adapted_request, request, raw_request)

        # Kick-start the generator to trigger validation before HTTP 200 is sent.
        # If validation fails (e.g., context length exceeded), we can still return
        # a proper HTTP 400 error response instead of streaming it as SSE payload.
        try:
            first_chunk = await generator.__anext__()
        except ValueError as e:
            return self.create_error_response(str(e))

        async def prepend_first_chunk():
            yield first_chunk
            async for chunk in generator:
                yield chunk

        return StreamingResponse(
            prepend_first_chunk(),
            media_type="text/event-stream",
            background=self.tokenizer_manager.create_abort_task(adapted_request),
        )
```
**EN:** This block implements the async method `_handle_streaming_request(adapted_request, request, raw_request)` on `OpenAIServingChat`. It focuses on Handle streaming chat completion request, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的异步方法 `_handle_streaming_request(adapted_request, request, raw_request)`。它围绕 `_handle_streaming_request` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 807-842: Implement async generate chat stream / 实现异步generate 聊天 流式
```python
    async def _generate_chat_stream(
        self,
        adapted_request: GenerateReqInput,
        request: ChatCompletionRequest,
        raw_request: Request,
    ) -> AsyncGenerator[str, None]:
        """Generate streaming chat completion response"""
        # Parsers for tool calls and reasoning
        parser_dict = {}
        reasoning_parser_dict = {}

        # State tracking for streaming
        is_firsts = {}
        stream_offsets = {}
        n_prev_tokens = {}
        has_tool_calls = {}
        finish_reasons = {}

        # Usage tracking
        prompt_tokens = {}
        reasoning_tokens = {}
        completion_tokens = {}
        cached_tokens = {}
        hidden_states = {}
        routed_experts = {}
        cached_tokens_details = {}

        stream_started = False
        try:
            include_usage, continuous_usage_stats = should_include_usage(
                request.stream_options,
                self.tokenizer_manager.server_args.stream_response_default_include_usage,
            )

            async for content in self.tokenizer_manager.generate_request(
                adapted_request, raw_request
```
**EN:** This block implements the async method `_generate_chat_stream(adapted_request, request, raw_request)` on `OpenAIServingChat`. It focuses on Generate streaming chat completion response, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的异步方法 `_generate_chat_stream(adapted_request, request, raw_request)`。它围绕 `_generate_chat_stream` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 843-878: Continue async generate chat stream / 继续说明异步generate 聊天 流式
```python
            ):
                index = content.get("index", 0)

                prompt_tokens[index] = content["meta_info"].get("prompt_tokens", 0)
                completion_tokens[index] = content["meta_info"].get(
                    "completion_tokens", 0
                )
                reasoning_tokens[index] = content["meta_info"].get(
                    "reasoning_tokens", 0
                )
                cached_tokens[index] = content["meta_info"].get("cached_tokens", 0)
                hidden_states[index] = content["meta_info"].get("hidden_states", None)
                routed_experts[index] = content["meta_info"].get("routed_experts", None)
                cached_tokens_details[index] = content["meta_info"].get(
                    "cached_tokens_details", None
                )

                # Handle logprobs
                choice_logprobs = None
                if request.logprobs:
                    n_prev_token = n_prev_tokens.get(index, 0)
                    total_output_logprobs = content["meta_info"][
                        "output_token_logprobs_length"
                    ]
                    if n_prev_token < total_output_logprobs:
                        choice_logprobs = self._process_streaming_logprobs(
                            content, n_prev_token, total_output_logprobs
                        ).model_dump()
                    n_prev_tokens[index] = total_output_logprobs

                finish_reason = content["meta_info"].get("finish_reason", None)
                finish_reason_type = finish_reason["type"] if finish_reason else None

                # Track finish_reason for each index
                if finish_reason_type:
                    # Abort with an explicit error status_code is a system error
```
**EN:** This block implements the async method `_generate_chat_stream(adapted_request, request, raw_request)` on `OpenAIServingChat`. It focuses on Generate streaming chat completion response, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的异步方法 `_generate_chat_stream(adapted_request, request, raw_request)`。它围绕 `_generate_chat_stream` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 879-914: Continue async generate chat stream / 继续说明异步generate 聊天 流式
```python
                    # (timeout, OOM, validation): emit a streaming error chunk.
                    # A graceful abort (no status_code, e.g. user-initiated via
                    # /abort_request or session lifecycle cleanup) falls through
                    # to the normal chunk path, matching the non-stream behavior
                    # in tokenizer_manager._handle_abort_finish_reason.
                    if finish_reason_type == "abort" and isinstance(
                        finish_reason.get("status_code"), HTTPStatus
                    ):
                        code = finish_reason["status_code"]
                        error = self.create_streaming_error_response(
                            finish_reason.get("message", "Generation aborted."),
                            code.name,
                            code.value,
                        )
                        yield f"data: {error}\n\n"
                        break
                    finish_reasons[index] = finish_reason

                # First chunk with role
                if is_firsts.get(index, True):
                    is_firsts[index] = False
                    yield _fast_sse_content(
                        chunk_id=content["meta_info"]["id"],
                        created=int(time.time()),
                        model=request.model,
                        index=index,
                        role="assistant",
                        content="",
                    )
                    stream_started = True

                offset = stream_offsets.get(index, 0)
                if self.tokenizer_manager.server_args.incremental_streaming_output:
                    # content["text"] is already the incremental delta
                    delta = content["text"]
                else:
```
**EN:** This block implements the async method `_generate_chat_stream(adapted_request, request, raw_request)` on `OpenAIServingChat`. It focuses on Generate streaming chat completion response, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的异步方法 `_generate_chat_stream(adapted_request, request, raw_request)`。它围绕 `_generate_chat_stream` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 915-950: Continue async generate chat stream / 继续说明异步generate 聊天 流式
```python
                    delta = content["text"][offset:]
                stream_offsets[index] = len(content["text"])

                # Handle reasoning content
                if self.reasoning_parser and request.separate_reasoning:
                    reasoning_text, delta = self._process_reasoning_stream(
                        index, delta, reasoning_parser_dict, content, request
                    )
                    if reasoning_text:
                        usage = None
                        if continuous_usage_stats:
                            usage = UsageProcessor.calculate_token_usage(
                                prompt_tokens=prompt_tokens.get(index, 0),
                                reasoning_tokens=reasoning_tokens.get(index, 0),
                                completion_tokens=completion_tokens.get(index, 0),
                            ).model_dump()

                        yield _fast_sse_content(
                            chunk_id=content["meta_info"]["id"],
                            created=int(time.time()),
                            model=request.model,
                            index=index,
                            reasoning_content=reasoning_text,
                            usage=usage,
                        )

                # Handle tool calls
                if (
                    request.tool_choice != "none"
                    and request.tools
                    and self.tool_call_parser
                ):
                    async for chunk in self._process_tool_call_stream(
                        index,
                        delta,
                        parser_dict,
```
**EN:** This block implements the async method `_generate_chat_stream(adapted_request, request, raw_request)` on `OpenAIServingChat`. It focuses on Generate streaming chat completion response, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的异步方法 `_generate_chat_stream(adapted_request, request, raw_request)`。它围绕 `_generate_chat_stream` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 951-986: Continue async generate chat stream / 继续说明异步generate 聊天 流式
```python
                        content,
                        request,
                        has_tool_calls,
                        continuous_usage_stats,
                    ):
                        if chunk:
                            yield chunk

                    # Send any remaining tool call arguments when generation finishes
                    if finish_reason_type is not None and index in parser_dict:
                        parser = parser_dict[index]
                        remaining_chunk = self._check_for_unstreamed_tool_args(
                            parser, content, request, index
                        )
                        if remaining_chunk:
                            yield remaining_chunk

                else:
                    # Regular content
                    if delta:
                        usage = None
                        if continuous_usage_stats:
                            usage = UsageProcessor.calculate_token_usage(
                                prompt_tokens=prompt_tokens.get(index, 0),
                                reasoning_tokens=reasoning_tokens.get(index, 0),
                                completion_tokens=completion_tokens.get(index, 0),
                            ).model_dump()

                        yield _fast_sse_content(
                            chunk_id=content["meta_info"]["id"],
                            created=int(time.time()),
                            model=request.model,
                            index=index,
                            content=delta,
                            logprobs=choice_logprobs,
                            usage=usage,
```
**EN:** This block implements the async method `_generate_chat_stream(adapted_request, request, raw_request)` on `OpenAIServingChat`. It focuses on Generate streaming chat completion response, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的异步方法 `_generate_chat_stream(adapted_request, request, raw_request)`。它围绕 `_generate_chat_stream` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 987-1022: Continue async generate chat stream / 继续说明异步generate 聊天 流式
```python
                        )

            # Send finish_reason chunks for each index that completed
            for idx, finish_reason_data in finish_reasons.items():
                finish_reason_type = finish_reason_data["type"]

                # Change finish_reason to "tool_calls" if we had tool calls and stopped naturally
                final_finish_reason = finish_reason_type
                if has_tool_calls.get(idx, False) and finish_reason_type == "stop":
                    final_finish_reason = "tool_calls"

                matched_stop = finish_reason_data.get("matched")
                yield _fast_sse_content(
                    chunk_id=content["meta_info"]["id"],
                    created=int(time.time()),
                    model=request.model,
                    index=idx,
                    finish_reason=final_finish_reason,
                    matched_stop=matched_stop,
                )

            # Send hidden states if requested
            if request.return_hidden_states and hidden_states:
                for index, choice_hidden_states in hidden_states.items():
                    if choice_hidden_states:
                        last_token_hidden_states = (
                            choice_hidden_states[-1]
                            if len(choice_hidden_states) > 1
                            else []
                        )
                        hidden_states_chunk = ChatCompletionStreamResponse(
                            id=content["meta_info"]["id"],
                            created=int(time.time()),
                            choices=[
                                ChatCompletionResponseStreamChoice(
                                    index=index,
```
**EN:** This block implements the async method `_generate_chat_stream(adapted_request, request, raw_request)` on `OpenAIServingChat`. It focuses on Generate streaming chat completion response, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的异步方法 `_generate_chat_stream(adapted_request, request, raw_request)`。它围绕 `_generate_chat_stream` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1023-1058: Continue async generate chat stream / 继续说明异步generate 聊天 流式
```python
                                    delta=DeltaMessage(
                                        hidden_states=last_token_hidden_states
                                    ),
                                    finish_reason=None,  # Hidden states don't need finish_reason
                                )
                            ],
                            model=request.model,
                        )
                        yield f"data: {hidden_states_chunk.model_dump_json()}\n\n"

            sglext_routed = None
            if request.return_routed_experts and routed_experts:
                sglext_routed = next(
                    (v for v in routed_experts.values() if v is not None), None
                )

            sglext_details = None
            if request.return_cached_tokens_details and cached_tokens_details:
                first_details = next(
                    (v for v in cached_tokens_details.values() if v is not None), None
                )
                if first_details is not None:
                    sglext_details = cached_tokens_details_from_dict(first_details)

            if sglext_routed is not None or sglext_details is not None:
                sglext_chunk = ChatCompletionStreamResponse(
                    id=content["meta_info"]["id"],
                    created=int(time.time()),
                    choices=[],  # sglext is at response level
                    model=request.model,
                    sglext=SglExt(
                        routed_experts=sglext_routed,
                        cached_tokens_details=sglext_details,
                    ),
                )
                yield f"data: {sglext_chunk.model_dump_json()}\n\n"
```
**EN:** This block implements the async method `_generate_chat_stream(adapted_request, request, raw_request)` on `OpenAIServingChat`. It focuses on Generate streaming chat completion response, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的异步方法 `_generate_chat_stream(adapted_request, request, raw_request)`。它围绕 `_generate_chat_stream` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1059-1085: Continue async generate chat stream / 继续说明异步generate 聊天 流式
```python

            # Additional usage chunk
            if include_usage:
                usage = UsageProcessor.calculate_streaming_usage(
                    prompt_tokens,
                    reasoning_tokens,
                    completion_tokens,
                    cached_tokens=cached_tokens,
                    n_choices=request.n,
                    enable_cache_report=self.tokenizer_manager.server_args.enable_cache_report,
                )
                usage_chunk = ChatCompletionStreamResponse(
                    id=content["meta_info"]["id"],
                    created=int(time.time()),
                    choices=[],  # Empty choices array as per OpenAI spec
                    model=request.model,
                    usage=usage,
                )
                yield f"data: {usage_chunk.model_dump_json()}\n\n"

        except ValueError as e:
            if not stream_started:
                raise
            error = self.create_streaming_error_response(str(e))
            yield f"data: {error}\n\n"

        yield "data: [DONE]\n\n"
```
**EN:** This block implements the async method `_generate_chat_stream(adapted_request, request, raw_request)` on `OpenAIServingChat`. It focuses on Generate streaming chat completion response, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的异步方法 `_generate_chat_stream(adapted_request, request, raw_request)`。它围绕 `_generate_chat_stream` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1087-1110: Implement async handle non streaming request / 实现异步handle non streaming 请求
```python
    async def _handle_non_streaming_request(
        self,
        adapted_request: GenerateReqInput,
        request: ChatCompletionRequest,
        raw_request: Request,
    ) -> Union[ChatCompletionResponse, ErrorResponse, ORJSONResponse]:
        """Handle non-streaming chat completion request"""
        try:
            ret = await self.tokenizer_manager.generate_request(
                adapted_request, raw_request
            ).__anext__()
        except ValueError as e:
            return self.create_error_response(str(e))

        if not isinstance(ret, list):
            ret = [ret]

        response = self._build_chat_response(
            request,
            ret,
            int(time.time()),
        )

        return response
```
**EN:** This block implements the async method `_handle_non_streaming_request(adapted_request, request, raw_request)` on `OpenAIServingChat`. It focuses on Handle non-streaming chat completion request, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的异步方法 `_handle_non_streaming_request(adapted_request, request, raw_request)`。它围绕 `_handle_non_streaming_request` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1112-1147: Implement build chat response / 实现build 聊天 响应
```python
    def _build_chat_response(
        self,
        request: ChatCompletionRequest,
        ret: List[Dict[str, Any]],
        created: int,
    ) -> Union[ChatCompletionResponse, ORJSONResponse]:
        """Build chat completion response from generation results"""
        choices = []

        # Build sglext at response level (from first ret_item, as these are per-request)
        first_ret = ret[0]
        routed_experts = process_routed_experts_from_ret(first_ret, request)
        cached_tokens_details = process_cached_tokens_details_from_ret(
            first_ret, request
        )
        response_sglext = None
        if routed_experts or cached_tokens_details:
            response_sglext = SglExt(
                routed_experts=routed_experts,
                cached_tokens_details=cached_tokens_details,
            )

        for idx, ret_item in enumerate(ret):
            # Process logprobs
            choice_logprobs = None
            if request.logprobs:
                choice_logprobs = self._process_response_logprobs(ret_item)

            # Handle hidden states
            hidden_states = process_hidden_states_from_ret(ret_item, request)

            finish_reason = ret_item["meta_info"]["finish_reason"]
            text = ret_item["text"]

            # Handle reasoning content
            reasoning_text = None
```
**EN:** This block implements the method `_build_chat_response(request, ret, created)` on `OpenAIServingChat`. It focuses on Build chat completion response from generation results, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_build_chat_response(request, ret, created)`。它围绕 `_build_chat_response` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1148-1183: Continue build chat response / 继续说明build 聊天 响应
```python
            reasoning_parser = self.reasoning_parser
            if reasoning_parser and request.separate_reasoning:
                is_force_reasoning = (
                    self.template_manager.force_reasoning
                    or self._get_reasoning_from_request(request)
                )
                try:
                    parser = ReasoningParser(
                        model_type=reasoning_parser,
                        stream_reasoning=False,
                        force_reasoning=is_force_reasoning,
                        request=request,
                    )
                    reasoning_text, text = parser.parse_non_stream(text)
                except Exception as e:
                    logger.error(f"Reasoning parsing error: {e}")
                    return self.create_error_response(
                        "Failed to parse reasoning content",
                        err_type="InternalServerError",
                        status_code=500,
                    )

            # Handle tool calls
            tool_calls = None
            if (
                request.tool_choice != "none"
                and request.tools
                and self.tool_call_parser
            ):
                history_tool_calls_cnt = self._get_history_tool_calls_cnt(request)
                tool_calls, text, finish_reason = self._process_tool_calls(
                    text,
                    request.tools,
                    finish_reason,
                    request.tool_choice,
                    history_tool_calls_cnt,
```
**EN:** This block implements the method `_build_chat_response(request, ret, created)` on `OpenAIServingChat`. It focuses on Build chat completion response from generation results, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_build_chat_response(request, ret, created)`。它围绕 `_build_chat_response` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1184-1219: Continue build chat response / 继续说明build 聊天 响应
```python
                )

            choice_data = ChatCompletionResponseChoice(
                index=idx,
                message=ChatMessage(
                    role="assistant",
                    content=text if text else None,
                    tool_calls=tool_calls,
                    reasoning_content=reasoning_text if reasoning_text else None,
                ),
                logprobs=choice_logprobs,
                finish_reason=finish_reason["type"] if finish_reason else None,
                matched_stop=(
                    finish_reason["matched"]
                    if finish_reason and "matched" in finish_reason
                    else None
                ),
                hidden_states=hidden_states,
            )
            choices.append(choice_data)

        # Calculate usage
        usage = UsageProcessor.calculate_response_usage(
            ret,
            n_choices=request.n,
            enable_cache_report=self.tokenizer_manager.server_args.enable_cache_report,
        )

        return ChatCompletionResponse(
            id=ret[0]["meta_info"]["id"],
            created=created,
            model=request.model,
            choices=choices,
            usage=usage,
            metadata={"weight_version": ret[0]["meta_info"]["weight_version"]},
            sglext=response_sglext,
```
**EN:** This block implements the method `_build_chat_response(request, ret, created)` on `OpenAIServingChat`. It focuses on Build chat completion response from generation results, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_build_chat_response(request, ret, created)`。它围绕 `_build_chat_response` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1220-1220: Continue build chat response / 继续说明build 聊天 响应
```python
        )
```
**EN:** This block implements the method `_build_chat_response(request, ret, created)` on `OpenAIServingChat`. It focuses on Build chat completion response from generation results, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_build_chat_response(request, ret, created)`。它围绕 `_build_chat_response` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1222-1262: Implement process logprobs tokens / 实现process logprobs tokens
```python
    def _process_logprobs_tokens(
        self, logprobs: LogProbs, use_token_index: bool = False
    ) -> List[ChatCompletionTokenLogprob]:
        """Common helper to process logprobs tokens for both streaming and non-streaming

        Args:
            logprobs: LogProbs data from model
            use_token_index: True for non-streaming (use token_idx), False for streaming (use index 0)
        """
        token_logprobs = []

        for token_idx, (token, logprob) in enumerate(
            zip(logprobs.tokens, logprobs.token_logprobs)
        ):
            token_bytes = list(token.encode("utf-8"))
            top_logprobs = []
            if logprobs.top_logprobs:
                # - Non-streaming (use_token_index=True): uses token_idx for full data
                # - Streaming (use_token_index=False): uses index 0 for pre-sliced data
                top_logprobs_idx = token_idx if use_token_index else 0
                for top_token, top_logprob in logprobs.top_logprobs[
                    top_logprobs_idx
                ].items():
                    top_token_bytes = list(top_token.encode("utf-8"))
                    top_logprobs.append(
                        TopLogprob(
                            token=top_token,
                            bytes=top_token_bytes,
                            logprob=top_logprob,
                        )
                    )
            token_logprobs.append(
                ChatCompletionTokenLogprob(
                    token=token,
                    bytes=token_bytes,
                    logprob=logprob,
                    top_logprobs=top_logprobs,
                )
            )

        return token_logprobs
```
**EN:** This block implements the method `_process_logprobs_tokens(logprobs, use_token_index)` on `OpenAIServingChat`. It focuses on Common helper to process logprobs tokens for both streaming and non-streaming Args: logprobs: LogProbs data from model use_token_index: True for non-streaming (use token_idx), False for streaming (use index 0), so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_process_logprobs_tokens(logprobs, use_token_index)`。它围绕 `_process_logprobs_tokens` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1264-1272: Implement process response logprobs / 实现process 响应 logprobs
```python
    def _process_response_logprobs(self, ret_item: Dict[str, Any]) -> ChoiceLogprobs:
        """Process logprobs for non-streaming response"""
        logprobs = to_openai_style_logprobs(
            output_token_logprobs=ret_item["meta_info"]["output_token_logprobs"],
            output_top_logprobs=ret_item["meta_info"].get("output_top_logprobs", None),
        )

        token_logprobs = self._process_logprobs_tokens(logprobs, use_token_index=True)
        return ChoiceLogprobs(content=token_logprobs)
```
**EN:** This block implements the method `_process_response_logprobs(ret_item)` on `OpenAIServingChat`. It focuses on Process logprobs for non-streaming response, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_process_response_logprobs(ret_item)`。它围绕 `_process_response_logprobs` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1274-1292: Implement process tool call id / 实现process 工具 call id
```python
    def _process_tool_call_id(
        self,
        call_item: ToolCallItem,
        history_tool_calls_cnt: int,
    ) -> str:
        """Process for generating a new and unique `tool_call_id`"""
        if self.tool_call_parser != "kimi_k2":
            # A simple uuid is sufficient for all models except for Kimi-K2.
            tool_call_id = f"call_{uuid.uuid4().hex[:24]}"
            return tool_call_id
        else:
            # Align with Kimi-K2 format: functions.{name}:{index}
            # Kimi-K2 allows multiple tool_calls in one message; SGLang sets call_item.tool_index to the *local* position inside that message.
            # Therefore, the index must be corrected by using `history_tool_calls_cnt + call_item.tool_index` to ensure globally unique and properly ordered.
            tool_call_id = f"functions.{call_item.name}:{history_tool_calls_cnt+call_item.tool_index}"
            logger.debug(
                f"Process tool call idx, parser: {self.tool_call_parser}, tool_call_id: {tool_call_id}, history_cnt: {history_tool_calls_cnt}"
            )
            return tool_call_id
```
**EN:** This block implements the method `_process_tool_call_id(call_item, history_tool_calls_cnt)` on `OpenAIServingChat`. It focuses on Process for generating a new and unique `tool_call_id`, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_process_tool_call_id(call_item, history_tool_calls_cnt)`。它围绕 `_process_tool_call_id` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1294-1329: Implement process tool calls / 实现process 工具 calls
```python
    def _process_tool_calls(
        self,
        text: str,
        tools: List[Any],
        finish_reason: Dict[str, Any],
        tool_choice: Optional[Union[str, ToolChoice]] = None,
        history_tool_calls_cnt: int = 0,
    ) -> ToolCallProcessingResult:
        """Process tool calls in the response"""

        is_required = tool_choice == "required" or isinstance(tool_choice, ToolChoice)

        # Try model-specific parser when output is in native format.
        # For required/named: only use parser when structural_tag was used
        # as constraint (mirrors the streaming path). For auto: always try.
        if self.tool_call_parser:
            parser = FunctionCallParser(tools, self.tool_call_parser)
            should_try_parser = (
                not is_required or parser.detector.supports_structural_tag()
            )
            if should_try_parser and parser.has_tool_call(text):
                original_finish_type = finish_reason["type"]
                if finish_reason["type"] == "stop":
                    finish_reason["type"] = "tool_calls"
                    finish_reason["matched"] = None
                try:
                    text, call_info_list = parser.parse_non_stream(text)
                    tool_calls = []
                    for call_info in call_info_list:
                        tool_id = self._process_tool_call_id(
                            call_info, history_tool_calls_cnt
                        )
                        tool_calls.append(
                            ToolCall(
                                id=tool_id,
                                index=getattr(call_info, "tool_index", None),
```
**EN:** This block implements the method `_process_tool_calls(text, tools, finish_reason, tool_choice, history_tool_calls_cnt)` on `OpenAIServingChat`. It focuses on Process tool calls in the response, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_process_tool_calls(text, tools, finish_reason, tool_choice, history_tool_calls_cnt)`。它围绕 `_process_tool_calls` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1330-1365: Continue process tool calls / 继续说明process 工具 calls
```python
                                function=FunctionResponse(
                                    name=call_info.name,
                                    arguments=call_info.parameters,
                                ),
                            )
                        )
                    return ToolCallProcessingResult(tool_calls, text, finish_reason)
                except Exception as e:
                    logger.error(f"Tool call parsing error: {e}")
                    finish_reason["type"] = original_finish_type
                    return ToolCallProcessingResult(None, text, finish_reason)

        # json_schema constraint → JSON array output for required/named
        if is_required:
            original_finish_type = finish_reason["type"]
            if finish_reason["type"] == "stop":
                finish_reason["type"] = "tool_calls"
                finish_reason["matched"] = None
            try:
                tool_call_data = orjson.loads(text)
                tool_calls = []
                for i, tool in enumerate(tool_call_data):
                    call_info = ToolCallItem(
                        tool_index=i,
                        name=tool["name"],
                        parameters=json.dumps(tool["parameters"], ensure_ascii=False),
                    )
                    tool_id = self._process_tool_call_id(
                        call_info, history_tool_calls_cnt
                    )
                    tool_calls.append(
                        ToolCall(
                            id=tool_id,
                            index=i,
                            function=FunctionResponse(
                                name=tool["name"],
```
**EN:** This block implements the method `_process_tool_calls(text, tools, finish_reason, tool_choice, history_tool_calls_cnt)` on `OpenAIServingChat`. It focuses on Process tool calls in the response, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_process_tool_calls(text, tools, finish_reason, tool_choice, history_tool_calls_cnt)`。它围绕 `_process_tool_calls` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1366-1378: Continue process tool calls / 继续说明process 工具 calls
```python
                                arguments=json.dumps(
                                    tool["parameters"], ensure_ascii=False
                                ),
                            ),
                        )
                    )
                return ToolCallProcessingResult(tool_calls, "", finish_reason)
            except Exception as e:
                logger.error(f"Tool call parsing error: {e}")
                finish_reason["type"] = original_finish_type
                return ToolCallProcessingResult(None, text, finish_reason)

        return ToolCallProcessingResult(None, text, finish_reason)
```
**EN:** This block implements the method `_process_tool_calls(text, tools, finish_reason, tool_choice, history_tool_calls_cnt)` on `OpenAIServingChat`. It focuses on Process tool calls in the response, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_process_tool_calls(text, tools, finish_reason, tool_choice, history_tool_calls_cnt)`。它围绕 `_process_tool_calls` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1380-1402: Implement process streaming logprobs / 实现process streaming logprobs
```python
    def _process_streaming_logprobs(
        self,
        content: Dict[str, Any],
        n_prev_token: int,
        total_output_logprobs: int,
    ) -> ChoiceLogprobs:
        """Process logprobs for streaming response"""
        output_token_logprobs = content["meta_info"]["output_token_logprobs"]
        output_top_logprobs = content["meta_info"].get("output_top_logprobs", [])
        if not self.tokenizer_manager.server_args.incremental_streaming_output:
            output_token_logprobs = output_token_logprobs[
                n_prev_token:total_output_logprobs
            ]
            output_top_logprobs = output_top_logprobs[
                n_prev_token:total_output_logprobs
            ]
        logprobs = to_openai_style_logprobs(
            output_token_logprobs=output_token_logprobs,
            output_top_logprobs=output_top_logprobs,
        )

        token_logprobs = self._process_logprobs_tokens(logprobs, use_token_index=False)
        return ChoiceLogprobs(content=token_logprobs)
```
**EN:** This block implements the method `_process_streaming_logprobs(content, n_prev_token, total_output_logprobs)` on `OpenAIServingChat`. It focuses on Process logprobs for streaming response, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_process_streaming_logprobs(content, n_prev_token, total_output_logprobs)`。它围绕 `_process_streaming_logprobs` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1404-1425: Implement process reasoning stream / 实现process reasoning 流式
```python
    def _process_reasoning_stream(
        self,
        index: int,
        delta: str,
        reasoning_parser_dict: Dict[int, ReasoningParser],
        content: Dict[str, Any],
        request: ChatCompletionRequest,
    ) -> tuple[Optional[str], str]:
        """Process reasoning content in streaming response"""
        if index not in reasoning_parser_dict:
            is_force_reasoning = (
                self.template_manager.force_reasoning
                or self._get_reasoning_from_request(request)
            )
            reasoning_parser_dict[index] = ReasoningParser(
                self.reasoning_parser,
                request.stream_reasoning,
                is_force_reasoning,
                request,
            )
        reasoning_parser = reasoning_parser_dict[index]
        return reasoning_parser.parse_stream_chunk(delta)
```
**EN:** This block implements the method `_process_reasoning_stream(index, delta, reasoning_parser_dict, content, request)` on `OpenAIServingChat`. It focuses on Process reasoning content in streaming response, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_process_reasoning_stream(index, delta, reasoning_parser_dict, content, request)`。它围绕 `_process_reasoning_stream` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1427-1445: Implement get history tool calls cnt / 实现get history 工具 calls cnt
```python
    def _get_history_tool_calls_cnt(self, request: ChatCompletionRequest) -> int:
        """Counts the number of tool calls in the request's message history.

        NOTE: This method is only useful for models that include self-increasing
        history tool call idx in tool calls id, such as kimi-k2

        Args:
            request: The chat completion request object.

        Returns:
            The total number of tool calls in the history, or 0 if not applicable.
        """
        messages = getattr(request, "messages", [])
        idx = 0
        for msg in messages:
            if msg.role == "assistant":
                tool_calls = getattr(msg, "tool_calls", None)
                idx += len(list(tool_calls)) if tool_calls is not None else 0  # noqa
        return idx
```
**EN:** This block implements the method `_get_history_tool_calls_cnt(request)` on `OpenAIServingChat`. It focuses on Counts the number of tool calls in the request's message history., so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_get_history_tool_calls_cnt(request)`。它围绕 `_get_history_tool_calls_cnt` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1447-1457: Implement patch mistral skip special tokens / 实现patch mistral skip special tokens
```python
    def _patch_mistral_skip_special_tokens(
        self, request: ChatCompletionRequest
    ) -> None:
        """Mistral uses special tokens ([THINK]/[/THINK]) for reasoning markers,
        which get stripped when skip_special_tokens=True."""
        if (
            self.reasoning_parser in ["mistral"]
            and request.reasoning_effort is not None
            and request.reasoning_effort != "none"
        ):
            request.skip_special_tokens = False
```
**EN:** This block implements the method `_patch_mistral_skip_special_tokens(request)` on `OpenAIServingChat`. It focuses on Mistral uses special tokens ([THINK]/[/THINK]) for reasoning markers, which get stripped when skip_special_tokens=True., so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_patch_mistral_skip_special_tokens(request)`。它围绕 `_patch_mistral_skip_special_tokens` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1459-1494: Implement get reasoning from request / 实现get reasoning from 请求
```python
    def _get_reasoning_from_request(self, request: ChatCompletionRequest) -> bool:
        """Determine whether reasoning mode should be enabled for this request.

        NOTE: This is predefined based on model's chat template
        """
        if not self.reasoning_parser:
            return False

        if self.reasoning_parser == "hunyuan":
            # Hy3-preview template emits no <think> when reasoning_effort is
            # "no_think" / "none" / unset; forcing reasoning would route all
            # output into reasoning_content.
            return request.reasoning_effort not in (None, "none", "no_think")

        config = self.template_manager.reasoning_config
        if config is None:
            # Fallback to parser-level defaults when template toggle config
            # cannot be inferred (e.g., parser-only <think> templates).
            mode = (
                self._reasoning_detector.reasoning_default
                if self._reasoning_detector is not None
                else None
            )
            if mode is None:
                return False
            if mode == "always":
                return True
            if mode == "mistral":
                return (
                    request.reasoning_effort is not None
                    and request.reasoning_effort != "none"
                )
            if mode in ("thinking", "enable_thinking"):
                return (
                    not request.chat_template_kwargs
                    or request.chat_template_kwargs.get(mode) is not False
```
**EN:** This block implements the method `_get_reasoning_from_request(request)` on `OpenAIServingChat`. It focuses on Determine whether reasoning mode should be enabled for this request., so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_get_reasoning_from_request(request)`。它围绕 `_get_reasoning_from_request` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1495-1528: Continue get reasoning from request / 继续说明get reasoning from 请求
```python
                )
            if mode in ("explicit_thinking", "explicit_enable_thinking"):
                toggle = mode.replace("explicit_", "")
                return (
                    request.chat_template_kwargs is not None
                    and request.chat_template_kwargs.get(toggle) is True
                )
            logger.warning(
                "Unknown reasoning_default mode '%s', defaulting to reasoning disabled",
                mode,
            )
            return False

        if config.special_case == "always":
            return True

        if config.special_case == "mistral":
            return (
                request.reasoning_effort is not None
                and request.reasoning_effort != "none"
            )

        if config.toggle_param is None or config.default_enabled is None:
            return False

        if config.default_enabled:
            return (
                not request.chat_template_kwargs
                or request.chat_template_kwargs.get(config.toggle_param) is not False
            )
        return (
            request.chat_template_kwargs is not None
            and request.chat_template_kwargs.get(config.toggle_param) is True
        )
```
**EN:** This block implements the method `_get_reasoning_from_request(request)` on `OpenAIServingChat`. It focuses on Determine whether reasoning mode should be enabled for this request., so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_get_reasoning_from_request(request)`。它围绕 `_get_reasoning_from_request` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1530-1565: Implement async process tool call stream / 实现异步process 工具 call 流式
```python
    async def _process_tool_call_stream(
        self,
        index: int,
        delta: str,
        parser_dict: Dict[int, FunctionCallParser],
        content: Dict[str, Any],
        request: ChatCompletionRequest,
        has_tool_calls: Dict[int, bool],
        continuous_usage_stats: bool = False,
    ):
        """Process tool calls in streaming response"""
        if index not in parser_dict:
            is_required = request.tool_choice == "required" or isinstance(
                request.tool_choice, ToolChoice
            )
            # For required/named tool choice: use JsonArrayParser when the
            # constrained output is plain JSON (detector doesn't support
            # structural_tag or no parser configured). Use FunctionCallParser
            # only when the detector supports structural_tag and will produce
            # native format output.
            if is_required:
                use_native_parser = False
                if self.tool_call_parser:
                    probe = FunctionCallParser(
                        tools=request.tools,
                        tool_call_parser=self.tool_call_parser,
                    )
                    use_native_parser = probe.detector.supports_structural_tag()
                if use_native_parser:
                    parser_dict[index] = probe
                else:
                    parser_dict[index] = JsonArrayParser()
            else:
                parser_dict[index] = FunctionCallParser(
                    tools=request.tools,
                    tool_call_parser=self.tool_call_parser,
```
**EN:** This block implements the async method `_process_tool_call_stream(index, delta, parser_dict, content, request, ...)` on `OpenAIServingChat`. It focuses on Process tool calls in streaming response, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的异步方法 `_process_tool_call_stream(index, delta, parser_dict, content, request, ...)`。它围绕 `_process_tool_call_stream` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1566-1601: Continue async process tool call stream / 继续说明异步process 工具 call 流式
```python
                )

        parser = parser_dict[index]

        # Handle both FunctionCallParser and JsonArrayParser
        if isinstance(parser, JsonArrayParser):
            result = parser.parse_streaming_increment(delta, request.tools)
            normal_text, calls = result.normal_text, result.calls
        else:
            normal_text, calls = parser.parse_stream_chunk(delta)

        # Yield normal text
        if normal_text:
            choice_data = ChatCompletionResponseStreamChoice(
                index=index,
                delta=DeltaMessage(content=normal_text),
                finish_reason=None,
            )
            chunk = ChatCompletionStreamResponse(
                id=content["meta_info"]["id"],
                created=int(time.time()),
                choices=[choice_data],
                model=request.model,
            )

            # Add usage stats if continuous_usage_stats is enabled
            if continuous_usage_stats:
                prompt_tokens = content["meta_info"].get("prompt_tokens", 0)
                completion_tokens = content["meta_info"].get("completion_tokens", 0)
                reasoning_tokens = content["meta_info"].get("reasoning_tokens", 0)
                chunk.usage = UsageProcessor.calculate_token_usage(
                    prompt_tokens=prompt_tokens,
                    completion_tokens=completion_tokens,
                    reasoning_tokens=reasoning_tokens,
                )
```
**EN:** This block implements the async method `_process_tool_call_stream(index, delta, parser_dict, content, request, ...)` on `OpenAIServingChat`. It focuses on Process tool calls in streaming response, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的异步方法 `_process_tool_call_stream(index, delta, parser_dict, content, request, ...)`。它围绕 `_process_tool_call_stream` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1602-1637: Continue async process tool call stream / 继续说明异步process 工具 call 流式
```python
            yield f"data: {chunk.model_dump_json()}\n\n"

        # Yield tool calls
        history_tool_calls_cnt = self._get_history_tool_calls_cnt(request)
        for call_item in calls:
            # Mark that this choice has tool calls
            has_tool_calls[index] = True

            # Tool call ID should be generated only once per tool call
            if call_item.name:
                # First chunk: include ID and function name
                tool_call_id = self._process_tool_call_id(
                    call_item, history_tool_calls_cnt
                )
                function_name = call_item.name
            else:
                # Subsequent chunks: null ID and name for argument deltas
                tool_call_id = None
                function_name = None

            tool_call = ToolCall(
                id=tool_call_id,
                index=call_item.tool_index,
                function=FunctionResponse(
                    name=function_name,
                    arguments=call_item.parameters,
                ),
            )

            choice_data = ChatCompletionResponseStreamChoice(
                index=index,
                delta=DeltaMessage(tool_calls=[tool_call]),
                finish_reason=None,
            )
            chunk = ChatCompletionStreamResponse(
                id=content["meta_info"]["id"],
```
**EN:** This block implements the async method `_process_tool_call_stream(index, delta, parser_dict, content, request, ...)` on `OpenAIServingChat`. It focuses on Process tool calls in streaming response, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的异步方法 `_process_tool_call_stream(index, delta, parser_dict, content, request, ...)`。它围绕 `_process_tool_call_stream` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1638-1654: Continue async process tool call stream / 继续说明异步process 工具 call 流式
```python
                created=int(time.time()),
                choices=[choice_data],
                model=request.model,
            )

            # Add usage stats if continuous_usage_stats is enabled
            if continuous_usage_stats:
                prompt_tokens = content["meta_info"].get("prompt_tokens", 0)
                completion_tokens = content["meta_info"].get("completion_tokens", 0)
                reasoning_tokens = content["meta_info"].get("reasoning_tokens", 0)
                chunk.usage = UsageProcessor.calculate_token_usage(
                    prompt_tokens=prompt_tokens,
                    completion_tokens=completion_tokens,
                    reasoning_tokens=reasoning_tokens,
                )

            yield f"data: {chunk.model_dump_json()}\n\n"
```
**EN:** This block implements the async method `_process_tool_call_stream(index, delta, parser_dict, content, request, ...)` on `OpenAIServingChat`. It focuses on Process tool calls in streaming response, so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的异步方法 `_process_tool_call_stream(index, delta, parser_dict, content, request, ...)`。它围绕 `_process_tool_call_stream` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1656-1691: Implement check for unstreamed tool args / 实现检查 for unstreamed 工具 args
```python
    def _check_for_unstreamed_tool_args(
        self,
        parser: Union[FunctionCallParser, JsonArrayParser],
        content: Dict[str, Any],
        request: ChatCompletionRequest,
        index: int,
    ) -> Optional[str]:
        """
        Check for any remaining tool call arguments that need to be streamed
        when generation finishes. This ensures tool calls are properly completed
        even if the model generates the final arguments in the last chunk.
        """
        # Get the detector - either from FunctionCallParser or directly if json detector
        detector = parser.detector if hasattr(parser, "detector") else parser

        # Only check if we have tool calls and the detector has tracked data
        if (
            not hasattr(detector, "prev_tool_call_arr")
            or not detector.prev_tool_call_arr
        ):
            return None

        if (
            not hasattr(detector, "streamed_args_for_tool")
            or not detector.streamed_args_for_tool
        ):
            return None

        # Get the last tool call that was being processed
        tool_index = len(detector.prev_tool_call_arr) - 1
        if tool_index < 0 or tool_index >= len(detector.streamed_args_for_tool):
            return None

        # Get expected vs actual arguments
        expected_args = detector.prev_tool_call_arr[tool_index].get("arguments", {})
        expected_call = json.dumps(expected_args, ensure_ascii=False)
```
**EN:** This block implements the method `_check_for_unstreamed_tool_args(parser, content, request, index)` on `OpenAIServingChat`. It focuses on Check for any remaining tool call arguments that need to be streamed when generation finishes., so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_check_for_unstreamed_tool_args(parser, content, request, index)`。它围绕 `_check_for_unstreamed_tool_args` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

### Lines 1692-1727: Continue check for unstreamed tool args / 继续说明检查 for unstreamed 工具 args
```python
        actual_call = detector.streamed_args_for_tool[tool_index]

        # Check if there are remaining arguments to send
        remaining_call = (
            expected_call.replace(actual_call, "", 1)
            if actual_call in expected_call
            else ""
        )

        if remaining_call:
            # Create tool call chunk with remaining arguments
            tool_call = ToolCall(
                id=None,  # No ID for argument deltas
                index=tool_index,
                function=FunctionResponse(
                    name=None,  # No name for argument deltas
                    arguments=remaining_call,
                ),
            )

            choice_data = ChatCompletionResponseStreamChoice(
                index=index,
                delta=DeltaMessage(tool_calls=[tool_call]),
                finish_reason=None,  # Don't send finish_reason with this chunk
            )

            chunk = ChatCompletionStreamResponse(
                id=content["meta_info"]["id"],
                created=int(time.time()),
                choices=[choice_data],
                model=request.model,
            )

            return f"data: {chunk.model_dump_json()}\n\n"

        return None
```
**EN:** This block implements the method `_check_for_unstreamed_tool_args(parser, content, request, index)` on `OpenAIServingChat`. It focuses on Check for any remaining tool call arguments that need to be streamed when generation finishes., so the class can advance the serving chat workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingChat` 上的方法 `_check_for_unstreamed_tool_args(parser, content, request, index)`。它围绕 `_check_for_unstreamed_tool_args` 所承担的 serving 聊天 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: _StreamDelta, _StreamChoice, _StreamChunk, OpenAIServingChat
- **Main callables / 主要可调用对象**: _fast_sse_content, normalize_tool_content, _extract_max_dynamic_patch
- **Domain focus / 领域焦点**: serving chat / serving 聊天
- **Concurrency / 并发特征**: async/await appears throughout the module / 模块中多处使用 async/await 协调并发流程

## Dependencies / 依赖关系
- **Standard Library / 标准库**: copy, json, logging, time, typing, uuid
- **Third-party / 第三方库**: __future__, fastapi, fastapi.responses, http, jinja2, jsonschema, msgspec, orjson
- **Local Modules / 本地模块**: sglang.srt.entrypoints.openai, sglang.srt.entrypoints.openai.protocol, sglang.srt.entrypoints.openai.serving_base, sglang.srt.entrypoints.openai.usage_processor, sglang.srt.entrypoints.openai.utils, sglang.srt.environ, sglang.srt.function_call.core_types, sglang.srt.function_call.function_call_parser, sglang.srt.function_call.json_array_parser, sglang.srt.function_call.utils, sglang.srt.managers.io_struct, sglang.srt.managers.template_manager
