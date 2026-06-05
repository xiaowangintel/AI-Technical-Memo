# test_moe_topk_sigmoid.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_moe_topk_sigmoid.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `MoE topk sigmoid` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `MoE topk sigmoid` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Imports and module setup
````python
import itertools
import sys

import pytest
import torch
from sgl_kernel import topk_sigmoid
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 9-44: `test_topk_sigmoid` definition
````python
@pytest.mark.parametrize(
    "num_tokens, num_experts, topk",
    list(
        itertools.product(
            [1, 16, 128, 512, 1024, 2048],  # num_tokens
            [4, 8, 16, 32, 64, 128, 256],  # num_experts
            [1, 2, 4],  # topk
        )
    ),
)
def test_topk_sigmoid(num_tokens, num_experts, topk):
    gating_output = torch.randn(
        (num_tokens, num_experts), dtype=torch.float32, device="cuda"
    )

    topk_weights = torch.empty((num_tokens, topk), dtype=torch.float32, device="cuda")
    topk_indices = torch.empty((num_tokens, topk), dtype=torch.int32, device="cuda")

    topk_sigmoid(
        topk_weights,
        topk_indices,
        gating_output,
    )

    # Native torch implementation
    sigmoid_output = torch.sigmoid(gating_output)
    topk_weights_ref, topk_indices_ref = torch.topk(sigmoid_output, topk, dim=-1)

    # Verify the top-k weights and indices match the torch native ones
    assert torch.allclose(
        topk_weights_ref, topk_weights, atol=1e-3, rtol=1e-3
    ), f"Weights mismatch: torch={topk_weights_ref} vs SGLang={topk_weights}"

    assert torch.allclose(
        topk_indices_ref.int(), topk_indices, atol=0, rtol=0
    ), f"Indices mismatch: torch={topk_indices_ref}, SGLang={topk_indices}"
````
**EN:** This section defines the test `test_topk_sigmoid`. It sets up inputs, runs the target path, and checks the expected result. It also performs explicit assertions or shape checks before continuing. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_topk_sigmoid`。它会准备输入、执行目标路径，并检查预期结果。 它还会在继续执行前进行显式断言或形状检查。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 47-87: `test_topk_sigmoid_dtype_regression` definition
````python
@pytest.mark.parametrize(
    "num_tokens, num_experts, topk, dtype",
    list(
        itertools.product(
            [1, 16, 128, 512, 1024, 2048],  # num_tokens
            [4, 8, 16, 32, 64, 128, 256],  # num_experts
            [1, 2, 4],  # topk
            [torch.float16, torch.bfloat16, torch.float32],  # dtype
        )
    ),
)
def test_topk_sigmoid_dtype_regression(num_tokens, num_experts, topk, dtype):
    gating_output = torch.randn((num_tokens, num_experts), dtype=dtype, device="cuda")

    topk_weights = torch.empty((num_tokens, topk), dtype=torch.float32, device="cuda")
    topk_indices = torch.empty((num_tokens, topk), dtype=torch.int32, device="cuda")

    topk_sigmoid(
        topk_weights,
        topk_indices,
        gating_output,
    )

    topk_weights_ref = torch.empty(
        (num_tokens, topk), dtype=torch.float32, device="cuda"
    )
    topk_indices_ref = torch.empty((num_tokens, topk), dtype=torch.int32, device="cuda")

    topk_sigmoid(
        topk_weights_ref,
        topk_indices_ref,
        gating_output.float(),
    )

    assert torch.allclose(
        topk_weights_ref, topk_weights, atol=1e-3, rtol=1e-3
    ), f"Weights mismatch: SGLang old interface={topk_weights_ref} vs SGLang new interface={topk_weights}"

    assert torch.allclose(
        topk_indices_ref.int(), topk_indices, atol=0, rtol=0
    ), f"Indices mismatch: SGLang old interface={topk_indices_ref}, SGLang new interface={topk_indices}"
````
**EN:** This section defines the test `test_topk_sigmoid_dtype_regression`. It sets up inputs, runs the target path, and checks the expected result. It also performs explicit assertions or shape checks before continuing. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_topk_sigmoid_dtype_regression`。它会准备输入、执行目标路径，并检查预期结果。 它还会在继续执行前进行显式断言或形状检查。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 90-136: `test_topk_sigmoid_renormalize` definition
````python
@pytest.mark.parametrize(
    "num_tokens, num_experts, topk",
    list(
        itertools.product(
            [1, 16, 128, 512, 1024, 2048],  # num_tokens
            [4, 8, 16, 32, 64, 128, 256],  # num_experts
            [1, 2, 4],  # topk
        )
    ),
)
def test_topk_sigmoid_renormalize(num_tokens, num_experts, topk):
    gating_output = torch.randn(
        (num_tokens, num_experts), dtype=torch.bfloat16, device="cuda"
    )

    topk_weights = torch.empty((num_tokens, topk), dtype=torch.float32, device="cuda")
    topk_indices = torch.empty((num_tokens, topk), dtype=torch.int32, device="cuda")

    topk_sigmoid(
        topk_weights,
        topk_indices,
        gating_output,
        renormalize=True,
    )

    topk_weights_ref = torch.empty(
        (num_tokens, topk), dtype=torch.float32, device="cuda"
    )
    topk_indices_ref = torch.empty((num_tokens, topk), dtype=torch.int32, device="cuda")
    token_expert_indices_ref = torch.empty(
        (num_tokens, topk), dtype=torch.int32, device="cuda"
    )

    topk_sigmoid(
        topk_weights_ref,
        topk_indices_ref,
        gating_output,
    )
    topk_weights_ref = topk_weights_ref / topk_weights_ref.sum(dim=-1, keepdim=True)

    assert torch.allclose(
        topk_weights_ref, topk_weights, atol=1e-3, rtol=1e-3
    ), f"Weights mismatch: SGLang w/o fused renormalize={topk_weights_ref} vs SGLang w/ fused renormalize={topk_weights}"

    assert torch.allclose(
        topk_indices_ref.int(), topk_indices, atol=0, rtol=0
    ), f"Indices mismatch: SGLang w/o fused renormalize={topk_indices_ref}, SGLang w/ fused renormalize={topk_indices}"
````
**EN:** This section defines the test `test_topk_sigmoid_renormalize`. It sets up inputs, runs the target path, and checks the expected result. It also performs explicit assertions or shape checks before continuing. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_topk_sigmoid_renormalize`。它会准备输入、执行目标路径，并检查预期结果。 它还会在继续执行前进行显式断言或形状检查。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 139-180: `test_topk_sigmoid_renormalize_correction_bias` definition
````python
@pytest.mark.parametrize(
    "num_tokens, num_experts, topk",
    list(
        itertools.product(
            [1, 16, 128, 512, 1024, 2048],  # num_tokens
            [4, 8, 16, 32, 48, 64, 128, 256],  # num_experts
            [1, 2, 4],  # topk
        )
    ),
)
def test_topk_sigmoid_renormalize_correction_bias(num_tokens, num_experts, topk):
    gating_output = torch.randn(
        (num_tokens, num_experts), dtype=torch.float32, device="cuda"
    )
    correction_bias = torch.randn((num_experts), dtype=torch.float32, device="cuda")

    topk_weights = torch.empty((num_tokens, topk), dtype=torch.float32, device="cuda")
    topk_indices = torch.empty((num_tokens, topk), dtype=torch.int32, device="cuda")

    topk_sigmoid(
        topk_weights,
        topk_indices,
        gating_output,
        renormalize=True,
        correction_bias=correction_bias,
    )

    # Native torch implementation
    sigmoid_output = torch.sigmoid(gating_output)
    sigmoid_scores = sigmoid_output.view(-1, num_experts) + correction_bias.unsqueeze(0)
    _, topk_indices_ref = torch.topk(sigmoid_scores, k=topk, dim=-1)
    topk_weights_ref = sigmoid_output.gather(1, topk_indices_ref)
    topk_weights_ref = topk_weights_ref / topk_weights_ref.sum(dim=-1, keepdim=True)

    # Verify the top-k weights and indices match the torch native ones
    assert torch.allclose(
        topk_weights_ref, topk_weights, atol=1e-3, rtol=1e-3
    ), f"Weights mismatch: torch={topk_weights_ref} vs SGLang={topk_weights}"

    assert torch.allclose(
        topk_indices_ref.int(), topk_indices, atol=0, rtol=0
    ), f"Indices mismatch: torch={topk_indices_ref}, SGLang={topk_indices}"
````
**EN:** This section defines the test `test_topk_sigmoid_renormalize_correction_bias`. It sets up inputs, runs the target path, and checks the expected result. It also performs explicit assertions or shape checks before continuing. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_topk_sigmoid_renormalize_correction_bias`。它会准备输入、执行目标路径，并检查预期结果。 它还会在继续执行前进行显式断言或形状检查。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 183-184: Command-line entry point
````python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `test_topk_sigmoid`, `test_topk_sigmoid_dtype_regression`, `test_topk_sigmoid_renormalize`, `test_topk_sigmoid_renormalize_correction_bias`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`
- **External / 外部**: `itertools`, `pytest`, `sys`, `torch`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
