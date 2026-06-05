# auth.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/auth.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable SGLang runtime helpers for authentication helpers. / 为 SGLang 运行时提供面向鉴权辅助逻辑的可复用辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Module setup and shared state / 模块设置与共享状态
```python
"""Auth utilities for HTTP servers.

This module is intentionally lightweight (no torch import) so it can be used in unit tests.
"""

from __future__ import annotations

import secrets
from dataclasses import dataclass
from enum import Enum
from typing import Any, Optional
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `secrets`, `dataclasses`, `enum`, `typing`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `secrets`, `dataclasses`, `enum`, `typing`。

### Lines 14-17: Class `AuthDecision` declaration / 类 `AuthDecision` 声明
```python
@dataclass(frozen=True)
class AuthDecision:
    allowed: bool
    error_status_code: int = 401  # Only meaningful when allowed=False
```
**EN:** This class establishes `AuthDecision` as a compact data container for the surrounding logic. The main stored fields include `allowed`, `error_status_code`.
**CN:** 该类将 `AuthDecision` 定义为周边逻辑的紧凑的数据容器。 其主要存储字段包括 `allowed`, `error_status_code`。

### Lines 20-25: Class `AuthLevel` declaration / 类 `AuthLevel` 声明
```python
class AuthLevel(str, Enum):
    """Per-endpoint auth level (attached to endpoint function via `@auth_level`)."""

    NORMAL = "normal"
    ADMIN_OPTIONAL = "admin_optional"
    ADMIN_FORCE = "admin_force"
```
**EN:** This class establishes `AuthLevel` as the main container/coordinator for the surrounding logic. It inherits from `str`, `Enum`. The main stored fields include `NORMAL`, `ADMIN_OPTIONAL`, `ADMIN_FORCE`.
**CN:** 该类将 `AuthLevel` 定义为周边逻辑的主要封装体或协调者。 它继承自 `str`, `Enum`。 其主要存储字段包括 `NORMAL`, `ADMIN_OPTIONAL`, `ADMIN_FORCE`。

### Lines 28-35: Function `auth_level` / 函数 `auth_level`
```python
def auth_level(level: AuthLevel):
    """Mark endpoint with auth level (stored in endpoint metadata)."""

    def decorator(func):
        func._auth_level = level
        return func

    return decorator
```
**EN:** This function implements `auth_level`. State updates are written into `func._auth_level`.
**CN:** 该函数实现了 `auth_level`。 状态更新主要写入 `func._auth_level`。

### Lines 38-59: Function `_get_auth_level_from_app_and_scope` / 函数 `_get_auth_level_from_app_and_scope`
```python
def _get_auth_level_from_app_and_scope(app: Any, scope: dict) -> AuthLevel:
    """Best-effort resolve auth level by matching the request to a route."""
    # Import lazily to keep this module unit-test friendly (FastAPI/Starlette are not
    # required unless you actually use the middleware / route matching).
    from starlette.routing import Match

    # Prefer app.router.routes when available; fall back to app.routes.
    routes = getattr(getattr(app, "router", None), "routes", None) or getattr(
        app, "routes", []
    )

    for route in routes:
        try:
            match, child_scope = route.matches(scope)
        except Exception:
            continue
        if match == Match.FULL:
            endpoint = child_scope.get("endpoint") or getattr(route, "endpoint", None)
            level = getattr(endpoint, "_auth_level", None)
            return level if isinstance(level, AuthLevel) else AuthLevel.NORMAL

    return AuthLevel.NORMAL
```
**EN:** This function implements `_get_auth_level_from_app_and_scope`. It primarily calls `getattr`, `route.matches`, `child_scope.get`, `isinstance` to complete its work. State updates are written into `routes`, `endpoint`, `level`. The implementation relies on conditional branches, iteration, error handling.
**CN:** 该函数实现了 `_get_auth_level_from_app_and_scope`。 它主要通过调用 `getattr`, `route.matches`, `child_scope.get`, `isinstance` 来完成任务。 状态更新主要写入 `routes`, `endpoint`, `level`。 实现中使用了条件分支、迭代逻辑、错误处理。

### Lines 62-71: Function `app_has_admin_force_endpoints` / 函数 `app_has_admin_force_endpoints`
```python
def app_has_admin_force_endpoints(app: Any) -> bool:
    """Return True if any route endpoint is marked as ADMIN_FORCE."""
    routes = getattr(getattr(app, "router", None), "routes", None) or getattr(
        app, "routes", []
    )
    for route in routes:
        endpoint = getattr(route, "endpoint", None)
        if getattr(endpoint, "_auth_level", None) == AuthLevel.ADMIN_FORCE:
            return True
    return False
```
**EN:** This function implements `app_has_admin_force_endpoints`. It primarily calls `getattr` to complete its work. State updates are written into `routes`, `endpoint`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `app_has_admin_force_endpoints`。 它主要通过调用 `getattr` 来完成任务。 状态更新主要写入 `routes`, `endpoint`。 实现中使用了条件分支、迭代逻辑。

### Lines 74-146: Function `decide_request_auth` / 函数 `decide_request_auth`
```python
def decide_request_auth(
    *,
    method: str,
    path: str,
    authorization_header: Optional[str],
    api_key: Optional[str],
    admin_api_key: Optional[str],
    auth_level: AuthLevel,
) -> AuthDecision:
    """Pure auth decision function (easy to unit test).

    Auth levels:
    - NORMAL: legacy behavior (api_key protects all endpoints when configured)
    - ADMIN_OPTIONAL: can be accessed without any key (if no keys configured),
      or with api_key/admin_api_key depending on server config.
    - ADMIN_FORCE: requires admin_api_key; if admin_api_key is NOT configured,
      it must be rejected (403) even if api_key is provided.

    NOTE :
    - Health/metrics endpoints are always allowed (even when api_key/admin_api_key is set),
      to support k8s/liveness/readiness and Prometheus scraping without embedding secrets.
    - We match them by prefix to cover common variants like /health_generate.
    """
    if method == "OPTIONS":
        return AuthDecision(allowed=True)

    if path.startswith("/health") or path.startswith("/metrics"):
        return AuthDecision(allowed=True)
# ... omitted for brevity ...
    # - if api_key is configured, require api_key (even if admin_api_key is also configured)
    # - otherwise allow (including the "admin_api_key only" case)
    if api_key:
        return AuthDecision(allowed=_check_bearer_token(authorization_header, api_key))

    return AuthDecision(allowed=True)
```
**EN:** This function implements `decide_request_auth`. It primarily calls `AuthDecision`, `path.startswith`, `authorization_header.split`, `secrets.compare_digest`, `_check_bearer_token`, `len` to complete its work. State updates are written into `parts`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `decide_request_auth`。 它主要通过调用 `AuthDecision`, `path.startswith`, `authorization_header.split`, `secrets.compare_digest`, `_check_bearer_token`, `len` 来完成任务。 状态更新主要写入 `parts`。 实现中使用了条件分支。

### Lines 149-208: Function `add_api_key_middleware` / 函数 `add_api_key_middleware`
```python
def add_api_key_middleware(
    app,
    *,
    api_key: Optional[str],
    admin_api_key: Optional[str],
):
    """Add middleware for three endpoint auth levels: normal/admin_optional/admin_force."""
    # Import lazily so `decide_request_auth()` can be unit-tested without FastAPI installed.
    from fastapi.responses import ORJSONResponse
    from starlette.requests import Request

    class _ApiKeyASGIMiddleware:
        """ASGI-native middleware to preserve client disconnect events."""

        def __init__(self, app, *, api_key, admin_api_key, fastapi_app):
            self.app = app
            self.api_key = api_key
            self.admin_api_key = admin_api_key
            self.fastapi_app = fastapi_app

        async def __call__(self, scope, receive, send):
            if scope["type"] != "http":
                await self.app(scope, receive, send)
                return

            request = Request(scope, receive=receive)
            path = request.url.path
            authz = request.headers.get("Authorization")
# ... omitted for brevity ...
    app.add_middleware(
        _ApiKeyASGIMiddleware,
        api_key=api_key,
        admin_api_key=admin_api_key,
        fastapi_app=app,
    )
```
**EN:** This function implements `add_api_key_middleware`. It primarily calls `app.add_middleware`, `Request`, `request.headers.get`, `_get_auth_level_from_app_and_scope`, `decide_request_auth`, `ORJSONResponse` to complete its work. State updates are written into `self.app`, `self.api_key`, `self.admin_api_key`, `self.fastapi_app`, `request`, `path`. The implementation relies on conditional branches, async awaits.
**CN:** 该函数实现了 `add_api_key_middleware`。 它主要通过调用 `app.add_middleware`, `Request`, `request.headers.get`, `_get_auth_level_from_app_and_scope`, `decide_request_auth`, `ORJSONResponse` 来完成任务。 状态更新主要写入 `self.app`, `self.api_key`, `self.admin_api_key`, `self.fastapi_app`, `request`, `path`。 实现中使用了条件分支、异步等待。

## Key Concepts / 关键概念
- **Classes / 类**: `AuthDecision`, `AuthLevel`, `_ApiKeyASGIMiddleware`
- **Functions / 函数**: `auth_level`, `_get_auth_level_from_app_and_scope`, `app_has_admin_force_endpoints`, `decide_request_auth`, `add_api_key_middleware`, `decorator`, `_check_bearer_token`, `__init__`
- **Async functions / 异步函数**: `__call__`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: None / 无
- **External / 外部依赖**: `starlette.routing`, `fastapi.responses`, `starlette.requests`
- **Standard library / 标准库**: `__future__`, `secrets`, `dataclasses`, `enum`, `typing`
