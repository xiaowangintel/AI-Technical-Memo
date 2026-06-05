# test_nvfp4_blockwise_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/test_nvfp4_blockwise_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import sys". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import sys”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Imports and module setup
```python
import sys

import pytest
import torch

from sglang.jit_kernel.nvfp4 import (
    cutlass_fp4_group_mm,
    scaled_fp4_experts_quant,
    scaled_fp4_quant,
)
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=5, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=120, suite="nightly-kernel-1-gpu", nightly=True)

FLOAT4_E2M1_MAX = 6.0
FLOAT8_E4M3_MAX = torch.finfo(torch.float8_e4m3fn).max
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 20-23: Function `_nvfp4_supported`
```python
def _nvfp4_supported() -> bool:
    return torch.cuda.is_available() and torch.cuda.get_device_capability() >= (10, 0)
```
**EN:** This block defines `_nvfp4_supported`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_nvfp4_supported`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 24-27: Function `_round_up`
```python
def _round_up(x: int, y: int) -> int:
    return ((x + y - 1) // y) * y
```
**EN:** This block defines `_round_up`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_round_up`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 28-36: Function `_build_expert_offsets`
```python
def _build_expert_offsets(
    m_per_expert: list[int], device: torch.device
) -> torch.Tensor:
    offsets = [0]
    for m in m_per_expert:
        offsets.append(offsets[-1] + m)
    return torch.tensor(offsets, dtype=torch.int32, device=device)
```
**EN:** This block defines `_build_expert_offsets`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_build_expert_offsets`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 37-49: Function `_build_blockscale_offsets`
```python
def _build_blockscale_offsets(
    m_per_expert: list[int], device: torch.device
) -> torch.Tensor:
    offsets = [0]
    for m in m_per_expert:
        offsets.append(offsets[-1] + _round_up(m, 128))
    return torch.tensor(offsets, dtype=torch.int32, device=device)


@pytest.mark.skipif(
    not _nvfp4_supported(), reason="NVFP4 requires compute capability >= 10.0"
)
@pytest.mark.parametrize("dtype", [torch.float16, torch.bfloat16])
```
**EN:** This block defines `_build_blockscale_offsets`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_build_blockscale_offsets`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 50-137: Function `test_nvfp4_blockwise_moe_grouped_mm`
```python
def test_nvfp4_blockwise_moe_grouped_mm(dtype: torch.dtype) -> None:
    torch.manual_seed(0)
    device = torch.device("cuda")

    num_experts = 4
    m_per_expert = [33, 17, 48, 29]
    n = 256
    k = 128

    expert_offsets_full = _build_expert_offsets(m_per_expert, device)
    blockscale_offsets_full = _build_blockscale_offsets(m_per_expert, device)

    total_m = int(expert_offsets_full[-1].item())
    a = torch.randn((total_m, k), device=device, dtype=dtype) * 0.1
    b = torch.randn((num_experts, n, k), device=device, dtype=dtype) * 0.1

    a_global_scale = torch.empty((num_experts,), device=device, dtype=torch.float32)
    for i in range(num_experts):
        start = int(expert_offsets_full[i].item())
        end = int(expert_offsets_full[i + 1].item())
        amax = a[start:end].abs().max().to(torch.float32)
        a_global_scale[i] = FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX / amax

    b_global_scale = torch.empty((num_experts,), device=device, dtype=torch.float32)
    for i in range(num_experts):
        bmax = b[i].abs().max().to(torch.float32)
        b_global_scale[i] = FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX / bmax

    a_fp4, a_blockscale = scaled_fp4_experts_quant(
        a,
        a_global_scale,
        expert_offsets_full,
        blockscale_offsets_full,
        topk=1,
    )

    b_fp4 = torch.empty((num_experts, n, k // 2), device=device, dtype=torch.uint8)
    b_blockscale = torch.empty(
        (num_experts, _round_up(n, 128), _round_up(k // 16, 4)),
        device=device,
        dtype=torch.float8_e4m3fn,
    )
    for i in range(num_experts):
        b_fp4_i, b_scale_i = scaled_fp4_quant(b[i], b_global_scale[i])
        b_fp4[i].copy_(b_fp4_i)
        b_blockscale[i].copy_(b_scale_i)

    alphas = (1.0 / (a_global_scale * b_global_scale)).to(torch.float32)

    params = {
        "ab_strides": torch.full((num_experts,), k, dtype=torch.int64, device=device),
        "c_strides": torch.full((num_experts,), n, dtype=torch.int64, device=device),
        "problem_sizes": torch.tensor(
            [[m, n, k] for m in m_per_expert], dtype=torch.int32, device=device
        ),
        "expert_offsets": expert_offsets_full[:-1].contiguous(),
        "blockscale_offsets": blockscale_offsets_full[:-1].contiguous(),
        "a_ptrs": torch.empty((num_experts,), dtype=torch.int64, device=device),
        "b_ptrs": torch.empty((num_experts,), dtype=torch.int64, device=device),
        "out_ptrs": torch.empty((num_experts,), dtype=torch.int64, device=device),
        "a_scales_ptrs": torch.empty((num_experts,), dtype=torch.int64, device=device),
        "b_scales_ptrs": torch.empty((num_experts,), dtype=torch.int64, device=device),
        "alpha_ptrs": torch.empty((num_experts,), dtype=torch.int64, device=device),
        "layout_sfa": torch.empty((num_experts, 5), dtype=torch.int64, device=device),
        "layout_sfb": torch.empty((num_experts, 5), dtype=torch.int64, device=device),
    }

    out = cutlass_fp4_group_mm(
        a_fp4,
        b_fp4,
        a_blockscale,
        b_blockscale,
        alphas,
        dtype,
        params,
    )

    ref = torch.empty((total_m, n), device=device, dtype=dtype)
    for i in range(num_experts):
        start = int(expert_offsets_full[i].item())
# ...
```
**EN:** This block defines `test_nvfp4_blockwise_moe_grouped_mm`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_nvfp4_blockwise_moe_grouped_mm`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `sys`
- `pytest`
- `torch`
- `sglang.jit_kernel.nvfp4 -> (`
- `sglang.test.ci.ci_register -> register_cuda_ci`
