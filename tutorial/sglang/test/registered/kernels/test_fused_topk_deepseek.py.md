# test_fused_topk_deepseek.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/kernels/test_fused_topk_deepseek.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates fused topk deepseek behavior in SGLang's kernels area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 kernels 领域中与 fused topk deepseek 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: module imports and dependencies / 模块导入与依赖
```python
import sys

import pytest
import torch

from sglang.srt.layers.moe.topk import biased_grouped_topk_gpu, biased_grouped_topk_impl
from sglang.test.ci.ci_register import register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `sys`, `pytest`, `torch`, `sglang.srt.layers.moe.topk`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sys`, `pytest`, `torch`, `sglang.srt.layers.moe.topk`。

### Lines 9-9: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=2, suite="nightly-1-gpu", nightly=True)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 12-91: test case fused topk deepseek (part 1/2) / 测试用例 fused topk deepseek（第 1/2 部分）
```python
@pytest.mark.parametrize(
    "seq_length",
    list(range(1, 10))
    + [16, 32, 64, 128, 256, 512, 1024, 2048, 4096, 8192, 16384, 32768, 65536],
)
@pytest.mark.parametrize(
    "params",
    [
        (128, 4, 2, 4),  # 128 experts configuration
        (256, 8, 4, 8),  # DeepSeek V3 config - most important to test
        (64, 2, 2, 4),  # Smaller configuration
    ],
)
@pytest.mark.parametrize("apply_routed_scaling_factor_on_output", [False, True])
def test_fused_topk_deepseek(seq_length, params, apply_routed_scaling_factor_on_output):
    """
    Test the fused_topk_deepseek code path in biased_grouped_topk_gpu.
    """
    num_experts, num_expert_group, topk_group, topk = params
    dtype = torch.float32

    torch.manual_seed(seq_length)
    hidden_states = torch.randn(seq_length, 128, dtype=dtype, device="cuda")
    gating_output = torch.randn(seq_length, num_experts, dtype=dtype, device="cuda")
    correction_bias = torch.randn(num_experts, dtype=dtype, device="cuda")

    routed_scaling_factor = 2.5 if apply_routed_scaling_factor_on_output else None

    # Fused implementation (uses fused_topk_deepseek when conditions are met)
    output, indices = biased_grouped_topk_gpu(
        hidden_states,
        gating_output,
        correction_bias,
        topk=topk,
        renormalize=True,
        num_expert_group=num_expert_group,
        topk_group=topk_group,
        num_fused_shared_experts=0,
        routed_scaling_factor=routed_scaling_factor,
        apply_routed_scaling_factor_on_output=apply_routed_scaling_factor_on_output,
    )

    # Reference implementation (pure PyTorch)
    ref_output, ref_indices = biased_grouped_topk_impl(
        hidden_states,
        gating_output,
        correction_bias,
        topk=topk,
        renormalize=True,
        num_expert_group=num_expert_group,
        topk_group=topk_group,
        num_fused_shared_experts=0,
        routed_scaling_factor=routed_scaling_factor,
        apply_routed_scaling_factor_on_output=apply_routed_scaling_factor_on_output,
    )

    # Check 1: Row-wise sums should match (invariant to tie-breaking)
    output_sum = output.sum(dim=-1)
    ref_output_sum = ref_output.sum(dim=-1)
    sum_check = torch.allclose(output_sum, ref_output_sum, rtol=1e-03, atol=1e-04)

    # Check 2: Scatter-based comparison with allowance for tie-breaking
    res = torch.zeros(seq_length, num_experts, dtype=torch.float32, device="cuda")
    ref = torch.zeros(seq_length, num_experts, dtype=torch.float32, device="cuda")

    res.scatter_(1, indices.long(), output)
    ref.scatter_(1, ref_indices.long(), ref_output)

    diff = torch.abs(ref - res)
    atol = (
        5e-03
        if (seq_length >= 4096 and apply_routed_scaling_factor_on_output)
        else 1e-03
    )
    num_large_diffs = (diff > atol).sum().item()

    # Allow a small number of differences for tie-breaking situations
    max_allowed_diffs = max(16, seq_length // 500)
    scatter_check = num_large_diffs <= max_allowed_diffs

```
**EN:** Test the fused_topk_deepseek code path in biased_grouped_topk_gpu. This test exercises `test_fused_topk_deepseek` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** Test the fused_topk_deepseek code path in biased_grouped_topk_gpu. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_topk_deepseek`。 这一段对应同一逻辑块的第 1 部分。

### Lines 92-95: test case fused topk deepseek (part 2/2) / 测试用例 fused topk deepseek（第 2/2 部分）
```python
    assert sum_check and scatter_check, (
        f"Output mismatch at seq_length {seq_length}, params {params}, "
        f"apply_routed_scaling_factor_on_output {apply_routed_scaling_factor_on_output}"
    )
```
**EN:** Test the fused_topk_deepseek code path in biased_grouped_topk_gpu. This test exercises `test_fused_topk_deepseek` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** Test the fused_topk_deepseek code path in biased_grouped_topk_gpu. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_topk_deepseek`。 这一段对应同一逻辑块的第 2 部分。

### Lines 98-99: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `test_fused_topk_deepseek`: Test the fused_topk_deepseek code path in biased_grouped_topk_gpu. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_topk_deepseek`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `sys`
- **Third-party modules / 第三方模块**: `pytest`, `torch`
- **Internal modules / 内部模块**: `sglang.srt.layers.moe.topk`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 99
