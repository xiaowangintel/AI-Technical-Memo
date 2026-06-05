# test_moe_fused_gate.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_moe_fused_gate.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `MoE fused gate` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `MoE fused gate` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Imports and module setup
````python
import sys
from typing import Optional

import pytest
import torch
from sgl_kernel import moe_fused_gate
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 9-94: `biased_grouped_topk_impl` definition
````python
def biased_grouped_topk_impl(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    correction_bias: torch.Tensor,
    topk: int,
    renormalize: bool,
    num_expert_group: Optional[int] = None,
    topk_group: Optional[int] = None,
    num_fused_shared_experts: int = 0,
    routed_scaling_factor: Optional[float] = None,
    apply_routed_scaling_factor_on_output: Optional[bool] = False,
):
    assert hidden_states.shape[0] == gating_output.shape[0], "Number of tokens mismatch"

    scores = gating_output.sigmoid()
    num_token = scores.shape[0]
    num_experts = scores.shape[1]
    scores_for_choice = scores.view(num_token, -1) + correction_bias.unsqueeze(0)
    group_scores = (
        scores_for_choice.view(num_token, num_expert_group, -1)
        .topk(2, dim=-1)[0]
        .sum(dim=-1)
    )  # [n, n_group]
    group_idx = torch.topk(group_scores, k=topk_group, dim=-1, sorted=False)[
        1
    ]  # [n, top_k_group]
    group_mask = torch.zeros_like(group_scores)  # [n, n_group]
    group_mask.scatter_(1, group_idx, 1)  # [n, n_group]
    score_mask = (
        group_mask.unsqueeze(-1)
        .expand(num_token, num_expert_group, scores.shape[-1] // num_expert_group)
        .reshape(num_token, -1)
    )  # [n, e]
    tmp_scores = scores_for_choice.masked_fill(
        ~score_mask.bool(), float("-inf")
    )  # [n, e]

    topk_excluding_shared = topk - num_fused_shared_experts
    _, routed_topk_ids = torch.topk(
        tmp_scores,
        k=topk_excluding_shared,
        dim=-1,
        sorted=False,
    )
    routed_topk_weights = scores.gather(1, routed_topk_ids)

    if num_fused_shared_experts > 0:
        topk_ids = torch.empty(
            (num_token, topk),
            dtype=routed_topk_ids.dtype,
            device=routed_topk_ids.device,
        )
        topk_weights = torch.empty(
            (num_token, topk),
            dtype=routed_topk_weights.dtype,
            device=routed_topk_weights.device,
        )
        topk_ids[:, :topk_excluding_shared] = routed_topk_ids
        topk_weights[:, :topk_excluding_shared] = routed_topk_weights

        scale = 1.0 if routed_scaling_factor is None else float(routed_scaling_factor)
        routed_sum = routed_topk_weights.sum(dim=-1, keepdim=True)

        for i in range(num_fused_shared_experts):
            topk_ids[:, topk_excluding_shared + i] = num_experts + i
            topk_weights[:, topk_excluding_shared + i] = routed_sum[:, 0] / scale
    else:
        topk_ids = routed_topk_ids
        topk_weights = routed_topk_weights

    if renormalize:
        if num_fused_shared_experts > 0:
            topk_weights_sum = topk_weights[:, :topk_excluding_shared].sum(
                dim=-1, keepdim=True
            )
        else:
            topk_weights_sum = topk_weights.sum(dim=-1, keepdim=True)
        topk_weights = topk_weights / topk_weights_sum
        if apply_routed_scaling_factor_on_output:
            scale = (
                1.0 if routed_scaling_factor is None else float(routed_scaling_factor)
            )
            topk_weights *= scale

    topk_weights, topk_ids = topk_weights.to(torch.float32), topk_ids.to(torch.int32)
    return topk_weights, topk_ids
````
**EN:** This section defines `biased_grouped_topk_impl` and implements the core logic associated with biased grouped topk impl. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `biased_grouped_topk_impl`，并实现与 biased grouped topk impl 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。

### Lines 97-121: `biased_grouped_topk` definition
````python
def biased_grouped_topk(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    correction_bias: torch.Tensor,
    topk: int,
    renormalize: bool,
    num_expert_group: Optional[int] = None,
    topk_group: Optional[int] = None,
    num_fused_shared_experts: int = 0,
    routed_scaling_factor: Optional[float] = None,
    num_token_non_padded: Optional[torch.Tensor] = None,
    apply_routed_scaling_factor_on_output: Optional[bool] = False,
):
    return biased_grouped_topk_impl(
        hidden_states,
        gating_output,
        correction_bias,
        topk,
        renormalize,
        num_expert_group,
        topk_group,
        num_fused_shared_experts=num_fused_shared_experts,
        routed_scaling_factor=routed_scaling_factor,
        apply_routed_scaling_factor_on_output=apply_routed_scaling_factor_on_output,
    )
````
**EN:** This section defines `biased_grouped_topk` and implements the core logic associated with biased grouped topk.
**CN:** 该部分定义 `biased_grouped_topk`，并实现与 biased grouped topk 相关的核心逻辑。

### Lines 124-215: `test_moe_fused_gate_combined` definition
````python
@pytest.mark.parametrize(
    "seq_length",
    list(range(1, 10))
    + [16, 32, 64, 128, 256, 512, 1024, 2048, 4096, 8192, 16384, 32768, 65536],
)
@pytest.mark.parametrize(
    "params",
    [
        (128, 4, 2, 4),
        (256, 8, 4, 8),  # deepseek v3
        (512, 16, 8, 16),
    ],
)
@pytest.mark.parametrize("num_fused_shared_experts", [0, 1, 2])
@pytest.mark.parametrize("apply_routed_scaling_factor_on_output", [False, True])
def test_moe_fused_gate_combined(
    seq_length, params, num_fused_shared_experts, apply_routed_scaling_factor_on_output
):
    num_experts, num_expert_group, topk_group, topk = params
    dtype = torch.float32

    torch.manual_seed(seq_length)
    tensor = torch.rand((seq_length, num_experts), dtype=dtype, device="cuda")
    scores = tensor.clone()
    bias = torch.rand(num_experts, dtype=dtype, device="cuda")
    topk = topk + num_fused_shared_experts

    output, indices = moe_fused_gate(
        tensor,
        bias,
        num_expert_group=num_expert_group,
        topk_group=topk_group,
        topk=topk,
        num_fused_shared_experts=num_fused_shared_experts,
        routed_scaling_factor=2.5,
        apply_routed_scaling_factor_on_output=apply_routed_scaling_factor_on_output,
    )
    ref_output, ref_indices = biased_grouped_topk(
        scores,
        scores,
        bias,
        topk=topk,
        renormalize=True,
        num_expert_group=num_expert_group,
        topk_group=topk_group,
        num_fused_shared_experts=num_fused_shared_experts,
        routed_scaling_factor=2.5,
        apply_routed_scaling_factor_on_output=apply_routed_scaling_factor_on_output,
    )

    # When num_fused_shared_experts > 0, ignore the comparison of the last topk dimension
    if num_fused_shared_experts > 0:
        original_indices = indices.clone()
        original_ref_indices = ref_indices.clone()

        indices = indices[:, :-1]
        ref_indices = ref_indices[:, :-1]

        valid_min = num_experts
        valid_max = num_experts + num_fused_shared_experts
        shared_indices = original_indices[:, -1]
        shared_ref_indices = original_ref_indices[:, -1]
        if shared_indices is not None:
            assert torch.all(
                (shared_indices >= valid_min) & (shared_indices < valid_max)
            ), f"Shared expert indices out of range: found values outside [{valid_min}, {valid_max})"
        if shared_ref_indices is not None:
            assert torch.all(
                (shared_ref_indices >= valid_min) & (shared_ref_indices < valid_max)
            ), f"Shared expert reference indices out of range: found values outside [{valid_min}, {valid_max})"

    idx_check = torch.allclose(
        ref_indices.sort()[0].to(torch.int32),
        indices.sort()[0].to(torch.int32),
        rtol=1e-04,
        atol=1e-05,
    )
    output_check = torch.allclose(
        ref_output.sort()[0].to(torch.float32),
        output.sort()[0].to(torch.float32),
        rtol=1e-02,
        atol=1e-03,
    )

    assert idx_check, (
        f"Indices mismatch at seq_length {seq_length}, dtype {dtype}, "
        f"params {params}, num_fused_shared_experts {num_fused_shared_experts}"
    )
    assert output_check, (
        f"Output mismatch at seq_length {seq_length}, dtype {dtype}, "
        f"params {params}, num_fused_shared_experts {num_fused_shared_experts}"
    )
````
**EN:** This section defines the test `test_moe_fused_gate_combined`. It sets up inputs, runs the target path, and checks the expected result. It also performs explicit assertions or shape checks before continuing. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_moe_fused_gate_combined`。它会准备输入、执行目标路径，并检查预期结果。 它还会在继续执行前进行显式断言或形状检查。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 218-219: Command-line entry point
````python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `biased_grouped_topk_impl`, `biased_grouped_topk`, `test_moe_fused_gate_combined`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`
- **External / 外部**: `pytest`, `sys`, `torch`, `typing`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
