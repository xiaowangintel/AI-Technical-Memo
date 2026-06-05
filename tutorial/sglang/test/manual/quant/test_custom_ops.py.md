# test_custom_ops.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/quant/test_custom_ops.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `custom ops` scenario in `test/manual/quant`. It uses SGLang's shared test infrastructure to configure models or services and verify expected functional behavior. / 该手动测试模块覆盖 `test/manual/quant` 中的 `custom ops` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Scenario logic / 场景逻辑
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/8ca7a71df787ad711ad3ac70a5bd2eb2bb398938/tests/quantization/test_fp8.py

import sys

import pytest
import torch

from sglang.srt.layers.quantization.fp8_kernel import is_fp8_fnuz, scaled_fp8_quant
from sglang.srt.utils import is_cuda, is_hip

_is_cuda = is_cuda()
_is_hip = is_hip()
_is_fp8_fnuz = is_fp8_fnuz()
fp8_dtype = torch.float8_e4m3fnuz if _is_fp8_fnuz else torch.float8_e4m3fn


@pytest.mark.parametrize("dtype", [torch.float16, torch.bfloat16])
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `is_cuda`, `is_hip`, `is_fp8_fnuz` and `parametrize`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 20-44: Test routines around test_scaled_fp8_quant_per_tensor / 测试例程
```python
def test_scaled_fp8_quant_per_tensor(dtype) -> None:

    def quantize_ref_per_tensor(tensor, inv_scale):
        # The reference implementation that fully aligns to
        # the kernel being tested.
        finfo = torch.finfo(fp8_dtype)
        scale = inv_scale.reciprocal()
        qweight = (tensor.to(torch.float32) * scale).clamp(min=finfo.min, max=finfo.max)
        qweight = qweight.to(fp8_dtype)
        return qweight

    def dequantize_per_tensor(tensor, inv_scale, dtype):
        fake_qweight = tensor.to(dtype)
        dq_weight = fake_qweight * inv_scale
        return dq_weight

    # Note that we use a shape % 8 != 0 to cover edge cases,
    # because scaled_fp8_quant is vectorized by 8.
    x = (torch.randn(size=(11, 11), device="cuda") * 13).to(dtype)

    # Test Per Tensor Dynamic quantization
    # scale = max(abs(x)) / FP8_E4M3_MAX
    y, scale = scaled_fp8_quant(x, None)
    ref_y = quantize_ref_per_tensor(x, scale)
    torch.testing.assert_close(y, ref_y)
```
**EN:** This range defines concrete test routine(s) `test_scaled_fp8_quant_per_tensor`, `quantize_ref_per_tensor` and `dequantize_per_tensor`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `finfo`, `reciprocal`, `to` and `clamp`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 45-57: Assertions and result checks / 断言与结果检查
```python
    torch.testing.assert_close(
        dequantize_per_tensor(y, scale, dtype),
        dequantize_per_tensor(ref_y, scale, dtype),
    )

    # Test Per Tensor Static quantization
    y, _ = scaled_fp8_quant(x, scale)
    ref_y = quantize_ref_per_tensor(x, scale)
    torch.testing.assert_close(y, ref_y)
    torch.testing.assert_close(
        dequantize_per_tensor(y, scale, dtype),
        dequantize_per_tensor(ref_y, scale, dtype),
    )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assert_close`, `dequantize_per_tensor`, `scaled_fp8_quant` and `quantize_ref_per_tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 58-152: Script entry point / 脚本入口
```python


if _is_cuda or _is_hip:

    @pytest.mark.parametrize("dtype", [torch.float16, torch.bfloat16])
    def test_scaled_fp8_quant_per_token_dynamic(dtype) -> None:
        def quantize_ref_per_token(tensor, inv_scale):
            # The reference implementation that fully aligns to
            # the kernel being tested.
            finfo = torch.finfo(fp8_dtype)
            scale = inv_scale.reciprocal()
            qweight = (tensor.to(torch.float32) * scale).clamp(
                min=finfo.min, max=finfo.max
            )
            qweight = qweight.to(fp8_dtype)
            return qweight

        def dequantize_per_token(tensor, inv_scale, dtype):
            fake_qweight = tensor.to(dtype)
            dq_weight = fake_qweight * inv_scale
            return dq_weight

        # Note that we use a shape % 8 = 0,
        # because per_token_quant_fp8 is vectorized by 8 elements.
        x = (torch.randn(size=(11, 16), device="cuda") * 13).to(dtype)

        # Test Per Tensor Dynamic quantization
        # scale = max(abs(x)) / FP8_E4M3_MAX
        y, scale = scaled_fp8_quant(x, None, use_per_token_if_dynamic=True)
        ref_y = quantize_ref_per_token(x, scale)
        torch.testing.assert_close(y, ref_y)
        torch.testing.assert_close(
            dequantize_per_token(y, scale, dtype),
            dequantize_per_token(ref_y, scale, dtype),
        )

    @pytest.mark.parametrize("dtype", [torch.float16, torch.bfloat16])
    def test_scaled_fp8_quant_with_padding(dtype) -> None:
        original_rows = 5
        x = (torch.randn(size=(original_rows, 16), device="cuda") * 13).to(dtype)

        padding_size = 10

        # Test with dynamic quantization
        y_dynamic, scale_dynamic = scaled_fp8_quant(
            x, None, num_token_padding=padding_size
        )

        # Verify output shape has the padded size
        assert y_dynamic.shape[0] == padding_size
        assert y_dynamic.shape[1] == x.shape[1]

        # Verify that the actual data in the non-padded region is correctly quantized
        y_without_padding, scale_without_padding = scaled_fp8_quant(x, None)
        torch.testing.assert_close(y_dynamic[:original_rows], y_without_padding)

        # Test with static quantization
        # First get a scale
        _, scale = scaled_fp8_quant(x, None)

        # Then use it for static quantization with padding
        y_static, _ = scaled_fp8_quant(x, scale, num_token_padding=padding_size)

        # Verify output shape has the padded size
        assert y_static.shape[0] == padding_size
        assert y_static.shape[1] == x.shape[1]

        # Verify that the actual data in the non-padded region is correctly quantized
        y_static_without_padding, _ = scaled_fp8_quant(x, scale)
        torch.testing.assert_close(y_static[:original_rows], y_static_without_padding)

        # Test with per-token dynamic quantization
        y_per_token, scale_per_token = scaled_fp8_quant(
            x, None, num_token_padding=padding_size, use_per_token_if_dynamic=True
        )

        # Verify output shape has the padded size
        assert y_per_token.shape[0] == padding_size
        assert y_per_token.shape[1] == x.shape[1]

        # Verify that the actual data in the non-padded region is correctly quantized
        y_per_token_without_padding, scale_per_token_without_padding = scaled_fp8_quant(
            x, None, use_per_token_if_dynamic=True
        )
        torch.testing.assert_close(
            y_per_token[:original_rows], y_per_token_without_padding
        )
        torch.testing.assert_close(
            scale_per_token[:original_rows], scale_per_token_without_padding
        )


if __name__ == "__main__":
    # Run the specific test function directly
    sys.exit(pytest.main([__file__]))
```
**EN:** This range defines concrete test routine(s) `test_scaled_fp8_quant_per_token_dynamic`, `quantize_ref_per_token`, `dequantize_per_token` and `test_scaled_fp8_quant_with_padding`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `parametrize`, `finfo`, `reciprocal` and `to`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Pytest markers / Pytest 标记
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- HTTP/API interaction / HTTP/API 交互
- Token-level inspection / Token 级分析

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `sys`
- **Third-party / 第三方库**: `pytest`, `torch`
- **Project Modules / 项目模块**: `sglang.srt.layers.quantization.fp8_kernel`, `sglang.srt.utils`
