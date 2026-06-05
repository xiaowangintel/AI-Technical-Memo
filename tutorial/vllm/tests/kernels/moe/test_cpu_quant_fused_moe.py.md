# test_cpu_quant_fused_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_cpu_quant_fused_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_cpu_quant_fused_moe, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_cpu_quant_fused_moe 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-3)
```python
"""Tests for CPU quantized fused MoE kernels (FP8 W8A16 and MXFP4 W4A16)."""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 5-13)
```python
import math
import sys

import pytest
import torch
import torch.nn.functional as F

from vllm.platforms import current_platform
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as math, sys, pytest, torch; and vLLM components like vllm.platforms, vllm.utils.torch_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 math、sys、pytest、torch；vLLM 内部组件，例如 vllm.platforms、vllm.utils.torch_utils。

### Top-level block starting at line 15 (lines 15-16)
```python
if not current_platform.is_cpu():
    pytest.skip("skipping CPU-only tests", allow_module_level=True)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Imports and shared setup (lines 18-18)
```python
import vllm._custom_ops as ops  # noqa: E402
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in and vLLM components like vllm._custom_ops.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括vLLM 内部组件，例如 vllm._custom_ops。

### Top-level block starting at line 20 (lines 20-21)
```python
if not hasattr(torch.ops._C, "fused_experts_cpu"):
    pytest.skip("fused_experts_cpu op not available", allow_module_level=True)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Function `_silu_and_mul` (lines 24-26)
```python
def _silu_and_mul(x: torch.Tensor) -> torch.Tensor:
    d = x.shape[-1] // 2
    return F.silu(x[..., :d]) * x[..., d:]
```
**EN:** This helper function implements the shared logic for silu and mul. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 silu and mul 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `_prepack_experts` (lines 29-31)
```python
def _prepack_experts(w: torch.Tensor) -> torch.Tensor:
    """VNNI-prepack expert weights via ``convert_weight_packed``."""
    return torch.ops._C.convert_weight_packed(w)
```
**EN:** This helper function implements the shared logic for prepack experts. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 prepack experts 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Constants and module state (lines 36-40)
```python
BLOCK_SIZE = [128, 128]  # [block_n, block_k]

_FP8_INFO = torch.finfo(torch.float8_e4m3fn)
FP8_SCALE = _FP8_INFO.max  # 448.0
FACTOR_FOR_SCALE = 1e-3
```
**EN:** This block centralizes shared constants and parameter grids, including BLOCK_SIZE, _FP8_INFO, FP8_SCALE, FACTOR_FOR_SCALE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 BLOCK_SIZE、_FP8_INFO、FP8_SCALE、FACTOR_FOR_SCALE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `_block_dequant_weight` (lines 43-76)
```python
def _block_dequant_weight(
    weight: torch.Tensor,
    scales: torch.Tensor,
    block_size: list[int],
) -> torch.Tensor:
    """Block-dequantize FP8 weight [E, N, K] -> float [E, N, K]."""
    E, N, K = weight.shape
    block_n, block_k = block_size
    pad_N = (block_n - N % block_n) % block_n
    pad_K = (block_k - K % block_k) % block_k

    if pad_N > 0 or pad_K > 0:
        weight = F.pad(weight, (0, pad_K, 0, pad_N))

    n_tiles = math.ceil(N / block_n)
    k_tiles = math.ceil(K / block_k)

    weight_block = (
        weight.view(E, n_tiles, block_n, k_tiles, block_k)
        .permute(0, 1, 3, 2, 4)
        .float()
        .contiguous()
    )
    weight_scaled = (
        (weight_block * scales.view(E, n_tiles, k_tiles, 1, 1))
        .permute(0, 1, 3, 2, 4)
        .contiguous()
    )
    if pad_N > 0 or pad_K > 0:
        weight_scaled = weight_scaled.view(E, N + pad_N, K + pad_K)
        weight_scaled = weight_scaled[..., :N, :K].contiguous()
    else:
        weight_scaled = weight_scaled.view(E, N, K)
    return weight_scaled
```
**EN:** This helper function implements the shared logic for block dequant weight. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 block dequant weight 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `ref_w8a16_block_fp8_moe` (lines 79-113)
```python
def ref_w8a16_block_fp8_moe(
    a: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    w1_s: torch.Tensor,
    w2_s: torch.Tensor,
    topk_weight: torch.Tensor,
    topk_ids: torch.Tensor,
    block_size: list[int],
) -> torch.Tensor:
    """Reference FP8 W8A16 block-scaled fused MoE in pure torch."""
    B, D = a.shape
    topk = topk_ids.size(1)

    w1_dq = _block_dequant_weight(w1, w1_s, block_size)
    w2_dq = _block_dequant_weight(w2, w2_s, block_size)

    a_exp = a.view(B, -1, D).repeat(1, topk, 1).reshape(-1, D).float()
    out = torch.zeros(B * topk, w2_dq.shape[1], dtype=torch.float32)

    topk_weight_flat = topk_weight.view(-1)
    topk_ids_flat = topk_ids.view(-1)

    for i in range(w1_dq.shape[0]):
        mask = topk_ids_flat == i
        if mask.sum():
            ic0 = torch.matmul(a_exp[mask], w1_dq[i].transpose(0, 1))
            ic1 = _silu_and_mul(ic0)
            out[mask] = torch.matmul(ic1, w2_dq[i].transpose(0, 1))

    return (
        (out.view(B, -1, w2_dq.shape[1]) * topk_weight_flat.view(B, -1, 1))
        .sum(dim=1)
        .to(a.dtype)
    )
```
**EN:** This helper acts as a reference implementation for w8a16 block FP8 MoE. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数为 w8a16 block FP8 MoE 提供参考实现。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `_make_fp8_moe_weights` (lines 116-144)
```python
def _make_fp8_moe_weights(
    E: int,
    N: int,
    K: int,
    block_size: list[int],
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
    """Generate random FP8 MoE weights with random block scales."""
    block_n, block_k = block_size

    w1 = (
        (torch.randn(E, 2 * N, K) * FP8_SCALE)
        .clamp(min=-FP8_SCALE, max=FP8_SCALE)
        .to(torch.float8_e4m3fn)
    )
    w2 = (
        (torch.randn(E, K, N) * FP8_SCALE)
        .clamp(min=-FP8_SCALE, max=FP8_SCALE)
        .to(torch.float8_e4m3fn)
    )

    w1_s = (
        torch.randn(E, math.ceil(2 * N / block_n), math.ceil(K / block_k))
        * FACTOR_FOR_SCALE
    )
    w2_s = (
        torch.randn(E, math.ceil(K / block_n), math.ceil(N / block_k))
        * FACTOR_FOR_SCALE
    )
    return w1, w2, w1_s, w2_s
```
**EN:** This helper function implements the shared logic for make FP8 MoE weights. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 make FP8 MoE weights 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Constants and module state (lines 147-158)
```python
FP8_NUM_TOKENS = [1, 2, 64, 121]
FP8_MOE_CONFIGS = [
    (256, 512, 8, 2),
    (256, 512, 8, 4),
    (512, 256, 8, 2),
    (512, 256, 8, 4),
    (512, 512, 8, 2),
    (512, 512, 8, 4),
    (768, 2048, 8, 2),
    (768, 2048, 8, 4),
    (768, 2048, 128, 8),
]
```
**EN:** This block centralizes shared constants and parameter grids, including FP8_NUM_TOKENS, FP8_MOE_CONFIGS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 FP8_NUM_TOKENS、FP8_MOE_CONFIGS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_w8a16_block_fp8_cpu_fused_moe` (lines 161-216)
```python
@pytest.mark.parametrize("M", FP8_NUM_TOKENS)
@pytest.mark.parametrize("N,K,E,topk", FP8_MOE_CONFIGS)
@pytest.mark.parametrize("seed", [0])
def test_w8a16_block_fp8_cpu_fused_moe(M, N, K, E, topk, seed):
    """Test fused_experts_cpu FP8 W8A16 against dequantised torch reference."""
    set_random_seed(seed)

    a = torch.randn(M, K, dtype=torch.bfloat16) / math.sqrt(K)
    w1, w2, w1_s, w2_s = _make_fp8_moe_weights(E, N, K, BLOCK_SIZE)

    score = torch.randn(M, E, dtype=torch.bfloat16)
    score = torch.softmax(score, dim=-1, dtype=torch.float32)
    topk_weight, topk_ids = torch.topk(score, topk)
    topk_ids = topk_ids.to(torch.int32)

    ref_out = ref_w8a16_block_fp8_moe(
        a, w1, w2, w1_s, w2_s, topk_weight, topk_ids, BLOCK_SIZE
    )

    pw1, pw2 = _prepack_experts(w1), _prepack_experts(w2)

    # Test inplace=False against reference
    out = ops.fused_experts_cpu(
        a.clone(),
        pw1,
        pw2,
        topk_weight,
        topk_ids,
        False,
        ops.CPUQuantMethod.FP8_W8A16,
        w1_s,
        w2_s,
        None,
        None,
        BLOCK_SIZE,
        is_vnni=True,
    )
    torch.testing.assert_close(ref_out.bfloat16(), out, atol=1e-2, rtol=1e-2)

    # Test inplace=True produces identical output
    out_inplace = ops.fused_experts_cpu(
        a.clone(),
        pw1,
        pw2,
        topk_weight,
        topk_ids,
        True,
        ops.CPUQuantMethod.FP8_W8A16,
        w1_s,
        w2_s,
        None,
        None,
        BLOCK_SIZE,
        is_vnni=True,
    )
    torch.testing.assert_close(out_inplace, out, atol=0, rtol=0)
```
**EN:** This pytest case verifies w8a16 block FP8 CPU fused MoE. It is parameterized across 3 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as M, N, K, E. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 w8a16 block FP8 CPU fused MoE 的行为。 它通过 3 组参数化输入覆盖多种场景；它会使用诸如 M、N、K、E 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

### Class `MXFP4QuantizeUtil` (lines 222-229)
```python
class MXFP4QuantizeUtil:
    """MXFP4 quantization utility."""

    E2M1_max = 6.0
    E2M1_values = [0, 0.5, 1, 1.5, 2, 3, 4, 6]
    E2M1_bounds = torch.tensor([0.25, 0.75, 1.25, 1.75, 2.5, 3.5, 5])
    block_size = 32
```
**EN:** This helper class groups the state and behavior needed for MXFP4QuantizeUtil. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 MXFP4QuantizeUtil 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `MXFP4QuantizeUtil.quantize` (lines 230-261)
```python
    @classmethod
    def quantize(cls, input: torch.Tensor) -> tuple:
        """Quantize BF16 tensor to MXFP4 packed uint8 format."""

        def cast_fp4(x):
            sign = torch.sign(x)
            sign_bit = (2 - sign) // 2
            ord_ = torch.sum(
                (x.abs().unsqueeze(-1) - cls.E2M1_bounds.to(x.device)) > 0, dim=-1
            )
            fp4_val = (sign_bit * 0b1000 + ord_).to(torch.uint8)
            return fp4_val

        def fuse_uint4_to_uint8(x):
            left_side = x[..., 0::2]
            right_side = x[..., 1::2]
            new_data = right_side.clone() << 4
            new_data[..., : left_side.shape[-1]] += left_side
            return new_data

        original_shape = input.shape
        input = input.view(-1, cls.block_size)
        input_amax = input.abs().max(dim=-1, keepdim=True).values
        descale = input_amax / cls.E2M1_max
        min_value = torch.tensor(-127.0, device=descale.device)
        e8m0_scale = torch.ceil(torch.maximum(torch.log2(descale), min_value))

        input = (input / torch.exp2(e8m0_scale)).view(original_shape)
        input_q = cast_fp4(input)
        input_q = fuse_uint4_to_uint8(input_q)
        e8m0_scale = (e8m0_scale + 127).to(torch.uint8)
        return input_q, e8m0_scale
```
**EN:** This method on `MXFP4QuantizeUtil` implements quantize. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `MXFP4QuantizeUtil` 中的这个方法实现了 quantize。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `MXFP4QuantizeUtil.dequantize` (lines 263-289)
```python
    @classmethod
    def dequantize(cls, quantized_data, dtype: torch.dtype, scale):
        """Dequantize MXFP4 packed tensor back to float."""

        def unfuse_uint8_to_uint4(x):
            left_side = x & 0x0F
            right_side = (x >> 4) & 0x0F
            shape = list(x.shape)
            shape[-1] = shape[-1] * 2
            result = torch.zeros(shape, dtype=torch.uint8, device=x.device)
            result[..., 0::2] = left_side
            result[..., 1::2] = right_side
            return result

        e8m0_scale = scale
        x_unfused = unfuse_uint8_to_uint4(quantized_data)
        sign = 1 - 2 * ((x_unfused & 0b1000) >> 3).to(torch.float32)
        magnitude = (x_unfused & 0b0111).to(torch.long)
        values = torch.tensor(cls.E2M1_values, device=quantized_data.device)
        original_shape = magnitude.shape
        x_float = values[magnitude.reshape(-1)].reshape(original_shape)
        x_float = sign.float() * x_float
        x_float = x_float.reshape(-1, cls.block_size)
        scale_factor = torch.exp2(e8m0_scale.float() - 127)
        scale_factor = scale_factor.reshape(-1, 1)
        x_float = x_float * scale_factor
        return x_float.reshape(original_shape).to(dtype)
```
**EN:** This method on `MXFP4QuantizeUtil` implements dequantize. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `MXFP4QuantizeUtil` 中的这个方法实现了 dequantize。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `_swiglu` (lines 292-300)
```python
def _swiglu(x: torch.Tensor, alpha: float, limit: float) -> torch.Tensor:
    """SwigLU activation used in GPT-OSS.
    Input is interleaved: [gate_0, up_0, gate_1, up_1, ...] in last dim.
    """
    gate = x[..., 0::2]
    up = x[..., 1::2]
    gate_clamped = torch.clamp(gate, max=limit)
    up_clamped = torch.clamp(up, min=-limit, max=limit)
    return gate_clamped * torch.sigmoid(alpha * gate_clamped) * (up_clamped + 1)
```
**EN:** This helper function implements the shared logic for swiglu. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 swiglu 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `ref_mxfp4_fused_moe` (lines 303-327)
```python
def ref_mxfp4_fused_moe(
    a: torch.Tensor,
    w1_dq: torch.Tensor,
    w2_dq: torch.Tensor,
    topk_weight: torch.Tensor,
    topk_ids: torch.Tensor,
    topk: int,
) -> torch.Tensor:
    """Reference MXFP4 fused MoE with SiLU activation."""
    B, D = a.shape
    a_f = a.float()
    out = torch.zeros(B * topk, w2_dq.shape[1], dtype=torch.float32)
    topk_ids_flat = topk_ids.view(-1)

    for i in range(w1_dq.shape[0]):
        mask = topk_ids_flat == i
        if mask.sum() == 0:
            continue
        token_indices = torch.where(mask)[0]
        source_indices = token_indices // topk
        ic0 = torch.matmul(a_f[source_indices], w1_dq[i].float().T)
        ic1 = _silu_and_mul(ic0)
        out[mask] = torch.matmul(ic1, w2_dq[i].float().T)

    return (out.view(B, topk, -1) * topk_weight.unsqueeze(-1)).sum(dim=1).to(a.dtype)
```
**EN:** This helper acts as a reference implementation for mxfp4 fused MoE. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数为 mxfp4 fused MoE 提供参考实现。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `ref_mxfp4_fused_moe_gptoss` (lines 330-362)
```python
def ref_mxfp4_fused_moe_gptoss(
    a: torch.Tensor,
    w1_dq: torch.Tensor,
    w2_dq: torch.Tensor,
    w1_bias: torch.Tensor,
    w2_bias: torch.Tensor,
    topk_weight: torch.Tensor,
    topk_ids: torch.Tensor,
    alpha: float,
    limit: float,
) -> torch.Tensor:
    """Reference MXFP4 fused MoE with SwigLU+bias (GPT-OSS style)."""
    B, D = a.shape
    topk = topk_ids.shape[1]
    a_f = a.float()
    E = w1_dq.shape[0]
    out = torch.zeros(B * topk, w2_dq.shape[1], dtype=torch.float32)
    topk_ids_flat = topk_ids.view(-1)

    for i in range(E):
        mask = topk_ids_flat == i
        if mask.sum() == 0:
            continue
        token_indices = torch.where(mask)[0]
        source_indices = token_indices // topk
        ic0 = torch.matmul(a_f[source_indices], w1_dq[i].float().T)
        ic0 = ic0 + w1_bias[i].float()
        ic1 = _swiglu(ic0, alpha, limit)
        ic2 = torch.matmul(ic1, w2_dq[i].float().T)
        ic2 = ic2 + w2_bias[i].float()
        out[mask] = ic2

    return (out.view(B, topk, -1) * topk_weight.unsqueeze(-1)).sum(dim=1).to(a.dtype)
```
**EN:** This helper acts as a reference implementation for mxfp4 fused MoE gptoss. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数为 mxfp4 fused MoE gptoss 提供参考实现。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `_prepack_mxfp4_experts` (lines 365-371)
```python
def _prepack_mxfp4_experts(
    w: torch.Tensor, w_scale: torch.Tensor
) -> tuple[torch.Tensor, torch.Tensor]:
    """VNNI-prepack MXFP4 weights and repack scales."""
    packed_w = torch.ops._C.convert_weight_packed(w)
    packed_s = torch.ops._C.convert_scale_packed(w_scale)
    return packed_w, packed_s
```
**EN:** This helper function implements the shared logic for prepack mxfp4 experts. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 prepack mxfp4 experts 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Constants and module state (lines 374-380)
```python
MXFP4_NUM_TOKENS = [1, 2, 32, 121]
MXFP4_MOE_CONFIGS = [
    (128, 128, 4, 2),
    (256, 256, 8, 4),
    (352, 256, 8, 4),
    (512, 320, 8, 4),
]
```
**EN:** This block centralizes shared constants and parameter grids, including MXFP4_NUM_TOKENS, MXFP4_MOE_CONFIGS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 MXFP4_NUM_TOKENS、MXFP4_MOE_CONFIGS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_mxfp4_cpu_fused_moe` (lines 383-433)
```python
@pytest.mark.parametrize("M", MXFP4_NUM_TOKENS)
@pytest.mark.parametrize("N,K,E,topk", MXFP4_MOE_CONFIGS)
@pytest.mark.parametrize("seed", [0])
def test_mxfp4_cpu_fused_moe(M, N, K, E, topk, seed):
    """Test fused_experts_mxfp4_cpu against dequantized torch reference."""
    set_random_seed(seed)
    dtype = torch.bfloat16

    a = torch.randn(M, K, dtype=dtype) / 10

    # Generate and quantize weights
    w1_bf16 = torch.randn(E, 2 * N, K, dtype=dtype) / 10
    w1q, w1s = MXFP4QuantizeUtil.quantize(w1_bf16)
    w1s = w1s.reshape(E, 2 * N, K // 32)
    w1dq = MXFP4QuantizeUtil.dequantize(w1q, dtype, w1s)

    w2_bf16 = torch.randn(E, K, N, dtype=dtype) / 10
    w2q, w2s = MXFP4QuantizeUtil.quantize(w2_bf16)
    w2s = w2s.reshape(E, K, N // 32)
    w2dq = MXFP4QuantizeUtil.dequantize(w2q, dtype, w2s)

    # Routing
    score = torch.randn(M, E, dtype=dtype)
    score = torch.softmax(score, dim=-1, dtype=torch.float32)
    topk_weight, topk_ids = torch.topk(score, topk)
    topk_ids = topk_ids.to(torch.int32)

    # Reference
    ref_out = ref_mxfp4_fused_moe(a, w1dq, w2dq, topk_weight, topk_ids, topk)

    # Pack weights for kernel
    pw1, pw1s = _prepack_mxfp4_experts(w1q, w1s)
    pw2, pw2s = _prepack_mxfp4_experts(w2q, w2s)

    # Kernel
    out = ops.fused_experts_cpu(
        a.clone(),
        pw1,
        pw2,
        topk_weight,
        topk_ids,
        False,  # inplace
        ops.CPUQuantMethod.MXFP4,
        pw1s,  # w1_scale
        pw2s,  # w2_scale
        None,  # w1_zero
        None,  # w2_zero
        None,  # block_size
    )

    torch.testing.assert_close(ref_out.bfloat16(), out, atol=1e-2, rtol=1e-2)
```
**EN:** This pytest case verifies mxfp4 CPU fused MoE. It is parameterized across 3 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as M, N, K, E. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 mxfp4 CPU fused MoE 的行为。 它通过 3 组参数化输入覆盖多种场景；它会使用诸如 M、N、K、E 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

### Function `test_mxfp4_cpu_fused_moe_bias_swiglu` (lines 436-496)
```python
@pytest.mark.parametrize("M", [1, 32])
@pytest.mark.parametrize("N,K,E,topk", [(128, 128, 4, 2), (64, 64, 4, 2)])
@pytest.mark.parametrize("seed", [0])
def test_mxfp4_cpu_fused_moe_bias_swiglu(M, N, K, E, topk, seed):
    """Test fused_experts_mxfp4_cpu with bias and SwigLU activation (GPT-OSS)."""
    set_random_seed(seed)
    dtype = torch.bfloat16
    alpha = 1.702
    limit = 7.0

    a = torch.randn(M, K, dtype=dtype) / 10

    # Generate and quantize weights
    w1_bf16 = torch.randn(E, 2 * N, K, dtype=dtype) / 10
    w1q, w1s = MXFP4QuantizeUtil.quantize(w1_bf16)
    w1s = w1s.reshape(E, 2 * N, K // 32)
    w1dq = MXFP4QuantizeUtil.dequantize(w1q, dtype, w1s)
    w1_b = torch.randn(E, 2 * N, dtype=torch.float32) / 10

    w2_bf16 = torch.randn(E, K, N, dtype=dtype) / 10
    w2q, w2s = MXFP4QuantizeUtil.quantize(w2_bf16)
    w2s = w2s.reshape(E, K, N // 32)
    w2dq = MXFP4QuantizeUtil.dequantize(w2q, dtype, w2s)
    w2_b = torch.randn(E, K, dtype=torch.float32) / 10

    # Routing
    score = torch.randn(M, E, dtype=dtype)
    score = torch.softmax(score, dim=-1, dtype=torch.float32)
    topk_weight, topk_ids = torch.topk(score, topk)
    topk_ids = topk_ids.to(torch.int32)

    # Reference
    ref_out = ref_mxfp4_fused_moe_gptoss(
        a, w1dq, w2dq, w1_b, w2_b, topk_weight, topk_ids, alpha, limit
    )

    # Pack weights for kernel
    pw1, pw1s = _prepack_mxfp4_experts(w1q, w1s)
    pw2, pw2s = _prepack_mxfp4_experts(w2q, w2s)

    # Kernel
    out = ops.fused_experts_cpu(
        a.clone(),
        pw1,
        pw2,
        topk_weight,
        topk_ids,
        False,  # inplace
        ops.CPUQuantMethod.MXFP4,
        pw1s,  # w1_scale
        pw2s,  # w2_scale
        None,  # w1_zero
        None,  # w2_zero
        None,  # block_size
        w1_bias=w1_b,
        w2_bias=w2_b,
        alpha=alpha,
        limit=limit,
    )

    torch.testing.assert_close(ref_out.bfloat16(), out, atol=1e-2, rtol=1e-2)
```
**EN:** This pytest case verifies mxfp4 CPU fused MoE bias swiglu. It is parameterized across 3 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as M, N, K, E. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 mxfp4 CPU fused MoE bias swiglu 的行为。 它通过 3 组参数化输入覆盖多种场景；它会使用诸如 M、N、K、E 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

### Top-level block starting at line 499 (lines 499-500)
```python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v"]))
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `math`
- `sys`
- `pytest`
- `torch`
- `torch.nn.functional`
- `vllm.platforms -> current_platform`
- `vllm.utils.torch_utils -> set_random_seed`
- `vllm._custom_ops`
