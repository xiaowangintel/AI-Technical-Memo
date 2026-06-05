# test_diffusion_nvfp4_scaled_mm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/diffusion/test_diffusion_nvfp4_scaled_mm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import sys". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import sys”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-35: Imports and module setup
```python
import sys

import flashinfer
import pytest
import torch

from sglang.jit_kernel.nvfp4 import cutlass_scaled_fp4_mm, scaled_fp4_quant
from sglang.multimodal_gen.runtime.layers.quantization import (
    modelopt_quant as diffusion_modelopt_quant,
)
from sglang.multimodal_gen.runtime.layers.quantization.modelopt_quant import (
    ModelOptFp4Config,
    ModelOptFp4LinearMethod,
)
from sglang.multimodal_gen.runtime.platforms import current_platform
from sglang.srt.layers.quantization.modelopt_quant import pad_nvfp4_weight
from sglang.test.ci.ci_register import register_cuda_ci

# B200-only correctness coverage for diffusion NVFP4 scaled mm.
register_cuda_ci(est_time=15, suite="base-b-kernel-unit-1-gpu-b200")

DEVICE = "cuda"
DTYPE = torch.bfloat16
BLOCK_SIZE = 16
FLOAT4_E2M1_MAX = 6.0
FLOAT8_E4M3_MAX = torch.finfo(torch.float8_e4m3fn).max
FP4_VALUE_LUT = (0.0, 0.5, 1.0, 1.5, 2.0, 3.0, 4.0, 6.0)
DEEPGEMM_FP4_MAX_DIFF = 0.02
TEST_CASES = [
    pytest.param(19, 150, 80, id="padding_regression"),
    pytest.param(512, 6144, 128, id="flux2_projection_shape"),
]
FLUX2_PROJECTION_SHAPE = (512, 6144, 128)
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 36-39: Function `_nvfp4_supported`
```python
def _nvfp4_supported() -> bool:
    return torch.cuda.is_available() and torch.cuda.get_device_capability() >= (10, 0)
```
**EN:** This block defines `_nvfp4_supported`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_nvfp4_supported`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 40-44: Function `_make_global_scale`
```python
def _make_global_scale(x: torch.Tensor) -> torch.Tensor:
    max_abs = torch.amax(x.abs()).clamp_min_(1e-6)
    return (FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX / max_abs).to(torch.float32)
```
**EN:** This block defines `_make_global_scale`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_make_global_scale`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 45-53: Function `_calc_diff`
```python
def _calc_diff(x: torch.Tensor, y: torch.Tensor) -> float:
    x, y = x.double(), y.double()
    denominator = (x * x + y * y).sum()
    if denominator == 0:
        return 0.0
    sim = 2 * (x * y).sum() / denominator
    return (1 - sim).item()
```
**EN:** This block defines `_calc_diff`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_calc_diff`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 54-57: Function `_swap_fp4_nibbles`
```python
def _swap_fp4_nibbles(packed: torch.Tensor) -> torch.Tensor:
    return ((packed >> 4) | (packed << 4)).contiguous()
```
**EN:** This block defines `_swap_fp4_nibbles`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_swap_fp4_nibbles`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 58-61: Function `_fp4_lut`
```python
def _fp4_lut(device: torch.device) -> torch.Tensor:
    return torch.tensor(FP4_VALUE_LUT, dtype=torch.float32, device=device)
```
**EN:** This block defines `_fp4_lut`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_fp4_lut`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 62-76: Function `_unpack_fp4_bytes`
```python
def _unpack_fp4_bytes(packed: torch.Tensor) -> torch.Tensor:
    assert packed.dtype == torch.uint8
    lut = _fp4_lut(packed.device)

    def _decode(nibbles: torch.Tensor) -> torch.Tensor:
        values = lut[(nibbles & 0x7).to(torch.long)]
        return torch.where((nibbles & 0x8) != 0, -values, values)

    low = _decode(packed & 0x0F)
    high = _decode((packed & 0xF0) >> 4)
    return torch.stack((low, high), dim=-1).reshape(
        packed.shape[0], packed.shape[1] * 2
    )
```
**EN:** This block defines `_unpack_fp4_bytes`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_unpack_fp4_bytes`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 77-91: Function `_swizzled_to_linear`
```python
def _swizzled_to_linear(
    scales_swizzled: torch.Tensor,
    rows: int,
    cols: int,
) -> torch.Tensor:
    scales_swizzled = scales_swizzled.view(torch.float8_e4m3fn)
    row_tiles = (rows + 128 - 1) // 128
    tile_cols = BLOCK_SIZE * 4
    col_tiles = (cols + tile_cols - 1) // tile_cols
    tmp = scales_swizzled.reshape(1, row_tiles, col_tiles, 32, 4, 4)
    tmp = tmp.permute(0, 1, 4, 3, 2, 5)
    linear = tmp.reshape(row_tiles * 128, col_tiles * tile_cols // BLOCK_SIZE)
    return linear[:rows, : cols // BLOCK_SIZE]
```
**EN:** This block defines `_swizzled_to_linear`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_swizzled_to_linear`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 92-103: Function `_dequantize_nvfp4`
```python
def _dequantize_nvfp4(
    packed: torch.Tensor,
    scales_swizzled: torch.Tensor,
    global_scale: torch.Tensor,
) -> torch.Tensor:
    rows, packed_cols = packed.shape
    cols = packed_cols * 2
    unpacked = _unpack_fp4_bytes(packed).reshape(rows, cols // BLOCK_SIZE, BLOCK_SIZE)
    scales_linear = _swizzled_to_linear(scales_swizzled, rows, cols).to(torch.float32)
    return (unpacked * (scales_linear / global_scale).unsqueeze(-1)).reshape(rows, cols)
```
**EN:** This block defines `_dequantize_nvfp4`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_dequantize_nvfp4`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 104-116: Function `_quantize_weight_for_checkpoint`
```python
def _quantize_weight_for_checkpoint(
    weight: torch.Tensor, weight_global_scale: torch.Tensor
) -> tuple[torch.Tensor, torch.Tensor]:
    weight_fp4, weight_scale_linear = flashinfer.fp4_quantize(
        weight,
        weight_global_scale,
        is_sf_swizzled_layout=False,
    )
    if weight_scale_linear.dtype == torch.uint8:
        weight_scale_linear = weight_scale_linear.view(torch.float8_e4m3fn)
    return weight_fp4, weight_scale_linear.contiguous()
```
**EN:** This block defines `_quantize_weight_for_checkpoint`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_quantize_weight_for_checkpoint`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 117-131: Function `_set_diffusion_fp4_backend`
```python
def _set_diffusion_fp4_backend(
    monkeypatch: pytest.MonkeyPatch, backend: str | None
) -> None:
    if backend is None:
        monkeypatch.delenv(
            "SGLANG_DIFFUSION_FLASHINFER_FP4_GEMM_BACKEND", raising=False
        )
    else:
        monkeypatch.setenv("SGLANG_DIFFUSION_FLASHINFER_FP4_GEMM_BACKEND", backend)

    current_platform.__class__.get_modelopt_flashinfer_fp4_backend.cache_clear()
    current_platform.__class__.get_modelopt_fp4_gemm_op.cache_clear()
    diffusion_modelopt_quant._get_fp4_gemm_op.cache_clear()
```
**EN:** This block defines `_set_diffusion_fp4_backend`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_set_diffusion_fp4_backend`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 132-232: Function `_build_layer`
```python
def _build_layer(
    weight_fp4: torch.Tensor,
    weight_scale_linear: torch.Tensor,
    input_global_scale: torch.Tensor,
    weight_global_scale: torch.Tensor,
    *,
    weight_scale_device: torch.device | str | None = None,
) -> tuple[ModelOptFp4LinearMethod, torch.nn.Module]:
    output_size, input_size_half = weight_fp4.shape
    input_size = input_size_half * 2
    method = ModelOptFp4LinearMethod(
        ModelOptFp4Config(is_checkpoint_nvfp4_serialized=True, group_size=BLOCK_SIZE)
    )
    layer = torch.nn.Module()
    method.create_weights(
        layer,
        input_size_per_partition=input_size,
        output_partition_sizes=[output_size],
        input_size=input_size,
        output_size=output_size,
        params_dtype=DTYPE,
        weight_loader=lambda *args, **kwargs: None,
    )
    layer = layer.to(device=DEVICE)

    checkpoint_weight = _swap_fp4_nibbles(weight_fp4)
    layer.weight.data.copy_(checkpoint_weight)
    layer.input_scale.data.copy_(
        (1.0 / input_global_scale).reshape_as(layer.input_scale)
    )
    layer.weight_scale_2.data.copy_(
        (1.0 / weight_global_scale).reshape_as(layer.weight_scale_2)
    )
    layer.weight_scale.data.copy_(weight_scale_linear)
    if weight_scale_device is not None:
        layer.weight_scale = torch.nn.Parameter(
            layer.weight_scale.detach().to(weight_scale_device), requires_grad=False
        )

    method.process_weights_after_loading(layer)

    _, flashinfer_backend = current_platform.get_modelopt_fp4_gemm_op()
    if flashinfer_backend == "trtllm":
        expected_weight, _ = pad_nvfp4_weight(
            weight_fp4, n_alignment=128, k_alignment=0
        )
        expected_scale = weight_scale_linear
        if expected_scale.shape[0] != expected_weight.shape[0]:
            pad_n = expected_weight.shape[0] - expected_scale.shape[0]
            expected_scale = torch.nn.functional.pad(expected_scale, (0, 0, 0, pad_n))

        expected_padding_cols = 0
        if expected_scale.shape[1] % 4 != 0:
            padded_scale_k = ((expected_scale.shape[1] + 4 - 1) // 4) * 4
            pad_scale_k = padded_scale_k - expected_scale.shape[1]
            expected_scale = torch.nn.functional.pad(
                expected_scale, (0, pad_scale_k, 0, 0)
            )
            pad_weight_k = pad_scale_k * 8
            expected_weight = torch.nn.functional.pad(
                expected_weight, (0, pad_weight_k, 0, 0)
            )
            expected_padding_cols = pad_weight_k

        expected_weight = flashinfer.shuffle_matrix_a(
            expected_weight.view(torch.uint8), 128
        )
        expected_scale = (
            flashinfer.shuffle_matrix_sf_a(expected_scale.view(torch.uint8), 128)
            .reshape(expected_scale.shape)
            .view(torch.float8_e4m3fn)
        )

        assert torch.equal(layer.weight, expected_weight)
        assert torch.equal(
            layer.weight_scale_interleaved.view(torch.uint8),
            expected_scale.view(torch.uint8),
        )
        assert layer.weights_padding_cols == expected_padding_cols
    else:
# ...
```
**EN:** This block defines `_build_layer`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_build_layer`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 233-250: Function `_resolve_mode`
```python
def _resolve_mode(mode: str):
    if mode == "jit_cutlass":
        return scaled_fp4_quant, cutlass_scaled_fp4_mm, None
    if mode == "flashinfer2":
        return flashinfer.fp4_quantize, flashinfer.mm_fp4, "cudnn"
    if mode == "flashinfer_trtllm":
        return flashinfer.fp4_quantize, flashinfer.mm_fp4, "trtllm"
    raise ValueError(f"Unknown mode: {mode}")


@pytest.mark.skipif(
    not _nvfp4_supported(),
    reason="Diffusion NVFP4 scaled mm correctness requires Blackwell GPUs",
)
@pytest.mark.parametrize(
    "backend", [None, "flashinfer_trtllm"], ids=["default", "flashinfer_trtllm"]
)
@pytest.mark.parametrize("m,n,k", TEST_CASES)
```
**EN:** This block defines `_resolve_mode`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_resolve_mode`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 251-274: Function `test_checkpoint_processing`
```python
def test_checkpoint_processing(
    monkeypatch: pytest.MonkeyPatch, backend: str | None, m: int, n: int, k: int
) -> None:
    _set_diffusion_fp4_backend(monkeypatch, backend)
    generator = torch.Generator(device=DEVICE)
    generator.manual_seed(20260404 + m + n + k)

    weight = torch.randn((n, k), device=DEVICE, dtype=DTYPE, generator=generator)
    input_global_scale = torch.tensor(512.0, device=DEVICE, dtype=torch.float32)
    weight_global_scale = _make_global_scale(weight)
    weight_fp4, weight_scale_linear = _quantize_weight_for_checkpoint(
        weight, weight_global_scale
    )

    _build_layer(
        weight_fp4, weight_scale_linear, input_global_scale, weight_global_scale
    )


@pytest.mark.skipif(
    not _nvfp4_supported(),
    reason="Diffusion NVFP4 scaled mm correctness requires Blackwell GPUs",
)
@pytest.mark.parametrize("mode", ["jit_cutlass", "flashinfer2"])
```
**EN:** This block defines `test_checkpoint_processing`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_checkpoint_processing`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 275-326: Function `test_flux2_shape_correctness`
```python
def test_flux2_shape_correctness(mode: str) -> None:
    m, n, k = FLUX2_PROJECTION_SHAPE
    quantize_op, gemm_op, gemm_backend = _resolve_mode(mode)
    generator = torch.Generator(device=DEVICE)
    generator.manual_seed(20260404 + m + n + k)

    x = torch.randn((m, k), device=DEVICE, dtype=DTYPE, generator=generator)
    weight = torch.randn((n, k), device=DEVICE, dtype=DTYPE, generator=generator)
    input_global_scale = _make_global_scale(x)
    weight_global_scale = _make_global_scale(weight)
    alpha = (1.0 / (input_global_scale * weight_global_scale)).to(torch.float32)

    x_fp4, x_scale_swizzled = quantize_op(x, input_global_scale)
    weight_fp4, weight_scale_swizzled = quantize_op(weight, weight_global_scale)
    if x_scale_swizzled.dtype == torch.uint8:
        x_scale_swizzled = x_scale_swizzled.view(torch.float8_e4m3fn)
    if weight_scale_swizzled.dtype == torch.uint8:
        weight_scale_swizzled = weight_scale_swizzled.view(torch.float8_e4m3fn)

    expected = torch.matmul(
        _dequantize_nvfp4(x_fp4, x_scale_swizzled, input_global_scale),
        _dequantize_nvfp4(weight_fp4, weight_scale_swizzled, weight_global_scale).t(),
    )

    if gemm_backend is None:
        actual = gemm_op(
            x_fp4,
            weight_fp4,
            x_scale_swizzled,
            weight_scale_swizzled,
            alpha,
            DTYPE,
        )
    else:
        actual = gemm_op(
            x_fp4,
            weight_fp4.t(),
            x_scale_swizzled,
            weight_scale_swizzled.t(),
            alpha,
            DTYPE,
            backend=gemm_backend,
        )

    diff = _calc_diff(actual, expected.to(dtype=DTYPE))
    assert diff < DEEPGEMM_FP4_MAX_DIFF, f"{mode=}, {m=}, {n=}, {k=}, {diff=:.6f}"


@pytest.mark.skipif(
    not _nvfp4_supported(),
    reason="Diffusion NVFP4 scaled mm correctness requires Blackwell GPUs",
)
```
**EN:** This block defines `test_flux2_shape_correctness`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_flux2_shape_correctness`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 327-372: Function `test_flux2_shape_correctness_flashinfer_trtllm`
```python
def test_flux2_shape_correctness_flashinfer_trtllm(
    monkeypatch: pytest.MonkeyPatch,
) -> None:
    _set_diffusion_fp4_backend(monkeypatch, "flashinfer_trtllm")

    m, n, k = FLUX2_PROJECTION_SHAPE
    generator = torch.Generator(device=DEVICE)
    generator.manual_seed(20260404 + m + n + k + 17)

    x = torch.randn((m, k), device=DEVICE, dtype=DTYPE, generator=generator)
    weight = torch.randn((n, k), device=DEVICE, dtype=DTYPE, generator=generator)
    input_global_scale = _make_global_scale(x)
    weight_global_scale = _make_global_scale(weight)
    weight_fp4, weight_scale_linear = _quantize_weight_for_checkpoint(
        weight, weight_global_scale
    )

    method, layer = _build_layer(
        weight_fp4, weight_scale_linear, input_global_scale, weight_global_scale
    )
    actual = method.apply(layer, x)

    x_fp4, x_scale_swizzled = flashinfer.fp4_quantize(x, input_global_scale)
    weight_fp4_ref, weight_scale_swizzled = flashinfer.fp4_quantize(
        weight, weight_global_scale
    )
    if x_scale_swizzled.dtype == torch.uint8:
        x_scale_swizzled = x_scale_swizzled.view(torch.float8_e4m3fn)
    if weight_scale_swizzled.dtype == torch.uint8:
        weight_scale_swizzled = weight_scale_swizzled.view(torch.float8_e4m3fn)

    expected = torch.matmul(
        _dequantize_nvfp4(x_fp4, x_scale_swizzled, input_global_scale),
        _dequantize_nvfp4(
            weight_fp4_ref, weight_scale_swizzled, weight_global_scale
        ).t(),
    )

    diff = _calc_diff(actual, expected.to(dtype=DTYPE))
    assert diff < DEEPGEMM_FP4_MAX_DIFF, f"{m=}, {n=}, {k=}, {diff=:.6f}"


@pytest.mark.skipif(
    not _nvfp4_supported(),
    reason="Diffusion NVFP4 scaled mm correctness requires Blackwell GPUs",
)
```
**EN:** This block defines `test_flux2_shape_correctness_flashinfer_trtllm`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_flux2_shape_correctness_flashinfer_trtllm`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 373-399: Function `test_checkpoint_processing_flashinfer_trtllm_cpu_weight_scale`
```python
def test_checkpoint_processing_flashinfer_trtllm_cpu_weight_scale(
    monkeypatch: pytest.MonkeyPatch,
) -> None:
    _set_diffusion_fp4_backend(monkeypatch, "flashinfer_trtllm")

    m, n, k = FLUX2_PROJECTION_SHAPE
    generator = torch.Generator(device=DEVICE)
    generator.manual_seed(20260413 + m + n + k)

    weight = torch.randn((n, k), device=DEVICE, dtype=DTYPE, generator=generator)
    input_global_scale = torch.tensor(512.0, device=DEVICE, dtype=torch.float32)
    weight_global_scale = _make_global_scale(weight)
    weight_fp4, weight_scale_linear = _quantize_weight_for_checkpoint(
        weight, weight_global_scale
    )

    _build_layer(
        weight_fp4,
        weight_scale_linear,
        input_global_scale,
        weight_global_scale,
        weight_scale_device="cpu",
    )


if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v", "-s"]))
```
**EN:** This block defines `test_checkpoint_processing_flashinfer_trtllm_cpu_weight_scale`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_checkpoint_processing_flashinfer_trtllm_cpu_weight_scale`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `sys`
- `flashinfer`
- `pytest`
- `torch`
- `sglang.jit_kernel.nvfp4 -> cutlass_scaled_fp4_mm`
- `sglang.multimodal_gen.runtime.layers.quantization -> (`
- `sglang.multimodal_gen.runtime.layers.quantization.modelopt_quant -> (`
- `sglang.multimodal_gen.runtime.platforms -> current_platform`
- `sglang.srt.layers.quantization.modelopt_quant -> pad_nvfp4_weight`
- `sglang.test.ci.ci_register -> register_cuda_ci`
