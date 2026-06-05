# test_gptq_kernel.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_gptq_kernel.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `GPTQ kernel` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `GPTQ kernel` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Imports and module setup
````python
import sys

import pytest
import torch
from sgl_kernel import gptq_gemm

from sglang.srt.layers.quantization.utils import pack_cols, pack_rows
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 10-50: `torch_dequantize` definition
````python
def torch_dequantize(q_weight, q_zeros, scales, g_idx, use_shuffle, bit, K, N):
    assert bit == 4, "Reference dequantization only supports 4-bit"
    group_size = K // scales.shape[0]
    pack_factor = 32 // bit

    # unpack q_weight: (K//pack_factor, N) -> (K, N)
    unpacked_q_weight = torch.empty(
        q_weight.shape[0] * pack_factor,
        q_weight.shape[1],
        dtype=torch.uint8,
        device=q_weight.device,
    )
    for i in range(pack_factor):
        unpacked_q_weight[i::pack_factor, :] = (q_weight >> (i * 4)) & 0x0F

    # unpack q_zeros: (num_groups, N//pack_factor) -> (num_groups, N)
    unpacked_q_zeros = torch.empty(
        q_zeros.shape[0],
        q_zeros.shape[1] * pack_factor,
        dtype=torch.uint8,
        device=q_zeros.device,
    )
    for i in range(pack_factor):
        unpacked_q_zeros[:, i::pack_factor] = (q_zeros >> (i * 4)) & 0x0F

    unpacked_q_zeros += 1
    unpacked_q_zeros = unpacked_q_zeros.to(scales.dtype)

    scale_zeros = unpacked_q_zeros * scales  # (num_groups, N)

    current_g_idx = torch.tensor(
        [i // group_size for i in range(K)], dtype=torch.int32, device=q_weight.device
    )

    scale_mat = scales[current_g_idx]  # (K, N)
    scale_zeros_mat = scale_zeros[current_g_idx]  # (K, N)

    # dequant: weight * scale - scale_zeros
    dequantized_b = unpacked_q_weight.to(scales.dtype) * scale_mat - scale_zeros_mat

    return dequantized_b.reshape(K, N)
````
**EN:** This section defines `torch_dequantize` and implements the core logic associated with torch dequantize. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `torch_dequantize`，并实现与 torch dequantize 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。

### Lines 53-62: `torch_gptq_gemm` definition
````python
def torch_gptq_gemm(
    a, b_q_weight, b_gptq_qzeros, b_gptq_scales, b_g_idx, use_shuffle, bit
):
    K, N = a.shape[1], b_q_weight.shape[1]

    b_dequant = torch_dequantize(
        b_q_weight, b_gptq_qzeros, b_gptq_scales, b_g_idx, use_shuffle, bit, K, N
    )
    c = torch.matmul(a, b_dequant)
    return c
````
**EN:** This section defines `torch_gptq_gemm` and implements the core logic associated with torch GPTQ GEMM.
**CN:** 该部分定义 `torch_gptq_gemm`，并实现与 torch GPTQ GEMM 相关的核心逻辑。

### Lines 65-116: `_test_gptq_gemm_once` definition
````python
def _test_gptq_gemm_once(M, N, K, bit, group_size, use_shuffle, dtype, device="cuda"):

    b_fp = torch.randn(K, N, dtype=dtype, device=device)

    assert K % group_size == 0, "K must be divisible by group_size"
    num_groups = K // group_size

    if use_shuffle:
        return
    else:
        g_idx = torch.tensor(
            [i // group_size for i in range(K)], dtype=torch.int32, device=device
        )
        b_shuffled = b_fp[g_idx]

    b_grouped = b_shuffled.reshape(num_groups, group_size, N)

    b_max = torch.max(b_grouped, dim=1, keepdim=True)[0]
    b_min = torch.min(b_grouped, dim=1, keepdim=True)[0]

    scales = (b_max - b_min) / (2**bit - 1)
    scales = scales.clamp(min=1e-6)

    zeros_float = (-b_min / scales).round()

    q_b = (
        (b_grouped / scales + zeros_float).round().clamp(0, 2**bit - 1).to(torch.uint8)
    )

    q_zeros_unpacked = zeros_float.to(torch.uint8) - 1

    b_q_weight = pack_rows(q_b.reshape(K, N), bit, K, N)

    q_zeros_unpacked = q_zeros_unpacked.reshape(num_groups, N)
    b_gptq_qzeros = pack_cols(q_zeros_unpacked, bit, num_groups, N)
    b_gptq_scales = scales.squeeze(1)

    a = torch.randn(M, K, dtype=dtype, device=device)

    c_ref = torch_gptq_gemm(
        a, b_q_weight, b_gptq_qzeros, b_gptq_scales, g_idx, use_shuffle, bit
    )
    c_out = gptq_gemm(
        a, b_q_weight, b_gptq_qzeros, b_gptq_scales, g_idx, use_shuffle, bit
    )

    rtol = 4e-2
    atol = 4e-2
    torch.testing.assert_close(c_ref, c_out, rtol=rtol, atol=atol)
    print(
        f"✅ Test passed: M={M}, N={N}, K={K}, bit={bit}, group_size={group_size}, use_shuffle={use_shuffle}, dtype={dtype}"
    )
````
**EN:** This section defines `_test_gptq_gemm_once` and implements the core logic associated with test GPTQ GEMM once. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `_test_gptq_gemm_once`，并实现与 test GPTQ GEMM once 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。

### Lines 119-129: `test_gptq_gemm` definition
````python
@pytest.mark.parametrize("M", [1, 8, 128])
@pytest.mark.parametrize("N", [2048, 4096])
@pytest.mark.parametrize("K", [2048, 4096])
@pytest.mark.parametrize("bit", [4])
@pytest.mark.parametrize("group_size", [128])
@pytest.mark.parametrize("use_shuffle", [False])
@pytest.mark.parametrize("dtype", [torch.float16])
def test_gptq_gemm(M, N, K, bit, group_size, use_shuffle, dtype):
    if not torch.cuda.is_available():
        pytest.skip("CUDA not available")
    _test_gptq_gemm_once(M, N, K, bit, group_size, use_shuffle, dtype, "cuda")
````
**EN:** This section defines the test `test_gptq_gemm`. It sets up inputs, runs the target path, and checks the expected result. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_gptq_gemm`。它会准备输入、执行目标路径，并检查预期结果。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 132-133: Command-line entry point
````python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v"]))
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `torch_dequantize`, `torch_gptq_gemm`, `_test_gptq_gemm_once`, `test_gptq_gemm`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`, `sglang.srt.layers.quantization.utils`
- **External / 外部**: `pytest`, `sys`, `torch`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
