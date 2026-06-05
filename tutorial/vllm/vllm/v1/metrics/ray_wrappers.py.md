# ray_wrappers.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/metrics/ray_wrappers.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_get_replica_id`, `RayPrometheusMetric`, `RayGaugeWrapper` for the V1 `metrics` subsystem. / 为 V1 的 `metrics` 子系统实现 `_get_replica_id`, `RayPrometheusMetric`, `RayGaugeWrapper`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import copy
import time

from vllm.distributed.kv_transfer.kv_connector.v1.metrics import KVConnectorProm
from vllm.v1.metrics.loggers import PrometheusStatLogger
from vllm.v1.metrics.perf import PerfMetricsProm
from vllm.v1.spec_decode.metrics import SpecDecodingProm

try:
    from ray import serve as ray_serve
    from ray.util import metrics as ray_metrics
    from ray.util.metrics import Metric
except ImportError:
    ray_metrics = None
    ray_serve = None
import regex as re
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

### `_get_replica_id` function / `_get_replica_id` 函数
```python
def _get_replica_id() -> str | None:
    """Get the current Ray Serve replica ID, or None if not in a Serve context."""
    if ray_serve is None:
        return None
    try:
        return ray_serve.get_replica_context().replica_id.unique_id
    except ray_serve.exceptions.RayServeException:
        return None
```
**EN:** This function implements `_get_replica_id` within the module. The docstring frames it as: Get the current Ray Serve replica ID, or None if not in a Serve context. Key calls include `get_replica_context`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_get_replica_id`，其作用域位于the module。 关键调用包括 `get_replica_context`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RayPrometheusMetric` class / `RayPrometheusMetric` 类
```python
class RayPrometheusMetric:
    _is_labeled: bool = False
```
**EN:** Introduces the `RayPrometheusMetric` class. Core methods include `__init__`, `_get_tag_keys`, `_build_tags`, `labels`, `_get_sanitized_opentelemetry_name`.
**CN:** 这里定义 `RayPrometheusMetric` 类。核心方法包括 `__init__`, `_get_tag_keys`, `_build_tags`, `labels`, `_get_sanitized_opentelemetry_name`。

### `RayPrometheusMetric.__init__` method / `RayPrometheusMetric.__init__` 方法
```python
    def __init__(self):
        if ray_metrics is None:
            raise ImportError("RayPrometheusMetric requires Ray to be installed.")
        self.metric: Metric = None
        self._tags: dict[str, str] = {"ReplicaId": _get_replica_id() or ""}
```
**EN:** This method initializes the object state within `RayPrometheusMetric`. Key calls include `ImportError`, `_get_replica_id`. It touches state such as `metric`, `_tags`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`RayPrometheusMetric`。 关键调用包括 `ImportError`, `_get_replica_id`。 它会读写 `metric`, `_tags` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RayPrometheusMetric._get_tag_keys` method / `RayPrometheusMetric._get_tag_keys` 方法
```python
    @staticmethod
    def _get_tag_keys(labelnames: list[str] | None) -> tuple[str, ...]:
        labels = list(labelnames) if labelnames else []
        labels.append("ReplicaId")
        return tuple(labels)
```
**EN:** This method implements `_get_tag_keys` within `RayPrometheusMetric`. Key calls include `append`, `tuple`, `list`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_get_tag_keys`，其作用域位于`RayPrometheusMetric`。 关键调用包括 `append`, `tuple`, `list`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RayPrometheusMetric._build_tags` method / `RayPrometheusMetric._build_tags` 方法
```python
    def _build_tags(self, *labels, **labelskwargs) -> dict[str, str]:
        if labels:
            # -1 because ReplicaId was added automatically
            expected = len(self.metric._tag_keys) - 1
            if len(labels) != expected:
                raise ValueError(
                    "Number of labels must match the number of tag keys. "
                    f"Expected {expected}, got {len(labels)}"
                )
            labelskwargs.update(zip(self.metric._tag_keys, labels))

        labelskwargs["ReplicaId"] = _get_replica_id() or ""

        return {k: v if isinstance(v, str) else str(v) for k, v in labelskwargs.items()}
```
**EN:** This method implements `_build_tags` within `RayPrometheusMetric`. Key calls include `update`, `_get_replica_id`, `len`, `ValueError`, `zip`, `isinstance`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_build_tags`，其作用域位于`RayPrometheusMetric`。 关键调用包括 `update`, `_get_replica_id`, `len`, `ValueError`, `zip`, `isinstance`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RayPrometheusMetric.labels` method / `RayPrometheusMetric.labels` 方法
```python
    def labels(self, *labels, **labelskwargs) -> "RayPrometheusMetric":
        if self._is_labeled:
            raise ValueError("labels() cannot be called on an already-labeled metric.")
        clone = copy.copy(self)
        clone._tags = self._build_tags(*labels, **labelskwargs)
        clone._is_labeled = True
        return clone
```
**EN:** This method implements `labels` within `RayPrometheusMetric`. Key calls include `copy`, `_build_tags`, `ValueError`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `labels`，其作用域位于`RayPrometheusMetric`。 关键调用包括 `copy`, `_build_tags`, `ValueError`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RayPrometheusMetric._get_sanitized_opentelemetry_name` method / `RayPrometheusMetric._get_sanitized_opentelemetry_name` 方法
```python
    @staticmethod
    def _get_sanitized_opentelemetry_name(name: str) -> str:
        """
        For compatibility with Ray + OpenTelemetry, the metric name must be
        sanitized. In particular, this replaces disallowed character (e.g., ':')
        with '_' in the metric name.
        Allowed characters: a-z, A-Z, 0-9, _

        # ruff: noqa: E501
        Ref: https://github.com/open-telemetry/opentelemetry-cpp/blob/main/sdk/src/metrics/instrument_metadata_validator.cc#L22-L23
        Ref: https://github.com/ray-project/ray/blob/master/src/ray/stats/metric.cc#L107
        """

        return re.sub(r"[^a-zA-Z0-9_]", "_", name)
```
**EN:** This method implements `_get_sanitized_opentelemetry_name` within `RayPrometheusMetric`. The docstring frames it as: For compatibility with Ray + OpenTelemetry, the metric name must be sanitized. Key calls include `sub`.
**CN:** 该方法会实现 `_get_sanitized_opentelemetry_name`，其作用域位于`RayPrometheusMetric`。 关键调用包括 `sub`。

### `RayGaugeWrapper` class / `RayGaugeWrapper` 类
```python
class RayGaugeWrapper(RayPrometheusMetric):
    """Wraps around ray.util.metrics.Gauge to provide same API as
    prometheus_client.Gauge"""
```
**EN:** Introduces the `RayGaugeWrapper` class on top of `RayPrometheusMetric`. Core methods include `__init__`, `set`, `set_to_current_time`. Docstring signal: Wraps around ray.util.metrics.Gauge to provide same API as prometheus_client.Gauge
**CN:** 这里定义 `RayGaugeWrapper` 类，其基类包括 `RayPrometheusMetric`。核心方法包括 `__init__`, `set`, `set_to_current_time`。

### `RayGaugeWrapper.__init__` method / `RayGaugeWrapper.__init__` 方法
```python
    def __init__(
        self,
        name: str,
        documentation: str | None = "",
        labelnames: list[str] | None = None,
        multiprocess_mode: str | None = "",
    ):
        # All Ray metrics are keyed by WorkerId, so multiprocess modes like
        # "mostrecent", "all", "sum" do not apply. This logic can be manually
        # implemented at the observability layer (Prometheus/Grafana).
        del multiprocess_mode

        super().__init__()
        tag_keys = self._get_tag_keys(labelnames)
        name = self._get_sanitized_opentelemetry_name(name)

        self.metric = ray_metrics.Gauge(
            name=name,
            description=documentation,
            tag_keys=tag_keys,
        )
```
**EN:** This method initializes the object state within `RayGaugeWrapper`. Key calls include `__init__`, `_get_tag_keys`, `_get_sanitized_opentelemetry_name`, `Gauge`, `super`. It touches state such as `metric`.
**CN:** 该方法会初始化对象状态，其作用域位于`RayGaugeWrapper`。 关键调用包括 `__init__`, `_get_tag_keys`, `_get_sanitized_opentelemetry_name`, `Gauge`, `super`。 它会读写 `metric` 等状态。

### `RayGaugeWrapper.set` method / `RayGaugeWrapper.set` 方法
```python
    def set(self, value: int | float):
        return self.metric.set(value, tags=self._tags)
```
**EN:** This method implements `set` within `RayGaugeWrapper`. Key calls include `set`.
**CN:** 该方法会实现 `set`，其作用域位于`RayGaugeWrapper`。 关键调用包括 `set`。

### `RayGaugeWrapper.set_to_current_time` method / `RayGaugeWrapper.set_to_current_time` 方法
```python
    def set_to_current_time(self):
        # ray metrics doesn't have set_to_current time, https://docs.ray.io/en/latest/_modules/ray/util/metrics.html
        return self.set(time.time())
```
**EN:** This method stores a value into object state within `RayGaugeWrapper`. Key calls include `set`, `time`.
**CN:** 该方法会将值写入对象状态，其作用域位于`RayGaugeWrapper`。 关键调用包括 `set`, `time`。

### `RayCounterWrapper` class / `RayCounterWrapper` 类
```python
class RayCounterWrapper(RayPrometheusMetric):
    """Wraps around ray.util.metrics.Counter to provide same API as
    prometheus_client.Counter"""
```
**EN:** Introduces the `RayCounterWrapper` class on top of `RayPrometheusMetric`. Core methods include `__init__`, `inc`. Docstring signal: Wraps around ray.util.metrics.Counter to provide same API as prometheus_client.Counter
**CN:** 这里定义 `RayCounterWrapper` 类，其基类包括 `RayPrometheusMetric`。核心方法包括 `__init__`, `inc`。

### `RayCounterWrapper.__init__` method / `RayCounterWrapper.__init__` 方法
```python
    def __init__(
        self,
        name: str,
        documentation: str | None = "",
        labelnames: list[str] | None = None,
    ):
        super().__init__()
        tag_keys = self._get_tag_keys(labelnames)
        name = self._get_sanitized_opentelemetry_name(name)
        self.metric = ray_metrics.Counter(
            name=name,
            description=documentation,
            tag_keys=tag_keys,
        )
```
**EN:** This method initializes the object state within `RayCounterWrapper`. Key calls include `__init__`, `_get_tag_keys`, `_get_sanitized_opentelemetry_name`, `Counter`, `super`. It touches state such as `metric`.
**CN:** 该方法会初始化对象状态，其作用域位于`RayCounterWrapper`。 关键调用包括 `__init__`, `_get_tag_keys`, `_get_sanitized_opentelemetry_name`, `Counter`, `super`。 它会读写 `metric` 等状态。

### `RayCounterWrapper.inc` method / `RayCounterWrapper.inc` 方法
```python
    def inc(self, value: int | float = 1.0):
        if value == 0:
            return
        return self.metric.inc(value, tags=self._tags)
```
**EN:** This method implements `inc` within `RayCounterWrapper`. Key calls include `inc`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `inc`，其作用域位于`RayCounterWrapper`。 关键调用包括 `inc`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RayHistogramWrapper` class / `RayHistogramWrapper` 类
```python
class RayHistogramWrapper(RayPrometheusMetric):
    """Wraps around ray.util.metrics.Histogram to provide same API as
    prometheus_client.Histogram"""
```
**EN:** Introduces the `RayHistogramWrapper` class on top of `RayPrometheusMetric`. Core methods include `__init__`, `observe`. Docstring signal: Wraps around ray.util.metrics.Histogram to provide same API as prometheus_client.Histogram
**CN:** 这里定义 `RayHistogramWrapper` 类，其基类包括 `RayPrometheusMetric`。核心方法包括 `__init__`, `observe`。

### `RayHistogramWrapper.__init__` method / `RayHistogramWrapper.__init__` 方法
```python
    def __init__(
        self,
        name: str,
        documentation: str | None = "",
        labelnames: list[str] | None = None,
        buckets: list[float] | None = None,
    ):
        super().__init__()
        tag_keys = self._get_tag_keys(labelnames)
        name = self._get_sanitized_opentelemetry_name(name)

        boundaries = buckets if buckets else []
        self.metric = ray_metrics.Histogram(
            name=name,
            description=documentation,
            tag_keys=tag_keys,
            boundaries=boundaries,
        )
```
**EN:** This method initializes the object state within `RayHistogramWrapper`. Key calls include `__init__`, `_get_tag_keys`, `_get_sanitized_opentelemetry_name`, `Histogram`, `super`. It touches state such as `metric`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`RayHistogramWrapper`。 关键调用包括 `__init__`, `_get_tag_keys`, `_get_sanitized_opentelemetry_name`, `Histogram`, `super`。 它会读写 `metric` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RayHistogramWrapper.observe` method / `RayHistogramWrapper.observe` 方法
```python
    def observe(self, value: int | float):
        return self.metric.observe(value, tags=self._tags)
```
**EN:** This method implements `observe` within `RayHistogramWrapper`. Key calls include `observe`.
**CN:** 该方法会实现 `observe`，其作用域位于`RayHistogramWrapper`。 关键调用包括 `observe`。

### `RaySpecDecodingProm` class / `RaySpecDecodingProm` 类
```python
class RaySpecDecodingProm(SpecDecodingProm):
    """
    RaySpecDecodingProm is used by RayMetrics to log to Ray metrics.
    Provides the same metrics as SpecDecodingProm but uses Ray's
    util.metrics library.
    """

    _counter_cls = RayCounterWrapper
```
**EN:** Introduces the `RaySpecDecodingProm` class on top of `SpecDecodingProm`. Core methods include its methods defined below. Docstring signal: RaySpecDecodingProm is used by RayMetrics to log to Ray metrics.
**CN:** 这里定义 `RaySpecDecodingProm` 类，其基类包括 `SpecDecodingProm`。核心方法包括 下方定义的方法。

### `RayKVConnectorProm` class / `RayKVConnectorProm` 类
```python
class RayKVConnectorProm(KVConnectorProm):
    """
    RayKVConnectorProm is used by RayMetrics to log Ray
    metrics. Provides the same metrics as KV connectors but
    uses Ray's util.metrics library.
    """

    _gauge_cls = RayGaugeWrapper
    _counter_cls = RayCounterWrapper
    _histogram_cls = RayHistogramWrapper
```
**EN:** Introduces the `RayKVConnectorProm` class on top of `KVConnectorProm`. Core methods include its methods defined below. Docstring signal: RayKVConnectorProm is used by RayMetrics to log Ray metrics.
**CN:** 这里定义 `RayKVConnectorProm` 类，其基类包括 `KVConnectorProm`。核心方法包括 下方定义的方法。

### `RayPerfMetricsProm` class / `RayPerfMetricsProm` 类
```python
class RayPerfMetricsProm(PerfMetricsProm):
    """
    RayPerfMetricsProm is used by RayMetrics to log Ray
    metrics. Provides the same MFU metrics as PerfMetricsProm
    uses Ray's util.metrics library.
    """

    _counter_cls = RayCounterWrapper
```
**EN:** Introduces the `RayPerfMetricsProm` class on top of `PerfMetricsProm`. Core methods include its methods defined below. Docstring signal: RayPerfMetricsProm is used by RayMetrics to log Ray metrics.
**CN:** 这里定义 `RayPerfMetricsProm` 类，其基类包括 `PerfMetricsProm`。核心方法包括 下方定义的方法。

### `RayPrometheusStatLogger` class / `RayPrometheusStatLogger` 类
```python
class RayPrometheusStatLogger(PrometheusStatLogger):
    """RayPrometheusStatLogger uses Ray metrics instead."""

    _gauge_cls = RayGaugeWrapper
    _counter_cls = RayCounterWrapper
    _histogram_cls = RayHistogramWrapper
    _spec_decoding_cls = RaySpecDecodingProm
    _kv_connector_cls = RayKVConnectorProm
    _perf_metrics_cls = RayPerfMetricsProm
```
**EN:** Introduces the `RayPrometheusStatLogger` class on top of `PrometheusStatLogger`. Core methods include `_unregister_vllm_metrics`. Docstring signal: RayPrometheusStatLogger uses Ray metrics instead.
**CN:** 这里定义 `RayPrometheusStatLogger` 类，其基类包括 `PrometheusStatLogger`。核心方法包括 `_unregister_vllm_metrics`。

### `RayPrometheusStatLogger._unregister_vllm_metrics` method / `RayPrometheusStatLogger._unregister_vllm_metrics` 方法
```python
    @staticmethod
    def _unregister_vllm_metrics():
        # No-op on purpose
        pass
```
**EN:** This method implements `_unregister_vllm_metrics` within `RayPrometheusStatLogger`.
**CN:** 该方法会实现 `_unregister_vllm_metrics`，其作用域位于`RayPrometheusStatLogger`。

## Key Concepts / 关键概念
- `_get_replica_id`: top-level helper or orchestration entry point. / `_get_replica_id`：顶层辅助函数或编排入口。
- `RayPrometheusMetric`: central class or interface in this module. / `RayPrometheusMetric`：本模块中的核心类或接口。
- `RayGaugeWrapper`: central class or interface in this module. / `RayGaugeWrapper`：本模块中的核心类或接口。
- `RayCounterWrapper`: central class or interface in this module. / `RayCounterWrapper`：本模块中的核心类或接口。
- `RayHistogramWrapper`: central class or interface in this module. / `RayHistogramWrapper`：本模块中的核心类或接口。
- `RaySpecDecodingProm`: central class or interface in this module. / `RaySpecDecodingProm`：本模块中的核心类或接口。
- `RayKVConnectorProm`: central class or interface in this module. / `RayKVConnectorProm`：本模块中的核心类或接口。
- `RayPerfMetricsProm`: central class or interface in this module. / `RayPerfMetricsProm`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `copy`, `time`
- External / 外部依赖: `ray`, `regex`
- Internal vLLM / 内部依赖: `vllm.distributed.kv_transfer.kv_connector.v1.metrics`, `vllm.v1.metrics.loggers`, `vllm.v1.metrics.perf`, `vllm.v1.spec_decode.metrics`
