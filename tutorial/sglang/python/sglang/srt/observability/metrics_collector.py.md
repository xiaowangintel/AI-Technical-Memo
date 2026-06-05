# metrics_collector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/observability/metrics_collector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the metrics, tracing, and monitoring part of the SRT runtime and implements logic centered on `metrics_collector`. The module docstring frames it as: "Utilities for Prometheus Metrics Collection." / 该模块属于 SRT 运行时的指标、追踪与监控部分，主要实现围绕 `metrics_collector` 的逻辑。 它对外提供的主要入口包括 `QueueCount`, `SchedulerStats`, `compute_routing_key_stats`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-43: Module imports, constants, and setup
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
"""Utilities for Prometheus Metrics Collection."""

from __future__ import annotations

import dataclasses
import logging
import os
import time
from collections import Counter
from dataclasses import dataclass, field
from typing import TYPE_CHECKING, Any, Dict, List, Optional, Set, Union

from sglang.srt.disaggregation.utils import DisaggregationMode
from sglang.srt.environ import envs
from sglang.srt.model_executor.forward_batch_info import ForwardMode
from sglang.srt.observability.utils import exponential_buckets, generate_buckets
from sglang.srt.server_args import ServerArgs
from sglang.srt.utils import get_bool_env_var
from sglang.srt.utils.gauge_histogram import GaugeHistogram

if TYPE_CHECKING:
    from prometheus_client import Gauge

    from sglang.srt.managers.schedule_batch import Req

SGLANG_TEST_REQUEST_TIME_STATS = get_bool_env_var("SGLANG_TEST_REQUEST_TIME_STATS")

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; reads environment-driven configuration.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；读取环境变量驱动的配置。

### Lines 44-50: Class QueueCount
```python
@dataclass
class QueueCount:
    """Holds both the total count and optional per-priority breakdown for a queue."""

    total: int = 0
    by_priority: Optional[Dict[int, int]] = None

```
**EN:** This range introduces `QueueCount` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Holds both the total count and optional per-priority breakdown for a queue."
**CN:** 这一段引入 `QueueCount`，并定义其后续方法依赖的结构或元数据。

### Lines 51-61: Method QueueCount.from_reqs
```python
    @classmethod
    def from_reqs(cls, reqs: List[Req], enable_priority_scheduling: bool = False):
        # NOTE: If requests have priority=None (no --default-priority-value set),
        # Counter will produce {None: N}, resulting in priority="None" Prometheus labels.
        # Set --default-priority-value when enabling priority scheduling to avoid this.
        by_priority = (
            dict(Counter(req.priority for req in reqs))
            if enable_priority_scheduling
            else None
        )
        return cls(total=len(reqs), by_priority=by_priority)
```
**EN:** This callable implements `QueueCount.from_reqs`. It takes `cls`, `reqs`, `enable_priority_scheduling` and mainly constructs data from an external representation.
**CN:** 这一可调用对象实现了 `QueueCount.from_reqs`。它接收 `cls`, `reqs`, `enable_priority_scheduling`，主要用于从外部表示构造数据。

### Lines 64-123: Class SchedulerStats (part 1/2)
```python
@dataclass
class SchedulerStats:
    # Basics
    num_running_reqs: QueueCount = field(default_factory=QueueCount)
    num_queue_reqs: QueueCount = field(default_factory=QueueCount)
    num_grammar_queue_reqs: int = 0
    gen_throughput: float = 0.0
    cache_hit_rate: float = 0.0
    decode_sum_seq_lens: int = 0

    # Memory pool usage ratios (0.0–1.0).
    # Each pool tracks: used = total - available - evictable, usage = used / total.
    #
    # token_usage:      max(full, swa, mamba) — the bottleneck across all pools.
    #                   FIXME: misleadingly named "token_usage"; rename requires API deprecation.
    # full_token_usage: full-attention KV cache pool usage (always active).
    # swa_token_usage:  sliding-window attention KV cache pool usage (hybrid SWA models only, e.g. Gemma2).
    # mamba_usage:      Mamba SSM state pool usage (hybrid SSM models only, e.g. Jamba).
    token_usage: float = 0.0
    full_token_usage: float = 0.0
    swa_token_usage: float = 0.0
    mamba_usage: float = 0.0

    # Absolute token counts for the full-attention KV cache pool.
    # Invariant: kv_available_tokens + kv_evictable_tokens + kv_used_tokens <= max_total_num_tokens
    # (the gap accounts for protected/session-held tokens not exposed here).
    # max_total_num_tokens is emitted once at startup via emit_constants.
    #
    # kv_available_tokens:  free (unallocated) slots in the pool.
    # kv_evictable_tokens:  slots holding radix-cached KV data that can be evicted for new requests.
    # kv_used_tokens:       actively used slots (locked by running requests). Equals full_num_used.
    # num_used_tokens:      max(full_num_used, swa_num_used) for hybrid-SWA models, else full_num_used.
    #                       Does NOT include the mamba pool.
    num_used_tokens: int = 0
    kv_available_tokens: int = 0
    kv_evictable_tokens: int = 0
    kv_used_tokens: int = 0

    swa_available_tokens: int = 0
    swa_evictable_tokens: int = 0
    swa_used_tokens: int = 0
    mamba_available_tokens: int = 0
    mamba_evictable_tokens: int = 0
    mamba_used_tokens: int = 0

    # Speculative decoding
    spec_accept_length: float = 0.0
    spec_accept_rate: float = 0.0

    # Retract
    num_retracted_reqs: int = 0
    num_paused_reqs: int = 0

    # PD disaggregation
    num_prefill_bootstrap_queue_reqs: QueueCount = field(default_factory=QueueCount)
    num_prefill_inflight_queue_reqs: QueueCount = field(default_factory=QueueCount)
    num_decode_prealloc_queue_reqs: QueueCount = field(default_factory=QueueCount)
    num_decode_transfer_queue_reqs: QueueCount = field(default_factory=QueueCount)
    kv_transfer_speed_gb_s: float = 0.0
    kv_transfer_latency_ms: float = 0.0
```
**EN:** This range introduces `SchedulerStats` and defines the structure or metadata that its methods rely on. This chunk is part 1 of 2 for the same logical block. In this range it handles grammar or regular-expression constraints.
**CN:** 这一段引入 `SchedulerStats`，并定义其后续方法依赖的结构或元数据。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会处理语法或正则约束。

### Lines 124-152: Class SchedulerStats (part 2/2)
```python
    pending_prealloc_token_usage: float = 0.0

    # Utilization
    utilization: float = 0.0
    fwd_occupancy: float = float("nan")

    # Scheduler policy
    new_token_ratio: float = 0.0

    # CUDA graph
    is_cuda_graph: int = 0

    # LoRA pool metrics
    lora_pool_slots_used: int = 0
    lora_pool_slots_total: int = 0
    lora_pool_utilization: float = 0.0

    # HiCache metrics
    hicache_host_used_tokens: int = 0
    hicache_host_total_tokens: int = 0

    # Streaming session metrics
    num_streaming_sessions: int = 0
    streaming_session_held_tokens: int = 0

    # Routing key metrics
    num_unique_running_routing_keys: int = 0
    routing_key_running_req_counts: List[int] = field(default_factory=list)
    routing_key_all_req_counts: List[int] = field(default_factory=list)
```
**EN:** This range introduces `SchedulerStats` and defines the structure or metadata that its methods rely on. This chunk is part 2 of 2 for the same logical block. In this range it records metrics or tracing signals; manages graph capture or replay logic.
**CN:** 这一段引入 `SchedulerStats`，并定义其后续方法依赖的结构或元数据。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会记录指标或追踪信号；管理图捕获或回放逻辑。

### Lines 153-157: Module-level constants and helpers
```python


ROUTING_KEY_REQ_COUNT_BUCKET_BOUNDS = [1, 2, 3, 5, 7, 10, 20, 50, 100, 200]


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 158-163: Function compute_routing_key_stats
```python
def compute_routing_key_stats(routing_keys: List[Optional[str]]) -> tuple:
    """Returns (num_unique_keys, per_key_counts)."""
    from collections import Counter

    key_counts = Counter(k for k in routing_keys if k is not None)
    return len(key_counts), list(key_counts.values())
```
**EN:** This callable implements `compute_routing_key_stats`. It takes `routing_keys` and mainly implements compute routing key stats. The docstring states: "Returns (num_unique_keys, per_key_counts)." In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `compute_routing_key_stats`。它接收 `routing_keys`，主要用于实现 compute routing key stats 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 166-171: Class DPCooperationInfo
```python
@dataclass
class DPCooperationInfo:
    # Users can derive that, except for cases with idle, num_decode_ranks=world_size-num_prefill_ranks
    # We do not provide `num_decode_ranks` to avoid cardinality explosion.
    num_prefill_ranks: int

```
**EN:** This range introduces `DPCooperationInfo` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `DPCooperationInfo`，并定义其后续方法依赖的结构或元数据。

### Lines 172-180: Method DPCooperationInfo.create
```python
    @staticmethod
    def create(forward_modes: List[int]):
        return DPCooperationInfo(
            # Count ranks that are doing any extend-like work.
            # With overlap scheduling, prefill can appear as MIXED rather than EXTEND.
            num_prefill_ranks=sum(
                1 for mode in forward_modes if ForwardMode(mode).is_extend()
            ),
        )
```
**EN:** This callable implements `DPCooperationInfo.create`. It takes `forward_modes` and mainly constructs new objects or contexts.
**CN:** 这一可调用对象实现了 `DPCooperationInfo.create`。它接收 `forward_modes`，主要用于构造新的对象或上下文。

### Lines 182-183: Method DPCooperationInfo.to_labels
```python
    def to_labels(self):
        return dataclasses.asdict(self)
```
**EN:** This callable implements `DPCooperationInfo.to_labels` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `DPCooperationInfo.to_labels`，主要用于将数据转换为另一种表示。

### Lines 186-192: Class SchedulerMetricsCollectorContext
```python
@dataclass(kw_only=True, frozen=True, slots=True)
class SchedulerMetricsCollectorContext:
    enable_metrics: bool
    is_stats_logging_rank: bool
    current_scheduler_metrics_enabled: bool
    enable_kv_cache_events: bool
    collector: Optional["SchedulerMetricsCollector"]
```
**EN:** This range introduces `SchedulerMetricsCollectorContext` and defines the structure or metadata that its methods rely on. In this range it records metrics or tracing signals.
**CN:** 这一段引入 `SchedulerMetricsCollectorContext`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 195-196: Class SchedulerMetricsCollector
```python
class SchedulerMetricsCollector:

```
**EN:** This range introduces `SchedulerMetricsCollector` and defines the structure or metadata that its methods rely on. In this range it records metrics or tracing signals.
**CN:** 这一段引入 `SchedulerMetricsCollector`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 197-256: Method SchedulerMetricsCollector.__init__ (part 1/13)
```python
    def __init__(
        self,
        labels: Dict[str, str],
        enable_lora: bool = False,
        enable_hierarchical_cache: bool = False,
        enable_streaming_session: bool = False,
        server_args: Optional["ServerArgs"] = None,
    ) -> None:
        # We need to import prometheus_client after setting the env variable `PROMETHEUS_MULTIPROC_DIR`
        from prometheus_client import Counter, Gauge, Histogram, Summary

        self.labels = labels
        self.enable_lora = enable_lora
        self.enable_hierarchical_cache = enable_hierarchical_cache
        self.enable_streaming_session = enable_streaming_session
        self.last_log_time = time.perf_counter()
        self._known_priorities: Set[int] = set()

        # =================================================================
        # Basics
        # =================================================================
        self.num_running_reqs = Gauge(
            name="sglang:num_running_reqs",
            documentation="The number of running requests.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.num_queue_reqs = Gauge(
            name="sglang:num_queue_reqs",
            documentation="The number of requests in the waiting queue.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.num_grammar_queue_reqs = Gauge(
            name="sglang:num_grammar_queue_reqs",
            documentation="The number of requests in the grammar waiting queue.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.gen_throughput = Gauge(
            name="sglang:gen_throughput",
            documentation="The generation throughput (token/s).",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.cache_hit_rate = Gauge(
            name="sglang:cache_hit_rate",
            documentation="The prefix cache hit rate.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.decode_sum_seq_lens = Gauge(
            name="sglang:decode_sum_seq_lens",
            documentation="The sum of all sequence lengths in decode.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )

        # =================================================================
        # Memory pool usage ratios
```
**EN:** This callable implements `SchedulerMetricsCollector.__init__`. It takes `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session` and mainly initializes instance state and defaults. This chunk is part 1 of 13 for the same logical block. In this range it sets up imports and shared symbols; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.__init__`。它接收 `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 1/13 部分。 在这一范围内，它会建立导入关系并准备共享符号；处理语法或正则约束。

### Lines 257-316: Method SchedulerMetricsCollector.__init__ (part 2/13)
```python
        # =================================================================
        self.token_usage = Gauge(
            name="sglang:token_usage",
            documentation="The token usage.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.full_token_usage = Gauge(
            name="sglang:full_token_usage",
            documentation="The token usage for full attention layers.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.swa_token_usage = Gauge(
            name="sglang:swa_token_usage",
            documentation="The token usage for SWA layers.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.mamba_usage = Gauge(
            name="sglang:mamba_usage",
            documentation="The token usage for Mamba layers.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )

        # =================================================================
        # Absolute token counts
        # =================================================================
        self.num_used_tokens = Gauge(
            name="sglang:num_used_tokens",
            documentation="The number of used tokens.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.kv_available_tokens = Gauge(
            name="sglang:kv_available_tokens",
            documentation="Number of free token slots in the KV cache pool.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.kv_evictable_tokens = Gauge(
            name="sglang:kv_evictable_tokens",
            documentation="Number of evictable (radix-cached) token slots in the KV cache pool.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.kv_used_tokens = Gauge(
            name="sglang:kv_used_tokens",
            documentation="Number of actively used token slots in the KV cache pool.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.swa_available_tokens = Gauge(
            name="sglang:swa_available_tokens",
            documentation="Number of free token slots in the SWA pool (hybrid-SWA only).",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.swa_evictable_tokens = Gauge(
```
**EN:** This callable implements `SchedulerMetricsCollector.__init__`. It takes `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session` and mainly initializes instance state and defaults. This chunk is part 2 of 13 for the same logical block.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.__init__`。它接收 `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 2/13 部分。

### Lines 317-376: Method SchedulerMetricsCollector.__init__ (part 3/13)
```python
            name="sglang:swa_evictable_tokens",
            documentation="Number of evictable (radix-cached) token slots in the SWA pool.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.swa_used_tokens = Gauge(
            name="sglang:swa_used_tokens",
            documentation="Number of actively used token slots in the SWA pool.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.mamba_available_tokens = Gauge(
            name="sglang:mamba_available_tokens",
            documentation="Number of free state slots in the mamba SSM pool (hybrid-SSM only).",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.mamba_evictable_tokens = Gauge(
            name="sglang:mamba_evictable_tokens",
            documentation="Number of evictable (radix-cached) state slots in the mamba SSM pool.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.mamba_used_tokens = Gauge(
            name="sglang:mamba_used_tokens",
            documentation="Number of actively used state slots in the mamba SSM pool.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )

        # =================================================================
        # Speculative decoding
        # =================================================================
        self.spec_accept_length = Gauge(
            name="sglang:spec_accept_length",
            documentation="Mean acceptance length of speculative decoding (accepted drafts + bonus token per forward).",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.spec_accept_rate = Gauge(
            name="sglang:spec_accept_rate",
            documentation="Speculative acceptance rate (`accepted drafts / proposed drafts` in batch).",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )

        # =================================================================
        # Retract
        # =================================================================
        # TODO maybe remove this old gauge in favor of the new counter
        self.num_retracted_reqs = Gauge(
            name="sglang:num_retracted_reqs",
            documentation="The number of retracted requests.",
            labelnames=labels.keys(),
        )
        self.num_retracted_reqs_total = Counter(
            # The name is `requests` instead of `reqs` to avoid dup name error
            name="sglang:num_retracted_requests_total",
            documentation="Total number of retracted requests.",
            labelnames=labels.keys(),
```
**EN:** This callable implements `SchedulerMetricsCollector.__init__`. It takes `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session` and mainly initializes instance state and defaults. This chunk is part 3 of 13 for the same logical block.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.__init__`。它接收 `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 3/13 部分。

### Lines 377-436: Method SchedulerMetricsCollector.__init__ (part 4/13)
```python
        )
        self.num_retracted_input_tokens_total = Counter(
            name="sglang:num_retracted_input_tokens_total",
            documentation="Total number of retracted input tokens.",
            labelnames=labels.keys(),
        )
        self.num_retracted_output_tokens_total = Counter(
            name="sglang:num_retracted_output_tokens_total",
            documentation="Total number of retracted output tokens.",
            labelnames=labels.keys(),
        )
        self.num_paused_reqs = Gauge(
            name="sglang:num_paused_reqs",
            documentation="The number of paused requests by async weight sync.",
            labelnames=labels.keys(),
        )

        # =================================================================
        # PD disaggregation
        # =================================================================
        self.num_prefill_bootstrap_queue_reqs = Gauge(
            name="sglang:num_prefill_bootstrap_queue_reqs",
            documentation="The number of requests in the prefill bootstrap queue.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.num_prefill_inflight_queue_reqs = Gauge(
            name="sglang:num_prefill_inflight_queue_reqs",
            documentation="The number of requests in the prefill inflight queue.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.num_decode_prealloc_queue_reqs = Gauge(
            name="sglang:num_decode_prealloc_queue_reqs",
            documentation="The number of requests in the decode prealloc queue.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.num_decode_transfer_queue_reqs = Gauge(
            name="sglang:num_decode_transfer_queue_reqs",
            documentation="The number of requests in the decode transfer queue.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.kv_transfer_speed_gb_s = Histogram(
            name="sglang:kv_transfer_speed_gb_s",
            documentation="Histogram of KV cache transfer speed in GB/s.",
            labelnames=labels.keys(),
            buckets=(0.1, 0.5, 1, 5, 10, 25, 50, 100, 200, 400),
        )
        self.kv_transfer_latency_ms = Histogram(
            name="sglang:kv_transfer_latency_ms",
            documentation="Histogram of KV cache transfer latency in ms.",
            labelnames=labels.keys(),
            buckets=(1, 2, 5, 10, 25, 50, 100, 250, 500, 1000, 2500, 5000),
        )
        self.pending_prealloc_token_usage = Gauge(
            name="sglang:pending_prealloc_token_usage",
            documentation="The token usage for pending preallocated tokens (not preallocated yet).",
            labelnames=labels.keys(),
```
**EN:** This callable implements `SchedulerMetricsCollector.__init__`. It takes `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session` and mainly initializes instance state and defaults. This chunk is part 4 of 13 for the same logical block. In this range it manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.__init__`。它接收 `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 4/13 部分。 在这一范围内，它会管理模型权重或检查点。

### Lines 437-496: Method SchedulerMetricsCollector.__init__ (part 5/13)
```python
            multiprocess_mode="mostrecent",
        )
        self.num_bootstrap_failed_reqs = Counter(
            name="sglang:num_bootstrap_failed_reqs_total",
            documentation="The number of bootstrap failed requests.",
            labelnames=labels.keys(),
        )
        self.num_transfer_failed_reqs = Counter(
            name="sglang:num_transfer_failed_reqs_total",
            documentation="The number of transfer failed requests.",
            labelnames=labels.keys(),
        )
        self.num_prefill_retries_total = Counter(
            name="sglang:num_prefill_retries_total",
            documentation="Total number of prefill retries.",
            labelnames=labels.keys(),
        )
        self.kv_transfer_bootstrap_ms = Histogram(
            name="sglang:kv_transfer_bootstrap_ms",
            documentation="Histogram of KV transfer bootstrap time in ms.",
            labelnames=labels.keys(),
            buckets=(1, 2, 5, 10, 25, 50, 100, 250, 500, 1000, 2500),
        )
        self.kv_transfer_alloc_ms = Histogram(
            name="sglang:kv_transfer_alloc_ms",
            documentation="Histogram of KV transfer allocation waiting time in ms.",
            labelnames=labels.keys(),
            buckets=(1, 2, 5, 10, 25, 50, 100, 250, 500, 1000, 2500),
        )
        self.kv_transfer_total_mb = Histogram(
            name="sglang:kv_transfer_total_mb",
            documentation="Histogram of KV cache transfer size in MB.",
            labelnames=labels.keys(),
            buckets=(1, 5, 10, 50, 100, 500, 1000, 5000, 10000),
        )

        # =================================================================
        # Utilization
        # =================================================================
        self.utilization = Gauge(
            name="sglang:utilization",
            documentation="The utilization.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.fwd_occupancy = Gauge(
            name="sglang:fwd_occupancy",
            documentation="Forward pass GPU occupancy percentage.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )

        # =================================================================
        # Scheduler policy
        # =================================================================
        self.new_token_ratio = Gauge(
            name="sglang:new_token_ratio",
            documentation="The new token ratio.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
```
**EN:** This callable implements `SchedulerMetricsCollector.__init__`. It takes `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session` and mainly initializes instance state and defaults. This chunk is part 5 of 13 for the same logical block.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.__init__`。它接收 `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 5/13 部分。

### Lines 497-556: Method SchedulerMetricsCollector.__init__ (part 6/13)
```python
        )

        # =================================================================
        # CUDA graph
        # =================================================================
        # TODO maybe remove this old gauge in favor of the new counter
        self.is_cuda_graph = Gauge(
            name="sglang:is_cuda_graph",
            documentation="Whether the batch is using CUDA graph.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.cuda_graph_passes_total = Counter(
            name="sglang:cuda_graph_passes_total",
            documentation="Total number of forward passes categorized by CUDA graph.",
            labelnames=list(labels.keys()) + ["mode"],
        )

        # =================================================================
        # LoRA pool metrics (only created when LoRA is enabled)
        # =================================================================
        if self.enable_lora:
            self.lora_pool_slots_used = Gauge(
                name="sglang:lora_pool_slots_used",
                documentation="Number of LoRA adapter slots currently occupied in GPU memory.",
                labelnames=labels.keys(),
                multiprocess_mode="mostrecent",
            )
            self.lora_pool_slots_total = Gauge(
                name="sglang:lora_pool_slots_total",
                documentation="Total number of LoRA adapter slots available (max_loras_per_batch).",
                labelnames=labels.keys(),
                multiprocess_mode="mostrecent",
            )
            self.lora_pool_utilization = Gauge(
                name="sglang:lora_pool_utilization",
                documentation="LoRA pool utilization ratio (used/total). 1.0 means pool is full.",
                labelnames=labels.keys(),
                multiprocess_mode="mostrecent",
            )

        # =================================================================
        # HiCache metrics (only created when hierarchical cache is enabled)
        # =================================================================
        if self.enable_hierarchical_cache:
            self.hicache_host_used_tokens = Gauge(
                name="sglang:hicache_host_used_tokens",
                documentation="Number of tokens currently used in the host KV cache.",
                labelnames=labels.keys(),
                multiprocess_mode="mostrecent",
            )
            self.hicache_host_total_tokens = Gauge(
                name="sglang:hicache_host_total_tokens",
                documentation="Total capacity of the host KV cache in tokens.",
                labelnames=labels.keys(),
                multiprocess_mode="mostrecent",
            )

        # =================================================================
        # Streaming session metrics (only created when streaming sessions are enabled)
```
**EN:** This callable implements `SchedulerMetricsCollector.__init__`. It takes `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session` and mainly initializes instance state and defaults. This chunk is part 6 of 13 for the same logical block. In this range it records metrics or tracing signals; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.__init__`。它接收 `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 6/13 部分。 在这一范围内，它会记录指标或追踪信号；管理图捕获或回放逻辑。

### Lines 557-616: Method SchedulerMetricsCollector.__init__ (part 7/13)
```python
        # =================================================================
        if self.enable_streaming_session:
            self.num_streaming_sessions = Gauge(
                name="sglang:num_streaming_sessions",
                documentation="The number of streaming sessions.",
                labelnames=labels.keys(),
                multiprocess_mode="mostrecent",
            )
            self.streaming_session_held_tokens = Gauge(
                name="sglang:streaming_session_held_tokens",
                documentation="The number of KV tokens currently held by streaming session slots.",
                labelnames=labels.keys(),
                multiprocess_mode="mostrecent",
            )

        # =================================================================
        # Routing key metrics
        # =================================================================
        self.num_unique_running_routing_keys = Gauge(
            name="sglang:num_unique_running_routing_keys",
            documentation="Number of unique routing keys in running batch.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.routing_key_running_req_count = GaugeHistogram(
            name="sglang:routing_key_running_req_count",
            documentation="Distribution of routing keys by running request count (gt < count <= le).",
            labelnames=list(labels.keys()),
            bucket_bounds=ROUTING_KEY_REQ_COUNT_BUCKET_BOUNDS,
        )
        self.routing_key_all_req_count = GaugeHistogram(
            name="sglang:routing_key_all_req_count",
            documentation="Distribution of routing keys by running+waiting request count (gt < count <= le).",
            labelnames=list(labels.keys()),
            bucket_bounds=ROUTING_KEY_REQ_COUNT_BUCKET_BOUNDS,
        )

        # =================================================================
        # Request latency
        # =================================================================
        self.queue_time = Histogram(
            name="sglang:queue_time_seconds",
            documentation="Histogram of queueing time in seconds.",
            labelnames=labels.keys(),
            buckets=[
                0.000,
                0.001,
                0.005,
                0.010,
                0.050,
                0.100,
                0.200,
                0.500,
                1,
                2,
                3,
                4,
                5,
                10,
                15,
```
**EN:** This callable implements `SchedulerMetricsCollector.__init__`. It takes `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session` and mainly initializes instance state and defaults. This chunk is part 7 of 13 for the same logical block. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.__init__`。它接收 `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 7/13 部分。 在这一范围内，它会记录指标或追踪信号。

### Lines 617-676: Method SchedulerMetricsCollector.__init__ (part 8/13)
```python
                20,
                30,
                40,
                50,
                60,
                70,
                80,
                90,
                100,
                200,
                300,
                400,
                500,
                600,
                700,
                800,
                900,
                1000,
                1200,
                1400,
                1600,
                1800,
                2000,
                2500,
                3000,
            ],
        )
        self.per_stage_req_latency_seconds = Histogram(
            name="sglang:per_stage_req_latency_seconds",
            documentation="The latency of each stage of requests.",
            # captures latency in range [1ms - ~1191s]
            buckets=exponential_buckets(start=0.001, width=1.62, length=30),
            labelnames=list(labels.keys()) + ["stage"],
        )

        # =================================================================
        # Grammar
        # =================================================================
        self.grammar_compilation_time = Histogram(
            name="sglang:grammar_compilation_time_seconds",
            documentation="Histogram of grammar compilation time in seconds.",
            labelnames=labels.keys(),
            buckets=[
                0.0,
                0.01,
                0.02,
                0.05,
                0.1,
                0.2,
                0.5,
                1,
                2,
                5,
                10,
                20,
                30,
                60,
                90,
                120,
                240,
```
**EN:** This callable implements `SchedulerMetricsCollector.__init__`. It takes `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session` and mainly initializes instance state and defaults. This chunk is part 8 of 13 for the same logical block. In this range it handles grammar or regular-expression constraints; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.__init__`。它接收 `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 8/13 部分。 在这一范围内，它会处理语法或正则约束；管理图捕获或回放逻辑。

### Lines 677-736: Method SchedulerMetricsCollector.__init__ (part 9/13)
```python
            ],
        )
        self.num_grammar_cache_hit = Counter(
            name="sglang:num_grammar_cache_hit_total",
            documentation="Number of grammar cache hits.",
            labelnames=labels.keys(),
        )
        self.num_grammar_aborted = Counter(
            name="sglang:num_grammar_aborted_total",
            documentation="Number of grammar aborted requests.",
            labelnames=labels.keys(),
        )
        self.num_grammar_timeout = Counter(
            name="sglang:num_grammar_timeout_total",
            documentation="Number of grammar timeouts.",
            labelnames=labels.keys(),
        )
        self.num_grammar_total = Counter(
            name="sglang:num_grammar_total",
            documentation="Number of the total grammar requests.",
            labelnames=labels.keys(),
        )
        self.grammar_schema_count = Histogram(
            name="sglang:grammar_schema_count",
            documentation="Histogram of grammar schema count.",
            labelnames=labels.keys(),
            buckets=[
                0,
                1,
                2,
                5,
                10,
                20,
                30,
                40,
                60,
                80,
                100,
                120,
                140,
                160,
                180,
                200,
                300,
                400,
                500,
                700,
                1000,
            ],
        )
        self.grammar_ebnf_size = Histogram(
            name="sglang:grammar_ebnf_size",
            documentation="Histogram of grammar EBNF size.",
            labelnames=labels.keys(),
            buckets=[
                0,
                50,
                100,
                200,
                300,
```
**EN:** This callable implements `SchedulerMetricsCollector.__init__`. It takes `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session` and mainly initializes instance state and defaults. This chunk is part 9 of 13 for the same logical block. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.__init__`。它接收 `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 9/13 部分。 在这一范围内，它会处理语法或正则约束。

### Lines 737-796: Method SchedulerMetricsCollector.__init__ (part 10/13)
```python
                500,
                1000,
                2000,
                3000,
                5000,
                10000,
                20000,
                30000,
                50000,
                100000,
            ],
        )

        tree_traversal_time_buckets = [
            0.0,
            0.01,
            0.02,
            0.05,
            0.1,
            0.2,
            0.5,
            1,
            2,
            5,
            10,
            15,
            30,
            60,
            90,
            120,
            240,
        ]
        self.grammar_tree_traversal_time_avg = Histogram(
            name="sglang:grammar_tree_traversal_time_avg",
            documentation="Histogram of average grammar tree traversal time in seconds.",
            labelnames=labels.keys(),
            buckets=tree_traversal_time_buckets,
        )
        self.grammar_tree_traversal_time_max = Histogram(
            name="sglang:grammar_tree_traversal_time_max",
            documentation="Histogram of max grammar tree traversal time in seconds.",
            labelnames=labels.keys(),
            buckets=tree_traversal_time_buckets,
        )

        # =================================================================
        # Execution
        # =================================================================
        if (
            labels["moe_ep_rank"] == 0
        ) and envs.SGLANG_ENABLE_EPLB_BALANCEDNESS_METRIC.get():
            self.eplb_balancedness = Summary(
                name="sglang:eplb_balancedness",
                documentation="Balancedness of MoE in expert parallelism.",
                labelnames=list(labels.keys()) + ["forward_mode"],
            )

        self.realtime_tokens_total = Counter(
            name="sglang:realtime_tokens_total",
            documentation=(
```
**EN:** This callable implements `SchedulerMetricsCollector.__init__`. It takes `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session` and mainly initializes instance state and defaults. This chunk is part 10 of 13 for the same logical block. In this range it reads environment-driven configuration; records metrics or tracing signals; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.__init__`。它接收 `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 10/13 部分。 在这一范围内，它会读取环境变量驱动的配置；记录指标或追踪信号；处理语法或正则约束。

### Lines 797-856: Method SchedulerMetricsCollector.__init__ (part 11/13)
```python
                "Total number of tokens processed (updated on each log interval). "
                "mode: prefill_compute, prefill_cache, decode."
            ),
            labelnames=list(labels.keys()) + ["mode"],
        )
        self.forward_execution_seconds_total = Counter(
            name="sglang:forward_execution_seconds_total",
            documentation=(
                "Total time that GPU is busy executing model forward passes. "
                "Refer to ForwardMode for category labels."
            ),
            labelnames=list(labels.keys()) + ["category"],
        )
        self.estimated_flops_per_gpu_total = Counter(
            name="sglang:estimated_flops_per_gpu_total",
            documentation=(
                "Estimated number of floating point operations per GPU "
                "(for Model FLOPs Utilization calculations)."
            ),
            labelnames=labels.keys(),
        )
        self.estimated_read_bytes_per_gpu_total = Counter(
            name="sglang:estimated_read_bytes_per_gpu_total",
            documentation=(
                "Estimated number of bytes read from memory per GPU "
                "(for Model FLOPs Utilization calculations)."
            ),
            labelnames=labels.keys(),
        )
        self.estimated_write_bytes_per_gpu_total = Counter(
            name="sglang:estimated_write_bytes_per_gpu_total",
            documentation=(
                "Estimated number of bytes written to memory per GPU "
                "(for Model FLOPs Utilization calculations)."
            ),
            labelnames=labels.keys(),
        )

        self.dp_cooperation_realtime_tokens_total = Counter(
            name="sglang:dp_cooperation_realtime_tokens_total",
            documentation=(
                "Total number of tokens processed with labels about DP cooperation. "
                "mode: prefill_compute, prefill_cache, decode."
            ),
            labelnames=list(labels.keys()) + ["mode", "num_prefill_ranks"],
        )
        self.dp_cooperation_forward_execution_seconds_total = Counter(
            name="sglang:dp_cooperation_forward_execution_seconds_total",
            documentation=(
                "Total time that GPU is busy executing model forward passes, "
                "with labels about DP cooperation. "
                "Refer to ForwardMode for category labels."
            ),
            labelnames=list(labels.keys()) + ["category", "num_prefill_ranks"],
        )

        # =================================================================
        # Prefill delayer
        # =================================================================
        max_delay = server_args.prefill_delayer_max_delay_passes
```
**EN:** This callable implements `SchedulerMetricsCollector.__init__`. It takes `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session` and mainly initializes instance state and defaults. This chunk is part 11 of 13 for the same logical block. In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.__init__`。它接收 `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 11/13 部分。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 857-916: Method SchedulerMetricsCollector.__init__ (part 12/13)
```python
        self.prefill_delayer_wait_forward_passes = Histogram(
            name="sglang:prefill_delayer_wait_forward_passes",
            documentation="Histogram of forward passes waited by prefill delayer.",
            labelnames=labels.keys(),
            buckets=sorted(
                set(
                    x
                    for x in (
                        server_args.prefill_delayer_forward_passes_buckets
                        or [5, 20, 50, 100, 200]
                    )
                    if x < max_delay
                )
                # Need bucket "<=0" for zero-delay cases, and "max_delay-1" to distinguish "max_delay" timeout passes
                | {0, max_delay - 1}
            ),
        )
        self.prefill_delayer_wait_seconds = Histogram(
            name="sglang:prefill_delayer_wait_seconds",
            documentation="Histogram of wait time in seconds by prefill delayer.",
            labelnames=labels.keys(),
            buckets=sorted(
                set(
                    server_args.prefill_delayer_wait_seconds_buckets
                    or [1, 2, 5, 10, 20, 50, 100, 200, 500]
                )
                # Need bucket "<=0" for zero-delay cases
                | {0}
            ),
        )
        self.prefill_delayer_outcomes_total = Counter(
            name="sglang:prefill_delayer_outcomes_total",
            documentation="Prefill delayer outcome counts.",
            labelnames=[
                *labels.keys(),
                "input_estimation",
                "output_allow",
                "output_reason",
                "actual_execution",
            ],
        )

        # =================================================================
        # Constants (set once at startup via emit_constants)
        # =================================================================
        self.max_total_num_tokens = Gauge(
            name="sglang:max_total_num_tokens",
            documentation="Maximum total number of tokens in the KV cache pool.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.max_running_requests_under_SLO = Gauge(
            name="sglang:max_running_requests_under_SLO",
            documentation="The maximum number of running requests under SLO.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.engine_startup_time = Gauge(
            name="sglang:engine_startup_time",
            documentation="The time taken for the engine to start up.",
```
**EN:** This callable implements `SchedulerMetricsCollector.__init__`. It takes `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session` and mainly initializes instance state and defaults. This chunk is part 12 of 13 for the same logical block.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.__init__`。它接收 `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 12/13 部分。

### Lines 917-949: Method SchedulerMetricsCollector.__init__ (part 13/13)
```python
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.engine_load_weights_time = Gauge(
            name="sglang:engine_load_weights_time",
            documentation="The time taken for the engine to load weights.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.page_size = Gauge(
            name="sglang:page_size",
            documentation="KV cache page size in tokens.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.num_pages = Gauge(
            name="sglang:num_pages",
            documentation="Number of KV cache pages.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.context_len = Gauge(
            name="sglang:context_len",
            documentation="Maximum context length.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
        self.startup_available_gpu_memory_gb = Gauge(
            name="sglang:startup_available_gpu_memory_gb",
            documentation="Available GPU memory in GB at startup.",
            labelnames=labels.keys(),
            multiprocess_mode="mostrecent",
        )
```
**EN:** This callable implements `SchedulerMetricsCollector.__init__`. It takes `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session` and mainly initializes instance state and defaults. This chunk is part 13 of 13 for the same logical block. In this range it manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.__init__`。它接收 `labels`, `enable_lora`, `enable_hierarchical_cache`, `enable_streaming_session`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 13/13 部分。 在这一范围内，它会管理模型权重或检查点。

### Lines 951-1005: Method SchedulerMetricsCollector.init_new
```python
    @classmethod
    def init_new(
        cls,
        *,
        server_args: "ServerArgs",
        ps: Any,
        tp_rank: int,
        pp_rank: int,
        dp_rank: Optional[int],
        enable_priority_scheduling: bool,
        enable_lora: bool,
        enable_hierarchical_cache: bool,
    ) -> "SchedulerMetricsCollectorContext":
        enable_metrics = server_args.enable_metrics
        is_stats_logging_rank = ps.attn_tp_rank == 0
        current_scheduler_metrics_enabled = enable_metrics and (
            is_stats_logging_rank or server_args.enable_metrics_for_all_schedulers
        )
        enable_kv_cache_events = bool(
            server_args.kv_events_config
            and ps.attn_tp_rank == 0
            and ps.attn_cp_rank == 0
        )
        collector: Optional["SchedulerMetricsCollector"] = None
        if enable_metrics:
            engine_type = DisaggregationMode.to_engine_type(
                server_args.disaggregation_mode
            )
            labels = {
                "model_name": server_args.served_model_name,
                "engine_type": engine_type,
                "tp_rank": tp_rank,
                "pp_rank": pp_rank,
                "moe_ep_rank": ps.moe_ep_rank,
            }
            if enable_priority_scheduling:
                labels["priority"] = ""
            if dp_rank is not None:
                labels["dp_rank"] = dp_rank
            if server_args.extra_metric_labels:
                labels.update(server_args.extra_metric_labels)
            collector = cls(
                labels=labels,
                enable_lora=enable_lora,
                enable_hierarchical_cache=enable_hierarchical_cache,
                enable_streaming_session=server_args.enable_streaming_session,
                server_args=server_args,
            )
        return SchedulerMetricsCollectorContext(
            enable_metrics=enable_metrics,
            is_stats_logging_rank=is_stats_logging_rank,
            current_scheduler_metrics_enabled=current_scheduler_metrics_enabled,
            enable_kv_cache_events=enable_kv_cache_events,
            collector=collector,
        )
```
**EN:** This callable implements `SchedulerMetricsCollector.init_new`. It takes `cls` and mainly implements init new. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.init_new`。它接收 `cls`，主要用于实现 init new 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 1007-1009: Method SchedulerMetricsCollector._log_gauge
```python
    def _log_gauge(self, gauge: Gauge, data: Union[int, float]) -> None:
        # Convenience function for logging a scalar to gauge.
        gauge.labels(**self.labels).set(data)
```
**EN:** This callable implements `SchedulerMetricsCollector._log_gauge`. It takes `gauge`, `data` and mainly implements log gauge.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector._log_gauge`。它接收 `gauge`, `data`，主要用于实现 log gauge 相关逻辑。

### Lines 1011-1023: Method SchedulerMetricsCollector._log_gauge_queue_count
```python
    def _log_gauge_queue_count(self, gauge: Gauge, data: QueueCount) -> None:
        # Log a QueueCount to gauge: total under default labels, per-priority breakdown under priority="<int>".
        # NOTE: When priority scheduling is enabled, the total is recorded under
        # priority="" (the default label value). Per-priority breakdowns are recorded
        # with priority="<int>". Grafana queries should use priority="" for totals.
        gauge.labels(**self.labels).set(data.total)
        if data.by_priority is not None:
            self._known_priorities.update(data.by_priority.keys())
            for priority in self._known_priorities:
                value = data.by_priority.get(priority, 0)
                labels = dict(self.labels)
                labels["priority"] = str(priority)
                gauge.labels(**labels).set(value)
```
**EN:** This callable implements `SchedulerMetricsCollector._log_gauge_queue_count`. It takes `gauge`, `data` and mainly implements log gauge queue count.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector._log_gauge_queue_count`。它接收 `gauge`, `data`，主要用于实现 log gauge queue count 相关逻辑。

### Lines 1025-1026: Method SchedulerMetricsCollector._log_histogram
```python
    def _log_histogram(self, histogram, data: Union[int, float]) -> None:
        histogram.labels(**self.labels).observe(data)
```
**EN:** This callable implements `SchedulerMetricsCollector._log_histogram`. It takes `histogram`, `data` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector._log_histogram`。它接收 `histogram`, `data`，主要用于将数据转换为另一种表示。

### Lines 1028-1029: Method SchedulerMetricsCollector.increment_bootstrap_failed_reqs
```python
    def increment_bootstrap_failed_reqs(self) -> None:
        self.num_bootstrap_failed_reqs.labels(**self.labels).inc(1)
```
**EN:** This callable implements `SchedulerMetricsCollector.increment_bootstrap_failed_reqs` and mainly implements increment bootstrap failed reqs.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.increment_bootstrap_failed_reqs`，主要用于实现 increment bootstrap failed reqs 相关逻辑。

### Lines 1031-1032: Method SchedulerMetricsCollector.increment_transfer_failed_reqs
```python
    def increment_transfer_failed_reqs(self) -> None:
        self.num_transfer_failed_reqs.labels(**self.labels).inc(1)
```
**EN:** This callable implements `SchedulerMetricsCollector.increment_transfer_failed_reqs` and mainly implements increment transfer failed reqs.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.increment_transfer_failed_reqs`，主要用于实现 increment transfer failed reqs 相关逻辑。

### Lines 1034-1036: Method SchedulerMetricsCollector.increment_prefill_retries
```python
    def increment_prefill_retries(self, count: int) -> None:
        if count > 0:
            self.num_prefill_retries_total.labels(**self.labels).inc(count)
```
**EN:** This callable implements `SchedulerMetricsCollector.increment_prefill_retries`. It takes `count` and mainly implements increment prefill retries.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.increment_prefill_retries`。它接收 `count`，主要用于实现 increment prefill retries 相关逻辑。

### Lines 1038-1046: Method SchedulerMetricsCollector.observe_kv_transfer_metrics
```python
    def observe_kv_transfer_metrics(
        self,
        latency_ms: float,
        total_mb: float,
        speed_gb_s: float,
    ) -> None:
        self._log_histogram(self.kv_transfer_latency_ms, latency_ms)
        self._log_histogram(self.kv_transfer_total_mb, total_mb)
        self._log_histogram(self.kv_transfer_speed_gb_s, speed_gb_s)
```
**EN:** This callable implements `SchedulerMetricsCollector.observe_kv_transfer_metrics`. It takes `latency_ms`, `total_mb`, `speed_gb_s` and mainly implements observe kv transfer metrics. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.observe_kv_transfer_metrics`。它接收 `latency_ms`, `total_mb`, `speed_gb_s`，主要用于实现 observe kv transfer metrics 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 1048-1054: Method SchedulerMetricsCollector.observe_kv_transfer_bootstrap
```python
    def observe_kv_transfer_bootstrap(
        self,
        bootstrap_ms: float,
        alloc_ms: float,
    ) -> None:
        self._log_histogram(self.kv_transfer_bootstrap_ms, bootstrap_ms)
        self._log_histogram(self.kv_transfer_alloc_ms, alloc_ms)
```
**EN:** This callable implements `SchedulerMetricsCollector.observe_kv_transfer_bootstrap`. It takes `bootstrap_ms`, `alloc_ms` and mainly implements observe kv transfer bootstrap.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.observe_kv_transfer_bootstrap`。它接收 `bootstrap_ms`, `alloc_ms`，主要用于实现 observe kv transfer bootstrap 相关逻辑。

### Lines 1056-1058: Method SchedulerMetricsCollector.observe_per_stage_req_latency
```python
    def observe_per_stage_req_latency(self, stage: str, latency: float) -> None:
        labels_with_stage = {**self.labels, "stage": stage}
        self.per_stage_req_latency_seconds.labels(**labels_with_stage).observe(latency)
```
**EN:** This callable implements `SchedulerMetricsCollector.observe_per_stage_req_latency`. It takes `stage`, `latency` and mainly implements observe per stage req latency.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.observe_per_stage_req_latency`。它接收 `stage`, `latency`，主要用于实现 observe per stage req latency 相关逻辑。

### Lines 1060-1061: Method SchedulerMetricsCollector.observe_queue_time
```python
    def observe_queue_time(self, latency: float) -> None:
        self._log_histogram(self.queue_time, latency)
```
**EN:** This callable implements `SchedulerMetricsCollector.observe_queue_time`. It takes `latency` and mainly implements observe queue time.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.observe_queue_time`。它接收 `latency`，主要用于实现 observe queue time 相关逻辑。

### Lines 1063-1084: Method SchedulerMetricsCollector.observe_prefill_delayer_outcome
```python
    def observe_prefill_delayer_outcome(
        self,
        forward_passes: int,
        wait_seconds: float,
        input_estimation: str,
        output_allow: bool,
        output_reason: str,
        actual_execution: bool,
    ) -> None:
        if output_allow and actual_execution:
            self._log_histogram(
                self.prefill_delayer_wait_forward_passes, forward_passes
            )
            self._log_histogram(self.prefill_delayer_wait_seconds, wait_seconds)

        self.prefill_delayer_outcomes_total.labels(
            **self.labels,
            input_estimation=input_estimation,
            output_allow=str(output_allow).lower(),
            output_reason=output_reason,
            actual_execution=str(actual_execution).lower(),
        ).inc(1)
```
**EN:** This callable implements `SchedulerMetricsCollector.observe_prefill_delayer_outcome`. It takes `forward_passes`, `wait_seconds`, `input_estimation`, `output_allow` and mainly implements observe prefill delayer outcome.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.observe_prefill_delayer_outcome`。它接收 `forward_passes`, `wait_seconds`, `input_estimation`, `output_allow`，主要用于实现 observe prefill delayer outcome 相关逻辑。

### Lines 1086-1098: Method SchedulerMetricsCollector.increment_retracted_reqs
```python
    def increment_retracted_reqs(
        self,
        num_retracted_reqs: int,
        num_retracted_input_tokens: int,
        num_retracted_output_tokens: int,
    ) -> None:
        self.num_retracted_reqs_total.labels(**self.labels).inc(num_retracted_reqs)
        self.num_retracted_input_tokens_total.labels(**self.labels).inc(
            num_retracted_input_tokens
        )
        self.num_retracted_output_tokens_total.labels(**self.labels).inc(
            num_retracted_output_tokens
        )
```
**EN:** This callable implements `SchedulerMetricsCollector.increment_retracted_reqs`. It takes `num_retracted_reqs`, `num_retracted_input_tokens`, `num_retracted_output_tokens` and mainly implements increment retracted reqs.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.increment_retracted_reqs`。它接收 `num_retracted_reqs`, `num_retracted_input_tokens`, `num_retracted_output_tokens`，主要用于实现 increment retracted reqs 相关逻辑。

### Lines 1100-1102: Method SchedulerMetricsCollector.increment_decode_cuda_graph_pass
```python
    def increment_decode_cuda_graph_pass(self, value: bool) -> None:
        mode = "decode_cuda_graph" if value else "decode_none"
        self.cuda_graph_passes_total.labels(**self.labels, mode=mode).inc(1)
```
**EN:** This callable implements `SchedulerMetricsCollector.increment_decode_cuda_graph_pass`. It takes `value` and mainly implements increment decode cuda graph pass. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.increment_decode_cuda_graph_pass`。它接收 `value`，主要用于实现 increment decode cuda graph pass 相关逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 1104-1106: Method SchedulerMetricsCollector.increment_prefill_cuda_graph_pass
```python
    def increment_prefill_cuda_graph_pass(self, value: bool) -> None:
        mode = "prefill_cuda_graph" if value else "prefill_none"
        self.cuda_graph_passes_total.labels(**self.labels, mode=mode).inc(1)
```
**EN:** This callable implements `SchedulerMetricsCollector.increment_prefill_cuda_graph_pass`. It takes `value` and mainly implements increment prefill cuda graph pass. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.increment_prefill_cuda_graph_pass`。它接收 `value`，主要用于实现 increment prefill cuda graph pass 相关逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 1108-1113: Method SchedulerMetricsCollector.increment_eplb_balancedness
```python
    def increment_eplb_balancedness(
        self, forward_mode: str, balancedness: float
    ) -> None:
        self.eplb_balancedness.labels(**self.labels, forward_mode=forward_mode).observe(
            balancedness
        )
```
**EN:** This callable implements `SchedulerMetricsCollector.increment_eplb_balancedness`. It takes `forward_mode`, `balancedness` and mainly implements increment eplb balancedness.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.increment_eplb_balancedness`。它接收 `forward_mode`, `balancedness`，主要用于实现 increment eplb balancedness 相关逻辑。

### Lines 1115-1135: Method SchedulerMetricsCollector.increment_realtime_tokens
```python
    def increment_realtime_tokens(
        self,
        dp_cooperation_info: Optional[DPCooperationInfo],
        prefill_compute_tokens=0,
        prefill_cache_tokens=0,
        decode_tokens=0,
    ):
        for mode, delta in [
            ("prefill_compute", prefill_compute_tokens),
            ("prefill_cache", prefill_cache_tokens),
            ("decode", decode_tokens),
        ]:
            if delta == 0:
                continue
            self.realtime_tokens_total.labels(**self.labels, mode=mode).inc(delta)
            if dp_cooperation_info is not None:
                self.dp_cooperation_realtime_tokens_total.labels(
                    **self.labels,
                    mode=mode,
                    **dp_cooperation_info.to_labels(),
                ).inc(delta)
```
**EN:** This callable implements `SchedulerMetricsCollector.increment_realtime_tokens`. It takes `dp_cooperation_info`, `prefill_compute_tokens`, `prefill_cache_tokens`, `decode_tokens` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.increment_realtime_tokens`。它接收 `dp_cooperation_info`, `prefill_compute_tokens`, `prefill_cache_tokens`, `decode_tokens`，主要用于将数据转换为另一种表示。

### Lines 1137-1151: Method SchedulerMetricsCollector.increment_forward_execution_seconds
```python
    def increment_forward_execution_seconds(
        self,
        category: str,
        t: float,
        dp_cooperation_info: Optional[DPCooperationInfo] = None,
    ):
        self.forward_execution_seconds_total.labels(
            **self.labels, category=category
        ).inc(t)
        if dp_cooperation_info is not None:
            self.dp_cooperation_forward_execution_seconds_total.labels(
                **self.labels,
                category=category,
                **dp_cooperation_info.to_labels(),
            ).inc(t)
```
**EN:** This callable implements `SchedulerMetricsCollector.increment_forward_execution_seconds`. It takes `category`, `t`, `dp_cooperation_info` and mainly implements increment forward execution seconds.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.increment_forward_execution_seconds`。它接收 `category`, `t`, `dp_cooperation_info`，主要用于实现 increment forward execution seconds 相关逻辑。

### Lines 1153-1170: Method SchedulerMetricsCollector.increment_estimated_perf
```python
    def increment_estimated_perf(
        self,
        num_flops_per_gpu: float = 0.0,
        num_read_bytes_per_gpu: float = 0.0,
        num_write_bytes_per_gpu: float = 0.0,
    ) -> None:
        if num_flops_per_gpu > 0:
            self.estimated_flops_per_gpu_total.labels(**self.labels).inc(
                num_flops_per_gpu
            )
        if num_read_bytes_per_gpu > 0:
            self.estimated_read_bytes_per_gpu_total.labels(**self.labels).inc(
                num_read_bytes_per_gpu
            )
        if num_write_bytes_per_gpu > 0:
            self.estimated_write_bytes_per_gpu_total.labels(**self.labels).inc(
                num_write_bytes_per_gpu
            )
```
**EN:** This callable implements `SchedulerMetricsCollector.increment_estimated_perf`. It takes `num_flops_per_gpu`, `num_read_bytes_per_gpu`, `num_write_bytes_per_gpu` and mainly implements increment estimated perf.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.increment_estimated_perf`。它接收 `num_flops_per_gpu`, `num_read_bytes_per_gpu`, `num_write_bytes_per_gpu`，主要用于实现 increment estimated perf 相关逻辑。

### Lines 1172-1231: Method SchedulerMetricsCollector.log_stats (part 1/2)
```python
    def log_stats(self, stats: SchedulerStats) -> None:
        # Basics
        self._log_gauge_queue_count(self.num_running_reqs, stats.num_running_reqs)
        self._log_gauge_queue_count(self.num_queue_reqs, stats.num_queue_reqs)
        self._log_gauge(self.num_grammar_queue_reqs, stats.num_grammar_queue_reqs)
        self._log_gauge(self.gen_throughput, stats.gen_throughput)
        self._log_gauge(self.cache_hit_rate, stats.cache_hit_rate)
        self._log_gauge(self.decode_sum_seq_lens, stats.decode_sum_seq_lens)

        # Memory pool usage ratios
        self._log_gauge(self.token_usage, stats.token_usage)
        self._log_gauge(self.full_token_usage, stats.full_token_usage)
        self._log_gauge(self.swa_token_usage, stats.swa_token_usage)
        self._log_gauge(self.mamba_usage, stats.mamba_usage)

        # Absolute token counts
        self._log_gauge(self.num_used_tokens, stats.num_used_tokens)
        self._log_gauge(self.kv_available_tokens, stats.kv_available_tokens)
        self._log_gauge(self.kv_evictable_tokens, stats.kv_evictable_tokens)
        self._log_gauge(self.kv_used_tokens, stats.kv_used_tokens)
        self._log_gauge(self.swa_available_tokens, stats.swa_available_tokens)
        self._log_gauge(self.swa_evictable_tokens, stats.swa_evictable_tokens)
        self._log_gauge(self.swa_used_tokens, stats.swa_used_tokens)
        self._log_gauge(self.mamba_available_tokens, stats.mamba_available_tokens)
        self._log_gauge(self.mamba_evictable_tokens, stats.mamba_evictable_tokens)
        self._log_gauge(self.mamba_used_tokens, stats.mamba_used_tokens)

        # Speculative decoding
        self._log_gauge(self.spec_accept_length, stats.spec_accept_length)
        self._log_gauge(self.spec_accept_rate, stats.spec_accept_rate)

        # Retract
        self._log_gauge(self.num_retracted_reqs, stats.num_retracted_reqs)
        self._log_gauge(self.num_paused_reqs, stats.num_paused_reqs)

        # PD disaggregation
        self._log_gauge_queue_count(
            self.num_prefill_bootstrap_queue_reqs,
            stats.num_prefill_bootstrap_queue_reqs,
        )
        self._log_gauge_queue_count(
            self.num_prefill_inflight_queue_reqs, stats.num_prefill_inflight_queue_reqs
        )
        self._log_gauge_queue_count(
            self.num_decode_prealloc_queue_reqs, stats.num_decode_prealloc_queue_reqs
        )
        self._log_gauge_queue_count(
            self.num_decode_transfer_queue_reqs, stats.num_decode_transfer_queue_reqs
        )
        self._log_gauge(
            self.pending_prealloc_token_usage, stats.pending_prealloc_token_usage
        )

        # Utilization
        self._log_gauge(self.utilization, stats.utilization)
        self._log_gauge(self.fwd_occupancy, stats.fwd_occupancy)

        # Scheduler policy
        self._log_gauge(self.new_token_ratio, stats.new_token_ratio)

```
**EN:** This callable implements `SchedulerMetricsCollector.log_stats`. It takes `stats` and mainly implements log stats. This chunk is part 1 of 2 for the same logical block. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.log_stats`。它接收 `stats`，主要用于实现 log stats 相关逻辑。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会处理语法或正则约束。

### Lines 1232-1268: Method SchedulerMetricsCollector.log_stats (part 2/2)
```python
        # CUDA graph
        self._log_gauge(self.is_cuda_graph, stats.is_cuda_graph)

        # LoRA pool metrics
        if self.enable_lora:
            self._log_gauge(self.lora_pool_slots_used, stats.lora_pool_slots_used)
            self._log_gauge(self.lora_pool_slots_total, stats.lora_pool_slots_total)
            self._log_gauge(self.lora_pool_utilization, stats.lora_pool_utilization)

        # HiCache metrics
        if self.enable_hierarchical_cache:
            self._log_gauge(
                self.hicache_host_used_tokens, stats.hicache_host_used_tokens
            )
            self._log_gauge(
                self.hicache_host_total_tokens, stats.hicache_host_total_tokens
            )

        # Streaming session metrics
        if self.enable_streaming_session:
            self._log_gauge(self.num_streaming_sessions, stats.num_streaming_sessions)
            self._log_gauge(
                self.streaming_session_held_tokens, stats.streaming_session_held_tokens
            )

        # Routing key metrics
        self._log_gauge(
            self.num_unique_running_routing_keys, stats.num_unique_running_routing_keys
        )
        self.routing_key_running_req_count.set_by_current_observations(
            self.labels, stats.routing_key_running_req_counts
        )
        self.routing_key_all_req_count.set_by_current_observations(
            self.labels, stats.routing_key_all_req_counts
        )

        self.last_log_time = time.perf_counter()
```
**EN:** This callable implements `SchedulerMetricsCollector.log_stats`. It takes `stats` and mainly implements log stats. This chunk is part 2 of 2 for the same logical block. In this range it records metrics or tracing signals; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.log_stats`。它接收 `stats`，主要用于实现 log stats 相关逻辑。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会记录指标或追踪信号；管理图捕获或回放逻辑。

### Lines 1270-1293: Method SchedulerMetricsCollector.log_grammar_stats
```python
    def log_grammar_stats(self, grammar_stats) -> None:
        if grammar_stats.compilation_time is not None:
            self._log_histogram(
                self.grammar_compilation_time, grammar_stats.compilation_time
            )
        if grammar_stats.schema_count is not None:
            self._log_histogram(self.grammar_schema_count, grammar_stats.schema_count)
        if grammar_stats.ebnf_size is not None:
            self._log_histogram(self.grammar_ebnf_size, grammar_stats.ebnf_size)
        tree_times = grammar_stats.tree_traversal_time
        if tree_times:
            max_time = max(tree_times)
            avg_time = sum(tree_times) / len(tree_times)
            self._log_histogram(self.grammar_tree_traversal_time_max, max_time)
            self._log_histogram(self.grammar_tree_traversal_time_avg, avg_time)
        if grammar_stats.is_cache_hit:
            self.num_grammar_cache_hit.labels(**self.labels).inc(1)
        if grammar_stats.is_grammar_aborted:
            self.num_grammar_aborted.labels(**self.labels).inc(1)
        if grammar_stats.num_timeout > 0:
            self.num_grammar_timeout.labels(**self.labels).inc(
                grammar_stats.num_timeout
            )
        self.num_grammar_total.labels(**self.labels).inc(1)
```
**EN:** This callable implements `SchedulerMetricsCollector.log_grammar_stats`. It takes `grammar_stats` and mainly implements log grammar stats. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.log_grammar_stats`。它接收 `grammar_stats`，主要用于实现 log grammar stats 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 1295-1318: Method SchedulerMetricsCollector.emit_constants
```python
    def emit_constants(
        self,
        max_total_num_tokens: int,
        max_running_requests_under_SLO: Optional[int],
        engine_startup_time: float,
        engine_load_weights_time: float,
        page_size: int,
        num_pages: int,
        context_len: int,
        startup_available_gpu_memory_gb: float,
    ) -> None:
        self._log_gauge(self.max_total_num_tokens, max_total_num_tokens)
        if max_running_requests_under_SLO is not None:
            self._log_gauge(
                self.max_running_requests_under_SLO, max_running_requests_under_SLO
            )
        self._log_gauge(self.engine_startup_time, engine_startup_time)
        self._log_gauge(self.engine_load_weights_time, engine_load_weights_time)
        self._log_gauge(self.page_size, page_size)
        self._log_gauge(self.num_pages, num_pages)
        self._log_gauge(self.context_len, context_len)
        self._log_gauge(
            self.startup_available_gpu_memory_gb, startup_available_gpu_memory_gb
        )
```
**EN:** This callable implements `SchedulerMetricsCollector.emit_constants`. It takes `max_total_num_tokens`, `max_running_requests_under_SLO`, `engine_startup_time`, `engine_load_weights_time` and mainly implements emit constants. In this range it manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `SchedulerMetricsCollector.emit_constants`。它接收 `max_total_num_tokens`, `max_running_requests_under_SLO`, `engine_startup_time`, `engine_load_weights_time`，主要用于实现 emit constants 相关逻辑。 在这一范围内，它会管理模型权重或检查点。

### Lines 1321-1321: Class TokenizerMetricsCollector
```python
class TokenizerMetricsCollector:
```
**EN:** This range introduces `TokenizerMetricsCollector` and defines the structure or metadata that its methods rely on. In this range it records metrics or tracing signals.
**CN:** 这一段引入 `TokenizerMetricsCollector`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 1322-1381: Method TokenizerMetricsCollector.__init__ (part 1/4)
```python
    def __init__(
        self,
        server_args: Optional[ServerArgs] = None,
        labels: Dict[str, str] = None,
        bucket_time_to_first_token: Optional[List[float]] = None,
        bucket_inter_token_latency: Optional[List[float]] = None,
        bucket_e2e_request_latency: Optional[List[float]] = None,
    ) -> None:
        # We need to import prometheus_client after setting the env variable `PROMETHEUS_MULTIPROC_DIR`
        from prometheus_client import Counter, Histogram

        self.labels = labels or {}

        self.prompt_tokens_total = Counter(
            name="sglang:prompt_tokens_total",
            documentation="Number of prefill tokens processed.",
            labelnames=labels.keys(),
        )
        self.generation_tokens_total = Counter(
            name="sglang:generation_tokens_total",
            documentation="Number of generation tokens processed.",
            labelnames=labels.keys(),
        )

        default_bucket_prompt_tokens = [
            100,
            300,
            500,
            700,
            1000,
            1500,
            2000,
            3000,
            4000,
            5000,
            6000,
            7000,
            8000,
            9000,
            10000,
            12500,
            15000,
            17500,
            20000,
            22500,
            25000,
            27500,
            30000,
            35000,
            40000,
            60000,
            80000,
            100000,
            200000,
            300000,
            400000,
            600000,
            800000,
            1000000,
            1100000,
```
**EN:** This callable implements `TokenizerMetricsCollector.__init__`. It takes `server_args`, `labels`, `bucket_time_to_first_token`, `bucket_inter_token_latency` and mainly initializes instance state and defaults. This chunk is part 1 of 4 for the same logical block. In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `TokenizerMetricsCollector.__init__`。它接收 `server_args`, `labels`, `bucket_time_to_first_token`, `bucket_inter_token_latency`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 1/4 部分。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 1382-1441: Method TokenizerMetricsCollector.__init__ (part 2/4)
```python
        ]
        self.prompt_tokens_histogram = Histogram(
            name="sglang:prompt_tokens_histogram",
            documentation="Histogram of prompt token length.",
            labelnames=labels.keys(),
            buckets=generate_buckets(
                server_args.prompt_tokens_buckets, default_bucket_prompt_tokens
            ),
        )
        self.uncached_prompt_tokens_histogram = Histogram(
            name="sglang:uncached_prompt_tokens_histogram",
            documentation="Histogram of uncached (compute) prompt token length.",
            labelnames=labels.keys(),
            buckets=generate_buckets(
                server_args.prompt_tokens_buckets, default_bucket_prompt_tokens
            ),
        )
        self.generation_tokens_histogram = Histogram(
            name="sglang:generation_tokens_histogram",
            documentation="Histogram of generation token length.",
            labelnames=labels.keys(),
            buckets=generate_buckets(
                server_args.generation_tokens_buckets,
                default_bucket_prompt_tokens,
            ),
        )

        self.cached_tokens_total = Counter(
            name="sglang:cached_tokens_total",
            documentation="Number of cached prompt tokens by source (device/host/storage).",
            labelnames=list(labels.keys()) + ["cache_source"],
        )

        self.num_requests_total = Counter(
            name="sglang:num_requests_total",
            documentation="Number of requests processed.",
            labelnames=labels.keys(),
        )

        self.get_loads_duration_seconds = Histogram(
            name="sglang:get_loads_duration_seconds",
            documentation="Time spent serving /v1/loads requests (seconds).",
            labelnames=labels.keys(),
            buckets=(0.0001, 0.0005, 0.001, 0.005, 0.01, 0.05, 0.1, 0.5, 1.0),
        )

        self.num_so_requests_total = Counter(
            name="sglang:num_so_requests_total",
            documentation="Number of structured output requests processed.",
            labelnames=labels.keys(),
        )

        self.num_aborted_requests_total = Counter(
            name="sglang:num_aborted_requests_total",
            documentation="Number of requests aborted.",
            labelnames=labels.keys(),
        )

        if bucket_time_to_first_token is None:
            bucket_time_to_first_token = [
```
**EN:** This callable implements `TokenizerMetricsCollector.__init__`. It takes `server_args`, `labels`, `bucket_time_to_first_token`, `bucket_inter_token_latency` and mainly initializes instance state and defaults. This chunk is part 2 of 4 for the same logical block.
**CN:** 这一可调用对象实现了 `TokenizerMetricsCollector.__init__`。它接收 `server_args`, `labels`, `bucket_time_to_first_token`, `bucket_inter_token_latency`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 2/4 部分。

### Lines 1442-1501: Method TokenizerMetricsCollector.__init__ (part 3/4)
```python
                0.1,
                0.2,
                0.4,
                0.6,
                0.8,
                1,
                2,
                4,
                6,
                8,
                10,
                20,
                40,
                60,
                80,
                100,
                200,
                400,
            ]

        if bucket_e2e_request_latency is None:
            bucket_e2e_request_latency = [
                0.1,
                0.2,
                0.4,
                0.6,
                0.8,
                1,
                2,
                4,
                6,
                8,
                10,
                20,
                40,
                60,
                80,
                100,
                200,
                400,
                600,
                1200,
                1800,
                2400,
            ]

        if bucket_inter_token_latency is None:
            bucket_inter_token_latency = [
                0.002,
                0.004,
                0.006,
                0.008,
                0.010,
                0.015,
                0.020,
                0.025,
                0.030,
                0.035,
                0.040,
                0.060,
```
**EN:** This callable implements `TokenizerMetricsCollector.__init__`. It takes `server_args`, `labels`, `bucket_time_to_first_token`, `bucket_inter_token_latency` and mainly initializes instance state and defaults. This chunk is part 3 of 4 for the same logical block.
**CN:** 这一可调用对象实现了 `TokenizerMetricsCollector.__init__`。它接收 `server_args`, `labels`, `bucket_time_to_first_token`, `bucket_inter_token_latency`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 3/4 部分。

### Lines 1502-1534: Method TokenizerMetricsCollector.__init__ (part 4/4)
```python
                0.080,
                0.100,
                0.200,
                0.400,
                0.600,
                0.800,
                1.000,
                2.000,
                4.000,
                6.000,
                8.000,
            ]

        self.histogram_time_to_first_token = Histogram(
            name="sglang:time_to_first_token_seconds",
            documentation="Histogram of time to first token in seconds.",
            labelnames=labels.keys(),
            buckets=bucket_time_to_first_token,
        )

        self.histogram_inter_token_latency = Histogram(
            name="sglang:inter_token_latency_seconds",
            documentation="Histogram of inter-token latency in seconds.",
            labelnames=labels.keys(),
            buckets=bucket_inter_token_latency,
        )

        self.histogram_e2e_request_latency = Histogram(
            name="sglang:e2e_request_latency_seconds",
            documentation="Histogram of End-to-end request latency in seconds",
            labelnames=labels.keys(),
            buckets=bucket_e2e_request_latency,
        )
```
**EN:** This callable implements `TokenizerMetricsCollector.__init__`. It takes `server_args`, `labels`, `bucket_time_to_first_token`, `bucket_inter_token_latency` and mainly initializes instance state and defaults. This chunk is part 4 of 4 for the same logical block.
**CN:** 这一可调用对象实现了 `TokenizerMetricsCollector.__init__`。它接收 `server_args`, `labels`, `bucket_time_to_first_token`, `bucket_inter_token_latency`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 4/4 部分。

### Lines 1536-1584: Method TokenizerMetricsCollector.observe_one_finished_request
```python
    def observe_one_finished_request(
        self,
        labels: Dict[str, str],
        prompt_tokens: int,
        generation_tokens: int,
        cached_tokens: int,
        e2e_latency: float,
        has_grammar: bool,
        cached_tokens_details: Optional[Dict[str, Any]] = None,
    ):
        self.prompt_tokens_total.labels(**labels).inc(prompt_tokens)
        self.generation_tokens_total.labels(**labels).inc(generation_tokens)

        # Report cached tokens with detailed source breakdown
        if cached_tokens > 0:
            if cached_tokens_details:
                # Report by cache source (device/host, and storage if L3 enabled)
                def report_cache_source(source: str, value: int):
                    if value > 0:
                        source_labels = {**labels, "cache_source": source}
                        self.cached_tokens_total.labels(**source_labels).inc(value)

                report_cache_source("device", cached_tokens_details.get("device", 0))
                report_cache_source("host", cached_tokens_details.get("host", 0))

                # Storage fields are only present when L3 storage backend is enabled
                if "storage" in cached_tokens_details:
                    storage_tokens = cached_tokens_details.get("storage", 0)
                    if storage_tokens > 0:
                        backend = (
                            cached_tokens_details.get("storage_backend") or "unknown"
                        )
                        report_cache_source(f"storage_{backend}", storage_tokens)
            else:
                # Fallback for backward compatibility
                labels_total = {**labels, "cache_source": "total"}
                self.cached_tokens_total.labels(**labels_total).inc(cached_tokens)

        self.num_requests_total.labels(**labels).inc(1)
        if has_grammar:
            self.num_so_requests_total.labels(**labels).inc(1)
        self.histogram_e2e_request_latency.labels(**labels).observe(float(e2e_latency))
        self.prompt_tokens_histogram.labels(**labels).observe(float(prompt_tokens))
        self.uncached_prompt_tokens_histogram.labels(**labels).observe(
            float(prompt_tokens - cached_tokens)
        )
        self.generation_tokens_histogram.labels(**labels).observe(
            float(generation_tokens)
        )
```
**EN:** This callable implements `TokenizerMetricsCollector.observe_one_finished_request`. It takes `labels`, `prompt_tokens`, `generation_tokens`, `cached_tokens` and mainly implements observe one finished request. In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `TokenizerMetricsCollector.observe_one_finished_request`。它接收 `labels`, `prompt_tokens`, `generation_tokens`, `cached_tokens`，主要用于实现 observe one finished request 相关逻辑。 在这一范围内，它会处理语法或正则约束。

### Lines 1586-1587: Method TokenizerMetricsCollector.observe_time_to_first_token
```python
    def observe_time_to_first_token(self, labels: Dict[str, str], value: float):
        self.histogram_time_to_first_token.labels(**labels).observe(value)
```
**EN:** This callable implements `TokenizerMetricsCollector.observe_time_to_first_token`. It takes `labels`, `value` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `TokenizerMetricsCollector.observe_time_to_first_token`。它接收 `labels`, `value`，主要用于将数据转换为另一种表示。

### Lines 1589-1600: Method TokenizerMetricsCollector.check_time_to_first_token_straggler
```python
    def check_time_to_first_token_straggler(self, value: float) -> bool:
        his = self.histogram_time_to_first_token.labels(**self.labels)
        total_observations = sum(bucket._value for bucket in his._buckets)
        if total_observations < 100:
            return False
        p99_threshold = total_observations * 0.99
        cumulative_count = 0
        for i, bucket in enumerate(his._buckets):
            cumulative_count += bucket._value
            if cumulative_count > p99_threshold:
                return value >= his._upper_bounds[i]
        return False
```
**EN:** This callable implements `TokenizerMetricsCollector.check_time_to_first_token_straggler`. It takes `value` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `TokenizerMetricsCollector.check_time_to_first_token_straggler`。它接收 `value`，主要用于将数据转换为另一种表示。

### Lines 1602-1615: Method TokenizerMetricsCollector.observe_inter_token_latency
```python
    def observe_inter_token_latency(
        self, labels: Dict[str, str], internval: float, num_new_tokens: int
    ):
        adjusted_interval = internval / num_new_tokens

        # A faster version of the Histogram::observe which observes multiple values at the same time.
        # reference: https://github.com/prometheus/client_python/blob/v0.21.1/prometheus_client/metrics.py#L639
        his = self.histogram_inter_token_latency.labels(**labels)
        his._sum.inc(internval)

        for i, bound in enumerate(his._upper_bounds):
            if adjusted_interval <= bound:
                his._buckets[i].inc(num_new_tokens)
                break
```
**EN:** This callable implements `TokenizerMetricsCollector.observe_inter_token_latency`. It takes `labels`, `internval`, `num_new_tokens` and mainly converts data into another representation. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TokenizerMetricsCollector.observe_inter_token_latency`。它接收 `labels`, `internval`, `num_new_tokens`，主要用于将数据转换为另一种表示。 在这一范围内，它会记录指标或追踪信号。

### Lines 1617-1618: Method TokenizerMetricsCollector.observe_one_aborted_request
```python
    def observe_one_aborted_request(self, labels: Dict[str, str]):
        self.num_aborted_requests_total.labels(**labels).inc(1)
```
**EN:** This callable implements `TokenizerMetricsCollector.observe_one_aborted_request`. It takes `labels` and mainly implements observe one aborted request.
**CN:** 这一可调用对象实现了 `TokenizerMetricsCollector.observe_one_aborted_request`。它接收 `labels`，主要用于实现 observe one aborted request 相关逻辑。

### Lines 1621-1626: Class StorageMetrics
```python
@dataclass
class StorageMetrics:
    prefetch_pgs: List[int] = field(default_factory=list)
    backup_pgs: List[int] = field(default_factory=list)
    prefetch_bandwidth: List[float] = field(default_factory=list)
    backup_bandwidth: List[float] = field(default_factory=list)
```
**EN:** This range introduces `StorageMetrics` and defines the structure or metadata that its methods rely on. In this range it records metrics or tracing signals.
**CN:** 这一段引入 `StorageMetrics`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 1629-1629: Class StorageMetricsCollector
```python
class StorageMetricsCollector:
```
**EN:** This range introduces `StorageMetricsCollector` and defines the structure or metadata that its methods rely on. In this range it records metrics or tracing signals.
**CN:** 这一段引入 `StorageMetricsCollector`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 1630-1689: Method StorageMetricsCollector.__init__ (part 1/2)
```python
    def __init__(
        self,
        labels: Dict[str, str],
    ):
        from prometheus_client import Counter, Histogram

        self.labels = labels

        self.prefetched_tokens_total = Counter(
            name="sglang:prefetched_tokens_total",
            documentation="Number of prefetched prompt tokens.",
            labelnames=labels.keys(),
        )

        self.backuped_tokens_total = Counter(
            name="sglang:backuped_tokens_total",
            documentation="Number of backuped tokens.",
            labelnames=labels.keys(),
        )

        bucket_io = [
            1,
            5,
            10,
            50,
            100,
        ]

        bucket_bandwidth = [
            0.1,
            0.5,
            1,
            5,
            10,
            50,
            100,
        ]

        self.histogram_prefetch_pgs = Histogram(
            name="sglang:prefetch_pgs",
            documentation="Histogram of prefetch pages of batches.",
            labelnames=labels.keys(),
            buckets=bucket_io,
        )

        self.histogram_backup_pgs = Histogram(
            name="sglang:backup_pgs",
            documentation="Histogram of backup pages of batches.",
            labelnames=labels.keys(),
            buckets=bucket_io,
        )

        self.histogram_prefetch_bandwidth = Histogram(
            name="sglang:prefetch_bandwidth",
            documentation="Histogram of prefetch bandwidth in GB/s.",
            labelnames=labels.keys(),
            buckets=bucket_bandwidth,
        )

        self.histogram_backup_bandwidth = Histogram(
```
**EN:** This callable implements `StorageMetricsCollector.__init__`. It takes `labels` and mainly initializes instance state and defaults. This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `StorageMetricsCollector.__init__`。它接收 `labels`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 1690-1694: Method StorageMetricsCollector.__init__ (part 2/2)
```python
            name="sglang:backup_bandwidth",
            documentation="Histogram of backup bandwidth in GB/s.",
            labelnames=labels.keys(),
            buckets=bucket_bandwidth,
        )
```
**EN:** This callable implements `StorageMetricsCollector.__init__`. It takes `labels` and mainly initializes instance state and defaults. This chunk is part 2 of 2 for the same logical block.
**CN:** 这一可调用对象实现了 `StorageMetricsCollector.__init__`。它接收 `labels`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 2/2 部分。

### Lines 1696-1698: Method StorageMetricsCollector.log_prefetched_tokens
```python
    def log_prefetched_tokens(self, prefetched_tokens: int):
        if prefetched_tokens > 0:
            self.prefetched_tokens_total.labels(**self.labels).inc(prefetched_tokens)
```
**EN:** This callable implements `StorageMetricsCollector.log_prefetched_tokens`. It takes `prefetched_tokens` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `StorageMetricsCollector.log_prefetched_tokens`。它接收 `prefetched_tokens`，主要用于将数据转换为另一种表示。

### Lines 1700-1702: Method StorageMetricsCollector.log_backuped_tokens
```python
    def log_backuped_tokens(self, backuped_tokens: int):
        if backuped_tokens > 0:
            self.backuped_tokens_total.labels(**self.labels).inc(backuped_tokens)
```
**EN:** This callable implements `StorageMetricsCollector.log_backuped_tokens`. It takes `backuped_tokens` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `StorageMetricsCollector.log_backuped_tokens`。它接收 `backuped_tokens`，主要用于将数据转换为另一种表示。

### Lines 1704-1705: Method StorageMetricsCollector._log_histogram
```python
    def _log_histogram(self, histogram, data: Union[int, float]):
        histogram.labels(**self.labels).observe(data)
```
**EN:** This callable implements `StorageMetricsCollector._log_histogram`. It takes `histogram`, `data` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `StorageMetricsCollector._log_histogram`。它接收 `histogram`, `data`，主要用于将数据转换为另一种表示。

### Lines 1707-1720: Method StorageMetricsCollector.log_storage_metrics
```python
    def log_storage_metrics(self, storage_metrics: Optional[StorageMetrics] = None):
        if storage_metrics is None:
            return

        assert isinstance(storage_metrics, StorageMetrics)

        for v in storage_metrics.prefetch_pgs:
            self._log_histogram(self.histogram_prefetch_pgs, v)
        for v in storage_metrics.backup_pgs:
            self._log_histogram(self.histogram_backup_pgs, v)
        for v in storage_metrics.prefetch_bandwidth:
            self._log_histogram(self.histogram_prefetch_bandwidth, v)
        for v in storage_metrics.backup_bandwidth:
            self._log_histogram(self.histogram_backup_bandwidth, v)
```
**EN:** This callable implements `StorageMetricsCollector.log_storage_metrics`. It takes `storage_metrics` and mainly converts data into another representation. In this range it performs defensive checks on invalid state; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `StorageMetricsCollector.log_storage_metrics`。它接收 `storage_metrics`，主要用于将数据转换为另一种表示。 在这一范围内，它会对非法状态执行防御性检查；记录指标或追踪信号。

### Lines 1723-1723: Class ExpertDispatchCollector
```python
class ExpertDispatchCollector:
```
**EN:** This range introduces `ExpertDispatchCollector` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `ExpertDispatchCollector`，并定义其后续方法依赖的结构或元数据。

### Lines 1724-1733: Method ExpertDispatchCollector.__init__
```python
    def __init__(self, ep_size: int) -> None:
        from prometheus_client import Histogram

        ep_size_buckets = [i for i in range(ep_size)]
        self.eplb_gpu_physical_count = Histogram(
            name="sglang:eplb_gpu_physical_count",
            documentation="The selected count of physical experts on each layer and GPU rank.",
            labelnames={"layer"},
            buckets=ep_size_buckets,
        )
```
**EN:** This callable implements `ExpertDispatchCollector.__init__`. It takes `ep_size` and mainly initializes instance state and defaults. In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `ExpertDispatchCollector.__init__`。它接收 `ep_size`，主要用于初始化实例状态与默认值。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 1736-1736: Class RadixCacheMetricsCollector
```python
class RadixCacheMetricsCollector:
```
**EN:** This range introduces `RadixCacheMetricsCollector` and defines the structure or metadata that its methods rely on. In this range it records metrics or tracing signals.
**CN:** 这一段引入 `RadixCacheMetricsCollector`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 1737-1796: Method RadixCacheMetricsCollector.__init__ (part 1/2)
```python
    def __init__(
        self,
        labels: Dict[str, str],
    ) -> None:
        # We need to import prometheus_client after setting the env variable `PROMETHEUS_MULTIPROC_DIR`
        from prometheus_client import Counter, Histogram

        self.labels = labels

        bucket_eviction_duration = get_histogram_conf_from_env(
            "SGLANG_BUCKET_EVICTION_DURATION"
        )
        if bucket_eviction_duration is None:
            bucket_eviction_duration = [
                0.001,
                0.002,
                0.003,
                0.004,
                0.005,
                0.006,
                0.007,
                0.008,
                0.009,
                0.01,
                0.02,
                0.03,
                0.04,
                0.05,
                0.1,
                0.2,
                0.5,
                1.0,
            ]
        bucket_load_back_duration = get_histogram_conf_from_env(
            "SGLANG_BUCKET_LOAD_BACK_DURATION"
        )
        if bucket_load_back_duration is None:
            bucket_load_back_duration = [
                0.001,
                0.002,
                0.003,
                0.004,
                0.005,
                0.006,
                0.007,
                0.008,
                0.009,
                0.01,
                0.02,
                0.03,
                0.04,
                0.05,
                0.1,
                0.2,
                0.5,
                1.0,
            ]
        self.eviction_duration_seconds = Histogram(
            name="sglang:eviction_duration_seconds",
            documentation="Time taken to evict memory from GPU to CPU in seconds.",
```
**EN:** This callable implements `RadixCacheMetricsCollector.__init__`. It takes `labels` and mainly initializes instance state and defaults. This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `RadixCacheMetricsCollector.__init__`。它接收 `labels`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 1797-1818: Method RadixCacheMetricsCollector.__init__ (part 2/2)
```python
            labelnames=labels.keys(),
            buckets=bucket_eviction_duration,
        )

        self.eviction_num_tokens = Counter(
            name="sglang:evicted_tokens_total",
            documentation="The number of tokens evicted from GPU to CPU.",
            labelnames=labels.keys(),
        )

        self.load_back_duration_seconds = Histogram(
            name="sglang:load_back_duration_seconds",
            documentation="Time taken to load memory from CPU to GPU in seconds.",
            labelnames=labels.keys(),
            buckets=bucket_load_back_duration,
        )

        self.load_back_num_tokens = Counter(
            name="sglang:load_back_tokens_total",
            documentation="The number of tokens loaded from CPU to GPU.",
            labelnames=labels.keys(),
        )
```
**EN:** This callable implements `RadixCacheMetricsCollector.__init__`. It takes `labels` and mainly initializes instance state and defaults. This chunk is part 2 of 2 for the same logical block. In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `RadixCacheMetricsCollector.__init__`。它接收 `labels`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 1820-1821: Method RadixCacheMetricsCollector.increment_eviction_num_tokens
```python
    def increment_eviction_num_tokens(self, num_tokens: int) -> None:
        self.eviction_num_tokens.labels(**self.labels).inc(num_tokens)
```
**EN:** This callable implements `RadixCacheMetricsCollector.increment_eviction_num_tokens`. It takes `num_tokens` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `RadixCacheMetricsCollector.increment_eviction_num_tokens`。它接收 `num_tokens`，主要用于将数据转换为另一种表示。

### Lines 1823-1824: Method RadixCacheMetricsCollector.increment_load_back_num_tokens
```python
    def increment_load_back_num_tokens(self, num_tokens: int) -> None:
        self.load_back_num_tokens.labels(**self.labels).inc(num_tokens)
```
**EN:** This callable implements `RadixCacheMetricsCollector.increment_load_back_num_tokens`. It takes `num_tokens` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `RadixCacheMetricsCollector.increment_load_back_num_tokens`。它接收 `num_tokens`，主要用于将数据转换为另一种表示。

### Lines 1826-1827: Method RadixCacheMetricsCollector.observe_eviction_duration
```python
    def observe_eviction_duration(self, duration_seconds: float) -> None:
        self.eviction_duration_seconds.labels(**self.labels).observe(duration_seconds)
```
**EN:** This callable implements `RadixCacheMetricsCollector.observe_eviction_duration`. It takes `duration_seconds` and mainly implements observe eviction duration.
**CN:** 这一可调用对象实现了 `RadixCacheMetricsCollector.observe_eviction_duration`。它接收 `duration_seconds`，主要用于实现 observe eviction duration 相关逻辑。

### Lines 1829-1830: Method RadixCacheMetricsCollector.observe_load_back_duration
```python
    def observe_load_back_duration(self, duration_seconds: float) -> None:
        self.load_back_duration_seconds.labels(**self.labels).observe(duration_seconds)
```
**EN:** This callable implements `RadixCacheMetricsCollector.observe_load_back_duration`. It takes `duration_seconds` and mainly loads external data or weights.
**CN:** 这一可调用对象实现了 `RadixCacheMetricsCollector.observe_load_back_duration`。它接收 `duration_seconds`，主要用于加载外部数据或权重。

### Lines 1833-1844: Function get_histogram_conf_from_env
```python
def get_histogram_conf_from_env(env_var_name: str) -> Optional[List[float]]:
    """
    Get the histogram configuration from the environment variable.
    env value should be like "0.1,0.2,0.5,1,2"
    """
    if env_var_name not in os.environ:
        return None
    # if the env var is not set or empty, return None
    env_var_value = os.environ[env_var_name]
    if not env_var_value:
        return None
    return [float(x) for x in env_var_value.split(",")]
```
**EN:** This callable implements `get_histogram_conf_from_env`. It takes `env_var_name` and mainly constructs data from an external representation. The docstring states: "Get the histogram configuration from the environment variable." In this range it sets up imports and shared symbols; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `get_histogram_conf_from_env`。它接收 `env_var_name`，主要用于从外部表示构造数据。 在这一范围内，它会建立导入关系并准备共享符号；读取环境变量驱动的配置。

## Key Concepts / 关键概念
- `SGLANG_TEST_REQUEST_TIME_STATS`: module constant or capability flag / 模块常量或能力标记
- `QueueCount`: core class or state container / 核心类或状态容器
- `SchedulerStats`: core class or state container / 核心类或状态容器
- `ROUTING_KEY_REQ_COUNT_BUCKET_BOUNDS`: module constant or capability flag / 模块常量或能力标记
- `compute_routing_key_stats`: implements compute routing key stats / 实现 compute routing key stats 相关逻辑
- `DPCooperationInfo`: core class or state container / 核心类或状态容器
- `SchedulerMetricsCollectorContext`: core class or state container / 核心类或状态容器
- `SchedulerMetricsCollector`: core class or state container / 核心类或状态容器
- `TokenizerMetricsCollector`: core class or state container / 核心类或状态容器
- `StorageMetrics`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`, `logging`, `os`, `time`, `collections`, `typing`
- **Third-party / 第三方**: `prometheus_client`
- **Internal modules / 内部模块**: `sglang.srt.disaggregation.utils`, `sglang.srt.environ`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.observability.utils`, `sglang.srt.server_args`, `sglang.srt.utils`, `sglang.srt.utils.gauge_histogram`, `sglang.srt.managers.schedule_batch`
