# test_kimi_k2_moe_fused_gate.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_kimi_k2_moe_fused_gate.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `Kimi k2 MoE fused gate` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `Kimi k2 MoE fused gate` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Imports and module setup
````python
import sys

import pytest
import torch
from sgl_kernel import kimi_k2_moe_fused_gate

from sglang.srt.layers.moe.topk import kimi_k2_biased_topk_impl
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 10-64: `test_kimi_k2_moe_fused_gate` definition
````python
@pytest.mark.parametrize(
    "seq_length",
    list(range(1, 10))
    + [16, 32, 64, 128, 256, 512, 1024, 2048, 4096, 8192, 16384, 32768, 65536],
)
@pytest.mark.parametrize("topk", [6])  # Kimi K2 uses topk=6
@pytest.mark.parametrize("dtype", [torch.float32])
@pytest.mark.parametrize("apply_routed_scaling_factor_on_output", [False, True])
def test_kimi_k2_moe_fused_gate(
    seq_length, topk, dtype, apply_routed_scaling_factor_on_output
):
    num_experts = 384  # Kimi K2: only support 384 experts
    renormalize = True
    routed_scaling_factor = 2.872  # Kimi K2's routed scaling factor

    torch.manual_seed(seq_length)
    tensor = torch.rand((seq_length, num_experts), dtype=dtype, device="cuda")
    scores = tensor.clone()
    bias = torch.rand(num_experts, dtype=dtype, device="cuda")

    # Test our fused kernel
    output, indices = kimi_k2_moe_fused_gate(
        tensor,
        bias,
        topk=topk,
        renormalize=renormalize,
        routed_scaling_factor=routed_scaling_factor,
        apply_routed_scaling_factor_on_output=apply_routed_scaling_factor_on_output,
    )

    # Reference implementation
    ref_output, ref_indices = kimi_k2_biased_topk_impl(
        scores,
        scores,
        bias,
        topk=topk,
        renormalize=renormalize,
        routed_scaling_factor=routed_scaling_factor,
        apply_routed_scaling_factor_on_output=apply_routed_scaling_factor_on_output,
    )

    # Check weights match (after sorting)
    # Weights are the most important - they determine the actual MoE output
    output_check = torch.allclose(
        ref_output.sort()[0].to(torch.float32),
        output.sort()[0].to(torch.float32),
        rtol=1e-02,
        atol=1e-03,
    )

    assert output_check, (
        f"Output mismatch at seq_length {seq_length}, dtype {dtype}, "
        f"num_experts {num_experts}, topk {topk}, "
        f"apply_routed_scaling_factor_on_output {apply_routed_scaling_factor_on_output}"
    )
````
**EN:** This section defines the test `test_kimi_k2_moe_fused_gate`. It sets up inputs, runs the target path, and checks the expected result. It also performs explicit assertions or shape checks before continuing. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_kimi_k2_moe_fused_gate`。它会准备输入、执行目标路径，并检查预期结果。 它还会在继续执行前进行显式断言或形状检查。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 67-122: `test_kimi_k2_specific_case` definition
````python
@pytest.mark.parametrize("seq_length", [1024, 4096])
@pytest.mark.parametrize("num_experts", [384])
@pytest.mark.parametrize("topk", [6])
def test_kimi_k2_specific_case(seq_length, num_experts, topk):
    """Test specifically for Kimi K2 configuration: 384 experts, topk=6"""
    dtype = torch.float32
    renormalize = True
    routed_scaling_factor = 2.872

    torch.manual_seed(42)
    tensor = torch.rand((seq_length, num_experts), dtype=dtype, device="cuda")
    scores = tensor.clone()
    bias = torch.rand(num_experts, dtype=dtype, device="cuda")

    output, indices = kimi_k2_moe_fused_gate(
        tensor,
        bias,
        topk=topk,
        renormalize=renormalize,
        routed_scaling_factor=routed_scaling_factor,
        apply_routed_scaling_factor_on_output=False,
    )

    ref_output, ref_indices = kimi_k2_biased_topk_impl(
        scores,
        scores,
        bias,
        topk=topk,
        renormalize=renormalize,
        routed_scaling_factor=routed_scaling_factor,
        apply_routed_scaling_factor_on_output=False,
    )

    # Verify output shapes
    assert output.shape == (seq_length, topk)
    assert indices.shape == (seq_length, topk)
    assert output.dtype == torch.float32
    assert indices.dtype == torch.int32

    # Verify weights are normalized (sum to 1 per token if renormalize=True)
    if renormalize:
        weight_sums = output.sum(dim=-1)
        assert torch.allclose(
            weight_sums, torch.ones_like(weight_sums), rtol=1e-3, atol=1e-4
        )

    # Check weights match (after sorting)
    # Weights are the most important - they determine the actual MoE output
    output_check = torch.allclose(
        ref_output.sort()[0].to(torch.float32),
        output.sort()[0].to(torch.float32),
        rtol=1e-02,
        atol=1e-03,
    )

    assert output_check, f"Output mismatch for Kimi K2 specific case"
````
**EN:** This section defines the test `test_kimi_k2_specific_case`. It sets up inputs, runs the target path, and checks the expected result. Docstring summary: Test specifically for Kimi K2 configuration: 384 experts, topk=6 It also performs explicit assertions or shape checks before continuing. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_kimi_k2_specific_case`。它会准备输入、执行目标路径，并检查预期结果。 文档字符串摘要：Test specifically for Kimi K2 configuration: 384 experts, topk=6 它还会在继续执行前进行显式断言或形状检查。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 125-126: Command-line entry point
````python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `test_kimi_k2_moe_fused_gate`, `test_kimi_k2_specific_case`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`, `sglang.srt.layers.moe.topk`
- **External / 外部**: `pytest`, `sys`, `torch`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
