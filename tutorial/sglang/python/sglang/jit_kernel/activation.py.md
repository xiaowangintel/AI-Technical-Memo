# activation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/activation.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Imports and module setup
```python
from __future__ import annotations

from typing import TYPE_CHECKING, Optional

import torch

from sglang.jit_kernel.utils import (
    cache_once,
    get_jit_cuda_arch,
    is_arch_support_pdl,
    is_hip_runtime,
    load_jit,
    make_cpp_args,
)
from sglang.srt.utils.custom_op import register_custom_op

if TYPE_CHECKING:
    from tvm_ffi.module import Module
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 21-31: Function `_fast_math_flags`
```python
def _fast_math_flags() -> list[str]:
    # Mirrors sgl-kernel's CMake policy: fast-math on SM90, precise on
    # SM100+ (Blackwell needs bit-exact expf), off on HIP (clang rejects).
    if is_hip_runtime():
        return []
    if get_jit_cuda_arch().major >= 10:
        return []
    return ["--use_fast_math"]


@cache_once
```
**EN:** This block defines `_fast_math_flags`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_fast_math_flags`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 32-52: Function `_jit_activation_module`
```python
def _jit_activation_module(dtype: torch.dtype) -> Module:
    args = make_cpp_args(dtype, is_arch_support_pdl())
    return load_jit(
        "activation",
        *args,
        cuda_files=["elementwise/activation.cuh"],
        extra_cuda_cflags=_fast_math_flags(),
        cuda_wrappers=[
            ("run_activation", f"ActivationKernel<{args}>::run_activation"),
            (
                "run_activation_filtered",
                f"ActivationKernel<{args}>::run_activation_filtered",
            ),
        ],
    )


SUPPORTED_ACTIVATIONS = {"silu", "gelu", "gelu_tanh"}


@register_custom_op(mutates_args=["out"])
```
**EN:** This block defines `_jit_activation_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_activation_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 53-63: Function `_run_activation_inplace`
```python
def _run_activation_inplace(
    op_name: str, input: torch.Tensor, out: torch.Tensor
) -> None:
    hidden_size = input.shape[-1] // 2
    module = _jit_activation_module(input.dtype)
    input_2d = input.view(-1, hidden_size * 2)
    out_2d = out.view(-1, hidden_size)
    module.run_activation(input_2d, out_2d, op_name)


@register_custom_op(mutates_args=["out"])
```
**EN:** This block defines `_run_activation_inplace`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_run_activation_inplace`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 64-77: Function `_run_activation_filtered_inplace`
```python
def _run_activation_filtered_inplace(
    op_name: str,
    input: torch.Tensor,
    out: torch.Tensor,
    expert_ids: torch.Tensor,
    expert_step: int,
) -> None:
    hidden_size = input.shape[-1] // 2
    module = _jit_activation_module(input.dtype)
    input_2d = input.view(-1, hidden_size * 2)
    out_2d = out.view(-1, hidden_size)
    module.run_activation_filtered(input_2d, out_2d, expert_ids, expert_step, op_name)
```
**EN:** This block defines `_run_activation_filtered_inplace`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_run_activation_filtered_inplace`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 78-102: Function `run_activation`
```python
def run_activation(
    op_name: str,
    input: torch.Tensor,
    out: Optional[torch.Tensor],
    expert_ids: Optional[torch.Tensor] = None,
    expert_step: int = 1,
) -> torch.Tensor:
    """Apply ``op_name`` activation followed by element-wise multiplication.

    When ``expert_ids`` is provided, output rows are skipped for tokens whose
    routed expert id is ``-1``. ``expert_step`` is 1 for per-token routing and
    ``BLOCK_SIZE_M`` for sorted/TMA routing — i.e. ``expert_ids[token_id //
    expert_step]`` is consulted before computing each row.
    """
    assert op_name in SUPPORTED_ACTIVATIONS, f"Unsupported activation: {op_name}"
    hidden_size = input.shape[-1] // 2
    if out is None:
        out = input.new_empty(*input.shape[:-1], hidden_size)
    if expert_ids is None:
        _run_activation_inplace(op_name, input, out)
    else:
        _run_activation_filtered_inplace(op_name, input, out, expert_ids, expert_step)
    return out
```
**EN:** This block defines `run_activation`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `run_activation`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 103-111: Function `silu_and_mul`
```python
def silu_and_mul(
    input: torch.Tensor,
    out: Optional[torch.Tensor] = None,
    expert_ids: Optional[torch.Tensor] = None,
    expert_step: int = 1,
) -> torch.Tensor:
    return run_activation("silu", input, out, expert_ids, expert_step)
```
**EN:** This block defines `silu_and_mul`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `silu_and_mul`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 112-120: Function `gelu_and_mul`
```python
def gelu_and_mul(
    input: torch.Tensor,
    out: Optional[torch.Tensor] = None,
    expert_ids: Optional[torch.Tensor] = None,
    expert_step: int = 1,
) -> torch.Tensor:
    return run_activation("gelu", input, out, expert_ids, expert_step)
```
**EN:** This block defines `gelu_and_mul`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `gelu_and_mul`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 121-127: Function `gelu_tanh_and_mul`
```python
def gelu_tanh_and_mul(
    input: torch.Tensor,
    out: Optional[torch.Tensor] = None,
    expert_ids: Optional[torch.Tensor] = None,
    expert_step: int = 1,
) -> torch.Tensor:
    return run_activation("gelu_tanh", input, out, expert_ids, expert_step)
```
**EN:** This block defines `gelu_tanh_and_mul`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `gelu_tanh_and_mul`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `typing -> TYPE_CHECKING`
- `torch`
- `sglang.jit_kernel.utils -> (`
- `sglang.srt.utils.custom_op -> register_custom_op`
- `tvm_ffi.module -> Module`
