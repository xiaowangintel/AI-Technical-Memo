# test_grouped_quant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/rocm/aiter/test_grouped_quant.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Grouped Quant behavior in the ROCm test area through focused pytest scenarios. It focuses on scenarios such as ROCm Aiter Group FP8 Quant Fake Implementation, ROCm Aiter Group FP8 Quant Torch Compile With Cudagraph, ROCm Aiter Group FP8 Quant Different Shapes. / 该文件在 ROCm 测试域中，通过有针对性的 pytest 场景验证 Grouped Quant 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-27)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# This is a test for the AITER group_fp8_quant op.
# It tests if the AITER op is
# 1. correctly defined the relationship between
#    implementation and fake function
# 2. can be used with torch.compile
# 3. can be used with CUDA graphs
# This file will be skipped if AITER is not installed
# and the platform is not ROCm.

import importlib.util

import pytest
import torch

# this import statement is needed to ensure the ops are registered
from vllm._aiter_ops import rocm_aiter_ops
from vllm.platforms import current_platform

# Check if aiter package is installed
aiter_available = importlib.util.find_spec("aiter") is not None

pytestmark = pytest.mark.skipif(
    not (current_platform.is_rocm() and aiter_available),
    reason="AITER ops are only available on ROCm with aiter package installed",
)
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `importlib.util`, `pytest`, `torch`, `vllm._aiter_ops`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Test: test_rocm_aiter_group_fp8_quant_fake_implementation (lines 30-46)
```python
def test_rocm_aiter_group_fp8_quant_fake_implementation():
    """Test that the fake implementation is correctly
    defined for torch.ops.vllm.rocm_aiter_group_fp8_quant."""
    # Create test tensors
    M = 128
    N = 4096
    group_size = 128

    input_tensor = torch.randn((M, N), dtype=torch.bfloat16, device="cuda")

    # Verify the op's fake implementation using torch.library.opcheck
    # This checks that the fake function returns tensors with correct shapes and dtypes
    torch.library.opcheck(
        torch.ops.vllm.rocm_aiter_group_fp8_quant,
        (input_tensor, group_size),
        test_utils=("test_faketensor",),
    )
```
**EN:** Test that the fake implementation is correctly defined for torch.ops.vllm.rocm_aiter_group_fp8_quant. The body exercises logic via `torch.randn`, `torch.library.opcheck` before asserting the expected outcome.
**CN:** 该测试用例验证 ROCm Aiter Group FP8 Quant Fake Implementation 在特定场景下的行为。 函数体会先通过 `torch.randn`, `torch.library.opcheck` 驱动目标逻辑，再断言预期结果。

### Test: test_rocm_aiter_group_fp8_quant_torch_compile_with_cudagraph (lines 49-109)
```python
def test_rocm_aiter_group_fp8_quant_torch_compile_with_cudagraph():
    """Test that rocm_aiter_ops.group_fp8_quant
    with group size 128 can be used with
    torch.compile in cudagraph mode."""
    # Create test tensors
    M = 128
    N = 4096
    group_size = 128

    input_tensor = torch.randn((M, N), dtype=torch.bfloat16, device="cuda")

    # Define a function that uses the op
    def group_fp8_quant_fn(x):
        return rocm_aiter_ops.group_fp8_quant(x, group_size)

    # Compile with cudagraph mode
    compiled_fn = torch.compile(
        group_fp8_quant_fn,
        fullgraph=True,
# ... omitted for brevity ...

    # Test with different input (reusing compiled graph)
    input_tensor_2 = torch.randn((M, N), dtype=torch.bfloat16, device="cuda")
    x_fp8_eager_2, scales_eager_2 = group_fp8_quant_fn(input_tensor_2)
    x_fp8_compiled_2, scales_compiled_2 = compiled_fn(input_tensor_2)

    # Verify second run also produces correct results
    assert torch.allclose(
        x_fp8_compiled_2.to(torch.float32),
        x_fp8_eager_2.to(torch.float32),
        rtol=1e-2,
        atol=1e-2,
    )
    assert torch.allclose(scales_compiled_2, scales_eager_2, rtol=1e-3, atol=1e-3)
```
**EN:** Test that rocm_aiter_ops.group_fp8_quant with group size 128 can be used with torch.compile in cudagraph mode. The body exercises logic via `torch.randn`, `torch.compile`, `group_fp8_quant_fn` before asserting the expected outcome.
**CN:** 该测试用例验证 ROCm Aiter Group FP8 Quant Torch Compile With Cudagraph 在特定场景下的行为。 函数体会先通过 `torch.randn`, `torch.compile`, `group_fp8_quant_fn` 驱动目标逻辑，再断言预期结果。

### Test: test_rocm_aiter_group_fp8_quant_different_shapes (lines 112-137)
```python
def test_rocm_aiter_group_fp8_quant_different_shapes():
    """Test rocm_aiter_ops.group_fp8_quant with different input shapes."""
    group_size = 128

    test_shapes = [
        (64, 2048),
        (256, 8192),
        (32, 1024),
        (512, 4096),
    ]

    for M, N in test_shapes:
        input_tensor = torch.randn((M, N), dtype=torch.bfloat16, device="cuda")

        x_fp8, scales = rocm_aiter_ops.group_fp8_quant(input_tensor, group_size)

        # Verify shapes
        assert x_fp8.shape == (M, N)
        expected_scale_cols = (N + group_size - 1) // group_size
        assert scales.shape == (M, expected_scale_cols)

        # Verify dtypes
        from aiter import dtypes

        assert x_fp8.dtype == dtypes.fp8
        assert scales.dtype == torch.float32
```
**EN:** Test rocm_aiter_ops.group_fp8_quant with different input shapes. The body exercises logic via `torch.randn`, `rocm_aiter_ops.group_fp8_quant` before asserting the expected outcome.
**CN:** 该测试用例验证 ROCm Aiter Group FP8 Quant Different Shapes 在特定场景下的行为。 函数体会先通过 `torch.randn`, `rocm_aiter_ops.group_fp8_quant` 驱动目标逻辑，再断言预期结果。

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
- **Standard library / 标准库**: `importlib.util`
- **Third-party / 第三方依赖**: `pytest`, `torch`, `aiter`
- **vLLM internal / vLLM 内部依赖**: `vllm._aiter_ops`, `vllm.platforms`
