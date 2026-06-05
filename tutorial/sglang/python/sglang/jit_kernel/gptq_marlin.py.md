# gptq_marlin.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/gptq_marlin.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and module setup
```python
from __future__ import annotations

from typing import TYPE_CHECKING, Optional

import torch

from sglang.jit_kernel.utils import cache_once, load_jit, make_cpp_args
from sglang.kernel_api_logging import debug_kernel_api

if TYPE_CHECKING:
    from sgl_kernel.scalar_type import ScalarType
    from tvm_ffi.module import Module

# Constants matching device::marlin:: in marlin.cuh
_MAX_THREAD_N = 256


@cache_once
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 19-28: Function `_jit_gptq_marlin_module`
```python
def _jit_gptq_marlin_module(dtype: torch.dtype) -> Module:
    args = make_cpp_args(dtype)
    return load_jit(
        "gptq_marlin",
        *args,
        cuda_files=["gemm/marlin/gptq_marlin.cuh"],
        cuda_wrappers=[("gptq_marlin_gemm", f"gptq_marlin_gemm<{args}>")],
    )
```
**EN:** This block defines `_jit_gptq_marlin_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_gptq_marlin_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 29-35: Function `_or_empty`
```python
def _or_empty(
    t: Optional[torch.Tensor], device: torch.device, dtype: torch.dtype
) -> torch.Tensor:
    return t if t is not None else torch.empty(0, device=device, dtype=dtype)


@debug_kernel_api
```
**EN:** This block defines `_or_empty`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_or_empty`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 36-117: Function `gptq_marlin_gemm`
```python
def gptq_marlin_gemm(
    a: torch.Tensor,
    c: Optional[torch.Tensor],
    b_q_weight: torch.Tensor,
    b_scales: torch.Tensor,
    global_scale: Optional[torch.Tensor],
    b_zeros: Optional[torch.Tensor],
    g_idx: Optional[torch.Tensor],
    perm: Optional[torch.Tensor],
    workspace: torch.Tensor,
    b_q_type: ScalarType,
    size_m: int,
    size_n: int,
    size_k: int,
    is_k_full: bool = True,
    use_atomic_add: bool = False,
    use_fp32_reduce: bool = False,
    is_zp_float: bool = False,
) -> torch.Tensor:
    device = a.device

    # Allocate output if not provided
    if c is None:
        c = torch.empty((size_m, size_n), dtype=a.dtype, device=device)

    # Early return for zero-size M
    if size_m == 0:
        return c

    # Determine activation ordering
    has_act_order = (
        g_idx is not None
        and perm is not None
        and g_idx.numel() > 0
        and perm.numel() > 0
    )

    # Allocate c_tmp for fp32 reduce
    if use_fp32_reduce:
        sms = torch.cuda.get_device_properties(device).multi_processor_count
        max_m_block = min(((size_m + 15) // 16) * 16, 64)
        c_tmp = torch.empty(
            sms * max_m_block * _MAX_THREAD_N,
            dtype=torch.float32,
            device=device,
        )
    else:
        c_tmp = torch.empty(0, dtype=torch.float32, device=device)

    # Allocate a_tmp for act_order column permutation
    if has_act_order:
        a_tmp = torch.empty((size_m, size_k), dtype=a.dtype, device=device)
    else:
        a_tmp = torch.empty(0, dtype=a.dtype, device=device)

    # Convert Optional tensors to empty tensors
    global_scale_t = _or_empty(global_scale, device, a.dtype)
    b_zeros_t = _or_empty(b_zeros, device, torch.int32)
    g_idx_t = _or_empty(g_idx, device, torch.int32)
    perm_t = _or_empty(perm, device, torch.int32)

    module = _jit_gptq_marlin_module(a.dtype)
    module.gptq_marlin_gemm(
        a,
        b_q_weight,
        b_scales,
        global_scale_t,
        b_zeros_t,
        g_idx_t,
        perm_t,
        c,
        c_tmp,
        a_tmp,
        workspace,
        b_q_type.id,
        is_k_full,
        use_atomic_add,
        use_fp32_reduce,
        is_zp_float,
    )
# ...
```
**EN:** This block defines `gptq_marlin_gemm`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `gptq_marlin_gemm`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `typing -> TYPE_CHECKING`
- `torch`
- `sglang.jit_kernel.utils -> cache_once`
- `sglang.kernel_api_logging -> debug_kernel_api`
- `sgl_kernel.scalar_type -> ScalarType`
- `tvm_ffi.module -> Module`
