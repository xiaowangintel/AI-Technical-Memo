# elementwise.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/python/sgl_kernel/elementwise.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module wraps native operators related to `elementwise`, performs Python-side validation, and exposes a convenient interface. / 该模块封装与 `elementwise` 相关的原生算子，执行 Python 侧校验，并暴露便捷接口。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Imports and module setup
````python
from typing import Optional

import torch
from sgl_kernel.utils import is_arch_support_pdl
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 6-11: Optional dependency handling
````python
try:
    import flashinfer.norm as _flashinfer_norm

    _has_flashinfer = True
except ImportError:
    _has_flashinfer = False
````
**EN:** This block attempts an operation that may fail, then provides fallback behavior so the module can keep working in reduced mode.
**CN:** 该部分尝试执行可能失败的操作，并提供回退行为，使模块能够以降级模式继续工作。

### Lines 13-13: Constants and configuration
````python
_FLASHINFER_NORM_SUPPORTED_DTYPES = {torch.float16, torch.bfloat16}
````
**EN:** This block defines shared constants or configuration values such as `_FLASHINFER_NORM_SUPPORTED_DTYPES`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `_FLASHINFER_NORM_SUPPORTED_DTYPES`），供后续函数或控制流程复用。

### Lines 16-28: `_rmsnorm_internal` definition
````python
def _rmsnorm_internal(
    input: torch.Tensor,
    weight: torch.Tensor,
    eps: float,
    out: Optional[torch.Tensor],
    enable_pdl: Optional[bool],
) -> torch.Tensor:
    if out is None:
        out = torch.empty_like(input)
    if enable_pdl is None:
        enable_pdl = is_arch_support_pdl()
    torch.ops.sgl_kernel.rmsnorm.default(out, input, weight, eps, enable_pdl)
    return out
````
**EN:** This section defines `_rmsnorm_internal` and implements the core logic associated with RMSNorm internal. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `_rmsnorm_internal`，并实现与 RMSNorm internal 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 31-42: `_fused_add_rmsnorm_internal` definition
````python
def _fused_add_rmsnorm_internal(
    input: torch.Tensor,
    residual: torch.Tensor,
    weight: torch.Tensor,
    eps: float,
    enable_pdl: Optional[bool],
) -> None:
    if enable_pdl is None:
        enable_pdl = is_arch_support_pdl()
    torch.ops.sgl_kernel.fused_add_rmsnorm.default(
        input, residual, weight, eps, enable_pdl
    )
````
**EN:** This section defines `_fused_add_rmsnorm_internal` and implements the core logic associated with fused add RMSNorm internal. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `_fused_add_rmsnorm_internal`，并实现与 fused add RMSNorm internal 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 45-57: `_gemma_rmsnorm_internal` definition
````python
def _gemma_rmsnorm_internal(
    input: torch.Tensor,
    weight: torch.Tensor,
    eps: float,
    out: Optional[torch.Tensor],
    enable_pdl: Optional[bool],
) -> torch.Tensor:
    if out is None:
        out = torch.empty_like(input)
    if enable_pdl is None:
        enable_pdl = is_arch_support_pdl()
    torch.ops.sgl_kernel.gemma_rmsnorm.default(out, input, weight, eps, enable_pdl)
    return out
````
**EN:** This section defines `_gemma_rmsnorm_internal` and implements the core logic associated with gemma RMSNorm internal. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `_gemma_rmsnorm_internal`，并实现与 gemma RMSNorm internal 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 60-71: `_gemma_fused_add_rmsnorm_internal` definition
````python
def _gemma_fused_add_rmsnorm_internal(
    input: torch.Tensor,
    residual: torch.Tensor,
    weight: torch.Tensor,
    eps: float,
    enable_pdl: Optional[bool],
) -> None:
    if enable_pdl is None:
        enable_pdl = is_arch_support_pdl()
    torch.ops.sgl_kernel.gemma_fused_add_rmsnorm.default(
        input, residual, weight, eps, enable_pdl
    )
````
**EN:** This section defines `_gemma_fused_add_rmsnorm_internal` and implements the core logic associated with gemma fused add RMSNorm internal. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `_gemma_fused_add_rmsnorm_internal`，并实现与 gemma fused add RMSNorm internal 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 72-75: Comments and local context
````python


# These implementations extensively draw from and build upon the FlashInfer project https://github.com/flashinfer-ai/flashinfer
# Kudos to @yzh119
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 76-122: `rmsnorm` definition
````python
def rmsnorm(
    input: torch.Tensor,
    weight: torch.Tensor,
    eps: float = 1e-6,
    out: Optional[torch.Tensor] = None,
    enable_pdl: Optional[bool] = None,
) -> torch.Tensor:
    r"""Root mean square normalization.

    ``out[i] = (input[i] / RMS(input)) * weight[i]``

    Parameters
    ----------
    input: torch.Tensor
        Input tensor, shape (batch_size, hidden_size).
    weight: torch.Tensor
        Weight tensor, shape (hidden_size,).
    eps: float
        Epsilon for numerical stability.
    out: Optional[torch.Tensor]
        The output tensor, if specified, the kernel will update this tensor inplace.
    enable_pdl: Optional[bool]
        Whether to enable `programmatic dependent launch
        <https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programmatic-dependent-launch-and-synchronization>`_
        If None, will be automatically enabled on Hopper architecture.

    Returns
    -------
    output: torch.Tensor
        Normalized tensor, shape (batch_size, hidden_size).
    """
    # torch.compiler.is_dynamo_compiling(): FlashInfer norm paths are not safe under
    # torch.compile(..., fullgraph=True). Dynamo traces into FlashInfer's JIT module
    # loading path, which calls Path.exists() / os.stat() — both untraceable — causing
    # the entire compilation to fail. We fall back to the internal implementation while
    # tracing as a temporary workaround. Once the upstream fix is merged and we upgrade
    # FlashInfer, this check can be removed.
    # See: https://github.com/flashinfer-ai/flashinfer/issues/2734
    #      https://github.com/flashinfer-ai/flashinfer/pull/2733
    if (
        _has_flashinfer
        and input.dtype in _FLASHINFER_NORM_SUPPORTED_DTYPES
        and not torch.compiler.is_dynamo_compiling()
    ):
        return _flashinfer_norm.rmsnorm(input, weight, eps, out, enable_pdl)
    else:
        return _rmsnorm_internal(input, weight, eps, out, enable_pdl)
````
**EN:** This section defines `rmsnorm` and implements the core logic associated with RMSNorm. Docstring summary: Root mean square normalization.
**CN:** 该部分定义 `rmsnorm`，并实现与 RMSNorm 相关的核心逻辑。 文档字符串摘要：Root mean square normalization.

### Lines 125-162: `fused_add_rmsnorm` definition
````python
def fused_add_rmsnorm(
    input: torch.Tensor,
    residual: torch.Tensor,
    weight: torch.Tensor,
    eps: float = 1e-6,
    enable_pdl: Optional[bool] = None,
) -> None:
    r"""Fused add root mean square normalization.

    Step 1:
    ``residual[i] += input[i]``

    Step 2:
    ``input[i] = (residual[i] / RMS(residual)) * weight[i]``

    Parameters
    ----------
    input: torch.Tensor
        Input tensor, shape (batch_size, hidden_size).
    residual: torch.Tensor
        Residual tensor, shape (batch_size, hidden_size).
    weight: torch.Tensor
        Weight tensor, shape (hidden_size,).
    eps: float
        Epsilon for numerical stability.
    enable_pdl: Optional[bool]
        Whether to enable `programmatic dependent launch
        <https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programmatic-dependent-launch-and-synchronization>`_
        If None, will be automatically enabled on Hopper architecture.
    """
    if (
        _has_flashinfer
        and input.dtype in _FLASHINFER_NORM_SUPPORTED_DTYPES
        and not torch.compiler.is_dynamo_compiling()
    ):
        _flashinfer_norm.fused_add_rmsnorm(input, residual, weight, eps, enable_pdl)
    else:
        _fused_add_rmsnorm_internal(input, residual, weight, eps, enable_pdl)
````
**EN:** This section defines `fused_add_rmsnorm` and implements the core logic associated with fused add RMSNorm. Docstring summary: Fused add root mean square normalization.
**CN:** 该部分定义 `fused_add_rmsnorm`，并实现与 fused add RMSNorm 相关的核心逻辑。 文档字符串摘要：Fused add root mean square normalization.

### Lines 165-203: `gemma_rmsnorm` definition
````python
def gemma_rmsnorm(
    input: torch.Tensor,
    weight: torch.Tensor,
    eps: float = 1e-6,
    out: Optional[torch.Tensor] = None,
    enable_pdl: Optional[bool] = None,
) -> torch.Tensor:
    r"""Gemma-style root mean square normalization.

    ``out[i] = (input[i] / RMS(input)) * (weight[i] + 1)``

    Parameters
    ----------
    input: torch.Tensor
        Input tensor, shape (batch_size, hidden_size).
    weight: torch.Tensor
        Weight tensor, shape (hidden_size,).
    eps: float
        Epsilon for numerical stability.
    out: Optional[torch.Tensor]
        The output tensor, if specified, the kernel will update this tensor inplace.
    enable_pdl: Optional[bool]
        Whether to enable `programmatic dependent launch
        <https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programmatic-dependent-launch-and-synchronization>`_
        If None, will be automatically enabled on Hopper architecture.

    Returns
    -------
    output: torch.Tensor
        Gemma Normalized tensor, shape (batch_size, hidden_size).
    """
    if (
        _has_flashinfer
        and input.dtype in _FLASHINFER_NORM_SUPPORTED_DTYPES
        and not torch.compiler.is_dynamo_compiling()
    ):
        return _flashinfer_norm.gemma_rmsnorm(input, weight, eps, out, enable_pdl)
    else:
        return _gemma_rmsnorm_internal(input, weight, eps, out, enable_pdl)
````
**EN:** This section defines `gemma_rmsnorm` and implements the core logic associated with gemma RMSNorm. Docstring summary: Gemma-style root mean square normalization.
**CN:** 该部分定义 `gemma_rmsnorm`，并实现与 gemma RMSNorm 相关的核心逻辑。 文档字符串摘要：Gemma-style root mean square normalization.

### Lines 206-245: `gemma_fused_add_rmsnorm` definition
````python
def gemma_fused_add_rmsnorm(
    input: torch.Tensor,
    residual: torch.Tensor,
    weight: torch.Tensor,
    eps: float = 1e-6,
    enable_pdl: Optional[bool] = None,
) -> None:
    r"""Gemma-style fused add root mean square normalization.

    Step 1:
    ``residual[i] += input[i]``

    Step 2:
    ``input[i] = (residual[i] / RMS(residual)) * (weight + 1)``

    Parameters
    ----------
    input: torch.Tensor
        Input tensor, shape (batch_size, hidden_size).
    residual: torch.Tensor
        Residual tensor, shape (batch_size, hidden_size).
    weight: torch.Tensor
        Weight tensor, shape (hidden_size,).
    eps: float
        Epsilon for numerical stability.
    enable_pdl: Optional[bool]
        Whether to enable `programmatic dependent launch
        <https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programmatic-dependent-launch-and-synchronization>`_
        If None, will be automatically enabled on Hopper architecture.
    """
    if (
        _has_flashinfer
        and input.dtype in _FLASHINFER_NORM_SUPPORTED_DTYPES
        and not torch.compiler.is_dynamo_compiling()
    ):
        _flashinfer_norm.gemma_fused_add_rmsnorm(
            input, residual, weight, eps, enable_pdl
        )
    else:
        _gemma_fused_add_rmsnorm_internal(input, residual, weight, eps, enable_pdl)
````
**EN:** This section defines `gemma_fused_add_rmsnorm` and implements the core logic associated with gemma fused add RMSNorm. Docstring summary: Gemma-style fused add root mean square normalization.
**CN:** 该部分定义 `gemma_fused_add_rmsnorm`，并实现与 gemma fused add RMSNorm 相关的核心逻辑。 文档字符串摘要：Gemma-style fused add root mean square normalization.

### Lines 248-255: `_check_shape` definition
````python
def _check_shape(input: torch.Tensor, output: torch.Tensor) -> None:
    assert input.ndim == output.ndim, f"{input.ndim} != {output.ndim}"
    assert (
        input.shape[:-1] == output.shape[:-1]
    ), f"{input.shape[:-1]} != {output.shape[:-1]}"
    assert (
        input.shape[-1] == 2 * output.shape[-1]
    ), f"{input.shape[-1]} != {2 * output.shape[-1]}"
````
**EN:** This section defines `_check_shape` and implements the core logic associated with check shape. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `_check_shape`，并实现与 check shape 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。

### Lines 258-270: `silu_and_mul` definition
````python
def silu_and_mul(input: torch.Tensor, out: torch.Tensor = None) -> torch.Tensor:
    if input.shape[-1] * input.dtype.itemsize % 16 != 0:
        raise ValueError("The pointers must be multiple of 16 bytes.")
    if out is not None:
        _check_shape(input, out)
    else:
        out = torch.empty(
            input.shape[:-1] + (input.shape[-1] // 2,),
            device=input.device,
            dtype=input.dtype,
        )
    torch.ops.sgl_kernel.silu_and_mul.default(out, input)
    return out
````
**EN:** This section defines `silu_and_mul` and implements the core logic associated with silu and multiply. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `silu_and_mul`，并实现与 silu and multiply 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 273-285: `gelu_tanh_and_mul` definition
````python
def gelu_tanh_and_mul(input: torch.Tensor, out: torch.Tensor = None) -> torch.Tensor:
    if input.shape[-1] * input.dtype.itemsize % 16 != 0:
        raise ValueError("The pointers must be multiple of 16 bytes.")
    if out is not None:
        _check_shape(input, out)
    else:
        out = torch.empty(
            input.shape[:-1] + (input.shape[-1] // 2,),
            device=input.device,
            dtype=input.dtype,
        )
    torch.ops.sgl_kernel.gelu_tanh_and_mul.default(out, input)
    return out
````
**EN:** This section defines `gelu_tanh_and_mul` and implements the core logic associated with gelu tanh and multiply. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `gelu_tanh_and_mul`，并实现与 gelu tanh and multiply 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 288-300: `gelu_and_mul` definition
````python
def gelu_and_mul(input: torch.Tensor, out: torch.Tensor = None) -> torch.Tensor:
    if input.shape[-1] * input.dtype.itemsize % 16 != 0:
        raise ValueError("The pointers must be multiple of 16 bytes.")
    if out is not None:
        _check_shape(input, out)
    else:
        out = torch.empty(
            input.shape[:-1] + (input.shape[-1] // 2,),
            device=input.device,
            dtype=input.dtype,
        )
    torch.ops.sgl_kernel.gelu_and_mul.default(out, input)
    return out
````
**EN:** This section defines `gelu_and_mul` and implements the core logic associated with gelu and multiply. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `gelu_and_mul`，并实现与 gelu and multiply 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 303-324: Conditional logic
````python
if torch.version.hip is not None:

    def gelu_quick(input: torch.Tensor, out: torch.Tensor = None) -> torch.Tensor:
        """
        Quick-GELU:  y = x * sigmoid(1.702 * x)

        The CUDA/HIP kernel uses 128-bit (16-byte) vector loads & stores,
        so the last-dimension byte length must be a multiple of 16 bytes.
        """
        if input.shape[-1] * input.dtype.itemsize % 16 != 0:
            raise ValueError(
                f"The last dimension ({input.shape[-1]}) x itemsize "
                f"({input.dtype.itemsize}) must be a multiple of 16 bytes."
            )

        if out is not None:
            assert input.shape == out.shape, f"{input.shape} != {out.shape}"
        else:
            out = torch.empty_like(input)

        torch.ops.sgl_kernel.gelu_quick(out, input)
        return out
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 327-337: `rotary_embedding` definition
````python
def rotary_embedding(
    positions: torch.Tensor,
    query: torch.Tensor,
    key: torch.Tensor,
    head_size: int,
    cos_sin_cache: torch.Tensor,
    is_neox: bool = True,
):
    torch.ops.sgl_kernel.rotary_embedding.default(
        positions, query, key, head_size, cos_sin_cache, is_neox
    )
````
**EN:** This section defines `rotary_embedding` and implements the core logic associated with rotary embedding. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `rotary_embedding`，并实现与 rotary embedding 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 340-341: `copy_to_gpu_no_ce` definition
````python
def copy_to_gpu_no_ce(input: torch.Tensor, output: torch.Tensor):
    torch.ops.sgl_kernel.copy_to_gpu_no_ce(input, output)
````
**EN:** This section defines `copy_to_gpu_no_ce` and implements the core logic associated with copy to gpu no ce. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `copy_to_gpu_no_ce`，并实现与 copy to gpu no ce 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 344-349: `concat_mla_k` definition
````python
def concat_mla_k(
    k: torch.Tensor,
    k_nope: torch.Tensor,
    k_rope: torch.Tensor,
):
    torch.ops.sgl_kernel.concat_mla_k(k, k_nope, k_rope)
````
**EN:** This section defines `concat_mla_k` and implements the core logic associated with concat MLA k. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `concat_mla_k`，并实现与 concat MLA k 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 352-361: `concat_mla_absorb_q` definition
````python
def concat_mla_absorb_q(
    a: torch.Tensor,
    b: torch.Tensor,
):
    *batch_dims, _ = a.shape
    out = torch.empty(
        (*batch_dims, a.shape[-1] + b.shape[-1]), device=a.device, dtype=a.dtype
    )
    torch.ops.sgl_kernel.concat_mla_absorb_q(a, b, out)
    return out
````
**EN:** This section defines `concat_mla_absorb_q` and implements the core logic associated with concat MLA absorb q. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `concat_mla_absorb_q`，并实现与 concat MLA absorb q 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

## Key Concepts / 关键概念
- **Role / 角色**: Python wrapper over compiled ops / 已编译算子的 Python 封装
- **Primary symbols / 主要符号**: `_rmsnorm_internal`, `_fused_add_rmsnorm_internal`, `_gemma_rmsnorm_internal`, `_gemma_fused_add_rmsnorm_internal`, `rmsnorm`, `fused_add_rmsnorm`, `gemma_rmsnorm`, `gemma_fused_add_rmsnorm`
- **Core technologies / 核心技术**: compiled custom ops / 已编译自定义算子, PyTorch tensors / PyTorch 张量

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel.utils`
- **External / 外部**: `flashinfer.norm`, `torch`, `typing`
- **Runtime hooks / 运行时钩子**: `torch.ops.sgl_kernel` custom operator namespace
