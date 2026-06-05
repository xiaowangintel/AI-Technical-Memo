# test_int8_kernel.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/quantization/test_int8_kernel.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / quantization / test_int8_kernel, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / quantization / test_int8_kernel 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 5-16)
```python
import itertools

import pytest
import torch

from vllm.model_executor.layers.activation import SiluAndMul
from vllm.model_executor.layers.fused_moe import fused_experts
from vllm.model_executor.layers.fused_moe.config import FusedMoEQuantConfig
from vllm.model_executor.layers.quantization.utils.int8_utils import (
    per_token_quant_int8,
)
from vllm.platforms import current_platform
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as itertools, pytest, torch; and vLLM components like vllm.model_executor.layers.activation, vllm.model_executor.layers.fused_moe, vllm.model_executor.layers.fused_moe.config, vllm.model_executor.layers.quantization.utils.int8_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 itertools、pytest、torch；vLLM 内部组件，例如 vllm.model_executor.layers.activation、vllm.model_executor.layers.fused_moe、vllm.model_executor.layers.fused_moe.config、vllm.model_executor.layers.quantization.utils.int8_utils。

### Top-level block starting at line 18 (lines 18-19)
```python
if current_platform.get_device_capability() < (7, 0):
    pytest.skip("INT8 Triton requires CUDA 7.0 or higher", allow_module_level=True)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Function `native_w8a8_per_token_matmul` (lines 22-42)
```python
def native_w8a8_per_token_matmul(A, B, As, Bs, output_dtype=torch.float16):
    """Matrix multiplication function that supports per-token input
    quantization and per-column weight quantization"""
    A = A.to(torch.float32)
    B = B.to(torch.float32)

    assert A.shape[-1] == B.shape[-1], "Dimension mismatch"
    assert B.ndim == 2 and B.is_contiguous(), "B must be a 2D contiguous tensor"

    # Reshape input
    M = A.numel() // A.shape[-1]
    B = B.t()  # Transpose weight matrix
    N, K = B.shape
    origin_C_shape = A.shape[:-1] + (K,)
    A = A.reshape(M, N)

    # As is per-token [M, 1], Bs is per-column [1, K]
    C = torch.matmul(A, B)  # [M, K]
    C = As * C * Bs.view(1, -1)  # Broadcast per-column scale

    return C.reshape(origin_C_shape).to(output_dtype)
```
**EN:** This helper function implements the shared logic for native w8a8 per token matmul. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 native w8a8 per token matmul 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `torch_w8a8_per_column_moe` (lines 45-82)
```python
def torch_w8a8_per_column_moe(a, w1, w2, w1_s, w2_s, topk, topk_weight, topk_ids):
    """This function performs fused moe with per-column int8 quantization
    using native torch."""

    B, D = a.shape
    # Perform per-token quantization
    a_q, a_s = per_token_quant_int8(a)
    # Repeat tokens to match topk
    a_q = a_q.view(B, -1, D).repeat(1, topk, 1).reshape(-1, D)
    # Also repeat the scale
    a_s = a_s.view(B, -1, 1).repeat(1, topk, 1).reshape(-1, 1)  # [B*topk, 1]

    out = torch.zeros(B * topk, w2.shape[1], dtype=a.dtype, device=a.device)

    # Calculate routing
    topk_weight = topk_weight.view(-1)
    topk_ids = topk_ids.view(-1)
    # Process each expert
    for i in range(w1.shape[0]):
        mask = topk_ids == i
        if mask.sum():
            # First MLP layer: note that a_s is now per-token
            inter_out = native_w8a8_per_token_matmul(
                a_q[mask], w1[i], a_s[mask], w1_s[i], output_dtype=a.dtype
            )
            # Activation function
            act_out = SiluAndMul().forward_native(inter_out)
            # Quantize activation output with per-token
            act_out_q, act_out_s = per_token_quant_int8(act_out)

            # Second MLP layer
            out[mask] = native_w8a8_per_token_matmul(
                act_out_q, w2[i], act_out_s, w2_s[i], output_dtype=a.dtype
            )
    # Apply routing weights and sum
    return (
        out.view(B, -1, w2.shape[1]) * topk_weight.view(B, -1, 1).to(out.dtype)
    ).sum(dim=1)
```
**EN:** This helper function implements the shared logic for torch w8a8 per column MoE. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 torch w8a8 per column MoE 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `setup_cuda` (lines 85-88)
```python
@pytest.fixture(autouse=True, scope="module")
def setup_cuda():
    """Sets the default CUDA device for all tests in this module."""
    torch.set_default_device("cuda")
```
**EN:** This fixture prepares reusable state for setup CUDA. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该 fixture 为 setup CUDA 准备可复用的测试状态。 它把周边模块中的共用逻辑封装成可复用单元。

### Constants and module state (lines 91-97)
```python
DTYPES = [torch.half, torch.bfloat16]
M = [1, 33]
N = [128, 1024]
K = [256, 4096]
E = [8]
TOP_KS = [2, 6]
SEEDS = [0]
```
**EN:** This block centralizes shared constants and parameter grids, including DTYPES, M, N, K, E. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 DTYPES、M、N、K、E。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_w8a8_fp8_fused_moe` (lines 100-155)
```python
@pytest.mark.parametrize(
    "M, N, K, E, topk, dtype, seed",
    itertools.product(M, N, K, E, TOP_KS, DTYPES, SEEDS),
)
@torch.inference_mode()
def test_w8a8_fp8_fused_moe(default_vllm_config, M, N, K, E, topk, dtype, seed):
    torch.manual_seed(seed)
    # Initialize int8 quantization parameters
    factor_for_scale = 1e-2
    int8_max = 127
    int8_min = -128

    # Input tensor
    # M * K
    a = torch.randn((M, K), dtype=dtype) / 10

    # Generate int8 weights
    w1_fp32 = (torch.rand((E, 2 * N, K), dtype=torch.float32) - 0.5) * 2
    w1 = (w1_fp32 * int8_max).clamp(min=int8_min, max=int8_max).to(torch.int8)

    w2_fp32 = (torch.rand((E, K, N), dtype=torch.float32) - 0.5) * 2
    w2 = (w2_fp32 * int8_max).clamp(min=int8_min, max=int8_max).to(torch.int8)

    # Generate scale for each column (per-column quantization)
    w1_s = torch.rand(E, 2 * N, device=w1_fp32.device) * factor_for_scale
    w2_s = torch.rand(E, K, device=w2_fp32.device) * factor_for_scale
    score = torch.randn((M, E), dtype=dtype)
    score = torch.softmax(score, dim=-1, dtype=torch.float32)
    topk_weights, topk_ids = torch.topk(score, topk)

    ref_out = torch_w8a8_per_column_moe(
        a, w1, w2, w1_s, w2_s, topk, topk_weights, topk_ids
    )

    quant_config = FusedMoEQuantConfig.make(
        torch.int8,
        per_act_token_quant=True,
        block_shape=None,
        w1_scale=w1_s,
        w2_scale=w2_s,
    )

    out = fused_experts(
        a,
        w1,
        w2,
        topk_weights,
        topk_ids,
        quant_config=quant_config,
    )

    # Check results
    rel_diff = torch.mean(
        torch.abs(out.to(torch.float32) - ref_out.to(torch.float32))
    ) / torch.mean(torch.abs(ref_out.to(torch.float32)))
    assert rel_diff < 0.05
```
**EN:** This pytest case verifies w8a8 FP8 fused MoE. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as default_vllm_config, M, N, K. assertions at the end lock in the intended behavior or graph shape. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 w8a8 FP8 fused MoE 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 default_vllm_config、M、N、K 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态；代码会控制随机性以保证场景可复现。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。
- **Reusable fixtures / 可复用 fixture:** Fixtures package common setup so each test can focus on the scenario under study. / fixture 封装公共准备逻辑，让每个测试更专注于待验证场景。

## Dependencies / 依赖关系
- `itertools`
- `pytest`
- `torch`
- `vllm.model_executor.layers.activation -> SiluAndMul`
- `vllm.model_executor.layers.fused_moe -> fused_experts`
- `vllm.model_executor.layers.fused_moe.config -> FusedMoEQuantConfig`
- `vllm.model_executor.layers.quantization.utils.int8_utils -> per_token_quant_int8`
- `vllm.platforms -> current_platform`
