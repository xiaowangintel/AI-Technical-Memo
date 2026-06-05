# test_awq_dequantize.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/test_awq_dequantize.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import itertools". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import itertools”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Imports and module setup
```python
import itertools
import sys

import pytest
import torch

from sglang.jit_kernel.awq_dequantize import awq_dequantize as jit_awq_dequantize
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=9, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=120, suite="nightly-kernel-1-gpu", nightly=True)

try:
    from sgl_kernel import awq_dequantize as aot_awq_dequantize

    AOT_AVAILABLE = True
except ImportError:
    AOT_AVAILABLE = False
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 21-39: Function `reverse_awq_order`
```python
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


# qweights - [R     , C // 8], int32
# scales   - [R // G, C     ], float16
# zeros    - [R // G, C // 8], int32
```
**EN:** This block defines `reverse_awq_order`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `reverse_awq_order`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 40-80: Function `awq_dequantize_torch`
```python
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


@pytest.mark.parametrize(
    "qweight_row,qweight_col,is_bf16_act",
    list(
        itertools.product(
            [128, 256, 512, 1024, 3584],
            [16, 32, 64, 128, 448],
            [True, False],
        )
    ),
)
```
**EN:** This block defines `awq_dequantize_torch`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `awq_dequantize_torch`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 81-128: Function `test_awq_dequantize_jit_vs_torch`
```python
def test_awq_dequantize_jit_vs_torch(
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
    jit_out = jit_awq_dequantize(qweight, scales, qzeros)

    # Compare results (approximate due to different computation paths)
    torch.testing.assert_close(
        torch_out.to(torch.float32), jit_out.to(torch.float32), rtol=1e-3, atol=1e-5
    )


@pytest.mark.parametrize(
    "qweight_row,qweight_col,is_bf16_act",
    list(
        itertools.product(
            [128, 256, 512, 1024, 3584],
            [16, 32, 64, 128, 448],
            [True, False],
        )
    ),
)
```
**EN:** This block defines `test_awq_dequantize_jit_vs_torch`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_awq_dequantize_jit_vs_torch`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 129-169: Function `test_awq_dequantize_jit_vs_aot`
```python
def test_awq_dequantize_jit_vs_aot(
    qweight_row: int, qweight_col: int, is_bf16_act: bool
):
    if not AOT_AVAILABLE:
        pytest.skip("sgl_kernel AOT not available")

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
    aot_out = aot_awq_dequantize(qweight, scales, qzeros)
    jit_out = jit_awq_dequantize(qweight, scales, qzeros)

    # Bitwise equality
    torch.testing.assert_close(jit_out, aot_out, rtol=0, atol=0)


if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v", "-s"]))
```
**EN:** This block defines `test_awq_dequantize_jit_vs_aot`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_awq_dequantize_jit_vs_aot`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `itertools`
- `sys`
- `pytest`
- `torch`
- `sglang.jit_kernel.awq_dequantize -> awq_dequantize`
- `sglang.test.ci.ci_register -> register_cuda_ci`
- `sgl_kernel -> awq_dequantize`
