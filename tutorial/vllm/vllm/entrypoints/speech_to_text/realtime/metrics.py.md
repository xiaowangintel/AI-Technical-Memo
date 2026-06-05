# metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/speech_to_text/realtime/metrics.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Defines observability and metrics helpers for runtime behavior. Scope: speech-to-text realtime. / 定义运行时行为相关的可观测性与指标辅助逻辑。 作用域：语音转文本 / 实时。

## Line-by-Line Analysis / 逐行分析
### Lines 3-12 — Module overview
```python
"""ASGI middleware for WebSocket Prometheus metrics.

Modeled after prometheus-fastapi-instrumentator, this middleware
transparently instruments WebSocket endpoints with standard metrics
without requiring changes to handler code.

NOTE: This module intentionally has zero vllm imports so that it can
be extracted into a standalone package (similar to
prometheus-fastapi-instrumentator) in the future. Please keep it that way.
"""
```
**EN:** The module header documents the main contract: ASGI middleware for WebSocket Prometheus metrics.
**CN:** 模块开头的文档字符串说明了该文件的主要职责、使用边界或运行约束。

### Lines 14-18 — Imports and shared dependencies
```python
import time
from collections.abc import Awaitable

from prometheus_client import Counter, Gauge, Histogram
from starlette.types import ASGIApp, Message, Receive, Scope, Send
```
**EN:** This import block pulls in standard-library modules such as `time`, `collections`, uses third-party packages like `prometheus_client`, `starlette`.
**CN:** 该导入块引入 `time`, `collections` 等标准库模块，使用 `prometheus_client`, `starlette` 等第三方库。

### Lines 22-37 — Module constants and state
```python
_active_sessions = Gauge(
    name="vllm:websocket_connections_active",
    documentation="Number of currently active WebSocket connections.",
    multiprocess_mode="livesum",
)

_total_sessions = Counter(
    name="vllm:websocket_connections_total",
    documentation="Total number of WebSocket connections.",
)

_session_duration = Histogram(
    name="vllm:websocket_connection_duration_seconds",
    documentation="Duration of WebSocket connections in seconds.",
    buckets=[0.5, 1, 2.5, 5, 10, 30, 60, 120, 300, 600, 1800],
)
```
**EN:** This block initializes `_active_sessions`, `_total_sessions`, `_session_duration`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `_active_sessions`, `_total_sessions`, `_session_duration`，为后续逻辑准备模块级常量或共享状态。

### Lines 40-78 — Class `WebSocketMetricsMiddleware`
```python
class WebSocketMetricsMiddleware:
    """Pure ASGI middleware that instruments WebSocket connections.

    Tracks active connections (gauge), total connections (counter),
    and connection duration (histogram) for all WebSocket endpoints.

    Usage::

        app.add_middleware(WebSocketMetricsMiddleware)
    """

    def __init__(self, app: ASGIApp) -> None:
        self.app = app

    def __call__(self, scope: Scope, receive: Receive, send: Send) -> Awaitable[None]:
        if scope["type"] != "websocket":
            return self.app(scope, receive, send)

        return self._handle_websocket(scope, receive, send)

    async def _handle_websocket(
        self, scope: Scope, receive: Receive, send: Send
    ) -> None:
        start_time: float | None = None

        async def send_wrapper(message: Message) -> None:
            nonlocal start_time
            if message["type"] == "websocket.accept":
                start_time = time.monotonic()
                _active_sessions.inc()
                _total_sessions.inc()
            await send(message)

        try:
            await self.app(scope, receive, send_wrapper)
        finally:
            if start_time is not None:
                _active_sessions.dec()
                _session_duration.observe(time.monotonic() - start_time)
```
**EN:** Class `WebSocketMetricsMiddleware` is introduced here. Its docstring describes the intent as: Pure ASGI middleware that instruments WebSocket connections.
**CN:** 这里定义类 `WebSocketMetricsMiddleware`。其文档字符串说明了该类的职责与使用方式。

## Key Concepts / 关键概念
- FastAPI routing and application assembly / FastAPI 路由与应用组装
- Async request handling / 异步请求处理
- Realtime/WebSocket handling / 实时/WebSocket 处理
- Metrics and observability / 指标与可观测性
- Middleware pipeline / 中间件链路
- Speech-to-text serving stack / 语音转文本服务栈

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `time`, `collections`
- **Third-party / 第三方**: `prometheus_client`, `starlette`
