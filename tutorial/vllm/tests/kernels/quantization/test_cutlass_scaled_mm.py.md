# test_cutlass_scaled_mm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/quantization/test_cutlass_scaled_mm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / quantization / test_cutlass_scaled_mm, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / quantization / test_cutlass_scaled_mm 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-6)
```python
"""Tests for cutlass kernels

Run `pytest tests/kernels/quantization/test_cutlass_scaled_mm.py`.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 8-16)
```python
import random

import pytest
import torch

from tests.kernels.utils import baseline_scaled_mm, opcheck, to_fp8, to_int8
from vllm import _custom_ops as ops
from vllm.platforms import current_platform
from vllm.utils.math_utils import cdiv
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as random, pytest, torch; shared test helpers from tests.kernels.utils; and vLLM components like vllm, vllm.platforms, vllm.utils.math_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 random、pytest、torch；共享测试辅助模块，例如 tests.kernels.utils；vLLM 内部组件，例如 vllm、vllm.platforms、vllm.utils.math_utils。

### Top-level block starting at line 18 (lines 18-19)
```python
if not current_platform.is_cuda():
    pytest.skip("These tests use CUTLASS which requires CUDA", allow_module_level=True)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 21-65)
```python
MNK_FACTORS = [
    (1, 256, 128),
    (1, 16384, 1024),
    (1, 24576, 496),
    (16, 256, 496),
    (16, 16384, 128),
    (16, 24576, 4096),
    (32, 8192, 4096),
    (32, 16384, 4096),
    (33, 1024, 1024),
    (33, 8192, 128),
    (64, 2048, 496),
    (64, 16384, 1024),
    (100, 8192, 496),
    (128, 32768, 4096),
    (256, 4096, 4096),
    (512, 256, 1024),
    (512, 8192, 4096),
    (512, 16384, 128),
    (512, 24576, 128),
]

# Shapes with N or K not divisible by 16.  These exercise the padding path
# inside CutlassFP8ScaledMMLinearKernel.apply_scaled_mm (e.g. Qwen2.5-VL
# vision MLP dims).
UNALIGNED_MNK_FACTORS = [
    (32, 3420, 1280),
    (32, 1280, 6840),
    (1, 3420, 1280),
    (64, 6840, 1280),
    (16, 100, 200),
    (33, 255, 513),
]

CUDA_DEVICES = [
    f"cuda:{i}" for i in range(1 if torch.accelerator.device_count() == 1 else 2)
]

# -1 means full extent in that dimension
TENSORWISE_GROUP_SHAPE = (-1, -1)
PER_TOKEN_GROUP_SHAPE = (1, -1)
PER_OUT_CH_GROUP_SHAPE = (-1, 1)

capability = current_platform.get_device_capability()
capability = capability[0] * 10 + capability[1]
```
**EN:** This block centralizes shared constants and parameter grids, including MNK_FACTORS, UNALIGNED_MNK_FACTORS, CUDA_DEVICES, TENSORWISE_GROUP_SHAPE, PER_TOKEN_GROUP_SHAPE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 MNK_FACTORS、UNALIGNED_MNK_FACTORS、CUDA_DEVICES、TENSORWISE_GROUP_SHAPE、PER_TOKEN_GROUP_SHAPE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `rand_int8` (lines 68-69)
```python
def rand_int8(shape: tuple, device: str = "cuda"):
    return to_int8(torch.rand(shape, device=device) * 255 - 128)
```
**EN:** This helper function implements the shared logic for rand int8. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 rand int8 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `group_scale_helper` (lines 72-73)
```python
def group_scale_helper(shape, group_shape):
    return [shape[i] if s < 0 else s for i, s in enumerate(group_shape)]
```
**EN:** This helper function implements the shared logic for group scale helper. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 group scale helper 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `scale_shape` (lines 76-79)
```python
def scale_shape(shape, group_shape):
    assert len(shape) == len(group_shape)
    group_shape = group_scale_helper(shape, group_shape)
    return tuple(cdiv(shape[i], group_shape[i]) for i in range(len(group_shape)))
```
**EN:** This helper function implements the shared logic for scale shape. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 scale shape 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `cutlass_fp8_gemm_helper` (lines 82-115)
```python
def cutlass_fp8_gemm_helper(
    m: int,
    n: int,
    k: int,
    a_scale_group_shape: tuple,
    b_scale_group_shape: tuple,
    use_bias: bool,
    out_dtype: type[torch.dtype] = torch.bfloat16,
    device: str = "cuda",
):
    # Test for a cutlass kernel with per-token activation quantization
    # and per-output channel weight quantization.
    a = to_fp8(torch.randn((m, k), device=device))
    b = to_fp8(torch.randn((n, k), device=device).t())

    a_scales_shape = scale_shape(a.shape, a_scale_group_shape)
    b_scales_shape = scale_shape(b.shape, b_scale_group_shape)

    scale_a = torch.randn(a_scales_shape, device=device, dtype=torch.float32)
    scale_b = torch.randn(b_scales_shape, device=device, dtype=torch.float32)

    # make scales M-major for blockwise quant, doesn't affect 1D scales
    scale_a = scale_a.t().contiguous().t()
    # make scales K-major for blockwise quant, doesn't affect 1D scales
    scale_b = scale_b.t().contiguous().t()

    bias = torch.rand((n,), device=device, dtype=out_dtype) * 10 if use_bias else None

    out = ops.cutlass_scaled_mm(a, b, scale_a, scale_b, out_dtype, bias)
    baseline = baseline_scaled_mm(a, b, scale_a, scale_b, out_dtype, bias)

    torch.testing.assert_close(out, baseline, rtol=5e-1, atol=1.5e-1)

    opcheck(torch.ops._C.cutlass_scaled_mm, (out, a, b, scale_a, scale_b, bias))
```
**EN:** This helper function implements the shared logic for cutlass FP8 gemm helper. it also validates that the custom operator entry point is wired correctly. numeric results are compared against a reference with explicit tolerances.
**CN:** 该辅助函数实现了 cutlass FP8 gemm helper 所需的共享逻辑。 它还会校验自定义算子入口是否正确接线；数值结果会在显式容差下与参考结果进行比较。

### Function `cutlass_int8_gemm_helper` (lines 118-146)
```python
def cutlass_int8_gemm_helper(
    m: int,
    n: int,
    k: int,
    a_scale_group_shape: tuple,
    b_scale_group_shape: tuple,
    use_bias: bool,
    out_dtype: type[torch.dtype] = torch.bfloat16,
    device: str = "cuda",
):
    # Test for a cutlass kernel with per-token activation quantization
    # and per-output channel weight quantization.
    a = to_int8(torch.randn((m, k), device=device) * 5)
    b = to_int8(torch.randn((n, k), device=device).t() * 5)

    a_scales_shape = scale_shape(a.shape, a_scale_group_shape)
    b_scales_shape = scale_shape(b.shape, b_scale_group_shape)

    scale_a = torch.randn(a_scales_shape, device=device, dtype=torch.float32)
    scale_b = torch.randn(b_scales_shape, device=device, dtype=torch.float32)

    bias = torch.rand((n,), device=device, dtype=out_dtype) * 10 if use_bias else None

    out = ops.cutlass_scaled_mm(a, b, scale_a, scale_b, out_dtype, bias)
    baseline = baseline_scaled_mm(a, b, scale_a, scale_b, out_dtype, bias)

    torch.testing.assert_close(out, baseline, rtol=1e-1, atol=1e0)

    opcheck(torch.ops._C.cutlass_scaled_mm, (out, a, b, scale_a, scale_b, bias))
```
**EN:** This helper function implements the shared logic for cutlass int8 gemm helper. it also validates that the custom operator entry point is wired correctly. numeric results are compared against a reference with explicit tolerances.
**CN:** 该辅助函数实现了 cutlass int8 gemm helper 所需的共享逻辑。 它还会校验自定义算子入口是否正确接线；数值结果会在显式容差下与参考结果进行比较。

### Function `test_cutlass_fp8_gemm` (lines 149-164)
```python
@pytest.mark.parametrize("m,n,k", MNK_FACTORS)
@pytest.mark.parametrize(
    "a_scale_group_shape", [PER_TOKEN_GROUP_SHAPE, TENSORWISE_GROUP_SHAPE]
)
@pytest.mark.parametrize(
    "b_scale_group_shape", [PER_OUT_CH_GROUP_SHAPE, TENSORWISE_GROUP_SHAPE]
)
@pytest.mark.parametrize("use_bias", [True, False])
@pytest.mark.skipif(
    not current_platform.has_device_capability(89),
    reason="FP8 is not supported on this GPU type.",
)
def test_cutlass_fp8_gemm(
    m: int, n: int, k: int, a_scale_group_shape, b_scale_group_shape, use_bias: bool
):
    cutlass_fp8_gemm_helper(m, n, k, a_scale_group_shape, b_scale_group_shape, use_bias)
```
**EN:** This pytest case verifies cutlass FP8 gemm. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, n, k, a_scale_group_shape. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 cutlass FP8 gemm 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 m、n、k、a_scale_group_shape 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

### Function `test_cutlass_fp8_gemm_padded` (lines 167-220)
```python
@pytest.mark.parametrize("m,n,k", UNALIGNED_MNK_FACTORS)
@pytest.mark.parametrize(
    "a_scale_group_shape", [PER_TOKEN_GROUP_SHAPE, TENSORWISE_GROUP_SHAPE]
)
@pytest.mark.parametrize(
    "b_scale_group_shape", [PER_OUT_CH_GROUP_SHAPE, TENSORWISE_GROUP_SHAPE]
)
@pytest.mark.parametrize("use_bias", [True, False])
@pytest.mark.skipif(
    not current_platform.has_device_capability(89),
    reason="FP8 is not supported on this GPU type.",
)
def test_cutlass_fp8_gemm_padded(
    m: int, n: int, k: int, a_scale_group_shape, b_scale_group_shape, use_bias: bool
):
    """Test CUTLASS FP8 GEMM with padding for non-16-aligned N/K dims.

    Exercises CutlassFP8ScaledMMLinearKernel.apply_scaled_mm which pads
    inputs to satisfy CUTLASS alignment requirements — the path taken by
    models like Qwen2.5-VL whose vision MLP has non-16-aligned dims.
    """
    from vllm.model_executor.kernels.linear.scaled_mm.cutlass import (
        CutlassFP8ScaledMMLinearKernel,
    )

    a = to_fp8(torch.randn((m, k), device="cuda"))
    b = to_fp8(torch.randn((n, k), device="cuda").t())

    a_scales_shape = scale_shape(a.shape, a_scale_group_shape)
    b_scales_shape = scale_shape(b.shape, b_scale_group_shape)

    scale_a = torch.randn(a_scales_shape, device="cuda", dtype=torch.float32)
    scale_b = torch.randn(b_scales_shape, device="cuda", dtype=torch.float32)

    scale_a = scale_a.t().contiguous().t()
    scale_b = scale_b.t().contiguous().t()

    out_dtype = torch.bfloat16
    bias = torch.rand((n,), device="cuda", dtype=out_dtype) * 10 if use_bias else None

    baseline = baseline_scaled_mm(a, b, scale_a, scale_b, out_dtype, bias)

    kernel = object.__new__(CutlassFP8ScaledMMLinearKernel)
    out = kernel.apply_scaled_mm(
        A=a,
        B=b,
        out_dtype=out_dtype,
        As=scale_a,
        Bs=scale_b,
        bias=bias,
        output_shape=[m, n],
    )

    torch.testing.assert_close(out, baseline, rtol=5e-1, atol=1.5e-1)
```
**EN:** This pytest case verifies cutlass FP8 gemm padded. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, n, k, a_scale_group_shape. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 cutlass FP8 gemm padded 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 m、n、k、a_scale_group_shape 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

### Function `test_cutlass_fp8_blockwise_scale_gemm` (lines 223-241)
```python
@pytest.mark.parametrize("m,n,k", MNK_FACTORS)
@pytest.mark.parametrize(
    "a_scale_group_shape,b_scale_group_shape", [((1, 128), (128, 128))]
)
@pytest.mark.parametrize("use_bias", [False])
@pytest.mark.skipif(
    not current_platform.has_device_capability(90),
    reason="FP8 blockwise is not supported on this GPU type.",
)
def test_cutlass_fp8_blockwise_scale_gemm(
    m: int, n: int, k: int, a_scale_group_shape, b_scale_group_shape, use_bias: bool
):
    if k % b_scale_group_shape[0] != 0 or n % b_scale_group_shape[1] != 0:
        return
    if m % a_scale_group_shape[0] != 0 or k % a_scale_group_shape[1] != 0:
        return
    if m % 4 != 0 and current_platform.has_device_capability(100):
        return
    cutlass_fp8_gemm_helper(m, n, k, a_scale_group_shape, b_scale_group_shape, use_bias)
```
**EN:** This pytest case verifies cutlass FP8 blockwise scale gemm. It is parameterized across 3 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, n, k, a_scale_group_shape. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 cutlass FP8 blockwise scale gemm 的行为。 它通过 3 组参数化输入覆盖多种场景；它会使用诸如 m、n、k、a_scale_group_shape 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

### Function `test_cutlass_int8_gemm` (lines 244-257)
```python
@pytest.mark.parametrize("m,n,k", MNK_FACTORS)
@pytest.mark.parametrize(
    "a_scale_group_shape", [PER_TOKEN_GROUP_SHAPE, TENSORWISE_GROUP_SHAPE]
)
@pytest.mark.parametrize(
    "b_scale_group_shape", [PER_OUT_CH_GROUP_SHAPE, TENSORWISE_GROUP_SHAPE]
)
@pytest.mark.parametrize("use_bias", [True, False])
def test_cutlass_int8_gemm(
    m: int, n: int, k: int, a_scale_group_shape, b_scale_group_shape, use_bias: bool
):
    cutlass_int8_gemm_helper(
        m, n, k, a_scale_group_shape, b_scale_group_shape, use_bias
    )
```
**EN:** This pytest case verifies cutlass int8 gemm. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, n, k, a_scale_group_shape.
**CN:** 该 pytest 用例验证 cutlass int8 gemm 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 m、n、k、a_scale_group_shape 等 fixture 或输入。

### Function `test_cutlass_int8_gemm_output_dtype` (lines 260-282)
```python
@pytest.mark.parametrize(
    "a_scale_group_shape", [PER_TOKEN_GROUP_SHAPE, TENSORWISE_GROUP_SHAPE]
)
@pytest.mark.parametrize(
    "b_scale_group_shape", [PER_OUT_CH_GROUP_SHAPE, TENSORWISE_GROUP_SHAPE]
)
@pytest.mark.parametrize("out_dtype", [torch.bfloat16, torch.float16])
@pytest.mark.parametrize("use_bias", [True, False])
def test_cutlass_int8_gemm_output_dtype(
    a_scale_group_shape,
    b_scale_group_shape,
    out_dtype: type[torch.dtype],
    use_bias: bool,
):
    cutlass_int8_gemm_helper(
        512,
        512,
        512,
        a_scale_group_shape,
        b_scale_group_shape,
        use_bias,
        out_dtype=out_dtype,
    )
```
**EN:** This pytest case verifies cutlass int8 gemm output dtype. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as a_scale_group_shape, b_scale_group_shape, out_dtype, use_bias.
**CN:** 该 pytest 用例验证 cutlass int8 gemm output dtype 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 a_scale_group_shape、b_scale_group_shape、out_dtype、use_bias 等 fixture 或输入。

### Function `test_cutlass_fp8_gemm_output_dtype` (lines 285-311)
```python
@pytest.mark.parametrize(
    "a_scale_group_shape", [PER_TOKEN_GROUP_SHAPE, TENSORWISE_GROUP_SHAPE]
)
@pytest.mark.parametrize(
    "b_scale_group_shape", [PER_OUT_CH_GROUP_SHAPE, TENSORWISE_GROUP_SHAPE]
)
@pytest.mark.parametrize("out_dtype", [torch.bfloat16, torch.float16])
@pytest.mark.parametrize("use_bias", [True, False])
@pytest.mark.skipif(
    not current_platform.has_device_capability(89),
    reason="FP8 is not supported on this GPU type.",
)
def test_cutlass_fp8_gemm_output_dtype(
    a_scale_group_shape,
    b_scale_group_shape,
    out_dtype: type[torch.dtype],
    use_bias: bool,
):
    cutlass_fp8_gemm_helper(
        512,
        512,
        512,
        a_scale_group_shape,
        b_scale_group_shape,
        use_bias,
        out_dtype=out_dtype,
    )
```
**EN:** This pytest case verifies cutlass FP8 gemm output dtype. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as a_scale_group_shape, b_scale_group_shape, out_dtype, use_bias. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 cutlass FP8 gemm output dtype 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 a_scale_group_shape、b_scale_group_shape、out_dtype、use_bias 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

### Function `test_cutlass_fp8_blockwise_scale_gemm_dtype` (lines 314-337)
```python
@pytest.mark.parametrize(
    "a_scale_group_shape,b_scale_group_shape", [((1, 128), (128, 128))]
)
@pytest.mark.parametrize("out_dtype", [torch.bfloat16, torch.float16])
@pytest.mark.parametrize("use_bias", [False])
@pytest.mark.skipif(
    not current_platform.has_device_capability(90),
    reason="FP8 blockwise is not supported on this GPU type.",
)
def test_cutlass_fp8_blockwise_scale_gemm_dtype(
    a_scale_group_shape,
    b_scale_group_shape,
    out_dtype: type[torch.dtype],
    use_bias: bool,
):
    cutlass_fp8_gemm_helper(
        512,
        512,
        512,
        a_scale_group_shape,
        b_scale_group_shape,
        use_bias,
        out_dtype=out_dtype,
    )
```
**EN:** This pytest case verifies cutlass FP8 blockwise scale gemm dtype. It is parameterized across 3 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as a_scale_group_shape, b_scale_group_shape, out_dtype, use_bias. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 cutlass FP8 blockwise scale gemm dtype 的行为。 它通过 3 组参数化输入覆盖多种场景；它会使用诸如 a_scale_group_shape、b_scale_group_shape、out_dtype、use_bias 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

### Function `test_cutlass_fp8_gemm_devices` (lines 340-364)
```python
@pytest.mark.parametrize(
    "a_scale_group_shape", [PER_TOKEN_GROUP_SHAPE, TENSORWISE_GROUP_SHAPE]
)
@pytest.mark.parametrize(
    "b_scale_group_shape", [PER_OUT_CH_GROUP_SHAPE, TENSORWISE_GROUP_SHAPE]
)
@pytest.mark.parametrize("use_bias", [True, False])
@pytest.mark.parametrize("device", CUDA_DEVICES)
@pytest.mark.skipif(
    not current_platform.has_device_capability(89),
    reason="FP8 is not supported on this GPU type.",
)
def test_cutlass_fp8_gemm_devices(
    a_scale_group_shape, b_scale_group_shape, use_bias: bool, device: str
):
    cutlass_fp8_gemm_helper(
        512,
        512,
        512,
        a_scale_group_shape,
        b_scale_group_shape,
        use_bias,
        torch.bfloat16,
        device,
    )
```
**EN:** This pytest case verifies cutlass FP8 gemm devices. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as a_scale_group_shape, b_scale_group_shape, use_bias, device. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 cutlass FP8 gemm devices 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 a_scale_group_shape、b_scale_group_shape、use_bias、device 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

### Function `test_cutlass_int8_gemm_devices` (lines 367-387)
```python
@pytest.mark.parametrize(
    "a_scale_group_shape", [PER_TOKEN_GROUP_SHAPE, TENSORWISE_GROUP_SHAPE]
)
@pytest.mark.parametrize(
    "b_scale_group_shape", [PER_OUT_CH_GROUP_SHAPE, TENSORWISE_GROUP_SHAPE]
)
@pytest.mark.parametrize("use_bias", [True, False])
@pytest.mark.parametrize("device", CUDA_DEVICES)
def test_cutlass_int8_gemm_devices(
    a_scale_group_shape, b_scale_group_shape, use_bias: bool, device: str
):
    cutlass_int8_gemm_helper(
        512,
        512,
        512,
        a_scale_group_shape,
        b_scale_group_shape,
        use_bias,
        out_dtype=torch.bfloat16,
        device=device,
    )
```
**EN:** This pytest case verifies cutlass int8 gemm devices. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as a_scale_group_shape, b_scale_group_shape, use_bias, device.
**CN:** 该 pytest 用例验证 cutlass int8 gemm devices 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 a_scale_group_shape、b_scale_group_shape、use_bias、device 等 fixture 或输入。

### Function `test_cutlass_fp8_gemm_m_sweep` (lines 395-413)
```python
@pytest.mark.parametrize(
    "a_scale_group_shape", [PER_TOKEN_GROUP_SHAPE, TENSORWISE_GROUP_SHAPE]
)
@pytest.mark.parametrize(
    "b_scale_group_shape", [PER_OUT_CH_GROUP_SHAPE, TENSORWISE_GROUP_SHAPE]
)
@pytest.mark.parametrize("use_bias", [True, False])
@pytest.mark.skipif(
    not current_platform.has_device_capability(89),
    reason="FP8 is not supported on this GPU type.",
)
def test_cutlass_fp8_gemm_m_sweep(
    a_scale_group_shape, b_scale_group_shape, use_bias: bool
):
    for nk in range(32, 128, 32):
        for m in range(1, 128):
            cutlass_fp8_gemm_helper(
                m, nk, nk, a_scale_group_shape, b_scale_group_shape, use_bias
            )
```
**EN:** This pytest case verifies cutlass FP8 gemm m sweep. It is parameterized across 3 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as a_scale_group_shape, b_scale_group_shape, use_bias. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 cutlass FP8 gemm m sweep 的行为。 它通过 3 组参数化输入覆盖多种场景；它会使用诸如 a_scale_group_shape、b_scale_group_shape、use_bias 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

### Function `test_cutlass_int8_gemm_m_sweep` (lines 416-430)
```python
@pytest.mark.parametrize(
    "a_scale_group_shape", [PER_TOKEN_GROUP_SHAPE, TENSORWISE_GROUP_SHAPE]
)
@pytest.mark.parametrize(
    "b_scale_group_shape", [PER_OUT_CH_GROUP_SHAPE, TENSORWISE_GROUP_SHAPE]
)
@pytest.mark.parametrize("use_bias", [True, False])
def test_cutlass_int8_gemm_m_sweep(
    a_scale_group_shape, b_scale_group_shape, use_bias: bool
):
    for nk in range(32, 128, 32):
        for m in range(1, 128):
            cutlass_int8_gemm_helper(
                m, nk, nk, a_scale_group_shape, b_scale_group_shape, use_bias
            )
```
**EN:** This pytest case verifies cutlass int8 gemm m sweep. It is parameterized across 3 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as a_scale_group_shape, b_scale_group_shape, use_bias.
**CN:** 该 pytest 用例验证 cutlass int8 gemm m sweep 的行为。 它通过 3 组参数化输入覆盖多种场景；它会使用诸如 a_scale_group_shape、b_scale_group_shape、use_bias 等 fixture 或输入。

### Function `test_cutlass_int8_azp_bias_fold` (lines 433-479)
```python
@pytest.mark.parametrize("m", [32, 64, 128])
@pytest.mark.parametrize("n", [16, 32, 64])
@pytest.mark.parametrize("k", [64, 128, 256])
@pytest.mark.parametrize("out_dtype", [torch.bfloat16, torch.float16])
@pytest.mark.skip
def test_cutlass_int8_azp_bias_fold(m: int, n: int, k: int, out_dtype: torch.dtype):
    # Currently, the test is failing because folding azp into
    # 16-bit bias loses too much precision
    scale_a = torch.randn((1, 1), device="cuda", dtype=torch.float32) / 10
    scale_b = torch.randn((1, n), device="cuda", dtype=torch.float32) / 10

    aq_i8 = rand_int8((m, k))
    bq_i8 = rand_int8((n, k)).t()

    aq_i32 = aq_i8.to(dtype=torch.int32)
    bq_i32 = bq_i8.to(dtype=torch.int32)

    aq_f32 = aq_i8.to(dtype=torch.float32)
    bq_f32 = bq_i8.to(dtype=torch.float32)

    b_dq = scale_b * bq_f32

    azp_a = torch.rand((1,), device="cuda", dtype=torch.float32) * 10 + 1.5
    azp_aq_i8 = (azp_a / scale_a).to(dtype=torch.int8)
    azp_a = azp_aq_i8.to(dtype=torch.float32) * scale_a  # correct for rounding

    a_dq = scale_a * (aq_i32 + azp_aq_i8).to(dtype=torch.float32)
    torch.testing.assert_close(a_dq, scale_a * aq_f32 + azp_a)

    baseline_dq = torch.mm(a_dq, b_dq).to(out_dtype)

    J = torch.ones((1, k), device="cuda", dtype=torch.float32)
    azp_bias = (azp_a * scale_b * (J @ bq_f32)).to(out_dtype)
    assert azp_bias.shape == (1, n)
    assert azp_bias[0, :].shape == (n,)

    baseline_q = (
        scale_a.to(device="cpu")
        * scale_b.to(device="cpu")
        * ((aq_i32 + azp_aq_i8).to(device="cpu") @ bq_i32.to(device="cpu"))
    ).to(dtype=out_dtype, device="cuda")

    out = ops.cutlass_scaled_mm(
        aq_i8, bq_i8, scale_a, scale_b, out_dtype=out_dtype, bias=azp_bias[0, :]
    )
    torch.testing.assert_close(out, baseline_dq, rtol=1e-2, atol=1e0)
    torch.testing.assert_close(out, baseline_q, rtol=1e-2, atol=1e0)
```
**EN:** This pytest case verifies cutlass int8 azp bias fold. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, n, k, out_dtype. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 cutlass int8 azp bias fold 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 m、n、k、out_dtype 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较。

### Function `test_cutlass_int8_azp` (lines 482-554)
```python
@pytest.mark.parametrize("m", [32, 64, 128])
@pytest.mark.parametrize("n", [16, 32, 64])
@pytest.mark.parametrize("k", [64, 128, 256])
@pytest.mark.parametrize("out_dtype", [torch.bfloat16, torch.float16])
@pytest.mark.parametrize("use_bias", [True, False])
@pytest.mark.parametrize("azp_per_token", [True, False])
def test_cutlass_int8_azp(
    m: int, n: int, k: int, out_dtype: torch.dtype, use_bias: bool, azp_per_token: bool
):
    m_azp = m if azp_per_token else 1
    scale_a = torch.randn((m_azp, 1), device="cuda", dtype=torch.float32) / 10
    scale_b = torch.randn((1, n), device="cuda", dtype=torch.float32) / 10

    aq_i8 = rand_int8((m, k))
    aq_i32 = aq_i8.to(dtype=torch.int32)
    aq_f32 = aq_i8.to(dtype=torch.float32)

    bq_i8 = rand_int8((n, k)).t()
    bq_i32 = bq_i8.to(dtype=torch.int32)
    bq_f32 = bq_i8.to(dtype=torch.float32)
    b_dq = scale_b * bq_f32

    azp_a = torch.rand((m_azp, 1), device="cuda", dtype=torch.float32) * 10 + 1.5
    azp_aq_i8 = (azp_a / scale_a).to(dtype=torch.int8)
    azp_a = azp_aq_i8.to(dtype=torch.float32) * scale_a  # correct for rounding

    a_dq = scale_a * (aq_i32 - azp_aq_i8).to(dtype=torch.float32)
    torch.testing.assert_close(a_dq, scale_a * aq_f32 - azp_a, rtol=1e-4, atol=1e-3)

    if use_bias:
        bias = torch.rand((1, n), device="cuda", dtype=out_dtype) * 10 + 2.5
    else:
        bias = torch.zeros((1, n), device="cuda", dtype=out_dtype)

    baseline_dq = (torch.mm(a_dq, b_dq) + bias).to(out_dtype)

    # int32 mm not supported on CUDA
    a_noazp_i32_cpu = (aq_i32 - azp_aq_i8).to(device="cpu")
    cq = (a_noazp_i32_cpu @ bq_i32.to(device="cpu")).to(device="cuda")
    baseline_q = (scale_a * scale_b * cq + bias).to(dtype=out_dtype)

    # Hadamard is just the sum of the cols
    azp_adj_i32 = bq_i32.sum(dim=0, keepdim=True, dtype=torch.int32)
    azp_i32 = azp_aq_i8.to(dtype=torch.int32)
    func_bias = bias if use_bias else None

    if azp_per_token:
        out = ops.cutlass_scaled_mm_azp(
            aq_i8, bq_i8, scale_a, scale_b, out_dtype, azp_adj_i32, azp_i32, func_bias
        )
    else:
        azp_with_adj_i32 = azp_i32 * azp_adj_i32
        out = ops.cutlass_scaled_mm_azp(
            aq_i8, bq_i8, scale_a, scale_b, out_dtype, azp_with_adj_i32, None, func_bias
        )

    # bfloat16 precision is 7-bit mantissa -> 2^-8 ~ 0.4%
    # float16 precision is 10-bit mantissa -> 2^-11 ~ 0.05%
    rtol = 1e-2 if out_dtype == torch.bfloat16 else 1e-3
    atol = 1e-3
    torch.testing.assert_close(out, baseline_dq, rtol=rtol, atol=atol)
    torch.testing.assert_close(out, baseline_q, rtol=rtol, atol=atol)

    if azp_per_token:
        opcheck(
            torch.ops._C.cutlass_scaled_mm_azp,
            (out, aq_i8, bq_i8, scale_a, scale_b, azp_adj_i32, azp_i32, func_bias),
        )
    else:
        opcheck(
            torch.ops._C.cutlass_scaled_mm_azp,
            (out, aq_i8, bq_i8, scale_a, scale_b, azp_with_adj_i32, None, func_bias),
        )
```
**EN:** This pytest case verifies cutlass int8 azp. It is parameterized across 6 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, n, k, out_dtype. it also validates that the custom operator entry point is wired correctly. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 cutlass int8 azp 的行为。 它通过 6 组参数化输入覆盖多种场景；它会使用诸如 m、n、k、out_dtype 等 fixture 或输入；它还会校验自定义算子入口是否正确接线；数值结果会在显式容差下与参考结果进行比较。

### Function `test_cutlass_subset` (lines 558-573)
```python
def test_cutlass_subset():
    big_m, big_n, big_k = 1024, 1024, 1024
    m, n, k = 512, 512, 512

    whole_a = to_int8(torch.randn((big_m, big_k), device="cuda") * 5)
    whole_b = to_int8(torch.randn((big_n, big_k), device="cuda").t() * 5)
    a = whole_a[0:m, 0:k]
    b = whole_b[0:k, 0:n]

    scale_a = torch.randn((1, 1), device="cuda", dtype=torch.float32) / 10
    scale_b = torch.randn((1, 1), device="cuda", dtype=torch.float32) / 10

    out = ops.cutlass_scaled_mm(a, b, scale_a, scale_b, out_dtype=torch.bfloat16)
    baseline = baseline_scaled_mm(a, b, scale_a, scale_b, out_dtype=torch.bfloat16)

    torch.testing.assert_close(out, baseline, rtol=1e-1, atol=1e0)
```
**EN:** This pytest case verifies cutlass subset. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 cutlass subset 的行为。 数值结果会在显式容差下与参考结果进行比较。

### Class `CutlassLayer` (lines 577-577)
```python
class CutlassLayer(torch.nn.Module):
```
**EN:** This helper class groups the state and behavior needed for CutlassLayer. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 CutlassLayer 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `CutlassLayer.__init__` (lines 578-583)
```python
    def __init__(self, b, scale_a, scale_b, out_dtype):
        super().__init__()
        self.b = b
        self.scale_a = scale_a
        self.scale_b = scale_b
        self.out_dtype = out_dtype
```
**EN:** This method implements the initialization for `CutlassLayer`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `CutlassLayer` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `CutlassLayer.forward` (lines 585-588)
```python
    def forward(self, a):
        return ops.cutlass_scaled_mm(
            a, self.b, self.scale_a, self.scale_b, self.out_dtype
        )
```
**EN:** This method on `CutlassLayer` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `CutlassLayer` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_cutlass_cuda_graph` (lines 591-620)
```python
@pytest.mark.parametrize("per_act_token", [True, False])
@pytest.mark.parametrize("per_out_ch", [True, False])
def test_cutlass_cuda_graph(per_act_token: bool, per_out_ch: bool):
    m, n, k = 512, 512, 512

    a = to_int8(torch.randn((m, k), device="cuda"))
    b = to_int8(torch.randn((n, k), device="cuda").t())

    m_a_scales = m if per_act_token else 1
    n_b_scales = n if per_out_ch else 1

    scale_a = torch.randn((m_a_scales, 1), device="cuda", dtype=torch.float32) / 10
    scale_b = torch.randn((1, n_b_scales), device="cuda", dtype=torch.float32) / 10

    # Construct a trivial model with a single layer that calls a CUTLASS kernel
    model = CutlassLayer(b, scale_a, scale_b, torch.bfloat16)

    # Run the model with a cuda graph
    stream = torch.cuda.Stream()
    with torch.cuda.stream(stream):
        g = torch.cuda.CUDAGraph()
        with torch.cuda.graph(g):
            out = model(a)
    out.zero_()
    g.replay()

    baseline = torch.mm(
        scale_a * a.to(dtype=torch.float32), scale_b * b.to(dtype=torch.float32)
    ).to(torch.bfloat16)
    torch.testing.assert_close(out, baseline, rtol=1e-1, atol=1e0)
```
**EN:** This pytest case verifies cutlass CUDA graph. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as per_act_token, per_out_ch. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 cutlass CUDA graph 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 per_act_token、per_out_ch 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较。

### Function `test_cutlass_support_opcheck` (lines 623-624)
```python
def test_cutlass_support_opcheck():
    opcheck(torch.ops._C.cutlass_scaled_mm_supports_fp8, (capability,))
```
**EN:** This pytest case verifies cutlass support opcheck. it also validates that the custom operator entry point is wired correctly.
**CN:** 该 pytest 用例验证 cutlass support opcheck 的行为。 它还会校验自定义算子入口是否正确接线。

### Function `test_cutlass_fp8_group_gemm` (lines 627-752)
```python
@pytest.mark.parametrize("num_experts", [8, 64])
@pytest.mark.parametrize("per_act_token", [True, False])
@pytest.mark.parametrize("per_out_ch", [True, False])
@pytest.mark.parametrize("use_bias", [False])
@pytest.mark.skipif(
    (lambda x: x is None or not ops.cutlass_group_gemm_supported(x.to_int()))(
        current_platform.get_device_capability()
    ),
    reason="Grouped gemm is not supported on this GPU type.",
)
def test_cutlass_fp8_group_gemm(
    num_experts: int, per_act_token: bool, per_out_ch: bool, use_bias: bool
):
    # Device and dtype setup
    device = "cuda"
    out_dtype = torch.half

    # Create separate A, B, C tensors for each group
    a_tensors = []
    b_tensors = []
    a_scales_tensors = []
    b_scales_tensors = []
    baseline_tensors = []

    expert_offsets = torch.zeros((num_experts + 1), device=device, dtype=torch.int64)

    problem_sizes = torch.zeros((num_experts, 3), device=device, dtype=torch.int32)

    if not per_act_token:
        one_scale_a = torch.randn((1, 1), device=device, dtype=torch.float32)

    alignment = 16  # 128 // 8
    # For variation, each group has dimensions
    n_g = alignment * random.randint(1, 64)
    k_g = alignment * random.randint(1, 64)
    for g in range(num_experts):
        m_g = alignment * random.randint(1, 64)

        expert_offsets[g + 1] = expert_offsets[g] + m_g
        problem_sizes[g][0] = m_g
# ... excerpt ...
        a_scales_tensors_stacked = one_scale_a

    b_scales_tensors_stacked = torch.empty(
        (num_experts, n_b_scales), device=device, dtype=torch.float32
    )
    for g in range(num_experts):
        b_scales_tensors_stacked[g] = b_scales_tensors[g]

    out_tensors_stacked = torch.zeros(
        (expert_offsets[num_experts], n_g), device=device, dtype=out_dtype
    )

    ab_strides = torch.full(
        (num_experts,), a_tensors_stacked.stride(0), device="cuda", dtype=torch.int64
    )
    c_strides = torch.full(
        (num_experts,), out_tensors_stacked.stride(0), device="cuda", dtype=torch.int64
    )

    ops.cutlass_moe_mm(
        out_tensors_stacked,
        a_tensors_stacked,
        b_tensors_stacked,
        a_scales_tensors_stacked,
        b_scales_tensors_stacked,
        expert_offsets[:-1],
        problem_sizes,
        ab_strides,
        ab_strides,
        c_strides,
        per_act_token,
        per_out_ch,
    )

    # Validate each group's result against the baseline
    for g in range(num_experts):
        baseline = baseline_tensors[g]
        c = out_tensors_stacked[expert_offsets[g] : expert_offsets[g + 1]]
        torch.testing.assert_close(c, baseline, rtol=1e-2, atol=5e-4)
```
**EN:** This pytest case verifies cutlass FP8 group gemm. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_experts, per_act_token, per_out_ch, use_bias. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 cutlass FP8 group gemm 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 num_experts、per_act_token、per_out_ch、use_bias 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `random`
- `pytest`
- `torch`
- `tests.kernels.utils -> baseline_scaled_mm, opcheck, to_fp8, to_int8`
- `vllm -> _custom_ops`
- `vllm.platforms -> current_platform`
- `vllm.utils.math_utils -> cdiv`
