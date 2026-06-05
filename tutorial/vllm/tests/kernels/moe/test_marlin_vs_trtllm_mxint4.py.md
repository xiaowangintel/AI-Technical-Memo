# test_marlin_vs_trtllm_mxint4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_marlin_vs_trtllm_mxint4.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_marlin_vs_trtllm_mxint4, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_marlin_vs_trtllm_mxint4 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-3)
```python
"""Test comparing Marlin INT4 MoE vs FlashInfer TRT-LLM MXINT4 MoE."""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 5-19)
```python
import pytest
import torch

from vllm.model_executor.layers.fused_moe.experts.marlin_moe import (
    fused_marlin_moe,
)
from vllm.model_executor.layers.fused_moe.router.grouped_topk_router import (
    grouped_topk,
)
from vllm.model_executor.layers.quantization.utils.flashinfer_mxint4_moe import (
    prepare_static_weights_for_trtllm_mxint4_moe,
)
from vllm.platforms import current_platform
from vllm.scalar_type import scalar_types
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; and vLLM components like vllm.model_executor.layers.fused_moe.experts.marlin_moe, vllm.model_executor.layers.fused_moe.router.grouped_topk_router, vllm.model_executor.layers.quantization.utils.flashinfer_mxint4_moe, vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；vLLM 内部组件，例如 vllm.model_executor.layers.fused_moe.experts.marlin_moe、vllm.model_executor.layers.fused_moe.router.grouped_topk_router、vllm.model_executor.layers.quantization.utils.flashinfer_mxint4_moe、vllm.platforms。

### Function `mxint4_quantize` (lines 22-45)
```python
def mxint4_quantize(
    x: torch.Tensor, sf_vec_size: int = 32
) -> tuple[torch.Tensor, torch.Tensor]:
    """Quantize BF16 tensor to MXINT4 with block scaling (group_size=sf_vec_size).

    Returns:
        - uint8 packed (2 INT4/byte): [..., k//2] - stores SIGNED INT4 [-8, 7]
        - scales in BF16: [..., k//sf_vec_size]
    """
    x_reshaped = x.reshape(-1, sf_vec_size)
    x_max = x_reshaped.max(dim=-1, keepdim=True)[0].to(torch.float32)
    x_min = x_reshaped.min(dim=-1, keepdim=True)[0].to(torch.float32)
    x_max = x_max * 8.0 / 7.0
    amax = torch.where(x_max > -x_min, x_max, -x_min)
    scales = amax / 8.0
    x_scaled = x_reshaped * scales.reciprocal()
    x_int8 = (
        x_scaled.round().clamp(-8, 7).to(torch.int8).reshape(-1, sf_vec_size // 2, 2)
    )
    x_int4 = (x_int8[..., 0] & 0x0F) | ((x_int8[..., 1] & 0x0F) << 4)
    return (
        x_int4.to(torch.uint8).reshape(*x.shape[:-1], x.shape[-1] // 2),
        scales.to(x.dtype).reshape(*x.shape[:-1], x.shape[-1] // sf_vec_size),
    )
```
**EN:** This helper function implements the shared logic for mxint4 quantize. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 mxint4 quantize 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `mxint4_quantize_moe_weights` (lines 48-70)
```python
def mxint4_quantize_moe_weights(
    weights_bf16: torch.Tensor, group_size: int = 32
) -> tuple[torch.Tensor, torch.Tensor]:
    """Quantize MoE weights [e, n, k] to MxInt4 format.

    Args:
        weights_bf16: BF16 weights of shape [num_experts, out_features, in_features]
        group_size: Quantization group size (default: 32)

    Returns:
        - weights_mxint4: Quantized weights [e, n, k//2] uint8
        - scales_mxint4: Quantization scales [e, n, k//group_size] bf16
    """
    e = weights_bf16.shape[0]
    weight_list = []
    scale_list = []

    for i in range(e):
        w_q, w_s = mxint4_quantize(weights_bf16[i], sf_vec_size=group_size)
        weight_list.append(w_q)
        scale_list.append(w_s)

    return torch.stack(weight_list), torch.stack(scale_list)
```
**EN:** This helper function implements the shared logic for mxint4 quantize MoE weights. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 mxint4 quantize MoE weights 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Constants and module state (lines 73-77)
```python
__all__ = [
    "mxint4_quantize",
    "mxint4_quantize_moe_weights",
    "marlin_quantize_moe_weights",
]
```
**EN:** This block prepares module-level state that later helpers and tests reuse.
**CN:** 该代码块准备了后续辅助函数和测试会复用的模块级状态。

### Function `marlin_quantize_moe_weights` (lines 80-114)
```python
def marlin_quantize_moe_weights(
    weights_bf16: torch.Tensor, group_size: int = 32
) -> tuple[torch.Tensor, torch.Tensor]:
    """Quantize MoE weights [e, n, k] to Marlin INT4 format.

    Args:
        weights_bf16: BF16 weights of shape [num_experts, out_features, in_features]
        group_size: Quantization group size (default: 32)

    Returns:
        - weights_marlin: Marlin quantized weights [e, k//8, n] int32
        - scales_marlin: Marlin quantization scales [e, k//group_size, n] bf16
    """
    from vllm.model_executor.layers.quantization.utils.marlin_utils_test import (
        marlin_quantize,
    )

    e, n, k = weights_bf16.shape
    weight_list = []
    scale_list = []

    for i in range(e):
        # Transpose for Marlin: [n, k] → [k, n]
        w_t = weights_bf16[i].T.contiguous()
        _, w_q, w_s, _, _, _ = marlin_quantize(
            w_t, scalar_types.uint4b8, group_size, act_order=False
        )
        weight_list.append(w_q)
        scale_list.append(w_s)

    # Stack to get [e, ...] shape
    weights_marlin = torch.stack(weight_list)  # [e, k // 8, n]
    scales_marlin = torch.stack(scale_list)  # [e, k // group_size, n]

    return weights_marlin, scales_marlin
```
**EN:** This helper function implements the shared logic for marlin quantize MoE weights. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 marlin quantize MoE weights 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Constants and module state (lines 117-119)
```python
TRTLLM_GEN_AVAILABLE = (
    current_platform.is_cuda() and current_platform.is_device_capability_family(100)
)
```
**EN:** This block centralizes shared constants and parameter grids, including TRTLLM_GEN_AVAILABLE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 TRTLLM_GEN_AVAILABLE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_marlin_vs_trtllm_mxint4_moe_kimik2` (lines 122-272)
```python
@pytest.mark.skipif(not TRTLLM_GEN_AVAILABLE, reason="Skip for non SM100")
@pytest.mark.parametrize("m", [1, 33])
@pytest.mark.parametrize("n", [7168])
@pytest.mark.parametrize("k", [512])
@pytest.mark.parametrize("e", [384])
@pytest.mark.parametrize("topk", [8])
@pytest.mark.parametrize("group_size", [32])
def test_marlin_vs_trtllm_mxint4_moe_kimik2(monkeypatch, m, n, k, e, topk, group_size):
    """Compare Marlin INT4 MoE vs FlashInfer TRT-LLM MXINT4 MoE.

    Uses mxint4_quantize() to generate common INT4 weights + BF16 scales,
    then runs both Marlin and TRT-LLM kernels and compares outputs.
    """
    pytest.importorskip("flashinfer")
    monkeypatch.setenv("VLLM_USE_FLASHINFER_MOE_INT4", "1")

    set_random_seed(0)

    dtype = torch.bfloat16

    # DeepSeekV3 routing config (from Kimi-K2-Thinking config.json)
    n_group = 1  # n_group from model config
    topk_group = 1  # topk_group from model config
    routed_scaling = 2.827  # routed_scaling_factor from model config

    # Input - realistic activation range for LLM (after LayerNorm: mean~0, std~1)
    a = torch.randn((m, k), device="cuda", dtype=dtype) * 0.5

    # Generate routing logits and bias (DeepSeekV3 expects float logits)
    # Realistic ranges: logits typically [-3, 3], bias [-2, 2]
    routing_logits = torch.randn((m, e), device="cuda", dtype=torch.float32) * 1.5
    routing_bias = torch.randn(e, device="cuda", dtype=torch.float32) * 0.8

    # 1. Generate BF16 weights (SHARED between both paths)
    # Realistic weight initialization: Xavier/Glorot uniform scaling
    # std = sqrt(2 / (fan_in + fan_out))
    std_w1 = (2.0 / (k + 2 * n)) ** 0.5
    std_w2 = (2.0 / (n + k)) ** 0.5
    w1_bf16 = torch.randn((e, 2 * n, k), device="cuda", dtype=dtype) * std_w1
    w2_bf16 = torch.randn((e, k, n), device="cuda", dtype=dtype) * std_w2
# ... excerpt ...
        quant_type_id=scalar_types.uint4b8.id,
        global_num_experts=e,
        expert_map=None,
        global_scale1=None,
        global_scale2=None,
        g_idx1=None,
        g_idx2=None,
        input_global_scale1=None,
        input_global_scale2=None,
        sort_indices1=None,
        sort_indices2=None,
        w1_zeros=None,
        w2_zeros=None,
        input_dtype=dtype,
        is_k_full=True,
    )

    # Sanity check: manually compute BF16 reference for comparison
    # Use same routing as Marlin path for consistency
    bf16_output = torch.zeros((m, k), device="cuda", dtype=dtype)
    for token_idx in range(m):
        for expert_rank in range(topk):
            expert_id = topk_ids[token_idx, expert_rank].item()
            weight = topk_weights[token_idx, expert_rank].item()
            # w1: [2*n, k] @ [k] -> [2*n]
            up_gate = a[token_idx] @ w1_bf16[expert_id].T  # [2*n]
            gate, up = up_gate.chunk(2, dim=0)
            intermediate = torch.nn.functional.silu(gate) * up  # [n]
            # w2: [k, n] @ [n] -> [k]
            expert_out = intermediate @ w2_bf16[expert_id].T  # [k]
            bf16_output[token_idx] += weight * expert_out
    # Compare against BF16 reference.
    torch.testing.assert_close(marlin_output, bf16_output, atol=0.3, rtol=1.0)
    torch.testing.assert_close(trtllm_output, bf16_output, atol=0.3, rtol=1.0)

    # Compare against each other for sanity.
    # Note: Different quantization schemes (UINT4b8 vs signed MXINT4) cause
    # some differences
    torch.testing.assert_close(marlin_output, trtllm_output, atol=0.3, rtol=6.0)
```
**EN:** This pytest case verifies marlin vs trtllm mxint4 MoE kimik2. It is parameterized across 6 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as monkeypatch, m, n, k. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 marlin vs trtllm mxint4 MoE kimik2 的行为。 它通过 6 组参数化输入覆盖多种场景；它会使用诸如 monkeypatch、m、n、k 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

### Function `test_flashinfer_trtllm_mxint4_moe_wrapper` (lines 275-365)
```python
@pytest.mark.skipif(not TRTLLM_GEN_AVAILABLE, reason="Skip for non SM100")
@pytest.mark.parametrize("m", [1, 33])
@pytest.mark.parametrize("n", [7168])
@pytest.mark.parametrize("k", [512])
@pytest.mark.parametrize("e", [384])
@pytest.mark.parametrize("topk", [8])
@torch.inference_mode()
def test_flashinfer_trtllm_mxint4_moe_wrapper(m, n, k, e, topk):
    """Test that the flashinfer_trtllm_mxint4_moe wrapper matches the raw
    trtllm_mxint4_block_scale_moe kernel call."""
    pytest.importorskip("flashinfer")
    from flashinfer import RoutingMethodType
    from flashinfer.fused_moe import trtllm_mxint4_block_scale_moe

    from vllm.model_executor.layers.quantization.utils.flashinfer_mxint4_moe import (
        flashinfer_trtllm_mxint4_moe,
    )

    set_random_seed(0)
    dtype = torch.bfloat16

    a = torch.randn((m, k), device="cuda", dtype=dtype) * 0.5
    router_logits = torch.randn((m, e), device="cuda", dtype=torch.float32) * 1.5
    routing_bias = torch.randn(e, device="cuda", dtype=torch.float32) * 0.8

    std_w1 = (2.0 / (k + 2 * n)) ** 0.5
    std_w2 = (2.0 / (n + k)) ** 0.5
    w1_bf16 = torch.randn((e, 2 * n, k), device="cuda", dtype=dtype) * std_w1
    w2_bf16 = torch.randn((e, k, n), device="cuda", dtype=dtype) * std_w2

    w1_int4, w1_scales = mxint4_quantize_moe_weights(w1_bf16)
    w2_int4, w2_scales = mxint4_quantize_moe_weights(w2_bf16)

    prepared = prepare_static_weights_for_trtllm_mxint4_moe(
        gemm1_weights=w1_int4,
        gemm1_scales=w1_scales,
        gemm2_weights=w2_int4,
        gemm2_scales=w2_scales,
    )

# ... excerpt ...
        num_experts=e,
        top_k=topk,
        n_group=1,
        topk_group=1,
        intermediate_size=n,
        local_expert_offset=0,
        local_num_experts=e,
        routed_scaling_factor=None,
        routing_method_type=RoutingMethodType.DeepSeekV3,
        enable_pdl=None,
        output=None,
        tune_max_num_tokens=8192,
    )
    if not isinstance(raw_out, torch.Tensor):
        raw_out = raw_out[0]
    raw_out = raw_out.to(dtype)

    # Wrapper call
    wrapper_out = flashinfer_trtllm_mxint4_moe(
        x=a,
        router_logits=router_logits,
        w13_weight_packed=prepared["gemm1_weights"],
        w13_weight_scale=prepared["gemm1_scales"],
        w2_weight_packed=prepared["gemm2_weights"],
        w2_weight_scale=prepared["gemm2_scales"],
        global_num_experts=e,
        top_k=topk,
        intermediate_size_per_partition=n,
        local_num_experts=e,
        ep_rank=0,
        num_expert_group=1,
        topk_group=1,
        e_score_correction_bias=routing_bias,
        routing_method_type=RoutingMethodType.DeepSeekV3,
    )

    assert wrapper_out.shape == (m, k)
    assert wrapper_out.dtype == dtype
    torch.testing.assert_close(wrapper_out, raw_out, atol=0.0, rtol=0.0)
```
**EN:** This pytest case verifies flashinfer trtllm mxint4 MoE wrapper. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, n, k, e. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 flashinfer trtllm mxint4 MoE wrapper 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 m、n、k、e 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm.model_executor.layers.fused_moe.experts.marlin_moe -> fused_marlin_moe`
- `vllm.model_executor.layers.fused_moe.router.grouped_topk_router -> grouped_topk`
- `vllm.model_executor.layers.quantization.utils.flashinfer_mxint4_moe -> prepare_static_weights_for_trtllm_mxint4_moe`
- `vllm.platforms -> current_platform`
- `vllm.scalar_type -> scalar_types`
- `vllm.utils.torch_utils -> set_random_seed`
