# test_activation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/test_activation.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import sys". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import sys”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: Imports and module setup
```python
import sys

import pytest
import torch
import torch.nn.functional as F

from sglang.jit_kernel.activation import SUPPORTED_ACTIVATIONS, run_activation
from sglang.jit_kernel.utils import get_ci_test_range
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=20, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=30, suite="nightly-kernel-1-gpu", nightly=True)


OPS = SUPPORTED_ACTIVATIONS
DTYPES = [torch.float16, torch.bfloat16, torch.float32]
SHAPES = get_ci_test_range(
    full_range=[
        (7, 16),
        (83, 1024),
        (3, 5, 16),
        (2, 3, 512),
        (1, 17, 4096),
        *[(2**x, 2048) for x in range(0, 15, 2)],
        *[(2**x, 65536) for x in range(0, 5, 2)],
    ],
    ci_range=[(7, 16), (2, 3, 512)],
)
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 31-43: Function `_reference`
```python
def _reference(op_name: str, x: torch.Tensor) -> torch.Tensor:
    d = x.shape[-1] // 2
    lhs = x[..., :d].float()
    rhs = x[..., d:]
    if op_name == "silu":
        act = F.silu(lhs)
    elif op_name == "gelu":
        act = F.gelu(lhs, approximate="none")
    else:
        act = F.gelu(lhs, approximate="tanh")
    return act.to(dtype=x.dtype) * rhs
```
**EN:** This block defines `_reference`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_reference`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 44-52: Function `_tolerances`
```python
def _tolerances(dtype: torch.dtype) -> tuple[float, float]:
    if dtype == torch.float32:
        return 1e-4, 1e-4
    return 1e-2, 1e-2


@pytest.mark.parametrize("op_name", OPS)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("shape", SHAPES)
```
**EN:** This block defines `_tolerances`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_tolerances`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 53-65: Function `test_activation_correctness`
```python
def test_activation_correctness(
    op_name: str, dtype: torch.dtype, shape: tuple[int, ...]
) -> None:
    x = torch.randn(shape, dtype=dtype, device="cuda")
    out = run_activation(op_name, x, None)
    expected = _reference(op_name, x)
    atol, rtol = _tolerances(dtype)
    torch.testing.assert_close(out, expected, atol=atol, rtol=rtol)


@pytest.mark.parametrize("op_name", OPS)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("shape", SHAPES)
```
**EN:** This block defines `test_activation_correctness`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_activation_correctness`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 66-88: Function `test_activation_out_param`
```python
def test_activation_out_param(
    op_name: str, dtype: torch.dtype, shape: tuple[int, ...]
) -> None:
    x = torch.randn(shape, dtype=dtype, device="cuda")
    out = torch.empty(shape[:-1] + (shape[-1] // 2,), dtype=dtype, device="cuda")
    result = run_activation(op_name, x, out)
    assert result is out
    expected = _reference(op_name, x)
    atol, rtol = _tolerances(dtype)
    torch.testing.assert_close(out, expected, atol=atol, rtol=rtol)


FILTER_SHAPES = get_ci_test_range(
    full_range=[(83, 1024), (256, 2048), (1024, 4096)],
    ci_range=[(83, 1024)],
)
EXPERT_STEPS = [1, 16]


@pytest.mark.parametrize("op_name", OPS)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("shape", FILTER_SHAPES)
@pytest.mark.parametrize("expert_step", EXPERT_STEPS)
```
**EN:** This block defines `test_activation_out_param`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_activation_out_param`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 89-130: Function `test_activation_filter_expert`
```python
def test_activation_filter_expert(
    op_name: str,
    dtype: torch.dtype,
    shape: tuple[int, int],
    expert_step: int,
) -> None:
    """expert_ids[token // expert_step] == -1 must leave the output row untouched."""
    num_tokens = shape[0]
    x = torch.randn(shape, dtype=dtype, device="cuda")
    # Pre-fill out with a sentinel so we can detect untouched rows.
    sentinel = float("nan")
    out = torch.full(
        shape[:-1] + (shape[-1] // 2,),
        sentinel,
        dtype=dtype,
        device="cuda",
    )

    num_groups = (num_tokens + expert_step - 1) // expert_step
    expert_ids = torch.randint(
        low=0, high=8, size=(num_groups,), dtype=torch.int32, device="cuda"
    )
    skip_mask = torch.rand(num_groups, device="cuda") < 0.4
    expert_ids[skip_mask] = -1

    result = run_activation(op_name, x, out, expert_ids, expert_step)
    assert result is out

    token_skip = skip_mask[torch.arange(num_tokens, device="cuda") // expert_step]
    expected = _reference(op_name, x)
    atol, rtol = _tolerances(dtype)

    kept = ~token_skip
    if kept.any():
        torch.testing.assert_close(out[kept], expected[kept], atol=atol, rtol=rtol)
    if token_skip.any():
        assert torch.isnan(
            out[token_skip]
        ).all(), "filter_expert kernel touched rows whose expert_id is -1"


@pytest.mark.parametrize("op_name", OPS)
```
**EN:** This block defines `test_activation_filter_expert`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_activation_filter_expert`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 131-146: Function `test_activation_filter_expert_all_skipped`
```python
def test_activation_filter_expert_all_skipped(op_name: str) -> None:
    """If every expert id is -1, the output must be left entirely untouched."""
    shape = (32, 512)
    x = torch.randn(shape, dtype=torch.bfloat16, device="cuda")
    out = torch.full(
        shape[:-1] + (shape[-1] // 2,),
        float("nan"),
        dtype=torch.bfloat16,
        device="cuda",
    )
    expert_ids = torch.full((shape[0],), -1, dtype=torch.int32, device="cuda")
    run_activation(op_name, x, out, expert_ids, 1)
    assert torch.isnan(out).all()


@pytest.mark.parametrize("op_name", OPS)
```
**EN:** This block defines `test_activation_filter_expert_all_skipped`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_activation_filter_expert_all_skipped`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 147-159: Function `test_activation_filter_expert_none_skipped`
```python
def test_activation_filter_expert_none_skipped(op_name: str) -> None:
    """No -1 in expert_ids must yield bit-identical output to the unfiltered path."""
    shape = (64, 512)
    dtype = torch.bfloat16
    x = torch.randn(shape, dtype=dtype, device="cuda")
    expert_ids = torch.zeros((shape[0],), dtype=torch.int32, device="cuda")
    out_filtered = run_activation(op_name, x, None, expert_ids, 1)
    out_unfiltered = run_activation(op_name, x, None)
    torch.testing.assert_close(out_filtered, out_unfiltered, atol=0.0, rtol=0.0)


if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v", "-s"]))
```
**EN:** This block defines `test_activation_filter_expert_none_skipped`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_activation_filter_expert_none_skipped`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `sys`
- `pytest`
- `torch`
- `torch.nn.functional as F`
- `sglang.jit_kernel.activation -> SUPPORTED_ACTIVATIONS`
- `sglang.jit_kernel.utils -> get_ci_test_range`
- `sglang.test.ci.ci_register -> register_cuda_ci`
