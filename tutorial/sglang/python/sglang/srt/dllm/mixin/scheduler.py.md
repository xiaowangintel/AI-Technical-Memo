# scheduler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/dllm/mixin/scheduler.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed LLM coordination part of the SRT runtime and implements logic centered on `scheduler`. It exposes primary entry points such as `SchedulerDllmMixin`, `DllmManager`. / 该模块属于 SRT 运行时的分布式 LLM 协调部分，主要实现围绕 `scheduler` 的逻辑。 它对外提供的主要入口包括 `SchedulerDllmMixin`, `DllmManager`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Module imports, constants, and setup
```python
from __future__ import annotations

import logging
from typing import TYPE_CHECKING, List, Optional, Set, Union

from sglang.srt.dllm.config import DllmConfig
from sglang.srt.dllm.mixin.req import DllmReqPhase
from sglang.srt.managers.schedule_batch import Req, ScheduleBatch
from sglang.srt.managers.schedule_policy import AddReqResult, PrefillAdder
from sglang.srt.mem_cache.common import release_kv_cache
from sglang.srt.model_executor.forward_batch_info import ForwardMode
from sglang.srt.observability.req_time_stats import set_time_batch

logger = logging.getLogger(__name__)

if TYPE_CHECKING:
    from sglang.srt.managers.scheduler import GenerationBatchResult, Scheduler


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断。

### Lines 20-20: Class SchedulerDllmMixin
```python
class SchedulerDllmMixin:
```
**EN:** This range introduces `SchedulerDllmMixin` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `SchedulerDllmMixin`，并定义其后续方法依赖的结构或元数据。

### Lines 21-27: Method SchedulerDllmMixin.init_diffusion_llm
```python
    def init_diffusion_llm(self: Scheduler):
        self.dllm_config = (
            DllmConfig.from_server_args(self.server_args)
            if self.server_args.dllm_algorithm is not None
            else None
        )
        self.dllm_manager = DllmManager(dllm_config=self.dllm_config)
```
**EN:** This callable implements `SchedulerDllmMixin.init_diffusion_llm` and mainly implements init diffusion llm.
**CN:** 这一可调用对象实现了 `SchedulerDllmMixin.init_diffusion_llm`，主要用于实现 init diffusion llm 相关逻辑。

### Lines 29-61: Method SchedulerDllmMixin.get_new_batch_dllm
```python
    def get_new_batch_dllm(self: Scheduler) -> Optional[ScheduleBatch]:
        """Generate a new batch for DLLM (Diffusion LLM) scheduling."""
        if self.enable_priority_preemption:
            self.running_batch.batch_is_full = False

        # Early exit if batch is full or no requests available
        if self._should_skip_prefill():
            return None

        running_bs = len(self.running_batch.reqs)
        self.policy.calc_priority(self.waiting_queue)

        # Create prefill adder with resource constraints
        adder = self._create_dllm_prefill_adder(running_bs)

        # Initialize DLLM manager and transfer requests
        self.dllm_manager.init_next_round()
        self._fetch_waiting_reqs()

        # Process batches
        forward_mode = self._process_dllm_batches(adder)

        can_run_list = adder.can_run_list
        if not can_run_list:
            return None

        # Record metrics and update state
        set_time_batch(can_run_list, "set_forward_entry_time")
        self._update_state_for_batch(can_run_list, adder, running_bs)

        # Create and prepare batch
        new_batch = self._create_dllm_batch(can_run_list, forward_mode)
        return new_batch
```
**EN:** This callable implements `SchedulerDllmMixin.get_new_batch_dllm` and mainly retrieves a value or derived view. The docstring states: "Generate a new batch for DLLM (Diffusion LLM) scheduling." In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerDllmMixin.get_new_batch_dllm`，主要用于获取某个值或派生视图。 在这一范围内，它会记录指标或追踪信号。

### Lines 63-101: Method SchedulerDllmMixin.process_batch_result_dllm
```python
    def process_batch_result_dllm(
        self: Scheduler,
        batch: ScheduleBatch,
        result: GenerationBatchResult,
    ):
        if result.copy_done is not None:
            result.copy_done.synchronize()

        if result.next_token_ids:
            self.token_to_kv_pool_allocator.free_group_begin()

            for idx in range(batch.batch_size()):
                req = batch.reqs[idx]

                next_token_ids = result.next_token_ids[idx].tolist()
                new_tokens = len(next_token_ids)
                if new_tokens == 0:
                    continue

                req.fill_ids[-new_tokens:] = next_token_ids[:]
                self.metrics_reporter.num_generated_tokens += new_tokens

                req.output_ids.extend(next_token_ids)
                req.check_finished(new_accepted_len=new_tokens)

                if req.finished():
                    release_kv_cache(req, self.tree_cache)
                    req.time_stats.set_completion_time()

            self.output_streamer.stream_output(batch.reqs, batch.return_logprob)
            self.token_to_kv_pool_allocator.free_group_end()

        can_run_cuda_graph = getattr(result, "can_run_cuda_graph", False)
        self.metrics_reporter.report_prefill_stats(
            batch=batch,
            prefill_stats=batch.prefill_stats,
            can_run_cuda_graph=can_run_cuda_graph,
            dp_cooperation_info=batch.dp_cooperation_info,
        )
```
**EN:** This callable implements `SchedulerDllmMixin.process_batch_result_dllm`. It takes `batch`, `result` and mainly implements process batch result dllm. In this range it records metrics or tracing signals; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `SchedulerDllmMixin.process_batch_result_dllm`。它接收 `batch`, `result`，主要用于实现 process batch result dllm 相关逻辑。 在这一范围内，它会记录指标或追踪信号；管理图捕获或回放逻辑。

### Lines 103-113: Method SchedulerDllmMixin._fetch_waiting_reqs
```python
    def _fetch_waiting_reqs(self: Scheduler):
        # Calculate how many requests can be added to DLLM manager
        max_dllm_capacity = self.dllm_config.max_running_requests - len(
            self.dllm_manager.waiting_queue
        )
        num_requests_to_add = min(max_dllm_capacity, len(self.waiting_queue))

        if num_requests_to_add > 0:
            requests_to_add = self.waiting_queue[:num_requests_to_add]
            self.dllm_manager.add_waiting_reqs(requests_to_add)
            self.waiting_queue = self.waiting_queue[num_requests_to_add:]
```
**EN:** This callable implements `SchedulerDllmMixin._fetch_waiting_reqs` and mainly implements fetch waiting reqs.
**CN:** 这一可调用对象实现了 `SchedulerDllmMixin._fetch_waiting_reqs`，主要用于实现 fetch waiting reqs 相关逻辑。

### Lines 115-131: Method SchedulerDllmMixin._should_skip_prefill
```python
    def _should_skip_prefill(self: Scheduler) -> bool:
        """Check if DLLM prefill should be skipped."""
        if (
            self.running_batch.batch_is_full or not self.waiting_queue
        ) and self.dllm_manager.is_empty():
            return True

        running_bs = len(self.running_batch.reqs)
        if (
            self.get_num_allocatable_reqs(running_bs) <= 0
            and self.dllm_manager.is_empty()
            and not self.enable_priority_preemption
        ):
            self.running_batch.batch_is_full = True
            return True

        return False
```
**EN:** This callable implements `SchedulerDllmMixin._should_skip_prefill` and mainly implements should skip prefill. The docstring states: "Check if DLLM prefill should be skipped."
**CN:** 这一可调用对象实现了 `SchedulerDllmMixin._should_skip_prefill`，主要用于实现 should skip prefill 相关逻辑。

### Lines 133-147: Method SchedulerDllmMixin._create_dllm_prefill_adder
```python
    def _create_dllm_prefill_adder(self: Scheduler, running_bs: int) -> PrefillAdder:
        """Create a prefill adder configured for DLLM scheduling."""
        return PrefillAdder(
            self.page_size,
            self.tree_cache,
            self.token_to_kv_pool_allocator,
            self.running_batch,
            self.new_token_ratio,
            self.max_prefill_tokens,
            self.chunked_prefill_size,
            running_bs if self.is_mixed_chunk else 0,
            self.priority_scheduling_preemption_threshold,
            prefill_max_requests=self.server_args.prefill_max_requests,
            dllm_config=self.dllm_config,
        )
```
**EN:** This callable implements `SchedulerDllmMixin._create_dllm_prefill_adder`. It takes `running_bs` and mainly adds configuration entries or arguments. The docstring states: "Create a prefill adder configured for DLLM scheduling."
**CN:** 这一可调用对象实现了 `SchedulerDllmMixin._create_dllm_prefill_adder`。它接收 `running_bs`，主要用于添加配置项或参数。

### Lines 149-172: Method SchedulerDllmMixin._process_dllm_batches
```python
    def _process_dllm_batches(self: Scheduler, adder: PrefillAdder) -> ForwardMode:
        """Process prefill or decode batches for DLLM."""
        forward_mode = ForwardMode.DLLM_EXTEND

        # Try prefill batch first
        prefill_reqs = self.dllm_manager.get_prefill_requests()
        if prefill_reqs:
            self._process_batch_by_phase(
                adder,
                prefill_reqs,
                DllmReqPhase.STAGING_PREFILL,
                DllmReqPhase.INCOMING_PREFILL,
            )
        else:
            # Fall back to decode batch
            decode_reqs = self.dllm_manager.get_decode_requests()
            self._process_batch_by_phase(
                adder,
                decode_reqs,
                DllmReqPhase.STAGING_DECODE,
                DllmReqPhase.INCOMING_DECODE,
            )

        return forward_mode
```
**EN:** This callable implements `SchedulerDllmMixin._process_dllm_batches`. It takes `adder` and mainly implements process dllm batches. The docstring states: "Process prefill or decode batches for DLLM."
**CN:** 这一可调用对象实现了 `SchedulerDllmMixin._process_dllm_batches`。它接收 `adder`，主要用于实现 process dllm batches 相关逻辑。

### Lines 174-190: Method SchedulerDllmMixin._process_batch_by_phase
```python
    def _process_batch_by_phase(
        self,
        adder: PrefillAdder,
        batch: List[Req],
        staging_phase: DllmReqPhase,
        incoming_phase: DllmReqPhase,
    ) -> None:
        """Process a batch, separating staging and incoming requests."""
        staging_reqs = [req for req in batch if req.dllm_phase == staging_phase]
        if staging_reqs:
            staging_result = self.process_dllm_staging_reqs(adder, staging_reqs)
            if staging_result != AddReqResult.CONTINUE:
                return

        incoming_reqs = [req for req in batch if req.dllm_phase == incoming_phase]
        if incoming_reqs:
            self.process_dllm_incoming_reqs(adder, incoming_reqs)
```
**EN:** This callable implements `SchedulerDllmMixin._process_batch_by_phase`. It takes `adder`, `batch`, `staging_phase`, `incoming_phase` and mainly implements process batch by phase. The docstring states: "Process a batch, separating staging and incoming requests."
**CN:** 这一可调用对象实现了 `SchedulerDllmMixin._process_batch_by_phase`。它接收 `adder`, `batch`, `staging_phase`, `incoming_phase`，主要用于实现 process batch by phase 相关逻辑。

### Lines 192-207: Method SchedulerDllmMixin._update_state_for_batch
```python
    def _update_state_for_batch(
        self: Scheduler, can_run_list: List[Req], adder: PrefillAdder, running_bs: int
    ) -> None:
        """Update state for the batch."""

        if adder.preempt_list:
            for req in adder.preempt_list:
                self._add_request_to_queue(req)

        if can_run_list:
            self.dllm_manager.add_staging_reqs(can_run_list)
            self.dllm_manager.increment_chunked_count()

        self.adder = adder
        self.can_run_list = can_run_list
        self.running_bs = len(self.running_batch.reqs)
```
**EN:** This callable implements `SchedulerDllmMixin._update_state_for_batch`. It takes `can_run_list`, `adder`, `running_bs` and mainly updates existing runtime state. The docstring states: "Update state for the batch."
**CN:** 这一可调用对象实现了 `SchedulerDllmMixin._update_state_for_batch`。它接收 `can_run_list`, `adder`, `running_bs`，主要用于更新现有运行时状态。

### Lines 209-236: Method SchedulerDllmMixin._create_dllm_batch
```python
    def _create_dllm_batch(
        self: Scheduler, can_run_list: List[Req], forward_mode: ForwardMode
    ) -> ScheduleBatch:
        """Create and prepare a new DLLM batch."""
        new_batch = ScheduleBatch.init_new(
            can_run_list,
            self.req_to_token_pool,
            self.token_to_kv_pool_allocator,
            self.tree_cache,
            self.model_config,
            self.enable_overlap,
            self.spec_algorithm,
            dllm_config=self.dllm_config,
        )
        new_batch.prepare_for_extend()
        new_batch.forward_mode = forward_mode
        new_batch.decoding_reqs = None

        # Record prefill stats for logging after forward
        from sglang.srt.managers.scheduler_components.metrics_reporter import (
            PrefillStats,
        )

        new_batch.prefill_stats = PrefillStats.from_adder(
            self.adder, self.running_batch.reqs, self.enable_priority_scheduling
        )

        return new_batch
```
**EN:** This callable implements `SchedulerDllmMixin._create_dllm_batch`. It takes `can_run_list`, `forward_mode` and mainly constructs new objects or contexts. The docstring states: "Create and prepare a new DLLM batch." In this range it sets up imports and shared symbols; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerDllmMixin._create_dllm_batch`。它接收 `can_run_list`, `forward_mode`，主要用于构造新的对象或上下文。 在这一范围内，它会建立导入关系并准备共享符号；记录指标或追踪信号。

### Lines 238-270: Method SchedulerDllmMixin.process_dllm_incoming_reqs
```python
    def process_dllm_incoming_reqs(
        self: Scheduler, adder: PrefillAdder, reqs: List[Req]
    ) -> AddReqResult:
        """Process incoming DLLM requests with resource allocation and preemption."""
        res = AddReqResult.CONTINUE
        for req in reqs:
            # Check if batch is full
            running_bs = len(self.running_batch.reqs)
            if len(adder.can_run_list) >= self.get_num_allocatable_reqs(running_bs):
                self.running_batch.batch_is_full = True

            # Try preemption if batch is full
            if self.running_batch.batch_is_full:
                if (
                    not self.enable_priority_preemption
                    or not adder.preempt_to_schedule(req, self.server_args)
                ):
                    break

            # Prepare and add request
            req.init_next_round_input(self.tree_cache)
            res = adder.add_one_req(
                req,
                has_chunked_req=True,
                truncation_align_size=self.truncation_align_size,
            )

            if res != AddReqResult.CONTINUE:
                if res == AddReqResult.NO_TOKEN:
                    self.running_batch.batch_is_full = True
                break

        return res
```
**EN:** This callable implements `SchedulerDllmMixin.process_dllm_incoming_reqs`. It takes `adder`, `reqs` and mainly implements process dllm incoming reqs. The docstring states: "Process incoming DLLM requests with resource allocation and preemption."
**CN:** 这一可调用对象实现了 `SchedulerDllmMixin.process_dllm_incoming_reqs`。它接收 `adder`, `reqs`，主要用于实现 process dllm incoming reqs 相关逻辑。

### Lines 272-281: Method SchedulerDllmMixin.process_dllm_staging_reqs
```python
    def process_dllm_staging_reqs(
        self: Scheduler, adder: PrefillAdder, reqs: List[Req]
    ) -> AddReqResult:
        """Process staging DLLM requests with resource allocation."""
        for req in reqs:
            res = adder.add_dllm_staging_req(req)
            if res == AddReqResult.NO_TOKEN:
                return res

        return AddReqResult.CONTINUE
```
**EN:** This callable implements `SchedulerDllmMixin.process_dllm_staging_reqs`. It takes `adder`, `reqs` and mainly implements process dllm staging reqs. The docstring states: "Process staging DLLM requests with resource allocation."
**CN:** 这一可调用对象实现了 `SchedulerDllmMixin.process_dllm_staging_reqs`。它接收 `adder`, `reqs`，主要用于实现 process dllm staging reqs 相关逻辑。

### Lines 284-292: Class DllmManager
```python
class DllmManager:
    """
    Manager for Diffusion LLM request scheduling.

    Maintains two queues:
    - waiting_queue: The requests waiting to be scheduled with max running requests limit
    - staging_queue: Requests allocated resources by PrefillAdder
    """

```
**EN:** This range introduces `DllmManager` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Manager for Diffusion LLM request scheduling."
**CN:** 这一段引入 `DllmManager`，并定义其后续方法依赖的结构或元数据。

### Lines 293-299: Method DllmManager.__init__
```python
    def __init__(self, dllm_config: Optional[DllmConfig] = None):
        self.dllm_config = dllm_config
        self.max_running_reqs = (
            dllm_config.max_running_requests if dllm_config is not None else 1
        )
        self.waiting_queue: List[Req] = []
        self.staging_queue: List[Req] = []
```
**EN:** This callable implements `DllmManager.__init__`. It takes `dllm_config` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `DllmManager.__init__`。它接收 `dllm_config`，主要用于初始化实例状态与默认值。

### Lines 301-303: Method DllmManager.get_prefill_requests
```python
    def get_prefill_requests(self) -> List[Req]:
        """Get all prefill requests from waiting queue."""
        return [req for req in self.waiting_queue if req.is_dllm_prefill()]
```
**EN:** This callable implements `DllmManager.get_prefill_requests` and mainly retrieves a value or derived view. The docstring states: "Get all prefill requests from waiting queue." In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `DllmManager.get_prefill_requests`，主要用于获取某个值或派生视图。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 305-307: Method DllmManager.get_decode_requests
```python
    def get_decode_requests(self) -> List[Req]:
        """Get all decode requests from waiting queue."""
        return [req for req in self.waiting_queue if not req.is_dllm_prefill()]
```
**EN:** This callable implements `DllmManager.get_decode_requests` and mainly retrieves a value or derived view. The docstring states: "Get all decode requests from waiting queue." In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `DllmManager.get_decode_requests`，主要用于获取某个值或派生视图。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 309-319: Method DllmManager.add_waiting_reqs
```python
    def add_waiting_reqs(self, reqs: Union[Req, List[Req]]) -> None:
        """Add requests to waiting queue with redundancy check."""
        assert self.dllm_config is not None, "Diffusion LLM config is not set."

        reqs_to_add = reqs if isinstance(reqs, list) else [reqs]

        # Check for duplicate request IDs
        if self._has_duplicate_reqs(reqs_to_add):
            raise RuntimeError("Redundant requests detected in dLLM requests.")

        self.waiting_queue.extend(reqs_to_add)
```
**EN:** This callable implements `DllmManager.add_waiting_reqs`. It takes `reqs` and mainly adds configuration entries or arguments. The docstring states: "Add requests to waiting queue with redundancy check." In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `DllmManager.add_waiting_reqs`。它接收 `reqs`，主要用于添加配置项或参数。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 321-324: Method DllmManager.add_staging_reqs
```python
    def add_staging_reqs(self, reqs: Union[Req, List[Req]]) -> None:
        """Add requests to staging queue (allocated by PrefillAdder)."""
        reqs_to_add = reqs if isinstance(reqs, list) else [reqs]
        self.staging_queue.extend(reqs_to_add)
```
**EN:** This callable implements `DllmManager.add_staging_reqs`. It takes `reqs` and mainly adds configuration entries or arguments. The docstring states: "Add requests to staging queue (allocated by PrefillAdder)."
**CN:** 这一可调用对象实现了 `DllmManager.add_staging_reqs`。它接收 `reqs`，主要用于添加配置项或参数。

### Lines 326-329: Method DllmManager._has_duplicate_reqs
```python
    def _has_duplicate_reqs(self, reqs: List[Req]) -> bool:
        """Check if any request ID already exists in waiting queue."""
        existing_rids: Set[str] = {r.rid for r in self.waiting_queue}
        return any(req.rid in existing_rids for req in reqs)
```
**EN:** This callable implements `DllmManager._has_duplicate_reqs`. It takes `reqs` and mainly implements has duplicate reqs. The docstring states: "Check if any request ID already exists in waiting queue."
**CN:** 这一可调用对象实现了 `DllmManager._has_duplicate_reqs`。它接收 `reqs`，主要用于实现 has duplicate reqs 相关逻辑。

### Lines 331-333: Method DllmManager.any_staging_reqs
```python
    def any_staging_reqs(self) -> bool:
        """Check if there are requests in staging queue."""
        return self.dllm_config is not None and len(self.staging_queue) > 0
```
**EN:** This callable implements `DllmManager.any_staging_reqs` and mainly implements any staging reqs. The docstring states: "Check if there are requests in staging queue."
**CN:** 这一可调用对象实现了 `DllmManager.any_staging_reqs`，主要用于实现 any staging reqs 相关逻辑。

### Lines 335-339: Method DllmManager.is_empty
```python
    def is_empty(self) -> bool:
        """Check if both queues are empty or DLLM is not configured."""
        if self.dllm_config is None:
            return True
        return len(self.waiting_queue) == 0
```
**EN:** This callable implements `DllmManager.is_empty` and mainly implements is empty. The docstring states: "Check if both queues are empty or DLLM is not configured."
**CN:** 这一可调用对象实现了 `DllmManager.is_empty`，主要用于实现 is empty 相关逻辑。

### Lines 341-344: Method DllmManager.increment_chunked_count
```python
    def increment_chunked_count(self) -> None:
        """Increment chunked count for all staging requests."""
        for req in self.staging_queue:
            req.is_chunked += 1
```
**EN:** This callable implements `DllmManager.increment_chunked_count` and mainly implements increment chunked count. The docstring states: "Increment chunked count for all staging requests."
**CN:** 这一可调用对象实现了 `DllmManager.increment_chunked_count`，主要用于实现 increment chunked count 相关逻辑。

### Lines 346-349: Method DllmManager.filter_finished_reqs
```python
    def filter_finished_reqs(self) -> None:
        """Remove finished requests from both queues."""
        self.waiting_queue = [req for req in self.waiting_queue if not req.finished()]
        self.staging_queue = [req for req in self.staging_queue if not req.finished()]
```
**EN:** This callable implements `DllmManager.filter_finished_reqs` and mainly implements filter finished reqs. The docstring states: "Remove finished requests from both queues." In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `DllmManager.filter_finished_reqs`，主要用于实现 filter finished reqs 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 351-355: Method DllmManager.init_next_round
```python
    def init_next_round(self) -> None:
        """Initialize staging requests for next round and clear staging queue."""
        for req in self.staging_queue:
            req.init_next_round_input()
        self.staging_queue = []
```
**EN:** This callable implements `DllmManager.init_next_round` and mainly implements init next round. The docstring states: "Initialize staging requests for next round and clear staging queue."
**CN:** 这一可调用对象实现了 `DllmManager.init_next_round`，主要用于实现 init next round 相关逻辑。

## Key Concepts / 关键概念
- `SchedulerDllmMixin`: core class or state container / 核心类或状态容器
- `DllmManager`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.dllm.config`, `sglang.srt.dllm.mixin.req`, `sglang.srt.managers.schedule_batch`, `sglang.srt.managers.schedule_policy`, `sglang.srt.mem_cache.common`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.observability.req_time_stats`, `sglang.srt.managers.scheduler`, `sglang.srt.managers.scheduler_components.metrics_reporter`
