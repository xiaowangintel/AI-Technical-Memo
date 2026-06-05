# reference_mxfp4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/quantization/reference_mxfp4.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Reference Mxfp4 behavior in the Quantization test area through focused pytest scenarios. It focuses on scenarios such as E8m0 To Half, Upcast Fp4 To FP16 Or BF16, Dq Mxfp4 Torch. / 该文件在 Quantization 测试域中，通过有针对性的 pytest 场景验证 Reference Mxfp4 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-25)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import torch

BFLOAT16_EXP_BIAS = 127
BFLOAT16_MANTISSA_BITS = 7
BFLOAT16_EXP_BITS = 8

FLOAT16_EXP_BIAS = 15
FLOAT16_MANTISSA_BITS = 10
FLOAT16_EXP_BITS = 5

FLOAT8_E8M0_MAX_EXP = 127
FLOAT4_EXP_BIAS = 1
FLOAT4_MANTISSA_BITS = 1

FLOAT16_VAL_TO_ADD = 1 << (FLOAT16_MANTISSA_BITS - FLOAT4_MANTISSA_BITS - 1)
FLOAT16_SIGN_EXPONENT_MASK = (
    (1 << (FLOAT16_EXP_BITS + 1)) - 1
) << FLOAT16_MANTISSA_BITS

BFLOAT16_VAL_TO_ADD = 1 << (BFLOAT16_MANTISSA_BITS - FLOAT4_MANTISSA_BITS - 1)
BFLOAT16_SIGN_EXPONENT_MASK = (
    (1 << (BFLOAT16_EXP_BITS + 1)) - 1
) << BFLOAT16_MANTISSA_BITS
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `torch`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: e8m0_to_half (lines 28-36)
```python
def e8m0_to_half(scale, half_dtype: torch.dtype):
    assert scale.dtype == torch.uint8

    scale_exp = scale.to(torch.int16) - 127

    # This can be implemented with bitwise operations in a proper kernel.
    scale_half = 2.0 ** (scale_exp.to(torch.float))

    return scale_half.to(half_dtype)
```
**EN:** Implements a reusable helper for E8m0 To Half, reducing duplication across related tests. It coordinates operations such as `scale_half.to`, `scale.to`, `scale_exp.to`.
**CN:** 该辅助函数为 E8m0 To Half 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `scale_half.to`, `scale.to`, `scale_exp.to` 等操作。

### Helper: upcast_fp4_to_fp16_or_bf16 (lines 39-88)
```python
def upcast_fp4_to_fp16_or_bf16(
    val, float_dtype: torch.dtype, half_exp_bias: int, half_mantissa_bits: int
):
    assert val.dtype == torch.uint8

    unpacked = torch.zeros(
        *val.shape[:-1], val.shape[-1] * 2, dtype=torch.uint8, device=val.device
    )
    unpacked[..., 1::2] = (val >> 4) & 0x0F  # Extract high 4 bits.
    unpacked[..., ::2] = val & 0x0F  # Extract low 4 bits.

    # Takes one float4 values represented as b0000xxxx,
    # and converts it to the corresponding float16 value.

    sign = unpacked >> 3

    exp = (unpacked >> 1) & 3
    new_mantissa = unpacked & 1

# ... omitted for brevity ...

    qdq_val = (
        (sign << 15)
        + (new_exp << half_mantissa_bits)
        + (new_mantissa << (half_mantissa_bits - 1))
    )

    assert qdq_val.max() <= 65535
    assert qdq_val.min() >= 0
    qdq_val = qdq_val.to(torch.uint16)

    result = qdq_val.view(float_dtype)

    return result
```
**EN:** Implements a reusable helper for Upcast Fp4 To FP16 Or BF16, reducing duplication across related tests. It coordinates operations such as `torch.zeros`, `torch.logical_and`, `new_mantissa.to`.
**CN:** 该辅助函数为 Upcast Fp4 To FP16 Or BF16 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `torch.zeros`, `torch.logical_and`, `new_mantissa.to` 等操作。

### Helper: dq_mxfp4_torch (lines 91-117)
```python
def dq_mxfp4_torch(
    x: torch.Tensor, scale: torch.Tensor, float_dtype: torch.dtype
) -> torch.Tensor:
    assert x.dtype == torch.uint8
    assert scale.dtype == torch.uint8

    if float_dtype == torch.float16:
        half_exp_bias = FLOAT16_EXP_BIAS
        half_mantissa_bits = FLOAT16_MANTISSA_BITS
    elif float_dtype == torch.bfloat16:
        half_exp_bias = BFLOAT16_EXP_BIAS
        half_mantissa_bits = BFLOAT16_MANTISSA_BITS

    scale_half = e8m0_to_half(scale, half_dtype=float_dtype)

    x_half = upcast_fp4_to_fp16_or_bf16(
        x,
        float_dtype=float_dtype,
        half_exp_bias=half_exp_bias,
        half_mantissa_bits=half_mantissa_bits,
    )

    x_half = x_half.reshape(*x_half.shape[:-1], -1, 32)
    x_half = x_half * scale_half[..., None]
    x_half = x_half.reshape(*x_half.shape[:-2], -1)

    return x_half
```
**EN:** Implements a reusable helper for Dq Mxfp4 Torch, reducing duplication across related tests. It coordinates operations such as `e8m0_to_half`, `upcast_fp4_to_fp16_or_bf16`, `x_half.reshape`.
**CN:** 该辅助函数为 Dq Mxfp4 Torch 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `e8m0_to_half`, `upcast_fp4_to_fp16_or_bf16`, `x_half.reshape` 等操作。

### Helper: fp16_to_fp4_simulate (lines 120-235)
```python
def fp16_to_fp4_simulate(
    val, half_mantissa_bits: int, half_exp_bits: int, half_exp_bias: int
):
    # Casts an fp16/bf16 input to the restricted values of float4_e2m1,
    # that is to say [0., 0.5, 1.0, 1.5, 2.0, 3.0, 4.0, 6.0, -0.0,
    # -0.5, -1.0, -1.5, -2.0, -3.0, -4.0, -6.0].

    float_type = val.dtype

    # "rshift_cuda" not implemented for 'UInt16'
    val_view = val.view(torch.int16)  # .to(torch.int32)

    exp = val_view >> half_mantissa_bits
    exp = exp & ((1 << half_exp_bits) - 1)

    exp = exp.view(torch.uint16).to(torch.int32)

    sign = (val_view >> (half_mantissa_bits + half_exp_bits)) & 1

# ... omitted for brevity ...

    qdq_val = (
        (sign << 15)
        + (new_exp << half_mantissa_bits)
        + (new_mantissa << (half_mantissa_bits - 1))
    )

    assert qdq_val.max() <= 65535
    assert qdq_val.min() >= 0
    assert qdq_val.dtype == torch.int32
    qdq_val = qdq_val.to(torch.uint16)

    result = qdq_val.view(float_type)
    return result
```
**EN:** Implements a reusable helper for FP16 To Fp4 Simulate, reducing duplication across related tests. It coordinates operations such as `val.view`, `exp.view(torch.uint16).to`, `torch.zeros`.
**CN:** 该辅助函数为 FP16 To Fp4 Simulate 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `val.view`, `exp.view(torch.uint16).to`, `torch.zeros` 等操作。

### Helper: qdq_mxfp4_torch (lines 238-292)
```python
def qdq_mxfp4_torch(
    x: torch.Tensor, scale_calculation_mode: str = "even"
) -> torch.Tensor:
    half_dtype = x.dtype

    if half_dtype == torch.float16:
        half_mantissa_bits = FLOAT16_MANTISSA_BITS
        half_exp_bits = FLOAT16_EXP_BITS
        half_exp_bias = FLOAT16_EXP_BIAS
        val_to_add = FLOAT16_VAL_TO_ADD
        sign_exponent_mask = FLOAT16_SIGN_EXPONENT_MASK
    elif half_dtype == torch.bfloat16:
        half_mantissa_bits = BFLOAT16_MANTISSA_BITS
        half_exp_bits = BFLOAT16_EXP_BITS
        half_exp_bias = BFLOAT16_EXP_BIAS
        val_to_add = BFLOAT16_VAL_TO_ADD
        sign_exponent_mask = BFLOAT16_SIGN_EXPONENT_MASK
    else:
        raise ValueError("not implemented")
# ... omitted for brevity ...
    scale = 2.0 ** (scale_exp - FLOAT8_E8M0_MAX_EXP)
    scale = scale.to(half_dtype)

    x = x / scale[..., None]

    x_fp4 = fp16_to_fp4_simulate(
        x,
        half_exp_bits=half_exp_bits,
        half_mantissa_bits=half_mantissa_bits,
        half_exp_bias=half_exp_bias,
    )

    x_fp4 = x_fp4 * scale[..., None]
    return x_fp4.reshape(*x_fp4.shape[:-2], -1)
```
**EN:** Implements a reusable helper for Qdq Mxfp4 Torch, reducing duplication across related tests. It coordinates operations such as `x.reshape`, `block_max.view(torch.uint16).to`, `torch.bitwise_and`.
**CN:** 该辅助函数为 Qdq Mxfp4 Torch 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `x.reshape`, `block_max.view(torch.uint16).to`, `torch.bitwise_and` 等操作。

## Key Concepts / 关键概念
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Quantization / 量化**
  - **EN:** The file covers low-precision model paths, kernels, or configuration rules for quantized inference.
  - **CN:** 该文件覆盖低精度模型路径、相关内核或量化推理配置规则。
- **Assertions / 断言**
  - **EN:** The file relies on explicit assertions to pin down expected state transitions and outputs.
  - **CN:** 该文件通过显式断言固定预期的状态变化与输出结果。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `torch`
