# nvfp4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/nvfp4.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: Imports and module setup
```python
from __future__ import annotations

import os
from typing import TYPE_CHECKING, Optional, Tuple

import torch

from sglang.jit_kernel.utils import cache_once, load_jit, override_jit_cuda_arch
from sglang.kernel_api_logging import debug_kernel_api
from sglang.srt.utils.custom_op import register_custom_op

if TYPE_CHECKING:
    from tvm_ffi.module import Module


_FLOAT4_E2M1_MAX = 6.0
_FLOAT8_E4M3_MAX = torch.finfo(torch.float8_e4m3fn).max


def _nvfp4_cuda_flags() -> list[str]:
    return [
        "-DNDEBUG",
        "-DFLASHINFER_ENABLE_F16",
        "-DCUTE_USE_PACKED_TUPLE=1",
        "-DCUTLASS_ENABLE_TENSOR_CORE_MMA=1",
        "-DCUTLASS_VERSIONS_GENERATED",
        "-DCUTLASS_TEST_LEVEL=0",
        "-DCUTLASS_TEST_ENABLE_CACHED_RESULTS=1",
        "-DCUTLASS_DEBUG_TRACE_LEVEL=0",
        "--expt-extended-lambda",
    ]
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 34-63: Function `_nvfp4_arch_env`
```python
def _nvfp4_arch_env():
    if not torch.cuda.is_available():
        raise RuntimeError("NVFP4 JIT kernels require CUDA.")
    major, minor = torch.cuda.get_device_capability()
    if major < 10:
        raise RuntimeError(
            f"NVFP4 JIT kernels require compute capability >= 10.0, got {major}.{minor}."
        )
    # NVFP4 kernels use architecture-family-specific instructions and must be
    # compiled for `sm_*a` targets (e.g. sm_100a), not plain sm_100.
    # JIT compilation targets only the current device, unlike AOT fat-binaries;
    # adding extra architectures here would clash with the single SGL_CUDA_ARCH
    # value injected by load_jit().
    return override_jit_cuda_arch(major, minor, suffix="a")


@torch.compiler.disable
def prewarm_nvfp4_jit_modules(
    *, include_expert_quant: bool = False, include_blockwise_moe: bool = False
) -> None:
    """Materialize NVFP4 JIT modules before torch.compile traces the model."""
    _jit_nvfp4_quant_module()
    _jit_nvfp4_scaled_mm_module()
    if include_expert_quant:
        _jit_nvfp4_expert_quant_module()
    if include_blockwise_moe:
        _jit_nvfp4_blockwise_moe_module()


@cache_once
```
**EN:** This block defines `_nvfp4_arch_env`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_nvfp4_arch_env`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 64-99: Function `_jit_nvfp4_quant_module`
```python
def _jit_nvfp4_quant_module() -> Module:
    with _nvfp4_arch_env():
        return load_jit(
            "nvfp4_quant",
            cuda_files=[
                "gemm/nvfp4/nvfp4_quant_kernels.cuh",
            ],
            cuda_wrappers=[
                ("scaled_fp4_quant", "scaled_fp4_quant_sm100a_sm120a"),
            ],
            extra_cuda_cflags=_nvfp4_cuda_flags(),
            extra_dependencies=["cutlass"],
        )


@cache_once
def _jit_nvfp4_expert_quant_module() -> Module:
    with _nvfp4_arch_env():
        return load_jit(
            "nvfp4_expert_quant",
            cuda_files=[
                "gemm/nvfp4/nvfp4_expert_quant.cuh",
            ],
            cuda_wrappers=[
                ("scaled_fp4_experts_quant", "scaled_fp4_experts_quant_sm100a"),
                (
                    "silu_and_mul_scaled_fp4_experts_quant",
                    "silu_and_mul_scaled_fp4_experts_quant_sm100a",
                ),
            ],
            extra_dependencies=["cutlass"],
            extra_cuda_cflags=_nvfp4_cuda_flags(),
        )


@cache_once
```
**EN:** This block defines `_jit_nvfp4_quant_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_nvfp4_quant_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 100-130: Function `_jit_nvfp4_scaled_mm_module`
```python
def _jit_nvfp4_scaled_mm_module() -> Module:
    with _nvfp4_arch_env():
        return load_jit(
            "nvfp4_scaled_mm",
            cuda_files=[
                "gemm/nvfp4/nvfp4_scaled_mm_kernels.cuh",
                "gemm/nvfp4/nvfp4_scaled_mm_entry.cuh",
            ],
            cuda_wrappers=[("cutlass_scaled_fp4_mm", "cutlass_scaled_fp4_mm")],
            extra_dependencies=["cutlass"],
            extra_cuda_cflags=_nvfp4_cuda_flags(),
        )


@cache_once
def _jit_nvfp4_blockwise_moe_module() -> Module:
    with _nvfp4_arch_env():
        return load_jit(
            "nvfp4_blockwise_moe",
            cuda_files=[
                "moe/nvfp4_blockwise_moe.cuh",
            ],
            cuda_wrappers=[
                ("cutlass_fp4_group_mm", "cutlass_fp4_group_mm_sm100a_sm120a")
            ],
            extra_dependencies=["cutlass"],
            extra_cuda_cflags=_nvfp4_cuda_flags(),
        )


@debug_kernel_api
```
**EN:** This block defines `_jit_nvfp4_scaled_mm_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_nvfp4_scaled_mm_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 131-216: Function `cutlass_scaled_fp4_mm`
```python
def cutlass_scaled_fp4_mm(
    a: torch.Tensor,
    b: torch.Tensor,
    block_scale_a: torch.Tensor,
    block_scale_b: torch.Tensor,
    alpha: torch.Tensor,
    out_dtype: torch.dtype,
) -> torch.Tensor:
    assert a.ndim == 2 and b.ndim == 2
    m, n = a.shape[0], b.shape[0]
    out = torch.empty((m, n), dtype=out_dtype, device=a.device)
    module = _jit_nvfp4_scaled_mm_module()
    module.cutlass_scaled_fp4_mm(out, a, b, block_scale_a, block_scale_b, alpha)
    return out


@debug_kernel_api
def cutlass_fp4_group_mm(
    a_fp4: torch.Tensor,
    b_fp4: torch.Tensor,
    a_blockscale: torch.Tensor,
    b_blockscale: torch.Tensor,
    alphas: torch.Tensor,
    out_dtype: torch.dtype,
    params: dict[str, torch.Tensor],
) -> torch.Tensor:
    m_topk = a_fp4.shape[0]
    n = b_fp4.shape[1]
    output = torch.empty((m_topk, n), device=a_fp4.device, dtype=out_dtype)
    num_experts = int(params["expert_offsets"].numel())
    device = a_fp4.device

    # Backward compatibility: older callers may not pass scratch tensors.
    a_ptrs = params.get(
        "a_ptrs", torch.empty((num_experts,), dtype=torch.int64, device=device)
    )
    b_ptrs = params.get(
        "b_ptrs", torch.empty((num_experts,), dtype=torch.int64, device=device)
    )
    out_ptrs = params.get(
        "out_ptrs", torch.empty((num_experts,), dtype=torch.int64, device=device)
    )
    a_scales_ptrs = params.get(
        "a_scales_ptrs", torch.empty((num_experts,), dtype=torch.int64, device=device)
    )
    b_scales_ptrs = params.get(
        "b_scales_ptrs", torch.empty((num_experts,), dtype=torch.int64, device=device)
    )
    alpha_ptrs = params.get(
        "alpha_ptrs", torch.empty((num_experts,), dtype=torch.int64, device=device)
    )
    layout_sfa = params.get(
        "layout_sfa", torch.empty((num_experts, 5), dtype=torch.int64, device=device)
    )
    layout_sfb = params.get(
        "layout_sfb", torch.empty((num_experts, 5), dtype=torch.int64, device=device)
    )

    _cutlass_fp4_group_mm_custom_op(
        output,
        a_fp4,
        b_fp4,
        a_blockscale,
        b_blockscale,
        alphas,
        params["ab_strides"],
        params["c_strides"],
        params["problem_sizes"],
        params["expert_offsets"],
        params["blockscale_offsets"],
        a_ptrs,
        b_ptrs,
        out_ptrs,
        a_scales_ptrs,
        b_scales_ptrs,
        alpha_ptrs,
        layout_sfa,
        layout_sfb,
    )
    return output
# ...
```
**EN:** This block defines `cutlass_scaled_fp4_mm`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `cutlass_scaled_fp4_mm`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 217-263: Function `_scaled_fp4_quant_custom_op`
```python
def _scaled_fp4_quant_custom_op(
    input: torch.Tensor,
    output: torch.Tensor,
    output_scale: torch.Tensor,
    input_global_scale: torch.Tensor,
) -> None:
    module = _jit_nvfp4_quant_module()
    module.scaled_fp4_quant(output, input, output_scale, input_global_scale)


@debug_kernel_api
def scaled_fp4_quant(
    input: torch.Tensor, input_global_scale: torch.Tensor
) -> Tuple[torch.Tensor, torch.Tensor]:
    """Quantize input tensor to FP4 and return packed FP4 tensor + swizzled scales."""
    assert input.ndim >= 1, f"input.ndim needs to be >= 1, but got {input.ndim}."
    other_dims = 1 if input.ndim == 1 else -1
    input = input.reshape(other_dims, input.shape[-1])
    m, n = input.shape
    block_size = 16
    device = input.device

    assert n % block_size == 0, f"last dim has to be multiple of 16, but got {n}."
    assert input.dtype in (
        torch.float16,
        torch.bfloat16,
    ), f"input.dtype needs to be fp16 or bf16 but got {input.dtype}."

    output = torch.empty((m, n // 2), device=device, dtype=torch.uint8)

    rounded_m = ((m + 128 - 1) // 128) * 128
    scale_n = n // block_size
    rounded_n = ((scale_n + 4 - 1) // 4) * 4
    if rounded_n > scale_n:
        output_scale = torch.zeros(
            (rounded_m, rounded_n // 4), device=device, dtype=torch.int32
        )
    else:
        output_scale = torch.empty(
            (rounded_m, rounded_n // 4), device=device, dtype=torch.int32
        )

    _scaled_fp4_quant_custom_op(input, output, output_scale, input_global_scale)
    output_scale = output_scale.view(torch.float8_e4m3fn)
    return output, output_scale
```
**EN:** This block defines `_scaled_fp4_quant_custom_op`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_scaled_fp4_quant_custom_op`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 264-297: Function `_shuffle_rows_torch`
```python
def _shuffle_rows_torch(
    input_tensor: torch.Tensor,
    dst2src_map: torch.Tensor,
    output_tensor_shape: tuple[int, int],
) -> torch.Tensor:
    # Keep compatibility when sgl-kernel is slimmed and shuffle_rows may not be present.
    output = input_tensor.index_select(0, dst2src_map.to(dtype=torch.int64))
    return output.view(output_tensor_shape)


@register_custom_op(
    op_name="scaled_fp4_experts_quant",
    mutates_args=["output", "output_scales"],
)
def _scaled_fp4_experts_quant_custom_op(
    output: torch.Tensor,
    output_scales: torch.Tensor,
    input_tensor: torch.Tensor,
    input_global_scale: torch.Tensor,
    expert_offsets: torch.Tensor,
    blockscale_offsets: torch.Tensor,
) -> None:
    module = _jit_nvfp4_expert_quant_module()
    module.scaled_fp4_experts_quant(
        output,
        output_scales,
        input_tensor,
        input_global_scale,
        expert_offsets,
        blockscale_offsets,
    )


@debug_kernel_api
```
**EN:** This block defines `_shuffle_rows_torch`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_shuffle_rows_torch`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 298-382: Function `scaled_fp4_experts_quant`
```python
def scaled_fp4_experts_quant(
    input_tensor: torch.Tensor,
    input_global_scale: torch.Tensor,
    expert_offsets: torch.Tensor,
    blockscale_offsets: torch.Tensor,
    topk: int,
    expert_map: Optional[torch.Tensor] = None,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Quantize packed MoE activations to NVFP4."""
    assert (
        input_tensor.ndim == 2
    ), f"input.ndim needs to be == 2, but got {input_tensor.ndim}."
    if expert_map is not None:
        m, k = input_tensor.shape
        output_tensor_shape = (m * topk, k)
        input_tensor = _shuffle_rows_torch(
            input_tensor, expert_map, output_tensor_shape
        )

    m_numtopk, k = input_tensor.shape
    max_tokens_per_expert = int(os.environ.get("MODELOPT_MAX_TOKENS_PER_EXPERT", 65536))
    assert m_numtopk <= max_tokens_per_expert * topk, (
        f"m_numtopk must be less than MAX_TOKENS_PER_EXPERT({max_tokens_per_expert})"
        f" for cutlass_moe_fp4, observed m_numtopk = {m_numtopk}. Use"
        " MODELOPT_MAX_TOKENS_PER_EXPERT to set this value."
    )
    scales_k = k // 16
    # output_scales is int32-packed FP8 scales, so second dim is in int32 units.
    padded_k_in_int32 = (scales_k + 3) // 4

    output = torch.empty(
        m_numtopk, k // 2, device=input_tensor.device, dtype=torch.uint8
    )
    if padded_k_in_int32 * 4 > scales_k:
        output_scales = torch.zeros(
            max_tokens_per_expert * topk,
            padded_k_in_int32,
            dtype=torch.int32,
            device=input_tensor.device,
        )
    else:
        output_scales = torch.empty(
            max_tokens_per_expert * topk,
            padded_k_in_int32,
            dtype=torch.int32,
            device=input_tensor.device,
        )

    _scaled_fp4_experts_quant_custom_op(
        output,
        output_scales,
        input_tensor,
        input_global_scale,
        expert_offsets,
        blockscale_offsets,
    )
    output_scales = output_scales.view(torch.float8_e4m3fn)
    return output, output_scales


@register_custom_op(
    op_name="scaled_fp4_grouped_quant",
    mutates_args=["output", "output_scales"],
)
def _scaled_fp4_grouped_quant_custom_op(
    input_tensor: torch.Tensor,
    output: torch.Tensor,
    output_scales: torch.Tensor,
    input_global_scale: torch.Tensor,
    mask: torch.Tensor,
) -> None:
    l, m, k = input_tensor.shape
    del l, m
    module = _jit_nvfp4_expert_quant_module()
    module.silu_and_mul_scaled_fp4_experts_quant(
        output.view(-1, k // 2),
        output_scales.view(-1, output_scales.shape[-1]),
        input_tensor.view(-1, k),
        input_global_scale,
        mask,
# ...
```
**EN:** This block defines `scaled_fp4_experts_quant`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `scaled_fp4_experts_quant`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 383-443: Function `scaled_fp4_grouped_quant`
```python
def scaled_fp4_grouped_quant(
    input_tensor: torch.Tensor,
    input_global_scale: torch.Tensor,
    mask: torch.Tensor,
):
    """Quantize grouped GEMM inputs to FP4 and return logical (m, k//2, l)."""
    device = input_tensor.device
    l, m, k = input_tensor.shape
    sf_vec_size = 16
    assert k % sf_vec_size == 0, f"k must be multiple of 16, but got {k}."

    scale_k = k // sf_vec_size
    padded_k = (scale_k + (4 - 1)) // 4 * 4
    padded_k_int32 = padded_k // 4
    padded_m = (m + (128 - 1)) // 128 * 128
    output = torch.empty(l, m, k // 2, device=device, dtype=torch.uint8)
    output_scales = torch.empty(
        l, padded_m, padded_k_int32, device=device, dtype=torch.int32
    )

    _scaled_fp4_grouped_quant_custom_op(
        input_tensor,
        output,
        output_scales,
        input_global_scale,
        mask,
    )

    output = output.permute(1, 2, 0)
    output_scales = output_scales.view(torch.float8_e4m3fn).view(
        l, padded_m // 128, padded_k // 4, 32, 4, 4
    )
    output_scales = output_scales.permute(3, 4, 1, 5, 2, 0)
    return output, output_scales


@register_custom_op(
    op_name="silu_and_mul_scaled_fp4_grouped_quant",
    mutates_args=["output", "output_scales"],
)
def _silu_and_mul_scaled_fp4_grouped_quant_custom_op(
    input_tensor: torch.Tensor,
    output: torch.Tensor,
    output_scales: torch.Tensor,
    input_global_scale: torch.Tensor,
    mask: torch.Tensor,
) -> None:
    l, m, k_by_2 = input_tensor.shape
    del l, m
    module = _jit_nvfp4_expert_quant_module()
    module.silu_and_mul_scaled_fp4_experts_quant(
        output.view(-1, output.shape[-1]),
        output_scales.view(-1, output_scales.shape[-1]),
        input_tensor.view(-1, k_by_2),
        input_global_scale,
        mask,
        True,
    )


@debug_kernel_api
```
**EN:** This block defines `scaled_fp4_grouped_quant`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `scaled_fp4_grouped_quant`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 444-539: Function `silu_and_mul_scaled_fp4_grouped_quant`
```python
def silu_and_mul_scaled_fp4_grouped_quant(
    input_tensor: torch.Tensor,
    input_global_scale: torch.Tensor,
    mask: torch.Tensor,
):
    """Apply SiLU-and-mul then quantize grouped GEMM inputs to FP4."""
    device = input_tensor.device
    l, m, k_by_2 = input_tensor.shape
    k = k_by_2 // 2
    sf_vec_size = 16
    assert k % sf_vec_size == 0, f"k must be multiple of 16, but got {k}."

    scale_k = k // sf_vec_size
    padded_k = (scale_k + (4 - 1)) // 4 * 4
    padded_k_int32 = padded_k // 4
    padded_m = (m + (128 - 1)) // 128 * 128
    output = torch.empty(l, m, k // 2, device=device, dtype=torch.uint8)
    output_scales = torch.empty(
        l, padded_m, padded_k_int32, device=device, dtype=torch.int32
    )

    _silu_and_mul_scaled_fp4_grouped_quant_custom_op(
        input_tensor,
        output,
        output_scales,
        input_global_scale,
        mask,
    )

    output = output.permute(1, 2, 0)
    output_scales = output_scales.view(torch.float8_e4m3fn).view(
        l, padded_m // 128, padded_k // 4, 32, 4, 4
    )
    output_scales = output_scales.permute(3, 4, 1, 5, 2, 0)
    return output, output_scales


@register_custom_op(
    op_name="cutlass_fp4_group_mm",
    mutates_args=[
        "output",
        "a_ptrs",
        "b_ptrs",
        "out_ptrs",
        "a_scales_ptrs",
        "b_scales_ptrs",
        "alpha_ptrs",
        "layout_sfa",
        "layout_sfb",
    ],
)
def _cutlass_fp4_group_mm_custom_op(
    output: torch.Tensor,
    a_fp4: torch.Tensor,
    b_fp4: torch.Tensor,
    a_blockscale: torch.Tensor,
    b_blockscale: torch.Tensor,
    alphas: torch.Tensor,
    ab_strides: torch.Tensor,
    c_strides: torch.Tensor,
    problem_sizes: torch.Tensor,
    expert_offsets: torch.Tensor,
    blockscale_offsets: torch.Tensor,
    a_ptrs: torch.Tensor,
    b_ptrs: torch.Tensor,
    out_ptrs: torch.Tensor,
    a_scales_ptrs: torch.Tensor,
    b_scales_ptrs: torch.Tensor,
    alpha_ptrs: torch.Tensor,
    layout_sfa: torch.Tensor,
    layout_sfb: torch.Tensor,
) -> None:
    module = _jit_nvfp4_blockwise_moe_module()
    module.cutlass_fp4_group_mm(
        output,
        a_fp4,
        b_fp4,
        a_blockscale,
        b_blockscale,
        alphas,
# ...
```
**EN:** This block defines `silu_and_mul_scaled_fp4_grouped_quant`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `silu_and_mul_scaled_fp4_grouped_quant`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 540-543: Function `suggest_nvfp4_global_scale`
```python
def suggest_nvfp4_global_scale(x: torch.Tensor) -> torch.Tensor:
    """Utility for tests/benchmarks: return global scale used by NVFP4 quantization."""
    tensor_amax = torch.abs(x).max().to(torch.float32)
    return _FLOAT8_E4M3_MAX * _FLOAT4_E2M1_MAX / tensor_amax
```
**EN:** This block defines `suggest_nvfp4_global_scale`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `suggest_nvfp4_global_scale`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `os`
- `typing -> TYPE_CHECKING`
- `torch`
- `sglang.jit_kernel.utils -> cache_once`
- `sglang.kernel_api_logging -> debug_kernel_api`
- `sglang.srt.utils.custom_op -> register_custom_op`
- `tvm_ffi.module -> Module`
