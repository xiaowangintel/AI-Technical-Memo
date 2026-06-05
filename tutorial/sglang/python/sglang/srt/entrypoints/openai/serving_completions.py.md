# serving_completions.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/openai/serving_completions.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements serving completions logic for the OpenAI-compatible serving layer. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 serving completions 相关的逻辑，并服务于 OpenAI 兼容服务层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-6: Import runtime dependencies / 导入运行时依赖
```python
import logging
import time
from http import HTTPStatus
from typing import TYPE_CHECKING, Any, AsyncGenerator, Dict, List, Optional, Union
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 8-9: Import runtime dependencies / 导入运行时依赖
```python
from fastapi import Request
from fastapi.responses import ORJSONResponse, StreamingResponse
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 11-34: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.entrypoints.openai.protocol import (
    CompletionRequest,
    CompletionResponse,
    CompletionResponseChoice,
    CompletionResponseStreamChoice,
    CompletionStreamResponse,
    ErrorResponse,
    SglExt,
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
from sglang.srt.managers.io_struct import GenerateReqInput
from sglang.srt.parser.code_completion_parser import (
    generate_completion_prompt_from_request,
)
from sglang.utils import convert_json_schema_to_str
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 36-38: Provide supporting module logic / 提供辅助模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.managers.template_manager import TemplateManager
    from sglang.srt.managers.tokenizer_manager import TokenizerManager
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 40-40: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 43-44: Provide supporting module logic / 提供辅助模块逻辑
```python
class OpenAIServingCompletion(OpenAIServingBase):
    """Handler for /v1/completion requests"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 46-52: Initialize OpenAIServingCompletion / 初始化 OpenAIServingCompletion
```python
    def __init__(
        self,
        tokenizer_manager: TokenizerManager,
        template_manager: TemplateManager,
    ):
        super().__init__(tokenizer_manager)
        self.template_manager = template_manager
```
**EN:** This block implements the initializer `__init__(tokenizer_manager, template_manager)` for `OpenAIServingCompletion`. It prepares the object state and connects the instance to the surrounding serving completions workflow.
**CN:** 该代码块实现 `OpenAIServingCompletion` 的初始化方法 `__init__(tokenizer_manager, template_manager)`。它负责准备对象状态，并把实例接入 serving completions 相关的运行流程。

### Lines 54-55: Implement request id prefix / 实现请求 id prefix
```python
    def _request_id_prefix(self) -> str:
        return "cmpl-"
```
**EN:** This block implements the method `_request_id_prefix()` on `OpenAIServingCompletion`. It focuses on handling the serving completions responsibilities represented by `_request_id_prefix`, so the class can advance the serving completions workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingCompletion` 上的方法 `_request_id_prefix()`。它围绕 `_request_id_prefix` 所承担的 serving completions 相关职责展开，使该类能够独立推进相应流程。

### Lines 57-63: Implement validate request / 实现validate 请求
```python
    def _validate_request(self, request: CompletionRequest) -> Optional[str]:
        """Validate that the input is valid."""
        prompt = request.prompt
        if not prompt or (isinstance(prompt, list) and all(not p for p in prompt)):
            return "Prompt cannot be empty"

        return None
```
**EN:** This block implements the method `_validate_request(request)` on `OpenAIServingCompletion`. It focuses on Validate that the input is valid., so the class can advance the serving completions workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingCompletion` 上的方法 `_validate_request(request)`。它围绕 `_validate_request` 所承担的 serving completions 相关职责展开，使该类能够独立推进相应流程。

### Lines 65-100: Implement convert to internal request / 实现convert to internal 请求
```python
    def _convert_to_internal_request(
        self,
        request: CompletionRequest,
        raw_request: Request = None,
    ) -> tuple[GenerateReqInput, CompletionRequest]:
        """Convert OpenAI completion request to internal format"""
        # NOTE: with openai API, the prompt's logprobs are always not computed
        if request.echo and request.logprobs:
            logger.warning(
                "Echo is not compatible with logprobs. "
                "To compute logprobs of input prompt, please use the native /generate API."
            )
        # Process prompt
        prompt = request.prompt
        if self.template_manager.completion_template_name is not None:
            prompt = generate_completion_prompt_from_request(request)

        # Set logprob start length based on echo and logprobs
        if request.echo and request.logprobs:
            logprob_start_len = 0
        else:
            logprob_start_len = -1

        # Build sampling parameters
        sampling_params = self._build_sampling_params(request)

        # Determine prompt format
        if isinstance(prompt, str) or (
            isinstance(prompt, list) and isinstance(prompt[0], str)
        ):
            prompt_kwargs = {"text": prompt}
        else:
            prompt_kwargs = {"input_ids": prompt}

        # Extract custom labels from raw request headers
        custom_labels = self.extract_custom_labels(raw_request)
```
**EN:** This block implements the method `_convert_to_internal_request(request, raw_request)` on `OpenAIServingCompletion`. It focuses on Convert OpenAI completion request to internal format, so the class can advance the serving completions workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingCompletion` 上的方法 `_convert_to_internal_request(request, raw_request)`。它围绕 `_convert_to_internal_request` 所承担的 serving completions 相关职责展开，使该类能够独立推进相应流程。

### Lines 101-135: Continue convert to internal request / 继续说明convert to internal 请求
```python

        # Extract routed_dp_rank from header (has higher priority than body)
        effective_routed_dp_rank = self.extract_routed_dp_rank_from_header(
            raw_request, request.routed_dp_rank
        )

        # Resolve LoRA adapter from model parameter or explicit lora_path
        lora_path = self._resolve_lora_path(request.model, request.lora_path)

        adapted_request = GenerateReqInput(
            **prompt_kwargs,
            sampling_params=sampling_params,
            return_logprob=request.logprobs is not None,
            top_logprobs_num=request.logprobs if request.logprobs is not None else 0,
            logprob_start_len=logprob_start_len,
            return_text_in_logprobs=True,
            stream=request.stream,
            lora_path=lora_path,
            bootstrap_host=request.bootstrap_host,
            bootstrap_port=request.bootstrap_port,
            bootstrap_room=request.bootstrap_room,
            routed_dp_rank=effective_routed_dp_rank,
            disagg_prefill_dp_rank=request.disagg_prefill_dp_rank,
            return_hidden_states=request.return_hidden_states,
            return_routed_experts=request.return_routed_experts,
            routed_experts_start_len=request.routed_experts_start_len,
            rid=request.rid,
            extra_key=self._compute_extra_key(request),
            priority=request.priority,
            routing_key=self.extract_routing_key(raw_request),
            custom_labels=custom_labels,
            custom_logit_processor=request.custom_logit_processor,
        )

        return adapted_request, request
```
**EN:** This block implements the method `_convert_to_internal_request(request, raw_request)` on `OpenAIServingCompletion`. It focuses on Convert OpenAI completion request to internal format, so the class can advance the serving completions workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingCompletion` 上的方法 `_convert_to_internal_request(request, raw_request)`。它围绕 `_convert_to_internal_request` 所承担的 serving completions 相关职责展开，使该类能够独立推进相应流程。

### Lines 137-179: Implement build sampling params / 实现build sampling params
```python
    def _build_sampling_params(self, request: CompletionRequest) -> Dict[str, Any]:
        """Build sampling parameters for the request"""
        # Start with common parameters
        sampling_params = {
            "temperature": request.temperature,
            "max_new_tokens": request.max_tokens,
            "min_new_tokens": request.min_tokens,
            "stop": request.stop,
            "stop_token_ids": request.stop_token_ids,
            "stop_regex": request.stop_regex,
            "top_p": request.top_p,
            "top_k": request.top_k,
            "min_p": request.min_p,
            "presence_penalty": request.presence_penalty,
            "frequency_penalty": request.frequency_penalty,
            "repetition_penalty": request.repetition_penalty,
            "regex": request.regex,
            "json_schema": request.json_schema,
            "ebnf": request.ebnf,
            "n": request.n,
            "no_stop_trim": request.no_stop_trim,
            "ignore_eos": request.ignore_eos,
            "skip_special_tokens": request.skip_special_tokens,
            "logit_bias": request.logit_bias,
            "custom_params": request.custom_params,
            "sampling_seed": request.seed,
        }

        # Handle response_format constraints
        if request.response_format and request.response_format.type == "json_schema":
            sampling_params["json_schema"] = convert_json_schema_to_str(
                request.response_format.json_schema.schema_
            )
        elif request.response_format and request.response_format.type == "json_object":
            sampling_params["json_schema"] = '{"type": "object"}'
        elif (
            request.response_format and request.response_format.type == "structural_tag"
        ):
            sampling_params["structural_tag"] = convert_json_schema_to_str(
                request.response_format.model_dump(by_alias=True)
            )

        return sampling_params
```
**EN:** This block implements the method `_build_sampling_params(request)` on `OpenAIServingCompletion`. It focuses on Build sampling parameters for the request, so the class can advance the serving completions workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingCompletion` 上的方法 `_build_sampling_params(request)`。它围绕 `_build_sampling_params` 所承担的 serving completions 相关职责展开，使该类能够独立推进相应流程。

### Lines 181-207: Implement async handle streaming request / 实现异步handle streaming 请求
```python
    async def _handle_streaming_request(
        self,
        adapted_request: GenerateReqInput,
        request: CompletionRequest,
        raw_request: Request,
    ) -> Union[StreamingResponse, ErrorResponse]:
        """Handle streaming completion request"""
        generator = self._generate_completion_stream(
            adapted_request, request, raw_request
        )

        # Kick-start the generator to trigger validation before HTTP 200 is sent.
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
**EN:** This block implements the async method `_handle_streaming_request(adapted_request, request, raw_request)` on `OpenAIServingCompletion`. It focuses on Handle streaming completion request, so the class can advance the serving completions workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingCompletion` 上的异步方法 `_handle_streaming_request(adapted_request, request, raw_request)`。它围绕 `_handle_streaming_request` 所承担的 serving completions 相关职责展开，使该类能够独立推进相应流程。

### Lines 209-244: Implement async generate completion stream / 实现异步generate 补全 流式
```python
    async def _generate_completion_stream(
        self,
        adapted_request: GenerateReqInput,
        request: CompletionRequest,
        raw_request: Request,
    ) -> AsyncGenerator[str, None]:
        """Generate streaming completion response"""
        created = int(time.time())

        # State tracking for streaming
        stream_offsets = {}
        n_prev_tokens = {}

        # Usage tracking
        prompt_tokens = {}
        completion_tokens = {}
        reasoning_tokens = {}
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
            ):
                index = content.get("index", 0)

                text = content["text"]
                prompt_tokens[index] = content["meta_info"].get("prompt_tokens", 0)
```
**EN:** This block implements the async method `_generate_completion_stream(adapted_request, request, raw_request)` on `OpenAIServingCompletion`. It focuses on Generate streaming completion response, so the class can advance the serving completions workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingCompletion` 上的异步方法 `_generate_completion_stream(adapted_request, request, raw_request)`。它围绕 `_generate_completion_stream` 所承担的 serving completions 相关职责展开，使该类能够独立推进相应流程。

### Lines 245-280: Continue async generate completion stream / 继续说明异步generate 补全 流式
```python
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

                is_first_chunk = index not in stream_offsets
                offset = stream_offsets.get(index, 0)
                # Handle echo for first chunk
                if is_first_chunk:  # The first chunk
                    if request.echo:
                        echo_text = self._get_echo_text(request, index)
                        text = echo_text + text

                # Handle logprobs
                logprobs = None
                if request.logprobs is not None:
                    # The first chunk and echo is enabled.
                    if is_first_chunk and request.echo:
                        input_token_logprobs = content["meta_info"][
                            "input_token_logprobs"
                        ]
                        input_top_logprobs = content["meta_info"]["input_top_logprobs"]
                    else:
                        input_token_logprobs = None
                        input_top_logprobs = None

                    n_prev_token = n_prev_tokens.get(index, 0)
                    total_output_logprobs = content["meta_info"][
```
**EN:** This block implements the async method `_generate_completion_stream(adapted_request, request, raw_request)` on `OpenAIServingCompletion`. It focuses on Generate streaming completion response, so the class can advance the serving completions workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingCompletion` 上的异步方法 `_generate_completion_stream(adapted_request, request, raw_request)`。它围绕 `_generate_completion_stream` 所承担的 serving completions 相关职责展开，使该类能够独立推进相应流程。

### Lines 281-316: Continue async generate completion stream / 继续说明异步generate 补全 流式
```python
                        "output_token_logprobs_length"
                    ]
                    if (
                        n_prev_token < total_output_logprobs
                        or input_token_logprobs is not None
                    ):
                        output_token_logprobs = content["meta_info"][
                            "output_token_logprobs"
                        ]
                        output_top_logprobs = content["meta_info"].get(
                            "output_top_logprobs", []
                        )
                        if (
                            not self.tokenizer_manager.server_args.incremental_streaming_output
                        ):
                            output_token_logprobs = output_token_logprobs[
                                n_prev_token:total_output_logprobs
                            ]
                            output_top_logprobs = output_top_logprobs[
                                n_prev_token:total_output_logprobs
                            ]
                        logprobs = to_openai_style_logprobs(
                            input_token_logprobs=input_token_logprobs,
                            input_top_logprobs=input_top_logprobs,
                            output_token_logprobs=output_token_logprobs,
                            output_top_logprobs=output_top_logprobs,
                        )
                    n_prev_tokens[index] = total_output_logprobs

                # Generate delta
                delta = text[offset:]
                stream_offsets[index] = len(content["text"])
                finish_reason = content["meta_info"].get("finish_reason", None)
                finish_reason_type = finish_reason["type"] if finish_reason else None

                # Abort with an explicit error status_code is a system error
```
**EN:** This block implements the async method `_generate_completion_stream(adapted_request, request, raw_request)` on `OpenAIServingCompletion`. It focuses on Generate streaming completion response, so the class can advance the serving completions workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingCompletion` 上的异步方法 `_generate_completion_stream(adapted_request, request, raw_request)`。它围绕 `_generate_completion_stream` 所承担的 serving completions 相关职责展开，使该类能够独立推进相应流程。

### Lines 317-352: Continue async generate completion stream / 继续说明异步generate 补全 流式
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

                choice_data = CompletionResponseStreamChoice(
                    index=index,
                    text=delta,
                    logprobs=logprobs,
                    finish_reason=finish_reason_type,
                    matched_stop=(
                        finish_reason["matched"]
                        if finish_reason and "matched" in finish_reason
                        else None
                    ),
                )
                chunk = CompletionStreamResponse(
                    id=content["meta_info"]["id"],
                    created=created,
                    object="text_completion",
                    choices=[choice_data],
                    model=request.model,
                )
```
**EN:** This block implements the async method `_generate_completion_stream(adapted_request, request, raw_request)` on `OpenAIServingCompletion`. It focuses on Generate streaming completion response, so the class can advance the serving completions workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingCompletion` 上的异步方法 `_generate_completion_stream(adapted_request, request, raw_request)`。它围绕 `_generate_completion_stream` 所承担的 serving completions 相关职责展开，使该类能够独立推进相应流程。

### Lines 353-388: Continue async generate completion stream / 继续说明异步generate 补全 流式
```python
                # Add usage stats if continuous_usage_stats is enabled
                if continuous_usage_stats:
                    chunk.usage = UsageProcessor.calculate_token_usage(
                        prompt_tokens=prompt_tokens.get(index, 0),
                        completion_tokens=completion_tokens.get(index, 0),
                        reasoning_tokens=reasoning_tokens.get(index, 0),
                    )

                yield f"data: {chunk.model_dump_json()}\n\n"
                stream_started = True

            if request.return_hidden_states and hidden_states:
                for index, choice_hidden_states in hidden_states.items():
                    if choice_hidden_states:
                        last_token_hidden_states = (
                            choice_hidden_states[-1]
                            if len(choice_hidden_states) > 1
                            else []
                        )
                        hidden_states_chunk = CompletionStreamResponse(
                            id=content["meta_info"]["id"],
                            created=created,
                            object="text_completion",
                            choices=[
                                CompletionResponseStreamChoice(
                                    index=index,
                                    text="",
                                    hidden_states=last_token_hidden_states,
                                    finish_reason=None,
                                )
                            ],
                            model=request.model,
                        )
                        yield f"data: {hidden_states_chunk.model_dump_json()}\n\n"

            sglext_routed = None
```
**EN:** This block implements the async method `_generate_completion_stream(adapted_request, request, raw_request)` on `OpenAIServingCompletion`. It focuses on Generate streaming completion response, so the class can advance the serving completions workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingCompletion` 上的异步方法 `_generate_completion_stream(adapted_request, request, raw_request)`。它围绕 `_generate_completion_stream` 所承担的 serving completions 相关职责展开，使该类能够独立推进相应流程。

### Lines 389-424: Continue async generate completion stream / 继续说明异步generate 补全 流式
```python
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
                sglext_chunk = CompletionStreamResponse(
                    id=content["meta_info"]["id"],
                    created=created,
                    object="text_completion",
                    choices=[],  # sglext is at response level
                    model=request.model,
                    sglext=SglExt(
                        routed_experts=sglext_routed,
                        cached_tokens_details=sglext_details,
                    ),
                )
                yield f"data: {sglext_chunk.model_dump_json()}\n\n"

            # Handle final usage chunk
            if include_usage:
                usage = UsageProcessor.calculate_streaming_usage(
                    prompt_tokens,
                    reasoning_tokens,
                    completion_tokens,
                    cached_tokens=cached_tokens,
                    n_choices=request.n,
                    enable_cache_report=self.tokenizer_manager.server_args.enable_cache_report,
```
**EN:** This block implements the async method `_generate_completion_stream(adapted_request, request, raw_request)` on `OpenAIServingCompletion`. It focuses on Generate streaming completion response, so the class can advance the serving completions workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingCompletion` 上的异步方法 `_generate_completion_stream(adapted_request, request, raw_request)`。它围绕 `_generate_completion_stream` 所承担的 serving completions 相关职责展开，使该类能够独立推进相应流程。

### Lines 425-442: Continue async generate completion stream / 继续说明异步generate 补全 流式
```python
                )
                final_usage_chunk = CompletionStreamResponse(
                    id=content["meta_info"]["id"],
                    created=created,
                    choices=[],
                    model=request.model,
                    usage=usage,
                )
                final_usage_data = final_usage_chunk.model_dump_json(exclude_none=True)
                yield f"data: {final_usage_data}\n\n"

        except Exception as e:
            if not stream_started:
                raise
            error = self.create_streaming_error_response(str(e))
            yield f"data: {error}\n\n"

        yield "data: [DONE]\n\n"
```
**EN:** This block implements the async method `_generate_completion_stream(adapted_request, request, raw_request)` on `OpenAIServingCompletion`. It focuses on Generate streaming completion response, so the class can advance the serving completions workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingCompletion` 上的异步方法 `_generate_completion_stream(adapted_request, request, raw_request)`。它围绕 `_generate_completion_stream` 所承担的 serving completions 相关职责展开，使该类能够独立推进相应流程。

### Lines 444-468: Implement async handle non streaming request / 实现异步handle non streaming 请求
```python
    async def _handle_non_streaming_request(
        self,
        adapted_request: GenerateReqInput,
        request: CompletionRequest,
        raw_request: Request,
    ) -> Union[CompletionResponse, ErrorResponse, ORJSONResponse]:
        """Handle non-streaming completion request"""
        try:
            generator = self.tokenizer_manager.generate_request(
                adapted_request, raw_request
            )
            ret = await generator.__anext__()
        except ValueError as e:
            return self.create_error_response(str(e))

        if not isinstance(ret, list):
            ret = [ret]

        response = self._build_completion_response(
            request,
            ret,
            int(time.time()),
        )

        return response
```
**EN:** This block implements the async method `_handle_non_streaming_request(adapted_request, request, raw_request)` on `OpenAIServingCompletion`. It focuses on Handle non-streaming completion request, so the class can advance the serving completions workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingCompletion` 上的异步方法 `_handle_non_streaming_request(adapted_request, request, raw_request)`。它围绕 `_handle_non_streaming_request` 所承担的 serving completions 相关职责展开，使该类能够独立推进相应流程。

### Lines 470-505: Implement build completion response / 实现build 补全 响应
```python
    def _build_completion_response(
        self,
        request: CompletionRequest,
        ret: List[Dict[str, Any]],
        created: int,
    ) -> CompletionResponse:
        """Build completion response from generation results"""
        choices = []
        echo = False

        # Prepare echo prompts if needed
        echo_prompts = []
        if request.echo:
            echo_prompts = self._prepare_echo_prompts(request)
            echo = True

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
            text = ret_item["text"]

            # Handle echo
            if echo:
                prompt_index = idx // request.n
                text = echo_prompts[prompt_index] + text
```
**EN:** This block implements the method `_build_completion_response(request, ret, created)` on `OpenAIServingCompletion`. It focuses on Build completion response from generation results, so the class can advance the serving completions workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingCompletion` 上的方法 `_build_completion_response(request, ret, created)`。它围绕 `_build_completion_response` 所承担的 serving completions 相关职责展开，使该类能够独立推进相应流程。

### Lines 506-541: Continue build completion response / 继续说明build 补全 响应
```python

            # Handle logprobs
            logprobs = None
            if request.logprobs is not None:
                if echo:
                    input_token_logprobs = ret_item["meta_info"]["input_token_logprobs"]
                    input_top_logprobs = ret_item["meta_info"]["input_top_logprobs"]
                else:
                    input_token_logprobs = None
                    input_top_logprobs = None

                logprobs = to_openai_style_logprobs(
                    input_token_logprobs=input_token_logprobs,
                    input_top_logprobs=input_top_logprobs,
                    output_token_logprobs=ret_item["meta_info"].get(
                        "output_token_logprobs", []
                    ),
                    output_top_logprobs=ret_item["meta_info"].get(
                        "output_top_logprobs", []
                    ),
                )

            # Handle hidden states
            hidden_states = process_hidden_states_from_ret(ret_item, request)

            finish_reason = ret_item["meta_info"]["finish_reason"]

            choice_data = CompletionResponseChoice(
                index=idx,
                text=text,
                logprobs=logprobs,
                finish_reason=finish_reason["type"] if finish_reason else None,
                matched_stop=(
                    finish_reason["matched"]
                    if finish_reason and "matched" in finish_reason
                    else None
```
**EN:** This block implements the method `_build_completion_response(request, ret, created)` on `OpenAIServingCompletion`. It focuses on Build completion response from generation results, so the class can advance the serving completions workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingCompletion` 上的方法 `_build_completion_response(request, ret, created)`。它围绕 `_build_completion_response` 所承担的 serving completions 相关职责展开，使该类能够独立推进相应流程。

### Lines 542-561: Continue build completion response / 继续说明build 补全 响应
```python
                ),
                hidden_states=hidden_states,
            )
            choices.append(choice_data)

        # Calculate usage
        cache_report = self.tokenizer_manager.server_args.enable_cache_report
        usage = UsageProcessor.calculate_response_usage(
            ret, n_choices=request.n, enable_cache_report=cache_report
        )

        return CompletionResponse(
            id=ret[0]["meta_info"]["id"],
            model=request.model,
            created=created,
            choices=choices,
            usage=usage,
            metadata={"weight_version": ret[0]["meta_info"]["weight_version"]},
            sglext=response_sglext,
        )
```
**EN:** This block implements the method `_build_completion_response(request, ret, created)` on `OpenAIServingCompletion`. It focuses on Build completion response from generation results, so the class can advance the serving completions workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingCompletion` 上的方法 `_build_completion_response(request, ret, created)`。它围绕 `_build_completion_response` 所承担的 serving completions 相关职责展开，使该类能够独立推进相应流程。

### Lines 563-585: Implement get echo text / 实现get echo text
```python
    def _get_echo_text(self, request: CompletionRequest, index: int) -> str:
        """Get echo text for streaming response"""
        if isinstance(request.prompt, str):
            # for the case of single str prompts
            return request.prompt
        elif isinstance(request.prompt, list):
            if isinstance(request.prompt[0], str):
                # for the case of multiple str prompts
                return request.prompt[index // request.n]
            elif isinstance(request.prompt[0], int):
                # for the case of single token ids prompt
                return self.tokenizer_manager.tokenizer.decode(
                    request.prompt, skip_special_tokens=True
                )
            elif isinstance(request.prompt[0], list) and isinstance(
                request.prompt[0][0], int
            ):
                # for the case of multiple token ids prompts
                return self.tokenizer_manager.tokenizer.decode(
                    request.prompt[index // request.n],
                    skip_special_tokens=True,
                )
        return ""
```
**EN:** This block implements the method `_get_echo_text(request, index)` on `OpenAIServingCompletion`. It focuses on Get echo text for streaming response, so the class can advance the serving completions workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingCompletion` 上的方法 `_get_echo_text(request, index)`。它围绕 `_get_echo_text` 所承担的 serving completions 相关职责展开，使该类能够独立推进相应流程。

### Lines 587-610: Implement prepare echo prompts / 实现prepare echo prompts
```python
    def _prepare_echo_prompts(self, request: CompletionRequest) -> List[str]:
        """Prepare echo prompts for non-streaming response"""
        # TODO: handle the case prompt is token ids
        if isinstance(request.prompt, list) and isinstance(request.prompt[0], str):
            # for the case of multiple str prompts
            return request.prompt
        elif isinstance(request.prompt, list) and isinstance(request.prompt[0], list):
            # for the case of multiple token ids prompts
            return [
                self.tokenizer_manager.tokenizer.decode(
                    prompt, skip_special_tokens=True
                )
                for prompt in request.prompt
            ]
        elif isinstance(request.prompt, list) and isinstance(request.prompt[0], int):
            # for the case of single token ids prompt
            return [
                self.tokenizer_manager.tokenizer.decode(
                    request.prompt, skip_special_tokens=True
                )
            ]
        else:
            # for the case of single str prompt
            return [request.prompt]
```
**EN:** This block implements the method `_prepare_echo_prompts(request)` on `OpenAIServingCompletion`. It focuses on Prepare echo prompts for non-streaming response, so the class can advance the serving completions workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingCompletion` 上的方法 `_prepare_echo_prompts(request)`。它围绕 `_prepare_echo_prompts` 所承担的 serving completions 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: OpenAIServingCompletion
- **Domain focus / 领域焦点**: serving completions / serving completions
- **Concurrency / 并发特征**: async/await appears throughout the module / 模块中多处使用 async/await 协调并发流程

## Dependencies / 依赖关系
- **Standard Library / 标准库**: logging, time, typing
- **Third-party / 第三方库**: __future__, fastapi, fastapi.responses, http
- **Local Modules / 本地模块**: sglang.srt.entrypoints.openai.protocol, sglang.srt.entrypoints.openai.serving_base, sglang.srt.entrypoints.openai.usage_processor, sglang.srt.entrypoints.openai.utils, sglang.srt.managers.io_struct, sglang.srt.managers.template_manager, sglang.srt.managers.tokenizer_manager, sglang.srt.parser.code_completion_parser, sglang.utils
