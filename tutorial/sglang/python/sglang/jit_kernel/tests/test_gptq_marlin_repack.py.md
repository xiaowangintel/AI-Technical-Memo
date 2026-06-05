# test_gptq_marlin_repack.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/test_gptq_marlin_repack.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import sys". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import sys”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-39: Imports and module setup
```python
import sys

import pytest
import torch
from sgl_kernel.scalar_type import scalar_types

from sglang.jit_kernel.gptq_marlin_repack import gptq_marlin_repack
from sglang.srt.layers.quantization.utils import (
    gptq_quantize_weights,
    pack_rows,
    sort_weights,
)
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_marlin_utils import get_weight_perm, marlin_weights

register_cuda_ci(est_time=16, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=120, suite="nightly-kernel-1-gpu", nightly=True)

MARLIN_K_CHUNKS = [128]
MARLIN_N_CHUNKS = [64, 256]

MNK_FACTORS = [
    (1, 1, 1),
    (1, 4, 8),
    (1, 7, 5),
    (13, 17, 67),
    (26, 37, 13),
    (67, 13, 11),
    (257, 13, 11),
    (658, 13, 11),
]


@pytest.mark.parametrize("k_chunk", MARLIN_K_CHUNKS)
@pytest.mark.parametrize("n_chunk", MARLIN_N_CHUNKS)
@pytest.mark.parametrize("quant_type", [scalar_types.uint4b8])
@pytest.mark.parametrize("group_size", [-1, 32, 64, 128])
@pytest.mark.parametrize("act_order", [False, True])
@pytest.mark.parametrize("mnk_factors", MNK_FACTORS)
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 40-96: Function `test_gptq_marlin_repack`
```python
def test_gptq_marlin_repack(
    k_chunk, n_chunk, quant_type, group_size, act_order, mnk_factors
):
    m_factor, n_factor, k_factor = mnk_factors

    size_k = k_chunk * k_factor
    size_n = n_chunk * n_factor

    # Filter act_order
    if act_order:
        if group_size == -1:
            return
        if group_size == size_k:
            return

    # Normalize group_size
    if group_size == -1:
        group_size = size_k
    assert group_size <= size_k

    if size_k % group_size != 0:
        pytest.skip("size_k must be divisible by group_size")

    # Create input
    b_weight = torch.randn((size_k, size_n), dtype=torch.float16, device="cuda")

    # Quantize (and apply act_order if provided)
    w_ref, q_w, s, g_idx, rand_perm = gptq_quantize_weights(
        b_weight, quant_type, group_size, act_order
    )

    q_w_gptq = pack_rows(q_w, quant_type.size_bits, size_k, size_n)

    # For act_order, sort the "weights" and "g_idx" so that group ids are
    # increasing
    sort_indices = torch.empty(0, dtype=torch.int, device=b_weight.device)
    if act_order:
        q_w, g_idx, sort_indices = sort_weights(q_w, g_idx)

    marlin_layout_perm = get_weight_perm(quant_type.size_bits)
    q_w_marlin_ref = marlin_weights(
        q_w, size_k, size_n, quant_type.size_bits, marlin_layout_perm
    )

    # Run JIT repack kernel
    jit_output = gptq_marlin_repack(
        q_w_gptq, sort_indices, size_k, size_n, quant_type.size_bits
    )

    torch.cuda.synchronize()

    # JIT should match the reference (computed from CPU marlin_weights)
    torch.testing.assert_close(jit_output, q_w_marlin_ref)


if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v", "-s"]))
```
**EN:** This block defines `test_gptq_marlin_repack`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_gptq_marlin_repack`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `sys`
- `pytest`
- `torch`
- `sgl_kernel.scalar_type -> scalar_types`
- `sglang.jit_kernel.gptq_marlin_repack -> gptq_marlin_repack`
- `sglang.srt.layers.quantization.utils -> (`
- `sglang.test.ci.ci_register -> register_cuda_ci`
- `sglang.test.test_marlin_utils -> get_weight_perm`
