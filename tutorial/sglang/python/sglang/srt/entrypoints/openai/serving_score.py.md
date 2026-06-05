# serving_score.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/openai/serving_score.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements serving score logic for the OpenAI-compatible serving layer. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 serving score 相关的逻辑，并服务于 OpenAI 兼容服务层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2: Import runtime dependencies / 导入运行时依赖
```python
import logging
from typing import Union
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 4-6: Import runtime dependencies / 导入运行时依赖
```python
import torch
from fastapi import Request
from fastapi.responses import ORJSONResponse
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 8-14: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.entrypoints.openai.protocol import (
    ErrorResponse,
    ScoringRequest,
    ScoringResponse,
    UsageInfo,
)
from sglang.srt.entrypoints.openai.serving_base import OpenAIServingBase
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 16-16: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 19-24: Provide supporting module logic / 提供辅助模块逻辑
```python
class OpenAIServingScore(OpenAIServingBase):
    """Handler for /v1/score requests"""

    # NOTE: /v1/rerank is not an official OpenAI endpoint. This module may be moved
    # to another module in the future.
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 25-26: Implement request id prefix / 实现请求 id prefix
```python
    def _request_id_prefix(self) -> str:
        return "score-"
```
**EN:** This block implements the method `_request_id_prefix()` on `OpenAIServingScore`. It focuses on handling the serving score responsibilities represented by `_request_id_prefix`, so the class can advance the serving score workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingScore` 上的方法 `_request_id_prefix()`。它围绕 `_request_id_prefix` 所承担的 serving score 相关职责展开，使该类能够独立推进相应流程。

### Lines 28-37: Implement convert to internal request / 实现convert to internal 请求
```python
    def _convert_to_internal_request(
        self,
        request: ScoringRequest,
        raw_request: Request = None,
    ) -> tuple[ScoringRequest, ScoringRequest]:
        """Convert OpenAI scoring request to internal format"""
        # For scoring, we pass the request directly as the tokenizer_manager
        # has a specialized score_request method that doesn't use GenerateReqInput

        return request, request
```
**EN:** This block implements the method `_convert_to_internal_request(request, raw_request)` on `OpenAIServingScore`. It focuses on Convert OpenAI scoring request to internal format, so the class can advance the serving score workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingScore` 上的方法 `_convert_to_internal_request(request, raw_request)`。它围绕 `_convert_to_internal_request` 所承担的 serving score 相关职责展开，使该类能够独立推进相应流程。

### Lines 39-74: Implement async handle non streaming request / 实现异步handle non streaming 请求
```python
    async def _handle_non_streaming_request(
        self,
        adapted_request: ScoringRequest,
        request: ScoringRequest,
        raw_request: Request,
    ) -> Union[ScoringResponse, ErrorResponse]:
        """Handle the scoring request"""
        try:
            # query_embed_overrides is [num_replacements][hidden_size] -> List[Tensor]
            query_embed_overrides = (
                [
                    torch.tensor(v, dtype=torch.float32)
                    for v in request.query_embed_overrides
                ]
                if request.query_embed_overrides is not None
                else None
            )
            # item_embed_overrides is [num_items][num_replacements][hidden_size] -> List[Optional[List[Tensor]]]
            item_embed_overrides = (
                [
                    (
                        [torch.tensor(v, dtype=torch.float32) for v in per_item]
                        if per_item is not None
                        else None
                    )
                    for per_item in request.item_embed_overrides
                ]
                if request.item_embed_overrides is not None
                else None
            )

            result = await self.tokenizer_manager.score_request(
                query=request.query,
                items=request.items,
                label_token_ids=request.label_token_ids,
                apply_softmax=request.apply_softmax,
```
**EN:** This block implements the async method `_handle_non_streaming_request(adapted_request, request, raw_request)` on `OpenAIServingScore`. It focuses on Handle the scoring request, so the class can advance the serving score workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingScore` 上的异步方法 `_handle_non_streaming_request(adapted_request, request, raw_request)`。它围绕 `_handle_non_streaming_request` 所承担的 serving score 相关职责展开，使该类能够独立推进相应流程。

### Lines 75-102: Continue async handle non streaming request / 继续说明异步handle non streaming 请求
```python
                item_first=request.item_first,
                embed_override_token_id=request.embed_override_token_id,
                query_embed_overrides=query_embed_overrides,
                item_embed_overrides=item_embed_overrides,
                request=raw_request,
                return_pooled_hidden_states=request.return_pooled_hidden_states,
            )

            phs_as_lists = None
            if result.pooled_hidden_states is not None:
                phs_as_lists = [
                    t.tolist() if t is not None else None
                    for t in result.pooled_hidden_states
                ]

            response = ScoringResponse(
                scores=result.scores,
                pooled_hidden_states=phs_as_lists,
                model=request.model,
                usage=UsageInfo(
                    prompt_tokens=result.prompt_tokens,
                    total_tokens=result.prompt_tokens,
                ),
            )
            return ORJSONResponse(content=response.model_dump())

        except ValueError as e:
            return self.create_error_response(str(e))
```
**EN:** This block implements the async method `_handle_non_streaming_request(adapted_request, request, raw_request)` on `OpenAIServingScore`. It focuses on Handle the scoring request, so the class can advance the serving score workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingScore` 上的异步方法 `_handle_non_streaming_request(adapted_request, request, raw_request)`。它围绕 `_handle_non_streaming_request` 所承担的 serving score 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: OpenAIServingScore
- **Domain focus / 领域焦点**: serving score / serving score
- **Concurrency / 并发特征**: async/await appears throughout the module / 模块中多处使用 async/await 协调并发流程

## Dependencies / 依赖关系
- **Standard Library / 标准库**: logging, typing
- **Third-party / 第三方库**: fastapi, fastapi.responses, torch
- **Local Modules / 本地模块**: sglang.srt.entrypoints.openai.protocol, sglang.srt.entrypoints.openai.serving_base
