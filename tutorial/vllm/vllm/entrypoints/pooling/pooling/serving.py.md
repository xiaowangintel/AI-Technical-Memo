# serving.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/pooling/pooling/serving.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the serving layer that coordinates preprocessing, engine calls, and response shaping. Scope: pooling pooling. / 实现服务层，负责预处理、引擎调用与响应整形。 作用域：池化 / 池化。

## Line-by-Line Analysis / 逐行分析
### Lines 3-28 — Imports and shared dependencies
```python
from fastapi.responses import JSONResponse, Response, StreamingResponse
from typing_extensions import assert_never

from vllm.logger import init_logger
from vllm.outputs import PoolingRequestOutput
from vllm.tasks import SupportedTask
from vllm.utils.serial_utils import EmbedDType, Endianness

from ..base.io_processor import PoolingIOProcessor
from ..base.serving import PoolingServingBase
from ..factories import init_pooling_io_processors
from ..typing import AnyPoolingRequest, PoolingServeContext
from ..utils import (
    BytesEncodingFormat,
    JsonEncodingFormat,
    build_pooling_bytes_streaming_response,
    get_json_response_cls,
    get_pooling_output_encoder,
    get_pooling_usage,
)
from .protocol import (
    IOProcessorRequest,
    PoolingRequest,
    PoolingResponse,
    PoolingResponseData,
)
```
**EN:** This import block uses third-party packages like `fastapi`, `typing_extensions`, depends on internal helpers such as `vllm.logger`, `vllm.outputs`, `vllm.tasks`, `vllm.utils.serial_utils`, `..base.io_processor`, `..base.serving`.
**CN:** 该导入块使用 `fastapi`, `typing_extensions` 等第三方库，依赖 `vllm.logger`, `vllm.outputs`, `vllm.tasks`, `vllm.utils.serial_utils`, `..base.io_processor`, `..base.serving` 等 vLLM 内部模块。

### Lines 30-30 — Shared module state
```python
logger = init_logger(__name__)
```
**EN:** This block initializes `logger`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`，为后续逻辑准备模块级常量或共享状态。

### Lines 33-184 — Class `ServingPooling`
```python
class ServingPooling(PoolingServingBase):
    request_id_prefix = "pooling"

    def __init__(
        self,
        *args,
        supported_tasks: tuple[SupportedTask, ...],
        **kwargs,
    ):
        super().__init__(*args, **kwargs)

        self.supported_tasks = supported_tasks
        self.pooling_task = self.model_config.get_pooling_task(supported_tasks)
        self.io_processors = init_pooling_io_processors(
            supported_tasks=supported_tasks,
            vllm_config=self.vllm_config,
            renderer=self.renderer,
            chat_template_config=self.chat_template_config,
        )
        self.json_response_cls = get_json_response_cls()

    def get_io_processor(self, request: AnyPoolingRequest) -> PoolingIOProcessor:
        assert isinstance(request, PoolingRequest)
        pooling_task = self._verify_pooling_task(request)
        return self.io_processors[pooling_task]

    def _verify_pooling_task(self, request: PoolingRequest) -> str:
        if getattr(request, "dimensions", None) is not None:
...
            pooling_outputs=final_res_batch,
            request_id=request_id,
            created_time=created_time,
            model_name=model_name,
            encoding_format=encoding_format,
            embed_dtype=embed_dtype,
            endianness=endianness,
        )
```
**EN:** Class `ServingPooling` is defined here, extending `PoolingServingBase`, and groups behavior through methods like `__init__`, `get_io_processor`, `_verify_pooling_task`, `_build_response`.
**CN:** 这里定义类 `ServingPooling`，其职责是封装相关状态与行为，并通过 `__init__`、`get_io_processor`、`_verify_pooling_task`、`_build_response` 等方法组织逻辑。

## Key Concepts / 关键概念
- FastAPI routing and application assembly / FastAPI 路由与应用组装
- Streaming responses / 流式响应
- Prompt or multimodal rendering / 提示词或多模态渲染
- Batch workflow handling / 批处理工作流
- Pooling task support / 池化任务支持
- Embedding generation / 嵌入生成
- Pooling task workflows / 池化任务工作流

## Dependencies / 依赖关系
- **Third-party / 第三方**: `fastapi`, `typing_extensions`
- **vLLM Internal / vLLM 内部**: `vllm.logger`, `vllm.outputs`, `vllm.tasks`, `vllm.utils.serial_utils`, `..base.io_processor`, `..base.serving`, `..factories`, `..typing`, `..utils`, `.protocol`
