# cast.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/cast.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Imports and module setup
```python
from __future__ import annotations

from typing import TYPE_CHECKING

import torch

from sglang.jit_kernel.utils import cache_once, load_jit, make_cpp_args

if TYPE_CHECKING:
    from tvm_ffi.module import Module


@cache_once
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 14-23: Function `_jit_cast_module`
```python
def _jit_cast_module(dtype: torch.dtype) -> Module:
    args = make_cpp_args(dtype)
    return load_jit(
        "cast",
        *args,
        cuda_files=["elementwise/cast.cuh"],
        cuda_wrappers=[("downcast_fp8", f"downcast_fp8<{args}>")],
    )
```
**EN:** This block defines `_jit_cast_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_cast_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 24-52: Function `downcast_fp8`
```python
def downcast_fp8(
    k: torch.Tensor,
    v: torch.Tensor,
    k_out: torch.Tensor,
    v_out: torch.Tensor,
    k_scale: torch.Tensor,
    v_scale: torch.Tensor,
    loc: torch.Tensor,
    mult: int = 1,
    offset: int = 0,
) -> None:
    """Fused downcast of KV cache tensors from bf16/fp16 to fp8 (E4M3).

    Scales each value by the inverse of its per-tensor scale, clamps to the
    fp8 representable range [-448, 448], then converts to fp8 storage.

    Args:
        k:       [input_sl, head, dim] bf16/fp16 CUDA tensor
        v:       [input_sl, head, dim] bf16/fp16 CUDA tensor
        k_out:   [out_sl, head, dim]   uint8 CUDA tensor (fp8 storage)
        v_out:   [out_sl, head, dim]   uint8 CUDA tensor (fp8 storage)
        k_scale: [1] float32 CUDA tensor, scale for k
        v_scale: [1] float32 CUDA tensor, scale for v
        loc:     [input_sl] int64 CUDA tensor, destination sequence indices
        mult:    stride multiplier for output index (default 1)
        offset:  offset added to output index (default 0)
    """
    module = _jit_cast_module(k.dtype)
    module.downcast_fp8(k, v, k_out, v_out, k_scale, v_scale, loc, mult, offset)
```
**EN:** This block defines `downcast_fp8`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `downcast_fp8`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `typing -> TYPE_CHECKING`
- `torch`
- `sglang.jit_kernel.utils -> cache_once`
- `tvm_ffi.module -> Module`
