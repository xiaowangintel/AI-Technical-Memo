# test_cpu_fp8_scaled_mm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/quantization/test_cpu_fp8_scaled_mm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / quantization / test_cpu_fp8_scaled_mm, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / quantization / test_cpu_fp8_scaled_mm 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-6)
```python
"""Tests for CPU FP8 W8A16 block-scaled GEMM kernel (fp8_scaled_mm_cpu).

Run `pytest tests/kernels/quantization/test_cpu_fp8_scaled_mm.py -v`.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 8-12)
```python
import pytest
import torch

from vllm import _custom_ops as ops
from vllm.platforms import current_platform
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; and vLLM components like vllm, vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；vLLM 内部组件，例如 vllm、vllm.platforms。

### Top-level block starting at line 14 (lines 14-15)
```python
if not current_platform.is_cpu():
    pytest.skip("skipping CPU-only tests", allow_module_level=True)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 17 (lines 17-18)
```python
if not ops._supports_cpu_fp8_w8a16:
    pytest.skip("fp8_scaled_mm_cpu op not available", allow_module_level=True)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 20-20)
```python
BLOCK_SIZE = [128, 128]
```
**EN:** This block centralizes shared constants and parameter grids, including BLOCK_SIZE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 BLOCK_SIZE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `cdiv` (lines 23-24)
```python
def cdiv(a: int, b: int) -> int:
    return -(a // -b)
```
**EN:** This helper function implements the shared logic for cdiv. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 cdiv 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `quantize_weight_block_fp8` (lines 27-71)
```python
def quantize_weight_block_fp8(
    weight: torch.Tensor,
    block_size: list[int],
) -> tuple[torch.Tensor, torch.Tensor]:
    """Quantize weight [N, K] to FP8 with block scales.

    Returns:
        fp8_weight: [N, K] float8_e4m3fn
        scales: [n_tiles, k_tiles] float32
    """
    N, K = weight.shape
    block_n, block_k = block_size
    fp8_max = torch.finfo(torch.float8_e4m3fn).max

    n_tiles = cdiv(N, block_n)
    k_tiles = cdiv(K, block_k)

    # Pad for even blocking
    pad_N = (block_n - (N % block_n)) % block_n
    pad_K = (block_k - (K % block_k)) % block_k
    if pad_N > 0 or pad_K > 0:
        weight = torch.nn.functional.pad(weight, (0, pad_K, 0, pad_N))

    # Reshape into blocks
    w_blocks = weight.view(n_tiles, block_n, k_tiles, block_k)
    w_blocks = w_blocks.permute(0, 2, 1, 3).contiguous()

    # Per-block scale
    abs_max = w_blocks.abs().amax(dim=(-2, -1), keepdim=True)
    scales = abs_max / fp8_max
    scales = torch.where(scales == 0, torch.ones_like(scales), scales)

    # Quantize
    q_fp8 = (w_blocks / scales).clamp(-fp8_max, fp8_max).to(torch.float8_e4m3fn)

    # Reshape back
    fp8_weight = (
        q_fp8.permute(0, 2, 1, 3)
        .contiguous()
        .view(N + pad_N, K + pad_K)[:N, :K]
        .contiguous()
    )

    scales = scales.view(n_tiles, k_tiles)
    return fp8_weight, scales
```
**EN:** This helper function implements the shared logic for quantize weight block FP8. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 quantize weight block FP8 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `dequant_weight_block_fp8` (lines 74-96)
```python
def dequant_weight_block_fp8(
    fp8_weight: torch.Tensor,
    scales: torch.Tensor,
    block_size: list[int],
    out_dtype: torch.dtype,
) -> torch.Tensor:
    """Dequantize FP8 weight back to float for reference computation."""
    N, K = fp8_weight.shape
    block_n, block_k = block_size
    n_tiles, k_tiles = scales.shape

    pad_N = (block_n - (N % block_n)) % block_n
    pad_K = (block_k - (K % block_k)) % block_k
    if pad_N > 0 or pad_K > 0:
        fp8_padded = torch.nn.functional.pad(fp8_weight.float(), (0, pad_K, 0, pad_N))
    else:
        fp8_padded = fp8_weight.float()

    w_blocks = fp8_padded.view(n_tiles, block_n, k_tiles, block_k)
    w_blocks = w_blocks.permute(0, 2, 1, 3).contiguous()
    dq = w_blocks * scales.view(n_tiles, k_tiles, 1, 1)
    dq = dq.permute(0, 2, 1, 3).contiguous().view(N + pad_N, K + pad_K)
    return dq[:N, :K].to(out_dtype)
```
**EN:** This helper function implements the shared logic for dequant weight block FP8. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 dequant weight block FP8 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `ref_fp8_block_scaled_mm` (lines 99-112)
```python
def ref_fp8_block_scaled_mm(
    x: torch.Tensor,
    fp8_weight: torch.Tensor,
    scales: torch.Tensor,
    block_size: list[int],
    bias: torch.Tensor | None,
    out_dtype: torch.dtype,
) -> torch.Tensor:
    """Reference: dequant FP8→float32, matmul in float32, cast to out_dtype."""
    w_dq = dequant_weight_block_fp8(fp8_weight, scales, block_size, torch.float32)
    out = torch.mm(x.float(), w_dq.t())
    if bias is not None:
        out = out + bias.float()
    return out.to(out_dtype)
```
**EN:** This helper acts as a reference implementation for FP8 block scaled mm. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数为 FP8 block scaled mm 提供参考实现。 它把周边模块中的共用逻辑封装成可复用单元。

### Constants and module state (lines 118-128)
```python
M_SIZES = [1, 4, 16, 64, 128]
# (N, K) — weight shape is [N, K], output has N columns.
NK_SIZES = [
    (128, 256),
    (256, 512),
    (512, 1024),
    (1024, 2048),
    (5120, 5120),
    (17408, 5120),
    (5120, 17408),
]
```
**EN:** This block centralizes shared constants and parameter grids, including M_SIZES, NK_SIZES. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 M_SIZES、NK_SIZES。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_cpu_fp8_scaled_mm` (lines 131-162)
```python
@pytest.mark.parametrize("M", M_SIZES)
@pytest.mark.parametrize("N,K", NK_SIZES)
@pytest.mark.parametrize("use_bias", [False, True])
def test_cpu_fp8_scaled_mm(M: int, N: int, K: int, use_bias: bool):
    """fp8_scaled_mm_cpu correctness against float reference."""
    torch.manual_seed(42)
    out_dtype = torch.bfloat16
    block_size = BLOCK_SIZE

    x = torch.randn(M, K, dtype=out_dtype) / (K**0.5)
    w_f32 = torch.randn(N, K, dtype=torch.float32) / (K**0.5)
    fp8_weight, scales = quantize_weight_block_fp8(w_f32, block_size)

    bias = torch.randn(N, dtype=torch.float32) * 0.1 if use_bias else None

    ref_out = ref_fp8_block_scaled_mm(
        x, fp8_weight, scales, block_size, bias, out_dtype
    )

    packed_weight = torch.ops._C.convert_weight_packed(fp8_weight)
    kernel_out = ops.fp8_scaled_mm_cpu(
        x,
        packed_weight,
        scales,
        block_size,
        bias,
        out_dtype,
        True,
    )

    assert kernel_out.dtype == out_dtype
    torch.testing.assert_close(kernel_out, ref_out, rtol=0.02, atol=0.01)
```
**EN:** This pytest case verifies CPU FP8 scaled mm. It is parameterized across 3 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as M, N, K, use_bias. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 CPU FP8 scaled mm 的行为。 它通过 3 组参数化输入覆盖多种场景；它会使用诸如 M、N、K、use_bias 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm -> _custom_ops`
- `vllm.platforms -> current_platform`
