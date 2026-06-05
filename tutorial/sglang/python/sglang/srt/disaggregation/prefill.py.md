# prefill.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/disaggregation/prefill.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements the prefill-side workflow for disaggregated serving. It prepares metadata, stages KV state, and sends transferable data toward decode workers. / 该文件实现了解耦式服务中的预填充侧流程，负责准备元数据、暂存 KV 状态，并将可传输的数据发送到解码工作节点。

## Line-by-Line Analysis / 逐行分析
### Lines 1-50: Module-level constants and helper logic
```python
"""
Life cycle of a request in the prefill server

1. Bootstrap Queue
    a. Initialize a sender for each request
    b. Use the queue to store requests whose bootstrap (handshake and preallocation) has not finished
    c. Poll senders to check bootstrap state
    d. Once bootstrap is complete, move request to Waiting Queue

2. Waiting Queue
    a. Use PrefillAdder to pop requests
    b. Run forward
    c. Add the request to Inflight Queue

3. Inflight Queue
    a. Poll (non-blocking) the sender of the request
    b. Once the transfer has finished, return the request
"""

from __future__ import annotations

import logging
from collections import deque
from http import HTTPStatus
from typing import TYPE_CHECKING, List, Optional

import torch

from sglang.srt.disaggregation.base import KVPoll
from sglang.srt.disaggregation.base.conn import StateType
from sglang.srt.disaggregation.common.conn import CommonKVManager
from sglang.srt.disaggregation.utils import (
    FAKE_BOOTSTRAP_HOST,
    DisaggregationMode,
    KVClassType,
    MetadataBuffers,
    ReqToMetadataIdxAllocator,
    TransferBackend,
    get_kv_class,
    is_mla_backend,
    poll_and_all_reduce_attn_cp_tp_group,
    prepare_abort,
    setup_state_kv_args,
)
from sglang.srt.environ import envs
from sglang.srt.managers.schedule_batch import (
    FINISH_ABORT,
    FINISH_LENGTH,
    Req,
    ScheduleBatch,
```
**EN:** This block contains module-level constants, helpers, or documentation for prefill-side KV transfer orchestration. It prepares shared state that later classes and functions build on. Notable operations include `bootstrap`, `Poll`, `import`.
**CN:** 这一段包含与预填充侧 KV 传输编排相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `bootstrap`、`Poll`、`import`。

### Lines 51-69: Module-level constants and helper logic
```python
)
from sglang.srt.mem_cache.common import (
    kv_to_page_indices,
    kv_to_page_num,
    maybe_cache_unfinished_req,
    release_kv_cache,
)
from sglang.srt.mem_cache.deepseek_v4_memory_pool import DeepSeekV4TokenToKVPool
from sglang.srt.observability.req_time_stats import set_schedule_time_batch

if TYPE_CHECKING:
    from torch.distributed import ProcessGroup

    from sglang.srt.managers.scheduler import GenerationBatchResult, Scheduler
    from sglang.srt.mem_cache.memory_pool import KVCache

logger = logging.getLogger(__name__)


```
**EN:** This block contains module-level constants, helpers, or documentation for prefill-side KV transfer orchestration. It prepares shared state that later classes and functions build on. Notable operations include `import`, `getLogger`.
**CN:** 这一段包含与预填充侧 KV 传输编排相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `import`、`getLogger`。

### Lines 70-88: Function `release_req_to_metadata_buffer`
```python
def release_req_to_metadata_buffer(
    req: Req, allocator: ReqToMetadataIdxAllocator
) -> None:
    """
    Release the metadata buffer index allocated for a request in prefill disaggregation mode.

    This function safely releases the metadata buffer index if it was allocated.

    Args:
        req: The request object that may have a metadata_buffer_index allocated
        allocator: The ReqToMetadataIdxAllocator instance to free the index
    """
    if (
        hasattr(req, "metadata_buffer_index")
        and req.metadata_buffer_index is not None
        and req.metadata_buffer_index >= 0
    ):
        allocator.free(req.metadata_buffer_index)
        req.metadata_buffer_index = -1
```
**EN:** This block defines the function `release_req_to_metadata_buffer`. It introduces the parameters, setup steps, and the main entry point for this piece of prefill-side KV transfer orchestration. Definitions introduced here include `release_req_to_metadata_buffer`. Notable operations include `free`.
**CN:** 这一段定义了function `release_req_to_metadata_buffer`，介绍了参数、初始化步骤，以及这部分预填充侧 KV 传输编排逻辑的主要入口。 此处引入的定义包括 `release_req_to_metadata_buffer`。 值得注意的操作包括 `free`。

### Lines 91-91: Class `PrefillBootstrapQueue` declaration
```python
class PrefillBootstrapQueue:
```
**EN:** This block declares the class `PrefillBootstrapQueue` and establishes its responsibility inside prefill-side KV transfer orchestration. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `PrefillBootstrapQueue`.
**CN:** 这一段声明了类 `PrefillBootstrapQueue`，并说明它在预填充侧 KV 传输编排中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `PrefillBootstrapQueue`。

### Lines 92-95: Supporting state inside `PrefillBootstrapQueue`
```python
    """
    Store the requests in bootstrapping
    """

```
**EN:** This block adds supporting state or helper logic inside `PrefillBootstrapQueue`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `PrefillBootstrapQueue` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 96-98: Method `__init__` signature and setup
```python
    def __init__(
        self,
        token_to_kv_pool: KVCache,
```
**EN:** This block defines the method `__init__` on `PrefillBootstrapQueue`. It introduces the parameters, setup steps, and the main entry point for this piece of prefill-side KV transfer orchestration. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `PrefillBootstrapQueue`），介绍了参数、初始化步骤，以及这部分预填充侧 KV 传输编排逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 99-122: Method `__init__` logic (part 1)
```python
        draft_token_to_kv_pool: Optional[KVCache],
        req_to_metadata_buffer_idx_allocator: ReqToMetadataIdxAllocator,
        metadata_buffers: MetadataBuffers,
        tp_rank: int,
        tp_size: int,
        gpu_id: int,
        bootstrap_port: int,
        gloo_group: ProcessGroup,
        max_total_num_tokens: int,
        scheduler: Scheduler,
        pp_rank: int,
        pp_size: int,
        transfer_backend: TransferBackend,
    ):
        self.token_to_kv_pool = token_to_kv_pool
        self.draft_token_to_kv_pool = draft_token_to_kv_pool
        self.is_mla_backend = is_mla_backend(token_to_kv_pool)
        self.metadata_buffers = metadata_buffers
        self.req_to_metadata_buffer_idx_allocator = req_to_metadata_buffer_idx_allocator
        self.tp_rank = tp_rank
        self.tp_size = tp_size
        self.pp_rank = pp_rank
        self.pp_size = pp_size
        self.gpu_id = gpu_id
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `is_mla_backend`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `is_mla_backend`。

### Lines 123-141: Method `__init__` logic (part 2)
```python
        self.bootstrap_port = bootstrap_port
        self.queue: List[Req] = []
        self.gloo_group = gloo_group
        self.max_total_num_tokens = max_total_num_tokens
        self.scheduler = scheduler
        self.transfer_backend = transfer_backend
        if envs.SGLANG_DISAGG_STAGING_BUFFER.get() and self.is_mla_backend:
            raise RuntimeError(
                "SGLANG_DISAGG_STAGING_BUFFER is designed for non-MLA models "
                "(e.g. GQA, MHA). MLA models should not set this flag."
            )
        self.kv_manager = self._init_kv_manager()

        if self.scheduler.tp_worker.is_hybrid_swa:
            # FIXME: current SWA allocation allocate full kv cache size in prefill
            self.max_total_num_tokens = min(
                self.max_total_num_tokens,
                self.scheduler.tp_worker.model_runner.swa_max_total_num_tokens,
            )
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `get`, `RuntimeError`, `_init_kv_manager`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `get`、`RuntimeError`、`_init_kv_manager`。

### Lines 143-145: Method `_init_kv_manager` signature and setup
```python
    def _init_kv_manager(self) -> CommonKVManager:
        kv_args_class = get_kv_class(self.transfer_backend, KVClassType.KVARGS)
        kv_args = kv_args_class()
```
**EN:** This block defines the method `_init_kv_manager` on `PrefillBootstrapQueue`. It introduces the parameters, setup steps, and the main entry point for this piece of prefill-side KV transfer orchestration. Definitions introduced here include `_init_kv_manager`. Notable operations include `get_kv_class`, `kv_args_class`.
**CN:** 这一段定义了method `_init_kv_manager`（属于 `PrefillBootstrapQueue`），介绍了参数、初始化步骤，以及这部分预填充侧 KV 传输编排逻辑的主要入口。 此处引入的定义包括 `_init_kv_manager`。 值得注意的操作包括 `get_kv_class`、`kv_args_class`。

### Lines 146-164: Method `_init_kv_manager` logic (part 1)
```python
        kv_args.engine_rank = self.tp_rank
        kv_args.pp_rank = self.pp_rank
        kv_args.system_dp_rank = self.scheduler.ps.dp_rank
        kv_args.prefill_start_layer = self.token_to_kv_pool.start_layer
        kv_args.prefill_end_layer = getattr(self.token_to_kv_pool, "end_layer", None)
        kv_args.mla_compression_ratios = None
        kv_data_ptrs, kv_data_lens, kv_item_lens = (
            self.token_to_kv_pool.get_contiguous_buf_infos()
        )

        if self.draft_token_to_kv_pool is not None:
            # We should also transfer draft model kv cache. The indices are
            # always shared with a target model.
            draft_kv_data_ptrs, draft_kv_data_lens, draft_kv_item_lens = (
                self.draft_token_to_kv_pool.get_contiguous_buf_infos()
            )
            kv_data_ptrs += draft_kv_data_ptrs
            kv_data_lens += draft_kv_data_lens
            kv_item_lens += draft_kv_item_lens
```
**EN:** This block continues `_init_kv_manager` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `get_contiguous_buf_infos`.
**CN:** 这一段延续了 `_init_kv_manager` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `get_contiguous_buf_infos`。

### Lines 165-189: Method `_init_kv_manager` logic (part 2)
```python

        kv_args.kv_data_ptrs = kv_data_ptrs
        kv_args.kv_data_lens = kv_data_lens
        kv_args.kv_item_lens = kv_item_lens
        if not self.is_mla_backend:
            kv_args.kv_head_num = self.token_to_kv_pool.head_num
            kv_args.total_kv_head_num = (
                self.scheduler.model_config.get_total_num_kv_heads()
            )
        kv_args.page_size = self.token_to_kv_pool.page_size

        kv_args.aux_data_ptrs, kv_args.aux_data_lens, kv_args.aux_item_lens = (
            self.metadata_buffers.get_buf_infos()
        )
        kv_args.ib_device = self.scheduler.server_args.disaggregation_ib_device
        kv_args.gpu_id = self.scheduler.ps.gpu_id

        req_to_token_pool = getattr(self.scheduler, "req_to_token_pool", None)
        setup_state_kv_args(
            kv_args,
            self.token_to_kv_pool,
            self.draft_token_to_kv_pool,
            self.scheduler.model_config.num_hidden_layers,
            req_to_token_pool=req_to_token_pool,
        )
```
**EN:** This block continues `_init_kv_manager` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `get_total_num_kv_heads`, `get_buf_infos`, `setup_state_kv_args`.
**CN:** 这一段延续了 `_init_kv_manager` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `get_total_num_kv_heads`、`get_buf_infos`、`setup_state_kv_args`。

### Lines 190-213: Method `_init_kv_manager` logic (part 3)
```python

        if isinstance(self.token_to_kv_pool, DeepSeekV4TokenToKVPool):
            # V4's KVCache is organized by compression-ratio
            # buckets rather than by layer.
            kv_args.mla_compression_ratios = list(
                self.token_to_kv_pool.compression_ratios
            )

        kv_manager_class = get_kv_class(self.transfer_backend, KVClassType.MANAGER)
        kv_manager = kv_manager_class(
            kv_args,
            DisaggregationMode.PREFILL,
            self.scheduler.server_args,
            self.is_mla_backend,
        )
        # Pass KV pool tensor refs to the manager for GPU gather (staging mode)
        if (
            envs.SGLANG_DISAGG_STAGING_BUFFER.get()
            and hasattr(kv_manager, "set_kv_buffer_tensors")
            and not self.is_mla_backend
        ):
            kv_pool = self.token_to_kv_pool
            if hasattr(kv_pool, "full_kv_pool"):
                kv_pool = kv_pool.full_kv_pool
```
**EN:** This block continues `_init_kv_manager` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `get_kv_class`, `kv_manager_class`, `gather`, `get`.
**CN:** 这一段延续了 `_init_kv_manager` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `get_kv_class`、`kv_manager_class`、`gather`、`get`。

### Lines 214-220: Method `_init_kv_manager` logic (part 4)
```python
            if hasattr(kv_pool, "k_buffer") and hasattr(kv_pool, "v_buffer"):
                kv_manager.set_kv_buffer_tensors(
                    kv_pool.k_buffer,
                    kv_pool.v_buffer,
                    kv_pool.page_size,
                )
        return kv_manager
```
**EN:** This block continues `_init_kv_manager` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `set_kv_buffer_tensors`.
**CN:** 这一段延续了 `_init_kv_manager` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `set_kv_buffer_tensors`。

### Lines 222-243: Method `add`
```python
    def add(self, req: Req, num_kv_heads: int) -> None:
        if self._check_if_req_exceed_kv_capacity(req):
            return

        backend = (
            TransferBackend.FAKE
            if req.bootstrap_host == FAKE_BOOTSTRAP_HOST
            else self.transfer_backend
        )
        kv_sender_class = get_kv_class(backend, KVClassType.SENDER)

        dest_tp_ranks = [self.tp_rank]

        req.disagg_kv_sender = kv_sender_class(
            mgr=self.kv_manager,
            bootstrap_addr=f"{req.bootstrap_host}:{self.bootstrap_port}",
            bootstrap_room=req.bootstrap_room,
            dest_tp_ranks=dest_tp_ranks,
            pp_rank=self.pp_rank,
        )
        self._process_req(req)
        self.queue.append(req)
```
**EN:** This block defines the method `add` on `PrefillBootstrapQueue`. It introduces the parameters, setup steps, and the main entry point for this piece of prefill-side KV transfer orchestration. Definitions introduced here include `add`. Notable operations include `_check_if_req_exceed_kv_capacity`, `get_kv_class`, `kv_sender_class`, `_process_req`.
**CN:** 这一段定义了method `add`（属于 `PrefillBootstrapQueue`），介绍了参数、初始化步骤，以及这部分预填充侧 KV 传输编排逻辑的主要入口。 此处引入的定义包括 `add`。 值得注意的操作包括 `_check_if_req_exceed_kv_capacity`、`get_kv_class`、`kv_sender_class`、`_process_req`。

### Lines 245-247: Method `extend`
```python
    def extend(self, reqs: List[Req], num_kv_heads: int) -> None:
        for req in reqs:
            self.add(req, num_kv_heads)
```
**EN:** This block defines the method `extend` on `PrefillBootstrapQueue`. It introduces the parameters, setup steps, and the main entry point for this piece of prefill-side KV transfer orchestration. Definitions introduced here include `extend`. Notable operations include `add`.
**CN:** 这一段定义了method `extend`（属于 `PrefillBootstrapQueue`），介绍了参数、初始化步骤，以及这部分预填充侧 KV 传输编排逻辑的主要入口。 此处引入的定义包括 `extend`。 值得注意的操作包括 `add`。

### Lines 249-257: Method `_check_if_req_exceed_kv_capacity`
```python
    def _check_if_req_exceed_kv_capacity(self, req: Req) -> bool:
        if len(req.origin_input_ids) > self.max_total_num_tokens:
            message = f"Request {req.rid} exceeds the maximum number of tokens: {len(req.origin_input_ids)} > {self.max_total_num_tokens}"
            logger.error(message)
            req.time_stats.trace_ctx.abort(abort_info={"reason": message})
            prepare_abort(req, message, status_code=HTTPStatus.BAD_REQUEST)
            self.scheduler.output_streamer.stream_output([req], req.return_logprob)
            return True
        return False
```
**EN:** This block defines the method `_check_if_req_exceed_kv_capacity` on `PrefillBootstrapQueue`. It introduces the parameters, setup steps, and the main entry point for this piece of prefill-side KV transfer orchestration. Definitions introduced here include `_check_if_req_exceed_kv_capacity`. Notable operations include `error`, `abort`, `prepare_abort`, `stream_output`.
**CN:** 这一段定义了method `_check_if_req_exceed_kv_capacity`（属于 `PrefillBootstrapQueue`），介绍了参数、初始化步骤，以及这部分预填充侧 KV 传输编排逻辑的主要入口。 此处引入的定义包括 `_check_if_req_exceed_kv_capacity`。 值得注意的操作包括 `error`、`abort`、`prepare_abort`、`stream_output`。

### Lines 259-263: Method `_process_req`
```python
    def _process_req(self, req: Req) -> None:
        """
        Set max_new_tokens = 1, so PrefillAdder memory estimation is accurate
        """
        req.sampling_params.max_new_tokens = 1
```
**EN:** This block defines the method `_process_req` on `PrefillBootstrapQueue`. It introduces the parameters, setup steps, and the main entry point for this piece of prefill-side KV transfer orchestration. Definitions introduced here include `_process_req`.
**CN:** 这一段定义了method `_process_req`（属于 `PrefillBootstrapQueue`），介绍了参数、初始化步骤，以及这部分预填充侧 KV 传输编排逻辑的主要入口。 此处引入的定义包括 `_process_req`。

### Lines 265-275: Method `pop_bootstrapped` signature and setup
```python
    def pop_bootstrapped(
        self,
        return_failed_reqs: bool = False,
        rids_to_check: Optional[List[str]] = None,
    ) -> List[Req]:
        """
        pop the reqs which has finished bootstrapping

        return_failed_reqs: For PP, on rank 0, also return the failed reqs to notify the next rank
        rids_to_check: For PP, on rank > 0, check the rids from the previous rank has consensus with the current rank.
        """
```
**EN:** This block defines the method `pop_bootstrapped` on `PrefillBootstrapQueue`. It introduces the parameters, setup steps, and the main entry point for this piece of prefill-side KV transfer orchestration. Definitions introduced here include `pop_bootstrapped`.
**CN:** 这一段定义了method `pop_bootstrapped`（属于 `PrefillBootstrapQueue`），介绍了参数、初始化步骤，以及这部分预填充侧 KV 传输编排逻辑的主要入口。 此处引入的定义包括 `pop_bootstrapped`。

### Lines 276-307: Method `pop_bootstrapped` logic (part 1)
```python

        bootstrapped_reqs = []
        failed_reqs = []
        indices_to_remove = set()

        if len(self.queue) == 0:
            if return_failed_reqs is False:
                return []
            else:
                return [], []

        polls = poll_and_all_reduce_attn_cp_tp_group(
            [req.disagg_kv_sender for req in self.queue],
            self.scheduler.attn_cp_cpu_group,
            self.scheduler.attn_tp_cpu_group,
        )

        for i, (req, poll) in enumerate(zip(self.queue, polls)):
            if rids_to_check is not None:
                # if req not in reqs_info_to_check, skip
                if req.rid not in rids_to_check:
                    continue

            if poll == KVPoll.Bootstrapping:
                continue
            elif poll == KVPoll.Failed:
                error_message = f"Prefill bootstrap failed for request rank={self.tp_rank} {req.rid=} {req.bootstrap_room=}"
                try:
                    req.disagg_kv_sender.failure_exception()
                except Exception as e:
                    error_message += f" with exception {e}"
                logger.error(error_message)
```
**EN:** This block continues `pop_bootstrapped` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `poll_and_all_reduce_attn_cp_tp_group`, `failure_exception`, `error`.
**CN:** 这一段延续了 `pop_bootstrapped` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `poll_and_all_reduce_attn_cp_tp_group`、`failure_exception`、`error`。

### Lines 308-342: Method `pop_bootstrapped` logic (part 2)
```python
                req.time_stats.trace_ctx.abort(abort_info={"reason": error_message})
                prepare_abort(
                    req, error_message, status_code=HTTPStatus.INTERNAL_SERVER_ERROR
                )
                self.scheduler.output_streamer.stream_output([req], req.return_logprob)
                indices_to_remove.add(i)
                failed_reqs.append(req)
                if self.scheduler.metrics_reporter.enable_metrics:
                    self.scheduler.metrics_collector.increment_bootstrap_failed_reqs()
                if self.scheduler.enable_hicache_storage:
                    # to release prefetch events associated with the request
                    self.scheduler.tree_cache.release_aborted_request(req.rid)
                continue

            # KV.WaitingForInput - decode is ready to receive. initialize the kv sender
            req.time_stats.set_bootstrap_done_time()
            num_kv_indices = len(req.origin_input_ids)
            if self.req_to_metadata_buffer_idx_allocator.available_size() == 0:
                break

            req.metadata_buffer_index = (
                self.req_to_metadata_buffer_idx_allocator.alloc()
            )
            assert req.metadata_buffer_index is not None

            # Cal number of pages to send
            # if decode has a cached prefix, we need to send the delta indices
            # otherwise, send the entire request
            decode_prefix_len = req.disagg_kv_sender.pop_decode_prefix_len()
            req.start_send_idx = decode_prefix_len
            num_kv_indices_to_send = num_kv_indices - decode_prefix_len
            num_pages = kv_to_page_num(
                num_kv_indices_to_send, self.token_to_kv_pool.page_size
            )
            req.disagg_kv_sender.init(num_pages, req.metadata_buffer_index)
```
**EN:** This block continues `pop_bootstrapped` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `abort`, `prepare_abort`, `stream_output`, `add`.
**CN:** 这一段延续了 `pop_bootstrapped` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `abort`、`prepare_abort`、`stream_output`、`add`。

### Lines 343-355: Method `pop_bootstrapped` logic (part 3)
```python

            bootstrapped_reqs.append(req)
            indices_to_remove.add(i)
            req.time_stats.set_wait_queue_entry_time()

        self.queue = [
            entry for i, entry in enumerate(self.queue) if i not in indices_to_remove
        ]

        if return_failed_reqs is False:
            return bootstrapped_reqs
        else:
            return bootstrapped_reqs, failed_reqs
```
**EN:** This block continues `pop_bootstrapped` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `append`, `add`, `set_wait_queue_entry_time`.
**CN:** 这一段延续了 `pop_bootstrapped` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `append`、`add`、`set_wait_queue_entry_time`。

### Lines 358-358: Class `SchedulerDisaggregationPrefillMixin` declaration
```python
class SchedulerDisaggregationPrefillMixin:
```
**EN:** This block declares the class `SchedulerDisaggregationPrefillMixin` and establishes its responsibility inside prefill-side KV transfer orchestration. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `SchedulerDisaggregationPrefillMixin`.
**CN:** 这一段声明了类 `SchedulerDisaggregationPrefillMixin`，并说明它在预填充侧 KV 传输编排中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `SchedulerDisaggregationPrefillMixin`。

### Lines 359-362: Supporting state inside `SchedulerDisaggregationPrefillMixin`
```python
    """
    Mixin for Scheduler to handle disaggregation prefill
    """

```
**EN:** This block adds supporting state or helper logic inside `SchedulerDisaggregationPrefillMixin`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `SchedulerDisaggregationPrefillMixin` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 363-372: Method `maybe_prefetch_staging_for_batch`
```python
    def maybe_prefetch_staging_for_batch(self: Scheduler, batch: ScheduleBatch) -> None:
        """Pre-send STAGING_REQ so decode allocates staging during GPU forward."""
        kv_mgr = self.disagg_prefill_bootstrap_queue.kv_manager
        prefetch = getattr(kv_mgr, "_prefetch_staging_reqs", None)
        if prefetch is None:
            return
        for req in batch.reqs:
            room = getattr(req, "bootstrap_room", None)
            if room is not None and room in kv_mgr.transfer_infos:
                prefetch(room)
```
**EN:** This block defines the method `maybe_prefetch_staging_for_batch` on `SchedulerDisaggregationPrefillMixin`. It introduces the parameters, setup steps, and the main entry point for this piece of prefill-side KV transfer orchestration. Definitions introduced here include `maybe_prefetch_staging_for_batch`. Notable operations include `prefetch`.
**CN:** 这一段定义了method `maybe_prefetch_staging_for_batch`（属于 `SchedulerDisaggregationPrefillMixin`），介绍了参数、初始化步骤，以及这部分预填充侧 KV 传输编排逻辑的主要入口。 此处引入的定义包括 `maybe_prefetch_staging_for_batch`。 值得注意的操作包括 `prefetch`。

### Lines 374-389: Method `get_next_disagg_prefill_batch_to_run`
```python
    def get_next_disagg_prefill_batch_to_run(
        self: Scheduler,
    ) -> Optional[ScheduleBatch]:
        # HACK (byronhsu): reset the batch_is_full flag because we never enter update_running_batch which resets it
        # Otherwise, it hangs under high concurrency
        self.running_batch.batch_is_full = False

        self.process_prefill_chunk()

        batch = self.get_new_batch_prefill()
        batch = self.dp_attn_adapter.maybe_prepare_mlp_sync_batch(batch)

        if batch:
            set_schedule_time_batch(batch)

        return batch
```
**EN:** This block defines the method `get_next_disagg_prefill_batch_to_run` on `SchedulerDisaggregationPrefillMixin`. It introduces the parameters, setup steps, and the main entry point for this piece of prefill-side KV transfer orchestration. Definitions introduced here include `get_next_disagg_prefill_batch_to_run`. Notable operations include `HACK`, `process_prefill_chunk`, `get_new_batch_prefill`, `maybe_prepare_mlp_sync_batch`.
**CN:** 这一段定义了method `get_next_disagg_prefill_batch_to_run`（属于 `SchedulerDisaggregationPrefillMixin`），介绍了参数、初始化步骤，以及这部分预填充侧 KV 传输编排逻辑的主要入口。 此处引入的定义包括 `get_next_disagg_prefill_batch_to_run`。 值得注意的操作包括 `HACK`、`process_prefill_chunk`、`get_new_batch_prefill`、`maybe_prepare_mlp_sync_batch`。

### Lines 391-393: Method `event_loop_normal_disagg_prefill` signature and setup
```python
    @torch.no_grad()
    def event_loop_normal_disagg_prefill(self: Scheduler) -> None:
        """A normal scheduler loop for prefill worker in disaggregation mode."""
```
**EN:** This block defines the method `event_loop_normal_disagg_prefill` on `SchedulerDisaggregationPrefillMixin`. It introduces the parameters, setup steps, and the main entry point for this piece of prefill-side KV transfer orchestration. Definitions introduced here include `event_loop_normal_disagg_prefill`. Notable operations include `no_grad`.
**CN:** 这一段定义了method `event_loop_normal_disagg_prefill`（属于 `SchedulerDisaggregationPrefillMixin`），介绍了参数、初始化步骤，以及这部分预填充侧 KV 传输编排逻辑的主要入口。 此处引入的定义包括 `event_loop_normal_disagg_prefill`。 值得注意的操作包括 `no_grad`。

### Lines 394-417: Method `event_loop_normal_disagg_prefill` logic (part 1)
```python
        self.enable_staging = envs.SGLANG_DISAGG_STAGING_BUFFER.get()

        while True:
            # Receive requests
            recv_reqs = self.request_receiver.recv_requests()
            self.process_input_requests(recv_reqs)
            self.waiting_queue.extend(
                self.disagg_prefill_bootstrap_queue.pop_bootstrapped()
            )
            if self._engine_paused:
                continue

            # Get the next batch to run
            batch = self.get_next_disagg_prefill_batch_to_run()
            self.cur_batch = batch

            # Launch the current batch
            if batch:
                if self.enable_staging:
                    self.maybe_prefetch_staging_for_batch(batch)
                result = self.run_batch(batch)
                self.process_batch_result(batch, result)
            else:
                self.on_idle()
```
**EN:** This block continues `event_loop_normal_disagg_prefill` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `get`, `recv_requests`, `process_input_requests`, `extend`.
**CN:** 这一段延续了 `event_loop_normal_disagg_prefill` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `get`、`recv_requests`、`process_input_requests`、`extend`。

### Lines 418-422: Method `event_loop_normal_disagg_prefill` logic (part 2)
```python

            self.process_disagg_prefill_inflight_queue()

            # Update last_batch
            self.last_batch = batch
```
**EN:** This block continues `event_loop_normal_disagg_prefill` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `process_disagg_prefill_inflight_queue`.
**CN:** 这一段延续了 `event_loop_normal_disagg_prefill` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `process_disagg_prefill_inflight_queue`。

### Lines 424-426: Method `event_loop_overlap_disagg_prefill` signature and setup
```python
    @torch.no_grad()
    def event_loop_overlap_disagg_prefill(self: Scheduler) -> None:
        self.result_queue = deque()
```
**EN:** This block defines the method `event_loop_overlap_disagg_prefill` on `SchedulerDisaggregationPrefillMixin`. It introduces the parameters, setup steps, and the main entry point for this piece of prefill-side KV transfer orchestration. Definitions introduced here include `event_loop_overlap_disagg_prefill`. Notable operations include `no_grad`, `deque`.
**CN:** 这一段定义了method `event_loop_overlap_disagg_prefill`（属于 `SchedulerDisaggregationPrefillMixin`），介绍了参数、初始化步骤，以及这部分预填充侧 KV 传输编排逻辑的主要入口。 此处引入的定义包括 `event_loop_overlap_disagg_prefill`。 值得注意的操作包括 `no_grad`、`deque`。

### Lines 427-450: Method `event_loop_overlap_disagg_prefill` logic (part 1)
```python
        self.enable_staging = envs.SGLANG_DISAGG_STAGING_BUFFER.get()

        while True:
            # Receive requests
            recv_reqs = self.request_receiver.recv_requests()
            self.process_input_requests(recv_reqs)
            self.waiting_queue.extend(
                self.disagg_prefill_bootstrap_queue.pop_bootstrapped()
            )
            if self._engine_paused:
                continue

            # Get the next batch to run
            batch = self.get_next_disagg_prefill_batch_to_run()
            self.cur_batch = batch

            # Launch the current batch
            if batch:
                if self.enable_staging:
                    self.maybe_prefetch_staging_for_batch(batch)
                batch_result = self.run_batch(batch)
                self.result_queue.append((batch.copy(), batch_result))
            else:
                batch_result = None
```
**EN:** This block continues `event_loop_overlap_disagg_prefill` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `get`, `recv_requests`, `process_input_requests`, `extend`.
**CN:** 这一段延续了 `event_loop_overlap_disagg_prefill` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `get`、`recv_requests`、`process_input_requests`、`extend`。

### Lines 451-467: Method `event_loop_overlap_disagg_prefill` logic (part 2)
```python

            # Process the last batch
            if self.last_batch:
                tmp_batch, tmp_result = self.result_queue.popleft()
                self.process_batch_result(tmp_batch, tmp_result)
            elif batch is None:
                # When the server is idle, do self-check and re-init some states
                self.on_idle()

            self.process_disagg_prefill_inflight_queue()

            # Run sample of the current batch
            # It depends on the result of the last batch (e.g., grammar), so we run it after the last batch is processed.
            self.launch_batch_sample_if_needed(batch_result)

            # Update last_batch
            self.last_batch = batch
```
**EN:** This block continues `event_loop_overlap_disagg_prefill` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `popleft`, `process_batch_result`, `on_idle`, `process_disagg_prefill_inflight_queue`.
**CN:** 这一段延续了 `event_loop_overlap_disagg_prefill` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `popleft`、`process_batch_result`、`on_idle`、`process_disagg_prefill_inflight_queue`。

### Lines 469-477: Method `process_batch_result_disagg_prefill` signature and setup
```python
    def process_batch_result_disagg_prefill(
        self: Scheduler,
        batch: ScheduleBatch,
        result: GenerationBatchResult,
    ) -> None:
        """
        Transfer kv for prefill completed requests and add it into disagg_prefill_inflight_queue
        Adapted from process_batch_result_prefill
        """
```
**EN:** This block defines the method `process_batch_result_disagg_prefill` on `SchedulerDisaggregationPrefillMixin`. It introduces the parameters, setup steps, and the main entry point for this piece of prefill-side KV transfer orchestration. Definitions introduced here include `process_batch_result_disagg_prefill`.
**CN:** 这一段定义了method `process_batch_result_disagg_prefill`（属于 `SchedulerDisaggregationPrefillMixin`），介绍了参数、初始化步骤，以及这部分预填充侧 KV 传输编排逻辑的主要入口。 此处引入的定义包括 `process_batch_result_disagg_prefill`。

### Lines 478-512: Method `process_batch_result_disagg_prefill` logic (part 1)
```python
        (
            logits_output,
            next_token_ids,
            extend_input_len_per_req,
            extend_logprob_start_len_per_req,
            copy_done,
        ) = (
            result.logits_output,
            result.next_token_ids,
            result.extend_input_len_per_req,
            result.extend_logprob_start_len_per_req,
            result.copy_done,
        )

        if copy_done is not None:
            copy_done.synchronize()
        if result.routed_experts_output is not None:
            result.routed_experts_output.finalize()
            result.routed_experts_output = None
        if result.indexer_topk_output is not None:
            result.indexer_topk_output.finalize()
            result.indexer_topk_output = None

        logprob_pt = 0
        # Transfer kv for prefill completed requests and add it into disagg_prefill_inflight_queue
        next_token_ids = result.next_token_ids.tolist()
        if batch.return_logprob:
            if logits_output.next_token_logprobs is not None:
                logits_output.next_token_logprobs = (
                    logits_output.next_token_logprobs.tolist()
                )
            if logits_output.input_token_logprobs is not None:
                logits_output.input_token_logprobs = tuple(
                    logits_output.input_token_logprobs.tolist()
                )
```
**EN:** This block continues `process_batch_result_disagg_prefill` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `synchronize`, `finalize`, `tolist`.
**CN:** 这一段延续了 `process_batch_result_disagg_prefill` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `synchronize`、`finalize`、`tolist`。

### Lines 513-548: Method `process_batch_result_disagg_prefill` logic (part 2)
```python

        for i, (req, next_token_id) in enumerate(
            zip(batch.reqs, next_token_ids, strict=True)
        ):
            if req.is_chunked <= 0:
                req.time_stats.set_prefill_finished_time()

                # There is no output_ids for prefill
                req.output_ids.append(next_token_id)
                maybe_cache_unfinished_req(req, self.tree_cache)
                self.disagg_prefill_inflight_queue.append(req)
                if self.spec_algorithm.is_eagle() and batch.spec_info is not None:
                    req.output_topk_p = batch.spec_info.topk_p[i]
                    req.output_topk_index = batch.spec_info.topk_index[i]
                    req.hidden_states_tensor = (
                        batch.spec_info.hidden_states[i].cpu().clone()
                    )
                else:
                    req.hidden_states_tensor = None
                if req.return_logprob:
                    assert extend_logprob_start_len_per_req is not None
                    assert extend_input_len_per_req is not None
                    extend_logprob_start_len = extend_logprob_start_len_per_req[i]
                    extend_input_len = extend_input_len_per_req[i]
                    num_input_logprobs = extend_input_len - extend_logprob_start_len
                    self.batch_result_processor.logprob_result_processor.add_logprob_return_values(
                        i,
                        req,
                        logprob_pt,
                        next_token_ids,
                        num_input_logprobs,
                        logits_output,
                    )
                    logprob_pt += num_input_logprobs
                self.send_kv_chunk(req, last_chunk=True)
                req.time_stats.set_prefill_transfer_queue_entry_time()
```
**EN:** This block continues `process_batch_result_disagg_prefill` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `set_prefill_finished_time`, `append`, `maybe_cache_unfinished_req`, `is_eagle`.
**CN:** 这一段延续了 `process_batch_result_disagg_prefill` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `set_prefill_finished_time`、`append`、`maybe_cache_unfinished_req`、`is_eagle`。

### Lines 549-583: Method `process_batch_result_disagg_prefill` logic (part 3)
```python

                if req.grammar is not None:
                    # FIXME: this try-except block is for handling unexpected xgrammar issue.
                    try:
                        req.grammar.accept_token(next_token_id)
                    except ValueError as e:
                        # Grammar accept_token can raise ValueError if the token is not in the grammar.
                        # This can happen if the grammar is not set correctly or the token is invalid.
                        error_message = f"Grammar accept_token failed for req {req.rid} with token {next_token_id}: {e}"
                        release_kv_cache(req, self.tree_cache)
                        prepare_abort(
                            req,
                            error_message,
                            status_code=HTTPStatus.INTERNAL_SERVER_ERROR,
                        )
                    req.grammar.finished = req.finished()
            else:
                # being chunked reqs' prefill is not finished
                req.is_chunked -= 1

                if req.return_logprob:
                    extend_logprob_start_len = extend_logprob_start_len_per_req[i]
                    extend_input_len = extend_input_len_per_req[i]
                    if extend_logprob_start_len < extend_input_len:
                        # Update input logprobs.
                        num_input_logprobs = extend_input_len - extend_logprob_start_len
                        self.batch_result_processor.logprob_result_processor.add_input_logprob_return_values(
                            i,
                            req,
                            logits_output,
                            logprob_pt,
                            num_input_logprobs,
                            last_prefill_chunk=False,
                        )
                        logprob_pt += num_input_logprobs
```
**EN:** This block continues `process_batch_result_disagg_prefill` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `accept_token`, `release_kv_cache`, `prepare_abort`, `finished`.
**CN:** 这一段延续了 `process_batch_result_disagg_prefill` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `accept_token`、`release_kv_cache`、`prepare_abort`、`finished`。

### Lines 584-595: Method `process_batch_result_disagg_prefill` logic (part 4)
```python

                if self.enable_overlap:
                    self.send_kv_chunk(req, last_chunk=False, end_idx=req.tmp_end_idx)
                req.time_stats.set_last_chunked_prefill_finish_time()

        can_run_cuda_graph = getattr(result, "can_run_cuda_graph", False)
        self.metrics_reporter.report_prefill_stats(
            batch=batch,
            prefill_stats=batch.prefill_stats,
            can_run_cuda_graph=can_run_cuda_graph,
            dp_cooperation_info=batch.dp_cooperation_info,
        )
```
**EN:** This block continues `process_batch_result_disagg_prefill` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `send_kv_chunk`, `set_last_chunked_prefill_finish_time`, `report_prefill_stats`.
**CN:** 这一段延续了 `process_batch_result_disagg_prefill` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `send_kv_chunk`、`set_last_chunked_prefill_finish_time`、`report_prefill_stats`。

### Lines 597-603: Method `process_disagg_prefill_inflight_queue` signature and setup
```python
    def process_disagg_prefill_inflight_queue(
        self: Scheduler, rids_to_check: Optional[List[str]] = None
    ) -> List[Req]:
        """
        Poll the requests in the middle of transfer. If done, return the request.
        rids_to_check: For PP, on rank > 0, check the rids from the previous rank has consensus with the current rank.
        """
```
**EN:** This block defines the method `process_disagg_prefill_inflight_queue` on `SchedulerDisaggregationPrefillMixin`. It introduces the parameters, setup steps, and the main entry point for this piece of prefill-side KV transfer orchestration. Definitions introduced here include `process_disagg_prefill_inflight_queue`.
**CN:** 这一段定义了method `process_disagg_prefill_inflight_queue`（属于 `SchedulerDisaggregationPrefillMixin`），介绍了参数、初始化步骤，以及这部分预填充侧 KV 传输编排逻辑的主要入口。 此处引入的定义包括 `process_disagg_prefill_inflight_queue`。

### Lines 604-637: Method `process_disagg_prefill_inflight_queue` logic (part 1)
```python
        if len(self.disagg_prefill_inflight_queue) == 0:
            return []

        done_reqs = []

        polls = poll_and_all_reduce_attn_cp_tp_group(
            [req.disagg_kv_sender for req in self.disagg_prefill_inflight_queue],
            self.attn_cp_cpu_group,
            self.attn_tp_cpu_group,
        )

        undone_reqs: List[Req] = []
        # Check .poll() for the reqs in disagg_prefill_inflight_queue. If Success, respond to the client and remove it from the queue
        for req, poll in zip(self.disagg_prefill_inflight_queue, polls):

            if rids_to_check is not None:
                if req.rid not in rids_to_check:
                    undone_reqs.append(req)
                    continue

                # In PP mode, the previous rank may have reached a terminal
                # state (Success/Failed) while this rank's local poll is still
                # in a transient state due to clock skew or propagation delay.
                # Treat non-terminal states as undone instead of crashing.
                if poll not in (
                    KVPoll.Success,
                    KVPoll.Failed,
                ):
                    logger.warning_once(
                        f"PP rank {self.ps.pp_rank}: unexpected poll state {poll} for rid {req.rid} "
                        f"from consensus; treating as undone",
                    )
                    undone_reqs.append(req)
                    continue
```
**EN:** This block continues `process_disagg_prefill_inflight_queue` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `poll_and_all_reduce_attn_cp_tp_group`, `poll`, `append`, `state`.
**CN:** 这一段延续了 `process_disagg_prefill_inflight_queue` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `poll_and_all_reduce_attn_cp_tp_group`、`poll`、`append`、`state`。

### Lines 638-669: Method `process_disagg_prefill_inflight_queue` logic (part 2)
```python

            if poll in [KVPoll.WaitingForInput, KVPoll.Transferring]:
                undone_reqs.append(req)
            elif poll == KVPoll.Success:  # transfer done
                release_kv_cache(req, self.tree_cache)  # unlock the tree
                req.finished_reason = FINISH_LENGTH(length=0)
                # FIXME: clean up req's data in transfer engine
                if hasattr(req.disagg_kv_sender, "clear"):
                    req.disagg_kv_sender.clear()
                done_reqs.append(req)
                req.time_stats.set_prefill_kv_transfer_finish_time()
            elif poll == KVPoll.Failed:
                error_message = f"Prefill transfer failed for request rank={self.ps.tp_rank} {req.rid=} {req.bootstrap_room=}"
                try:
                    req.disagg_kv_sender.failure_exception()
                except Exception as e:
                    error_message += f" with exception {e}"
                logger.warning(error_message)
                req.time_stats.trace_ctx.abort(abort_info={"reason": error_message})
                release_kv_cache(req, self.tree_cache)  # unlock the tree
                prepare_abort(
                    req, error_message, status_code=HTTPStatus.INTERNAL_SERVER_ERROR
                )
                done_reqs.append(req)
                if self.metrics_reporter.enable_metrics:
                    self.metrics_collector.increment_transfer_failed_reqs()
            else:
                logger.warning_once(
                    f"Unexpected polling state {poll} for rid {req.rid} in inflight queue; "
                    f"treating as undone",
                )
                undone_reqs.append(req)
```
**EN:** This block continues `process_disagg_prefill_inflight_queue` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `append`, `release_kv_cache`, `FINISH_LENGTH`, `clear`.
**CN:** 这一段延续了 `process_disagg_prefill_inflight_queue` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `append`、`release_kv_cache`、`FINISH_LENGTH`、`clear`。

### Lines 670-699: Method `process_disagg_prefill_inflight_queue` logic (part 3)
```python

        for req in done_reqs:
            req.time_stats.set_completion_time()

        for req in done_reqs:
            if isinstance(req.finished_reason, FINISH_ABORT):
                continue
            if req.bootstrap_host == FAKE_BOOTSTRAP_HOST:
                continue
            kv_mgr = getattr(req.disagg_kv_sender, "kv_mgr", None)
            if kv_mgr and getattr(kv_mgr, "is_dummy_cp_rank", False):
                continue
            metrics = req.time_stats.compute_and_observe_kv_transfer_metrics(
                req.disagg_kv_sender.get_transfer_metric()
            )
            if metrics:
                # Update last-value for REST API
                if "latency_ms" in metrics:
                    self.metrics_reporter.kv_transfer_latency_ms = metrics["latency_ms"]
                if "speed_gb_s" in metrics:
                    self.metrics_reporter.kv_transfer_speed_gb_s = metrics["speed_gb_s"]

        # Stream requests which have finished transfer
        self.output_streamer.stream_output(
            done_reqs,
            any(req.return_logprob for req in done_reqs),
            None,
        )
        for req in done_reqs:
            req: Req
```
**EN:** This block continues `process_disagg_prefill_inflight_queue` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `set_completion_time`, `compute_and_observe_kv_transfer_metrics`, `get_transfer_metric`, `stream_output`.
**CN:** 这一段延续了 `process_disagg_prefill_inflight_queue` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `set_completion_time`、`compute_and_observe_kv_transfer_metrics`、`get_transfer_metric`、`stream_output`。

### Lines 700-707: Method `process_disagg_prefill_inflight_queue` logic (part 4)
```python

            release_req_to_metadata_buffer(
                req, self.req_to_metadata_buffer_idx_allocator
            )

        self.disagg_prefill_inflight_queue = undone_reqs

        return done_reqs
```
**EN:** This block continues `process_disagg_prefill_inflight_queue` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `release_req_to_metadata_buffer`.
**CN:** 这一段延续了 `process_disagg_prefill_inflight_queue` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `release_req_to_metadata_buffer`。

### Lines 709-725: Method `get_transferred_rids`
```python
    def get_transferred_rids(self: Scheduler) -> List[str]:
        """
        Used by PP, get the transferred rids but **do not pop**
        """
        polls = poll_and_all_reduce_attn_cp_tp_group(
            [req.disagg_kv_sender for req in self.disagg_prefill_inflight_queue],
            self.attn_cp_cpu_group,
            self.attn_tp_cpu_group,
        )

        transferred_rids: List[str] = []

        for req, poll in zip(self.disagg_prefill_inflight_queue, polls):
            if poll == KVPoll.Success or poll == KVPoll.Failed:
                transferred_rids.append(req.rid)

        return transferred_rids
```
**EN:** This block defines the method `get_transferred_rids` on `SchedulerDisaggregationPrefillMixin`. It introduces the parameters, setup steps, and the main entry point for this piece of prefill-side KV transfer orchestration. Definitions introduced here include `get_transferred_rids`. Notable operations include `poll_and_all_reduce_attn_cp_tp_group`, `append`.
**CN:** 这一段定义了method `get_transferred_rids`（属于 `SchedulerDisaggregationPrefillMixin`），介绍了参数、初始化步骤，以及这部分预填充侧 KV 传输编排逻辑的主要入口。 此处引入的定义包括 `get_transferred_rids`。 值得注意的操作包括 `poll_and_all_reduce_attn_cp_tp_group`、`append`。

### Lines 727-729: Method `process_prefill_chunk` signature and setup
```python
    def process_prefill_chunk(self: Scheduler) -> None:
        chunked_req_to_exclude = set()
        if self.chunked_req:
```
**EN:** This block defines the method `process_prefill_chunk` on `SchedulerDisaggregationPrefillMixin`. It introduces the parameters, setup steps, and the main entry point for this piece of prefill-side KV transfer orchestration. Definitions introduced here include `process_prefill_chunk`.
**CN:** 这一段定义了method `process_prefill_chunk`（属于 `SchedulerDisaggregationPrefillMixin`），介绍了参数、初始化步骤，以及这部分预填充侧 KV 传输编排逻辑的主要入口。 此处引入的定义包括 `process_prefill_chunk`。

### Lines 730-753: Method `process_prefill_chunk` logic (part 1)
```python
            chunked_req_to_exclude.add(self.chunked_req)
            maybe_cache_unfinished_req(self.chunked_req, self.tree_cache, chunked=True)
            if self.enable_overlap:
                # Delay KV transfer to process_batch_result_disagg_prefill when overlap is enabled to ensure results are resolved
                self.chunked_req.tmp_end_idx = min(
                    len(self.chunked_req.fill_ids),
                    len(self.chunked_req.origin_input_ids),
                )
            else:
                self.send_kv_chunk(self.chunked_req)
            self.running_batch.batch_is_full = False

        if self.last_batch and self.last_batch.forward_mode.is_extend():
            if self.last_batch.chunked_req:
                # In the context pipeline parallelism, after the last chunk, the current microbatch still track outdated chunked_req.
                # We need to discard it.
                chunked_req_to_exclude.add(self.last_batch.chunked_req)

            last_bs = self.last_batch.batch_size()
            self.last_batch.filter_batch(
                chunked_req_to_exclude=list(chunked_req_to_exclude)
            )
            if self.last_batch.batch_size() < last_bs:
                self.running_batch.batch_is_full = False
```
**EN:** This block continues `process_prefill_chunk` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `add`, `maybe_cache_unfinished_req`, `send_kv_chunk`, `is_extend`.
**CN:** 这一段延续了 `process_prefill_chunk` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `add`、`maybe_cache_unfinished_req`、`send_kv_chunk`、`is_extend`。

### Lines 755-763: Method `send_kv_chunk` signature and setup
```python
    def send_kv_chunk(
        self: Scheduler,
        req: Req,
        last_chunk: bool = False,
        end_idx: Optional[int] = None,
    ) -> None:
        """
        Send a prefilled chunk to the decode server
        """
```
**EN:** This block defines the method `send_kv_chunk` on `SchedulerDisaggregationPrefillMixin`. It introduces the parameters, setup steps, and the main entry point for this piece of prefill-side KV transfer orchestration. Definitions introduced here include `send_kv_chunk`.
**CN:** 这一段定义了method `send_kv_chunk`（属于 `SchedulerDisaggregationPrefillMixin`），介绍了参数、初始化步骤，以及这部分预填充侧 KV 传输编排逻辑的主要入口。 此处引入的定义包括 `send_kv_chunk`。

### Lines 764-794: Method `send_kv_chunk` logic (part 1)
```python
        page_size = self.token_to_kv_pool_allocator.page_size
        start_idx = req.start_send_idx
        end_idx = (
            end_idx
            if end_idx is not None
            else min(len(req.fill_ids), len(req.origin_input_ids))
        )

        if not last_chunk:
            # if not the last chunk and the last page is partial, delay the last partial page to the next send
            end_idx = end_idx - end_idx % page_size

        if end_idx < start_idx:
            logger.debug(
                "send_kv_chunk skip: rid=%s start_send_idx=%s end_idx=%s",
                req.rid,
                start_idx,
                end_idx,
            )
            return

        kv_indices = (
            self.req_to_token_pool.req_to_token[req.req_pool_idx, start_idx:end_idx]
            .cpu()
            .numpy()
        )
        state_indices: Optional[List] = None
        if last_chunk:
            self.disagg_metadata_buffers.set_buf(req)

            seq_len = len(req.fill_ids)
```
**EN:** This block continues `send_kv_chunk` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `debug`, `cpu`, `numpy`, `set_buf`.
**CN:** 这一段延续了 `send_kv_chunk` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `debug`、`cpu`、`numpy`、`set_buf`。

### Lines 795-825: Method `send_kv_chunk` logic (part 2)
```python

            def _mamba_payload():
                return [
                    self.req_to_token_pool.req_index_to_mamba_index_mapping[
                        req.req_pool_idx
                    ]
                    .cpu()
                    .numpy()
                ]

            def _swa_payload():
                window_size = self.sliding_window_size
                window_start = max(0, seq_len - window_size)
                window_start = (window_start // page_size) * page_size
                window_kv_indices_full = self.req_to_token_pool.req_to_token[
                    req.req_pool_idx, window_start:seq_len
                ]
                window_kv_indices_swa = (
                    self.token_to_kv_pool_allocator.translate_loc_from_full_to_swa(
                        window_kv_indices_full
                    )
                )
                return kv_to_page_indices(
                    window_kv_indices_swa.cpu().numpy(), page_size
                )

            def _nsa_payload():
                kv_indices_full = self.req_to_token_pool.req_to_token[
                    req.req_pool_idx, :seq_len
                ]
                return kv_to_page_indices(kv_indices_full.cpu().numpy(), page_size)
```
**EN:** This block continues `send_kv_chunk` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Definitions introduced here include `_mamba_payload`, `_swa_payload`, `_nsa_payload`. Notable operations include `cpu`, `numpy`, `translate_loc_from_full_to_swa`, `kv_to_page_indices`.
**CN:** 这一段延续了 `send_kv_chunk` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 此处引入的定义包括 `_mamba_payload`、`_swa_payload`、`_nsa_payload`。 值得注意的操作包括 `cpu`、`numpy`、`translate_loc_from_full_to_swa`、`kv_to_page_indices`。

### Lines 826-845: Method `send_kv_chunk` logic (part 3)
```python

            state_types = (
                self.disagg_prefill_bootstrap_queue.kv_manager.kv_args.state_types
            )
            state_indices = []
            for st in state_types:
                if st == StateType.MAMBA:
                    state_indices.append(_mamba_payload())
                elif st == StateType.SWA:
                    state_indices.append(_swa_payload())
                elif st == StateType.NSA:
                    state_indices.append(_nsa_payload())
                else:
                    state_indices.append(None)

        page_indices = kv_to_page_indices(kv_indices, page_size)
        if not req.disagg_kv_sender.should_send_kv_chunk(len(page_indices), last_chunk):
            return
        req.disagg_kv_sender.send(page_indices, state_indices)
        req.start_send_idx = end_idx
```
**EN:** This block continues `send_kv_chunk` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding prefill-side KV transfer orchestration workflow. Notable operations include `append`, `_mamba_payload`, `_swa_payload`, `_nsa_payload`.
**CN:** 这一段延续了 `send_kv_chunk` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的预填充侧 KV 传输编排工作流。 值得注意的操作包括 `append`、`_mamba_payload`、`_swa_payload`、`_nsa_payload`。

## Key Concepts / 关键概念
- `release_req_to_metadata_buffer`: Function that performs release req to metadata buffer for the surrounding workflow. / `release_req_to_metadata_buffer`：在周边工作流中执行“releasereqto元数据缓冲区”相关任务的函数。
- `PrefillBootstrapQueue`: Class that encapsulates prefill bootstrap queue behavior in this module. / `PrefillBootstrapQueue`：封装与“预填充bootstrapqueue”相关行为的类。
- `SchedulerDisaggregationPrefillMixin`: Class that encapsulates scheduler disaggregation prefill mixin behavior in this module. / `SchedulerDisaggregationPrefillMixin`：封装与“scheduler解耦部署预填充mixin”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `collections`, `http`, `typing`
- **External packages / 外部依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.disaggregation.base`, `sglang.srt.disaggregation.base.conn`, `sglang.srt.disaggregation.common.conn`, `sglang.srt.disaggregation.utils`, `sglang.srt.environ`, `sglang.srt.managers.schedule_batch`, `sglang.srt.mem_cache.common`, `sglang.srt.mem_cache.deepseek_v4_memory_pool`, `sglang.srt.observability.req_time_stats`, `sglang.srt.managers.scheduler`, `sglang.srt.mem_cache.memory_pool`
