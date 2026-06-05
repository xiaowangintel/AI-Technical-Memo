# gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/python/sgl_kernel/gemm.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module wraps native operators related to `GEMM`, performs Python-side validation, and exposes a convenient interface. / 该模块封装与 `GEMM` 相关的原生算子，执行 Python 侧校验，并暴露便捷接口。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Imports and module setup
````python
from typing import Optional

import torch
from sgl_kernel.utils import _get_cache_buf
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 7-10: `awq_dequantize` definition
````python
def awq_dequantize(
    qweight: torch.Tensor, scales: torch.Tensor, qzeros: torch.Tensor
) -> torch.ByteTensor:
    return torch.ops.sgl_kernel.awq_dequantize.default(qweight, scales, qzeros)
````
**EN:** This section defines `awq_dequantize` and implements the core logic associated with AWQ dequantize. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `awq_dequantize`，并实现与 AWQ dequantize 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 13-21: `int8_scaled_mm` definition
````python
def int8_scaled_mm(mat_a, mat_b, scales_a, scales_b, out_dtype, bias=None):
    return torch.ops.sgl_kernel.int8_scaled_mm.default(
        mat_a,
        mat_b,
        scales_a,
        scales_b,
        out_dtype,
        bias,
    )
````
**EN:** This section defines `int8_scaled_mm` and implements the core logic associated with INT8 scaled mm. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `int8_scaled_mm`，并实现与 INT8 scaled mm 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 24-31: `fp8_blockwise_scaled_mm` definition
````python
def fp8_blockwise_scaled_mm(mat_a, mat_b, scales_a, scales_b, out_dtype):
    return torch.ops.sgl_kernel.fp8_blockwise_scaled_mm.default(
        mat_a,
        mat_b,
        scales_a,
        scales_b,
        out_dtype,
    )
````
**EN:** This section defines `fp8_blockwise_scaled_mm` and implements the core logic associated with FP8 blockwise scaled mm. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `fp8_blockwise_scaled_mm`，并实现与 FP8 blockwise scaled mm 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 34-42: `fp8_scaled_mm` definition
````python
def fp8_scaled_mm(mat_a, mat_b, scales_a, scales_b, out_dtype, bias=None):
    return torch.ops.sgl_kernel.fp8_scaled_mm.default(
        mat_a,
        mat_b,
        scales_a,
        scales_b,
        out_dtype,
        bias,
    )
````
**EN:** This section defines `fp8_scaled_mm` and implements the core logic associated with FP8 scaled mm. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `fp8_scaled_mm`，并实现与 FP8 scaled mm 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 45-62: `_bmm_fp8_internal` definition
````python
def _bmm_fp8_internal(
    workspace_buffer: torch.Tensor,
    A: torch.Tensor,
    B: torch.Tensor,
    D: torch.Tensor,
    A_scale: torch.Tensor,
    B_scale: torch.Tensor,
) -> None:
    cublas_handle = torch.cuda.current_blas_handle()
    torch.ops.sgl_kernel.bmm_fp8.default(
        A,
        B,
        D,
        A_scale,
        B_scale,
        workspace_buffer,
        cublas_handle,
    )
````
**EN:** This section defines `_bmm_fp8_internal` and implements the core logic associated with BMM FP8 internal. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `_bmm_fp8_internal`，并实现与 BMM FP8 internal 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 65-81: `bmm_fp8` definition
````python
def bmm_fp8(
    A: torch.Tensor,
    B: torch.Tensor,
    A_scale: torch.Tensor,
    B_scale: torch.Tensor,
    dtype: torch.dtype,
    out: Optional[torch.Tensor] = None,
) -> torch.Tensor:
    if out is None:
        out = torch.empty(
            (A.shape[0], A.shape[1], B.shape[2]),
            device=A.device,
            dtype=dtype,
        )
    workspace_buffer = _get_cache_buf("bmm_fp8_workspace", 32 * 1024 * 1024, A.device)
    _bmm_fp8_internal(workspace_buffer, A, B, out, A_scale, B_scale)
    return out
````
**EN:** This section defines `bmm_fp8` and implements the core logic associated with BMM FP8.
**CN:** 该部分定义 `bmm_fp8`，并实现与 BMM FP8 相关的核心逻辑。

### Lines 84-96: `dsv3_fused_a_gemm` definition
````python
def dsv3_fused_a_gemm(
    mat_a: torch.Tensor,
    mat_b: torch.Tensor,
    output: Optional[torch.Tensor] = None,
) -> torch.Tensor:
    if output is None:
        output = torch.empty(
            (mat_a.shape[0], mat_b.shape[1]),
            device=mat_a.device,
            dtype=mat_a.dtype,
        )
    torch.ops.sgl_kernel.dsv3_fused_a_gemm.default(output, mat_a, mat_b)
    return output
````
**EN:** This section defines `dsv3_fused_a_gemm` and implements the core logic associated with DeepSeek-V3 fused a GEMM. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `dsv3_fused_a_gemm`，并实现与 DeepSeek-V3 fused a GEMM 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 99-134: `sgl_per_token_group_quant_8bit` definition
````python
def sgl_per_token_group_quant_8bit(
    input: torch.Tensor,
    output_q: torch.Tensor,
    output_s: torch.Tensor,
    group_size: int,
    eps: float,
    fp8_min: float,
    fp8_max: float,
    scale_ue8m0: bool = False,
    fuse_silu_and_mul: bool = False,
    masked_m: Optional[torch.Tensor] = None,
    enable_v2: Optional[bool] = None,
) -> None:
    _V2_KERNEL_SUPPORTED_GROUP_SIZES = [16, 32, 64, 128]
    if enable_v2 is None:
        enable_v2 = group_size in _V2_KERNEL_SUPPORTED_GROUP_SIZES

    if enable_v2:
        return torch.ops.sgl_kernel.sgl_per_token_group_quant_8bit_v2.default(
            input,
            output_q,
            output_s,
            group_size,
            eps,
            fp8_min,
            fp8_max,
            scale_ue8m0,
            fuse_silu_and_mul,
            masked_m,
        )

    assert not fuse_silu_and_mul, "only v2 support fuse_silu_and_mul"
    assert masked_m is None, "only v2 support masked_m"
    torch.ops.sgl_kernel.sgl_per_token_group_quant_8bit.default(
        input, output_q, output_s, group_size, eps, fp8_min, fp8_max, scale_ue8m0
    )
````
**EN:** This section defines `sgl_per_token_group_quant_8bit` and implements the core logic associated with SGL per token group quantization 8bit. It also performs explicit assertions or shape checks before continuing. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `sgl_per_token_group_quant_8bit`，并实现与 SGL per token group quantization 8bit 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 135-137: Comments and local context
````python


# For legacy usage
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 138-139: Constants and configuration
````python
sgl_per_token_group_quant_fp8 = sgl_per_token_group_quant_8bit
sgl_per_token_group_quant_int8 = sgl_per_token_group_quant_8bit
````
**EN:** This block defines shared constants or configuration values such as `sgl_per_token_group_quant_fp8`, `sgl_per_token_group_quant_int8`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `sgl_per_token_group_quant_fp8`, `sgl_per_token_group_quant_int8`），供后续函数或控制流程复用。

### Lines 142-147: `sgl_per_token_quant_fp8` definition
````python
def sgl_per_token_quant_fp8(
    input: torch.Tensor,
    output_q: torch.Tensor,
    output_s: torch.Tensor,
) -> None:
    torch.ops.sgl_kernel.sgl_per_token_quant_fp8.default(input, output_q, output_s)
````
**EN:** This section defines `sgl_per_token_quant_fp8` and implements the core logic associated with SGL per token quantization FP8. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `sgl_per_token_quant_fp8`，并实现与 SGL per token quantization FP8 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 150-169: `qserve_w4a8_per_chn_gemm` definition
````python
def qserve_w4a8_per_chn_gemm(
    in_feats: torch.Tensor,
    kernel: torch.Tensor,
    wscales: torch.Tensor,
    ascales: torch.Tensor,
    w_szs: torch.Tensor,
    a_ssums: torch.Tensor,
    out_feats: Optional[torch.Tensor] = None,
) -> torch.Tensor:
    if out_feats is None:
        # NOTE(HandH1998): qserve_w4a8_per_chn_gemm only supports out dtype=torch.float16 now
        out_feats = torch.empty(
            (in_feats.shape[0], kernel.shape[0]),
            device=in_feats.device,
            dtype=torch.float16,
        )
    torch.ops.sgl_kernel.qserve_w4a8_per_chn_gemm.default(
        in_feats, kernel, wscales, ascales, w_szs, a_ssums, out_feats
    )
    return out_feats
````
**EN:** This section defines `qserve_w4a8_per_chn_gemm` and implements the core logic associated with QServe w4a8 per channel GEMM. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `qserve_w4a8_per_chn_gemm`，并实现与 QServe w4a8 per channel GEMM 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 172-191: `qserve_w4a8_per_group_gemm` definition
````python
def qserve_w4a8_per_group_gemm(
    in_feats: torch.Tensor,
    kernel: torch.Tensor,
    zeros: torch.Tensor,
    scales_i8: torch.Tensor,
    wscales: torch.Tensor,
    ascales: torch.Tensor,
    out_feats: Optional[torch.Tensor] = None,
) -> torch.Tensor:
    if out_feats is None:
        # NOTE(HandH1998): qserve_w4a8_per_group_gemm only supports out dtype=torch.float16 now
        out_feats = torch.empty(
            (in_feats.shape[0], kernel.shape[0]),
            device=in_feats.device,
            dtype=torch.float16,
        )
    torch.ops.sgl_kernel.qserve_w4a8_per_group_gemm.default(
        in_feats, kernel, zeros, scales_i8, wscales, ascales, out_feats
    )
    return out_feats
````
**EN:** This section defines `qserve_w4a8_per_group_gemm` and implements the core logic associated with QServe w4a8 per group GEMM. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `qserve_w4a8_per_group_gemm`，并实现与 QServe w4a8 per group GEMM 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 194-210: `dsv3_router_gemm` definition
````python
def dsv3_router_gemm(
    hidden_states: torch.Tensor,
    router_weights: torch.Tensor,
    out_dtype: torch.dtype = torch.bfloat16,
) -> torch.Tensor:
    output = torch.empty(
        hidden_states.shape[0],
        router_weights.shape[0],
        device=hidden_states.device,
        dtype=out_dtype,
    )
    torch.ops.sgl_kernel.dsv3_router_gemm(
        output,
        hidden_states,
        router_weights,
    )
    return output
````
**EN:** This section defines `dsv3_router_gemm` and implements the core logic associated with DeepSeek-V3 router GEMM. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `dsv3_router_gemm`，并实现与 DeepSeek-V3 router GEMM 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 213-220: `shuffle_rows` definition
````python
def shuffle_rows(input_tensor, dst2src_map, output_tensor_shape):
    output_tensor = torch.empty(
        output_tensor_shape,
        device=input_tensor.device,
        dtype=input_tensor.dtype,
    )
    torch.ops.sgl_kernel.shuffle_rows.default(input_tensor, dst2src_map, output_tensor)
    return output_tensor
````
**EN:** This section defines `shuffle_rows` and implements the core logic associated with shuffle rows. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `shuffle_rows`，并实现与 shuffle rows 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 221-223: Comments and local context
````python


# GPTQ kernels
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 224-235: `gptq_gemm` definition
````python
def gptq_gemm(
    a: torch.Tensor,
    b_q_weight: torch.Tensor,
    b_gptq_qzeros: torch.Tensor,
    b_gptq_scales: torch.Tensor,
    b_g_idx: torch.Tensor,
    use_shuffle: bool,
    bit: int,
) -> torch.Tensor:
    return torch.ops.sgl_kernel.gptq_gemm(
        a, b_q_weight, b_gptq_qzeros, b_gptq_scales, b_g_idx, use_shuffle, bit
    )
````
**EN:** This section defines `gptq_gemm` and implements the core logic associated with GPTQ GEMM. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `gptq_gemm`，并实现与 GPTQ GEMM 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 238-239: `gptq_shuffle` definition
````python
def gptq_shuffle(q_weight: torch.Tensor, q_perm: torch.Tensor, bit: int) -> None:
    torch.torch.ops.sgl_kernel.gptq_shuffle(q_weight, q_perm, bit)
````
**EN:** This section defines `gptq_shuffle` and implements the core logic associated with GPTQ shuffle. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `gptq_shuffle`，并实现与 GPTQ shuffle 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

## Key Concepts / 关键概念
- **Role / 角色**: Python wrapper over compiled ops / 已编译算子的 Python 封装
- **Primary symbols / 主要符号**: `awq_dequantize`, `int8_scaled_mm`, `fp8_blockwise_scaled_mm`, `fp8_scaled_mm`, `_bmm_fp8_internal`, `bmm_fp8`, `dsv3_fused_a_gemm`, `sgl_per_token_group_quant_8bit`
- **Core technologies / 核心技术**: compiled custom ops / 已编译自定义算子, PyTorch tensors / PyTorch 张量

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel.utils`
- **External / 外部**: `torch`, `typing`
- **Runtime hooks / 运行时钩子**: `torch.ops.sgl_kernel` custom operator namespace
