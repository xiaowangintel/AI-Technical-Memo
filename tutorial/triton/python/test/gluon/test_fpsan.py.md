# test_fpsan.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/test/gluon/test_fpsan.py`
- **EN:** Pytest module covering fpsan behavior in Triton's Python tests. It contains 98 top-level definition(s) and 10 imported module reference(s).
- **CN:** 这是一个 pytest 模块，用于覆盖 Triton Python 测试中的 fpsan 行为。 该文件包含 98 个顶层定义，以及 10 个导入模块引用。

## Line-by-Line Analysis / 逐行分析

### Lines 1-22

```python
# ruff: noqa: F821
import itertools
import numpy as np
import pytest
import torch

import triton
from triton.experimental import gluon
from triton.experimental.gluon import language as gl
from triton import language as tl
from triton._internal_testing import is_blackwell, is_cuda, is_hip, is_hip_cdna3, is_hip_cdna4, is_hip_gfx1250, is_hopper, is_interpreter
from triton.experimental.gluon.language.nvidia import hopper
from triton.experimental.gluon.language.nvidia.blackwell import (
    TensorMemoryLayout,
    TensorMemoryScalesLayout,
    allocate_tensor_memory,
    mbarrier,
    tcgen05_commit,
    tcgen05_copy,
    tcgen05_mma,
    tcgen05_mma_scaled,
)
```
- **EN:** Imports the modules used in this scope: `itertools`, `numpy`, `pytest`, `torch`, `triton`, `triton.experimental`, `triton.experimental.gluon`, `triton._internal_testing`, `triton.experimental.gluon.language.nvidia`, `triton.experimental.gluon.language.nvidia.blackwell`. Relevant themes: layout transformation reasoning.
- **CN:** 导入此作用域使用的模块：`itertools`、`numpy`、`pytest`、`torch`、`triton`、`triton.experimental`、`triton.experimental.gluon`、`triton._internal_testing`、`triton.experimental.gluon.language.nvidia`、`triton.experimental.gluon.language.nvidia.blackwell`。 相关主题：布局变换推理。

### Lines 23-24

```python

THREADS_PER_WARP = triton.runtime.driver.active.get_current_target().warp_size
```
- **EN:** Prepares or updates state through `THREADS_PER_WARP`. Invokes `triton.runtime.driver.active.get_current_target` to execute the test logic. Relevant themes: runtime driver interaction.
- **CN:** 通过 `THREADS_PER_WARP` 准备或更新状态。 调用 `triton.runtime.driver.active.get_current_target` 执行测试逻辑。 相关主题：运行时驱动交互。

### Lines 25-27

```python


def _hip_device_supports_fpsan():
```
- **EN:** Defines the helper function `_hip_device_supports_fpsan`. Key calls include `is_hip_cdna3`, `is_hip_cdna4`, `is_hip_gfx1250`.
- **CN:** 定义辅助函数 `_hip_device_supports_fpsan`。 关键调用包括 `is_hip_cdna3`、`is_hip_cdna4`、`is_hip_gfx1250`。

#### Lines 28-28

```python
    return is_hip_cdna3() or is_hip_cdna4() or is_hip_gfx1250()
```
- **EN:** Invokes `is_hip_cdna3`, `is_hip_cdna4`, `is_hip_gfx1250` to execute the test logic.
- **CN:** 调用 `is_hip_cdna3`、`is_hip_cdna4`、`is_hip_gfx1250` 执行测试逻辑。

### Lines 29-32

```python


def _require_cuda_backend(device: str):
    # CUDA and HIP both use torch device 'cuda'. fpsan is plumbed through both CUDAOptions and HIPOptions.
```
- **EN:** Defines the helper function `_require_cuda_backend`. Parameters: `device`. Key calls include `is_interpreter`, `pytest.skip`, `is_hip`, `torch.cuda.is_available`, `is_cuda`, `_hip_device_supports_fpsan`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `_require_cuda_backend`。 参数：`device`。 关键调用包括 `is_interpreter`、`pytest.skip`、`is_hip`、`torch.cuda.is_available`、`is_cuda`、`_hip_device_supports_fpsan`。 该作用域涉及PyTorch 张量准备与校验。

#### Lines 33-34

```python
    if device != "cuda":
        pytest.skip("fpsan tests require torch device 'cuda'")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 35-36

```python
    if is_interpreter():
        pytest.skip("fpsan tests require a real backend (not the interpreter)")
```
- **EN:** Invokes `is_interpreter`, `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `is_interpreter`、`pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 37-38

```python
    if not (is_cuda() or is_hip()):
        pytest.skip("fpsan tests require CUDA or HIP")
```
- **EN:** Invokes `pytest.skip`, `is_cuda`, `is_hip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip`、`is_cuda`、`is_hip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 39-40

```python
    if is_hip() and not _hip_device_supports_fpsan():
        pytest.skip("fpsan is not supported on this HIP device")
```
- **EN:** Invokes `is_hip`, `pytest.skip`, `_hip_device_supports_fpsan` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `is_hip`、`pytest.skip`、`_hip_device_supports_fpsan` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 41-42

```python
    if not torch.cuda.is_available():
        pytest.skip("CUDA is not available")
```
- **EN:** Invokes `torch.cuda.is_available`, `pytest.skip` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.cuda.is_available`、`pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

### Lines 43-45

```python


def _as_u32(x_i32: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `_as_u32`. Parameters: `x_i32`. Key calls include `x_i32.view`.
- **CN:** 定义辅助函数 `_as_u32`。 参数：`x_i32`。 关键调用包括 `x_i32.view`。

#### Lines 46-46

```python
    assert x_i32.dtype == np.int32
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

#### Lines 47-47

```python
    return x_i32.view(np.uint32)
```
- **EN:** Invokes `x_i32.view` to execute the test logic.
- **CN:** 调用 `x_i32.view` 执行测试逻辑。

### Lines 48-50

```python


def _u32_to_i32(x_u32: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `_u32_to_i32`. Parameters: `x_u32`. Key calls include `x_u32.view`.
- **CN:** 定义辅助函数 `_u32_to_i32`。 参数：`x_u32`。 关键调用包括 `x_u32.view`。

#### Lines 51-51

```python
    assert x_u32.dtype == np.uint32
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

#### Lines 52-52

```python
    return x_u32.view(np.int32)
```
- **EN:** Invokes `x_u32.view` to execute the test logic.
- **CN:** 调用 `x_u32.view` 执行测试逻辑。

### Lines 53-55

```python


def _low_mask_u64(bitwidth: int) -> np.uint64:
```
- **EN:** Defines the helper function `_low_mask_u64`. Parameters: `bitwidth`. Key calls include `np.uint64`.
- **CN:** 定义辅助函数 `_low_mask_u64`。 参数：`bitwidth`。 关键调用包括 `np.uint64`。

#### Lines 56-56

```python
    return np.uint64(2**bitwidth - 1)
```
- **EN:** Invokes `np.uint64` to execute the test logic.
- **CN:** 调用 `np.uint64` 执行测试逻辑。

### Lines 57-59

```python


def _inv_odd_u64(a: np.uint64) -> np.uint64:
```
- **EN:** Defines the helper function `_inv_odd_u64`. Parameters: `a`. Key calls include `np.errstate`, `np.uint64`.
- **CN:** 定义辅助函数 `_inv_odd_u64`。 参数：`a`。 关键调用包括 `np.errstate`、`np.uint64`。

#### Lines 60-64

```python
    with np.errstate(over="ignore"):
        x = np.uint64(2) - a
        for _ in range(5):
            x = np.uint64(x * (np.uint64(2) - np.uint64(a * x)))
        return x
```
- **EN:** Invokes `np.errstate`, `np.uint64` to execute the test logic. Uses context managers to control resources or expectations.
- **CN:** 调用 `np.errstate`、`np.uint64` 执行测试逻辑。 使用上下文管理器控制资源或预期行为。

### Lines 65-67

```python


def _mix_config(bitwidth: int, one_bits: int) -> tuple[np.uint64, np.uint64, np.uint64, int, np.uint64, np.uint64]:
```
- **EN:** Defines the helper function `_mix_config`. Parameters: `bitwidth`, `one_bits`. Key calls include `np.errstate`, `_low_mask_u64`, `np.uint64`, `_inv_odd_u64`. This scope touches autotuning logic.
- **CN:** 定义辅助函数 `_mix_config`。 参数：`bitwidth`、`one_bits`。 关键调用包括 `np.errstate`、`_low_mask_u64`、`np.uint64`、`_inv_odd_u64`。 该作用域涉及自动调优逻辑。

#### Lines 68-77

```python
    with np.errstate(over="ignore"):
        full_mask = _low_mask_u64(bitwidth)
        sign_mask = np.uint64(1 << (bitwidth - 1))
        mag_mask = sign_mask - np.uint64(1)
        shift = int((one_bits & -one_bits).bit_length() - 1)
        y = (np.uint64(one_bits) * np.uint64(922291)) & mag_mask
        z = y ^ (y >> np.uint64(shift))
        mul_b_pos = _inv_odd_u64(z) & full_mask
        mul_b_neg = (mul_b_pos * mag_mask) & full_mask
        return full_mask, sign_mask, mag_mask, shift, mul_b_pos, mul_b_neg
```
- **EN:** Invokes `np.errstate`, `_low_mask_u64`, `np.uint64`, `_inv_odd_u64` to execute the test logic. Uses context managers to control resources or expectations.
- **CN:** 调用 `np.errstate`、`_low_mask_u64`、`np.uint64`、`_inv_odd_u64` 执行测试逻辑。 使用上下文管理器控制资源或预期行为。

### Lines 78-80

```python


def _xor_shift_right_u64(x: np.ndarray, shift: int) -> np.ndarray:
```
- **EN:** Defines the helper function `_xor_shift_right_u64`. Parameters: `x`, `shift`. Key calls include `np.uint64`.
- **CN:** 定义辅助函数 `_xor_shift_right_u64`。 参数：`x`、`shift`。 关键调用包括 `np.uint64`。

#### Lines 81-81

```python
    return x ^ (x >> np.uint64(shift))
```
- **EN:** Invokes `np.uint64` to execute the test logic.
- **CN:** 调用 `np.uint64` 执行测试逻辑。

### Lines 82-84

```python


def _inverse_xor_shift_right_u64(x: np.ndarray, shift: int, bitwidth: int) -> np.ndarray:
```
- **EN:** Defines the helper function `_inverse_xor_shift_right_u64`. Parameters: `x`, `shift`, `bitwidth`. Key calls include `_xor_shift_right_u64`.
- **CN:** 定义辅助函数 `_inverse_xor_shift_right_u64`。 参数：`x`、`shift`、`bitwidth`。 关键调用包括 `_xor_shift_right_u64`。

#### Lines 85-87

```python
    while shift < bitwidth:
        x = _xor_shift_right_u64(x, shift)
        shift *= 2
```
- **EN:** Invokes `_xor_shift_right_u64` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `_xor_shift_right_u64` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 88-88

```python
    return x
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 89-91

```python


def _mix_float_bits_to_payload_u64(bits, bitwidth: int, one_bits: int) -> np.ndarray:
```
- **EN:** Defines the helper function `_mix_float_bits_to_payload_u64`. Parameters: `bits`, `bitwidth`, `one_bits`. Key calls include `_mix_config`, `np.where`, `_xor_shift_right_u64`, `bits.astype`, `np.uint64`. This scope touches autotuning logic.
- **CN:** 定义辅助函数 `_mix_float_bits_to_payload_u64`。 参数：`bits`、`bitwidth`、`one_bits`。 关键调用包括 `_mix_config`、`np.where`、`_xor_shift_right_u64`、`bits.astype`、`np.uint64`。 该作用域涉及自动调优逻辑。

#### Lines 92-99

```python
    full_mask, sign_mask, mag_mask, shift, mul_b_pos, mul_b_neg = _mix_config(bitwidth, one_bits)
    x = bits.astype(np.uint64) & full_mask
    neg = (x & sign_mask) != 0
    sign = np.where(neg, sign_mask, np.uint64(0))
    y = (((x ^ sign) * np.uint64(922291)) & mag_mask)
    z = _xor_shift_right_u64(y, shift)
    factor = np.where(neg, mul_b_neg, mul_b_pos)
    return (((z * factor) & mag_mask) ^ sign) & full_mask
```
- **EN:** Prepares or updates state through `full_mask`, `sign_mask`, `mag_mask`, `shift`, `mul_b_pos`, `mul_b_neg`, `x`, `neg`, and 4 more. Invokes `_mix_config`, `bits.astype`, `np.where`, `np.uint64`, `_xor_shift_right_u64` to execute the test logic. Relevant themes: autotuning logic.
- **CN:** 通过 `full_mask`、`sign_mask`、`mag_mask`、`shift`、`mul_b_pos`、`mul_b_neg`、`x`、`neg` 等另外 4 项 准备或更新状态。 调用 `_mix_config`、`bits.astype`、`np.where`、`np.uint64`、`_xor_shift_right_u64` 执行测试逻辑。 相关主题：自动调优逻辑。

### Lines 100-102

```python


def _unmix_payload_u64_to_float_bits(payload, bitwidth: int, one_bits: int) -> np.ndarray:
```
- **EN:** Defines the helper function `_unmix_payload_u64_to_float_bits`. Parameters: `payload`, `bitwidth`, `one_bits`. Key calls include `_mix_config`, `np.where`, `_inverse_xor_shift_right_u64`, `payload.astype`, `np.uint64`, `_inv_odd_u64`. This scope touches autotuning logic.
- **CN:** 定义辅助函数 `_unmix_payload_u64_to_float_bits`。 参数：`payload`、`bitwidth`、`one_bits`。 关键调用包括 `_mix_config`、`np.where`、`_inverse_xor_shift_right_u64`、`payload.astype`、`np.uint64`、`_inv_odd_u64`。 该作用域涉及自动调优逻辑。

#### Lines 103-111

```python
    full_mask, sign_mask, mag_mask, shift, mul_b_pos, mul_b_neg = _mix_config(bitwidth, one_bits)
    v = payload.astype(np.uint64) & full_mask
    neg = (v & sign_mask) != 0
    sign = np.where(neg, sign_mask, np.uint64(0))
    factor = np.where(neg, _inv_odd_u64(mul_b_neg), _inv_odd_u64(mul_b_pos))
    z = (((v ^ sign) * factor) & mag_mask)
    y = _inverse_xor_shift_right_u64(z, shift, bitwidth)
    x = (y * _inv_odd_u64(np.uint64(922291))) & mag_mask
    return (x ^ sign) & full_mask
```
- **EN:** Prepares or updates state through `full_mask`, `sign_mask`, `mag_mask`, `shift`, `mul_b_pos`, `mul_b_neg`, `v`, `neg`, and 5 more. Invokes `_mix_config`, `payload.astype`, `np.where`, `np.uint64`, `_inv_odd_u64`, `_inverse_xor_shift_right_u64` to execute the test logic. Relevant themes: autotuning logic.
- **CN:** 通过 `full_mask`、`sign_mask`、`mag_mask`、`shift`、`mul_b_pos`、`mul_b_neg`、`v`、`neg` 等另外 5 项 准备或更新状态。 调用 `_mix_config`、`payload.astype`、`np.where`、`np.uint64`、`_inv_odd_u64`、`_inverse_xor_shift_right_u64` 执行测试逻辑。 相关主题：自动调优逻辑。

### Lines 112-114

```python


def _mix_f32_bits_to_payload_u32(x_i32: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `_mix_f32_bits_to_payload_u32`. Parameters: `x_i32`. Key calls include `_mix_float_bits_to_payload_u64`, `_as_u32`.
- **CN:** 定义辅助函数 `_mix_f32_bits_to_payload_u32`。 参数：`x_i32`。 关键调用包括 `_mix_float_bits_to_payload_u64`、`_as_u32`。

#### Lines 115-115

```python
    return _mix_float_bits_to_payload_u64(_as_u32(x_i32), 32, 0x3F800000).astype(np.uint32)
```
- **EN:** Invokes `_mix_float_bits_to_payload_u64`, `_as_u32` to execute the test logic.
- **CN:** 调用 `_mix_float_bits_to_payload_u64`、`_as_u32` 执行测试逻辑。

### Lines 116-118

```python


def _unmix_payload_u32_to_f32_bits_i32(v_u32: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `_unmix_payload_u32_to_f32_bits_i32`. Parameters: `v_u32`. Key calls include `_u32_to_i32`, `_unmix_payload_u64_to_float_bits`.
- **CN:** 定义辅助函数 `_unmix_payload_u32_to_f32_bits_i32`。 参数：`v_u32`。 关键调用包括 `_u32_to_i32`、`_unmix_payload_u64_to_float_bits`。

#### Lines 119-119

```python
    assert v_u32.dtype == np.uint32
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

#### Lines 120-120

```python
    return _u32_to_i32(_unmix_payload_u64_to_float_bits(v_u32, 32, 0x3F800000).astype(np.uint32))
```
- **EN:** Invokes `_u32_to_i32`, `_unmix_payload_u64_to_float_bits` to execute the test logic.
- **CN:** 调用 `_u32_to_i32`、`_unmix_payload_u64_to_float_bits` 执行测试逻辑。

### Lines 121-123

```python


def _signed_cast_payload_u64(payload, src_bitwidth: int, dst_bitwidth: int) -> np.ndarray:
```
- **EN:** Defines the helper function `_signed_cast_payload_u64`. Parameters: `payload`, `src_bitwidth`, `dst_bitwidth`. Key calls include `np.uint64`, `payload.astype`, `_low_mask_u64`, `np.where`.
- **CN:** 定义辅助函数 `_signed_cast_payload_u64`。 参数：`payload`、`src_bitwidth`、`dst_bitwidth`。 关键调用包括 `np.uint64`、`payload.astype`、`_low_mask_u64`、`np.where`。

#### Lines 124-124

```python
    x = payload.astype(np.uint64) & _low_mask_u64(src_bitwidth)
```
- **EN:** Prepares or updates state through `x`. Invokes `payload.astype`, `_low_mask_u64` to execute the test logic.
- **CN:** 通过 `x` 准备或更新状态。 调用 `payload.astype`、`_low_mask_u64` 执行测试逻辑。

#### Lines 125-126

```python
    if dst_bitwidth <= src_bitwidth:
        return x & _low_mask_u64(dst_bitwidth)
```
- **EN:** Invokes `_low_mask_u64` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `_low_mask_u64` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 127-130

```python

    sign = np.uint64(1 << (src_bitwidth - 1))
    extension = _low_mask_u64(dst_bitwidth) ^ _low_mask_u64(src_bitwidth)
    return np.where((x & sign) != 0, x | extension, x) & _low_mask_u64(dst_bitwidth)
```
- **EN:** Prepares or updates state through `sign`, `extension`. Invokes `np.uint64`, `_low_mask_u64`, `np.where` to execute the test logic.
- **CN:** 通过 `sign`、`extension` 准备或更新状态。 调用 `np.uint64`、`_low_mask_u64`、`np.where` 执行测试逻辑。

### Lines 131-133

```python


def _payload_u32_to_f32_bits_i32(x_u64: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `_payload_u32_to_f32_bits_i32`. Parameters: `x_u64`. Key calls include `_unmix_payload_u32_to_f32_bits_i32`, `np.uint64`.
- **CN:** 定义辅助函数 `_payload_u32_to_f32_bits_i32`。 参数：`x_u64`。 关键调用包括 `_unmix_payload_u32_to_f32_bits_i32`、`np.uint64`。

#### Lines 134-134

```python
    return _unmix_payload_u32_to_f32_bits_i32((x_u64 & np.uint64(0xFFFFFFFF)).astype(np.uint32))
```
- **EN:** Invokes `_unmix_payload_u32_to_f32_bits_i32`, `np.uint64` to execute the test logic.
- **CN:** 调用 `_unmix_payload_u32_to_f32_bits_i32`、`np.uint64` 执行测试逻辑。

### Lines 135-137

```python


def _expected_add_i32(x_i32: np.ndarray, y_i32: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `_expected_add_i32`. Parameters: `x_i32`, `y_i32`. Key calls include `_payload_u32_to_f32_bits_i32`, `_mix_f32_bits_to_payload_u32`.
- **CN:** 定义辅助函数 `_expected_add_i32`。 参数：`x_i32`、`y_i32`。 关键调用包括 `_payload_u32_to_f32_bits_i32`、`_mix_f32_bits_to_payload_u32`。

#### Lines 138-140

```python
    x_u32 = _mix_f32_bits_to_payload_u32(x_i32).astype(np.uint64)
    y_u32 = _mix_f32_bits_to_payload_u32(y_i32).astype(np.uint64)
    return _payload_u32_to_f32_bits_i32(x_u32 + y_u32)
```
- **EN:** Prepares or updates state through `x_u32`, `y_u32`. Invokes `_mix_f32_bits_to_payload_u32`, `_payload_u32_to_f32_bits_i32` to execute the test logic.
- **CN:** 通过 `x_u32`、`y_u32` 准备或更新状态。 调用 `_mix_f32_bits_to_payload_u32`、`_payload_u32_to_f32_bits_i32` 执行测试逻辑。

### Lines 141-143

```python


def _expected_sub_i32(x_i32: np.ndarray, y_i32: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `_expected_sub_i32`. Parameters: `x_i32`, `y_i32`. Key calls include `_payload_u32_to_f32_bits_i32`, `_mix_f32_bits_to_payload_u32`.
- **CN:** 定义辅助函数 `_expected_sub_i32`。 参数：`x_i32`、`y_i32`。 关键调用包括 `_payload_u32_to_f32_bits_i32`、`_mix_f32_bits_to_payload_u32`。

#### Lines 144-146

```python
    x_u32 = _mix_f32_bits_to_payload_u32(x_i32).astype(np.uint64)
    y_u32 = _mix_f32_bits_to_payload_u32(y_i32).astype(np.uint64)
    return _payload_u32_to_f32_bits_i32(x_u32 - y_u32)
```
- **EN:** Prepares or updates state through `x_u32`, `y_u32`. Invokes `_mix_f32_bits_to_payload_u32`, `_payload_u32_to_f32_bits_i32` to execute the test logic.
- **CN:** 通过 `x_u32`、`y_u32` 准备或更新状态。 调用 `_mix_f32_bits_to_payload_u32`、`_payload_u32_to_f32_bits_i32` 执行测试逻辑。

### Lines 147-149

```python


def _expected_neg_i32(x_i32: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `_expected_neg_i32`. Parameters: `x_i32`. Key calls include `_payload_u32_to_f32_bits_i32`, `_mix_f32_bits_to_payload_u32`, `np.uint64`.
- **CN:** 定义辅助函数 `_expected_neg_i32`。 参数：`x_i32`。 关键调用包括 `_payload_u32_to_f32_bits_i32`、`_mix_f32_bits_to_payload_u32`、`np.uint64`。

#### Lines 150-151

```python
    x_u32 = _mix_f32_bits_to_payload_u32(x_i32).astype(np.uint64)
    return _payload_u32_to_f32_bits_i32(np.uint64(0) - x_u32)
```
- **EN:** Prepares or updates state through `x_u32`. Invokes `_mix_f32_bits_to_payload_u32`, `_payload_u32_to_f32_bits_i32`, `np.uint64` to execute the test logic.
- **CN:** 通过 `x_u32` 准备或更新状态。 调用 `_mix_f32_bits_to_payload_u32`、`_payload_u32_to_f32_bits_i32`、`np.uint64` 执行测试逻辑。

### Lines 152-154

```python


def _expected_mul_i32(x_i32: np.ndarray, y_i32: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `_expected_mul_i32`. Parameters: `x_i32`, `y_i32`. Key calls include `_payload_u32_to_f32_bits_i32`, `_mix_f32_bits_to_payload_u32`.
- **CN:** 定义辅助函数 `_expected_mul_i32`。 参数：`x_i32`、`y_i32`。 关键调用包括 `_payload_u32_to_f32_bits_i32`、`_mix_f32_bits_to_payload_u32`。

#### Lines 155-157

```python
    x_u32 = _mix_f32_bits_to_payload_u32(x_i32).astype(np.uint64)
    y_u32 = _mix_f32_bits_to_payload_u32(y_i32).astype(np.uint64)
    return _payload_u32_to_f32_bits_i32(x_u32 * y_u32)
```
- **EN:** Prepares or updates state through `x_u32`, `y_u32`. Invokes `_mix_f32_bits_to_payload_u32`, `_payload_u32_to_f32_bits_i32` to execute the test logic.
- **CN:** 通过 `x_u32`、`y_u32` 准备或更新状态。 调用 `_mix_f32_bits_to_payload_u32`、`_payload_u32_to_f32_bits_i32` 执行测试逻辑。

### Lines 158-160

```python


def _expected_min_i32(x_i32: np.ndarray, y_i32: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `_expected_min_i32`. Parameters: `x_i32`, `y_i32`. Key calls include `_u32_to_i32`, `_unmix_payload_u32_to_f32_bits_i32`, `_mix_f32_bits_to_payload_u32`, `np.minimum`.
- **CN:** 定义辅助函数 `_expected_min_i32`。 参数：`x_i32`、`y_i32`。 关键调用包括 `_u32_to_i32`、`_unmix_payload_u32_to_f32_bits_i32`、`_mix_f32_bits_to_payload_u32`、`np.minimum`。

#### Lines 161-163

```python
    x = _u32_to_i32(_mix_f32_bits_to_payload_u32(x_i32))
    y = _u32_to_i32(_mix_f32_bits_to_payload_u32(y_i32))
    return _unmix_payload_u32_to_f32_bits_i32(np.minimum(x, y).astype(np.int32).view(np.uint32))
```
- **EN:** Prepares or updates state through `x`, `y`. Invokes `_u32_to_i32`, `_mix_f32_bits_to_payload_u32`, `_unmix_payload_u32_to_f32_bits_i32`, `np.minimum` to execute the test logic.
- **CN:** 通过 `x`、`y` 准备或更新状态。 调用 `_u32_to_i32`、`_mix_f32_bits_to_payload_u32`、`_unmix_payload_u32_to_f32_bits_i32`、`np.minimum` 执行测试逻辑。

### Lines 164-166

```python


def _expected_max_i32(x_i32: np.ndarray, y_i32: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `_expected_max_i32`. Parameters: `x_i32`, `y_i32`. Key calls include `_u32_to_i32`, `_unmix_payload_u32_to_f32_bits_i32`, `_mix_f32_bits_to_payload_u32`, `np.maximum`.
- **CN:** 定义辅助函数 `_expected_max_i32`。 参数：`x_i32`、`y_i32`。 关键调用包括 `_u32_to_i32`、`_unmix_payload_u32_to_f32_bits_i32`、`_mix_f32_bits_to_payload_u32`、`np.maximum`。

#### Lines 167-169

```python
    x = _u32_to_i32(_mix_f32_bits_to_payload_u32(x_i32))
    y = _u32_to_i32(_mix_f32_bits_to_payload_u32(y_i32))
    return _unmix_payload_u32_to_f32_bits_i32(np.maximum(x, y).astype(np.int32).view(np.uint32))
```
- **EN:** Prepares or updates state through `x`, `y`. Invokes `_u32_to_i32`, `_mix_f32_bits_to_payload_u32`, `_unmix_payload_u32_to_f32_bits_i32`, `np.maximum` to execute the test logic.
- **CN:** 通过 `x`、`y` 准备或更新状态。 调用 `_u32_to_i32`、`_mix_f32_bits_to_payload_u32`、`_unmix_payload_u32_to_f32_bits_i32`、`np.maximum` 执行测试逻辑。

### Lines 170-176

```python


def _expected_srem_i32(x_i32: np.ndarray, y_i32: np.ndarray) -> np.ndarray:
    # Match LLVM srem semantics: remainder after trunc-toward-zero division.
    # NOTE: Python/NumPy '%' uses floor division for negatives, so we implement explicitly.
    #
    # In fpsan mode we force denominator non-zero using `den | 1` in the *payload* domain.
```
- **EN:** Defines the helper function `_expected_srem_i32`. Parameters: `x_i32`, `y_i32`. Key calls include `_unmix_payload_u32_to_f32_bits_i32`, `_u32_to_i32`, `_mix_f32_bits_to_payload_u32`, `np.uint32`, `np.sign`, `np.abs`.
- **CN:** 定义辅助函数 `_expected_srem_i32`。 参数：`x_i32`、`y_i32`。 关键调用包括 `_unmix_payload_u32_to_f32_bits_i32`、`_u32_to_i32`、`_mix_f32_bits_to_payload_u32`、`np.uint32`、`np.sign`、`np.abs`。

#### Lines 177-182

```python
    x = _u32_to_i32(_mix_f32_bits_to_payload_u32(x_i32)).astype(np.int64)
    y_safe_u32 = (_mix_f32_bits_to_payload_u32(y_i32) | np.uint32(1)).astype(np.uint32)
    y = _u32_to_i32(y_safe_u32).astype(np.int64)
    q = (np.sign(x) * np.sign(y) * (np.abs(x) // np.abs(y))).astype(np.int64)
    r = (x - q * y).astype(np.int32).view(np.uint32)
    return _unmix_payload_u32_to_f32_bits_i32(r)
```
- **EN:** Prepares or updates state through `x`, `y_safe_u32`, `y`, `q`, `r`. Invokes `_u32_to_i32`, `_mix_f32_bits_to_payload_u32`, `np.uint32`, `np.sign`, `np.abs`, `_unmix_payload_u32_to_f32_bits_i32` to execute the test logic.
- **CN:** 通过 `x`、`y_safe_u32`、`y`、`q`、`r` 准备或更新状态。 调用 `_u32_to_i32`、`_mix_f32_bits_to_payload_u32`、`np.uint32`、`np.sign`、`np.abs`、`_unmix_payload_u32_to_f32_bits_i32` 执行测试逻辑。

### Lines 183-185

```python


def murmur64Mixer(h: np.uint64) -> np.uint64:
```
- **EN:** Defines the helper function `murmur64Mixer`. Parameters: `h`. Key calls include `np.errstate`, `np.uint64`.
- **CN:** 定义辅助函数 `murmur64Mixer`。 参数：`h`。 关键调用包括 `np.errstate`、`np.uint64`。

#### Lines 186-193

```python
    with np.errstate(over="ignore"):
        h = np.uint64(h)
        h ^= h >> np.uint64(33)
        h = np.uint64(h * np.uint64(0xff51afd7ed558ccd))
        h ^= h >> np.uint64(33)
        h = np.uint64(h * np.uint64(0xc4ceb9fe1a85ec53))
        h ^= h >> np.uint64(33)
        return np.uint64(h)
```
- **EN:** Invokes `np.errstate`, `np.uint64` to execute the test logic. Uses context managers to control resources or expectations.
- **CN:** 调用 `np.errstate`、`np.uint64` 执行测试逻辑。 使用上下文管理器控制资源或预期行为。

### Lines 194-212

```python


OP_TO_ID_U64 = {
    "exp": np.uint64(0),
    "log": np.uint64(1),
    "exp2": np.uint64(2),
    "log2": np.uint64(3),
    "cos": np.uint64(4),
    "sin": np.uint64(5),
    "sqrt": np.uint64(6),
    "rsqrt": np.uint64(7),
    "erf": np.uint64(8),
    "floor": np.uint64(9),
    "ceil": np.uint64(10),
    "sqrt_rn": np.uint64(11),
}

OP_TO_TAG_U32 = {name: np.uint32(murmur64Mixer(op_id) & np.uint64(0xFFFFFFFF)) for name, op_id in OP_TO_ID_U64.items()}
UNARY_TAG_MULTIPLIER_U64 = np.uint64(314159)
```
- **EN:** Prepares or updates state through `OP_TO_ID_U64`, `OP_TO_TAG_U32`, `UNARY_TAG_MULTIPLIER_U64`. Invokes `np.uint64`, `np.uint32`, `OP_TO_ID_U64.items`, `murmur64Mixer` to execute the test logic.
- **CN:** 通过 `OP_TO_ID_U64`、`OP_TO_TAG_U32`、`UNARY_TAG_MULTIPLIER_U64` 准备或更新状态。 调用 `np.uint64`、`np.uint32`、`OP_TO_ID_U64.items`、`murmur64Mixer` 执行测试逻辑。

### Lines 213-215

```python


def _expected_unary_tag_payload_u32(x_u32: np.ndarray, op: str) -> np.ndarray:
```
- **EN:** Defines the helper function `_expected_unary_tag_payload_u32`. Parameters: `x_u32`, `op`. Key calls include `x_u32.astype`, `np.uint64`.
- **CN:** 定义辅助函数 `_expected_unary_tag_payload_u32`。 参数：`x_u32`、`op`。 关键调用包括 `x_u32.astype`、`np.uint64`。

#### Lines 216-219

```python
    tag = OP_TO_TAG_U32[op].astype(np.uint64)
    x = x_u32.astype(np.uint64)
    out_u64 = (((x * UNARY_TAG_MULTIPLIER_U64) & np.uint64(0xFFFFFFFF)) ^ tag) * UNARY_TAG_MULTIPLIER_U64
    return (out_u64 & np.uint64(0xFFFFFFFF)).astype(np.uint32)
```
- **EN:** Prepares or updates state through `tag`, `x`, `out_u64`. Invokes `x_u32.astype`, `np.uint64` to execute the test logic.
- **CN:** 通过 `tag`、`x`、`out_u64` 准备或更新状态。 调用 `x_u32.astype`、`np.uint64` 执行测试逻辑。

### Lines 220-222

```python


def _expected_u32_inv(x_u32: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `_expected_u32_inv`. Parameters: `x_u32`. Key calls include `np.uint64`, `x.astype`, `x_u32.astype`.
- **CN:** 定义辅助函数 `_expected_u32_inv`。 参数：`x_u32`。 关键调用包括 `np.uint64`、`x.astype`、`x_u32.astype`。

#### Lines 223-225

```python
    mask = np.uint64(0xFFFFFFFF)
    a = x_u32.astype(np.uint64) | np.uint64(1)
    x = (np.uint64(2) - a) & mask
```
- **EN:** Prepares or updates state through `mask`, `a`, `x`. Invokes `np.uint64`, `x_u32.astype` to execute the test logic.
- **CN:** 通过 `mask`、`a`、`x` 准备或更新状态。 调用 `np.uint64`、`x_u32.astype` 执行测试逻辑。

#### Lines 226-228

```python
    for _ in range(4):
        factor = (np.uint64(2) - ((a * x) & mask)) & mask
        x = (x * factor) & mask
```
- **EN:** Invokes `np.uint64` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `np.uint64` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 229-230

```python
    x = (x & np.uint64(0xFFFFFFFE)) | (x_u32.astype(np.uint64) & np.uint64(1))
    return x.astype(np.uint32)
```
- **EN:** Prepares or updates state through `x`. Invokes `np.uint64`, `x_u32.astype`, `x.astype` to execute the test logic.
- **CN:** 通过 `x` 准备或更新状态。 调用 `np.uint64`、`x_u32.astype`、`x.astype` 执行测试逻辑。

### Lines 231-234

```python


def _expected_div_payload_i32(x_i32: np.ndarray, y_i32: np.ndarray) -> np.ndarray:
    # fpsan division is defined as num_bits * u32_inv(den_bits) mod 2^32.
```
- **EN:** Defines the helper function `_expected_div_payload_i32`. Parameters: `x_i32`, `y_i32`. Key calls include `_payload_u32_to_f32_bits_i32`, `_mix_f32_bits_to_payload_u32`, `_expected_u32_inv`.
- **CN:** 定义辅助函数 `_expected_div_payload_i32`。 参数：`x_i32`、`y_i32`。 关键调用包括 `_payload_u32_to_f32_bits_i32`、`_mix_f32_bits_to_payload_u32`、`_expected_u32_inv`。

#### Lines 235-237

```python
    num = _mix_f32_bits_to_payload_u32(x_i32).astype(np.uint64)
    inv = _expected_u32_inv(_mix_f32_bits_to_payload_u32(y_i32)).astype(np.uint64)
    return _payload_u32_to_f32_bits_i32(num * inv)
```
- **EN:** Prepares or updates state through `num`, `inv`. Invokes `_mix_f32_bits_to_payload_u32`, `_expected_u32_inv`, `_payload_u32_to_f32_bits_i32` to execute the test logic.
- **CN:** 通过 `num`、`inv` 准备或更新状态。 调用 `_mix_f32_bits_to_payload_u32`、`_expected_u32_inv`、`_payload_u32_to_f32_bits_i32` 执行测试逻辑。

### Lines 238-240

```python


def _expected_exp2_i32(x_i32: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `_expected_exp2_i32`. Parameters: `x_i32`. Key calls include `np.uint64`, `np.ones_like`, `_unmix_payload_u32_to_f32_bits_i32`, `np.where`, `y.astype`, `_mix_f32_bits_to_payload_u32`.
- **CN:** 定义辅助函数 `_expected_exp2_i32`。 参数：`x_i32`。 关键调用包括 `np.uint64`、`np.ones_like`、`_unmix_payload_u32_to_f32_bits_i32`、`np.where`、`y.astype`、`_mix_f32_bits_to_payload_u32`。

#### Lines 241-244

```python
    c = np.uint64(0xa343836d)
    mask = np.uint64(0xFFFFFFFF)
    x = _mix_f32_bits_to_payload_u32(x_i32).astype(np.uint64)
    y = np.ones_like(x, dtype=np.uint64)
```
- **EN:** Prepares or updates state through `c`, `mask`, `x`, `y`. Invokes `np.uint64`, `_mix_f32_bits_to_payload_u32`, `np.ones_like` to execute the test logic.
- **CN:** 通过 `c`、`mask`、`x`、`y` 准备或更新状态。 调用 `np.uint64`、`_mix_f32_bits_to_payload_u32`、`np.ones_like` 执行测试逻辑。

#### Lines 245-248

```python
    for i in range(32):
        y = (y * y) & mask
        factor = np.where((x & np.uint64(1 << (31 - i))) == 0, np.uint64(1), c)
        y = (y * factor) & mask
```
- **EN:** Invokes `np.where`, `np.uint64` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `np.where`、`np.uint64` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 249-249

```python
    return _unmix_payload_u32_to_f32_bits_i32(y.astype(np.uint32))
```
- **EN:** Invokes `_unmix_payload_u32_to_f32_bits_i32`, `y.astype` to execute the test logic.
- **CN:** 调用 `_unmix_payload_u32_to_f32_bits_i32`、`y.astype` 执行测试逻辑。

### Lines 250-252

```python


def _expected_exp_i32(x_i32: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `_expected_exp_i32`. Parameters: `x_i32`. Key calls include `np.uint64`, `_expected_exp2_i32`, `_unmix_payload_u32_to_f32_bits_i32`, `_mix_f32_bits_to_payload_u32`.
- **CN:** 定义辅助函数 `_expected_exp_i32`。 参数：`x_i32`。 关键调用包括 `np.uint64`、`_expected_exp2_i32`、`_unmix_payload_u32_to_f32_bits_i32`、`_mix_f32_bits_to_payload_u32`。

#### Lines 253-256

```python
    x = _mix_f32_bits_to_payload_u32(x_i32).astype(np.uint64)
    rcp_log2 = np.uint64(0x236ee9bf)
    scaled = ((x * rcp_log2) & np.uint64(0xFFFFFFFF)).astype(np.uint32)
    return _expected_exp2_i32(_unmix_payload_u32_to_f32_bits_i32(scaled))
```
- **EN:** Prepares or updates state through `x`, `rcp_log2`, `scaled`. Invokes `_mix_f32_bits_to_payload_u32`, `np.uint64`, `_expected_exp2_i32`, `_unmix_payload_u32_to_f32_bits_i32` to execute the test logic.
- **CN:** 通过 `x`、`rcp_log2`、`scaled` 准备或更新状态。 调用 `_mix_f32_bits_to_payload_u32`、`np.uint64`、`_expected_exp2_i32`、`_unmix_payload_u32_to_f32_bits_i32` 执行测试逻辑。

### Lines 257-259

```python


def _expected_cossin_payload_u32(x_u32: np.ndarray) -> tuple[np.ndarray, np.ndarray]:
```
- **EN:** Defines the helper function `_expected_cossin_payload_u32`. Parameters: `x_u32`. Key calls include `np.uint64`, `x_u32.astype`, `np.ones_like`, `np.zeros_like`, `np.errstate`, `c.astype`, and 3 more.
- **CN:** 定义辅助函数 `_expected_cossin_payload_u32`。 参数：`x_u32`。 关键调用包括 `np.uint64`、`x_u32.astype`、`np.ones_like`、`np.zeros_like`、`np.errstate`、`c.astype` 等另外 3 项。

#### Lines 260-266

```python
    mask = np.uint64(0xFFFFFFFF)
    rcp5 = int(_inv_odd_u64(np.uint64(5)) & mask)
    a = np.uint64((-3 * rcp5) & 0xFFFFFFFF)
    b = np.uint64((4 * rcp5) & 0xFFFFFFFF)
    x = x_u32.astype(np.uint64)
    c = np.ones_like(x, dtype=np.uint64)
    s = np.zeros_like(x, dtype=np.uint64)
```
- **EN:** Prepares or updates state through `mask`, `rcp5`, `a`, `b`, `x`, `c`, `s`. Invokes `np.uint64`, `_inv_odd_u64`, `x_u32.astype`, `np.ones_like`, `np.zeros_like` to execute the test logic.
- **CN:** 通过 `mask`、`rcp5`、`a`、`b`、`x`、`c`、`s` 准备或更新状态。 调用 `np.uint64`、`_inv_odd_u64`、`x_u32.astype`、`np.ones_like`、`np.zeros_like` 执行测试逻辑。

#### Lines 267-275

```python
    with np.errstate(over="ignore"):
        for i in range(32):
            c_double = (c * c - s * s) & mask
            s_double = (np.uint64(2) * c * s) & mask
            c_inc = (a * c_double - b * s_double) & mask
            s_inc = (a * s_double + b * c_double) & mask
            inc = (x & np.uint64(1 << (31 - i))) != 0
            c = np.where(inc, c_inc, c_double) & mask
            s = np.where(inc, s_inc, s_double) & mask
```
- **EN:** Invokes `np.errstate`, `np.where`, `np.uint64` to execute the test logic. Uses context managers to control resources or expectations.
- **CN:** 调用 `np.errstate`、`np.where`、`np.uint64` 执行测试逻辑。 使用上下文管理器控制资源或预期行为。

#### Lines 276-276

```python
    return c.astype(np.uint32), s.astype(np.uint32)
```
- **EN:** Invokes `c.astype`, `s.astype` to execute the test logic.
- **CN:** 调用 `c.astype`、`s.astype` 执行测试逻辑。

### Lines 277-279

```python


def _expected_cos_i32(x_i32: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `_expected_cos_i32`. Parameters: `x_i32`. Key calls include `_expected_cossin_payload_u32`, `_unmix_payload_u32_to_f32_bits_i32`, `_mix_f32_bits_to_payload_u32`.
- **CN:** 定义辅助函数 `_expected_cos_i32`。 参数：`x_i32`。 关键调用包括 `_expected_cossin_payload_u32`、`_unmix_payload_u32_to_f32_bits_i32`、`_mix_f32_bits_to_payload_u32`。

#### Lines 280-281

```python
    c, _ = _expected_cossin_payload_u32(_mix_f32_bits_to_payload_u32(x_i32))
    return _unmix_payload_u32_to_f32_bits_i32(c)
```
- **EN:** Prepares or updates state through `c`, `_`. Invokes `_expected_cossin_payload_u32`, `_mix_f32_bits_to_payload_u32`, `_unmix_payload_u32_to_f32_bits_i32` to execute the test logic.
- **CN:** 通过 `c`、`_` 准备或更新状态。 调用 `_expected_cossin_payload_u32`、`_mix_f32_bits_to_payload_u32`、`_unmix_payload_u32_to_f32_bits_i32` 执行测试逻辑。

### Lines 282-284

```python


def _expected_sin_i32(x_i32: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `_expected_sin_i32`. Parameters: `x_i32`. Key calls include `_expected_cossin_payload_u32`, `_unmix_payload_u32_to_f32_bits_i32`, `_mix_f32_bits_to_payload_u32`.
- **CN:** 定义辅助函数 `_expected_sin_i32`。 参数：`x_i32`。 关键调用包括 `_expected_cossin_payload_u32`、`_unmix_payload_u32_to_f32_bits_i32`、`_mix_f32_bits_to_payload_u32`。

#### Lines 285-286

```python
    _, s = _expected_cossin_payload_u32(_mix_f32_bits_to_payload_u32(x_i32))
    return _unmix_payload_u32_to_f32_bits_i32(s)
```
- **EN:** Prepares or updates state through `_`, `s`. Invokes `_expected_cossin_payload_u32`, `_mix_f32_bits_to_payload_u32`, `_unmix_payload_u32_to_f32_bits_i32` to execute the test logic.
- **CN:** 通过 `_`、`s` 准备或更新状态。 调用 `_expected_cossin_payload_u32`、`_mix_f32_bits_to_payload_u32`、`_unmix_payload_u32_to_f32_bits_i32` 执行测试逻辑。

### Lines 287-290

```python


def _expected_unary_tag_i32(x_i32: np.ndarray, op: str) -> np.ndarray:
    # Keep this mapping in sync with UnaryOpId in FpSanitizer.cpp.
```
- **EN:** Defines the helper function `_expected_unary_tag_i32`. Parameters: `x_i32`, `op`. Key calls include `_expected_unary_tag_payload_u32`, `_unmix_payload_u32_to_f32_bits_i32`, `_mix_f32_bits_to_payload_u32`.
- **CN:** 定义辅助函数 `_expected_unary_tag_i32`。 参数：`x_i32`、`op`。 关键调用包括 `_expected_unary_tag_payload_u32`、`_unmix_payload_u32_to_f32_bits_i32`、`_mix_f32_bits_to_payload_u32`。

#### Lines 291-292

```python
    out_u32 = _expected_unary_tag_payload_u32(_mix_f32_bits_to_payload_u32(x_i32), op)
    return _unmix_payload_u32_to_f32_bits_i32(out_u32)
```
- **EN:** Prepares or updates state through `out_u32`. Invokes `_expected_unary_tag_payload_u32`, `_mix_f32_bits_to_payload_u32`, `_unmix_payload_u32_to_f32_bits_i32` to execute the test logic.
- **CN:** 通过 `out_u32` 准备或更新状态。 调用 `_expected_unary_tag_payload_u32`、`_mix_f32_bits_to_payload_u32`、`_unmix_payload_u32_to_f32_bits_i32` 执行测试逻辑。

### Lines 293-295

```python


def stable_string_hash_u64(s: str) -> np.uint64:
```
- **EN:** Defines the helper function `stable_string_hash_u64`. Parameters: `s`. Key calls include `np.errstate`, `np.uint64`, `s.encode`.
- **CN:** 定义辅助函数 `stable_string_hash_u64`。 参数：`s`。 关键调用包括 `np.errstate`、`np.uint64`、`s.encode`。

#### Lines 296-301

```python
    with np.errstate(over="ignore"):
        h = np.uint64(14695981039346656037)
        for c in s.encode("utf-8"):
            h ^= np.uint64(c)
            h = np.uint64(h * np.uint64(1099511628211))
        return h
```
- **EN:** Invokes `np.errstate`, `np.uint64`, `s.encode` to execute the test logic. Uses context managers to control resources or expectations.
- **CN:** 调用 `np.errstate`、`np.uint64`、`s.encode` 执行测试逻辑。 使用上下文管理器控制资源或预期行为。

### Lines 302-304

```python


def _expected_extern_unary_tag_i32(x_i32: np.ndarray, symbol: str) -> np.ndarray:
```
- **EN:** Defines the helper function `_expected_extern_unary_tag_i32`. Parameters: `x_i32`, `symbol`. Key calls include `_expected_extern_variadic_tag_i32`.
- **CN:** 定义辅助函数 `_expected_extern_unary_tag_i32`。 参数：`x_i32`、`symbol`。 关键调用包括 `_expected_extern_variadic_tag_i32`。

#### Lines 305-305

```python
    return _expected_extern_variadic_tag_i32([x_i32], symbol)
```
- **EN:** Invokes `_expected_extern_variadic_tag_i32` to execute the test logic.
- **CN:** 调用 `_expected_extern_variadic_tag_i32` 执行测试逻辑。

### Lines 306-308

```python


def _expected_extern_binary_tag_i32(x_i32: np.ndarray, y_i32: np.ndarray, symbol: str) -> np.ndarray:
```
- **EN:** Defines the helper function `_expected_extern_binary_tag_i32`. Parameters: `x_i32`, `y_i32`, `symbol`. Key calls include `_expected_extern_variadic_tag_i32`.
- **CN:** 定义辅助函数 `_expected_extern_binary_tag_i32`。 参数：`x_i32`、`y_i32`、`symbol`。 关键调用包括 `_expected_extern_variadic_tag_i32`。

#### Lines 309-309

```python
    return _expected_extern_variadic_tag_i32([x_i32, y_i32], symbol)
```
- **EN:** Invokes `_expected_extern_variadic_tag_i32` to execute the test logic.
- **CN:** 调用 `_expected_extern_variadic_tag_i32` 执行测试逻辑。

### Lines 310-312

```python


def _rotl_u32(x_u32: np.ndarray, amount: int) -> np.ndarray:
```
- **EN:** Defines the helper function `_rotl_u32`. Parameters: `x_u32`, `amount`. Key calls include `x_u32.astype`, `out_u32.astype`, `np.uint64`.
- **CN:** 定义辅助函数 `_rotl_u32`。 参数：`x_u32`、`amount`。 关键调用包括 `x_u32.astype`、`out_u32.astype`、`np.uint64`。

#### Lines 313-313

```python
    amount = amount & 31
```
- **EN:** Prepares or updates state through `amount`.
- **CN:** 通过 `amount` 准备或更新状态。

#### Lines 314-315

```python
    if amount == 0:
        return x_u32
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 316-318

```python
    x = x_u32.astype(np.uint64)
    out_u32 = ((x << np.uint64(amount)) | (x >> np.uint64(32 - amount))) & np.uint64(0xFFFFFFFF)
    return out_u32.astype(np.uint32)
```
- **EN:** Prepares or updates state through `x`, `out_u32`. Invokes `x_u32.astype`, `np.uint64`, `out_u32.astype` to execute the test logic.
- **CN:** 通过 `x`、`out_u32` 准备或更新状态。 调用 `x_u32.astype`、`np.uint64`、`out_u32.astype` 执行测试逻辑。

### Lines 319-321

```python


def _expected_extern_variadic_tag_i32(args_i32: list[np.ndarray], symbol: str, float_args=None) -> np.ndarray:
```
- **EN:** Defines the helper function `_expected_extern_variadic_tag_i32`. Parameters: `args_i32`, `symbol`, `float_args`. Key calls include `np.uint64`, `np.zeros_like`, `enumerate`, `_unmix_payload_u32_to_f32_bits_i32`, `_as_u32`, `zip`, and 3 more.
- **CN:** 定义辅助函数 `_expected_extern_variadic_tag_i32`。 参数：`args_i32`、`symbol`、`float_args`。 关键调用包括 `np.uint64`、`np.zeros_like`、`enumerate`、`_unmix_payload_u32_to_f32_bits_i32`、`_as_u32`、`zip` 等另外 3 项。

#### Lines 322-323

```python
    if float_args is None:
        float_args = [True] * len(args_i32)
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 324-325

```python
    tag = np.uint64(stable_string_hash_u64(symbol) & np.uint64(0xFFFFFFFF))
    total_u64 = np.zeros_like(_as_u32(args_i32[0]), dtype=np.uint64)
```
- **EN:** Prepares or updates state through `tag`, `total_u64`. Invokes `np.uint64`, `stable_string_hash_u64`, `np.zeros_like`, `_as_u32` to execute the test logic.
- **CN:** 通过 `tag`、`total_u64` 准备或更新状态。 调用 `np.uint64`、`stable_string_hash_u64`、`np.zeros_like`、`_as_u32` 执行测试逻辑。

#### Lines 326-329

```python
    for i, (arg, is_float) in enumerate(zip(args_i32, float_args)):
        arg_u32 = _mix_f32_bits_to_payload_u32(arg) if is_float else _as_u32(arg)
        rotated = _rotl_u32(arg_u32, i).astype(np.uint64)
        total_u64 = (total_u64 + rotated) & np.uint64(0xFFFFFFFF)
```
- **EN:** Invokes `enumerate`, `zip`, `_mix_f32_bits_to_payload_u32`, `_as_u32`, `np.uint64`, `_rotl_u32` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `enumerate`、`zip`、`_mix_f32_bits_to_payload_u32`、`_as_u32`、`np.uint64`、`_rotl_u32` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 330-331

```python
    out_u32 = (total_u64 ^ tag).astype(np.uint32)
    return _unmix_payload_u32_to_f32_bits_i32(out_u32)
```
- **EN:** Prepares or updates state through `out_u32`. Invokes `_unmix_payload_u32_to_f32_bits_i32` to execute the test logic.
- **CN:** 通过 `out_u32` 准备或更新状态。 调用 `_unmix_payload_u32_to_f32_bits_i32` 执行测试逻辑。

### Lines 332-379

```python


UNARY_EXTERN_SYMBOLS = {
    "cuda": [
        ("tan", "__nv_tanf"),
        ("tanh", "__nv_tanhf"),
        ("log1p", "__nv_log1pf"),
        ("cbrt", "__nv_cbrtf"),
        ("round", "__nv_roundf"),
    ],
    "hip": [
        ("tan", "__ocml_tan_f32"),
        ("tanh", "__ocml_tanh_f32"),
        ("log1p", "__ocml_log1p_f32"),
        ("round", "__ocml_round_f32"),
    ],
}

BINARY_EXTERN_SYMBOLS = {
    "cuda": [
        ("atan2", "__nv_atan2f"),
        ("hypot", "__nv_hypotf"),
        ("pow", "__nv_powf"),
    ],
    "hip": [
        ("atan2", "__ocml_atan2_f32"),
        ("hypot", "__ocml_hypot_f32"),
        ("pow", "__ocml_pow_f32"),
    ],
}

TERNARY_EXTERN_SYMBOLS = {
    "cuda": [
        ("fma", "__nv_fmaf"),
    ],
    "hip": [
        ("fma", "__ocml_fma_f32"),
    ],
}

MIXED_EXTERN_SYMBOLS = {
    "cuda": [
        ("ldexp", "__nv_ldexpf"),
    ],
    "hip": [
        ("ldexp", "__ocml_ldexp_f32"),
    ],
}
```
- **EN:** Prepares or updates state through `UNARY_EXTERN_SYMBOLS`, `BINARY_EXTERN_SYMBOLS`, `TERNARY_EXTERN_SYMBOLS`, `MIXED_EXTERN_SYMBOLS`.
- **CN:** 通过 `UNARY_EXTERN_SYMBOLS`、`BINARY_EXTERN_SYMBOLS`、`TERNARY_EXTERN_SYMBOLS`、`MIXED_EXTERN_SYMBOLS` 准备或更新状态。

### Lines 380-382

```python


def _extern_backend_name() -> str:
```
- **EN:** Defines the helper function `_extern_backend_name`. Key calls include `is_hip`.
- **CN:** 定义辅助函数 `_extern_backend_name`。 关键调用包括 `is_hip`。

#### Lines 383-384

```python
    if is_hip():
        return "hip"
```
- **EN:** Invokes `is_hip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `is_hip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 385-385

```python
    return "cuda"
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 386-391

```python


EXTERN_UNARY_CASES = UNARY_EXTERN_SYMBOLS[_extern_backend_name()]
EXTERN_BINARY_CASES = BINARY_EXTERN_SYMBOLS[_extern_backend_name()]
EXTERN_TERNARY_CASES = TERNARY_EXTERN_SYMBOLS[_extern_backend_name()]
EXTERN_MIXED_CASES = MIXED_EXTERN_SYMBOLS[_extern_backend_name()]
```
- **EN:** Prepares or updates state through `EXTERN_UNARY_CASES`, `EXTERN_BINARY_CASES`, `EXTERN_TERNARY_CASES`, `EXTERN_MIXED_CASES`. Invokes `_extern_backend_name` to execute the test logic.
- **CN:** 通过 `EXTERN_UNARY_CASES`、`EXTERN_BINARY_CASES`、`EXTERN_TERNARY_CASES`、`EXTERN_MIXED_CASES` 准备或更新状态。 调用 `_extern_backend_name` 执行测试逻辑。

### Lines 392-394

```python


def _as_payload_np_i32(x) -> np.ndarray:
```
- **EN:** Defines the helper function `_as_payload_np_i32`. Parameters: `x`. Key calls include `isinstance`, `TypeError`, `x.astype`, `x.view`, `type`, `x.detach`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `_as_payload_np_i32`。 参数：`x`。 关键调用包括 `isinstance`、`TypeError`、`x.astype`、`x.view`、`type`、`x.detach`。 该作用域涉及PyTorch 张量准备与校验。

#### Lines 395-396

```python
    if isinstance(x, torch.Tensor):
        x = x.detach().cpu().numpy()
```
- **EN:** Invokes `isinstance`, `x.detach` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `isinstance`、`x.detach` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

#### Lines 397-398

```python
    if not isinstance(x, np.ndarray):
        raise TypeError(f"unsupported input type: {type(x)}")
```
- **EN:** Invokes `isinstance`, `TypeError`, `type` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `isinstance`、`TypeError`、`type` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 399-400

```python
    if x.dtype == np.int32:
        return x.astype(np.int32, copy=False)
```
- **EN:** Invokes `x.astype` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `x.astype` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 401-402

```python
    if x.dtype == np.uint32:
        return x.view(np.int32)
```
- **EN:** Invokes `x.view` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `x.view` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 403-404

```python
    if x.dtype == np.float32:
        return x.view(np.int32)
```
- **EN:** Invokes `x.view` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `x.view` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 405-405

```python
    raise TypeError(f"unsupported dtype for payload comparison: {x.dtype}")
```
- **EN:** Invokes `TypeError` to execute the test logic.
- **CN:** 调用 `TypeError` 执行测试逻辑。

### Lines 406-408

```python


def _assert_payload_equal(actual, expected) -> None:
```
- **EN:** Defines the helper function `_assert_payload_equal`. Parameters: `actual`, `expected`. Key calls include `np.testing.assert_array_equal`, `_as_payload_np_i32`.
- **CN:** 定义辅助函数 `_assert_payload_equal`。 参数：`actual`、`expected`。 关键调用包括 `np.testing.assert_array_equal`、`_as_payload_np_i32`。

#### Lines 409-409

```python
    np.testing.assert_array_equal(_as_payload_np_i32(actual), _as_payload_np_i32(expected))
```
- **EN:** Invokes `np.testing.assert_array_equal`, `_as_payload_np_i32` to execute the test logic.
- **CN:** 调用 `np.testing.assert_array_equal`、`_as_payload_np_i32` 执行测试逻辑。

### Lines 410-412

```python


def _payload_equal(a, b) -> bool:
```
- **EN:** Defines the helper function `_payload_equal`. Parameters: `a`, `b`. Key calls include `np.array_equal`, `_as_payload_np_i32`.
- **CN:** 定义辅助函数 `_payload_equal`。 参数：`a`、`b`。 关键调用包括 `np.array_equal`、`_as_payload_np_i32`。

#### Lines 413-413

```python
    return np.array_equal(_as_payload_np_i32(a), _as_payload_np_i32(b))
```
- **EN:** Invokes `np.array_equal`, `_as_payload_np_i32` to execute the test logic.
- **CN:** 调用 `np.array_equal`、`_as_payload_np_i32` 执行测试逻辑。

### Lines 414-418

```python


@gluon.jit
def _binop_kernel(x_ptr, y_ptr, out_ptr, n_elements, OP: gl.constexpr, BLOCK: gl.constexpr,
                  THREADS_PER_WARP: gl.constexpr):
```
- **EN:** Defines the helper function `_binop_kernel`. Decorators: `gluon.jit`. Parameters: `x_ptr`, `y_ptr`, `out_ptr`, `n_elements`, `OP`, `BLOCK`, `THREADS_PER_WARP`. Key calls include `gl.program_id`, `gl.BlockedLayout`, `gl.load`, `gl.store`, `gl.arange`, `gl.minimum`, and 3 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `_binop_kernel`。 装饰器：`gluon.jit`。 参数：`x_ptr`、`y_ptr`、`out_ptr`、`n_elements`、`OP`、`BLOCK`、`THREADS_PER_WARP`。 关键调用包括 `gl.program_id`、`gl.BlockedLayout`、`gl.load`、`gl.store`、`gl.arange`、`gl.minimum` 等另外 3 项。 该作用域涉及布局变换推理。

#### Lines 419-425

```python
    pid = gl.program_id(0)
    layout: gl.constexpr = gl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4],
                                            order=[0])
    offs = pid * BLOCK + gl.arange(0, BLOCK, layout=layout)
    mask = offs < n_elements
    x = gl.load(x_ptr + offs, mask=mask, other=0.0)
    y = gl.load(y_ptr + offs, mask=mask, other=0.0)
```
- **EN:** Prepares or updates state through `pid`, `layout`, `offs`, `mask`, `x`, `y`. Invokes `gl.program_id`, `gl.BlockedLayout`, `gl.arange`, `gl.load` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `pid`、`layout`、`offs`、`mask`、`x`、`y` 准备或更新状态。 调用 `gl.program_id`、`gl.BlockedLayout`、`gl.arange`、`gl.load` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 426-444

```python

    if OP == "add":
        z = x + y
    elif OP == "sub":
        z = x - y
    elif OP == "mul":
        z = x * y
    elif OP == "min":
        z = gl.minimum(x, y)
    elif OP == "max":
        z = gl.maximum(x, y)
    elif OP == "truediv":
        z = x / y
    elif OP == "fdiv":
        z = gl.fdiv(x, y)
    elif OP == "mod":
        z = x % y
    else:
        gl.static_assert(False, "unsupported OP")
```
- **EN:** Invokes `gl.minimum`, `gl.maximum`, `gl.fdiv`, `gl.static_assert` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `gl.minimum`、`gl.maximum`、`gl.fdiv`、`gl.static_assert` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 445-446

```python

    gl.store(out_ptr + offs, z, mask=mask)
```
- **EN:** Invokes `gl.store` to execute the test logic.
- **CN:** 调用 `gl.store` 执行测试逻辑。

### Lines 447-451

```python


@gluon.jit
def _constant_identity_kernel(x_ptr, out_ptr, n_elements, OP: gl.constexpr, BLOCK: gl.constexpr,
                              THREADS_PER_WARP: gl.constexpr):
```
- **EN:** Defines the helper function `_constant_identity_kernel`. Decorators: `gluon.jit`. Parameters: `x_ptr`, `out_ptr`, `n_elements`, `OP`, `BLOCK`, `THREADS_PER_WARP`. Key calls include `gl.program_id`, `gl.BlockedLayout`, `gl.load`, `gl.store`, `gl.arange`, `gl.static_assert`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `_constant_identity_kernel`。 装饰器：`gluon.jit`。 参数：`x_ptr`、`out_ptr`、`n_elements`、`OP`、`BLOCK`、`THREADS_PER_WARP`。 关键调用包括 `gl.program_id`、`gl.BlockedLayout`、`gl.load`、`gl.store`、`gl.arange`、`gl.static_assert`。 该作用域涉及布局变换推理。

#### Lines 452-457

```python
    pid = gl.program_id(0)
    layout: gl.constexpr = gl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4],
                                            order=[0])
    offs = pid * BLOCK + gl.arange(0, BLOCK, layout=layout)
    mask = offs < n_elements
    x = gl.load(x_ptr + offs, mask=mask, other=0.0)
```
- **EN:** Prepares or updates state through `pid`, `layout`, `offs`, `mask`, `x`. Invokes `gl.program_id`, `gl.BlockedLayout`, `gl.arange`, `gl.load` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `pid`、`layout`、`offs`、`mask`、`x` 准备或更新状态。 调用 `gl.program_id`、`gl.BlockedLayout`、`gl.arange`、`gl.load` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 458-464

```python

    if OP == "mul_one":
        z = x * 1.0
    elif OP == "add_zero":
        z = x + 0.0
    else:
        gl.static_assert(False, "unsupported OP")
```
- **EN:** Invokes `gl.static_assert` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `gl.static_assert` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 465-466

```python

    gl.store(out_ptr + offs, z, mask=mask)
```
- **EN:** Invokes `gl.store` to execute the test logic.
- **CN:** 调用 `gl.store` 执行测试逻辑。

### Lines 467-470

```python


@gluon.jit
def _reciprocal_involution_kernel(x_ptr, out_ptr, n_elements, BLOCK: gl.constexpr, THREADS_PER_WARP: gl.constexpr):
```
- **EN:** Defines the helper function `_reciprocal_involution_kernel`. Decorators: `gluon.jit`. Parameters: `x_ptr`, `out_ptr`, `n_elements`, `BLOCK`, `THREADS_PER_WARP`. Key calls include `gl.program_id`, `gl.BlockedLayout`, `gl.load`, `gl.store`, `gl.arange`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `_reciprocal_involution_kernel`。 装饰器：`gluon.jit`。 参数：`x_ptr`、`out_ptr`、`n_elements`、`BLOCK`、`THREADS_PER_WARP`。 关键调用包括 `gl.program_id`、`gl.BlockedLayout`、`gl.load`、`gl.store`、`gl.arange`。 该作用域涉及布局变换推理。

#### Lines 471-478

```python
    pid = gl.program_id(0)
    layout: gl.constexpr = gl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4],
                                            order=[0])
    offs = pid * BLOCK + gl.arange(0, BLOCK, layout=layout)
    mask = offs < n_elements
    x = gl.load(x_ptr + offs, mask=mask, other=0.0)
    z = 1.0 / (1.0 / x)
    gl.store(out_ptr + offs, z, mask=mask)
```
- **EN:** Prepares or updates state through `pid`, `layout`, `offs`, `mask`, `x`, `z`. Invokes `gl.program_id`, `gl.BlockedLayout`, `gl.arange`, `gl.load`, `gl.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `pid`、`layout`、`offs`、`mask`、`x`、`z` 准备或更新状态。 调用 `gl.program_id`、`gl.BlockedLayout`、`gl.arange`、`gl.load`、`gl.store` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 479-482

```python


@pytest.mark.parametrize("op", ["mul_one", "add_zero"])
def test_constant_identity_noop(device, op, fresh_knobs):
```
- **EN:** Defines the test function `test_constant_identity_noop`. Decorators: `pytest.mark.parametrize('op', ['mul_one', 'add_zero'])`. Parameters: `device`, `op`, `fresh_knobs`. Key calls include `pytest.mark.parametrize`, `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, and 3 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义测试函数 `test_constant_identity_noop`。 装饰器：`pytest.mark.parametrize('op', ['mul_one', 'add_zero'])`。 参数：`device`、`op`、`fresh_knobs`。 关键调用包括 `pytest.mark.parametrize`、`_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty` 等另外 3 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、随机数据生成。

#### Lines 483-505

```python
    _require_cuda_backend(device)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"

    n_elements = 1024
    BLOCK = 256

    g = torch.Generator(device="cuda")
    g.manual_seed(2)
    x = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    out = torch.empty((n_elements, ), dtype=torch.int32, device="cuda")

    grid = (triton.cdiv(n_elements, BLOCK), )
    _constant_identity_kernel[grid](
        triton.TensorWrapper(x, dtype=torch.float32),
        triton.TensorWrapper(out, dtype=torch.float32),
        n_elements,
        OP=op,
        BLOCK=BLOCK,
        THREADS_PER_WARP=THREADS_PER_WARP,
    )

    _assert_payload_equal(out, x)
```
- **EN:** Prepares or updates state through `fresh_knobs`, `n_elements`, `BLOCK`, `g`, `x`, `out`, `grid`. Invokes `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, `triton.cdiv`, and 2 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `fresh_knobs`、`n_elements`、`BLOCK`、`g`、`x`、`out`、`grid` 准备或更新状态。 调用 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty`、`triton.cdiv` 等另外 2 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 506-508

```python


def test_reciprocal_involution(device, fresh_knobs):
```
- **EN:** Defines the test function `test_reciprocal_involution`. Parameters: `device`, `fresh_knobs`. Key calls include `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, `_assert_payload_equal`, and 2 more. This scope touches PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义测试函数 `test_reciprocal_involution`。 参数：`device`、`fresh_knobs`。 关键调用包括 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty`、`_assert_payload_equal` 等另外 2 项。 该作用域涉及PyTorch 张量准备与校验、随机数据生成。

#### Lines 509-530

```python
    _require_cuda_backend(device)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"

    n_elements = 1024
    BLOCK = 256

    g = torch.Generator(device="cuda")
    g.manual_seed(3)
    x = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    out = torch.empty((n_elements, ), dtype=torch.int32, device="cuda")

    grid = (triton.cdiv(n_elements, BLOCK), )
    _reciprocal_involution_kernel[grid](
        triton.TensorWrapper(x, dtype=torch.float32),
        triton.TensorWrapper(out, dtype=torch.float32),
        n_elements,
        BLOCK=BLOCK,
        THREADS_PER_WARP=THREADS_PER_WARP,
    )

    _assert_payload_equal(out, x)
```
- **EN:** Prepares or updates state through `fresh_knobs`, `n_elements`, `BLOCK`, `g`, `x`, `out`, `grid`. Invokes `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, `triton.cdiv`, and 2 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `fresh_knobs`、`n_elements`、`BLOCK`、`g`、`x`、`out`、`grid` 准备或更新状态。 调用 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty`、`triton.cdiv` 等另外 2 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 531-546

```python


@pytest.mark.parametrize(
    "op,expected_fn",
    [
        ("add", _expected_add_i32),
        ("sub", _expected_sub_i32),
        ("mul", _expected_mul_i32),
        ("min", _expected_min_i32),
        ("max", _expected_max_i32),
        ("truediv", _expected_div_payload_i32),
        ("fdiv", _expected_div_payload_i32),
        ("mod", _expected_srem_i32),
    ],
)
def test_binops_payload_semantics(device, op, expected_fn, fresh_knobs):
```
- **EN:** Defines the test function `test_binops_payload_semantics`. Decorators: `pytest.mark.parametrize('op,expected_fn', [('add', _expected_add_i32), ('sub', _expected_sub_i32), ('mul', _expected_mul_i32), ('min', _expected_min_i32), ('max', _expected_max_i32), ('truediv', _expected_div_payload_i32), ('fdiv', _expected_div_payload_i32), ('mod', _expected_srem_i32)])`. Parameters: `device`, `op`, `expected_fn`, `fresh_knobs`. Key calls include `pytest.mark.parametrize`, `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, and 7 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义测试函数 `test_binops_payload_semantics`。 装饰器：`pytest.mark.parametrize('op,expected_fn', [('add', _expected_add_i32), ('sub', _expected_sub_i32), ('mul', _expected_mul_i32), ('min', _expected_min_i32), ('max', _expected_max_i32), ('truediv', _expected_div_payload_i32), ('fdiv', _expected_div_payload_i32), ('mod', _expected_srem_i32)])`。 参数：`device`、`op`、`expected_fn`、`fresh_knobs`。 关键调用包括 `pytest.mark.parametrize`、`_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty` 等另外 7 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、随机数据生成。

#### Lines 547-570

```python
    _require_cuda_backend(device)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"

    # Use int32 storage but treat it as float32 via TensorWrapper so fpsan operates on payload bits.
    n_elements = 1024
    BLOCK = 256

    g = torch.Generator(device="cuda")
    g.manual_seed(0)
    x = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    y = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    out = torch.empty((n_elements, ), dtype=torch.int32, device="cuda")

    xw = triton.TensorWrapper(x, dtype=torch.float32)
    yw = triton.TensorWrapper(y, dtype=torch.float32)
    outw = triton.TensorWrapper(out, dtype=torch.float32)

    grid = (triton.cdiv(n_elements, BLOCK), )
    _binop_kernel[grid](xw, yw, outw, n_elements, OP=op, BLOCK=BLOCK, THREADS_PER_WARP=THREADS_PER_WARP)

    out_np = out.cpu().numpy().astype(np.int32, copy=False)
    exp_np = expected_fn(x.cpu().numpy().astype(np.int32, copy=False), y.cpu().numpy().astype(np.int32, copy=False))
    _assert_payload_equal(out_np, exp_np)
```
- **EN:** Prepares or updates state through `fresh_knobs`, `n_elements`, `BLOCK`, `g`, `x`, `y`, `out`, `xw`, and 5 more. Invokes `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, `triton.TensorWrapper`, and 6 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `fresh_knobs`、`n_elements`、`BLOCK`、`g`、`x`、`y`、`out`、`xw` 等另外 5 项 准备或更新状态。 调用 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty`、`triton.TensorWrapper` 等另外 6 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 571-581

```python


@pytest.mark.parametrize(
    "op,expected_fn",
    [
        ("truediv", _expected_div_payload_i32),
        ("fdiv", _expected_div_payload_i32),
        ("mod", _expected_srem_i32),
    ],
)
def test_binops_payload_semantics_zero_denominator(device, op, expected_fn, fresh_knobs):
```
- **EN:** Defines the test function `test_binops_payload_semantics_zero_denominator`. Decorators: `pytest.mark.parametrize('op,expected_fn', [('truediv', _expected_div_payload_i32), ('fdiv', _expected_div_payload_i32), ('mod', _expected_srem_i32)])`. Parameters: `device`, `op`, `expected_fn`, `fresh_knobs`. Key calls include `pytest.mark.parametrize`, `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, and 7 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义测试函数 `test_binops_payload_semantics_zero_denominator`。 装饰器：`pytest.mark.parametrize('op,expected_fn', [('truediv', _expected_div_payload_i32), ('fdiv', _expected_div_payload_i32), ('mod', _expected_srem_i32)])`。 参数：`device`、`op`、`expected_fn`、`fresh_knobs`。 关键调用包括 `pytest.mark.parametrize`、`_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty` 等另外 7 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、随机数据生成。

#### Lines 582-606

```python
    _require_cuda_backend(device)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"

    n_elements = 1024
    BLOCK = 256

    g = torch.Generator(device="cuda")
    g.manual_seed(123)
    x = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    y = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    y[::7] = 0

    out = torch.empty((n_elements, ), dtype=torch.int32, device="cuda")

    xw = triton.TensorWrapper(x, dtype=torch.float32)
    yw = triton.TensorWrapper(y, dtype=torch.float32)
    outw = triton.TensorWrapper(out, dtype=torch.float32)

    grid = (triton.cdiv(n_elements, BLOCK), )
    _binop_kernel[grid](xw, yw, outw, n_elements, OP=op, BLOCK=BLOCK, THREADS_PER_WARP=THREADS_PER_WARP)

    out_np = out.cpu().numpy().astype(np.int32, copy=False)
    exp_np = expected_fn(x.cpu().numpy().astype(np.int32, copy=False), y.cpu().numpy().astype(np.int32, copy=False))
    _assert_payload_equal(out_np, exp_np)
```
- **EN:** Prepares or updates state through `fresh_knobs`, `n_elements`, `BLOCK`, `g`, `x`, `y`, `out`, `xw`, and 5 more. Invokes `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, `triton.TensorWrapper`, and 6 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `fresh_knobs`、`n_elements`、`BLOCK`、`g`、`x`、`y`、`out`、`xw` 等另外 5 项 准备或更新状态。 调用 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty`、`triton.TensorWrapper` 等另外 6 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 607-611

```python


@gluon.jit
def _unary_math_kernel(x_ptr, out_ptr, n_elements, OP: gl.constexpr, BLOCK: gl.constexpr,
                       THREADS_PER_WARP: gl.constexpr):
```
- **EN:** Defines the helper function `_unary_math_kernel`. Decorators: `gluon.jit`. Parameters: `x_ptr`, `out_ptr`, `n_elements`, `OP`, `BLOCK`, `THREADS_PER_WARP`. Key calls include `gl.program_id`, `gl.BlockedLayout`, `gl.load`, `gl.store`, `gl.arange`, `getattr`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `_unary_math_kernel`。 装饰器：`gluon.jit`。 参数：`x_ptr`、`out_ptr`、`n_elements`、`OP`、`BLOCK`、`THREADS_PER_WARP`。 关键调用包括 `gl.program_id`、`gl.BlockedLayout`、`gl.load`、`gl.store`、`gl.arange`、`getattr`。 该作用域涉及布局变换推理。

#### Lines 612-617

```python
    pid = gl.program_id(0)
    layout: gl.constexpr = gl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4],
                                            order=[0])
    offs = pid * BLOCK + gl.arange(0, BLOCK, layout=layout)
    mask = offs < n_elements
    x = gl.load(x_ptr + offs, mask=mask, other=0.0)
```
- **EN:** Prepares or updates state through `pid`, `layout`, `offs`, `mask`, `x`. Invokes `gl.program_id`, `gl.BlockedLayout`, `gl.arange`, `gl.load` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `pid`、`layout`、`offs`、`mask`、`x` 准备或更新状态。 调用 `gl.program_id`、`gl.BlockedLayout`、`gl.arange`、`gl.load` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 618-621

```python
    if OP == "neg":
        z = -x
    else:
        z = getattr(gl, OP)(x)
```
- **EN:** Invokes `getattr` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `getattr` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 622-622

```python
    gl.store(out_ptr + offs, z, mask=mask)
```
- **EN:** Invokes `gl.store` to execute the test logic.
- **CN:** 调用 `gl.store` 执行测试逻辑。

### Lines 623-627

```python


@gluon.jit
def _exp_binary_identity_kernel(x_ptr, y_ptr, out_ptr, n_elements, MODE: gl.constexpr, BLOCK: gl.constexpr,
                                THREADS_PER_WARP: gl.constexpr):
```
- **EN:** Defines the helper function `_exp_binary_identity_kernel`. Decorators: `gluon.jit`. Parameters: `x_ptr`, `y_ptr`, `out_ptr`, `n_elements`, `MODE`, `BLOCK`, `THREADS_PER_WARP`. Key calls include `gl.program_id`, `gl.BlockedLayout`, `gl.load`, `gl.store`, `gl.arange`, `gl.exp`, and 1 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `_exp_binary_identity_kernel`。 装饰器：`gluon.jit`。 参数：`x_ptr`、`y_ptr`、`out_ptr`、`n_elements`、`MODE`、`BLOCK`、`THREADS_PER_WARP`。 关键调用包括 `gl.program_id`、`gl.BlockedLayout`、`gl.load`、`gl.store`、`gl.arange`、`gl.exp` 等另外 1 项。 该作用域涉及布局变换推理。

#### Lines 628-634

```python
    pid = gl.program_id(0)
    layout: gl.constexpr = gl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4],
                                            order=[0])
    offs = pid * BLOCK + gl.arange(0, BLOCK, layout=layout)
    mask = offs < n_elements
    x = gl.load(x_ptr + offs, mask=mask, other=0.0)
    y = gl.load(y_ptr + offs, mask=mask, other=0.0)
```
- **EN:** Prepares or updates state through `pid`, `layout`, `offs`, `mask`, `x`, `y`. Invokes `gl.program_id`, `gl.BlockedLayout`, `gl.arange`, `gl.load` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `pid`、`layout`、`offs`、`mask`、`x`、`y` 准备或更新状态。 调用 `gl.program_id`、`gl.BlockedLayout`、`gl.arange`、`gl.load` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 635-640

```python
    if MODE == "exp_add":
        z = gl.exp(x + y)
    elif MODE == "exp_mul":
        z = gl.exp(x) * gl.exp(y)
    else:
        gl.static_assert(False, "unsupported MODE")
```
- **EN:** Invokes `gl.exp`, `gl.static_assert` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `gl.exp`、`gl.static_assert` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 641-641

```python
    gl.store(out_ptr + offs, z, mask=mask)
```
- **EN:** Invokes `gl.store` to execute the test logic.
- **CN:** 调用 `gl.store` 执行测试逻辑。

### Lines 642-646

```python


@gluon.jit
def _exp_scaled_identity_kernel(x_ptr, out_ptr, n_elements, MODE: gl.constexpr, BLOCK: gl.constexpr,
                                THREADS_PER_WARP: gl.constexpr):
```
- **EN:** Defines the helper function `_exp_scaled_identity_kernel`. Decorators: `gluon.jit`. Parameters: `x_ptr`, `out_ptr`, `n_elements`, `MODE`, `BLOCK`, `THREADS_PER_WARP`. Key calls include `gl.program_id`, `gl.BlockedLayout`, `gl.load`, `gl.store`, `gl.arange`, `gl.exp`, and 2 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `_exp_scaled_identity_kernel`。 装饰器：`gluon.jit`。 参数：`x_ptr`、`out_ptr`、`n_elements`、`MODE`、`BLOCK`、`THREADS_PER_WARP`。 关键调用包括 `gl.program_id`、`gl.BlockedLayout`、`gl.load`、`gl.store`、`gl.arange`、`gl.exp` 等另外 2 项。 该作用域涉及布局变换推理。

#### Lines 647-652

```python
    pid = gl.program_id(0)
    layout: gl.constexpr = gl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4],
                                            order=[0])
    offs = pid * BLOCK + gl.arange(0, BLOCK, layout=layout)
    mask = offs < n_elements
    x = gl.load(x_ptr + offs, mask=mask, other=0.0)
```
- **EN:** Prepares or updates state through `pid`, `layout`, `offs`, `mask`, `x`. Invokes `gl.program_id`, `gl.BlockedLayout`, `gl.arange`, `gl.load` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `pid`、`layout`、`offs`、`mask`、`x` 准备或更新状态。 调用 `gl.program_id`、`gl.BlockedLayout`、`gl.arange`、`gl.load` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 653-658

```python
    if MODE == "exp":
        z = gl.exp(x)
    elif MODE == "exp2_scaled":
        z = gl.exp2(x * 1.44269504)
    else:
        gl.static_assert(False, "unsupported MODE")
```
- **EN:** Invokes `gl.exp`, `gl.exp2`, `gl.static_assert` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `gl.exp`、`gl.exp2`、`gl.static_assert` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 659-659

```python
    gl.store(out_ptr + offs, z, mask=mask)
```
- **EN:** Invokes `gl.store` to execute the test logic.
- **CN:** 调用 `gl.store` 执行测试逻辑。

### Lines 660-664

```python


@gluon.jit
def _exp_inverse_identity_kernel(x_ptr, out_ptr, n_elements, MODE: gl.constexpr, BLOCK: gl.constexpr,
                                 THREADS_PER_WARP: gl.constexpr):
```
- **EN:** Defines the helper function `_exp_inverse_identity_kernel`. Decorators: `gluon.jit`. Parameters: `x_ptr`, `out_ptr`, `n_elements`, `MODE`, `BLOCK`, `THREADS_PER_WARP`. Key calls include `gl.program_id`, `gl.BlockedLayout`, `gl.load`, `gl.store`, `gl.arange`, `gl.exp`, and 1 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `_exp_inverse_identity_kernel`。 装饰器：`gluon.jit`。 参数：`x_ptr`、`out_ptr`、`n_elements`、`MODE`、`BLOCK`、`THREADS_PER_WARP`。 关键调用包括 `gl.program_id`、`gl.BlockedLayout`、`gl.load`、`gl.store`、`gl.arange`、`gl.exp` 等另外 1 项。 该作用域涉及布局变换推理。

#### Lines 665-670

```python
    pid = gl.program_id(0)
    layout: gl.constexpr = gl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4],
                                            order=[0])
    offs = pid * BLOCK + gl.arange(0, BLOCK, layout=layout)
    mask = offs < n_elements
    x = gl.load(x_ptr + offs, mask=mask, other=0.0)
```
- **EN:** Prepares or updates state through `pid`, `layout`, `offs`, `mask`, `x`. Invokes `gl.program_id`, `gl.BlockedLayout`, `gl.arange`, `gl.load` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `pid`、`layout`、`offs`、`mask`、`x` 准备或更新状态。 调用 `gl.program_id`、`gl.BlockedLayout`、`gl.arange`、`gl.load` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 671-676

```python
    if MODE == "exp_neg":
        z = gl.exp(-x)
    elif MODE == "exp_recip":
        z = 1.0 / gl.exp(x)
    else:
        gl.static_assert(False, "unsupported MODE")
```
- **EN:** Invokes `gl.exp`, `gl.static_assert` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `gl.exp`、`gl.static_assert` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 677-677

```python
    gl.store(out_ptr + offs, z, mask=mask)
```
- **EN:** Invokes `gl.store` to execute the test logic.
- **CN:** 调用 `gl.store` 执行测试逻辑。

### Lines 678-682

```python


@gluon.jit
def _cossin_identity_kernel(x_ptr, y_ptr, lhs_ptr, rhs_ptr, n_elements, MODE: gl.constexpr, BLOCK: gl.constexpr,
                            THREADS_PER_WARP: gl.constexpr):
```
- **EN:** Defines the helper function `_cossin_identity_kernel`. Decorators: `gluon.jit`. Parameters: `x_ptr`, `y_ptr`, `lhs_ptr`, `rhs_ptr`, `n_elements`, `MODE`, `BLOCK`, `THREADS_PER_WARP`. Key calls include `gl.program_id`, `gl.BlockedLayout`, `gl.load`, `gl.sin`, `gl.cos`, `gl.store`, and 2 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `_cossin_identity_kernel`。 装饰器：`gluon.jit`。 参数：`x_ptr`、`y_ptr`、`lhs_ptr`、`rhs_ptr`、`n_elements`、`MODE`、`BLOCK`、`THREADS_PER_WARP`。 关键调用包括 `gl.program_id`、`gl.BlockedLayout`、`gl.load`、`gl.sin`、`gl.cos`、`gl.store` 等另外 2 项。 该作用域涉及布局变换推理。

#### Lines 683-693

```python
    pid = gl.program_id(0)
    layout: gl.constexpr = gl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4],
                                            order=[0])
    offs = pid * BLOCK + gl.arange(0, BLOCK, layout=layout)
    mask = offs < n_elements
    x = gl.load(x_ptr + offs, mask=mask, other=0.0)
    y = gl.load(y_ptr + offs, mask=mask, other=0.0)
    sx = gl.sin(x)
    sy = gl.sin(y)
    cx = gl.cos(x)
    cy = gl.cos(y)
```
- **EN:** Prepares or updates state through `pid`, `layout`, `offs`, `mask`, `x`, `y`, `sx`, `sy`, and 2 more. Invokes `gl.program_id`, `gl.BlockedLayout`, `gl.arange`, `gl.load`, `gl.sin`, `gl.cos` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `pid`、`layout`、`offs`、`mask`、`x`、`y`、`sx`、`sy` 等另外 2 项 准备或更新状态。 调用 `gl.program_id`、`gl.BlockedLayout`、`gl.arange`、`gl.load`、`gl.sin`、`gl.cos` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 694-711

```python

    if MODE == "sin_add":
        lhs = gl.sin(x + y)
        rhs = sx * cy + cx * sy
    elif MODE == "sin_sub":
        lhs = gl.sin(x - y)
        rhs = sx * cy - cx * sy
    elif MODE == "cos_add":
        lhs = gl.cos(x + y)
        rhs = cx * cy - sx * sy
    elif MODE == "cos_sub":
        lhs = gl.cos(x - y)
        rhs = cx * cy + sx * sy
    elif MODE == "unit":
        lhs = cx * cx + sx * sx
        rhs = x * 0.0 + 1.0
    else:
        gl.static_assert(False, "unsupported MODE")
```
- **EN:** Invokes `gl.sin`, `gl.cos`, `gl.static_assert` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `gl.sin`、`gl.cos`、`gl.static_assert` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 712-714

```python

    gl.store(lhs_ptr + offs, lhs, mask=mask)
    gl.store(rhs_ptr + offs, rhs, mask=mask)
```
- **EN:** Invokes `gl.store` to execute the test logic.
- **CN:** 调用 `gl.store` 执行测试逻辑。

### Lines 715-735

```python


@pytest.mark.parametrize(
    "op",
    [
        "exp",
        "exp2",
        "neg",
        "log",
        "log2",
        "cos",
        "sin",
        "sqrt",
        "sqrt_rn",
        "rsqrt",
        "erf",
        "floor",
        "ceil",
    ],
)
def test_unary_math_identity(device, op, fresh_knobs):
```
- **EN:** Defines the test function `test_unary_math_identity`. Decorators: `pytest.mark.parametrize('op', ['exp', 'exp2', 'neg', 'log', 'log2', 'cos', 'sin', 'sqrt', 'sqrt_rn', 'rsqrt', 'erf', 'floor', 'ceil'])`. Parameters: `device`, `op`, `fresh_knobs`. Key calls include `pytest.mark.parametrize`, `_require_cuda_backend`, `np.random.RandomState`, `xf.view`, `torch.tensor`, `torch.empty`, and 10 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义测试函数 `test_unary_math_identity`。 装饰器：`pytest.mark.parametrize('op', ['exp', 'exp2', 'neg', 'log', 'log2', 'cos', 'sin', 'sqrt', 'sqrt_rn', 'rsqrt', 'erf', 'floor', 'ceil'])`。 参数：`device`、`op`、`fresh_knobs`。 关键调用包括 `pytest.mark.parametrize`、`_require_cuda_backend`、`np.random.RandomState`、`xf.view`、`torch.tensor`、`torch.empty` 等另外 10 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、随机数据生成。

#### Lines 736-758

```python
    _require_cuda_backend(device)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"

    n_elements = 1024
    BLOCK = 256
    rs = np.random.RandomState(0)
    # Includes negative values for log/sqrt on purpose; fpsan works on payload bits.
    xf = rs.randn(n_elements).astype(np.float32)
    x_bits = xf.view(np.int32)

    x = torch.tensor(x_bits, dtype=torch.int32, device="cuda")
    out = torch.empty((n_elements, ), dtype=torch.int32, device="cuda")

    grid = (triton.cdiv(n_elements, BLOCK), )
    _unary_math_kernel[grid](
        triton.TensorWrapper(x, dtype=torch.float32),
        triton.TensorWrapper(out, dtype=torch.float32),
        n_elements,
        OP=op,
        BLOCK=BLOCK,
        THREADS_PER_WARP=THREADS_PER_WARP,
    )
```
- **EN:** Prepares or updates state through `fresh_knobs`, `n_elements`, `BLOCK`, `rs`, `xf`, `x_bits`, `x`, `out`, and 1 more. Invokes `_require_cuda_backend`, `np.random.RandomState`, `rs.randn`, `xf.view`, `torch.tensor`, `torch.empty`, and 2 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `fresh_knobs`、`n_elements`、`BLOCK`、`rs`、`xf`、`x_bits`、`x`、`out` 等另外 1 项 准备或更新状态。 调用 `_require_cuda_backend`、`np.random.RandomState`、`rs.randn`、`xf.view`、`torch.tensor`、`torch.empty` 等另外 2 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 759-771

```python

    if op == "exp":
        exp_bits = _expected_exp_i32(x_bits)
    elif op == "exp2":
        exp_bits = _expected_exp2_i32(x_bits)
    elif op == "neg":
        exp_bits = _expected_neg_i32(x_bits)
    elif op == "cos":
        exp_bits = _expected_cos_i32(x_bits)
    elif op == "sin":
        exp_bits = _expected_sin_i32(x_bits)
    else:
        exp_bits = _expected_unary_tag_i32(x_bits, op)
```
- **EN:** Invokes `_expected_exp_i32`, `_expected_exp2_i32`, `_expected_neg_i32`, `_expected_cos_i32`, `_expected_sin_i32`, `_expected_unary_tag_i32` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `_expected_exp_i32`、`_expected_exp2_i32`、`_expected_neg_i32`、`_expected_cos_i32`、`_expected_sin_i32`、`_expected_unary_tag_i32` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 772-772

```python
    _assert_payload_equal(out, exp_bits)
```
- **EN:** Invokes `_assert_payload_equal` to execute the test logic.
- **CN:** 调用 `_assert_payload_equal` 执行测试逻辑。

### Lines 773-775

```python


def test_exp_add_mul_identity(device, fresh_knobs):
```
- **EN:** Defines the test function `test_exp_add_mul_identity`. Parameters: `device`, `fresh_knobs`. Key calls include `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, `triton.TensorWrapper`, and 2 more. This scope touches PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义测试函数 `test_exp_add_mul_identity`。 参数：`device`、`fresh_knobs`。 关键调用包括 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty`、`triton.TensorWrapper` 等另外 2 项。 该作用域涉及PyTorch 张量准备与校验、随机数据生成。

#### Lines 776-801

```python
    _require_cuda_backend(device)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"

    n_elements = 1024
    BLOCK = 256

    g = torch.Generator(device="cuda")
    g.manual_seed(0)
    x = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    y = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    out_add = torch.empty((n_elements, ), dtype=torch.int32, device="cuda")
    out_mul = torch.empty((n_elements, ), dtype=torch.int32, device="cuda")

    xw = triton.TensorWrapper(x, dtype=torch.float32)
    yw = triton.TensorWrapper(y, dtype=torch.float32)
    out_add_w = triton.TensorWrapper(out_add, dtype=torch.float32)
    out_mul_w = triton.TensorWrapper(out_mul, dtype=torch.float32)

    grid = (triton.cdiv(n_elements, BLOCK), )
    _exp_binary_identity_kernel[grid](xw, yw, out_add_w, n_elements, MODE="exp_add", BLOCK=BLOCK,
                                      THREADS_PER_WARP=THREADS_PER_WARP)
    _exp_binary_identity_kernel[grid](xw, yw, out_mul_w, n_elements, MODE="exp_mul", BLOCK=BLOCK,
                                      THREADS_PER_WARP=THREADS_PER_WARP)

    _assert_payload_equal(out_add, out_mul)
```
- **EN:** Prepares or updates state through `fresh_knobs`, `n_elements`, `BLOCK`, `g`, `x`, `y`, `out_add`, `out_mul`, and 5 more. Invokes `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, `triton.TensorWrapper`, and 2 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `fresh_knobs`、`n_elements`、`BLOCK`、`g`、`x`、`y`、`out_add`、`out_mul` 等另外 5 项 准备或更新状态。 调用 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty`、`triton.TensorWrapper` 等另外 2 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 802-804

```python


def test_exp_exp2_scaled_identity(device, fresh_knobs):
```
- **EN:** Defines the test function `test_exp_exp2_scaled_identity`. Parameters: `device`, `fresh_knobs`. Key calls include `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, `triton.TensorWrapper`, and 2 more. This scope touches PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义测试函数 `test_exp_exp2_scaled_identity`。 参数：`device`、`fresh_knobs`。 关键调用包括 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty`、`triton.TensorWrapper` 等另外 2 项。 该作用域涉及PyTorch 张量准备与校验、随机数据生成。

#### Lines 805-828

```python
    _require_cuda_backend(device)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"

    n_elements = 1024
    BLOCK = 256

    g = torch.Generator(device="cuda")
    g.manual_seed(1)
    x = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    out_exp = torch.empty((n_elements, ), dtype=torch.int32, device="cuda")
    out_exp2 = torch.empty((n_elements, ), dtype=torch.int32, device="cuda")

    xw = triton.TensorWrapper(x, dtype=torch.float32)
    out_exp_w = triton.TensorWrapper(out_exp, dtype=torch.float32)
    out_exp2_w = triton.TensorWrapper(out_exp2, dtype=torch.float32)

    grid = (triton.cdiv(n_elements, BLOCK), )
    _exp_scaled_identity_kernel[grid](xw, out_exp_w, n_elements, MODE="exp", BLOCK=BLOCK,
                                      THREADS_PER_WARP=THREADS_PER_WARP)
    _exp_scaled_identity_kernel[grid](xw, out_exp2_w, n_elements, MODE="exp2_scaled", BLOCK=BLOCK,
                                      THREADS_PER_WARP=THREADS_PER_WARP)

    _assert_payload_equal(out_exp, out_exp2)
```
- **EN:** Prepares or updates state through `fresh_knobs`, `n_elements`, `BLOCK`, `g`, `x`, `out_exp`, `out_exp2`, `xw`, and 3 more. Invokes `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, `triton.TensorWrapper`, and 2 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `fresh_knobs`、`n_elements`、`BLOCK`、`g`、`x`、`out_exp`、`out_exp2`、`xw` 等另外 3 项 准备或更新状态。 调用 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty`、`triton.TensorWrapper` 等另外 2 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 829-831

```python


def test_exp_neg_reciprocal_identity(device, fresh_knobs):
```
- **EN:** Defines the test function `test_exp_neg_reciprocal_identity`. Parameters: `device`, `fresh_knobs`. Key calls include `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, `triton.TensorWrapper`, and 2 more. This scope touches PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义测试函数 `test_exp_neg_reciprocal_identity`。 参数：`device`、`fresh_knobs`。 关键调用包括 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty`、`triton.TensorWrapper` 等另外 2 项。 该作用域涉及PyTorch 张量准备与校验、随机数据生成。

#### Lines 832-855

```python
    _require_cuda_backend(device)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"

    n_elements = 1024
    BLOCK = 256

    g = torch.Generator(device="cuda")
    g.manual_seed(4)
    x = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    out_neg = torch.empty((n_elements, ), dtype=torch.int32, device="cuda")
    out_recip = torch.empty((n_elements, ), dtype=torch.int32, device="cuda")

    xw = triton.TensorWrapper(x, dtype=torch.float32)
    out_neg_w = triton.TensorWrapper(out_neg, dtype=torch.float32)
    out_recip_w = triton.TensorWrapper(out_recip, dtype=torch.float32)

    grid = (triton.cdiv(n_elements, BLOCK), )
    _exp_inverse_identity_kernel[grid](xw, out_neg_w, n_elements, MODE="exp_neg", BLOCK=BLOCK,
                                       THREADS_PER_WARP=THREADS_PER_WARP)
    _exp_inverse_identity_kernel[grid](xw, out_recip_w, n_elements, MODE="exp_recip", BLOCK=BLOCK,
                                       THREADS_PER_WARP=THREADS_PER_WARP)

    _assert_payload_equal(out_neg, out_recip)
```
- **EN:** Prepares or updates state through `fresh_knobs`, `n_elements`, `BLOCK`, `g`, `x`, `out_neg`, `out_recip`, `xw`, and 3 more. Invokes `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, `triton.TensorWrapper`, and 2 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `fresh_knobs`、`n_elements`、`BLOCK`、`g`、`x`、`out_neg`、`out_recip`、`xw` 等另外 3 项 准备或更新状态。 调用 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty`、`triton.TensorWrapper` 等另外 2 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 856-859

```python


@pytest.mark.parametrize("mode", ["sin_add", "sin_sub", "cos_add", "cos_sub", "unit"])
def test_cossin_angle_identities(device, mode, fresh_knobs):
```
- **EN:** Defines the test function `test_cossin_angle_identities`. Decorators: `pytest.mark.parametrize('mode', ['sin_add', 'sin_sub', 'cos_add', 'cos_sub', 'unit'])`. Parameters: `device`, `mode`, `fresh_knobs`. Key calls include `pytest.mark.parametrize`, `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, and 3 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义测试函数 `test_cossin_angle_identities`。 装饰器：`pytest.mark.parametrize('mode', ['sin_add', 'sin_sub', 'cos_add', 'cos_sub', 'unit'])`。 参数：`device`、`mode`、`fresh_knobs`。 关键调用包括 `pytest.mark.parametrize`、`_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty` 等另外 3 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、随机数据生成。

#### Lines 860-883

```python
    _require_cuda_backend(device)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"

    n_elements = 1024
    BLOCK = 256

    g = torch.Generator(device="cuda")
    g.manual_seed(5)
    x = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    y = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    lhs = torch.empty((n_elements, ), dtype=torch.int32, device="cuda")
    rhs = torch.empty((n_elements, ), dtype=torch.int32, device="cuda")

    xw = triton.TensorWrapper(x, dtype=torch.float32)
    yw = triton.TensorWrapper(y, dtype=torch.float32)
    lhsw = triton.TensorWrapper(lhs, dtype=torch.float32)
    rhsw = triton.TensorWrapper(rhs, dtype=torch.float32)

    grid = (triton.cdiv(n_elements, BLOCK), )
    _cossin_identity_kernel[grid](xw, yw, lhsw, rhsw, n_elements, MODE=mode, BLOCK=BLOCK,
                                  THREADS_PER_WARP=THREADS_PER_WARP)

    _assert_payload_equal(lhs, rhs)
```
- **EN:** Prepares or updates state through `fresh_knobs`, `n_elements`, `BLOCK`, `g`, `x`, `y`, `lhs`, `rhs`, and 5 more. Invokes `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, `triton.TensorWrapper`, and 2 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `fresh_knobs`、`n_elements`、`BLOCK`、`g`、`x`、`y`、`lhs`、`rhs` 等另外 5 项 准备或更新状态。 调用 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty`、`triton.TensorWrapper` 等另外 2 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 884-888

```python


@gluon.jit
def _extern_unary_math_kernel(x_ptr, out_ptr, n_elements, OP: gl.constexpr, BLOCK: gl.constexpr,
                              THREADS_PER_WARP: gl.constexpr):
```
- **EN:** Defines the helper function `_extern_unary_math_kernel`. Decorators: `gluon.jit`. Parameters: `x_ptr`, `out_ptr`, `n_elements`, `OP`, `BLOCK`, `THREADS_PER_WARP`. Key calls include `gl.program_id`, `gl.BlockedLayout`, `gl.load`, `gl.store`, `gl.arange`, `gl.extra.libdevice.tan`, and 5 more. This scope touches layout transformation reasoning, libdevice coverage.
- **CN:** 定义辅助函数 `_extern_unary_math_kernel`。 装饰器：`gluon.jit`。 参数：`x_ptr`、`out_ptr`、`n_elements`、`OP`、`BLOCK`、`THREADS_PER_WARP`。 关键调用包括 `gl.program_id`、`gl.BlockedLayout`、`gl.load`、`gl.store`、`gl.arange`、`gl.extra.libdevice.tan` 等另外 5 项。 该作用域涉及布局变换推理、libdevice 覆盖。

#### Lines 889-894

```python
    pid = gl.program_id(0)
    layout: gl.constexpr = gl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4],
                                            order=[0])
    offs = pid * BLOCK + gl.arange(0, BLOCK, layout=layout)
    mask = offs < n_elements
    x = gl.load(x_ptr + offs, mask=mask, other=0.0)
```
- **EN:** Prepares or updates state through `pid`, `layout`, `offs`, `mask`, `x`. Invokes `gl.program_id`, `gl.BlockedLayout`, `gl.arange`, `gl.load` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `pid`、`layout`、`offs`、`mask`、`x` 准备或更新状态。 调用 `gl.program_id`、`gl.BlockedLayout`、`gl.arange`、`gl.load` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 895-906

```python
    if OP == "tan":
        z = gl.extra.libdevice.tan(x)
    elif OP == "tanh":
        z = gl.extra.libdevice.tanh(x)
    elif OP == "log1p":
        z = gl.extra.libdevice.log1p(x)
    elif OP == "cbrt":
        z = gl.extra.libdevice.cbrt(x)
    elif OP == "round":
        z = gl.extra.libdevice.round(x)
    else:
        gl.static_assert(False, "unsupported OP")
```
- **EN:** Invokes `gl.extra.libdevice.tan`, `gl.extra.libdevice.tanh`, `gl.extra.libdevice.log1p`, `gl.extra.libdevice.cbrt`, `gl.extra.libdevice.round`, `gl.static_assert` to execute the test logic. Branches on runtime or test conditions. Relevant themes: libdevice coverage.
- **CN:** 调用 `gl.extra.libdevice.tan`、`gl.extra.libdevice.tanh`、`gl.extra.libdevice.log1p`、`gl.extra.libdevice.cbrt`、`gl.extra.libdevice.round`、`gl.static_assert` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：libdevice 覆盖。

#### Lines 907-907

```python
    gl.store(out_ptr + offs, z, mask=mask)
```
- **EN:** Invokes `gl.store` to execute the test logic.
- **CN:** 调用 `gl.store` 执行测试逻辑。

### Lines 908-914

```python


@pytest.mark.parametrize(
    "op,symbol",
    EXTERN_UNARY_CASES,
)
def test_extern_unary_payload_semantics(device, op, symbol, fresh_knobs):
```
- **EN:** Defines the test function `test_extern_unary_payload_semantics`. Decorators: `pytest.mark.parametrize('op,symbol', EXTERN_UNARY_CASES)`. Parameters: `device`, `op`, `symbol`, `fresh_knobs`. Key calls include `pytest.mark.parametrize`, `_require_cuda_backend`, `np.random.RandomState`, `xf.view`, `torch.tensor`, `torch.empty`, and 5 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义测试函数 `test_extern_unary_payload_semantics`。 装饰器：`pytest.mark.parametrize('op,symbol', EXTERN_UNARY_CASES)`。 参数：`device`、`op`、`symbol`、`fresh_knobs`。 关键调用包括 `pytest.mark.parametrize`、`_require_cuda_backend`、`np.random.RandomState`、`xf.view`、`torch.tensor`、`torch.empty` 等另外 5 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、随机数据生成。

#### Lines 915-939

```python
    _require_cuda_backend(device)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"

    n_elements = 1024
    BLOCK = 256
    rs = np.random.RandomState(11)
    xf = rs.randn(n_elements).astype(np.float32)
    x_bits = xf.view(np.int32)

    x = torch.tensor(x_bits, dtype=torch.int32, device="cuda")
    out = torch.empty((n_elements, ), dtype=torch.int32, device="cuda")

    grid = (triton.cdiv(n_elements, BLOCK), )
    _extern_unary_math_kernel[grid](
        triton.TensorWrapper(x, dtype=torch.float32),
        triton.TensorWrapper(out, dtype=torch.float32),
        n_elements,
        OP=op,
        BLOCK=BLOCK,
        THREADS_PER_WARP=THREADS_PER_WARP,
    )

    exp_bits = _expected_extern_unary_tag_i32(x_bits, symbol)
    _assert_payload_equal(out, exp_bits)
```
- **EN:** Prepares or updates state through `fresh_knobs`, `n_elements`, `BLOCK`, `rs`, `xf`, `x_bits`, `x`, `out`, and 2 more. Invokes `_require_cuda_backend`, `np.random.RandomState`, `rs.randn`, `xf.view`, `torch.tensor`, `torch.empty`, and 4 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `fresh_knobs`、`n_elements`、`BLOCK`、`rs`、`xf`、`x_bits`、`x`、`out` 等另外 2 项 准备或更新状态。 调用 `_require_cuda_backend`、`np.random.RandomState`、`rs.randn`、`xf.view`、`torch.tensor`、`torch.empty` 等另外 4 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 940-944

```python


@gluon.jit
def _extern_binary_math_kernel(x_ptr, y_ptr, out_ptr, n_elements, OP: gl.constexpr, BLOCK: gl.constexpr,
                               THREADS_PER_WARP: gl.constexpr):
```
- **EN:** Defines the helper function `_extern_binary_math_kernel`. Decorators: `gluon.jit`. Parameters: `x_ptr`, `y_ptr`, `out_ptr`, `n_elements`, `OP`, `BLOCK`, `THREADS_PER_WARP`. Key calls include `gl.program_id`, `gl.BlockedLayout`, `gl.load`, `gl.store`, `gl.arange`, `gl.extra.libdevice.atan2`, and 3 more. This scope touches layout transformation reasoning, libdevice coverage.
- **CN:** 定义辅助函数 `_extern_binary_math_kernel`。 装饰器：`gluon.jit`。 参数：`x_ptr`、`y_ptr`、`out_ptr`、`n_elements`、`OP`、`BLOCK`、`THREADS_PER_WARP`。 关键调用包括 `gl.program_id`、`gl.BlockedLayout`、`gl.load`、`gl.store`、`gl.arange`、`gl.extra.libdevice.atan2` 等另外 3 项。 该作用域涉及布局变换推理、libdevice 覆盖。

#### Lines 945-951

```python
    pid = gl.program_id(0)
    layout: gl.constexpr = gl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4],
                                            order=[0])
    offs = pid * BLOCK + gl.arange(0, BLOCK, layout=layout)
    mask = offs < n_elements
    x = gl.load(x_ptr + offs, mask=mask, other=0.0)
    y = gl.load(y_ptr + offs, mask=mask, other=0.0)
```
- **EN:** Prepares or updates state through `pid`, `layout`, `offs`, `mask`, `x`, `y`. Invokes `gl.program_id`, `gl.BlockedLayout`, `gl.arange`, `gl.load` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `pid`、`layout`、`offs`、`mask`、`x`、`y` 准备或更新状态。 调用 `gl.program_id`、`gl.BlockedLayout`、`gl.arange`、`gl.load` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 952-959

```python
    if OP == "atan2":
        z = gl.extra.libdevice.atan2(x, y)
    elif OP == "hypot":
        z = gl.extra.libdevice.hypot(x, y)
    elif OP == "pow":
        z = gl.extra.libdevice.pow(x, y)
    else:
        gl.static_assert(False, "unsupported OP")
```
- **EN:** Invokes `gl.extra.libdevice.atan2`, `gl.extra.libdevice.hypot`, `gl.extra.libdevice.pow`, `gl.static_assert` to execute the test logic. Branches on runtime or test conditions. Relevant themes: libdevice coverage.
- **CN:** 调用 `gl.extra.libdevice.atan2`、`gl.extra.libdevice.hypot`、`gl.extra.libdevice.pow`、`gl.static_assert` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：libdevice 覆盖。

#### Lines 960-960

```python
    gl.store(out_ptr + offs, z, mask=mask)
```
- **EN:** Invokes `gl.store` to execute the test logic.
- **CN:** 调用 `gl.store` 执行测试逻辑。

### Lines 961-965

```python


@gluon.jit
def _extern_ternary_math_kernel(x_ptr, y_ptr, z_ptr, out_ptr, n_elements, OP: gl.constexpr, BLOCK: gl.constexpr,
                                THREADS_PER_WARP: gl.constexpr):
```
- **EN:** Defines the helper function `_extern_ternary_math_kernel`. Decorators: `gluon.jit`. Parameters: `x_ptr`, `y_ptr`, `z_ptr`, `out_ptr`, `n_elements`, `OP`, `BLOCK`, `THREADS_PER_WARP`. Key calls include `gl.program_id`, `gl.BlockedLayout`, `gl.load`, `gl.store`, `gl.arange`, `gl.extra.libdevice.fma`, and 1 more. This scope touches layout transformation reasoning, libdevice coverage.
- **CN:** 定义辅助函数 `_extern_ternary_math_kernel`。 装饰器：`gluon.jit`。 参数：`x_ptr`、`y_ptr`、`z_ptr`、`out_ptr`、`n_elements`、`OP`、`BLOCK`、`THREADS_PER_WARP`。 关键调用包括 `gl.program_id`、`gl.BlockedLayout`、`gl.load`、`gl.store`、`gl.arange`、`gl.extra.libdevice.fma` 等另外 1 项。 该作用域涉及布局变换推理、libdevice 覆盖。

#### Lines 966-973

```python
    pid = gl.program_id(0)
    layout: gl.constexpr = gl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4],
                                            order=[0])
    offs = pid * BLOCK + gl.arange(0, BLOCK, layout=layout)
    mask = offs < n_elements
    x = gl.load(x_ptr + offs, mask=mask, other=0.0)
    y = gl.load(y_ptr + offs, mask=mask, other=0.0)
    z = gl.load(z_ptr + offs, mask=mask, other=0.0)
```
- **EN:** Prepares or updates state through `pid`, `layout`, `offs`, `mask`, `x`, `y`, `z`. Invokes `gl.program_id`, `gl.BlockedLayout`, `gl.arange`, `gl.load` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `pid`、`layout`、`offs`、`mask`、`x`、`y`、`z` 准备或更新状态。 调用 `gl.program_id`、`gl.BlockedLayout`、`gl.arange`、`gl.load` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 974-977

```python
    if OP == "fma":
        out = gl.extra.libdevice.fma(x, y, z)
    else:
        gl.static_assert(False, "unsupported OP")
```
- **EN:** Invokes `gl.extra.libdevice.fma`, `gl.static_assert` to execute the test logic. Branches on runtime or test conditions. Relevant themes: libdevice coverage.
- **CN:** 调用 `gl.extra.libdevice.fma`、`gl.static_assert` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：libdevice 覆盖。

#### Lines 978-978

```python
    gl.store(out_ptr + offs, out, mask=mask)
```
- **EN:** Invokes `gl.store` to execute the test logic.
- **CN:** 调用 `gl.store` 执行测试逻辑。

### Lines 979-983

```python


@gluon.jit
def _extern_mixed_math_kernel(x_ptr, y_ptr, out_ptr, n_elements, OP: gl.constexpr, BLOCK: gl.constexpr,
                              THREADS_PER_WARP: gl.constexpr):
```
- **EN:** Defines the helper function `_extern_mixed_math_kernel`. Decorators: `gluon.jit`. Parameters: `x_ptr`, `y_ptr`, `out_ptr`, `n_elements`, `OP`, `BLOCK`, `THREADS_PER_WARP`. Key calls include `gl.program_id`, `gl.BlockedLayout`, `gl.load`, `gl.store`, `gl.arange`, `gl.extra.libdevice.ldexp`, and 1 more. This scope touches layout transformation reasoning, libdevice coverage.
- **CN:** 定义辅助函数 `_extern_mixed_math_kernel`。 装饰器：`gluon.jit`。 参数：`x_ptr`、`y_ptr`、`out_ptr`、`n_elements`、`OP`、`BLOCK`、`THREADS_PER_WARP`。 关键调用包括 `gl.program_id`、`gl.BlockedLayout`、`gl.load`、`gl.store`、`gl.arange`、`gl.extra.libdevice.ldexp` 等另外 1 项。 该作用域涉及布局变换推理、libdevice 覆盖。

#### Lines 984-990

```python
    pid = gl.program_id(0)
    layout: gl.constexpr = gl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4],
                                            order=[0])
    offs = pid * BLOCK + gl.arange(0, BLOCK, layout=layout)
    mask = offs < n_elements
    x = gl.load(x_ptr + offs, mask=mask, other=0.0)
    y = gl.load(y_ptr + offs, mask=mask, other=0)
```
- **EN:** Prepares or updates state through `pid`, `layout`, `offs`, `mask`, `x`, `y`. Invokes `gl.program_id`, `gl.BlockedLayout`, `gl.arange`, `gl.load` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `pid`、`layout`、`offs`、`mask`、`x`、`y` 准备或更新状态。 调用 `gl.program_id`、`gl.BlockedLayout`、`gl.arange`、`gl.load` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 991-994

```python
    if OP == "ldexp":
        out = gl.extra.libdevice.ldexp(x, y)
    else:
        gl.static_assert(False, "unsupported OP")
```
- **EN:** Invokes `gl.extra.libdevice.ldexp`, `gl.static_assert` to execute the test logic. Branches on runtime or test conditions. Relevant themes: libdevice coverage.
- **CN:** 调用 `gl.extra.libdevice.ldexp`、`gl.static_assert` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：libdevice 覆盖。

#### Lines 995-995

```python
    gl.store(out_ptr + offs, out, mask=mask)
```
- **EN:** Invokes `gl.store` to execute the test logic.
- **CN:** 调用 `gl.store` 执行测试逻辑。

### Lines 996-1002

```python


@pytest.mark.parametrize(
    "op,symbol",
    EXTERN_BINARY_CASES,
)
def test_extern_binary_payload_semantics(device, op, symbol, fresh_knobs):
```
- **EN:** Defines the test function `test_extern_binary_payload_semantics`. Decorators: `pytest.mark.parametrize('op,symbol', EXTERN_BINARY_CASES)`. Parameters: `device`, `op`, `symbol`, `fresh_knobs`. Key calls include `pytest.mark.parametrize`, `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, and 6 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义测试函数 `test_extern_binary_payload_semantics`。 装饰器：`pytest.mark.parametrize('op,symbol', EXTERN_BINARY_CASES)`。 参数：`device`、`op`、`symbol`、`fresh_knobs`。 关键调用包括 `pytest.mark.parametrize`、`_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty` 等另外 6 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、随机数据生成。

#### Lines 1003-1027

```python
    _require_cuda_backend(device)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"

    n_elements = 1024
    BLOCK = 256
    g = torch.Generator(device="cuda")
    g.manual_seed(23)
    x = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    y = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    out = torch.empty((n_elements, ), dtype=torch.int32, device="cuda")

    xw = triton.TensorWrapper(x, dtype=torch.float32)
    yw = triton.TensorWrapper(y, dtype=torch.float32)
    outw = triton.TensorWrapper(out, dtype=torch.float32)

    grid = (triton.cdiv(n_elements, BLOCK), )
    _extern_binary_math_kernel[grid](xw, yw, outw, n_elements, OP=op, BLOCK=BLOCK, THREADS_PER_WARP=THREADS_PER_WARP)

    exp_bits = _expected_extern_binary_tag_i32(
        x.cpu().numpy().astype(np.int32, copy=False),
        y.cpu().numpy().astype(np.int32, copy=False),
        symbol,
    )
    _assert_payload_equal(out, exp_bits)
```
- **EN:** Prepares or updates state through `fresh_knobs`, `n_elements`, `BLOCK`, `g`, `x`, `y`, `out`, `xw`, and 4 more. Invokes `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, `triton.TensorWrapper`, and 5 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `fresh_knobs`、`n_elements`、`BLOCK`、`g`、`x`、`y`、`out`、`xw` 等另外 4 项 准备或更新状态。 调用 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty`、`triton.TensorWrapper` 等另外 5 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 1028-1034

```python


@pytest.mark.parametrize(
    "op,symbol",
    EXTERN_TERNARY_CASES,
)
def test_extern_ternary_payload_semantics(device, op, symbol, fresh_knobs):
```
- **EN:** Defines the test function `test_extern_ternary_payload_semantics`. Decorators: `pytest.mark.parametrize('op,symbol', EXTERN_TERNARY_CASES)`. Parameters: `device`, `op`, `symbol`, `fresh_knobs`. Key calls include `pytest.mark.parametrize`, `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, and 7 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义测试函数 `test_extern_ternary_payload_semantics`。 装饰器：`pytest.mark.parametrize('op,symbol', EXTERN_TERNARY_CASES)`。 参数：`device`、`op`、`symbol`、`fresh_knobs`。 关键调用包括 `pytest.mark.parametrize`、`_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty` 等另外 7 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、随机数据生成。

#### Lines 1035-1065

```python
    _require_cuda_backend(device)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"

    n_elements = 1024
    BLOCK = 256
    g = torch.Generator(device="cuda")
    g.manual_seed(29)
    x = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    y = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    z = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    out = torch.empty((n_elements, ), dtype=torch.int32, device="cuda")

    xw = triton.TensorWrapper(x, dtype=torch.float32)
    yw = triton.TensorWrapper(y, dtype=torch.float32)
    zw = triton.TensorWrapper(z, dtype=torch.float32)
    outw = triton.TensorWrapper(out, dtype=torch.float32)

    grid = (triton.cdiv(n_elements, BLOCK), )
    _extern_ternary_math_kernel[grid](xw, yw, zw, outw, n_elements, OP=op, BLOCK=BLOCK,
                                      THREADS_PER_WARP=THREADS_PER_WARP)

    exp_bits = _expected_extern_variadic_tag_i32(
        [
            x.cpu().numpy().astype(np.int32, copy=False),
            y.cpu().numpy().astype(np.int32, copy=False),
            z.cpu().numpy().astype(np.int32, copy=False),
        ],
        symbol,
    )
    _assert_payload_equal(out, exp_bits)
```
- **EN:** Prepares or updates state through `fresh_knobs`, `n_elements`, `BLOCK`, `g`, `x`, `y`, `z`, `out`, and 6 more. Invokes `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, `triton.TensorWrapper`, and 6 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `fresh_knobs`、`n_elements`、`BLOCK`、`g`、`x`、`y`、`z`、`out` 等另外 6 项 准备或更新状态。 调用 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty`、`triton.TensorWrapper` 等另外 6 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 1066-1072

```python


@pytest.mark.parametrize(
    "op,symbol",
    EXTERN_MIXED_CASES,
)
def test_extern_mixed_payload_semantics(device, op, symbol, fresh_knobs):
```
- **EN:** Defines the test function `test_extern_mixed_payload_semantics`. Decorators: `pytest.mark.parametrize('op,symbol', EXTERN_MIXED_CASES)`. Parameters: `device`, `op`, `symbol`, `fresh_knobs`. Key calls include `pytest.mark.parametrize`, `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, and 6 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义测试函数 `test_extern_mixed_payload_semantics`。 装饰器：`pytest.mark.parametrize('op,symbol', EXTERN_MIXED_CASES)`。 参数：`device`、`op`、`symbol`、`fresh_knobs`。 关键调用包括 `pytest.mark.parametrize`、`_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty` 等另外 6 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、随机数据生成。

#### Lines 1073-1099

```python
    _require_cuda_backend(device)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"

    n_elements = 1024
    BLOCK = 256
    g = torch.Generator(device="cuda")
    g.manual_seed(31)
    x = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    y = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    out = torch.empty((n_elements, ), dtype=torch.int32, device="cuda")

    xw = triton.TensorWrapper(x, dtype=torch.float32)
    outw = triton.TensorWrapper(out, dtype=torch.float32)

    grid = (triton.cdiv(n_elements, BLOCK), )
    _extern_mixed_math_kernel[grid](xw, y, outw, n_elements, OP=op, BLOCK=BLOCK, THREADS_PER_WARP=THREADS_PER_WARP)

    exp_bits = _expected_extern_variadic_tag_i32(
        [
            x.cpu().numpy().astype(np.int32, copy=False),
            y.cpu().numpy().astype(np.int32, copy=False),
        ],
        symbol,
        float_args=[True, False],
    )
    _assert_payload_equal(out, exp_bits)
```
- **EN:** Prepares or updates state through `fresh_knobs`, `n_elements`, `BLOCK`, `g`, `x`, `y`, `out`, `xw`, and 3 more. Invokes `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, `triton.TensorWrapper`, and 5 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `fresh_knobs`、`n_elements`、`BLOCK`、`g`、`x`、`y`、`out`、`xw` 等另外 3 项 准备或更新状态。 调用 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty`、`triton.TensorWrapper` 等另外 5 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 1100-1102

```python


def _expected_fma_i32(x_i32: np.ndarray, y_i32: np.ndarray, z_i32: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `_expected_fma_i32`. Parameters: `x_i32`, `y_i32`, `z_i32`. Key calls include `_expected_add_i32`, `_expected_mul_i32`.
- **CN:** 定义辅助函数 `_expected_fma_i32`。 参数：`x_i32`、`y_i32`、`z_i32`。 关键调用包括 `_expected_add_i32`、`_expected_mul_i32`。

#### Lines 1103-1103

```python
    return _expected_add_i32(_expected_mul_i32(x_i32, y_i32), z_i32)
```
- **EN:** Invokes `_expected_add_i32`, `_expected_mul_i32` to execute the test logic.
- **CN:** 调用 `_expected_add_i32`、`_expected_mul_i32` 执行测试逻辑。

### Lines 1104-1106

```python


def _expected_trunc_ext_roundtrip_i32(x_i32: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `_expected_trunc_ext_roundtrip_i32`. Parameters: `x_i32`. Key calls include `_mix_f32_bits_to_payload_u32`, `_signed_cast_payload_u64`, `_unmix_payload_u32_to_f32_bits_i32`.
- **CN:** 定义辅助函数 `_expected_trunc_ext_roundtrip_i32`。 参数：`x_i32`。 关键调用包括 `_mix_f32_bits_to_payload_u32`、`_signed_cast_payload_u64`、`_unmix_payload_u32_to_f32_bits_i32`。

#### Lines 1107-1110

```python
    x_u32 = _mix_f32_bits_to_payload_u32(x_i32)
    trunc_u16 = _signed_cast_payload_u64(x_u32, 32, 16)
    out_u32 = _signed_cast_payload_u64(trunc_u16, 16, 32).astype(np.uint32)
    return _unmix_payload_u32_to_f32_bits_i32(out_u32)
```
- **EN:** Prepares or updates state through `x_u32`, `trunc_u16`, `out_u32`. Invokes `_mix_f32_bits_to_payload_u32`, `_signed_cast_payload_u64`, `_unmix_payload_u32_to_f32_bits_i32` to execute the test logic.
- **CN:** 通过 `x_u32`、`trunc_u16`、`out_u32` 准备或更新状态。 调用 `_mix_f32_bits_to_payload_u32`、`_signed_cast_payload_u64`、`_unmix_payload_u32_to_f32_bits_i32` 执行测试逻辑。

### Lines 1111-1113

```python


def _expected_ext_f16_to_f32_i32(x_i16: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `_expected_ext_f16_to_f32_i32`. Parameters: `x_i16`. Key calls include `_mix_float_bits_to_payload_u64`, `_unmix_payload_u32_to_f32_bits_i32`, `x_i16.view`, `_signed_cast_payload_u64`.
- **CN:** 定义辅助函数 `_expected_ext_f16_to_f32_i32`。 参数：`x_i16`。 关键调用包括 `_mix_float_bits_to_payload_u64`、`_unmix_payload_u32_to_f32_bits_i32`、`x_i16.view`、`_signed_cast_payload_u64`。

#### Lines 1114-1116

```python
    payload_u16 = _mix_float_bits_to_payload_u64(x_i16.view(np.uint16), 16, 0x3C00)
    out_u32 = _signed_cast_payload_u64(payload_u16, 16, 32).astype(np.uint32)
    return _unmix_payload_u32_to_f32_bits_i32(out_u32)
```
- **EN:** Prepares or updates state through `payload_u16`, `out_u32`. Invokes `_mix_float_bits_to_payload_u64`, `x_i16.view`, `_signed_cast_payload_u64`, `_unmix_payload_u32_to_f32_bits_i32` to execute the test logic.
- **CN:** 通过 `payload_u16`、`out_u32` 准备或更新状态。 调用 `_mix_float_bits_to_payload_u64`、`x_i16.view`、`_signed_cast_payload_u64`、`_unmix_payload_u32_to_f32_bits_i32` 执行测试逻辑。

### Lines 1117-1120

```python


@gluon.jit
def _fma_kernel(x_ptr, y_ptr, z_ptr, out_ptr, n_elements, BLOCK: gl.constexpr, THREADS_PER_WARP: gl.constexpr):
```
- **EN:** Defines the helper function `_fma_kernel`. Decorators: `gluon.jit`. Parameters: `x_ptr`, `y_ptr`, `z_ptr`, `out_ptr`, `n_elements`, `BLOCK`, `THREADS_PER_WARP`. Key calls include `gl.program_id`, `gl.BlockedLayout`, `gl.load`, `gl.fma`, `gl.store`, `gl.arange`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `_fma_kernel`。 装饰器：`gluon.jit`。 参数：`x_ptr`、`y_ptr`、`z_ptr`、`out_ptr`、`n_elements`、`BLOCK`、`THREADS_PER_WARP`。 关键调用包括 `gl.program_id`、`gl.BlockedLayout`、`gl.load`、`gl.fma`、`gl.store`、`gl.arange`。 该作用域涉及布局变换推理。

#### Lines 1121-1130

```python
    pid = gl.program_id(0)
    layout: gl.constexpr = gl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4],
                                            order=[0])
    offs = pid * BLOCK + gl.arange(0, BLOCK, layout=layout)
    mask = offs < n_elements
    x = gl.load(x_ptr + offs, mask=mask, other=0.0)
    y = gl.load(y_ptr + offs, mask=mask, other=0.0)
    z = gl.load(z_ptr + offs, mask=mask, other=0.0)
    out = gl.fma(x, y, z)
    gl.store(out_ptr + offs, out, mask=mask)
```
- **EN:** Prepares or updates state through `pid`, `layout`, `offs`, `mask`, `x`, `y`, `z`, `out`. Invokes `gl.program_id`, `gl.BlockedLayout`, `gl.arange`, `gl.load`, `gl.fma`, `gl.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `pid`、`layout`、`offs`、`mask`、`x`、`y`、`z`、`out` 准备或更新状态。 调用 `gl.program_id`、`gl.BlockedLayout`、`gl.arange`、`gl.load`、`gl.fma`、`gl.store` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1131-1133

```python


def test_fma_payload_semantics(device, fresh_knobs):
```
- **EN:** Defines the test function `test_fma_payload_semantics`. Parameters: `device`, `fresh_knobs`. Key calls include `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, `triton.TensorWrapper`, and 7 more. This scope touches PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义测试函数 `test_fma_payload_semantics`。 参数：`device`、`fresh_knobs`。 关键调用包括 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty`、`triton.TensorWrapper` 等另外 7 项。 该作用域涉及PyTorch 张量准备与校验、随机数据生成。

#### Lines 1134-1162

```python
    _require_cuda_backend(device)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"

    n_elements = 1024
    BLOCK = 256

    g = torch.Generator(device="cuda")
    g.manual_seed(7)
    x = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    y = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    z = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    out = torch.empty((n_elements, ), dtype=torch.int32, device="cuda")

    xw = triton.TensorWrapper(x, dtype=torch.float32)
    yw = triton.TensorWrapper(y, dtype=torch.float32)
    zw = triton.TensorWrapper(z, dtype=torch.float32)
    outw = triton.TensorWrapper(out, dtype=torch.float32)

    grid = (triton.cdiv(n_elements, BLOCK), )
    _fma_kernel[grid](xw, yw, zw, outw, n_elements, BLOCK=BLOCK, THREADS_PER_WARP=THREADS_PER_WARP)

    out_np = out.cpu().numpy().astype(np.int32, copy=False)
    exp_np = _expected_fma_i32(
        x.cpu().numpy().astype(np.int32, copy=False),
        y.cpu().numpy().astype(np.int32, copy=False),
        z.cpu().numpy().astype(np.int32, copy=False),
    )
    _assert_payload_equal(out_np, exp_np)
```
- **EN:** Prepares or updates state through `fresh_knobs`, `n_elements`, `BLOCK`, `g`, `x`, `y`, `z`, `out`, and 7 more. Invokes `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.empty`, `triton.TensorWrapper`, and 7 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `fresh_knobs`、`n_elements`、`BLOCK`、`g`、`x`、`y`、`z`、`out` 等另外 7 项 准备或更新状态。 调用 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.empty`、`triton.TensorWrapper` 等另外 7 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 1163-1166

```python


@gluon.jit
def _cast_trunc_ext_kernel(x_ptr, out_ptr, n_elements, BLOCK: gl.constexpr, THREADS_PER_WARP: gl.constexpr):
```
- **EN:** Defines the helper function `_cast_trunc_ext_kernel`. Decorators: `gluon.jit`. Parameters: `x_ptr`, `out_ptr`, `n_elements`, `BLOCK`, `THREADS_PER_WARP`. Key calls include `gl.program_id`, `gl.BlockedLayout`, `gl.load`, `x.to`, `y.to`, `gl.store`, and 1 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `_cast_trunc_ext_kernel`。 装饰器：`gluon.jit`。 参数：`x_ptr`、`out_ptr`、`n_elements`、`BLOCK`、`THREADS_PER_WARP`。 关键调用包括 `gl.program_id`、`gl.BlockedLayout`、`gl.load`、`x.to`、`y.to`、`gl.store` 等另外 1 项。 该作用域涉及布局变换推理。

#### Lines 1167-1175

```python
    pid = gl.program_id(0)
    layout: gl.constexpr = gl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4],
                                            order=[0])
    offs = pid * BLOCK + gl.arange(0, BLOCK, layout=layout)
    mask = offs < n_elements
    x = gl.load(x_ptr + offs, mask=mask, other=0.0)
    y = x.to(gl.float16)
    z = y.to(gl.float32)
    gl.store(out_ptr + offs, z, mask=mask)
```
- **EN:** Prepares or updates state through `pid`, `layout`, `offs`, `mask`, `x`, `y`, `z`. Invokes `gl.program_id`, `gl.BlockedLayout`, `gl.arange`, `gl.load`, `x.to`, `y.to`, and 1 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `pid`、`layout`、`offs`、`mask`、`x`、`y`、`z` 准备或更新状态。 调用 `gl.program_id`、`gl.BlockedLayout`、`gl.arange`、`gl.load`、`x.to`、`y.to` 等另外 1 项 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1176-1178

```python


def test_cast_trunc_ext_payload_semantics(device, fresh_knobs):
```
- **EN:** Defines the test function `test_cast_trunc_ext_payload_semantics`. Parameters: `device`, `fresh_knobs`. Key calls include `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.tensor`, `torch.empty`, and 7 more. This scope touches PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义测试函数 `test_cast_trunc_ext_payload_semantics`。 参数：`device`、`fresh_knobs`。 关键调用包括 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.tensor`、`torch.empty` 等另外 7 项。 该作用域涉及PyTorch 张量准备与校验、随机数据生成。

#### Lines 1179-1202

```python
    _require_cuda_backend(device)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"

    n_elements = 1024
    BLOCK = 256

    g = torch.Generator(device="cuda")
    g.manual_seed(17)
    x = torch.randint(-(2**31), 2**31 - 1, (n_elements, ), dtype=torch.int32, device="cuda", generator=g)
    special_f32_bits = np.asarray([-1.0, 0.0, 1.0], dtype=np.float32).view(np.int32)
    x[:3] = torch.tensor(special_f32_bits, dtype=torch.int32, device="cuda")
    out = torch.empty((n_elements, ), dtype=torch.int32, device="cuda")

    xw = triton.TensorWrapper(x, dtype=torch.float32)
    outw = triton.TensorWrapper(out, dtype=torch.float32)

    grid = (triton.cdiv(n_elements, BLOCK), )
    _cast_trunc_ext_kernel[grid](xw, outw, n_elements, BLOCK=BLOCK, THREADS_PER_WARP=THREADS_PER_WARP)

    out_np = out.cpu().numpy().astype(np.int32, copy=False)
    exp_np = _expected_trunc_ext_roundtrip_i32(x.cpu().numpy().astype(np.int32, copy=False))
    _assert_payload_equal(out_np, exp_np)
    _assert_payload_equal(out_np[:3], special_f32_bits)
```
- **EN:** Prepares or updates state through `fresh_knobs`, `n_elements`, `BLOCK`, `g`, `x`, `special_f32_bits`, `out`, `xw`, and 4 more. Invokes `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `np.asarray`, `torch.tensor`, and 7 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `fresh_knobs`、`n_elements`、`BLOCK`、`g`、`x`、`special_f32_bits`、`out`、`xw` 等另外 4 项 准备或更新状态。 调用 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`np.asarray`、`torch.tensor` 等另外 7 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 1203-1206

```python


@gluon.jit
def _cast_ext_kernel(x_ptr, out_ptr, n_elements, BLOCK: gl.constexpr, THREADS_PER_WARP: gl.constexpr):
```
- **EN:** Defines the helper function `_cast_ext_kernel`. Decorators: `gluon.jit`. Parameters: `x_ptr`, `out_ptr`, `n_elements`, `BLOCK`, `THREADS_PER_WARP`. Key calls include `gl.program_id`, `gl.BlockedLayout`, `gl.load`, `x.to`, `gl.store`, `gl.arange`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `_cast_ext_kernel`。 装饰器：`gluon.jit`。 参数：`x_ptr`、`out_ptr`、`n_elements`、`BLOCK`、`THREADS_PER_WARP`。 关键调用包括 `gl.program_id`、`gl.BlockedLayout`、`gl.load`、`x.to`、`gl.store`、`gl.arange`。 该作用域涉及布局变换推理。

#### Lines 1207-1214

```python
    pid = gl.program_id(0)
    layout: gl.constexpr = gl.BlockedLayout(size_per_thread=[2], threads_per_warp=[THREADS_PER_WARP], warps_per_cta=[4],
                                            order=[0])
    offs = pid * BLOCK + gl.arange(0, BLOCK, layout=layout)
    mask = offs < n_elements
    x = gl.load(x_ptr + offs, mask=mask, other=0.0)
    z = x.to(gl.float32)
    gl.store(out_ptr + offs, z, mask=mask)
```
- **EN:** Prepares or updates state through `pid`, `layout`, `offs`, `mask`, `x`, `z`. Invokes `gl.program_id`, `gl.BlockedLayout`, `gl.arange`, `gl.load`, `x.to`, `gl.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `pid`、`layout`、`offs`、`mask`、`x`、`z` 准备或更新状态。 调用 `gl.program_id`、`gl.BlockedLayout`、`gl.arange`、`gl.load`、`x.to`、`gl.store` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 1215-1217

```python


def test_cast_ext_payload_semantics(device, fresh_knobs):
```
- **EN:** Defines the test function `test_cast_ext_payload_semantics`. Parameters: `device`, `fresh_knobs`. Key calls include `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `torch.tensor`, `torch.empty`, and 7 more. This scope touches PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义测试函数 `test_cast_ext_payload_semantics`。 参数：`device`、`fresh_knobs`。 关键调用包括 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`torch.tensor`、`torch.empty` 等另外 7 项。 该作用域涉及PyTorch 张量准备与校验、随机数据生成。

#### Lines 1218-1242

```python
    _require_cuda_backend(device)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"

    n_elements = 1024
    BLOCK = 256

    g = torch.Generator(device="cuda")
    g.manual_seed(19)
    x = torch.randint(-(2**15), 2**15 - 1, (n_elements, ), dtype=torch.int16, device="cuda", generator=g)
    special_f16_bits = np.asarray([-1.0, 0.0, 1.0], dtype=np.float16).view(np.int16)
    special_f32_bits = np.asarray([-1.0, 0.0, 1.0], dtype=np.float32).view(np.int32)
    x[:3] = torch.tensor(special_f16_bits, dtype=torch.int16, device="cuda")
    out = torch.empty((n_elements, ), dtype=torch.int32, device="cuda")

    xw = triton.TensorWrapper(x, dtype=torch.float16)
    outw = triton.TensorWrapper(out, dtype=torch.float32)

    grid = (triton.cdiv(n_elements, BLOCK), )
    _cast_ext_kernel[grid](xw, outw, n_elements, BLOCK=BLOCK, THREADS_PER_WARP=THREADS_PER_WARP)

    out_np = out.cpu().numpy().astype(np.int32, copy=False)
    exp_np = _expected_ext_f16_to_f32_i32(x.cpu().numpy().astype(np.int16, copy=False))
    _assert_payload_equal(out_np, exp_np)
    _assert_payload_equal(out_np[:3], special_f32_bits)
```
- **EN:** Prepares or updates state through `fresh_knobs`, `n_elements`, `BLOCK`, `g`, `x`, `special_f16_bits`, `special_f32_bits`, `out`, and 5 more. Invokes `_require_cuda_backend`, `torch.Generator`, `g.manual_seed`, `torch.randint`, `np.asarray`, `torch.tensor`, and 7 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `fresh_knobs`、`n_elements`、`BLOCK`、`g`、`x`、`special_f16_bits`、`special_f32_bits`、`out` 等另外 5 项 准备或更新状态。 调用 `_require_cuda_backend`、`torch.Generator`、`g.manual_seed`、`torch.randint`、`np.asarray`、`torch.tensor` 等另外 7 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 1243-1246

```python


def _mm_payload_u32(a_i32: np.ndarray, b_i32: np.ndarray, c_i32: np.ndarray = None) -> np.ndarray:
    # Computes: c + a @ b in Z/(2^32) on mixed f32 payload bits.
```
- **EN:** Defines the helper function `_mm_payload_u32`. Parameters: `a_i32`, `b_i32`, `c_i32`. Key calls include `np.empty`, `np.uint64`, `_unmix_payload_u32_to_f32_bits_i32`, `out.astype`, `_mix_f32_bits_to_payload_u32`.
- **CN:** 定义辅助函数 `_mm_payload_u32`。 参数：`a_i32`、`b_i32`、`c_i32`。 关键调用包括 `np.empty`、`np.uint64`、`_unmix_payload_u32_to_f32_bits_i32`、`out.astype`、`_mix_f32_bits_to_payload_u32`。

#### Lines 1247-1251

```python
    a_u = _mix_f32_bits_to_payload_u32(a_i32).astype(np.uint64)
    b_u = _mix_f32_bits_to_payload_u32(b_i32).astype(np.uint64)
    c_u = _mix_f32_bits_to_payload_u32(c_i32).astype(np.uint64) if c_i32 is not None else None
    m, k = a_u.shape
    k2, n = b_u.shape
```
- **EN:** Prepares or updates state through `a_u`, `b_u`, `c_u`, `m`, `k`, `k2`, `n`. Invokes `_mix_f32_bits_to_payload_u32` to execute the test logic.
- **CN:** 通过 `a_u`、`b_u`、`c_u`、`m`、`k`、`k2`、`n` 准备或更新状态。 调用 `_mix_f32_bits_to_payload_u32` 执行测试逻辑。

#### Lines 1252-1252

```python
    assert k == k2
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

#### Lines 1253-1254

```python
    out = np.empty((m, n), dtype=np.uint64)
    mask = np.uint64(0xFFFFFFFF)
```
- **EN:** Prepares or updates state through `out`, `mask`. Invokes `np.empty`, `np.uint64` to execute the test logic.
- **CN:** 通过 `out`、`mask` 准备或更新状态。 调用 `np.empty`、`np.uint64` 执行测试逻辑。

#### Lines 1255-1260

```python
    for i in range(m):
        for j in range(n):
            s = c_u[i, j] if c_u is not None else 0
            for kk in range(k):
                s = (s + (a_u[i, kk] * b_u[kk, j])) & mask
            out[i, j] = s
```
- **EN:** Iterates across cases or data tiles.
- **CN:** 通过循环覆盖多个用例或数据分块。

#### Lines 1261-1261

```python
    return _unmix_payload_u32_to_f32_bits_i32(out.astype(np.uint32))
```
- **EN:** Invokes `_unmix_payload_u32_to_f32_bits_i32`, `out.astype` to execute the test logic.
- **CN:** 调用 `_unmix_payload_u32_to_f32_bits_i32`、`out.astype` 执行测试逻辑。

### Lines 1262-1264

```python


def _bmm_payload_u32(a_i32: np.ndarray, b_i32: np.ndarray, c_i32: np.ndarray = None) -> np.ndarray:
```
- **EN:** Defines the helper function `_bmm_payload_u32`. Parameters: `a_i32`, `b_i32`, `c_i32`. Key calls include `np.empty`, `_mm_payload_u32`.
- **CN:** 定义辅助函数 `_bmm_payload_u32`。 参数：`a_i32`、`b_i32`、`c_i32`。 关键调用包括 `np.empty`、`_mm_payload_u32`。

#### Lines 1265-1269

```python
    assert a_i32.ndim == 3
    assert b_i32.ndim == 3
    assert c_i32 is None or c_i32.ndim == 3
    assert a_i32.shape[0] == b_i32.shape[0]
    assert c_i32 is None or a_i32.shape[0] == c_i32.shape[0]
```
- **EN:** Validates behavior with 5 assertion(s).
- **CN:** 通过 5 个断言验证行为。

#### Lines 1270-1270

```python
    out = np.empty((a_i32.shape[0], a_i32.shape[1], b_i32.shape[2]), dtype=np.int32)
```
- **EN:** Prepares or updates state through `out`. Invokes `np.empty` to execute the test logic.
- **CN:** 通过 `out` 准备或更新状态。 调用 `np.empty` 执行测试逻辑。

#### Lines 1271-1273

```python
    for batch in range(a_i32.shape[0]):
        c_batch = c_i32[batch] if c_i32 is not None else None
        out[batch] = _mm_payload_u32(a_i32[batch], b_i32[batch], c_batch)
```
- **EN:** Invokes `_mm_payload_u32` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `_mm_payload_u32` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 1274-1274

```python
    return out
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 1275-1277

```python


def _unpack_element(data: np.ndarray, row: int, col: int, pack: int, pack_axis: int = 1) -> np.uint64:
```
- **EN:** Defines the helper function `_unpack_element`. Parameters: `data`, `row`, `col`, `pack`, `pack_axis`. Key calls include `np.uint64`.
- **CN:** 定义辅助函数 `_unpack_element`。 参数：`data`、`row`、`col`、`pack`、`pack_axis`。 关键调用包括 `np.uint64`。

#### Lines 1278-1283

```python
    if pack_axis == 1:
        raw = np.uint64(data[row, col // pack])
        nibble_idx = col
    else:
        raw = np.uint64(data[row // pack, col])
        nibble_idx = row
```
- **EN:** Invokes `np.uint64` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `np.uint64` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1284-1285

```python
    if pack == 2:
        return (raw >> np.uint64(4 * (nibble_idx % pack))) & np.uint64(0x0F)
```
- **EN:** Invokes `np.uint64` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `np.uint64` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1286-1286

```python
    return raw
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 1287-1289

```python


def _mix_float_scalar(val: np.uint64, bitwidth: int, one_bits: int) -> np.uint64:
```
- **EN:** Defines the helper function `_mix_float_scalar`. Parameters: `val`, `bitwidth`, `one_bits`. Key calls include `_mix_float_bits_to_payload_u64`, `np.uint64`, `np.asarray`.
- **CN:** 定义辅助函数 `_mix_float_scalar`。 参数：`val`、`bitwidth`、`one_bits`。 关键调用包括 `_mix_float_bits_to_payload_u64`、`np.uint64`、`np.asarray`。

#### Lines 1290-1291

```python
    mixed = _mix_float_bits_to_payload_u64(np.asarray([val], dtype=np.uint64), bitwidth, one_bits)
    return np.uint64(mixed[0])
```
- **EN:** Prepares or updates state through `mixed`. Invokes `_mix_float_bits_to_payload_u64`, `np.asarray`, `np.uint64` to execute the test logic.
- **CN:** 通过 `mixed` 准备或更新状态。 调用 `_mix_float_bits_to_payload_u64`、`np.asarray`、`np.uint64` 执行测试逻辑。

### Lines 1292-1294

```python


def _mix_dot_scaled_elem(val: np.uint64, elem_type: str) -> np.uint64:
```
- **EN:** Defines the helper function `_mix_dot_scaled_elem`. Parameters: `val`, `elem_type`. Key calls include `_mix_float_scalar`.
- **CN:** 定义辅助函数 `_mix_dot_scaled_elem`。 参数：`val`、`elem_type`。 关键调用包括 `_mix_float_scalar`。

#### Lines 1295-1297

```python
    if elem_type in ("e4m3", "e5m2"):
        one_bits = 0x38 if elem_type == "e4m3" else 0x3C
        return _mix_float_scalar(val, 8, one_bits)
```
- **EN:** Invokes `_mix_float_scalar` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `_mix_float_scalar` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1298-1299

```python
    if elem_type == "bf16":
        return _mix_float_scalar(val, 16, 0x3F80)
```
- **EN:** Invokes `_mix_float_scalar` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `_mix_float_scalar` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1300-1300

```python
    return val
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 1301-1303

```python


def _signed_cast_payload_scalar(payload: np.uint64, src_bitwidth: int, dst_bitwidth: int) -> np.uint64:
```
- **EN:** Defines the helper function `_signed_cast_payload_scalar`. Parameters: `payload`, `src_bitwidth`, `dst_bitwidth`. Key calls include `_signed_cast_payload_u64`, `np.uint64`, `np.asarray`.
- **CN:** 定义辅助函数 `_signed_cast_payload_scalar`。 参数：`payload`、`src_bitwidth`、`dst_bitwidth`。 关键调用包括 `_signed_cast_payload_u64`、`np.uint64`、`np.asarray`。

#### Lines 1304-1305

```python
    casted = _signed_cast_payload_u64(np.asarray([payload], dtype=np.uint64), src_bitwidth, dst_bitwidth)
    return np.uint64(casted[0])
```
- **EN:** Prepares or updates state through `casted`. Invokes `_signed_cast_payload_u64`, `np.asarray`, `np.uint64` to execute the test logic.
- **CN:** 通过 `casted` 准备或更新状态。 调用 `_signed_cast_payload_u64`、`np.asarray`、`np.uint64` 执行测试逻辑。

### Lines 1306-1308

```python


def _dot_scaled_compute_payload_elem(val: np.uint64, elem_type: str, compute_type: str) -> np.uint64:
```
- **EN:** Defines the helper function `_dot_scaled_compute_payload_elem`. Parameters: `val`, `elem_type`, `compute_type`. Key calls include `_mix_dot_scaled_elem`, `_signed_cast_payload_scalar`, `_mix_float_scalar`, `np.uint64`.
- **CN:** 定义辅助函数 `_dot_scaled_compute_payload_elem`。 参数：`val`、`elem_type`、`compute_type`。 关键调用包括 `_mix_dot_scaled_elem`、`_signed_cast_payload_scalar`、`_mix_float_scalar`、`np.uint64`。

#### Lines 1309-1309

```python
    assert compute_type in ("bf16", "fp16")
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

#### Lines 1310-1310

```python
    compute_width = 16
```
- **EN:** Prepares or updates state through `compute_width`.
- **CN:** 通过 `compute_width` 准备或更新状态。

#### Lines 1311-1313

```python
    if elem_type in ("e4m3", "e5m2"):
        payload = _mix_dot_scaled_elem(val, elem_type)
        return _signed_cast_payload_scalar(payload, 8, compute_width)
```
- **EN:** Invokes `_mix_dot_scaled_elem`, `_signed_cast_payload_scalar` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `_mix_dot_scaled_elem`、`_signed_cast_payload_scalar` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1314-1316

```python
    if elem_type == "bf16":
        payload = _mix_float_scalar(val, 16, 0x3F80)
        return _signed_cast_payload_scalar(payload, 16, compute_width)
```
- **EN:** Invokes `_mix_float_scalar`, `_signed_cast_payload_scalar` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `_mix_float_scalar`、`_signed_cast_payload_scalar` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1317-1319

```python
    if elem_type == "fp16":
        payload = _mix_float_scalar(val, 16, 0x3C00)
        return _signed_cast_payload_scalar(payload, 16, compute_width)
```
- **EN:** Invokes `_mix_float_scalar`, `_signed_cast_payload_scalar` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `_mix_float_scalar`、`_signed_cast_payload_scalar` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1320-1324

```python

    # Match sanitized fp4_to_fp: unpacked e2m1 bits are zero-extended into the
    # destination floating-point payload.  Float6 formats use the same fallback
    # until the sanitizer has dtype-specific float6 mixing.
    return val & np.uint64(0xFFFF)
```
- **EN:** Invokes `np.uint64` to execute the test logic.
- **CN:** 调用 `np.uint64` 执行测试逻辑。

### Lines 1325-1327

```python


def _dot_scaled_scale_payload(raw_scale: np.uint64, compute_type: str) -> np.uint64:
```
- **EN:** Defines the helper function `_dot_scaled_scale_payload`. Parameters: `raw_scale`, `compute_type`. Key calls include `ValueError`, `_mix_float_scalar`, `_signed_cast_payload_scalar`, `np.uint64`.
- **CN:** 定义辅助函数 `_dot_scaled_scale_payload`。 参数：`raw_scale`、`compute_type`。 关键调用包括 `ValueError`、`_mix_float_scalar`、`_signed_cast_payload_scalar`、`np.uint64`。

#### Lines 1328-1330

```python
    if compute_type == "bf16":
        raw_bf16 = (raw_scale & np.uint64(0xFF)) << np.uint64(7)
        return _mix_float_scalar(raw_bf16, 16, 0x3F80)
```
- **EN:** Invokes `_mix_float_scalar`, `np.uint64` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `_mix_float_scalar`、`np.uint64` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1331-1334

```python
    if compute_type == "fp16":
        raw_f32 = (raw_scale & np.uint64(0xFF)) << np.uint64(23)
        payload_f32 = _mix_float_scalar(raw_f32, 32, 0x3F800000)
        return _signed_cast_payload_scalar(payload_f32, 32, 16)
```
- **EN:** Invokes `_mix_float_scalar`, `_signed_cast_payload_scalar`, `np.uint64` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `_mix_float_scalar`、`_signed_cast_payload_scalar`、`np.uint64` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1335-1335

```python
    raise ValueError(f"unsupported dot_scaled compute type: {compute_type}")
```
- **EN:** Invokes `ValueError` to execute the test logic.
- **CN:** 调用 `ValueError` 执行测试逻辑。

### Lines 1336-1339

```python


def _dot_scaled_payload_u32(a_data: np.ndarray, b_data: np.ndarray, a_scale, b_scale, a_pack: int, b_pack: int,
                            type_a: str, type_b: str) -> np.ndarray:
```
- **EN:** Defines the helper function `_dot_scaled_payload_u32`. Parameters: `a_data`, `b_data`, `a_scale`, `b_scale`, `a_pack`, `b_pack`, `type_a`, `type_b`. Key calls include `np.uint64`, `np.zeros`, `itertools.product`, `_unmix_payload_u32_to_f32_bits_i32`, `out.astype`, `_unpack_element`, and 3 more.
- **CN:** 定义辅助函数 `_dot_scaled_payload_u32`。 参数：`a_data`、`b_data`、`a_scale`、`b_scale`、`a_pack`、`b_pack`、`type_a`、`type_b`。 关键调用包括 `np.uint64`、`np.zeros`、`itertools.product`、`_unmix_payload_u32_to_f32_bits_i32`、`out.astype`、`_unpack_element` 等另外 3 项。

#### Lines 1340-1345

```python
    M, N = a_data.shape[0], b_data.shape[1]
    K = a_data.shape[1] * a_pack
    compute_type = "fp16" if "fp16" in (type_a, type_b) else "bf16"
    compute_mask = np.uint64(0xFFFF)
    mask = np.uint64(0xFFFFFFFF)
    out = np.zeros((M, N), dtype=np.uint64)
```
- **EN:** Prepares or updates state through `M`, `N`, `K`, `compute_type`, `compute_mask`, `mask`, `out`. Invokes `np.uint64`, `np.zeros` to execute the test logic.
- **CN:** 通过 `M`、`N`、`K`、`compute_type`、`compute_mask`、`mask`、`out` 准备或更新状态。 调用 `np.uint64`、`np.zeros` 执行测试逻辑。

#### Lines 1346-1362

```python
    for i, j in itertools.product(range(M), range(N)):
        s = np.uint64(0)
        for kk in range(K):
            a_val = _unpack_element(a_data, i, kk, a_pack, pack_axis=1)
            b_val = _unpack_element(b_data, kk, j, b_pack, pack_axis=0)
            a_val = _dot_scaled_compute_payload_elem(a_val, type_a, compute_type)
            b_val = _dot_scaled_compute_payload_elem(b_val, type_b, compute_type)
            if a_scale is not None:
                a_scale_val = _dot_scaled_scale_payload(np.uint64(a_scale[i, kk // 32]), compute_type)
                a_val = (a_val * a_scale_val) & compute_mask
            if b_scale is not None:
                b_scale_val = _dot_scaled_scale_payload(np.uint64(b_scale[j, kk // 32]), compute_type)
                b_val = (b_val * b_scale_val) & compute_mask
            a_val = _signed_cast_payload_scalar(a_val, 16, 32)
            b_val = _signed_cast_payload_scalar(b_val, 16, 32)
            s = (s + a_val * b_val) & mask
        out[i, j] = s
```
- **EN:** Invokes `itertools.product`, `np.uint64`, `_unpack_element`, `_dot_scaled_compute_payload_elem`, `_signed_cast_payload_scalar`, `_dot_scaled_scale_payload` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `itertools.product`、`np.uint64`、`_unpack_element`、`_dot_scaled_compute_payload_elem`、`_signed_cast_payload_scalar`、`_dot_scaled_scale_payload` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 1363-1363

```python
    return _unmix_payload_u32_to_f32_bits_i32(out.astype(np.uint32))
```
- **EN:** Invokes `_unmix_payload_u32_to_f32_bits_i32`, `out.astype` to execute the test logic.
- **CN:** 调用 `_unmix_payload_u32_to_f32_bits_i32`、`out.astype` 执行测试逻辑。

### Lines 1364-1368

```python


def _mm_scaled_payload_u32(a_u8: np.ndarray, b_u8: np.ndarray, a_scale_u8: np.ndarray, b_scale_u8: np.ndarray,
                           c_i32: np.ndarray = None, a_pack: int = 1, b_pack: int = 1,
                           elem_type: str = "e2m1") -> np.ndarray:
```
- **EN:** Defines the helper function `_mm_scaled_payload_u32`. Parameters: `a_u8`, `b_u8`, `a_scale_u8`, `b_scale_u8`, `c_i32`, `a_pack`, `b_pack`, `elem_type`. Nested definitions in this scope: `unpack_payload_matrix`, `compute_payload_matrix`, `scale_payload_matrix`. Key calls include `a_scale_u8.astype`, `b_scale_u8.astype`, `compute_payload_matrix`, `scale_payload_matrix`, `np.uint64`, `_unmix_payload_u32_to_f32_bits_i32`, and 9 more.
- **CN:** 定义辅助函数 `_mm_scaled_payload_u32`。 参数：`a_u8`、`b_u8`、`a_scale_u8`、`b_scale_u8`、`c_i32`、`a_pack`、`b_pack`、`elem_type`。 该作用域中的嵌套定义：`unpack_payload_matrix`、`compute_payload_matrix`、`scale_payload_matrix`。 关键调用包括 `a_scale_u8.astype`、`b_scale_u8.astype`、`compute_payload_matrix`、`scale_payload_matrix`、`np.uint64`、`_unmix_payload_u32_to_f32_bits_i32` 等另外 9 项。

#### Lines 1369-1375

```python
    a_scale = a_scale_u8.astype(np.uint64)
    b_scale = b_scale_u8.astype(np.uint64)
    c_u = _mix_f32_bits_to_payload_u32(c_i32).astype(np.uint64) if c_i32 is not None else None

    m = a_u8.shape[0]
    n = b_u8.shape[1]
    k = a_u8.shape[1] * a_pack
```
- **EN:** Prepares or updates state through `a_scale`, `b_scale`, `c_u`, `m`, `n`, `k`. Invokes `a_scale_u8.astype`, `b_scale_u8.astype`, `_mix_f32_bits_to_payload_u32` to execute the test logic.
- **CN:** 通过 `a_scale`、`b_scale`、`c_u`、`m`、`n`、`k` 准备或更新状态。 调用 `a_scale_u8.astype`、`b_scale_u8.astype`、`_mix_f32_bits_to_payload_u32` 执行测试逻辑。

#### Lines 1376-1378

```python
    assert k == b_u8.shape[0] * b_pack
    assert a_scale.shape == (m, k // 32)
    assert b_scale.shape == (n, k // 32)
```
- **EN:** Validates behavior with 3 assertion(s).
- **CN:** 通过 3 个断言验证行为。

#### Lines 1379-1380

```python

    def unpack_payload_matrix(data: np.ndarray, pack: int, pack_axis: int) -> np.ndarray:
```
- **EN:** Defines the helper function `unpack_payload_matrix`. Parameters: `data`, `pack`, `pack_axis`. Key calls include `np.empty`, `data.astype`, `np.uint64`.
- **CN:** 定义辅助函数 `unpack_payload_matrix`。 参数：`data`、`pack`、`pack_axis`。 关键调用包括 `np.empty`、`data.astype`、`np.uint64`。

##### Lines 1381-1382

```python
        if pack == 1:
            return data.astype(np.uint64)
```
- **EN:** Invokes `data.astype` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `data.astype` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 1383-1383

```python
        assert pack == 2
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

##### Lines 1384-1388

```python
        if pack_axis == 1:
            out = np.empty((data.shape[0], data.shape[1] * pack), dtype=np.uint64)
            out[:, 0::2] = data.astype(np.uint64) & np.uint64(0x0F)
            out[:, 1::2] = (data.astype(np.uint64) >> np.uint64(4)) & np.uint64(0x0F)
            return out
```
- **EN:** Invokes `np.empty`, `data.astype`, `np.uint64` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `np.empty`、`data.astype`、`np.uint64` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 1389-1392

```python
        out = np.empty((data.shape[0] * pack, data.shape[1]), dtype=np.uint64)
        out[0::2, :] = data.astype(np.uint64) & np.uint64(0x0F)
        out[1::2, :] = (data.astype(np.uint64) >> np.uint64(4)) & np.uint64(0x0F)
        return out
```
- **EN:** Prepares or updates state through `out`. Invokes `np.empty`, `data.astype`, `np.uint64` to execute the test logic.
- **CN:** 通过 `out` 准备或更新状态。 调用 `np.empty`、`data.astype`、`np.uint64` 执行测试逻辑。

#### Lines 1393-1394

```python

    def compute_payload_matrix(data: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `compute_payload_matrix`. Parameters: `data`. Key calls include `_mix_float_bits_to_payload_u64`, `_signed_cast_payload_u64`, `np.uint64`.
- **CN:** 定义辅助函数 `compute_payload_matrix`。 参数：`data`。 关键调用包括 `_mix_float_bits_to_payload_u64`、`_signed_cast_payload_u64`、`np.uint64`。

##### Lines 1395-1398

```python
        if elem_type in ("e4m3", "e5m2"):
            one_bits = 0x38 if elem_type == "e4m3" else 0x3C
            payload = _mix_float_bits_to_payload_u64(data, 8, one_bits)
            return _signed_cast_payload_u64(payload, 8, 16)
```
- **EN:** Invokes `_mix_float_bits_to_payload_u64`, `_signed_cast_payload_u64` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `_mix_float_bits_to_payload_u64`、`_signed_cast_payload_u64` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 1399-1399

```python
        return data & np.uint64(0xFFFF)
```
- **EN:** Invokes `np.uint64` to execute the test logic.
- **CN:** 调用 `np.uint64` 执行测试逻辑。

#### Lines 1400-1401

```python

    def scale_payload_matrix(raw_scale: np.ndarray) -> np.ndarray:
```
- **EN:** Defines the helper function `scale_payload_matrix`. Parameters: `raw_scale`. Key calls include `_mix_float_bits_to_payload_u64`, `np.uint64`.
- **CN:** 定义辅助函数 `scale_payload_matrix`。 参数：`raw_scale`。 关键调用包括 `_mix_float_bits_to_payload_u64`、`np.uint64`。

##### Lines 1402-1403

```python
        raw_bf16 = (raw_scale & np.uint64(0xFF)) << np.uint64(7)
        return _mix_float_bits_to_payload_u64(raw_bf16, 16, 0x3F80)
```
- **EN:** Prepares or updates state through `raw_bf16`. Invokes `np.uint64`, `_mix_float_bits_to_payload_u64` to execute the test logic.
- **CN:** 通过 `raw_bf16` 准备或更新状态。 调用 `np.uint64`、`_mix_float_bits_to_payload_u64` 执行测试逻辑。

#### Lines 1404-1412

```python

    a_payload = compute_payload_matrix(unpack_payload_matrix(a_u8, a_pack, pack_axis=1))
    b_payload = compute_payload_matrix(unpack_payload_matrix(b_u8, b_pack, pack_axis=0))
    a_scale_payload = scale_payload_matrix(a_scale)
    b_scale_payload = scale_payload_matrix(b_scale)

    out = c_u.copy() if c_u is not None else np.zeros((m, n), dtype=np.uint64)
    compute_mask = np.uint64(0xFFFF)
    mask32 = np.uint64(0xFFFFFFFF)
```
- **EN:** Prepares or updates state through `a_payload`, `b_payload`, `a_scale_payload`, `b_scale_payload`, `out`, `compute_mask`, `mask32`. Invokes `compute_payload_matrix`, `unpack_payload_matrix`, `scale_payload_matrix`, `c_u.copy`, `np.zeros`, `np.uint64` to execute the test logic.
- **CN:** 通过 `a_payload`、`b_payload`、`a_scale_payload`、`b_scale_payload`、`out`、`compute_mask`、`mask32` 准备或更新状态。 调用 `compute_payload_matrix`、`unpack_payload_matrix`、`scale_payload_matrix`、`c_u.copy`、`np.zeros`、`np.uint64` 执行测试逻辑。

#### Lines 1413-1420

```python
    for group in range(k // 32):
        start = group * 32
        end = start + 32
        lhs = (a_payload[:, start:end] * a_scale_payload[:, group:group + 1]) & compute_mask
        rhs = (b_payload[start:end, :] * b_scale_payload[:, group][None, :]) & compute_mask
        lhs = _signed_cast_payload_u64(lhs, 16, 32)
        rhs = _signed_cast_payload_u64(rhs, 16, 32)
        out = (out + (lhs @ rhs)) & mask32
```
- **EN:** Invokes `_signed_cast_payload_u64` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `_signed_cast_payload_u64` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 1421-1421

```python
    return _unmix_payload_u32_to_f32_bits_i32(out.astype(np.uint32))
```
- **EN:** Invokes `_unmix_payload_u32_to_f32_bits_i32`, `out.astype` to execute the test logic.
- **CN:** 调用 `_unmix_payload_u32_to_f32_bits_i32`、`out.astype` 执行测试逻辑。

### Lines 1422-1424

```python


def test_dot_fma(device, fresh_knobs):
```
- **EN:** Defines the test function `test_dot_fma`. Parameters: `device`, `fresh_knobs`. Nested definitions in this scope: `kernel`. Key calls include `_require_cuda_backend`, `gl.constexpr`, `np.random.RandomState`, `rs.randint`, `_mm_payload_u32`, `torch.tensor`, and 11 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_dot_fma`。 参数：`device`、`fresh_knobs`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `_require_cuda_backend`、`gl.constexpr`、`np.random.RandomState`、`rs.randint`、`_mm_payload_u32`、`torch.tensor` 等另外 11 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 1425-1430

```python
    _require_cuda_backend(device)

    B = 16
    BLOCK = gl.constexpr(B)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"
```
- **EN:** Prepares or updates state through `B`, `BLOCK`, `fresh_knobs`. Invokes `_require_cuda_backend`, `gl.constexpr` to execute the test logic.
- **CN:** 通过 `B`、`BLOCK`、`fresh_knobs` 准备或更新状态。 调用 `_require_cuda_backend`、`gl.constexpr` 执行测试逻辑。

#### Lines 1431-1433

```python

    @gluon.jit
    def kernel(a_ptr, b_ptr, c_ptr, out_ptr, THREADS_PER_WARP: gl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_ptr`, `b_ptr`, `c_ptr`, `out_ptr`, `THREADS_PER_WARP`. Key calls include `gl.BlockedLayout`, `gl.DotOperandLayout`, `gl.convert_layout`, `gl.load`, `gl.dot_fma`, `gl.store`, and 2 more. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_ptr`、`b_ptr`、`c_ptr`、`out_ptr`、`THREADS_PER_WARP`。 关键调用包括 `gl.BlockedLayout`、`gl.DotOperandLayout`、`gl.convert_layout`、`gl.load`、`gl.dot_fma`、`gl.store` 等另外 2 项。 该作用域涉及布局变换推理、随机数据生成。

##### Lines 1434-1452

```python
        layout: gl.constexpr = gl.BlockedLayout([1, 1], [THREADS_PER_WARP, 1], [4, 1], [1, 0])
        lhs_layout: gl.constexpr = gl.DotOperandLayout(parent=layout, operand_index=0, k_width=0)
        rhs_layout: gl.constexpr = gl.DotOperandLayout(parent=layout, operand_index=1, k_width=0)

        offs_m = gl.arange(0, BLOCK, layout=gl.SliceLayout(1, layout))[:, None]
        offs_n = gl.arange(0, BLOCK, layout=gl.SliceLayout(0, layout))[None, :]
        # Important: build separate offsets for A and B.
        # dot_fma expects operands to represent A[M,K] and B[K,N]. Using the same
        # linearized (m,n) offsets for both makes B effectively transposed.
        offs_k = gl.arange(0, BLOCK, layout=gl.SliceLayout(0, layout))[None, :]
        a_offs = offs_m * BLOCK + offs_k
        b_offs = offs_n * BLOCK + offs_m  # load B^T so dot_fma produces A @ B
        out_offs = offs_m * BLOCK + offs_n

        a = gl.convert_layout(gl.load(a_ptr + a_offs), lhs_layout)
        b = gl.convert_layout(gl.load(b_ptr + b_offs), rhs_layout)
        c = gl.load(c_ptr + out_offs)
        out = gl.dot_fma(a, b, c)
        gl.store(out_ptr + out_offs, out)
```
- **EN:** Prepares or updates state through `layout`, `lhs_layout`, `rhs_layout`, `offs_m`, `offs_n`, `offs_k`, `a_offs`, `b_offs`, and 5 more. Invokes `gl.BlockedLayout`, `gl.DotOperandLayout`, `gl.arange`, `gl.SliceLayout`, `gl.convert_layout`, `gl.load`, and 2 more to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `layout`、`lhs_layout`、`rhs_layout`、`offs_m`、`offs_n`、`offs_k`、`a_offs`、`b_offs` 等另外 5 项 准备或更新状态。 调用 `gl.BlockedLayout`、`gl.DotOperandLayout`、`gl.arange`、`gl.SliceLayout`、`gl.convert_layout`、`gl.load` 等另外 2 项 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

#### Lines 1453-1472

```python

    rs = np.random.RandomState(0)
    a_bits = rs.randint(-(2**31), 2**31 - 1, size=(B, B), dtype=np.int32)
    b_bits = rs.randint(-(2**31), 2**31 - 1, size=(B, B), dtype=np.int32)
    c_bits = rs.randint(-(2**31), 2**31 - 1, size=(B, B), dtype=np.int32)
    exp_bits = _mm_payload_u32(a_bits, b_bits.T, c_bits)

    a = torch.tensor(a_bits, device="cuda", dtype=torch.int32)
    b = torch.tensor(b_bits, device="cuda", dtype=torch.int32)
    c = torch.tensor(c_bits, device="cuda", dtype=torch.int32)
    out = torch.empty((B, B), device="cuda", dtype=torch.int32)

    # Wrap int storage as fp32 so fpsan operates on payload bits.
    aw = triton.TensorWrapper(a, dtype=torch.float32)
    bw = triton.TensorWrapper(b, dtype=torch.float32)
    cw = triton.TensorWrapper(c, dtype=torch.float32)
    outw = triton.TensorWrapper(out, dtype=torch.float32)

    compiled = kernel[(1, )](aw, bw, cw, outw, THREADS_PER_WARP=THREADS_PER_WARP)
    ttgir = compiled.asm["ttgir"]
```
- **EN:** Prepares or updates state through `rs`, `a_bits`, `b_bits`, `c_bits`, `exp_bits`, `a`, `b`, `c`, and 7 more. Invokes `np.random.RandomState`, `rs.randint`, `_mm_payload_u32`, `torch.tensor`, `torch.empty`, `triton.TensorWrapper` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `rs`、`a_bits`、`b_bits`、`c_bits`、`exp_bits`、`a`、`b`、`c` 等另外 7 项 准备或更新状态。 调用 `np.random.RandomState`、`rs.randint`、`_mm_payload_u32`、`torch.tensor`、`torch.empty`、`triton.TensorWrapper` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 1473-1474

```python
    assert "ttng.tc_gen5_mma" not in ttgir
    assert "ttng.warp_group_dot" not in ttgir
```
- **EN:** Validates behavior with 2 assertion(s).
- **CN:** 通过 2 个断言验证行为。

#### Lines 1475-1476

```python

    _assert_payload_equal(out, exp_bits)
```
- **EN:** Invokes `_assert_payload_equal` to execute the test logic.
- **CN:** 调用 `_assert_payload_equal` 执行测试逻辑。

### Lines 1477-1479

```python


def test_dot_fma_batched(device, fresh_knobs):
```
- **EN:** Defines the test function `test_dot_fma_batched`. Parameters: `device`, `fresh_knobs`. Nested definitions in this scope: `kernel`. Key calls include `_require_cuda_backend`, `gl.constexpr`, `np.random.RandomState`, `rs.randint`, `_bmm_payload_u32`, `torch.tensor`, and 11 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_dot_fma_batched`。 参数：`device`、`fresh_knobs`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `_require_cuda_backend`、`gl.constexpr`、`np.random.RandomState`、`rs.randint`、`_bmm_payload_u32`、`torch.tensor` 等另外 11 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 1480-1487

```python
    _require_cuda_backend(device)

    BATCH_SIZE = 2
    B = 16
    BATCH = gl.constexpr(BATCH_SIZE)
    BLOCK = gl.constexpr(B)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"
```
- **EN:** Prepares or updates state through `BATCH_SIZE`, `B`, `BATCH`, `BLOCK`, `fresh_knobs`. Invokes `_require_cuda_backend`, `gl.constexpr` to execute the test logic.
- **CN:** 通过 `BATCH_SIZE`、`B`、`BATCH`、`BLOCK`、`fresh_knobs` 准备或更新状态。 调用 `_require_cuda_backend`、`gl.constexpr` 执行测试逻辑。

#### Lines 1488-1490

```python

    @gluon.jit
    def kernel(a_ptr, b_ptr, c_ptr, out_ptr, THREADS_PER_WARP: gl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_ptr`, `b_ptr`, `c_ptr`, `out_ptr`, `THREADS_PER_WARP`. Key calls include `gl.BlockedLayout`, `gl.DotOperandLayout`, `gl.convert_layout`, `gl.load`, `gl.dot_fma`, `gl.store`, and 2 more. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_ptr`、`b_ptr`、`c_ptr`、`out_ptr`、`THREADS_PER_WARP`。 关键调用包括 `gl.BlockedLayout`、`gl.DotOperandLayout`、`gl.convert_layout`、`gl.load`、`gl.dot_fma`、`gl.store` 等另外 2 项。 该作用域涉及布局变换推理、随机数据生成。

##### Lines 1491-1509

```python
        layout: gl.constexpr = gl.BlockedLayout([1, 1, 1], [1, THREADS_PER_WARP, 1], [1, 4, 1], [2, 1, 0])
        lhs_layout: gl.constexpr = gl.DotOperandLayout(parent=layout, operand_index=0, k_width=0)
        rhs_layout: gl.constexpr = gl.DotOperandLayout(parent=layout, operand_index=1, k_width=0)

        offs_batch = gl.arange(0, BATCH, layout=gl.SliceLayout(1, parent=gl.SliceLayout(2, layout)))[:, None, None]
        offs_m = gl.arange(0, BLOCK, layout=gl.SliceLayout(0, parent=gl.SliceLayout(2, layout)))[None, :, None]
        offs_n = gl.arange(0, BLOCK, layout=gl.SliceLayout(0, parent=gl.SliceLayout(1, layout)))[None, None, :]
        offs_k_a = gl.arange(0, BLOCK, layout=gl.SliceLayout(0, parent=gl.SliceLayout(1, layout)))[None, None, :]
        offs_k_b = gl.arange(0, BLOCK, layout=gl.SliceLayout(0, parent=gl.SliceLayout(2, layout)))[None, :, None]

        a_offs = offs_batch * BLOCK * BLOCK + offs_m * BLOCK + offs_k_a
        b_offs = offs_batch * BLOCK * BLOCK + offs_k_b * BLOCK + offs_n
        out_offs = offs_batch * BLOCK * BLOCK + offs_m * BLOCK + offs_n

        a = gl.convert_layout(gl.load(a_ptr + a_offs), lhs_layout)
        b = gl.convert_layout(gl.load(b_ptr + b_offs), rhs_layout)
        c = gl.load(c_ptr + out_offs)
        out = gl.dot_fma(a, b, c)
        gl.store(out_ptr + out_offs, out)
```
- **EN:** Prepares or updates state through `layout`, `lhs_layout`, `rhs_layout`, `offs_batch`, `offs_m`, `offs_n`, `offs_k_a`, `offs_k_b`, and 7 more. Invokes `gl.BlockedLayout`, `gl.DotOperandLayout`, `gl.arange`, `gl.SliceLayout`, `gl.convert_layout`, `gl.load`, and 2 more to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `layout`、`lhs_layout`、`rhs_layout`、`offs_batch`、`offs_m`、`offs_n`、`offs_k_a`、`offs_k_b` 等另外 7 项 准备或更新状态。 调用 `gl.BlockedLayout`、`gl.DotOperandLayout`、`gl.arange`、`gl.SliceLayout`、`gl.convert_layout`、`gl.load` 等另外 2 项 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

#### Lines 1510-1528

```python

    rs = np.random.RandomState(1)
    a_bits = rs.randint(-(2**31), 2**31 - 1, size=(BATCH_SIZE, B, B), dtype=np.int32)
    b_bits = rs.randint(-(2**31), 2**31 - 1, size=(BATCH_SIZE, B, B), dtype=np.int32)
    c_bits = rs.randint(-(2**31), 2**31 - 1, size=(BATCH_SIZE, B, B), dtype=np.int32)
    exp_bits = _bmm_payload_u32(a_bits, b_bits, c_bits)

    a = torch.tensor(a_bits, device="cuda", dtype=torch.int32)
    b = torch.tensor(b_bits, device="cuda", dtype=torch.int32)
    c = torch.tensor(c_bits, device="cuda", dtype=torch.int32)
    out = torch.empty((BATCH_SIZE, B, B), device="cuda", dtype=torch.int32)

    aw = triton.TensorWrapper(a, dtype=torch.float32)
    bw = triton.TensorWrapper(b, dtype=torch.float32)
    cw = triton.TensorWrapper(c, dtype=torch.float32)
    outw = triton.TensorWrapper(out, dtype=torch.float32)

    compiled = kernel[(1, )](aw, bw, cw, outw, THREADS_PER_WARP=THREADS_PER_WARP)
    ttgir = compiled.asm["ttgir"]
```
- **EN:** Prepares or updates state through `rs`, `a_bits`, `b_bits`, `c_bits`, `exp_bits`, `a`, `b`, `c`, and 7 more. Invokes `np.random.RandomState`, `rs.randint`, `_bmm_payload_u32`, `torch.tensor`, `torch.empty`, `triton.TensorWrapper` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `rs`、`a_bits`、`b_bits`、`c_bits`、`exp_bits`、`a`、`b`、`c` 等另外 7 项 准备或更新状态。 调用 `np.random.RandomState`、`rs.randint`、`_bmm_payload_u32`、`torch.tensor`、`torch.empty`、`triton.TensorWrapper` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 1529-1530

```python
    assert "ttng.tc_gen5_mma" not in ttgir
    assert "ttng.warp_group_dot" not in ttgir
```
- **EN:** Validates behavior with 2 assertion(s).
- **CN:** 通过 2 个断言验证行为。

#### Lines 1531-1532

```python

    _assert_payload_equal(out, exp_bits)
```
- **EN:** Invokes `_assert_payload_equal` to execute the test logic.
- **CN:** 调用 `_assert_payload_equal` 执行测试逻辑。

### Lines 1533-1537

```python


@pytest.mark.skipif(not is_hopper(), reason="Requires Hopper")
@pytest.mark.parametrize(("use_acc", "is_async"), [(False, False), (True, False), (True, True)])
def test_warpgroup_mma(device, use_acc, is_async, fresh_knobs):
```
- **EN:** Defines the test function `test_warpgroup_mma`. Decorators: `pytest.mark.skipif(not is_hopper(), reason='Requires Hopper')`, `pytest.mark.parametrize(('use_acc', 'is_async'), [(False, False), (True, False), (True, True)])`. Parameters: `device`, `use_acc`, `is_async`, `fresh_knobs`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `_require_cuda_backend`, `gl.constexpr`, `np.random.RandomState`, `rs.randint`, and 18 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_warpgroup_mma`。 装饰器：`pytest.mark.skipif(not is_hopper(), reason='Requires Hopper')`、`pytest.mark.parametrize(('use_acc', 'is_async'), [(False, False), (True, False), (True, True)])`。 参数：`device`、`use_acc`、`is_async`、`fresh_knobs`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`_require_cuda_backend`、`gl.constexpr`、`np.random.RandomState`、`rs.randint` 等另外 18 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 1538-1543

```python
    _require_cuda_backend(device)

    B = 64
    BLOCK = gl.constexpr(B)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"
```
- **EN:** Prepares or updates state through `B`, `BLOCK`, `fresh_knobs`. Invokes `_require_cuda_backend`, `gl.constexpr` to execute the test logic.
- **CN:** 通过 `B`、`BLOCK`、`fresh_knobs` 准备或更新状态。 调用 `_require_cuda_backend`、`gl.constexpr` 执行测试逻辑。

#### Lines 1544-1546

```python

    @gluon.jit
    def kernel(a_ptr, b_ptr, c_ptr, out_ptr, USE_ACC: gl.constexpr, IS_ASYNC: gl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_ptr`, `b_ptr`, `c_ptr`, `out_ptr`, `USE_ACC`, `IS_ASYNC`. Key calls include `gl.BlockedLayout`, `gl.NVMMADistributedLayout`, `gl.load`, `gl.NVMMASharedLayout.get_default_for`, `gl.allocate_shared_memory`, `gl.convert_layout`, and 6 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_ptr`、`b_ptr`、`c_ptr`、`out_ptr`、`USE_ACC`、`IS_ASYNC`。 关键调用包括 `gl.BlockedLayout`、`gl.NVMMADistributedLayout`、`gl.load`、`gl.NVMMASharedLayout.get_default_for`、`gl.allocate_shared_memory`、`gl.convert_layout` 等另外 6 项。 该作用域涉及布局变换推理。

##### Lines 1547-1565

```python
        layout: gl.constexpr = gl.BlockedLayout([1, 1], [32, 1], [gl.num_warps(), 1], [1, 0])
        acc_layout: gl.constexpr = gl.NVMMADistributedLayout(version=[3, 0], warps_per_cta=[4, 1],
                                                             instr_shape=[16, 32, 16])

        offs_m = gl.arange(0, BLOCK, layout=gl.SliceLayout(1, layout))[:, None]
        offs_n = gl.arange(0, BLOCK, layout=gl.SliceLayout(0, layout))[None, :]
        offs_k_row = gl.arange(0, BLOCK, layout=gl.SliceLayout(1, layout))[:, None]
        offs_k_col = gl.arange(0, BLOCK, layout=gl.SliceLayout(0, layout))[None, :]

        a_tile = gl.load(a_ptr + offs_m * BLOCK + offs_k_col)
        b_tile = gl.load(b_ptr + offs_k_row * BLOCK + offs_n)
        c_tile = gl.load(c_ptr + offs_m * BLOCK + offs_n)

        smem_layout: gl.constexpr = gl.NVMMASharedLayout.get_default_for([BLOCK, BLOCK], gl.float32)
        smem_a = gl.allocate_shared_memory(gl.float32, [BLOCK, BLOCK], smem_layout, a_tile)
        smem_b = gl.allocate_shared_memory(gl.float32, [BLOCK, BLOCK], smem_layout, b_tile)

        acc = gl.convert_layout(c_tile, acc_layout)
        acc = hopper.warpgroup_mma(smem_a, smem_b, acc, use_acc=USE_ACC, precision="tf32", is_async=IS_ASYNC)
```
- **EN:** Prepares or updates state through `layout`, `acc_layout`, `offs_m`, `offs_n`, `offs_k_row`, `offs_k_col`, `a_tile`, `b_tile`, and 5 more. Invokes `gl.BlockedLayout`, `gl.num_warps`, `gl.NVMMADistributedLayout`, `gl.arange`, `gl.SliceLayout`, `gl.load`, and 4 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`acc_layout`、`offs_m`、`offs_n`、`offs_k_row`、`offs_k_col`、`a_tile`、`b_tile` 等另外 5 项 准备或更新状态。 调用 `gl.BlockedLayout`、`gl.num_warps`、`gl.NVMMADistributedLayout`、`gl.arange`、`gl.SliceLayout`、`gl.load` 等另外 4 项 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 1566-1567

```python
        if IS_ASYNC:
            acc = hopper.warpgroup_mma_wait(num_outstanding=0, deps=[acc])
```
- **EN:** Invokes `hopper.warpgroup_mma_wait` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `hopper.warpgroup_mma_wait` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 1568-1569

```python
        out = gl.convert_layout(acc, layout)
        gl.store(out_ptr + offs_m * BLOCK + offs_n, out)
```
- **EN:** Prepares or updates state through `out`. Invokes `gl.convert_layout`, `gl.store` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `out` 准备或更新状态。 调用 `gl.convert_layout`、`gl.store` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1570-1589

```python

    rs = np.random.RandomState(0)
    a_bits = rs.randint(-(2**31), 2**31 - 1, size=(B, B), dtype=np.int32)
    b_bits = rs.randint(-(2**31), 2**31 - 1, size=(B, B), dtype=np.int32)
    c_bits = rs.randint(-(2**31), 2**31 - 1, size=(B, B), dtype=np.int32)
    exp_bits = _mm_payload_u32(a_bits, b_bits, c_bits if use_acc else None)

    a = torch.tensor(a_bits, device="cuda", dtype=torch.int32)
    b = torch.tensor(b_bits, device="cuda", dtype=torch.int32)
    c = torch.tensor(c_bits, device="cuda", dtype=torch.int32)
    out = torch.empty((B, B), device="cuda", dtype=torch.int32)

    aw = triton.TensorWrapper(a, dtype=torch.float32)
    bw = triton.TensorWrapper(b, dtype=torch.float32)
    cw = triton.TensorWrapper(c, dtype=torch.float32)
    outw = triton.TensorWrapper(out, dtype=torch.float32)

    kernel[(1, )](aw, bw, cw, outw, USE_ACC=use_acc, IS_ASYNC=is_async)

    _assert_payload_equal(out, exp_bits)
```
- **EN:** Prepares or updates state through `rs`, `a_bits`, `b_bits`, `c_bits`, `exp_bits`, `a`, `b`, `c`, and 5 more. Invokes `np.random.RandomState`, `rs.randint`, `_mm_payload_u32`, `torch.tensor`, `torch.empty`, `triton.TensorWrapper`, and 1 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `rs`、`a_bits`、`b_bits`、`c_bits`、`exp_bits`、`a`、`b`、`c` 等另外 5 项 准备或更新状态。 调用 `np.random.RandomState`、`rs.randint`、`_mm_payload_u32`、`torch.tensor`、`torch.empty`、`triton.TensorWrapper` 等另外 1 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 1590-1595

```python


@pytest.mark.skipif(not (is_hip_cdna4() or is_hip_gfx1250()), reason="Requires DotScaledOp support (CDNA4, or GFX1250)")
@pytest.mark.parametrize("type_a", ["e2m1", "e4m3", "e5m2"])
@pytest.mark.parametrize("type_b", ["e2m1", "e4m3", "e5m2", "bf16"])
def test_dot_scaled(device, type_a, type_b, fresh_knobs):
```
- **EN:** Defines the test function `test_dot_scaled`. Decorators: `pytest.mark.skipif(not (is_hip_cdna4() or is_hip_gfx1250()), reason='Requires DotScaledOp support (CDNA4, or GFX1250)')`, `pytest.mark.parametrize('type_a', ['e2m1', 'e4m3', 'e5m2'])`, `pytest.mark.parametrize('type_b', ['e2m1', 'e4m3', 'e5m2', 'bf16'])`. Parameters: `device`, `type_a`, `type_b`, `fresh_knobs`. Nested definitions in this scope: `allocator`, `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `_require_cuda_backend`, `triton.set_allocator`, `np.random.RandomState`, `torch.tensor`, and 11 more. This scope touches pytest parametrization, Triton compilation or JIT kernels, Triton language operations, PyTorch tensor setup and checks.
- **CN:** 定义测试函数 `test_dot_scaled`。 装饰器：`pytest.mark.skipif(not (is_hip_cdna4() or is_hip_gfx1250()), reason='Requires DotScaledOp support (CDNA4, or GFX1250)')`、`pytest.mark.parametrize('type_a', ['e2m1', 'e4m3', 'e5m2'])`、`pytest.mark.parametrize('type_b', ['e2m1', 'e4m3', 'e5m2', 'bf16'])`。 参数：`device`、`type_a`、`type_b`、`fresh_knobs`。 该作用域中的嵌套定义：`allocator`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`_require_cuda_backend`、`triton.set_allocator`、`np.random.RandomState`、`torch.tensor` 等另外 11 项。 该作用域涉及pytest 参数化、Triton 编译或 JIT kernel、Triton language 操作、PyTorch 张量准备与校验。

#### Lines 1596-1600

```python
    _require_cuda_backend(device)

    B = 32
    K = 64
    SCALE_K = K // 32
```
- **EN:** Prepares or updates state through `B`, `K`, `SCALE_K`. Invokes `_require_cuda_backend` to execute the test logic.
- **CN:** 通过 `B`、`K`、`SCALE_K` 准备或更新状态。 调用 `_require_cuda_backend` 执行测试逻辑。

#### Lines 1601-1602

```python

    def allocator(size: int, alignment: int, stream):
```
- **EN:** Defines the helper function `allocator`. Parameters: `size`, `alignment`, `stream`. Key calls include `torch.empty`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `allocator`。 参数：`size`、`alignment`、`stream`。 关键调用包括 `torch.empty`。 该作用域涉及PyTorch 张量准备与校验。

##### Lines 1603-1603

```python
        return torch.empty(size, device="cuda", dtype=torch.int32)
```
- **EN:** Invokes `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 1604-1606

```python

    triton.set_allocator(allocator)
    fresh_knobs.compilation.instrumentation_mode = "fpsan"
```
- **EN:** Prepares or updates state through `fresh_knobs`. Invokes `triton.set_allocator` to execute the test logic.
- **CN:** 通过 `fresh_knobs` 准备或更新状态。 调用 `triton.set_allocator` 执行测试逻辑。

#### Lines 1607-1610

```python

    @triton.jit
    def kernel(a_ptr, a_scale_ptr, b_ptr, b_scale_ptr, out_ptr, BLOCK_M: tl.constexpr, BLOCK_N: tl.constexpr,
               BLOCK_K: tl.constexpr, TYPE_A: tl.constexpr, TYPE_B: tl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `triton.jit`. Parameters: `a_ptr`, `a_scale_ptr`, `b_ptr`, `b_scale_ptr`, `out_ptr`, `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, and 2 more. Key calls include `tl.load`, `tl.dot_scaled`, `tl.store`, `tl.arange`. This scope touches Triton compilation or JIT kernels, Triton language operations.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`triton.jit`。 参数：`a_ptr`、`a_scale_ptr`、`b_ptr`、`b_scale_ptr`、`out_ptr`、`BLOCK_M`、`BLOCK_N`、`BLOCK_K` 等另外 2 项。 关键调用包括 `tl.load`、`tl.dot_scaled`、`tl.store`、`tl.arange`。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作。

##### Lines 1611-1631

```python
        DIV_FACTOR_A: tl.constexpr = 2 if TYPE_A == "e2m1" else 1
        DIV_FACTOR_B: tl.constexpr = 2 if TYPE_B == "e2m1" else 1
        PACKED_BLOCK_K_A: tl.constexpr = BLOCK_K // DIV_FACTOR_A
        PACKED_BLOCK_K_B: tl.constexpr = BLOCK_K // DIV_FACTOR_B
        SCALE_BLOCK_K: tl.constexpr = BLOCK_K // 32

        offs_am = tl.arange(0, BLOCK_M)[:, None]
        offs_bn = tl.arange(0, BLOCK_N)[None, :]
        offs_ak = tl.arange(0, PACKED_BLOCK_K_A)[None, :]
        offs_bk = tl.arange(0, PACKED_BLOCK_K_B)[:, None]

        a = tl.load(a_ptr + offs_am * PACKED_BLOCK_K_A + offs_ak)
        b = tl.load(b_ptr + offs_bk * BLOCK_N + offs_bn)

        offs_scale_ak = tl.arange(0, SCALE_BLOCK_K)[None, :]
        offs_scale_bk = tl.arange(0, SCALE_BLOCK_K)[None, :]
        a_scale = tl.load(a_scale_ptr + offs_am * SCALE_BLOCK_K + offs_scale_ak)
        b_scale = tl.load(b_scale_ptr + tl.arange(0, BLOCK_N)[:, None] * SCALE_BLOCK_K + offs_scale_bk)

        c = tl.dot_scaled(a, a_scale, TYPE_A, b, b_scale, TYPE_B)
        tl.store(out_ptr + offs_am * BLOCK_N + offs_bn, c)
```
- **EN:** Prepares or updates state through `DIV_FACTOR_A`, `DIV_FACTOR_B`, `PACKED_BLOCK_K_A`, `PACKED_BLOCK_K_B`, `SCALE_BLOCK_K`, `offs_am`, `offs_bn`, `offs_ak`, and 8 more. Invokes `tl.arange`, `tl.load`, `tl.dot_scaled`, `tl.store` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `DIV_FACTOR_A`、`DIV_FACTOR_B`、`PACKED_BLOCK_K_A`、`PACKED_BLOCK_K_B`、`SCALE_BLOCK_K`、`offs_am`、`offs_bn`、`offs_ak` 等另外 8 项 准备或更新状态。 调用 `tl.arange`、`tl.load`、`tl.dot_scaled`、`tl.store` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 1632-1647

```python

    a_pack = 2 if type_a == "e2m1" else 1
    b_pack = 2 if type_b == "e2m1" else 1
    packed_k_a = K // a_pack
    packed_k_b = K // b_pack

    rs = np.random.RandomState(1)
    a_bits = rs.randint(0, 256, size=(B, packed_k_a)).astype(np.uint8)
    b_bits = rs.randint(0, 256, size=(packed_k_b, B)).astype(np.uint8)
    a_scale_bits = rs.randint(0, 255, size=(B, SCALE_K)).astype(np.uint8)
    b_scale_bits = rs.randint(0, 255, size=(B, SCALE_K)).astype(np.uint8)

    a = torch.tensor(a_bits, device="cuda", dtype=torch.uint8)
    b = torch.tensor(b_bits, device="cuda", dtype=torch.uint8)
    a_scale = torch.tensor(a_scale_bits, device="cuda", dtype=torch.uint8)
    b_scale = torch.tensor(b_scale_bits, device="cuda", dtype=torch.uint8)
```
- **EN:** Prepares or updates state through `a_pack`, `b_pack`, `packed_k_a`, `packed_k_b`, `rs`, `a_bits`, `b_bits`, `a_scale_bits`, and 5 more. Invokes `np.random.RandomState`, `rs.randint`, `torch.tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `a_pack`、`b_pack`、`packed_k_a`、`packed_k_b`、`rs`、`a_bits`、`b_bits`、`a_scale_bits` 等另外 5 项 准备或更新状态。 调用 `np.random.RandomState`、`rs.randint`、`torch.tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 1648-1651

```python

    if type_b == "bf16":
        b_bits = rs.randint(0, 65536, size=(packed_k_b, B)).astype(np.uint16)
        b = torch.tensor(b_bits, device="cuda", dtype=torch.uint16).view(torch.bfloat16)
```
- **EN:** Invokes `rs.randint`, `torch.tensor` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 调用 `rs.randint`、`torch.tensor` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 1652-1661

```python

    exp_bits = _dot_scaled_payload_u32(a_bits, b_bits, a_scale_bits, None if type_b == "bf16" else b_scale_bits, a_pack,
                                       b_pack, type_a, type_b)

    out = torch.empty((B, B), device="cuda", dtype=torch.int32)
    outw = triton.TensorWrapper(out, dtype=torch.float32)

    kernel[(1, )](a, a_scale, b, b_scale, outw, BLOCK_M=B, BLOCK_N=B, BLOCK_K=K, TYPE_A=type_a, TYPE_B=type_b)

    _assert_payload_equal(out, exp_bits)
```
- **EN:** Prepares or updates state through `exp_bits`, `out`, `outw`. Invokes `_dot_scaled_payload_u32`, `torch.empty`, `triton.TensorWrapper`, `_assert_payload_equal` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `exp_bits`、`out`、`outw` 准备或更新状态。 调用 `_dot_scaled_payload_u32`、`torch.empty`、`triton.TensorWrapper`、`_assert_payload_equal` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 1662-1666

```python


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
@pytest.mark.parametrize("use_acc", [False, True])
def test_tcgen05_mma(device, use_acc, fresh_knobs):
```
- **EN:** Defines the test function `test_tcgen05_mma`. Decorators: `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`, `pytest.mark.parametrize('use_acc', [False, True])`. Parameters: `device`, `use_acc`, `fresh_knobs`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `_require_cuda_backend`, `gl.constexpr`, `np.random.RandomState`, `rs.randint`, and 28 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_tcgen05_mma`。 装饰器：`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`、`pytest.mark.parametrize('use_acc', [False, True])`。 参数：`device`、`use_acc`、`fresh_knobs`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`_require_cuda_backend`、`gl.constexpr`、`np.random.RandomState`、`rs.randint` 等另外 28 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 1667-1672

```python
    _require_cuda_backend(device)

    B = 64
    BLOCK = gl.constexpr(B)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"
```
- **EN:** Prepares or updates state through `B`, `BLOCK`, `fresh_knobs`. Invokes `_require_cuda_backend`, `gl.constexpr` to execute the test logic.
- **CN:** 通过 `B`、`BLOCK`、`fresh_knobs` 准备或更新状态。 调用 `_require_cuda_backend`、`gl.constexpr` 执行测试逻辑。

#### Lines 1673-1675

```python

    @gluon.jit
    def kernel(a_ptr, b_ptr, c_ptr, out_ptr, USE_ACC: gl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_ptr`, `b_ptr`, `c_ptr`, `out_ptr`, `USE_ACC`. Key calls include `gl.BlockedLayout`, `gl.load`, `gl.NVMMASharedLayout.get_default_for`, `gl.allocate_shared_memory`, `smem_a.store`, `smem_b.store`, and 17 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_ptr`、`b_ptr`、`c_ptr`、`out_ptr`、`USE_ACC`。 关键调用包括 `gl.BlockedLayout`、`gl.load`、`gl.NVMMASharedLayout.get_default_for`、`gl.allocate_shared_memory`、`smem_a.store`、`smem_b.store` 等另外 17 项。 该作用域涉及布局变换推理。

##### Lines 1676-1699

```python
        layout: gl.constexpr = gl.BlockedLayout([1, 1], [32, 1], [gl.num_warps(), 1], [1, 0])

        offs_m = gl.arange(0, BLOCK, layout=gl.SliceLayout(1, layout))[:, None]
        offs_n = gl.arange(0, BLOCK, layout=gl.SliceLayout(0, layout))[None, :]
        offs_k_row = gl.arange(0, BLOCK, layout=gl.SliceLayout(1, layout))[:, None]
        offs_k_col = gl.arange(0, BLOCK, layout=gl.SliceLayout(0, layout))[None, :]

        a_offs = offs_m * BLOCK + offs_k_col
        b_offs = offs_k_row * BLOCK + offs_n
        out_offs = offs_m * BLOCK + offs_n

        a_tile = gl.load(a_ptr + a_offs)
        b_tile = gl.load(b_ptr + b_offs)

        smem_layout_a: gl.constexpr = gl.NVMMASharedLayout.get_default_for([BLOCK, BLOCK], gl.float32)
        smem_layout_b: gl.constexpr = gl.NVMMASharedLayout.get_default_for([BLOCK, BLOCK], gl.float32)
        smem_a = gl.allocate_shared_memory(gl.float32, [BLOCK, BLOCK], smem_layout_a)
        smem_b = gl.allocate_shared_memory(gl.float32, [BLOCK, BLOCK], smem_layout_b)
        smem_a.store(a_tile)
        smem_b.store(b_tile)

        tmem_layout: gl.constexpr = TensorMemoryLayout((BLOCK, BLOCK), col_stride=1)
        acc_tmem = allocate_tensor_memory(gl.float32, [BLOCK, BLOCK], layout=tmem_layout)
        acc_reg_layout: gl.constexpr = acc_tmem.get_reg_layout()
```
- **EN:** Prepares or updates state through `layout`, `offs_m`, `offs_n`, `offs_k_row`, `offs_k_col`, `a_offs`, `b_offs`, `out_offs`, and 9 more. Invokes `gl.BlockedLayout`, `gl.num_warps`, `gl.arange`, `gl.SliceLayout`, `gl.load`, `gl.NVMMASharedLayout.get_default_for`, and 6 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`offs_m`、`offs_n`、`offs_k_row`、`offs_k_col`、`a_offs`、`b_offs`、`out_offs` 等另外 9 项 准备或更新状态。 调用 `gl.BlockedLayout`、`gl.num_warps`、`gl.arange`、`gl.SliceLayout`、`gl.load`、`gl.NVMMASharedLayout.get_default_for` 等另外 6 项 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 1700-1703

```python
        if USE_ACC:
            c_tile = gl.load(c_ptr + out_offs)
            acc_init = gl.convert_layout(c_tile, acc_reg_layout)
            acc_tmem.store(acc_init)
```
- **EN:** Invokes `gl.load`, `gl.convert_layout`, `acc_tmem.store` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `gl.load`、`gl.convert_layout`、`acc_tmem.store` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

##### Lines 1704-1716

```python

        bar = gl.allocate_shared_memory(gl.int64, [1], gl.constexpr(mbarrier.MBarrierLayout()))
        mbarrier.init(bar, count=1)

        smem_b_T = smem_b.permute((1, 0))
        tcgen05_mma(smem_a, smem_b_T, acc_tmem, use_acc=USE_ACC, pred=True, mbarriers=[bar])

        mbarrier.wait(bar, phase=0, deps=[smem_a, smem_b])
        mbarrier.invalidate(bar)

        out = acc_tmem.load()
        out = gl.convert_layout(out, layout)
        gl.store(out_ptr + out_offs, out)
```
- **EN:** Prepares or updates state through `bar`, `smem_b_T`, `out`. Invokes `gl.allocate_shared_memory`, `gl.constexpr`, `mbarrier.MBarrierLayout`, `mbarrier.init`, `smem_b.permute`, `tcgen05_mma`, and 5 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `bar`、`smem_b_T`、`out` 准备或更新状态。 调用 `gl.allocate_shared_memory`、`gl.constexpr`、`mbarrier.MBarrierLayout`、`mbarrier.init`、`smem_b.permute`、`tcgen05_mma` 等另外 5 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1717-1736

```python

    rs = np.random.RandomState(0)
    a_bits = rs.randint(-(2**31), 2**31 - 1, size=(B, B), dtype=np.int32)
    b_bits = rs.randint(-(2**31), 2**31 - 1, size=(B, B), dtype=np.int32)
    c_bits = rs.randint(-(2**31), 2**31 - 1, size=(B, B), dtype=np.int32)
    exp_bits = _mm_payload_u32(a_bits, b_bits.T, c_bits if use_acc else None)

    a = torch.tensor(a_bits, device="cuda", dtype=torch.int32)
    b = torch.tensor(b_bits, device="cuda", dtype=torch.int32)
    c = torch.tensor(c_bits, device="cuda", dtype=torch.int32)
    out = torch.empty((B, B), device="cuda", dtype=torch.int32)

    aw = triton.TensorWrapper(a, dtype=torch.float32)
    bw = triton.TensorWrapper(b, dtype=torch.float32)
    cw = triton.TensorWrapper(c, dtype=torch.float32)
    outw = triton.TensorWrapper(out, dtype=torch.float32)

    kernel[(1, )](aw, bw, cw, outw, USE_ACC=use_acc)

    _assert_payload_equal(out, exp_bits)
```
- **EN:** Prepares or updates state through `rs`, `a_bits`, `b_bits`, `c_bits`, `exp_bits`, `a`, `b`, `c`, and 5 more. Invokes `np.random.RandomState`, `rs.randint`, `_mm_payload_u32`, `torch.tensor`, `torch.empty`, `triton.TensorWrapper`, and 1 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `rs`、`a_bits`、`b_bits`、`c_bits`、`exp_bits`、`a`、`b`、`c` 等另外 5 项 准备或更新状态。 调用 `np.random.RandomState`、`rs.randint`、`_mm_payload_u32`、`torch.tensor`、`torch.empty`、`triton.TensorWrapper` 等另外 1 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 1737-1741

```python


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
@pytest.mark.parametrize("elem_type", ["e2m1", "e4m3", "e5m2"])
def test_tcgen05_mma_scaled(device, elem_type, fresh_knobs):
```
- **EN:** Defines the test function `test_tcgen05_mma_scaled`. Decorators: `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`, `pytest.mark.parametrize('elem_type', ['e2m1', 'e4m3', 'e5m2'])`. Parameters: `device`, `elem_type`, `fresh_knobs`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `pytest.mark.parametrize`, `_require_cuda_backend`, `gl.constexpr`, `np.random.RandomState`, `rs.randint`, and 34 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_tcgen05_mma_scaled`。 装饰器：`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`、`pytest.mark.parametrize('elem_type', ['e2m1', 'e4m3', 'e5m2'])`。 参数：`device`、`elem_type`、`fresh_knobs`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`pytest.mark.parametrize`、`_require_cuda_backend`、`gl.constexpr`、`np.random.RandomState`、`rs.randint` 等另外 34 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 1742-1748

```python
    _require_cuda_backend(device)

    B = 128
    BLOCK = gl.constexpr(B)
    SCALE_K = gl.constexpr(B // 32)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"
```
- **EN:** Prepares or updates state through `B`, `BLOCK`, `SCALE_K`, `fresh_knobs`. Invokes `_require_cuda_backend`, `gl.constexpr` to execute the test logic.
- **CN:** 通过 `B`、`BLOCK`、`SCALE_K`、`fresh_knobs` 准备或更新状态。 调用 `_require_cuda_backend`、`gl.constexpr` 执行测试逻辑。

#### Lines 1749-1751

```python

    @gluon.jit
    def kernel(a_ptr, b_ptr, a_scale_ptr, b_scale_ptr, c_ptr, out_ptr, TYPE: gl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_ptr`, `b_ptr`, `a_scale_ptr`, `b_scale_ptr`, `c_ptr`, `out_ptr`, `TYPE`. Key calls include `gl.BlockedLayout`, `gl.NVMMASharedLayout.get_default_for`, `TensorMemoryScalesLayout`, `gl.load`, `gl.allocate_shared_memory`, `TensorMemoryLayout`, and 21 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_ptr`、`b_ptr`、`a_scale_ptr`、`b_scale_ptr`、`c_ptr`、`out_ptr`、`TYPE`。 关键调用包括 `gl.BlockedLayout`、`gl.NVMMASharedLayout.get_default_for`、`TensorMemoryScalesLayout`、`gl.load`、`gl.allocate_shared_memory`、`TensorMemoryLayout` 等另外 21 项。 该作用域涉及布局变换推理。

##### Lines 1752-1770

```python
        layout: gl.constexpr = gl.BlockedLayout([1, 1], [32, 1], [gl.num_warps(), 1], [1, 0])
        IS_FP4: gl.constexpr = TYPE == "e2m1"
        PACK_FACTOR: gl.constexpr = 2 if IS_FP4 else 1
        PACKED_K: gl.constexpr = BLOCK // PACK_FACTOR
        ELEM_DTYPE: gl.constexpr = gl.uint8 if IS_FP4 else (gl.float8e4nv if TYPE == "e4m3" else gl.float8e5)
        a_nvmma_layout: gl.constexpr = gl.NVMMASharedLayout.get_default_for([BLOCK, PACKED_K], ELEM_DTYPE)
        b_nvmma_layout: gl.constexpr = (gl.NVMMASharedLayout.get_default_for([BLOCK, PACKED_K], ELEM_DTYPE)
                                        if IS_FP4 else gl.NVMMASharedLayout(swizzle_byte_width=128, transposed=False,
                                                                            element_bitwidth=8, rank=2))
        scale_layout: gl.constexpr = TensorMemoryScalesLayout()

        offs_m = gl.arange(0, BLOCK, layout=gl.SliceLayout(1, layout))[:, None]
        offs_n = gl.arange(0, BLOCK, layout=gl.SliceLayout(0, layout))[None, :]
        offs_k_row = gl.arange(0, PACKED_K, layout=gl.SliceLayout(1, layout))[:, None]
        offs_k_col = gl.arange(0, PACKED_K, layout=gl.SliceLayout(0, layout))[None, :]

        a_tile = gl.load(a_ptr + offs_m * PACKED_K + offs_k_col)
        c_tile = gl.load(c_ptr + offs_m * BLOCK + offs_n)
        a_smem = gl.allocate_shared_memory(ELEM_DTYPE, [BLOCK, PACKED_K], a_nvmma_layout, a_tile)
```
- **EN:** Prepares or updates state through `layout`, `IS_FP4`, `PACK_FACTOR`, `PACKED_K`, `ELEM_DTYPE`, `a_nvmma_layout`, `b_nvmma_layout`, `scale_layout`, and 7 more. Invokes `gl.BlockedLayout`, `gl.num_warps`, `gl.NVMMASharedLayout.get_default_for`, `gl.NVMMASharedLayout`, `TensorMemoryScalesLayout`, `gl.arange`, and 3 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`IS_FP4`、`PACK_FACTOR`、`PACKED_K`、`ELEM_DTYPE`、`a_nvmma_layout`、`b_nvmma_layout`、`scale_layout` 等另外 7 项 准备或更新状态。 调用 `gl.BlockedLayout`、`gl.num_warps`、`gl.NVMMASharedLayout.get_default_for`、`gl.NVMMASharedLayout`、`TensorMemoryScalesLayout`、`gl.arange` 等另外 3 项 执行测试逻辑。 相关主题：布局变换推理。

##### Lines 1771-1778

```python
        if IS_FP4:
            b_tile = gl.load(b_ptr + offs_m * PACKED_K + offs_k_col)
            b_smem = gl.allocate_shared_memory(ELEM_DTYPE, [BLOCK, PACKED_K], b_nvmma_layout, b_tile)
            b_mma = b_smem.permute((1, 0))
        else:
            b_tile = gl.load(b_ptr + offs_k_row * BLOCK + offs_n)
            b_smem = gl.allocate_shared_memory(ELEM_DTYPE, [PACKED_K, BLOCK], b_nvmma_layout, b_tile)
            b_mma = b_smem
```
- **EN:** Invokes `gl.load`, `gl.allocate_shared_memory`, `b_smem.permute` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `gl.load`、`gl.allocate_shared_memory`、`b_smem.permute` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

##### Lines 1779-1802

```python

        tmem_layout: gl.constexpr = TensorMemoryLayout((BLOCK, BLOCK), col_stride=1)
        acc_tmem = allocate_tensor_memory(gl.float32, [BLOCK, BLOCK], layout=tmem_layout)
        acc_tmem.store(gl.convert_layout(c_tile, acc_tmem.get_reg_layout()))

        a_scale_tmem = allocate_tensor_memory(gl.int8, [BLOCK, SCALE_K], layout=scale_layout)
        b_scale_tmem = allocate_tensor_memory(gl.int8, [BLOCK, SCALE_K], layout=scale_layout)
        a_scale_reg_layout: gl.constexpr = a_scale_tmem.get_reg_layout()
        b_scale_reg_layout: gl.constexpr = b_scale_tmem.get_reg_layout()
        scale_offs_k = gl.arange(0, SCALE_K, layout=gl.SliceLayout(0, a_scale_reg_layout))[None, :]
        scale_offs_m = gl.arange(0, BLOCK, layout=gl.SliceLayout(1, a_scale_reg_layout))[:, None]
        scale_offs_n = gl.arange(0, BLOCK, layout=gl.SliceLayout(1, b_scale_reg_layout))[:, None]
        a_scale_tmem.store(gl.load(a_scale_ptr + scale_offs_m * SCALE_K + scale_offs_k))
        b_scale_tmem.store(gl.load(b_scale_ptr + scale_offs_n * SCALE_K + scale_offs_k))

        bar = gl.allocate_shared_memory(gl.int64, [1], gl.constexpr(mbarrier.MBarrierLayout()))
        mbarrier.init(bar, count=1)
        tcgen05_mma_scaled(a_smem, b_mma, acc_tmem, a_scale_tmem, b_scale_tmem, TYPE, TYPE, use_acc=True,
                           mbarriers=[bar])
        mbarrier.wait(bar, phase=0)
        mbarrier.invalidate(bar)

        out = gl.convert_layout(acc_tmem.load(), layout)
        gl.store(out_ptr + offs_m * BLOCK + offs_n, out)
```
- **EN:** Prepares or updates state through `tmem_layout`, `acc_tmem`, `a_scale_tmem`, `b_scale_tmem`, `a_scale_reg_layout`, `b_scale_reg_layout`, `scale_offs_k`, `scale_offs_m`, and 3 more. Invokes `TensorMemoryLayout`, `allocate_tensor_memory`, `acc_tmem.store`, `gl.convert_layout`, `acc_tmem.get_reg_layout`, `a_scale_tmem.get_reg_layout`, and 15 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `tmem_layout`、`acc_tmem`、`a_scale_tmem`、`b_scale_tmem`、`a_scale_reg_layout`、`b_scale_reg_layout`、`scale_offs_k`、`scale_offs_m` 等另外 3 项 准备或更新状态。 调用 `TensorMemoryLayout`、`allocate_tensor_memory`、`acc_tmem.store`、`gl.convert_layout`、`acc_tmem.get_reg_layout`、`a_scale_tmem.get_reg_layout` 等另外 15 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1803-1808

```python

    rs = np.random.RandomState(0)
    pack_factor = 2 if elem_type == "e2m1" else 1
    packed_k = B // pack_factor
    a_bits = rs.randint(0 if elem_type == "e2m1" else 20, 256 if elem_type == "e2m1" else 40, size=(B, packed_k),
                        dtype=np.uint8)
```
- **EN:** Prepares or updates state through `rs`, `pack_factor`, `packed_k`, `a_bits`. Invokes `np.random.RandomState`, `rs.randint` to execute the test logic. Relevant themes: random-data generation.
- **CN:** 通过 `rs`、`pack_factor`、`packed_k`、`a_bits` 准备或更新状态。 调用 `np.random.RandomState`、`rs.randint` 执行测试逻辑。 相关主题：随机数据生成。

#### Lines 1809-1814

```python
    if elem_type == "e2m1":
        b_bits = rs.randint(0, 256, size=(B, packed_k), dtype=np.uint8)
        b_ref_bits = b_bits.T
    else:
        b_bits = rs.randint(20, 40, size=(packed_k, B), dtype=np.uint8)
        b_ref_bits = b_bits
```
- **EN:** Invokes `rs.randint` to execute the test logic. Branches on runtime or test conditions. Relevant themes: random-data generation.
- **CN:** 调用 `rs.randint` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：随机数据生成。

#### Lines 1815-1819

```python
    a_scale_bits = rs.randint(1, 4, size=(B, B // 32), dtype=np.int8)
    b_scale_bits = rs.randint(1, 4, size=(B, B // 32), dtype=np.int8)
    c_bits = rs.randint(-(2**31), 2**31 - 1, size=(B, B), dtype=np.int32)
    exp_bits = _mm_scaled_payload_u32(a_bits, b_ref_bits, a_scale_bits.view(np.uint8), b_scale_bits.view(np.uint8),
                                      c_bits, a_pack=pack_factor, b_pack=pack_factor, elem_type=elem_type)
```
- **EN:** Prepares or updates state through `a_scale_bits`, `b_scale_bits`, `c_bits`, `exp_bits`. Invokes `rs.randint`, `_mm_scaled_payload_u32`, `a_scale_bits.view`, `b_scale_bits.view` to execute the test logic. Relevant themes: random-data generation.
- **CN:** 通过 `a_scale_bits`、`b_scale_bits`、`c_bits`、`exp_bits` 准备或更新状态。 调用 `rs.randint`、`_mm_scaled_payload_u32`、`a_scale_bits.view`、`b_scale_bits.view` 执行测试逻辑。 相关主题：随机数据生成。

#### Lines 1820-1827

```python

    if elem_type == "e2m1":
        a = torch.tensor(a_bits, device="cuda", dtype=torch.uint8)
        b = torch.tensor(b_bits, device="cuda", dtype=torch.uint8)
    else:
        torch_dtype = torch.float8_e4m3fn if elem_type == "e4m3" else torch.float8_e5m2
        a = torch.tensor(a_bits, device="cuda", dtype=torch.uint8).view(torch_dtype)
        b = torch.tensor(b_bits, device="cuda", dtype=torch.uint8).view(torch_dtype)
```
- **EN:** Invokes `torch.tensor` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.tensor` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

#### Lines 1828-1838

```python
    a_scale = torch.tensor(a_scale_bits, device="cuda", dtype=torch.int8)
    b_scale = torch.tensor(b_scale_bits, device="cuda", dtype=torch.int8)
    c = torch.tensor(c_bits, device="cuda", dtype=torch.int32)
    out = torch.empty((B, B), device="cuda", dtype=torch.int32)

    cw = triton.TensorWrapper(c, dtype=torch.float32)
    outw = triton.TensorWrapper(out, dtype=torch.float32)

    kernel[(1, )](a, b, a_scale, b_scale, cw, outw, TYPE=elem_type)

    _assert_payload_equal(out, exp_bits)
```
- **EN:** Prepares or updates state through `a_scale`, `b_scale`, `c`, `out`, `cw`, `outw`. Invokes `torch.tensor`, `torch.empty`, `triton.TensorWrapper`, `_assert_payload_equal` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `a_scale`、`b_scale`、`c`、`out`、`cw`、`outw` 准备或更新状态。 调用 `torch.tensor`、`torch.empty`、`triton.TensorWrapper`、`_assert_payload_equal` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 1839-1842

```python


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_tmem_index_subslice(device, fresh_knobs):
```
- **EN:** Defines the test function `test_tmem_index_subslice`. Decorators: `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`. Parameters: `device`, `fresh_knobs`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `_require_cuda_backend`, `gl.constexpr`, `np.random.RandomState`, `rs.randint`, `x_bits.copy`, and 19 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_tmem_index_subslice`。 装饰器：`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`。 参数：`device`、`fresh_knobs`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`_require_cuda_backend`、`gl.constexpr`、`np.random.RandomState`、`rs.randint`、`x_bits.copy` 等另外 19 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 1843-1849

```python
    _require_cuda_backend(device)

    B = 64
    BLOCK = gl.constexpr(B)
    SLICE_N = gl.constexpr(32)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"
```
- **EN:** Prepares or updates state through `B`, `BLOCK`, `SLICE_N`, `fresh_knobs`. Invokes `_require_cuda_backend`, `gl.constexpr` to execute the test logic.
- **CN:** 通过 `B`、`BLOCK`、`SLICE_N`、`fresh_knobs` 准备或更新状态。 调用 `_require_cuda_backend`、`gl.constexpr` 执行测试逻辑。

#### Lines 1850-1852

```python

    @gluon.jit
    def kernel(x_ptr, out_ptr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `x_ptr`, `out_ptr`. Key calls include `gl.BlockedLayout`, `gl.load`, `TensorMemoryLayout`, `allocate_tensor_memory`, `tmem.index`, `view.slice`, and 8 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`x_ptr`、`out_ptr`。 关键调用包括 `gl.BlockedLayout`、`gl.load`、`TensorMemoryLayout`、`allocate_tensor_memory`、`tmem.index`、`view.slice` 等另外 8 项。 该作用域涉及布局变换推理。

##### Lines 1853-1870

```python
        layout: gl.constexpr = gl.BlockedLayout([1, 1], [32, 1], [gl.num_warps(), 1], [1, 0])
        offs_m = gl.arange(0, BLOCK, layout=gl.SliceLayout(1, layout))[:, None]
        offs_n = gl.arange(0, SLICE_N, layout=gl.SliceLayout(0, layout))[None, :]
        offs = offs_m * SLICE_N + offs_n

        x = gl.load(x_ptr + offs)

        tmem_layout: gl.constexpr = TensorMemoryLayout((BLOCK, BLOCK), col_stride=1)
        tmem = allocate_tensor_memory(gl.float32, [2, BLOCK, BLOCK], layout=tmem_layout)
        view = tmem.index(1)
        sub = view.slice(0, SLICE_N)

        sub_reg_layout: gl.constexpr = sub.get_reg_layout()
        x_reg = gl.convert_layout(x, sub_reg_layout)
        sub.store(x_reg)
        out = sub.load()
        out = gl.convert_layout(out, layout)
        gl.store(out_ptr + offs, out)
```
- **EN:** Prepares or updates state through `layout`, `offs_m`, `offs_n`, `offs`, `x`, `tmem_layout`, `tmem`, `view`, and 4 more. Invokes `gl.BlockedLayout`, `gl.num_warps`, `gl.arange`, `gl.SliceLayout`, `gl.load`, `TensorMemoryLayout`, and 8 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`offs_m`、`offs_n`、`offs`、`x`、`tmem_layout`、`tmem`、`view` 等另外 4 项 准备或更新状态。 调用 `gl.BlockedLayout`、`gl.num_warps`、`gl.arange`、`gl.SliceLayout`、`gl.load`、`TensorMemoryLayout` 等另外 8 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1871-1884

```python

    rs = np.random.RandomState(0)
    x_bits = rs.randint(-(2**31), 2**31 - 1, size=(B, 32), dtype=np.int32)
    exp_bits = x_bits.copy()

    x = torch.tensor(x_bits, device="cuda", dtype=torch.int32)
    out = torch.empty((B, 32), device="cuda", dtype=torch.int32)

    xw = triton.TensorWrapper(x, dtype=torch.float32)
    outw = triton.TensorWrapper(out, dtype=torch.float32)

    kernel[(1, )](xw, outw)

    _assert_payload_equal(out, exp_bits)
```
- **EN:** Prepares or updates state through `rs`, `x_bits`, `exp_bits`, `x`, `out`, `xw`, `outw`. Invokes `np.random.RandomState`, `rs.randint`, `x_bits.copy`, `torch.tensor`, `torch.empty`, `triton.TensorWrapper`, and 1 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `rs`、`x_bits`、`exp_bits`、`x`、`out`、`xw`、`outw` 准备或更新状态。 调用 `np.random.RandomState`、`rs.randint`、`x_bits.copy`、`torch.tensor`、`torch.empty`、`triton.TensorWrapper` 等另外 1 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 1885-1888

```python


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_tmem_copy_scales_in_warp_specialize_partition(device, fresh_knobs):
```
- **EN:** Defines the test function `test_tmem_copy_scales_in_warp_specialize_partition`. Decorators: `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`. Parameters: `device`, `fresh_knobs`. Nested definitions in this scope: `copy_partition`, `default_partition`, `kernel`. Key calls include `pytest.mark.skipif`, `_require_cuda_backend`, `gl.constexpr`, `torch.randint`, `torch.empty`, `torch.testing.assert_close`, and 20 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_tmem_copy_scales_in_warp_specialize_partition`。 装饰器：`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`。 参数：`device`、`fresh_knobs`。 该作用域中的嵌套定义：`copy_partition`、`default_partition`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`_require_cuda_backend`、`gl.constexpr`、`torch.randint`、`torch.empty`、`torch.testing.assert_close` 等另外 20 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 1889-1896

```python
    _require_cuda_backend(device)

    smem_h = 64
    smem_w = 16
    SMEM_H = gl.constexpr(smem_h)
    SMEM_W = gl.constexpr(smem_w)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"
```
- **EN:** Prepares or updates state through `smem_h`, `smem_w`, `SMEM_H`, `SMEM_W`, `fresh_knobs`. Invokes `_require_cuda_backend`, `gl.constexpr` to execute the test logic.
- **CN:** 通过 `smem_h`、`smem_w`、`SMEM_H`、`SMEM_W`、`fresh_knobs` 准备或更新状态。 调用 `_require_cuda_backend`、`gl.constexpr` 执行测试逻辑。

#### Lines 1897-1899

```python

    @gluon.jit
    def copy_partition(smem, tmem, bar):
```
- **EN:** Defines the helper function `copy_partition`. Decorators: `gluon.jit`. Parameters: `smem`, `tmem`, `bar`. Key calls include `tcgen05_copy`, `tcgen05_commit`.
- **CN:** 定义辅助函数 `copy_partition`。 装饰器：`gluon.jit`。 参数：`smem`、`tmem`、`bar`。 关键调用包括 `tcgen05_copy`、`tcgen05_commit`。

##### Lines 1900-1901

```python
        tcgen05_copy(smem, tmem)
        tcgen05_commit(bar)
```
- **EN:** Invokes `tcgen05_copy`, `tcgen05_commit` to execute the test logic.
- **CN:** 调用 `tcgen05_copy`、`tcgen05_commit` 执行测试逻辑。

#### Lines 1902-1904

```python

    @gluon.jit
    def default_partition():
```
- **EN:** Defines the helper function `default_partition`. Decorators: `gluon.jit`.
- **CN:** 定义辅助函数 `default_partition`。 装饰器：`gluon.jit`。

##### Lines 1905-1905

```python
        pass
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

#### Lines 1906-1908

```python

    @gluon.jit
    def kernel(in_ptr, out_ptr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `in_ptr`, `out_ptr`. Key calls include `gl.BlockedLayout`, `gl.load`, `gl.SharedLinearLayout`, `gl.allocate_shared_memory`, `smem.store`, `TensorMemoryScalesLayout`, and 11 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`in_ptr`、`out_ptr`。 关键调用包括 `gl.BlockedLayout`、`gl.load`、`gl.SharedLinearLayout`、`gl.allocate_shared_memory`、`smem.store`、`TensorMemoryScalesLayout` 等另外 11 项。 该作用域涉及布局变换推理。

##### Lines 1909-1944

```python
        blocked: gl.constexpr = gl.BlockedLayout([1, 4], [32, 1], [gl.num_warps(), 1], [1, 0])
        in_ptrs = (in_ptr + gl.arange(0, SMEM_H)[:, None] * SMEM_W + gl.arange(0, SMEM_W)[None, :])
        value = gl.load(gl.set_auto_layout(in_ptrs, blocked))

        smem_layout: gl.constexpr = gl.SharedLinearLayout(offset_bases=[
            [0, 1],
            [0, 2],
            [32, 0],
            [0, 4],
            [1, 0],
            [2, 0],
            [4, 0],
            [8, 0],
            [16, 0],
            [0, 8],
        ])
        smem = gl.allocate_shared_memory(gl.int8, (SMEM_H, SMEM_W), layout=smem_layout)
        smem.store(value)

        tmem_layout: gl.constexpr = TensorMemoryScalesLayout()
        tmem = allocate_tensor_memory(gl.int8, (SMEM_H, SMEM_W), layout=tmem_layout)
        bar = gl.allocate_shared_memory(gl.int64, [1], gl.constexpr(mbarrier.MBarrierLayout()))
        mbarrier.init(bar, count=1)

        gl.warp_specialize(
            [
                (default_partition, ()),
                (copy_partition, (smem, tmem, bar)),
            ],
            [1],
            [32],
        )

        mbarrier.wait(bar, phase=0)
        mbarrier.invalidate(bar)
        gl.store(out_ptr, 1)
```
- **EN:** Prepares or updates state through `blocked`, `in_ptrs`, `value`, `smem_layout`, `smem`, `tmem_layout`, `tmem`, `bar`. Invokes `gl.BlockedLayout`, `gl.num_warps`, `gl.arange`, `gl.load`, `gl.set_auto_layout`, `gl.SharedLinearLayout`, and 11 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `blocked`、`in_ptrs`、`value`、`smem_layout`、`smem`、`tmem_layout`、`tmem`、`bar` 准备或更新状态。 调用 `gl.BlockedLayout`、`gl.num_warps`、`gl.arange`、`gl.load`、`gl.set_auto_layout`、`gl.SharedLinearLayout` 等另外 11 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1945-1949

```python

    x = torch.randint(size=(smem_h, smem_w), low=-100, high=100, dtype=torch.int8, device=device)
    out = torch.empty((), device=device, dtype=torch.int32)
    kernel[(1, )](x, out, num_warps=4)
    torch.testing.assert_close(out, torch.ones_like(out))
```
- **EN:** Prepares or updates state through `x`, `out`. Invokes `torch.randint`, `torch.empty`, `torch.testing.assert_close`, `torch.ones_like` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `x`、`out` 准备或更新状态。 调用 `torch.randint`、`torch.empty`、`torch.testing.assert_close`、`torch.ones_like` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 1950-1953

```python


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_tmem_store_in_warp_specialize_partition_visible_to_parent(device, fresh_knobs):
```
- **EN:** Defines the test function `test_tmem_store_in_warp_specialize_partition_visible_to_parent`. Decorators: `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`. Parameters: `device`, `fresh_knobs`. Nested definitions in this scope: `store_one_partition`, `default_partition`, `kernel`. Key calls include `pytest.mark.skipif`, `_require_cuda_backend`, `gl.constexpr`, `torch.empty`, `torch.testing.assert_close`, `tmem.get_reg_layout`, and 20 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 定义测试函数 `test_tmem_store_in_warp_specialize_partition_visible_to_parent`。 装饰器：`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`。 参数：`device`、`fresh_knobs`。 该作用域中的嵌套定义：`store_one_partition`、`default_partition`、`kernel`。 关键调用包括 `pytest.mark.skipif`、`_require_cuda_backend`、`gl.constexpr`、`torch.empty`、`torch.testing.assert_close`、`tmem.get_reg_layout` 等另外 20 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理。

#### Lines 1954-1959

```python
    _require_cuda_backend(device)

    B = 64
    BLOCK = gl.constexpr(B)

    fresh_knobs.compilation.instrumentation_mode = "fpsan"
```
- **EN:** Prepares or updates state through `B`, `BLOCK`, `fresh_knobs`. Invokes `_require_cuda_backend`, `gl.constexpr` to execute the test logic.
- **CN:** 通过 `B`、`BLOCK`、`fresh_knobs` 准备或更新状态。 调用 `_require_cuda_backend`、`gl.constexpr` 执行测试逻辑。

#### Lines 1960-1962

```python

    @gluon.jit
    def store_one_partition(tmem, bar):
```
- **EN:** Defines the helper function `store_one_partition`. Decorators: `gluon.jit`. Parameters: `tmem`, `bar`. Key calls include `tmem.get_reg_layout`, `gl.full`, `tmem.store`, `mbarrier.arrive`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `store_one_partition`。 装饰器：`gluon.jit`。 参数：`tmem`、`bar`。 关键调用包括 `tmem.get_reg_layout`、`gl.full`、`tmem.store`、`mbarrier.arrive`。 该作用域涉及布局变换推理。

##### Lines 1963-1966

```python
        reg_layout: gl.constexpr = tmem.get_reg_layout()
        one = gl.full((BLOCK, BLOCK), 1.0, gl.float32, reg_layout)
        tmem.store(one)
        mbarrier.arrive(bar, count=1)
```
- **EN:** Prepares or updates state through `reg_layout`, `one`. Invokes `tmem.get_reg_layout`, `gl.full`, `tmem.store`, `mbarrier.arrive` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `reg_layout`、`one` 准备或更新状态。 调用 `tmem.get_reg_layout`、`gl.full`、`tmem.store`、`mbarrier.arrive` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1967-1969

```python

    @gluon.jit
    def default_partition():
```
- **EN:** Defines the helper function `default_partition`. Decorators: `gluon.jit`.
- **CN:** 定义辅助函数 `default_partition`。 装饰器：`gluon.jit`。

##### Lines 1970-1970

```python
        pass
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

#### Lines 1971-1973

```python

    @gluon.jit
    def kernel(out_ptr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `out_ptr`. Key calls include `gl.BlockedLayout`, `TensorMemoryLayout`, `allocate_tensor_memory`, `tmem.get_reg_layout`, `gl.full`, `tmem.store`, and 13 more. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`out_ptr`。 关键调用包括 `gl.BlockedLayout`、`TensorMemoryLayout`、`allocate_tensor_memory`、`tmem.get_reg_layout`、`gl.full`、`tmem.store` 等另外 13 项。 该作用域涉及布局变换推理。

##### Lines 1974-1996

```python
        layout: gl.constexpr = gl.BlockedLayout([1, 1], [32, 1], [gl.num_warps(), 1], [1, 0])
        offs_m = gl.arange(0, BLOCK, layout=gl.SliceLayout(1, layout))[:, None]
        offs_n = gl.arange(0, BLOCK, layout=gl.SliceLayout(0, layout))[None, :]
        offs = offs_m * BLOCK + offs_n

        tmem_layout: gl.constexpr = TensorMemoryLayout((BLOCK, BLOCK), col_stride=1)
        tmem = allocate_tensor_memory(gl.float32, [BLOCK, BLOCK], layout=tmem_layout)
        reg_layout: gl.constexpr = tmem.get_reg_layout()
        zero = gl.full((BLOCK, BLOCK), 0.0, gl.float32, reg_layout)
        tmem.store(zero)

        bar = gl.allocate_shared_memory(gl.int64, [1], gl.constexpr(mbarrier.MBarrierLayout()))
        mbarrier.init(bar, count=1)
        gl.warp_specialize([
            (default_partition, ()),
            (store_one_partition, (tmem, bar)),
        ], [4], [32])
        mbarrier.wait(bar, phase=0, deps=[tmem])
        mbarrier.invalidate(bar)

        out = tmem.load()
        out = gl.convert_layout(out, layout)
        gl.store(out_ptr + offs, out)
```
- **EN:** Prepares or updates state through `layout`, `offs_m`, `offs_n`, `offs`, `tmem_layout`, `tmem`, `reg_layout`, `zero`, and 2 more. Invokes `gl.BlockedLayout`, `gl.num_warps`, `gl.arange`, `gl.SliceLayout`, `TensorMemoryLayout`, `allocate_tensor_memory`, and 13 more to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `layout`、`offs_m`、`offs_n`、`offs`、`tmem_layout`、`tmem`、`reg_layout`、`zero` 等另外 2 项 准备或更新状态。 调用 `gl.BlockedLayout`、`gl.num_warps`、`gl.arange`、`gl.SliceLayout`、`TensorMemoryLayout`、`allocate_tensor_memory` 等另外 13 项 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 1997-2001

```python

    out = torch.empty((B, B), device=device, dtype=torch.float32)
    kernel[(1, )](out, num_warps=4)

    torch.testing.assert_close(out, torch.ones_like(out), rtol=0, atol=0)
```
- **EN:** Prepares or updates state through `out`. Invokes `torch.empty`, `torch.testing.assert_close`, `torch.ones_like` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `out` 准备或更新状态。 调用 `torch.empty`、`torch.testing.assert_close`、`torch.ones_like` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 2002-2004

```python


def test_reduction(device, fresh_knobs):
```
- **EN:** Defines the test function `test_reduction`. Parameters: `device`, `fresh_knobs`. Nested definitions in this scope: `reduce_kernel`. Key calls include `_require_cuda_backend`, `torch.manual_seed`, `torch.randn`, `_payload_equal`, `tl.load`, `tl.sum`, and 5 more. This scope touches Triton compilation or JIT kernels, Triton language operations, PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义测试函数 `test_reduction`。 参数：`device`、`fresh_knobs`。 该作用域中的嵌套定义：`reduce_kernel`。 关键调用包括 `_require_cuda_backend`、`torch.manual_seed`、`torch.randn`、`_payload_equal`、`tl.load`、`tl.sum` 等另外 5 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、PyTorch 张量准备与校验、随机数据生成。

#### Lines 2005-2005

```python
    _require_cuda_backend(device)
```
- **EN:** Invokes `_require_cuda_backend` to execute the test logic.
- **CN:** 调用 `_require_cuda_backend` 执行测试逻辑。

#### Lines 2006-2010

```python

    @triton.jit
    def reduce_kernel(a_ptr, c_ptr, M: tl.constexpr, N: tl.constexpr, stride_ak: tl.constexpr, stride_am: tl.constexpr,
                      stride_an: tl.constexpr, ORDER: tl.constexpr):
```
- **EN:** Defines the helper function `reduce_kernel`. Decorators: `triton.jit`. Parameters: `a_ptr`, `c_ptr`, `M`, `N`, `stride_ak`, `stride_am`, `stride_an`, `ORDER`. Key calls include `tl.load`, `tl.sum`, `tl.store`, `tl.program_id`, `tl.arange`. This scope touches Triton compilation or JIT kernels, Triton language operations.
- **CN:** 定义辅助函数 `reduce_kernel`。 装饰器：`triton.jit`。 参数：`a_ptr`、`c_ptr`、`M`、`N`、`stride_ak`、`stride_am`、`stride_an`、`ORDER`。 关键调用包括 `tl.load`、`tl.sum`、`tl.store`、`tl.program_id`、`tl.arange`。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作。

##### Lines 2011-2017

```python
        a_ptr += tl.program_id(0).to(tl.int64) * stride_ak
        c_ptr += tl.program_id(0).to(tl.int64)
        a_ptrs = a_ptr + (tl.arange(0, M)[:, None] * stride_am + (tl.arange(0, N)[None, :]) * stride_an)
        a = tl.load(a_ptrs)
        r1 = tl.sum(a, axis=ORDER)
        r2 = tl.sum(r1, axis=0)
        tl.store(c_ptr, r2)
```
- **EN:** Prepares or updates state through `a_ptr`, `c_ptr`, `a_ptrs`, `a`, `r1`, `r2`. Invokes `tl.program_id`, `tl.arange`, `tl.load`, `tl.sum`, `tl.store` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `a_ptr`、`c_ptr`、`a_ptrs`、`a`、`r1`、`r2` 准备或更新状态。 调用 `tl.program_id`、`tl.arange`、`tl.load`、`tl.sum`、`tl.store` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 2018-2028

```python

    # we run K parallel tests so as to make non-associativity much more
    # likely to manifest:
    K, M, N = 100, 128, 128
    torch.manual_seed(0)
    a = torch.randn((K, M, N), dtype=torch.float32, device="cuda")
    c1 = torch.empty((K, ), dtype=torch.float32).to('cuda')
    c2 = torch.empty((K, ), dtype=torch.float32).to('cuda')

    reduce_kernel[(K, )](a, c1, M, N, a.stride(0), a.stride(1), a.stride(2), ORDER=0)
    reduce_kernel[(K, )](a, c2, M, N, a.stride(0), a.stride(1), a.stride(2), ORDER=1)
```
- **EN:** Prepares or updates state through `K`, `M`, `N`, `a`, `c1`, `c2`. Invokes `torch.manual_seed`, `torch.randn`, `torch.empty`, `a.stride` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `K`、`M`、`N`、`a`、`c1`、`c2` 准备或更新状态。 调用 `torch.manual_seed`、`torch.randn`、`torch.empty`、`a.stride` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 2029-2029

```python
    assert not _payload_equal(c1, c2)
```
- **EN:** Invokes `_payload_equal` to execute the test logic. Validates behavior with 1 assertion(s).
- **CN:** 调用 `_payload_equal` 执行测试逻辑。 通过 1 个断言验证行为。

#### Lines 2030-2034

```python

    fresh_knobs.compilation.instrumentation_mode = "fpsan"

    reduce_kernel[(K, )](a, c1, M, N, a.stride(0), a.stride(1), a.stride(2), ORDER=0)
    reduce_kernel[(K, )](a, c2, M, N, a.stride(0), a.stride(1), a.stride(2), ORDER=1)
```
- **EN:** Prepares or updates state through `fresh_knobs`. Invokes `a.stride` to execute the test logic.
- **CN:** 通过 `fresh_knobs` 准备或更新状态。 调用 `a.stride` 执行测试逻辑。

#### Lines 2035-2035

```python
    assert _payload_equal(c1, c2)
```
- **EN:** Invokes `_payload_equal` to execute the test logic. Validates behavior with 1 assertion(s).
- **CN:** 调用 `_payload_equal` 执行测试逻辑。 通过 1 个断言验证行为。

### Lines 2036-2038

```python


def test_reduction_matches_loop(device, fresh_knobs):
```
- **EN:** Defines the test function `test_reduction_matches_loop`. Parameters: `device`, `fresh_knobs`. Nested definitions in this scope: `reduce_sum_kernel`, `loop_sum_kernel`. Key calls include `_require_cuda_backend`, `torch.tensor`, `pattern.repeat`, `torch.empty`, `_assert_payload_equal`, `tl.load`, and 7 more. This scope touches Triton compilation or JIT kernels, Triton language operations, PyTorch tensor setup and checks.
- **CN:** 定义测试函数 `test_reduction_matches_loop`。 参数：`device`、`fresh_knobs`。 该作用域中的嵌套定义：`reduce_sum_kernel`、`loop_sum_kernel`。 关键调用包括 `_require_cuda_backend`、`torch.tensor`、`pattern.repeat`、`torch.empty`、`_assert_payload_equal`、`tl.load` 等另外 7 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、PyTorch 张量准备与校验。

#### Lines 2039-2039

```python
    _require_cuda_backend(device)
```
- **EN:** Invokes `_require_cuda_backend` to execute the test logic.
- **CN:** 调用 `_require_cuda_backend` 执行测试逻辑。

#### Lines 2040-2042

```python

    @triton.jit
    def reduce_sum_kernel(x_ptr, out_ptr, N: tl.constexpr):
```
- **EN:** Defines the helper function `reduce_sum_kernel`. Decorators: `triton.jit`. Parameters: `x_ptr`, `out_ptr`, `N`. Key calls include `tl.load`, `tl.store`, `tl.sum`, `tl.arange`. This scope touches Triton compilation or JIT kernels, Triton language operations.
- **CN:** 定义辅助函数 `reduce_sum_kernel`。 装饰器：`triton.jit`。 参数：`x_ptr`、`out_ptr`、`N`。 关键调用包括 `tl.load`、`tl.store`、`tl.sum`、`tl.arange`。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作。

##### Lines 2043-2044

```python
        x = tl.load(x_ptr + tl.arange(0, N))
        tl.store(out_ptr, tl.sum(x, axis=0))
```
- **EN:** Prepares or updates state through `x`. Invokes `tl.load`, `tl.arange`, `tl.store`, `tl.sum` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `x` 准备或更新状态。 调用 `tl.load`、`tl.arange`、`tl.store`、`tl.sum` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 2045-2047

```python

    @triton.jit
    def loop_sum_kernel(x_ptr, out_ptr, N: tl.constexpr):
```
- **EN:** Defines the helper function `loop_sum_kernel`. Decorators: `triton.jit`. Parameters: `x_ptr`, `out_ptr`, `N`. Key calls include `tl.full`, `tl.static_range`, `tl.store`, `tl.load`. This scope touches Triton compilation or JIT kernels, Triton language operations.
- **CN:** 定义辅助函数 `loop_sum_kernel`。 装饰器：`triton.jit`。 参数：`x_ptr`、`out_ptr`、`N`。 关键调用包括 `tl.full`、`tl.static_range`、`tl.store`、`tl.load`。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作。

##### Lines 2048-2048

```python
        acc = tl.full([], 0.0, tl.float32)
```
- **EN:** Prepares or updates state through `acc`. Invokes `tl.full` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `acc` 准备或更新状态。 调用 `tl.full` 执行测试逻辑。 相关主题：Triton language 操作。

##### Lines 2049-2050

```python
        for i in tl.static_range(0, N):
            acc += tl.load(x_ptr + i)
```
- **EN:** Invokes `tl.static_range`, `tl.load` to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations.
- **CN:** 调用 `tl.static_range`、`tl.load` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作。

##### Lines 2051-2051

```python
        tl.store(out_ptr, acc)
```
- **EN:** Invokes `tl.store` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 调用 `tl.store` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 2052-2060

```python

    N = 256
    pattern = torch.tensor([1e20, 1.0, -1e20, 1.0], dtype=torch.float32, device="cuda")
    x = pattern.repeat(N // pattern.numel())
    reduce_out = torch.empty((1, ), dtype=torch.float32, device="cuda")
    loop_out = torch.empty((1, ), dtype=torch.float32, device="cuda")

    reduce_sum_kernel[(1, )](x, reduce_out, N=N)
    loop_sum_kernel[(1, )](x, loop_out, N=N)
```
- **EN:** Prepares or updates state through `N`, `pattern`, `x`, `reduce_out`, `loop_out`. Invokes `torch.tensor`, `pattern.repeat`, `pattern.numel`, `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `N`、`pattern`、`x`、`reduce_out`、`loop_out` 准备或更新状态。 调用 `torch.tensor`、`pattern.repeat`、`pattern.numel`、`torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 2061-2061

```python
    assert not _payload_equal(reduce_out, loop_out)
```
- **EN:** Invokes `_payload_equal` to execute the test logic. Validates behavior with 1 assertion(s).
- **CN:** 调用 `_payload_equal` 执行测试逻辑。 通过 1 个断言验证行为。

#### Lines 2062-2067

```python

    fresh_knobs.compilation.instrumentation_mode = "fpsan"

    reduce_sum_kernel[(1, )](x, reduce_out, N=N)
    loop_sum_kernel[(1, )](x, loop_out, N=N)
    _assert_payload_equal(reduce_out, loop_out)
```
- **EN:** Prepares or updates state through `fresh_knobs`. Invokes `_assert_payload_equal` to execute the test logic.
- **CN:** 通过 `fresh_knobs` 准备或更新状态。 调用 `_assert_payload_equal` 执行测试逻辑。

### Lines 2068-2071

```python


@pytest.mark.skipif(not (is_hip_cdna3() or is_hip_cdna4()), reason="Requires CDNA3 or CDNA4")
def test_mfma_dot(device, fresh_knobs):
```
- **EN:** Defines the test function `test_mfma_dot`. Decorators: `pytest.mark.skipif(not (is_hip_cdna3() or is_hip_cdna4()), reason='Requires CDNA3 or CDNA4')`. Parameters: `device`, `fresh_knobs`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `_require_cuda_backend`, `gl.BlockedLayout`, `gl.amd.AMDMFMALayout`, `np.random.RandomState`, `rs.randint`, and 14 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_mfma_dot`。 装饰器：`pytest.mark.skipif(not (is_hip_cdna3() or is_hip_cdna4()), reason='Requires CDNA3 or CDNA4')`。 参数：`device`、`fresh_knobs`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`_require_cuda_backend`、`gl.BlockedLayout`、`gl.amd.AMDMFMALayout`、`np.random.RandomState`、`rs.randint` 等另外 14 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 2072-2084

```python
    _require_cuda_backend(device)

    M, N, K = 16, 16, 32

    fresh_knobs.compilation.instrumentation_mode = "fpsan"

    cdna_version = 3 if is_hip_cdna3() else 4
    nonkdim = 32
    kdim = 8 if cdna_version == 3 else 16
    k_width_val = 4 if cdna_version == 3 else 8

    blocked = gl.BlockedLayout([4, 4], [4, 16], [4, 1], [1, 0])
    mfma_layout = gl.amd.AMDMFMALayout(cdna_version, [nonkdim, nonkdim, kdim], True, [4, 1])
```
- **EN:** Prepares or updates state through `M`, `N`, `K`, `fresh_knobs`, `cdna_version`, `nonkdim`, `kdim`, `k_width_val`, and 2 more. Invokes `_require_cuda_backend`, `is_hip_cdna3`, `gl.BlockedLayout`, `gl.amd.AMDMFMALayout` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 通过 `M`、`N`、`K`、`fresh_knobs`、`cdna_version`、`nonkdim`、`kdim`、`k_width_val` 等另外 2 项 准备或更新状态。 调用 `_require_cuda_backend`、`is_hip_cdna3`、`gl.BlockedLayout`、`gl.amd.AMDMFMALayout` 执行测试逻辑。 相关主题：布局变换推理。

#### Lines 2085-2088

```python

    @gluon.jit
    def kernel(a_ptr, b_ptr, c_ptr, out_ptr, BLOCK_M: gl.constexpr, BLOCK_N: gl.constexpr, BLOCK_K: gl.constexpr,
               blocked: gl.constexpr, k_width: gl.constexpr, mfma_layout: gl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_ptr`, `b_ptr`, `c_ptr`, `out_ptr`, `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, `blocked`, and 2 more. Key calls include `gl.DotOperandLayout`, `gl.arange`, `gl.load`, `gl.convert_layout`, `gl.amd.cdna3.mfma`, `gl.store`, and 1 more. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_ptr`、`b_ptr`、`c_ptr`、`out_ptr`、`BLOCK_M`、`BLOCK_N`、`BLOCK_K`、`blocked` 等另外 2 项。 关键调用包括 `gl.DotOperandLayout`、`gl.arange`、`gl.load`、`gl.convert_layout`、`gl.amd.cdna3.mfma`、`gl.store` 等另外 1 项。 该作用域涉及布局变换推理、随机数据生成。

##### Lines 2089-2107

```python
        dot_a_layout: gl.constexpr = gl.DotOperandLayout(operand_index=0, parent=mfma_layout, k_width=k_width)
        dot_b_layout: gl.constexpr = gl.DotOperandLayout(operand_index=1, parent=mfma_layout, k_width=k_width)

        offs_am = gl.arange(0, BLOCK_M, layout=gl.SliceLayout(1, blocked))
        offs_bn = gl.arange(0, BLOCK_N, layout=gl.SliceLayout(0, blocked))
        offs_ak = gl.arange(0, BLOCK_K, layout=gl.SliceLayout(0, blocked))
        offs_bk = gl.arange(0, BLOCK_K, layout=gl.SliceLayout(1, blocked))

        a = gl.load(a_ptr + offs_am[:, None] * BLOCK_K + offs_ak[None, :])
        b = gl.load(b_ptr + offs_bk[:, None] * BLOCK_N + offs_bn[None, :])
        c = gl.load(c_ptr + offs_am[:, None] * BLOCK_N + offs_bn[None, :])

        a1 = gl.convert_layout(a, layout=dot_a_layout)
        b1 = gl.convert_layout(b, layout=dot_b_layout)
        c_acc = gl.convert_layout(c, layout=mfma_layout)

        result = gl.amd.cdna3.mfma(a1, b1, c_acc)
        result = gl.convert_layout(result, layout=blocked)
        gl.store(out_ptr + offs_am[:, None] * BLOCK_N + offs_bn[None, :], result)
```
- **EN:** Prepares or updates state through `dot_a_layout`, `dot_b_layout`, `offs_am`, `offs_bn`, `offs_ak`, `offs_bk`, `a`, `b`, and 5 more. Invokes `gl.DotOperandLayout`, `gl.arange`, `gl.SliceLayout`, `gl.load`, `gl.convert_layout`, `gl.amd.cdna3.mfma`, and 1 more to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `dot_a_layout`、`dot_b_layout`、`offs_am`、`offs_bn`、`offs_ak`、`offs_bk`、`a`、`b` 等另外 5 项 准备或更新状态。 调用 `gl.DotOperandLayout`、`gl.arange`、`gl.SliceLayout`、`gl.load`、`gl.convert_layout`、`gl.amd.cdna3.mfma` 等另外 1 项 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

#### Lines 2108-2128

```python

    rs = np.random.RandomState(0)
    a_bits = rs.randint(-(2**31), 2**31 - 1, size=(M, K), dtype=np.int32)
    b_bits = rs.randint(-(2**31), 2**31 - 1, size=(K, N), dtype=np.int32)
    c_bits = rs.randint(-(2**31), 2**31 - 1, size=(M, N), dtype=np.int32)
    exp_bits = _mm_payload_u32(a_bits, b_bits, c_bits)

    a = torch.tensor(a_bits, device="cuda", dtype=torch.int32)
    b = torch.tensor(b_bits, device="cuda", dtype=torch.int32)
    c = torch.tensor(c_bits, device="cuda", dtype=torch.int32)
    out = torch.empty((M, N), device="cuda", dtype=torch.int32)

    aw = triton.TensorWrapper(a, dtype=torch.float32)
    bw = triton.TensorWrapper(b, dtype=torch.float32)
    cw = triton.TensorWrapper(c, dtype=torch.float32)
    outw = triton.TensorWrapper(out, dtype=torch.float32)

    kernel[(1, )](aw, bw, cw, outw, BLOCK_M=M, BLOCK_N=N, BLOCK_K=K, blocked=blocked, k_width=k_width_val,
                  mfma_layout=mfma_layout)

    _assert_payload_equal(out, exp_bits)
```
- **EN:** Prepares or updates state through `rs`, `a_bits`, `b_bits`, `c_bits`, `exp_bits`, `a`, `b`, `c`, and 5 more. Invokes `np.random.RandomState`, `rs.randint`, `_mm_payload_u32`, `torch.tensor`, `torch.empty`, `triton.TensorWrapper`, and 1 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 通过 `rs`、`a_bits`、`b_bits`、`c_bits`、`exp_bits`、`a`、`b`、`c` 等另外 5 项 准备或更新状态。 调用 `np.random.RandomState`、`rs.randint`、`_mm_payload_u32`、`torch.tensor`、`torch.empty`、`triton.TensorWrapper` 等另外 1 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、布局变换推理、随机数据生成。

### Lines 2129-2132

```python


@pytest.mark.skipif(not is_hip_gfx1250(), reason="Requires gfx1250")
def test_wmma_dot(device, fresh_knobs):
```
- **EN:** Defines the test function `test_wmma_dot`. Decorators: `pytest.mark.skipif(not is_hip_gfx1250(), reason='Requires gfx1250')`. Parameters: `device`, `fresh_knobs`. Nested definitions in this scope: `kernel`. Key calls include `pytest.mark.skipif`, `_require_cuda_backend`, `np.random.RandomState`, `rs.randint`, `_mm_payload_u32`, `torch.tensor`, and 13 more. This scope touches PyTorch tensor setup and checks, layout transformation reasoning, random-data generation.
- **CN:** 定义测试函数 `test_wmma_dot`。 装饰器：`pytest.mark.skipif(not is_hip_gfx1250(), reason='Requires gfx1250')`。 参数：`device`、`fresh_knobs`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `pytest.mark.skipif`、`_require_cuda_backend`、`np.random.RandomState`、`rs.randint`、`_mm_payload_u32`、`torch.tensor` 等另外 13 项。 该作用域涉及PyTorch 张量准备与校验、布局变换推理、随机数据生成。

#### Lines 2133-2136

```python
    _require_cuda_backend(device)

    B = 32
    fresh_knobs.compilation.instrumentation_mode = "fpsan"
```
- **EN:** Prepares or updates state through `B`, `fresh_knobs`. Invokes `_require_cuda_backend` to execute the test logic.
- **CN:** 通过 `B`、`fresh_knobs` 准备或更新状态。 调用 `_require_cuda_backend` 执行测试逻辑。

#### Lines 2137-2139

```python

    @gluon.jit
    def kernel(a_ptr, b_ptr, c_ptr, out_ptr, BLOCK: gl.constexpr, INSTR_SHAPE_K: gl.constexpr, K_WIDTH: gl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `gluon.jit`. Parameters: `a_ptr`, `b_ptr`, `c_ptr`, `out_ptr`, `BLOCK`, `INSTR_SHAPE_K`, `K_WIDTH`. Key calls include `gl.BlockedLayout`, `gl.amd.AMDWMMALayout`, `gl.load`, `gl.convert_layout`, `gl.amd.gfx1250.wmma`, `gl.SliceLayout`, and 3 more. This scope touches layout transformation reasoning, random-data generation.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`gluon.jit`。 参数：`a_ptr`、`b_ptr`、`c_ptr`、`out_ptr`、`BLOCK`、`INSTR_SHAPE_K`、`K_WIDTH`。 关键调用包括 `gl.BlockedLayout`、`gl.amd.AMDWMMALayout`、`gl.load`、`gl.convert_layout`、`gl.amd.gfx1250.wmma`、`gl.SliceLayout` 等另外 3 项。 该作用域涉及布局变换推理、随机数据生成。

##### Lines 2140-2160

```python
        blocked: gl.constexpr = gl.BlockedLayout([1, 8], [4, 8], [4, 1], [1, 0])
        wmma: gl.constexpr = gl.amd.AMDWMMALayout(3, True, [[0, 1], [1, 0]], [], [16, 16, INSTR_SHAPE_K])

        offs_m = gl.arange(0, BLOCK, layout=gl.SliceLayout(1, blocked))[:, None]
        offs_k = gl.arange(0, BLOCK, layout=gl.SliceLayout(0, blocked))[None, :]
        offs_bk = gl.arange(0, BLOCK, layout=gl.SliceLayout(1, blocked))[:, None]
        offs_n = gl.arange(0, BLOCK, layout=gl.SliceLayout(0, blocked))[None, :]

        a = gl.load(a_ptr + offs_m * BLOCK + offs_k)
        b = gl.load(b_ptr + offs_bk * BLOCK + offs_n)
        c = gl.load(c_ptr + offs_m * BLOCK + offs_n)
        c = gl.convert_layout(c, wmma)

        a = gl.convert_layout(a, gl.DotOperandLayout(0, wmma, K_WIDTH))
        b = gl.convert_layout(b, gl.DotOperandLayout(1, wmma, K_WIDTH))
        acc = gl.amd.gfx1250.wmma(a, b, c)

        out_layout: gl.constexpr = gl.SliceLayout(1, wmma)
        offs_cm = gl.arange(0, BLOCK, layout=out_layout)[:, None]
        offs_cn = gl.arange(0, BLOCK, layout=gl.SliceLayout(0, wmma))[None, :]
        gl.store(out_ptr + offs_cm * BLOCK + offs_cn, acc)
```
- **EN:** Prepares or updates state through `blocked`, `wmma`, `offs_m`, `offs_k`, `offs_bk`, `offs_n`, `a`, `b`, and 5 more. Invokes `gl.BlockedLayout`, `gl.amd.AMDWMMALayout`, `gl.arange`, `gl.SliceLayout`, `gl.load`, `gl.convert_layout`, and 3 more to execute the test logic. Relevant themes: layout transformation reasoning, random-data generation.
- **CN:** 通过 `blocked`、`wmma`、`offs_m`、`offs_k`、`offs_bk`、`offs_n`、`a`、`b` 等另外 5 项 准备或更新状态。 调用 `gl.BlockedLayout`、`gl.amd.AMDWMMALayout`、`gl.arange`、`gl.SliceLayout`、`gl.load`、`gl.convert_layout` 等另外 3 项 执行测试逻辑。 相关主题：布局变换推理、随机数据生成。

#### Lines 2161-2180

```python

    rs = np.random.RandomState(0)
    a_bits = rs.randint(-(2**31), 2**31 - 1, size=(B, B), dtype=np.int32)
    b_bits = rs.randint(-(2**31), 2**31 - 1, size=(B, B), dtype=np.int32)
    c_bits = rs.randint(-(2**31), 2**31 - 1, size=(B, B), dtype=np.int32)
    exp_bits = _mm_payload_u32(a_bits, b_bits, c_bits)

    a = torch.tensor(a_bits, device="cuda", dtype=torch.int32)
    b = torch.tensor(b_bits, device="cuda", dtype=torch.int32)
    c = torch.tensor(c_bits, device="cuda", dtype=torch.int32)
    out = torch.empty((B, B), device="cuda", dtype=torch.int32)

    aw = triton.TensorWrapper(a, dtype=torch.float32)
    bw = triton.TensorWrapper(b, dtype=torch.float32)
    cw = triton.TensorWrapper(c, dtype=torch.float32)
    outw = triton.TensorWrapper(out, dtype=torch.float32)

    kernel[(1, )](aw, bw, cw, outw, BLOCK=B, INSTR_SHAPE_K=4, K_WIDTH=2)

    _assert_payload_equal(out, exp_bits)
```
- **EN:** Prepares or updates state through `rs`, `a_bits`, `b_bits`, `c_bits`, `exp_bits`, `a`, `b`, `c`, and 5 more. Invokes `np.random.RandomState`, `rs.randint`, `_mm_payload_u32`, `torch.tensor`, `torch.empty`, `triton.TensorWrapper`, and 1 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `rs`、`a_bits`、`b_bits`、`c_bits`、`exp_bits`、`a`、`b`、`c` 等另外 5 项 准备或更新状态。 调用 `np.random.RandomState`、`rs.randint`、`_mm_payload_u32`、`torch.tensor`、`torch.empty`、`triton.TensorWrapper` 等另外 1 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

## Key Concepts / 关键概念

- **EN:** Top-level scopes such as `_hip_device_supports_fpsan`, `_require_cuda_backend`, `_as_u32`, `_u32_to_i32`, `_low_mask_u64`, `_inv_odd_u64`, `_mix_config`, `_xor_shift_right_u64`
  **CN:** 顶层作用域，例如 `_hip_device_supports_fpsan`、`_require_cuda_backend`、`_as_u32`、`_u32_to_i32`、`_low_mask_u64`、`_inv_odd_u64`、`_mix_config`、`_xor_shift_right_u64`
- **EN:** pytest parametrization
  **CN:** pytest 参数化
- **EN:** Triton compilation or JIT kernels
  **CN:** Triton 编译或 JIT kernel
- **EN:** Triton language operations
  **CN:** Triton language 操作
- **EN:** PyTorch tensor setup and checks
  **CN:** PyTorch 张量准备与校验
- **EN:** autotuning logic
  **CN:** 自动调优逻辑
- **EN:** runtime driver interaction
  **CN:** 运行时驱动交互
- **EN:** layout transformation reasoning
  **CN:** 布局变换推理

## Dependencies / 依赖关系

- **EN:** External or absolute imports include `itertools`, `numpy`, `pytest`, `torch`, `triton`, `triton.experimental`, `triton.experimental.gluon`, `triton._internal_testing`, `triton.experimental.gluon.language.nvidia`, `triton.experimental.gluon.language.nvidia.blackwell`.
  **CN:** 外部或绝对导入包括 `itertools`、`numpy`、`pytest`、`torch`、`triton`、`triton.experimental`、`triton.experimental.gluon`、`triton._internal_testing`、`triton.experimental.gluon.language.nvidia`、`triton.experimental.gluon.language.nvidia.blackwell`。
- **EN:** Execution centers on top-level definitions such as `_hip_device_supports_fpsan`, `_require_cuda_backend`, `_as_u32`, `_u32_to_i32`, `_low_mask_u64`, `_inv_odd_u64`, `_mix_config`, `_xor_shift_right_u64`, `_inverse_xor_shift_right_u64`, `_mix_float_bits_to_payload_u64`.
  **CN:** 执行逻辑主要围绕顶层定义展开，例如 `_hip_device_supports_fpsan`、`_require_cuda_backend`、`_as_u32`、`_u32_to_i32`、`_low_mask_u64`、`_inv_odd_u64`、`_mix_config`、`_xor_shift_right_u64`、`_inverse_xor_shift_right_u64`、`_mix_float_bits_to_payload_u64`。
- **EN:** Runtime behavior also depends on pytest collection, Python execution semantics, and the imported Triton/PyTorch utilities visible above.
  **CN:** 运行时行为还依赖 pytest 的收集机制、Python 执行语义，以及上文可见的 Triton/PyTorch 工具。
