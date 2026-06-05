# test_qknorm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/test_qknorm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import itertools". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import itertools”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Imports and module setup
```python
import itertools
import sys

import pytest
import torch
import triton

from sglang.jit_kernel.utils import get_ci_test_range
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=37, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=148, suite="nightly-kernel-1-gpu", nightly=True)
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 15-29: Imports and module setup
```python
def sglang_aot_qknorm(
    q: torch.Tensor,
    k: torch.Tensor,
    q_weight: torch.Tensor,
    k_weight: torch.Tensor,
) -> None:
    from sgl_kernel import rmsnorm

    head_dim = q.shape[-1]
    q = q.view(-1, head_dim)
    k = k.view(-1, head_dim)
    rmsnorm(q, q_weight, out=q)
    rmsnorm(k, k_weight, out=k)
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 30-40: Imports and module setup
```python
def sglang_jit_qknorm(
    q: torch.Tensor,
    k: torch.Tensor,
    q_weight: torch.Tensor,
    k_weight: torch.Tensor,
) -> None:
    from sglang.jit_kernel.norm import fused_inplace_qknorm

    fused_inplace_qknorm(q, k, q_weight, k_weight)
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 41-53: Imports and module setup
```python
def flashinfer_qknorm(
    q: torch.Tensor,
    k: torch.Tensor,
    q_weight: torch.Tensor,
    k_weight: torch.Tensor,
) -> None:
    from flashinfer.norm import rmsnorm

    rmsnorm(q, q_weight, out=q)
    rmsnorm(k, k_weight, out=k)


@torch.compile()
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 54-86: Function `torch_impl_qknorm`
```python
def torch_impl_qknorm(
    q: torch.Tensor,
    k: torch.Tensor,
    q_weight: torch.Tensor,
    k_weight: torch.Tensor,
    eps: float = 1e-6,
) -> None:
    q_mean = q.float().pow(2).mean(dim=-1, keepdim=True)
    k_mean = k.float().pow(2).mean(dim=-1, keepdim=True)
    q_norm = (q_mean + eps).rsqrt()
    k_norm = (k_mean + eps).rsqrt()
    q.copy_(q.float() * q_norm * q_weight.float())
    k.copy_(k.float() * k_norm * k_weight.float())


BS_LIST = [2**n for n in range(0, 14)]
BS_LIST += [x + 1 + i for i, x in enumerate(BS_LIST)]
BS_LIST = get_ci_test_range(BS_LIST, [1, 9, 256, 4109])
N_K_LIST = get_ci_test_range([2, 4], [2, 4])
N_Q_LIST = get_ci_test_range([8, 16], [8, 16])
HEAD_DIM_LIST = get_ci_test_range([64, 128, 256, 512, 1024], [64, 256, 1024])
DEVICE = "cuda"
DTYPE = torch.bfloat16

# NOTE(dark): sgl_kernel use flashinfer template, which is bitwise identical to flashinfer impl.
# However, sgl-jit-kernel, flashinfer, torch_impl, may have small numerical differences.
# so we allow a small rel/abs tolerance in correctness test.


@pytest.mark.parametrize(
    "batch_size,n_k,n_q,head_dim",
    list(itertools.product(BS_LIST, N_K_LIST, N_Q_LIST, HEAD_DIM_LIST)),
)
```
**EN:** This block defines `torch_impl_qknorm`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `torch_impl_qknorm`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 87-101: Function `test_qknorm`
```python
def test_qknorm(batch_size: int, n_k: int, n_q: int, head_dim: int) -> None:
    q = torch.randn(batch_size, n_q, head_dim, device=DEVICE, dtype=DTYPE)
    k = torch.randn(batch_size, n_k, head_dim, device=DEVICE, dtype=DTYPE)
    q_weight = torch.randn(head_dim, device=DEVICE, dtype=DTYPE)
    k_weight = torch.randn(head_dim, device=DEVICE, dtype=DTYPE)
    q_k_aot = (q.clone(), k.clone())
    q_k_jit = (q.clone(), k.clone())
    sglang_aot_qknorm(q_k_aot[0], q_k_aot[1], q_weight, k_weight)
    sglang_jit_qknorm(q_k_jit[0], q_k_jit[1], q_weight, k_weight)
    triton.testing.assert_close(q_k_aot[0], q_k_jit[0], atol=1e-2, rtol=1e-2)
    triton.testing.assert_close(q_k_aot[1], q_k_jit[1], atol=1e-2, rtol=1e-2)


if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v", "-s"]))
```
**EN:** This block defines `test_qknorm`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_qknorm`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

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
- `sgl_kernel -> rmsnorm`
- `sglang.jit_kernel.norm -> fused_inplace_qknorm`
- `flashinfer.norm -> rmsnorm`
