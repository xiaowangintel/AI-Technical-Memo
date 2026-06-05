# test_int8_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_int8_gemm.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `INT8 GEMM` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `INT8 GEMM` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Imports and module setup
````python
import sys

import pytest
import torch
from sgl_kernel import int8_scaled_mm
from utils import is_sm10x
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 9-10: `to_int8` definition
````python
def to_int8(tensor: torch.Tensor) -> torch.Tensor:
    return torch.round(tensor.clamp(min=-128, max=127)).to(dtype=torch.int8)
````
**EN:** This section defines `to_int8` and implements the core logic associated with to INT8.
**CN:** 该部分定义 `to_int8`，并实现与 to INT8 相关的核心逻辑。

### Lines 13-19: `torch_scaled_mm` definition
````python
def torch_scaled_mm(a, b, scale_a, scale_b, out_dtype, bias):
    o = torch.matmul(a.to(torch.float32), b.to(torch.float32))
    if bias is not None:
        o = o.to(torch.float32) * scale_a.view(-1, 1) * scale_b.view(1, -1) + bias
    else:
        o = o.to(torch.float32) * scale_a.view(-1, 1) * scale_b.view(1, -1)
    return o.to(out_dtype)
````
**EN:** This section defines `torch_scaled_mm` and implements the core logic associated with torch scaled mm.
**CN:** 该部分定义 `torch_scaled_mm`，并实现与 torch scaled mm 相关的核心逻辑。

### Lines 22-33: `_test_accuracy_once` definition
````python
def _test_accuracy_once(M, N, K, with_bias, out_dtype, device):
    a = to_int8(torch.randn((M, K), device=device) * 5)
    b = to_int8(torch.randn((N, K), device=device).t() * 5)
    scale_a = torch.randn((M,), device="cuda", dtype=torch.float32)
    scale_b = torch.randn((N,), device="cuda", dtype=torch.float32)
    if with_bias:
        bias = torch.randn((N,), device="cuda", dtype=out_dtype) * 10
    else:
        bias = None
    o = int8_scaled_mm(a, b, scale_a, scale_b, out_dtype, bias)
    o1 = torch_scaled_mm(a, b, scale_a, scale_b, out_dtype, bias)
    torch.testing.assert_close(o, o1)
````
**EN:** This section defines `_test_accuracy_once` and implements the core logic associated with test accuracy once.
**CN:** 该部分定义 `_test_accuracy_once`，并实现与 test accuracy once 相关的核心逻辑。

### Lines 36-46: `test_accuracy` definition
````python
@pytest.mark.skipif(
    is_sm10x(),
    reason="int8_scaled_mm is only supported on sm90 and lower",
)
@pytest.mark.parametrize("M", [1, 16, 32, 64, 128, 512, 1024, 4096, 8192])
@pytest.mark.parametrize("N", [16, 128, 512, 1024, 4096, 8192, 16384])
@pytest.mark.parametrize("K", [512, 1024, 4096, 8192, 16384])
@pytest.mark.parametrize("with_bias", [True, False])
@pytest.mark.parametrize("out_dtype", [torch.float16, torch.bfloat16])
def test_accuracy(M, N, K, with_bias, out_dtype):
    _test_accuracy_once(M, N, K, with_bias, out_dtype, "cuda")
````
**EN:** This section defines the test `test_accuracy`. It sets up inputs, runs the target path, and checks the expected result. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_accuracy`。它会准备输入、执行目标路径，并检查预期结果。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 49-50: Command-line entry point
````python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `to_int8`, `torch_scaled_mm`, `_test_accuracy_once`, `test_accuracy`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`
- **External / 外部**: `pytest`, `sys`, `torch`, `utils`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
