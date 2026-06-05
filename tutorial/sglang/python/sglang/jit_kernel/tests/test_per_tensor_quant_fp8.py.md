# test_per_tensor_quant_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/test_per_tensor_quant_fp8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import itertools". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import itertools”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Imports and module setup
```python
import itertools
import sys
from typing import Optional, Tuple

import pytest
import torch

from sglang.jit_kernel.per_tensor_quant_fp8 import per_tensor_quant_fp8
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=16, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=120, suite="nightly-kernel-1-gpu", nightly=True)

try:
    from sglang.srt.utils import is_hip

    _is_hip = is_hip()
except ImportError:
    _is_hip = False

fp8_type_ = torch.float8_e4m3fnuz if _is_hip else torch.float8_e4m3fn
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 24-38: Function `sglang_scaled_fp8_quant`
```python
def sglang_scaled_fp8_quant(
    input: torch.Tensor,
    scale: Optional[torch.Tensor] = None,
) -> Tuple[torch.Tensor, torch.Tensor]:
    fp8_type_: torch.dtype = torch.float8_e4m3fn
    output = torch.empty_like(input, device=input.device, dtype=fp8_type_)
    is_static = True
    if scale is None:
        scale = torch.zeros(1, device=input.device, dtype=torch.float32)
        is_static = False
    per_tensor_quant_fp8(input, output, scale, is_static)

    return output, scale
```
**EN:** This block defines `sglang_scaled_fp8_quant`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `sglang_scaled_fp8_quant`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 39-50: Function `torch_scaled_fp8_quant`
```python
def torch_scaled_fp8_quant(tensor, inv_scale):
    finfo = torch.finfo(torch.float8_e4m3fn)
    scale = inv_scale.reciprocal()
    qweight = (tensor.to(torch.float32) * scale).clamp(min=finfo.min, max=finfo.max)
    qweight = qweight.to(torch.float8_e4m3fn)
    return qweight


@pytest.mark.parametrize(
    "num_tokens,hidden_dim",
    list(itertools.product([128, 256, 512], [512, 2048, 4096])),
)
```
**EN:** This block defines `torch_scaled_fp8_quant`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `torch_scaled_fp8_quant`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 51-66: Function `test_jit_per_tensor_quant_compare_implementations`
```python
def test_jit_per_tensor_quant_compare_implementations(
    num_tokens: int,
    hidden_dim: int,
):
    device = torch.device("cuda")
    x = torch.rand((num_tokens, hidden_dim), dtype=torch.float16, device=device)

    sglang_out, sglang_scale = sglang_scaled_fp8_quant(x)
    torch_out = torch_scaled_fp8_quant(x, sglang_scale)

    torch.testing.assert_close(
        sglang_out.float(), torch_out.float(), rtol=1e-3, atol=1e-3
    )


@pytest.mark.parametrize("shape", [(4, 8, 64), (2, 16, 128), (19260817, 1, 1)])
```
**EN:** This block defines `test_jit_per_tensor_quant_compare_implementations`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_jit_per_tensor_quant_compare_implementations`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 67-91: Function `test_jit_per_tensor_quant_supports_3d`
```python
def test_jit_per_tensor_quant_supports_3d(shape):
    device = torch.device("cuda")
    x = torch.rand(shape, dtype=torch.bfloat16, device=device)
    out = torch.empty_like(x, device=x.device, dtype=fp8_type_)
    scale = torch.zeros(1, device=x.device, dtype=torch.float32)

    per_tensor_quant_fp8(x, out, scale, is_static=False)

    x_2d = x.flatten(0, -2)
    out_ref_2d = torch_scaled_fp8_quant(x_2d, scale)
    out_ref = out_ref_2d.reshape(shape)

    torch.testing.assert_close(out.float(), out_ref.float(), rtol=1e-3, atol=1e-3)

    scale = torch.rand(1, dtype=torch.float32, device=device)
    sglang_out, _ = sglang_scaled_fp8_quant(x, scale)
    torch_out = torch_scaled_fp8_quant(x, scale)

    torch.testing.assert_close(
        sglang_out.float(), torch_out.float(), rtol=1e-3, atol=1e-3
    )


if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v", "-s"]))
```
**EN:** This block defines `test_jit_per_tensor_quant_supports_3d`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_jit_per_tensor_quant_supports_3d`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `itertools`
- `sys`
- `typing -> Optional`
- `pytest`
- `torch`
- `sglang.jit_kernel.per_tensor_quant_fp8 -> per_tensor_quant_fp8`
- `sglang.test.ci.ci_register -> register_cuda_ci`
- `sglang.srt.utils -> is_hip`
