# fused_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/fused_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Fused MoE Triton kernels. / 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 5-37 — imports and setup
```python
import functools
import json
import os
from collections.abc import Callable
from typing import Any

import torch

import vllm.envs as envs
import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm import _custom_ops as ops
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.activation import (
    MoEActivation,
    apply_moe_activation,
)
from vllm.model_executor.layers.fused_moe.config import (
    FUSED_MOE_UNQUANTIZED_CONFIG,
    FusedMoEQuantConfig,
    _get_config_dtype_str,
)
from vllm.model_executor.layers.fused_moe.moe_align_block_size import (
    moe_align_block_size,
)
from vllm.model_executor.layers.fused_moe.utils import (
    disable_inplace,
    moe_kernel_quantize_input,
)
from vllm.platforms import current_platform
from vllm.triton_utils import tl, triton
from vllm.utils.torch_utils import direct_register_custom_op

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 40-57 — function `write_zeros_to_output`
```python
@triton.jit
def write_zeros_to_output(
    c_ptr,
    stride_cm,
    stride_cn,
    pid_n,
    N,
    offs_token,
    token_mask,
    BLOCK_SIZE_M,
    BLOCK_SIZE_N,
    compute_type,
):
    accumulator = tl.zeros((BLOCK_SIZE_M, BLOCK_SIZE_N), dtype=compute_type)
    offs_cn = pid_n * BLOCK_SIZE_N + tl.arange(0, BLOCK_SIZE_N)
    c_ptrs = c_ptr + stride_cm * offs_token[:, None] + stride_cn * offs_cn[None, :]
    c_mask = token_mask[:, None] & (offs_cn[None, :] < N)
    tl.store(c_ptrs, accumulator, mask=c_mask)
```
**EN:** This function defines `write_zeros_to_output`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `c_ptr`, `stride_cm`, `stride_cn`, `pid_n`, `N`, `offs_token`. Key calls include `tl.zeros`, `tl.store`, `tl.arange`. It writes or updates `accumulator`, `offs_cn`, `c_ptrs`, `c_mask`.
**CN:** 该函数定义 `write_zeros_to_output`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `c_ptr`, `stride_cm`, `stride_cn`, `pid_n`, `N`, `offs_token`。 关键调用包括 `tl.zeros`, `tl.store`, `tl.arange`。 它会写入或更新 `accumulator`, `offs_cn`, `c_ptrs`, `c_mask`。

### Lines 60-291 — function `fused_moe_kernel_gptq_awq`
```python
@triton.jit
def fused_moe_kernel_gptq_awq(
    # Pointers to matrices
    a_ptr,
    b_ptr,
    c_ptr,
    b_scale_ptr,
    b_zp_ptr,
    topk_weights_ptr,
    sorted_token_ids_ptr,
    expert_ids_ptr,
    num_tokens_post_padded_ptr,
    # Matrix dimensions
    N: tl.constexpr,
    K: tl.constexpr,
    EM,
    num_valid_tokens,
    # The stride variables represent how much to increase the ptr by when
    # moving by 1 element in a particular dimension. E.g. `stride_am` is
    # how much to increase `a_ptr` by to get the element one row down
    # (A has M rows).
    stride_am,
    stride_ak,
    stride_be,
    stride_bk,
    stride_bn,
    stride_cm,
    stride_cn,
    stride_bse,
    stride_bsk,
    stride_bsn,
    stride_bze,
    stride_bzk,
    stride_bzn,
    block_k_diviable: tl.constexpr,
    group_size: tl.constexpr,
    # Meta-parameters
    BLOCK_SIZE_M: tl.constexpr,
    BLOCK_SIZE_N: tl.constexpr,
    BLOCK_SIZE_K: tl.constexpr,
    GROUP_SIZE_M: tl.constexpr,
    SPLIT_K: tl.constexpr,
    MUL_ROUTED_WEIGHT: tl.constexpr,
    top_k: tl.constexpr,
    compute_type: tl.constexpr,
    has_zp: tl.constexpr,
    use_int4_w4a16: tl.constexpr,
    use_int8_w8a16: tl.constexpr,
):
    """
    Implements the fused computation for a Mixture of Experts (MOE) using
    token and expert matrices.

    Key Parameters:
    - A: The input tensor representing tokens with shape (*, K), where '*' can
        be any shape representing batches and K is the feature dimension of
# ... omitted for brevity ...
    c_mask = token_mask[:, None] & (offs_cn[None, :] < N)
    tl.store(c_ptrs, accumulator, mask=c_mask)
```
**EN:** This function defines `fused_moe_kernel_gptq_awq`. Implements the fused computation for a Mixture of Experts (MOE) using token and expert matrices. The main inputs are `a_ptr`, `b_ptr`, `c_ptr`, `b_scale_ptr`, `b_zp_ptr`, `topk_weights_ptr`. Key calls include `tl.program_id`, `tl.cdiv`, `min`, `tl.load`, `tl.load.to`, `tl.arange`. It writes or updates `pid`, `num_pid_m`, `num_pid_n`, `num_pid_in_group`, `group_id`, `first_pid_m`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `fused_moe_kernel_gptq_awq`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `a_ptr`, `b_ptr`, `c_ptr`, `b_scale_ptr`, `b_zp_ptr`, `topk_weights_ptr`。 关键调用包括 `tl.program_id`, `tl.cdiv`, `min`, `tl.load`, `tl.load.to`, `tl.arange`。 它会写入或更新 `pid`, `num_pid_m`, `num_pid_n`, `num_pid_in_group`, `group_id`, `first_pid_m`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 294-555 — function `fused_moe_kernel`
```python
@triton.jit
def fused_moe_kernel(
    # Pointers to matrices
    a_ptr,
    b_ptr,
    c_ptr,
    b_bias_ptr,
    a_scale_ptr,
    b_scale_ptr,
    topk_weights_ptr,
    sorted_token_ids_ptr,
    expert_ids_ptr,
    num_tokens_post_padded_ptr,
    # Matrix dimensions
    N,
    K,
    EM,
    num_valid_tokens,
    # The stride variables represent how much to increase the ptr by when
    # moving by 1 element in a particular dimension. E.g. `stride_am` is
    # how much to increase `a_ptr` by to get the element one row down
    # (A has M rows).
    stride_am,
    stride_ak,
    stride_be,
    stride_bk,
    stride_bn,
    stride_cm,
    stride_cn,
    stride_asm,
    stride_ask,
    stride_bse,
    stride_bsk,
    stride_bsn,
    stride_bbe,  # bias expert stride
    stride_bbn,  # bias N stride
    # Block size for block-wise quantization
    group_n: tl.constexpr,
    group_k: tl.constexpr,
    naive_block_assignment: tl.constexpr,
    # Meta-parameters
    BLOCK_SIZE_M: tl.constexpr,
    BLOCK_SIZE_N: tl.constexpr,
    BLOCK_SIZE_K: tl.constexpr,
    GROUP_SIZE_M: tl.constexpr,
    SPLIT_K: tl.constexpr,
    MUL_ROUTED_WEIGHT: tl.constexpr,
    top_k: tl.constexpr,
    compute_type: tl.constexpr,
    use_fp8_w8a8: tl.constexpr,
    use_int8_w8a8: tl.constexpr,
    use_int8_w8a16: tl.constexpr,
    per_channel_quant: tl.constexpr,
    HAS_BIAS: tl.constexpr,
):
    """
# ... omitted for brevity ...
    c_mask = token_mask[:, None] & (offs_cn[None, :] < N)
    tl.store(c_ptrs, accumulator, mask=c_mask)
```
**EN:** This function defines `fused_moe_kernel`. Implements the fused computation for a Mixture of Experts (MOE) using token and expert matrices. The main inputs are `a_ptr`, `b_ptr`, `c_ptr`, `b_bias_ptr`, `a_scale_ptr`, `b_scale_ptr`. Key calls include `tl.program_id`, `tl.cdiv`, `min`, `tl.arange.to`, `tl.load`, `offs_token.to`. It writes or updates `pid`, `num_pid_m`, `num_pid_n`, `num_pid_in_group`, `group_id`, `first_pid_m`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `fused_moe_kernel`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `a_ptr`, `b_ptr`, `c_ptr`, `b_bias_ptr`, `a_scale_ptr`, `b_scale_ptr`。 关键调用包括 `tl.program_id`, `tl.cdiv`, `min`, `tl.arange.to`, `tl.load`, `offs_token.to`。 它会写入或更新 `pid`, `num_pid_m`, `num_pid_n`, `num_pid_in_group`, `group_id`, `first_pid_m`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 560-613 — function `invoke_fused_moe_wna16_cuda_kernel`
```python
def invoke_fused_moe_wna16_cuda_kernel(
    A: torch.Tensor,
    B: torch.Tensor,
    C: torch.Tensor,
    B_scale: torch.Tensor | None,
    B_zp: torch.Tensor | None,
    topk_weights: torch.Tensor | None,
    sorted_token_ids: torch.Tensor | None,
    expert_ids: torch.Tensor,
    num_tokens_post_padded: torch.Tensor,
    mul_routed_weight: bool,
    top_k: int,
    config: dict[str, Any],
    block_shape: list[int],
):
    assert B_scale is not None and B_scale.ndim == 3
    assert B_zp is None or B_zp.ndim == 3
    assert block_shape is None or block_shape[0] == 0

    M = A.size(0)
    num_tokens = M * top_k
    bit = 4

    config = config.copy()
    config.update(
        get_moe_wna16_block_config(
            config=config,
            use_moe_wna16_cuda=True,
            num_valid_tokens=num_tokens,
            size_k=A.size(1),
            size_n=B.size(1),
            num_experts=B.size(1),
            group_size=block_shape[1],
            real_top_k=top_k,
            block_size_m=config["BLOCK_SIZE_M"],
        )
    )

    ops.moe_wna16_gemm(
        A,
        C,
        B,
        B_scale,
        B_zp,
# ... omitted for brevity ...
        bit,
    )
```
**EN:** This function defines `invoke_fused_moe_wna16_cuda_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `A`, `B`, `C`, `B_scale`, `B_zp`, `topk_weights`. Key calls include `A.size`, `config.copy`, `config.update`, `ops.moe_wna16_gemm`, `get_moe_wna16_block_config`, `B.size`. It writes or updates `M`, `num_tokens`, `bit`, `config`.
**CN:** 该函数定义 `invoke_fused_moe_wna16_cuda_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `A`, `B`, `C`, `B_scale`, `B_zp`, `topk_weights`。 关键调用包括 `A.size`, `config.copy`, `config.update`, `ops.moe_wna16_gemm`, `get_moe_wna16_block_config`, `B.size`。 它会写入或更新 `M`, `num_tokens`, `bit`, `config`。

### Lines 618-705 — function `invoke_fused_moe_wna16_triton_kernel`
```python
def invoke_fused_moe_wna16_triton_kernel(
    A: torch.Tensor,
    B: torch.Tensor,
    C: torch.Tensor,
    B_scale: torch.Tensor | None,
    B_zp: torch.Tensor | None,
    topk_weights: torch.Tensor | None,
    sorted_token_ids: torch.Tensor,
    expert_ids: torch.Tensor,
    num_tokens_post_padded: torch.Tensor,
    mul_routed_weight: bool,
    top_k: int,
    config: dict[str, Any],
    compute_type: tl.dtype,
    use_int8_w8a16: bool,
    use_int4_w4a16: bool,
    block_shape: list[int] | None,
):
    assert B_scale is not None and B_scale.ndim == 3
    assert B_zp is None or B_zp.ndim == 3
    assert block_shape is not None and block_shape[0] == 0

    M = A.size(0)
    num_tokens = M * top_k

    EM = sorted_token_ids.size(0)
    if A.size(0) < config["BLOCK_SIZE_M"]:
        # optimize for small batch_size.
        # We assume that top_ids of each token is unique,
        # so num_valid_experts <= batch_size <= BLOCK_SIZE_M,
        # and we can skip some invalid blocks.
        EM = min(sorted_token_ids.size(0), A.size(0) * top_k * config["BLOCK_SIZE_M"])
    grid = lambda META: (
        triton.cdiv(EM, META["BLOCK_SIZE_M"])
        * triton.cdiv(B.size(1), META["BLOCK_SIZE_N"]),
    )
    config = config.copy()
    config.update(
        get_moe_wna16_block_config(
            config=config,
            use_moe_wna16_cuda=False,
            num_valid_tokens=num_tokens,
            size_k=A.size(1),
            size_n=B.size(1),
# ... omitted for brevity ...
        **config,
    )
```
**EN:** This function defines `invoke_fused_moe_wna16_triton_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `A`, `B`, `C`, `B_scale`, `B_zp`, `topk_weights`. Key calls include `A.size`, `sorted_token_ids.size`, `config.copy`, `config.update`, `fused_moe_kernel_gptq_awq`, `min`. It writes or updates `M`, `num_tokens`, `EM`, `grid`, `config`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `invoke_fused_moe_wna16_triton_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `A`, `B`, `C`, `B_scale`, `B_zp`, `topk_weights`。 关键调用包括 `A.size`, `sorted_token_ids.size`, `config.copy`, `config.update`, `fused_moe_kernel_gptq_awq`, `min`。 它会写入或更新 `M`, `num_tokens`, `EM`, `grid`, `config`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 708-816 — function `invoke_fused_moe_triton_kernel`
```python
def invoke_fused_moe_triton_kernel(
    A: torch.Tensor,
    B: torch.Tensor,
    C: torch.Tensor,
    A_scale: torch.Tensor | None,
    B_scale: torch.Tensor | None,
    topk_weights: torch.Tensor | None,
    sorted_token_ids: torch.Tensor | None,
    expert_ids: torch.Tensor,
    num_tokens_post_padded: torch.Tensor,
    mul_routed_weight: bool,
    top_k: int,
    config: dict[str, Any],
    compute_type: tl.dtype,
    use_fp8_w8a8: bool,
    use_int8_w8a8: bool,
    use_int8_w8a16: bool,
    use_int4_w4a16: bool,
    per_channel_quant: bool,
    block_shape: list[int] | None = None,
    B_bias: torch.Tensor | None = None,
):
    assert topk_weights is not None or not mul_routed_weight
    assert topk_weights is None or topk_weights.stride(1) == 1
    assert sorted_token_ids is None or sorted_token_ids.stride(0) == 1

    if use_fp8_w8a8 or use_int8_w8a8:
        assert B_scale is not None
        assert block_shape is None or triton.cdiv(
            B.size(-2), block_shape[0]
        ) == B_scale.size(-2)
        assert block_shape is None or triton.cdiv(
            B.size(-1), block_shape[1]
        ) == B_scale.size(-1)
    elif use_int8_w8a16 or use_int4_w4a16:
        assert B_scale is not None
        assert block_shape is None or block_shape[0] == 0
    else:
        assert A_scale is None
        assert B_scale is None

    M = A.size(0)
    num_tokens = M * top_k
    if sorted_token_ids is not None:
# ... omitted for brevity ...
        **config,
    )
```
**EN:** This function defines `invoke_fused_moe_triton_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `A`, `B`, `C`, `A_scale`, `B_scale`, `topk_weights`. Key calls include `A.size`, `config.copy`, `config.pop`, `fused_moe_kernel`, `sorted_token_ids.size`, `min`. It writes or updates `M`, `num_tokens`, `grid`, `HAS_BIAS`, `config`, `BLOCK_SIZE_K`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `invoke_fused_moe_triton_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `A`, `B`, `C`, `A_scale`, `B_scale`, `topk_weights`。 关键调用包括 `A.size`, `config.copy`, `config.pop`, `fused_moe_kernel`, `sorted_token_ids.size`, `min`。 它会写入或更新 `M`, `num_tokens`, `grid`, `HAS_BIAS`, `config`, `BLOCK_SIZE_K`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 819-919 — function `dispatch_fused_moe_kernel`
```python
def dispatch_fused_moe_kernel(
    A: torch.Tensor,
    B: torch.Tensor,
    C: torch.Tensor,
    A_scale: torch.Tensor | None,
    B_scale: torch.Tensor | None,
    B_zp: torch.Tensor | None,
    topk_weights: torch.Tensor | None,
    sorted_token_ids: torch.Tensor | None,
    expert_ids: torch.Tensor,
    num_tokens_post_padded: torch.Tensor,
    mul_routed_weight: bool,
    top_k: int,
    config: dict[str, Any],
    compute_type: tl.dtype,
    use_fp8_w8a8: bool,
    use_int8_w8a8: bool,
    use_int8_w8a16: bool,
    use_int4_w4a16: bool,
    per_channel_quant: bool,
    block_shape: list[int] | None = None,
    B_bias: torch.Tensor | None = None,
) -> None:
    assert topk_weights is not None or not mul_routed_weight
    assert topk_weights is None or topk_weights.stride(1) == 1
    assert sorted_token_ids is None or sorted_token_ids.stride(0) == 1

    M = A.size(0)
    num_tokens = M * top_k

    if (use_int8_w8a16 or use_int4_w4a16) and (
        block_shape is not None and block_shape[1] > 0
    ):
        assert B_bias is None

        use_moe_wna16_cuda = should_moe_wna16_use_cuda(
            num_valid_tokens=num_tokens,
            group_size=block_shape[1],
            num_experts=B.size(0),
            bit=4 if use_int4_w4a16 else 8,
        )

        if use_moe_wna16_cuda:
            invoke_fused_moe_wna16_cuda_kernel(
# ... omitted for brevity ...
            B_bias,
        )
```
**EN:** This function defines `dispatch_fused_moe_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `A`, `B`, `C`, `A_scale`, `B_scale`, `B_zp`. Key calls include `A.size`, `should_moe_wna16_use_cuda`, `invoke_fused_moe_wna16_triton_kernel`, `invoke_fused_moe_triton_kernel`, `topk_weights.stride`, `sorted_token_ids.stride`. It writes or updates `M`, `num_tokens`, `use_moe_wna16_cuda`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `dispatch_fused_moe_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `A`, `B`, `C`, `A_scale`, `B_scale`, `B_zp`。 关键调用包括 `A.size`, `should_moe_wna16_use_cuda`, `invoke_fused_moe_wna16_triton_kernel`, `invoke_fused_moe_triton_kernel`, `topk_weights.stride`, `sorted_token_ids.stride`。 它会写入或更新 `M`, `num_tokens`, `use_moe_wna16_cuda`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 922-958 — function `compute_identity_kernel`
```python
@triton.jit
def compute_identity_kernel(
    top_k: int,
    hidden_states_ptr: tl.tensor,
    expert_scales_ptr: tl.tensor,
    num_tokens: int,
    output_ptr: tl.tensor,
    hidden_dim: int,
    scales_stride: int,
    BLOCK_SIZE: tl.constexpr,
) -> None:
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
**EN:** This function defines `compute_identity_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `top_k`, `hidden_states_ptr`, `expert_scales_ptr`, `num_tokens`, `output_ptr`, `hidden_dim`. Key calls include `tl.program_id`, `tl.load`, `tl.zeros`, `range`, `tl.store`, `tl.arange`. It writes or updates `pid`, `batch_id`, `dim_offset`, `h`, `result`, `scale`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `compute_identity_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `top_k`, `hidden_states_ptr`, `expert_scales_ptr`, `num_tokens`, `output_ptr`, `hidden_dim`。 关键调用包括 `tl.program_id`, `tl.load`, `tl.zeros`, `range`, `tl.store`, `tl.arange`。 它会写入或更新 `pid`, `batch_id`, `dim_offset`, `h`, `result`, `scale`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 961-997 — function `zero_experts_compute_triton`
```python
def zero_experts_compute_triton(
    expert_indices: torch.Tensor,
    expert_scales: torch.Tensor,
    num_experts: int,
    zero_expert_type: str,
    hidden_states: torch.Tensor,
) -> torch.Tensor:
    N = expert_indices.numel()
    top_k = expert_indices.size(-1)
    grid = lambda meta: (triton.cdiv(N, meta["BLOCK_SIZE"]),)

    if zero_expert_type == "identity":
        zero_expert_mask = expert_indices < num_experts
        zero_expert_scales = expert_scales.clone()
        zero_expert_scales[zero_expert_mask] = 0.0

    normal_expert_mask = expert_indices >= num_experts
    expert_indices[normal_expert_mask] = 0
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
**EN:** This function defines `zero_experts_compute_triton`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `expert_indices`, `expert_scales`, `num_experts`, `zero_expert_type`, `hidden_states`. Key calls include `expert_indices.numel`, `expert_indices.size`, `torch.zeros_like.to`, `hidden_states.size`, `compute_identity_kernel`, `expert_scales.clone`. It writes or updates `N`, `top_k`, `grid`, `normal_expert_mask`, `output`, `hidden_dim`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `zero_experts_compute_triton`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `expert_indices`, `expert_scales`, `num_experts`, `zero_expert_type`, `hidden_states`。 关键调用包括 `expert_indices.numel`, `expert_indices.size`, `torch.zeros_like.to`, `hidden_states.size`, `compute_identity_kernel`, `expert_scales.clone`。 它会写入或更新 `N`, `top_k`, `grid`, `normal_expert_mask`, `output`, `hidden_dim`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 1001-1012 — function `get_config_file_name`
```python
def get_config_file_name(
    E: int, N: int, dtype: str | None, block_shape: list[int] | None = None
) -> str:
    device_name = current_platform.get_device_name().replace(" ", "_")
    # Set device_name to H200 if a device from the H200 family is detected
    if "H200" in device_name.split("_"):
        device_name = "NVIDIA_H200"
    dtype_selector = "" if not dtype else f",dtype={dtype}"
    block_shape_selector = (
        "" if not block_shape or not all(block_shape) else f",block_shape={block_shape}"
    ).replace(" ", "")
    return f"E={E},N={N},device_name={device_name}{dtype_selector}{block_shape_selector}.json"  # noqa: E501
```
**EN:** This function defines `get_config_file_name`. It derives and returns computed metadata or outputs needed by later stages. The main inputs are `E`, `N`, `dtype`, `block_shape`. Key calls include `current_platform.get_device_name.replace`, `replace`, `device_name.split`, `current_platform.get_device_name`, `all`. It writes or updates `device_name`, `dtype_selector`, `block_shape_selector`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `get_config_file_name`。 它推导并返回后续阶段所需的元数据或输出。 其主要输入参数包括 `E`, `N`, `dtype`, `block_shape`。 关键调用包括 `current_platform.get_device_name.replace`, `replace`, `device_name.split`, `current_platform.get_device_name`, `all`。 它会写入或更新 `device_name`, `dtype_selector`, `block_shape_selector`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 1016-1078 — function `get_moe_configs`
```python
@functools.lru_cache
def get_moe_configs(
    E: int,
    N: int,
    dtype: str | None,
    block_n: int | None = None,
    block_k: int | None = None,
) -> dict[int, Any] | None:
    """
    Return optimized configurations for the fused MoE kernel.

    The return value will be a dictionary that maps an irregular grid of
    batch sizes to configurations of the fused_moe kernel. To evaluate the
    kernel on a given batch size bs, the closest batch size in the grid should
    be picked and the associated configuration chosen to invoke the kernel.
    """

    # Avoid optimizing for the batch invariant case. Use default config
    if envs.VLLM_BATCH_INVARIANT:
        return None

    # First look up if an optimized configuration is available in the configs
    # directory
    block_shape = [block_n, block_k] if block_n and block_k else None
    json_file_name = get_config_file_name(E, N, dtype, block_shape)

    config_file_paths = []

    # note that we prioritize user defined config
    user_defined_config_folder = envs.VLLM_TUNED_CONFIG_FOLDER
    if user_defined_config_folder is not None:
        user_defined_config_file_path = os.path.join(
            user_defined_config_folder, json_file_name
        )
        config_file_paths.append(user_defined_config_file_path)

    default_config_file_path = os.path.join(
        os.path.dirname(os.path.realpath(__file__)), "configs", json_file_name
    )
    config_file_paths.append(default_config_file_path)

    for config_file_path in config_file_paths:
        if os.path.exists(config_file_path):
            with open(config_file_path) as f:
# ... omitted for brevity ...
    )
    return None
```
**EN:** This function defines `get_moe_configs`. Return optimized configurations for the fused MoE kernel. The main inputs are `E`, `N`, `dtype`, `block_n`, `block_k`. Key calls include `get_config_file_name`, `os.path.join`, `config_file_paths.append`, `logger.warning_once`, `os.path.dirname`, `os.path.exists`. It writes or updates `block_shape`, `json_file_name`, `config_file_paths`, `user_defined_config_folder`, `default_config_file_path`, `user_defined_config_file_path`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `get_moe_configs`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `E`, `N`, `dtype`, `block_n`, `block_k`。 关键调用包括 `get_config_file_name`, `os.path.join`, `config_file_paths.append`, `logger.warning_once`, `os.path.dirname`, `os.path.exists`。 它会写入或更新 `block_shape`, `json_file_name`, `config_file_paths`, `user_defined_config_folder`, `default_config_file_path`, `user_defined_config_file_path`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 1081-1120 — function `_ensure_block_size_k_divisible`
```python
def _ensure_block_size_k_divisible(
    size_k: int, block_size_k: int, group_size: int
) -> int:
    """Ensure block_size_k is a divisor of size_k and divisible by group_size.

    This ensures BLOCK_SIZE_K compatibility with MoeWNA16 CUDA kernel which
    requires size_k % BLOCK_SIZE_K == 0 and BLOCK_SIZE_K % group_size == 0.

    Args:
        size_k: The size_k dimension that must be divisible by result.
        block_size_k: Preferred block size (will be adjusted if needed).
        group_size: The result must be divisible by this.

    Returns:
        A valid BLOCK_SIZE_K that divides size_k and is divisible by group_size.
    """
    # Fast path: already valid
    if size_k % block_size_k == 0 and block_size_k % group_size == 0:
        return block_size_k

    # Find the largest value that:
    # 1. Divides size_k (size_k % candidate == 0)
    # 2. Is divisible by group_size (candidate % group_size == 0)
    # 3. Is <= block_size_k (prefer smaller values close to block_size_k)
    #
    # Strategy: Search from min(block_size_k, size_k) down to group_size,
    # stepping by group_size to ensure divisibility by group_size
    max_search = min(block_size_k, size_k)
    start = (max_search // group_size) * group_size
    for candidate in range(start, group_size - 1, -group_size):
        if size_k % candidate == 0:
            return candidate

    # Fallback: if group_size divides size_k, use it
    # This should always be true with correct group_size configuration
    if size_k % group_size == 0:
        return group_size

    # This should not happen with correct group_size, but ensure divisibility
    return size_k
```
**EN:** This function defines `_ensure_block_size_k_divisible`. Ensure block_size_k is a divisor of size_k and divisible by group_size. The main inputs are `size_k`, `block_size_k`, `group_size`. Key calls include `min`, `range`. It writes or updates `max_search`, `start`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `_ensure_block_size_k_divisible`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `size_k`, `block_size_k`, `group_size`。 关键调用包括 `min`, `range`。 它会写入或更新 `max_search`, `start`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 1123-1191 — function `get_moe_wna16_block_config`
```python
def get_moe_wna16_block_config(
    config: dict[str, int],
    use_moe_wna16_cuda: bool,
    num_valid_tokens: int,
    size_k: int,
    size_n: int,
    num_experts: int,
    group_size: int,
    real_top_k: int,
    block_size_m: int,
):
    if "BLOCK_SIZE_N" in config and "BLOCK_SIZE_K" in config:
        # optimal block config is set
        return {}
    if not use_moe_wna16_cuda:
        # triton moe wna16 kernel
        if num_valid_tokens // real_top_k == 1:
            # if bs=1, use a smaller BLOCK_SIZE_N
            return {"BLOCK_SIZE_N": 32, "BLOCK_SIZE_K": 64}
        else:
            return {"BLOCK_SIZE_N": 64, "BLOCK_SIZE_K": 32}
    else:
        # cuda moe wna16 kernel
        # set default block_size 128, and increase them when num_blocks
        # is too large.
        block_size_n = 128
        block_size_k = 128
        if block_size_k <= group_size:
            block_size_k = group_size

        num_n_blocks = size_k // block_size_k
        num_k_blocks = size_n // block_size_k
        num_m_blocks = (
            num_valid_tokens + block_size_m - 1
        ) / block_size_m + num_experts
        if num_valid_tokens // real_top_k <= block_size_m:
            num_m_blocks = min(num_m_blocks, num_valid_tokens)
        num_blocks = num_m_blocks * num_n_blocks * num_k_blocks

        if size_k % 256 == 0 and num_blocks >= 256 and block_size_k < 256:
            block_size_k = 256
            num_blocks = num_blocks // (256 // block_size_k)

        if (
# ... omitted for brevity ...

        return {"BLOCK_SIZE_N": block_size_n, "BLOCK_SIZE_K": block_size_k}
```
**EN:** This function defines `get_moe_wna16_block_config`. It derives and returns computed metadata or outputs needed by later stages. The main inputs are `config`, `use_moe_wna16_cuda`, `num_valid_tokens`, `size_k`, `size_n`, `num_experts`. Key calls include `_ensure_block_size_k_divisible`, `min`. It writes or updates `block_size_n`, `block_size_k`, `num_n_blocks`, `num_k_blocks`, `num_m_blocks`, `num_blocks`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `get_moe_wna16_block_config`。 它推导并返回后续阶段所需的元数据或输出。 其主要输入参数包括 `config`, `use_moe_wna16_cuda`, `num_valid_tokens`, `size_k`, `size_n`, `num_experts`。 关键调用包括 `_ensure_block_size_k_divisible`, `min`。 它会写入或更新 `block_size_n`, `block_size_k`, `num_n_blocks`, `num_k_blocks`, `num_m_blocks`, `num_blocks`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `write_zeros_to_output`, `fused_moe_kernel_gptq_awq`, `fused_moe_kernel`, `invoke_fused_moe_wna16_cuda_kernel` / [CN] 核心符号：`write_zeros_to_output`, `fused_moe_kernel_gptq_awq`, `fused_moe_kernel`, `invoke_fused_moe_wna16_cuda_kernel`

## Dependencies / 依赖关系
- **External**: `functools`, `json`, `os`, `collections.abc`, `typing`, `torch` / **外部依赖**: `functools`, `json`, `os`, `collections.abc`, `typing`, `torch`
- **Internal**: `vllm.envs`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.moe_align_block_size`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.platforms`, `vllm.triton_utils` / **内部依赖**: `vllm.envs`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.moe_align_block_size`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.platforms`, `vllm.triton_utils`
- **Runtime traits**: platform-aware dispatch, Triton kernels, distributed collectives / **运行时特征**: platform-aware dispatch, Triton kernels, distributed collectives
