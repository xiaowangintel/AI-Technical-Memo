# serving.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/pooling/classify/serving.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the serving layer that coordinates preprocessing, engine calls, and response shaping. Scope: pooling classification. / 实现服务层，负责预处理、引擎调用与响应整形。 作用域：池化 / 分类。

## Line-by-Line Analysis / 逐行分析
### Lines 4-20 — Imports and shared dependencies
```python
from typing import TypeAlias

import numpy as np
from fastapi.responses import JSONResponse

from vllm.entrypoints.openai.engine.protocol import UsageInfo
from vllm.logger import init_logger
from vllm.outputs import ClassificationOutput

from ..base.serving import PoolingServing
from ..typing import PoolingServeContext
from .io_processor import ClassifyIOProcessor
from .protocol import (
    ClassificationData,
    ClassificationRequest,
    ClassificationResponse,
)
```
**EN:** This import block pulls in standard-library modules such as `typing`, uses third-party packages like `numpy`, `fastapi`, depends on internal helpers such as `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.outputs`, `..base.serving`, `..typing`, `.io_processor`.
**CN:** 该导入块引入 `typing` 等标准库模块，使用 `numpy`, `fastapi` 等第三方库，依赖 `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.outputs`, `..base.serving`, `..typing`, `.io_processor` 等 vLLM 内部模块。

### Lines 22-25 — Shared module state
```python
logger = init_logger(__name__)


ClassificationServeContext: TypeAlias = PoolingServeContext[ClassificationRequest]
```
**EN:** This block initializes `logger`, `ClassificationServeContext`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`, `ClassificationServeContext`，为后续逻辑准备模块级常量或共享状态。

### Lines 28-72 — Class `ServingClassification`
```python
class ServingClassification(PoolingServing):
    request_id_prefix = "classify"

    def init_io_processor(self, *args, **kwargs) -> ClassifyIOProcessor:
        return ClassifyIOProcessor(*args, **kwargs)

    def _build_response(
        self,
        ctx: ClassificationServeContext,
    ) -> JSONResponse:
        id2label = getattr(self.model_config.hf_config, "id2label", {})
        num_prompt_tokens = 0
        items: list[ClassificationData] = []
        for idx, final_res in enumerate(ctx.final_res_batch):
            classify_res = ClassificationOutput.from_base(final_res.outputs)

            probs = classify_res.probs
            predicted_index = int(np.argmax(probs))
            label = id2label.get(predicted_index)

            item = ClassificationData(
                index=idx,
                label=label,
                probs=probs,
                num_classes=len(probs),
            )

            items.append(item)
...
            id=ctx.request_id,
            created=ctx.created_time,
            model=ctx.model_name,
            data=items,
            usage=usage,
        )

        return JSONResponse(content=response.model_dump())
```
**EN:** Class `ServingClassification` is defined here, extending `PoolingServing`, and groups behavior through methods like `init_io_processor`, `_build_response`.
**CN:** 这里定义类 `ServingClassification`，其职责是封装相关状态与行为，并通过 `init_io_processor`、`_build_response` 等方法组织逻辑。

## Key Concepts / 关键概念
- FastAPI routing and application assembly / FastAPI 路由与应用组装
- Tokenization or token-level processing / 分词或 token 级处理
- Batch workflow handling / 批处理工作流
- Pooling task support / 池化任务支持
- Pooling task workflows / 池化任务工作流

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `typing`
- **Third-party / 第三方**: `numpy`, `fastapi`
- **vLLM Internal / vLLM 内部**: `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.outputs`, `..base.serving`, `..typing`, `.io_processor`, `.protocol`
