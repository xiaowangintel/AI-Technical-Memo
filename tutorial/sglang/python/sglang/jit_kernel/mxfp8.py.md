# mxfp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/mxfp8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and module setup
```python
from __future__ import annotations

from typing import TYPE_CHECKING

import torch

from sglang.jit_kernel.utils import (
    cache_once,
    load_jit,
    make_cpp_args,
    override_jit_cuda_arch,
)

if TYPE_CHECKING:
    from tvm_ffi.module import Module
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 18-27: Function `_mxfp8_cuda_flags`
```python
def _mxfp8_cuda_flags() -> list[str]:
    return [
        "-DNDEBUG",
        "-DCUTLASS_ENABLE_TENSOR_CORE_MMA=1",
        "-DCUTLASS_VERSIONS_GENERATED",
        "-DCUTLASS_DEBUG_TRACE_LEVEL=0",
        "--expt-extended-lambda",
    ]
```
**EN:** This block defines `_mxfp8_cuda_flags`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_mxfp8_cuda_flags`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 28-44: Function `_mxfp8_arch_env`
```python
def _mxfp8_arch_env():
    if not torch.cuda.is_available():
        raise RuntimeError("MXFP8 JIT kernels require CUDA.")
    major, minor = torch.cuda.get_device_capability()
    if major < 10:
        raise RuntimeError(
            f"MXFP8 JIT kernels require compute capability >= 10.0, got {major}.{minor}."
        )
    # MXFP8 kernels use architecture-family-specific instructions and must be
    # compiled for `sm_*a` targets (e.g. sm_100a), not plain sm_100.
    # JIT compilation targets only the current device, unlike AOT fat-binaries;
    # adding extra architectures here would clash with the single SGL_CUDA_ARCH
    # value injected by load_jit().
    return override_jit_cuda_arch(major, minor, suffix="a")


@cache_once
```
**EN:** This block defines `_mxfp8_arch_env`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_mxfp8_arch_env`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 45-65: Function `_jit_es_sm100_mxfp8_blockscaled_group_quant`
```python
def _jit_es_sm100_mxfp8_blockscaled_group_quant(dtype: torch.dtype) -> Module:
    args = make_cpp_args(dtype)
    with _mxfp8_arch_env():
        return load_jit(
            "es_sm100_mxfp8_blockscaled_group_quant",
            *args,
            cuda_files=[
                "moe/expert_specialization/es_sm100_mxfp8_blockscaled_group_quant.cuh"
            ],
            cuda_wrappers=[
                (
                    "es_sm100_mxfp8_blockscaled_group_quant",
                    f"EsSm100MXFP8BlockscaledGroupQuant<{args}>::run",
                )
            ],
            extra_dependencies=["cutlass"],
            extra_cuda_cflags=_mxfp8_cuda_flags(),
        )


@cache_once
```
**EN:** This block defines `_jit_es_sm100_mxfp8_blockscaled_group_quant`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_es_sm100_mxfp8_blockscaled_group_quant`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 66-85: Function `_jit_es_sm100_mxfp8_blockscaled_moe_group_gemm`
```python
def _jit_es_sm100_mxfp8_blockscaled_moe_group_gemm(dtype: torch.dtype) -> Module:
    args = make_cpp_args(dtype)
    with _mxfp8_arch_env():
        return load_jit(
            "es_sm100_mxfp8_blockscaled_moe_group_gemm",
            *args,
            cuda_files=[
                "moe/expert_specialization/es_sm100_mxfp8_blockscaled_moe_group_gemm.cuh"
            ],
            cuda_wrappers=[
                (
                    "es_sm100_mxfp8_blockscaled_moe_group_gemm",
                    f"EsSm100MXFP8BlockscaledMoeGroupGemm<{args}>::run",
                )
            ],
            extra_dependencies=["cutlass"],
            extra_cuda_cflags=_mxfp8_cuda_flags(),
        )
```
**EN:** This block defines `_jit_es_sm100_mxfp8_blockscaled_moe_group_gemm`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_es_sm100_mxfp8_blockscaled_moe_group_gemm`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 86-104: Function `es_sm100_mxfp8_blockscaled_grouped_quant`
```python
def es_sm100_mxfp8_blockscaled_grouped_quant(
    input: torch.Tensor,
    tokens_per_expert: torch.Tensor,
    expert_offsets: torch.Tensor,
    blockscale_offsets: torch.Tensor,
    quant_output: torch.Tensor,
    scale_factor: torch.Tensor,
) -> None:
    module = _jit_es_sm100_mxfp8_blockscaled_group_quant(input.dtype)
    module.es_sm100_mxfp8_blockscaled_group_quant(
        input,
        tokens_per_expert,
        expert_offsets,
        blockscale_offsets,
        quant_output,
        scale_factor,
    )
```
**EN:** This block defines `es_sm100_mxfp8_blockscaled_grouped_quant`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `es_sm100_mxfp8_blockscaled_grouped_quant`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 105-136: Function `es_sm100_mxfp8_blockscaled_moe_grouped_gemm`
```python
def es_sm100_mxfp8_blockscaled_moe_grouped_gemm(
    a: torch.Tensor,
    b: torch.Tensor,
    sfa: torch.Tensor,
    sfb: torch.Tensor,
    expert_offsets: torch.Tensor,
    blockscale_offsets: torch.Tensor,
    tokens_per_expert: torch.Tensor,
    workspace: torch.Tensor,
    dtype: torch.dtype,
) -> torch.Tensor:
    num_experts, m, tokens = a.shape[0], a.shape[1], b.shape[0]
    d = torch.empty((tokens, m), device=a.device, dtype=dtype)
    d_ptrs = torch.empty((num_experts,), device=a.device, dtype=torch.int64)
    b_ptrs = torch.empty((num_experts,), device=a.device, dtype=torch.int64)
    sfb_ptrs = torch.empty((num_experts,), device=a.device, dtype=torch.int64)
    module = _jit_es_sm100_mxfp8_blockscaled_moe_group_gemm(dtype)
    module.es_sm100_mxfp8_blockscaled_moe_group_gemm(
        a,
        b,
        sfa,
        sfb,
        expert_offsets,
        blockscale_offsets,
        tokens_per_expert,
        b_ptrs,
        sfb_ptrs,
        d,
        d_ptrs,
        workspace,
    )
    return d
```
**EN:** This block defines `es_sm100_mxfp8_blockscaled_moe_grouped_gemm`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `es_sm100_mxfp8_blockscaled_moe_grouped_gemm`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `typing -> TYPE_CHECKING`
- `torch`
- `sglang.jit_kernel.utils -> (`
- `tvm_ffi.module -> Module`
