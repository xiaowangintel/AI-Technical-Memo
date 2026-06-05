# gptq_marlin_repack.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/gptq_marlin_repack.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and module setup
```python
from __future__ import annotations

from typing import TYPE_CHECKING

import torch

from sglang.jit_kernel.utils import cache_once, load_jit
from sglang.kernel_api_logging import debug_kernel_api

if TYPE_CHECKING:
    from tvm_ffi.module import Module

# Constants matching device::marlin:: in marlin.cuh
_TILE_SIZE = 16


@cache_once
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 18-26: Function `_jit_gptq_marlin_repack_module`
```python
def _jit_gptq_marlin_repack_module() -> Module:
    return load_jit(
        "gptq_marlin_repack",
        cuda_files=["gemm/marlin/gptq_marlin_repack.cuh"],
        cuda_wrappers=[("gptq_marlin_repack", "gptq_marlin_repack")],
    )


@debug_kernel_api
```
**EN:** This block defines `_jit_gptq_marlin_repack_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_gptq_marlin_repack_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 27-45: Function `gptq_marlin_repack`
```python
def gptq_marlin_repack(
    b_q_weight: torch.Tensor,
    perm: torch.Tensor,
    size_k: int,
    size_n: int,
    num_bits: int,
) -> torch.Tensor:
    pack_factor = 32 // num_bits

    # Allocate output tensor
    out = torch.empty(
        (size_k // _TILE_SIZE, size_n * _TILE_SIZE // pack_factor),
        dtype=b_q_weight.dtype,
        device=b_q_weight.device,
    )

    module = _jit_gptq_marlin_repack_module()
    module.gptq_marlin_repack(b_q_weight, perm, out, size_k, size_n, num_bits)
    return out
```
**EN:** This block defines `gptq_marlin_repack`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `gptq_marlin_repack`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `typing -> TYPE_CHECKING`
- `torch`
- `sglang.jit_kernel.utils -> cache_once`
- `sglang.kernel_api_logging -> debug_kernel_api`
- `tvm_ffi.module -> Module`
