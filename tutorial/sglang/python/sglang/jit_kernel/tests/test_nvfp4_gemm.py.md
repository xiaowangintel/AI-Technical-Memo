# test_nvfp4_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/test_nvfp4_gemm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import sys". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import sys”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Imports and module setup
```python
import sys

import pytest
import torch

from sglang.jit_kernel.nvfp4 import cutlass_scaled_fp4_mm, scaled_fp4_quant
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=5, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=120, suite="nightly-kernel-1-gpu", nightly=True)
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 13-40: Function `_nvfp4_supported`
```python
def _nvfp4_supported() -> bool:
    return torch.cuda.is_available() and torch.cuda.get_device_capability() >= (10, 0)


DTYPES = [torch.float16, torch.bfloat16]
SHAPES = [
    (128, 128, 64),
    (128, 128, 128),
    (256, 128, 64),
    (128, 256, 128),
    (150, 128, 64),
]

FLOAT4_E2M1_MAX = 6.0
FLOAT8_E4M3_MAX = torch.finfo(torch.float8_e4m3fn).max

K_E2M1_TO_FLOAT = [
    0.0,
    0.5,
    1.0,
    1.5,
    2.0,
    3.0,
    4.0,
    6.0,
]
```
**EN:** This block defines `_nvfp4_supported`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_nvfp4_supported`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 41-47: Function `e2m1_to_fp32`
```python
def e2m1_to_fp32(int4_value: int) -> float:
    sign_bit = int4_value & 0x8
    int4_abs_value = int4_value & 0x7
    float_result = K_E2M1_TO_FLOAT[int4_abs_value]
    return -float_result if sign_bit else float_result
```
**EN:** This block defines `e2m1_to_fp32`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `e2m1_to_fp32`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 48-58: Function `break_fp4_bytes`
```python
def break_fp4_bytes(a: torch.Tensor) -> torch.Tensor:
    assert a.dtype == torch.uint8
    m, n = a.shape
    a = a.flatten()
    high_half_byte = (a & 0xF0) >> 4
    low_half_byte = a & 0x0F
    f_h = torch.tensor([e2m1_to_fp32(x) for x in high_half_byte], device=a.device)
    f_l = torch.tensor([e2m1_to_fp32(x) for x in low_half_byte], device=a.device)
    return torch.stack((f_l, f_h), dim=-1).reshape(m, n * 2)
```
**EN:** This block defines `break_fp4_bytes`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `break_fp4_bytes`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 59-72: Function `convert_swizzled_to_linear`
```python
def convert_swizzled_to_linear(
    a_sf_swizzled: torch.Tensor, m: int, k: int, block_size: int
) -> torch.Tensor:
    sf_m, sf_k = a_sf_swizzled.shape
    del sf_m, sf_k
    m_tiles = (m + 128 - 1) // 128
    f = block_size * 4
    k_tiles = (k + f - 1) // f
    tmp = torch.reshape(a_sf_swizzled, (1, m_tiles, k_tiles, 32, 4, 4))
    tmp = torch.permute(tmp, (0, 1, 4, 3, 2, 5))
    out = tmp.reshape(m_tiles * 128, k_tiles * f // block_size)
    return out[0:m, 0 : k // block_size]
```
**EN:** This block defines `convert_swizzled_to_linear`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `convert_swizzled_to_linear`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 73-89: Function `dequantize_to_dtype`
```python
def dequantize_to_dtype(
    tensor_fp4: torch.Tensor,
    tensor_sf: torch.Tensor,
    global_scale: torch.Tensor,
    block_size: int = 16,
) -> torch.Tensor:
    assert tensor_fp4.dtype == torch.uint8
    m, packed_k = tensor_fp4.shape
    k = packed_k * 2
    tensor_f32 = break_fp4_bytes(tensor_fp4)
    tensor_f32 = tensor_f32.reshape(m, k // block_size, block_size)
    tensor_sf = tensor_sf.view(torch.float8_e4m3fn)
    tensor_sf = convert_swizzled_to_linear(tensor_sf, m, k, block_size)
    tensor_sf_dtype = tensor_sf.to(torch.float32) / global_scale
    return (tensor_f32 * tensor_sf_dtype.unsqueeze(-1)).reshape(m, k)
```
**EN:** This block defines `dequantize_to_dtype`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `dequantize_to_dtype`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 90-108: Function `get_ref_results`
```python
def get_ref_results(
    a_fp4: torch.Tensor,
    b_fp4: torch.Tensor,
    a_sf: torch.Tensor,
    b_sf: torch.Tensor,
    a_global_scale: torch.Tensor,
    b_global_scale: torch.Tensor,
    block_size: int,
) -> torch.Tensor:
    a_in_dtype = dequantize_to_dtype(a_fp4, a_sf, a_global_scale, block_size=block_size)
    b_in_dtype = dequantize_to_dtype(b_fp4, b_sf, b_global_scale, block_size=block_size)
    return torch.matmul(a_in_dtype, b_in_dtype.t())


@pytest.mark.skipif(
    not _nvfp4_supported(), reason="NVFP4 requires compute capability >= 10.0"
)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("shape", SHAPES)
```
**EN:** This block defines `get_ref_results`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `get_ref_results`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 109-152: Function `test_nvfp4_gemm`
```python
def test_nvfp4_gemm(dtype: torch.dtype, shape: tuple[int, int, int]) -> None:
    m, n, packed_k = shape
    k = packed_k * 2
    block_size = 16

    a_dtype = torch.randn((m, k), dtype=dtype, device="cuda")
    b_dtype = torch.randn((n, k), dtype=dtype, device="cuda")

    a_global_scale = (
        (FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX) / torch.amax(a_dtype.flatten(), dim=-1)
    ).to(torch.float32)
    b_global_scale = (
        (FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX) / torch.amax(b_dtype.flatten(), dim=-1)
    ).to(torch.float32)

    alpha = 1.0 / (a_global_scale * b_global_scale)

    a_fp4, a_scale_interleaved = scaled_fp4_quant(a_dtype, a_global_scale)
    b_fp4, b_scale_interleaved = scaled_fp4_quant(b_dtype, b_global_scale)

    expected_out = get_ref_results(
        a_fp4,
        b_fp4,
        a_scale_interleaved,
        b_scale_interleaved,
        a_global_scale,
        b_global_scale,
        block_size,
    )

    out = cutlass_scaled_fp4_mm(
        a_fp4,
        b_fp4,
        a_scale_interleaved,
        b_scale_interleaved,
        alpha,
        dtype,
    )

    torch.testing.assert_close(out, expected_out.to(dtype=dtype), atol=1e-1, rtol=1e-1)


if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v", "-s"]))
```
**EN:** This block defines `test_nvfp4_gemm`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_nvfp4_gemm`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `sys`
- `pytest`
- `torch`
- `sglang.jit_kernel.nvfp4 -> cutlass_scaled_fp4_mm`
- `sglang.test.ci.ci_register -> register_cuda_ci`
