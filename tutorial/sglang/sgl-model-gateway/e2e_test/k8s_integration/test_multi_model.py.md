# test_multi_model.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/k8s_integration/test_multi_model.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module exercises multi model behavior in the Kubernetes integration suite. It encodes scenarios, assertions, and shared setup used to verify correctness and regressions. / 该测试模块覆盖 Kubernetes 集成测试 中与 multi model 相关的行为，包含用于验证正确性与回归问题的场景、断言以及共享初始化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Module docstring
```python
"""Multi-model selector isolation integration test.

Two gateways watch the same namespace with disjoint --selector values
(model=llama vs model=qwen). Each must register only the workers carrying
its own label and ignore the other gateway's pool — proving that
PodInfo::should_include (sgl-model-gateway/src/service_discovery.rs:99)
honors the configured label selector when running multiple gateways
side-by-side (the typical multi-tenant deployment pattern).

Run with:
    cd e2e_test/k8s_integration
    pytest test_multi_model.py -v -s
"""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 15-33: Imports and dependencies
```python
from __future__ import annotations

import json
import logging
import subprocess
from pathlib import Path

import pytest
from conftest import (  # pytest's rootdir adds the test dir to sys.path
    KUBECTL_CONTEXT,
    NAMESPACE,
    _cleanup_port_forward,
    _get_workers,
    _kubectl,
    _poll_until,
    _port_forward_start,
    _wait_for_deployment_ready,
    _wait_for_pod_ready,
)
```
**EN:** This block imports `__future__`, `json`, `logging`, `subprocess`, and 3 more, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 35-40: Module constants and configuration
```python
logger = logging.getLogger(__name__)

MANIFESTS_DIR = Path(__file__).parent / "manifests"

LLAMA_GATEWAY_HTTP_PORT = 30003
QWEN_GATEWAY_HTTP_PORT = 30004
```
**EN:** This section defines module-level names such as `logger`, `MANIFESTS_DIR`, `LLAMA_GATEWAY_HTTP_PORT`, `QWEN_GATEWAY_HTTP_PORT`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 43-92: Helper function `_deploy_model_worker`
```python
def _deploy_model_worker(name: str, model: str):
    """Deploy a fake-worker with both `app=fake-worker` and `model=<...>` labels.

    Including `app=fake-worker` matches the existing baseline label used by
    the default gateway's selector — but only the *model-specific* gateway
    has the second `model=...` constraint, so worker-to-gateway mapping is
    determined by the model label alone.
    """
    pod_manifest = {
        "apiVersion": "v1",
        "kind": "Pod",
        "metadata": {
            "name": name,
            "namespace": NAMESPACE,
            "labels": {"app": "fake-worker", "model": model},
        },
        "spec": {
            "containers": [
                {
                    "name": "worker",
                    "image": "python:3.12-slim",
                    "imagePullPolicy": "IfNotPresent",
                    "command": ["python3", "/app/fake_worker.py"],
                    "ports": [{"containerPort": 8000}],
                    "readinessProbe": {
                        "httpGet": {"path": "/health", "port": 8000},
                        "initialDelaySeconds": 2,
                        "periodSeconds": 3,
                    },
                    "volumeMounts": [{"name": "app", "mountPath": "/app"}],
                }
            ],
            "volumes": [{"name": "app", "configMap": {"name": "fake-worker-script"}}],
        },
    }
    proc = subprocess.run(
        ["kubectl", "--context", KUBECTL_CONTEXT, "apply", "-f", "-"],
        input=json.dumps(pod_manifest),
        capture_output=True,
        text=True,
        check=False,
    )
    if proc.returncode != 0:
        # Surface kubectl's actual error (webhook denial, schema, missing
        # configmap, etc.) instead of an opaque CalledProcessError.
        raise RuntimeError(
            f"Failed to deploy worker {name} (rc={proc.returncode}): "
            f"stderr={proc.stderr.strip()!r}"
        )
    logger.info("Deployed worker %s with model=%s", name, model)
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 95-108: Helper function `_safe_force_delete`
```python
def _safe_force_delete(name: str):
    try:
        _kubectl(
            "delete",
            "pod",
            name,
            "-n",
            NAMESPACE,
            "--ignore-not-found",
            "--force",
            "--grace-period=0",
        )
    except Exception as e:
        logger.warning("Cleanup failed for pod %s: %s", name, e)
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 109-148: Fixture `multimodel_gateways`
```python


@pytest.fixture(scope="module")
def multimodel_gateways(deploy_base):
    """Deploy llama + qwen gateways and start port-forwards to both.

    Module-scoped: every test in this file shares the deployment to keep the
    suite fast (gateway startup + readiness is the slowest step). Cleanup
    runs in `finally:` so a port-forward failure cannot leak Deployments.
    """
    manifest = MANIFESTS_DIR / "gateway-multimodel.yaml"
    _kubectl("apply", "-f", str(manifest))

    pf_llama: subprocess.Popen | None = None
    pf_qwen: subprocess.Popen | None = None
    try:
        _wait_for_deployment_ready("smg-gateway-llama")
        _wait_for_deployment_ready("smg-gateway-qwen")
        pf_llama = _port_forward_start(
            NAMESPACE,
            "smg-gateway-llama",
            LLAMA_GATEWAY_HTTP_PORT,
            LLAMA_GATEWAY_HTTP_PORT,
        )
        pf_qwen = _port_forward_start(
            NAMESPACE,
            "smg-gateway-qwen",
            QWEN_GATEWAY_HTTP_PORT,
            QWEN_GATEWAY_HTTP_PORT,
        )
        yield (
            f"http://127.0.0.1:{LLAMA_GATEWAY_HTTP_PORT}",
            f"http://127.0.0.1:{QWEN_GATEWAY_HTTP_PORT}",
        )
    finally:
        if pf_llama is not None:
            _cleanup_port_forward("llama_gateway", pf_llama)
        if pf_qwen is not None:
            _cleanup_port_forward("qwen_gateway", pf_qwen)
        _kubectl("delete", "-f", str(manifest), "--ignore-not-found", check=False)
```
**EN:** This pytest fixture prepares `multimodel_gateways` so multiple tests can reuse the same setup or cleanup sequence.
**CN:** 这个 pytest 夹具为 `multimodel_gateways` 准备共享的初始化或清理流程，方便多个测试复用。

### Lines 151-204: Test class `TestMultiModelSelectorIsolation`
```python
class TestMultiModelSelectorIsolation:
    """Each gateway sees only the worker pool that matches its selector."""

    def test_each_gateway_sees_only_its_model_pool(self, multimodel_gateways):
        llama_url, qwen_url = multimodel_gateways
        llama_workers = ["model-llama-a", "model-llama-b"]
        qwen_workers = ["model-qwen-a", "model-qwen-b"]

        try:
            for name in llama_workers:
                _deploy_model_worker(name, model="llama")
            for name in qwen_workers:
                _deploy_model_worker(name, model="qwen")
            for name in llama_workers + qwen_workers:
                _wait_for_pod_ready(name)

            _poll_until(
                lambda: _get_workers(llama_url)["total"] >= len(llama_workers),
                f"llama gateway sees {len(llama_workers)} workers",
                timeout=30,
                interval=3,
            )
            _poll_until(
                lambda: _get_workers(qwen_url)["total"] >= len(qwen_workers),
                f"qwen gateway sees {len(qwen_workers)} workers",
                timeout=30,
                interval=3,
            )

            llama_view = _get_workers(llama_url)
            qwen_view = _get_workers(qwen_url)
            llama_urls = sorted(w["url"] for w in llama_view.get("workers", []))
            qwen_urls = sorted(w["url"] for w in qwen_view.get("workers", []))
            logger.info("Llama gateway workers: %s", llama_urls)
            logger.info("Qwen gateway workers:  %s", qwen_urls)

            assert llama_view["total"] == len(llama_workers), (
                f"Llama gateway should see exactly {len(llama_workers)} workers, "
                f"got {llama_view['total']}: {llama_urls}"
            )
            assert qwen_view["total"] == len(qwen_workers), (
                f"Qwen gateway should see exactly {len(qwen_workers)} workers, "
                f"got {qwen_view['total']}: {qwen_urls}"
            )

            # No URL should appear in both views — that would mean a
            # selector mismatch leaked a worker into the wrong gateway.
            cross_talk = set(llama_urls) & set(qwen_urls)
            assert (
                not cross_talk
            ), f"Workers leaked across model selectors: {cross_talk}"
        finally:
            for name in llama_workers + qwen_workers:
                _safe_force_delete(name)
```
**EN:** This test class groups related scenarios around `TestMultiModelSelectorIsolation` and organizes shared assertions or helpers in one place.
**CN:** 这个测试类围绕 `TestMultiModelSelectorIsolation` 组织相关场景，并将共享断言或辅助逻辑集中在同一位置。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Kubernetes lifecycle and reconciliation scenarios / Kubernetes 生命周期与协调场景
- Primary classes: `TestMultiModelSelectorIsolation` / 主要类：`TestMultiModelSelectorIsolation`
- Reusable functions: `_deploy_model_worker`, `_safe_force_delete`, `multimodel_gateways` / 可复用函数：`_deploy_model_worker`, `_safe_force_delete`, `multimodel_gateways`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `json`, `logging`, `pathlib`, `subprocess`
- **Third-party / 第三方**: `conftest`, `pytest`
- **Internal / 内部模块**: None explicitly imported / 未显式导入
