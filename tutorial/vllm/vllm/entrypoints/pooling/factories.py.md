# factories.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/pooling/factories.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Constructs and registers feature-specific components and routers. Scope: pooling. / 构建并注册特定功能组件与路由。 作用域：池化。

## Line-by-Line Analysis / 逐行分析
### Lines 4-16 — Imports and shared dependencies
```python
from typing import TYPE_CHECKING

from fastapi import FastAPI

from vllm.config import ModelConfig, VllmConfig
from vllm.entrypoints.chat_utils import ChatTemplateConfig
from vllm.logger import init_logger
from vllm.plugins.io_processors import has_io_processor
from vllm.renderers import BaseRenderer
from vllm.tasks import POOLING_TASKS, SCORE_TYPE_MAP, SupportedTask

from .base.io_processor import PoolingIOProcessor
from .utils import enable_scoring_api
```
**EN:** This import block pulls in standard-library modules such as `typing`, uses third-party packages like `fastapi`, depends on internal helpers such as `vllm.config`, `vllm.entrypoints.chat_utils`, `vllm.logger`, `vllm.plugins.io_processors`, `vllm.renderers`, `vllm.tasks`.
**CN:** 该导入块引入 `typing` 等标准库模块，使用 `fastapi` 等第三方库，依赖 `vllm.config`, `vllm.entrypoints.chat_utils`, `vllm.logger`, `vllm.plugins.io_processors`, `vllm.renderers`, `vllm.tasks` 等 vLLM 内部模块。

### Lines 18-32 — Conditional top-level flow
```python
if TYPE_CHECKING:
    from argparse import Namespace

    from starlette.datastructures import State

    from vllm.engine.protocol import EngineClient
    from vllm.entrypoints.logger import RequestLogger
    from vllm.entrypoints.sagemaker.api_router import (
        EndpointFn,
        GetHandlerFn,
        RequestType,
    )

else:
    RequestLogger = object
```
**EN:** This top-level conditional toggles behavior based on runtime configuration or platform state.
**CN:** 该顶层条件分支会根据运行时配置或平台状态切换行为。

### Lines 35-35 — Shared module state
```python
logger = init_logger(__name__)
```
**EN:** This block initializes `logger`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`，为后续逻辑准备模块级常量或共享状态。

### Lines 38-101 — Function `init_pooling_io_processors`
```python
def init_pooling_io_processors(
    supported_tasks: tuple[SupportedTask, ...],
    vllm_config: VllmConfig,
    renderer: BaseRenderer,
    chat_template_config: ChatTemplateConfig,
) -> dict[str, PoolingIOProcessor]:
    model_config = vllm_config.model_config
    processors: dict[str, type[PoolingIOProcessor]] = {}
    pooling_task = model_config.get_pooling_task(supported_tasks)

    if pooling_task == "classify":
        from .classify.io_processor import ClassifyIOProcessor

        processors["classify"] = ClassifyIOProcessor

    if pooling_task == "token_classify":
        from .classify.io_processor import TokenClassifyIOProcessor

        processors["token_classify"] = TokenClassifyIOProcessor

    if pooling_task == "embed":
        from .embed.io_processor import EmbedIOProcessor

        processors["embed"] = EmbedIOProcessor

    if pooling_task == "token_embed":
        from .embed.io_processor import TokenEmbedIOProcessor

...
    return {
        task: processor_cls(
            vllm_config=vllm_config,
            renderer=renderer,
            chat_template_config=chat_template_config,
        )
        for task, processor_cls in processors.items()
    }
```
**EN:** This function `init_pooling_io_processors` implements the `initializes pooling io processors` step within the module flow.
**CN:** 该函数 `init_pooling_io_processors` 实现了模块流程中的“初始化池化ioprocessors”步骤。

### Lines 104-134 — Function `register_pooling_api_routers`
```python
def register_pooling_api_routers(
    app: FastAPI,
    supported_tasks: tuple["SupportedTask", ...],
    model_config: ModelConfig | None = None,
):
    if model_config is None:
        return

    pooling_task = model_config.get_pooling_task(supported_tasks)

    if pooling_task is not None:
        from .pooling.api_router import router as pooling_router

        app.include_router(pooling_router)

    if "classify" in supported_tasks:
        from .classify.api_router import (
            router as classify_router,
        )

        app.include_router(classify_router)

    if "embed" in supported_tasks:
        from .embed.api_router import router as embed_router

        app.include_router(embed_router)

    if enable_scoring_api(supported_tasks, model_config):
        from .scoring.api_router import router as score_router

        app.include_router(score_router)
```
**EN:** This function `register_pooling_api_routers` implements the `registers pooling API routers` step within the module flow.
**CN:** 该函数 `register_pooling_api_routers` 实现了模块流程中的“注册池化APIrouters”步骤。

### Lines 137-211 — Function `init_pooling_state`
```python
def init_pooling_state(
    engine_client: "EngineClient",
    state: "State",
    args: "Namespace",
    request_logger: RequestLogger | None,
    supported_tasks: tuple["SupportedTask", ...],
):
    model_config = engine_client.model_config
    if model_config is None:
        return

    from vllm.entrypoints.chat_utils import load_chat_template
    from vllm.tasks import POOLING_TASKS

    from .classify.serving import ServingClassification
    from .embed.serving import ServingEmbedding
    from .pooling.serving import ServingPooling
    from .scoring.serving import ServingScores

    resolved_chat_template = load_chat_template(args.chat_template)
    pooling_task = model_config.get_pooling_task(supported_tasks)

    chat_template_config = ChatTemplateConfig(
        chat_template=resolved_chat_template,
        chat_template_content_format=args.chat_template_content_format,
        trust_request_chat_template=args.trust_request_chat_template,
    )

...
            chat_template_config=chat_template_config,
            enable_flash_late_interaction=getattr(
                args, "enable_flash_late_interaction", True
            ),
        )
        if enable_scoring_api(supported_tasks, model_config)
        else None
    )
```
**EN:** This function `init_pooling_state` interacts with the model engine.
**CN:** 该函数 `init_pooling_state` 与模型引擎交互。

### Lines 214-265 — Function `get_pooling_invocation_types`
```python
def get_pooling_invocation_types(
    supported_tasks: tuple["SupportedTask", ...],
    model_config: ModelConfig | None = None,
):
    # NOTE: Items defined earlier take higher priority
    invocation_types: list[tuple[RequestType, tuple[GetHandlerFn, EndpointFn]]] = []

    if model_config is None:
        return invocation_types

    pooling_task = model_config.get_pooling_task(supported_tasks)

    if pooling_task == "embed":
        from .embed.api_router import create_embedding, embedding
        from .embed.protocol import EmbeddingRequest

        invocation_types += [
            (EmbeddingRequest, (embedding, create_embedding)),
        ]

    if pooling_task == "classify":
        from .classify.api_router import classify, create_classify
        from .classify.protocol import ClassificationRequest

        invocation_types += [
            (ClassificationRequest, (classify, create_classify)),
        ]

...
        from .pooling.api_router import create_pooling, pooling
        from .pooling.protocol import PoolingRequest

        invocation_types += [
            (PoolingRequest, (pooling, create_pooling)),
        ]

    return invocation_types
```
**EN:** This function `get_pooling_invocation_types` implements the `gets pooling invocation types` step within the module flow.
**CN:** 该函数 `get_pooling_invocation_types` 实现了模块流程中的“获取池化invocationtypes”步骤。

## Key Concepts / 关键概念
- FastAPI routing and application assembly / FastAPI 路由与应用组装
- Engine-backed serving orchestration / 基于引擎的服务编排
- Tokenization or token-level processing / 分词或 token 级处理
- Prompt or multimodal rendering / 提示词或多模态渲染
- Pooling task support / 池化任务支持
- Embedding generation / 嵌入生成
- Scoring/ranking logic / 评分/排序逻辑
- Pooling task workflows / 池化任务工作流

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `typing`, `argparse`
- **Third-party / 第三方**: `fastapi`, `starlette`
- **vLLM Internal / vLLM 内部**: `vllm.config`, `vllm.entrypoints.chat_utils`, `vllm.logger`, `vllm.plugins.io_processors`, `vllm.renderers`, `vllm.tasks`, `.base.io_processor`, `.utils`, `vllm.engine.protocol`, `vllm.entrypoints.logger`, `vllm.entrypoints.sagemaker.api_router`, `.classify.serving`
