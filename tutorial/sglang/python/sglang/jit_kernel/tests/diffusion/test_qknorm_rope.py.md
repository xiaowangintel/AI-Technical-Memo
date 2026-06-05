# test_qknorm_rope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/diffusion/test_qknorm_rope.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import itertools". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import itertools”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Imports and module setup
```python
import itertools
import sys

import pytest
import torch
import triton

from sglang.jit_kernel.utils import get_ci_test_range
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=44, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=176, suite="nightly-kernel-1-gpu", nightly=True)

DEVICE = "cuda"
DTYPE = torch.bfloat16
MAX_SEQ_LEN = 131072
ROPE_BASE = 10000.0
ATOL = 8e-2
RTOL = 1e-2
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 22-38: Function `create_cos_sin_cache`
```python
def create_cos_sin_cache(
    rotary_dim: int,
    max_position: int = MAX_SEQ_LEN,
    base: float = ROPE_BASE,
) -> torch.Tensor:
    inv_freq = 1.0 / (
        base
        ** (
            torch.arange(0, rotary_dim, 2, dtype=torch.float32, device=DEVICE)
            / rotary_dim
        )
    )
    t = torch.arange(max_position, dtype=torch.float32, device=DEVICE)
    freqs = torch.einsum("i,j->ij", t, inv_freq)
    return torch.cat((freqs.cos(), freqs.sin()), dim=-1)
```
**EN:** This block defines `create_cos_sin_cache`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `create_cos_sin_cache`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 39-62: Imports and module setup
```python
def split_qknorm_rope(
    q: torch.Tensor,
    k: torch.Tensor,
    q_weight: torch.Tensor,
    k_weight: torch.Tensor,
    cos_sin_cache: torch.Tensor,
    positions: torch.Tensor,
    is_neox: bool,
) -> None:
    from flashinfer.rope import apply_rope_with_cos_sin_cache_inplace

    from sglang.jit_kernel.norm import fused_inplace_qknorm

    fused_inplace_qknorm(q, k, q_weight, k_weight)
    apply_rope_with_cos_sin_cache_inplace(
        positions=positions.long(),
        query=q.view(q.shape[0], -1),
        key=k.view(k.shape[0], -1),
        head_size=q.shape[-1],
        cos_sin_cache=cos_sin_cache,
        is_neox=is_neox,
    )
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 63-111: Imports and module setup
```python
def fused_qknorm_rope(
    q: torch.Tensor,
    k: torch.Tensor,
    q_weight: torch.Tensor,
    k_weight: torch.Tensor,
    cos_sin_cache: torch.Tensor,
    positions: torch.Tensor,
    is_neox: bool,
) -> None:
    from sglang.jit_kernel.diffusion.qknorm_rope import fused_inplace_qknorm_rope

    fused_inplace_qknorm_rope(
        q,
        k,
        q_weight,
        k_weight,
        cos_sin_cache,
        positions,
        is_neox=is_neox,
        rope_dim=cos_sin_cache.shape[-1],
    )


BS_LIST = [2**n for n in range(13)]
BS_LIST += [x + 1 for x in BS_LIST]
BS_LIST = get_ci_test_range(BS_LIST, [1, 9, 129, 257, 2049, 4097])
HEADS_LIST = get_ci_test_range([8, 16, 24, 32], [8, 24])
HEAD_DIM_LIST = get_ci_test_range([64, 128, 256], [64, 128, 256])
IS_NEOX_LIST = [False, True]
POSITION_DTYPES = [torch.int32, torch.int64]
ROPE_DIM_CHOICES = {
    64: [64],
    128: [64, 128],
    256: [64, 128, 256],
}


@pytest.mark.parametrize(
    "batch_size,num_heads,head_dim,is_neox,position_dtype",
    list(
        itertools.product(
            BS_LIST,
            HEADS_LIST,
            HEAD_DIM_LIST,
            IS_NEOX_LIST,
            POSITION_DTYPES,
        )
    ),
)
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 112-153: Function `test_qknorm_rope`
```python
def test_qknorm_rope(
    batch_size: int,
    num_heads: int,
    head_dim: int,
    is_neox: bool,
    position_dtype: torch.dtype,
) -> None:
    rope_dims = ROPE_DIM_CHOICES[head_dim]
    for rope_dim in rope_dims:
        if is_neox:
            elems_per_thread = head_dim // 32
            rotary_lanes = rope_dim // elems_per_thread
            if rotary_lanes < 2 or rotary_lanes & (rotary_lanes - 1):
                continue

        q = torch.randn(batch_size, num_heads, head_dim, device=DEVICE, dtype=DTYPE)
        k = torch.randn(batch_size, num_heads, head_dim, device=DEVICE, dtype=DTYPE)
        q_weight = torch.randn(head_dim, device=DEVICE, dtype=DTYPE)
        k_weight = torch.randn(head_dim, device=DEVICE, dtype=DTYPE)
        positions = torch.randint(
            0, MAX_SEQ_LEN, (batch_size,), device=DEVICE, dtype=position_dtype
        )
        cos_sin_cache = create_cos_sin_cache(rope_dim)

        q_ref, k_ref = q.clone(), k.clone()
        q_fused, k_fused = q.clone(), k.clone()

        split_qknorm_rope(
            q_ref, k_ref, q_weight, k_weight, cos_sin_cache, positions, is_neox
        )
        fused_qknorm_rope(
            q_fused, k_fused, q_weight, k_weight, cos_sin_cache, positions, is_neox
        )

        # The split baseline mixes a separate BF16 qknorm kernel with FlashInfer RoPE,
        # which differs from the fused path by about one BF16 rounding step on H200.
        triton.testing.assert_close(q_ref, q_fused, atol=ATOL, rtol=RTOL)
        triton.testing.assert_close(k_ref, k_fused, atol=ATOL, rtol=RTOL)


if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v", "-s"]))
```
**EN:** This block defines `test_qknorm_rope`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_qknorm_rope`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `itertools`
- `sys`
- `pytest`
- `torch`
- `triton`
- `sglang.jit_kernel.utils -> get_ci_test_range`
- `sglang.test.ci.ci_register -> register_cuda_ci`
- `flashinfer.rope -> apply_rope_with_cos_sin_cache_inplace`
- `sglang.jit_kernel.norm -> fused_inplace_qknorm`
- `sglang.jit_kernel.diffusion.qknorm_rope -> fused_inplace_qknorm_rope`
