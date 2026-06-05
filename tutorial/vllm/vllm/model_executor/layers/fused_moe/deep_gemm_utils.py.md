# deep_gemm_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/deep_gemm_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Taken from https://github.com/ModelTC/LightLLM/blob/8ed97c74c18f11505b048b1ba00ba5c0cef8bff6/lightllm/common/fused_moe/deepep_scatter_gather.py and updated to fit vllm needs and terminology. / 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 8-14 — imports and setup
```python
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.model_executor.layers.fused_moe.utils import count_expert_num_tokens
from vllm.triton_utils import tl, triton
from vllm.utils.deep_gemm import get_mk_alignment_for_contiguous_layout
from vllm.utils.math_utils import round_up
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 17-23 — function `expert_num_tokens_round_up_and_sum`
```python
def expert_num_tokens_round_up_and_sum(
    expert_num_tokens: torch.Tensor, alignment: int
) -> int:
    # Round up each element in expert_num_tokens to the nearest multiple of
    # alignment.
    ent = (expert_num_tokens.to(torch.int64) + (alignment - 1)) // alignment * alignment
    return torch.sum(ent).item()
```
**EN:** This function defines `expert_num_tokens_round_up_and_sum`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `expert_num_tokens`, `alignment`. Key calls include `torch.sum.item`, `torch.sum`, `expert_num_tokens.to`. It writes or updates `ent`.
**CN:** 该函数定义 `expert_num_tokens_round_up_and_sum`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `expert_num_tokens`, `alignment`。 关键调用包括 `torch.sum.item`, `torch.sum`, `expert_num_tokens.to`。 它会写入或更新 `ent`。

### Lines 26-44 — function `compute_aligned_M`
```python
def compute_aligned_M(
    M: int,
    num_topk: int,
    local_num_experts: int,
    alignment: int,
    expert_tokens_meta: mk.ExpertTokensMetadata | None,
):
    if (expert_tokens_meta is not None) and (
        expert_tokens_meta.expert_num_tokens_cpu is not None
    ):
        return expert_num_tokens_round_up_and_sum(
            expert_tokens_meta.expert_num_tokens_cpu, alignment=alignment
        )

    # expert_num_tokens information is not available on the cpu.
    # compute the max required size.
    M_sum = (M * num_topk) + local_num_experts * (alignment - 1)
    M_sum = round_up(M_sum, alignment)
    return M_sum
```
**EN:** This function defines `compute_aligned_M`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `M`, `num_topk`, `local_num_experts`, `alignment`, `expert_tokens_meta`. Key calls include `round_up`, `expert_num_tokens_round_up_and_sum`. It writes or updates `M_sum`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `compute_aligned_M`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `M`, `num_topk`, `local_num_experts`, `alignment`, `expert_tokens_meta`。 关键调用包括 `round_up`, `expert_num_tokens_round_up_and_sum`。 它会写入或更新 `M_sum`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 47-51 — function `apply_expert_map`
```python
@triton.jit
def apply_expert_map(expert_id, expert_map):
    if expert_id != -1:
        expert_id = tl.load(expert_map + expert_id).to(expert_id.dtype)
    return expert_id
```
**EN:** This function defines `apply_expert_map`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `expert_id`, `expert_map`. Key calls include `tl.load.to`, `tl.load`. It writes or updates `expert_id`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `apply_expert_map`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `expert_id`, `expert_map`。 关键调用包括 `tl.load.to`, `tl.load`。 它会写入或更新 `expert_id`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 54-57 — function `round_up_128`
```python
@triton.jit
def round_up_128(x: int) -> int:
    y = 128
    return ((x + y - 1) // y) * y
```
**EN:** This function defines `round_up_128`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `x`. It writes or updates `y`.
**CN:** 该函数定义 `round_up_128`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `x`。 它会写入或更新 `y`。

### Lines 60-101 — function `_fwd_kernel_ep_scatter_1`
```python
@triton.jit
def _fwd_kernel_ep_scatter_1(
    num_recv_tokens_per_expert,
    expert_start_loc,
    m_indices,
    num_experts: tl.constexpr,
    BLOCK_E: tl.constexpr,
    BLOCK_EXPERT_NUM: tl.constexpr,
):
    cur_expert = tl.program_id(0)

    offset_cumsum = tl.arange(0, BLOCK_EXPERT_NUM)
    tokens_per_expert = tl.load(
        num_recv_tokens_per_expert + offset_cumsum,
        mask=offset_cumsum < num_experts,
        other=0,
    )
    tokens_per_expert = round_up_128(tokens_per_expert)
    cumsum = tl.cumsum(tokens_per_expert) - tokens_per_expert

    # Extract this block's offset from the register vector (warp shuffle,
    # no global memory round-trip) then write it once to expert_start_loc.
    cur_expert_start = tl.sum(
        tl.where(offset_cumsum == cur_expert, cumsum, tl.zeros_like(cumsum))
    )
    tl.store(expert_start_loc + cur_expert, cur_expert_start)
    cur_expert_token_num = tl.load(num_recv_tokens_per_expert + cur_expert)

    m_indices_start_ptr = m_indices + cur_expert_start
    off_expert = tl.arange(0, BLOCK_E)

    # any rows in the per-expert aligned region that do not correspond to
    # real tokens are left untouched here and should remain initialized to
    # -1 so DeepGEMM can skip them
    for start_m in tl.range(0, cur_expert_token_num, BLOCK_E):
        offs = start_m + off_expert
        mask = offs < cur_expert_token_num
        tl.store(
            m_indices_start_ptr + offs,
            cur_expert,
            mask=mask,
        )
```
**EN:** This function defines `_fwd_kernel_ep_scatter_1`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `num_recv_tokens_per_expert`, `expert_start_loc`, `m_indices`, `num_experts`, `BLOCK_E`, `BLOCK_EXPERT_NUM`. Key calls include `tl.program_id`, `tl.arange`, `tl.load`, `round_up_128`, `tl.sum`, `tl.store`. It writes or updates `cur_expert`, `offset_cumsum`, `tokens_per_expert`, `cumsum`, `cur_expert_start`, `cur_expert_token_num`. The body uses loops to cover different runtime cases.
**CN:** 该函数定义 `_fwd_kernel_ep_scatter_1`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `num_recv_tokens_per_expert`, `expert_start_loc`, `m_indices`, `num_experts`, `BLOCK_E`, `BLOCK_EXPERT_NUM`。 关键调用包括 `tl.program_id`, `tl.arange`, `tl.load`, `round_up_128`, `tl.sum`, `tl.store`。 它会写入或更新 `cur_expert`, `offset_cumsum`, `tokens_per_expert`, `cumsum`, `cur_expert_start`, `cur_expert_token_num`。 函数体通过循环来覆盖不同的运行时场景。

### Lines 104-171 — function `_fwd_kernel_ep_scatter_2`
```python
@triton.jit
def _fwd_kernel_ep_scatter_2(
    total_token_num,
    expert_start_loc,
    recv_x,
    recv_x_stride0,
    recv_x_stride1,
    recv_x_scale,
    recv_x_scale_stride0,
    recv_x_scale_stride1,
    recv_topk,
    recv_topk_stride0,
    recv_topk_stride1,
    output_tensor,
    output_tensor_stride0,
    output_tensor_stride1,
    output_tensor_scale,
    output_tensor_scale_stride0,
    output_tensor_scale_stride1,
    output_index,
    output_index_stride0,
    output_index_stride1,
    topk_num: tl.constexpr,
    expert_map,
    HAS_EXPERT_MAP: tl.constexpr,
    HIDDEN_SIZE: tl.constexpr,
    HIDDEN_SIZE_PAD: tl.constexpr,
    SCALE_HIDDEN_SIZE: tl.constexpr,
    SCALE_HIDDEN_SIZE_PAD: tl.constexpr,
):
    start_token_id = tl.program_id(0)
    grid_num = tl.num_programs(0)

    offset_in = tl.arange(0, HIDDEN_SIZE_PAD)
    mask = offset_in < HIDDEN_SIZE

    offset_in_s = tl.arange(0, SCALE_HIDDEN_SIZE_PAD)
    mask_s = offset_in_s < SCALE_HIDDEN_SIZE

    output_tensor_stride0 = output_tensor_stride0.to(tl.int64)

    for token_id in range(start_token_id, total_token_num, grid_num):
        to_copy = tl.load(recv_x + token_id * recv_x_stride0 + offset_in, mask=mask)
        to_copy_s = tl.load(
            recv_x_scale + token_id * recv_x_scale_stride0 + offset_in_s, mask=mask_s
        )

        for topk_index in tl.range(0, topk_num, 1, num_stages=4):
            expert_id = tl.load(recv_topk + token_id * recv_topk_stride0 + topk_index)

            if HAS_EXPERT_MAP:
                expert_id = apply_expert_map(expert_id, expert_map)

            if expert_id >= 0:
                dest_token_index = tl.atomic_add(expert_start_loc + expert_id, 1)
                dest_token_index_i64 = dest_token_index.to(tl.int64)
# ... omitted for brevity ...
                tl.store(output_tensor_ptr + offset_in, to_copy, mask=mask)
                tl.store(output_tensor_scale_ptr + offset_in_s, to_copy_s, mask=mask_s)
```
**EN:** This function defines `_fwd_kernel_ep_scatter_2`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `total_token_num`, `expert_start_loc`, `recv_x`, `recv_x_stride0`, `recv_x_stride1`, `recv_x_scale`. Key calls include `tl.program_id`, `tl.num_programs`, `tl.arange`, `output_tensor_stride0.to`, `range`, `tl.load`. It writes or updates `start_token_id`, `grid_num`, `offset_in`, `mask`, `offset_in_s`, `mask_s`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `_fwd_kernel_ep_scatter_2`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `total_token_num`, `expert_start_loc`, `recv_x`, `recv_x_stride0`, `recv_x_stride1`, `recv_x_scale`。 关键调用包括 `tl.program_id`, `tl.num_programs`, `tl.arange`, `output_tensor_stride0.to`, `range`, `tl.load`。 它会写入或更新 `start_token_id`, `grid_num`, `offset_in`, `mask`, `offset_in_s`, `mask_s`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 174-240 — function `ep_scatter`
```python
@torch.no_grad()
def ep_scatter(
    recv_x: torch.Tensor,
    recv_x_scale: torch.Tensor,
    recv_topk: torch.Tensor,
    num_recv_tokens_per_expert: torch.Tensor,
    expert_map: torch.Tensor | None,
    expert_start_loc: torch.Tensor,
    output_tensor: torch.Tensor,
    output_tensor_scale: torch.Tensor,
    m_indices: torch.Tensor,
    output_index: torch.Tensor,
):
    BLOCK_E = 128  # token num of per expert is aligned to 128
    BLOCK_D = 128  # block size of quantization
    num_warps = 8
    num_experts = num_recv_tokens_per_expert.shape[0]
    hidden_size = recv_x.shape[1]
    # grid = (triton.cdiv(hidden_size, BLOCK_D), num_experts)
    grid = num_experts

    assert m_indices.shape[0] % BLOCK_E == 0
    assert expert_start_loc.shape[0] == num_experts

    _fwd_kernel_ep_scatter_1[(grid,)](
        num_recv_tokens_per_expert,
        expert_start_loc,
        m_indices,
        num_experts=num_experts,
        num_warps=num_warps,
        BLOCK_E=BLOCK_E,
        BLOCK_EXPERT_NUM=triton.next_power_of_2(num_experts),
    )

    grid = min(recv_topk.shape[0], 1024 * 8)

    _fwd_kernel_ep_scatter_2[(grid,)](
        recv_topk.shape[0],
        expert_start_loc,
        recv_x,
        recv_x.stride(0),
        recv_x.stride(1),
        recv_x_scale,
        recv_x_scale.stride(0),
# ... omitted for brevity ...
    )
    return
```
**EN:** This function defines `ep_scatter`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `recv_x`, `recv_x_scale`, `recv_topk`, `num_recv_tokens_per_expert`, `expert_map`, `expert_start_loc`. Key calls include `torch.no_grad`, `_fwd_kernel_ep_scatter_1`, `min`, `_fwd_kernel_ep_scatter_2`, `recv_x.stride`, `recv_x_scale.stride`. It writes or updates `BLOCK_E`, `BLOCK_D`, `num_warps`, `num_experts`, `hidden_size`, `grid`.
**CN:** 该函数定义 `ep_scatter`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `recv_x`, `recv_x_scale`, `recv_topk`, `num_recv_tokens_per_expert`, `expert_map`, `expert_start_loc`。 关键调用包括 `torch.no_grad`, `_fwd_kernel_ep_scatter_1`, `min`, `_fwd_kernel_ep_scatter_2`, `recv_x.stride`, `recv_x_scale.stride`。 它会写入或更新 `BLOCK_E`, `BLOCK_D`, `num_warps`, `num_experts`, `hidden_size`, `grid`。

### Lines 243-302 — function `_fwd_kernel_ep_gather`
```python
@triton.jit
def _fwd_kernel_ep_gather(
    total_token_num,
    input_tensor,
    input_tensor_stride0,
    input_tensor_stride1,
    recv_topk_ids,
    recv_topk_ids_stride0,
    recv_topk_ids_stride1,
    recv_topk_weight,
    recv_topk_weight_stride0,
    recv_topk_weight_stride1,
    input_index,
    input_index_stride0,
    input_index_stride1,
    output_tensor,
    output_tensor_stride0,
    output_tensor_stride1,
    topk_num: tl.constexpr,
    expert_map,
    HAS_EXPERT_MAP: tl.constexpr,
    BLOCK_D: tl.constexpr,
):
    cur_block = tl.program_id(0)
    start_cur_token = tl.program_id(1)
    grid_num = tl.num_programs(1)

    for cur_token in range(start_cur_token, total_token_num, grid_num):
        off_d = tl.arange(0, BLOCK_D)
        accumulator = tl.zeros([BLOCK_D], dtype=tl.float32)
        for topk_index in range(0, topk_num):
            expert_id = tl.load(
                recv_topk_ids + cur_token * recv_topk_ids_stride0 + topk_index
            )

            if HAS_EXPERT_MAP:
                expert_id = apply_expert_map(expert_id, expert_map)

            if expert_id >= 0:
                source_token_index = tl.load(
                    input_index + cur_token * input_index_stride0 + topk_index
                )
                acc_weight = tl.load(
                    recv_topk_weight + cur_token * recv_topk_weight_stride0 + topk_index
                )
                tmp = tl.load(
                    input_tensor
                    + source_token_index * input_tensor_stride0
                    + cur_block * BLOCK_D
                    + off_d
                )
                accumulator += tmp.to(tl.float32) * acc_weight

        tl.store(
            output_tensor
            + cur_token * output_tensor_stride0
            + cur_block * BLOCK_D
            + off_d,
            accumulator.to(output_tensor.dtype.element_ty),
        )
```
**EN:** This function defines `_fwd_kernel_ep_gather`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `total_token_num`, `input_tensor`, `input_tensor_stride0`, `input_tensor_stride1`, `recv_topk_ids`, `recv_topk_ids_stride0`. Key calls include `tl.program_id`, `tl.num_programs`, `range`, `tl.arange`, `tl.zeros`, `tl.store`. It writes or updates `cur_block`, `start_cur_token`, `grid_num`, `off_d`, `accumulator`, `expert_id`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `_fwd_kernel_ep_gather`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `total_token_num`, `input_tensor`, `input_tensor_stride0`, `input_tensor_stride1`, `recv_topk_ids`, `recv_topk_ids_stride0`。 关键调用包括 `tl.program_id`, `tl.num_programs`, `range`, `tl.arange`, `tl.zeros`, `tl.store`。 它会写入或更新 `cur_block`, `start_cur_token`, `grid_num`, `off_d`, `accumulator`, `expert_id`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 305-344 — function `ep_gather`
```python
@torch.no_grad()
def ep_gather(
    input_tensor: torch.Tensor,
    recv_topk_ids: torch.Tensor,
    recv_topk_weight: torch.Tensor,
    input_index: torch.Tensor,
    expert_map: torch.Tensor | None,
    output_tensor: torch.Tensor,
):
    num_warps = 2
    num_tokens = output_tensor.shape[0]
    hidden_size = input_tensor.shape[1]
    BLOCK_D = min(hidden_size, 1024)
    assert hidden_size % BLOCK_D == 0
    grid = (triton.cdiv(hidden_size, BLOCK_D), min(num_tokens, 1024))

    _fwd_kernel_ep_gather[grid](
        num_tokens,
        input_tensor,
        input_tensor.stride(0),
        input_tensor.stride(1),
        recv_topk_ids,
        recv_topk_ids.stride(0),
        recv_topk_ids.stride(1),
        recv_topk_weight,
        recv_topk_weight.stride(0),
        recv_topk_weight.stride(1),
        input_index,
        input_index.stride(0),
        input_index.stride(1),
        output_tensor,
        output_tensor.stride(0),
        output_tensor.stride(1),
        topk_num=recv_topk_ids.shape[1],
        expert_map=expert_map,
        HAS_EXPERT_MAP=expert_map is not None,
        num_warps=num_warps,
        BLOCK_D=BLOCK_D,
    )
    return
```
**EN:** This function defines `ep_gather`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `input_tensor`, `recv_topk_ids`, `recv_topk_weight`, `input_index`, `expert_map`, `output_tensor`. Key calls include `torch.no_grad`, `min`, `_fwd_kernel_ep_gather`, `triton.cdiv`, `input_tensor.stride`, `recv_topk_ids.stride`. It writes or updates `num_warps`, `num_tokens`, `hidden_size`, `BLOCK_D`, `grid`.
**CN:** 该函数定义 `ep_gather`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `input_tensor`, `recv_topk_ids`, `recv_topk_weight`, `input_index`, `expert_map`, `output_tensor`。 关键调用包括 `torch.no_grad`, `min`, `_fwd_kernel_ep_gather`, `triton.cdiv`, `input_tensor.stride`, `recv_topk_ids.stride`。 它会写入或更新 `num_warps`, `num_tokens`, `hidden_size`, `BLOCK_D`, `grid`。

### Lines 347-417 — function `deepgemm_moe_permute`
```python
def deepgemm_moe_permute(
    aq: torch.Tensor,
    aq_scale: torch.Tensor,
    topk_ids: torch.Tensor,
    local_num_experts: int,
    expert_map: torch.Tensor | None,
    expert_tokens_meta: mk.ExpertTokensMetadata | None,
    aq_out: torch.Tensor | None = None,
):
    assert aq.ndim == 2
    assert topk_ids.dtype.is_signed, "The kernel uses -1 to represent invalid topk_ids"
    H = aq.size(1)
    device = aq.device

    block_m, block_k = get_mk_alignment_for_contiguous_layout()

    M_sum = compute_aligned_M(
        M=topk_ids.size(0),
        num_topk=topk_ids.size(1),
        local_num_experts=local_num_experts,
        alignment=block_m,
        expert_tokens_meta=expert_tokens_meta,
    )

    expert_start_loc = torch.empty(
        (local_num_experts), device=device, dtype=torch.int32
    )

    assert aq_out is None or aq_out.shape == (M_sum, H)
    if aq_out is None:
        aq_out = torch.empty((M_sum, H), device=device, dtype=aq.dtype)

    aq_scale_out = torch.empty(
        (M_sum, H // block_k), device=device, dtype=torch.float32
    )

    # DeepGEMM uses negative values in m_indices (here expert_ids) to mark
    # completely invalid / padded blocks that should be skipped. We always
    # initialize expert_ids to -1 so any row that is not explicitly written
    # by the scatter kernel will be treated as invalid and skipped by
    # DeepGEMM's scheduler.
    expert_ids = torch.full(
        (M_sum,),
        fill_value=-1,
# ... omitted for brevity ...

    return aq_out, aq_scale_out, expert_ids, inv_perm
```
**EN:** This function defines `deepgemm_moe_permute`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `aq`, `aq_scale`, `topk_ids`, `local_num_experts`, `expert_map`, `expert_tokens_meta`. Key calls include `aq.size`, `get_mk_alignment_for_contiguous_layout`, `compute_aligned_M`, `torch.empty`, `torch.full`, `ep_scatter`. It writes or updates `H`, `device`, `block_m`, `block_k`, `M_sum`, `expert_start_loc`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `deepgemm_moe_permute`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `aq`, `aq_scale`, `topk_ids`, `local_num_experts`, `expert_map`, `expert_tokens_meta`。 关键调用包括 `aq.size`, `get_mk_alignment_for_contiguous_layout`, `compute_aligned_M`, `torch.empty`, `torch.full`, `ep_scatter`。 它会写入或更新 `H`, `device`, `block_m`, `block_k`, `M_sum`, `expert_start_loc`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 420-435 — function `deepgemm_unpermute_and_reduce`
```python
def deepgemm_unpermute_and_reduce(
    a: torch.Tensor,  # Grouped gemm output
    topk_ids: torch.Tensor,
    topk_weights: torch.Tensor,
    inv_perm: torch.Tensor,
    expert_map: torch.Tensor | None,
    output: torch.Tensor,
):
    return ep_gather(
        input_tensor=a,
        recv_topk_ids=topk_ids,
        recv_topk_weight=topk_weights,
        input_index=inv_perm,
        expert_map=expert_map,
        output_tensor=output,
    )
```
**EN:** This function defines `deepgemm_unpermute_and_reduce`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `a`, `topk_ids`, `topk_weights`, `inv_perm`, `expert_map`, `output`. Key calls include `ep_gather`.
**CN:** 该函数定义 `deepgemm_unpermute_and_reduce`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `a`, `topk_ids`, `topk_weights`, `inv_perm`, `expert_map`, `output`。 关键调用包括 `ep_gather`。

## Key Concepts / 关键概念
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `expert_num_tokens_round_up_and_sum`, `compute_aligned_M`, `apply_expert_map`, `round_up_128` / [CN] 核心符号：`expert_num_tokens_round_up_and_sum`, `compute_aligned_M`, `apply_expert_map`, `round_up_128`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.triton_utils`, `vllm.utils.deep_gemm`, `vllm.utils.math_utils` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.triton_utils`, `vllm.utils.deep_gemm`, `vllm.utils.math_utils`
- **Runtime traits**: Triton kernels / **运行时特征**: Triton kernels
