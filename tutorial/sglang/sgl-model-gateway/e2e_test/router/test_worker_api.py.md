# test_worker_api.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/router/test_worker_api.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module exercises worker api behavior in the router end-to-end suite. It encodes scenarios, assertions, and shared setup used to verify correctness and regressions. / 该测试模块覆盖 路由端到端测试 中与 worker api 相关的行为，包含用于验证正确性与回归问题的场景、断言以及共享初始化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Module docstring
```python
"""Tests for gateway worker management APIs.

Tests the gateway's worker management endpoints:
- GET /workers - List all workers
- POST /add_worker - Add a worker dynamically
- POST /remove_worker - Remove a worker dynamically
- GET /v1/models - List available models

Usage:
    pytest e2e_test/router/test_worker_api.py -v
"""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 13-18: Imports and dependencies
```python
from __future__ import annotations

import logging

import pytest
from infra import ConnectionMode, Gateway, ModelPool
```
**EN:** This block imports `__future__`, `logging`, `pytest`, `infra`, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 20-20: Module constants and configuration
```python
logger = logging.getLogger(__name__)
```
**EN:** This section defines module-level names such as `logger`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 21-62: Test class `TestWorkerAPI`
```python


@pytest.mark.e2e
@pytest.mark.parametrize("setup_backend", ["grpc", "http"], indirect=True)
class TestWorkerAPI:
    """Tests for worker management APIs using setup_backend fixture."""

    def test_list_workers(self, setup_backend):
        """Test listing workers via /workers endpoint."""
        backend, model, client, gateway = setup_backend

        workers = gateway.list_workers()
        assert len(workers) >= 1, "Expected at least one worker"
        logger.info("Found %d workers", len(workers))

        for worker in workers:
            logger.info(
                "Worker: id=%s, url=%s, status=%s",
                worker.id,
                worker.url,
                worker.status,
            )
            assert worker.url, "Worker should have a URL"

    def test_list_models(self, setup_backend):
        """Test listing models via /v1/models endpoint."""
        backend, model, client, gateway = setup_backend

        models = gateway.list_models()
        assert len(models) >= 1, "Expected at least one model"
        logger.info("Found %d models", len(models))

        for m in models:
            logger.info("Model: %s", m.get("id", "unknown"))
            assert "id" in m, "Model should have an id"

    def test_health_endpoint(self, setup_backend):
        """Test health check endpoint."""
        backend, model, client, gateway = setup_backend

        assert gateway.health(), "Gateway should be healthy"
        logger.info("Gateway health check passed")
```
**EN:** This test class groups related scenarios around `TestWorkerAPI` and organizes shared assertions or helpers in one place.
**CN:** 这个测试类围绕 `TestWorkerAPI` 组织相关场景，并将共享断言或辅助逻辑集中在同一位置。

### Lines 63-163: Test class `TestIGWMode`
```python


@pytest.mark.e2e
class TestIGWMode:
    """Tests for IGW mode - start gateway empty, add workers via API.

    Workers are launched on-demand via model_pool.get().
    """

    def test_igw_start_empty(self, model_pool: ModelPool):
        """Test starting gateway in IGW mode with no workers."""
        gateway = Gateway()
        gateway.start(igw_mode=True)

        try:
            assert gateway.health(), "Gateway should be healthy"
            assert gateway.igw_mode, "Gateway should be in IGW mode"

            workers = gateway.list_workers()
            logger.info("IGW gateway started with %d workers", len(workers))
        finally:
            gateway.shutdown()

    def test_igw_add_worker(self, model_pool: ModelPool):
        """Test adding a worker to IGW gateway."""
        http_instance = model_pool.get("llama-8b", ConnectionMode.HTTP)

        gateway = Gateway()
        try:
            gateway.start(igw_mode=True)

            # Add worker
            success, result = gateway.add_worker(http_instance.worker_url)
            assert success, f"Failed to add worker: {result}"
            logger.info("Added worker: %s", result)

            # Verify worker was added
            workers = gateway.list_workers()
            assert len(workers) >= 1, "Expected at least one worker"
            logger.info("Worker count: %d", len(workers))

            # Verify models are available
            models = gateway.list_models()
            logger.info("Models available: %d", len(models))
        finally:
            gateway.shutdown()
            http_instance.release()

    def test_igw_add_and_remove_worker(self, model_pool: ModelPool):
        """Test adding and removing workers dynamically."""
        http_instance = model_pool.get("llama-8b", ConnectionMode.HTTP)

        gateway = Gateway()
        try:
            gateway.start(igw_mode=True)

            # Add worker
            success, _ = gateway.add_worker(http_instance.worker_url)
            assert success, "Failed to add worker"

            initial_count = len(gateway.list_workers())
            logger.info("Worker count after add: %d", initial_count)

            # Remove worker
            success, msg = gateway.remove_worker(http_instance.worker_url)
            if success:
                logger.info("Removed worker: %s", msg)
                final_count = len(gateway.list_workers())
                logger.info("Worker count after remove: %d", final_count)
            else:
                logger.warning("Remove worker not supported: %s", msg)
        finally:
            gateway.shutdown()
            http_instance.release()

    def test_igw_multiple_workers(self, model_pool: ModelPool):
        """Test adding multiple workers (HTTP + gRPC) to IGW gateway."""
        http_instance = model_pool.get("llama-8b", ConnectionMode.HTTP)
        grpc_instance = model_pool.get("llama-8b", ConnectionMode.GRPC)

        gateway = Gateway()
        try:
            gateway.start(igw_mode=True)

            # Add both workers
            success1, _ = gateway.add_worker(http_instance.worker_url)
            success2, _ = gateway.add_worker(grpc_instance.worker_url)

            if not success1 or not success2:
                pytest.skip("Dynamic worker management not fully supported")

            workers = gateway.list_workers()
            logger.info("Worker count: %d", len(workers))
            assert len(workers) >= 2, "Expected at least 2 workers"

            for w in workers:
                logger.info("Worker: id=%s, url=%s", w.id, w.url)
        finally:
            gateway.shutdown()
            grpc_instance.release()
            http_instance.release()
```
**EN:** This test class groups related scenarios around `TestIGWMode` and organizes shared assertions or helpers in one place.
**CN:** 这个测试类围绕 `TestIGWMode` 组织相关场景，并将共享断言或辅助逻辑集中在同一位置。

### Lines 164-225: Test class `TestDisableHealthCheck`
```python


@pytest.mark.e2e
class TestDisableHealthCheck:
    """Tests for --disable-health-check CLI option."""

    def test_disable_health_check_workers_immediately_healthy(
        self, model_pool: ModelPool
    ):
        """Test that workers are immediately healthy when health checks are disabled."""
        http_instance = model_pool.get("llama-8b", ConnectionMode.HTTP)

        gateway = Gateway()
        try:
            gateway.start(
                igw_mode=True,
                extra_args=["--disable-health-check"],
            )

            # Add worker - should be immediately healthy since health checks are disabled
            success, worker_id = gateway.add_worker(
                http_instance.worker_url,
                wait_ready=True,
                ready_timeout=10,  # Short timeout since it should be immediate
            )
            assert success, f"Failed to add worker: {worker_id}"
            logger.info("Added worker with health checks disabled: %s", worker_id)

            # Verify worker is healthy
            workers = gateway.list_workers()
            assert len(workers) >= 1, "Expected at least one worker"

            for worker in workers:
                logger.info(
                    "Worker: id=%s, status=%s, disable_health_check=%s",
                    worker.id,
                    worker.status,
                    worker.metadata.get("disable_health_check"),
                )
                # Worker should be healthy immediately
                assert (
                    worker.status == "healthy"
                ), "Worker should be healthy when health checks disabled"
        finally:
            gateway.shutdown()
            http_instance.release()

    def test_disable_health_check_gateway_starts_without_health_checker(
        self, model_pool: ModelPool
    ):
        """Test that gateway starts successfully with health checks disabled."""
        gateway = Gateway()
        gateway.start(
            igw_mode=True,
            extra_args=["--disable-health-check"],
        )

        try:
            assert gateway.health(), "Gateway should be healthy"
            logger.info("Gateway started with health checks disabled")
        finally:
            gateway.shutdown()
```
**EN:** This test class groups related scenarios around `TestDisableHealthCheck` and organizes shared assertions or helpers in one place.
**CN:** 这个测试类围绕 `TestDisableHealthCheck` 组织相关场景，并将共享断言或辅助逻辑集中在同一位置。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Routing, worker selection, or load balancing / 路由、工作节点选择或负载均衡
- Primary classes: `TestWorkerAPI`, `TestIGWMode`, `TestDisableHealthCheck` / 主要类：`TestWorkerAPI`, `TestIGWMode`, `TestDisableHealthCheck`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`
- **Third-party / 第三方**: `pytest`
- **Internal / 内部模块**: `infra`
