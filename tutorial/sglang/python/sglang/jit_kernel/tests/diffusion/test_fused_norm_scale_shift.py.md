# test_fused_norm_scale_shift.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/diffusion/test_fused_norm_scale_shift.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import sys". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import sys”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-44: Imports and module setup
```python
import sys
from typing import Optional, Tuple

import pytest
import torch
from einops import rearrange
from torch import Tensor

from sglang.jit_kernel.diffusion.cutedsl.scale_residual_norm_scale_shift import (
    fused_norm_scale_shift,
    fused_scale_residual_norm_scale_shift,
    validate_scale_shift,
)
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=28, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=120, suite="nightly-kernel-1-gpu", nightly=True)

DEVICE = "cuda"
SHAPE_MAP = {
    "1": lambda B, S, F, D: (1,),
    "D": lambda B, S, F, D: (D,),
    "1D": lambda B, S, F, D: (1, D),
    "BD": lambda B, S, F, D: (B, D),
    "11D": lambda B, S, F, D: (1, 1, D),
    "B1D": lambda B, S, F, D: (B, 1, D),
    "1SD": lambda B, S, F, D: (1, S, D),
    "BSD": lambda B, S, F, D: (B, S, D),
    "BF1D": lambda B, S, F, D: (B, F, 1, D),
}
SHAPES = [
    # (B, S, F, D)
    (1, 115200, 1, 3072),  # Hunyuan
    (1, 32760, 1, 1536),  # Wan
    (1, 6, 1, 3072),  # Qwen
    (1, 1024, 8, 3072),
    (4, 512, 16, 3072),
]
DTYPES = [torch.float16, torch.bfloat16, torch.float32]
NORM_TYPES = ["layer", "rms"]
AFFINE_MODES = ["D", "NAT"]
INDEX_MODES = ["BSD", "1", "1SD", "BD", "B1D", "D", "1D", "11D", "BF1D"]
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 45-49: Function `_tol`
```python
def _tol(dtype: torch.dtype):
    return 1e-5 if dtype == torch.float32 else 5e-2


@pytest.fixture(autouse=True)
```
**EN:** This block defines `_tol`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_tol`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 50-55: Function `cuda_setup`
```python
def cuda_setup():
    if not torch.cuda.is_available():
        pytest.skip("CUDA required")
    torch.cuda.manual_seed(0)
```
**EN:** This block defines `cuda_setup`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `cuda_setup`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 56-68: Function `_apply_scale_shift`
```python
def _apply_scale_shift(y: Tensor, scale: Tensor, shift: Tensor) -> Tensor:
    if scale.ndim == 4:
        num_frame = scale.shape[1]
        return rearrange(
            rearrange(y, "b (f l) d -> b f l d", f=num_frame) * (1 + scale) + shift,
            "b f l d -> b (f l) d",
        )
    else:
        scale = rearrange(scale, "b d -> b 1 d") if scale.ndim == 2 else scale
        shift = rearrange(shift, "b d -> b 1 d") if shift.ndim == 2 else shift
        return y * (1 + scale) + shift
```
**EN:** This block defines `_apply_scale_shift`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_apply_scale_shift`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 69-88: Function `fused_norm_scale_shift_ref`
```python
def fused_norm_scale_shift_ref(
    x: Tensor,
    weight: Optional[Tensor],
    bias: Optional[Tensor],
    scale: Tensor,
    shift: Tensor,
    norm_type: str,
    eps: float,
) -> Tensor:
    original_dtype = x.dtype
    x, weight, bias, scale, shift = (
        v.float() if v is not None else v for v in [x, weight, bias, scale, shift]
    )
    if norm_type == "layer":
        norm = torch.layer_norm(x, x.shape[-1:], eps=eps, weight=weight, bias=bias)
    else:
        norm = torch.rms_norm(x, x.shape[-1:], eps=eps, weight=weight)
    return _apply_scale_shift(norm, scale, shift).to(original_dtype)
```
**EN:** This block defines `fused_norm_scale_shift_ref`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `fused_norm_scale_shift_ref`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 89-122: Function `fused_scale_residual_norm_scale_shift_ref`
```python
def fused_scale_residual_norm_scale_shift_ref(
    residual: Tensor,
    x: Tensor,
    gate: Optional[Tensor] | int,
    weight: Optional[Tensor],
    bias: Optional[Tensor],
    scale: Tensor,
    shift: Tensor,
    norm_type: str,
    eps: float,
):
    original_dtype = x.dtype
    residual, x, gate, weight, bias, scale, shift = (
        v.float() if isinstance(v, Tensor) else v
        for v in [residual, x, gate, weight, bias, scale, shift]
    )
    if isinstance(gate, int):
        x = residual + gate * x
    else:
        if gate.ndim == 4:
            num_frame = gate.shape[1]
            x_fld = rearrange(x, "b (f l) d -> b f l d", f=num_frame)
            x = residual + rearrange(x_fld * gate, "b f l d -> b (f l) d")
        else:
            gate = rearrange(gate, "b d -> b 1 d") if gate.ndim == 2 else gate
            x = residual + gate * x
    if norm_type == "layer":
        norm = torch.layer_norm(x, x.shape[-1:], eps=eps, weight=weight, bias=bias)
    else:
        norm = torch.rms_norm(x, x.shape[-1:], eps=eps, weight=weight)
    y_ref = _apply_scale_shift(norm, scale, shift)
    return y_ref.to(original_dtype), x.to(original_dtype)
```
**EN:** This block defines `fused_scale_residual_norm_scale_shift_ref`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `fused_scale_residual_norm_scale_shift_ref`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 123-128: Function `_make_tensor`
```python
def _make_tensor(index_mode: str, shape: Tuple, dtype: torch.dtype):
    if index_mode == "NAT":
        return None
    return torch.randn(*SHAPE_MAP[index_mode](*shape), device=DEVICE, dtype=dtype)
```
**EN:** This block defines `_make_tensor`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_make_tensor`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 129-139: Function `test_validate_scale_shift_rejects_non_divisible_frames`
```python
def test_validate_scale_shift_rejects_non_divisible_frames():
    with pytest.raises(ValueError, match=r"S\(10\) must be divisible by F\(4\)"):
        validate_scale_shift(
            torch.empty((1, 4, 1, 256), device=DEVICE, dtype=torch.float16),
            1,
            10,
            256,
        )


@torch.no_grad()
```
**EN:** This block defines `test_validate_scale_shift_rejects_non_divisible_frames`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_validate_scale_shift_rejects_non_divisible_frames`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 140-162: Function `run_norm_scale_shift`
```python
def run_norm_scale_shift(
    shape=SHAPES[0],
    dtype=DTYPES[0],
    affine_dtype=DTYPES[0],
    scale_dtype=DTYPES[0],
    shift_dtype=DTYPES[0],
    norm_type=NORM_TYPES[0],
    affine_mode=AFFINE_MODES[0],
    scale_mode="BSD",
    shift_mode="BSD",
    eps=1e-5,
):
    x = _make_tensor("BSD", shape, dtype)
    weight = _make_tensor(affine_mode, shape, affine_dtype)
    bias = _make_tensor(affine_mode, shape, affine_dtype)
    scale = _make_tensor(scale_mode, shape, scale_dtype)
    shift = _make_tensor(shift_mode, shape, shift_dtype)
    y_dev = fused_norm_scale_shift(x, weight, bias, scale, shift, norm_type, eps)
    y_ref = fused_norm_scale_shift_ref(x, weight, bias, scale, shift, norm_type, eps)
    torch.testing.assert_close(y_dev, y_ref, atol=_tol(dtype), rtol=_tol(dtype))


@torch.no_grad()
```
**EN:** This block defines `run_norm_scale_shift`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `run_norm_scale_shift`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 163-193: Function `run_scale_resi_norm_scale_shift`
```python
def run_scale_resi_norm_scale_shift(
    shape=SHAPES[0],
    dtype=DTYPES[0],
    affine_dtype=DTYPES[0],
    scale_dtype=DTYPES[0],
    shift_dtype=DTYPES[0],
    norm_type=NORM_TYPES[0],
    affine_mode=AFFINE_MODES[0],
    gate_mode="B1D",
    scale_mode="BSD",
    shift_mode="BSD",
    eps=1e-5,
):
    residual = _make_tensor("BSD", shape, dtype)
    x = _make_tensor("BSD", shape, dtype)
    gate = _make_tensor(gate_mode, shape, dtype)
    weight = _make_tensor(affine_mode, shape, affine_dtype)
    bias = _make_tensor(affine_mode, shape, affine_dtype)
    scale = _make_tensor(scale_mode, shape, scale_dtype)
    shift = _make_tensor(shift_mode, shape, shift_dtype)
    y_dev, res_dev = fused_scale_residual_norm_scale_shift(
        residual, x, gate, weight, bias, scale, shift, norm_type, eps
    )
    y_ref, res_ref = fused_scale_residual_norm_scale_shift_ref(
        residual, x, gate, weight, bias, scale, shift, norm_type, eps
    )
    torch.testing.assert_close(y_dev, y_ref, atol=_tol(dtype), rtol=_tol(dtype))
    torch.testing.assert_close(res_dev, res_ref, atol=_tol(dtype), rtol=_tol(dtype))


@pytest.mark.parametrize("norm_type", NORM_TYPES)
```
**EN:** This block defines `run_scale_resi_norm_scale_shift`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `run_scale_resi_norm_scale_shift`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 194-219: Class `TestFusedNormScaleShift`
```python
class TestFusedNormScaleShift:
    @pytest.mark.parametrize("shape", SHAPES)
    @pytest.mark.parametrize("dtype", DTYPES)
    def test_shape_dtype(self, shape, dtype, norm_type):
        run_norm_scale_shift(shape=shape, dtype=dtype, norm_type=norm_type)

    @pytest.mark.parametrize("dtype", DTYPES)
    def test_dtype_0(self, dtype, norm_type):
        run_norm_scale_shift(affine_dtype=dtype, norm_type=norm_type)

    @pytest.mark.parametrize("dtype", DTYPES)
    def test_dtype_1(self, dtype, norm_type):
        run_norm_scale_shift(scale_dtype=dtype, shift_dtype=dtype, norm_type=norm_type)

    @pytest.mark.parametrize("affine_mode", AFFINE_MODES)
    def test_normtype_affine(self, affine_mode, norm_type):
        run_norm_scale_shift(affine_mode=affine_mode, norm_type=norm_type)

    @pytest.mark.parametrize("index_mode", INDEX_MODES)
    def test_index_mode(self, index_mode, norm_type):
        run_norm_scale_shift(
            scale_mode=index_mode, shift_mode=index_mode, norm_type=norm_type
        )


@pytest.mark.parametrize("norm_type", NORM_TYPES)
```
**EN:** This block declares the `TestFusedNormScaleShift` class and establishes the behavior or state it encapsulates. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段声明了 `TestFusedNormScaleShift` 类，并建立其封装的行为或状态。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 220-252: Class `TestFusedScaleResidualNormScaleShift`
```python
class TestFusedScaleResidualNormScaleShift:
    @pytest.mark.parametrize("shape", SHAPES)
    @pytest.mark.parametrize("dtype", DTYPES)
    def test_shape_dtype(self, shape, dtype, norm_type):
        run_scale_resi_norm_scale_shift(shape=shape, dtype=dtype, norm_type=norm_type)

    @pytest.mark.parametrize("dtype", DTYPES)
    def test_dtype_0(self, dtype, norm_type):
        run_scale_resi_norm_scale_shift(affine_dtype=dtype, norm_type=norm_type)

    @pytest.mark.parametrize("dtype", DTYPES)
    def test_dtype_1(self, dtype, norm_type):
        run_scale_resi_norm_scale_shift(
            scale_dtype=dtype, shift_dtype=dtype, norm_type=norm_type
        )

    @pytest.mark.parametrize("affine_mode", AFFINE_MODES)
    def test_normtype_affine(self, affine_mode, norm_type):
        run_scale_resi_norm_scale_shift(affine_mode=affine_mode, norm_type=norm_type)

    @pytest.mark.parametrize("index_mode", INDEX_MODES)
    def test_scale_shift_index_mode(self, index_mode, norm_type):
        run_scale_resi_norm_scale_shift(
            scale_mode=index_mode, shift_mode=index_mode, norm_type=norm_type
        )

    @pytest.mark.parametrize("index_mode", INDEX_MODES)
    def test_gate_index_mode(self, index_mode, norm_type):
        run_scale_resi_norm_scale_shift(gate_mode=index_mode, norm_type=norm_type)


if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v", "-s"]))
```
**EN:** This block declares the `TestFusedScaleResidualNormScaleShift` class and establishes the behavior or state it encapsulates. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段声明了 `TestFusedScaleResidualNormScaleShift` 类，并建立其封装的行为或状态。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `sys`
- `typing -> Optional`
- `pytest`
- `torch`
- `einops -> rearrange`
- `torch -> Tensor`
- `sglang.jit_kernel.diffusion.cutedsl.scale_residual_norm_scale_shift -> (`
- `sglang.test.ci.ci_register -> register_cuda_ci`
