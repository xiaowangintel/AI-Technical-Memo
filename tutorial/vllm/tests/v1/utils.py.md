# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides shared helpers reused by the v1 v1 tests. / 为 v1 `v1` 测试提供可复用的辅助函数。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-7)
```python
import pytest
import regex as re
import requests

from tests.utils import RemoteOpenAIServer
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, regex, requests`. Local helpers come from `tests.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, regex, requests`。 本地测试辅助逻辑来自 `tests.utils`。

### get_prometheus_metrics (lines 12-66)
```python
def get_prometheus_metrics(server: RemoteOpenAIServer) -> dict[str, dict[str, float]]:
    """Fetch and parse Prometheus metrics from the /metrics endpoint.

    Returns:
        Dict mapping metric names to their values grouped by labels.
        For example: {"vllm:request_success": {
            "engine=0": 5.0, "engine=1": 3.0}
        }
    """
    try:
        response = requests.get(server.url_for("metrics"), timeout=10)
        response.raise_for_status()
        metrics: dict[str, dict[str, float]] = {}
        # Regex patterns for Prometheus metrics
        metric_with_labels = re.compile(
            r"^([a-zA-Z_:][a-zA-Z0-9_:]*)\{([^}]*)\}\s+([\d\.\-\+e]+)$"
    # ... excerpt omitted for brevity ...
                        continue
        return metrics
    except Exception as e:
        pytest.fail(f"Failed to fetch Prometheus metrics: {e}")
        return {}
```
**EN:** Helper function `get_prometheus_metrics` encapsulates reusable logic for `prometheus metrics`. Inputs: `server`. Key calls include `requests.get, response.raise_for_status, re.compile, text.split, server.url_for, line.strip`.
**CN:** 辅助函数 `get_prometheus_metrics` 封装了与 `prometheus metrics` 相关的可复用逻辑。 输入参数：`server`。 关键调用包括 `requests.get, response.raise_for_status, re.compile, text.split, server.url_for, line.strip`。

### get_engine_request_counts (lines 69-91)
```python
def get_engine_request_counts(metrics: dict[str, dict[str, float]]) -> dict[str, float]:
    """Extract request counts per engine from Prometheus metrics.

    Returns:
        Dict mapping engine indices to request counts.
        For example: {"0": 15.0, "1": 12.0}
    """
    engine_counts = {}

    # Look for request success metrics with engine labels
    success_metrics = metrics.get("vllm:request_success_total", {})
    engine_pattern = re.compile(r'engine="([^"]*)"')

    for labels, count in success_metrics.items():
        # Extract engine ID from labels using regex
        match = engine_pattern.search(labels)
        if match:
            engine_id = match.group(1)
            if engine_id not in engine_counts:
                engine_counts[engine_id] = 0.0
            engine_counts[engine_id] += count

    return engine_counts
```
**EN:** Helper function `get_engine_request_counts` encapsulates reusable logic for `engine request counts`. Inputs: `metrics`. Key calls include `metrics.get, re.compile, success_metrics.items, engine_pattern.search, match.group`.
**CN:** 辅助函数 `get_engine_request_counts` 封装了与 `引擎 request counts` 相关的可复用逻辑。 输入参数：`metrics`。 关键调用包括 `metrics.get, re.compile, success_metrics.items, engine_pattern.search, match.group`。

### check_request_balancing (lines 94-125)
```python
def check_request_balancing(server: RemoteOpenAIServer, dp_size: int):
    """Check request balancing via Prometheus metrics if dp_size > 1.

    Args:
        server: The RemoteOpenAIServer instance
        dp_size: Number of data parallel ranks
    """
    if dp_size <= 1:
        return

    # Get metrics after all requests are completed
    metrics = get_prometheus_metrics(server)
    engine_counts = get_engine_request_counts(metrics)

    # Check that multiple engines received requests
    engines_with_requests = [
        engine for engine, count in engine_counts.items() if count > 0
    ]
    assert len(engines_with_requests) == dp_size, (
        f"Expected requests to be distributed across multiple engines,"
        f" but only engine(s) {engines_with_requests} received "
        f"requests. Engine counts: {engine_counts}"
    )

    # Verify that the load is reasonably balanced
    # (no engine should handle all requests)
    total_requests = sum(engine_counts.values())

    for count in engine_counts.values():
        assert count > total_requests // (dp_size + 1), (
            f"requests are imbalanced: {engine_counts}"
        )
```
**EN:** Helper function `check_request_balancing` encapsulates reusable logic for `check request balancing`. Inputs: `server, dp_size`. Key calls include `get_prometheus_metrics, get_engine_request_counts, sum, engine_counts.values, len, engine_counts.items`. It includes 2 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `check_request_balancing` 封装了与 `check request balancing` 相关的可复用逻辑。 输入参数：`server, dp_size`。 关键调用包括 `get_prometheus_metrics, get_engine_request_counts, sum, engine_counts.values, len, engine_counts.items`。 其中包含 2 个内部断言，用于保护前置假设。

## Key Concepts / 关键概念
- **EN:** Engine-core request/response orchestration
- **CN:** 引擎核心的请求/响应编排

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, regex, requests`.
- **CN:** 外部库：`pytest, regex, requests`。
- **EN:** Local test helpers: `tests.utils`.
- **CN:** 本地测试辅助模块：`tests.utils`。
