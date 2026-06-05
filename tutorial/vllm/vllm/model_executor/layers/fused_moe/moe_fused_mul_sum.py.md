# moe_fused_mul_sum.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/moe_fused_mul_sum.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fused Mixture-of-Experts routing, kernels, and runtime helpers / 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-7 — imports and setup
```python
import torch
from torch._subclasses.fake_tensor import FakeTensor

from vllm.platforms import current_platform
from vllm.triton_utils import tl, triton
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 10-63 — function `moe_fused_mul_sum_kernel`
```python
@triton.jit
def moe_fused_mul_sum_kernel(
    inputs_ptr,
    topk_weights_ptr,
    outputs_ptr,
    top_ids_ptr,
    expert_map_ptr,
    num_tokens,
    stride_m,
    has_expert_map: tl.constexpr,
    top_k: tl.constexpr,
    size: tl.constexpr,
    BLOCK_M: tl.constexpr,
    BLOCK_K: tl.constexpr,
):
    pid_k = tl.program_id(0)
    pid_m = tl.program_id(1)

    offs_m = pid_m * BLOCK_M + tl.arange(0, BLOCK_M)
    offs_k = pid_k * BLOCK_K + tl.arange(0, BLOCK_K)

    m_mask = offs_m < num_tokens
    k_mask = offs_k < size
    mask = m_mask[:, None] & k_mask[None, :]

    a_base = inputs_ptr + (offs_m * stride_m)[:, None] + offs_k[None, :]
    b_base = topk_weights_ptr + offs_m * top_k

    acc = tl.zeros((BLOCK_M, BLOCK_K), dtype=tl.float32)

    for n in tl.static_range(top_k):
        b_val = tl.load(b_base + n, mask=m_mask, other=0.0).to(tl.float32)
        if has_expert_map:
            id_val = tl.load(top_ids_ptr + offs_m * top_k + n, mask=m_mask, other=0)
            expert_mask = tl.load(expert_map_ptr + id_val) >= 0
            a_vec = tl.load(
                a_base + n * size,
                mask=mask & expert_mask[:, None],
                other=0.0,
            ).to(tl.float32)
        else:
            a_vec = tl.load(
                a_base + n * size,
                mask=mask,
                other=0.0,
            ).to(tl.float32)
        acc += a_vec * b_val[:, None]

    out_ptrs = outputs_ptr + (offs_m * size)[:, None] + offs_k[None, :]
    tl.store(
        out_ptrs,
        acc.to(outputs_ptr.dtype.element_ty),
        mask=mask,
    )
```
**EN:** This function defines `moe_fused_mul_sum_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `inputs_ptr`, `topk_weights_ptr`, `outputs_ptr`, `top_ids_ptr`, `expert_map_ptr`, `num_tokens`. Key calls include `tl.program_id`, `tl.zeros`, `tl.static_range`, `tl.store`, `tl.arange`, `tl.load.to`. It writes or updates `pid_k`, `pid_m`, `offs_m`, `offs_k`, `m_mask`, `k_mask`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `moe_fused_mul_sum_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `inputs_ptr`, `topk_weights_ptr`, `outputs_ptr`, `top_ids_ptr`, `expert_map_ptr`, `num_tokens`。 关键调用包括 `tl.program_id`, `tl.zeros`, `tl.static_range`, `tl.store`, `tl.arange`, `tl.load.to`。 它会写入或更新 `pid_k`, `pid_m`, `offs_m`, `offs_k`, `m_mask`, `k_mask`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 66-132 — function `_heuristic_config`
```python
def _heuristic_config(
    num_tokens: int,
    top_k: int,
    size: int,
    element_size: int,
):
    is_fp32 = element_size > 2
    is_sm90_plus = current_platform.has_device_capability(90)
    is_sm80_before = not current_platform.has_device_capability(80)

    if current_platform.has_device_capability(90):
        # SM90/SM100+: prefer small tiles + many CTAs.
        if is_fp32:
            BLOCK_M = 1 if num_tokens <= 4 else 2
        else:
            if num_tokens <= 4:
                BLOCK_M = 1
            elif num_tokens <= 128:
                BLOCK_M = 2
            else:
                BLOCK_M = 4
    elif is_fp32:
        if num_tokens <= 4:
            BLOCK_M = 1
        elif num_tokens <= 32:
            BLOCK_M = 2
        elif num_tokens <= 128:
            BLOCK_M = 4
        else:
            BLOCK_M = 4
    else:
        if num_tokens <= 4:
            BLOCK_M = 1
        elif num_tokens <= 32:
            BLOCK_M = 2
        elif num_tokens <= 128:
            BLOCK_M = 4
        elif num_tokens <= 1024:
            BLOCK_M = 16
        else:
            BLOCK_M = 8

    if is_fp32:
        max_block_k = 256
# ... omitted for brevity ...

    return BLOCK_M, BLOCK_K, num_warps, num_stages
```
**EN:** This function defines `_heuristic_config`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `num_tokens`, `top_k`, `size`, `element_size`. Key calls include `current_platform.has_device_capability`, `min`, `max`, `triton.next_power_of_2`. It writes or updates `is_fp32`, `is_sm90_plus`, `is_sm80_before`, `BLOCK_K`, `total`, `max_block_k`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_heuristic_config`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `num_tokens`, `top_k`, `size`, `element_size`。 关键调用包括 `current_platform.has_device_capability`, `min`, `max`, `triton.next_power_of_2`。 它会写入或更新 `is_fp32`, `is_sm90_plus`, `is_sm80_before`, `BLOCK_K`, `total`, `max_block_k`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 135-202 — function `moe_fused_mul_sum`
```python
def moe_fused_mul_sum(
    inputs: torch.Tensor,
    topk_weights: torch.Tensor,
    outputs: torch.Tensor | None = None,
    topk_ids: torch.Tensor | None = None,
    expert_map: torch.Tensor | None = None,
) -> torch.Tensor:
    """
    Fused kernel for MoE (Mixture of Experts) to perform weighted summation
    of expert outputs.

    Args:
        inputs: The output from experts.
            Shape: (num_tokens, top_k, hidden_size).
        topk_weights: The weights assigned to each expert for each token.
            Shape: (num_tokens, top_k).
        outputs: Optional pre-allocated output tensor.
            Shape: (num_tokens, hidden_size).
        topk_ids: Optional indices of the top-k experts. Used when
            `expert_map` is provided. Shape: (num_tokens, top_k).
        expert_map: Optional mapping for Expert Parallelism. A value < 0
            indicates an invalid token/expert pair that will be skipped.

    Returns:
        The fused weighted sum of expert outputs.
        Shape: (num_tokens, hidden_size).
    """
    assert inputs.ndim == 3
    assert topk_weights.ndim == 2
    assert inputs.is_contiguous()
    assert topk_weights.is_contiguous()
    assert inputs.dtype in (torch.float32, torch.float16, torch.bfloat16)
    assert topk_weights.dtype in (torch.float32, torch.float16, torch.bfloat16)

    num_tokens, top_k, size = inputs.shape
    output_shape = (num_tokens, size)
    if outputs is None:
        outputs = torch.empty(output_shape, dtype=inputs.dtype, device=inputs.device)

    assert outputs.shape == output_shape
    assert topk_weights.shape == (num_tokens, top_k)

    if not isinstance(inputs, FakeTensor):
        BLOCK_M, BLOCK_K, num_warps, num_stages = _heuristic_config(
# ... omitted for brevity ...

    return outputs
```
**EN:** This function defines `moe_fused_mul_sum`. Fused kernel for MoE (Mixture of Experts) to perform weighted summation of expert outputs. The main inputs are `inputs`, `topk_weights`, `outputs`, `topk_ids`, `expert_map`. Key calls include `inputs.is_contiguous`, `topk_weights.is_contiguous`, `torch.empty`, `isinstance`, `_heuristic_config`, `moe_fused_mul_sum_kernel`. It writes or updates `num_tokens`, `top_k`, `size`, `output_shape`, `outputs`, `BLOCK_M`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `moe_fused_mul_sum`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `inputs`, `topk_weights`, `outputs`, `topk_ids`, `expert_map`。 关键调用包括 `inputs.is_contiguous`, `topk_weights.is_contiguous`, `torch.empty`, `isinstance`, `_heuristic_config`, `moe_fused_mul_sum_kernel`。 它会写入或更新 `num_tokens`, `top_k`, `size`, `output_shape`, `outputs`, `BLOCK_M`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `moe_fused_mul_sum_kernel`, `_heuristic_config`, `moe_fused_mul_sum` / [CN] 核心符号：`moe_fused_mul_sum_kernel`, `_heuristic_config`, `moe_fused_mul_sum`

## Dependencies / 依赖关系
- **External**: `torch`, `torch._subclasses.fake_tensor` / **外部依赖**: `torch`, `torch._subclasses.fake_tensor`
- **Internal**: `vllm.platforms`, `vllm.triton_utils` / **内部依赖**: `vllm.platforms`, `vllm.triton_utils`
- **Runtime traits**: platform-aware dispatch, Triton kernels / **运行时特征**: platform-aware dispatch, Triton kernels
