# setup_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/fixtures/setup_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module exercises setup backend behavior in the end-to-end fixture layer. It encodes scenarios, assertions, and shared setup used to verify correctness and regressions. / 该测试模块覆盖 端到端测试夹具 中与 setup backend 相关的行为，包含用于验证正确性与回归问题的场景、断言以及共享初始化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Module docstring
```python
"""Backend setup fixtures for E2E tests.

This module provides fixtures for launching gateways/routers for different backends.
"""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 6-12: Imports and dependencies
```python
from __future__ import annotations

import logging
import os
from typing import TYPE_CHECKING

import pytest
```
**EN:** This block imports `__future__`, `logging`, `os`, `typing`, and 1 more, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 14-15: If block
```python
if TYPE_CHECKING:
    from infra import ModelPool
```
**EN:** This top-level block contributes supporting logic to the module and is executed or defined as part of file initialization.
**CN:** 这个顶层代码块为模块提供辅助逻辑，并在文件初始化阶段被执行或定义。

### Lines 17-17: Imports and dependencies
```python
from .markers import get_marker_kwargs, get_marker_value
```
**EN:** This block imports `.markers`, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 19-19: Module constants and configuration
```python
logger = logging.getLogger(__name__)
```
**EN:** This section defines module-level names such as `logger`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 20-133: Fixture `setup_backend`
```python


@pytest.fixture
def setup_backend(request: pytest.FixtureRequest, model_pool: "ModelPool"):
    """Function-scoped fixture that launches a router for each test.

    Routers are cheap to start (~1-2s) compared to workers (~30-60s), so we
    launch a fresh router per test for isolation while reusing the expensive
    workers from the session-scoped model_pool fixture.

    NOTE: This used to be ``scope="class"`` to amortize router startup across
    tests in the same class. Class-scoped fixtures don't survive
    pytest-parallel's ``--tests-per-worker N`` thread dispatch — its fixture-
    finalize handling for non-function scopes is buggy (the project hasn't
    had a real release since 2019). The class teardown silently never fired,
    so model_pool references acquired in setup leaked indefinitely, blocking
    eviction and deadlocking any subsequent test that needed a different
    model. Function scope walks the canonical pytest finalize path for
    every test, so each acquire is paired with a real release and the pool
    can evict cleanly.

    Backend types:
    - "http", "grpc": Gets existing worker from model_pool, launches router
    - "pd": Launches prefill/decode workers via model_pool, launches PD router
    - "openai", "xai", etc.: Launches cloud router (no local workers)

    Configuration via markers:
    - @pytest.mark.model("model-id"): Override default model
    - @pytest.mark.workers(count=1): Number of regular workers behind router
    - @pytest.mark.workers(prefill=1, decode=1): PD worker configuration
    - @pytest.mark.gateway(policy="round_robin", timeout=60): Gateway configuration

    Returns:
        Tuple of (backend_name, model_path, openai_client, gateway)

    Usage:
        @pytest.mark.parametrize("setup_backend", ["http"], indirect=True)
        class TestBasic:
            def test_chat(self, setup_backend):
                backend, model, client, gateway = setup_backend
    """
    import openai
    from infra import (
        DEFAULT_MODEL,
        DEFAULT_ROUTER_TIMEOUT,
        ENV_MODEL,
        ENV_SKIP_BACKEND_SETUP,
        LOCAL_MODES,
        ConnectionMode,
        Gateway,
        WorkerIdentity,
        WorkerType,
    )

    backend_name = request.param

    # Skip if requested
    if os.environ.get(ENV_SKIP_BACKEND_SETUP, "").lower() in ("1", "true", "yes"):
        pytest.skip(f"{ENV_SKIP_BACKEND_SETUP} is set")

    # Get model from marker or env var or default
    model_id = get_marker_value(request, "model")
    if model_id is None:
        model_id = os.environ.get(ENV_MODEL, DEFAULT_MODEL)

    # Get worker configuration from marker
    workers_config = get_marker_kwargs(
        request, "workers", defaults={"count": 1, "prefill": None, "decode": None}
    )

    # Get gateway configuration from marker
    gateway_config = get_marker_kwargs(
        request,
        "gateway",
        defaults={
            "policy": "round_robin",
            "timeout": DEFAULT_ROUTER_TIMEOUT,
            "extra_args": None,
        },
    )

    # PD disaggregation backend
    if backend_name == "pd":
        yield from _setup_pd_backend(
            request, model_pool, model_id, workers_config, gateway_config
        )
        return

    # Check if this is a local backend (grpc, http)
    try:
        connection_mode = ConnectionMode(backend_name)
        is_local = connection_mode in LOCAL_MODES
    except ValueError:
        is_local = False
        connection_mode = None

    # Local backends: use worker from pool + launch gateway
    if is_local:
        yield from _setup_local_backend(
            request,
            model_pool,
            backend_name,
            model_id,
            connection_mode,
            workers_config,
            gateway_config,
        )
        return

    # Get storage backend from marker (default: memory)
    storage_backend = get_marker_value(request, "storage", default="memory")

    # Cloud backends: launch cloud router
    yield from _setup_cloud_backend(backend_name, storage_backend, gateway_config)
```
**EN:** This pytest fixture prepares `setup_backend` so multiple tests can reuse the same setup or cleanup sequence.
**CN:** 这个 pytest 夹具为 `setup_backend` 准备共享的初始化或清理流程，方便多个测试复用。

### Lines 136-289: Helper function `_setup_pd_backend`
```python
def _setup_pd_backend(
    request: pytest.FixtureRequest,
    model_pool: "ModelPool",
    model_id: str,
    workers_config: dict,
    gateway_config: dict,
):
    """Setup PD disaggregation backend."""
    import openai
    from infra import ConnectionMode, Gateway, WorkerIdentity, WorkerType

    logger.info("Setting up PD backend for model %s", model_id)

    # Get PD configuration from workers marker
    num_prefill = workers_config.get("prefill") or 1
    num_decode = workers_config.get("decode") or 1
    logger.info("PD config: %d prefill, %d decode workers", num_prefill, num_decode)

    prefills: list = []
    decodes: list = []
    gateway = None

    # Single try/finally guarantees release() runs for every acquired
    # worker, even if Gateway.start() / OpenAI() raise after acquisition.
    # See _setup_local_backend for the full rationale.
    try:
        # Try to use pre-launched PD workers, or launch additional ones if needed
        # get_workers_by_type auto-acquires all returned workers
        existing_prefills = model_pool.get_workers_by_type(model_id, WorkerType.PREFILL)
        existing_decodes = model_pool.get_workers_by_type(model_id, WorkerType.DECODE)

        # Calculate how many more we need
        missing_prefill = max(0, num_prefill - len(existing_prefills))
        missing_decode = max(0, num_decode - len(existing_decodes))

        if missing_prefill == 0 and missing_decode == 0:
            prefills = existing_prefills[:num_prefill]
            decodes = existing_decodes[:num_decode]
            # Release excess workers we won't use
            for w in existing_prefills[num_prefill:]:
                w.release()
            for w in existing_decodes[num_decode:]:
                w.release()
            logger.info(
                "Using pre-launched PD workers: %d prefill, %d decode",
                len(prefills),
                len(decodes),
            )
        else:
            # Build WorkerIdentity list for missing workers
            workers_to_launch: list[WorkerIdentity] = []
            for i in range(missing_prefill):
                workers_to_launch.append(
                    WorkerIdentity(
                        model_id,
                        ConnectionMode.HTTP,
                        WorkerType.PREFILL,
                        len(existing_prefills) + i,
                    )
                )
            for i in range(missing_decode):
                workers_to_launch.append(
                    WorkerIdentity(
                        model_id,
                        ConnectionMode.HTTP,
                        WorkerType.DECODE,
                        len(existing_decodes) + i,
                    )
                )

            logger.info(
                "Have %d/%d prefill, %d/%d decode. Launching %d more workers",
                len(existing_prefills),
                num_prefill,
                len(existing_decodes),
                num_decode,
                len(workers_to_launch),
            )
            new_instances = model_pool.launch_workers(
                workers_to_launch, startup_timeout=300
            )

            if not new_instances:
                # Existing workers will be released by the outer finally.
                prefills = existing_prefills
                decodes = existing_decodes
                pytest.fail(
                    f"Failed to launch PD workers: needed {len(workers_to_launch)} workers "
                    f"but could not allocate GPUs (all in use or timeout)"
                )

            # Acquire newly launched instances (launch_workers doesn't auto-acquire)
            for inst in new_instances:
                inst.acquire()

            new_prefills = [
                w for w in new_instances if w.worker_type == WorkerType.PREFILL
            ]
            new_decodes = [
                w for w in new_instances if w.worker_type == WorkerType.DECODE
            ]
            prefills = existing_prefills + new_prefills
            decodes = existing_decodes + new_decodes

        # All workers in prefills and decodes are now acquired

        if not prefills or not decodes:
            pytest.fail(
                f"PD setup incomplete: have {len(prefills)} prefill, "
                f"{len(decodes)} decode "
                f"(need {num_prefill} prefill, {num_decode} decode)"
            )

        model_path = prefills[0].model_path

        gateway = Gateway()
        gateway.start(
            prefill_workers=prefills,
            decode_workers=decodes,
            policy=gateway_config["policy"],
            timeout=gateway_config["timeout"],
            extra_args=gateway_config["extra_args"],
        )

        client = openai.OpenAI(
            base_url=f"{gateway.base_url}/v1",
            api_key="not-used",
        )

        logger.info(
            "Setup PD backend: model=%s, %d prefill + %d decode workers, "
            "gateway=%s, policy=%s",
            model_id,
            len(prefills),
            len(decodes),
            gateway.base_url,
            gateway_config["policy"],
        )

        yield "pd", model_path, client, gateway
    finally:
        if gateway is not None:
            logger.info("Tearing down PD gateway")
            try:
                gateway.shutdown()
            except Exception:
                logger.exception("Gateway shutdown failed; continuing teardown")
        for worker in prefills + decodes:
            try:
                worker.release()
            except Exception:
                logger.exception(
                    "Release failed for %s; continuing teardown", worker.key
                )
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 292-406: Helper function `_setup_local_backend`
```python
def _setup_local_backend(
    request: pytest.FixtureRequest,
    model_pool: "ModelPool",
    backend_name: str,
    model_id: str,
    connection_mode,
    workers_config: dict,
    gateway_config: dict,
):
    """Setup local backend (grpc, http)."""
    import openai
    from infra import Gateway, WorkerIdentity, WorkerType

    num_workers = workers_config.get("count") or 1
    instances: list = []  # Track instances for reference counting
    gateway = None

    # Single try/finally guarantees release() runs for every acquired
    # instance — even when Gateway.start() / OpenAI() / launch_workers()
    # raise after acquisition. Without this, a failed gateway start in
    # one test pinned the worker as is_in_use=True forever, so subsequent
    # tests that needed a different model couldn't evict and deadlocked
    # in model_pool.get().
    try:
        try:
            if num_workers > 1:
                # get_workers_by_type auto-acquires all returned workers
                all_existing = model_pool.get_workers_by_type(
                    model_id, WorkerType.REGULAR
                )
                existing_for_mode = [
                    w for w in all_existing if w.mode == connection_mode
                ]

                # Release workers we won't use (wrong mode)
                for w in all_existing:
                    if w not in existing_for_mode:
                        w.release()

                if len(existing_for_mode) >= num_workers:
                    instances = existing_for_mode[:num_workers]
                    # Release excess workers we won't use
                    for w in existing_for_mode[num_workers:]:
                        w.release()
                else:
                    missing = num_workers - len(existing_for_mode)
                    workers_to_launch = [
                        WorkerIdentity(
                            model_id,
                            connection_mode,
                            WorkerType.REGULAR,
                            len(existing_for_mode) + i,
                        )
                        for i in range(missing)
                    ]
                    new_instances = model_pool.launch_workers(
                        workers_to_launch, startup_timeout=300
                    )
                    # Acquire newly launched instances
                    for inst in new_instances:
                        inst.acquire()
                    instances = existing_for_mode + new_instances

                if not instances:
                    pytest.fail(f"Failed to get {num_workers} workers for {model_id}")
                worker_urls = [inst.worker_url for inst in instances]
                model_path = instances[0].model_path
            else:
                # get() auto-acquires the returned instance
                instance = model_pool.get(model_id, connection_mode)
                instances = [instance]
                worker_urls = [instance.worker_url]
                model_path = instance.model_path
        except RuntimeError as e:
            pytest.fail(str(e))

        gateway = Gateway()
        gateway.start(
            worker_urls=worker_urls,
            model_path=model_path,
            policy=gateway_config["policy"],
            timeout=gateway_config["timeout"],
            extra_args=gateway_config["extra_args"],
        )

        client = openai.OpenAI(
            base_url=f"{gateway.base_url}/v1",
            api_key="not-used",
        )

        logger.info(
            "Setup %s backend: model=%s, workers=%d, gateway=%s, policy=%s",
            backend_name,
            model_id,
            num_workers,
            gateway.base_url,
            gateway_config["policy"],
        )

        yield backend_name, model_path, client, gateway
    finally:
        if gateway is not None:
            logger.info("Tearing down gateway for %s backend", backend_name)
            try:
                gateway.shutdown()
            except Exception:
                logger.exception("Gateway shutdown failed; continuing teardown")
        # Release references to allow eviction. Each release is
        # independently fault-isolated so one failure can't strand the
        # rest of the acquired instances.
        for inst in instances:
            try:
                inst.release()
            except Exception:
                logger.exception("Release failed for %s; continuing teardown", inst.key)
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 409-454: Helper function `_setup_cloud_backend`
```python
def _setup_cloud_backend(
    backend_name: str,
    storage_backend: str = "memory",
    gateway_config: dict | None = None,
):
    """Setup cloud backend (openai, xai, etc.).

    Args:
        backend_name: Cloud backend name (openai, xai).
        storage_backend: History storage backend (memory, oracle).
        gateway_config: Gateway configuration from marker.
    """
    import openai
    from infra import THIRD_PARTY_MODELS, launch_cloud_gateway

    if backend_name not in THIRD_PARTY_MODELS:
        pytest.fail(f"Unknown cloud runtime: {backend_name}")

    cfg = THIRD_PARTY_MODELS[backend_name]
    api_key_env = cfg.get("api_key_env")

    if api_key_env and not os.environ.get(api_key_env):
        pytest.skip(f"{api_key_env} not set, skipping {backend_name} tests")

    extra_args = gateway_config.get("extra_args") if gateway_config else None

    logger.info(
        "Launching cloud backend: %s with storage=%s", backend_name, storage_backend
    )
    gateway = launch_cloud_gateway(
        backend_name,
        history_backend=storage_backend,
        extra_args=extra_args,
    )

    api_key = os.environ.get(api_key_env) if api_key_env else "not-used"
    client = openai.OpenAI(
        base_url=f"{gateway.base_url}/v1",
        api_key=api_key,
    )

    try:
        yield backend_name, cfg["model"], client, gateway
    finally:
        logger.info("Tearing down cloud backend: %s", backend_name)
        gateway.shutdown()
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 455-495: Fixture `backend_router`
```python


@pytest.fixture
def backend_router(request: pytest.FixtureRequest, model_pool: "ModelPool"):
    """Function-scoped fixture for launching a fresh router per test.

    This launches a new Gateway for each test, pointing to workers from the pool.
    Use for tests that need isolated router state.

    Usage:
        @pytest.mark.parametrize("backend_router", ["grpc", "http"], indirect=True)
        def test_router_state(backend_router):
            gateway = backend_router
    """
    from infra import DEFAULT_MODEL, ENV_MODEL, ConnectionMode, Gateway

    backend_name = request.param
    model_id = os.environ.get(ENV_MODEL, DEFAULT_MODEL)

    connection_mode = ConnectionMode(backend_name)

    try:
        # get() auto-acquires the returned instance
        instance = model_pool.get(model_id, connection_mode)
    except KeyError:
        pytest.skip(f"Model {model_id}:{backend_name} not available in pool")
    except RuntimeError as e:
        pytest.fail(str(e))

    gateway = Gateway()
    gateway.start(
        worker_urls=[instance.worker_url],
        model_path=instance.model_path,
    )

    try:
        yield gateway
    finally:
        gateway.shutdown()
        # Release reference to allow eviction
        instance.release()
```
**EN:** This pytest fixture prepares `backend_router` so multiple tests can reuse the same setup or cleanup sequence.
**CN:** 这个 pytest 夹具为 `backend_router` 准备共享的初始化或清理流程，方便多个测试复用。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Reusable functions: `setup_backend`, `_setup_pd_backend`, `_setup_local_backend`, `_setup_cloud_backend`, and 1 more / 可复用函数：`setup_backend`, `_setup_pd_backend`, `_setup_local_backend`, `_setup_cloud_backend`, and 1 more

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `os`, `typing`
- **Third-party / 第三方**: `openai`, `pytest`
- **Internal / 内部模块**: `.markers`, `infra`
