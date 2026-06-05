# deepep_waterfill.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/deepep_waterfill.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements distributed communication and tensor movement with Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `WaterfillDispatchPlan`, `_empty_expanded`, `_count_routed_per_rank_kernel`, and `_waterfill_expand_kernel` and connects them to backend-specific paths such as `Triton` and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了分布式通信与张量搬运，并结合混合专家路由与专家执行。它提供了 `WaterfillDispatchPlan`、`_empty_expanded`、`_count_routed_per_rank_kernel` 以及 `_waterfill_expand_kernel` 等符号，并把这些符号连接到 `Triton` 和 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: File header and module overview
```python
# Copyright 2023-2026 SGLang Team
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
```
**EN:** This opening block contains comments, licensing text, or other context that frames the rest of the file before executable code begins.
**CN:** 这一开头部分包含注释、许可证文本或其他上下文信息，用来为后续可执行代码建立背景。

### Lines 14-30: Imports, constants, and runtime setup
```python
"""DeepEP Waterfill: shared expert as 9th routed expert, dispatched to least-loaded rank."""

from typing import NamedTuple, Optional, Tuple

import torch
import triton
import triton.language as tl
from torch import Tensor

from sglang.srt.environ import envs
from sglang.srt.layers.moe.topk import StandardTopKOutput

LOCAL_SHARED_MARKER = -1  # Invalid expert ID; DeepEP ignores expert_id < 0.
_LOCAL_PREF_NUMER = 11  # local-rank preference = 11/10
_LOCAL_PREF_DENOM = 10
```
**EN:** This section prepares the module namespace. It imports `typing.NamedTuple`, `typing.Optional`, `typing.Tuple`, `torch`, `triton`, and `triton.language`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `LOCAL_SHARED_MARKER`, `_LOCAL_PREF_NUMER`, and `_LOCAL_PREF_DENOM` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `typing.NamedTuple`、`typing.Optional`、`typing.Tuple`、`torch`、`triton` 以及 `triton.language`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `LOCAL_SHARED_MARKER`、`_LOCAL_PREF_NUMER` 以及 `_LOCAL_PREF_DENOM` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 31-39: Class `WaterfillDispatchPlan` declaration and shared state
```python
class WaterfillDispatchPlan(NamedTuple):
    """Inputs needed by the fused DeepEP Waterfill expansion path."""

    # Effective rank load consumed by the fused kernel.
    rank_load: Tensor
    allow_all_ranks: bool
    target_total: int
```
**EN:** This block introduces class `WaterfillDispatchPlan` and the state shared by its methods. It inherits from `NamedTuple`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Inputs needed by the fused DeepEP Waterfill expansion path.
**CN:** 该代码块引入类 `WaterfillDispatchPlan`，并定义其方法共享的状态。 它继承自 `NamedTuple`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 40-48: Internal helper `_empty_expanded`
```python
def _empty_expanded(topk_ids: Tensor, topk_weights: Tensor):
    """Return empty expanded tensors for zero-token batches."""
    topk, d = topk_ids.shape[1], topk_ids.device
    return (
        torch.empty(0, topk + 1, dtype=topk_ids.dtype, device=d),
        torch.empty(0, topk + 1, dtype=topk_weights.dtype, device=d),
    )
```
**EN:** This block defines `_empty_expanded` and contains the main logic for this step. It mainly invokes `torch.empty`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `topk` and `d` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_empty_expanded`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty`，说明该流程会编排底层辅助函数或计算内核。 像 `topk` 和 `d` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 49-84: Internal helper `_count_routed_per_rank_kernel`
```python
@triton.jit
def _count_routed_per_rank_kernel(
    topk_ids_ptr,  # [num_tokens, topk]
    counts_ptr,  # [world_size] output (atomic add)
    num_tokens,
    topk: tl.constexpr,
    experts_per_rank,
    world_size: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
):
    """Count routed tokens per rank using block-level histogram."""
    pid = tl.program_id(0)
    token_idx = pid * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    mask = token_idx < num_tokens

    for r in range(world_size):
        rank_count = tl.zeros([BLOCK_SIZE], dtype=tl.int64)

        for k in range(topk):
            expert_id = tl.load(
                topk_ids_ptr + token_idx * topk + k, mask=mask, other=-1
            ).to(tl.int64)
            valid = expert_id >= 0
            target_rank = expert_id // experts_per_rank
            target_rank = tl.minimum(tl.maximum(target_rank, 0), world_size - 1)
            rank_count += tl.where(
                mask & valid & (target_rank == r),
                tl.full([BLOCK_SIZE], 1, dtype=tl.int64),
                tl.zeros([BLOCK_SIZE], dtype=tl.int64),
            )

        block_total = tl.sum(rank_count)
        if block_total > 0:
            tl.atomic_add(counts_ptr + r, block_total)
```
**EN:** This block defines `_count_routed_per_rank_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `range`, `tl.arange`, `tl.zeros`, and `tl.sum`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `token_idx`, `mask`, `rank_count`, and `block_total` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_count_routed_per_rank_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`range`、`tl.arange`、`tl.zeros` 以及 `tl.sum`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`token_idx`、`mask`、`rank_count` 以及 `block_total` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 85-266: Internal helper `_waterfill_expand_kernel`
```python
@triton.jit
def _waterfill_expand_kernel(
    topk_ids_ptr,
    topk_weights_ptr,
    rank_load_ptr,
    expanded_ids_ptr,
    expanded_weights_ptr,
    num_tokens,
    topk: tl.constexpr,
    old_experts_per_rank,
    new_experts_per_rank,
    world_size: tl.constexpr,
    source_rank,
    shared_weight,
    local_marker,
    local_pref_numer,
    local_pref_denom,
    precomputed_target_total,
    ALLOW_ALL_RANKS: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
):
    """Fused waterfill + expand. ID remap: old_id -> old_id + old_id // old_epr."""
    pid = tl.program_id(0)
    token_idx = pid * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    mask = token_idx < num_tokens

    r_idx = tl.arange(0, world_size)
    rank_load_vec = tl.load(rank_load_ptr + r_idx, mask=r_idx < world_size, other=0).to(
        tl.int64
    )
    total_effective_k = tl.sum(rank_load_vec)
    total_tokens_global_k = total_effective_k // topk
    derived_target_total = (
        total_effective_k + total_tokens_global_k + world_size - 1
    ) // world_size
    target_total = tl.where(
        precomputed_target_total > 0,
        precomputed_target_total,
        derived_target_total,
    )

    # Step 1: Select destination rank for shared expert (waterfill sampling).
    source_count = tl.load(rank_load_ptr + source_rank)
    best_count = tl.where(mask, source_count, 2**30)
    best_rank = tl.full([BLOCK_SIZE], source_rank, dtype=tl.int64)
    has_valid = tl.zeros([BLOCK_SIZE], dtype=tl.int1)
    src_rank_i32 = tl.full([BLOCK_SIZE], source_rank, dtype=tl.int32)

    if ALLOW_ALL_RANKS:
        candidate_mask = tl.full([BLOCK_SIZE], (1 << world_size) - 1, dtype=tl.int32)
        for r in range(world_size):
            target_count = tl.load(rank_load_ptr + r).to(tl.int64)
            better = (
                target_count * local_pref_numer < best_count * local_pref_denom
            ) & mask
            best_count = tl.where(better, target_count, best_count)
            best_rank = tl.where(
                better, tl.full([BLOCK_SIZE], r, dtype=tl.int64), best_rank
            )
    else:
        candidate_mask = (tl.full([BLOCK_SIZE], 1, dtype=tl.int32) << src_rank_i32).to(
            tl.int32
        )

    for k in range(topk):
        expert_id = tl.load(
            topk_ids_ptr + token_idx * topk + k, mask=mask, other=-1
        ).to(tl.int64)
        valid = expert_id >= 0
        has_valid = has_valid | valid

        if not ALLOW_ALL_RANKS:
            target_rank = expert_id // old_experts_per_rank
            target_rank = tl.minimum(tl.maximum(target_rank, 0), world_size - 1)
            target_rank_i32 = target_rank.to(tl.int32)
            shift_amt = tl.where(valid, target_rank_i32, 0)
            bit = tl.full([BLOCK_SIZE], 1, dtype=tl.int32) << shift_amt
            candidate_mask = tl.where(
                valid & mask, candidate_mask | bit, candidate_mask
            )

            target_count = tl.load(
                rank_load_ptr + target_rank, mask=mask & valid, other=2**30
            )

            better = (
                (target_count * local_pref_numer < best_count * local_pref_denom)
                & valid
                & mask
            )
            best_count = tl.where(better, target_count, best_count)
            best_rank = tl.where(better, target_rank, best_rank)

    total_w = tl.zeros([BLOCK_SIZE], dtype=tl.int32)
    for r in range(world_size):
        present = ((candidate_mask >> r) & 1) == 1
        rank_load_r = tl.load(rank_load_ptr + r).to(tl.int64)
        w = tl.where(target_total > rank_load_r, target_total - rank_load_r, 0).to(
            tl.int32
        )
        w_vec = tl.full([BLOCK_SIZE], w, dtype=tl.int32)
        w_vec = tl.where(
            src_rank_i32 == r,
            w_vec,
            (w_vec * local_pref_denom) // local_pref_numer,
        )
        total_w += tl.where(present, w_vec, 0)

    token_seed = token_idx.to(tl.uint32) ^ (
        src_rank_i32.to(tl.uint32) * tl.full([BLOCK_SIZE], 0x9E3779B9, dtype=tl.uint32)
    )
    token_seed = token_seed * tl.full([BLOCK_SIZE], 1664525, dtype=tl.uint32) + tl.full(
        [BLOCK_SIZE], 1013904223, dtype=tl.uint32
    )
    u = tl.where(total_w > 0, token_seed % total_w.to(tl.uint32), 0).to(tl.int32)

    chosen = src_rank_i32
    cum = tl.zeros([BLOCK_SIZE], dtype=tl.int32)
    for r in range(world_size):
        present = ((candidate_mask >> r) & 1) == 1
        rank_load_r = tl.load(rank_load_ptr + r).to(tl.int64)
        w = tl.where(target_total > rank_load_r, target_total - rank_load_r, 0).to(
            tl.int32
        )
        w_vec = tl.full([BLOCK_SIZE], w, dtype=tl.int32)
        w_vec = tl.where(
            src_rank_i32 == r,
            w_vec,
            (w_vec * local_pref_denom) // local_pref_numer,
        )
        w_vec = tl.where(present, w_vec, 0)
        pick = (total_w > 0) & present & (u >= cum) & (u < (cum + w_vec))
        chosen = tl.where(pick, r, chosen)
        cum += w_vec

    best_rank = tl.where(total_w > 0, chosen.to(tl.int64), best_rank)

    # Step 2: Compute shared expert ID and local mask.
    is_local = best_rank == source_rank
    local_shared_id = source_rank * new_experts_per_rank + old_experts_per_rank
    remote_shared_id = best_rank * new_experts_per_rank + old_experts_per_rank
    shared_expert_id = tl.where(
        is_local,
        tl.full([BLOCK_SIZE], local_shared_id, dtype=tl.int64),
        remote_shared_id,
    ).to(tl.int64)
    shared_expert_id = tl.where(
        has_valid,
        shared_expert_id,
        tl.full([BLOCK_SIZE], local_marker, dtype=tl.int64),
    )

    # Step 3: Copy and remap topk_ids, copy weights.
    for k in range(topk):
        old_id = tl.load(topk_ids_ptr + token_idx * topk + k, mask=mask, other=-1).to(
            tl.int64
        )
        valid_id = old_id >= 0
        new_id = tl.where(valid_id, old_id + (old_id // old_experts_per_rank), old_id)
        tl.store(expanded_ids_ptr + token_idx * (topk + 1) + k, new_id, mask=mask)

    for k in range(topk):
        val = tl.load(topk_weights_ptr + token_idx * topk + k, mask=mask, other=0.0)
        expert_id = tl.load(
            topk_ids_ptr + token_idx * topk + k, mask=mask, other=-1
        ).to(tl.int64)
        val = tl.where(expert_id >= 0, val, 0.0)
        tl.store(expanded_weights_ptr + token_idx * (topk + 1) + k, val, mask=mask)

    # Step 4: Write shared expert column.
    tl.store(
        expanded_ids_ptr + token_idx * (topk + 1) + topk,
        shared_expert_id,
        mask=mask,
    )
    tl.store(
        expanded_weights_ptr + token_idx * (topk + 1) + topk,
        tl.where(has_valid, shared_weight, 0.0),
        mask=mask,
    )
```
**EN:** This block defines `_waterfill_expand_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.arange`, `tl.load.to`, `tl.sum`, and `tl.where`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `token_idx`, `mask`, `r_idx`, and `rank_load_vec` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_waterfill_expand_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.arange`、`tl.load.to`、`tl.sum` 以及 `tl.where`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`token_idx`、`mask`、`r_idx` 以及 `rank_load_vec` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 267-323: Function `materialize_waterfill_dispatch_fused` and its core logic
```python
def materialize_waterfill_dispatch_fused(
    topk_ids: Tensor,
    topk_weights: Tensor,
    rank_load: Tensor,
    num_routed_experts: int,
    world_size: int,
    source_rank: int,
    shared_weight: float,
    allow_all_ranks: bool = False,
    target_total: int = 0,
) -> Tuple[Tensor, Tensor]:
    """Run fused Waterfill rank selection and DeepEP TopK expansion.

    The Triton kernel intentionally selects each token's shared-expert rank and
    writes the expanded DeepEP TopK layout in one pass.
    """
    num_tokens = topk_ids.shape[0]
    topk = topk_ids.shape[1]
    old_experts_per_rank = num_routed_experts // world_size
    new_experts_per_rank = old_experts_per_rank + 1
    device = topk_ids.device

    if num_tokens == 0:
        return _empty_expanded(topk_ids, topk_weights)

    expanded_topk_ids = torch.empty(
        num_tokens, topk + 1, dtype=topk_ids.dtype, device=device
    )
    expanded_topk_weights = torch.empty(
        num_tokens, topk + 1, dtype=topk_weights.dtype, device=device
    )
    BLOCK_SIZE = 256
    grid = ((num_tokens + BLOCK_SIZE - 1) // BLOCK_SIZE,)
    _waterfill_expand_kernel[grid](
        topk_ids,
        topk_weights,
        rank_load,
        expanded_topk_ids,
        expanded_topk_weights,
        num_tokens,
        topk,
        old_experts_per_rank,
        new_experts_per_rank,
        world_size,
        source_rank,
        shared_weight,
        LOCAL_SHARED_MARKER,
        _LOCAL_PREF_NUMER,
        _LOCAL_PREF_DENOM,
        target_total,
        allow_all_ranks,
        BLOCK_SIZE,
    )

    return expanded_topk_ids, expanded_topk_weights
```
**EN:** This block defines `materialize_waterfill_dispatch_fused` and contains the main logic for this step. It mainly invokes `torch.empty`, `_waterfill_expand_kernel`, and `_empty_expanded`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_tokens`, `topk`, `old_experts_per_rank`, `new_experts_per_rank`, and `device` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `materialize_waterfill_dispatch_fused`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty`、`_waterfill_expand_kernel` 以及 `_empty_expanded`，说明该流程会编排底层辅助函数或计算内核。 像 `num_tokens`、`topk`、`old_experts_per_rank`、`new_experts_per_rank` 以及 `device` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 324-360: Function `expand_topk_with_shared_expert` and its core logic
```python
@torch.compile(dynamic=True)
def expand_topk_with_shared_expert(
    topk_ids: Tensor,
    topk_weights: Tensor,
    num_routed_experts: int,
    world_size: int,
    source_rank: int,
    shared_weight: float,
) -> Tuple[Tensor, Tensor]:
    """Expand topk [N, 8] → [N, 9] with ID remap; shared expert always local."""
    num_tokens = topk_ids.shape[0]
    topk = topk_ids.shape[1]
    device = topk_ids.device
    old_epr = num_routed_experts // world_size
    new_epr = old_epr + 1
    has_valid = (topk_ids >= 0).any(dim=1)
    valid_mask = topk_ids >= 0
    old_ranks = torch.where(valid_mask, topk_ids // old_epr, torch.zeros_like(topk_ids))
    expanded_topk_ids = torch.empty(
        num_tokens, topk + 1, dtype=topk_ids.dtype, device=device
    )
    expanded_topk_ids[:, :topk] = torch.where(
        valid_mask, topk_ids + old_ranks, topk_ids
    )

    shared_id = source_rank * new_epr + old_epr
    expanded_topk_ids[:, topk] = torch.where(has_valid, shared_id, LOCAL_SHARED_MARKER)
    expanded_topk_weights = torch.empty(
        num_tokens, topk + 1, dtype=topk_weights.dtype, device=device
    )
    expanded_topk_weights[:, :topk] = torch.where(valid_mask, topk_weights, 0.0)
    expanded_topk_weights[:, topk] = torch.where(has_valid, shared_weight, 0.0).to(
        topk_weights.dtype
    )
    return expanded_topk_ids, expanded_topk_weights
```
**EN:** This block defines `expand_topk_with_shared_expert` and contains the main logic for this step. Decorators like `torch.compile` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `torch.compile`, `any`, `torch.where`, `torch.empty`, and `torch.where.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_tokens`, `topk`, `device`, `old_epr`, and `new_epr` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `expand_topk_with_shared_expert`，并承载这一阶段的核心逻辑。 像 `torch.compile` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `torch.compile`、`any`、`torch.where`、`torch.empty` 以及 `torch.where.to`，说明该流程会编排底层辅助函数或计算内核。 像 `num_tokens`、`topk`、`device`、`old_epr` 以及 `new_epr` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 361-365: Class `DeepEPWaterfillBalancer` declaration and shared state
```python
class DeepEPWaterfillBalancer:
    """Waterfill load balancer: shared expert fused as real routed expert (topk 8→9)."""

    MIN_BATCH_FOR_BALANCE = 64
```
**EN:** This block introduces class `DeepEPWaterfillBalancer` and the state shared by its methods. The class docstring summarizes its role: Waterfill load balancer: shared expert fused as real routed expert (topk 8→9).
**CN:** 该代码块引入类 `DeepEPWaterfillBalancer`，并定义其方法共享的状态。 类级文档进一步概括了它的职责。

### Lines 366-384: `DeepEPWaterfillBalancer` initialization and state setup
```python
    def __init__(
        self,
        num_routed_experts: int,
        world_size: int,
        rank: int,
        layer_id: int,
        routed_scaling_factor: float = 1.0,
    ):
        self.num_routed_experts = num_routed_experts
        self.world_size = world_size
        self.rank = rank
        self.layer_id = layer_id
        self.old_experts_per_rank = num_routed_experts // world_size
        self.shared_weight = (
            1.0 / routed_scaling_factor if routed_scaling_factor != 0 else 1.0
        )
        self._counts_buf: Optional[Tensor] = None
        self.use_static_waterfill = not envs.SGLANG_DISABLE_STATIC_WATERFILL.get()
```
**EN:** This block defines `DeepEPWaterfillBalancer.__init__` and contains the main logic for this step. It mainly invokes `envs.SGLANG_DISABLE_STATIC_WATERFILL.get`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.num_routed_experts`, `self.world_size`, `self.rank`, `self.layer_id`, and `self.old_experts_per_rank` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `DeepEPWaterfillBalancer.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `envs.SGLANG_DISABLE_STATIC_WATERFILL.get`，说明该流程会编排底层辅助函数或计算内核。 像 `self.num_routed_experts`、`self.world_size`、`self.rank`、`self.layer_id` 以及 `self.old_experts_per_rank` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 385-409: Function `DeepEPWaterfillBalancer.count_local_routed` and its core logic
```python
    def count_local_routed(self, topk_ids: Tensor) -> Tensor:
        """Count routed tokens per rank via Triton kernel (uses original expert IDs)."""
        if self._counts_buf is None:
            self._counts_buf = torch.zeros(
                self.world_size, dtype=torch.int64, device=topk_ids.device
            )
        buf = self._counts_buf
        buf.zero_()
        num_tokens = topk_ids.shape[0]
        if num_tokens == 0:
            return buf
        topk = topk_ids.shape[1]
        BLOCK_SIZE = 256
        grid = ((num_tokens + BLOCK_SIZE - 1) // BLOCK_SIZE,)
        _count_routed_per_rank_kernel[grid](
            topk_ids,
            buf,
            num_tokens,
            topk,
            self.old_experts_per_rank,
            self.world_size,
            BLOCK_SIZE=BLOCK_SIZE,
        )
        return buf
```
**EN:** This block defines `DeepEPWaterfillBalancer.count_local_routed` and contains the main logic for this step. It mainly invokes `buf.zero_`, `_count_routed_per_rank_kernel`, and `torch.zeros`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `buf`, `num_tokens`, `topk`, `BLOCK_SIZE`, and `grid` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPWaterfillBalancer.count_local_routed`，并承载这一阶段的核心逻辑。 它主要调用 `buf.zero_`、`_count_routed_per_rank_kernel` 以及 `torch.zeros`，说明该流程会编排底层辅助函数或计算内核。 像 `buf`、`num_tokens`、`topk`、`BLOCK_SIZE` 以及 `grid` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 410-413: Internal helper `DeepEPWaterfillBalancer._is_low_batch`
```python
    def _is_low_batch(self, num_tokens: int) -> bool:
        """Return whether waterfill should skip balancing for small batches."""
        return num_tokens < self.MIN_BATCH_FOR_BALANCE
```
**EN:** This block defines `DeepEPWaterfillBalancer._is_low_batch` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPWaterfillBalancer._is_low_batch`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 414-417: Internal helper `DeepEPWaterfillBalancer._can_skip_dispatch_plan_for_low_batch`
```python
    def _can_skip_dispatch_plan_for_low_batch(self, num_tokens: int) -> bool:
        """Return whether static mode can skip dispatch-plan setup entirely."""
        return self.use_static_waterfill and self._is_low_batch(num_tokens)
```
**EN:** This block defines `DeepEPWaterfillBalancer._can_skip_dispatch_plan_for_low_batch` and contains the main logic for this step. It mainly invokes `self._is_low_batch`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPWaterfillBalancer._can_skip_dispatch_plan_for_low_batch`，并承载这一阶段的核心逻辑。 它主要调用 `self._is_low_batch`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 418-427: Internal helper `DeepEPWaterfillBalancer._build_static_dispatch_plan`
```python
    def _build_static_dispatch_plan(
        self, routed_counts: Tensor
    ) -> WaterfillDispatchPlan:
        """Build static-mode Waterfill inputs from current local routed counts."""
        return WaterfillDispatchPlan(
            rank_load=routed_counts,
            allow_all_ranks=True,
            target_total=0,
        )
```
**EN:** This block defines `DeepEPWaterfillBalancer._build_static_dispatch_plan` and contains the main logic for this step. It mainly invokes `WaterfillDispatchPlan`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPWaterfillBalancer._build_static_dispatch_plan`，并承载这一阶段的核心逻辑。 它主要调用 `WaterfillDispatchPlan`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 428-456: Internal helper `DeepEPWaterfillBalancer._build_dynamic_dispatch_plan`
```python
    def _build_dynamic_dispatch_plan(
        self,
        routed_counts: Tensor,
        local_tokens_per_rank: Optional[Tensor],
        topk: int,
    ) -> WaterfillDispatchPlan:
        """Build dynamic waterfill inputs from globally reduced routed counts."""
        # Dynamic Waterfill balances against effective rank load: globally
        # reduced routed counts plus each rank's active token count.
        rank_load = (
            routed_counts + local_tokens_per_rank
            if local_tokens_per_rank is not None
            else routed_counts
        )
        total_routed_t = routed_counts.sum()
        total_tokens_global_t = total_routed_t // topk
        total_effective_t = rank_load.sum()
        max_effective_t = rank_load.max()
        target_total = int(
            (total_effective_t + total_tokens_global_t + self.world_size - 1)
            // self.world_size
        )
        allow_all_ranks = bool(max_effective_t <= target_total)
        return WaterfillDispatchPlan(
            rank_load=rank_load,
            allow_all_ranks=allow_all_ranks,
            target_total=target_total,
        )
```
**EN:** This block defines `DeepEPWaterfillBalancer._build_dynamic_dispatch_plan` and contains the main logic for this step. It mainly invokes `routed_counts.sum`, `rank_load.sum`, `rank_load.max`, `int`, and `bool`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `rank_load`, `total_routed_t`, `total_tokens_global_t`, `total_effective_t`, and `max_effective_t` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPWaterfillBalancer._build_dynamic_dispatch_plan`，并承载这一阶段的核心逻辑。 它主要调用 `routed_counts.sum`、`rank_load.sum`、`rank_load.max`、`int` 以及 `bool`，说明该流程会编排底层辅助函数或计算内核。 像 `rank_load`、`total_routed_t`、`total_tokens_global_t`、`total_effective_t` 以及 `max_effective_t` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 457-477: Internal helper `DeepEPWaterfillBalancer._all_reduce_dynamic_rank_load`
```python
    @staticmethod
    def _all_reduce_dynamic_rank_load(
        local_routed_counts: Tensor, num_tokens: int
    ) -> Tuple[Tensor, Tensor]:
        """Aggregate dynamic load with SGLang EP communication."""
        from sglang.srt.distributed import get_moe_ep_group
        from sglang.srt.distributed.communication_op import (
            moe_expert_parallel_all_reduce,
        )

        group = get_moe_ep_group()
        world = group.world_size
        buf = torch.zeros(
            world * 2, dtype=torch.int64, device=local_routed_counts.device
        )
        buf[:world] = local_routed_counts
        rank = group.rank_in_group
        buf[world + rank : world + rank + 1].fill_(num_tokens)
        buf = moe_expert_parallel_all_reduce(buf)
        return buf[:world], buf[world:]
```
**EN:** This block defines `DeepEPWaterfillBalancer._all_reduce_dynamic_rank_load` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `get_moe_ep_group`, `torch.zeros`, `buf.fill_`, and `moe_expert_parallel_all_reduce`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `group`, `world`, `buf`, and `rank` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPWaterfillBalancer._all_reduce_dynamic_rank_load`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `get_moe_ep_group`、`torch.zeros`、`buf.fill_` 以及 `moe_expert_parallel_all_reduce`，说明该流程会编排底层辅助函数或计算内核。 像 `group`、`world`、`buf` 以及 `rank` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 478-498: Internal helper `DeepEPWaterfillBalancer._build_dispatch_plan`
```python
    def _build_dispatch_plan(
        self, topk_ids: Tensor, num_tokens: int
    ) -> Optional[WaterfillDispatchPlan]:
        """Prepare dispatch state for the waterfill selection boundary."""
        local_routed_counts = self.count_local_routed(topk_ids)
        if self.use_static_waterfill:
            return self._build_static_dispatch_plan(local_routed_counts)

        global_routed_counts, local_tokens_per_rank = (
            DeepEPWaterfillBalancer._all_reduce_dynamic_rank_load(
                local_routed_counts, num_tokens
            )
        )
        if self._is_low_batch(num_tokens):
            return None
        return self._build_dynamic_dispatch_plan(
            global_routed_counts,
            local_tokens_per_rank=local_tokens_per_rank,
            topk=topk_ids.shape[1],
        )
```
**EN:** This block defines `DeepEPWaterfillBalancer._build_dispatch_plan` and contains the main logic for this step. It mainly invokes `self.count_local_routed`, `DeepEPWaterfillBalancer._all_reduce_dynamic_rank_load`, `self._is_low_batch`, `self._build_dynamic_dispatch_plan`, and `self._build_static_dispatch_plan`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `local_routed_counts`, `global_routed_counts`, and `local_tokens_per_rank` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPWaterfillBalancer._build_dispatch_plan`，并承载这一阶段的核心逻辑。 它主要调用 `self.count_local_routed`、`DeepEPWaterfillBalancer._all_reduce_dynamic_rank_load`、`self._is_low_batch`、`self._build_dynamic_dispatch_plan` 以及 `self._build_static_dispatch_plan`，说明该流程会编排底层辅助函数或计算内核。 像 `local_routed_counts`、`global_routed_counts` 以及 `local_tokens_per_rank` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 499-531: Internal helper `DeepEPWaterfillBalancer._materialize_dispatch`
```python
    def _materialize_dispatch(
        self,
        topk_ids: Tensor,
        topk_weights: Tensor,
        dispatch_plan: WaterfillDispatchPlan,
    ) -> Tuple[Tensor, Tensor]:
        """Expand TopK using local expansion or fused Waterfill."""
        num_tokens = topk_ids.shape[0]
        if num_tokens == 0:
            return _empty_expanded(topk_ids, topk_weights)

        if self._is_low_batch(num_tokens):
            return expand_topk_with_shared_expert(
                topk_ids,
                topk_weights,
                self.num_routed_experts,
                self.world_size,
                self.rank,
                self.shared_weight,
            )

        return materialize_waterfill_dispatch_fused(
            topk_ids,
            topk_weights,
            dispatch_plan.rank_load,
            self.num_routed_experts,
            self.world_size,
            self.rank,
            self.shared_weight,
            allow_all_ranks=dispatch_plan.allow_all_ranks,
            target_total=dispatch_plan.target_total,
        )
```
**EN:** This block defines `DeepEPWaterfillBalancer._materialize_dispatch` and contains the main logic for this step. It mainly invokes `self._is_low_batch`, `materialize_waterfill_dispatch_fused`, `_empty_expanded`, and `expand_topk_with_shared_expert`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_tokens` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPWaterfillBalancer._materialize_dispatch`，并承载这一阶段的核心逻辑。 它主要调用 `self._is_low_batch`、`materialize_waterfill_dispatch_fused`、`_empty_expanded` 以及 `expand_topk_with_shared_expert`，说明该流程会编排底层辅助函数或计算内核。 像 `num_tokens` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 532-544: Internal helper `DeepEPWaterfillBalancer._with_expanded_topk`
```python
    @staticmethod
    def _with_expanded_topk(
        topk_output: StandardTopKOutput,
        expanded_ids: Tensor,
        expanded_weights: Tensor,
    ) -> StandardTopKOutput:
        """Wrap expanded tensors back into SGLang's StandardTopKOutput."""
        return StandardTopKOutput(
            topk_weights=expanded_weights,
            topk_ids=expanded_ids,
            router_logits=topk_output.router_logits,
        )
```
**EN:** This block defines `DeepEPWaterfillBalancer._with_expanded_topk` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `StandardTopKOutput`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPWaterfillBalancer._with_expanded_topk`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `StandardTopKOutput`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 545-557: Internal helper `DeepEPWaterfillBalancer._expand_local_shared`
```python
    def _expand_local_shared(
        self, topk_output: StandardTopKOutput
    ) -> StandardTopKOutput:
        expanded_ids, expanded_weights = expand_topk_with_shared_expert(
            topk_output.topk_ids,
            topk_output.topk_weights,
            self.num_routed_experts,
            self.world_size,
            self.rank,
            self.shared_weight,
        )
        return self._with_expanded_topk(topk_output, expanded_ids, expanded_weights)
```
**EN:** This block defines `DeepEPWaterfillBalancer._expand_local_shared` and contains the main logic for this step. It mainly invokes `expand_topk_with_shared_expert` and `self._with_expanded_topk`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `expanded_ids` and `expanded_weights` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPWaterfillBalancer._expand_local_shared`，并承载这一阶段的核心逻辑。 它主要调用 `expand_topk_with_shared_expert` 和 `self._with_expanded_topk`，说明该流程会编排底层辅助函数或计算内核。 像 `expanded_ids` 和 `expanded_weights` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 558-584: Function `DeepEPWaterfillBalancer.expand_topk` and its core logic
```python
    def expand_topk(
        self, topk_output: StandardTopKOutput, num_tokens: int
    ) -> StandardTopKOutput:
        """Expand topk [N, 8] -> [N, 9] with waterfill-assigned shared expert."""
        if self._can_skip_dispatch_plan_for_low_batch(num_tokens):
            # Static mode can use local expansion without communication for small
            # decode-sized batches. Dynamic mode still all-reduces before local
            # expansion so all ranks participate consistently.
            return self._expand_local_shared(topk_output)

        dispatch_plan = self._build_dispatch_plan(topk_output.topk_ids, num_tokens)
        if dispatch_plan is None:
            if num_tokens == 0:
                expanded_ids, expanded_weights = _empty_expanded(
                    topk_output.topk_ids, topk_output.topk_weights
                )
                return self._with_expanded_topk(
                    topk_output, expanded_ids, expanded_weights
                )
            else:
                return self._expand_local_shared(topk_output)
        expanded_ids, expanded_weights = self._materialize_dispatch(
            topk_output.topk_ids,
            topk_output.topk_weights,
            dispatch_plan,
        )
        return self._with_expanded_topk(topk_output, expanded_ids, expanded_weights)
```
**EN:** This block defines `DeepEPWaterfillBalancer.expand_topk` and contains the main logic for this step. It mainly invokes `self._can_skip_dispatch_plan_for_low_batch`, `self._build_dispatch_plan`, `self._materialize_dispatch`, `self._with_expanded_topk`, and `self._expand_local_shared`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `dispatch_plan`, `expanded_ids`, and `expanded_weights` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPWaterfillBalancer.expand_topk`，并承载这一阶段的核心逻辑。 它主要调用 `self._can_skip_dispatch_plan_for_low_batch`、`self._build_dispatch_plan`、`self._materialize_dispatch`、`self._with_expanded_topk` 以及 `self._expand_local_shared`，说明该流程会编排底层辅助函数或计算内核。 像 `dispatch_plan`、`expanded_ids` 以及 `expanded_weights` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `WaterfillDispatchPlan`, `_empty_expanded`, `_count_routed_per_rank_kernel`, `_waterfill_expand_kernel`, and `materialize_waterfill_dispatch_fused`. / **主要符号**：核心入口包括 `WaterfillDispatchPlan`、`_empty_expanded`、`_count_routed_per_rank_kernel`、`_waterfill_expand_kernel` 以及 `materialize_waterfill_dispatch_fused`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Sampling policy**: Covers post-processing of logits and token selection strategies. / **采样策略**：涵盖 logits 后处理与 token 选择策略。

## Dependencies / 依赖关系
- **Standard library**: `typing.NamedTuple`, `typing.Optional`, and `typing.Tuple` / **标准库**：`typing.NamedTuple`、`typing.Optional` 以及 `typing.Tuple`
- **Third-party**: `torch`, `triton`, `triton.language`, and `torch.Tensor` / **第三方依赖**：`torch`、`triton`、`triton.language` 以及 `torch.Tensor`
- **Internal SGLang modules**: `sglang.srt.environ.envs`, `sglang.srt.layers.moe.topk.StandardTopKOutput`, `sglang.srt.distributed.get_moe_ep_group`, and `sglang.srt.distributed.communication_op.moe_expert_parallel_all_reduce` / **SGLang 内部模块**：`sglang.srt.environ.envs`、`sglang.srt.layers.moe.topk.StandardTopKOutput`、`sglang.srt.distributed.get_moe_ep_group` 以及 `sglang.srt.distributed.communication_op.moe_expert_parallel_all_reduce`
