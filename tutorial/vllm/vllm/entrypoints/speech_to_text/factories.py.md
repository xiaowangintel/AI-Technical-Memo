# factories.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/speech_to_text/factories.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Constructs and registers feature-specific components and routers. Scope: speech-to-text. / 构建并注册特定功能组件与路由。 作用域：语音转文本。

## Line-by-Line Analysis / 逐行分析
### Lines 5-7 — Imports and shared dependencies
```python
from typing import TYPE_CHECKING

from fastapi import FastAPI
```
**EN:** This import block pulls in standard-library modules such as `typing`, uses third-party packages like `fastapi`.
**CN:** 该导入块引入 `typing` 等标准库模块，使用 `fastapi` 等第三方库。

### Lines 9-18 — Conditional top-level flow
```python
if TYPE_CHECKING:
    from argparse import Namespace

    from starlette.datastructures import State

    from vllm.engine.protocol import EngineClient
    from vllm.entrypoints.logger import RequestLogger
    from vllm.tasks import SupportedTask
else:
    RequestLogger = object
```
**EN:** This top-level conditional toggles behavior based on runtime configuration or platform state.
**CN:** 该顶层条件分支会根据运行时配置或平台状态切换行为。

### Lines 21-37 — Function `register_speech_to_text_api_routers`
```python
def register_speech_to_text_api_routers(
    app: FastAPI,
    supported_tasks: tuple["SupportedTask", ...],
):
    if "realtime" in supported_tasks:
        from .realtime.api_router import router as realtime_router

        app.include_router(realtime_router)

    if "transcription" in supported_tasks:
        from .transcription.api_router import router as transcription_router

        app.include_router(transcription_router)

        from .translation.api_router import router as translation_router

        app.include_router(translation_router)
```
**EN:** This function `register_speech_to_text_api_routers` implements the `registers speech converts to text API routers` step within the module flow.
**CN:** 该函数 `register_speech_to_text_api_routers` 实现了模块流程中的“注册语音转换为文本APIrouters”步骤。

### Lines 40-43 — Function `add_websocket_metrics_middleware`
```python
def add_websocket_metrics_middleware(app: FastAPI):
    from .realtime.metrics import WebSocketMetricsMiddleware

    app.add_middleware(WebSocketMetricsMiddleware)
```
**EN:** This function `add_websocket_metrics_middleware` installs middleware behavior, prepares socket resources.
**CN:** 该函数 `add_websocket_metrics_middleware` 安装中间件行为，准备套接字资源。

### Lines 46-79 — Function `init_speech_to_text_state`
```python
def init_speech_to_text_state(
    engine_client: "EngineClient",
    state: "State",
    args: "Namespace",
    request_logger: RequestLogger | None,
    supported_tasks: tuple["SupportedTask", ...],
):
    if "transcription" in supported_tasks:
        from .transcription.serving import OpenAIServingTranscription

        state.openai_serving_transcription = OpenAIServingTranscription(
            engine_client,
            state.openai_serving_models,
            request_logger=request_logger,
            enable_force_include_usage=args.enable_force_include_usage,
        )

        from .translation.serving import OpenAIServingTranslation

        state.openai_serving_translation = OpenAIServingTranslation(
            engine_client,
            state.openai_serving_models,
            request_logger=request_logger,
            enable_force_include_usage=args.enable_force_include_usage,
        )

    if "realtime" in supported_tasks:
        from .realtime.serving import OpenAIServingRealtime

        state.openai_serving_realtime = OpenAIServingRealtime(
            engine_client,
            state.openai_serving_models,
            request_logger=request_logger,
        )
```
**EN:** This function `init_speech_to_text_state` interacts with the model engine.
**CN:** 该函数 `init_speech_to_text_state` 与模型引擎交互。

## Key Concepts / 关键概念
- FastAPI routing and application assembly / FastAPI 路由与应用组装
- Engine-backed serving orchestration / 基于引擎的服务编排
- Realtime/WebSocket handling / 实时/WebSocket 处理
- Metrics and observability / 指标与可观测性
- Middleware pipeline / 中间件链路
- Speech transcription flow / 语音转写流程
- Speech translation flow / 语音翻译流程
- Realtime session flow / 实时会话流程

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `typing`, `argparse`
- **Third-party / 第三方**: `fastapi`, `starlette`
- **vLLM Internal / vLLM 内部**: `vllm.engine.protocol`, `vllm.entrypoints.logger`, `vllm.tasks`, `.realtime.metrics`, `.realtime.api_router`, `.transcription.api_router`, `.translation.api_router`, `.transcription.serving`, `.translation.serving`, `.realtime.serving`
