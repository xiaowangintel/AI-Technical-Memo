# test_rmsnorm_hf.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/test_rmsnorm_hf.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: """"Tests for the JIT rmsnorm_hf kernel (HF LlamaRMSNorm semantics)."""". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“"""Tests for the JIT rmsnorm_hf kernel (HF LlamaRMSNorm semantics)."""”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Imports and module setup
```python
"""Tests for the JIT rmsnorm_hf kernel (HF LlamaRMSNorm semantics)."""

import itertools
import sys

import pytest
import torch

from sglang.jit_kernel.rmsnorm_hf import (
    is_supported_rmsnorm_hf_hidden_size,
    rmsnorm_hf,
)
from sglang.jit_kernel.utils import get_ci_test_range
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=30, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=120, suite="nightly-kernel-1-gpu", nightly=True)

EPS = 1e-5
DEVICE = "cuda"
DTYPES = [torch.float16, torch.bfloat16]
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 24-31: Function `hf_rmsnorm_reference`
```python
def hf_rmsnorm_reference(x: torch.Tensor, w: torch.Tensor, eps: float) -> torch.Tensor:
    """HF LlamaRMSNorm: normalize fp32, cast normalized x to dtype, then multiply weight."""
    x_fp32 = x.to(torch.float32)
    variance = x_fp32.pow(2).mean(-1, keepdim=True)
    x_normed = x_fp32 * torch.rsqrt(variance + eps)
    return w * x_normed.to(x.dtype)
```
**EN:** This block defines `hf_rmsnorm_reference`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `hf_rmsnorm_reference`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 32-55: Function `sgl_rmsnorm_reference`
```python
def sgl_rmsnorm_reference(x: torch.Tensor, w: torch.Tensor, eps: float) -> torch.Tensor:
    """Old sgl_kernel.rmsnorm semantics — weight multiply in fp32, cast at the end."""
    x_fp32 = x.to(torch.float32)
    variance = x_fp32.pow(2).mean(-1, keepdim=True)
    x_normed = x_fp32 * torch.rsqrt(variance + eps)
    return (x_normed * w.to(torch.float32)).to(x.dtype)


BS_LIST = get_ci_test_range(
    [1, 2, 4, 7, 16, 64, 128, 512, 1024, 4096],
    [1, 16, 1024],
)
HIDDEN_SIZE_LIST = get_ci_test_range(
    # Warp-kernel shapes (q/k RMSNorm head dims) + CTA-kernel shapes.
    [32, 64, 96, 128, 256, 512, 1024, 2048, 3072, 4096, 8192, 16384],
    [128, 512, 4096, 16384],
)


@pytest.mark.parametrize(
    "batch_size,hidden_size",
    list(itertools.product(BS_LIST, HIDDEN_SIZE_LIST)),
)
@pytest.mark.parametrize("dtype", DTYPES)
```
**EN:** This block defines `sgl_rmsnorm_reference`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `sgl_rmsnorm_reference`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 56-71: Function `test_rmsnorm_hf_correctness`
```python
def test_rmsnorm_hf_correctness(
    batch_size: int, hidden_size: int, dtype: torch.dtype
) -> None:
    torch.manual_seed(0)
    x = torch.randn(batch_size, hidden_size, device=DEVICE, dtype=dtype)
    w = torch.randn(hidden_size, device=DEVICE, dtype=dtype)
    out = rmsnorm_hf(x, w, EPS)
    ref = hf_rmsnorm_reference(x, w, EPS)
    # Loose atol — the kernel's block-reduce order differs from PyTorch's
    # `mean`, producing ~1 fp16 ULP of drift on some shapes.
    # The SGL-semantics regression guard below is what catches the cast-order
    # bug this PR fixes; it's reduction-order-invariant.
    torch.testing.assert_close(out, ref, atol=1e-2, rtol=1e-2)


@pytest.mark.parametrize("dtype", DTYPES)
```
**EN:** This block defines `test_rmsnorm_hf_correctness`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_rmsnorm_hf_correctness`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 72-84: Function `test_rmsnorm_hf_out_param`
```python
def test_rmsnorm_hf_out_param(dtype: torch.dtype) -> None:
    torch.manual_seed(0)
    x = torch.randn(8, 4096, device=DEVICE, dtype=dtype)
    w = torch.randn(4096, device=DEVICE, dtype=dtype)
    out = torch.empty_like(x)
    result = rmsnorm_hf(x, w, EPS, out=out)
    assert result.data_ptr() == out.data_ptr()
    torch.testing.assert_close(
        out, hf_rmsnorm_reference(x, w, EPS), atol=1e-2, rtol=1e-2
    )


@pytest.mark.parametrize("dtype", DTYPES)
```
**EN:** This block defines `test_rmsnorm_hf_out_param`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_rmsnorm_hf_out_param`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 85-103: Function `test_rmsnorm_hf_matches_hf_not_sgl`
```python
def test_rmsnorm_hf_matches_hf_not_sgl(dtype: torch.dtype) -> None:
    """Regression guard: kernel must follow HF (cast-before-mul), not the old
    sgl_kernel.rmsnorm semantics (fp32-mul-then-cast). Reduction-order drift
    prevents a bit-exact assert against HF, so instead assert the kernel is
    strictly closer to HF than to the SGL reference."""
    torch.manual_seed(0)
    x = torch.randn(64, 4096, device=DEVICE, dtype=dtype)
    w = torch.randn(4096, device=DEVICE, dtype=dtype)
    out = rmsnorm_hf(x, w, EPS).float()
    hf_ref = hf_rmsnorm_reference(x, w, EPS).float()
    sgl_ref = sgl_rmsnorm_reference(x, w, EPS).float()
    assert (sgl_ref - hf_ref).abs().max() > 0, "inputs don't exercise the difference"
    diff_hf = (out - hf_ref).abs().max().item()
    diff_sgl = (out - sgl_ref).abs().max().item()
    assert (
        diff_hf < diff_sgl
    ), f"kernel closer to SGL than HF (hf={diff_hf}, sgl={diff_sgl})"
```
**EN:** This block defines `test_rmsnorm_hf_matches_hf_not_sgl`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_rmsnorm_hf_matches_hf_not_sgl`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 104-130: Function `test_rmsnorm_hf_empty_input`
```python
def test_rmsnorm_hf_empty_input() -> None:
    """Empty input must short-circuit: the C++ launcher rejects num_tokens=0."""
    x = torch.empty(0, 4096, device=DEVICE, dtype=torch.float16)
    w = torch.randn(4096, device=DEVICE, dtype=torch.float16)
    out = rmsnorm_hf(x, w, EPS)
    assert out.shape == x.shape and out.numel() == 0


@pytest.mark.parametrize(
    ("hidden_size", "expected"),
    [
        (16, False),
        (32, True),
        (64, True),
        (96, True),
        (128, True),
        (256, True),
        (288, True),
        (384, True),
        (500, False),
        (512, True),
        (3072, True),
        (4096, True),
        (8192, True),
        (4097, False),
    ],
)
```
**EN:** This block defines `test_rmsnorm_hf_empty_input`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_rmsnorm_hf_empty_input`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 131-136: Function `test_is_supported_hidden_size`
```python
def test_is_supported_hidden_size(hidden_size: int, expected: bool) -> None:
    assert is_supported_rmsnorm_hf_hidden_size(hidden_size) is expected


if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v", "-s"]))
```
**EN:** This block defines `test_is_supported_hidden_size`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_is_supported_hidden_size`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `itertools`
- `sys`
- `pytest`
- `torch`
- `sglang.jit_kernel.rmsnorm_hf -> (`
- `sglang.jit_kernel.utils -> get_ci_test_range`
- `sglang.test.ci.ci_register -> register_cuda_ci`
