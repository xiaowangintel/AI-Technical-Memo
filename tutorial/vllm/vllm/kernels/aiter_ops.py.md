# aiter_ops.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/kernels/aiter_ops.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Wraps AITER custom operators behind Python functions with fallback checks. / 在 Python 函数层封装 AITER 自定义算子，并带有回退检查。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-11)
```python
import functools

import torch
from torch import Tensor
from torch.library import Library

from vllm import ir
from vllm.platforms import current_platform
from vllm.utils.torch_utils import direct_register_custom_op
```
**EN:** This import block loads `functools`, `torch`, `torch.library`, `vllm`, `vllm.platforms`, `vllm.utils.torch_utils`, establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `aiter_ops.py`.
**CN:** 该导入代码块加载了 `functools`, `torch`, `torch.library`, `vllm`, `vllm.platforms`, `vllm.utils.torch_utils`，为 `aiter_ops.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Top-level block (lines 13-13)
```python
current_platform.import_kernels()
```
**EN:** This top-level control-flow block coordinates module initialization before the main kernels are used.
**CN:** 这个顶层控制流代码块在主内核使用前协调模块初始化流程。

### Function `is_aiter_found` (lines 16-19)
```python
def is_aiter_found() -> bool:
    from importlib.util import find_spec

    return find_spec("aiter") is not None
```
**EN:** This predicate implements `is_aiter_found`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `find_spec`.
**CN:** 该函数 `is_aiter_found` 封装了此模块中的一段关键运行时逻辑，重点处理 is aiter found 相关工作。 它内部会调用 `find_spec` 等例程。

### Constants / assignments (lines 22-22)
```python
aiter_lib = Library("vllm_aiter", "FRAGMENT")
```
**EN:** This assignment block initializes `aiter_lib`, capturing shared state or configuration consumed by later helper functions and kernel classes.
**CN:** 该赋值代码块初始化了 `aiter_lib`，保存后续辅助函数和内核类会消费的共享状态或配置。

### Top-level block (lines 23-27)
```python
"""
This library holds torch custom ops for wrapped AITER ops.
Many AITER ops want to remain invisible to torch.compile even after lowering.
They are thus wrapped into torch custom ops inside the IR op implementations.
"""
```
**EN:** This top-level control-flow block coordinates module initialization before the main kernels are used.
**CN:** 这个顶层控制流代码块在主内核使用前协调模块初始化流程。

### Constants / assignments (lines 29-31)
```python
direct_register_aiter_op = functools.partial(
    direct_register_custom_op, target_lib=aiter_lib
)
```
**EN:** This assignment block initializes `direct_register_aiter_op`, capturing shared state or configuration consumed by later helper functions and kernel classes.
**CN:** 该赋值代码块初始化了 `direct_register_aiter_op`，保存后续辅助函数和内核类会消费的共享状态或配置。

### Top-level block (lines 32-32)
```python
"""Syntactic sugar for registering AITER custom ops."""
```
**EN:** This top-level control-flow block coordinates module initialization before the main kernels are used.
**CN:** 这个顶层控制流代码块在主内核使用前协调模块初始化流程。

### Constants / assignments (lines 34-34)
```python
AITER_SUPPORTED = is_aiter_found()
```
**EN:** This block defines module constants (`AITER_SUPPORTED`) that encode defaults, feature flags, or operator metadata used later in the file.
**CN:** 该代码块定义了模块常量（`AITER_SUPPORTED`），用于表示默认值、特性开关或后续逻辑要使用的算子元数据。

### Top-level block (lines 35-35)
```python
"""Most kernels in this file are supported if AITER is installed."""
```
**EN:** This top-level control-flow block coordinates module initialization before the main kernels are used.
**CN:** 这个顶层控制流代码块在主内核使用前协调模块初始化流程。

### Constants / assignments (lines 37-41)
```python
rms_no_var_16bit_only = (
    lambda x, weight, epsilon, variance_size=None: variance_size is None
    and x.dtype in (torch.float16, torch.bfloat16)
    and (weight is None or weight.dtype == x.dtype)
)
```
**EN:** This assignment block initializes `rms_no_var_16bit_only`, capturing shared state or configuration consumed by later helper functions and kernel classes.
**CN:** 该赋值代码块初始化了 `rms_no_var_16bit_only`，保存后续辅助函数和内核类会消费的共享状态或配置。

### Top-level block (lines 42-43)
```python
"""AITER rms_norm only supports float16 and bfloat16 acts, no var_size override,
and requires weight dtype to match x dtype."""
```
**EN:** This top-level control-flow block coordinates module initialization before the main kernels are used.
**CN:** 这个顶层控制流代码块在主内核使用前协调模块初始化流程。

### Function `rms_norm` (lines 49-56)
```python
def rms_norm(
    x: Tensor, weight: Tensor | None, epsilon: float, variance_size: int | None = None
) -> Tensor:
    assert variance_size is None
    assert x.dtype in (torch.float16, torch.bfloat16)
    if weight is None:
        weight = torch.ones(x.shape[-1], device=x.device, dtype=x.dtype)
    return torch.ops.vllm_aiter.rms_norm(x, weight, epsilon)
```
**EN:** This helper implements `rms_norm`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `register_impl`, `rms_norm`, `ones`.
**CN:** 该函数 `rms_norm` 封装了此模块中的一段关键运行时逻辑，重点处理 rms norm 相关工作。 它内部会调用 `register_impl`, `rms_norm`, `ones` 等例程。

### Function `_rms_norm_impl` (lines 59-68)
```python
def _rms_norm_impl(x: Tensor, weight: Tensor, variance_epsilon: float) -> Tensor:
    from aiter import rms_norm

    if x.dim() > 2:
        x_original_shape = x.shape
        x = x.reshape(-1, x_original_shape[-1])
        x = rms_norm(x, weight, variance_epsilon)
        return x.reshape(x_original_shape)

    return rms_norm(x, weight, variance_epsilon)
```
**EN:** This helper implements `_rms_norm_impl`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `rms_norm`, `dim`, `reshape`.
**CN:** 该函数 `_rms_norm_impl` 封装了此模块中的一段关键运行时逻辑，重点处理 rms norm impl 相关工作。 它内部会调用 `rms_norm`, `dim`, `reshape` 等例程。

### Function `_rms_norm_fake` (lines 71-72)
```python
def _rms_norm_fake(x: Tensor, weight: Tensor, variance_epsilon: float) -> Tensor:
    return torch.empty_like(x)
```
**EN:** This helper implements `_rms_norm_fake`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `empty_like`.
**CN:** 该函数 `_rms_norm_fake` 封装了此模块中的一段关键运行时逻辑，重点处理 rms norm fake 相关工作。 它内部会调用 `empty_like` 等例程。

### Top-level block (lines 75-77)
```python
direct_register_aiter_op(
    op_name="rms_norm", op_func=_rms_norm_impl, fake_impl=_rms_norm_fake
)
```
**EN:** This top-level control-flow block coordinates module initialization before the main kernels are used.
**CN:** 这个顶层控制流代码块在主内核使用前协调模块初始化流程。

### Constants / assignments (lines 79-83)
```python
rms_add_no_var_16bit_only = (
    lambda x, x_residual, weight, epsilon, variance_size=None: variance_size is None
    and x.dtype in (torch.float16, torch.bfloat16)
    and (weight is None or weight.dtype == x.dtype)
)
```
**EN:** This assignment block initializes `rms_add_no_var_16bit_only`, capturing shared state or configuration consumed by later helper functions and kernel classes.
**CN:** 该赋值代码块初始化了 `rms_add_no_var_16bit_only`，保存后续辅助函数和内核类会消费的共享状态或配置。

### Top-level block (lines 84-87)
```python
"""
AITER fused_add_rms_norm only supports 16-bit activations and no var_size override.
Requires weight dtype to match x dtype.
"""
```
**EN:** This top-level control-flow block coordinates module initialization before the main kernels are used.
**CN:** 这个顶层控制流代码块在主内核使用前协调模块初始化流程。

### Function `fused_add_rms_norm` (lines 93-104)
```python
def fused_add_rms_norm(
    x: Tensor,
    x_residual: Tensor,
    weight: Tensor | None,
    epsilon: float,
    variance_size: int | None = None,
) -> tuple[Tensor, Tensor]:
    assert variance_size is None
    assert x.dtype in (torch.float16, torch.bfloat16)
    if weight is None:
        weight = torch.ones(x.shape[-1], device=x.device, dtype=x.dtype)
    return torch.ops.vllm_aiter.fused_add_rms_norm(x, x_residual, weight, epsilon)
```
**EN:** This helper implements `fused_add_rms_norm`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `register_impl`, `fused_add_rms_norm`, `ones`.
**CN:** 该函数 `fused_add_rms_norm` 封装了此模块中的一段关键运行时逻辑，重点处理 fused add rms norm 相关工作。 它内部会调用 `register_impl`, `fused_add_rms_norm`, `ones` 等例程。

### Function `_rocm_aiter_rmsnorm2d_fwd_with_add_impl` (lines 107-128)
```python
def _rocm_aiter_rmsnorm2d_fwd_with_add_impl(
    x: torch.Tensor,
    residual: torch.Tensor,
    weight: torch.Tensor,
    variance_epsilon: float,
) -> tuple[torch.Tensor, torch.Tensor]:
    from aiter import rmsnorm2d_fwd_with_add

    # TODO can out = x and residual_out = residual to save memory?
    #  Need to check if the kernel supports in-place residual output
    #  (if yes set mutates_args and inplace)
    residual_out = torch.empty_like(residual)
    out = torch.empty_like(x)
    rmsnorm2d_fwd_with_add(
        out,  # output
        x,  # input
        residual,  # residual input
        residual_out,  # residual output
        weight,
        variance_epsilon,
    )
    return out, residual_out
```
**EN:** This helper implements `_rocm_aiter_rmsnorm2d_fwd_with_add_impl`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `empty_like`, `rmsnorm2d_fwd_with_add`.
**CN:** 该函数 `_rocm_aiter_rmsnorm2d_fwd_with_add_impl` 封装了此模块中的一段关键运行时逻辑，重点处理 rocm aiter rmsnorm 2 d fwd with add impl 相关工作。 它内部会调用 `empty_like`, `rmsnorm2d_fwd_with_add` 等例程。

### Function `_rocm_aiter_rmsnorm2d_fwd_with_add_fake` (lines 131-139)
```python
def _rocm_aiter_rmsnorm2d_fwd_with_add_fake(
    x: torch.Tensor,
    residual: torch.Tensor,
    weight: torch.Tensor,
    variance_epsilon: float,
) -> tuple[torch.Tensor, torch.Tensor]:
    residual_out = torch.empty_like(residual)
    out = torch.empty_like(x)
    return out, residual_out
```
**EN:** This helper implements `_rocm_aiter_rmsnorm2d_fwd_with_add_fake`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `empty_like`.
**CN:** 该函数 `_rocm_aiter_rmsnorm2d_fwd_with_add_fake` 封装了此模块中的一段关键运行时逻辑，重点处理 rocm aiter rmsnorm 2 d fwd with add fake 相关工作。 它内部会调用 `empty_like` 等例程。

### Top-level block (lines 142-146)
```python
direct_register_aiter_op(
    op_name="fused_add_rms_norm",
    op_func=_rocm_aiter_rmsnorm2d_fwd_with_add_impl,
    fake_impl=_rocm_aiter_rmsnorm2d_fwd_with_add_fake,
)
```
**EN:** This top-level control-flow block coordinates module initialization before the main kernels are used.
**CN:** 这个顶层控制流代码块在主内核使用前协调模块初始化流程。

## Key Concepts / 关键概念
- **AITER backend / AITER 后端**
  - **EN:** The module routes execution through AITER custom operators or kernel helpers.
  - **CN:** 该模块通过 AITER 自定义算子或内核辅助逻辑完成执行。
- **ROCm support / ROCm 支持**
  - **EN:** The implementation targets ROCm-capable devices and their kernel constraints.
  - **CN:** 该实现面向 ROCm 设备及其内核约束。
- **Kernel registration / 内核注册**
  - **EN:** The file registers operators, backends, or preset configurations with PyTorch/vLLM.
  - **CN:** 该文件向 PyTorch/vLLM 注册算子、后端或预设配置。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.platforms`, `vllm.utils.torch_utils`
- **External / 外部依赖**: `functools`, `torch`, `torch.library`
