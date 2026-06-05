# metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/kv_transfer/kv_connector/v1/metrics.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements a v1 KV-transfer connector, interface, or helper. / 实现 v1 KV 传输连接器、接口或辅助模块。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化
```python
from dataclasses import dataclass, field
from typing import Any, TypeAlias, TypeVar

from prometheus_client import Counter, Gauge, Histogram

from vllm.config import KVTransferConfig, VllmConfig
from vllm.distributed.kv_transfer.kv_connector.factory import KVConnectorFactory
from vllm.logger import init_logger
```
**EN:** This block imports `dataclasses`, `typing`, `prometheus_client`, `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.factory`, `vllm.logger` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `dataclasses`, `typing`, `prometheus_client`, `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.factory`, `vllm.logger`，为后续实现准备运行时、类型与辅助 API。

### Module constants / 模块常量
```python
PromMetric: TypeAlias = Gauge | Counter | Histogram
PromMetricT = TypeVar("PromMetricT", bound=PromMetric)

logger = init_logger(__name__)
```
**EN:** This section defines module-level aliases, constants, or shared state such as `PromMetric`, `PromMetricT`, `logger`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `PromMetric`, `PromMetricT`, `logger`，供后续代码复用。

### Class `KVConnectorStats` / 类 `KVConnectorStats`
```python
@dataclass
class KVConnectorStats:
    """
    Base class for KV Connector Stats, a container for transfer performance
    metrics or otherwise important telemetry from the connector.
    All sub-classes need to be serializable as stats are sent from worker to
    logger process.
    """

    data: dict[str, Any] = field(default_factory=dict)

    def reset(self):
        """Reset the stats, clear the state."""
        raise NotImplementedError

    def aggregate(self, other: "KVConnectorStats") -> "KVConnectorStats":
        """
        Aggregate stats with another `KVConnectorStats` object.
        """
        raise NotImplementedError

    def reduce(self) -> dict[str, int | float]:
        """
        Reduce the observations collected during a time interval to one or
        more representative values (eg avg/median/sum of the series).
        This is meant to be called by the logger to produce a summary of the
        stats for the last time interval.
        """
        raise NotImplementedError

    def is_empty(self) -> bool:
        """Return True if the stats are empty."""
        raise NotImplementedError
```
**EN:** Declares `KVConnectorStats`, a dataclass. It packages structured data fields such as `data`. The docstring summarizes its role as: Base class for KV Connector Stats, a container for transfer performance metrics or otherwise important telemetry from the connector. All....
**CN:** 声明 `KVConnectorStats`，它是一个数据类。 它封装了 `data` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Class `KVConnectorLogging` / 类 `KVConnectorLogging`
```python
class KVConnectorLogging:
    def __init__(self, kv_transfer_config: KVTransferConfig | None):
        # Instantiate the connector's stats class.
        if kv_transfer_config and kv_transfer_config.kv_connector:
            self.connector_cls = KVConnectorFactory.get_connector_class(
                kv_transfer_config
            )
        self.reset()

    def reset(self):
        self.transfer_stats_accumulator: KVConnectorStats | None = None

    def observe(self, transfer_stats_data: dict[str, Any]):
        # Should not be called when a KVConnector is not configured.
        assert self.connector_cls is not None
        # Called periodically when connector syncs with the scheduler.
        # Note that this is not the same as the logging interval.
        # We expect transfer_stats_data to be aggregated across all workers and
        # consist of observations from a single connector or a MultiConnector.
        transfer_stats = self.connector_cls.build_kv_connector_stats(
            transfer_stats_data
        )
        if transfer_stats is None:
            logger.warning_once(
                "The connector %s is collecting stats but "
                "does not implement the "
                "`build_kv_connector_stats` method. "
                "Stats will not be logged.",
                self.connector_cls,
            )
            return

        if self.transfer_stats_accumulator is None:
            self.transfer_stats_accumulator = transfer_stats
        else:
            # Accumulate last interval stats.
            self.transfer_stats_accumulator = self.transfer_stats_accumulator.aggregate(
                transfer_stats
            )

    def log(self, log_fn=logger.info):
        """Log transfer metrics periodically, similar to throughput logging"""
        if (
            self.transfer_stats_accumulator
            and not self.transfer_stats_accumulator.is_empty()
        ):
            # Produce a single cumulative stats object for the last time
            # interval from the recorded observations.
            xfer_metrics = self.transfer_stats_accumulator.reduce()
            xfer_metrics_str = ", ".join(f"{k}={v}" for k, v in xfer_metrics.items())
            log_fn("KV Transfer metrics: %s", xfer_metrics_str)

            # Reset metrics for next interval
            self.reset()
```
**EN:** Declares `KVConnectorLogging`, a class. Key methods include `__init__`, `reset`, `observe`, `log`.
**CN:** 声明 `KVConnectorLogging`，它是一个类。 关键方法包括 `__init__`, `reset`, `observe`, `log`。

### Class `KVConnectorPromMetrics` / 类 `KVConnectorPromMetrics`
```python
class KVConnectorPromMetrics:
    """
    A base class for per-connector Prometheus metric registration
    and recording.
    """

    def __init__(
        self,
        vllm_config: VllmConfig,
        metric_types: dict[type[PromMetric], type[PromMetricT]],
        labelnames: list[str],
        per_engine_labelvalues: dict[int, list[object]],
    ):
        self._kv_transfer_config = vllm_config.kv_transfer_config
        self._gauge_cls = metric_types[Gauge]
        self._counter_cls = metric_types[Counter]
        self._histogram_cls = metric_types[Histogram]
        self._labelnames = labelnames
        self.per_engine_labelvalues = per_engine_labelvalues

    def observe(self, transfer_stats_data: dict[str, Any], engine_idx: int = 0):
        """
        Record the supplied transfer statistics to Prometheus metrics. These
        statistics are engine-specific, and should be recorded to a metric
        with the appropriate 'engine' label. These metric instances can be
        created using the create_metric_per_engine() helper method.
        """
        raise NotImplementedError
```
**EN:** Declares `KVConnectorPromMetrics`, a class. Key methods include `__init__`, `observe`. The docstring summarizes its role as: A base class for per-connector Prometheus metric registration and recording.
**CN:** 声明 `KVConnectorPromMetrics`，它是一个类。 关键方法包括 `__init__`, `observe`。 文档字符串概括了它在整体流程中的职责。

### Class `KVConnectorProm` / 类 `KVConnectorProm`
```python
class KVConnectorProm:
    """
    Support for registering per-connector Prometheus metrics, and
    recording transfer statistics to those metrics. Uses
    KVConnectorBase.build_prom_metrics().
    """

    _gauge_cls = Gauge
    _counter_cls = Counter
    _histogram_cls = Histogram

    def __init__(
        self,
        vllm_config: VllmConfig,
        labelnames: list[str],
        per_engine_labelvalues: dict[int, list[object]],
    ):
        self.prom_metrics: KVConnectorPromMetrics | None = None
        kv_transfer_config = vllm_config.kv_transfer_config
        if kv_transfer_config and kv_transfer_config.kv_connector:
            connector_cls = KVConnectorFactory.get_connector_class(kv_transfer_config)
            metric_types = {
                Gauge: self._gauge_cls,
                Counter: self._counter_cls,
                Histogram: self._histogram_cls,
            }
            self.prom_metrics = connector_cls.build_prom_metrics(
                vllm_config,
                metric_types,
                labelnames,
                per_engine_labelvalues,
            )

    def observe(self, transfer_stats_data: dict[str, Any], engine_idx: int = 0):
        if self.prom_metrics is None:
            return
        self.prom_metrics.observe(transfer_stats_data, engine_idx)
```
**EN:** Declares `KVConnectorProm`, a class. Key methods include `__init__`, `observe`. The docstring summarizes its role as: Support for registering per-connector Prometheus metrics, and recording transfer statistics to those metrics. Uses....
**CN:** 声明 `KVConnectorProm`，它是一个类。 关键方法包括 `__init__`, `observe`。 文档字符串概括了它在整体流程中的职责。

## Key Concepts / 关键概念
- KV cache movement across workers / 跨工作进程的 KV 缓存迁移
- `KVConnectorStats`: dataclass interface or data carrier / `KVConnectorStats`：数据类接口或数据载体
- `KVConnectorLogging`: class interface or data carrier / `KVConnectorLogging`：类接口或数据载体
- `KVConnectorPromMetrics`: class interface or data carrier / `KVConnectorPromMetrics`：类接口或数据载体
- `KVConnectorProm`: class interface or data carrier / `KVConnectorProm`：类接口或数据载体

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Third-party / 第三方**: `prometheus_client`
- **Internal modules / 内部模块**: `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.factory`, `vllm.logger`
