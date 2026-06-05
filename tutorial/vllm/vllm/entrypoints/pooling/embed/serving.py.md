# serving.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/pooling/embed/serving.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the serving layer that coordinates preprocessing, engine calls, and response shaping. Scope: pooling embedding. / 实现服务层，负责预处理、引擎调用与响应整形。 作用域：池化 / 嵌入。

## Line-by-Line Analysis / 逐行分析
### Lines 3-34 — Imports and shared dependencies
```python
from typing import TypeAlias, cast

from fastapi.responses import JSONResponse, Response, StreamingResponse
from typing_extensions import assert_never

from vllm.logger import init_logger
from vllm.outputs import PoolingRequestOutput
from vllm.utils.serial_utils import EmbedDType, Endianness

from ..base.serving import PoolingServing
from ..typing import PoolingServeContext
from ..utils import (
    BytesEncodingFormat,
    JsonEncodingFormat,
    build_pooling_bytes_streaming_response,
    encode_pooling_output_float,
    encode_pooling_output_float_or_ndarray,
    get_json_response_cls,
    get_pooling_output_encoder,
    get_pooling_usage,
)
from .io_processor import EmbedIOProcessor
from .protocol import (
    CohereBilledUnits,
    CohereEmbedRequest,
    CohereEmbedResponse,
    CohereMeta,
    EmbeddingRequest,
    EmbeddingResponse,
    EmbeddingResponseData,
    build_typed_embeddings,
)
```
**EN:** This import block pulls in standard-library modules such as `typing`, uses third-party packages like `fastapi`, `typing_extensions`, depends on internal helpers such as `vllm.logger`, `vllm.outputs`, `vllm.utils.serial_utils`, `..base.serving`, `..typing`, `..utils`.
**CN:** 该导入块引入 `typing` 等标准库模块，使用 `fastapi`, `typing_extensions` 等第三方库，依赖 `vllm.logger`, `vllm.outputs`, `vllm.utils.serial_utils`, `..base.serving`, `..typing`, `..utils` 等 vLLM 内部模块。

### Lines 36-39 — Shared module state
```python
logger = init_logger(__name__)


EmbeddingServeContext: TypeAlias = PoolingServeContext[EmbeddingRequest]
```
**EN:** This block initializes `logger`, `EmbeddingServeContext`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`, `EmbeddingServeContext`，为后续逻辑准备模块级常量或共享状态。

### Lines 42-209 — Class `ServingEmbedding`
```python
class ServingEmbedding(PoolingServing):
    """Embedding API supporting both OpenAI and Cohere formats."""

    request_id_prefix = "embd"
    io_processor: EmbedIOProcessor

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

        self.json_response_cls = get_json_response_cls()

    def init_io_processor(self, *args, **kwargs) -> EmbedIOProcessor:
        return EmbedIOProcessor(*args, **kwargs)

    def _build_response(
        self,
        ctx: PoolingServeContext,
    ) -> Response:
        if isinstance(ctx.request, CohereEmbedRequest):
            return self._build_cohere_response_from_ctx(ctx)
        return self._build_openai_response(ctx)

    def _build_openai_response(
        self,
        ctx: EmbeddingServeContext,
    ) -> JSONResponse | StreamingResponse:
        encoding_format = ctx.request.encoding_format
        embed_dtype = ctx.request.embed_dtype
...
            meta=CohereMeta(
                billed_units=CohereBilledUnits(
                    input_tokens=input_tokens,
                    image_tokens=image_tokens,
                ),
            ),
        )
        return self.json_response_cls(content=response.model_dump(exclude_none=True))
```
**EN:** Class `ServingEmbedding` is introduced here. Its docstring describes the intent as: Embedding API supporting both OpenAI and Cohere formats.
**CN:** 这里定义类 `ServingEmbedding`。其文档字符串说明了该类的职责与使用方式。

## Key Concepts / 关键概念
- FastAPI routing and application assembly / FastAPI 路由与应用组装
- Streaming responses / 流式响应
- Tokenization or token-level processing / 分词或 token 级处理
- Batch workflow handling / 批处理工作流
- Pooling task support / 池化任务支持
- Embedding generation / 嵌入生成
- Pooling task workflows / 池化任务工作流

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `typing`
- **Third-party / 第三方**: `fastapi`, `typing_extensions`
- **vLLM Internal / vLLM 内部**: `vllm.logger`, `vllm.outputs`, `vllm.utils.serial_utils`, `..base.serving`, `..typing`, `..utils`, `.io_processor`, `.protocol`
