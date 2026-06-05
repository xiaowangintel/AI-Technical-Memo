# test_grouped_topk.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/test_grouped_topk.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import itertools". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import itertools”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: Imports and module setup
```python
import itertools
import sys

import pytest
import torch

from sglang.jit_kernel.grouped_topk import grouped_topk as jit_grouped_topk
from sglang.jit_kernel.utils import get_ci_test_range
from sglang.srt.layers.moe.topk import biased_grouped_topk_impl
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=30, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=120, suite="nightly-kernel-1-gpu", nightly=True)


CORRECTNESS_CASES = get_ci_test_range(
    full_range=list(
        itertools.product(
            [1, 17, 128],
            [16, 32, 64, 128, 192, 256, 384, 512],
            [1, 2, 3, 4, 5, 6, 7, 8],
        )
    ),
    ci_range=[
        (1, 16, 3),  # smallest non-power-of-two topk
        (17, 128, 6),  # Nemotron-3-Nano shape that exposed the bug
        (128, 192, 8),  # Hunyuan-3 shape, power-of-two topk sanity case
        (33, 512, 7),  # largest expert-count tier with non-power-of-two topk
    ],
)
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 33-42: Function `_make_inputs`
```python
def _make_inputs(num_tokens: int, num_experts: int, seed: int):
    torch.manual_seed(seed)
    hidden_states = torch.empty((num_tokens, 1), dtype=torch.float32, device="cuda")
    gating_output = torch.randn(
        (num_tokens, num_experts), dtype=torch.float32, device="cuda"
    )
    correction_bias = torch.randn(num_experts, dtype=torch.float32, device="cuda") * 0.1
    return hidden_states, gating_output, correction_bias
```
**EN:** This block defines `_make_inputs`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_make_inputs`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 43-53: Function `_scatter_by_expert`
```python
def _scatter_by_expert(
    weights: torch.Tensor, ids: torch.Tensor, num_experts: int
) -> torch.Tensor:
    dense = torch.zeros(
        (weights.shape[0], num_experts), dtype=torch.float32, device=weights.device
    )
    dense.scatter_(1, ids.long(), weights)
    return dense


@pytest.mark.parametrize("num_tokens,num_experts,topk", CORRECTNESS_CASES)
```
**EN:** This block defines `_scatter_by_expert`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_scatter_by_expert`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 54-98: Function `test_grouped_topk_renormalize_matches_reference`
```python
def test_grouped_topk_renormalize_matches_reference(
    num_tokens: int, num_experts: int, topk: int
) -> None:
    hidden_states, gating_output, correction_bias = _make_inputs(
        num_tokens, num_experts, seed=1000 + num_experts * 10 + topk
    )
    scaling_factor = 2.826 if (num_experts, topk) == (192, 8) else 1.0

    topk_weights, topk_ids = jit_grouped_topk(
        gating_output,
        correction_bias,
        1,
        1,
        topk,
        True,
        scaling_factor,
    )
    ref_weights, ref_ids = biased_grouped_topk_impl(
        hidden_states,
        gating_output,
        correction_bias,
        topk,
        True,
        1,
        1,
        routed_scaling_factor=scaling_factor,
        apply_routed_scaling_factor_on_output=True,
    )
    torch.cuda.synchronize()

    torch.testing.assert_close(
        _scatter_by_expert(topk_weights, topk_ids, num_experts),
        _scatter_by_expert(ref_weights, ref_ids, num_experts),
        rtol=1e-5,
        atol=1e-6,
    )
    torch.testing.assert_close(
        topk_weights.sum(dim=-1),
        torch.full((num_tokens,), scaling_factor, dtype=torch.float32, device="cuda"),
        rtol=1e-5,
        atol=1e-6,
    )


@pytest.mark.parametrize("topk", [3, 5, 6, 7])
```
**EN:** This block defines `test_grouped_topk_renormalize_matches_reference`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_grouped_topk_renormalize_matches_reference`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 99-139: Function `test_grouped_topk_non_power_of_two_renormalize`
```python
def test_grouped_topk_non_power_of_two_renormalize(topk: int) -> None:
    hidden_states, gating_output, correction_bias = _make_inputs(
        num_tokens=64, num_experts=128, seed=2000 + topk
    )

    topk_weights, topk_ids = jit_grouped_topk(
        gating_output,
        correction_bias,
        1,
        1,
        topk,
        True,
        1.0,
    )
    ref_weights, ref_ids = biased_grouped_topk_impl(
        hidden_states,
        gating_output,
        correction_bias,
        topk,
        True,
        1,
        1,
        routed_scaling_factor=1.0,
        apply_routed_scaling_factor_on_output=True,
    )
    torch.cuda.synchronize()

    torch.testing.assert_close(
        _scatter_by_expert(topk_weights, topk_ids, 128),
        _scatter_by_expert(ref_weights, ref_ids, 128),
        rtol=1e-5,
        atol=1e-6,
    )
    torch.testing.assert_close(
        topk_weights.sum(dim=-1),
        torch.ones((64,), dtype=torch.float32, device="cuda"),
        rtol=1e-5,
        atol=1e-6,
    )
```
**EN:** This block defines `test_grouped_topk_non_power_of_two_renormalize`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_grouped_topk_non_power_of_two_renormalize`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 140-175: Function `test_grouped_topk_negative_choice_scores_match_reference`
```python
def test_grouped_topk_negative_choice_scores_match_reference() -> None:
    hidden_states, gating_output, correction_bias = _make_inputs(
        num_tokens=64, num_experts=128, seed=23758
    )
    correction_bias.fill_(-2.0)

    topk_weights, topk_ids = jit_grouped_topk(
        gating_output,
        correction_bias,
        1,
        1,
        6,
        True,
        1.0,
    )
    ref_weights, ref_ids = biased_grouped_topk_impl(
        hidden_states,
        gating_output,
        correction_bias,
        6,
        True,
        1,
        1,
        routed_scaling_factor=1.0,
        apply_routed_scaling_factor_on_output=True,
    )
    torch.cuda.synchronize()

    torch.testing.assert_close(
        _scatter_by_expert(topk_weights, topk_ids, 128),
        _scatter_by_expert(ref_weights, ref_ids, 128),
        rtol=1e-5,
        atol=1e-6,
    )
```
**EN:** This block defines `test_grouped_topk_negative_choice_scores_match_reference`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_grouped_topk_negative_choice_scores_match_reference`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 176-210: Function `test_grouped_topk_without_renormalize_matches_reference`
```python
def test_grouped_topk_without_renormalize_matches_reference() -> None:
    hidden_states, gating_output, correction_bias = _make_inputs(
        num_tokens=64, num_experts=128, seed=3006
    )

    topk_weights, topk_ids = jit_grouped_topk(
        gating_output,
        correction_bias,
        1,
        1,
        6,
        False,
        1.0,
    )
    ref_weights, ref_ids = biased_grouped_topk_impl(
        hidden_states,
        gating_output,
        correction_bias,
        6,
        False,
        1,
        1,
    )
    torch.cuda.synchronize()

    torch.testing.assert_close(
        _scatter_by_expert(topk_weights, topk_ids, 128),
        _scatter_by_expert(ref_weights, ref_ids, 128),
        rtol=1e-5,
        atol=1e-6,
    )


if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v", "-s"]))
```
**EN:** This block defines `test_grouped_topk_without_renormalize_matches_reference`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_grouped_topk_without_renormalize_matches_reference`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `itertools`
- `sys`
- `pytest`
- `torch`
- `sglang.jit_kernel.grouped_topk -> grouped_topk`
- `sglang.jit_kernel.utils -> get_ci_test_range`
- `sglang.srt.layers.moe.topk -> biased_grouped_topk_impl`
- `sglang.test.ci.ci_register -> register_cuda_ci`
