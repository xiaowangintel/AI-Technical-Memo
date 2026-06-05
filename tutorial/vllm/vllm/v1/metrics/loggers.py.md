# loggers.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/metrics/loggers.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `StatLoggerBase`, `load_stat_logger_plugin_factories`, `AggregateStatLoggerBase` for the V1 `metrics` subsystem. / 为 V1 的 `metrics` 子系统实现 `StatLoggerBase`, `load_stat_logger_plugin_factories`, `AggregateStatLoggerBase`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import logging
import time
from abc import ABC, abstractmethod
from collections.abc import Callable

from prometheus_client import Counter, Gauge, Histogram

import vllm.envs as envs
from vllm.compilation.cuda_graph import CUDAGraphLogging
from vllm.config import SupportsMetricsInfo, VllmConfig
from vllm.distributed.kv_transfer.kv_connector.v1.metrics import (
    KVConnectorLogging,
    KVConnectorProm,
)
from vllm.logger import init_logger
from vllm.plugins import STAT_LOGGER_PLUGINS_GROUP, load_plugins_by_group
from vllm.v1.engine import FinishReason
from vllm.v1.metrics.perf import PerfMetricsLogging, PerfMetricsProm
from vllm.v1.metrics.prometheus import unregister_vllm_metrics
from vllm.v1.metrics.stats import (
    CachingMetrics,
    IterationStats,
    MultiModalCacheStats,
    PromptTokenStats,
    SchedulerStats,
)
from vllm.v1.metrics.utils import create_metric_per_engine
from vllm.v1.spec_decode.metrics import SpecDecodingLogging, SpecDecodingProm

logger = init_logger(__name__)

# User-facing reason labels for waiting request breakdown
WAITING_REASON_CAPACITY = "capacity"
WAITING_REASON_DEFERRED = "deferred"

PerEngineStatLoggerFactory = Callable[[VllmConfig, int], "StatLoggerBase"]
AggregateStatLoggerFactory = type["AggregateStatLoggerBase"]
StatLoggerFactory = AggregateStatLoggerFactory | PerEngineStatLoggerFactory
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`, `WAITING_REASON_CAPACITY`, `WAITING_REASON_DEFERRED`, `PerEngineStatLoggerFactory`, `AggregateStatLoggerFactory`, `StatLoggerFactory`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`, `WAITING_REASON_CAPACITY`, `WAITING_REASON_DEFERRED`, `PerEngineStatLoggerFactory`, `AggregateStatLoggerFactory`, `StatLoggerFactory`。

### `StatLoggerBase` class / `StatLoggerBase` 类
```python
class StatLoggerBase(ABC):
    """Interface for logging metrics.

    API users may define custom loggers that implement this interface.
    However, note that the `SchedulerStats` and `IterationStats` classes
    are not considered stable interfaces and may change in future versions.
    """
```
**EN:** Declares the `StatLoggerBase` interface. Downstream implementations are expected to provide methods such as `__init__`, `record`, `log_engine_initialized`, `log`, `record_sleep_state`.
**CN:** `StatLoggerBase` 声明了一组接口约定。下游实现需要提供 `__init__`, `record`, `log_engine_initialized`, `log`, `record_sleep_state` 等方法。

### `StatLoggerBase.__init__` method / `StatLoggerBase.__init__` 方法
```python
    @abstractmethod
    def __init__(self, vllm_config: VllmConfig, engine_index: int = 0): ...
```
**EN:** This method initializes the object state within `StatLoggerBase`.
**CN:** 该方法会初始化对象状态，其作用域位于`StatLoggerBase`。

### `StatLoggerBase.record` method / `StatLoggerBase.record` 方法
```python
    @abstractmethod
    def record(
        self,
        scheduler_stats: SchedulerStats | None,
        iteration_stats: IterationStats | None,
        mm_cache_stats: MultiModalCacheStats | None = None,
        engine_idx: int = 0,
    ): ...
```
**EN:** This method implements `record` within `StatLoggerBase`.
**CN:** 该方法会实现 `record`，其作用域位于`StatLoggerBase`。

### `StatLoggerBase.log` method / `StatLoggerBase.log` 方法
```python
    def log(self):  # noqa
        pass
```
**EN:** This method implements `log` within `StatLoggerBase`.
**CN:** 该方法会实现 `log`，其作用域位于`StatLoggerBase`。

### `StatLoggerBase.record_sleep_state` method / `StatLoggerBase.record_sleep_state` 方法
```python
    def record_sleep_state(self, is_awake: int, level: int):  # noqa
        pass
```
**EN:** This method implements `record_sleep_state` within `StatLoggerBase`.
**CN:** 该方法会实现 `record_sleep_state`，其作用域位于`StatLoggerBase`。

### `load_stat_logger_plugin_factories` function / `load_stat_logger_plugin_factories` 函数
```python
def load_stat_logger_plugin_factories() -> list[StatLoggerFactory]:
    factories: list[StatLoggerFactory] = []

    for name, plugin_class in load_plugins_by_group(STAT_LOGGER_PLUGINS_GROUP).items():
        if not isinstance(plugin_class, type) or not issubclass(
            plugin_class, StatLoggerBase
        ):
            raise TypeError(
                f"Stat logger plugin {name!r} must be a subclass of "
                f"StatLoggerBase (got {plugin_class!r})."
            )

        factories.append(plugin_class)

    return factories
```
**EN:** This function loads external or cached state within the module. Key calls include `items`, `append`, `load_plugins_by_group`, `TypeError`, `isinstance`, `issubclass`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会加载外部或缓存状态，其作用域位于the module。 关键调用包括 `items`, `append`, `load_plugins_by_group`, `TypeError`, `isinstance`, `issubclass`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `AggregateStatLoggerBase` class / `AggregateStatLoggerBase` 类
```python
class AggregateStatLoggerBase(StatLoggerBase):
    """Abstract base class for loggers that
    aggregate across multiple DP engines."""
```
**EN:** Introduces the `AggregateStatLoggerBase` class on top of `StatLoggerBase`. Core methods include `__init__`. Docstring signal: Abstract base class for loggers that aggregate across multiple DP engines.
**CN:** 这里定义 `AggregateStatLoggerBase` 类，其基类包括 `StatLoggerBase`。核心方法包括 `__init__`。

### `AggregateStatLoggerBase.__init__` method / `AggregateStatLoggerBase.__init__` 方法
```python
    @abstractmethod
    def __init__(self, vllm_config: VllmConfig, engine_indexes: list[int]): ...
```
**EN:** This method initializes the object state within `AggregateStatLoggerBase`.
**CN:** 该方法会初始化对象状态，其作用域位于`AggregateStatLoggerBase`。

### `LoggingStatLogger` class / `LoggingStatLogger` 类
```python
class LoggingStatLogger(StatLoggerBase):
```
**EN:** Introduces the `LoggingStatLogger` class on top of `StatLoggerBase`. Core methods include `__init__`, `_reset`, `_enable_perf_stats`, `_track_iteration_stats`, `_get_throughput`, `log_prefix`.
**CN:** 这里定义 `LoggingStatLogger` 类，其基类包括 `StatLoggerBase`。核心方法包括 `__init__`, `_reset`, `_enable_perf_stats`, `_track_iteration_stats`, `_get_throughput`, `log_prefix`。

### `LoggingStatLogger.__init__` method / `LoggingStatLogger.__init__` 方法
```python
    def __init__(self, vllm_config: VllmConfig, engine_index: int = 0):
        self.engine_index = engine_index
        self.vllm_config = vllm_config
        self._reset(time.monotonic())

        self.last_scheduler_stats = SchedulerStats()

        # Caching metrics. This cannot be reset.
        # TODO: Make the interval configurable.
        self.prefix_caching_metrics = CachingMetrics()
        self.connector_prefix_caching_metrics = CachingMetrics()
        self.mm_caching_metrics = CachingMetrics()

        self.spec_decoding_logging = SpecDecodingLogging()
        kv_transfer_config = self.vllm_config.kv_transfer_config
        self.kv_connector_logging = KVConnectorLogging(kv_transfer_config)
        self.cudagraph_logging = None
        if self.vllm_config.observability_config.cudagraph_metrics:
            self.cudagraph_logging = CUDAGraphLogging(
                self.vllm_config.compilation_config.cudagraph_mode,
                self.vllm_config.compilation_config.cudagraph_capture_sizes,
            )
        self.last_prompt_throughput: float = 0.0
        self.last_generation_throughput: float = 0.0
        self.engine_is_idle = False
        self.aggregated = False

        if self._enable_perf_stats():
            self.perf_metrics_logging = PerfMetricsLogging(vllm_config)
```
**EN:** This method initializes the object state within `LoggingStatLogger`. Key calls include `_reset`, `SchedulerStats`, `CachingMetrics`, `SpecDecodingLogging`, `KVConnectorLogging`, `_enable_perf_stats`. It touches state such as `engine_index`, `vllm_config`, `last_scheduler_stats`, `prefix_caching_metrics`, `connector_prefix_caching_metrics`, `mm_caching_metrics`, `spec_decoding_logging`, `kv_connector_logging`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`LoggingStatLogger`。 关键调用包括 `_reset`, `SchedulerStats`, `CachingMetrics`, `SpecDecodingLogging`, `KVConnectorLogging`, `_enable_perf_stats`。 它会读写 `engine_index`, `vllm_config`, `last_scheduler_stats`, `prefix_caching_metrics`, `connector_prefix_caching_metrics`, `mm_caching_metrics`, `spec_decoding_logging`, `kv_connector_logging` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `LoggingStatLogger._update_stats` method / `LoggingStatLogger._update_stats` 方法
```python
    def _update_stats(self):
        now = time.monotonic()
        prompt_throughput = self._get_throughput(self.num_prompt_tokens, now)
        generation_throughput = self._get_throughput(self.num_generation_tokens, now)

        self._reset(now)
        self.engine_is_idle = not any(
            (
                prompt_throughput,
                generation_throughput,
                self.last_prompt_throughput,
                self.last_generation_throughput,
            )
        )
        self.last_generation_throughput = generation_throughput
        self.last_prompt_throughput = prompt_throughput
```
**EN:** This method implements `_update_stats` within `LoggingStatLogger`. Key calls include `monotonic`, `_get_throughput`, `_reset`, `any`. It touches state such as `engine_is_idle`, `last_generation_throughput`, `last_prompt_throughput`.
**CN:** 该方法会实现 `_update_stats`，其作用域位于`LoggingStatLogger`。 关键调用包括 `monotonic`, `_get_throughput`, `_reset`, `any`。 它会读写 `engine_is_idle`, `last_generation_throughput`, `last_prompt_throughput` 等状态。

### `LoggingStatLogger.aggregate_scheduler_stats` method / `LoggingStatLogger.aggregate_scheduler_stats` 方法
```python
    def aggregate_scheduler_stats(self):
        # noop for per engine loggers
        return
```
**EN:** This method implements `aggregate_scheduler_stats` within `LoggingStatLogger`.
**CN:** 该方法会实现 `aggregate_scheduler_stats`，其作用域位于`LoggingStatLogger`。

### `LoggingStatLogger.log` method / `LoggingStatLogger.log` 方法
```python
    def log(self):
        self._update_stats()
        self.aggregate_scheduler_stats()
        # Avoid log noise on an idle production system
        log_fn = logger.debug if self.engine_is_idle else logger.info
        # Format and print output.
        log_parts = [
            "Avg prompt throughput: %.1f tokens/s",
            "Avg generation throughput: %.1f tokens/s",
            "Running: %d reqs",
            "Waiting: %d reqs",
        ]
        total_waiting = (
            self.last_scheduler_stats.num_waiting_reqs
            + self.last_scheduler_stats.num_skipped_waiting_reqs
        )
        log_args: list[int | float | str] = [
            self.last_prompt_throughput,
            self.last_generation_throughput,
            self.last_scheduler_stats.num_running_reqs,
            total_waiting,
        ]

        if self.last_scheduler_stats.num_skipped_waiting_reqs > 0:
            log_parts.append("Deferred: %d reqs")
            log_args.append(self.last_scheduler_stats.num_skipped_waiting_reqs)

        if self.num_preemptions > 0:
            log_parts.append("Preemptions: %d")
            log_args.append(self.num_preemptions)

        log_parts.extend(
            [
                "GPU KV cache usage: %.1f%%",
                "Prefix cache hit rate: %.1f%%",
            ]
        )
        log_args.extend(
            [
                self.last_scheduler_stats.kv_cache_usage * 100,
                self.prefix_caching_metrics.hit_rate * 100,
            ]
        )

        if envs.VLLM_COMPUTE_NANS_IN_LOGITS:
            log_parts.append("Corrupted: %d reqs")
            log_args.append(self.num_corrupted_reqs)
        if not self.connector_prefix_caching_metrics.empty:
            log_parts.append("External prefix cache hit rate: %.1f%%")
            log_args.append(self.connector_prefix_caching_metrics.hit_rate * 100)
        if not self.mm_caching_metrics.empty:
            log_parts.append("MM cache hit rate: %.1f%%")
            log_args.append(self.mm_caching_metrics.hit_rate * 100)

        log_fn(
            self.log_prefix + ", ".join(log_parts),
            *log_args,
        )

        self.spec_decoding_logging.log(log_fn=log_fn)
        self.kv_connector_logging.log(log_fn=log_fn)
        if self.cudagraph_logging is not None:
            self.cudagraph_logging.log(log_fn=log_fn)
        if self._enable_perf_stats():
            self.perf_metrics_logging.log(log_fn=log_fn, log_prefix=self.log_prefix)
```
**EN:** This method implements `log` within `LoggingStatLogger`. Key calls include `_update_stats`, `aggregate_scheduler_stats`, `extend`, `log_fn`, `log`, `_enable_perf_stats`. The control flow contains 8 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `log`，其作用域位于`LoggingStatLogger`。 关键调用包括 `_update_stats`, `aggregate_scheduler_stats`, `extend`, `log_fn`, `log`, `_enable_perf_stats`。 控制流包含 8 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AggregatedLoggingStatLogger` class / `AggregatedLoggingStatLogger` 类
```python
class AggregatedLoggingStatLogger(LoggingStatLogger, AggregateStatLoggerBase):
```
**EN:** Introduces the `AggregatedLoggingStatLogger` class on top of `LoggingStatLogger`, `AggregateStatLoggerBase`. Core methods include `__init__`, `log_prefix`, `_enable_perf_stats`, `record`, `aggregate_scheduler_stats`, `log`.
**CN:** 这里定义 `AggregatedLoggingStatLogger` 类，其基类包括 `LoggingStatLogger`, `AggregateStatLoggerBase`。核心方法包括 `__init__`, `log_prefix`, `_enable_perf_stats`, `record`, `aggregate_scheduler_stats`, `log`。

### `AggregatedLoggingStatLogger.__init__` method / `AggregatedLoggingStatLogger.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        engine_indexes: list[int],
    ):
        self.engine_indexes = engine_indexes
        self.last_scheduler_stats_dict: dict[int, SchedulerStats] = {
            idx: SchedulerStats() for idx in self.engine_indexes
        }
        LoggingStatLogger.__init__(self, vllm_config, engine_index=-1)
        self.aggregated = True
```
**EN:** This method initializes the object state within `AggregatedLoggingStatLogger`. Key calls include `__init__`, `SchedulerStats`. It touches state such as `engine_indexes`, `last_scheduler_stats_dict`, `aggregated`.
**CN:** 该方法会初始化对象状态，其作用域位于`AggregatedLoggingStatLogger`。 关键调用包括 `__init__`, `SchedulerStats`。 它会读写 `engine_indexes`, `last_scheduler_stats_dict`, `aggregated` 等状态。

### `AggregatedLoggingStatLogger.record` method / `AggregatedLoggingStatLogger.record` 方法
```python
    def record(
        self,
        scheduler_stats: SchedulerStats | None,
        iteration_stats: IterationStats | None,
        mm_cache_stats: MultiModalCacheStats | None = None,
        engine_idx: int = 0,
    ):
        if engine_idx not in self.engine_indexes:
            logger.warning("Unexpected engine_idx: %d", engine_idx)
            return
        LoggingStatLogger.record(
            self,
            scheduler_stats,
            iteration_stats,
            mm_cache_stats=mm_cache_stats,
            engine_idx=engine_idx,
        )
        if scheduler_stats is not None:
            self.last_scheduler_stats_dict[engine_idx] = scheduler_stats
```
**EN:** This method implements `record` within `AggregatedLoggingStatLogger`. Key calls include `record`, `warning`. It touches state such as `last_scheduler_stats_dict`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `record`，其作用域位于`AggregatedLoggingStatLogger`。 关键调用包括 `record`, `warning`。 它会读写 `last_scheduler_stats_dict` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AggregatedLoggingStatLogger.aggregate_scheduler_stats` method / `AggregatedLoggingStatLogger.aggregate_scheduler_stats` 方法
```python
    def aggregate_scheduler_stats(self):
        self.last_scheduler_stats = SchedulerStats()
        for last_scheduler_stats in self.last_scheduler_stats_dict.values():
            self.last_scheduler_stats.num_waiting_reqs += (
                last_scheduler_stats.num_waiting_reqs
            )
            self.last_scheduler_stats.num_running_reqs += (
                last_scheduler_stats.num_running_reqs
            )
            self.last_scheduler_stats.num_skipped_waiting_reqs += (
                last_scheduler_stats.num_skipped_waiting_reqs
            )
            self.last_scheduler_stats.kv_cache_usage += (
                last_scheduler_stats.kv_cache_usage
            )
        self.last_scheduler_stats.kv_cache_usage /= len(self.last_scheduler_stats_dict)
```
**EN:** This method implements `aggregate_scheduler_stats` within `AggregatedLoggingStatLogger`. Key calls include `SchedulerStats`, `values`, `len`. It touches state such as `last_scheduler_stats`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `aggregate_scheduler_stats`，其作用域位于`AggregatedLoggingStatLogger`。 关键调用包括 `SchedulerStats`, `values`, `len`。 它会读写 `last_scheduler_stats` 等状态。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `AggregatedLoggingStatLogger.log_engine_initialized` method / `AggregatedLoggingStatLogger.log_engine_initialized` 方法
```python
    def log_engine_initialized(self):
        if self.vllm_config.cache_config.num_gpu_blocks:
            logger.info(
                "%d Engines: vllm cache_config_info with initialization "
                "after num_gpu_blocks is: %d",
                len(self.engine_indexes),
                self.vllm_config.cache_config.num_gpu_blocks,
            )
```
**EN:** This method implements `log_engine_initialized` within `AggregatedLoggingStatLogger`. Key calls include `info`, `len`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `log_engine_initialized`，其作用域位于`AggregatedLoggingStatLogger`。 关键调用包括 `info`, `len`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `PerEngineStatLoggerAdapter` class / `PerEngineStatLoggerAdapter` 类
```python
class PerEngineStatLoggerAdapter(AggregateStatLoggerBase):
```
**EN:** Introduces the `PerEngineStatLoggerAdapter` class on top of `AggregateStatLoggerBase`. Core methods include `__init__`, `record`, `log`, `log_engine_initialized`.
**CN:** 这里定义 `PerEngineStatLoggerAdapter` 类，其基类包括 `AggregateStatLoggerBase`。核心方法包括 `__init__`, `record`, `log`, `log_engine_initialized`。

### `PerEngineStatLoggerAdapter.__init__` method / `PerEngineStatLoggerAdapter.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        engine_indexes: list[int],
        per_engine_stat_logger_factory: PerEngineStatLoggerFactory,
    ) -> None:
        self.per_engine_stat_loggers = {}
        self.engine_indexes = engine_indexes
        for engine_index in engine_indexes:
            self.per_engine_stat_loggers[engine_index] = per_engine_stat_logger_factory(
                vllm_config, engine_index
            )
```
**EN:** This method initializes the object state within `PerEngineStatLoggerAdapter`. Key calls include `per_engine_stat_logger_factory`. It touches state such as `per_engine_stat_loggers`, `engine_indexes`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`PerEngineStatLoggerAdapter`。 关键调用包括 `per_engine_stat_logger_factory`。 它会读写 `per_engine_stat_loggers`, `engine_indexes` 等状态。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `PerEngineStatLoggerAdapter.record` method / `PerEngineStatLoggerAdapter.record` 方法
```python
    def record(
        self,
        scheduler_stats: SchedulerStats | None,
        iteration_stats: IterationStats | None,
        mm_cache_stats: MultiModalCacheStats | None = None,
        engine_idx: int = 0,
    ):
        if engine_idx not in self.per_engine_stat_loggers:
            logger.warning("Unexpected engine_idx: %d", engine_idx)
            return
        self.per_engine_stat_loggers[engine_idx].record(
            scheduler_stats,
            iteration_stats,
            mm_cache_stats=mm_cache_stats,
            engine_idx=engine_idx,
        )
```
**EN:** This method implements `record` within `PerEngineStatLoggerAdapter`. Key calls include `record`, `warning`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `record`，其作用域位于`PerEngineStatLoggerAdapter`。 关键调用包括 `record`, `warning`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `PerEngineStatLoggerAdapter.log` method / `PerEngineStatLoggerAdapter.log` 方法
```python
    def log(self):
        for per_engine_stat_logger in self.per_engine_stat_loggers.values():
            per_engine_stat_logger.log()
```
**EN:** This method implements `log` within `PerEngineStatLoggerAdapter`. Key calls include `values`, `log`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `log`，其作用域位于`PerEngineStatLoggerAdapter`。 关键调用包括 `values`, `log`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `PerEngineStatLoggerAdapter.log_engine_initialized` method / `PerEngineStatLoggerAdapter.log_engine_initialized` 方法
```python
    def log_engine_initialized(self):
        for per_engine_stat_logger in self.per_engine_stat_loggers.values():
            per_engine_stat_logger.log_engine_initialized()
```
**EN:** This method implements `log_engine_initialized` within `PerEngineStatLoggerAdapter`. Key calls include `values`, `log_engine_initialized`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `log_engine_initialized`，其作用域位于`PerEngineStatLoggerAdapter`。 关键调用包括 `values`, `log_engine_initialized`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `PrometheusStatLogger` class / `PrometheusStatLogger` 类
```python
class PrometheusStatLogger(AggregateStatLoggerBase):
    _gauge_cls = Gauge
    _counter_cls = Counter
    _histogram_cls = Histogram
    _spec_decoding_cls = SpecDecodingProm
    _kv_connector_cls = KVConnectorProm
    _perf_metrics_cls = PerfMetricsProm
```
**EN:** Introduces the `PrometheusStatLogger` class on top of `AggregateStatLoggerBase`. Core methods include `__init__`, `log_metrics_info`, `record`, `record_sleep_state`, `log_engine_initialized`.
**CN:** 这里定义 `PrometheusStatLogger` 类，其基类包括 `AggregateStatLoggerBase`。核心方法包括 `__init__`, `log_metrics_info`, `record`, `record_sleep_state`, `log_engine_initialized`。

### `PrometheusStatLogger.__init__` method / `PrometheusStatLogger.__init__` 方法
```python
    def __init__(
        self, vllm_config: VllmConfig, engine_indexes: list[int] | None = None
    ):
        if engine_indexes is None:
            engine_indexes = [0]

        self.engine_indexes = engine_indexes

        unregister_vllm_metrics()
        self.vllm_config = vllm_config
        # Use this flag to hide metrics that were deprecated in
        # a previous release and which will be removed future
        self.show_hidden_metrics = vllm_config.observability_config.show_hidden_metrics
        self.kv_cache_metrics_enabled = (
            vllm_config.observability_config.kv_cache_metrics
        )

        labelnames = ["model_name", "engine"]
        model_name = vllm_config.model_config.served_model_name
        max_model_len = vllm_config.model_config.max_model_len

        self.per_engine_labelvalues: dict[int, list[object]] = {
            idx: [model_name, str(idx)] for idx in engine_indexes
        }
        per_engine_labelvalues = self.per_engine_labelvalues

        self.spec_decoding_prom = self._spec_decoding_cls(
            vllm_config.speculative_config, labelnames, per_engine_labelvalues
        )
        self.kv_connector_prom = self._kv_connector_cls(
            vllm_config, labelnames, per_engine_labelvalues
        )
        self.perf_metrics_prom = self._perf_metrics_cls(
            vllm_config, labelnames, per_engine_labelvalues
        )

        #
        # Scheduler state
        #
        gauge_scheduler_running = self._gauge_cls(
            name="vllm:num_requests_running",
            documentation="Number of requests in model execution batches.",
            multiprocess_mode="mostrecent",
            labelnames=labelnames,
        )
        self.gauge_scheduler_running = create_metric_per_engine(
            gauge_scheduler_running, per_engine_labelvalues
        )

        gauge_scheduler_waiting = self._gauge_cls(
    # ... omitted for brevity ...
        #

        # TODO: This metric might be incorrect in case of using multiple
        # api_server counts which uses prometheus mp.
        self.gauge_lora_info: Gauge | None = None
        if vllm_config.lora_config is not None:
            if len(self.engine_indexes) > 1:
                logger.warning(
                    "vllm:lora_requests_info prometheus metrics may be "
                    "incorrect/misleading with data parallel deployments."
                )
            self.labelname_max_lora = "max_lora"
            self.labelname_waiting_lora_adapters = "waiting_lora_adapters"
            self.labelname_running_lora_adapters = "running_lora_adapters"
            self.max_lora = vllm_config.lora_config.max_loras
            self.gauge_lora_info = self._gauge_cls(
                name="vllm:lora_requests_info",
                documentation="Running stats on lora requests.",
                multiprocess_mode="sum",
                labelnames=[
                    self.labelname_max_lora,
                    self.labelname_waiting_lora_adapters,
                    self.labelname_running_lora_adapters,
                ],
            )
```
**EN:** This method initializes the object state within `PrometheusStatLogger`. Key calls include `unregister_vllm_metrics`, `_spec_decoding_cls`, `_kv_connector_cls`, `_perf_metrics_cls`, `_gauge_cls`, `create_metric_per_engine`. It touches state such as `engine_indexes`, `vllm_config`, `show_hidden_metrics`, `kv_cache_metrics_enabled`, `per_engine_labelvalues`, `spec_decoding_prom`, `kv_connector_prom`, `perf_metrics_prom`. The control flow contains 5 branch(es) and 4 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会初始化对象状态，其作用域位于`PrometheusStatLogger`。 关键调用包括 `unregister_vllm_metrics`, `_spec_decoding_cls`, `_kv_connector_cls`, `_perf_metrics_cls`, `_gauge_cls`, `create_metric_per_engine`。 它会读写 `engine_indexes`, `vllm_config`, `show_hidden_metrics`, `kv_cache_metrics_enabled`, `per_engine_labelvalues`, `spec_decoding_prom`, `kv_connector_prom`, `perf_metrics_prom` 等状态。 控制流包含 5 个分支和 4 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `PrometheusStatLogger.log_metrics_info` method / `PrometheusStatLogger.log_metrics_info` 方法
```python
    def log_metrics_info(self, type: str, config_obj: SupportsMetricsInfo):
        metrics_info = config_obj.metrics_info()
        metrics_info["engine"] = ""

        name, documentation = None, None
        if type == "cache_config":
            name = "vllm:cache_config_info"
            documentation = "Information of the LLMEngine CacheConfig"
        assert name is not None, f"Unknown metrics info type {type}"

        # Info type metrics are syntactic sugar for a gauge permanently set to 1
        # Since prometheus multiprocessing mode does not support Info, emulate
        # info here with a gauge.
        info_gauge = self._gauge_cls(
            name=name,
            documentation=documentation,
            multiprocess_mode="mostrecent",
            labelnames=metrics_info.keys(),
        )
        for engine_index in self.engine_indexes:
            metrics_info = config_obj.metrics_info()
            metrics_info["engine"] = str(engine_index)
            info_gauge.labels(**metrics_info).set(1)
```
**EN:** This method implements `log_metrics_info` within `PrometheusStatLogger`. Key calls include `metrics_info`, `_gauge_cls`, `str`, `set`, `keys`, `labels`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `log_metrics_info`，其作用域位于`PrometheusStatLogger`。 关键调用包括 `metrics_info`, `_gauge_cls`, `str`, `set`, `keys`, `labels`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `PrometheusStatLogger.record` method / `PrometheusStatLogger.record` 方法
```python
    def record(
        self,
        scheduler_stats: SchedulerStats | None,
        iteration_stats: IterationStats | None,
        mm_cache_stats: MultiModalCacheStats | None = None,
        engine_idx: int = 0,
    ):
        """Log to prometheus."""
        if scheduler_stats is not None:
            self.gauge_scheduler_running[engine_idx].set(
                scheduler_stats.num_running_reqs
            )
            total_waiting = (
                scheduler_stats.num_waiting_reqs
                + scheduler_stats.num_skipped_waiting_reqs
            )
            self.gauge_scheduler_waiting[engine_idx].set(total_waiting)
            self.gauge_waiting_by_reason[WAITING_REASON_CAPACITY][engine_idx].set(
                scheduler_stats.num_waiting_reqs
            )
            self.gauge_waiting_by_reason[WAITING_REASON_DEFERRED][engine_idx].set(
                scheduler_stats.num_skipped_waiting_reqs
            )
            self.gauge_kv_cache_usage[engine_idx].set(scheduler_stats.kv_cache_usage)

            self.counter_prefix_cache_queries[engine_idx].inc(
                scheduler_stats.prefix_cache_stats.queries
            )
            self.counter_prefix_cache_hits[engine_idx].inc(
                scheduler_stats.prefix_cache_stats.hits
            )

            if scheduler_stats.connector_prefix_cache_stats is not None:
                self.counter_connector_prefix_cache_queries[engine_idx].inc(
                    scheduler_stats.connector_prefix_cache_stats.queries
                )
                self.counter_connector_prefix_cache_hits[engine_idx].inc(
                    scheduler_stats.connector_prefix_cache_stats.hits
                )

            if scheduler_stats.spec_decoding_stats is not None:
                self.spec_decoding_prom.observe(
                    scheduler_stats.spec_decoding_stats, engine_idx
                )

            if scheduler_stats.kv_connector_stats is not None:
                self.kv_connector_prom.observe(
                    scheduler_stats.kv_connector_stats, engine_idx
                )

    # ... omitted for brevity ...
                finished_request.inference_time
            )
            self.histogram_decode_time_request[engine_idx].observe(
                finished_request.decode_time
            )
            # Calculate prefill KV compute (excludes cached tokens)
            prefill_kv_computed = finished_request.num_prompt_tokens - max(
                finished_request.num_cached_tokens, 0
            )
            self.histogram_prefill_kv_computed_request[engine_idx].observe(
                prefill_kv_computed
            )
            self.histogram_num_prompt_tokens_request[engine_idx].observe(
                finished_request.num_prompt_tokens
            )
            self.histogram_num_generation_tokens_request[engine_idx].observe(
                finished_request.num_generation_tokens
            )
            self.histogram_request_time_per_output_token[engine_idx].observe(
                finished_request.mean_time_per_output_token
            )
            if finished_request.max_tokens_param:
                self.histogram_max_tokens_request[engine_idx].observe(
                    finished_request.max_tokens_param
                )
```
**EN:** This method implements `record` within `PrometheusStatLogger`. The docstring frames it as: Log to prometheus. Key calls include `inc`, `observe`, `set`, `join`, `set_to_current_time`, `get_by_source`. The control flow contains 11 branch(es) and 8 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `record`，其作用域位于`PrometheusStatLogger`。 关键调用包括 `inc`, `observe`, `set`, `join`, `set_to_current_time`, `get_by_source`。 控制流包含 11 个分支和 8 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `PrometheusStatLogger.record_sleep_state` method / `PrometheusStatLogger.record_sleep_state` 方法
```python
    def record_sleep_state(self, sleep: int = 0, level: int = 0):
        awake = 1
        discard_all = 0
        weights_offloaded = 0

        if sleep == 1:
            awake = 0
            if level == 1:
                weights_offloaded = 1
            elif level == 2:
                discard_all = 1

        for engine_idx in self.engine_indexes:
            self.gauge_engine_sleep_state["discard_all"][engine_idx].set(discard_all)
            self.gauge_engine_sleep_state["weights_offloaded"][engine_idx].set(
                weights_offloaded
            )
            self.gauge_engine_sleep_state["awake"][engine_idx].set(awake)
```
**EN:** This method implements `record_sleep_state` within `PrometheusStatLogger`. Key calls include `set`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `record_sleep_state`，其作用域位于`PrometheusStatLogger`。 关键调用包括 `set`。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `build_buckets` function / `build_buckets` 函数
```python
def build_buckets(mantissa_lst: list[int], max_value: int) -> list[int]:
    """
    Builds a list of buckets with increasing powers of 10 multiplied by
    mantissa values until the value exceeds the specified maximum.

    """
    exponent = 0
    buckets: list[int] = []
    while True:
        for m in mantissa_lst:
            value = m * 10**exponent
            if value <= max_value:
                buckets.append(value)
            else:
                return buckets
        exponent += 1
```
**EN:** This function builds derived structures within the module. The docstring frames it as: Builds a list of buckets with increasing powers of 10 multiplied by mantissa values until the value exceeds the specified maximum. Key calls include `append`. The control flow contains 1 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会构建派生结构，其作用域位于the module。 关键调用包括 `append`。 控制流包含 1 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `build_1_2_5_buckets` function / `build_1_2_5_buckets` 函数
```python
def build_1_2_5_buckets(max_value: int) -> list[int]:
    """
    Example:
    >>> build_1_2_5_buckets(100)
    [1, 2, 5, 10, 20, 50, 100]
    """
    return build_buckets([1, 2, 5], max_value)
```
**EN:** This function builds derived structures within the module. The docstring frames it as: Example: >>> build_1_2_5_buckets(100) [1, 2, 5, 10, 20, 50, 100] Key calls include `build_buckets`.
**CN:** 该函数会构建派生结构，其作用域位于the module。 关键调用包括 `build_buckets`。

### `StatLoggerManager` class / `StatLoggerManager` 类
```python
class StatLoggerManager:
    """
    StatLoggerManager:
        Logging happens at the level of the EngineCore (per scheduler).
         * DP: >1 EngineCore per AsyncLLM - loggers for each EngineCore.
         * With Local Logger, just make N copies for N EngineCores.
         * With Prometheus, we need a single logger with N "labels"

        This class abstracts away this implementation detail from
        the AsyncLLM, allowing the AsyncLLM to just call .record()
        and .log() to a simple interface.
    """
```
**EN:** Introduces the `StatLoggerManager` class. Core methods include `__init__`, `record`, `record_sleep_state`, `log`, `log_engine_initialized`. Docstring signal: StatLoggerManager: Logging happens at the level of the EngineCore (per scheduler).
**CN:** 这里定义 `StatLoggerManager` 类。核心方法包括 `__init__`, `record`, `record_sleep_state`, `log`, `log_engine_initialized`。

### `StatLoggerManager.__init__` method / `StatLoggerManager.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        engine_idxs: list[int] | None = None,
        custom_stat_loggers: list[StatLoggerFactory] | None = None,
        enable_default_loggers: bool = True,
        aggregate_engine_logging: bool = False,
        client_count: int = 1,
    ):
        self.engine_indexes = engine_idxs if engine_idxs else [0]
        self.stat_loggers: list[AggregateStatLoggerBase] = []
        stat_logger_factories: list[StatLoggerFactory] = []
        if custom_stat_loggers is not None:
            stat_logger_factories.extend(custom_stat_loggers)
        if enable_default_loggers and logger.isEnabledFor(logging.INFO):
            if client_count > 1:
                logger.warning(
                    "AsyncLLM created with api_server_count more than 1; "
                    "disabling stats logging to avoid incomplete stats."
                )
            else:
                default_logger_factory = (
                    AggregatedLoggingStatLogger
                    if aggregate_engine_logging
                    else LoggingStatLogger
                )
                stat_logger_factories.append(default_logger_factory)
        custom_prometheus_logger: bool = False
        for stat_logger_factory in stat_logger_factories:
            if isinstance(stat_logger_factory, type) and issubclass(
                stat_logger_factory, AggregateStatLoggerBase
            ):
                global_stat_logger = stat_logger_factory(
                    vllm_config=vllm_config,
                    engine_indexes=self.engine_indexes,
                )
                if isinstance(global_stat_logger, PrometheusStatLogger):
                    custom_prometheus_logger = True
            else:
                # per engine logger
                global_stat_logger = PerEngineStatLoggerAdapter(
                    vllm_config=vllm_config,
                    engine_indexes=self.engine_indexes,
                    per_engine_stat_logger_factory=stat_logger_factory,  # type: ignore[arg-type]
                )
            self.stat_loggers.append(global_stat_logger)
        if not custom_prometheus_logger:
            self.stat_loggers.append(
                PrometheusStatLogger(vllm_config, self.engine_indexes)
            )
```
**EN:** This method initializes the object state within `StatLoggerManager`. Key calls include `extend`, `isEnabledFor`, `append`, `warning`, `isinstance`, `issubclass`. It touches state such as `engine_indexes`, `stat_loggers`. The control flow contains 8 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`StatLoggerManager`。 关键调用包括 `extend`, `isEnabledFor`, `append`, `warning`, `isinstance`, `issubclass`。 它会读写 `engine_indexes`, `stat_loggers` 等状态。 控制流包含 8 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `StatLoggerManager.record` method / `StatLoggerManager.record` 方法
```python
    def record(
        self,
        scheduler_stats: SchedulerStats | None,
        iteration_stats: IterationStats | None,
        mm_cache_stats: MultiModalCacheStats | None = None,
        engine_idx: int | None = None,
    ):
        if engine_idx is None:
            engine_idx = 0
        for stat_logger in self.stat_loggers:
            stat_logger.record(
                scheduler_stats,
                iteration_stats,
                mm_cache_stats=mm_cache_stats,
                engine_idx=engine_idx,
            )
```
**EN:** This method implements `record` within `StatLoggerManager`. Key calls include `record`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `record`，其作用域位于`StatLoggerManager`。 关键调用包括 `record`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `StatLoggerManager.record_sleep_state` method / `StatLoggerManager.record_sleep_state` 方法
```python
    def record_sleep_state(self, sleep: int = 0, level: int = 0):
        for logger in self.stat_loggers:
            logger.record_sleep_state(sleep, level)
```
**EN:** This method implements `record_sleep_state` within `StatLoggerManager`. Key calls include `record_sleep_state`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `record_sleep_state`，其作用域位于`StatLoggerManager`。 关键调用包括 `record_sleep_state`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `StatLoggerManager.log` method / `StatLoggerManager.log` 方法
```python
    def log(self):
        for logger in self.stat_loggers:
            logger.log()
```
**EN:** This method implements `log` within `StatLoggerManager`. Key calls include `log`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `log`，其作用域位于`StatLoggerManager`。 关键调用包括 `log`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `StatLoggerBase`: central class or interface in this module. / `StatLoggerBase`：本模块中的核心类或接口。
- `load_stat_logger_plugin_factories`: top-level helper or orchestration entry point. / `load_stat_logger_plugin_factories`：顶层辅助函数或编排入口。
- `AggregateStatLoggerBase`: central class or interface in this module. / `AggregateStatLoggerBase`：本模块中的核心类或接口。
- `LoggingStatLogger`: central class or interface in this module. / `LoggingStatLogger`：本模块中的核心类或接口。
- `AggregatedLoggingStatLogger`: central class or interface in this module. / `AggregatedLoggingStatLogger`：本模块中的核心类或接口。
- `PerEngineStatLoggerAdapter`: central class or interface in this module. / `PerEngineStatLoggerAdapter`：本模块中的核心类或接口。
- `PrometheusStatLogger`: central class or interface in this module. / `PrometheusStatLogger`：本模块中的核心类或接口。
- `build_buckets`: top-level helper or orchestration entry point. / `build_buckets`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `logging`, `time`, `abc`, `collections`
- External / 外部依赖: `prometheus_client`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.compilation.cuda_graph`, `vllm.config`, `vllm.distributed.kv_transfer.kv_connector.v1.metrics`, `vllm.logger`, `vllm.plugins`, `vllm.v1.engine`, `vllm.v1.metrics.perf`, `vllm.v1.metrics.prometheus`, `vllm.v1.metrics.stats`, `vllm.v1.metrics.utils`, `vllm.v1.spec_decode.metrics`
