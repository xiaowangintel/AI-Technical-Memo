# musa.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/python/sgl_kernel/musa.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module wraps native operators related to `MUSA`, performs Python-side validation, and exposes a convenient interface. / 该模块封装与 `MUSA` 相关的原生算子，执行 Python 侧校验，并暴露便捷接口。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Imports and module setup
````python
from typing import Optional, Union

import torch
from sgl_kernel.utils import _to_tensor_scalar_tuple
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 7-26: `musa_batched_rotary_embedding_contiguous` definition
````python
def musa_batched_rotary_embedding_contiguous(
    positions: torch.Tensor,
    query: torch.Tensor,
    key: torch.Tensor,
    head_size: int,
    cos_sin_cache: torch.Tensor,
    is_neox: bool,
    rot_dim: int,
    cos_sin_cache_offsets: torch.Tensor,
) -> None:
    return torch.ops.sgl_kernel.musa_batched_rotary_embedding_contiguous(
        positions,
        query,
        key,
        head_size,
        cos_sin_cache,
        is_neox,
        rot_dim,
        cos_sin_cache_offsets,
    )
````
**EN:** This section defines `musa_batched_rotary_embedding_contiguous` and implements the core logic associated with MUSA batched rotary embedding contiguous. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `musa_batched_rotary_embedding_contiguous`，并实现与 MUSA batched rotary embedding contiguous 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 29-44: `musa_rotary_embedding_contiguous` definition
````python
def musa_rotary_embedding_contiguous(
    positions: torch.Tensor,
    query: torch.Tensor,
    key: torch.Tensor,
    head_size: int,
    cos_sin_cache: torch.Tensor,
    is_neox: bool,
) -> None:
    return torch.ops.sgl_kernel.musa_rotary_embedding_contiguous(
        positions,
        query,
        key,
        head_size,
        cos_sin_cache,
        is_neox,
    )
````
**EN:** This section defines `musa_rotary_embedding_contiguous` and implements the core logic associated with MUSA rotary embedding contiguous. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `musa_rotary_embedding_contiguous`，并实现与 MUSA rotary embedding contiguous 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 47-72: `musa_fused_moe_gemv` definition
````python
def musa_fused_moe_gemv(
    A: torch.Tensor,
    B: torch.Tensor,
    C: torch.Tensor,
    A_scale,
    B_scale,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    mul_routed_weight: bool,
    topk: int,
    use_int4_w4a16: bool,
    use_swigelu: bool,
) -> None:
    return torch.ops.sgl_kernel.musa_fused_moe_gemv(
        A,
        B,
        C,
        A_scale,
        B_scale,
        topk_weights,
        topk_ids,
        mul_routed_weight,
        topk,
        use_int4_w4a16,
        use_swigelu,
    )
````
**EN:** This section defines `musa_fused_moe_gemv` and implements the core logic associated with MUSA fused MoE GEMV. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `musa_fused_moe_gemv`，并实现与 MUSA fused MoE GEMV 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 75-152: `musa_fused_gemv` definition
````python
def musa_fused_gemv(
    x: torch.Tensor,
    qweight: torch.Tensor,
    x_scales: Optional[torch.Tensor] = None,
    qweight_scales: Optional[torch.Tensor] = None,
    use_swigelu: bool = False,
    use_rms_norm: bool = False,
    gamma: Optional[torch.Tensor] = None,
    eps: float = 1e-6,
):
    use_int4_w4a16 = False
    out_shape = x.shape[:-1] + (
        qweight.shape[0] if not use_swigelu else qweight.shape[0] // 2,
    )
    assert not (
        use_swigelu and use_rms_norm
    ), "gemv only fused one activation (swigelu or rms_norm)!"

    if use_rms_norm:
        if gamma is None:
            assert False, "rms_norm gamma is None!"

    # fp8 grouped matmul
    if qweight.dtype == torch.float8_e4m3fn:
        assert qweight_scales is not None, "FP8 grouped matmul weight scales is None!"
        output = torch.empty(out_shape, device=x.device, dtype=torch.bfloat16)
        torch.ops.sgl_kernel.musa_fused_gemv(
            x,
            qweight,
            output,
            x_scales,
            qweight_scales,
            use_int4_w4a16,
            use_swigelu,
            use_rms_norm,
            gamma,
            eps,
        )
        return output
    # w4a16 gemv
    elif qweight_scales is not None:
        assert (
            x.dtype == torch.bfloat16 or x.dtype == torch.float16
        ), "W4A16 gemv only support bfloat16 or float16!"
        use_int4_w4a16 = True
        out_shape = x.shape[:-1] + (
            qweight.shape[0] if not use_swigelu else qweight.shape[0] // 2,
        )
        output = torch.empty(out_shape, device=x.device, dtype=x.dtype)
        torch.ops.sgl_kernel.musa_fused_gemv(
            x,
            qweight,
            output,
            None,
            qweight_scales,
            use_int4_w4a16,
            use_swigelu,
            use_rms_norm,
            gamma,
            eps,
        )
        return output
    # general gemv
    else:
        output = torch.empty(out_shape, device=x.device, dtype=x.dtype)
        torch.ops.sgl_kernel.musa_fused_gemv(
            x,
            qweight,
            output,
            None,
            None,
            use_int4_w4a16,
            use_swigelu,
            use_rms_norm,
            gamma,
            eps,
        )
        return output
````
**EN:** This section defines `musa_fused_gemv` and implements the core logic associated with MUSA fused GEMV. It also performs explicit assertions or shape checks before continuing. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `musa_fused_gemv`，并实现与 MUSA fused GEMV 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 155-170: `musa_fused_mul_add` definition
````python
def musa_fused_mul_add(
    self: torch.Tensor,
    bias: Optional[torch.Tensor],
    scale: Optional[float],
    accurate: bool = True,
):
    # if accurate == False, then we call inplace op: bias += (self * scale)
    if not accurate:
        bias.add_(self, alpha=scale)
        return bias

    # otherwise, we call custom outplace op, act: output = self * scale + bias
    output = torch.empty_like(self)
    torch.ops.sgl_kernel.musa_fused_mul_add(output, self, bias, scale)

    return output
````
**EN:** This section defines `musa_fused_mul_add` and implements the core logic associated with MUSA fused multiply add. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `musa_fused_mul_add`，并实现与 MUSA fused multiply add 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 173-184: `_top_k_renorm_probs_internal` definition
````python
def _top_k_renorm_probs_internal(
    probs: torch.Tensor,
    maybe_top_k_arr: Optional[torch.Tensor],
    top_k_val: int,
) -> torch.Tensor:
    probs = probs.float()
    maybe_top_k_arr = maybe_top_k_arr.int() if maybe_top_k_arr is not None else None
    renorm_probs = torch.empty_like(probs)
    torch.ops.sgl_kernel.top_k_renorm_probs.default(
        probs, renorm_probs, maybe_top_k_arr, top_k_val
    )
    return renorm_probs
````
**EN:** This section defines `_top_k_renorm_probs_internal` and implements the core logic associated with top k renorm probs internal. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `_top_k_renorm_probs_internal`，并实现与 top k renorm probs internal 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 187-191: `top_k_renorm_probs` definition
````python
def top_k_renorm_probs(
    probs: torch.Tensor,
    top_k: Union[torch.Tensor, int],
) -> torch.Tensor:
    return _top_k_renorm_probs_internal(probs, *_to_tensor_scalar_tuple(top_k))
````
**EN:** This section defines `top_k_renorm_probs` and implements the core logic associated with top k renorm probs.
**CN:** 该部分定义 `top_k_renorm_probs`，并实现与 top k renorm probs 相关的核心逻辑。

### Lines 194-205: `_top_p_renorm_probs_internal` definition
````python
def _top_p_renorm_probs_internal(
    probs: torch.Tensor,
    maybe_top_p_arr: Optional[torch.Tensor],
    top_p_val: float,
) -> torch.Tensor:
    probs = probs.float()
    maybe_top_p_arr = maybe_top_p_arr.float() if maybe_top_p_arr is not None else None
    renorm_probs = torch.empty_like(probs)
    torch.ops.sgl_kernel.top_p_renorm_probs.default(
        probs, renorm_probs, maybe_top_p_arr, top_p_val
    )
    return renorm_probs
````
**EN:** This section defines `_top_p_renorm_probs_internal` and implements the core logic associated with top p renorm probs internal. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `_top_p_renorm_probs_internal`，并实现与 top p renorm probs internal 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 208-212: `top_p_renorm_probs` definition
````python
def top_p_renorm_probs(
    probs: torch.Tensor,
    top_p: Union[torch.Tensor, float],
) -> torch.Tensor:
    return _top_p_renorm_probs_internal(probs, *_to_tensor_scalar_tuple(top_p))
````
**EN:** This section defines `top_p_renorm_probs` and implements the core logic associated with top p renorm probs.
**CN:** 该部分定义 `top_p_renorm_probs`，并实现与 top p renorm probs 相关的核心逻辑。

### Lines 215-236: `_top_p_sampling_from_probs_internal` definition
````python
def _top_p_sampling_from_probs_internal(
    probs: torch.Tensor,
    indices: Optional[torch.Tensor],
    maybe_top_p_arr: Optional[torch.Tensor],
    top_p_val: float,
    deterministic: bool,
    generator: Optional[torch.Generator],
) -> torch.Tensor:
    device = probs.device
    probs = probs.float()
    maybe_top_p_arr = maybe_top_p_arr.float() if maybe_top_p_arr is not None else None
    samples = torch.empty(probs.size(0), dtype=torch.int32, device=device)
    torch.ops.sgl_kernel.top_p_sampling_from_probs.default(
        probs,
        samples,
        indices,
        maybe_top_p_arr,
        top_p_val,
        deterministic,
        generator,
    )
    return samples
````
**EN:** This section defines `_top_p_sampling_from_probs_internal` and implements the core logic associated with top p sampling from probs internal. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `_top_p_sampling_from_probs_internal`，并实现与 top p sampling from probs internal 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 239-251: `top_p_sampling_from_probs` definition
````python
def top_p_sampling_from_probs(
    probs: torch.Tensor,
    top_p: Union[torch.Tensor, float],
    indices: Optional[torch.Tensor] = None,
    deterministic: bool = True,
    generator: Optional[torch.Generator] = None,
    check_nan: bool = False,
) -> torch.Tensor:
    if check_nan and torch.any(torch.isnan(probs)):
        raise ValueError("Input probs contains NaN.")
    return _top_p_sampling_from_probs_internal(
        probs, indices, *_to_tensor_scalar_tuple(top_p), deterministic, generator
    )
````
**EN:** This section defines `top_p_sampling_from_probs` and implements the core logic associated with top p sampling from probs.
**CN:** 该部分定义 `top_p_sampling_from_probs`，并实现与 top p sampling from probs 相关的核心逻辑。

### Lines 254-280: `_top_k_top_p_sampling_from_probs_internal` definition
````python
def _top_k_top_p_sampling_from_probs_internal(
    probs: torch.Tensor,
    indices: Optional[torch.Tensor],
    maybe_top_k_arr: Optional[torch.Tensor],
    top_k_val: int,
    maybe_top_p_arr: Optional[torch.Tensor],
    top_p_val: float,
    deterministic: bool,
    generator: Optional[torch.Generator],
) -> torch.Tensor:
    device = probs.device
    probs = probs.float()
    maybe_top_k_arr = maybe_top_k_arr.int() if maybe_top_k_arr is not None else None
    maybe_top_p_arr = maybe_top_p_arr.float() if maybe_top_p_arr is not None else None
    samples = torch.empty(probs.size(0), dtype=torch.int32, device=device)
    torch.ops.sgl_kernel.musa_top_k_top_p_sampling_from_probs.default(
        probs,
        samples,
        indices,
        maybe_top_k_arr,
        top_k_val,
        maybe_top_p_arr,
        top_p_val,
        deterministic,
        generator,
    )
    return samples
````
**EN:** This section defines `_top_k_top_p_sampling_from_probs_internal` and implements the core logic associated with top k top p sampling from probs internal. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `_top_k_top_p_sampling_from_probs_internal`，并实现与 top k top p sampling from probs internal 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 283-314: `top_k_top_p_sampling_from_probs` definition
````python
def top_k_top_p_sampling_from_probs(
    probs: torch.Tensor,
    top_k: Union[torch.Tensor, int],
    top_p: Union[torch.Tensor, float],
    indices: Optional[torch.Tensor] = None,
    filter_apply_order: str = "top_k_first",
    deterministic: bool = True,
    generator: Optional[torch.Generator] = None,
    check_nan: bool = False,
) -> torch.Tensor:
    if filter_apply_order == "top_k_first":
        renorm_probs = top_k_renorm_probs(probs, top_k)
        return top_p_sampling_from_probs(
            renorm_probs,
            top_p,
            indices,
            deterministic,
            generator=generator,
            check_nan=check_nan,
        )
    if filter_apply_order == "joint":
        if check_nan and torch.any(torch.isnan(probs)):
            raise ValueError("Input probs contains NaN.")
        return _top_k_top_p_sampling_from_probs_internal(
            probs,
            indices,
            *_to_tensor_scalar_tuple(top_k),
            *_to_tensor_scalar_tuple(top_p),
            deterministic,
            generator,
        )
    raise ValueError(f"Invalid filter_apply_order: {filter_apply_order}")
````
**EN:** This section defines `top_k_top_p_sampling_from_probs` and implements the core logic associated with top k top p sampling from probs.
**CN:** 该部分定义 `top_k_top_p_sampling_from_probs`，并实现与 top k top p sampling from probs 相关的核心逻辑。

### Lines 317-338: `_min_p_sampling_from_probs_internal` definition
````python
def _min_p_sampling_from_probs_internal(
    probs: torch.Tensor,
    indices: Optional[torch.Tensor],
    maybe_min_p_arr: Optional[torch.Tensor],
    min_p_val: float,
    deterministic: bool,
    generator: Optional[torch.Generator],
) -> torch.Tensor:
    device = probs.device
    probs = probs.float()
    maybe_min_p_arr = maybe_min_p_arr.float() if maybe_min_p_arr is not None else None
    samples = torch.empty(probs.size(0), dtype=torch.int32, device=device)
    torch.ops.sgl_kernel.min_p_sampling_from_probs.default(
        probs,
        samples,
        indices,
        maybe_min_p_arr,
        min_p_val,
        deterministic,
        generator,
    )
    return samples
````
**EN:** This section defines `_min_p_sampling_from_probs_internal` and implements the core logic associated with min p sampling from probs internal. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `_min_p_sampling_from_probs_internal`，并实现与 min p sampling from probs internal 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 341-353: `min_p_sampling_from_probs` definition
````python
def min_p_sampling_from_probs(
    probs: torch.Tensor,
    min_p: Union[torch.Tensor, float],
    indices: Optional[torch.Tensor] = None,
    deterministic: bool = True,
    generator: Optional[torch.Generator] = None,
    check_nan: bool = False,
) -> torch.Tensor:
    if check_nan and torch.any(torch.isnan(probs)):
        raise ValueError("Input probs contains NaN.")
    return _min_p_sampling_from_probs_internal(
        probs, indices, *_to_tensor_scalar_tuple(min_p), deterministic, generator
    )
````
**EN:** This section defines `min_p_sampling_from_probs` and implements the core logic associated with min p sampling from probs.
**CN:** 该部分定义 `min_p_sampling_from_probs`，并实现与 min p sampling from probs 相关的核心逻辑。

## Key Concepts / 关键概念
- **Role / 角色**: Python wrapper over compiled ops / 已编译算子的 Python 封装
- **Primary symbols / 主要符号**: `musa_batched_rotary_embedding_contiguous`, `musa_rotary_embedding_contiguous`, `musa_fused_moe_gemv`, `musa_fused_gemv`, `musa_fused_mul_add`, `_top_k_renorm_probs_internal`, `top_k_renorm_probs`, `_top_p_renorm_probs_internal`
- **Core technologies / 核心技术**: compiled custom ops / 已编译自定义算子, PyTorch tensors / PyTorch 张量

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel.utils`
- **External / 外部**: `torch`, `typing`
- **Runtime hooks / 运行时钩子**: `torch.ops.sgl_kernel` custom operator namespace
