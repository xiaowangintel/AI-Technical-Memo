# test_fp8_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_fp8_gemm.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `FP8 GEMM` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `FP8 GEMM` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Imports and module setup
````python
import sys

import pytest
import torch
from sgl_kernel import fp8_scaled_mm
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 8-16: `torch_scaled_mm` definition
````python
def torch_scaled_mm(a, b, scale_a, scale_b, out_dtype, bias):
    o = torch.matmul(a.to(torch.float32), b.to(torch.float32))
    o = o.to(torch.float32)
    temp1 = o * scale_a.view(-1, 1)
    temp2 = temp1 * scale_b.view(1, -1)
    final = temp2.to(out_dtype)
    if bias is not None:
        final = final + bias.view(1, -1)
    return final
````
**EN:** This section defines `torch_scaled_mm` and implements the core logic associated with torch scaled mm.
**CN:** 该部分定义 `torch_scaled_mm`，并实现与 torch scaled mm 相关的核心逻辑。

### Lines 19-38: `_test_accuracy_once` definition
````python
def _test_accuracy_once(M, N, K, with_bias, out_dtype, device):
    fp8_info = torch.finfo(torch.float8_e4m3fn)
    fp8_max, fp8_min = fp8_info.max, fp8_info.min
    a_fp32 = (torch.rand(M, K, dtype=torch.float32, device=device) - 0.5) * 2 * fp8_max
    a_fp8 = a_fp32.clamp(min=fp8_min, max=fp8_max).to(torch.float8_e4m3fn)
    b_fp32 = (torch.rand(N, K, dtype=torch.float32, device=device) - 0.5) * 2 * fp8_max
    b_fp8 = b_fp32.clamp(min=fp8_min, max=fp8_max).to(torch.float8_e4m3fn)
    scale_a = torch.randn((M,), device=device, dtype=torch.float32) * 0.001
    scale_b = torch.randn((N,), device=device, dtype=torch.float32) * 0.001
    if with_bias:
        bias = torch.randn((N,), device=device, dtype=out_dtype)
    else:
        bias = None
    b_fp8 = b_fp8.t()
    o = torch_scaled_mm(a_fp8, b_fp8, scale_a, scale_b, out_dtype, bias)
    o1 = fp8_scaled_mm(a_fp8, b_fp8, scale_a, scale_b, out_dtype, bias)
    rtol = 0.02
    atol = 1
    torch.testing.assert_close(o, o1, rtol=rtol, atol=atol)
    print(f"M={M}, N={N}, K={K}, with_bias={with_bias}, out_dtype={out_dtype}: OK")
````
**EN:** This section defines `_test_accuracy_once` and implements the core logic associated with test accuracy once.
**CN:** 该部分定义 `_test_accuracy_once`，并实现与 test accuracy once 相关的核心逻辑。

### Lines 41-47: `test_accuracy` definition
````python
@pytest.mark.parametrize("M", [1, 128, 512, 1024, 4096])
@pytest.mark.parametrize("N", [16, 128, 512, 1024, 4096])
@pytest.mark.parametrize("K", [512, 1024, 4096, 8192, 16384])
@pytest.mark.parametrize("with_bias", [True, False])
@pytest.mark.parametrize("out_dtype", [torch.bfloat16, torch.float16])
def test_accuracy(M, N, K, with_bias, out_dtype):
    _test_accuracy_once(M, N, K, with_bias, out_dtype, "cuda")
````
**EN:** This section defines the test `test_accuracy`. It sets up inputs, runs the target path, and checks the expected result. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_accuracy`。它会准备输入、执行目标路径，并检查预期结果。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 50-51: Command-line entry point
````python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `torch_scaled_mm`, `_test_accuracy_once`, `test_accuracy`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`
- **External / 外部**: `pytest`, `sys`, `torch`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
