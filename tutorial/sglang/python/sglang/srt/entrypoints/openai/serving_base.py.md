# serving_base.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/openai/serving_base.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements serving base logic for the OpenAI-compatible serving layer. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 serving base 相关的逻辑，并服务于 OpenAI 兼容服务层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-7: Import runtime dependencies / 导入运行时依赖
```python
import json
import logging
import uuid
from abc import ABC, abstractmethod
from typing import TYPE_CHECKING, Any, List, Optional, Tuple, Union
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 9-11: Import runtime dependencies / 导入运行时依赖
```python
import orjson
from fastapi import HTTPException, Request
from fastapi.responses import ORJSONResponse, StreamingResponse
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 13-17: Import runtime dependencies / 导入运行时依赖
```python
from sglang.srt.entrypoints.openai.encoding_dsv32 import DS32EncodingError
from sglang.srt.entrypoints.openai.protocol import ErrorResponse, OpenAIServingRequest
from sglang.srt.managers.io_struct import EmbeddingReqInput, GenerateReqInput
from sglang.srt.observability.req_time_stats import monotonic_time
from sglang.srt.server_args import ServerArgs
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 19-20: Provide supporting module logic / 提供辅助模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.managers.tokenizer_manager import TokenizerManager
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 22-27: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)


# Base class for specific endpoint handlers
class OpenAIServingBase(ABC):
    """Abstract base class for OpenAI endpoint handlers"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 29-38: Initialize OpenAIServingBase / 初始化 OpenAIServingBase
```python
    def __init__(self, tokenizer_manager: TokenizerManager):
        self.tokenizer_manager = tokenizer_manager
        self.allowed_custom_labels = (
            set(
                self.tokenizer_manager.server_args.tokenizer_metrics_allowed_custom_labels
            )
            if isinstance(self.tokenizer_manager.server_args, ServerArgs)
            and self.tokenizer_manager.server_args.tokenizer_metrics_allowed_custom_labels
            else None
        )
```
**EN:** This block implements the initializer `__init__(tokenizer_manager)` for `OpenAIServingBase`. It prepares the object state and connects the instance to the surrounding serving base workflow.
**CN:** 该代码块实现 `OpenAIServingBase` 的初始化方法 `__init__(tokenizer_manager)`。它负责准备对象状态，并把实例接入 serving base 相关的运行流程。

### Lines 40-53: Implement parse model parameter / 实现parse 模型 parameter
```python
    def _parse_model_parameter(self, model: str) -> Tuple[str, Optional[str]]:
        """Parse 'base-model:adapter-name' syntax to extract LoRA adapter.

        Returns (base_model, adapter_name) or (model, None) if no colon present.
        """
        if ":" not in model:
            return model, None

        # Split on first colon only to handle model paths with multiple colons
        parts = model.split(":", 1)
        base_model = parts[0].strip()
        adapter_name = parts[1].strip() or None

        return base_model, adapter_name
```
**EN:** This block implements the method `_parse_model_parameter(model)` on `OpenAIServingBase`. It focuses on Parse 'base-model:adapter-name' syntax to extract LoRA adapter., so the class can advance the serving base workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingBase` 上的方法 `_parse_model_parameter(model)`。它围绕 `_parse_model_parameter` 所承担的 serving base 相关职责展开，使该类能够独立推进相应流程。

### Lines 55-71: Implement resolve lora path / 实现resolve lora path
```python
    def _resolve_lora_path(
        self,
        request_model: str,
        explicit_lora_path: Optional[Union[str, List[Optional[str]]]],
    ) -> Optional[Union[str, List[Optional[str]]]]:
        """Resolve LoRA adapter with priority: model parameter > explicit lora_path.

        Returns adapter name or None. Supports both single values and lists (batches).
        """
        _, adapter_from_model = self._parse_model_parameter(request_model)

        # Model parameter adapter takes precedence
        if adapter_from_model is not None:
            return adapter_from_model

        # Fall back to explicit lora_path
        return explicit_lora_path
```
**EN:** This block implements the method `_resolve_lora_path(request_model, explicit_lora_path)` on `OpenAIServingBase`. It focuses on Resolve LoRA adapter with priority: model parameter > explicit lora_path., so the class can advance the serving base workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingBase` 上的方法 `_resolve_lora_path(request_model, explicit_lora_path)`。它围绕 `_resolve_lora_path` 所承担的 serving base 相关职责展开，使该类能够独立推进相应流程。

### Lines 73-108: Implement async handle request / 实现异步handle 请求
```python
    async def handle_request(
        self, request: OpenAIServingRequest, raw_request: Request
    ) -> Union[Any, StreamingResponse, ErrorResponse]:
        """Handle the specific request type with common pattern
        If you want to override this method, you should be careful to record the validation time.
        """
        received_time = monotonic_time()

        try:
            # Validate request
            error_msg = self._validate_request(request)
            if error_msg:
                return self.create_error_response(error_msg)

            # Log the raw OpenAI request payload before conversion to tokenized form.
            request_logger = self.tokenizer_manager.request_logger
            if request_logger.log_requests and request_logger.log_requests_level >= 2:
                request_logger.log_openai_received_request(request, request=raw_request)

            # Convert to internal format
            adapted_request, processed_request = self._convert_to_internal_request(
                request, raw_request
            )

            if isinstance(adapted_request, (GenerateReqInput, EmbeddingReqInput)):
                # Only set timing fields if adapted_request supports them
                adapted_request.received_time = received_time

            # Note(Xinyuan): raw_request below is only used for detecting the connection of the client
            if hasattr(request, "stream") and request.stream:
                return await self._handle_streaming_request(
                    adapted_request, processed_request, raw_request
                )
            else:
                return await self._handle_non_streaming_request(
                    adapted_request, processed_request, raw_request
```
**EN:** This block implements the async method `handle_request(request, raw_request)` on `OpenAIServingBase`. It focuses on Handle the specific request type with common pattern If you want to override this method, you should be careful to record the validation time., so the class can advance the serving base workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingBase` 上的异步方法 `handle_request(request, raw_request)`。它围绕 `handle_request` 所承担的 serving base 相关职责展开，使该类能够独立推进相应流程。

### Lines 109-133: Continue async handle request / 继续说明异步handle 请求
```python
                )
        except HTTPException as e:
            return self.create_error_response(
                message=e.detail, err_type=str(e.status_code), status_code=e.status_code
            )
        except ValueError as e:
            return self.create_error_response(
                message=str(e),
                err_type="BadRequest",
                status_code=400,
            )
        except DS32EncodingError as e:
            logger.info(f"DS32EncodingError: {e}")
            return self.create_error_response(
                message=str(e),
                err_type="BadRequest",
                status_code=400,
            )
        except Exception as e:
            logger.exception(f"Error in request: {e}")
            return self.create_error_response(
                message=f"Internal server error: {str(e)}",
                err_type="InternalServerError",
                status_code=500,
            )
```
**EN:** This block implements the async method `handle_request(request, raw_request)` on `OpenAIServingBase`. It focuses on Handle the specific request type with common pattern If you want to override this method, you should be careful to record the validation time., so the class can advance the serving base workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingBase` 上的异步方法 `handle_request(request, raw_request)`。它围绕 `handle_request` 所承担的 serving base 相关职责展开，使该类能够独立推进相应流程。

### Lines 134-135: Provide supporting module logic / 提供辅助模块逻辑
```python

    @abstractmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 136-138: Implement request id prefix / 实现请求 id prefix
```python
    def _request_id_prefix(self) -> str:
        """Generate request ID based on request type"""
        pass
```
**EN:** This block implements the method `_request_id_prefix()` on `OpenAIServingBase`. It focuses on Generate request ID based on request type, so the class can advance the serving base workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingBase` 上的方法 `_request_id_prefix()`。它围绕 `_request_id_prefix` 所承担的 serving base 相关职责展开，使该类能够独立推进相应流程。

### Lines 140-149: Implement generate request id base / 实现generate 请求 id base
```python
    def _generate_request_id_base(self, request: OpenAIServingRequest) -> Optional[str]:
        """Generate request ID based on request type"""
        return None

        # TODO(chang): the rid is used in io_strcut check and often violates `The rid should be a list` AssertionError
        # Temporarily return None in this function until the rid logic is clear.
        if rid := getattr(request, "rid", None):
            return rid

        return f"{self._request_id_prefix()}{uuid.uuid4().hex}"
```
**EN:** This block implements the method `_generate_request_id_base(request)` on `OpenAIServingBase`. It focuses on Generate request ID based on request type, so the class can advance the serving base workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingBase` 上的方法 `_generate_request_id_base(request)`。它围绕 `_generate_request_id_base` 所承担的 serving base 相关职责展开，使该类能够独立推进相应流程。

### Lines 151-162: Implement compute extra key / 实现compute extra key
```python
    def _compute_extra_key(self, request: OpenAIServingRequest) -> Optional[str]:
        """Compute the final extra_key by concatenating cache_salt and extra_key if both are provided."""
        parts = []
        for key in ["cache_salt", "extra_key"]:
            value = getattr(request, key, None)
            if value:
                if not isinstance(value, str):
                    raise TypeError(
                        f"Value of {key} must be a string, but got {type(value).__name__}"
                    )
                parts.append(value)
        return "".join(parts) if parts else None
```
**EN:** This block implements the method `_compute_extra_key(request)` on `OpenAIServingBase`. It focuses on Compute the final extra_key by concatenating cache_salt and extra_key if both are provided., so the class can advance the serving base workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingBase` 上的方法 `_compute_extra_key(request)`。它围绕 `_compute_extra_key` 所承担的 serving base 相关职责展开，使该类能够独立推进相应流程。

### Lines 163-164: Provide supporting module logic / 提供辅助模块逻辑
```python

    @abstractmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 165-171: Implement convert to internal request / 实现convert to internal 请求
```python
    def _convert_to_internal_request(
        self,
        request: OpenAIServingRequest,
        raw_request: Request = None,
    ) -> tuple[GenerateReqInput, OpenAIServingRequest]:
        """Convert OpenAI request to internal format"""
        pass
```
**EN:** This block implements the method `_convert_to_internal_request(request, raw_request)` on `OpenAIServingBase`. It focuses on Convert OpenAI request to internal format, so the class can advance the serving base workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingBase` 上的方法 `_convert_to_internal_request(request, raw_request)`。它围绕 `_convert_to_internal_request` 所承担的 serving base 相关职责展开，使该类能够独立推进相应流程。

### Lines 173-187: Implement async handle streaming request / 实现异步handle streaming 请求
```python
    async def _handle_streaming_request(
        self,
        adapted_request: GenerateReqInput,
        request: OpenAIServingRequest,
        raw_request: Request,
    ) -> Union[StreamingResponse, ErrorResponse, ORJSONResponse]:
        """Handle streaming request

        Override this method in child classes that support streaming requests.
        """
        return self.create_error_response(
            message=f"{self.__class__.__name__} does not support streaming requests",
            err_type="NotImplementedError",
            status_code=501,
        )
```
**EN:** This block implements the async method `_handle_streaming_request(adapted_request, request, raw_request)` on `OpenAIServingBase`. It focuses on Handle streaming request Override this method in child classes that support streaming requests., so the class can advance the serving base workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingBase` 上的异步方法 `_handle_streaming_request(adapted_request, request, raw_request)`。它围绕 `_handle_streaming_request` 所承担的 serving base 相关职责展开，使该类能够独立推进相应流程。

### Lines 189-203: Implement async handle non streaming request / 实现异步handle non streaming 请求
```python
    async def _handle_non_streaming_request(
        self,
        adapted_request: GenerateReqInput,
        request: OpenAIServingRequest,
        raw_request: Request,
    ) -> Union[Any, ErrorResponse, ORJSONResponse]:
        """Handle non-streaming request

        Override this method in child classes that support non-streaming requests.
        """
        return self.create_error_response(
            message=f"{self.__class__.__name__} does not support non-streaming requests",
            err_type="NotImplementedError",
            status_code=501,
        )
```
**EN:** This block implements the async method `_handle_non_streaming_request(adapted_request, request, raw_request)` on `OpenAIServingBase`. It focuses on Handle non-streaming request Override this method in child classes that support non-streaming requests., so the class can advance the serving base workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingBase` 上的异步方法 `_handle_non_streaming_request(adapted_request, request, raw_request)`。它围绕 `_handle_non_streaming_request` 所承担的 serving base 相关职责展开，使该类能够独立推进相应流程。

### Lines 205-207: Implement validate request / 实现validate 请求
```python
    def _validate_request(self, _: OpenAIServingRequest) -> Optional[str]:
        """Validate request"""
        pass
```
**EN:** This block implements the method `_validate_request(_)` on `OpenAIServingBase`. It focuses on Validate request, so the class can advance the serving base workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingBase` 上的方法 `_validate_request(_)`。它围绕 `_validate_request` 所承担的 serving base 相关职责展开，使该类能够独立推进相应流程。

### Lines 209-225: Implement create error response / 实现create error 响应
```python
    def create_error_response(
        self,
        message: str,
        err_type: str = "BadRequestError",
        status_code: int = 400,
        param: Optional[str] = None,
    ) -> ORJSONResponse:
        """Create an error response"""
        # TODO: remove fastapi dependency in openai and move response handling to the entrypoint
        error = ErrorResponse(
            object="error",
            message=message,
            type=err_type,
            param=param,
            code=status_code,
        )
        return ORJSONResponse(content=error.model_dump(), status_code=status_code)
```
**EN:** This block implements the method `create_error_response(message, err_type, status_code, param)` on `OpenAIServingBase`. It focuses on Create an error response, so the class can advance the serving base workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingBase` 上的方法 `create_error_response(message, err_type, status_code, param)`。它围绕 `create_error_response` 所承担的 serving base 相关职责展开，使该类能够独立推进相应流程。

### Lines 227-241: Implement create streaming error response / 实现create streaming error 响应
```python
    def create_streaming_error_response(
        self,
        message: str,
        err_type: str = "BadRequestError",
        status_code: int = 400,
    ) -> str:
        """Create a streaming error response"""
        error = ErrorResponse(
            object="error",
            message=message,
            type=err_type,
            param=None,
            code=status_code,
        )
        return json.dumps({"error": error.model_dump()})
```
**EN:** This block implements the method `create_streaming_error_response(message, err_type, status_code)` on `OpenAIServingBase`. It focuses on Create a streaming error response, so the class can advance the serving base workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingBase` 上的方法 `create_streaming_error_response(message, err_type, status_code)`。它围绕 `create_streaming_error_response` 所承担的 serving base 相关职责展开，使该类能够独立推进相应流程。

### Lines 243-270: Implement extract custom labels / 实现extract custom labels
```python
    def extract_custom_labels(self, raw_request):
        if (
            not self.allowed_custom_labels
            or not self.tokenizer_manager.server_args.tokenizer_metrics_custom_labels_header
        ):
            return None

        custom_labels = None
        header = (
            self.tokenizer_manager.server_args.tokenizer_metrics_custom_labels_header
        )
        try:
            raw_labels = (
                orjson.loads(raw_request.headers.get(header))
                if raw_request and raw_request.headers.get(header)
                else None
            )
        except json.JSONDecodeError as e:
            logger.exception(f"Error in request: {e}")
            raw_labels = None

        if isinstance(raw_labels, dict):
            custom_labels = {
                label: value
                for label, value in raw_labels.items()
                if label in self.allowed_custom_labels
            }
        return custom_labels
```
**EN:** This block implements the method `extract_custom_labels(raw_request)` on `OpenAIServingBase`. It focuses on handling the serving base responsibilities represented by `extract_custom_labels`, so the class can advance the serving base workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingBase` 上的方法 `extract_custom_labels(raw_request)`。它围绕 `extract_custom_labels` 所承担的 serving base 相关职责展开，使该类能够独立推进相应流程。

### Lines 272-275: Implement extract routing key / 实现extract routing key
```python
    def extract_routing_key(self, raw_request):
        if raw_request is None:
            return None
        return raw_request.headers.get("x-smg-routing-key")
```
**EN:** This block implements the method `extract_routing_key(raw_request)` on `OpenAIServingBase`. It focuses on handling the serving base responsibilities represented by `extract_routing_key`, so the class can advance the serving base workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingBase` 上的方法 `extract_routing_key(raw_request)`。它围绕 `extract_routing_key` 所承担的 serving base 相关职责展开，使该类能够独立推进相应流程。

### Lines 277-306: Implement extract routed dp rank from header / 实现extract routed dp rank from header
```python
    def extract_routed_dp_rank_from_header(
        self, raw_request: Request, body_routed_dp_rank: Optional[int] = None
    ) -> Optional[int]:
        """Extract routed_dp_rank from HTTP header, with higher priority than routed_dp_rank in body.

        Header name: X-Data-Parallel-Rank (case-insensitive in HTTP/1.1/2)
        """
        if raw_request is None:
            return body_routed_dp_rank

        header_value = raw_request.headers.get("x-data-parallel-rank")
        if header_value is not None:
            try:
                header_dp_rank = int(header_value)
                if (
                    body_routed_dp_rank is not None
                    and header_dp_rank != body_routed_dp_rank
                ):
                    logger.debug(
                        f"X-Data-Parallel-Rank header ({header_dp_rank}) overrides "
                        f"body routed_dp_rank ({body_routed_dp_rank})"
                    )
                return header_dp_rank
            except ValueError:
                raise HTTPException(
                    status_code=400,
                    detail=f"Invalid X-Data-Parallel-Rank header: must be an integer, got '{header_value}'",
                )

        return body_routed_dp_rank
```
**EN:** This block implements the method `extract_routed_dp_rank_from_header(raw_request, body_routed_dp_rank)` on `OpenAIServingBase`. It focuses on Extract routed_dp_rank from HTTP header, with higher priority than routed_dp_rank in body., so the class can advance the serving base workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingBase` 上的方法 `extract_routed_dp_rank_from_header(raw_request, body_routed_dp_rank)`。它围绕 `extract_routed_dp_rank_from_header` 所承担的 serving base 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: OpenAIServingBase
- **Domain focus / 领域焦点**: serving base / serving base
- **Concurrency / 并发特征**: async/await appears throughout the module / 模块中多处使用 async/await 协调并发流程

## Dependencies / 依赖关系
- **Standard Library / 标准库**: json, logging, typing, uuid
- **Third-party / 第三方库**: __future__, abc, fastapi, fastapi.responses, orjson
- **Local Modules / 本地模块**: sglang.srt.entrypoints.openai.encoding_dsv32, sglang.srt.entrypoints.openai.protocol, sglang.srt.managers.io_struct, sglang.srt.managers.tokenizer_manager, sglang.srt.observability.req_time_stats, sglang.srt.server_args
