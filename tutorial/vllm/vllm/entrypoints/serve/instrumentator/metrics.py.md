# metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/serve/instrumentator/metrics.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Defines observability and metrics helpers for runtime behavior. Scope: serving instrumentation. / 定义运行时行为相关的可观测性与指标辅助逻辑。 作用域：服务扩展 / 可观测性。

## Line-by-Line Analysis / 逐行分析
### Lines 5-12 — Imports and shared dependencies
```python
import prometheus_client
import regex as re
from fastapi import FastAPI, Response
from prometheus_client import make_asgi_app
from prometheus_fastapi_instrumentator import Instrumentator
from starlette.routing import Mount

from vllm.v1.metrics.prometheus import get_prometheus_registry
```
**EN:** This import block uses third-party packages like `prometheus_client`, `regex`, `fastapi`, `prometheus_fastapi_instrumentator`, `starlette`, depends on internal helpers such as `vllm.v1.metrics.prometheus`.
**CN:** 该导入块使用 `prometheus_client`, `regex`, `fastapi`, `prometheus_fastapi_instrumentator`, `starlette` 等第三方库，依赖 `vllm.v1.metrics.prometheus` 等 vLLM 内部模块。

### Lines 15-16 — Class `PrometheusResponse`
```python
class PrometheusResponse(Response):
    media_type = prometheus_client.CONTENT_TYPE_LATEST
```
**EN:** Class `PrometheusResponse` acts as a protocol/data model with about 1 field declarations and helper methods such as its inherited interface.
**CN:** 类 `PrometheusResponse` 充当协议/数据模型，包含约 1 个字段声明，并提供 继承接口 等辅助方法。

### Lines 19-45 — Function `attach_router`
```python
def attach_router(app: FastAPI):
    """Mount prometheus metrics to a FastAPI app."""

    registry = get_prometheus_registry()

    # `response_class=PrometheusResponse` is needed to return an HTTP response
    # with header "Content-Type: text/plain; version=0.0.4; charset=utf-8"
    # instead of the default "application/json" which is incorrect.
    # See https://github.com/trallnag/prometheus-fastapi-instrumentator/issues/163#issue-1296092364
    Instrumentator(
        excluded_handlers=[
            "/metrics",
            "/health",
            "/load",
            "/ping",
            "/version",
            "/server_info",
        ],
        registry=registry,
    ).add().instrument(app).expose(app, response_class=PrometheusResponse)

    # Add prometheus asgi middleware to route /metrics requests
    metrics_route = Mount("/metrics", make_asgi_app(registry=registry))

    # Workaround for 307 Redirect for /metrics
    metrics_route.path_regex = re.compile("^/metrics(?P<path>.*)$")
    app.routes.append(metrics_route)
```
**EN:** This function `attach_router` is documented as: Mount prometheus metrics to a FastAPI app.
**CN:** 这里定义函数 `attach_router`，其文档字符串说明了主要职责与调用约定。

## Key Concepts / 关键概念
- FastAPI routing and application assembly / FastAPI 路由与应用组装
- Metrics and observability / 指标与可观测性
- Middleware pipeline / 中间件链路
- Auxiliary serving endpoints / 辅助服务端点

## Dependencies / 依赖关系
- **Third-party / 第三方**: `prometheus_client`, `regex`, `fastapi`, `prometheus_fastapi_instrumentator`, `starlette`
- **vLLM Internal / vLLM 内部**: `vllm.v1.metrics.prometheus`
