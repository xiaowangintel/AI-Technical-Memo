# test_cutlass_w4a8_moe_mm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_cutlass_w4a8_moe_mm.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `cutlass w4a8 MoE mm` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `cutlass w4a8 MoE mm` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Imports and module setup
````python
import sys

import pytest
import torch
from sgl_kernel import cutlass_w4a8_moe_mm
from utils import is_hopper

from sglang.jit_kernel.per_tensor_quant_fp8 import per_tensor_quant_fp8
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 11-24: `pack_int4_values_to_int8` definition
````python
def pack_int4_values_to_int8(int4_values_interleaved: torch.Tensor) -> torch.Tensor:
    if int4_values_interleaved.shape[-1] % 2 != 0:
        raise ValueError(
            "the last dim size of int4_values_interleaved tensor must be even."
        )

    input_tensor_int8 = int4_values_interleaved.to(torch.int8)

    low_nibbles = input_tensor_int8[..., 0::2]
    high_nibbles = input_tensor_int8[..., 1::2]

    packed_tensor = (high_nibbles << 4) | (low_nibbles & 0x0F)

    return packed_tensor.to(torch.int8)
````
**EN:** This section defines `pack_int4_values_to_int8` and implements the core logic associated with pack INT4 values to INT8.
**CN:** 该部分定义 `pack_int4_values_to_int8`，并实现与 pack INT4 values to INT8 相关的核心逻辑。

### Lines 27-49: `pack_interleave` definition
````python
def pack_interleave(num_experts, ref_weight, ref_scale):
    n, k = ref_weight.shape[1], ref_weight.shape[2]

    weight = pack_int4_values_to_int8(ref_weight.cpu()).cuda()
    w_q = weight.view((num_experts, n, k // 2)).view(torch.int8)
    w_q = w_q.contiguous()

    alignment = 4 if k % 512 == 0 else 1
    scale_interleaved = ref_scale.reshape(
        ref_scale.shape[0],
        ref_scale.shape[1],
        (ref_scale.shape[2] // alignment),
        alignment,
    )  # [E, N, K/4, 4]
    scale_interleaved = scale_interleaved.permute(0, 2, 1, 3)  # [E, K/4, N, 4]
    scale_interleaved = scale_interleaved.reshape(
        ref_scale.shape[0],
        ref_scale.shape[2] // alignment,
        ref_scale.shape[1] * alignment,
    )  # [E, K/4, N*4]
    w_scale = scale_interleaved.contiguous()

    return w_q, w_scale
````
**EN:** This section defines `pack_interleave` and implements the core logic associated with pack interleave.
**CN:** 该部分定义 `pack_interleave`，并实现与 pack interleave 相关的核心逻辑。

### Lines 52-140: `test_int4_fp8_grouped_gemm_single_expert` definition
````python
@pytest.mark.skipif(
    not is_hopper(),
    reason="cutlass_w4a8_moe_mm is only supported on sm90",
)
@pytest.mark.parametrize("batch_size", [1, 2, 4, 8, 16])
def test_int4_fp8_grouped_gemm_single_expert(batch_size):
    # Test parameters
    num_experts = 1
    m = batch_size  # batch size
    k = 512  # input dimension
    n = 1024  # output dimension
    torch.manual_seed(0)
    dtype = torch.bfloat16
    device = "cuda"
    debug = False

    print(f"\nTesting with batch_size={batch_size}")

    # Create input tensors with ones
    if debug:
        a = torch.ones(m, k, dtype=torch.bfloat16, device=device)
        ref_w = torch.ones(num_experts, n, k, dtype=torch.int8, device=device)
        ref_w_scale = torch.ones(num_experts, n, k // 128, dtype=dtype, device=device)
    else:
        a = torch.randn(m, k, dtype=dtype, device=device)
        ref_w = torch.randint(
            -8, 8, (num_experts, n, k), dtype=torch.int8, device=device
        )
        affine_coeff = 0.005
        ref_w_scale = (
            torch.randn(num_experts, n, k // 128, dtype=dtype, device=device)
            * affine_coeff
        )

    w, w_scale = pack_interleave(num_experts, ref_w, ref_w_scale)

    # Create expert offsets and problem sizes
    expert_offsets = torch.tensor([0, m], dtype=torch.int32, device=device)
    problem_sizes = torch.tensor([[n, m, k]], dtype=torch.int32, device=device)

    a_strides = torch.full((num_experts, 3), k, device=device, dtype=torch.int64)
    c_strides = torch.full((num_experts, 3), n, device=device, dtype=torch.int64)
    b_strides = a_strides
    s_strides = c_strides

    # Quantize input
    a_q, a_scale = _per_tensor_quant_fp8(a)

    # Create output tensor
    c = torch.empty((m, n), dtype=torch.bfloat16, device=device)
    cutlass_w4a8_moe_mm(
        c,
        a_q,
        w,
        a_scale,
        w_scale,
        expert_offsets[:-1],
        problem_sizes,
        a_strides,
        b_strides,
        c_strides,
        s_strides,
        128,
        8,
    )
    c = c.to(dtype)

    # Reference implementation
    experts_selection_result = torch.full((m,), 0)
    c_ref = ref_grouped_gemm(
        c, a_q, a_scale, ref_w, ref_w_scale, num_experts, experts_selection_result
    )

    # Compare results
    try:
        torch.testing.assert_close(c, c_ref, rtol=1e-2, atol=0.1)
    except AssertionError as e:
        # torch.set_printoptions(threshold=10_000)
        print(f"  FAILURE: tensors are NOT close.")
        print(f"    Ref tensor: {c_ref.flatten()}")
        print(f"    Cutlass tensor: {c.flatten()}")
        print(
            f"    Max absolute difference: {torch.max(torch.abs(c.to(c_ref.dtype) - c_ref))}"
        )
        print(
            f"    Mean absolute difference: {torch.mean(torch.abs(c.to(c_ref.dtype) - c_ref))}"
        )
        print(f"    AssertionError: {e}")
        raise
````
**EN:** This section defines the test `test_int4_fp8_grouped_gemm_single_expert`. It sets up inputs, runs the target path, and checks the expected result. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_int4_fp8_grouped_gemm_single_expert`。它会准备输入、执行目标路径，并检查预期结果。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 143-156: `_per_tensor_quant_fp8` definition
````python
def _per_tensor_quant_fp8(
    x: torch.Tensor,
    dtype: torch.dtype = torch.float8_e4m3fn,
):
    assert x.is_contiguous(), "`x` is not contiguous"

    x_q = torch.empty_like(x, device=x.device, dtype=dtype)
    x_s = torch.empty(
        1,
        device=x.device,
        dtype=torch.float32,
    )
    per_tensor_quant_fp8(x, x_q, x_s, is_static=False)
    return x_q, x_s
````
**EN:** This section defines `_per_tensor_quant_fp8` and implements the core logic associated with per tensor quantization FP8. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `_per_tensor_quant_fp8`，并实现与 per tensor quantization FP8 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。

### Lines 159-264: `test_int4_fp8_grouped_gemm_multi_experts` definition
````python
@pytest.mark.skipif(
    not is_hopper(),
    reason="cutlass_w4a8_moe_mm is only supported on sm90",
)
@pytest.mark.parametrize("batch_size", [2, 4, 8, 16, 32])
@pytest.mark.parametrize("k", [256, 512, 1024, 2048, 4096, 7168])
@pytest.mark.parametrize("n", [256, 512, 1024, 2048, 7168])
@pytest.mark.parametrize("num_experts", [2, 4, 6, 8])
def test_int4_fp8_grouped_gemm_multi_experts(batch_size, k, n, num_experts):
    torch.manual_seed(0)
    dtype = torch.bfloat16
    device = "cuda"
    debug = False

    print(
        f"\nTesting with batch_size={batch_size}, k={k}, n={n}, num_experts={num_experts}"
    )

    if debug:
        a = torch.ones(batch_size, k, dtype=torch.bfloat16, device=device)
        ref_w = torch.ones(num_experts, n, k, dtype=torch.int8, device=device)
        ref_w_scale = torch.ones(num_experts, n, k // 128, dtype=dtype, device=device)
    else:
        a = torch.randn(batch_size, k, dtype=dtype, device=device)
        ref_w = torch.randint(
            -8, 8, (num_experts, n, k), dtype=torch.int8, device=device
        )
        affine_coeff = 0.005
        ref_w_scale = (
            torch.randn(num_experts, n, k // 128, dtype=dtype, device=device)
            * affine_coeff
        )

    w, w_scale = pack_interleave(num_experts, ref_w, ref_w_scale)

    # random select experts
    experts_selection_result = torch.randint(
        0, num_experts, (batch_size,), device=device
    )
    permutation = torch.argsort(experts_selection_result)
    expert_token_counts = torch.bincount(
        experts_selection_result, minlength=num_experts
    )

    # Create problem sizes and offsets for active experts
    problem_sizes = []
    for i in range(num_experts):
        problem_sizes.append([n, expert_token_counts[i].item(), k])
    problem_sizes = torch.tensor(problem_sizes, dtype=torch.int32, device=device)

    expert_offsets = []
    offset = 0
    for i in range(num_experts):
        expert_offsets.append(offset)
        offset += problem_sizes[i][1].item()
    expert_offsets = torch.tensor(expert_offsets, dtype=torch.int32, device=device)

    # Permute input and quantize
    a_q, a_scale = _per_tensor_quant_fp8(a)
    a_q_perm = a_q[permutation]

    # Create stride tensors
    a_strides = torch.full((num_experts, 3), k, device=device, dtype=torch.int64)
    c_strides = torch.full((num_experts, 3), n, device=device, dtype=torch.int64)
    b_strides = a_strides
    s_strides = c_strides

    c_perm = torch.empty((batch_size, n), dtype=torch.bfloat16, device=device)
    cutlass_w4a8_moe_mm(
        c_perm,
        a_q_perm,
        w,
        a_scale,
        w_scale,
        expert_offsets,
        problem_sizes,
        a_strides,
        b_strides,
        c_strides,
        s_strides,
        128,
        8,
    )

    # Un-permute the result
    c = torch.empty_like(c_perm)
    c[permutation] = c_perm
    c = c.to(dtype)

    c_ref = ref_grouped_gemm(
        c, a_q, a_scale, ref_w, ref_w_scale, num_experts, experts_selection_result
    )

    # Compare results
    try:
        torch.testing.assert_close(c, c_ref, rtol=1e-2, atol=0.1)
    except AssertionError as e:
        print(f"  FAILURE: tensors are NOT close.")
        print(
            f"    Max absolute difference: {torch.max(torch.abs(c.to(c_ref.dtype) - c_ref))}"
        )
        print(
            f"    Mean absolute difference: {torch.mean(torch.abs(c.to(c_ref.dtype) - c_ref))}"
        )
        print(f"    AssertionError: {e}")
        raise
````
**EN:** This section defines the test `test_int4_fp8_grouped_gemm_multi_experts`. It sets up inputs, runs the target path, and checks the expected result. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_int4_fp8_grouped_gemm_multi_experts`。它会准备输入、执行目标路径，并检查预期结果。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 267-283: `ref_grouped_gemm` definition
````python
def ref_grouped_gemm(
    c, a_q, a_scale, w, w_scale, num_experts, experts_selection_result
):
    dtype = torch.bfloat16
    c_ref = torch.zeros_like(c)
    for i in range(num_experts):
        token_idx = torch.where(experts_selection_result == i)[0]
        if len(token_idx) == 0:
            continue
        a = a_q[token_idx]

        ref_w_scale_repeat = w_scale[i].repeat_interleave(128, dim=1).to(torch.float32)
        ref_w = w[i].to(torch.float32) * ref_w_scale_repeat
        c = torch.matmul(a.to(torch.float32), ref_w.t()) * a_scale
        c_ref[token_idx] = c.to(dtype)

    return c_ref
````
**EN:** This section defines `ref_grouped_gemm` and implements the core logic associated with ref grouped GEMM.
**CN:** 该部分定义 `ref_grouped_gemm`，并实现与 ref grouped GEMM 相关的核心逻辑。

### Lines 286-287: Command-line entry point
````python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `pack_int4_values_to_int8`, `pack_interleave`, `test_int4_fp8_grouped_gemm_single_expert`, `_per_tensor_quant_fp8`, `test_int4_fp8_grouped_gemm_multi_experts`, `ref_grouped_gemm`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`, `sglang.jit_kernel.per_tensor_quant_fp8`
- **External / 外部**: `pytest`, `sys`, `torch`, `utils`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
