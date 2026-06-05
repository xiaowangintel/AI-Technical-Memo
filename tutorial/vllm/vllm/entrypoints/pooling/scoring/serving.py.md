# serving.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/pooling/scoring/serving.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the serving layer that coordinates preprocessing, engine calls, and response shaping. Scope: pooling scoring. / 实现服务层，负责预处理、引擎调用与响应整形。 作用域：池化 / 评分。

## Line-by-Line Analysis / 逐行分析
### Lines 4-30 — Imports and shared dependencies
```python
from fastapi.responses import JSONResponse, Response

from vllm import PoolingParams
from vllm.engine.protocol import EngineClient
from vllm.entrypoints.openai.engine.protocol import UsageInfo
from vllm.logger import init_logger
from vllm.outputs import PoolingRequestOutput, ScoringRequestOutput
from vllm.tasks import SCORE_TYPE_MAP, SupportedTask
from vllm.v1.pool.late_interaction import (
    build_late_interaction_doc_params,
    build_late_interaction_query_params,
)

from ..base.io_processor import PoolingIOProcessor
from ..base.serving import PoolingServing
from .io_processor import ScoringIOProcessors, ScoringServeContext
from .protocol import (
    RerankDocument,
    RerankRequest,
    RerankResponse,
    RerankResult,
    RerankUsage,
    ScoreRequest,
    ScoreResponse,
    ScoreResponseData,
)
from .typing import ScoreInput
```
**EN:** This import block uses third-party packages like `fastapi`, depends on internal helpers such as `vllm`, `vllm.engine.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.outputs`, `vllm.tasks`.
**CN:** 该导入块使用 `fastapi` 等第三方库，依赖 `vllm`, `vllm.engine.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.outputs`, `vllm.tasks` 等 vLLM 内部模块。

### Lines 32-32 — Shared module state
```python
logger = init_logger(__name__)
```
**EN:** This block initializes `logger`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`，为后续逻辑准备模块级常量或共享状态。

### Lines 35-287 — Class `ServingScores`
```python
class ServingScores(PoolingServing):
    request_id_prefix = "score"

    def __init__(
        self,
        engine_client: EngineClient,
        *args,
        supported_tasks: tuple[SupportedTask, ...],
        enable_flash_late_interaction: bool = True,
        **kwargs,
    ):
        pooling_task = engine_client.model_config.get_pooling_task(supported_tasks)
        score_type = SCORE_TYPE_MAP.get(pooling_task, None)  # type: ignore[arg-type]
        assert score_type is not None

        self.io_processor_name: str = score_type
        self.enable_flash_late_interaction = (
            self.io_processor_name == "late-interaction"
            and enable_flash_late_interaction
        )

        if self.enable_flash_late_interaction:
            self.io_processor_name = "flash-late-interaction"

        if engine_client.model_config.architecture == "JinaForRanking":
            self.io_processor_name = "jina-reranking-scoring"
            self.enable_flash_late_interaction = False

...
            prompt_request_ids=doc_keys,
            engine_inputs=doc_engine_inputs,
        )

        await self._prepare_generators(doc_ctx)
        await self._collect_batch(doc_ctx)

        ctx.final_res_batch = doc_ctx.final_res_batch
```
**EN:** Class `ServingScores` is defined here, extending `PoolingServing`, and groups behavior through methods like `__init__`, `init_io_processor`, `__call__`, `_build_response`.
**CN:** 这里定义类 `ServingScores`，其职责是封装相关状态与行为，并通过 `__init__`、`init_io_processor`、`__call__`、`_build_response` 等方法组织逻辑。

## Key Concepts / 关键概念
- FastAPI routing and application assembly / FastAPI 路由与应用组装
- Async request handling / 异步请求处理
- Engine-backed serving orchestration / 基于引擎的服务编排
- Tokenization or token-level processing / 分词或 token 级处理
- Batch workflow handling / 批处理工作流
- Pooling task support / 池化任务支持
- Embedding generation / 嵌入生成
- Scoring/ranking logic / 评分/排序逻辑

## Dependencies / 依赖关系
- **Third-party / 第三方**: `fastapi`
- **vLLM Internal / vLLM 内部**: `vllm`, `vllm.engine.protocol`, `vllm.entrypoints.openai.engine.protocol`, `vllm.logger`, `vllm.outputs`, `vllm.tasks`, `vllm.v1.pool.late_interaction`, `..base.io_processor`, `..base.serving`, `.io_processor`, `.protocol`, `.typing`
