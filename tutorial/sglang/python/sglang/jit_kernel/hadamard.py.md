# hadamard.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/hadamard.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Imports and module setup
```python
from __future__ import annotations

from typing import TYPE_CHECKING, Callable

import torch

from sglang.jit_kernel.utils import KERNEL_PATH, cache_once, load_jit, make_cpp_args

if TYPE_CHECKING:
    from tvm_ffi.module import Module


@cache_once
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 14-31: Function `_jit_hadamard_module`
```python
def _jit_hadamard_module(dtype: torch.dtype) -> Module:
    args = make_cpp_args(dtype)
    hadamard_include_dir = (KERNEL_PATH / "csrc" / "fast-hadamard-transform").resolve()
    return load_jit(
        "hadamard",
        *args,
        cuda_files=["fast-hadamard-transform/hadamard_jit.cuh"],
        cuda_wrappers=[
            ("hadamard_transform", f"HadamardKernel<{args}>::run"),
            ("hadamard_transform_12n", f"Hadamard12NKernel<{args}>::run"),
            ("hadamard_transform_20n", f"Hadamard20NKernel<{args}>::run"),
            ("hadamard_transform_28n", f"Hadamard28NKernel<{args}>::run"),
            ("hadamard_transform_40n", f"Hadamard40NKernel<{args}>::run"),
        ],
        extra_include_paths=[str(hadamard_include_dir)],
    )
```
**EN:** This block defines `_jit_hadamard_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_hadamard_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 32-58: Function `_hadamard_transform_impl`
```python
def _hadamard_transform_impl(
    x: torch.Tensor,
    scale: float,
    pad_multiple: int,
    kernel_fn: Callable,
) -> torch.Tensor:
    if not x.is_cuda:
        raise RuntimeError(f"{kernel_fn.__name__} only supports CUDA tensors")

    shapes_og = x.size()
    dim_og = x.size(-1)
    x = x.reshape(-1, dim_og)
    if x.stride(-1) != 1:
        x = x.contiguous()

    needs_pad = dim_og % pad_multiple != 0
    if needs_pad:
        x = torch.nn.functional.pad(x, (0, pad_multiple - dim_og % pad_multiple))

    out = torch.empty_like(x)
    kernel_fn(x, out, scale)

    if needs_pad:
        out = out[:, :dim_og]
    return out.reshape(shapes_og)
```
**EN:** This block defines `_hadamard_transform_impl`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_hadamard_transform_impl`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 59-63: Function `hadamard_transform`
```python
def hadamard_transform(x: torch.Tensor, scale: float = 1.0) -> torch.Tensor:
    module = _jit_hadamard_module(x.dtype)
    return _hadamard_transform_impl(x, scale, 8, module.hadamard_transform)
```
**EN:** This block defines `hadamard_transform`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `hadamard_transform`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 64-68: Function `hadamard_transform_12n`
```python
def hadamard_transform_12n(x: torch.Tensor, scale: float = 1.0) -> torch.Tensor:
    module = _jit_hadamard_module(x.dtype)
    return _hadamard_transform_impl(x, scale, 4 * 12, module.hadamard_transform_12n)
```
**EN:** This block defines `hadamard_transform_12n`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `hadamard_transform_12n`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 69-73: Function `hadamard_transform_20n`
```python
def hadamard_transform_20n(x: torch.Tensor, scale: float = 1.0) -> torch.Tensor:
    module = _jit_hadamard_module(x.dtype)
    return _hadamard_transform_impl(x, scale, 4 * 20, module.hadamard_transform_20n)
```
**EN:** This block defines `hadamard_transform_20n`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `hadamard_transform_20n`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 74-78: Function `hadamard_transform_28n`
```python
def hadamard_transform_28n(x: torch.Tensor, scale: float = 1.0) -> torch.Tensor:
    module = _jit_hadamard_module(x.dtype)
    return _hadamard_transform_impl(x, scale, 4 * 28, module.hadamard_transform_28n)
```
**EN:** This block defines `hadamard_transform_28n`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `hadamard_transform_28n`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 79-81: Function `hadamard_transform_40n`
```python
def hadamard_transform_40n(x: torch.Tensor, scale: float = 1.0) -> torch.Tensor:
    module = _jit_hadamard_module(x.dtype)
    return _hadamard_transform_impl(x, scale, 4 * 40, module.hadamard_transform_40n)
```
**EN:** This block defines `hadamard_transform_40n`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `hadamard_transform_40n`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `typing -> TYPE_CHECKING`
- `torch`
- `sglang.jit_kernel.utils -> KERNEL_PATH`
- `tvm_ffi.module -> Module`
