# topk.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/musa/kernels/topk.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements MUSA backend support for kernel-level operators inside the SGLang runtime. / 为 SGLang 运行时提供面向 MUSA 后端的内核级算子支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Module setup and shared state / 模块设置与共享状态
```python
from typing import (
    Optional,
)

import torch
import triton
import triton.language as tl
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `typing`, `torch`, `triton`, `triton.language`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `typing`, `torch`, `triton`, `triton.language`。

### Lines 10-13: Function `tanh` / 函数 `tanh`
```python
@triton.jit
def tanh(x):
    # Tanh is just a scaled sigmoid
    return 2 * tl.sigmoid(2 * x) - 1
```
**EN:** This function implements `tanh`. It primarily calls `tl.sigmoid` to complete its work.
**CN:** 该函数实现了 `tanh`。 它主要通过调用 `tl.sigmoid` 来完成任务。

### Lines 16-113: Function `topk_softmax_triton_kernel` / 函数 `topk_softmax_triton_kernel`
```python
@triton.autotune(
    configs=[
        triton.Config({}, num_warps=1, num_stages=1),
        triton.Config({}, num_warps=1, num_stages=2),
        triton.Config({}, num_warps=2, num_stages=1),
        triton.Config({}, num_warps=2, num_stages=2),
        triton.Config({}, num_warps=4, num_stages=1),
        triton.Config({}, num_warps=4, num_stages=2),
        triton.Config({}, num_warps=4, num_stages=3),
        triton.Config({}, num_warps=8, num_stages=1),
        triton.Config({}, num_warps=8, num_stages=2),
        triton.Config({}, num_warps=8, num_stages=3),
        triton.Config({}, num_warps=16, num_stages=1),
        triton.Config({}, num_warps=16, num_stages=2),
        triton.Config({}, num_warps=16, num_stages=3),
        triton.Config({}, num_warps=32, num_stages=1),
        triton.Config({}, num_warps=32, num_stages=2),
    ],
    key=["num_tokens", "num_experts", "has_correction_bias"],
)
@triton.jit
def topk_softmax_triton_kernel(
    gating_output_ptr,
    selected_expert_ptr,
    moe_weights_ptr,
    renormalize_flag,
    num_experts,
    num_tokens,  # for autotune key
# ... omitted for brevity ...
    )
    tl.store(
        selected_expert_ptr + curr_row_idx * K + k_offset,
        selected_local_final,
        mask=mask_topk,
    )
```
**EN:** This function implements `topk_softmax_triton_kernel`. It primarily calls `triton.autotune`, `tl.program_id`, `tl.zeros`, `tl.arange`, `tl.load`, `tl.cast` to complete its work. State updates are written into `curr_row_idx`, `FLOAT_MINIMUM`, `LOG2E`, `weights_local_final`, `selected_local_final`, `offset`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `topk_softmax_triton_kernel`。 它主要通过调用 `triton.autotune`, `tl.program_id`, `tl.zeros`, `tl.arange`, `tl.load`, `tl.cast` 来完成任务。 状态更新主要写入 `curr_row_idx`, `FLOAT_MINIMUM`, `LOG2E`, `weights_local_final`, `selected_local_final`, `offset`。 实现中使用了条件分支、迭代逻辑。

### Lines 116-156: Function `topk_softmax` / 函数 `topk_softmax`
```python
def topk_softmax(
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    gating_output: torch.Tensor,
    renormalize: bool = False,
    moe_softcapping: float = 0.0,
    correction_bias: Optional[torch.Tensor] = None,
) -> None:
    """
    Compute top-k softmax for MoE routing.

    Args:
        topk_weights: Output tensor for top-k weights [num_tokens, topk]
        topk_ids: Output tensor for top-k expert indices [num_tokens, topk]
        gating_output: Gating logits [num_tokens, num_experts]
        renormalize: Whether to renormalize the top-k weights
        moe_softcapping: Tanh softcapping value (0.0 to disable)
        correction_bias: Per-expert bias correction [num_experts], must be float32 if provided
    """

    num_tokens, num_experts = gating_output.shape
    topk = topk_weights.shape[-1]
    has_correction_bias = correction_bias is not None

    block_width_up = triton.next_power_of_2(num_experts)
    grid = (num_tokens,)

    topk_softmax_triton_kernel[grid](
# ... omitted for brevity ...
        correction_bias,
        has_correction_bias,
        K=topk,
        BLOCK_K=triton.next_power_of_2(topk),
        BLOCK_WIDTH_SIZE_UP=block_width_up,
    )
```
**EN:** This function implements `topk_softmax`. It primarily calls `triton.next_power_of_2`, `topk_softmax_triton_kernel` to complete its work. State updates are written into `topk`, `has_correction_bias`, `block_width_up`, `grid`.
**CN:** 该函数实现了 `topk_softmax`。 它主要通过调用 `triton.next_power_of_2`, `topk_softmax_triton_kernel` 来完成任务。 状态更新主要写入 `topk`, `has_correction_bias`, `block_width_up`, `grid`。

### Lines 159-261: Function `topk_sigmoid_triton_kernel` / 函数 `topk_sigmoid_triton_kernel`
```python
@triton.autotune(
    configs=[
        triton.Config({}, num_warps=1, num_stages=1),
        triton.Config({}, num_warps=1, num_stages=2),
        triton.Config({}, num_warps=2, num_stages=1),
        triton.Config({}, num_warps=2, num_stages=2),
        triton.Config({}, num_warps=4, num_stages=1),
        triton.Config({}, num_warps=4, num_stages=2),
        triton.Config({}, num_warps=4, num_stages=3),
        triton.Config({}, num_warps=8, num_stages=1),
        triton.Config({}, num_warps=8, num_stages=2),
        triton.Config({}, num_warps=8, num_stages=3),
        triton.Config({}, num_warps=16, num_stages=1),
        triton.Config({}, num_warps=16, num_stages=2),
        triton.Config({}, num_warps=16, num_stages=3),
        triton.Config({}, num_warps=32, num_stages=1),
        triton.Config({}, num_warps=32, num_stages=2),
    ],
    key=["num_tokens", "num_experts"],
)
@triton.jit
def topk_sigmoid_triton_kernel(
    gating_output_ptr,
    selected_expert_ptr,
    moe_weights_ptr,
    renormalize_flag,
    correction_bias_ptr,
    has_correction_bias: tl.constexpr,
# ... omitted for brevity ...
    )
    tl.store(
        selected_expert_ptr + curr_row_idx * K + k_offset,
        selected_local_final,
        mask=mask_topk,
    )
```
**EN:** This function implements `topk_sigmoid_triton_kernel`. It primarily calls `triton.autotune`, `tl.program_id`, `tl.zeros`, `tl.arange`, `tl.load`, `tl.cast` to complete its work. State updates are written into `curr_row_idx`, `FLOAT_MINIMUM`, `LOG2E`, `weights_local_final`, `selected_local_final`, `offset`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `topk_sigmoid_triton_kernel`。 它主要通过调用 `triton.autotune`, `tl.program_id`, `tl.zeros`, `tl.arange`, `tl.load`, `tl.cast` 来完成任务。 状态更新主要写入 `curr_row_idx`, `FLOAT_MINIMUM`, `LOG2E`, `weights_local_final`, `selected_local_final`, `offset`。 实现中使用了条件分支、迭代逻辑。

### Lines 264-300: Function `topk_sigmoid` / 函数 `topk_sigmoid`
```python
def topk_sigmoid(
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    gating_output: torch.Tensor,
    renormalize: bool = False,
    correction_bias: Optional[torch.Tensor] = None,
) -> None:
    """
    Compute top-k sigmoid for MoE routing.

    Args:
        topk_weights: Output tensor for top-k weights [num_tokens, topk]
        topk_ids: Output tensor for top-k expert indices [num_tokens, topk]
        gating_output: Gating logits [num_tokens, num_experts]
        renormalize: Whether to renormalize the top-k weights
        correction_bias: Per-expert bias correction [num_experts], must be float32 if provided
    """
    num_tokens, num_experts = gating_output.shape
    topk = topk_weights.shape[-1]
    has_correction_bias = correction_bias is not None

    block_width_up = triton.next_power_of_2(num_experts)
    grid = (num_tokens,)

    topk_sigmoid_triton_kernel[grid](
        gating_output,
        topk_ids,
        topk_weights,
# ... omitted for brevity ...
        num_experts,
        num_tokens,
        K=topk,
        BLOCK_K=triton.next_power_of_2(topk),
        BLOCK_WIDTH_SIZE_UP=block_width_up,
    )
```
**EN:** This function implements `topk_sigmoid`. It primarily calls `triton.next_power_of_2`, `topk_sigmoid_triton_kernel` to complete its work. State updates are written into `topk`, `has_correction_bias`, `block_width_up`, `grid`.
**CN:** 该函数实现了 `topk_sigmoid`。 它主要通过调用 `triton.next_power_of_2`, `topk_sigmoid_triton_kernel` 来完成任务。 状态更新主要写入 `topk`, `has_correction_bias`, `block_width_up`, `grid`。

## Key Concepts / 关键概念
- **Functions / 函数**: `tanh`, `topk_softmax_triton_kernel`, `topk_softmax`, `topk_sigmoid_triton_kernel`, `topk_sigmoid`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: None / 无
- **External / 外部依赖**: `torch`, `triton`, `triton.language`
- **Standard library / 标准库**: `typing`
