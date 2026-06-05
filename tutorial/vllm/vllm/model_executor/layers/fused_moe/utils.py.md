# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fused Mixture-of-Experts routing, kernels, and runtime helpers / 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-35 — imports and setup
```python
import functools
from math import prod

import torch
import torch.nn.functional as F

from vllm import _custom_ops as ops
from vllm.model_executor.layers.quantization.utils.fp8_utils import (
    per_token_group_quant_fp8,
)
from vllm.model_executor.layers.quantization.utils.int8_utils import (
    per_token_group_quant_int8,
    per_token_quant_int8,
)
from vllm.model_executor.layers.quantization.utils.mxfp4_utils import (
    quant_dequant_mxfp4,
)
from vllm.model_executor.layers.quantization.utils.mxfp6_utils import (
    quant_dequant_mxfp6,
)
from vllm.model_executor.layers.quantization.utils.mxfp8_utils import (
    mxfp8_e4m3_quantize,
)
from vllm.model_executor.layers.quantization.utils.nvfp4_emulation_utils import (
    ref_nvfp4_quant_dequant,
)
from vllm.model_executor.layers.quantization.utils.w8a8_utils import (
    per_tensor_dequantize,
)
from vllm.platforms import current_platform
from vllm.triton_utils import tl, triton
from vllm.utils.math_utils import cdiv
from vllm.utils.torch_utils import is_torch_equal_or_newer
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 38-67 — function `_count_expert_num_tokens`
```python
@triton.jit
def _count_expert_num_tokens(
    topk_ids_ptr,
    expert_num_tokens_ptr,
    num_experts,
    topk_numel,
    expert_map,
    HAS_EXPERT_MAP: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
):
    curr_expert = tl.program_id(0)

    offsets = tl.arange(0, BLOCK_SIZE)
    topk_ids_ptrs = topk_ids_ptr + offsets

    acc = tl.zeros((BLOCK_SIZE,), dtype=tl.int32)
    for x in range(tl.cdiv(topk_numel, BLOCK_SIZE)):
        mask = offsets < (topk_numel - x * BLOCK_SIZE)
        expert_ids = tl.load(topk_ids_ptrs, mask=mask, other=-1)
        if HAS_EXPERT_MAP:
            expert_map_ptrs = expert_map + expert_ids
            expert_map_mask = expert_ids >= 0
            expert_ids = tl.load(expert_map_ptrs, mask=expert_map_mask, other=-1)

        has_curr_expert = tl.where(expert_ids == curr_expert, 1, 0)
        acc = acc + has_curr_expert
        topk_ids_ptrs += BLOCK_SIZE

    if curr_expert < num_experts:
        tl.store(expert_num_tokens_ptr + curr_expert, tl.sum(acc))
```
**EN:** This function defines `_count_expert_num_tokens`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `topk_ids_ptr`, `expert_num_tokens_ptr`, `num_experts`, `topk_numel`, `expert_map`, `HAS_EXPERT_MAP`. Key calls include `tl.program_id`, `tl.arange`, `tl.zeros`, `range`, `tl.cdiv`, `tl.load`. It writes or updates `curr_expert`, `offsets`, `topk_ids_ptrs`, `acc`, `mask`, `expert_ids`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `_count_expert_num_tokens`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `topk_ids_ptr`, `expert_num_tokens_ptr`, `num_experts`, `topk_numel`, `expert_map`, `HAS_EXPERT_MAP`。 关键调用包括 `tl.program_id`, `tl.arange`, `tl.zeros`, `range`, `tl.cdiv`, `tl.load`。 它会写入或更新 `curr_expert`, `offsets`, `topk_ids_ptrs`, `acc`, `mask`, `expert_ids`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 70-107 — function `count_expert_num_tokens`
```python
def count_expert_num_tokens(
    topk_ids: torch.Tensor, num_local_experts: int, expert_map: torch.Tensor | None
) -> torch.Tensor:
    """
    Count the number to tokens assigned to each expert.

    Parameters:
    - topk_ids (torch.Tensor): Tensor mapping each token to its
    list of experts.
    - num_local_experts (int): Number of experts in this rank.
    - expert_map (Optional[torch.Tensor]):  A tensor mapping expert indices
    from the global expert space to the local expert space of the expert
    parallel shard.

    Returns:
    A tensor of size num_local_experts, where tensor[i] holds the number
    of tokens assigned to the ith expert.
    """
    assert topk_ids.dtype.is_signed, "The kernel uses -1 to represent invalid topk_ids"
    expert_num_tokens = torch.empty(
        (num_local_experts), device=topk_ids.device, dtype=torch.int32
    )

    grid = num_local_experts
    BLOCK_SIZE = min(topk_ids.numel(), 1024)
    BLOCK_SIZE = triton.next_power_of_2(BLOCK_SIZE)

    _count_expert_num_tokens[(grid,)](
        topk_ids,
        expert_num_tokens,
        num_local_experts,
        topk_ids.numel(),
        expert_map,
        HAS_EXPERT_MAP=expert_map is not None,
        BLOCK_SIZE=BLOCK_SIZE,
    )

    return expert_num_tokens
```
**EN:** This function defines `count_expert_num_tokens`. Count the number to tokens assigned to each expert. The main inputs are `topk_ids`, `num_local_experts`, `expert_map`. Key calls include `torch.empty`, `min`, `triton.next_power_of_2`, `_count_expert_num_tokens`, `topk_ids.numel`. It writes or updates `expert_num_tokens`, `grid`, `BLOCK_SIZE`.
**CN:** 该函数定义 `count_expert_num_tokens`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `topk_ids`, `num_local_experts`, `expert_map`。 关键调用包括 `torch.empty`, `min`, `triton.next_power_of_2`, `_count_expert_num_tokens`, `topk_ids.numel`。 它会写入或更新 `expert_num_tokens`, `grid`, `BLOCK_SIZE`。

### Lines 110-118 — function `_resize_cache`
```python
def _resize_cache(x: torch.Tensor, v: tuple[int, ...]) -> torch.Tensor:
    """
    Shrink the given tensor and apply the given view to it.  This is
    used to resize the intermediate fused_moe caches.
    """
    assert prod(v) <= x.numel(), (
        f"{v} ({prod(v)}) <= {x.shape} ({x.numel()})"
    )  # CUDAGRAPH unfriendly?
    return x.flatten()[: prod(v)].view(*v)
```
**EN:** This function defines `_resize_cache`. Shrink the given tensor and apply the given view to it. The main inputs are `x`, `v`. Key calls include `x.flatten.view`, `prod`, `x.numel`, `x.flatten`.
**CN:** 该函数定义 `_resize_cache`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `x`, `v`。 关键调用包括 `x.flatten.view`, `prod`, `x.numel`, `x.flatten`。

### Lines 121-126 — function `_nvfp4_quantize`
```python
def _nvfp4_quantize(
    A: torch.Tensor,
    A_scale: torch.Tensor | None,
    is_sf_swizzled_layout: bool,
) -> tuple[torch.Tensor, torch.Tensor]:
    return ops.scaled_fp4_quant(A, A_scale, is_sf_swizzled_layout=is_sf_swizzled_layout)
```
**EN:** This function defines `_nvfp4_quantize`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `A`, `A_scale`, `is_sf_swizzled_layout`. Key calls include `ops.scaled_fp4_quant`.
**CN:** 该函数定义 `_nvfp4_quantize`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `A`, `A_scale`, `is_sf_swizzled_layout`。 关键调用包括 `ops.scaled_fp4_quant`。

### Lines 129-152 — function `_fp8_quantize`
```python
def _fp8_quantize(
    A: torch.Tensor,
    A_scale: torch.Tensor | None,
    per_act_token: bool,
    block_shape: list[int] | None = None,
) -> tuple[torch.Tensor, torch.Tensor]:
    """
    Perform fp8 quantization on the inputs.  If a block_shape
    is provided, the output will be blocked.
    """
    if block_shape is None:
        # TODO(luka): use QuantFP8 custom op
        #  https://github.com/vllm-project/vllm/issues/20711
        A, A_scale = ops.scaled_fp8_quant(
            A, A_scale, use_per_token_if_dynamic=per_act_token
        )
    else:
        assert not per_act_token
        assert len(block_shape) == 2
        _, block_k = block_shape[0], block_shape[1]
        A, A_scale = per_token_group_quant_fp8(A, block_k)
        assert cdiv(A.size(-1), block_k) == A_scale.size(-1)

    return A, A_scale
```
**EN:** This function defines `_fp8_quantize`. Perform fp8 quantization on the inputs. The main inputs are `A`, `A_scale`, `per_act_token`, `block_shape`. Key calls include `ops.scaled_fp8_quant`, `per_token_group_quant_fp8`, `len`, `cdiv`, `A_scale.size`, `A.size`. It writes or updates `A`, `A_scale`, `_`, `block_k`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_fp8_quantize`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `A`, `A_scale`, `per_act_token`, `block_shape`。 关键调用包括 `ops.scaled_fp8_quant`, `per_token_group_quant_fp8`, `len`, `cdiv`, `A_scale.size`, `A.size`。 它会写入或更新 `A`, `A_scale`, `_`, `block_k`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 155-186 — function `_int8_quantize`
```python
def _int8_quantize(
    A: torch.Tensor,
    A_scale: torch.Tensor | None,
    per_act_token: bool,
    block_shape: list[int] | None = None,
) -> tuple[torch.Tensor, torch.Tensor]:
    """
    Perform int8 quantization on the inputs.  If a block_shape
    is provided, the output will be blocked.
    """

    # If weights are per-channel (per_channel_quant=True), then
    # activations apply per-token quantization. Otherwise, assume
    # activation tensor-wise fp8/int8 quantization, dynamic or static
    if block_shape is None:
        if per_act_token:
            A, A_scale = per_token_quant_int8(A)
        elif A_scale is not None:
            # Static per-tensor: use the optimized CUDA kernel
            A, A_scale, _ = ops.scaled_int8_quant(A, scale=A_scale)
        elif A_scale is None:
            # Dynamic per-tensor: compute scale then quantize via kernel
            A_scale = torch.clamp(A.abs().max() / 127.0, min=1e-10)
            A, A_scale, _ = ops.scaled_int8_quant(A, scale=A_scale)
    else:
        assert not per_act_token
        assert len(block_shape) == 2
        _, block_k = block_shape[0], block_shape[1]
        A, A_scale = per_token_group_quant_int8(A, block_k)
        assert cdiv(A.size(-1), block_k) == A_scale.size(-1)

    return A, A_scale
```
**EN:** This function defines `_int8_quantize`. Perform int8 quantization on the inputs. The main inputs are `A`, `A_scale`, `per_act_token`, `block_shape`. Key calls include `per_token_group_quant_int8`, `per_token_quant_int8`, `len`, `cdiv`, `A_scale.size`, `ops.scaled_int8_quant`. It writes or updates `_`, `block_k`, `A`, `A_scale`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_int8_quantize`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `A`, `A_scale`, `per_act_token`, `block_shape`。 关键调用包括 `per_token_group_quant_int8`, `per_token_quant_int8`, `len`, `cdiv`, `A_scale.size`, `ops.scaled_int8_quant`。 它会写入或更新 `_`, `block_k`, `A`, `A_scale`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 189-202 — function `_mxfp4_quantize`
```python
def _mxfp4_quantize(
    A: torch.Tensor,
    A_scale: torch.Tensor | None,
    per_act_token_quant: bool,
    block_shape: list[int] | None = None,
) -> tuple[torch.Tensor, None]:
    assert block_shape is None
    # TODO: native mxfp4 is currently not integrated in vllm,
    # so simulating even on devices supporting this data type natively.
    # Once integrated, `current_platform.supports_mx()` should be used to
    # control quantize+dequantize, or simply quantize here down to mxfp4.
    A = quant_dequant_mxfp4(A)

    return A, None
```
**EN:** This function defines `_mxfp4_quantize`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `A`, `A_scale`, `per_act_token_quant`, `block_shape`. Key calls include `quant_dequant_mxfp4`. It writes or updates `A`.
**CN:** 该函数定义 `_mxfp4_quantize`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `A`, `A_scale`, `per_act_token_quant`, `block_shape`。 关键调用包括 `quant_dequant_mxfp4`。 它会写入或更新 `A`。

### Lines 205-216 — function `_mxfp8_e4m3_quantize`
```python
def _mxfp8_e4m3_quantize(
    A: torch.Tensor,
    A_scale: torch.Tensor | None,
    per_act_token_quant: bool,
    block_shape: list[int] | None = None,
    is_sf_swizzled_layout: bool = False,
    mx_alignment: int = 0,
) -> tuple[torch.Tensor, torch.Tensor]:
    assert A_scale is None
    assert not per_act_token_quant
    assert block_shape is None or block_shape == [1, 32]
    return mxfp8_e4m3_quantize(A, is_sf_swizzled_layout, mx_alignment)
```
**EN:** This function defines `_mxfp8_e4m3_quantize`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `A`, `A_scale`, `per_act_token_quant`, `block_shape`, `is_sf_swizzled_layout`, `mx_alignment`. Key calls include `mxfp8_e4m3_quantize`.
**CN:** 该函数定义 `_mxfp8_e4m3_quantize`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `A`, `A_scale`, `per_act_token_quant`, `block_shape`, `is_sf_swizzled_layout`, `mx_alignment`。 关键调用包括 `mxfp8_e4m3_quantize`。

### Lines 219-233 — function `_mxfp6_e3m2_quantize`
```python
def _mxfp6_e3m2_quantize(
    A: torch.Tensor,
    A_scale: torch.Tensor | None,
    per_act_token_quant: bool,
    block_shape: list[int] | None = None,
) -> tuple[torch.Tensor, None]:
    assert block_shape is None

    # TODO: native mxfp6 is currently not integrated in vllm,
    # so simulating even on devices supporting this data type natively.
    # Eventually, there should be a check based on
    # `current_platform.supports_mx()` here.
    A = quant_dequant_mxfp6(A, quant_dtype="fp6_e3m2")

    return A, None
```
**EN:** This function defines `_mxfp6_e3m2_quantize`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `A`, `A_scale`, `per_act_token_quant`, `block_shape`. Key calls include `quant_dequant_mxfp6`. It writes or updates `A`.
**CN:** 该函数定义 `_mxfp6_e3m2_quantize`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `A`, `A_scale`, `per_act_token_quant`, `block_shape`。 关键调用包括 `quant_dequant_mxfp6`。 它会写入或更新 `A`。

### Lines 236-250 — function `_mxfp6_e2m3_quantize`
```python
def _mxfp6_e2m3_quantize(
    A: torch.Tensor,
    A_scale: torch.Tensor | None,
    per_act_token_quant: bool,
    block_shape: list[int] | None = None,
) -> tuple[torch.Tensor, None]:
    assert block_shape is None

    # TODO: native mxfp6 is currently not integrated in vllm,
    # so simulating even on devices supporting this data type natively.
    # Eventually, there should be a check based on
    # `current_platform.supports_mx()` here.
    A = quant_dequant_mxfp6(A, quant_dtype="fp6_e2m3")

    return A, None
```
**EN:** This function defines `_mxfp6_e2m3_quantize`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `A`, `A_scale`, `per_act_token_quant`, `block_shape`. Key calls include `quant_dequant_mxfp6`. It writes or updates `A`.
**CN:** 该函数定义 `_mxfp6_e2m3_quantize`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `A`, `A_scale`, `per_act_token_quant`, `block_shape`。 关键调用包括 `quant_dequant_mxfp6`。 它会写入或更新 `A`。

### Lines 253-343 — function `moe_kernel_quantize_input`
```python
def moe_kernel_quantize_input(
    A: torch.Tensor,
    A_scale: torch.Tensor | None,
    quant_dtype: None | torch.dtype | str,
    per_act_token_quant: bool,
    block_shape: list[int] | None = None,
    is_scale_swizzled: bool = True,
    ocp_mx_scheme: str | None = None,
    quantization_emulation: bool = False,
    mx_alignment: int = 0,
) -> tuple[torch.Tensor, torch.Tensor | None]:
    # Handle OCP MX scheme that requires QDQ (quantize-dequantize) for emulation
    if ocp_mx_scheme is not None:
        if ocp_mx_scheme in {"w_mxfp4", "w_mxfp4_a_mxfp4"}:
            pass  # No QDQ needed for these schemes
        elif ocp_mx_scheme.endswith("a_fp8"):
            # Perform QDQ (quantize and dequantize) on activation for emulation
            # purpose, because there is no native kernel for weight in ocp_mx_scheme
            # and activation in FP8. The implementation is based on existing
            # non-emulation ops.
            qA, qA_scale = ops.scaled_fp8_quant(
                A, A_scale, use_per_token_if_dynamic=False
            )
            A = per_tensor_dequantize(qA, qA_scale).to(A.dtype)
            # After QDQ, we don't need further quantization
            return A, None
        # else: For other schemes (e.g., *_a_mxfp6_e3m2, *_a_mxfp6_e2m3),
        # weights are already dequantized, and we proceed with normal
        # activation quantization below.

    if quant_dtype == current_platform.fp8_dtype():
        if quantization_emulation:
            raise NotImplementedError(
                f"moe_kernel_quantize_input does not support quant_dtype={quant_dtype}"
                " MOE quantization emulation. Please open an issue."
            )
        return _fp8_quantize(A, A_scale, per_act_token_quant, block_shape)
    elif quant_dtype == torch.int8:
        if quantization_emulation:
            raise NotImplementedError(
                "moe_kernel_quantize_input does not support quant_dtype=torch.int8"
                " MOE quantization emulation. Please open an issue."
            )
        return _int8_quantize(A, A_scale, per_act_token_quant, block_shape)
# ... omitted for brevity ...
    else:
        return A, A_scale
```
**EN:** This function defines `moe_kernel_quantize_input`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `A`, `A_scale`, `quant_dtype`, `per_act_token_quant`, `block_shape`, `is_scale_swizzled`. Key calls include `current_platform.fp8_dtype`, `_fp8_quantize`, `ocp_mx_scheme.endswith`, `NotImplementedError`, `_int8_quantize`, `ops.scaled_fp8_quant`. It writes or updates `qA`, `qA_scale`, `A`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `moe_kernel_quantize_input`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `A`, `A_scale`, `quant_dtype`, `per_act_token_quant`, `block_shape`, `is_scale_swizzled`。 关键调用包括 `current_platform.fp8_dtype`, `_fp8_quantize`, `ocp_mx_scheme.endswith`, `NotImplementedError`, `_int8_quantize`, `ops.scaled_fp8_quant`。 它会写入或更新 `qA`, `qA_scale`, `A`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 346-352 — function `normalize_scales_shape`
```python
def normalize_scales_shape(scales: torch.Tensor | None) -> torch.Tensor | None:
    if scales is not None:
        if scales.numel() == 1:
            scales = scales.view(1, 1)
        else:
            scales = scales.view(-1, scales.size(-1))
    return scales
```
**EN:** This function defines `normalize_scales_shape`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `scales`. Key calls include `scales.numel`, `scales.view`, `scales.size`. It writes or updates `scales`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `normalize_scales_shape`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `scales`。 关键调用包括 `scales.numel`, `scales.view`, `scales.size`。 它会写入或更新 `scales`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 355-368 — function `normalize_batched_scales_shape`
```python
def normalize_batched_scales_shape(
    scales: torch.Tensor | None,
    num_experts: int,
) -> torch.Tensor | None:
    if scales is not None and scales.ndim < 3:
        if scales.numel() == 1:
            scales = scales.view(1)
            scales = torch.repeat_interleave(scales, num_experts, dim=0).view(
                num_experts, 1, 1
            )
        else:
            scales = scales.view(num_experts, -1, scales.size(-1))

    return scales
```
**EN:** This function defines `normalize_batched_scales_shape`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `scales`, `num_experts`. Key calls include `scales.numel`, `scales.view`, `torch.repeat_interleave.view`, `scales.size`, `torch.repeat_interleave`. It writes or updates `scales`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `normalize_batched_scales_shape`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `scales`, `num_experts`。 关键调用包括 `scales.numel`, `scales.view`, `torch.repeat_interleave.view`, `scales.size`, `torch.repeat_interleave`。 它会写入或更新 `scales`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `_count_expert_num_tokens`, `count_expert_num_tokens`, `_resize_cache`, `_nvfp4_quantize` / [CN] 核心符号：`_count_expert_num_tokens`, `count_expert_num_tokens`, `_resize_cache`, `_nvfp4_quantize`

## Dependencies / 依赖关系
- **External**: `functools`, `math`, `torch`, `torch.nn.functional` / **外部依赖**: `functools`, `math`, `torch`, `torch.nn.functional`
- **Internal**: `vllm`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.model_executor.layers.quantization.utils.int8_utils`, `vllm.model_executor.layers.quantization.utils.mxfp4_utils`, `vllm.model_executor.layers.quantization.utils.mxfp6_utils`, `vllm.model_executor.layers.quantization.utils.mxfp8_utils`, `vllm.model_executor.layers.quantization.utils.nvfp4_emulation_utils`, `vllm.model_executor.layers.quantization.utils.w8a8_utils`, `vllm.platforms`, `vllm.triton_utils` / **内部依赖**: `vllm`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.model_executor.layers.quantization.utils.int8_utils`, `vllm.model_executor.layers.quantization.utils.mxfp4_utils`, `vllm.model_executor.layers.quantization.utils.mxfp6_utils`, `vllm.model_executor.layers.quantization.utils.mxfp8_utils`, `vllm.model_executor.layers.quantization.utils.nvfp4_emulation_utils`, `vllm.model_executor.layers.quantization.utils.w8a8_utils`, `vllm.platforms`, `vllm.triton_utils`
- **Runtime traits**: platform-aware dispatch, Triton kernels, distributed collectives / **运行时特征**: platform-aware dispatch, Triton kernels, distributed collectives
