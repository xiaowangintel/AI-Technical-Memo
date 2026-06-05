# serving_classify.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/openai/serving_classify.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements serving classify logic for the OpenAI-compatible serving layer. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 serving classify 相关的逻辑，并服务于 OpenAI 兼容服务层。它提供支撑这一服务链路所需的类、函数与辅助流程。

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
import uuid
from typing import TYPE_CHECKING, Any, Dict, List, Optional, Union
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 8-11: Import runtime dependencies / 导入运行时依赖
```python
import torch
import torch.nn.functional as F
from fastapi import Request
from fastapi.responses import ORJSONResponse
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 13-19: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.entrypoints.openai.protocol import (
    ClassifyRequest,
    ClassifyResponse,
    ErrorResponse,
)
from sglang.srt.entrypoints.openai.serving_base import OpenAIServingBase
from sglang.srt.managers.io_struct import EmbeddingReqInput
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 21-23: Provide supporting module logic / 提供辅助模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.managers.template_manager import TemplateManager
    from sglang.srt.managers.tokenizer_manager import TokenizerManager
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 25-25: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 28-29: Provide supporting module logic / 提供辅助模块逻辑
```python
class OpenAIServingClassify(OpenAIServingBase):
    """Handler for v1/classify requests"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 31-45: Initialize OpenAIServingClassify / 初始化 OpenAIServingClassify
```python
    def __init__(
        self,
        tokenizer_manager: TokenizerManager,
        template_manager: TemplateManager,
    ):
        super().__init__(tokenizer_manager)
        self.template_manager = template_manager
        self.id2label = self._get_id2label_mapping()
        self.model_name = (
            self.tokenizer_manager.served_model_name
            if self.tokenizer_manager.served_model_name
            else self.tokenizer_manager.server_args.model_path
        )
        if not self.id2label:
            raise ValueError("id2label mapping is missing")
```
**EN:** This block implements the initializer `__init__(tokenizer_manager, template_manager)` for `OpenAIServingClassify`. It prepares the object state and connects the instance to the surrounding serving classify workflow.
**CN:** 该代码块实现 `OpenAIServingClassify` 的初始化方法 `__init__(tokenizer_manager, template_manager)`。它负责准备对象状态，并把实例接入 serving classify 相关的运行流程。

### Lines 47-48: Implement request id prefix / 实现请求 id prefix
```python
    def _request_id_prefix(self) -> str:
        return "classify-"
```
**EN:** This block implements the method `_request_id_prefix()` on `OpenAIServingClassify`. It focuses on handling the serving classify responsibilities represented by `_request_id_prefix`, so the class can advance the serving classify workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingClassify` 上的方法 `_request_id_prefix()`。它围绕 `_request_id_prefix` 所承担的 serving classify 相关职责展开，使该类能够独立推进相应流程。

### Lines 50-77: Implement convert to internal request / 实现convert to internal 请求
```python
    def _convert_to_internal_request(
        self,
        request: ClassifyRequest,
        raw_request: Request = None,
    ) -> tuple[EmbeddingReqInput, ClassifyRequest]:
        """Convert OpenAI embedding request to internal format"""
        prompt = request.input

        if isinstance(prompt, str):
            # Single string input
            prompt_kwargs = {"text": prompt}
        elif isinstance(prompt, list):
            if len(prompt) > 0 and isinstance(prompt[0], str):
                prompt_kwargs = {"text": prompt}
            else:
                # List of integers (token IDs) or empty list
                prompt_kwargs = {"input_ids": prompt}
        else:
            # Other types (should not happen but handle gracefully)
            prompt_kwargs = {"input_ids": prompt}

        adapted_request = EmbeddingReqInput(
            **prompt_kwargs,
            rid=request.rid,
            priority=request.priority,
        )

        return adapted_request, request
```
**EN:** This block implements the method `_convert_to_internal_request(request, raw_request)` on `OpenAIServingClassify`. It focuses on Convert OpenAI embedding request to internal format, so the class can advance the serving classify workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingClassify` 上的方法 `_convert_to_internal_request(request, raw_request)`。它围绕 `_convert_to_internal_request` 所承担的 serving classify 相关职责展开，使该类能够独立推进相应流程。

### Lines 79-109: Implement validate request / 实现validate 请求
```python
    def _validate_request(self, request: ClassifyRequest) -> Optional[str]:
        """Validate that the input is not empty or whitespace only."""
        if not (input := request.input):
            return "Input cannot be empty"

        # Handle single string
        if isinstance(input, str):
            if not input.strip():
                return "Input cannot be empty or whitespace only"
            return None

        # Handle list inputs
        if isinstance(input, list):
            # Check first element to determine type
            first_item = input[0]

            if isinstance(first_item, str):
                # List of strings
                for i, item in enumerate(input):
                    if not isinstance(item, str):
                        return f"All items in input list must be strings"
                    if not item.strip():
                        return f"Input at index {i} cannot be empty or whitespace only"
            elif isinstance(first_item, int):
                # List of integers (token IDs)
                for i, item in enumerate(input):
                    if not isinstance(item, int):
                        return f"All items in input list must be integers"
                    if item < 0:
                        return f"Token ID at index {i} must be non-negative"
        return None
```
**EN:** This block implements the method `_validate_request(request)` on `OpenAIServingClassify`. It focuses on Validate that the input is not empty or whitespace only., so the class can advance the serving classify workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingClassify` 上的方法 `_validate_request(request)`。它围绕 `_validate_request` 所承担的 serving classify 相关职责展开，使该类能够独立推进相应流程。

### Lines 111-127: Implement get id2label mapping / 实现get id2label mapping
```python
    def _get_id2label_mapping(self) -> Optional[Dict[int, str]]:
        """Get id2label mapping from model config."""
        try:
            hf_config = self.tokenizer_manager.model_config.hf_config
            # Check for id2label in hf_config
            if hf_config.id2label:
                return hf_config.id2label
            # Check for num_labels and create default mapping if needed
            if hasattr(hf_config, "num_labels") and hf_config.num_labels:
                num_labels = hf_config.num_labels
                # Create default mapping: {0: "LABEL_0", 1: "LABEL_1", ...}
                return {i: f"LABEL_{i}" for i in range(num_labels)}

        except Exception as e:
            logger.warning(f"Failed to get id2label mapping: {e}")

        return None
```
**EN:** This block implements the method `_get_id2label_mapping()` on `OpenAIServingClassify`. It focuses on Get id2label mapping from model config., so the class can advance the serving classify workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingClassify` 上的方法 `_get_id2label_mapping()`。它围绕 `_get_id2label_mapping` 所承担的 serving classify 相关职责展开，使该类能够独立推进相应流程。

### Lines 129-149: Implement async handle non streaming request / 实现异步handle non streaming 请求
```python
    async def _handle_non_streaming_request(
        self,
        adapted_request: EmbeddingReqInput,
        request: ClassifyRequest,
        raw_request: Request,
    ) -> Union[ClassifyResponse, ErrorResponse, ORJSONResponse]:
        """Handle non-streaming classification request."""
        # Generate request ID

        try:
            ret = await self.tokenizer_manager.generate_request(
                adapted_request, raw_request
            ).__anext__()
        except ValueError as e:
            return self.create_error_response(str(e))

        if not isinstance(ret, list):
            ret = [ret]

        response = self._build_classify_response(ret)
        return response
```
**EN:** This block implements the async method `_handle_non_streaming_request(adapted_request, request, raw_request)` on `OpenAIServingClassify`. It focuses on Handle non-streaming classification request., so the class can advance the serving classify workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingClassify` 上的异步方法 `_handle_non_streaming_request(adapted_request, request, raw_request)`。它围绕 `_handle_non_streaming_request` 所承担的 serving classify 相关职责展开，使该类能够独立推进相应流程。

### Lines 151-186: Implement build classify response / 实现build classify 响应
```python
    def _build_classify_response(self, ret: List[Dict[str, Any]]) -> ClassifyResponse:
        request_id = f"{self._request_id_prefix()}{uuid.uuid4().hex}"
        created_time = int(time.time())
        classify_objects = []
        prompt_tokens = 0
        total_latency = 0.0

        for i, item in enumerate(ret):
            embedding = item.get("embedding", [])
            meta_info = item.get("meta_info", {})

            prompt_tokens += meta_info.get("prompt_tokens", 0)
            total_latency += meta_info.get("e2e_latency", 0.0)

            if embedding:
                try:
                    embedding_tensor = torch.tensor(embedding, dtype=torch.float32)
                    probs = F.softmax(embedding_tensor, dim=0).tolist()

                    predicted_class = torch.argmax(embedding_tensor).item()

                    label = self.id2label[predicted_class]

                except Exception as e:
                    logger.error(f"Error processing embedding for item {i}: {e}")
                    probs = [1.0]
                    label = "Default"
            else:
                probs = [1.0]
                label = "Default"

            classify_obj = {
                "index": i,
                "label": label,
                "probs": probs,
                "num_classes": len(probs),
```
**EN:** This block implements the method `_build_classify_response(ret)` on `OpenAIServingClassify`. It focuses on handling the serving classify responsibilities represented by `_build_classify_response`, so the class can advance the serving classify workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingClassify` 上的方法 `_build_classify_response(ret)`。它围绕 `_build_classify_response` 所承担的 serving classify 相关职责展开，使该类能够独立推进相应流程。

### Lines 187-204: Continue build classify response / 继续说明build classify 响应
```python
            }
            classify_objects.append(classify_obj)

        response = {
            "id": request_id,
            "object": "list",
            "created": created_time,
            "model": self.model_name,
            "data": classify_objects,
            "usage": {
                "prompt_tokens": prompt_tokens,
                "total_tokens": prompt_tokens,
                "completion_tokens": 0,
                "prompt_tokens_details": None,
            },
        }

        return ClassifyResponse(**response)
```
**EN:** This block implements the method `_build_classify_response(ret)` on `OpenAIServingClassify`. It focuses on handling the serving classify responsibilities represented by `_build_classify_response`, so the class can advance the serving classify workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingClassify` 上的方法 `_build_classify_response(ret)`。它围绕 `_build_classify_response` 所承担的 serving classify 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: OpenAIServingClassify
- **Domain focus / 领域焦点**: serving classify / serving classify
- **Concurrency / 并发特征**: async/await appears throughout the module / 模块中多处使用 async/await 协调并发流程

## Dependencies / 依赖关系
- **Standard Library / 标准库**: logging, time, typing, uuid
- **Third-party / 第三方库**: __future__, fastapi, fastapi.responses, torch, torch.nn.functional
- **Local Modules / 本地模块**: sglang.srt.entrypoints.openai.protocol, sglang.srt.entrypoints.openai.serving_base, sglang.srt.managers.io_struct, sglang.srt.managers.template_manager, sglang.srt.managers.tokenizer_manager
