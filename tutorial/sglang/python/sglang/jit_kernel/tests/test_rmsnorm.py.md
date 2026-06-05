# test_rmsnorm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/test_rmsnorm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import itertools". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import itertools”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and module setup
```python
import itertools
import sys

import pytest
import torch

from sglang.jit_kernel.utils import get_ci_test_range
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=45, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=240, suite="nightly-kernel-1-gpu", nightly=True)


EPS = 1e-6
DEVICE = "cuda"
DTYPES = [torch.float16, torch.bfloat16]
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 19-30: Imports and module setup
```python
def sglang_jit_rmsnorm(
    input: torch.Tensor,
    weight: torch.Tensor,
    *,
    output: torch.Tensor | None = None,
    eps: float = EPS,
) -> None:
    from sglang.jit_kernel.norm import rmsnorm

    rmsnorm(input, weight, out=output, eps=eps)
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 31-57: Imports and module setup
```python
def flashinfer_rmsnorm(
    input: torch.Tensor,
    weight: torch.Tensor,
    *,
    output: torch.Tensor,
    eps: float = EPS,
) -> None:
    from flashinfer.norm import rmsnorm

    rmsnorm(input, weight, out=output, eps=eps)


BS_LIST = [2**n for n in range(0, 14)]
BS_LIST += [x + 1 + i for i, x in enumerate(BS_LIST)]
BS_LIST = get_ci_test_range(BS_LIST, [1, 9, 256, 4109])
SUPPORTED_HIDDEN_SIZE_LIST = get_ci_test_range(
    [64, 128, 256, 512, *range(1024, 8192 + 1, 1024), 2304, 2560, 12288, 16384],
    [256, 1024, 16384],
)


@pytest.mark.parametrize(
    "batch_size,hidden_size",
    list(itertools.product(BS_LIST, SUPPORTED_HIDDEN_SIZE_LIST)),
)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("specify_out", [True, False])
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 58-78: Function `test_rmsnorm`
```python
def test_rmsnorm(
    batch_size: int, hidden_size: int, dtype: torch.dtype, specify_out: bool
) -> None:
    input = torch.randn(batch_size, hidden_size, device=DEVICE, dtype=dtype)
    weight = torch.randn(hidden_size, device=DEVICE, dtype=dtype)

    input_flashinfer = input.clone()
    output_flashinfer = torch.empty_like(input)
    flashinfer_rmsnorm(input_flashinfer, weight, output=output_flashinfer)

    if specify_out:
        output_sglang = torch.empty_like(input)
        sglang_jit_rmsnorm(input, weight, output=output_sglang)
    else:
        output_sglang = input.clone()
        sglang_jit_rmsnorm(output_sglang, weight, output=output_sglang)

    torch.testing.assert_close(output_sglang, output_flashinfer, atol=1e-2, rtol=1e-2)


@pytest.mark.parametrize("hidden_size", [64, 128, 256, 512, 8192, 8704, 16384])
```
**EN:** This block defines `test_rmsnorm`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_rmsnorm`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 79-99: Imports and module setup
```python
def test_rmsnorm_hidden_size_support(hidden_size: int) -> None:
    from sglang.jit_kernel.norm import _is_supported_rmsnorm_hidden_size

    assert _is_supported_rmsnorm_hidden_size(hidden_size)


@pytest.mark.parametrize(
    ("hidden_size", "expected"),
    [
        (64, "RMSNormWarpKernel"),
        (128, "RMSNormWarpKernel"),
        (256, "RMSNormWarpKernel"),
        (512, "RMSNormKernel"),
        (1536, "RMSNormKernel"),
        (2048, "RMSNormHalfKernel"),
        (2304, "RMSNormKernel"),  # NOTE: not 512 aligned
        (8192, "RMSNormHalfKernel"),
        (8704, "RMSNormHalfKernel"),
        (16384, "RMSNormHalfKernel"),
    ],
)
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 100-107: Imports and module setup
```python
def test_rmsnorm_kernel_dispatch(hidden_size: int, expected: str) -> None:
    from sglang.jit_kernel.norm import _rmsnorm_kernel_class

    assert _rmsnorm_kernel_class(hidden_size) == expected


if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v", "-s"]))
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `itertools`
- `sys`
- `pytest`
- `torch`
- `sglang.jit_kernel.utils -> get_ci_test_range`
- `sglang.test.ci.ci_register -> register_cuda_ci`
- `sglang.jit_kernel.norm -> rmsnorm`
- `flashinfer.norm -> rmsnorm`
- `sglang.jit_kernel.norm -> _is_supported_rmsnorm_hidden_size`
- `sglang.jit_kernel.norm -> _rmsnorm_kernel_class`
