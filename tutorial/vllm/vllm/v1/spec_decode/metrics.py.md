# metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/spec_decode/metrics.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `SpecDecodingStats`, `SpecDecodingLogging`, `SpecDecodingProm` for the V1 `spec_decode` subsystem. / 为 V1 的 `spec_decode` 子系统实现 `SpecDecodingStats`, `SpecDecodingLogging`, `SpecDecodingProm`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import time
from dataclasses import dataclass, field

import numpy as np
import prometheus_client

from vllm.config import SpeculativeConfig
from vllm.logger import init_logger
from vllm.v1.metrics.utils import create_metric_per_engine

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `SpecDecodingStats` class / `SpecDecodingStats` 类
```python
@dataclass
class SpecDecodingStats:
    """Per-step iteration decoding stats from scheduler.

    Each scheduler step, statistics on spec decoding performance are
    aggregated across requests by the scheduler and returned to the
    frontend in EngineCoreOutputs->SchedulerStats.
    """

    num_spec_tokens: int
    num_drafts: int = 0
    num_draft_tokens: int = 0
    num_accepted_tokens: int = 0
    num_accepted_tokens_per_pos: list[int] = field(default_factory=list)
```
**EN:** Uses `@dataclass` to package related state for `SpecDecodingStats`. Typical fields include `num_spec_tokens`, `num_drafts`, `num_draft_tokens`, `num_accepted_tokens`, `num_accepted_tokens_per_pos`.
**CN:** `SpecDecodingStats` 使用 `@dataclass` 打包相关状态。典型字段包括 `num_spec_tokens`, `num_drafts`, `num_draft_tokens`, `num_accepted_tokens`, `num_accepted_tokens_per_pos`。

### `SpecDecodingStats.new` method / `SpecDecodingStats.new` 方法
```python
    @classmethod
    def new(cls, num_spec_tokens: int) -> "SpecDecodingStats":
        return cls(
            num_spec_tokens=num_spec_tokens,
            num_accepted_tokens_per_pos=[0] * num_spec_tokens,
        )
```
**EN:** This method implements `new` within `SpecDecodingStats`. Key calls include `cls`.
**CN:** 该方法会实现 `new`，其作用域位于`SpecDecodingStats`。 关键调用包括 `cls`。

### `SpecDecodingStats.observe_draft` method / `SpecDecodingStats.observe_draft` 方法
```python
    def observe_draft(self, num_draft_tokens: int, num_accepted_tokens: int):
        self.num_drafts += 1
        self.num_draft_tokens += num_draft_tokens
        self.num_accepted_tokens += num_accepted_tokens
        assert num_accepted_tokens <= self.num_spec_tokens
        for i in range(num_accepted_tokens):
            self.num_accepted_tokens_per_pos[i] += 1
```
**EN:** This method implements `observe_draft` within `SpecDecodingStats`. Key calls include `range`. It touches state such as `num_drafts`, `num_draft_tokens`, `num_accepted_tokens`, `num_accepted_tokens_per_pos`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `observe_draft`，其作用域位于`SpecDecodingStats`。 关键调用包括 `range`。 它会读写 `num_drafts`, `num_draft_tokens`, `num_accepted_tokens`, `num_accepted_tokens_per_pos` 等状态。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `SpecDecodingLogging` class / `SpecDecodingLogging` 类
```python
class SpecDecodingLogging:
    """Aggregate and log spec decoding metrics.

    LoggingStatLogger aggregates per-iteration metrics over a set
    time interval using observe() and then logs them using log()
    before resetting to zero.
    """
```
**EN:** Introduces the `SpecDecodingLogging` class. Core methods include `__init__`, `reset`, `observe`, `log`. Docstring signal: Aggregate and log spec decoding metrics.
**CN:** 这里定义 `SpecDecodingLogging` 类。核心方法包括 `__init__`, `reset`, `observe`, `log`。

### `SpecDecodingLogging.__init__` method / `SpecDecodingLogging.__init__` 方法
```python
    def __init__(self):
        self.reset()
```
**EN:** This method initializes the object state within `SpecDecodingLogging`. Key calls include `reset`.
**CN:** 该方法会初始化对象状态，其作用域位于`SpecDecodingLogging`。 关键调用包括 `reset`。

### `SpecDecodingLogging.reset` method / `SpecDecodingLogging.reset` 方法
```python
    def reset(self):
        self.num_drafts: list[int] = []
        self.num_draft_tokens: list[int] = []
        self.num_accepted_tokens: list[int] = []
        self.accepted_tokens_per_pos_lists: list[list[int]] = []
        self.last_log_time = time.monotonic()
```
**EN:** This method implements `reset` within `SpecDecodingLogging`. Key calls include `monotonic`. It touches state such as `num_drafts`, `num_draft_tokens`, `num_accepted_tokens`, `accepted_tokens_per_pos_lists`, `last_log_time`.
**CN:** 该方法会实现 `reset`，其作用域位于`SpecDecodingLogging`。 关键调用包括 `monotonic`。 它会读写 `num_drafts`, `num_draft_tokens`, `num_accepted_tokens`, `accepted_tokens_per_pos_lists`, `last_log_time` 等状态。

### `SpecDecodingLogging.observe` method / `SpecDecodingLogging.observe` 方法
```python
    def observe(self, spec_decoding_stats: SpecDecodingStats):
        self.num_drafts.append(spec_decoding_stats.num_drafts)
        self.num_draft_tokens.append(spec_decoding_stats.num_draft_tokens)
        self.num_accepted_tokens.append(spec_decoding_stats.num_accepted_tokens)
        self.accepted_tokens_per_pos_lists.append(
            spec_decoding_stats.num_accepted_tokens_per_pos
        )
```
**EN:** This method implements `observe` within `SpecDecodingLogging`. Key calls include `append`.
**CN:** 该方法会实现 `observe`，其作用域位于`SpecDecodingLogging`。 关键调用包括 `append`。

### `SpecDecodingLogging.log` method / `SpecDecodingLogging.log` 方法
```python
    def log(self, log_fn=logger.info):
        if not self.num_drafts:
            return
        num_drafts = np.sum(self.num_drafts)
        num_draft_tokens = np.sum(self.num_draft_tokens)
        num_accepted_tokens = np.sum(self.num_accepted_tokens)
        draft_throughput = 0
        accepted_throughput = 0

        elapsed_time = time.monotonic() - self.last_log_time
        if elapsed_time > 0:
            draft_throughput = num_draft_tokens / elapsed_time
            accepted_throughput = num_accepted_tokens / elapsed_time

        draft_acceptance_rate = (
            num_accepted_tokens / num_draft_tokens * 100
            if num_draft_tokens > 0
            else float("nan")
        )

        # Conventionally, mean acceptance length includes the bonus token
        mean_acceptance_length = 1 + (num_accepted_tokens / num_drafts)

        pos_matrix = np.array(self.accepted_tokens_per_pos_lists)
        acceptance_rates = np.sum(pos_matrix, axis=0) / num_drafts
        rates_str = ", ".join(f"{p:.3f}" for p in acceptance_rates)

        log_fn(
            "SpecDecoding metrics: "
            "Mean acceptance length: %.2f, "
            "Accepted throughput: %.2f tokens/s, "
            "Drafted throughput: %.2f tokens/s, "
            "Accepted: %d tokens, "
            "Drafted: %d tokens, "
            "Per-position acceptance rate: %s, "
            "Avg Draft acceptance rate: %.1f%%",
            mean_acceptance_length,
            accepted_throughput,
            draft_throughput,
            num_accepted_tokens,
            num_draft_tokens,
            rates_str,
            draft_acceptance_rate,
        )
        self.reset()
```
**EN:** This method implements `log` within `SpecDecodingLogging`. Key calls include `sum`, `array`, `join`, `log_fn`, `reset`, `monotonic`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `log`，其作用域位于`SpecDecodingLogging`。 关键调用包括 `sum`, `array`, `join`, `log_fn`, `reset`, `monotonic`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `SpecDecodingProm` class / `SpecDecodingProm` 类
```python
class SpecDecodingProm:
    """Record spec decoding metrics in Prometheus.

    The acceptance rate can be calculated using a PromQL query:

      rate(vllm:spec_decode_num_accepted_tokens_total[$interval]) /
      rate(vllm:spec_decode_num_draft_tokens_total[$interval])

    The mean acceptance length (conventionally including bonus tokens)
    can be calculated using:

      1 + (
      rate(vllm:spec_decode_num_accepted_tokens_total[$interval]) /
      rate(vllm:spec_decode_num_drafts[$interval]))

    A per-position acceptance rate vector can be computed using

      vllm:spec_decode_num_accepted_tokens_per_pos[$interval] /
      vllm:spec_decode_num_drafts[$interval]
    """

    _counter_cls = prometheus_client.Counter
```
**EN:** Introduces the `SpecDecodingProm` class. Core methods include `__init__`, `observe`. Docstring signal: Record spec decoding metrics in Prometheus.
**CN:** 这里定义 `SpecDecodingProm` 类。核心方法包括 `__init__`, `observe`。

### `SpecDecodingProm.__init__` method / `SpecDecodingProm.__init__` 方法
```python
    def __init__(
        self,
        speculative_config: SpeculativeConfig | None,
        labelnames: list[str],
        per_engine_labelvalues: dict[int, list[object]],
    ):
        self.spec_decoding_enabled = speculative_config is not None
        if not self.spec_decoding_enabled:
            return

        counter_drafts = self._counter_cls(
            name="vllm:spec_decode_num_drafts",
            documentation="Number of spec decoding drafts.",
            labelnames=labelnames,
        )
        self.counter_spec_decode_num_drafts = create_metric_per_engine(
            counter_drafts, per_engine_labelvalues
        )

        counter_draft_tokens = self._counter_cls(
            name="vllm:spec_decode_num_draft_tokens",
            documentation="Number of draft tokens.",
            labelnames=labelnames,
        )
        self.counter_spec_decode_num_draft_tokens = create_metric_per_engine(
            counter_draft_tokens, per_engine_labelvalues
        )

        counter_accepted_tokens = self._counter_cls(
            name="vllm:spec_decode_num_accepted_tokens",
            documentation="Number of accepted tokens.",
            labelnames=labelnames,
        )
        self.counter_spec_decode_num_accepted_tokens = create_metric_per_engine(
            counter_accepted_tokens, per_engine_labelvalues
        )

        assert speculative_config is not None
        num_spec_tokens = (
            speculative_config.num_speculative_tokens
            if self.spec_decoding_enabled
            else 0
        )
        pos_labelnames = labelnames + ["position"]
        base_counter = self._counter_cls(
            name="vllm:spec_decode_num_accepted_tokens_per_pos",
            documentation="Accepted tokens per draft position.",
            labelnames=pos_labelnames,
        )
        self.counter_spec_decode_num_accepted_tokens_per_pos: dict[
            int, list[prometheus_client.Counter]
        ] = {
            idx: [base_counter.labels(*lv, str(pos)) for pos in range(num_spec_tokens)]
            for idx, lv in per_engine_labelvalues.items()
        }
```
**EN:** This method initializes the object state within `SpecDecodingProm`. Key calls include `_counter_cls`, `create_metric_per_engine`, `labels`, `items`, `str`, `range`. It touches state such as `spec_decoding_enabled`, `counter_spec_decode_num_drafts`, `counter_spec_decode_num_draft_tokens`, `counter_spec_decode_num_accepted_tokens`, `counter_spec_decode_num_accepted_tokens_per_pos`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`SpecDecodingProm`。 关键调用包括 `_counter_cls`, `create_metric_per_engine`, `labels`, `items`, `str`, `range`。 它会读写 `spec_decoding_enabled`, `counter_spec_decode_num_drafts`, `counter_spec_decode_num_draft_tokens`, `counter_spec_decode_num_accepted_tokens`, `counter_spec_decode_num_accepted_tokens_per_pos` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `SpecDecodingProm.observe` method / `SpecDecodingProm.observe` 方法
```python
    def observe(self, spec_decoding_stats: SpecDecodingStats, engine_idx: int = 0):
        if not self.spec_decoding_enabled:
            return
        self.counter_spec_decode_num_drafts[engine_idx].inc(
            spec_decoding_stats.num_drafts
        )
        self.counter_spec_decode_num_draft_tokens[engine_idx].inc(
            spec_decoding_stats.num_draft_tokens
        )
        self.counter_spec_decode_num_accepted_tokens[engine_idx].inc(
            spec_decoding_stats.num_accepted_tokens
        )
        for pos, counter in enumerate(
            self.counter_spec_decode_num_accepted_tokens_per_pos[engine_idx]
        ):
            counter.inc(spec_decoding_stats.num_accepted_tokens_per_pos[pos])
```
**EN:** This method implements `observe` within `SpecDecodingProm`. Key calls include `inc`, `enumerate`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `observe`，其作用域位于`SpecDecodingProm`。 关键调用包括 `inc`, `enumerate`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `SpecDecodingStats`: central class or interface in this module. / `SpecDecodingStats`：本模块中的核心类或接口。
- `SpecDecodingLogging`: central class or interface in this module. / `SpecDecodingLogging`：本模块中的核心类或接口。
- `SpecDecodingProm`: central class or interface in this module. / `SpecDecodingProm`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `time`, `dataclasses`
- External / 外部依赖: `numpy`, `prometheus_client`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.logger`, `vllm.v1.metrics.utils`
