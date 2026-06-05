# load_inquirer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/scheduler_components/load_inquirer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements load inquirer logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 load inquirer 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-6: Import runtime dependencies / 导入运行时依赖
```python
import logging
import time
from dataclasses import dataclass
from typing import TYPE_CHECKING, Callable
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 8-17: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.disaggregation.utils import DisaggregationMode
from sglang.srt.managers.io_struct import (
    DisaggregationMetrics,
    GetLoadsReqInput,
    GetLoadsReqOutput,
    LoRAMetrics,
    MemoryMetrics,
    QueueMetrics,
    SpeculativeMetrics,
)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 19-27: Provide supporting module logic / 提供辅助模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.distributed.parallel_state_wrapper import ParallelState
    from sglang.srt.managers.scheduler_components.pool_stats_observer import (
        SchedulerPoolStatsObserver,
    )
    from sglang.srt.managers.tp_worker import BaseTpWorker
    from sglang.srt.mem_cache.allocator import BaseTokenToKVPoolAllocator
    from sglang.srt.server_args import ServerArgs
    from sglang.srt.speculative.spec_info import SpeculativeAlgorithm
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 30-33: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)


@dataclass(kw_only=True, slots=True, frozen=True)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 34-209: Define class SchedulerLoadInquirer / 定义类 SchedulerLoadInquirer
```python
class SchedulerLoadInquirer:
    disaggregation_mode: "DisaggregationMode"
    ps: "ParallelState"
    server_args: "ServerArgs"
    max_total_num_tokens: int
    max_running_requests: int
    pool_stats_observer: "SchedulerPoolStatsObserver"
    tp_worker: "BaseTpWorker"
    token_to_kv_pool_allocator: "BaseTokenToKVPoolAllocator"
    spec_algorithm: "SpeculativeAlgorithm"
    get_running_batch: Callable
    get_waiting_queue: Callable
    get_stats: Callable
    get_chunked_req: Callable
    get_disagg_prefill_bootstrap_queue: Callable
    get_disagg_prefill_inflight_queue: Callable
    get_disagg_decode_prealloc_queue: Callable
    get_disagg_decode_transfer_queue: Callable
    get_spec_total_num_accept_tokens: Callable
    get_spec_total_num_forward_ct: Callable

    def _get_num_pending_tokens(self, chunk_deduct: int = 0) -> int:
        """Get the total number of tokens pending prefill.

        This includes tokens from waiting queue requests plus remaining tokens
        from the currently chunked request.

        Args:
            chunk_deduct: extra tokens to subtract from the chunked request's
                remaining count. At batch-scheduling time the current chunk
                has been planned but ``prefix_indices`` does not yet include it,
                so callers pass ``extend_input_len`` here. At load-reporting
                time ``prefix_indices`` is already up-to-date, so the default
                0 is correct.
        """
        num_pending_tokens = sum(req.seqlen for req in self.get_waiting_queue())
        if self.get_chunked_req() is not None:
            req = self.get_chunked_req()
            num_pending_tokens += req.seqlen - len(req.prefix_indices) - chunk_deduct
        return num_pending_tokens

    def get_loads(self, req: GetLoadsReqInput = None) -> GetLoadsReqOutput:
        """
        Get comprehensive load metrics for /v1/loads endpoint.

        Args:
            req: Request containing include list and optional dp_rank filter

        Returns:
            GetLoadsReqOutput with core metrics and optional detailed sections
        """
        if req is None:
            req = GetLoadsReqInput()

        include = set(req.include) if req.include else {"core"}
        include_all = "all" in include

        num_running_reqs = len(self.get_running_batch().reqs)

        waiting_queues = [self.get_waiting_queue()]
        if self.disaggregation_mode == DisaggregationMode.PREFILL:
            waiting_queues.append(self.get_disagg_prefill_bootstrap_queue().queue)
        elif self.disaggregation_mode == DisaggregationMode.DECODE:
            waiting_queues.append(self.get_disagg_decode_prealloc_queue().queue)
            waiting_queues.append(self.get_disagg_decode_transfer_queue().queue)
            waiting_queues.append(
                self.get_disagg_decode_prealloc_queue().retracted_queue
            )

        num_waiting_reqs = sum(len(queue) for queue in waiting_queues)
        num_used_tokens, kv_token_usage = (
            self.pool_stats_observer.get_pool_stats().get_kv_token_stats()
        )
        num_total_tokens = num_used_tokens + sum(
            req.seqlen for queue in waiting_queues for req in queue
        )

        memory = None
        if include_all or "memory" in include:
            try:
                memory = MemoryMetrics(
                    weight_gb=round(
                        self.tp_worker.model_runner.weight_load_mem_usage, 3
                    ),
                    kv_cache_gb=round(
                        self.token_to_kv_pool_allocator.get_kvcache().mem_usage, 3
                    ),
                    graph_gb=round(self.tp_worker.model_runner.graph_mem_usage, 3),
                    token_capacity=int(self.max_total_num_tokens),
                )
            except AttributeError as e:
                logger.debug(f"Memory metrics not available: {e}")

        speculative = None
        if include_all or "spec" in include:
            if (
                not self.spec_algorithm.is_none()
                and self.get_spec_total_num_forward_ct() > 0
            ):
                speculative = SpeculativeMetrics(
                    accept_length=(
                        self.get_spec_total_num_accept_tokens()
                        / self.get_spec_total_num_forward_ct()
                    ),
                    accept_rate=self.get_stats().spec_accept_rate,
                )

        lora = None
        if include_all or "lora" in include:
            if self.server_args.enable_lora:
                lora = LoRAMetrics(
                    slots_used=self.get_stats().lora_pool_slots_used,
                    slots_total=self.get_stats().lora_pool_slots_total,
                    utilization=self.get_stats().lora_pool_utilization,
                )

        disaggregation = None
        if include_all or "disagg" in include:
            mode_str = "null"
            prefill_bootstrap = 0
            prefill_inflight = 0
            decode_prealloc = 0
            decode_transfer = 0
            decode_retracted = 0

            if self.disaggregation_mode == DisaggregationMode.PREFILL:
                mode_str = "prefill"
                prefill_bootstrap = len(self.get_disagg_prefill_bootstrap_queue().queue)
                prefill_inflight = len(self.get_disagg_prefill_inflight_queue())
            elif self.disaggregation_mode == DisaggregationMode.DECODE:
                mode_str = "decode"
                decode_prealloc = len(self.get_disagg_decode_prealloc_queue().queue)
                decode_transfer = len(self.get_disagg_decode_transfer_queue().queue)
                decode_retracted = len(
                    self.get_disagg_decode_prealloc_queue().retracted_queue
                )

            disaggregation = DisaggregationMetrics(
                mode=mode_str,
                prefill_bootstrap_queue_reqs=prefill_bootstrap,
                prefill_inflight_queue_reqs=prefill_inflight,
                decode_prealloc_queue_reqs=decode_prealloc,
                decode_transfer_queue_reqs=decode_transfer,
                decode_retracted_queue_reqs=decode_retracted,
                kv_transfer_speed_gb_s=self.get_stats().kv_transfer_speed_gb_s,
                kv_transfer_latency_ms=self.get_stats().kv_transfer_latency_ms,
            )

        queues = None
        if include_all or "queues" in include:
            queues = QueueMetrics(
                waiting=len(self.get_waiting_queue()),
                grammar=self.get_stats().num_grammar_queue_reqs,
                paused=self.get_stats().num_paused_reqs,
                retracted=self.get_stats().num_retracted_reqs,
            )

        return GetLoadsReqOutput(
            dp_rank=self.ps.dp_rank,
            timestamp=time.time(),
            num_running_reqs=num_running_reqs,
            num_waiting_reqs=num_waiting_reqs,
            num_used_tokens=num_used_tokens,
            num_total_tokens=num_total_tokens,
            max_total_num_tokens=self.max_total_num_tokens,
            token_usage=round(kv_token_usage, 4),
            gen_throughput=round(self.get_stats().gen_throughput, 2),
            cache_hit_rate=round(self.get_stats().cache_hit_rate, 4),
            utilization=round(self.get_stats().utilization, 4),
            max_running_requests=self.max_running_requests,
            memory=memory,
            speculative=speculative,
            lora=lora,
            disaggregation=disaggregation,
            queues=queues,
        )
```
**EN:** This block declares the class `SchedulerLoadInquirer`. It centers on coordinating load inquirer behavior, with methods such as _get_num_pending_tokens, get_loads.
**CN:** 该代码块声明类 `SchedulerLoadInquirer`。它负责承载与 load inquirer 相关的核心状态与行为，并通过 _get_num_pending_tokens, get_loads 等方法组织实现。

## Key Concepts / 关键概念
- **Core types / 核心类型**: SchedulerLoadInquirer
- **Domain focus / 领域焦点**: load inquirer / load inquirer
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: dataclasses, logging, time, typing
- **Third-party / 第三方库**: __future__
- **Local Modules / 本地模块**: sglang.srt.disaggregation.utils, sglang.srt.distributed.parallel_state_wrapper, sglang.srt.managers.io_struct, sglang.srt.managers.scheduler_components.pool_stats_observer, sglang.srt.managers.tp_worker, sglang.srt.mem_cache.allocator, sglang.srt.server_args, sglang.srt.speculative.spec_info
