# test_awq_dequant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_awq_dequant.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `AWQ dequantization` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `AWQ dequantization` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Imports and module setup
````python
import itertools
import sys
from typing import Optional, Tuple

import pytest
import torch
from sgl_kernel import awq_dequantize
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 10-23: `reverse_awq_order` definition
````python
def reverse_awq_order(t: torch.Tensor):
    bits = 4
    AWQ_REVERSE_ORDER = [0, 4, 1, 5, 2, 6, 3, 7]
    reverse_order_tensor = torch.arange(
        t.shape[-1],
        dtype=torch.int32,
        device=t.device,
    )
    reverse_order_tensor = reverse_order_tensor.view(-1, 32 // bits)
    reverse_order_tensor = reverse_order_tensor[:, AWQ_REVERSE_ORDER]
    reverse_order_tensor = reverse_order_tensor.view(-1)

    t = t[:, reverse_order_tensor] & 0xF
    return t
````
**EN:** This section defines `reverse_awq_order` and implements the core logic associated with reverse AWQ order.
**CN:** 该部分定义 `reverse_awq_order`，并实现与 reverse AWQ order 相关的核心逻辑。

### Lines 24-28: Comments and local context
````python


# qweights - [R     , C // 8], int32
# scales   - [R // G, C     ], float16
# zeros    - [R // G, C // 8], int32
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 29-58: `awq_dequantize_torch` definition
````python
def awq_dequantize_torch(
    qweight: torch.Tensor, scales: torch.Tensor, qzeros: torch.Tensor, group_size: int
) -> torch.Tensor:

    if group_size == -1:
        group_size = qweight.shape[0]

    bits = 4
    shifts = torch.arange(0, 32, bits, device=qzeros.device)

    iweights = torch.bitwise_right_shift(qweight[:, :, None], shifts[None, None, :]).to(
        torch.int8
    )

    iweights = iweights.view(iweights.shape[0], -1)

    zeros = torch.bitwise_right_shift(qzeros[:, :, None], shifts[None, None, :]).to(
        torch.int8
    )
    zeros = zeros.view(qzeros.shape[0], -1)
    zeros = reverse_awq_order(zeros)

    iweights = reverse_awq_order(iweights)

    iweights = torch.bitwise_and(iweights, (2**bits) - 1)
    zeros = torch.bitwise_and(zeros, (2**bits) - 1)

    scales = scales.repeat_interleave(group_size, dim=0)
    zeros = zeros.repeat_interleave(group_size, dim=0)
    return (iweights - zeros) * scales
````
**EN:** This section defines `awq_dequantize_torch` and implements the core logic associated with AWQ dequantize torch.
**CN:** 该部分定义 `awq_dequantize_torch`，并实现与 AWQ dequantize torch 相关的核心逻辑。

### Lines 61-64: `sglang_awq_dequantize` definition
````python
def sglang_awq_dequantize(
    qweight: torch.Tensor, scales: torch.Tensor, qzeros: torch.Tensor
) -> torch.Tensor:
    return awq_dequantize(qweight, scales, qzeros)
````
**EN:** This section defines `sglang_awq_dequantize` and implements the core logic associated with sglang AWQ dequantize.
**CN:** 该部分定义 `sglang_awq_dequantize`，并实现与 sglang AWQ dequantize 相关的核心逻辑。

### Lines 67-112: `test_awq_dequant_compare_implementations` definition
````python
@pytest.mark.parametrize(
    "qweight_row,qweight_col,is_bf16_act",
    list(
        itertools.product(
            [3584, 18944, 128, 256, 512, 1024, 1536],
            [448, 576, 4736, 16, 32, 64, 128, 72],
            [True, False],
        )
    ),
)
def test_awq_dequant_compare_implementations(
    qweight_row: int, qweight_col: int, is_bf16_act: bool
):
    device = torch.device("cuda")
    qweight = torch.randint(
        0,
        torch.iinfo(torch.int32).max,
        (qweight_row, qweight_col),
        dtype=torch.int32,
        device=device,
    )
    group_size = qweight_row
    scales_row = qweight_row // group_size
    scales_col = qweight_col * 8

    if is_bf16_act:
        scales = torch.rand(scales_row, scales_col, dtype=torch.bfloat16, device=device)
    else:
        scales = torch.rand(scales_row, scales_col, dtype=torch.float16, device=device)

    qzeros = torch.randint(
        0,
        torch.iinfo(torch.int32).max,
        (scales_row, qweight_col),
        dtype=torch.int32,
        device=device,
    )

    # Run both implementations
    torch_out = awq_dequantize_torch(qweight, scales, qzeros, group_size)
    sglang_out = sglang_awq_dequantize(qweight, scales, qzeros)

    # Compare results
    torch.testing.assert_close(
        torch_out.to(torch.float32), sglang_out.to(torch.float32), rtol=1e-3, atol=1e-5
    )
````
**EN:** This section defines the test `test_awq_dequant_compare_implementations`. It sets up inputs, runs the target path, and checks the expected result. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_awq_dequant_compare_implementations`。它会准备输入、执行目标路径，并检查预期结果。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 115-116: Command-line entry point
````python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `reverse_awq_order`, `awq_dequantize_torch`, `sglang_awq_dequantize`, `test_awq_dequant_compare_implementations`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`
- **External / 外部**: `itertools`, `pytest`, `sys`, `torch`, `typing`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
