# test_fp8_blockwise_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_fp8_blockwise_moe.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `FP8 blockwise MoE` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `FP8 blockwise MoE` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Imports and module setup
````python
import random
import sys
from typing import Tuple

import pytest
import torch
from sgl_kernel import fp8_blockwise_scaled_grouped_mm
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 10-11: `cdiv` definition
````python
def cdiv(a: int, b: int) -> int:
    return -(a // -b)
````
**EN:** This section defines `cdiv` and implements the core logic associated with cdiv.
**CN:** 该部分定义 `cdiv`，并实现与 cdiv 相关的核心逻辑。

### Lines 14-15: `scale_shape` definition
````python
def scale_shape(shape, group_shape):
    return tuple(cdiv(shape[i], group_shape[i]) for i in range(len(group_shape)))
````
**EN:** This section defines `scale_shape` and implements the core logic associated with scale shape.
**CN:** 该部分定义 `scale_shape`，并实现与 scale shape 相关的核心逻辑。

### Lines 18-22: `to_fp8` definition
````python
def to_fp8(tensor: torch.Tensor) -> torch.Tensor:
    finfo = torch.finfo(torch.float8_e4m3fn)
    return torch.round(tensor.clamp(min=finfo.min, max=finfo.max)).to(
        dtype=torch.float8_e4m3fn
    )
````
**EN:** This section defines `to_fp8` and implements the core logic associated with to FP8.
**CN:** 该部分定义 `to_fp8`，并实现与 to FP8 相关的核心逻辑。

### Lines 23-25: Comments and local context
````python


# Copy from: https://github.com/deepseek-ai/DeepGEMM/blob/main/deep_gemm/utils.py
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 26-30: `calc_diff` definition
````python
def calc_diff(x, y):
    x, y = x.double(), y.double()
    denominator = (x * x + y * y).sum()
    sim = 2 * (x * y).sum() / denominator
    return 1 - sim
````
**EN:** This section defines `calc_diff` and implements the core logic associated with calc diff.
**CN:** 该部分定义 `calc_diff`，并实现与 calc diff 相关的核心逻辑。

### Lines 33-34: `ceil_div` definition
````python
def ceil_div(x: int, y: int) -> int:
    return (x + y - 1) // y
````
**EN:** This section defines `ceil_div` and implements the core logic associated with ceil div.
**CN:** 该部分定义 `ceil_div`，并实现与 ceil div 相关的核心逻辑。

### Lines 37-45: `per_token_cast_to_fp8` definition
````python
def per_token_cast_to_fp8(x: torch.Tensor) -> Tuple[torch.Tensor, torch.Tensor]:
    assert x.dim() == 2
    m, n = x.shape
    pad_size = (128 - (n % 128)) % 128
    x = torch.nn.functional.pad(x, (0, pad_size), value=0) if pad_size > 0 else x
    x_view = x.view(m, -1, 128)
    x_amax = x_view.abs().float().amax(dim=2).view(m, -1).clamp(1e-4)
    fp8_data = (x_view * (448.0 / x_amax.unsqueeze(2))).to(torch.float8_e4m3fn)
    return fp8_data.view(m, n + pad_size)[:, :n], (x_amax / 448.0).view(m, -1)
````
**EN:** This section defines `per_token_cast_to_fp8` and implements the core logic associated with per token cast to FP8. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `per_token_cast_to_fp8`，并实现与 per token cast to FP8 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。

### Lines 48-60: `per_block_cast_to_fp8` definition
````python
def per_block_cast_to_fp8(x: torch.Tensor) -> Tuple[torch.Tensor, torch.Tensor]:
    assert x.dim() == 2
    m, n = x.shape
    x_padded = torch.zeros(
        (ceil_div(m, 128) * 128, ceil_div(n, 128) * 128), dtype=x.dtype, device=x.device
    )
    x_padded[:m, :n] = x
    x_view = x_padded.view(-1, 128, x_padded.size(1) // 128, 128)
    x_amax = x_view.abs().float().amax(dim=(1, 3), keepdim=True).clamp(1e-4)
    x_scaled = (x_view * (448.0 / x_amax)).to(torch.float8_e4m3fn)
    return x_scaled.view_as(x_padded)[:m, :n].contiguous(), (x_amax / 448.0).view(
        x_view.size(0), x_view.size(2)
    )
````
**EN:** This section defines `per_block_cast_to_fp8` and implements the core logic associated with per block cast to FP8. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `per_block_cast_to_fp8`，并实现与 per block cast to FP8 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。

### Lines 63-87: `baseline_scaled_mm` definition
````python
def baseline_scaled_mm(
    a: torch.Tensor,
    b: torch.Tensor,
    scale_a: torch.Tensor,
    scale_b: torch.Tensor,
    out_dtype: type[torch.dtype],
) -> torch.Tensor:

    def group_broadcast(t, shape):
        for i, s in enumerate(shape):
            if t.shape[i] != s and t.shape[i] != 1:
                assert s % t.shape[i] == 0
                t = (
                    t.unsqueeze(i + 1)
                    .expand(*t.shape[: i + 1], s // t.shape[i], *t.shape[i + 1 :])
                    .flatten(i, i + 1)
                )
        return t

    scale_a = group_broadcast(scale_a, a.shape)
    scale_b = group_broadcast(scale_b, b.shape)

    return torch.mm(
        (scale_a * a.to(dtype=torch.float32)), (scale_b * b.to(dtype=torch.float32))
    ).to(out_dtype)
````
**EN:** This section defines `baseline_scaled_mm` and implements the core logic associated with baseline scaled mm. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `baseline_scaled_mm`，并实现与 baseline scaled mm 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。

### Lines 90-93: `is_blackwell_supported` definition
````python
def is_blackwell_supported(device=None) -> bool:
    return (torch.cuda.get_device_capability(device)[0] in [10, 12]) and (
        torch.version.cuda >= "12.8"
    )
````
**EN:** This section defines `is_blackwell_supported` and implements the core logic associated with is blackwell supported.
**CN:** 该部分定义 `is_blackwell_supported`，并实现与 is blackwell supported 相关的核心逻辑。

### Lines 96-99: `is_sm90_supported` definition
````python
def is_sm90_supported(device=None) -> bool:
    return (torch.cuda.get_device_capability(device)[0] == 9) and (
        torch.version.cuda >= "12.3"
    )
````
**EN:** This section defines `is_sm90_supported` and implements the core logic associated with is sm90 supported.
**CN:** 该部分定义 `is_sm90_supported`，并实现与 is sm90 supported 相关的核心逻辑。

### Lines 102-218: `test_fp8_blockwise_scaled_grouped_mm` definition
````python
@pytest.mark.skipif(
    not (is_blackwell_supported() or is_sm90_supported()),
    reason="fp8_blockwise_scaled_grouped_mm at sgl-kernel is only supported on sm100 or sm90",
)
@pytest.mark.parametrize("num_experts", [8, 16, 32, 64, 128])
@pytest.mark.parametrize("out_dtype", [torch.half, torch.bfloat16])
def test_fp8_blockwise_scaled_grouped_mm(num_experts, out_dtype):
    device = "cuda"
    alignment = 128
    n_g = random.randint(1, 64) * 128
    k_g = random.randint(1, 64) * 128

    expert_offsets = torch.zeros((num_experts + 1), device=device, dtype=torch.int32)
    problem_sizes = torch.zeros((num_experts, 3), device=device, dtype=torch.int32)
    layout_sfa = torch.zeros((num_experts, 5), device=device, dtype=torch.int32)
    layout_sfb = torch.zeros((num_experts, 5), device=device, dtype=torch.int32)

    a_tensors = []
    b_tensors = []
    a_scales_tensors = []
    b_scales_tensors = []
    baseline_tensors = []

    for g in range(num_experts):
        m_g = random.randint(1, 256)
        expert_offsets[g + 1] = expert_offsets[g] + m_g
        problem_sizes[g][:] = torch.tensor([m_g, n_g, k_g], device=device)

        a = torch.randn((m_g, k_g), device=device, dtype=out_dtype)  # (M, K):(K, 1)
        b = torch.randn((n_g, k_g), device=device, dtype=out_dtype).t()  # (K, N):(1, K)

        a_g, a_scale = per_token_cast_to_fp8(
            a
        )  # ag -- (M, K):(K, 1), a_scale() -- (M, k):(k, 1)
        b_g, b_scale = per_block_cast_to_fp8(
            b
        )  # bg -- (K, N):(N, 1), b_scale() -- (k, n):(n, 1)
        a_tensors.append(a_g)
        b_tensors.append(b_g)
        a_scales_tensors.append(a_scale)
        b_scales_tensors.append(b_scale)

        baseline = torch.mm(a, b)
        baseline_tensors.append(baseline)
    a_stack = torch.empty(
        (expert_offsets[-1], k_g), device=device, dtype=torch.float8_e4m3fn
    )
    b_stack = torch.empty(
        (num_experts, n_g, k_g), device=device, dtype=torch.float8_e4m3fn
    )
    a_scale_stack = torch.empty(
        (expert_offsets[-1], (k_g // 128)), device=device, dtype=torch.float32
    )
    b_scale_stack = torch.empty(
        (num_experts, n_g // 128, k_g // 128), device=device, dtype=torch.float32
    )

    for g in range(num_experts):
        # Matrix A is Row-Major.
        a_stack[expert_offsets[g] : expert_offsets[g + 1], :] = a_tensors[
            g
        ]  # a_stack[expert_offsets[g] : expert_offsets[g + 1], :] -- (M, K):(K, 1)
        b_stack[g] = b_tensors[g].t()  # b_stack[g] -- (N, K):(K, 1)

        # We need K-Major scale factor
        a_scale_stack[expert_offsets[g] : expert_offsets[g + 1], :] = a_scales_tensors[
            g
        ]
        b_scale_stack[g] = b_scales_tensors[
            g
        ].t()  # b_scale_stack[g] -- (k, n):(n, 1), we need transpose & contiguous later
    b_stack = b_stack.transpose(1, 2)  # Transpose Matrix B to Column-Major.
    b_scale_stack = b_scale_stack.transpose(1, 2)

    c_out = torch.empty((expert_offsets[-1], n_g), device=device, dtype=out_dtype)
    a_strides = torch.full(
        (num_experts,), a_stack.stride(0), device=device, dtype=torch.int64
    )
    c_strides = torch.full(
        (num_experts,), c_out.stride(0), device=device, dtype=torch.int64
    )
    workspace = torch.empty((1024 * 1024 * 1024), device=device, dtype=torch.uint8)
    a_ptrs = torch.empty((num_experts,), device=device, dtype=torch.int64)
    b_ptrs = torch.empty((num_experts,), device=device, dtype=torch.int64)
    out_ptrs = torch.empty((num_experts,), device=device, dtype=torch.int64)
    a_scales_ptrs = torch.empty((num_experts,), device=device, dtype=torch.int64)
    b_scales_ptrs = torch.empty((num_experts,), device=device, dtype=torch.int64)

    fp8_blockwise_scaled_grouped_mm(
        c_out,
        a_ptrs,
        b_ptrs,
        out_ptrs,
        a_scales_ptrs,
        b_scales_ptrs,
        a_stack,
        b_stack,
        a_scale_stack,
        b_scale_stack,
        a_strides,
        a_strides,
        c_strides,
        layout_sfa,
        layout_sfb,
        problem_sizes,
        expert_offsets[:-1],
        workspace,
    )

    for g in range(num_experts):
        baseline = baseline_tensors[g]
        actual = c_out[expert_offsets[g] : expert_offsets[g + 1]]
        diff = calc_diff(actual, baseline)
        assert diff < 0.001
        print(
            f"m_g={baseline.shape[0]} n_g={n_g} k_g={k_g} num_experts={num_experts}, out_dtype={out_dtype}, diff={diff:.5f}: OK"
        )
````
**EN:** This section defines the test `test_fp8_blockwise_scaled_grouped_mm`. It sets up inputs, runs the target path, and checks the expected result. It also performs explicit assertions or shape checks before continuing. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_fp8_blockwise_scaled_grouped_mm`。它会准备输入、执行目标路径，并检查预期结果。 它还会在继续执行前进行显式断言或形状检查。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 221-222: Command-line entry point
````python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `cdiv`, `scale_shape`, `to_fp8`, `calc_diff`, `ceil_div`, `per_token_cast_to_fp8`, `per_block_cast_to_fp8`, `baseline_scaled_mm`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`
- **External / 外部**: `pytest`, `random`, `sys`, `torch`, `typing`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
