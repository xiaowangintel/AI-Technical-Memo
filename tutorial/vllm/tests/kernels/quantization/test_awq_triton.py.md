# test_awq_triton.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/quantization/test_awq_triton.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / quantization / test_awq_triton, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / quantization / test_awq_triton 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-6)
```python
"""Tests for the AWQ Triton kernel.

Run `pytest tests/kernels/quantization/test_awq_triton.py`.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 8-16)
```python
import pytest
import torch

from vllm.model_executor.layers.quantization.awq_triton import (
    AWQ_TRITON_SUPPORTED_GROUP_SIZES,
    awq_dequantize_triton,
    awq_gemm_triton,
)
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; and vLLM components like vllm.model_executor.layers.quantization.awq_triton, vllm.utils.torch_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；vLLM 内部组件，例如 vllm.model_executor.layers.quantization.awq_triton、vllm.utils.torch_utils。

### Constants and module state (lines 18-18)
```python
device = "cuda"
```
**EN:** This block prepares module-level state that later helpers and tests reuse.
**CN:** 该代码块准备了后续辅助函数和测试会复用的模块级状态。

### Function `reverse_awq_order` (lines 21-34)
```python
def reverse_awq_order(t: torch.Tensor):
    bits = 4
    AWQ_REVERSE_ORDER = [0, 4, 1, 5, 2, 6, 3, 7]
    reverse_order_tensor = torch.arange(
        t.shape[-1],
        dtype=torch.int32,
        device=t.device,
    )
    reverse_order_tensor = reverse_order_tensor.view(-1, 32 // bits)
    reverse_order_tensor = reverse_order_tensor[:, AWQ_REVERSE_ORDER]
    reverse_order_tensor = reverse_order_tensor.view(-1)

    t = t[:, reverse_order_tensor] & 0xF
    return t
```
**EN:** This helper function implements the shared logic for reverse awq order. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 reverse awq order 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `awq_dequantize_torch` (lines 40-68)
```python
def awq_dequantize_torch(
    qweight: torch.Tensor, scales: torch.Tensor, qzeros: torch.Tensor, group_size: int
) -> torch.Tensor:
    if group_size == -1:
        group_size = qweight.shape[0]

    bits = 4
    shifts = torch.arange(0, 32, bits, device=qzeros.device)

    iweights = torch.bitwise_right_shift(qweight[:, :, None], shifts[None, None, :]).to(
        torch.int8
    )

    iweights = iweights.view(iweights.shape[0], -1)

    zeros = torch.bitwise_right_shift(qzeros[:, :, None], shifts[None, None, :]).to(
        torch.int8
    )
    zeros = zeros.view(qzeros.shape[0], -1)
    zeros = reverse_awq_order(zeros)

    iweights = reverse_awq_order(iweights)

    iweights = torch.bitwise_and(iweights, (2**bits) - 1)
    zeros = torch.bitwise_and(zeros, (2**bits) - 1)

    scales = scales.repeat_interleave(group_size, dim=0)
    zeros = zeros.repeat_interleave(group_size, dim=0)
    return (iweights - zeros) * scales
```
**EN:** This helper function implements the shared logic for awq dequantize torch. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 awq dequantize torch 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_dequantize` (lines 74-115)
```python
@pytest.mark.parametrize("qweight_rows", [3584, 18944, 128, 256, 512, 1024])
@pytest.mark.parametrize("qweight_cols", [448, 576, 4736, 16, 32, 64, 128])
@pytest.mark.parametrize("group_size", AWQ_TRITON_SUPPORTED_GROUP_SIZES)
def test_dequantize(qweight_rows, qweight_cols, group_size):
    if group_size == -1:
        group_size = qweight_rows

    qweight_dtype = torch.int32
    scales_rows = qweight_rows // group_size
    scales_cols = qweight_cols * 8
    scales_dtype = torch.float16
    zeros_rows = scales_rows
    zeros_cols = qweight_cols
    zeros_dtype = torch.int32

    set_random_seed(0)

    qweight = torch.randint(
        0,
        torch.iinfo(torch.int32).max,
        (qweight_rows, qweight_cols),
        dtype=qweight_dtype,
        device=device,
    )
    scales = torch.rand(scales_rows, scales_cols, dtype=scales_dtype, device=device)
    zeros = torch.randint(
        0,
        torch.iinfo(torch.int32).max,
        (zeros_rows, zeros_cols),
        dtype=zeros_dtype,
        device=device,
    )

    iweights_triton = awq_dequantize_triton(qweight, scales, zeros)

    assert not torch.any(torch.isinf(iweights_triton)) and not torch.any(
        torch.isnan(iweights_triton)
    )

    iweights_torch = awq_dequantize_torch(qweight, scales, zeros, group_size)

    torch.testing.assert_close(iweights_triton, iweights_torch)
```
**EN:** This pytest case verifies dequantize. It is parameterized across 3 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as qweight_rows, qweight_cols, group_size. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 dequantize 的行为。 它通过 3 组参数化输入覆盖多种场景；它会使用诸如 qweight_rows、qweight_cols、group_size 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

### Function `test_gemm` (lines 122-171)
```python
@pytest.mark.parametrize("N", [1, 2, 4, 8, 14, 17, 23, 32])
@pytest.mark.parametrize("K", [128])
@pytest.mark.parametrize("M", [16, 24, 32])
@pytest.mark.parametrize("group_size", AWQ_TRITON_SUPPORTED_GROUP_SIZES)
@pytest.mark.parametrize("splitK", [1, 8])
def test_gemm(N, K, M, splitK, group_size):
    if group_size == -1:
        group_size = K

    split_k_iters = splitK

    input_rows = N
    input_cols = K
    input_dtype = torch.float32
    qweight_rows = input_cols
    qweight_cols = M // 8
    scales_rows = qweight_rows // group_size
    scales_cols = M
    scales_dtype = torch.float32
    qzeros_rows = scales_rows
    qzeros_cols = qweight_cols

    set_random_seed(0)

    input = torch.rand((input_rows, input_cols), dtype=input_dtype, device=device)
    qweight = torch.randint(
        0, torch.iinfo(torch.int32).max, (qweight_rows, qweight_cols), device=device
    )
    qzeros = torch.randint(
        0, torch.iinfo(torch.int32).max, (qzeros_rows, qzeros_cols), device=device
    )
    scales = torch.rand((scales_rows, scales_cols), dtype=scales_dtype, device=device)

    output_triton = awq_gemm_triton(input, qweight, scales, qzeros, split_k_iters)

    assert not torch.any(torch.isinf(output_triton)) and not torch.any(
        torch.isnan(output_triton)
    )

    dequantized_weights = awq_dequantize_triton(qweight, scales, qzeros)

    output_torch = torch.matmul(input, dequantized_weights)

    assert not torch.any(torch.isinf(output_torch)) and not torch.any(
        torch.isnan(output_torch)
    )

    torch.testing.assert_close(
        output_triton.cpu(), output_torch.cpu(), atol=1e-1, rtol=1e-1
    )
```
**EN:** This pytest case verifies gemm. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as N, K, M, splitK. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 gemm 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 N、K、M、splitK 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm.model_executor.layers.quantization.awq_triton -> AWQ_TRITON_SUPPORTED_GROUP_SIZES, awq_dequantize_triton, awq_gemm_triton`
- `vllm.utils.torch_utils -> set_random_seed`
