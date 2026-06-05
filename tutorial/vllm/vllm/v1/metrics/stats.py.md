# stats.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/metrics/stats.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `BaseCacheStats`, `CachingMetrics`, `PrefixCacheStats` for the V1 `metrics` subsystem. / 为 V1 的 `metrics` 子系统实现 `BaseCacheStats`, `CachingMetrics`, `PrefixCacheStats`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import time
from collections import defaultdict, deque
from dataclasses import dataclass, field
from typing import TYPE_CHECKING, Any

import vllm.envs as envs
from vllm.compilation.cuda_graph import CUDAGraphStat
from vllm.v1.metrics.perf import PerfStats
from vllm.v1.spec_decode.metrics import SpecDecodingStats

if TYPE_CHECKING:
    from vllm.v1.engine import EngineCoreEvent, EngineCoreOutput, FinishReason
```
**EN:** Sets up the module namespace with standard-library helpers, and internal vLLM modules such as `vllm.envs`, `vllm.compilation.cuda_graph`, `vllm.v1.metrics.perf`, `vllm.v1.spec_decode.metrics`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具，以及 `vllm.envs`, `vllm.compilation.cuda_graph`, `vllm.v1.metrics.perf`, `vllm.v1.spec_decode.metrics` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `BaseCacheStats` class / `BaseCacheStats` 类
```python
@dataclass
class BaseCacheStats:
    """Stores cache hit statistics."""

    reset: bool = False
    """Whether the cache was reset."""

    requests: int = 0
    """The number of requests in this update."""

    queries: int = 0
    """The number of queries in these requests."""

    hits: int = 0
    """The number of hits in these requests."""
```
**EN:** Uses `@dataclass` to package related state for `BaseCacheStats`. Typical fields include `reset`, `requests`, `queries`, `hits`.
**CN:** `BaseCacheStats` 使用 `@dataclass` 打包相关状态。典型字段包括 `reset`, `requests`, `queries`, `hits`。

### `CachingMetrics` class / `CachingMetrics` 类
```python
class CachingMetrics:
    """Metrics for caching with a hit rate of the most recent N requests.
    Args:
        interval: The number of the most recent requests to aggregate.
            Defaults to 1000.
    """
```
**EN:** Introduces the `CachingMetrics` class. Core methods include `__init__`, `observe`, `reset`, `empty`, `hit_rate`. Docstring signal: Metrics for caching with a hit rate of the most recent N requests.
**CN:** 这里定义 `CachingMetrics` 类。核心方法包括 `__init__`, `observe`, `reset`, `empty`, `hit_rate`。

### `CachingMetrics.__init__` method / `CachingMetrics.__init__` 方法
```python
    def __init__(self, max_recent_requests: int = 1000) -> None:
        super().__init__()

        self.max_recent_requests = max_recent_requests
        # The current aggregated values.
        self.aggregated_requests = 0
        self.aggregated_query_total = 0
        self.aggregated_query_hit = 0

        # A deque of (requests, queries, hits) for the most recent requests.
        self.query_queue = deque[tuple[int, int, int]]()
```
**EN:** This method initializes the object state within `CachingMetrics`. Key calls include `__init__`, `super`. It touches state such as `max_recent_requests`, `aggregated_requests`, `aggregated_query_total`, `aggregated_query_hit`, `query_queue`.
**CN:** 该方法会初始化对象状态，其作用域位于`CachingMetrics`。 关键调用包括 `__init__`, `super`。 它会读写 `max_recent_requests`, `aggregated_requests`, `aggregated_query_total`, `aggregated_query_hit`, `query_queue` 等状态。

### `CachingMetrics.observe` method / `CachingMetrics.observe` 方法
```python
    def observe(self, stats: BaseCacheStats):
        """Observe the prefix caching for a set of requests.

        This function is called with information gathered when new requests
        are being scheduled and are looking for computed blocks.

        When there are more than `max_recent_requests` requests, the oldest set
        of requests are removed from the metrics.

        Args:
            stats: The prefix cache stats.
        """
        # reset_prefix_cache was invoked before the current update.
        # Reset the metrics before aggregating the current stats.
        if stats.reset:
            self.reset()

        # DO NOT appending empty stats to avoid helpful info get kicked out
        # due to sliding window.
        if stats.requests == 0:
            return

        # Update the metrics.
        self.query_queue.append((stats.requests, stats.queries, stats.hits))
        self.aggregated_requests += stats.requests
        self.aggregated_query_total += stats.queries
        self.aggregated_query_hit += stats.hits

        # Remove the oldest stats until number of requests does not exceed
        # the limit.
        # NOTE: We preserve the latest added stats regardless.
        while (
            len(self.query_queue) > 1
            and self.aggregated_requests > self.max_recent_requests
        ):
            old_requests, old_queries, old_hits = self.query_queue.popleft()
            self.aggregated_requests -= old_requests
            self.aggregated_query_total -= old_queries
            self.aggregated_query_hit -= old_hits
```
**EN:** This method implements `observe` within `CachingMetrics`. The docstring frames it as: Observe the prefix caching for a set of requests. Key calls include `append`, `reset`, `popleft`, `len`. It touches state such as `aggregated_requests`, `aggregated_query_total`, `aggregated_query_hit`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `observe`，其作用域位于`CachingMetrics`。 关键调用包括 `append`, `reset`, `popleft`, `len`。 它会读写 `aggregated_requests`, `aggregated_query_total`, `aggregated_query_hit` 等状态。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `CachingMetrics.reset` method / `CachingMetrics.reset` 方法
```python
    def reset(self):
        """Reset the metrics."""
        self.aggregated_requests = 0
        self.aggregated_query_total = 0
        self.aggregated_query_hit = 0
        self.query_queue.clear()
```
**EN:** This method implements `reset` within `CachingMetrics`. The docstring frames it as: Reset the metrics. Key calls include `clear`. It touches state such as `aggregated_requests`, `aggregated_query_total`, `aggregated_query_hit`.
**CN:** 该方法会实现 `reset`，其作用域位于`CachingMetrics`。 关键调用包括 `clear`。 它会读写 `aggregated_requests`, `aggregated_query_total`, `aggregated_query_hit` 等状态。

### `PrefixCacheStats` class / `PrefixCacheStats` 类
```python
@dataclass
class PrefixCacheStats(BaseCacheStats):
    """
    Stores prefix cache hit statistics.
    - `reset`: Whether `reset_prefix_cache` was invoked.
    - `queries`: Refers to the number of tokens that were queried.
    """

    preempted_requests: int = 0
    """The number of previously preempted requests in this update."""

    preempted_queries: int = 0
    """The `queries` number for preempted requests."""

    preempted_hits: int = 0
    """The `hits` number for preempted requests."""
```
**EN:** Uses `@dataclass` to package related state for `PrefixCacheStats`. Typical fields include `preempted_requests`, `preempted_queries`, `preempted_hits`.
**CN:** `PrefixCacheStats` 使用 `@dataclass` 打包相关状态。典型字段包括 `preempted_requests`, `preempted_queries`, `preempted_hits`。

### `PrefixCacheStats.record` method / `PrefixCacheStats.record` 方法
```python
    def record(self, num_tokens: int, num_hits: int, preempted: bool) -> None:
        """Aggregate request information into the stats."""
        if preempted:
            # Previously preempted request
            self.preempted_requests += 1
            self.preempted_queries += num_tokens
            self.preempted_hits += num_hits
        else:
            # New request
            self.requests += 1
            self.queries += num_tokens
            self.hits += num_hits
```
**EN:** This method implements `record` within `PrefixCacheStats`. The docstring frames it as: Aggregate request information into the stats. It touches state such as `preempted_requests`, `preempted_queries`, `preempted_hits`, `requests`, `queries`, `hits`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `record`，其作用域位于`PrefixCacheStats`。 它会读写 `preempted_requests`, `preempted_queries`, `preempted_hits`, `requests`, `queries`, `hits` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `MultiModalCacheStats` class / `MultiModalCacheStats` 类
```python
@dataclass
class MultiModalCacheStats(BaseCacheStats):
    """
    Stores multi-modal cache hit statistics.
    - `reset`: Whether `reset_mm_cache` was invoked.
    - `queries`: Refers to the number of multi-modal data items
      that were queried.
    """
```
**EN:** Uses `@dataclass` to package related state for `MultiModalCacheStats`. Typical fields include configuration and runtime data.
**CN:** `MultiModalCacheStats` 使用 `@dataclass` 打包相关状态。典型字段包括 配置与运行期数据。

### `MultiModalCacheStats.record` method / `MultiModalCacheStats.record` 方法
```python
    def record(self, num_queries: int, num_hits: int) -> None:
        """Aggregate request information into the stats."""
        self.requests += 1
        self.queries += num_queries
        self.hits += num_hits
```
**EN:** This method implements `record` within `MultiModalCacheStats`. The docstring frames it as: Aggregate request information into the stats. It touches state such as `requests`, `queries`, `hits`.
**CN:** 该方法会实现 `record`，其作用域位于`MultiModalCacheStats`。 它会读写 `requests`, `queries`, `hits` 等状态。

### `KVCacheEvictionEvent` class / `KVCacheEvictionEvent` 类
```python
@dataclass
class KVCacheEvictionEvent:
    """Single KV cache block eviction sample."""

    lifetime_seconds: float
    idle_seconds: float
    reuse_gaps_seconds: tuple[float, ...]
```
**EN:** Uses `@dataclass` to package related state for `KVCacheEvictionEvent`. Typical fields include `lifetime_seconds`, `idle_seconds`, `reuse_gaps_seconds`.
**CN:** `KVCacheEvictionEvent` 使用 `@dataclass` 打包相关状态。典型字段包括 `lifetime_seconds`, `idle_seconds`, `reuse_gaps_seconds`。

### `SchedulerStats` class / `SchedulerStats` 类
```python
@dataclass
class SchedulerStats:
    """Stats associated with the scheduler."""

    num_running_reqs: int = 0

    num_waiting_reqs: int = 0  # length of the "waiting" request queue
    num_skipped_waiting_reqs: int = 0  # length of the "skipped waiting" queue

    # These are used for internal DP load-balancing.
    step_counter: int = 0
    current_wave: int = 0

    kv_cache_usage: float = 0.0

    prefix_cache_stats: PrefixCacheStats = field(default_factory=PrefixCacheStats)
    connector_prefix_cache_stats: PrefixCacheStats | None = None

    kv_cache_eviction_events: list[KVCacheEvictionEvent] = field(default_factory=list)

    spec_decoding_stats: SpecDecodingStats | None = None
    kv_connector_stats: dict[str, Any] | None = None

    waiting_lora_adapters: dict[str, int] = field(default_factory=dict)
    running_lora_adapters: dict[str, int] = field(default_factory=dict)

    cudagraph_stats: CUDAGraphStat | None = None

    perf_stats: PerfStats | None = None
```
**EN:** Uses `@dataclass` to package related state for `SchedulerStats`. Typical fields include `num_running_reqs`, `num_waiting_reqs`, `num_skipped_waiting_reqs`, `step_counter`, `current_wave`, `kv_cache_usage`.
**CN:** `SchedulerStats` 使用 `@dataclass` 打包相关状态。典型字段包括 `num_running_reqs`, `num_waiting_reqs`, `num_skipped_waiting_reqs`, `step_counter`, `current_wave`, `kv_cache_usage`。

### `RequestStateStats` class / `RequestStateStats` 类
```python
@dataclass
class RequestStateStats:
    """Stats that need to be tracked across delta updates."""

    num_generation_tokens: int = 0

    # This is an engine frontend timestamp (wall-clock)
    arrival_time: float = 0.0

    # These are engine core timestamps (monotonic)
    queued_ts: float = 0.0
    scheduled_ts: float = 0.0
    first_token_ts: float = 0.0
    last_token_ts: float = 0.0

    # first token latency
    first_token_latency: float = 0.0

    # Track if this request is corrupted (NaNs in logits)
    is_corrupted: bool = False
```
**EN:** Uses `@dataclass` to package related state for `RequestStateStats`. Typical fields include `num_generation_tokens`, `arrival_time`, `queued_ts`, `scheduled_ts`, `first_token_ts`, `last_token_ts`.
**CN:** `RequestStateStats` 使用 `@dataclass` 打包相关状态。典型字段包括 `num_generation_tokens`, `arrival_time`, `queued_ts`, `scheduled_ts`, `first_token_ts`, `last_token_ts`。

### `FinishedRequestStats` class / `FinishedRequestStats` 类
```python
@dataclass
class FinishedRequestStats:
    """Stats associated with a finished request."""

    finish_reason: "FinishReason"
    request_id: str | None = None
    e2e_latency: float = 0.0
    num_prompt_tokens: int = 0
    num_generation_tokens: int = 0
    max_tokens_param: int | None = None
    queued_time: float = 0.0
    prefill_time: float = 0.0
    inference_time: float = 0.0
    decode_time: float = 0.0
    mean_time_per_output_token: float = 0.0
    is_corrupted: bool = False
    num_cached_tokens: int = 0
```
**EN:** Uses `@dataclass` to package related state for `FinishedRequestStats`. Typical fields include `finish_reason`, `request_id`, `e2e_latency`, `num_prompt_tokens`, `num_generation_tokens`, `max_tokens_param`.
**CN:** `FinishedRequestStats` 使用 `@dataclass` 打包相关状态。典型字段包括 `finish_reason`, `request_id`, `e2e_latency`, `num_prompt_tokens`, `num_generation_tokens`, `max_tokens_param`。

### `PrefillStats` class / `PrefillStats` 类
```python
@dataclass
class PrefillStats:
    """Breakdown of a scheduled prefill computation.

    Fields:
        num_prompt_tokens: Total number of tokens to be prefilled.
        num_computed_tokens: Tokens to be prefilled locally (actual compute work).
        num_cached_tokens: Tokens to be prefilled without actual compute work.
        num_local_cached_tokens: Tokens to be prefilled from local prefix cache.
        num_external_cached_tokens: Tokens to be prefilled from external KV transfer.
    """

    num_prompt_tokens: int = 0
    num_computed_tokens: int = 0
    num_cached_tokens: int = 0
    num_local_cached_tokens: int = 0
    num_external_cached_tokens: int = 0
```
**EN:** Uses `@dataclass` to package related state for `PrefillStats`. Typical fields include `num_prompt_tokens`, `num_computed_tokens`, `num_cached_tokens`, `num_local_cached_tokens`, `num_external_cached_tokens`.
**CN:** `PrefillStats` 使用 `@dataclass` 打包相关状态。典型字段包括 `num_prompt_tokens`, `num_computed_tokens`, `num_cached_tokens`, `num_local_cached_tokens`, `num_external_cached_tokens`。

### `PrefillStats.set` method / `PrefillStats.set` 方法
```python
    def set(
        self,
        num_prompt_tokens: int,
        num_local_cached_tokens: int,
        num_external_cached_tokens: int,
    ):
        num_cached_tokens = num_local_cached_tokens + num_external_cached_tokens
        assert num_cached_tokens <= num_prompt_tokens

        self.num_prompt_tokens = num_prompt_tokens
        self.num_computed_tokens = num_prompt_tokens - num_cached_tokens
        self.num_cached_tokens = num_cached_tokens
        self.num_local_cached_tokens = num_local_cached_tokens
        self.num_external_cached_tokens = num_external_cached_tokens
```
**EN:** This method implements `set` within `PrefillStats`. It touches state such as `num_prompt_tokens`, `num_computed_tokens`, `num_cached_tokens`, `num_local_cached_tokens`, `num_external_cached_tokens`.
**CN:** 该方法会实现 `set`，其作用域位于`PrefillStats`。 它会读写 `num_prompt_tokens`, `num_computed_tokens`, `num_cached_tokens`, `num_local_cached_tokens`, `num_external_cached_tokens` 等状态。

### `PromptTokenStats` class / `PromptTokenStats` 类
```python
@dataclass
class PromptTokenStats:
    """Breakdown of prompt tokens by source.

    Fields:
        computed: Tokens prefilled locally (actual compute work).
        local_cache_hit: Tokens from local prefix cache.
        external_kv_transfer: Tokens from external KV transfer.
        cached_tokens: Tokens skipped during prefill (from scheduler).
        total: Total prompt tokens.

    Invariants:
        computed + local_cache_hit + external_kv_transfer = total
        local_cache_hit + external_kv_transfer = cached_tokens
    """

    ALL_SOURCES: tuple[str, ...] = (
        "local_compute",
        "local_cache_hit",
        "external_kv_transfer",
    )

    computed: int = 0
    local_cache_hit: int = 0
    external_kv_transfer: int = 0
    cached_tokens: int = 0
    total: int = 0
```
**EN:** Uses `@dataclass` to package related state for `PromptTokenStats`. Typical fields include `ALL_SOURCES`, `computed`, `local_cache_hit`, `external_kv_transfer`, `cached_tokens`, `total`.
**CN:** `PromptTokenStats` 使用 `@dataclass` 打包相关状态。典型字段包括 `ALL_SOURCES`, `computed`, `local_cache_hit`, `external_kv_transfer`, `cached_tokens`, `total`。

### `PromptTokenStats.update_from_output` method / `PromptTokenStats.update_from_output` 方法
```python
    def update_from_output(self, prefill_stats: PrefillStats) -> None:
        """Update stats from a prefill output."""
        self.computed += prefill_stats.num_computed_tokens
        self.cached_tokens += prefill_stats.num_cached_tokens
        self.total += prefill_stats.num_prompt_tokens

        self.local_cache_hit += prefill_stats.num_local_cached_tokens
        self.external_kv_transfer += prefill_stats.num_external_cached_tokens
```
**EN:** This method updates existing state within `PromptTokenStats`. The docstring frames it as: Update stats from a prefill output. It touches state such as `computed`, `cached_tokens`, `total`, `local_cache_hit`, `external_kv_transfer`.
**CN:** 该方法会更新现有状态，其作用域位于`PromptTokenStats`。 它会读写 `computed`, `cached_tokens`, `total`, `local_cache_hit`, `external_kv_transfer` 等状态。

### `PromptTokenStats.get_by_source` method / `PromptTokenStats.get_by_source` 方法
```python
    def get_by_source(self, source: str) -> int:
        """Get token count by source label."""
        source_map = {
            "local_compute": self.computed,
            "local_cache_hit": self.local_cache_hit,
            "external_kv_transfer": self.external_kv_transfer,
        }
        if source not in source_map:
            raise ValueError(f"Unknown source: {source}")
        return source_map[source]
```
**EN:** This method returns or derives a value within `PromptTokenStats`. The docstring frames it as: Get token count by source label. Key calls include `ValueError`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`PromptTokenStats`。 关键调用包括 `ValueError`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `IterationStats` class / `IterationStats` 类
```python
class IterationStats:
    """Stats associated with a single set of EngineCoreOutputs."""
```
**EN:** Introduces the `IterationStats` class. Core methods include `__init__`, `__repr__`, `num_prompt_tokens`, `_time_since`, `update_from_output`, `update_from_events`. Docstring signal: Stats associated with a single set of EngineCoreOutputs.
**CN:** 这里定义 `IterationStats` 类。核心方法包括 `__init__`, `__repr__`, `num_prompt_tokens`, `_time_since`, `update_from_output`, `update_from_events`。

### `IterationStats.update_from_output` method / `IterationStats.update_from_output` 方法
```python
    def update_from_output(
        self,
        output: "EngineCoreOutput",
        engine_core_timestamp: float,
        is_prefilling: bool,
        req_stats: RequestStateStats,
        lora_states: "LoRARequestStates",
        lora_name: str | None,
    ):
        num_new_generation_tokens = len(output.new_token_ids)

        self.num_generation_tokens += num_new_generation_tokens
        if is_prefilling:
            if output.prefill_stats is not None:
                self.prompt_token_stats.update_from_output(output.prefill_stats)

            first_token_latency = self._time_since(req_stats.arrival_time)
            self.time_to_first_tokens_iter.append(first_token_latency)
            req_stats.first_token_latency = first_token_latency

        req_stats.num_generation_tokens += num_new_generation_tokens

        # Track if this request is corrupted (only check once per request)
        # Early exit if already marked as corrupted to avoid redundant checks
        if (
            envs.VLLM_COMPUTE_NANS_IN_LOGITS
            and not req_stats.is_corrupted
            and output.num_nans_in_logits > 0
        ):
            req_stats.is_corrupted = True

        # Process request-level engine core events
        if output.events is not None:
            self.update_from_events(
                output.request_id,
                output.events,
                is_prefilling,
                req_stats,
                lora_states,
                lora_name,
            )

        # Process the batch-level "new tokens" engine core event
        if is_prefilling:
            req_stats.first_token_ts = engine_core_timestamp
        else:
            itl = engine_core_timestamp - req_stats.last_token_ts
            self.inter_token_latencies_iter.append(itl)

        req_stats.last_token_ts = engine_core_timestamp
```
**EN:** This method updates existing state within `IterationStats`. Key calls include `len`, `_time_since`, `append`, `update_from_events`, `update_from_output`. It touches state such as `num_generation_tokens`. The control flow contains 5 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会更新现有状态，其作用域位于`IterationStats`。 关键调用包括 `len`, `_time_since`, `append`, `update_from_events`, `update_from_output`。 它会读写 `num_generation_tokens` 等状态。 控制流包含 5 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `IterationStats.update_from_events` method / `IterationStats.update_from_events` 方法
```python
    def update_from_events(
        self,
        req_id: str,
        events: list["EngineCoreEvent"],
        is_prefilling: bool,
        req_stats: RequestStateStats,
        lora_states: "LoRARequestStates",
        lora_name: str | None,
    ):
        # Avoid circular dependency
        from vllm.v1.engine import EngineCoreEventType

        for event in events:
            if event.type == EngineCoreEventType.QUEUED:
                req_stats.queued_ts = event.timestamp
                lora_states.request_waiting(req_id, lora_name)
            elif event.type == EngineCoreEventType.SCHEDULED:
                if req_stats.scheduled_ts == 0.0:  # ignore preemptions
                    req_stats.scheduled_ts = event.timestamp
                lora_states.request_running(req_id, lora_name)
            elif event.type == EngineCoreEventType.PREEMPTED:
                self.num_preempted_reqs += 1
                lora_states.request_waiting(req_id, lora_name)
```
**EN:** This method updates existing state within `IterationStats`. Key calls include `request_waiting`, `request_running`. It touches state such as `num_preempted_reqs`. The control flow contains 4 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会更新现有状态，其作用域位于`IterationStats`。 关键调用包括 `request_waiting`, `request_running`。 它会读写 `num_preempted_reqs` 等状态。 控制流包含 4 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `IterationStats.update_from_finished_request` method / `IterationStats.update_from_finished_request` 方法
```python
    def update_from_finished_request(
        self,
        finish_reason: "FinishReason",
        request_id: str,
        num_prompt_tokens: int,
        max_tokens_param: int | None,
        req_stats: RequestStateStats,
        num_cached_tokens: int = 0,
    ):
        e2e_latency = self._time_since(req_stats.arrival_time)

        # Queued interval is from first QUEUED event to first SCHEDULED
        queued_time = req_stats.scheduled_ts - req_stats.queued_ts

        # Prefill interval is from first SCHEDULED to first NEW_TOKEN
        # Any preemptions during prefill is included in the interval
        prefill_time = req_stats.first_token_ts - req_stats.scheduled_ts

        # Decode interval is from first NEW_TOKEN to last NEW_TOKEN
        # Any preemptions during decode are included
        decode_time = req_stats.last_token_ts - req_stats.first_token_ts

        # Inference interval is from first SCHEDULED to last NEW_TOKEN
        # Any preemptions during prefill or decode are included
        inference_time = req_stats.last_token_ts - req_stats.scheduled_ts

        # Do not count the token generated by the prefill phase
        mean_time_per_output_token = (
            decode_time / (req_stats.num_generation_tokens - 1)
            if req_stats.num_generation_tokens - 1 > 0
            else 0
        )

        finished_req = FinishedRequestStats(
            finish_reason=finish_reason,
            request_id=request_id,
            e2e_latency=e2e_latency,
            num_prompt_tokens=num_prompt_tokens,
            num_generation_tokens=req_stats.num_generation_tokens,
            max_tokens_param=max_tokens_param,
            queued_time=queued_time,
            prefill_time=prefill_time,
            inference_time=inference_time,
            decode_time=decode_time,
            mean_time_per_output_token=mean_time_per_output_token,
            is_corrupted=req_stats.is_corrupted,
            num_cached_tokens=num_cached_tokens,
        )
        self.finished_requests.append(finished_req)

        # Count corrupted requests when they finish (only once per request)
        if req_stats.is_corrupted:
            self.num_corrupted_reqs += 1
```
**EN:** This method updates existing state within `IterationStats`. Key calls include `_time_since`, `FinishedRequestStats`, `append`. It touches state such as `num_corrupted_reqs`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会更新现有状态，其作用域位于`IterationStats`。 关键调用包括 `_time_since`, `FinishedRequestStats`, `append`。 它会读写 `num_corrupted_reqs` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `LoRAStats` class / `LoRAStats` 类
```python
class LoRAStats:
    """Tracks waiting and running request IDs for a single LoRA."""
```
**EN:** Introduces the `LoRAStats` class. Core methods include `__init__`, `update`, `empty`. Docstring signal: Tracks waiting and running request IDs for a single LoRA.
**CN:** 这里定义 `LoRAStats` 类。核心方法包括 `__init__`, `update`, `empty`。

### `LoRAStats.__init__` method / `LoRAStats.__init__` 方法
```python
    def __init__(self):
        self.waiting: set[str] = set()
        self.running: set[str] = set()
```
**EN:** This method initializes the object state within `LoRAStats`. Key calls include `set`. It touches state such as `waiting`, `running`.
**CN:** 该方法会初始化对象状态，其作用域位于`LoRAStats`。 关键调用包括 `set`。 它会读写 `waiting`, `running` 等状态。

### `LoRAStats.update` method / `LoRAStats.update` 方法
```python
    def update(self, req_id: str, waiting: bool, running: bool):
        assert not (waiting and running)
        if waiting:
            self.waiting.add(req_id)
        else:
            self.waiting.discard(req_id)

        if running:
            self.running.add(req_id)
        else:
            self.running.discard(req_id)
```
**EN:** This method updates existing state within `LoRAStats`. Key calls include `add`, `discard`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会更新现有状态，其作用域位于`LoRAStats`。 关键调用包括 `add`, `discard`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `LoRAStats.empty` method / `LoRAStats.empty` 方法
```python
    @property
    def empty(self) -> bool:
        return not (self.waiting or self.running)
```
**EN:** This method implements `empty` within `LoRAStats`.
**CN:** 该方法会实现 `empty`，其作用域位于`LoRAStats`。

### `LoRARequestStates` class / `LoRARequestStates` 类
```python
class LoRARequestStates:
    """A per-LoRA count of running and waiting requests."""
```
**EN:** Introduces the `LoRARequestStates` class. Core methods include `__init__`, `_request_update`, `request_waiting`, `request_running`, `request_finished`, `update_scheduler_stats`. Docstring signal: A per-LoRA count of running and waiting requests.
**CN:** 这里定义 `LoRARequestStates` 类。核心方法包括 `__init__`, `_request_update`, `request_waiting`, `request_running`, `request_finished`, `update_scheduler_stats`。

### `LoRARequestStates.__init__` method / `LoRARequestStates.__init__` 方法
```python
    def __init__(self, log_stats: bool = False):
        self.log_stats = log_stats
        self.requests: defaultdict[str, LoRAStats] = defaultdict(LoRAStats)
```
**EN:** This method initializes the object state within `LoRARequestStates`. Key calls include `defaultdict`. It touches state such as `log_stats`, `requests`.
**CN:** 该方法会初始化对象状态，其作用域位于`LoRARequestStates`。 关键调用包括 `defaultdict`。 它会读写 `log_stats`, `requests` 等状态。

### `LoRARequestStates.request_running` method / `LoRARequestStates.request_running` 方法
```python
    def request_running(self, req_id: str, lora_name: str | None):
        self._request_update(req_id, lora_name, waiting=False, running=True)
```
**EN:** This method implements `request_running` within `LoRARequestStates`. Key calls include `_request_update`.
**CN:** 该方法会实现 `request_running`，其作用域位于`LoRARequestStates`。 关键调用包括 `_request_update`。

### `LoRARequestStates.update_scheduler_stats` method / `LoRARequestStates.update_scheduler_stats` 方法
```python
    def update_scheduler_stats(self, scheduler_stats: SchedulerStats | None):
        if not self.log_stats or scheduler_stats is None:
            return
        for lora_name, stats in self.requests.items():
            scheduler_stats.waiting_lora_adapters[lora_name] = len(stats.waiting)
            scheduler_stats.running_lora_adapters[lora_name] = len(stats.running)
```
**EN:** This method updates existing state within `LoRARequestStates`. Key calls include `items`, `len`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会更新现有状态，其作用域位于`LoRARequestStates`。 关键调用包括 `items`, `len`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `BaseCacheStats`: central class or interface in this module. / `BaseCacheStats`：本模块中的核心类或接口。
- `CachingMetrics`: central class or interface in this module. / `CachingMetrics`：本模块中的核心类或接口。
- `PrefixCacheStats`: central class or interface in this module. / `PrefixCacheStats`：本模块中的核心类或接口。
- `MultiModalCacheStats`: central class or interface in this module. / `MultiModalCacheStats`：本模块中的核心类或接口。
- `KVCacheEvictionEvent`: central class or interface in this module. / `KVCacheEvictionEvent`：本模块中的核心类或接口。
- `SchedulerStats`: central class or interface in this module. / `SchedulerStats`：本模块中的核心类或接口。
- `RequestStateStats`: central class or interface in this module. / `RequestStateStats`：本模块中的核心类或接口。
- `FinishedRequestStats`: central class or interface in this module. / `FinishedRequestStats`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `time`, `collections`, `dataclasses`, `typing`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.compilation.cuda_graph`, `vllm.v1.metrics.perf`, `vllm.v1.spec_decode.metrics`, `vllm.v1.engine`
