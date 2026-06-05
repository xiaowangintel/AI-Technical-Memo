# http_middleware_patch.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/http_middleware_patch.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable runtime utilities centered on `http_middleware_patch` and the surrounding SGLang serving stack. / 提供围绕 `http_middleware_patch` 及其所处 SGLang 服务栈的可复用运行时工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Module setup and shared state / 模块设置与共享状态
```python
"""
Fix @app.middleware("http") whose BaseHTTPMiddleware call_next replaces
ASGI ``receive``, breaking request.is_disconnected() and preventing
non-streaming request abort on client disconnect.

patch_app_http_middleware(app) replaces @app.middleware("http") with a
version whose call_next passes ``receive`` through untouched.
"""

from __future__ import annotations

from starlette.requests import Request
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `starlette.requests`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `starlette.requests`。

### Lines 15-16: Class `_SentResponse` declaration / 类 `_SentResponse` 声明
```python
class _SentResponse:
    """Response proxy returned after the real response was already sent."""
```
**EN:** This class establishes `_SentResponse` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`.
**CN:** 该类将 `_SentResponse` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__` 等方法。

### Lines 18-19: Method `_SentResponse.__init__` / 方法 `_SentResponse.__init__`
```python
    def __init__(self, status_code: int):
        self.status_code = status_code
```
**EN:** This method implements `__init__` on `_SentResponse`. State updates are written into `self.status_code`.
**CN:** 该方法（属于 `_SentResponse`）实现了 `__init__`。 状态更新主要写入 `self.status_code`。

### Lines 22-24: Class `_PureASGIDispatch` declaration / 类 `_PureASGIDispatch` 声明
```python
class _PureASGIDispatch:
    """Pure ASGI middleware providing a fixed call_next that passes
    ``receive`` through untouched (unlike BaseHTTPMiddleware)."""
```
**EN:** This class establishes `_PureASGIDispatch` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `__call__`.
**CN:** 该类将 `_PureASGIDispatch` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `__call__` 等方法。

### Lines 26-28: Method `_PureASGIDispatch.__init__` / 方法 `_PureASGIDispatch.__init__`
```python
    def __init__(self, app, dispatch):
        self.app = app
        self.dispatch = dispatch
```
**EN:** This method implements `__init__` on `_PureASGIDispatch`. State updates are written into `self.app`, `self.dispatch`.
**CN:** 该方法（属于 `_PureASGIDispatch`）实现了 `__init__`。 状态更新主要写入 `self.app`, `self.dispatch`。

### Lines 30-50: Method `_PureASGIDispatch.__call__` / 方法 `_PureASGIDispatch.__call__`
```python
    async def __call__(self, scope, receive, send):
        if scope["type"] != "http":
            await self.app(scope, receive, send)
            return

        request = Request(scope, receive)
        status_code = 500

        async def call_next(_request):
            nonlocal status_code

            async def send_and_capture(message):
                nonlocal status_code
                if message["type"] == "http.response.start":
                    status_code = message["status"]
                await send(message)

            await self.app(scope, receive, send_and_capture)
            return _SentResponse(status_code)

        await self.dispatch(request, call_next)
```
**EN:** This async method implements `__call__` on `_PureASGIDispatch`. It primarily calls `Request`, `_SentResponse`, `self.dispatch`, `self.app`, `send` to complete its work. State updates are written into `request`, `status_code`. The implementation relies on conditional branches, async awaits.
**CN:** 该方法（属于 `_PureASGIDispatch`）实现了 `__call__`。 它主要通过调用 `Request`, `_SentResponse`, `self.dispatch`, `self.app`, `send` 来完成任务。 状态更新主要写入 `request`, `status_code`。 实现中使用了条件分支、异步等待。

### Lines 53-67: Function `patch_app_http_middleware` / 函数 `patch_app_http_middleware`
```python
def patch_app_http_middleware(app):
    """Replace @app.middleware("http") with a fixed-call_next version."""
    _orig = app.middleware

    def _fixed(middleware_type):
        if middleware_type == "http":

            def decorator(fn):
                app.add_middleware(_PureASGIDispatch, dispatch=fn)
                return fn

            return decorator
        return _orig(middleware_type)

    app.middleware = _fixed
```
**EN:** This function implements `patch_app_http_middleware`. It primarily calls `_orig`, `app.add_middleware` to complete its work. State updates are written into `_orig`, `app.middleware`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `patch_app_http_middleware`。 它主要通过调用 `_orig`, `app.add_middleware` 来完成任务。 状态更新主要写入 `_orig`, `app.middleware`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Classes / 类**: `_SentResponse`, `_PureASGIDispatch`
- **Functions / 函数**: `patch_app_http_middleware`, `__init__`, `__init__`, `_fixed`, `decorator`
- **Async functions / 异步函数**: `__call__`, `call_next`, `send_and_capture`
- **Themes / 主题**: `patch`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: None / 无
- **External / 外部依赖**: `starlette.requests`
- **Standard library / 标准库**: `__future__`
