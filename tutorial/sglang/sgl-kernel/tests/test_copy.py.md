# test_copy.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_copy.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `copy` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `copy` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Imports and module setup
````python
import sys

import pytest
import sgl_kernel
import torch
from sgl_kernel.elementwise import copy_to_gpu_no_ce
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 9-14: `test_copy_to_gpu_no_ce` definition
````python
@pytest.mark.parametrize("size", [64, 72])
def test_copy_to_gpu_no_ce(size):
    tensor_cpu = torch.randint(0, 1000000, (size,), dtype=torch.int32, device="cpu")
    tensor_gpu = torch.empty_like(tensor_cpu, device="cuda")
    copy_to_gpu_no_ce(tensor_cpu, tensor_gpu)
    assert torch.all(tensor_cpu.cuda() == tensor_gpu)
````
**EN:** This section defines the test `test_copy_to_gpu_no_ce`. It sets up inputs, runs the target path, and checks the expected result. It also performs explicit assertions or shape checks before continuing. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_copy_to_gpu_no_ce`。它会准备输入、执行目标路径，并检查预期结果。 它还会在继续执行前进行显式断言或形状检查。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 17-18: Command-line entry point
````python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `test_copy_to_gpu_no_ce`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`, `sgl_kernel.elementwise`
- **External / 外部**: `pytest`, `sys`, `torch`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
