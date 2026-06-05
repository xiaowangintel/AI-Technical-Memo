# conftest.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/k8s_integration/conftest.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This pytest configuration module defines shared fixtures, hooks, and markers for the Kubernetes integration suite. It centralizes reusable test setup so individual test files stay focused on assertions. / 该 pytest 配置模块为 Kubernetes 集成测试 定义共享夹具、钩子与标记，将可复用的测试初始化集中管理，使各测试文件能专注于断言。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Module docstring
```python
"""Pytest configuration for K8s integration tests.

These tests require:
  - A kind cluster named 'smg-test'
  - The smg-gateway:test image loaded into kind
  - kubectl configured to use the kind-smg-test context

Setup: ./e2e_test/k8s_integration/setup.sh
Teardown: ./e2e_test/k8s_integration/setup.sh teardown
"""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 12-23: Imports and dependencies
```python
from __future__ import annotations

import json
import logging
import socket
import subprocess
import time
from pathlib import Path
from typing import Callable

import httpx
import pytest
```
**EN:** This block imports `__future__`, `json`, `logging`, `socket`, and 6 more, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 25-46: Module constants and configuration
```python
logger = logging.getLogger(__name__)

NAMESPACE = "smg-test"
MANIFESTS_DIR = Path(__file__).parent / "manifests"
FAKE_WORKER_SCRIPT = Path(__file__).parent / "fake_worker.py"
KUBECTL_CONTEXT = "kind-smg-test"

# Reconciliation interval matches ServiceDiscoveryConfig.check_interval in
# sgl-model-gateway/src/service_discovery.rs (currently 60s).
RECONCILIATION_INTERVAL_SECS = 60
RECONCILIATION_WAIT_SECS = RECONCILIATION_INTERVAL_SECS + 30

# Errors safe to retry while polling: connection-level failures only.
# httpx.HTTPStatusError (4xx/5xx) is intentionally NOT included — a gateway
# returning 5xx is the kind of regression these tests should surface, not
# silently swallow as "transient".
_TRANSIENT_ERRORS = (
    httpx.TransportError,
    httpx.TimeoutException,
    ConnectionError,
    OSError,
)
```
**EN:** This section defines module-level names such as `logger`, `NAMESPACE`, `MANIFESTS_DIR`, `FAKE_WORKER_SCRIPT`, and 4 more, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 49-54: Helper function `pytest_configure`
```python
def pytest_configure(config):
    config.addinivalue_line(
        "markers",
        "slow: marks tests that wait for multiple reconciliation cycles "
        "(deselect with '-m \"not slow\"')",
    )
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 57-62: Helper function `_kubectl`
```python
def _kubectl(
    *args: str, check: bool = True, capture: bool = True
) -> subprocess.CompletedProcess:
    cmd = ["kubectl", "--context", KUBECTL_CONTEXT, *args]
    logger.debug("Running: %s", " ".join(cmd))
    return subprocess.run(cmd, capture_output=capture, text=True, check=check)
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 65-73: Helper function `_kubectl_json`
```python
def _kubectl_json(*args: str) -> dict:
    result = _kubectl(*args, "-o", "json")
    try:
        return json.loads(result.stdout)
    except json.JSONDecodeError as e:
        raise RuntimeError(
            f"Failed to parse kubectl JSON output for args={args!r}. "
            f"stdout={result.stdout!r}, stderr={result.stderr!r}"
        ) from e
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 76-86: Helper function `_wait_for_pod_ready`
```python
def _wait_for_pod_ready(name: str, namespace: str = NAMESPACE, timeout: int = 120):
    """Wait until a pod is Ready."""
    logger.info("Waiting for pod %s to be ready (timeout=%ds)", name, timeout)
    _kubectl(
        "wait",
        "--for=condition=Ready",
        f"pod/{name}",
        "-n",
        namespace,
        f"--timeout={timeout}s",
    )
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 89-101: Helper function `_wait_for_deployment_ready`
```python
def _wait_for_deployment_ready(
    name: str, namespace: str = NAMESPACE, timeout: int = 180
):
    """Wait until a deployment has all replicas available."""
    logger.info("Waiting for deployment %s to be ready (timeout=%ds)", name, timeout)
    _kubectl(
        "rollout",
        "status",
        f"deployment/{name}",
        "-n",
        namespace,
        f"--timeout={timeout}s",
    )
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 104-106: Helper function `_get_gateway_url`
```python
def _get_gateway_url() -> str:
    """Return the gateway URL, assuming port-forward is active on localhost:30000."""
    return "http://127.0.0.1:30000"
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 109-111: Helper function `_get_metrics_url`
```python
def _get_metrics_url() -> str:
    """Return the metrics URL, assuming port-forward is active on localhost:29000."""
    return "http://127.0.0.1:29000"
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 114-126: Helper function `_wait_for_port`
```python
def _wait_for_port(port: int, proc: subprocess.Popen, timeout: int = 15):
    """Poll until a TCP connection to localhost:port succeeds."""
    deadline = time.time() + timeout
    while time.time() < deadline:
        if proc.poll() is not None:
            stderr = proc.stderr.read().decode() if proc.stderr else ""
            raise RuntimeError(f"port-forward process exited early: {stderr}")
        try:
            with socket.create_connection(("127.0.0.1", port), timeout=1):
                return
        except OSError:
            time.sleep(0.5)
    raise TimeoutError(f"Port {port} not ready after {timeout}s")
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 129-139: Helper function `_get_workers`
```python
def _get_workers(gateway_url: str) -> dict:
    """GET /workers from the gateway and validate the response shape."""
    resp = httpx.get(f"{gateway_url}/workers", timeout=10)
    resp.raise_for_status()
    data = resp.json()
    if not isinstance(data, dict) or "total" not in data:
        raise ValueError(
            f"/workers returned unexpected structure (missing 'total' key): "
            f"{json.dumps(data)[:200]}"
        )
    return data
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 142-144: Helper function `_get_worker_count`
```python
def _get_worker_count(gateway_url: str) -> int:
    """Return total worker count from /workers."""
    return _get_workers(gateway_url)["total"]
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 147-178: Helper function `_poll_until`
```python
def _poll_until(
    predicate: Callable[[], bool],
    description: str,
    timeout: int,
    interval: float = 5,
) -> bool:
    """Poll a predicate until it returns True, or raise TimeoutError.

    Only transient network errors (see _TRANSIENT_ERRORS) are retried.
    Programming errors (KeyError, TypeError, etc.) and HTTP status errors
    (httpx.HTTPStatusError on 4xx/5xx) propagate immediately so real bugs
    aren't masked as "still polling".
    """
    deadline = time.time() + timeout
    last_error: Exception | None = None
    attempts = 0
    while time.time() < deadline:
        try:
            attempts += 1
            if predicate():
                logger.info(
                    "Condition met: %s (after %d attempts)", description, attempts
                )
                return True
        except _TRANSIENT_ERRORS as e:
            last_error = e
            logger.debug("Transient error on attempt %d: %s", attempts, e)
        time.sleep(interval)
    msg = f"Timeout waiting for: {description} (after {timeout}s, {attempts} attempts)"
    if last_error:
        msg += f" — last error: {last_error}"
    raise TimeoutError(msg)
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 181-198: Helper function `_port_forward_start`
```python
def _port_forward_start(
    namespace: str, service: str, local_port: int, remote_port: int
) -> subprocess.Popen:
    """Start kubectl port-forward and verify the port is reachable."""
    cmd = [
        "kubectl",
        "--context",
        KUBECTL_CONTEXT,
        "port-forward",
        f"svc/{service}",
        f"{local_port}:{remote_port}",
        "-n",
        namespace,
    ]
    logger.info("Starting port-forward: %s", " ".join(cmd))
    proc = subprocess.Popen(cmd, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    _wait_for_port(local_port, proc)
    return proc
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 199-215: Fixture `k8s_cluster`
```python


@pytest.fixture(scope="session")
def k8s_cluster():
    """Ensure the kind cluster exists and is reachable."""
    result = subprocess.run(
        ["kind", "get", "clusters"],
        capture_output=True,
        text=True,
        check=True,
    )
    if "smg-test" not in result.stdout:
        pytest.skip("kind cluster 'smg-test' not found — run setup first")

    # Verify kubectl connectivity
    _kubectl("cluster-info")
    return True
```
**EN:** This pytest fixture prepares `k8s_cluster` so multiple tests can reuse the same setup or cleanup sequence.
**CN:** 这个 pytest 夹具为 `k8s_cluster` 准备共享的初始化或清理流程，方便多个测试复用。

### Lines 216-287: Fixture `deploy_base`
```python


@pytest.fixture(scope="session")
def deploy_base(k8s_cluster):
    """Ensure namespace, RBAC, configmap, and gateway are deployed.

    Does NOT tear down at session end — use setup.sh teardown for that.
    This allows running pytest multiple times without full re-setup.
    """
    # Create namespace (apply is idempotent — succeeds if already exists)
    _kubectl("apply", "-f", str(MANIFESTS_DIR / "namespace.yaml"))

    # Create/update the fake-worker script as a ConfigMap
    cm_result = _kubectl(
        "create",
        "configmap",
        "fake-worker-script",
        f"--from-file=fake_worker.py={FAKE_WORKER_SCRIPT}",
        "-n",
        NAMESPACE,
        "--dry-run=client",
        "-o",
        "yaml",
    )
    _apply_from_stdin(cm_result.stdout)

    # Apply RBAC
    _kubectl("apply", "-f", str(MANIFESTS_DIR / "rbac.yaml"))

    # Apply gateway deployment
    _kubectl("apply", "-f", str(MANIFESTS_DIR / "gateway.yaml"))

    # Wait for gateway to be ready
    _wait_for_deployment_ready("smg-gateway")

    # Clean up any residual test pods from previous runs
    result = _kubectl(
        "get",
        "pods",
        "-n",
        NAMESPACE,
        "-l",
        "app=fake-worker",
        "-o",
        "jsonpath={.items[*].metadata.name}",
        check=False,
    )
    if result.returncode != 0:
        # Listing failed during fixture setup — surface it instead of silently
        # leaving stale workers around, which would skew worker-count assertions
        # in subsequent tests.
        logger.warning(
            "Failed to list residual fake-worker pods (rc=%d): %s",
            result.returncode,
            result.stderr.strip(),
        )
    elif result.stdout.strip():
        for pod_name in result.stdout.strip().split():
            _kubectl(
                "delete",
                "pod",
                pod_name,
                "-n",
                NAMESPACE,
                "--force",
                "--grace-period=0",
                "--ignore-not-found",
            )
        # Wait a bit for cleanup
        time.sleep(5)

    yield
```
**EN:** This pytest fixture prepares `deploy_base` so multiple tests can reuse the same setup or cleanup sequence.
**CN:** 这个 pytest 夹具为 `deploy_base` 准备共享的初始化或清理流程，方便多个测试复用。

### Lines 290-299: Helper function `_apply_from_stdin`
```python
def _apply_from_stdin(yaml_content: str):
    """Apply a YAML manifest from stdin."""
    proc = subprocess.run(
        ["kubectl", "--context", KUBECTL_CONTEXT, "apply", "-f", "-"],
        input=yaml_content,
        capture_output=True,
        text=True,
        check=True,
    )
    return proc
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 302-328: Helper function `_cleanup_port_forward`
```python
def _cleanup_port_forward(name: str, pf: subprocess.Popen):
    """Terminate a port-forward process; always log final exit state."""
    try:
        pf.terminate()
        pf.wait(timeout=10)
    except subprocess.TimeoutExpired:
        logger.warning(
            "Port-forward %s did not exit on SIGTERM after 10s; killing", name
        )
        pf.kill()
        try:
            pf.wait(timeout=5)
        except subprocess.TimeoutExpired:
            logger.warning("Port-forward %s still running after SIGKILL", name)
    except Exception as e:
        logger.warning("Error cleaning up %s port-forward: %s", name, e)

    rc = pf.returncode
    stderr = pf.stderr.read().decode() if pf.stderr else ""
    # rc == -15 (-SIGTERM) is the clean shutdown case; anything else is worth
    # surfacing — including rc is None, which means terminate failed silently
    # and the process may still be alive.
    if rc != -15:
        suffix = f": {stderr.strip()}" if stderr.strip() else ""
        logger.warning("Port-forward %s exited rc=%s%s", name, rc, suffix)
    else:
        logger.debug("Port-forward %s exited cleanly (rc=%s)", name, rc)
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 329-343: Fixture `gateway_port_forward`
```python


@pytest.fixture(scope="session")
def gateway_port_forward(deploy_base):
    """Set up port-forwarding to the gateway service."""
    pf_http = _port_forward_start(NAMESPACE, "smg-gateway", 30000, 30000)
    try:
        pf_metrics = _port_forward_start(NAMESPACE, "smg-gateway", 29000, 29000)
    except Exception:
        pf_http.terminate()
        pf_http.wait()
        raise
    yield _get_gateway_url(), _get_metrics_url()
    _cleanup_port_forward("http", pf_http)
    _cleanup_port_forward("metrics", pf_metrics)
```
**EN:** This pytest fixture prepares `gateway_port_forward` so multiple tests can reuse the same setup or cleanup sequence.
**CN:** 这个 pytest 夹具为 `gateway_port_forward` 准备共享的初始化或清理流程，方便多个测试复用。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Kubernetes lifecycle and reconciliation scenarios / Kubernetes 生命周期与协调场景
- Reusable functions: `pytest_configure`, `_kubectl`, `_kubectl_json`, `_wait_for_pod_ready`, and 13 more / 可复用函数：`pytest_configure`, `_kubectl`, `_kubectl_json`, `_wait_for_pod_ready`, and 13 more

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `json`, `logging`, `pathlib`, `socket`, `subprocess`, `time`, `typing`
- **Third-party / 第三方**: `httpx`, `pytest`
- **Internal / 内部模块**: None explicitly imported / 未显式导入
