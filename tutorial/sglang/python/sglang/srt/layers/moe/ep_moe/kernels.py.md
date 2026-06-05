# kernels.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/ep_moe/kernels.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `_get_launch_config_1d`, `_get_launch_config_2d`, `deepep_permute_triton_kernel`, and `deepep_post_reorder_triton_kernel` and connects them to backend-specific paths such as `CUDA`, `Triton`, `CUTLASS`, and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行。它提供了 `_get_launch_config_1d`、`_get_launch_config_2d`、`deepep_permute_triton_kernel` 以及 `deepep_post_reorder_triton_kernel` 等符号，并把这些符号连接到 `CUDA`、`Triton`、`CUTLASS` 以及 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Imports, conditional backend setup, and runtime guards
```python
import logging

import torch
import triton

from sglang.srt.utils import ceil_div, is_cuda, is_musa

logger = logging.getLogger(__name__)

_is_cuda = is_cuda()
_is_musa = is_musa()

if _is_cuda or _is_musa:
    from sglang.srt.layers.quantization.fp8_kernel import (
        sglang_per_token_group_quant_fp8 as per_token_group_quant_fp8,
    )

import triton.language as tl
```
**EN:** This section prepares the module namespace. It imports `logging`, `torch`, `triton`, `sglang.srt.utils.ceil_div`, `sglang.srt.utils.is_cuda`, and `sglang.srt.utils.is_musa`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `logger`, `_is_cuda`, and `_is_musa` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `logging`、`torch`、`triton`、`sglang.srt.utils.ceil_div`、`sglang.srt.utils.is_cuda` 以及 `sglang.srt.utils.is_musa`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `logger`、`_is_cuda` 以及 `_is_musa` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 21-47: Internal helper `_get_launch_config_1d`
```python
def _get_launch_config_1d(device, numel):
    MAX_THREADS_PER_BLOCK = 1024
    MIN_THREADS_PER_BLOCK = 512
    MAX_WAVES = 8  # empirical numbers

    props = torch.cuda.get_device_properties(device)
    sm_count = props.multi_processor_count
    max_threads_per_sm = props.max_threads_per_multi_processor
    max_num_blocks = sm_count * max_threads_per_sm // MAX_THREADS_PER_BLOCK

    block_dim = MAX_THREADS_PER_BLOCK

    def get_num_blocks(block_dim):
        return triton.cdiv(numel, block_dim)

    while (
        block_dim > MIN_THREADS_PER_BLOCK
        and get_num_blocks(block_dim // 2) <= max_num_blocks
    ):
        block_dim = block_dim // 2

    num_blocks = get_num_blocks(block_dim)
    grid_dim = min(num_blocks, max_num_blocks * MAX_WAVES)

    return (grid_dim,), block_dim
```
**EN:** This block defines `_get_launch_config_1d` and contains the main logic for this step. It mainly invokes `torch.cuda.get_device_properties`, `get_num_blocks`, `min`, and `triton.cdiv`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `MAX_THREADS_PER_BLOCK`, `MIN_THREADS_PER_BLOCK`, `MAX_WAVES`, `props`, and `sm_count` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_get_launch_config_1d`，并承载这一阶段的核心逻辑。 它主要调用 `torch.cuda.get_device_properties`、`get_num_blocks`、`min` 以及 `triton.cdiv`，说明该流程会编排底层辅助函数或计算内核。 像 `MAX_THREADS_PER_BLOCK`、`MIN_THREADS_PER_BLOCK`、`MAX_WAVES`、`props` 以及 `sm_count` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 48-74: Internal helper `_get_launch_config_2d`
```python
def _get_launch_config_2d(device, m, n):
    MAX_THREADS_PER_BLOCK = 1024
    MIN_THREADS_PER_BLOCK = 512
    MAX_WAVES = 8  # empirical numbers

    props = torch.cuda.get_device_properties(device)
    sm_count = props.multi_processor_count
    max_threads_per_sm = props.max_threads_per_multi_processor
    max_num_blocks = sm_count * max_threads_per_sm // MAX_THREADS_PER_BLOCK

    block_dim = MAX_THREADS_PER_BLOCK

    def get_num_blocks(block_dim):
        return m * triton.cdiv(n, block_dim)

    while (
        block_dim > MIN_THREADS_PER_BLOCK
        and get_num_blocks(block_dim // 2) <= max_num_blocks
    ):
        block_dim = block_dim // 2

    grid_dim_x = triton.cdiv(n, block_dim)
    grid_dim_y = max(min(m, max_num_blocks * MAX_WAVES // grid_dim_x), 1)

    return (grid_dim_y, grid_dim_x), block_dim
```
**EN:** This block defines `_get_launch_config_2d` and contains the main logic for this step. It mainly invokes `torch.cuda.get_device_properties`, `triton.cdiv`, `max`, `min`, and `get_num_blocks`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `MAX_THREADS_PER_BLOCK`, `MIN_THREADS_PER_BLOCK`, `MAX_WAVES`, `props`, and `sm_count` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_get_launch_config_2d`，并承载这一阶段的核心逻辑。 它主要调用 `torch.cuda.get_device_properties`、`triton.cdiv`、`max`、`min` 以及 `get_num_blocks`，说明该流程会编排底层辅助函数或计算内核。 像 `MAX_THREADS_PER_BLOCK`、`MIN_THREADS_PER_BLOCK`、`MAX_WAVES`、`props` 以及 `sm_count` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 75-105: Function `deepep_permute_triton_kernel` and its core logic
```python
@triton.jit
def deepep_permute_triton_kernel(
    input_ptr,
    gateup_input_ptr,
    src2dst_ptr,
    topk_ids_ptr,
    a1_scales_ptr,
    topk,
    hidden_size,
    BLOCK_SIZE: tl.constexpr,
):
    OutDtype = gateup_input_ptr.dtype.element_ty

    src_idx = tl.program_id(0)
    src2dst_ptr = src2dst_ptr + src_idx * topk
    topk_ids_ptr = topk_ids_ptr + src_idx * topk

    src_ptr = input_ptr + src_idx * hidden_size

    for start_offset in tl.range(0, hidden_size, BLOCK_SIZE):
        offset = start_offset + tl.arange(0, BLOCK_SIZE)
        mask = offset < hidden_size
        in_data = tl.load(src_ptr + offset, mask=mask).to(OutDtype)

        for idx in range(topk):
            dst_idx = tl.load(src2dst_ptr + idx)
            if dst_idx >= 0:
                dst_ptr = gateup_input_ptr + dst_idx * hidden_size
                tl.store(dst_ptr + offset, in_data, mask=mask)
```
**EN:** This block defines `deepep_permute_triton_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.range`, `tl.load.to`, `range`, and `tl.arange`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `OutDtype`, `src_idx`, `src2dst_ptr`, `topk_ids_ptr`, and `src_ptr` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `deepep_permute_triton_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.range`、`tl.load.to`、`range` 以及 `tl.arange`，说明该流程会编排底层辅助函数或计算内核。 像 `OutDtype`、`src_idx`、`src2dst_ptr`、`topk_ids_ptr` 以及 `src_ptr` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 106-138: Function `deepep_post_reorder_triton_kernel` and its core logic
```python
@triton.jit
def deepep_post_reorder_triton_kernel(
    down_output_ptr,
    output_ptr,
    src2dst_ptr,
    topk_ids_ptr,
    topk_weights_ptr,
    topk,
    hidden_size,
    BLOCK_SIZE: tl.constexpr,
):
    InDtype = down_output_ptr.dtype.element_ty

    src_idx = tl.program_id(0)
    src2dst_ptr = src2dst_ptr + src_idx * topk
    topk_ids_ptr = topk_ids_ptr + src_idx * topk
    topk_weights_ptr = topk_weights_ptr + src_idx * topk

    store_ptr = output_ptr + src_idx * hidden_size
    for start_offset in tl.range(0, hidden_size, BLOCK_SIZE):
        offset = start_offset + tl.arange(0, BLOCK_SIZE)
        mask = offset < hidden_size
        sum_vec = tl.zeros([BLOCK_SIZE], dtype=InDtype)
        for idx in range(topk):
            dst_idx = tl.load(src2dst_ptr + idx)
            if dst_idx >= 0:
                weigh_scale = tl.load(topk_weights_ptr + idx).to(InDtype)
                load_ptr = down_output_ptr + dst_idx * hidden_size
                in_data = tl.load(load_ptr + offset, mask=mask)
                sum_vec += in_data * weigh_scale
        tl.store(store_ptr + offset, sum_vec, mask=mask)
```
**EN:** This block defines `deepep_post_reorder_triton_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.range`, `tl.zeros`, `range`, and `tl.store`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `InDtype`, `src_idx`, `src2dst_ptr`, `topk_ids_ptr`, and `topk_weights_ptr` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `deepep_post_reorder_triton_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.range`、`tl.zeros`、`range` 以及 `tl.store`，说明该流程会编排底层辅助函数或计算内核。 像 `InDtype`、`src_idx`、`src2dst_ptr`、`topk_ids_ptr` 以及 `topk_weights_ptr` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 139-149: `compute_src2dst_triton_kernel` step for src 2 dst Triton kernel
```python
@triton.jit
def compute_src2dst_triton_kernel(
    reorder_ids, src2dst, num_toks, BLOCK_SIZE: tl.constexpr
):
    pid = tl.program_id(axis=0)
    dst_id = pid * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    mask = dst_id < num_toks
    src_id = tl.load(reorder_ids + dst_id, mask=mask)
    tl.store(src2dst + src_id, dst_id, mask=mask)
```
**EN:** This block defines `compute_src2dst_triton_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.load`, `tl.store`, and `tl.arange`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `dst_id`, `mask`, and `src_id` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `compute_src2dst_triton_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.load`、`tl.store` 以及 `tl.arange`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`dst_id`、`mask` 以及 `src_id` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 150-161: Function `deepep_compute_src2dst_triton_kernel` and its core logic
```python
@triton.jit
def deepep_compute_src2dst_triton_kernel(
    reorder_ids, src2dst, num_toks, num_minus_one, BLOCK_SIZE: tl.constexpr
):
    pid = tl.program_id(axis=0)
    dst_id = pid * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    mask = dst_id < num_toks
    src_id = tl.load(reorder_ids + dst_id, mask=mask)
    num_invalid = tl.load(num_minus_one)
    tl.store(src2dst + src_id, dst_id - num_invalid, mask=mask)
```
**EN:** This block defines `deepep_compute_src2dst_triton_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.load`, `tl.store`, and `tl.arange`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `dst_id`, `mask`, `src_id`, and `num_invalid` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `deepep_compute_src2dst_triton_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.load`、`tl.store` 以及 `tl.arange`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`dst_id`、`mask`、`src_id` 以及 `num_invalid` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 162-183: Function `deepep_run_moe_deep_preprocess` and its core logic
```python
def deepep_run_moe_deep_preprocess(topk_ids: torch.Tensor, num_experts: int):
    reorder_topk_ids, reorder_ids = torch.sort(topk_ids.view(-1), stable=True)
    seg_indptr = torch.empty(num_experts + 1, device=topk_ids.device, dtype=torch.int64)
    src2dst = torch.empty(topk_ids.numel(), device=topk_ids.device, dtype=torch.int64)

    # Find offset
    expert_ids = torch.arange(
        num_experts + 1, device=topk_ids.device, dtype=reorder_topk_ids.dtype
    )
    torch.searchsorted(reorder_topk_ids, expert_ids, out=seg_indptr)
    num_minus_one = seg_indptr[0]
    seg_indptr = seg_indptr - num_minus_one

    BLOCK_SIZE = 512
    grid = (triton.cdiv(topk_ids.numel(), BLOCK_SIZE),)
    deepep_compute_src2dst_triton_kernel[grid](
        reorder_ids, src2dst, topk_ids.numel(), num_minus_one, BLOCK_SIZE
    )
    reorder_topk_ids = reorder_topk_ids[num_minus_one:]
    return reorder_topk_ids, src2dst, seg_indptr
```
**EN:** This block defines `deepep_run_moe_deep_preprocess` and contains the main logic for this step. It mainly invokes `torch.sort`, `torch.empty`, `torch.arange`, `torch.searchsorted`, and `deepep_compute_src2dst_triton_kernel`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `reorder_topk_ids`, `reorder_ids`, `seg_indptr`, `src2dst`, and `expert_ids` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `deepep_run_moe_deep_preprocess`，并承载这一阶段的核心逻辑。 它主要调用 `torch.sort`、`torch.empty`、`torch.arange`、`torch.searchsorted` 以及 `deepep_compute_src2dst_triton_kernel`，说明该流程会编排底层辅助函数或计算内核。 像 `reorder_topk_ids`、`reorder_ids`、`seg_indptr`、`src2dst` 以及 `expert_ids` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 184-200: `compute_seg_indptr_triton_kernel` step for seg indptr Triton kernel
```python
@triton.jit
def compute_seg_indptr_triton_kernel(reorder_topk_ids, seg_indptr, num_toks):
    expert_id_minus_1 = tl.program_id(0) - 1
    low = 0
    high = num_toks - 1
    target_location = -1
    while low <= high:
        mid = (low + high) // 2

        if tl.load(reorder_topk_ids + mid) > expert_id_minus_1:
            high = mid - 1
        else:
            low = mid + 1
            target_location = mid
    tl.store(seg_indptr + expert_id_minus_1 + 1, target_location + 1)
```
**EN:** This block defines `compute_seg_indptr_triton_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.store`, `tl.program_id`, and `tl.load`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `expert_id_minus_1`, `low`, `high`, `target_location`, and `mid` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `compute_seg_indptr_triton_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.store`、`tl.program_id` 以及 `tl.load`，说明该流程会编排底层辅助函数或计算内核。 像 `expert_id_minus_1`、`low`、`high`、`target_location` 以及 `mid` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 201-213: Function `cutlass_w4_run_moe_ep_preproess` and its core logic
```python
def cutlass_w4_run_moe_ep_preproess(topk_ids: torch.Tensor):
    _, reorder_ids = torch.sort(topk_ids.view(-1), stable=True)

    BLOCK_SIZE = 512
    grid = (triton.cdiv(topk_ids.numel(), BLOCK_SIZE),)
    src2dst = torch.empty(topk_ids.numel(), device=topk_ids.device, dtype=torch.int32)
    compute_src2dst_triton_kernel[grid](
        reorder_ids, src2dst, topk_ids.numel(), BLOCK_SIZE
    )

    return src2dst
```
**EN:** This block defines `cutlass_w4_run_moe_ep_preproess` and contains the main logic for this step. It mainly invokes `torch.sort`, `torch.empty`, `compute_src2dst_triton_kernel`, `topk_ids.view`, and `triton.cdiv`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `_`, `reorder_ids`, `BLOCK_SIZE`, `grid`, and `src2dst` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `cutlass_w4_run_moe_ep_preproess`，并承载这一阶段的核心逻辑。 它主要调用 `torch.sort`、`torch.empty`、`compute_src2dst_triton_kernel`、`topk_ids.view` 以及 `triton.cdiv`，说明该流程会编排底层辅助函数或计算内核。 像 `_`、`reorder_ids`、`BLOCK_SIZE`、`grid` 以及 `src2dst` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 214-258: Function `pre_reorder_triton_kernel_for_cutlass_moe` and its core logic
```python
@triton.jit
def pre_reorder_triton_kernel_for_cutlass_moe(
    input_ptr,
    gateup_input_ptr,
    src2dst_ptr,
    topk_ids_ptr,
    a1_scales_ptr,
    num_local_experts,
    topk,
    num_tokens,
    hidden_size,
    BLOCK_SIZE: tl.constexpr,
    NUM_STAGES: tl.constexpr,
):
    OutDtype = gateup_input_ptr.dtype.element_ty

    if a1_scales_ptr is not None:
        a1_scale = 1.0 / tl.load(a1_scales_ptr)
    else:
        a1_scale = 1.0

    offset = BLOCK_SIZE * tl.program_id(1) + tl.arange(0, BLOCK_SIZE)
    mask = offset < hidden_size

    start_src_idx = tl.program_id(0)
    step = tl.num_programs(0)

    for src_idx_int32 in tl.range(
        start_src_idx, num_tokens, step, num_stages=NUM_STAGES
    ):
        src_idx = src_idx_int32.to(tl.int64)
        token_src2dst_ptr = src2dst_ptr + src_idx * topk
        token_topk_ids_ptr = topk_ids_ptr + src_idx * topk

        src_ptr_offs = input_ptr + src_idx * hidden_size + offset
        dst_ptr_offs = gateup_input_ptr + offset
        in_data = tl.load(src_ptr_offs, mask=mask).to(tl.float32)
        out_data = (in_data * a1_scale).to(OutDtype)
        for idx in range(topk):
            expert_id = tl.load(token_topk_ids_ptr + idx)
            if expert_id != num_local_experts:
                dst_idx = tl.load(token_src2dst_ptr + idx)
                tl.store(dst_ptr_offs + dst_idx * hidden_size, out_data, mask=mask)
```
**EN:** This block defines `pre_reorder_triton_kernel_for_cutlass_moe` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.num_programs`, `tl.range`, `tl.arange`, and `src_idx_int32.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `OutDtype`, `offset`, `mask`, `start_src_idx`, and `step` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `pre_reorder_triton_kernel_for_cutlass_moe`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.num_programs`、`tl.range`、`tl.arange` 以及 `src_idx_int32.to`，说明该流程会编排底层辅助函数或计算内核。 像 `OutDtype`、`offset`、`mask`、`start_src_idx` 以及 `step` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 259-287: Function `pre_reorder_for_cutlass_moe` and its core logic
```python
def pre_reorder_for_cutlass_moe(
    input,
    gateup_input,
    src2dst,
    topk_ids,
    a1_scales,
    num_local_experts,
    topk,
    num_tokens,
    hidden_size,
):
    grid, block_dim = _get_launch_config_2d(input.device, num_tokens, hidden_size)

    pre_reorder_triton_kernel_for_cutlass_moe[grid](
        input_ptr=input,
        gateup_input_ptr=gateup_input,
        src2dst_ptr=src2dst,
        topk_ids_ptr=topk_ids,
        a1_scales_ptr=a1_scales,
        num_local_experts=num_local_experts,
        topk=topk,
        num_tokens=num_tokens,
        hidden_size=hidden_size,
        BLOCK_SIZE=block_dim,
        NUM_STAGES=3,
    )


# copy from https://github.com/ModelTC/lightllm/blob/a000ab69098654df4731f5b12587dd4e7f0a4f41/lightllm/common/fused_moe/moe_silu_and_mul_mix_quant_ep.py
```
**EN:** This block defines `pre_reorder_for_cutlass_moe` and contains the main logic for this step. It mainly invokes `_get_launch_config_2d` and `pre_reorder_triton_kernel_for_cutlass_moe`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `grid` and `block_dim` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `pre_reorder_for_cutlass_moe`，并承载这一阶段的核心逻辑。 它主要调用 `_get_launch_config_2d` 和 `pre_reorder_triton_kernel_for_cutlass_moe`，说明该流程会编排底层辅助函数或计算内核。 像 `grid` 和 `block_dim` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 288-365: Internal helper `_silu_and_mul_post_quant_kernel`
```python
@triton.jit
def _silu_and_mul_post_quant_kernel(
    input_ptr,
    stride_input_0,
    stride_input_1,
    stride_input_2,
    output_ptr,
    stride_output_0,
    stride_output_1,
    stride_output_2,
    output_scale_ptr,
    stride_output_scale_0,
    stride_output_scale_1,
    stride_output_scale_2,
    masked_m_ptr,
    size_n,
    fp8_max,
    fp8_min,
    BLOCK_N: tl.constexpr,
    NUM_STAGE: tl.constexpr,
    SCALE_UE8M0: tl.constexpr,
):
    expert_id = tl.program_id(2)
    token_id = tl.program_id(1)
    hidden_dim_block_index = tl.program_id(0)

    block_num_per_expert = tl.num_programs(1)

    token_num_cur_expert = tl.load(masked_m_ptr + expert_id)

    stride_input_0 = tl.cast(stride_input_0, dtype=tl.int64)
    stride_output_0 = tl.cast(stride_output_0, dtype=tl.int64)
    stride_input_1 = tl.cast(stride_input_1, dtype=tl.int64)
    stride_output_1 = tl.cast(stride_output_1, dtype=tl.int64)

    offs_in_d = hidden_dim_block_index * BLOCK_N + tl.arange(0, BLOCK_N)
    input_ptr_offs = input_ptr + expert_id * stride_input_0 + offs_in_d
    output_ptr_offs = output_ptr + expert_id * stride_output_0 + offs_in_d
    output_scale_offs = (
        output_scale_ptr
        + expert_id * stride_output_scale_0
        + hidden_dim_block_index * stride_output_scale_2
    )

    for token_index in tl.range(
        token_id, token_num_cur_expert, block_num_per_expert, num_stages=NUM_STAGE
    ):
        gate = tl.load(
            input_ptr_offs + token_index * stride_input_1,
            mask=offs_in_d < size_n,
            other=0.0,
        ).to(tl.float32)
        up = tl.load(
            input_ptr_offs + token_index * stride_input_1 + size_n,
            mask=offs_in_d < size_n,
            other=0.0,
        )
        gate = gate / (1 + tl.exp(-gate))
        gate = gate.to(input_ptr.dtype.element_ty)
        gate_up = up * gate
        _absmax = tl.maximum(tl.max(tl.abs(gate_up)), 1e-10)
        output_s = _absmax / fp8_max
        if SCALE_UE8M0:
            output_s = tl.exp2(tl.ceil(tl.log2(tl.abs(output_s))))
        output_q = tl.clamp(gate_up / output_s, fp8_min, fp8_max).to(
            output_ptr.dtype.element_ty
        )
        tl.store(
            output_ptr_offs + token_index * stride_output_1,
            output_q,
            mask=offs_in_d < size_n,
        )
        tl.store(
            output_scale_offs + token_index * stride_output_scale_1,
            output_s,
        )
```
**EN:** This block defines `_silu_and_mul_post_quant_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.num_programs`, `tl.load`, `tl.cast`, and `tl.range`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `expert_id`, `token_id`, `hidden_dim_block_index`, `block_num_per_expert`, and `token_num_cur_expert` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_silu_and_mul_post_quant_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.num_programs`、`tl.load`、`tl.cast` 以及 `tl.range`，说明该流程会编排底层辅助函数或计算内核。 像 `expert_id`、`token_id`、`hidden_dim_block_index`、`block_num_per_expert` 以及 `token_num_cur_expert` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 366-433: Function `silu_and_mul_masked_post_quant_fwd` and its core logic
```python
def silu_and_mul_masked_post_quant_fwd(
    input: torch.Tensor,
    output: torch.Tensor,
    output_scale: torch.Tensor,
    quant_group_size: int,
    masked_m: torch.Tensor,
    scale_ue8m0: bool = False,
):
    """
    input shape [expert_num, token_num_padded, hidden_dim]
    output shape [expert_num, token_num_padded, hidden_dim // 2], dtype fp8
    output_scale [expert_num token_num_paddded, hidden_dim // 2 // 128] dtype float32
    quant_group_size  int,
    masked_m shape [expert_num],
    """

    assert input.is_contiguous()
    assert output.dtype == torch.float8_e4m3fn
    assert output.is_contiguous()
    assert len(input.shape) == 3
    assert input.shape[0] == masked_m.shape[0]
    assert input.shape[-1] % 2 == 0

    size_n = input.shape[-1] // 2
    assert size_n % quant_group_size == 0

    expert_num = len(masked_m)

    if expert_num < 4:
        BLOCK_NUM_PER_EXPERT = 64
    else:
        BLOCK_NUM_PER_EXPERT = 32

    BLOCK_N = quant_group_size
    num_warps = 1
    NUM_STAGES = 6
    hidden_dim_split_block_num = triton.cdiv(size_n, BLOCK_N)
    assert BLOCK_N % quant_group_size == 0

    grid = (
        hidden_dim_split_block_num,
        BLOCK_NUM_PER_EXPERT,
        expert_num,
    )

    finfo = torch.finfo(torch.float8_e4m3fn)
    fp8_max = finfo.max
    fp8_min = -fp8_max

    _silu_and_mul_post_quant_kernel[grid](
        input,
        *input.stride(),
        output,
        *output.stride(),
        output_scale,
        *output_scale.stride(),
        masked_m,
        size_n,
        fp8_max,
        fp8_min,
        BLOCK_N=BLOCK_N,
        NUM_STAGE=NUM_STAGES,
        num_warps=num_warps,
        SCALE_UE8M0=scale_ue8m0,
    )
    return
```
**EN:** This block defines `silu_and_mul_masked_post_quant_fwd` and contains the main logic for this step. It mainly invokes `input.is_contiguous`, `output.is_contiguous`, `len`, `triton.cdiv`, and `torch.finfo`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `size_n`, `expert_num`, `BLOCK_N`, `num_warps`, and `NUM_STAGES` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `silu_and_mul_masked_post_quant_fwd`，并承载这一阶段的核心逻辑。 它主要调用 `input.is_contiguous`、`output.is_contiguous`、`len`、`triton.cdiv` 以及 `torch.finfo`，说明该流程会编排底层辅助函数或计算内核。 像 `size_n`、`expert_num`、`BLOCK_N`、`num_warps` 以及 `NUM_STAGES` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 434-488: Internal helper `_silu_and_mul_kernel`
```python
@triton.jit
def _silu_and_mul_kernel(
    input_ptr,
    stride_input_0,
    stride_input_1,
    stride_input_2,
    output_ptr,
    stride_output_0,
    stride_output_1,
    stride_output_2,
    masked_m_ptr,
    size_n,
    BLOCK_N: tl.constexpr,
    NUM_STAGE: tl.constexpr,
):
    expert_id = tl.program_id(2)
    token_id = tl.program_id(1)
    hidden_dim_block_index = tl.program_id(0)

    block_num_per_expert = tl.num_programs(1)

    token_num_cur_expert = tl.load(masked_m_ptr + expert_id)

    stride_input_0 = tl.cast(stride_input_0, dtype=tl.int64)
    stride_output_0 = tl.cast(stride_output_0, dtype=tl.int64)
    stride_input_1 = tl.cast(stride_input_1, dtype=tl.int64)
    stride_output_1 = tl.cast(stride_output_1, dtype=tl.int64)

    offs_in_d = hidden_dim_block_index * BLOCK_N + tl.arange(0, BLOCK_N)
    input_ptr_offs = input_ptr + expert_id * stride_input_0 + offs_in_d
    output_ptr_offs = output_ptr + expert_id * stride_output_0 + offs_in_d

    for token_index in tl.range(
        token_id, token_num_cur_expert, block_num_per_expert, num_stages=NUM_STAGE
    ):
        gate = tl.load(
            input_ptr_offs + token_index * stride_input_1,
            mask=offs_in_d < size_n,
            other=0.0,
        ).to(tl.float32)
        up = tl.load(
            input_ptr_offs + token_index * stride_input_1 + size_n,
            mask=offs_in_d < size_n,
            other=0.0,
        )
        gate = gate / (1 + tl.exp(-gate))
        gate = gate.to(input_ptr.dtype.element_ty)
        gate_up = up * gate
        tl.store(
            output_ptr_offs + token_index * stride_output_1,
            gate_up,
            mask=offs_in_d < size_n,
        )
```
**EN:** This block defines `_silu_and_mul_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.num_programs`, `tl.load`, `tl.cast`, and `tl.range`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `expert_id`, `token_id`, `hidden_dim_block_index`, `block_num_per_expert`, and `token_num_cur_expert` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_silu_and_mul_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.num_programs`、`tl.load`、`tl.cast` 以及 `tl.range`，说明该流程会编排底层辅助函数或计算内核。 像 `expert_id`、`token_id`、`hidden_dim_block_index`、`block_num_per_expert` 以及 `token_num_cur_expert` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 489-541: Function `silu_and_mul_masked_fwd` and its core logic
```python
def silu_and_mul_masked_fwd(
    input: torch.Tensor,
    output: torch.Tensor,
    masked_m: torch.Tensor,
):
    """
    input shape [expert_num, token_num_padded, hidden_dim], dtype bf16
    output shape [expert_num, token_num_padded, hidden_dim // 2], dtype bf16
    masked_m shape [expert_num]
    """

    assert input.is_contiguous()
    assert output.dtype == torch.bfloat16
    assert input.dtype == torch.bfloat16
    assert output.is_contiguous()
    assert len(input.shape) == 3
    assert input.shape[0] == masked_m.shape[0]
    assert input.shape[-1] % 2 == 0

    size_n = input.shape[-1] // 2
    expert_num = len(masked_m)

    if expert_num < 4:
        BLOCK_NUM_PER_EXPERT = 64
    else:
        BLOCK_NUM_PER_EXPERT = 32

    BLOCK_N = 128
    num_warps = 4
    NUM_STAGES = 4

    hidden_dim_split_block_num = triton.cdiv(size_n, BLOCK_N)

    grid = (
        hidden_dim_split_block_num,
        BLOCK_NUM_PER_EXPERT,
        expert_num,
    )

    _silu_and_mul_kernel[grid](
        input,
        *input.stride(),
        output,
        *output.stride(),
        masked_m,
        size_n,
        BLOCK_N=BLOCK_N,
        NUM_STAGE=NUM_STAGES,
        num_warps=num_warps,
    )
    return output
```
**EN:** This block defines `silu_and_mul_masked_fwd` and contains the main logic for this step. It mainly invokes `input.is_contiguous`, `output.is_contiguous`, `len`, `triton.cdiv`, and `_silu_and_mul_kernel`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `size_n`, `expert_num`, `BLOCK_N`, `num_warps`, and `NUM_STAGES` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `silu_and_mul_masked_fwd`，并承载这一阶段的核心逻辑。 它主要调用 `input.is_contiguous`、`output.is_contiguous`、`len`、`triton.cdiv` 以及 `_silu_and_mul_kernel`，说明该流程会编排底层辅助函数或计算内核。 像 `size_n`、`expert_num`、`BLOCK_N`、`num_warps` 以及 `NUM_STAGES` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 542-574: Function `silu_mul_static_tensorwise_quant_triton_kernel_for_cutlass_moe` and its core logic
```python
@triton.jit
def silu_mul_static_tensorwise_quant_triton_kernel_for_cutlass_moe(
    input_ptr,
    output_ptr,
    scale_ptr,
    num_tokens_tensor_ptr,
    intermediate_size,
    BLOCK_SIZE: tl.constexpr,
    NUM_STAGES: tl.constexpr,
):
    OutDtype = output_ptr.dtype.element_ty

    num_tokens = tl.load(num_tokens_tensor_ptr)
    numel = num_tokens * intermediate_size
    gate_ptr = input_ptr
    up_ptr = input_ptr + intermediate_size
    scale = 1.0 / tl.load(scale_ptr)

    start_idx = tl.program_id(0) * BLOCK_SIZE
    step = tl.num_programs(0) * BLOCK_SIZE

    for id in tl.range(start_idx, numel, step, num_stages=NUM_STAGES):
        ids = id + tl.arange(0, BLOCK_SIZE)
        token_ids = ids // intermediate_size
        mask = ids < numel

        offs = ids + token_ids * intermediate_size
        gate = tl.load(gate_ptr + offs, mask=mask, other=0.0).to(tl.float32)
        up = tl.load(up_ptr + offs, mask=mask, other=0.0).to(tl.float32)
        output = gate / (1 + tl.exp(-gate)) * up * scale
        tl.store(output_ptr + ids, output.to(OutDtype), mask=mask)
```
**EN:** This block defines `silu_mul_static_tensorwise_quant_triton_kernel_for_cutlass_moe` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.load`, `tl.range`, `tl.program_id`, `tl.num_programs`, and `tl.load.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `OutDtype`, `num_tokens`, `numel`, `gate_ptr`, and `up_ptr` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `silu_mul_static_tensorwise_quant_triton_kernel_for_cutlass_moe`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.load`、`tl.range`、`tl.program_id`、`tl.num_programs` 以及 `tl.load.to`，说明该流程会编排底层辅助函数或计算内核。 像 `OutDtype`、`num_tokens`、`numel`、`gate_ptr` 以及 `up_ptr` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 575-597: Function `silu_mul_static_tensorwise_quant_for_cutlass_moe` and its core logic
```python
def silu_mul_static_tensorwise_quant_for_cutlass_moe(
    input: torch.Tensor,
    output: torch.Tensor,
    scale: torch.Tensor,
    num_tokens_tensor: torch.Tensor,
    expected_num_tokens: int,
    intermediate_size: int,
):
    grid, block_dim = _get_launch_config_1d(
        input.device, expected_num_tokens * intermediate_size
    )

    silu_mul_static_tensorwise_quant_triton_kernel_for_cutlass_moe[grid](
        input_ptr=input,
        output_ptr=output,
        scale_ptr=scale,
        num_tokens_tensor_ptr=num_tokens_tensor,
        intermediate_size=intermediate_size,
        BLOCK_SIZE=block_dim,
        NUM_STAGES=3,
    )
```
**EN:** This block defines `silu_mul_static_tensorwise_quant_for_cutlass_moe` and contains the main logic for this step. It mainly invokes `_get_launch_config_1d` and `silu_mul_static_tensorwise_quant_triton_kernel_for_cutlass_moe`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `grid` and `block_dim` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `silu_mul_static_tensorwise_quant_for_cutlass_moe`，并承载这一阶段的核心逻辑。 它主要调用 `_get_launch_config_1d` 和 `silu_mul_static_tensorwise_quant_triton_kernel_for_cutlass_moe`，说明该流程会编排底层辅助函数或计算内核。 像 `grid` 和 `block_dim` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 598-647: Function `post_reorder_triton_kernel_for_cutlass_moe` and its core logic
```python
@triton.jit
def post_reorder_triton_kernel_for_cutlass_moe(
    down_output_ptr,
    output_ptr,
    src2dst_ptr,
    topk_ids_ptr,
    topk_weights_ptr,
    num_local_experts,
    topk,
    num_tokens,
    hidden_size,
    routed_scaling_factor: float,
    BLOCK_SIZE: tl.constexpr,
    NUM_STAGES: tl.constexpr,
):
    OutDtype = output_ptr.dtype.element_ty

    offset = BLOCK_SIZE * tl.program_id(1) + tl.arange(0, BLOCK_SIZE)
    mask = offset < hidden_size

    down_output_ptr_offs = down_output_ptr + offset
    output_ptr_offs = output_ptr + offset

    start_src_idx = tl.program_id(0)
    step = tl.num_programs(0)

    for src_idx_int32 in tl.range(
        start_src_idx, num_tokens, step, num_stages=NUM_STAGES
    ):
        src_idx = src_idx_int32.to(tl.int64)
        token_src2dst_ptr = src2dst_ptr + src_idx * topk
        token_topk_ids_ptr = topk_ids_ptr + src_idx * topk
        token_topk_weights_ptr = topk_weights_ptr + src_idx * topk

        sum_vec = tl.zeros([BLOCK_SIZE], dtype=tl.float32)
        for idx in range(topk):
            expert_id = tl.load(token_topk_ids_ptr + idx)
            if expert_id != num_local_experts:
                dst_idx_int32 = tl.load(token_src2dst_ptr + idx)
                dst_idx = dst_idx_int32.to(tl.int64)
                dst_idx = dst_idx
                weight_scale = tl.load(token_topk_weights_ptr + idx).to(tl.float32)
                load_ptr_offs = down_output_ptr_offs + dst_idx * hidden_size
                in_data = tl.load(load_ptr_offs, mask=mask).to(tl.float32)
                sum_vec += in_data * weight_scale
        sum_vec *= routed_scaling_factor
        store_ptr_offs = output_ptr_offs + src_idx * hidden_size
        tl.store(store_ptr_offs, sum_vec.to(OutDtype), mask=mask)
```
**EN:** This block defines `post_reorder_triton_kernel_for_cutlass_moe` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.num_programs`, `tl.range`, `tl.arange`, and `src_idx_int32.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `OutDtype`, `offset`, `mask`, `down_output_ptr_offs`, and `output_ptr_offs` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `post_reorder_triton_kernel_for_cutlass_moe`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.num_programs`、`tl.range`、`tl.arange` 以及 `src_idx_int32.to`，说明该流程会编排底层辅助函数或计算内核。 像 `OutDtype`、`offset`、`mask`、`down_output_ptr_offs` 以及 `output_ptr_offs` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 648-677: Function `post_reorder_for_cutlass_moe` and its core logic
```python
def post_reorder_for_cutlass_moe(
    down_output,
    output,
    src2dst,
    topk_ids,
    topk_weights,
    num_local_experts,
    topk,
    num_tokens,
    hidden_size,
    routed_scaling_factor: float,
):
    grid, block_dim = _get_launch_config_2d(down_output.device, num_tokens, hidden_size)

    post_reorder_triton_kernel_for_cutlass_moe[grid](
        down_output_ptr=down_output,
        output_ptr=output,
        src2dst_ptr=src2dst,
        topk_ids_ptr=topk_ids,
        topk_weights_ptr=topk_weights,
        num_local_experts=num_local_experts,
        topk=topk,
        num_tokens=num_tokens,
        hidden_size=hidden_size,
        routed_scaling_factor=routed_scaling_factor,
        BLOCK_SIZE=block_dim,
        NUM_STAGES=3,
    )
```
**EN:** This block defines `post_reorder_for_cutlass_moe` and contains the main logic for this step. It mainly invokes `_get_launch_config_2d` and `post_reorder_triton_kernel_for_cutlass_moe`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `grid` and `block_dim` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `post_reorder_for_cutlass_moe`，并承载这一阶段的核心逻辑。 它主要调用 `_get_launch_config_2d` 和 `post_reorder_triton_kernel_for_cutlass_moe`，说明该流程会编排底层辅助函数或计算内核。 像 `grid` 和 `block_dim` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 678-717: Function `post_reorder_triton_kernel` and its core logic
```python
@triton.jit
def post_reorder_triton_kernel(
    down_output_ptr,
    output_ptr,
    src2dst_ptr,
    topk_ids_ptr,
    topk_weights_ptr,
    topk,
    hidden_size,
    BLOCK_SIZE: tl.constexpr,
):
    InDtype = down_output_ptr.dtype.element_ty

    src_idx_int32 = tl.program_id(0)
    src_idx = src_idx_int32.to(tl.int64)
    src2dst_ptr = src2dst_ptr + src_idx * topk
    topk_ids_ptr = topk_ids_ptr + src_idx * topk
    topk_weights_ptr = topk_weights_ptr + src_idx * topk

    store_ptr = output_ptr + src_idx * hidden_size

    vec = tl.arange(0, BLOCK_SIZE)

    for start_offset in tl.range(0, hidden_size, BLOCK_SIZE):
        offset = start_offset + vec
        mask = offset < hidden_size

        sum_vec = tl.zeros([BLOCK_SIZE], dtype=InDtype)
        for idx in range(topk):
            expert_id = tl.load(topk_ids_ptr + idx)
            if expert_id > 0:
                dst_idx_int32 = tl.load(src2dst_ptr + idx)
                dst_idx = dst_idx_int32.to(tl.int64)
                weigh_scale = tl.load(topk_weights_ptr + idx).to(InDtype)
                load_ptr = down_output_ptr + dst_idx * hidden_size
                in_data = tl.load(load_ptr + offset, mask=mask)
                sum_vec += in_data * weigh_scale
        tl.store(store_ptr + offset, sum_vec, mask=mask)
```
**EN:** This block defines `post_reorder_triton_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `src_idx_int32.to`, `tl.arange`, `tl.range`, and `tl.zeros`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `InDtype`, `src_idx_int32`, `src_idx`, `src2dst_ptr`, and `topk_ids_ptr` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `post_reorder_triton_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`src_idx_int32.to`、`tl.arange`、`tl.range` 以及 `tl.zeros`，说明该流程会编排底层辅助函数或计算内核。 像 `InDtype`、`src_idx_int32`、`src_idx`、`src2dst_ptr` 以及 `topk_ids_ptr` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 718-750: Internal helper `_fwd_kernel_ep_scatter_1`
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
    cumsum = tl.cumsum(tokens_per_expert) - tokens_per_expert
    tl.store(expert_start_loc + offset_cumsum, cumsum, mask=offset_cumsum < num_experts)

    cur_expert_start = tl.load(expert_start_loc + cur_expert)
    cur_expert_token_num = tl.load(num_recv_tokens_per_expert + cur_expert)

    m_indices_start_ptr = m_indices + cur_expert_start
    off_expert = tl.arange(0, BLOCK_E)

    for start_m in tl.range(0, cur_expert_token_num, BLOCK_E, num_stages=4):
        tl.store(
            m_indices_start_ptr + start_m + off_expert,
            cur_expert,
        )
```
**EN:** This block defines `_fwd_kernel_ep_scatter_1` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.arange`, `tl.load`, `tl.store`, and `tl.range`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `cur_expert`, `offset_cumsum`, `tokens_per_expert`, `cumsum`, and `cur_expert_start` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_fwd_kernel_ep_scatter_1`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.arange`、`tl.load`、`tl.store` 以及 `tl.range`，说明该流程会编排底层辅助函数或计算内核。 像 `cur_expert`、`offset_cumsum`、`tokens_per_expert`、`cumsum` 以及 `cur_expert_start` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 751-832: Internal helper `_fwd_kernel_ep_scatter_2`
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
    HIDDEN_SIZE: tl.constexpr,
    HIDDEN_SIZE_PAD: tl.constexpr,
    SCALE_HIDDEN_SIZE: tl.constexpr,
    SCALE_HIDDEN_SIZE_PAD: tl.constexpr,
    # Platform-specific semaphore for atomic_add performance tuning
    ATOMIC_ADD_SEM: tl.constexpr,
    IS_FP8: tl.constexpr,
):
    start_token_id = tl.program_id(0)
    grid_num = tl.num_programs(0)

    offset_in = tl.arange(0, HIDDEN_SIZE_PAD)
    mask = offset_in < HIDDEN_SIZE

    index_in_s = tl.arange(0, SCALE_HIDDEN_SIZE_PAD)
    mask_s = index_in_s < SCALE_HIDDEN_SIZE

    for token_id_int32 in range(start_token_id, total_token_num, grid_num):
        token_id = token_id_int32.to(tl.int64)
        to_copy = tl.load(recv_x + token_id * recv_x_stride0 + offset_in, mask=mask)
        if IS_FP8:
            to_copy_s = tl.load(
                recv_x_scale
                + token_id * recv_x_scale_stride0
                + index_in_s * recv_x_scale_stride1,
                mask=mask_s,
            )

        for topk_idx_int32 in tl.range(0, topk_num, 1, num_stages=4):
            topk_index = topk_idx_int32.to(tl.int64)
            expert_id = tl.load(recv_topk + token_id * recv_topk_stride0 + topk_index)
            if expert_id >= 0:
                dest_token_index_int32 = tl.atomic_add(
                    expert_start_loc + expert_id, 1, sem=ATOMIC_ADD_SEM
                )
                dest_token_index = dest_token_index_int32.to(tl.int64)

                tl.store(
                    output_index + token_id * output_index_stride0 + topk_index,
                    dest_token_index_int32,
                )
                output_tensor_ptr = (
                    output_tensor + dest_token_index * output_tensor_stride0
                )
                tl.store(output_tensor_ptr + offset_in, to_copy, mask=mask)
                if IS_FP8:
                    output_tensor_scale_ptr = (
                        output_tensor_scale
                        + dest_token_index * output_tensor_scale_stride0
                    )
                    tl.store(
                        output_tensor_scale_ptr
                        + index_in_s * output_tensor_scale_stride1,
                        to_copy_s,
                        mask=mask_s,
                    )


# copy from https://github.com/ModelTC/lightllm/blob/main/lightllm/common/fused_moe/deepep_scatter_gather.py
```
**EN:** This block defines `_fwd_kernel_ep_scatter_2` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.num_programs`, `tl.arange`, `range`, and `token_id_int32.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `start_token_id`, `grid_num`, `offset_in`, `mask`, and `index_in_s` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_fwd_kernel_ep_scatter_2`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.num_programs`、`tl.arange`、`range` 以及 `token_id_int32.to`，说明该流程会编排底层辅助函数或计算内核。 像 `start_token_id`、`grid_num`、`offset_in`、`mask` 以及 `index_in_s` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 833-916: Function `ep_scatter` and its core logic
```python
@torch.no_grad()
def ep_scatter(
    recv_x: torch.Tensor,
    recv_x_scale: torch.Tensor,
    recv_topk: torch.Tensor,
    num_recv_tokens_per_expert: torch.Tensor,
    expert_start_loc: torch.Tensor,
    output_tensor: torch.Tensor,
    output_tensor_scale: torch.Tensor,
    m_indices: torch.Tensor,
    output_index: torch.Tensor,
    scale_ue8m0: bool = False,
):
    BLOCK_E = 128  # token num of per expert is aligned to 128
    BLOCK_D = 128  # block size of quantization
    num_warps = 8
    num_experts = num_recv_tokens_per_expert.shape[0]
    hidden_size = recv_x.shape[1]
    # grid = (triton.cdiv(hidden_size, BLOCK_D), num_experts)
    grid = num_experts

    scale_hidden_size = hidden_size // BLOCK_D
    if scale_ue8m0:
        # ue8m0 scales are packed here (4 scales per int32),
        # hence the effective size of this dimension is divided by 4.
        scale_hidden_size = ceil_div(scale_hidden_size, 4)

    assert m_indices.shape[0] % BLOCK_E == 0

    is_fp8 = recv_x_scale is not None and recv_x.dtype != torch.bfloat16
    if is_fp8:
        assert (
            recv_x_scale.dtype == output_tensor_scale.dtype
        ), f"recv_x_scale.dtype: {recv_x_scale.dtype}, output_tensor_scale.dtype: {output_tensor_scale.dtype}"
        assert (
            recv_x_scale.shape[1] == output_tensor_scale.shape[1] == scale_hidden_size
        )

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
        recv_x_scale.stride(0) if is_fp8 else 0,
        recv_x_scale.stride(1) if is_fp8 else 0,
        recv_topk,
        recv_topk.stride(0),
        recv_topk.stride(1),
        output_tensor,
        output_tensor.stride(0),
        output_tensor.stride(1),
        output_tensor_scale,
        output_tensor_scale.stride(0) if is_fp8 else 0,
        output_tensor_scale.stride(1) if is_fp8 else 0,
        output_index,
        output_index.stride(0),
        output_index.stride(1),
        topk_num=recv_topk.shape[1],
        num_warps=num_warps,
        HIDDEN_SIZE=hidden_size,
        HIDDEN_SIZE_PAD=triton.next_power_of_2(hidden_size),
        SCALE_HIDDEN_SIZE=scale_hidden_size,
        SCALE_HIDDEN_SIZE_PAD=triton.next_power_of_2(scale_hidden_size),
        # XXX (MUSA): Atomic add with "relaxed" semaphore on musa backend for better performance
        ATOMIC_ADD_SEM=None if not _is_musa else "relaxed",
        IS_FP8=is_fp8,
    )
    return
```
**EN:** This block defines `ep_scatter` and contains the main logic for this step. Decorators like `torch.no_grad` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `torch.no_grad`, `_fwd_kernel_ep_scatter_1`, `min`, `_fwd_kernel_ep_scatter_2`, and `ceil_div`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `BLOCK_E`, `BLOCK_D`, `num_warps`, `num_experts`, and `hidden_size` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `ep_scatter`，并承载这一阶段的核心逻辑。 像 `torch.no_grad` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `torch.no_grad`、`_fwd_kernel_ep_scatter_1`、`min`、`_fwd_kernel_ep_scatter_2` 以及 `ceil_div`，说明该流程会编排底层辅助函数或计算内核。 像 `BLOCK_E`、`BLOCK_D`、`num_warps`、`num_experts` 以及 `hidden_size` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 917-982: Internal helper `_fwd_kernel_ep_gather`
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
    BLOCK_D: tl.constexpr,
):
    cur_block_int32 = tl.program_id(0)
    cur_block = cur_block_int32.to(tl.int64)

    start_cur_token_int32 = tl.program_id(1)

    grid_num = tl.num_programs(1)

    for cur_token_int32 in range(start_cur_token_int32, total_token_num, grid_num):
        cur_token = cur_token_int32.to(tl.int64)

        off_d = tl.arange(0, BLOCK_D)
        accumulator = tl.zeros([BLOCK_D], dtype=tl.float32)

        for topk_index_int32 in range(0, topk_num):
            topk_index = topk_index_int32.to(tl.int64)

            expert_id = tl.load(
                recv_topk_ids + cur_token * recv_topk_ids_stride0 + topk_index
            )
            if expert_id >= 0:
                source_token_index_int32 = tl.load(
                    input_index + cur_token * input_index_stride0 + topk_index
                )
                source_token_index = source_token_index_int32.to(tl.int64)

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
**EN:** This block defines `_fwd_kernel_ep_gather` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `cur_block_int32.to`, `tl.num_programs`, `range`, and `cur_token_int32.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `cur_block_int32`, `cur_block`, `start_cur_token_int32`, `grid_num`, and `cur_token` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_fwd_kernel_ep_gather`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`cur_block_int32.to`、`tl.num_programs`、`range` 以及 `cur_token_int32.to`，说明该流程会编排底层辅助函数或计算内核。 像 `cur_block_int32`、`cur_block`、`start_cur_token_int32`、`grid_num` 以及 `cur_token` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 983-1022: Function `ep_gather` and its core logic
```python
@torch.no_grad()
def ep_gather(
    input_tensor: torch.Tensor,
    recv_topk_ids: torch.Tensor,
    recv_topk_weight: torch.Tensor,
    input_index: torch.Tensor,
    output_tensor: torch.Tensor,
):
    num_warps = 2
    num_tokens = output_tensor.shape[0]
    hidden_size = input_tensor.shape[1]
    BLOCK_D = 128 if hidden_size % 1024 != 0 else 1024  # block size of quantization
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
        num_warps=num_warps,
        BLOCK_D=BLOCK_D,
    )
    return


# copy from
# https://github.com/deepseek-ai/DeepGEMM/blob/bd2a77552886b98c205af12f8d7d2d61247c4b27/deep_gemm/jit_kernels/utils.py#L58
```
**EN:** This block defines `ep_gather` and contains the main logic for this step. Decorators like `torch.no_grad` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `torch.no_grad`, `_fwd_kernel_ep_gather`, `triton.cdiv`, `min`, and `input_tensor.stride`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_warps`, `num_tokens`, `hidden_size`, `BLOCK_D`, and `grid` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `ep_gather`，并承载这一阶段的核心逻辑。 像 `torch.no_grad` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `torch.no_grad`、`_fwd_kernel_ep_gather`、`triton.cdiv`、`min` 以及 `input_tensor.stride`，说明该流程会编排底层辅助函数或计算内核。 像 `num_warps`、`num_tokens`、`hidden_size`、`BLOCK_D` 以及 `grid` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1023-1041: `get_tma_aligned_size` getter for tma aligned size
```python
def get_tma_aligned_size(x: int, element_size: int) -> int:
    """
    Global memory address of TMA must be 16-byte aligned.
    Since we use column-major layout for the LHS scaling tensor,
        the M-axis of the LHS scaling tensor needs to be padded to a multiple of 16 bytes.

    Arguments:
        x: original M-axis shape of the LHS scaling tensor.
        element_size: element size of the LHS scaling tensor.

    Returns:
        M-axis shape of the LHS scaling tensor after padding.
    """
    tma_alignment_bytes = 16
    assert tma_alignment_bytes % element_size == 0
    alignment = tma_alignment_bytes // element_size
    return ceil_div(x, alignment) * alignment
```
**EN:** This block defines `get_tma_aligned_size` and contains the main logic for this step. It mainly invokes `ceil_div`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `tma_alignment_bytes` and `alignment` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_tma_aligned_size`，并承载这一阶段的核心逻辑。 它主要调用 `ceil_div`，说明该流程会编排底层辅助函数或计算内核。 像 `tma_alignment_bytes` 和 `alignment` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1042-1072: Internal helper `_tma_align_input_scale_kernel`
```python
@triton.jit
def _tma_align_input_scale_kernel(
    input_scale_ptr,
    output_ptr,
    m,
    k_div_block_size,
    input_scale_stride_m,
    input_scale_stride_k,
    output_stride_m,
    output_stride_k,
    BLOCK_SIZE_K: tl.constexpr,
):
    pid_m = tl.program_id(axis=0)
    grid_m = tl.num_programs(0)
    k_offsets = tl.arange(0, BLOCK_SIZE_K)

    for m_base in range(pid_m, m, grid_m):
        input_offset = (
            input_scale_ptr
            + m_base * input_scale_stride_m
            + k_offsets * input_scale_stride_k
        )
        input_data = tl.load(input_offset, mask=k_offsets < k_div_block_size)

        output_offset = (
            output_ptr + k_offsets * output_stride_k + m_base * output_stride_m
        )
        tl.store(output_offset, input_data, mask=k_offsets < k_div_block_size)


# copy from https://github.com/ModelTC/lightllm/blob/main/lightllm/common/quantization/triton_quant/fp8/fp8act_quant_kernel.py
```
**EN:** This block defines `_tma_align_input_scale_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.num_programs`, `tl.arange`, `range`, and `tl.load`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid_m`, `grid_m`, `k_offsets`, `input_offset`, and `input_data` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_tma_align_input_scale_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.num_programs`、`tl.arange`、`range` 以及 `tl.load`，说明该流程会编排底层辅助函数或计算内核。 像 `pid_m`、`grid_m`、`k_offsets`、`input_offset` 以及 `input_data` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1073-1097: Function `tma_align_input_scale` and its core logic
```python
def tma_align_input_scale(input_scale: torch.Tensor):
    assert input_scale.dim() == 2
    m, k_div_block_size = input_scale.shape
    padd_m = get_tma_aligned_size(m, input_scale.element_size())
    output = torch.empty(
        (k_div_block_size, padd_m), dtype=input_scale.dtype, device=input_scale.device
    )

    grid_m = min(m, 8192)
    BLOCK_SIZE_K = triton.next_power_of_2(k_div_block_size)

    _tma_align_input_scale_kernel[(grid_m,)](
        input_scale_ptr=input_scale,
        output_ptr=output,
        m=m,
        k_div_block_size=k_div_block_size,
        input_scale_stride_m=input_scale.stride(0),
        input_scale_stride_k=input_scale.stride(1),
        output_stride_m=output.stride(1),  # Note: these are swapped
        output_stride_k=output.stride(0),  # for column-major
        BLOCK_SIZE_K=BLOCK_SIZE_K,
    )
    return output.t()[:m]
```
**EN:** This block defines `tma_align_input_scale` and contains the main logic for this step. It mainly invokes `get_tma_aligned_size`, `torch.empty`, `min`, `triton.next_power_of_2`, and `_tma_align_input_scale_kernel`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `m`, `k_div_block_size`, `padd_m`, `output`, and `grid_m` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `tma_align_input_scale`，并承载这一阶段的核心逻辑。 它主要调用 `get_tma_aligned_size`、`torch.empty`、`min`、`triton.next_power_of_2` 以及 `_tma_align_input_scale_kernel`，说明该流程会编排底层辅助函数或计算内核。 像 `m`、`k_div_block_size`、`padd_m`、`output` 以及 `grid_m` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1098-1105: `compute_masked_m_triton_kernel` step for masked m Triton kernel
```python
@triton.jit
def compute_masked_m_triton_kernel(seg_indptr, masked_m):
    expert_id = tl.program_id(0)
    start = tl.load(seg_indptr + expert_id)
    end = tl.load(seg_indptr + expert_id + 1)
    tl.store(masked_m + expert_id, (end - start))
```
**EN:** This block defines `compute_masked_m_triton_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.load`, and `tl.store`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `expert_id`, `start`, and `end` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `compute_masked_m_triton_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.load` 以及 `tl.store`，说明该流程会编排底层辅助函数或计算内核。 像 `expert_id`、`start` 以及 `end` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1106-1126: Function `deepgemm_compute_src2dst_triton_kernel` and its core logic
```python
@triton.jit
def deepgemm_compute_src2dst_triton_kernel(
    topk_ids,
    reorder_ids,
    seg_indptr,
    src2dst,
    m_max,
    num_toks,
    BLOCK_SIZE: tl.constexpr,
):
    pid = tl.program_id(axis=0)
    dst_id = pid * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    mask = dst_id < num_toks
    src_id = tl.load(reorder_ids + dst_id, mask=mask)
    expert_id = tl.load(topk_ids + src_id, mask=(src_id < num_toks))
    expert_dst_start = tl.load(seg_indptr + expert_id, mask=(expert_id >= 0))
    expert_dst_offset = dst_id - expert_dst_start
    dst_id = expert_id * m_max + expert_dst_offset
    tl.store(src2dst + src_id, dst_id, mask=mask)
```
**EN:** This block defines `deepgemm_compute_src2dst_triton_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.load`, `tl.store`, and `tl.arange`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `dst_id`, `mask`, `src_id`, and `expert_id` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `deepgemm_compute_src2dst_triton_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.load`、`tl.store` 以及 `tl.arange`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`dst_id`、`mask`、`src_id` 以及 `expert_id` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1127-1167: Function `fill_gateup_input_triton_kernel` and its core logic
```python
@triton.jit
def fill_gateup_input_triton_kernel(
    input_ptr,
    scale_ptr,
    gateup_input_ptr,
    gateup_input_scale_ptr,
    src2dst_ptr,
    topk_ids_ptr,
    topk,
    hidden_size,
    scale_size,
    BLOCK_SIZE: tl.constexpr,
):

    src_idx_int32 = tl.program_id(0)
    src_idx = src_idx_int32.to(tl.int64)
    src2dst_ptr = src2dst_ptr + src_idx * topk
    topk_ids_ptr = topk_ids_ptr + src_idx * topk
    src_ptr = input_ptr + src_idx * hidden_size
    scale_src_ptr = scale_ptr + src_idx * scale_size

    vec = tl.arange(0, BLOCK_SIZE)
    for idx in range(topk):
        expert_id = tl.load(topk_ids_ptr + idx)
        if expert_id >= 0:
            dst_idx_int32 = tl.load(src2dst_ptr + idx)
            dst_idx = dst_idx_int32.to(tl.int64)
            dst_ptr = gateup_input_ptr + dst_idx * hidden_size
            for start_offset in tl.range(0, hidden_size, BLOCK_SIZE):
                offset = start_offset + vec
                mask = offset < hidden_size
                in_data = tl.load(src_ptr + offset, mask=mask)
                tl.store(dst_ptr + offset, in_data, mask=mask)
            scale_dst_ptr = gateup_input_scale_ptr + dst_idx * scale_size
            for start_offset in tl.range(0, scale_size, BLOCK_SIZE):
                offset = start_offset + vec
                mask = offset < scale_size
                in_scale = tl.load(scale_src_ptr + offset, mask=mask)
                tl.store(scale_dst_ptr + offset, in_scale, mask=mask)
```
**EN:** This block defines `fill_gateup_input_triton_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `src_idx_int32.to`, `tl.arange`, `range`, and `tl.load`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `src_idx_int32`, `src_idx`, `src2dst_ptr`, `topk_ids_ptr`, and `src_ptr` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `fill_gateup_input_triton_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`src_idx_int32.to`、`tl.arange`、`range` 以及 `tl.load`，说明该流程会编排底层辅助函数或计算内核。 像 `src_idx_int32`、`src_idx`、`src2dst_ptr`、`topk_ids_ptr` 以及 `src_ptr` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1168-1244: Function `moe_ep_deepgemm_preprocess` and its core logic
```python
def moe_ep_deepgemm_preprocess(
    topk_ids: torch.Tensor,
    num_local_experts: int,
    hidden_states: torch.Tensor,
    top_k: int,
    block_shape,
    output_dtype: torch.dtype = torch.float8_e4m3fn,
):
    reorder_topk_ids, reorder_ids = torch.sort(topk_ids.view(-1), stable=True)
    seg_indptr = torch.zeros(
        num_local_experts + 1, device=topk_ids.device, dtype=torch.int64
    )
    src2dst = torch.empty(topk_ids.numel(), device=topk_ids.device, dtype=torch.int32)
    masked_m = torch.empty(num_local_experts, device=topk_ids.device, dtype=torch.int32)

    compute_seg_indptr_triton_kernel[(num_local_experts + 1,)](
        reorder_topk_ids, seg_indptr, topk_ids.numel()
    )

    grid = lambda meta: (triton.cdiv(topk_ids.numel(), meta["BLOCK_SIZE"]),)
    compute_masked_m_triton_kernel[(num_local_experts,)](seg_indptr, masked_m)

    # For masked grouped GEMM, shape M should be multiple of the block M (current block M: {block_m}) https://github.com/deepseek-ai/DeepGEMM/blob/main/deep_gemm/jit_kernels/m_grouped_gemm.py#L165
    m_max = (hidden_states.size(0) // 256 + 1) * 256
    expected_m = (topk_ids.numel() - 1) // num_local_experts + 1
    gateup_input = torch.empty(
        (num_local_experts, m_max, hidden_states.size(1)),
        device=hidden_states.device,
        dtype=output_dtype,
    )

    deepgemm_compute_src2dst_triton_kernel[grid](
        topk_ids,
        reorder_ids,
        seg_indptr,
        src2dst,
        m_max,
        topk_ids.numel(),
        BLOCK_SIZE=256,
    )

    if block_shape is None:
        block_shape = [128, 128]
    assert len(block_shape) == 2
    block_n, block_k = block_shape[0], block_shape[1]

    # TODO: fuse this with the preprocess
    hidden_states, scale = per_token_group_quant_fp8(hidden_states, block_k)

    gateup_input_scale = torch.empty(
        (gateup_input.size(0), gateup_input.size(1), scale.size(1)),
        device=hidden_states.device,
        dtype=scale.dtype,
    )

    fill_gateup_input_triton_kernel[(hidden_states.shape[0],)](
        hidden_states,
        scale,
        gateup_input,
        gateup_input_scale,
        src2dst,
        topk_ids,
        top_k,
        hidden_states.size(1),
        scale.size(1),
        BLOCK_SIZE=1024,
    )

    return (
        masked_m,
        expected_m,
        src2dst,
        gateup_input,
        gateup_input_scale,
    )
```
**EN:** This block defines `moe_ep_deepgemm_preprocess` and contains the main logic for this step. It mainly invokes `torch.sort`, `torch.zeros`, `torch.empty`, `compute_seg_indptr_triton_kernel`, and `compute_masked_m_triton_kernel`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `reorder_topk_ids`, `reorder_ids`, `seg_indptr`, `src2dst`, and `masked_m` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `moe_ep_deepgemm_preprocess`，并承载这一阶段的核心逻辑。 它主要调用 `torch.sort`、`torch.zeros`、`torch.empty`、`compute_seg_indptr_triton_kernel` 以及 `compute_masked_m_triton_kernel`，说明该流程会编排底层辅助函数或计算内核。 像 `reorder_topk_ids`、`reorder_ids`、`seg_indptr`、`src2dst` 以及 `masked_m` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1245-1283: `compute_identity_kernel` step for identity kernel
```python
@triton.jit
def compute_identity_kernel(
    top_k,
    hidden_states_ptr,
    expert_scales_ptr,
    num_tokens,
    output_ptr,
    hidden_dim,
    scales_stride,
    BLOCK_SIZE: tl.constexpr,
):
    pid = tl.program_id(0)

    batch_id = pid // (hidden_dim // BLOCK_SIZE)
    dim_offset = pid % (hidden_dim // BLOCK_SIZE) * BLOCK_SIZE

    if batch_id >= num_tokens or dim_offset >= hidden_dim:
        return

    h = tl.load(
        hidden_states_ptr
        + batch_id * hidden_dim
        + dim_offset
        + tl.arange(0, BLOCK_SIZE),
        mask=(dim_offset + tl.arange(0, BLOCK_SIZE)) < hidden_dim,
    )

    result = tl.zeros([BLOCK_SIZE], dtype=tl.float32)
    for i in range(top_k):
        scale = tl.load(expert_scales_ptr + batch_id * scales_stride + i)
        result += h * scale

    tl.store(
        output_ptr + batch_id * hidden_dim + dim_offset + tl.arange(0, BLOCK_SIZE),
        result,
        mask=(dim_offset + tl.arange(0, BLOCK_SIZE)) < hidden_dim,
    )
```
**EN:** This block defines `compute_identity_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.load`, `tl.zeros`, `range`, and `tl.store`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `batch_id`, `dim_offset`, `h`, and `result` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `compute_identity_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.load`、`tl.zeros`、`range` 以及 `tl.store`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`batch_id`、`dim_offset`、`h` 以及 `result` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1284-1318: Function `zero_experts_compute_triton` and its core logic
```python
def zero_experts_compute_triton(
    expert_indices, expert_scales, num_experts, zero_expert_type, hidden_states
):
    N = expert_indices.numel()
    top_k = expert_indices.size(-1)
    grid = lambda meta: (triton.cdiv(N, meta["BLOCK_SIZE"]),)

    if zero_expert_type == "identity":
        zero_expert_mask = expert_indices < num_experts
        zero_expert_scales = expert_scales.clone()
        zero_expert_scales[zero_expert_mask] = 0.0

    normal_expert_mask = expert_indices >= num_experts
    expert_indices[normal_expert_mask] = -1
    expert_scales[normal_expert_mask] = 0.0

    output = torch.zeros_like(hidden_states).to(hidden_states.device)
    hidden_dim = hidden_states.size(-1)
    num_tokens = hidden_states.size(0)

    grid = lambda meta: (num_tokens * (hidden_dim // meta["BLOCK_SIZE"]),)
    compute_identity_kernel[grid](
        top_k,
        hidden_states,
        zero_expert_scales,
        num_tokens,
        output,
        hidden_dim,
        zero_expert_scales.stride(0),
        BLOCK_SIZE=256,
    )

    return output
```
**EN:** This block defines `zero_experts_compute_triton` and contains the main logic for this step. It mainly invokes `expert_indices.numel`, `expert_indices.size`, `torch.zeros_like.to`, `hidden_states.size`, and `compute_identity_kernel`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `N`, `top_k`, `grid`, `normal_expert_mask`, and `output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `zero_experts_compute_triton`，并承载这一阶段的核心逻辑。 它主要调用 `expert_indices.numel`、`expert_indices.size`、`torch.zeros_like.to`、`hidden_states.size` 以及 `compute_identity_kernel`，说明该流程会编排底层辅助函数或计算内核。 像 `N`、`top_k`、`grid`、`normal_expert_mask` 以及 `output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1319-1356: `compute_problem_sizes_w4a8_kernel` step for problem sizes w 4 a 8 kernel
```python
@triton.jit
def compute_problem_sizes_w4a8_kernel(
    masked_m_ptr,
    problem_sizes1_ptr,
    problem_sizes2_ptr,
    n,
    k,
    num_experts,
    BLOCK_SIZE: tl.constexpr,
):
    pid = tl.program_id(axis=0) * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    mask = pid < num_experts
    final_occurrences = tl.load(masked_m_ptr + pid, mask=mask, other=0)

    ps1_idx_0 = pid * 3
    ps1_idx_1 = ps1_idx_0 + 1
    ps1_idx_2 = ps1_idx_0 + 2

    ps2_idx_0 = pid * 3
    ps2_idx_1 = ps2_idx_0 + 1
    ps2_idx_2 = ps2_idx_0 + 2

    ps1_mask_0 = ps1_idx_0 < num_experts * 3
    ps1_mask_1 = ps1_idx_1 < num_experts * 3
    ps1_mask_2 = ps1_idx_2 < num_experts * 3
    ps2_mask_0 = ps2_idx_0 < num_experts * 3
    ps2_mask_1 = ps2_idx_1 < num_experts * 3
    ps2_mask_2 = ps2_idx_2 < num_experts * 3

    tl.store(problem_sizes1_ptr + ps1_idx_0, 2 * n, mask=ps1_mask_0)
    tl.store(problem_sizes1_ptr + ps1_idx_1, final_occurrences, mask=ps1_mask_1)
    tl.store(problem_sizes1_ptr + ps1_idx_2, k, mask=ps1_mask_2)

    tl.store(problem_sizes2_ptr + ps2_idx_0, k, mask=ps2_mask_0)
    tl.store(problem_sizes2_ptr + ps2_idx_1, final_occurrences, mask=ps2_mask_1)
    tl.store(problem_sizes2_ptr + ps2_idx_2, n, mask=ps2_mask_2)
```
**EN:** This block defines `compute_problem_sizes_w4a8_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.load`, `tl.store`, `tl.arange`, and `tl.program_id`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `mask`, `final_occurrences`, `ps1_idx_0`, and `ps1_idx_1` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `compute_problem_sizes_w4a8_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.load`、`tl.store`、`tl.arange` 以及 `tl.program_id`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`mask`、`final_occurrences`、`ps1_idx_0` 以及 `ps1_idx_1` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1357-1373: `compute_problem_sizes_w4a8` step for problem sizes w 4 a 8
```python
def compute_problem_sizes_w4a8(
    masked_m, problem_sizes1, problem_sizes2, n, k, num_experts
):
    BLOCK_SIZE = 256
    grid = lambda meta: (triton.cdiv(num_experts, meta["BLOCK_SIZE"]),)
    compute_problem_sizes_w4a8_kernel[grid](
        masked_m,
        problem_sizes1,
        problem_sizes2,
        n,
        k,
        num_experts,
        BLOCK_SIZE=BLOCK_SIZE,
    )
    return problem_sizes1, problem_sizes2
```
**EN:** This block defines `compute_problem_sizes_w4a8` and contains the main logic for this step. It mainly invokes `compute_problem_sizes_w4a8_kernel` and `triton.cdiv`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `BLOCK_SIZE` and `grid` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `compute_problem_sizes_w4a8`，并承载这一阶段的核心逻辑。 它主要调用 `compute_problem_sizes_w4a8_kernel` 和 `triton.cdiv`，说明该流程会编排底层辅助函数或计算内核。 像 `BLOCK_SIZE` 和 `grid` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1374-1390: Function `deepep_ll_get_cutlass_w4a8_moe_mm_data` and its core logic
```python
def deepep_ll_get_cutlass_w4a8_moe_mm_data(
    masked_m,
    problem_sizes1,
    problem_sizes2,
    num_experts,
    n,
    k,
):
    problem_sizes1, problem_sizes2 = compute_problem_sizes_w4a8(
        masked_m, problem_sizes1, problem_sizes2, n, k, num_experts
    )
    return (
        problem_sizes1.to(torch.int32),
        problem_sizes2.to(torch.int32),
    )
```
**EN:** This block defines `deepep_ll_get_cutlass_w4a8_moe_mm_data` and contains the main logic for this step. It mainly invokes `compute_problem_sizes_w4a8`, `problem_sizes1.to`, and `problem_sizes2.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `problem_sizes1` and `problem_sizes2` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `deepep_ll_get_cutlass_w4a8_moe_mm_data`，并承载这一阶段的核心逻辑。 它主要调用 `compute_problem_sizes_w4a8`、`problem_sizes1.to` 以及 `problem_sizes2.to`，说明该流程会编排底层辅助函数或计算内核。 像 `problem_sizes1` 和 `problem_sizes2` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1391-1456: Internal helper `_silu_and_mul_post_per_tensor_quant_kernel`
```python
@triton.jit
def _silu_and_mul_post_per_tensor_quant_kernel(
    input_ptr,
    stride_input_expert,
    stride_input_token,
    stride_input_dim,
    output_ptr,
    stride_output_expert,
    stride_output_token,
    stride_output_dim,
    scale_ptr,
    masked_m_ptr,
    inner_dim,
    fp8_max,
    fp8_min,
    BLOCK_N: tl.constexpr,
    NUM_STAGE: tl.constexpr,
):
    """
    Triton kernel: fused SiLU(gate) * up + per-tensor FP8 quantization.

    Shape:
        input:  [E, T_padded, 2*D]  -> gate: [:,:,D], up: [:,:,D]
        output: [E, T_padded, D], dtype=float8_e4m3fn
    """
    expert_id = tl.program_id(2)
    block_id_token = tl.program_id(1)
    block_id_dim = tl.program_id(0)

    num_token_blocks = tl.num_programs(1)

    token_num_cur_expert = tl.load(masked_m_ptr + expert_id)

    scale = 1.0 / tl.load(scale_ptr).to(tl.float32)

    stride_input_expert = tl.cast(stride_input_expert, tl.int32)
    stride_output_expert = tl.cast(stride_output_expert, tl.int32)
    stride_input_token = tl.cast(stride_input_token, tl.int32)
    stride_output_token = tl.cast(stride_output_token, tl.int32)

    offset_d = block_id_dim * BLOCK_N + tl.arange(0, BLOCK_N)
    mask_d = offset_d < inner_dim

    # base pointers for current expert and dim block
    input_base_offs = input_ptr + expert_id * stride_input_expert + offset_d
    output_base_offs = output_ptr + expert_id * stride_output_expert + offset_d

    for token_idx in tl.range(
        block_id_token, token_num_cur_expert, num_token_blocks, num_stages=NUM_STAGE
    ):
        gate_ptr = input_base_offs + token_idx * stride_input_token
        up_ptr = gate_ptr + inner_dim
        gate = tl.load(gate_ptr, mask=mask_d, other=0.0).to(tl.float32)
        up = tl.load(up_ptr, mask=mask_d, other=0.0).to(tl.float32)

        # SiLU: x * sigmoid(x)
        gate = gate / (1 + tl.exp(-gate))
        gate = gate.to(input_ptr.dtype.element_ty)
        gate_up = up * gate

        scaled = gate_up * scale
        output_q = tl.clamp(scaled, fp8_min, fp8_max).to(output_ptr.dtype.element_ty)
        out_ptr = output_base_offs + token_idx * stride_output_token
        tl.store(out_ptr, output_q, mask=mask_d)
```
**EN:** This block defines `_silu_and_mul_post_per_tensor_quant_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.num_programs`, `tl.load`, `tl.cast`, and `tl.range`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `expert_id`, `block_id_token`, `block_id_dim`, `num_token_blocks`, and `token_num_cur_expert` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_silu_and_mul_post_per_tensor_quant_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.num_programs`、`tl.load`、`tl.cast` 以及 `tl.range`，说明该流程会编排底层辅助函数或计算内核。 像 `expert_id`、`block_id_token`、`block_id_dim`、`num_token_blocks` 以及 `token_num_cur_expert` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1457-1512: Function `silu_and_mul_masked_post_per_tensor_quant_fwd` and its core logic
```python
def silu_and_mul_masked_post_per_tensor_quant_fwd(
    input: torch.Tensor,
    output: torch.Tensor,
    masked_m: torch.Tensor,
    scale: torch.Tensor,
) -> torch.Tensor:
    """
    Fused SiLU + Mul + Per-Tensor Quantization to FP8.

    Args:
        input: [expert_num, token_num_padded, 2 * inner_dim]
        output: [expert_num, token_num_padded, inner_dim], dtype=torch.float8_e4m3fn
        masked_m: [expert_num], actual token count for each expert
        scale: [1] or [expert_num], quantization scale (per-tensor or per-expert)

    Returns:
        output tensor
    """
    assert input.is_contiguous()
    assert output.is_contiguous()
    assert output.dtype == torch.float8_e4m3fn
    assert input.ndim == 3
    assert input.shape[0] == masked_m.shape[0]
    assert input.shape[-1] % 2 == 0
    assert scale.numel() == 1 or scale.shape[0] == input.shape[0]

    expert_num = input.shape[0]
    #  3584
    inner_dim = input.shape[-1] // 2

    BLOCK_N = 256
    BLOCK_M = 64 if expert_num < 4 else 32
    NUM_STAGES = 3
    hidden_dim_split_block_num = triton.cdiv(inner_dim, BLOCK_N)

    grid = (hidden_dim_split_block_num, BLOCK_M, expert_num)
    finfo = torch.finfo(torch.float8_e4m3fn)
    fp8_max = finfo.max
    fp8_min = -fp8_max

    _silu_and_mul_post_per_tensor_quant_kernel[grid](
        input,
        *input.stride(),
        output,
        *output.stride(),
        scale,
        masked_m,
        inner_dim,
        fp8_max,
        fp8_min,
        BLOCK_N=BLOCK_N,
        NUM_STAGE=NUM_STAGES,
    )
    return output
```
**EN:** This block defines `silu_and_mul_masked_post_per_tensor_quant_fwd` and contains the main logic for this step. It mainly invokes `input.is_contiguous`, `output.is_contiguous`, `triton.cdiv`, `torch.finfo`, and `_silu_and_mul_post_per_tensor_quant_kernel`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `expert_num`, `inner_dim`, `BLOCK_N`, `BLOCK_M`, and `NUM_STAGES` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `silu_and_mul_masked_post_per_tensor_quant_fwd`，并承载这一阶段的核心逻辑。 它主要调用 `input.is_contiguous`、`output.is_contiguous`、`triton.cdiv`、`torch.finfo` 以及 `_silu_and_mul_post_per_tensor_quant_kernel`，说明该流程会编排底层辅助函数或计算内核。 像 `expert_num`、`inner_dim`、`BLOCK_N`、`BLOCK_M` 以及 `NUM_STAGES` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1513-1554: Internal helper `_fp8_per_token_quant_to_per_tensor_quant_kernel`
```python
@triton.jit
def _fp8_per_token_quant_to_per_tensor_quant_kernel(
    x_ptr,
    x_scale_ptr,
    x_scale_stride0,
    x_scale_stride1,
    x_scale_stride2,
    masked_m_ptr,
    output_scale_ptr,
    output_ptr,
    m,
    k,
    K_SCALE_BLOCK_SIZE: tl.constexpr,
    K_BLOCK_SIZE: tl.constexpr,
):
    pid_k, pid_m, pid_e = (
        tl.program_id(axis=0),
        tl.program_id(axis=1),
        tl.program_id(axis=2),
    )
    pid_m_dim = tl.num_programs(1)

    token_id = pid_m
    last_effective_id = tl.load(masked_m_ptr + pid_e)

    if token_id >= last_effective_id:
        return
    output_scale_val_inv = 1.0 / tl.load(output_scale_ptr).to(tl.float32)
    k_offsets = pid_k * K_BLOCK_SIZE + tl.arange(0, K_BLOCK_SIZE)
    scale_offsets = (k_offsets // K_SCALE_BLOCK_SIZE) * x_scale_stride2

    x_ptrs = x_ptr + pid_e * m * k + k_offsets
    output_ptrs = output_ptr + pid_e * m * k + k_offsets
    x_scale_ptrs = x_scale_ptr + pid_e * x_scale_stride0 + scale_offsets

    for tok_idx in tl.range(token_id, last_effective_id, pid_m_dim):
        hidden = tl.load(x_ptrs + tok_idx * k).to(tl.float32)
        scale_fp32 = tl.load(x_scale_ptrs + tok_idx * x_scale_stride1).to(tl.float32)
        hidden = hidden * scale_fp32 * output_scale_val_inv
        tl.store(output_ptrs + tok_idx * k, hidden.to(output_ptr.dtype.element_ty))
```
**EN:** This block defines `_fp8_per_token_quant_to_per_tensor_quant_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.num_programs`, `tl.load`, `tl.range`, `tl.program_id`, and `tl.load.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid_k`, `pid_m`, `pid_e`, `pid_m_dim`, and `token_id` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_fp8_per_token_quant_to_per_tensor_quant_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.num_programs`、`tl.load`、`tl.range`、`tl.program_id` 以及 `tl.load.to`，说明该流程会编排底层辅助函数或计算内核。 像 `pid_k`、`pid_m`、`pid_e`、`pid_m_dim` 以及 `token_id` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1555-1583: Function `fp8_per_token_to_per_tensor_quant_triton` and its core logic
```python
def fp8_per_token_to_per_tensor_quant_triton(
    x: torch.Tensor,
    x_scale: torch.Tensor,
    masked_m: torch.Tensor,
    output_scale: torch.Tensor,
    output: torch.Tensor,
):
    K_SCALE_BLOCK_SIZE = 128
    assert len(x.shape) == 3 and x.size(2) % K_SCALE_BLOCK_SIZE == 0
    assert x.is_contiguous()
    assert x_scale.size(2) == x.size(2) // K_SCALE_BLOCK_SIZE
    assert output_scale.numel() == 1

    K_BLOCK_SIZE = 1024
    assert x.size(2) % K_BLOCK_SIZE == 0
    grid = (x.size(2) // K_BLOCK_SIZE, 32, x.size(0))
    _fp8_per_token_quant_to_per_tensor_quant_kernel[grid](
        x,
        x_scale,
        *x_scale.stride(),
        masked_m,
        output_scale,
        output,
        x.size(1),
        x.size(2),
        K_SCALE_BLOCK_SIZE=K_SCALE_BLOCK_SIZE,
        K_BLOCK_SIZE=K_BLOCK_SIZE,
        num_warps=8,
    )
```
**EN:** This block defines `fp8_per_token_to_per_tensor_quant_triton` and contains the main logic for this step. It mainly invokes `x.is_contiguous`, `_fp8_per_token_quant_to_per_tensor_quant_kernel`, `x_scale.size`, `output_scale.numel`, and `x.size`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `K_SCALE_BLOCK_SIZE`, `K_BLOCK_SIZE`, and `grid` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `fp8_per_token_to_per_tensor_quant_triton`，并承载这一阶段的核心逻辑。 它主要调用 `x.is_contiguous`、`_fp8_per_token_quant_to_per_tensor_quant_kernel`、`x_scale.size`、`output_scale.numel` 以及 `x.size`，说明该流程会编排底层辅助函数或计算内核。 像 `K_SCALE_BLOCK_SIZE`、`K_BLOCK_SIZE` 以及 `grid` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `_get_launch_config_1d`, `_get_launch_config_2d`, `deepep_permute_triton_kernel`, `deepep_post_reorder_triton_kernel`, and `compute_src2dst_triton_kernel`. / **主要符号**：核心入口包括 `_get_launch_config_1d`、`_get_launch_config_2d`、`deepep_permute_triton_kernel`、`deepep_post_reorder_triton_kernel` 以及 `compute_src2dst_triton_kernel`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `logging` / **标准库**：`logging`
- **Third-party**: `torch`, `triton`, and `triton.language` / **第三方依赖**：`torch`、`triton` 以及 `triton.language`
- **Internal SGLang modules**: `sglang.srt.utils.ceil_div`, `sglang.srt.utils.is_cuda`, `sglang.srt.utils.is_musa`, and `sglang.srt.layers.quantization.fp8_kernel.sglang_per_token_group_quant_fp8` / **SGLang 内部模块**：`sglang.srt.utils.ceil_div`、`sglang.srt.utils.is_cuda`、`sglang.srt.utils.is_musa` 以及 `sglang.srt.layers.quantization.fp8_kernel.sglang_per_token_group_quant_fp8`
