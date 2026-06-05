# test_per_token_quant_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_per_token_quant_fp8.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `per token quantization FP8` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `per token quantization FP8` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Imports and module setup
````python
import itertools
import sys
from typing import Optional, Tuple

import pytest
import torch
from sgl_kernel import sgl_per_token_quant_fp8

from sglang.srt.utils import is_hip
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 11-12: Constants and configuration
````python
_is_hip = is_hip()
fp8_type_ = torch.float8_e4m3fnuz if _is_hip else torch.float8_e4m3fn
````
**EN:** This block defines shared constants or configuration values such as `_is_hip`, `fp8_type_`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `_is_hip`, `fp8_type_`），供后续函数或控制流程复用。

### Lines 15-23: `torch_per_token_quant_fp8` definition
````python
def torch_per_token_quant_fp8(tensor, inv_scale):
    # The reference implementation that fully aligns to
    # the kernel being tested.
    finfo = torch.finfo(torch.float8_e4m3fn)
    inv_scale = inv_scale.view(-1, 1)
    scale = inv_scale.reciprocal()
    qweight = (tensor.to(torch.float32) * scale).clamp(min=finfo.min, max=finfo.max)
    qweight = qweight.to(torch.float8_e4m3fn)
    return qweight
````
**EN:** This section defines `torch_per_token_quant_fp8` and implements the core logic associated with torch per token quantization FP8.
**CN:** 该部分定义 `torch_per_token_quant_fp8`，并实现与 torch per token quantization FP8 相关的核心逻辑。

### Lines 26-35: `sglang_per_token_quant_fp8` definition
````python
def sglang_per_token_quant_fp8(
    input: torch.Tensor,
) -> Tuple[torch.Tensor, torch.Tensor]:
    scale = torch.zeros(input.size(0), device=input.device, dtype=torch.float32)
    output = torch.empty_like(input, device=input.device, dtype=fp8_type_)

    sgl_per_token_quant_fp8(input, output, scale)
    scale = scale.reshape(-1, 1)

    return output, scale
````
**EN:** This section defines `sglang_per_token_quant_fp8` and implements the core logic associated with sglang per token quantization FP8.
**CN:** 该部分定义 `sglang_per_token_quant_fp8`，并实现与 sglang per token quantization FP8 相关的核心逻辑。

### Lines 38-54: `test_per_token_quant_compare_implementations` definition
````python
@pytest.mark.parametrize(
    "num_tokens,hidden_dim",
    list(itertools.product([128, 256, 512], [512, 1076, 1368, 2048, 4096])),
)
def test_per_token_quant_compare_implementations(
    num_tokens: int,
    hidden_dim: int,
):
    device = torch.device("cuda")
    x = torch.rand((num_tokens, hidden_dim), dtype=torch.float16, device=device)

    sglang_out, sglang_scale = sglang_per_token_quant_fp8(x)
    torch_out = torch_per_token_quant_fp8(x, sglang_scale)

    torch.testing.assert_close(
        sglang_out.float(), torch_out.float(), rtol=1e-3, atol=1e-3
    )
````
**EN:** This section defines the test `test_per_token_quant_compare_implementations`. It sets up inputs, runs the target path, and checks the expected result. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_per_token_quant_compare_implementations`。它会准备输入、执行目标路径，并检查预期结果。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 57-58: Command-line entry point
````python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `torch_per_token_quant_fp8`, `sglang_per_token_quant_fp8`, `test_per_token_quant_compare_implementations`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`, `sglang.srt.utils`
- **External / 外部**: `itertools`, `pytest`, `sys`, `torch`, `typing`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
