# test_awq_marlin_moe_repack.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/test_awq_marlin_moe_repack.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import sys". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import sys”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and module setup
```python
import sys

import numpy as np
import pytest
import torch
from sgl_kernel.scalar_type import scalar_types

from sglang.jit_kernel.awq_marlin_repack import (
    awq_marlin_moe_repack as jit_awq_marlin_moe_repack,
)
from sglang.srt.layers.quantization.utils import pack_cols, quantize_weights
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=10, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=120, suite="nightly-kernel-1-gpu", nightly=True)
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 18-26: Function `_has_aot_awq_marlin_moe_repack`
```python
def _has_aot_awq_marlin_moe_repack() -> bool:
    return hasattr(torch.ops.sgl_kernel, "awq_marlin_moe_repack") and hasattr(
        torch.ops.sgl_kernel.awq_marlin_moe_repack, "default"
    )


AOT_AVAILABLE = _has_aot_awq_marlin_moe_repack()
```
**EN:** This block defines `_has_aot_awq_marlin_moe_repack`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_has_aot_awq_marlin_moe_repack`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 27-51: Function `awq_pack`
```python
def awq_pack(
    q_w: torch.Tensor,
    num_bits: int,
    size_k: int,
    size_n: int,
):
    assert q_w.shape == (size_k, size_n)

    if num_bits == 4:
        interleave = np.array([0, 2, 4, 6, 1, 3, 5, 7])
    elif num_bits == 8:
        interleave = np.array([0, 2, 1, 3])
    else:
        raise Exception("num_bits must be 4 or 8, got {}".format(num_bits))

    q_w = q_w.reshape((-1, len(interleave)))[:, interleave].ravel()
    q_w = q_w.reshape((-1, size_n)).contiguous()

    return pack_cols(q_w, num_bits, size_k, size_n)


@pytest.mark.parametrize("num_bits", [4])
@pytest.mark.parametrize("num_experts", [2, 4, 8])
@pytest.mark.parametrize("k_tiles,n_tiles", [(1, 1), (2, 2), (4, 4)])
@pytest.mark.parametrize("group_size", [16, 32])
```
**EN:** This block defines `awq_pack`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `awq_pack`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 52-92: Function `test_awq_marlin_moe_repack_jit_vs_aot`
```python
def test_awq_marlin_moe_repack_jit_vs_aot(
    num_bits, num_experts, k_tiles, n_tiles, group_size
):
    if not AOT_AVAILABLE:
        pytest.skip("sgl_kernel AOT not available")

    tile_k, tile_n = 16, 64
    size_k = k_tiles * tile_k
    size_n = n_tiles * tile_n
    pack_factor = 32 // num_bits

    # Create per-expert AWQ-packed weights
    b_q_weight = torch.empty(
        (num_experts, size_k, size_n // pack_factor),
        dtype=torch.int32,
        device="cuda",
    )
    for e in range(num_experts):
        b_weight = torch.randn((size_k, size_n), dtype=torch.float16, device="cuda")
        w_ref, q_w, s, zp = quantize_weights(
            b_weight, scalar_types.uint4, group_size, zero_points=True
        )
        b_q_weight[e] = awq_pack(q_w, num_bits, size_k, size_n)

    perm = torch.empty((num_experts, 0), dtype=torch.int32, device="cuda")

    out_jit = jit_awq_marlin_moe_repack(b_q_weight, perm, size_k, size_n, num_bits)
    out_aot = torch.ops.sgl_kernel.awq_marlin_moe_repack.default(
        b_q_weight, perm, size_k, size_n, num_bits
    )

    torch.cuda.synchronize()

    # Bitwise equality
    torch.testing.assert_close(out_jit, out_aot, rtol=0, atol=0)


@pytest.mark.parametrize("num_bits", [4])
@pytest.mark.parametrize("num_experts", [2, 4])
@pytest.mark.parametrize("k_tiles,n_tiles", [(1, 1), (2, 2)])
@pytest.mark.parametrize("group_size", [16, 32])
```
**EN:** This block defines `test_awq_marlin_moe_repack_jit_vs_aot`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_awq_marlin_moe_repack_jit_vs_aot`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 93-125: Function `test_awq_marlin_moe_repack_shape`
```python
def test_awq_marlin_moe_repack_shape(
    num_bits, num_experts, k_tiles, n_tiles, group_size
):
    tile_k, tile_n = 16, 64
    size_k = k_tiles * tile_k
    size_n = n_tiles * tile_n
    pack_factor = 32 // num_bits

    # Create per-expert AWQ-packed weights
    b_q_weight = torch.empty(
        (num_experts, size_k, size_n // pack_factor),
        dtype=torch.int32,
        device="cuda",
    )
    for e in range(num_experts):
        b_weight = torch.randn((size_k, size_n), dtype=torch.float16, device="cuda")
        w_ref, q_w, s, zp = quantize_weights(
            b_weight, scalar_types.uint4, group_size, zero_points=True
        )
        b_q_weight[e] = awq_pack(q_w, num_bits, size_k, size_n)

    perm = torch.empty((num_experts, 0), dtype=torch.int32, device="cuda")

    out = jit_awq_marlin_moe_repack(b_q_weight, perm, size_k, size_n, num_bits)
    torch.cuda.synchronize()

    assert out.is_cuda and out.dtype == torch.int32
    expected_shape = (num_experts, size_k // 16, size_n * (num_bits // 2))
    assert list(out.shape) == list(expected_shape)


if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v", "-s"]))
```
**EN:** This block defines `test_awq_marlin_moe_repack_shape`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_awq_marlin_moe_repack_shape`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `sys`
- `numpy as np`
- `pytest`
- `torch`
- `sgl_kernel.scalar_type -> scalar_types`
- `sglang.jit_kernel.awq_marlin_repack -> (`
- `sglang.srt.layers.quantization.utils -> pack_cols`
- `sglang.test.ci.ci_register -> register_cuda_ci`
