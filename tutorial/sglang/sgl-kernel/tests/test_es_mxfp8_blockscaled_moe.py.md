# test_es_mxfp8_blockscaled_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_es_mxfp8_blockscaled_moe.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `expert specialization MXFP8 blockscaled MoE` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `expert specialization MXFP8 blockscaled MoE` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Imports and module setup
````python
import random
import sys

import pytest
import torch
from sgl_kernel import (
    es_sm100_mxfp8_blockscaled_grouped_mm,
    es_sm100_mxfp8_blockscaled_grouped_quant,
)
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 11-11: Top-level logic
````python
random.seed(42)
````
**EN:** This top-level block contains executable module logic that does not fit into a simpler category.
**CN:** 该顶层代码块包含不适合归入更简单类别的模块执行逻辑。

### Lines 12-12: Top-level logic
````python
torch.manual_seed(42)
````
**EN:** This top-level block contains executable module logic that does not fit into a simpler category.
**CN:** 该顶层代码块包含不适合归入更简单类别的模块执行逻辑。

### Lines 13-13: Top-level logic
````python
torch.cuda.manual_seed(42)
````
**EN:** This top-level block contains executable module logic that does not fit into a simpler category.
**CN:** 该顶层代码块包含不适合归入更简单类别的模块执行逻辑。

### Lines 14-14: Top-level logic
````python
torch.cuda.manual_seed_all(42)
````
**EN:** This top-level block contains executable module logic that does not fit into a simpler category.
**CN:** 该顶层代码块包含不适合归入更简单类别的模块执行逻辑。

### Lines 17-18: `align` definition
````python
def align(val: int, alignment: int = 128) -> int:
    return int((val + alignment - 1) // alignment * alignment)
````
**EN:** This section defines `align` and implements the core logic associated with alignment.
**CN:** 该部分定义 `align`，并实现与 alignment 相关的核心逻辑。

### Lines 19-21: Comments and local context
````python


# Copy from: https://github.com/deepseek-ai/DeepGEMM/blob/main/deep_gemm/utils.py
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 22-26: `calc_diff` definition
````python
def calc_diff(x, y):
    x, y = x.double(), y.double()
    denominator = (x * x + y * y).sum()
    sim = 2 * (x * y).sum() / denominator
    return 1 - sim
````
**EN:** This section defines `calc_diff` and implements the core logic associated with calc diff.
**CN:** 该部分定义 `calc_diff`，并实现与 calc diff 相关的核心逻辑。

### Lines 29-32: `is_sm100_supported` definition
````python
def is_sm100_supported(device=None) -> bool:
    return (torch.cuda.get_device_capability(device)[0] == 10) and (
        torch.version.cuda >= "12.8"
    )
````
**EN:** This section defines `is_sm100_supported` and implements the core logic associated with is sm100 supported.
**CN:** 该部分定义 `is_sm100_supported`，并实现与 is sm100 supported 相关的核心逻辑。

### Lines 35-152: `test_es_sm100_mxfp8_blockscaled_grouped_mm` definition
````python
@pytest.mark.skipif(
    not is_sm100_supported(),
    reason="test_es_sm100_mxfp8_blockscaled_grouped_mm at sgl-kernel is only supported on sm100",
)
@pytest.mark.parametrize("num_experts", [8, 16, 32, 64])
@pytest.mark.parametrize("out_dtype", [torch.half, torch.bfloat16])
def test_es_sm100_mxfp8_blockscaled_grouped_mm(num_experts, out_dtype):
    device = "cuda"
    alignment = 128
    n_g = random.randint(1, 64) * alignment
    k_g = random.randint(1, 64) * alignment

    expert_offset = 0
    expert_offsets = []
    aux_expert_offset = 0
    aux_expert_offsets = []
    a_blockscale_offset = 0
    a_blockscale_offsets = []
    b_blockscale_offset = 0
    b_blockscale_offsets = []
    problem_sizes = []
    aux_problem_sizes = []
    a_list = []
    b_list = []
    ref_d_list = []

    for g in range(num_experts):
        m_g = random.randint(1, 512)
        expert_offsets.append(expert_offset)
        expert_offset += m_g
        aux_expert_offsets.append(aux_expert_offset)
        aux_expert_offset += n_g
        a_blockscale_offsets.append(a_blockscale_offset)
        a_blockscale_offset += align(m_g, 128)
        b_blockscale_offsets.append(b_blockscale_offset)
        b_blockscale_offset += n_g  # n_g already align to 128
        problem_sizes.append([m_g, n_g, k_g])
        aux_problem_sizes.append([n_g, m_g, k_g])

        a = torch.normal(
            0.0, std=1.0, size=(m_g, k_g), device=device, dtype=out_dtype
        )  # (M, K):(K, 1)
        b = torch.normal(
            0.0, std=1.0, size=(n_g, k_g), device=device, dtype=out_dtype
        )  # (N, K):(K, 1)

        a_list.append(a)
        b_list.append(b)
        ref_d = a @ b.T
        ref_d_list.append(ref_d)
    a = torch.concat(a_list, dim=0)
    b = torch.concat(b_list, dim=0)

    _problem_sizes = torch.tensor(problem_sizes).to(device=device, dtype=torch.int32)
    _aux_problem_sizes = torch.tensor(aux_problem_sizes).to(
        device=device, dtype=torch.int32
    )
    _expert_offsets = torch.tensor(expert_offsets).to(device=device, dtype=torch.int32)
    _aux_expert_offsets = torch.tensor(aux_expert_offsets).to(
        device=device, dtype=torch.int32
    )
    _a_blockscale_offsets = torch.tensor(a_blockscale_offsets).to(
        device=device, dtype=torch.int32
    )
    _b_blockscale_offsets = torch.tensor(b_blockscale_offsets).to(
        device=device, dtype=torch.int32
    )

    a_quant = torch.zeros_like(a, dtype=torch.float8_e4m3fn, device=device)
    a_scale_factor = torch.zeros(
        (a_blockscale_offset, k_g // 32), dtype=torch.uint8, device=device
    )

    b_quant = torch.zeros_like(b, dtype=torch.float8_e4m3fn, device=device)
    b_scale_factor = torch.zeros(
        (num_experts, n_g, k_g // 32), dtype=torch.uint8, device=device
    )

    es_sm100_mxfp8_blockscaled_grouped_quant(
        a,
        _problem_sizes,
        _expert_offsets,
        _a_blockscale_offsets,
        a_quant,
        a_scale_factor,
    )

    es_sm100_mxfp8_blockscaled_grouped_quant(
        b,
        _aux_problem_sizes,
        _aux_expert_offsets,
        _b_blockscale_offsets,
        b_quant,
        b_scale_factor,
    )
    b_quant = b_quant.view(num_experts, n_g, k_g).transpose(1, 2)
    b_scale_factor = b_scale_factor.view(num_experts, n_g, k_g // 32).transpose(1, 2)

    d = torch.empty((expert_offset, n_g), device=device, dtype=out_dtype)
    es_sm100_mxfp8_blockscaled_grouped_mm(
        d,
        a_quant,
        b_quant,
        a_scale_factor,
        b_scale_factor,
        _problem_sizes,
        _expert_offsets,
        _a_blockscale_offsets,
    )

    for g in range(num_experts):
        baseline = ref_d_list[g]
        actual = d[expert_offsets[g] : (expert_offsets[g] + problem_sizes[g][0])]
        diff = calc_diff(actual, baseline)
        assert diff < 0.001
        print(
            f"m_g={baseline.shape[0]} n_g={n_g} k_g={k_g} num_experts={num_experts}, out_dtype={out_dtype}, diff={diff:.5f}: OK"
        )
````
**EN:** This section defines the test `test_es_sm100_mxfp8_blockscaled_grouped_mm`. It sets up inputs, runs the target path, and checks the expected result. It also performs explicit assertions or shape checks before continuing. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_es_sm100_mxfp8_blockscaled_grouped_mm`。它会准备输入、执行目标路径，并检查预期结果。 它还会在继续执行前进行显式断言或形状检查。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 155-156: Command-line entry point
````python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `align`, `calc_diff`, `is_sm100_supported`, `test_es_sm100_mxfp8_blockscaled_grouped_mm`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`
- **External / 外部**: `pytest`, `random`, `sys`, `torch`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
