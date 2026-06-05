# reader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/metrics/reader.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `Metric`, `Counter`, `Vector` for the V1 `metrics` subsystem. / 为 V1 的 `metrics` 子系统实现 `Metric`, `Counter`, `Vector`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from dataclasses import dataclass

from prometheus_client import REGISTRY
from prometheus_client import Metric as PromMetric
from prometheus_client.samples import Sample
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `prometheus_client`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`prometheus_client` 等外部依赖。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `Metric` class / `Metric` 类
```python
@dataclass
class Metric:
    """A base class for prometheus metrics.

    Each metric may be associated with key=value labels, and
    in some cases a single vLLM instance may have multiple
    metrics with the same name but different sets of labels.
    """

    name: str
    labels: dict[str, str]
```
**EN:** Uses `@dataclass` to package related state for `Metric`. Typical fields include `name`, `labels`.
**CN:** `Metric` 使用 `@dataclass` 打包相关状态。典型字段包括 `name`, `labels`。

### `Counter` class / `Counter` 类
```python
@dataclass
class Counter(Metric):
    """A monotonically increasing integer counter."""

    value: int
```
**EN:** Uses `@dataclass` to package related state for `Counter`. Typical fields include `value`.
**CN:** `Counter` 使用 `@dataclass` 打包相关状态。典型字段包括 `value`。

### `Vector` class / `Vector` 类
```python
@dataclass
class Vector(Metric):
    """An ordered array of integer counters.

    This type - which doesn't exist in Prometheus - models one very
    specific metric, vllm:spec_decode_num_accepted_tokens_per_pos.
    """

    values: list[int]
```
**EN:** Uses `@dataclass` to package related state for `Vector`. Typical fields include `values`.
**CN:** `Vector` 使用 `@dataclass` 打包相关状态。典型字段包括 `values`。

### `Gauge` class / `Gauge` 类
```python
@dataclass
class Gauge(Metric):
    """A numerical value that can go up or down."""

    value: float
```
**EN:** Uses `@dataclass` to package related state for `Gauge`. Typical fields include `value`.
**CN:** `Gauge` 使用 `@dataclass` 打包相关状态。典型字段包括 `value`。

### `Histogram` class / `Histogram` 类
```python
@dataclass
class Histogram(Metric):
    """Observations recorded in configurable buckets.

    Buckets are represented by a dictionary. The key is
    the upper limit of the bucket, and the value is the
    observed count in that bucket. A '+Inf' key always
    exists.

    The count property is the total count across all
    buckets, identical to the count of the '+Inf' bucket.

    The sum property is the total sum of all observed
    values.
    """

    count: int
    sum: float
    buckets: dict[str, int]
```
**EN:** Uses `@dataclass` to package related state for `Histogram`. Typical fields include `count`, `sum`, `buckets`.
**CN:** `Histogram` 使用 `@dataclass` 打包相关状态。典型字段包括 `count`, `sum`, `buckets`。

### `get_metrics_snapshot` function / `get_metrics_snapshot` 函数
```python
def get_metrics_snapshot() -> list[Metric]:
    """An API for accessing in-memory Prometheus metrics.

    Example:
        >>> for metric in llm.get_metrics():
        ...     if isinstance(metric, Counter):
        ...         print(f"{metric} = {metric.value}")
        ...     elif isinstance(metric, Gauge):
        ...         print(f"{metric} = {metric.value}")
        ...     elif isinstance(metric, Histogram):
        ...         print(f"{metric}")
        ...         print(f"    sum = {metric.sum}")
        ...         print(f"    count = {metric.count}")
        ...         for bucket_le, value in metrics.buckets.items():
        ...             print(f"    {bucket_le} = {value}")
    """
    collected: list[Metric] = []
    for metric in REGISTRY.collect():
        if not metric.name.startswith("vllm:"):
            continue
        if metric.type == "gauge":
            samples = _get_samples(metric)
            for s in samples:
                collected.append(
                    Gauge(name=metric.name, labels=s.labels, value=s.value)
                )
        elif metric.type == "counter":
            samples = _get_samples(metric, "_total")
            if metric.name == "vllm:spec_decode_num_accepted_tokens_per_pos":
                #
                # Ugly vllm:num_accepted_tokens_per_pos special case.
                #
                # This metric is a vector of counters - for each spec
                # decoding token position, we observe the number of
                # accepted tokens using a Counter labeled with 'position'.
                # We convert these into a vector of integer values.
                #
                for labels, values in _digest_num_accepted_by_pos_samples(samples):
                    collected.append(
                        Vector(name=metric.name, labels=labels, values=values)
                    )
            else:
                for s in samples:
                    collected.append(
                        Counter(name=metric.name, labels=s.labels, value=int(s.value))
                    )

        elif metric.type == "histogram":
            #
            # A histogram has a number of '_bucket' samples where
            # the 'le' label represents the upper limit of the bucket.
            # We convert these bucketized values into a dict of values
            # indexed by the value of the 'le' label. The 'le=+Inf'
            # label is a special case, catching all values observed.
            #
            bucket_samples = _get_samples(metric, "_bucket")
            count_samples = _get_samples(metric, "_count")
            sum_samples = _get_samples(metric, "_sum")
            for labels, buckets, count_value, sum_value in _digest_histogram(
                bucket_samples, count_samples, sum_samples
            ):
                collected.append(
                    Histogram(
                        name=metric.name,
                        labels=labels,
                        buckets=buckets,
                        count=count_value,
                        sum=sum_value,
                    )
                )
        else:
            raise AssertionError(f"Unknown metric type {metric.type}")

    return collected
```
**EN:** This function returns or derives a value within the module. The docstring frames it as: An API for accessing in-memory Prometheus metrics. Key calls include `collect`, `startswith`, `_get_samples`, `append`, `Gauge`, `_digest_num_accepted_by_pos_samples`. The control flow contains 5 branch(es) and 5 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `collect`, `startswith`, `_get_samples`, `append`, `Gauge`, `_digest_num_accepted_by_pos_samples`。 控制流包含 5 个分支和 5 个循环，说明这里承担了较强的协调逻辑。

### `_get_samples` function / `_get_samples` 函数
```python
def _get_samples(metric: PromMetric, suffix: str | None = None) -> list[Sample]:
    name = (metric.name + suffix) if suffix is not None else metric.name
    return [s for s in metric.samples if s.name == name]
```
**EN:** This function implements `_get_samples` within the module. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_get_samples`，其作用域位于the module。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_strip_label` function / `_strip_label` 函数
```python
def _strip_label(labels: dict[str, str], key_to_remove: str) -> dict[str, str]:
    labels_copy = labels.copy()
    labels_copy.pop(key_to_remove)
    return labels_copy
```
**EN:** This function implements `_strip_label` within the module. Key calls include `copy`, `pop`.
**CN:** 该函数会实现 `_strip_label`，其作用域位于the module。 关键调用包括 `copy`, `pop`。

### `_digest_histogram` function / `_digest_histogram` 函数
```python
def _digest_histogram(
    bucket_samples: list[Sample], count_samples: list[Sample], sum_samples: list[Sample]
) -> list[tuple[dict[str, str], dict[str, int], int, float]]:
    #
    # In the case of DP, we have an indigestable
    # per-bucket-per-engine count as a list of labelled
    # samples, along with total and sum samples
    #
    # bucket_samples (in):
    #   labels = {bucket: 100, idx: 0}, value = 2
    #   labels = {bucket: 200, idx: 0}, value = 4
    #   labels = {bucket: Inf, idx: 0}, value = 10
    #   labels = {bucket: 100, idx: 1}, value = 1
    #   labels = {bucket: 200, idx: 2}, value = 5
    #   labels = {bucket: Inf, idx: 3}, value = 7
    # count_samples (in):
    #   labels = {idx: 0}, value = 10
    #   labels = {idx: 1}, value = 7
    # sum_samples (in):
    #   labels = {idx: 0}, value = 2000
    #   labels = {idx: 1}, value = 1200
    #
    # output: [
    #   {idx: 0}, {"100": 2, "200": 4, "Inf": 10}, 10, 2000
    #   {idx: 1}, {"100": 1, "200": 5, "Inf": 7},   7, 1200
    # ]
    buckets_by_labels: dict[frozenset[tuple[str, str]], dict[str, int]] = {}
    for s in bucket_samples:
        bucket = s.labels["le"]
        labels_key = frozenset(_strip_label(s.labels, "le").items())
        if labels_key not in buckets_by_labels:
            buckets_by_labels[labels_key] = {}
        buckets_by_labels[labels_key][bucket] = int(s.value)

    counts_by_labels: dict[frozenset[tuple[str, str]], int] = {}
    for s in count_samples:
        labels_key = frozenset(s.labels.items())
        counts_by_labels[labels_key] = int(s.value)

    sums_by_labels: dict[frozenset[tuple[str, str]], float] = {}
    for s in sum_samples:
        labels_key = frozenset(s.labels.items())
        sums_by_labels[labels_key] = s.value

    assert (
        set(buckets_by_labels.keys())
        == set(counts_by_labels.keys())
        == set(sums_by_labels.keys())
    )

    output = []
    label_keys = list(buckets_by_labels.keys())
    for k in label_keys:
        labels = dict(k)
        output.append(
            (labels, buckets_by_labels[k], counts_by_labels[k], sums_by_labels[k])
        )
    return output
```
**EN:** This function implements `_digest_histogram` within the module. Key calls include `list`, `frozenset`, `int`, `set`, `keys`, `dict`. The control flow contains 1 branch(es) and 4 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_digest_histogram`，其作用域位于the module。 关键调用包括 `list`, `frozenset`, `int`, `set`, `keys`, `dict`。 控制流包含 1 个分支和 4 个循环，说明这里承担了较强的协调逻辑。

### `_digest_num_accepted_by_pos_samples` function / `_digest_num_accepted_by_pos_samples` 函数
```python
def _digest_num_accepted_by_pos_samples(
    samples: list[Sample],
) -> list[tuple[dict[str, str], list[int]]]:
    #
    # In the case of DP, we have an indigestable
    # per-position-per-engine count as a list of
    # labelled samples
    #
    # samples (in):
    #   labels = {pos: 0, idx: 0}, value = 10
    #   labels = {pos: 1, idx: 0}, value = 7
    #   labels = {pos: 2, idx: 0}, value = 2
    #   labels = {pos: 0, idx: 1}, value = 5
    #   labels = {pos: 1, idx: 1}, value = 3
    #   labels = {pos: 2, idx: 1}, value = 1
    #
    # output: [
    #   {idx: 0}, [10, 7, 2]
    #   {idx: 1}, [5, 3, 1]
    # ]
    #
    max_pos = 0
    values_by_labels: dict[frozenset[tuple[str, str]], dict[int, int]] = {}

    for s in samples:
        position = int(s.labels["position"])
        max_pos = max(max_pos, position)

        labels_key = frozenset(_strip_label(s.labels, "position").items())
        if labels_key not in values_by_labels:
            values_by_labels[labels_key] = {}
        values_by_labels[labels_key][position] = int(s.value)

    output = []
    for labels_key, values_by_position in values_by_labels.items():
        labels = dict(labels_key)
        values = [0] * (max_pos + 1)
        for pos, val in values_by_position.items():
            values[pos] = val
        output.append((labels, values))
    return output
```
**EN:** This function implements `_digest_num_accepted_by_pos_samples` within the module. Key calls include `items`, `int`, `max`, `frozenset`, `dict`, `append`. The control flow contains 1 branch(es) and 3 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_digest_num_accepted_by_pos_samples`，其作用域位于the module。 关键调用包括 `items`, `int`, `max`, `frozenset`, `dict`, `append`。 控制流包含 1 个分支和 3 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `Metric`: central class or interface in this module. / `Metric`：本模块中的核心类或接口。
- `Counter`: central class or interface in this module. / `Counter`：本模块中的核心类或接口。
- `Vector`: central class or interface in this module. / `Vector`：本模块中的核心类或接口。
- `Gauge`: central class or interface in this module. / `Gauge`：本模块中的核心类或接口。
- `Histogram`: central class or interface in this module. / `Histogram`：本模块中的核心类或接口。
- `get_metrics_snapshot`: top-level helper or orchestration entry point. / `get_metrics_snapshot`：顶层辅助函数或编排入口。
- `_get_samples`: top-level helper or orchestration entry point. / `_get_samples`：顶层辅助函数或编排入口。
- `_strip_label`: top-level helper or orchestration entry point. / `_strip_label`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`
- External / 外部依赖: `prometheus_client`
