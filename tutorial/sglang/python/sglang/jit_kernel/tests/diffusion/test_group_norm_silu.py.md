# test_group_norm_silu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/diffusion/test_group_norm_silu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import sys". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import sys”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Imports and module setup
```python
import sys

import pytest
import torch
import torch.nn as nn
import torch.nn.functional as F

from sglang.jit_kernel.diffusion.group_norm_silu import apply_group_norm_silu
from sglang.jit_kernel.diffusion.triton.group_norm_silu import triton_group_norm_silu
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=8, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=120, suite="nightly-kernel-1-gpu", nightly=True)

DEVICE = "cuda"
DTYPES = [torch.float16, torch.bfloat16, torch.float32]
TEST_CASES = [
    pytest.param((2, 64, 32, 32), 32, id="image_2d"),
    pytest.param((1, 64, 4, 16, 16), 32, id="video_3d"),
    pytest.param((4, 128), 32, id="token_2d"),
]
LARGE_TILE_CASE = ((1, 128, 20, 256, 256), 32)
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 25-33: Function `_tol`
```python
def _tol(dtype: torch.dtype) -> tuple[float, float]:
    if dtype == torch.float32:
        return 1e-5, 1e-5
    if dtype == torch.bfloat16:
        return 7e-2, 2e-2
    return 3e-3, 3e-3


@pytest.fixture(autouse=True)
```
**EN:** This block defines `_tol`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_tol`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 34-39: Function `cuda_setup`
```python
def cuda_setup():
    if not torch.cuda.is_available():
        pytest.skip("CUDA required")
    torch.cuda.manual_seed(0)
```
**EN:** This block defines `cuda_setup`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `cuda_setup`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 40-52: Function `_reference`
```python
def _reference(
    x: torch.Tensor,
    weight: torch.Tensor,
    bias: torch.Tensor,
    num_groups: int,
    eps: float = 1e-5,
) -> torch.Tensor:
    return F.silu(F.group_norm(x, num_groups, weight=weight, bias=bias, eps=eps))


@torch.no_grad()
@pytest.mark.parametrize("shape,num_groups", TEST_CASES)
@pytest.mark.parametrize("dtype", DTYPES)
```
**EN:** This block defines `_reference`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_reference`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 53-70: Function `test_triton_group_norm_silu`
```python
def test_triton_group_norm_silu(
    shape: tuple[int, ...], num_groups: int, dtype: torch.dtype
) -> None:
    channels = shape[1]
    x = torch.randn(shape, device=DEVICE, dtype=dtype)
    weight = torch.randn(channels, device=DEVICE, dtype=dtype)
    bias = torch.randn(channels, device=DEVICE, dtype=dtype)

    actual = triton_group_norm_silu(x, weight, bias, num_groups=num_groups)
    expected = _reference(x, weight, bias, num_groups)

    atol, rtol = _tol(dtype)
    torch.testing.assert_close(actual, expected, atol=atol, rtol=rtol)


@torch.no_grad()
@pytest.mark.parametrize("shape,num_groups", TEST_CASES[:2])
@pytest.mark.parametrize("dtype", [torch.float16, torch.bfloat16])
```
**EN:** This block defines `test_triton_group_norm_silu`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_triton_group_norm_silu`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 71-89: Function `test_apply_group_norm_silu`
```python
def test_apply_group_norm_silu(
    shape: tuple[int, ...],
    num_groups: int,
    dtype: torch.dtype,
) -> None:
    norm = nn.GroupNorm(num_groups, shape[1], eps=1e-5, affine=True).to(
        device=DEVICE, dtype=dtype
    )
    activation = nn.SiLU()
    hidden_states = torch.randn(shape, device=DEVICE, dtype=dtype)

    actual = apply_group_norm_silu(hidden_states, norm, activation)
    expected = activation(norm(hidden_states))

    atol, rtol = _tol(dtype)
    torch.testing.assert_close(actual, expected, atol=atol, rtol=rtol)


@torch.no_grad()
```
**EN:** This block defines `test_apply_group_norm_silu`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_apply_group_norm_silu`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 90-104: Function `test_triton_group_norm_silu_large_tile_bf16`
```python
def test_triton_group_norm_silu_large_tile_bf16() -> None:
    shape, num_groups = LARGE_TILE_CASE
    x = torch.randn(shape, device=DEVICE, dtype=torch.bfloat16)
    weight = torch.randn(shape[1], device=DEVICE, dtype=torch.bfloat16)
    bias = torch.randn(shape[1], device=DEVICE, dtype=torch.bfloat16)

    actual = triton_group_norm_silu(x, weight, bias, num_groups=num_groups)
    expected = _reference(x, weight, bias, num_groups)

    atol, rtol = _tol(torch.bfloat16)
    torch.testing.assert_close(actual, expected, atol=atol, rtol=rtol)


if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v", "-s"]))
```
**EN:** This block defines `test_triton_group_norm_silu_large_tile_bf16`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_triton_group_norm_silu_large_tile_bf16`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `sys`
- `pytest`
- `torch`
- `torch.nn as nn`
- `torch.nn.functional as F`
- `sglang.jit_kernel.diffusion.group_norm_silu -> apply_group_norm_silu`
- `sglang.jit_kernel.diffusion.triton.group_norm_silu -> triton_group_norm_silu`
- `sglang.test.ci.ci_register -> register_cuda_ci`
