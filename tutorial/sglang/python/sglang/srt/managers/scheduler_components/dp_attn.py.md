# dp_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/scheduler_components/dp_attn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements dp attn logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 dp attn 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-4: Import runtime dependencies / 导入运行时依赖
```python
from dataclasses import dataclass
from typing import TYPE_CHECKING, Callable, Optional
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 6-6: Import runtime dependencies / 导入运行时依赖
```python
import torch
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 8-21: Import runtime dependencies / 导入运行时依赖
```python
from sglang.srt.batch_overlap.two_batch_overlap import TboDPAttentionPreparer
from sglang.srt.configs.model_config import ModelConfig
from sglang.srt.distributed.parallel_state import get_tp_group
from sglang.srt.distributed.parallel_state_wrapper import ParallelState
from sglang.srt.environ import envs
from sglang.srt.managers.schedule_batch import ScheduleBatch
from sglang.srt.mem_cache.allocator import BaseTokenToKVPoolAllocator
from sglang.srt.mem_cache.base_prefix_cache import BasePrefixCache
from sglang.srt.mem_cache.memory_pool import ReqToTokenPool
from sglang.srt.model_executor.forward_batch_info import ForwardMode
from sglang.srt.observability.metrics_collector import DPCooperationInfo
from sglang.srt.server_args import ServerArgs
from sglang.srt.speculative.spec_info import SpeculativeAlgorithm
from sglang.srt.utils.common import require_mlp_tp_gather
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 23-24: Provide supporting module logic / 提供辅助模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.distributed.parallel_state import GroupCoordinator
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 27-34: Provide supporting module logic / 提供辅助模块逻辑
```python
_ENABLE_METRICS_DP_ATTENTION = envs.SGLANG_ENABLE_METRICS_DP_ATTENTION.get()


@dataclass
class MLPSyncBatchInfo:
    dp_size: int
    tp_size: int
    cp_size: int
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 36-49: Provide supporting module logic / 提供辅助模块逻辑
```python
    num_tokens: int
    num_tokens_for_logprob: int
    can_cuda_graph: bool
    is_extend_in_batch: bool
    local_can_run_tbo: bool
    local_forward_mode: int

    # some gathered elements
    tp0_info: torch.Tensor = None
    global_num_tokens: list[int] = None
    global_num_tokens_for_logprob: list[int] = None
    tbo_split_seq_index: torch.Tensor = None
    global_forward_mode: int = None
    dp_cooperation_info: Optional[DPCooperationInfo] = None
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 51-63: Implement get local tensor / 实现get local 张量
```python
    def _get_local_tensor(self, device, dtype=torch.int64) -> torch.Tensor:
        return torch.tensor(
            [
                self.num_tokens,
                self.num_tokens_for_logprob,
                int(self.can_cuda_graph),
                int(self.is_extend_in_batch),
                int(self.local_can_run_tbo),
                self.local_forward_mode,
            ],
            device=device,
            dtype=dtype,
        )
```
**EN:** This block implements the method `_get_local_tensor(device, dtype)` on `MLPSyncBatchInfo`. It focuses on handling the dp attn responsibilities represented by `_get_local_tensor`, so the class can advance the dp attn workflow in a self-contained way.
**CN:** 该代码块实现 `MLPSyncBatchInfo` 上的方法 `_get_local_tensor(device, dtype)`。它围绕 `_get_local_tensor` 所承担的 dp attn 相关职责展开，使该类能够独立推进相应流程。

### Lines 65-77: Implement get fallback tensor / 实现get fallback 张量
```python
    def _get_fallback_tensor(self, device, dtype=torch.int64) -> torch.Tensor:
        return torch.tensor(
            [
                0,  # num_tokens
                0,  # num_tokens_for_logprob
                1,  # can_cuda_graph
                0,  # is_extend_in_batch
                1,  # local_can_run_tbo
                ForwardMode.IDLE.value,  # local_forward_mode
            ],
            device=device,
            dtype=dtype,
        )
```
**EN:** This block implements the method `_get_fallback_tensor(device, dtype)` on `MLPSyncBatchInfo`. It focuses on handling the dp attn responsibilities represented by `_get_fallback_tensor`, so the class can advance the dp attn workflow in a self-contained way.
**CN:** 该代码块实现 `MLPSyncBatchInfo` 上的方法 `_get_fallback_tensor(device, dtype)`。它围绕 `_get_fallback_tensor` 所承担的 dp attn 相关职责展开，使该类能够独立推进相应流程。

### Lines 79-110: Implement all gather / 实现all gather
```python
    def all_gather(self, device, group: torch.distributed.ProcessGroup):
        local_info_tensor = self._get_local_tensor(device=device)
        global_info_tensor = torch.empty(
            (self.dp_size, self.tp_size * self.cp_size, 6),
            dtype=torch.int64,
            device=device,
        )

        torch.distributed.all_gather_into_tensor(
            global_info_tensor.flatten(),
            local_info_tensor,
            group=group,
        )
        if device == "cpu":
            tp_active_ranks = get_tp_group().active_ranks_cpu
        else:
            tp_active_ranks = get_tp_group().active_ranks

        # Set fallback values for inactive ranks
        tp_info = global_info_tensor.view(self.dp_size * self.tp_size * self.cp_size, 6)
        tp_info[tp_active_ranks == 0] = self._get_fallback_tensor(device=device)

        tp0_info = global_info_tensor[:, 0, :]
        self.tp0_info = tp0_info
        # Perform only one Device-to-Host (D2H) memory copy
        cpu_data = tp0_info[:, :2].cpu()
        self.global_num_tokens = cpu_data[:, 0].tolist()
        self.global_num_tokens_for_logprob = cpu_data[:, 1].tolist()
        self.can_cuda_graph = bool(tp0_info[:, 2].min().item())
        self.is_extend_in_batch = bool(tp0_info[:, 3].max().item())
        if _ENABLE_METRICS_DP_ATTENTION:
            self.dp_cooperation_info = DPCooperationInfo.create(tp0_info[:, 5].tolist())
```
**EN:** This block implements the method `all_gather(device, group)` on `MLPSyncBatchInfo`. It focuses on handling the dp attn responsibilities represented by `all_gather`, so the class can advance the dp attn workflow in a self-contained way.
**CN:** 该代码块实现 `MLPSyncBatchInfo` 上的方法 `all_gather(device, group)`。它围绕 `all_gather` 所承担的 dp attn 相关职责展开，使该类能够独立推进相应流程。

### Lines 113-134: Implement update gather batch / 实现update gather 批处理
```python
def _update_gather_batch(
    batch: ScheduleBatch,
    mlp_sync_info: MLPSyncBatchInfo,
    require_mlp_tp_gather: bool,
    skip_all_gather=False,
):
    # TODO: handle the case when moe_dense_tp_size != 1
    if not require_mlp_tp_gather:
        batch.global_num_tokens = [mlp_sync_info.num_tokens]
        batch.global_num_tokens_for_logprob = [mlp_sync_info.num_tokens_for_logprob]
    else:
        batch.global_num_tokens = mlp_sync_info.global_num_tokens
        batch.global_num_tokens_for_logprob = (
            mlp_sync_info.global_num_tokens_for_logprob
        )
    if not skip_all_gather:
        batch.is_extend_in_batch = mlp_sync_info.is_extend_in_batch
        batch.tbo_split_seq_index = mlp_sync_info.tbo_split_seq_index
        batch.global_forward_mode = mlp_sync_info.global_forward_mode

    # Check forward mode for cuda graph
    batch.can_run_dp_cuda_graph = mlp_sync_info.can_cuda_graph
```
**EN:** This block implements the function `_update_gather_batch(batch, mlp_sync_info, require_mlp_tp_gather, skip_all_gather)`. It focuses on handling the dp attn responsibilities represented by `_update_gather_batch`, providing reusable behavior for the dp attn pipeline.
**CN:** 该代码块实现函数 `_update_gather_batch(batch, mlp_sync_info, require_mlp_tp_gather, skip_all_gather)`。它围绕 `_update_gather_batch` 所承担的 dp attn 相关职责展开，为对应处理链路提供可复用能力。

### Lines 137-172: Implement prepare mlp sync batch raw / 实现prepare mlp sync 批处理 raw
```python
def prepare_mlp_sync_batch_raw(
    local_batch: ScheduleBatch,
    dp_size: int,
    attn_tp_size: int,
    attn_cp_size: int,
    tp_group: GroupCoordinator,
    get_idle_batch: Callable[[], ScheduleBatch],
    disable_cuda_graph: bool,
    require_mlp_tp_gather: bool,
    disable_overlap_schedule: bool,
    offload_tags: set[str],
):
    # Check if other DP workers have running batches
    if local_batch is None or local_batch.forward_mode.is_prebuilt():
        num_tokens = 0
        num_tokens_for_logprob = 0
    elif local_batch.forward_mode.is_decode():
        num_tokens = local_batch.batch_size()
        num_tokens_for_logprob = num_tokens
    else:
        num_tokens = local_batch.extend_num_tokens
        num_tokens_for_logprob = sum(
            # We should have at least 1 token for sample in every case.
            max(extend_len - logprob_start_len, 1)
            for logprob_start_len, extend_len in zip(
                local_batch.extend_logprob_start_lens,
                local_batch.extend_lens,
            )
        )
        assert (
            local_batch.return_logprob
            or num_tokens_for_logprob == local_batch.batch_size()
        )

    skip_all_gather = envs.SGLANG_SCHEDULER_SKIP_ALL_GATHER.get()
    can_cuda_graph = (
```
**EN:** This block implements the function `prepare_mlp_sync_batch_raw(local_batch, dp_size, attn_tp_size, attn_cp_size, tp_group, ...)`. It focuses on handling the dp attn responsibilities represented by `prepare_mlp_sync_batch_raw`, providing reusable behavior for the dp attn pipeline.
**CN:** 该代码块实现函数 `prepare_mlp_sync_batch_raw(local_batch, dp_size, attn_tp_size, attn_cp_size, tp_group, ...)`。它围绕 `prepare_mlp_sync_batch_raw` 所承担的 dp attn 相关职责展开，为对应处理链路提供可复用能力。

### Lines 173-208: Continue prepare mlp sync batch raw / 继续说明prepare mlp sync 批处理 raw
```python
        local_batch is None
        or local_batch.forward_mode.is_decode_or_idle()
        or local_batch.forward_mode.is_prebuilt()
    ) and not disable_cuda_graph

    is_extend_in_batch = local_batch.forward_mode.is_extend() if local_batch else False
    if local_batch is not None:
        local_batch.is_extend_in_batch = is_extend_in_batch

    tbo_preparer = TboDPAttentionPreparer()
    if len(offload_tags) == 0 and (
        disable_overlap_schedule
        or envs.SGLANG_NCCL_ALL_GATHER_IN_OVERLAP_SCHEDULER_SYNC_BATCH.get()
    ):
        group = tp_group.device_group
        device = tp_group.device
    else:
        group = tp_group.cpu_group
        device = "cpu"

    local_can_run_tbo, local_forward_mode = tbo_preparer.prepare_all_gather(local_batch)

    mlp_sync_info = MLPSyncBatchInfo(
        dp_size=dp_size,
        tp_size=attn_tp_size,
        cp_size=attn_cp_size,
        num_tokens=num_tokens,
        num_tokens_for_logprob=num_tokens_for_logprob,
        can_cuda_graph=can_cuda_graph,
        is_extend_in_batch=is_extend_in_batch,
        local_can_run_tbo=local_can_run_tbo,
        local_forward_mode=local_forward_mode,
    )

    if not skip_all_gather:
        mlp_sync_info.all_gather(device=device, group=group)
```
**EN:** This block implements the function `prepare_mlp_sync_batch_raw(local_batch, dp_size, attn_tp_size, attn_cp_size, tp_group, ...)`. It focuses on handling the dp attn responsibilities represented by `prepare_mlp_sync_batch_raw`, providing reusable behavior for the dp attn pipeline.
**CN:** 该代码块实现函数 `prepare_mlp_sync_batch_raw(local_batch, dp_size, attn_tp_size, attn_cp_size, tp_group, ...)`。它围绕 `prepare_mlp_sync_batch_raw` 所承担的 dp attn 相关职责展开，为对应处理链路提供可复用能力。

### Lines 209-231: Continue prepare mlp sync batch raw / 继续说明prepare mlp sync 批处理 raw
```python

        mlp_sync_info.tbo_split_seq_index, mlp_sync_info.global_forward_mode = (
            tbo_preparer.compute_output(
                mlp_sync_info.tp0_info[:, 4:6],
            )
        )

    need_idle_batch = skip_all_gather or max(mlp_sync_info.global_num_tokens) > 0
    if need_idle_batch:
        batch_to_gather = local_batch
        if local_batch is None:
            batch_to_gather = local_batch = get_idle_batch()
        elif local_batch.forward_mode.is_prebuilt():
            # NOTE: for prebuilt batch, we add an inner idle batch to run MLP sync
            batch_to_gather = local_batch.inner_idle_batch = get_idle_batch()
        _update_gather_batch(
            batch_to_gather, mlp_sync_info, require_mlp_tp_gather, skip_all_gather
        )

    if _ENABLE_METRICS_DP_ATTENTION and local_batch is not None:
        local_batch.dp_cooperation_info = mlp_sync_info.dp_cooperation_info

    return local_batch
```
**EN:** This block implements the function `prepare_mlp_sync_batch_raw(local_batch, dp_size, attn_tp_size, attn_cp_size, tp_group, ...)`. It focuses on handling the dp attn responsibilities represented by `prepare_mlp_sync_batch_raw`, providing reusable behavior for the dp attn pipeline.
**CN:** 该代码块实现函数 `prepare_mlp_sync_batch_raw(local_batch, dp_size, attn_tp_size, attn_cp_size, tp_group, ...)`。它围绕 `prepare_mlp_sync_batch_raw` 所承担的 dp attn 相关职责展开，为对应处理链路提供可复用能力。

### Lines 232-234: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass(kw_only=True, slots=True, frozen=True)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 235-290: Define class SchedulerDPAttnAdapter / 定义类 SchedulerDPAttnAdapter
```python
class SchedulerDPAttnAdapter:
    tp_group: "GroupCoordinator"
    req_to_token_pool: ReqToTokenPool
    token_to_kv_pool_allocator: BaseTokenToKVPoolAllocator
    tree_cache: BasePrefixCache
    offload_tags: set[str]
    ps: ParallelState
    server_args: ServerArgs
    model_config: ModelConfig
    enable_overlap: bool
    spec_algorithm: SpeculativeAlgorithm
    get_require_mlp_sync: Callable[[], bool]

    def prepare_mlp_sync_batch(self, local_batch: ScheduleBatch):
        return prepare_mlp_sync_batch_raw(
            local_batch,
            dp_size=self.server_args.dp_size,
            attn_tp_size=self.ps.attn_tp_size,
            attn_cp_size=self.ps.attn_cp_size,
            tp_group=self.tp_group,
            get_idle_batch=self.get_idle_batch,
            disable_cuda_graph=self.server_args.disable_cuda_graph,
            require_mlp_tp_gather=require_mlp_tp_gather(self.server_args),
            disable_overlap_schedule=self.server_args.disable_overlap_schedule,
            offload_tags=self.offload_tags,
        )

    def maybe_prepare_mlp_sync_batch(
        self,
        batch: Optional[ScheduleBatch],
        need_sync: Optional[bool] = None,
    ) -> Optional[ScheduleBatch]:
        """
        Helper to prepare MLP sync batch for DP attention.
        Should be called after get_new_batch_prefill().

        Args:
            batch: The batch to process
            need_sync: If specified, overrides self.get_require_mlp_sync() for prepare_mlp_sync_batch decision
        """
        if need_sync if need_sync is not None else self.get_require_mlp_sync():
            batch = self.prepare_mlp_sync_batch(batch)
        return batch

    def get_idle_batch(self) -> ScheduleBatch:
        idle_batch = ScheduleBatch.init_new(
            [],
            self.req_to_token_pool,
            self.token_to_kv_pool_allocator,
            self.tree_cache,
            self.model_config,
            self.enable_overlap,
            self.spec_algorithm,
        )
        idle_batch.prepare_for_idle()
        return idle_batch
```
**EN:** This block declares the class `SchedulerDPAttnAdapter`. It centers on coordinating dp attn behavior, with methods such as prepare_mlp_sync_batch, maybe_prepare_mlp_sync_batch, get_idle_batch.
**CN:** 该代码块声明类 `SchedulerDPAttnAdapter`。它负责承载与 dp attn 相关的核心状态与行为，并通过 prepare_mlp_sync_batch, maybe_prepare_mlp_sync_batch, get_idle_batch 等方法组织实现。

## Key Concepts / 关键概念
- **Core types / 核心类型**: MLPSyncBatchInfo, SchedulerDPAttnAdapter
- **Main callables / 主要可调用对象**: _update_gather_batch, prepare_mlp_sync_batch_raw
- **Domain focus / 领域焦点**: dp attn / dp attn
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: dataclasses, typing
- **Third-party / 第三方库**: __future__, torch
- **Local Modules / 本地模块**: sglang.srt.batch_overlap.two_batch_overlap, sglang.srt.configs.model_config, sglang.srt.distributed.parallel_state, sglang.srt.distributed.parallel_state_wrapper, sglang.srt.environ, sglang.srt.managers.schedule_batch, sglang.srt.mem_cache.allocator, sglang.srt.mem_cache.base_prefix_cache, sglang.srt.mem_cache.memory_pool, sglang.srt.model_executor.forward_batch_info, sglang.srt.observability.metrics_collector, sglang.srt.server_args
