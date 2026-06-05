# test_mla_kv_pack_quantize_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/test_mla_kv_pack_quantize_fp8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import sys". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import sys”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Imports and module setup
```python
import sys

import pytest
import torch

from sglang.jit_kernel.mla_kv_pack_quantize_fp8 import mla_kv_pack_quantize_fp8
from sglang.jit_kernel.utils import get_ci_test_range
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=60, suite="base-b-kernel-unit-1-gpu-large")

DEVICE = "cuda"

SHAPES = get_ci_test_range(
    [(128, 64, 128), (64, 32, 64)],
    [(128, 64, 128)],
)
NUM_HEADS = get_ci_test_range([8, 16, 32, 64], [16, 32])
BATCH_SIZES = get_ci_test_range(
    [1, 4, 17, 64, 257, 1024, 4096, 16384],
    [1, 64, 1024, 16384],
)
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 25-46: Function `_ref`
```python
def _ref(k_nope, k_pe, v, k_scale_inv, v_scale_inv, fp8_dtype):
    s, h, qk_nope = k_nope.shape
    qk_rope = k_pe.shape[-1]
    v_head = v.shape[-1]
    if k_pe.dim() == 3:
        k_pe = k_pe.squeeze(1)

    k_bf16 = torch.empty(
        (s, h, qk_nope + qk_rope), dtype=k_nope.dtype, device=k_nope.device
    )
    k_bf16[..., :qk_nope] = k_nope
    k_bf16[..., qk_nope:] = k_pe.unsqueeze(1).expand(-1, h, -1)

    k_fp8 = (k_bf16.float() * k_scale_inv).to(fp8_dtype)
    v_fp8 = (v.float() * v_scale_inv).to(fp8_dtype)
    return k_fp8, v_fp8


@pytest.mark.parametrize("dtype", [torch.bfloat16, torch.float16])
@pytest.mark.parametrize("shape", SHAPES)
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("batch_size", BATCH_SIZES)
```
**EN:** This block defines `_ref`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_ref`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 47-68: Function `test_correctness`
```python
def test_correctness(dtype, shape, num_heads, batch_size):
    qk_nope, qk_rope, v_head = shape

    torch.manual_seed(0)
    k_nope = torch.randn((batch_size, num_heads, qk_nope), dtype=dtype, device=DEVICE)
    k_pe = torch.randn((batch_size, 1, qk_rope), dtype=dtype, device=DEVICE)
    v = torch.randn((batch_size, num_heads, v_head), dtype=dtype, device=DEVICE)

    k_scale_inv = 0.7
    v_scale_inv = 1.3

    k_fp8, v_fp8 = mla_kv_pack_quantize_fp8(
        k_nope, k_pe, v, k_scale_inv=k_scale_inv, v_scale_inv=v_scale_inv
    )

    k_ref, v_ref = _ref(k_nope, k_pe, v, k_scale_inv, v_scale_inv, torch.float8_e4m3fn)

    torch.testing.assert_close(k_fp8.float(), k_ref.float(), rtol=1e-2, atol=0.5)
    torch.testing.assert_close(v_fp8.float(), v_ref.float(), rtol=1e-2, atol=0.5)


@pytest.mark.parametrize("dtype", [torch.bfloat16])
```
**EN:** This block defines `test_correctness`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_correctness`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 69-87: Function `test_strided_inputs`
```python
def test_strided_inputs(dtype):
    s, h = 16, 32
    qk_nope, qk_rope, v_head = 128, 64, 128

    full = torch.randn(
        (s, h, qk_nope * 2), dtype=dtype, device=DEVICE, requires_grad=False
    )
    k_nope = full[..., qk_nope:]
    assert k_nope.stride(-1) == 1

    k_pe = torch.randn((s, 1, qk_rope), dtype=dtype, device=DEVICE)
    v = torch.randn((s, h, v_head), dtype=dtype, device=DEVICE)

    k_fp8, v_fp8 = mla_kv_pack_quantize_fp8(k_nope, k_pe, v)
    k_ref, v_ref = _ref(k_nope, k_pe, v, 1.0, 1.0, torch.float8_e4m3fn)
    torch.testing.assert_close(k_fp8.float(), k_ref.float(), rtol=1e-2, atol=0.5)
    torch.testing.assert_close(v_fp8.float(), v_ref.float(), rtol=1e-2, atol=0.5)
```
**EN:** This block defines `test_strided_inputs`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_strided_inputs`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 88-104: Function `test_kpe_2d_accepted`
```python
def test_kpe_2d_accepted():
    s, h = 8, 16
    qk_nope, qk_rope, v_head = 128, 64, 128
    dtype = torch.bfloat16

    k_nope = torch.randn((s, h, qk_nope), dtype=dtype, device=DEVICE)
    k_pe = torch.randn((s, qk_rope), dtype=dtype, device=DEVICE)
    v = torch.randn((s, h, v_head), dtype=dtype, device=DEVICE)

    k_fp8, v_fp8 = mla_kv_pack_quantize_fp8(k_nope, k_pe, v)
    k_ref, v_ref = _ref(k_nope, k_pe.unsqueeze(1), v, 1.0, 1.0, torch.float8_e4m3fn)
    torch.testing.assert_close(k_fp8.float(), k_ref.float(), rtol=1e-2, atol=0.5)
    torch.testing.assert_close(v_fp8.float(), v_ref.float(), rtol=1e-2, atol=0.5)


if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v", "-s"]))
```
**EN:** This block defines `test_kpe_2d_accepted`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_kpe_2d_accepted`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `sys`
- `pytest`
- `torch`
- `sglang.jit_kernel.mla_kv_pack_quantize_fp8 -> mla_kv_pack_quantize_fp8`
- `sglang.jit_kernel.utils -> get_ci_test_range`
- `sglang.test.ci.ci_register -> register_cuda_ci`
