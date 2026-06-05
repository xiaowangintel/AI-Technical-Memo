# grpc_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/grpc_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements grpc server logic for runtime entrypoints and service adapters. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 grpc 服务器 相关的逻辑，并服务于 运行时入口与服务适配层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11: Provide supporting module logic / 提供辅助模块逻辑
```python
"""
Thin gRPC server wrapper — delegates to smg-grpc-servicer package.

A lightweight HTTP sidecar is started alongside the gRPC server to expose:
- /metrics (Prometheus, when --enable-metrics is set)
- /start_profile, /stop_profile (profiling control)

The sidecar is started on --grpc-http-sidecar-port (default: --port + 1)
once the gRPC request manager is ready, regardless of whether --enable-metrics
is set.
"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 13-16: Import runtime dependencies / 导入运行时依赖
```python
import inspect
import json
import logging
import time
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 18-18: Import runtime dependencies / 导入运行时依赖
```python
from aiohttp import web
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 20-21: Import runtime dependencies / 导入运行时依赖
```python
from sglang.srt.managers.io_struct import ProfileReq, ProfileReqType
from sglang.srt.utils.common import get_bool_env_var
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 23-23: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 26-37: Implement async start sidecar server / 实现异步start sidecar 服务器
```python
async def _start_sidecar_server(host: str, port: int, app):
    """Start the aiohttp sidecar and return the runner for cleanup."""
    runner = web.AppRunner(app)
    await runner.setup()
    try:
        site = web.TCPSite(runner, host, port)
        await site.start()
    except BaseException:
        await runner.cleanup()
        raise
    logger.info("HTTP sidecar server started on http://%s:%d", host, port)
    return runner
```
**EN:** This block implements the async function `_start_sidecar_server(host, port, app)`. It focuses on Start the aiohttp sidecar and return the runner for cleanup., providing reusable behavior for the grpc server pipeline.
**CN:** 该代码块实现异步函数 `_start_sidecar_server(host, port, app)`。它围绕 `_start_sidecar_server` 所承担的 grpc 服务器 相关职责展开，为对应处理链路提供可复用能力。

### Lines 40-64: Implement add metrics routes / 实现add 指标 routes
```python
def _add_metrics_routes(app):
    """Add Prometheus /metrics endpoint to the aiohttp app."""
    from prometheus_client import (
        CollectorRegistry,
        multiprocess,
    )
    from prometheus_client.openmetrics.exposition import (
        CONTENT_TYPE_LATEST,
        generate_latest,
    )

    async def metrics_handler(request):
        try:
            registry = CollectorRegistry()
            multiprocess.MultiProcessCollector(registry)
            data = generate_latest(registry)
            return web.Response(
                body=data,
                headers={"Content-Type": CONTENT_TYPE_LATEST},
            )
        except Exception:
            logger.exception("Failed to generate Prometheus metrics")
            return web.Response(status=500, text="Failed to generate metrics")

    app.router.add_get("/metrics", metrics_handler)
```
**EN:** This block implements the function `_add_metrics_routes(app)`. It focuses on Add Prometheus /metrics endpoint to the aiohttp app., providing reusable behavior for the grpc server pipeline.
**CN:** 该代码块实现函数 `_add_metrics_routes(app)`。它围绕 `_add_metrics_routes` 所承担的 grpc 服务器 相关职责展开，为对应处理链路提供可复用能力。

### Lines 67-75: Implement check communicator results / 实现检查 communicator results
```python
def _check_communicator_results(results, action):
    """Return a web.Response error if results indicate failure, else None."""
    if not results:
        return web.Response(status=500, text="No response from scheduler\n")
    failures = [r for r in results if not r.success]
    if failures:
        msgs = " | ".join(r.message for r in failures)
        return web.Response(status=500, text=f"{action} failed: {msgs}\n")
    return None
```
**EN:** This block implements the function `_check_communicator_results(results, action)`. It focuses on Return a web.Response error if results indicate failure, else None., providing reusable behavior for the grpc server pipeline.
**CN:** 该代码块实现函数 `_check_communicator_results(results, action)`。它围绕 `_check_communicator_results` 所承担的 grpc 服务器 相关职责展开，为对应处理链路提供可复用能力。

### Lines 78-113: Implement add admin routes / 实现add admin routes
```python
def _add_admin_routes(app, request_manager):
    """Add admin endpoints to the aiohttp app.

    Endpoints: /start_profile, /stop_profile.
    Business logic (request construction, env var handling, response interpretation)
    lives here; request_manager only provides the transport to the scheduler.
    """

    async def start_profile_handler(request):
        try:
            if request.content_length and request.content_length > 0:
                try:
                    body = await request.json()
                except json.JSONDecodeError as e:
                    return web.Response(
                        status=400,
                        text=f"Invalid JSON in request body: {e}",
                    )
            else:
                body = {}

            # Build ProfileReq with env var overrides (same as tokenizer_communicator_mixin)
            with_stack = body.get("with_stack")
            env_with_stack = get_bool_env_var("SGLANG_PROFILE_WITH_STACK", "true")
            with_stack = (with_stack is not False) and env_with_stack
            record_shapes = body.get("record_shapes")
            env_record_shapes = get_bool_env_var("SGLANG_PROFILE_RECORD_SHAPES", "true")
            record_shapes = (record_shapes is not False) and env_record_shapes

            req = ProfileReq(
                type=ProfileReqType.START_PROFILE,
                output_dir=body.get("output_dir"),
                start_step=body.get("start_step"),
                num_steps=body.get("num_steps"),
                activities=body.get("activities"),
                with_stack=with_stack,
```
**EN:** This block implements the function `_add_admin_routes(app, request_manager)`. It focuses on Add admin endpoints to the aiohttp app., providing reusable behavior for the grpc server pipeline.
**CN:** 该代码块实现函数 `_add_admin_routes(app, request_manager)`。它围绕 `_add_admin_routes` 所承担的 grpc 服务器 相关职责展开，为对应处理链路提供可复用能力。

### Lines 114-149: Continue add admin routes / 继续说明add admin routes
```python
                record_shapes=record_shapes,
                profile_by_stage=body.get("profile_by_stage", False),
                profile_id=str(time.time()),
                merge_profiles=body.get("merge_profiles", False),
                profile_prefix=body.get("profile_prefix"),
                profile_stages=body.get("profile_stages"),
            )
            results = await request_manager.send_communicator_req(
                req, "profile_communicator", timeout=600.0
            )
            err = _check_communicator_results(results, "Start Profile")
            if err:
                return err
            return web.Response(text="Start profiling.\n")
        except Exception as e:
            logger.exception("Failed to start profile")
            return web.Response(
                status=500,
                text=f"Internal error: {type(e).__name__}. Check server logs.\n",
            )

    async def stop_profile_handler(request):
        try:
            req = ProfileReq(type=ProfileReqType.STOP_PROFILE)
            results = await request_manager.send_communicator_req(
                req, "profile_communicator", timeout=600.0
            )
            err = _check_communicator_results(results, "Stop profile")
            if err:
                return err
            return web.Response(text="Stop profiling. This will take some time.\n")
        except Exception as e:
            logger.exception("Failed to stop profile")
            return web.Response(
                status=500,
                text=f"Internal error: {type(e).__name__}. Check server logs.\n",
```
**EN:** This block implements the function `_add_admin_routes(app, request_manager)`. It focuses on Add admin endpoints to the aiohttp app., providing reusable behavior for the grpc server pipeline.
**CN:** 该代码块实现函数 `_add_admin_routes(app, request_manager)`。它围绕 `_add_admin_routes` 所承担的 grpc 服务器 相关职责展开，为对应处理链路提供可复用能力。

### Lines 150-153: Continue add admin routes / 继续说明add admin routes
```python
            )

    app.router.add_post("/start_profile", start_profile_handler)
    app.router.add_post("/stop_profile", stop_profile_handler)
```
**EN:** This block implements the function `_add_admin_routes(app, request_manager)`. It focuses on Add admin endpoints to the aiohttp app., providing reusable behavior for the grpc server pipeline.
**CN:** 该代码块实现函数 `_add_admin_routes(app, request_manager)`。它围绕 `_add_admin_routes` 所承担的 grpc 服务器 相关职责展开，为对应处理链路提供可复用能力。

### Lines 156-191: Implement async serve grpc / 实现异步serve grpc
```python
async def serve_grpc(server_args, model_info=None):
    """Start the standalone gRPC server with integrated scheduler."""
    try:
        from smg_grpc_servicer.sglang.server import serve_grpc as _serve_grpc
    except ImportError as e:
        raise ImportError(
            "gRPC mode requires the smg-grpc-servicer package. "
            "If not installed, run: pip install smg-grpc-servicer[sglang]. "
            "If already installed, there may be a broken import due to a "
            "version mismatch — see the chained exception above for details."
        ) from e

    sidecar_app = web.Application()
    sidecar_runner = None
    sidecar_port = (
        server_args.grpc_http_sidecar_port
        if server_args.grpc_http_sidecar_port is not None
        else server_args.port + 1
    )

    # Metrics setup: must set PROMETHEUS_MULTIPROC_DIR before scheduler
    # processes import prometheus_client, since the env var is inherited
    # at fork time.
    if server_args.enable_metrics:
        try:
            from sglang.srt.observability.func_timer import enable_func_timer
            from sglang.srt.utils import set_prometheus_multiproc_dir

            set_prometheus_multiproc_dir()
            enable_func_timer()
            _add_metrics_routes(sidecar_app)
        except Exception as e:
            logger.error(
                "Failed to set up metrics: %s. Continuing without metrics.",
                e,
                exc_info=True,
```
**EN:** This block implements the async function `serve_grpc(server_args, model_info)`. It focuses on Start the standalone gRPC server with integrated scheduler., providing reusable behavior for the grpc server pipeline.
**CN:** 该代码块实现异步函数 `serve_grpc(server_args, model_info)`。它围绕 `serve_grpc` 所承担的 grpc 服务器 相关职责展开，为对应处理链路提供可复用能力。

### Lines 192-227: Continue async serve grpc / 继续说明异步serve grpc
```python
            )

    async def _on_request_manager_ready(request_manager, srv_args, sched_info):
        nonlocal sidecar_runner
        try:
            _add_admin_routes(sidecar_app, request_manager)
        except Exception as e:
            logger.error(
                "Failed to set up admin routes: %s. "
                "Continuing without admin endpoints.",
                e,
                exc_info=True,
            )
        try:
            sidecar_runner = await _start_sidecar_server(
                server_args.host, sidecar_port, sidecar_app
            )
        except OSError as e:
            logger.error(
                "Failed to start HTTP sidecar server: %s. "
                "Continuing without metrics/profile endpoints.",
                e,
                exc_info=True,
            )
        except Exception as e:
            logger.error(
                "Unexpected error starting HTTP sidecar server: %s. "
                "Continuing without metrics/profile endpoints.",
                e,
                exc_info=True,
            )

    # Older smg-grpc-servicer releases (≤ 0.5.2) accept only (server_args,
    # model_info) and reject the on_request_manager_ready hook. The hook is
    # what calls _start_sidecar_server, so dropping the kwarg disables the
    # entire HTTP sidecar (Prometheus /metrics and /start_profile +
```
**EN:** This block implements the async function `serve_grpc(server_args, model_info)`. It focuses on Start the standalone gRPC server with integrated scheduler., providing reusable behavior for the grpc server pipeline.
**CN:** 该代码块实现异步函数 `serve_grpc(server_args, model_info)`。它围绕 `serve_grpc` 所承担的 grpc 服务器 相关职责展开，为对应处理链路提供可复用能力。

### Lines 228-263: Continue async serve grpc / 继续说明异步serve grpc
```python
    # /stop_profile). Core gRPC serving still works without it.
    serve_kwargs: dict = {}
    sidecar_supported = (
        "on_request_manager_ready" in inspect.signature(_serve_grpc).parameters
    )
    if sidecar_supported:
        serve_kwargs["on_request_manager_ready"] = _on_request_manager_ready
    elif server_args.enable_metrics:
        # User explicitly asked for metrics but the installed servicer can't
        # start the sidecar that serves them — fail loud rather than silently
        # produce a server with no /metrics endpoint.
        raise RuntimeError(
            "--enable-metrics requires smg-grpc-servicer ≥ 0.5.3 (the version "
            "that accepts 'on_request_manager_ready'); installed version "
            "lacks the hook so the HTTP sidecar would never start. Upgrade "
            "smg-grpc-servicer or remove --enable-metrics."
        )
    else:
        logger.warning(
            "Installed smg-grpc-servicer does not accept "
            "'on_request_manager_ready'; HTTP sidecar disabled "
            "(no /metrics, /start_profile, /stop_profile). "
            "Upgrade smg-grpc-servicer to ≥ 0.5.3 to enable it."
        )

    try:
        await _serve_grpc(server_args, model_info, **serve_kwargs)
    finally:
        if sidecar_runner is not None:
            try:
                await sidecar_runner.cleanup()
            except Exception as e:
                logger.exception(
                    "Failed to cleanly shut down HTTP sidecar server: %s",
                    e,
                )
```
**EN:** This block implements the async function `serve_grpc(server_args, model_info)`. It focuses on Start the standalone gRPC server with integrated scheduler., providing reusable behavior for the grpc server pipeline.
**CN:** 该代码块实现异步函数 `serve_grpc(server_args, model_info)`。它围绕 `serve_grpc` 所承担的 grpc 服务器 相关职责展开，为对应处理链路提供可复用能力。

## Key Concepts / 关键概念
- **Main callables / 主要可调用对象**: _start_sidecar_server, _add_metrics_routes, _check_communicator_results, _add_admin_routes, serve_grpc
- **Domain focus / 领域焦点**: grpc server / grpc 服务器
- **Concurrency / 并发特征**: async/await appears throughout the module / 模块中多处使用 async/await 协调并发流程

## Dependencies / 依赖关系
- **Standard Library / 标准库**: inspect, json, logging, time
- **Third-party / 第三方库**: aiohttp, prometheus_client, prometheus_client.openmetrics.exposition, smg_grpc_servicer.sglang.server
- **Local Modules / 本地模块**: sglang.srt.managers.io_struct, sglang.srt.observability.func_timer, sglang.srt.utils, sglang.srt.utils.common
