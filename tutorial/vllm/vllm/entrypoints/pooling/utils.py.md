# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/pooling/utils.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Provides reusable helper utilities shared across related entrypoints. Scope: pooling. / 提供在相关入口之间复用的辅助工具函数。 作用域：池化。

## Line-by-Line Analysis / 逐行分析
### Lines 4-27 — Imports and shared dependencies
```python
import importlib.util
import json
import math
from collections.abc import Callable, Sequence
from dataclasses import dataclass
from functools import lru_cache, partial
from typing import Any, Literal, cast

import pybase64
import torch
from fastapi.responses import JSONResponse, StreamingResponse

from vllm.config import ModelConfig
from vllm.entrypoints.openai.engine.protocol import UsageInfo
from vllm.logger import init_logger
from vllm.outputs import PoolingRequestOutput
from vllm.tasks import SupportedTask
from vllm.utils.serial_utils import (
    EMBED_DTYPES,
    EmbedDType,
    Endianness,
    binary2tensor,
    tensor2binary,
)
```
**EN:** This import block pulls in standard-library modules such as `importlib`, `json`, `math`, `collections`, `dataclasses`, `functools`, uses third-party packages like `pybase64`, `torch`, `fastapi`, depends on internal helpers such as `vllm.config`, `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.outputs`, `vllm.tasks`, `vllm.utils.serial_utils`.
**CN:** 该导入块引入 `importlib`, `json`, `math`, `collections`, `dataclasses`, `functools` 等标准库模块，使用 `pybase64`, `torch`, `fastapi` 等第三方库，依赖 `vllm.config`, `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.outputs`, `vllm.tasks`, `vllm.utils.serial_utils` 等 vLLM 内部模块。

### Lines 29-34 — Shared module state
```python
logger = init_logger(__name__)

JsonEncodingFormat = Literal["float", "base64"]
BytesEncodingFormat = Literal["bytes", "bytes_only"]
FloatEncodedPoolingOutput = list[float] | list[list[float]]
JsonEncodedPoolingOutput = FloatEncodedPoolingOutput | str
```
**EN:** This block initializes `logger`, `JsonEncodingFormat`, `BytesEncodingFormat`, `FloatEncodedPoolingOutput`, `JsonEncodedPoolingOutput`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`, `JsonEncodingFormat`, `BytesEncodingFormat`, `FloatEncodedPoolingOutput`, `JsonEncodedPoolingOutput`，为后续逻辑准备模块级常量或共享状态。

### Lines 38-44 — Class `MetadataItem`
```python
class MetadataItem:
    index: int
    embed_dtype: EmbedDType
    endianness: Endianness
    start: int
    end: int
    shape: tuple[int, ...]
```
**EN:** Class `MetadataItem` is defined here, as a standalone type, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `MetadataItem`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 47-66 — Function `build_metadata_items`
```python
def build_metadata_items(
    embed_dtype: EmbedDType,
    endianness: Endianness,
    shape: tuple[int, ...],
    n_request: int,
) -> list[MetadataItem]:
    n_bytes = EMBED_DTYPES[embed_dtype].nbytes
    size = math.prod(shape)

    return [
        MetadataItem(
            index=i,
            embed_dtype=embed_dtype,
            endianness=endianness,
            start=i * size * n_bytes,
            end=(i + 1) * size * n_bytes,
            shape=shape,
        )
        for i in range(n_request)
    ]
```
**EN:** This function `build_metadata_items` implements the `builds metadata items` step within the module flow.
**CN:** 该函数 `build_metadata_items` 实现了模块流程中的“构建metadataitems”步骤。

### Lines 69-72 — Function `encode_pooling_output_float`
```python
def encode_pooling_output_float(
    output: PoolingRequestOutput,
) -> FloatEncodedPoolingOutput:
    return output.outputs.data.tolist()
```
**EN:** This function `encode_pooling_output_float` implements the `encode pooling output float` step within the module flow.
**CN:** 该函数 `encode_pooling_output_float` 实现了模块流程中的“encode池化outputfloat”步骤。

### Lines 75-83 — Function `encode_pooling_output_float_or_ndarray`
```python
def encode_pooling_output_float_or_ndarray(output: PoolingRequestOutput) -> Any:
    """Return an ndarray when the response renderer can serialize NumPy."""
    try:
        data = output.outputs.data
        if not data.is_contiguous():
            data = data.contiguous()
        return data.numpy()
    except (RuntimeError, TypeError):
        return output.outputs.data.tolist()
```
**EN:** This function `encode_pooling_output_float_or_ndarray` is documented as: Return an ndarray when the response renderer can serialize NumPy.
**CN:** 这里定义函数 `encode_pooling_output_float_or_ndarray`，其文档字符串说明了主要职责与调用约定。

### Lines 86-92 — Function `encode_pooling_output_base64`
```python
def encode_pooling_output_base64(
    output: PoolingRequestOutput,
    embed_dtype: EmbedDType,
    endianness: Endianness,
) -> str:
    embedding_bytes = tensor2binary(output.outputs.data, embed_dtype, endianness)
    return pybase64.b64encode(embedding_bytes).decode("utf-8")
```
**EN:** This function `encode_pooling_output_base64` implements the `encode pooling output base64` step within the module flow.
**CN:** 该函数 `encode_pooling_output_base64` 实现了模块流程中的“encode池化outputbase64”步骤。

### Lines 95-125 — Function `encode_pooling_bytes`
```python
def encode_pooling_bytes(
    pooling_outputs: list[PoolingRequestOutput],
    embed_dtype: EmbedDType,
    endianness: Endianness,
) -> tuple[list[bytes], list[dict[str, Any]]]:
    items: list[dict[str, Any]] = []
    body: list[bytes] = []
    offset = 0
    for idx, output in enumerate(pooling_outputs):
        binary = tensor2binary(
            tensor=output.outputs.data,
            embed_dtype=embed_dtype,
            endianness=endianness,
        )
        size = len(binary)

        # Dictionary form of MetadataItem
        item = dict(
            index=idx,
            embed_dtype=embed_dtype,
            endianness=endianness,
            start=offset,
            end=offset + size,
            shape=output.outputs.data.shape,
        )

        body.append(binary)
        items.append(item)
        offset += size

    return body, items
```
**EN:** This function `encode_pooling_bytes` implements the `encode pooling bytes` step within the module flow.
**CN:** 该函数 `encode_pooling_bytes` 实现了模块流程中的“encode池化bytes”步骤。

### Lines 128-144 — Function `get_pooling_output_encoder`
```python
def get_pooling_output_encoder(
    encoding_format: JsonEncodingFormat,
    embed_dtype: EmbedDType,
    endianness: Endianness,
) -> Callable[[PoolingRequestOutput], JsonEncodedPoolingOutput]:
    return cast(
        Callable[[PoolingRequestOutput], JsonEncodedPoolingOutput],
        (
            encode_pooling_output_float
            if encoding_format == "float"
            else partial(
                encode_pooling_output_base64,
                embed_dtype=embed_dtype,
                endianness=endianness,
            )
        ),
    )
```
**EN:** This function `get_pooling_output_encoder` implements the `gets pooling output encoder` step within the module flow.
**CN:** 该函数 `get_pooling_output_encoder` 实现了模块流程中的“获取池化outputencoder”步骤。

### Lines 147-157 — Function `get_pooling_usage`
```python
def get_pooling_usage(
    pooling_outputs: Sequence[PoolingRequestOutput],
) -> UsageInfo:
    num_prompt_tokens = sum(
        len(output.prompt_token_ids) if output.prompt_token_ids is not None else 0
        for output in pooling_outputs
    )
    return UsageInfo(
        prompt_tokens=num_prompt_tokens,
        total_tokens=num_prompt_tokens,
    )
```
**EN:** This function `get_pooling_usage` implements the `gets pooling usage` step within the module flow.
**CN:** 该函数 `get_pooling_usage` 实现了模块流程中的“获取池化usage”步骤。

### Lines 160-167 — Function `get_pooling_usage_payload`
```python
def get_pooling_usage_payload(
    pooling_outputs: Sequence[PoolingRequestOutput],
) -> dict[str, int]:
    usage = get_pooling_usage(pooling_outputs)
    return {
        "prompt_tokens": usage.prompt_tokens,
        "total_tokens": usage.total_tokens,
    }
```
**EN:** This function `get_pooling_usage_payload` implements the `gets pooling usage payload` step within the module flow.
**CN:** 该函数 `get_pooling_usage_payload` 实现了模块流程中的“获取池化usagepayload”步骤。

### Lines 170-205 — Function `build_pooling_bytes_streaming_response`
```python
def build_pooling_bytes_streaming_response(
    pooling_outputs: list[PoolingRequestOutput],
    request_id: str,
    created_time: int,
    model_name: str,
    encoding_format: BytesEncodingFormat,
    embed_dtype: EmbedDType,
    endianness: Endianness,
) -> StreamingResponse:
    content, items = encode_pooling_bytes(
        pooling_outputs=pooling_outputs,
        embed_dtype=embed_dtype,
        endianness=endianness,
    )

    headers = (
        None
        if encoding_format == "bytes_only"
        else {
            "metadata": json.dumps(
                {
                    "id": request_id,
                    "created": created_time,
                    "model": model_name,
                    "data": items,
                    "usage": get_pooling_usage_payload(pooling_outputs),
                }
            )
        }
    )

    return StreamingResponse(
        content=content,
        headers=headers,
        media_type="application/octet-stream",
    )
```
**EN:** This function `build_pooling_bytes_streaming_response` serializes API responses.
**CN:** 该函数 `build_pooling_bytes_streaming_response` 序列化 API 响应。

### Lines 208-217 — Function `decode_pooling_output`
```python
def decode_pooling_output(items: list[MetadataItem], body: bytes) -> list[torch.Tensor]:
    return [
        binary2tensor(
            body[item.start : item.end],
            item.shape,
            item.embed_dtype,
            item.endianness,
        )
        for item in sorted(items, key=lambda x: x.index)
    ]
```
**EN:** This function `decode_pooling_output` implements the `decode pooling output` step within the module flow.
**CN:** 该函数 `decode_pooling_output` 实现了模块流程中的“decode池化output”步骤。

### Lines 221-229 — Function `get_json_response_cls`
```python
def get_json_response_cls() -> type[JSONResponse]:
    if importlib.util.find_spec("orjson") is not None:
        from fastapi.responses import ORJSONResponse

        return ORJSONResponse
    logger.warning_once(
        "To make v1/embeddings API fast, please install orjson by `pip install orjson`"
    )
    return JSONResponse
```
**EN:** This function `get_json_response_cls` implements the `gets json response cls` step within the module flow.
**CN:** 该函数 `get_json_response_cls` 实现了模块流程中的“获取json响应cls”步骤。

### Lines 232-250 — Function `enable_scoring_api`
```python
def enable_scoring_api(
    supported_tasks: tuple["SupportedTask", ...],
    model_config: ModelConfig | None = None,
) -> bool:
    if model_config is None:
        return False

    pooling_task = model_config.get_pooling_task(supported_tasks)
    if pooling_task in ("embed", "token_embed"):
        return True

    if pooling_task == "classify":
        num_labels = getattr(model_config.hf_config, "num_labels", 0)
        if num_labels != 1:
            logger.debug_once("Scoring API is only enabled for num_labels == 1.")
            return False
        return True

    return False
```
**EN:** This function `enable_scoring_api` implements the `enable scoring API` step within the module flow.
**CN:** 该函数 `enable_scoring_api` 实现了模块流程中的“enable评分API”步骤。

## Key Concepts / 关键概念
- FastAPI routing and application assembly / FastAPI 路由与应用组装
- Streaming responses / 流式响应
- Tokenization or token-level processing / 分词或 token 级处理
- Prompt or multimodal rendering / 提示词或多模态渲染
- Pooling task support / 池化任务支持
- Embedding generation / 嵌入生成
- Pooling task workflows / 池化任务工作流

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `importlib`, `json`, `math`, `collections`, `dataclasses`, `functools`, `typing`
- **Third-party / 第三方**: `pybase64`, `torch`, `fastapi`
- **vLLM Internal / vLLM 内部**: `vllm.config`, `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.outputs`, `vllm.tasks`, `vllm.utils.serial_utils`
