# elementwise.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/elementwise.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements runtime layer support for the SGLang SRT runtime. It exposes symbols such as `fused_softcap_kernel`, `fused_softcap`, `Softcap`, and `fused_dual_residual_rmsnorm_kernel` and connects them to backend-specific paths such as `CUDA`, `Triton`, `FlashInfer`, and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了运行时层支持。它提供了 `fused_softcap_kernel`、`fused_softcap`、`Softcap` 以及 `fused_dual_residual_rmsnorm_kernel` 等符号，并把这些符号连接到 `CUDA`、`Triton`、`FlashInfer` 以及 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-36: Imports, constants, and runtime setup
```python
from typing import Optional, Tuple

import torch
import triton
import triton.language as tl

from sglang.srt.utils import is_hip
from sglang.srt.utils.custom_op import register_custom_op

_is_hip = is_hip()


fused_softcap_autotune = triton.autotune(
    configs=[
        triton.Config(kwargs={"BLOCK_SIZE": 128}, num_warps=4),
        triton.Config(kwargs={"BLOCK_SIZE": 128}, num_warps=8),
        triton.Config(kwargs={"BLOCK_SIZE": 128}, num_warps=16),
        triton.Config(kwargs={"BLOCK_SIZE": 256}, num_warps=4),
        triton.Config(kwargs={"BLOCK_SIZE": 256}, num_warps=8),
        triton.Config(kwargs={"BLOCK_SIZE": 512}, num_warps=4),
        triton.Config(kwargs={"BLOCK_SIZE": 512}, num_warps=8),
        triton.Config(kwargs={"BLOCK_SIZE": 512}, num_warps=16),
        triton.Config(kwargs={"BLOCK_SIZE": 1024}, num_warps=4),
        triton.Config(kwargs={"BLOCK_SIZE": 1024}, num_warps=8),
        triton.Config(kwargs={"BLOCK_SIZE": 1024}, num_warps=16),
        triton.Config(kwargs={"BLOCK_SIZE": 1024}, num_warps=32),
        triton.Config(kwargs={"BLOCK_SIZE": 2048}, num_warps=32),
        triton.Config(kwargs={"BLOCK_SIZE": 4096}, num_warps=32),
        triton.Config(kwargs={"BLOCK_SIZE": 8192}, num_warps=32),
        triton.Config(kwargs={"BLOCK_SIZE": 16384}, num_warps=32),
        triton.Config(kwargs={"BLOCK_SIZE": 32768}, num_warps=32),
    ],
    key=["n_ele"],
)
```
**EN:** This section prepares the module namespace. It imports `typing.Optional`, `typing.Tuple`, `torch`, `triton`, `triton.language`, and `sglang.srt.utils.is_hip`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_hip` and `fused_softcap_autotune` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `typing.Optional`、`typing.Tuple`、`torch`、`triton`、`triton.language` 以及 `sglang.srt.utils.is_hip`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_hip` 和 `fused_softcap_autotune` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 37-58: Function `fused_softcap_kernel` and its core logic
```python
@triton.jit
def fused_softcap_kernel(
    output_ptr,
    input_ptr,
    n_ele,
    softcap_const: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
):
    pid = tl.program_id(axis=0)
    block_start = pid * BLOCK_SIZE
    offsets = block_start + tl.arange(0, BLOCK_SIZE)
    mask = offsets < n_ele
    x = tl.load(input_ptr + offsets, mask=mask)
    fx = x.to(tl.float32)
    fxs = fx / softcap_const
    exped = tl.exp(2 * fxs)
    top = exped - 1
    bottom = exped + 1
    output = top / bottom * softcap_const
    tl.store(output_ptr + offsets, output, mask=mask)
```
**EN:** This block defines `fused_softcap_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.load`, `x.to`, `tl.exp`, and `tl.store`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `block_start`, `offsets`, `mask`, and `x` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `fused_softcap_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.load`、`x.to`、`tl.exp` 以及 `tl.store`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`block_start`、`offsets`、`mask` 以及 `x` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 59-61: Module constants and shared configuration
```python
fused_softcap_kernel_autotuned = fused_softcap_autotune(fused_softcap_kernel)
```
**EN:** This section prepares the module namespace. Shared names such as `fused_softcap_kernel_autotuned` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 像 `fused_softcap_kernel_autotuned` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 62-75: Function `fused_softcap` and its core logic
```python
def fused_softcap(x, softcap_const, autotune=False):
    output = torch.empty_like(x, dtype=torch.float32)
    n_elements = output.numel()
    if autotune:
        grid = lambda meta: (triton.cdiv(n_elements, meta["BLOCK_SIZE"]),)
        fused_softcap_kernel_autotuned[grid](output, x, n_elements, softcap_const)
    else:
        fused_softcap_kernel[(triton.cdiv(n_elements, 128),)](
            output, x, n_elements, softcap_const, BLOCK_SIZE=128, num_warps=8
        )
    return output


# cast to float + softcap
```
**EN:** This block defines `fused_softcap` and contains the main logic for this step. It mainly invokes `torch.empty_like`, `output.numel`, `fused_softcap_kernel_autotuned`, `fused_softcap_kernel`, and `triton.cdiv`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `output`, `n_elements`, and `grid` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_softcap`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty_like`、`output.numel`、`fused_softcap_kernel_autotuned`、`fused_softcap_kernel` 以及 `triton.cdiv`，说明该流程会编排底层辅助函数或计算内核。 像 `output`、`n_elements` 以及 `grid` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 76-76: Class `Softcap` declaration and shared state
```python
class Softcap:
```
**EN:** This block introduces class `Softcap` and the state shared by its methods.
**CN:** 该代码块引入类 `Softcap`，并定义其方法共享的状态。

### Lines 77-79: `Softcap` initialization and state setup
```python
    def __init__(self, softcap_const: float):
        self.softcap_const = softcap_const
```
**EN:** This block defines `Softcap.__init__` and contains the main logic for this step. Intermediate names such as `self.softcap_const` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `Softcap.__init__`，并承载这一阶段的核心逻辑。 像 `self.softcap_const` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 80-82: `Softcap.__call__` callable execution entry
```python
    def __call__(self, *args, **kwargs):
        return self.forward(*args, **kwargs)
```
**EN:** This block defines `Softcap.__call__` and contains the main logic for this step. It mainly invokes `self.forward`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Softcap.__call__`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 83-88: `Softcap.forward` main forward path
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        if x.is_cuda:
            return self.forward_cuda(x)
        else:
            return self.forward_native(x)
```
**EN:** This block defines `Softcap.forward` and contains the main logic for this step. It mainly invokes `self.forward_cuda` and `self.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Softcap.forward`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_cuda` 和 `self.forward_native`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 89-91: `Softcap.forward_native` NATIVE execution path
```python
    def forward_native(self, x: torch.Tensor) -> torch.Tensor:
        return torch.tanh(x.float() / self.softcap_const) * self.softcap_const
```
**EN:** This block defines `Softcap.forward_native` and contains the main logic for this step. It mainly invokes `torch.tanh` and `x.float`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Softcap.forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `torch.tanh` 和 `x.float`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 92-95: `Softcap.forward_cuda` CUDA execution path
```python
    def forward_cuda(self, x: torch.Tensor, autotune=False) -> torch.Tensor:
        return fused_softcap(x, self.softcap_const, autotune=autotune)
```
**EN:** This block defines `Softcap.forward_cuda` and contains the main logic for this step. It mainly invokes `fused_softcap`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Softcap.forward_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `fused_softcap`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 96-137: Module constants and shared configuration
```python
rmsnorm_autotune = triton.autotune(
    configs=[
        triton.Config(kwargs={"BLOCK_SIZE": 1024}, num_warps=4, num_stages=1),
        triton.Config(kwargs={"BLOCK_SIZE": 1024}, num_warps=8, num_stages=1),
        triton.Config(kwargs={"BLOCK_SIZE": 1024}, num_warps=16, num_stages=1),
        triton.Config(kwargs={"BLOCK_SIZE": 1024}, num_warps=4),
        triton.Config(kwargs={"BLOCK_SIZE": 1024}, num_warps=8),
        triton.Config(kwargs={"BLOCK_SIZE": 1024}, num_warps=16),
        triton.Config(kwargs={"BLOCK_SIZE": 1024}, num_warps=4, num_stages=4),
        triton.Config(kwargs={"BLOCK_SIZE": 1024}, num_warps=8, num_stages=4),
        triton.Config(kwargs={"BLOCK_SIZE": 1024}, num_warps=16, num_stages=4),
        triton.Config(kwargs={"BLOCK_SIZE": 1024}, num_warps=8, num_stages=8),
        triton.Config(kwargs={"BLOCK_SIZE": 1024}, num_warps=16, num_stages=8),
        triton.Config(kwargs={"BLOCK_SIZE": 2048}, num_warps=8),
        triton.Config(kwargs={"BLOCK_SIZE": 2048}, num_warps=16),
        triton.Config(kwargs={"BLOCK_SIZE": 2048}, num_warps=8, num_stages=4),
        triton.Config(kwargs={"BLOCK_SIZE": 2048}, num_warps=16, num_stages=4),
        triton.Config(kwargs={"BLOCK_SIZE": 4096}, num_warps=8),
        triton.Config(kwargs={"BLOCK_SIZE": 4096}, num_warps=16),
        triton.Config(kwargs={"BLOCK_SIZE": 8192}, num_warps=8),
        triton.Config(kwargs={"BLOCK_SIZE": 8192}, num_warps=16),
        triton.Config(kwargs={"BLOCK_SIZE": 8192}, num_warps=32),
        triton.Config(kwargs={"BLOCK_SIZE": 8192}, num_warps=8, num_stages=1),
        triton.Config(kwargs={"BLOCK_SIZE": 8192}, num_warps=16, num_stages=1),
        triton.Config(kwargs={"BLOCK_SIZE": 8192}, num_warps=32, num_stages=1),
        triton.Config(kwargs={"BLOCK_SIZE": 8192}, num_warps=8, num_stages=4),
        triton.Config(kwargs={"BLOCK_SIZE": 8192}, num_warps=16, num_stages=4),
        triton.Config(kwargs={"BLOCK_SIZE": 8192}, num_warps=32, num_stages=4),
        triton.Config(kwargs={"BLOCK_SIZE": 16384}, num_warps=8),
        triton.Config(kwargs={"BLOCK_SIZE": 16384}, num_warps=16),
        triton.Config(kwargs={"BLOCK_SIZE": 16384}, num_warps=32),
        triton.Config(kwargs={"BLOCK_SIZE": 16384}, num_warps=8, num_stages=1),
        triton.Config(kwargs={"BLOCK_SIZE": 16384}, num_warps=16, num_stages=1),
        triton.Config(kwargs={"BLOCK_SIZE": 16384}, num_warps=32, num_stages=1),
        triton.Config(kwargs={"BLOCK_SIZE": 16384}, num_warps=8, num_stages=4),
        triton.Config(kwargs={"BLOCK_SIZE": 16384}, num_warps=16, num_stages=4),
        triton.Config(kwargs={"BLOCK_SIZE": 16384}, num_warps=32, num_stages=4),
    ],
    key=["hidden_dim"],
)
```
**EN:** This section prepares the module namespace. Shared names such as `rmsnorm_autotune` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 像 `rmsnorm_autotune` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 138-183: Function `fused_dual_residual_rmsnorm_kernel` and its core logic
```python
@triton.jit
def fused_dual_residual_rmsnorm_kernel(
    output_ptr,
    mid_ptr,
    activ_ptr,
    residual_ptr,
    weight1_ptr,
    weight2_ptr,
    eps: tl.constexpr,
    hidden_dim: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
):
    pid = tl.program_id(axis=0)
    input_start = pid * hidden_dim

    offsets = tl.arange(0, BLOCK_SIZE)
    mask = offsets < hidden_dim

    a_ = tl.load(activ_ptr + input_start + offsets, mask=mask, other=0.0)
    a = a_.to(tl.float32)
    rms = tl.sqrt(tl.sum(a * a, axis=0) / hidden_dim + eps)

    r = tl.load(residual_ptr + input_start + offsets, mask=mask, other=0.0)
    w1_ = tl.load(weight1_ptr + offsets, mask=mask, other=0.0)
    w1 = w1_.to(tl.float32)

    a2r = r + (a / rms * w1).to(r.dtype)
    tl.store(
        mid_ptr + input_start + offsets,
        a2r,
        mask=mask,
    )

    a2r = a2r.to(tl.float32)
    rms2 = tl.sqrt(tl.sum(a2r * a2r, axis=0) / hidden_dim + eps)

    w2_ = tl.load(weight2_ptr + offsets, mask=mask, other=0.0)
    w2 = w2_.to(tl.float32)

    tl.store(
        output_ptr + input_start + offsets,
        a2r / rms2 * w2,  # implicitly casts to output dtype here
        mask=mask,
    )
```
**EN:** This block defines `fused_dual_residual_rmsnorm_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.arange`, `tl.load`, `a_.to`, and `tl.sqrt`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `input_start`, `offsets`, `mask`, and `a_` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `fused_dual_residual_rmsnorm_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.arange`、`tl.load`、`a_.to` 以及 `tl.sqrt`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`input_start`、`offsets`、`mask` 以及 `a_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 184-188: Module constants and shared configuration
```python
fused_dual_residual_rmsnorm_kernel_autotune = rmsnorm_autotune(
    fused_dual_residual_rmsnorm_kernel
)
```
**EN:** This section prepares the module namespace. Shared names such as `fused_dual_residual_rmsnorm_kernel_autotune` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 像 `fused_dual_residual_rmsnorm_kernel_autotune` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 189-223: Function `fused_dual_residual_rmsnorm` and its core logic
```python
def fused_dual_residual_rmsnorm(x, residual, weight1, weight2, eps, autotune=False):
    assert len(x.shape) == 2
    assert (
        x.shape == residual.shape and x.dtype == residual.dtype
    ), f"{x.shape=} {residual.shape=} {x.dtype=} {residual.dtype=}"
    output, mid = torch.empty_like(x), torch.empty_like(x)
    bs, hidden_dim = x.shape
    if autotune:
        fused_dual_residual_rmsnorm_kernel_autotune[(bs,)](
            output, mid, x, residual, weight1, weight2, eps=eps, hidden_dim=hidden_dim
        )
    else:
        max_warps = 16 if _is_hip else 32
        config = {
            "BLOCK_SIZE": triton.next_power_of_2(hidden_dim),
            "num_warps": max(
                min(triton.next_power_of_2(triton.cdiv(hidden_dim, 256)), max_warps), 4
            ),
        }

        fused_dual_residual_rmsnorm_kernel[(bs,)](
            output,
            mid,
            x,
            residual,
            weight1,
            weight2,
            eps=eps,
            hidden_dim=hidden_dim,
            **config,
        )

    return output, mid
```
**EN:** This block defines `fused_dual_residual_rmsnorm` and contains the main logic for this step. It mainly invokes `len`, `torch.empty_like`, `fused_dual_residual_rmsnorm_kernel_autotune`, `fused_dual_residual_rmsnorm_kernel`, and `triton.next_power_of_2`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `output`, `mid`, `bs`, `hidden_dim`, and `max_warps` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_dual_residual_rmsnorm`，并承载这一阶段的核心逻辑。 它主要调用 `len`、`torch.empty_like`、`fused_dual_residual_rmsnorm_kernel_autotune`、`fused_dual_residual_rmsnorm_kernel` 以及 `triton.next_power_of_2`，说明该流程会编排底层辅助函数或计算内核。 像 `output`、`mid`、`bs`、`hidden_dim` 以及 `max_warps` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 224-254: Function `fused_rmsnorm_kernel` and its core logic
```python
@triton.jit
def fused_rmsnorm_kernel(
    output_ptr,
    activ_ptr,
    weight_ptr,
    eps: tl.constexpr,
    hidden_dim: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
):
    pid = tl.program_id(axis=0).to(tl.int64)
    input_start = pid * hidden_dim

    offsets = tl.arange(0, BLOCK_SIZE)
    mask = offsets < hidden_dim

    a_ = tl.load(activ_ptr + input_start + offsets, mask=mask, other=0.0)
    a = a_.to(tl.float32)
    rms = tl.sqrt(tl.sum(a * a, axis=0) / hidden_dim + eps)

    w1_ = tl.load(weight_ptr + offsets, mask=mask, other=0.0)
    w1 = w1_.to(tl.float32)

    a_rms = a / rms * w1

    tl.store(
        output_ptr + input_start + offsets,
        a_rms,  # implicitly casts to output dtype here
        mask=mask,
    )
```
**EN:** This block defines `fused_rmsnorm_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id.to`, `tl.arange`, `tl.load`, `a_.to`, and `tl.sqrt`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `input_start`, `offsets`, `mask`, and `a_` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `fused_rmsnorm_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id.to`、`tl.arange`、`tl.load`、`a_.to` 以及 `tl.sqrt`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`input_start`、`offsets`、`mask` 以及 `a_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 255-275: Function `fused_rmsnorm` and its core logic
```python
def fused_rmsnorm(x, weight, eps, autotune=False, inplace=False):
    assert len(x.shape) == 2
    if inplace:
        output = x
    else:
        output = torch.empty_like(x)
    bs, hidden_dim = x.shape
    max_warps = 16 if _is_hip else 32
    config = {
        "BLOCK_SIZE": triton.next_power_of_2(hidden_dim),
        "num_warps": max(
            min(triton.next_power_of_2(triton.cdiv(hidden_dim, 256)), max_warps), 4
        ),
    }

    fused_rmsnorm_kernel[(bs,)](
        output, x, weight, eps=eps, hidden_dim=hidden_dim, **config
    )
    return output
```
**EN:** This block defines `fused_rmsnorm` and contains the main logic for this step. It mainly invokes `fused_rmsnorm_kernel`, `len`, `torch.empty_like`, `triton.next_power_of_2`, and `max`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `bs`, `hidden_dim`, `max_warps`, `config`, and `output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_rmsnorm`，并承载这一阶段的核心逻辑。 它主要调用 `fused_rmsnorm_kernel`、`len`、`torch.empty_like`、`triton.next_power_of_2` 以及 `max`，说明该流程会编排底层辅助函数或计算内核。 像 `bs`、`hidden_dim`、`max_warps`、`config` 以及 `output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 276-281: Class `FusedDualResidualRMSNorm` declaration and shared state
```python
class FusedDualResidualRMSNorm:
    """
    Fused implementation of
    y = RMSNorm2(RMSNorm1(x) + residual))
    """
```
**EN:** This block introduces class `FusedDualResidualRMSNorm` and the state shared by its methods. The class docstring summarizes its role: Fused implementation of y = RMSNorm2(RMSNorm1(x) + residual))
**CN:** 该代码块引入类 `FusedDualResidualRMSNorm`，并定义其方法共享的状态。 类级文档进一步概括了它的职责。

### Lines 282-288: `FusedDualResidualRMSNorm` initialization and state setup
```python
    def __init__(self, rmsnorm1, rmsnorm2) -> None:  # the one after rmsnorm1
        self.rmsnorm1 = rmsnorm1
        self.rmsnorm2 = rmsnorm2
        self.variance_epsilon = self.rmsnorm1.variance_epsilon
        assert self.rmsnorm1.variance_epsilon == self.rmsnorm2.variance_epsilon
        assert self.rmsnorm1.weight.shape == self.rmsnorm2.weight.shape
```
**EN:** This block defines `FusedDualResidualRMSNorm.__init__` and contains the main logic for this step. Intermediate names such as `self.rmsnorm1`, `self.rmsnorm2`, and `self.variance_epsilon` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `FusedDualResidualRMSNorm.__init__`，并承载这一阶段的核心逻辑。 像 `self.rmsnorm1`、`self.rmsnorm2` 以及 `self.variance_epsilon` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 289-291: `FusedDualResidualRMSNorm.__call__` callable execution entry
```python
    def __call__(self, *args, **kwargs):
        return self.forward(*args, **kwargs)
```
**EN:** This block defines `FusedDualResidualRMSNorm.__call__` and contains the main logic for this step. It mainly invokes `self.forward`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FusedDualResidualRMSNorm.__call__`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 292-299: `FusedDualResidualRMSNorm.forward` main forward path
```python
    def forward(
        self, x: torch.Tensor, residual: torch.Tensor
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        if x.is_cuda:
            return self.forward_cuda(x, residual)
        else:
            return self.forward_flashinfer(x, residual)
```
**EN:** This block defines `FusedDualResidualRMSNorm.forward` and contains the main logic for this step. It mainly invokes `self.forward_cuda` and `self.forward_flashinfer`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FusedDualResidualRMSNorm.forward`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_cuda` 和 `self.forward_flashinfer`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 300-311: `FusedDualResidualRMSNorm.forward_cuda` CUDA execution path
```python
    def forward_cuda(
        self, x: torch.Tensor, residual: torch.Tensor, autotune=False
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        return fused_dual_residual_rmsnorm(
            x,
            residual,
            self.rmsnorm1.weight,
            self.rmsnorm2.weight,
            self.variance_epsilon,
            autotune=autotune,
        )
```
**EN:** This block defines `FusedDualResidualRMSNorm.forward_cuda` and contains the main logic for this step. It mainly invokes `fused_dual_residual_rmsnorm`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FusedDualResidualRMSNorm.forward_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `fused_dual_residual_rmsnorm`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 312-320: `FusedDualResidualRMSNorm.forward_flashinfer` FLASHINFER execution path
```python
    def forward_flashinfer(
        self,
        x: torch.Tensor,
        residual: torch.Tensor,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        normed1 = self.rmsnorm1(x)
        residual = normed1 + residual
        return self.rmsnorm2(residual), residual
```
**EN:** This block defines `FusedDualResidualRMSNorm.forward_flashinfer` and contains the main logic for this step. It mainly invokes `self.rmsnorm1` and `self.rmsnorm2`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `normed1` and `residual` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FusedDualResidualRMSNorm.forward_flashinfer`，并承载这一阶段的核心逻辑。 它主要调用 `self.rmsnorm1` 和 `self.rmsnorm2`，说明该流程会编排底层辅助函数或计算内核。 像 `normed1` 和 `residual` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 321-330: `FusedDualResidualRMSNorm.forward_native` NATIVE execution path
```python
    def forward_native(
        self,
        x: torch.Tensor,
        residual: torch.Tensor,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        normed1 = self.rmsnorm1.forward_native(x)
        residual = normed1 + residual
        return self.rmsnorm2.forward_native(residual), residual
```
**EN:** This block defines `FusedDualResidualRMSNorm.forward_native` and contains the main logic for this step. It mainly invokes `self.rmsnorm1.forward_native` and `self.rmsnorm2.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `normed1` and `residual` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FusedDualResidualRMSNorm.forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `self.rmsnorm1.forward_native` 和 `self.rmsnorm2.forward_native`，说明该流程会编排底层辅助函数或计算内核。 像 `normed1` 和 `residual` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 331-361: Function `experts_combine_kernel` and its core logic
```python
@triton.jit
def experts_combine_kernel(
    out_hidden_states,
    moe_hidden_states,
    mlp_hidden_states,
    combine_k: tl.constexpr,
    hidden_dim: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
):
    pid = tl.program_id(0)
    start_index_mlp = pid * hidden_dim
    start_index_rmoe = pid * hidden_dim * combine_k
    offsets = tl.arange(0, BLOCK_SIZE)
    mask = offsets < hidden_dim
    combine_k_offsets = tl.arange(0, combine_k)

    moe_x = tl.load(
        moe_hidden_states
        + start_index_rmoe
        + combine_k_offsets[:, None] * hidden_dim
        + offsets[None, :],
        mask=mask[None, :],
        other=0.0,
    )
    moe_x = tl.sum(moe_x, axis=0)
    mlp_x = tl.load(mlp_hidden_states + start_index_mlp + offsets, mask=mask, other=0.0)
    combined_x = (moe_x + mlp_x) / 1.4142135623730951

    tl.store(out_hidden_states + start_index_mlp + offsets, combined_x, mask=mask)
```
**EN:** This block defines `experts_combine_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.arange`, `tl.load`, `tl.sum`, and `tl.store`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `start_index_mlp`, `start_index_rmoe`, `offsets`, and `mask` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `experts_combine_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.arange`、`tl.load`、`tl.sum` 以及 `tl.store`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`start_index_mlp`、`start_index_rmoe`、`offsets` 以及 `mask` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 362-406: Function `experts_combine_triton` and its core logic
```python
@register_custom_op(out_shape="mlp_hidden_states")
def experts_combine_triton(
    moe_hidden_states: torch.Tensor,
    mlp_hidden_states: torch.Tensor,
    output_buffer: Optional[torch.Tensor] = None,
) -> torch.Tensor:
    assert moe_hidden_states.is_contiguous()
    assert mlp_hidden_states.is_contiguous()

    if len(moe_hidden_states.shape) == 2:
        combine_k = 1  # pre-combined
    else:
        combine_k = moe_hidden_states.shape[1]

    if output_buffer is None:
        out_hidden_states = torch.empty_like(mlp_hidden_states)
    else:
        flat_output_buffer = output_buffer.view(mlp_hidden_states.dtype).reshape(-1)
        assert flat_output_buffer.numel() >= mlp_hidden_states.numel()
        out_hidden_states = flat_output_buffer[: mlp_hidden_states.numel()].reshape(
            mlp_hidden_states.shape
        )

    bs, hidden_dim = mlp_hidden_states.shape

    config = {
        "BLOCK_SIZE": triton.next_power_of_2(hidden_dim),
        "num_warps": max(
            min(triton.next_power_of_2(triton.cdiv(hidden_dim, 1024)), 8), 4
        ),
    }

    experts_combine_kernel[(bs,)](
        out_hidden_states,
        moe_hidden_states,
        mlp_hidden_states,
        combine_k,
        hidden_dim,
        **config,
    )

    return out_hidden_states


# gelu on first half of vector
```
**EN:** This block defines `experts_combine_triton` and contains the main logic for this step. Decorators like `register_custom_op` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_custom_op`, `moe_hidden_states.is_contiguous`, `mlp_hidden_states.is_contiguous`, `experts_combine_kernel`, and `len`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `bs`, `hidden_dim`, `config`, `combine_k`, and `out_hidden_states` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `experts_combine_triton`，并承载这一阶段的核心逻辑。 像 `register_custom_op` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_custom_op`、`moe_hidden_states.is_contiguous`、`mlp_hidden_states.is_contiguous`、`experts_combine_kernel` 以及 `len`，说明该流程会编排底层辅助函数或计算内核。 像 `bs`、`hidden_dim`、`config`、`combine_k` 以及 `out_hidden_states` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 407-444: Function `gelu_and_mul_kernel` and its core logic
```python
@triton.jit
def gelu_and_mul_kernel(
    out_hidden_states_ptr,  # (bs, hidden_dim)
    out_scales_ptr,  # (bs,)
    hidden_states_ptr,  # (bs, hidden_dim * 2)
    quant_max: tl.constexpr,
    static_scale: tl.constexpr,
    hidden_dim: tl.constexpr,  # the output hidden_dim
    BLOCK_SIZE: tl.constexpr,
):
    pid = tl.program_id(axis=0)

    input_start = pid * hidden_dim * 2
    output_start = pid * hidden_dim

    input1_offs = tl.arange(0, BLOCK_SIZE)
    mask = tl.arange(0, BLOCK_SIZE) < hidden_dim  # shared for input1, input3, output
    input3_offs = hidden_dim + tl.arange(0, BLOCK_SIZE)
    output_offs = tl.arange(0, BLOCK_SIZE)

    x1 = tl.load(
        hidden_states_ptr + input_start + input1_offs, mask=mask, other=0.0
    ).to(tl.float32)
    x3 = tl.load(
        hidden_states_ptr + input_start + input3_offs, mask=mask, other=0.0
    ).to(tl.float32)

    # gelu
    # cast down before mul to better match training?
    gelu_x1 = 0.5 * (1.0 + tl.erf(x1 * 0.7071067811865475)) * x1
    out = x3 * gelu_x1.to(hidden_states_ptr.dtype.element_ty)

    if quant_max is not None:
        raise NotImplementedError()

    tl.store(out_hidden_states_ptr + output_start + output_offs, out, mask=mask)
```
**EN:** This block defines `gelu_and_mul_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.arange`, `tl.load.to`, `tl.store`, and `gelu_x1.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `input_start`, `output_start`, `input1_offs`, and `mask` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `gelu_and_mul_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.arange`、`tl.load.to`、`tl.store` 以及 `gelu_x1.to`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`input_start`、`output_start`、`input1_offs` 以及 `mask` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 445-500: Function `gelu_and_mul_triton` and its core logic
```python
def gelu_and_mul_triton(
    hidden_states,
    scales=None,
    quantize=None,  # dtype to quantize to
    out=None,
):
    bs, in_hidden_dim = hidden_states.shape
    hidden_dim = in_hidden_dim // 2

    if out is None:
        out_hidden_states = torch.empty(
            (bs, hidden_dim),
            dtype=quantize or hidden_states.dtype,
            device=hidden_states.device,
        )
    else:
        assert out.shape == (bs, hidden_dim)
        assert out.dtype == (quantize or hidden_states.dtype)
        out_hidden_states = out
    out_scales = None
    static_scale = False
    if quantize is not None:
        if scales is None:
            out_scales = torch.empty(
                (bs,), dtype=torch.float32, device=hidden_states.device
            )
        else:
            out_scales = scales
            static_scale = True

    max_warps = 16 if _is_hip else 32
    config = {
        # 8 ele per thread (not tuned)
        "num_warps": max(
            min(triton.next_power_of_2(triton.cdiv(hidden_dim, 8 * 32)), max_warps), 4
        ),
    }

    gelu_and_mul_kernel[(bs,)](
        out_hidden_states,
        out_scales,
        hidden_states,
        quant_max=torch.finfo(quantize).max if quantize is not None else None,
        static_scale=static_scale,
        hidden_dim=hidden_dim,
        BLOCK_SIZE=triton.next_power_of_2(hidden_dim),
        **config,
    )

    if quantize is not None:
        return out_hidden_states, out_scales
    else:
        return out_hidden_states, None


# silu on first half of vector
```
**EN:** This block defines `gelu_and_mul_triton` and contains the main logic for this step. It mainly invokes `gelu_and_mul_kernel`, `torch.empty`, `max`, `min`, and `triton.next_power_of_2`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `bs`, `in_hidden_dim`, `hidden_dim`, `out_scales`, and `static_scale` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `gelu_and_mul_triton`，并承载这一阶段的核心逻辑。 它主要调用 `gelu_and_mul_kernel`、`torch.empty`、`max`、`min` 以及 `triton.next_power_of_2`，说明该流程会编排底层辅助函数或计算内核。 像 `bs`、`in_hidden_dim`、`hidden_dim`、`out_scales` 以及 `static_scale` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 501-538: Function `silu_and_mul_kernel` and its core logic
```python
@triton.jit
def silu_and_mul_kernel(
    out_hidden_states_ptr,  # (bs, hidden_dim)
    out_scales_ptr,  # (bs,)
    hidden_states_ptr,  # (bs, hidden_dim * 2)
    quant_max: tl.constexpr,
    static_scale: tl.constexpr,
    hidden_dim: tl.constexpr,  # the output hidden_dim
    BLOCK_SIZE: tl.constexpr,
):
    pid = tl.program_id(axis=0)

    input_start = pid * hidden_dim * 2
    output_start = pid * hidden_dim

    input1_offs = tl.arange(0, BLOCK_SIZE)
    mask = tl.arange(0, BLOCK_SIZE) < hidden_dim  # shared for input1, input3, output
    input3_offs = hidden_dim + tl.arange(0, BLOCK_SIZE)
    output_offs = tl.arange(0, BLOCK_SIZE)

    x1 = tl.load(
        hidden_states_ptr + input_start + input1_offs, mask=mask, other=0.0
    ).to(tl.float32)
    x3 = tl.load(
        hidden_states_ptr + input_start + input3_offs, mask=mask, other=0.0
    ).to(tl.float32)

    # silu
    # cast down before mul to better match training?
    silu_x1 = x1 * tl.sigmoid(x1)
    out = x3 * silu_x1.to(hidden_states_ptr.dtype.element_ty)

    if quant_max is not None:
        raise NotImplementedError()

    tl.store(out_hidden_states_ptr + output_start + output_offs, out, mask=mask)
```
**EN:** This block defines `silu_and_mul_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.arange`, `tl.load.to`, `tl.store`, and `tl.sigmoid`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `input_start`, `output_start`, `input1_offs`, and `mask` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `silu_and_mul_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.arange`、`tl.load.to`、`tl.store` 以及 `tl.sigmoid`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`input_start`、`output_start`、`input1_offs` 以及 `mask` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 539-591: Function `silu_and_mul_triton` and its core logic
```python
def silu_and_mul_triton(
    hidden_states,
    scales=None,
    quantize=None,  # dtype to quantize to
    out=None,
):
    bs, in_hidden_dim = hidden_states.shape
    hidden_dim = in_hidden_dim // 2

    if out is None:
        out_hidden_states = torch.empty(
            (bs, hidden_dim),
            dtype=quantize or hidden_states.dtype,
            device=hidden_states.device,
        )
    else:
        assert out.shape == (bs, hidden_dim)
        assert out.dtype == (quantize or hidden_states.dtype)
        out_hidden_states = out
    out_scales = None
    static_scale = False
    if quantize is not None:
        if scales is None:
            out_scales = torch.empty(
                (bs,), dtype=torch.float32, device=hidden_states.device
            )
        else:
            out_scales = scales
            static_scale = True

    max_warps = 16 if _is_hip else 32
    config = {
        # 8 ele per thread (not tuned)
        "num_warps": max(
            min(triton.next_power_of_2(triton.cdiv(hidden_dim, 8 * 32)), max_warps), 4
        ),
    }

    silu_and_mul_kernel[(bs,)](
        out_hidden_states,
        out_scales,
        hidden_states,
        quant_max=torch.finfo(quantize).max if quantize is not None else None,
        static_scale=static_scale,
        hidden_dim=hidden_dim,
        BLOCK_SIZE=triton.next_power_of_2(hidden_dim),
        **config,
    )

    if quantize is not None:
        return out_hidden_states, out_scales
    else:
        return out_hidden_states, None
```
**EN:** This block defines `silu_and_mul_triton` and contains the main logic for this step. It mainly invokes `silu_and_mul_kernel`, `torch.empty`, `max`, `min`, and `triton.next_power_of_2`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `bs`, `in_hidden_dim`, `hidden_dim`, `out_scales`, and `static_scale` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `silu_and_mul_triton`，并承载这一阶段的核心逻辑。 它主要调用 `silu_and_mul_kernel`、`torch.empty`、`max`、`min` 以及 `triton.next_power_of_2`，说明该流程会编排底层辅助函数或计算内核。 像 `bs`、`in_hidden_dim`、`hidden_dim`、`out_scales` 以及 `static_scale` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `fused_softcap_kernel`, `fused_softcap`, `Softcap`, `fused_dual_residual_rmsnorm_kernel`, and `fused_dual_residual_rmsnorm`. / **主要符号**：核心入口包括 `fused_softcap_kernel`、`fused_softcap`、`Softcap`、`fused_dual_residual_rmsnorm_kernel` 以及 `fused_dual_residual_rmsnorm`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `typing.Optional` and `typing.Tuple` / **标准库**：`typing.Optional` 和 `typing.Tuple`
- **Third-party**: `torch`, `triton`, and `triton.language` / **第三方依赖**：`torch`、`triton` 以及 `triton.language`
- **Internal SGLang modules**: `sglang.srt.utils.is_hip` and `sglang.srt.utils.custom_op.register_custom_op` / **SGLang 内部模块**：`sglang.srt.utils.is_hip` 和 `sglang.srt.utils.custom_op.register_custom_op`
