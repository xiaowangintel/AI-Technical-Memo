# test_metrics_reader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/metrics/test_metrics_reader.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `metrics reader` behavior and regressions in the v1 stack. / 验证 v1 栈中 `metrics reader` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-13)
```python
import prometheus_client
import pytest

from vllm.v1.metrics.reader import (
    Counter,
    Gauge,
    Histogram,
    Vector,
    get_metrics_snapshot,
)
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `prometheus_client, pytest`. vLLM modules under test include `vllm.v1.metrics.reader`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `prometheus_client, pytest`。 被测试的 vLLM 模块包括 `vllm.v1.metrics.reader`。

### Module state / 模块级状态 (line 15)
```python
pytestmark = pytest.mark.cpu_test
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `pytestmark`. It also sets pytest marks that scope the whole file.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`pytestmark`。 该块还设置了作用于整个文件的 pytest 标记。

### test_registry (lines 19-23)
```python
def test_registry(monkeypatch):
    # Use a custom registry for tests
    test_registry = prometheus_client.CollectorRegistry(auto_describe=True)
    monkeypatch.setattr("vllm.v1.metrics.reader.REGISTRY", test_registry)
    return test_registry
```
**EN:** Test case covering `registry`. Inputs/fixtures: `monkeypatch`. It exercises `pytest.fixture, prometheus_client.CollectorRegistry, monkeypatch.setattr`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `registry` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `pytest.fixture, prometheus_client.CollectorRegistry, monkeypatch.setattr`。 主要通过 mock、回调或输出检查来完成验证。

### test_gauge_metric (lines 27-46)
```python
def test_gauge_metric(test_registry, num_engines):
    g = prometheus_client.Gauge(
        "vllm:test_gauge",
        "Test gauge metric",
        labelnames=["model", "engine_index"],
        registry=test_registry,
    )
    for i in range(num_engines):
        g.labels(model="foo", engine_index=str(i)).set(98.5)

    metrics = get_metrics_snapshot()
    assert len(metrics) == num_engines
    engine_labels = [str(i) for i in range(num_engines)]
    for m in metrics:
        assert isinstance(m, Gauge)
        assert m.name == "vllm:test_gauge"
        assert m.value == 98.5
        assert m.labels["model"] == "foo"
        assert m.labels["engine_index"] in engine_labels
        engine_labels.remove(m.labels["engine_index"])
```
**EN:** Parameterized test covering `gauge metric`. Parameter axes: `num_engines`. Inputs/fixtures: `test_registry, num_engines`. It exercises `mark.parametrize, prometheus_client.Gauge, range, get_metrics_snapshot, labels.set, len`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `gauge metric` 的测试用例。 参数维度：`num_engines`。 输入或 fixture：`test_registry, num_engines`。 该测试会调用 `mark.parametrize, prometheus_client.Gauge, range, get_metrics_snapshot, labels.set, len`。 代码主体包含 6 个显式断言。

### test_counter_metric (lines 50-69)
```python
def test_counter_metric(test_registry, num_engines):
    c = prometheus_client.Counter(
        "vllm:test_counter",
        "Test counter metric",
        labelnames=["model", "engine_index"],
        registry=test_registry,
    )
    for i in range(num_engines):
        c.labels(model="bar", engine_index=str(i)).inc(19)

    metrics = get_metrics_snapshot()
    assert len(metrics) == num_engines
    engine_labels = [str(i) for i in range(num_engines)]
    for m in metrics:
        assert isinstance(m, Counter)
        assert m.name == "vllm:test_counter"
        assert m.value == 19
        assert m.labels["model"] == "bar"
        assert m.labels["engine_index"] in engine_labels
        engine_labels.remove(m.labels["engine_index"])
```
**EN:** Parameterized test covering `counter metric`. Parameter axes: `num_engines`. Inputs/fixtures: `test_registry, num_engines`. It exercises `mark.parametrize, prometheus_client.Counter, range, get_metrics_snapshot, labels.inc, len`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `counter metric` 的测试用例。 参数维度：`num_engines`。 输入或 fixture：`test_registry, num_engines`。 该测试会调用 `mark.parametrize, prometheus_client.Counter, range, get_metrics_snapshot, labels.inc, len`。 代码主体包含 6 个显式断言。

### test_histogram_metric (lines 73-102)
```python
def test_histogram_metric(test_registry, num_engines):
    h = prometheus_client.Histogram(
        "vllm:test_histogram",
        "Test histogram metric",
        labelnames=["model", "engine_index"],
        buckets=[10, 20, 30, 40, 50],
        registry=test_registry,
    )
    for i in range(num_engines):
        hist = h.labels(model="blaa", engine_index=str(i))
        hist.observe(42)
        hist.observe(21)
        hist.observe(7)

    metrics = get_metrics_snapshot()
    assert len(metrics) == num_engines
    engine_labels = [str(i) for i in range(num_engines)]
    for m in metrics:
        assert isinstance(m, Histogram)
        assert m.name == "vllm:test_histogram"
        assert m.count == 3
        assert m.sum == 70
        assert m.buckets["10.0"] == 1
        assert m.buckets["20.0"] == 1
        assert m.buckets["30.0"] == 2
        assert m.buckets["40.0"] == 2
        assert m.buckets["50.0"] == 3
        assert m.labels["model"] == "blaa"
        assert m.labels["engine_index"] in engine_labels
        engine_labels.remove(m.labels["engine_index"])
```
**EN:** Parameterized test covering `histogram metric`. Parameter axes: `num_engines`. Inputs/fixtures: `test_registry, num_engines`. It exercises `mark.parametrize, prometheus_client.Histogram, range, get_metrics_snapshot, h.labels, hist.observe`. The body contains 12 explicit assertion(s).
**CN:** 该代码块是覆盖 `histogram metric` 的测试用例。 参数维度：`num_engines`。 输入或 fixture：`test_registry, num_engines`。 该测试会调用 `mark.parametrize, prometheus_client.Histogram, range, get_metrics_snapshot, h.labels, hist.observe`。 代码主体包含 12 个显式断言。

### test_vector_metric (lines 106-127)
```python
def test_vector_metric(test_registry, num_engines):
    c = prometheus_client.Counter(
        "vllm:spec_decode_num_accepted_tokens_per_pos",
        "Vector-like counter metric",
        labelnames=["position", "model", "engine_index"],
        registry=test_registry,
    )
    for i in range(num_engines):
        c.labels(position="0", model="llama", engine_index=str(i)).inc(10)
        c.labels(position="1", model="llama", engine_index=str(i)).inc(5)
        c.labels(position="2", model="llama", engine_index=str(i)).inc(1)

    metrics = get_metrics_snapshot()
    assert len(metrics) == num_engines
    engine_labels = [str(i) for i in range(num_engines)]
    for m in metrics:
        assert isinstance(m, Vector)
        assert m.name == "vllm:spec_decode_num_accepted_tokens_per_pos"
        assert m.values == [10, 5, 1]
        assert m.labels["model"] == "llama"
        assert m.labels["engine_index"] in engine_labels
        engine_labels.remove(m.labels["engine_index"])
```
**EN:** Parameterized test covering `vector metric`. Parameter axes: `num_engines`. Inputs/fixtures: `test_registry, num_engines`. It exercises `mark.parametrize, prometheus_client.Counter, range, get_metrics_snapshot, labels.inc, len`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `vector metric` 的测试用例。 参数维度：`num_engines`。 输入或 fixture：`test_registry, num_engines`。 该测试会调用 `mark.parametrize, prometheus_client.Counter, range, get_metrics_snapshot, labels.inc, len`。 代码主体包含 6 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围

## Dependencies / 依赖关系
- **EN:** External libraries: `prometheus_client, pytest`.
- **CN:** 外部库：`prometheus_client, pytest`。
- **EN:** vLLM modules under test: `vllm.v1.metrics.reader`.
- **CN:** 被测试的 vLLM 模块：`vllm.v1.metrics.reader`。
