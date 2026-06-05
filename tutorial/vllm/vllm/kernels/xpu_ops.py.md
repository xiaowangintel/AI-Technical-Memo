# xpu_ops.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/kernels/xpu_ops.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines Python wrappers for XPU-specific custom operators. / 定义 XPU 专用自定义算子的 Python 包装层。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-7)
```python
import torch
from torch import Tensor

from vllm import ir
from vllm.platforms import current_platform
```
**EN:** This import block loads `torch`, `vllm`, `vllm.platforms`, establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `xpu_ops.py`.
**CN:** 该导入代码块加载了 `torch`, `vllm`, `vllm.platforms`，为 `xpu_ops.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Top-level block (lines 9-9)
```python
current_platform.import_kernels()
```
**EN:** This top-level control-flow block coordinates module initialization before the main kernels are used.
**CN:** 这个顶层控制流代码块在主内核使用前协调模块初始化流程。

### Function `is_xpu_kernels_found` (lines 12-15)
```python
def is_xpu_kernels_found() -> bool:
    from importlib.util import find_spec

    return find_spec("vllm_xpu_kernels") is not None
```
**EN:** This predicate implements `is_xpu_kernels_found`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `find_spec`.
**CN:** 该函数 `is_xpu_kernels_found` 封装了此模块中的一段关键运行时逻辑，重点处理 is xpu kernels found 相关工作。 它内部会调用 `find_spec` 等例程。

### Constants / assignments (lines 18-18)
```python
XPU_KERNELS_SUPPORTED = is_xpu_kernels_found()
```
**EN:** This block defines module constants (`XPU_KERNELS_SUPPORTED`) that encode defaults, feature flags, or operator metadata used later in the file.
**CN:** 该代码块定义了模块常量（`XPU_KERNELS_SUPPORTED`），用于表示默认值、特性开关或后续逻辑要使用的算子元数据。

### Top-level block (lines 19-19)
```python
"""Kernels in this file are supported if vLLM XPU kernels are installed."""
```
**EN:** This top-level control-flow block coordinates module initialization before the main kernels are used.
**CN:** 这个顶层控制流代码块在主内核使用前协调模块初始化流程。

### Constants / assignments (lines 21-23)
```python
rms_no_var = lambda x, weight, epsilon, variance_size=None: variance_size is None and (
    weight is None or weight.dtype == x.dtype
)
```
**EN:** This assignment block initializes `rms_no_var`, capturing shared state or configuration consumed by later helper functions and kernel classes.
**CN:** 该赋值代码块初始化了 `rms_no_var`，保存后续辅助函数和内核类会消费的共享状态或配置。

### Function `rms_norm` (lines 29-38)
```python
def rms_norm(
    x: Tensor, weight: Tensor | None, epsilon: float, variance_size: int | None = None
) -> Tensor:
    if weight is None:
        # Kernel requires weight tensor, pass ones
        weight = torch.ones(x.shape[-1], device=x.device, dtype=x.dtype)
    assert variance_size is None
    output = torch.empty(x.shape, device=x.device, dtype=x.dtype)
    torch.ops._C.rms_norm(output, x, weight, epsilon)
    return output
```
**EN:** This helper implements `rms_norm`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `register_impl`, `empty`, `rms_norm`, `ones`.
**CN:** 该函数 `rms_norm` 封装了此模块中的一段关键运行时逻辑，重点处理 rms norm 相关工作。 它内部会调用 `register_impl`, `empty`, `rms_norm`, `ones` 等例程。

### Constants / assignments (lines 41-44)
```python
rms_add_no_var_size = (
    lambda x, x_residual, weight, epsilon, variance_size=None: variance_size is None
    and (weight is None or weight.dtype == x.dtype)
)
```
**EN:** This assignment block initializes `rms_add_no_var_size`, capturing shared state or configuration consumed by later helper functions and kernel classes.
**CN:** 该赋值代码块初始化了 `rms_add_no_var_size`，保存后续辅助函数和内核类会消费的共享状态或配置。

### Function `fused_add_rms_norm` (lines 53-66)
```python
def fused_add_rms_norm(
    x: Tensor,
    x_residual: Tensor,
    weight: Tensor | None,
    epsilon: float,
    variance_size: int | None = None,
) -> tuple[Tensor, Tensor]:
    if weight is None:
        # Kernel requires weight tensor, pass ones
        weight = torch.ones(x.shape[-1], device=x.device, dtype=x.dtype)

    assert variance_size is None
    torch.ops._C.fused_add_rms_norm(x, x_residual, weight, epsilon)
    return x, x_residual
```
**EN:** This helper implements `fused_add_rms_norm`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `register_impl`, `fused_add_rms_norm`, `ones`.
**CN:** 该函数 `fused_add_rms_norm` 封装了此模块中的一段关键运行时逻辑，重点处理 fused add rms norm 相关工作。 它内部会调用 `register_impl`, `fused_add_rms_norm`, `ones` 等例程。

## Key Concepts / 关键概念
- **XPU support / XPU 支持**
  - **EN:** The file adds Intel XPU-specific dispatch, operators, or compatibility checks.
  - **CN:** 该文件加入 Intel XPU 专用的分派、算子或兼容性检查。
- **Kernel registration / 内核注册**
  - **EN:** The file registers operators, backends, or preset configurations with PyTorch/vLLM.
  - **CN:** 该文件向 PyTorch/vLLM 注册算子、后端或预设配置。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.platforms`
- **External / 外部依赖**: `torch`
