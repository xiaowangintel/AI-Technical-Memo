# test_fp8_blockwise_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_fp8_blockwise_gemm.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `FP8 blockwise GEMM` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `FP8 blockwise GEMM` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Imports and module setup
````python
import os
import random
import sys
from typing import Optional, Type

import pytest
import torch
from sgl_kernel import fp8_blockwise_scaled_mm
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 11-12: `cdiv` definition
````python
def cdiv(a: int, b: int) -> int:
    return -(a // -b)
````
**EN:** This section defines `cdiv` and implements the core logic associated with cdiv.
**CN:** 该部分定义 `cdiv`，并实现与 cdiv 相关的核心逻辑。

### Lines 15-17: `scale_shape` definition
````python
def scale_shape(shape, group_shape):
    assert len(shape) == len(group_shape)
    return tuple(cdiv(shape[i], group_shape[i]) for i in range(len(group_shape)))
````
**EN:** This section defines `scale_shape` and implements the core logic associated with scale shape. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `scale_shape`，并实现与 scale shape 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。

### Lines 20-60: `baseline_scaled_mm` definition
````python
def baseline_scaled_mm(
    a: torch.Tensor,
    b: torch.Tensor,
    scale_a: torch.Tensor,
    scale_b: torch.Tensor,
    out_dtype: Type[torch.dtype],
    bias: Optional[torch.Tensor] = None,
) -> torch.Tensor:
    # We treat N-dimensional group scaling as extended numpy-style broadcasting
    # in numpy simply stretches dimensions with an extent of 1 to match the
    # the target shape by repeating the data along that dimension (broadcasting)
    # , we extend these semantics to say if the extent of a dimension in the
    # source shape is not 1 and does not match the target shape we repeat each
    # element along that dimension src_shape[dim] // target_shape[dim] times
    # example if we have:
    #       a = [[1, 2], and target_shape = (2, 4)
    #            [3, 4]]
    # then we would expand a to:
    #       a = [[1, 1, 2, 2],
    #            [3, 3, 4, 4]]
    # NOTE this function this function does not explicitly broadcast dimensions
    # with an extent of 1, since this can be done implicitly by pytorch
    def group_broadcast(t, shape):
        for i, s in enumerate(shape):
            if t.shape[i] != s and t.shape[i] != 1:
                assert s % t.shape[i] == 0
                t = (
                    t.unsqueeze(i + 1)
                    .expand(*t.shape[: i + 1], s // t.shape[i], *t.shape[i + 1 :])
                    .flatten(i, i + 1)
                )
        return t

    scale_a = group_broadcast(scale_a, a.shape)
    scale_b = group_broadcast(scale_b, b.shape)
    output = torch.mm(
        (scale_a * a.to(dtype=torch.float32)), (scale_b * b.to(dtype=torch.float32))
    ).to(out_dtype)
    if bias is not None:
        output = output + bias
    return output
````
**EN:** This section defines `baseline_scaled_mm` and implements the core logic associated with baseline scaled mm. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `baseline_scaled_mm`，并实现与 baseline scaled mm 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。

### Lines 63-82: `_test_accuracy_once` definition
````python
def _test_accuracy_once(M, N, K, out_dtype, device):
    fp8_info = torch.finfo(torch.float8_e4m3fn)
    fp8_max, fp8_min = fp8_info.max, fp8_info.min
    a_fp32 = (torch.rand(M, K, dtype=torch.float32, device=device) - 0.5) * 2 * fp8_max
    a_fp8 = a_fp32.clamp(min=fp8_min, max=fp8_max).to(torch.float8_e4m3fn)
    b_fp32 = (torch.rand(N, K, dtype=torch.float32, device=device) - 0.5) * 2 * fp8_max
    b_fp8 = b_fp32.clamp(min=fp8_min, max=fp8_max).to(torch.float8_e4m3fn).t()
    scale_a_group_shape = (1, 128)
    scale_b_group_shape = (128, 128)
    scale_a_shape = scale_shape(a_fp8.shape, scale_a_group_shape)
    scale_b_shape = scale_shape(b_fp8.shape, scale_b_group_shape)
    scale_a = torch.randn(scale_a_shape, device=device, dtype=torch.float32) * 0.001
    scale_b = torch.randn(scale_b_shape, device=device, dtype=torch.float32) * 0.001
    scale_a = scale_a.t().contiguous().t()
    scale_b = scale_b.t().contiguous().t()
    o = baseline_scaled_mm(a_fp8, b_fp8, scale_a, scale_b, out_dtype)
    o1 = fp8_blockwise_scaled_mm(a_fp8, b_fp8, scale_a, scale_b, out_dtype)
    rtol = 0.02
    atol = 1
    torch.testing.assert_close(o, o1, rtol=rtol, atol=atol)
````
**EN:** This section defines `_test_accuracy_once` and implements the core logic associated with test accuracy once.
**CN:** 该部分定义 `_test_accuracy_once`，并实现与 test accuracy once 相关的核心逻辑。

### Lines 85-90: `test_accuracy` definition
````python
@pytest.mark.parametrize("M", [1, 3, 5, 127, 128, 512, 1024, 4096])
@pytest.mark.parametrize("N", [128, 512, 1024, 4096, 8192, 14080])
@pytest.mark.parametrize("K", [512, 1024, 4096, 8192, 14080, 16384])
@pytest.mark.parametrize("out_dtype", [torch.bfloat16, torch.float16])
def test_accuracy(M, N, K, out_dtype):
    _test_accuracy_once(M, N, K, out_dtype, "cuda")
````
**EN:** This section defines the test `test_accuracy`. It sets up inputs, runs the target path, and checks the expected result. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_accuracy`。它会准备输入、执行目标路径，并检查预期结果。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 93-94: Command-line entry point
````python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `cdiv`, `scale_shape`, `baseline_scaled_mm`, `_test_accuracy_once`, `test_accuracy`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`
- **External / 外部**: `os`, `pytest`, `random`, `sys`, `torch`, `typing`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
