# test_ocp_mx_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_ocp_mx_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_ocp_mx_moe, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_ocp_mx_moe 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-13)
```python
import importlib.metadata
from dataclasses import dataclass
from importlib.util import find_spec

import pytest
import torch
from packaging import version

from vllm.platforms import current_platform
from vllm.utils.flashinfer import has_flashinfer
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as importlib.metadata, dataclasses, importlib.util, pytest; and vLLM components like vllm.platforms, vllm.utils.flashinfer.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 importlib.metadata、dataclasses、importlib.util、pytest；vLLM 内部组件，例如 vllm.platforms、vllm.utils.flashinfer。

### Constants and module state (lines 15-35)
```python
QUARK_MXFP4_AVAILABLE = find_spec("quark") is not None and version.parse(
    importlib.metadata.version("amd-quark")
) >= version.parse("0.8.99")

TRTLLM_GEN_MXFP4_AVAILABLE = (
    current_platform.is_cuda() and current_platform.is_device_capability_family(100)
)

TRTLLM_GEN_MXFP8_AVAILABLE = TRTLLM_GEN_MXFP4_AVAILABLE

HOPPER_MXFP4_BF16_AVAILABLE = (
    current_platform.is_cuda()
    and current_platform.is_device_capability(90)
    and has_flashinfer()
)

# ROCm platform and dependencies
ROCM_AVAILABLE = current_platform.is_rocm()
ROCM_TRITON_KERNELS_AVAILABLE = False
ROCM_AITER_AVAILABLE = False
ROCM_GFX950 = False
```
**EN:** This block centralizes shared constants and parameter grids, including QUARK_MXFP4_AVAILABLE, TRTLLM_GEN_MXFP4_AVAILABLE, TRTLLM_GEN_MXFP8_AVAILABLE, HOPPER_MXFP4_BF16_AVAILABLE, ROCM_AVAILABLE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 QUARK_MXFP4_AVAILABLE、TRTLLM_GEN_MXFP4_AVAILABLE、TRTLLM_GEN_MXFP8_AVAILABLE、HOPPER_MXFP4_BF16_AVAILABLE、ROCM_AVAILABLE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Top-level block starting at line 37 (lines 37-48)
```python
if ROCM_AVAILABLE:
    from vllm._aiter_ops import rocm_aiter_ops
    from vllm.platforms.rocm import on_gfx950
    from vllm.utils.import_utils import has_triton_kernels

    ROCM_TRITON_KERNELS_AVAILABLE = has_triton_kernels()
    ROCM_GFX950 = on_gfx950()
    ROCM_AITER_AVAILABLE = rocm_aiter_ops.is_enabled()

    if ROCM_AITER_AVAILABLE:
        from aiter.ops.triton.moe.quant_moe import upcast_from_mxfp
        from aiter.ops.triton.quant import dynamic_mxfp4_quant
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 50 (lines 50-60)
```python
if TRTLLM_GEN_MXFP4_AVAILABLE:
    from flashinfer import (
        fp4_quantize,
        mxfp8_quantize,
        reorder_rows_for_gated_act_gemm,
        shuffle_matrix_a,
        shuffle_matrix_sf_a,
        trtllm_fp4_block_scale_moe,
        trtllm_fp8_block_scale_moe,
    )
    from flashinfer.fp4_quantization import nvfp4_block_scale_interleave
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 62 (lines 62-66)
```python
if TRTLLM_GEN_MXFP8_AVAILABLE:
    from flashinfer.fused_moe.core import (
        Fp8QuantizationType,
        get_w2_permute_indices_with_cache,
    )
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Class `ModelCase` (lines 69-72)
```python
@dataclass
class ModelCase:
    model_id: str
    tp: int
```
**EN:** This dataclass packages the fields needed to describe ModelCase. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 ModelCase 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Function `enable_pickle` (lines 75-78)
```python
@pytest.fixture(scope="function", autouse=True)
def enable_pickle(monkeypatch):
    """`LLM.apply_model` requires pickling a function."""
    monkeypatch.setenv("VLLM_ALLOW_INSECURE_SERIALIZATION", "1")
```
**EN:** This fixture prepares reusable state for enable pickle. it consumes fixtures or inputs such as monkeypatch.
**CN:** 该 fixture 为 enable pickle 准备可复用的测试状态。 它会使用诸如 monkeypatch 等 fixture 或输入。

### Function `test_mxfp4_loading_and_execution_moe` (lines 81-128)
```python
@pytest.mark.parametrize(
    "model_case",
    [
        ModelCase("fxmarty/qwen_1.5-moe-a2.7b-mxfp4", tp=2),
        ModelCase("fxmarty/deepseek_r1_3_layers_mxfp4", tp=8),
        ModelCase("fxmarty/Llama-4-Scout-17B-16E-Instruct-2-layers-mxfp4", tp=1),
        ModelCase("fxmarty/Llama-3.1-70B-Instruct-2-layers-mxfp6", tp=1),
        ModelCase("fxmarty/Llama-3.1-70B-Instruct-2-layers-mxfp6", tp=4),
    ],
)
@pytest.mark.skipif(not QUARK_MXFP4_AVAILABLE, reason="amd-quark>=0.9 is not available")
def test_mxfp4_loading_and_execution_moe(vllm_runner, model_case: ModelCase):
    if torch.accelerator.device_count() < model_case.tp:
        pytest.skip(
            f"This test requires >={model_case.tp} gpus, got only "
            f"{torch.accelerator.device_count()}"
        )

    # `cudagraph_capture_sizes=[16]` to reduce load time.
    with vllm_runner(
        model_case.model_id,
        tensor_parallel_size=model_case.tp,
        load_format="dummy",
        compilation_config={"cudagraph_capture_sizes": [16]},
    ) as llm:
        # Disabled as check_model is broken: https://github.com/vllm-project/vllm/pull/18465#issuecomment-3329880562
        # def check_model(model):
        #     from vllm.model_executor.layers.quantization.quark.quark import (  # noqa: E501
        #         QuarkLinearMethod)
        #     from vllm.model_executor.layers.quantization.quark.schemes.quark_ocp_mx import QuarkOCP_MX  # noqa: E501
        #     from vllm.model_executor.layers.quantization.quark.quark_moe import (  # noqa: E501
        #         QuarkOCP_MX_MoEMethod)

        #     layer = model.model.layers[0]

        #     qkv_proj = layer.self_attn.qkv_proj

        #     assert isinstance(qkv_proj.quant_method, QuarkLinearMethod)
        #     assert isinstance(qkv_proj.scheme, QuarkOCP_MX)

        #     assert isinstance(layer.mlp.experts.quant_method,
        #                       QuarkOCP_MX_MoEMethod)

        # if model_case.model_id == "fxmarty/qwen_1.5-moe-a2.7b-mxfp4":
        #     llm.apply_model(check_model)

        output = llm.generate_greedy("Today I am in the French Alps and", max_tokens=20)
        assert output
```
**EN:** This pytest case verifies mxfp4 loading and execution MoE. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as vllm_runner, model_case. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 mxfp4 loading and execution MoE 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 vllm_runner、model_case 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Function `swiglu` (lines 131-139)
```python
def swiglu(x, alpha: float = 1.702, beta: float = 1.0, limit: float | None = None):
    # Note we add an extra bias of 1 to the linear layer
    # Uses chunked layout: first half is gate, second half is up
    x_glu, x_linear = torch.chunk(x, 2, dim=-1)
    if limit is not None:
        x_glu = x_glu.clamp(max=limit)
        x_linear = x_linear.clamp(min=-limit, max=limit)
    out_glu = x_glu * torch.sigmoid(alpha * x_glu)
    return out_glu * (x_linear + beta)
```
**EN:** This helper function implements the shared logic for swiglu. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 swiglu 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `swigluoai` (lines 142-149)
```python
def swigluoai(x, alpha: float = 1.702, limit: float = 7.0):
    # OAI swiglu uses interleaved layout: gate/up alternating
    # See SwigluOAIAndMul in vllm/model_executor/layers/activation.py
    gate, up = x[..., ::2], x[..., 1::2]
    gate = gate.clamp(max=limit)
    up = up.clamp(min=-limit, max=limit)
    glu = gate * torch.sigmoid(gate * alpha)
    return (up + 1) * glu
```
**EN:** This helper function implements the shared logic for swigluoai. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 swigluoai 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Constants and module state (lines 152-152)
```python
fp4_lookup_table = [0, 0.5, 1, 1.5, 2, 3, 4, 6, -0, -0.5, -1, -1.5, -2, -3, -4, -6]
```
**EN:** This block prepares module-level state that later helpers and tests reuse.
**CN:** 该代码块准备了后续辅助函数和测试会复用的模块级状态。

### Function `mxfp4_dequantize` (lines 155-173)
```python
def mxfp4_dequantize(x, scale):
    assert x.dtype == torch.uint8
    x = x.view(torch.uint8).to(torch.int32)
    x_unpacked = torch.zeros(
        *x.shape[:-1], x.shape[-1] * 2, dtype=torch.int32, device=x.device
    )
    x_unpacked[..., 0::2].copy_(x & 0xF)
    x_unpacked[..., 1::2].copy_((x >> 4) & 0xF)

    x_float = torch.zeros(x_unpacked.shape, dtype=torch.float32, device=x.device)
    for i, val in enumerate(fp4_lookup_table):
        x_float[x_unpacked == i] = val

    scale = scale.view(torch.uint8).to(torch.int32)
    scale = (scale << 23).view(torch.float32)
    scale = scale.reshape(*x.shape[:-1], -1)
    scale = torch.stack([scale] * 32, dim=-1).reshape(*x_float.shape)

    return x_float * scale
```
**EN:** This helper function implements the shared logic for mxfp4 dequantize. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 mxfp4 dequantize 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `mxfp8_dequantize` (lines 176-185)
```python
def mxfp8_dequantize(x, scale):
    assert x.dtype == torch.float8_e4m3fn
    x_float = x.to(torch.float32)

    scale = scale.view(torch.uint8).to(torch.int32)
    scale = (scale << 23).view(torch.float32)
    scale = scale.reshape(*x.shape[:-1], -1)
    scale = torch.stack([scale] * 32, dim=-1).reshape(*x_float.shape)

    return x_float * scale
```
**EN:** This helper function implements the shared logic for mxfp8 dequantize. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 mxfp8 dequantize 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `reference_moe` (lines 188-252)
```python
def reference_moe(
    roouting_logits,
    topk,
    num_experts,
    hidden_states,
    w13,
    bias13,
    w2,
    bias2,
    alpha,
    beta,
    limit,
    act_type,
    activation: str = "swiglu",
    use_interleaved_layout: bool = False,
):
    """
    Reference MoE implementation for accuracy testing.

    Args:
        activation: One of "swiglu", "silu", "relu2". Controls the activation
            function used after the first MLP.
        use_interleaved_layout: If True, uses interleaved gate/up layout
            (gate=x[..., ::2], up=x[..., 1::2]) as used by SWIGLUOAI.
            If False, uses chunked layout (gate, up = chunk(x, 2)) as used
            by standard swiglu/silu.
    """
    # renormalize routing
    experts = torch.topk(roouting_logits, k=topk, dim=-1, sorted=True)
    expert_weights = torch.nn.functional.softmax(experts.values, dim=1)
    expert_indices = experts.indices
    t = hidden_states.clone()
    # MLP #1
    mlp1_weight = w13[expert_indices, ...]
    mlp1_bias = bias13[expert_indices, ...]
    t = torch.einsum("beck,bk->bec", mlp1_weight, t) + mlp1_bias

    # Apply activation
    if activation in ("swiglu", "silu"):
        if use_interleaved_layout:
            # SWIGLUOAI: interleaved gate/up layout
            t = swigluoai(t, alpha=alpha, limit=limit)
        else:
            # Standard swiglu/silu: chunked layout
            t = swiglu(t, alpha=alpha, beta=beta, limit=limit)
    elif activation == "relu2":
        # RELU2_NO_MUL: relu(x)^2
        t = torch.relu(t)
        t = t * t
    else:
        raise ValueError(f"Unknown activation: {activation}")

    if act_type == "mxfp8":
        t_quantized, t_scale = mxfp8_quantize(
            t.to(torch.bfloat16), is_sf_swizzled_layout=False
        )
        t = mxfp8_dequantize(t_quantized, t_scale)
    # MLP #2
    mlp2_weight = w2[expert_indices, ...]
    mlp2_bias = bias2[expert_indices, ...]
    t = torch.einsum("beck,bek->bec", mlp2_weight, t) + mlp2_bias
    # Weighted sum of experts
    t = torch.einsum("bec,be->bc", t, expert_weights)
    assert t.shape == hidden_states.shape
    return t.to(torch.bfloat16)
```
**EN:** This helper function implements the shared logic for reference MoE. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 reference MoE 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `tg_mxfp4_moe` (lines 255-506)
```python
def tg_mxfp4_moe(
    router_logits,
    topk,
    num_experts,
    intermediate_size,
    hidden_size,
    hidden_states,
    hidden_states_scale,
    w13_weight,
    w13_weight_scale,
    w13_bias,
    w2_weight,
    w2_weight_scale,
    w2_bias,
    act_type,
    alpha,
    beta,
    limit,
    transpose_optimized: bool = False,
) -> torch.Tensor:
    sf_block_size = 32
    assert (
        w13_weight.dim() == 3
        and w13_weight.shape[0] == num_experts
        and w13_weight.shape[1] == intermediate_size * 2
        and w13_weight.shape[2] == hidden_size // 2
    )
    assert (
        w13_weight_scale.dim() == 3
        and w13_weight_scale.shape[0] == num_experts
        and w13_weight_scale.shape[1] == intermediate_size * 2
        and w13_weight_scale.shape[2] == hidden_size // sf_block_size
    )
    assert (
        w2_weight.dim() == 3
        and w2_weight.shape[0] == num_experts
        and w2_weight.shape[1] == hidden_size
        and w2_weight.shape[2] == intermediate_size // 2
    )
    assert (
# ... excerpt ...
    w13_bias = torch.stack(gemm1_bias_shuffled).reshape(num_experts, -1)

    w2_weight = torch.stack(gemm2_weights_shuffled)
    w2_weight_scale = (
        torch.stack(gemm2_scales_shuffled)
        .reshape(num_experts, hidden_size, intermediate_size // sf_block_size)
        .view(torch.float8_e4m3fn)
    )
    w2_bias = torch.stack(gemm2_bias_shuffled).reshape(num_experts, -1)

    tg_result = trtllm_fp4_block_scale_moe(
        routing_logits=router_logits.to(torch.bfloat16),
        routing_bias=None,
        hidden_states=hidden_states,
        hidden_states_scale=hidden_states_scale,
        gemm1_weights=w13_weight,
        gemm1_weights_scale=w13_weight_scale,
        gemm1_bias=w13_bias,
        gemm1_alpha=alpha,
        gemm1_beta=beta,
        gemm1_clamp_limit=limit,
        gemm2_weights=w2_weight,
        gemm2_weights_scale=w2_weight_scale,
        gemm2_bias=w2_bias,
        output1_scale_scalar=None,
        output1_scale_gate_scalar=None,
        output2_scale_scalar=None,
        num_experts=num_experts,
        top_k=topk,
        n_group=None,
        topk_group=None,
        intermediate_size=intermediate_size,
        local_expert_offset=0,
        local_num_experts=num_experts,
        routed_scaling_factor=None,
        routing_method_type=1,  # renormalize
        do_finalize=True,
    )[0]
    return tg_result
```
**EN:** This helper function implements the shared logic for tg mxfp4 MoE. assertions at the end lock in the intended behavior or graph shape. Only the key portions are shown here because the block is large.
**CN:** 该辅助函数实现了 tg mxfp4 MoE 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。 由于该代码块较大，这里只展示关键片段。

### Function `check_accuracy` (lines 509-529)
```python
def check_accuracy(a, b, atol, rtol, percent):
    """Allow a mismatch percentage of 1 - percent."""
    if torch.any(torch.isnan(a)):
        raise Exception("NaN in reference output")
    if torch.any(torch.isnan(b)):
        raise Exception("NaN in actual output")
    if torch.any(torch.isinf(a)):
        raise Exception("Inf in reference output")
    if torch.any(torch.isinf(b)):
        raise Exception("Inf in actual output")
    assert a.shape == b.shape, f"Shape mismatch: {a.shape} vs {b.shape}"

    left = torch.abs(a - b)
    right = atol + rtol * torch.abs(b)
    count = torch.sum(left > right)
    mismatch_percent = count / a.numel()
    if mismatch_percent > 1 - percent:
        raise Exception(
            f"Mismatch percentage is {mismatch_percent:.4f} for rtol {rtol} "
            f"(threshold: {1 - percent:.4f})"
        )
```
**EN:** This helper function implements the shared logic for check accuracy. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 check accuracy 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `test_trtllm_gen_mxfp4_fused_moe` (lines 532-672)
```python
@pytest.mark.parametrize("topk", [1, 4])
@pytest.mark.parametrize("num_experts", [32, 128])
@pytest.mark.parametrize("num_tokens", [1, 128, 1024])
@pytest.mark.parametrize("intermediate_size,hidden_size", [(3072, 3072)])
@pytest.mark.parametrize("alpha,beta,limit", [(1.0, 1.0, None), (1.702, 1.0, 7.0)])
@pytest.mark.parametrize("act_type", ["mxfp8", "bf16"])
@pytest.mark.parametrize("transpose_optimized", [False, True])
@pytest.mark.skipif(
    not TRTLLM_GEN_MXFP4_AVAILABLE,
    reason="nvidia gpu and compute capability sm100 is required for this test",
)
def test_trtllm_gen_mxfp4_fused_moe(
    topk: int,
    num_experts: int,
    num_tokens: int,
    intermediate_size: int,
    hidden_size: int,
    alpha: float,
    beta: float,
    limit: float | None,
    act_type: str,
    transpose_optimized: bool,
):
    seed = 42
    torch.manual_seed(seed)
    hidden_states = torch.randn(
        num_tokens, hidden_size, device="cuda:0", dtype=torch.bfloat16
    )
    w13 = torch.randn(
        num_experts,
        intermediate_size * 2,
        hidden_size,
        device="cuda:0",
        dtype=torch.bfloat16,
    )
    w2 = torch.randn(
        num_experts,
        hidden_size,
        intermediate_size,
        device="cuda:0",
# ... excerpt ...
            bias2_ref,
            alpha,
            beta,
            limit,
            act_type,
            activation="swiglu",
            use_interleaved_layout=False,
        )
        ref_result[start_idx:end_idx].copy_(chunk_result)

    # trtllm-gen result
    if alpha is not None:
        alpha = torch.full((num_experts,), alpha, device=hidden_states.device)
    if limit is not None:
        limit = torch.full((num_experts,), limit, device=hidden_states.device)
    if beta is not None:
        beta = torch.full((num_experts,), beta, device=hidden_states.device)
    tg_result = tg_mxfp4_moe(
        router_logits=router_logits,
        topk=topk,
        num_experts=num_experts,
        intermediate_size=intermediate_size,
        hidden_size=hidden_size,
        hidden_states=hidden_states,
        hidden_states_scale=hidden_states_scale,
        w13_weight=w13,
        w13_weight_scale=w13_scale,
        w13_bias=bias13,
        w2_weight=w2,
        w2_weight_scale=w2_scale,
        w2_bias=bias2,
        act_type=act_type,
        alpha=alpha,
        beta=beta,
        limit=limit,
        transpose_optimized=transpose_optimized,
    )
    # relatively loose check since the mxfp4 quantization is less accurate
    check_accuracy(ref_result, tg_result, atol=0, rtol=0.3, percent=0.8)
```
**EN:** This pytest case verifies trtllm gen mxfp4 fused MoE. It is parameterized across 7 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as topk, num_experts, num_tokens, intermediate_size. unsupported hardware, backend, or configuration combinations are skipped early. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 trtllm gen mxfp4 fused MoE 的行为。 它通过 7 组参数化输入覆盖多种场景；它会使用诸如 topk、num_experts、num_tokens、intermediate_size 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

### Function `_interleave_scales_lastdim_by4` (lines 675-685)
```python
def _interleave_scales_lastdim_by4(scales: torch.Tensor) -> torch.Tensor:
    """Interleave scales on the last dimension by groups of 4, matching
    the transformation in mxfp4.py's BF16 (Hopper) path."""
    s = scales.to(torch.uint8)
    s_shape = s.shape
    assert s_shape[-1] % 4 == 0
    s = s.reshape(*s_shape[:-1], s_shape[-1] // 4, 4)
    # Move the 4-group dimension before the row dimension
    permuted = s.permute(0, 2, 1, 3)
    # Merge the row dim with the 4-group dim
    return permuted.reshape(s_shape[0], s_shape[-1] // 4, s_shape[1] * 4)
```
**EN:** This helper function implements the shared logic for interleave scales lastdim by4. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 interleave scales lastdim by4 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `test_flashinfer_cutlass_mxfp4_fused_moe` (lines 688-836)
```python
@pytest.mark.parametrize("topk", [1, 4])
@pytest.mark.parametrize("num_experts", [32])
@pytest.mark.parametrize("num_tokens", [1, 128])
@pytest.mark.parametrize("intermediate_size,hidden_size", [(3072, 3072)])
@pytest.mark.parametrize("alpha,beta,limit", [(1.0, 1.0, None), (1.702, 1.0, 7.0)])
@pytest.mark.skipif(
    not HOPPER_MXFP4_BF16_AVAILABLE,
    reason="nvidia gpu sm90 and flashinfer are required for this test",
)
def test_flashinfer_cutlass_mxfp4_fused_moe(
    topk: int,
    num_experts: int,
    num_tokens: int,
    intermediate_size: int,
    hidden_size: int,
    alpha: float,
    beta: float,
    limit: float | None,
):
    torch.manual_seed(42)
    device = "cuda:0"

    # Inputs
    hidden_states = torch.randn(
        num_tokens, hidden_size, device=device, dtype=torch.bfloat16
    )
    # Random MXFP4 weights and scales (uint8), contiguous [w1; w3]
    w13_q = torch.randint(
        0,
        256,
        (num_experts, 2 * intermediate_size, hidden_size // 2),
        device=device,
        dtype=torch.uint8,
    )
    w13_scale = torch.randint(
        118,
        123,
        (num_experts, 2 * intermediate_size, hidden_size // 32),
        device=device,
        dtype=torch.uint8,
# ... excerpt ...
    token_final_scales, token_selected_experts = torch.topk(
        routing_weights, topk, dim=-1
    )
    token_final_scales = token_final_scales / token_final_scales.sum(
        dim=-1, keepdim=True
    )
    token_selected_experts = token_selected_experts.to(torch.int).contiguous()

    out = torch.empty_like(hidden_states, dtype=torch.bfloat16)
    if alpha is not None:
        alpha = torch.full((num_experts,), alpha, device=hidden_states.device)
    if beta is not None:
        beta = torch.full((num_experts,), beta, device=hidden_states.device)
    if limit is not None:
        limit = torch.full((num_experts,), limit, device=hidden_states.device)

    _ = flashinfer_cutlass_fused_moe(
        input=hidden_states,
        token_selected_experts=token_selected_experts,
        token_final_scales=token_final_scales,
        fc1_expert_weights=w13_q_swapped,
        fc2_expert_weights=w2_q,
        output_dtype=torch.bfloat16,
        output=out,
        quant_scales=[w13_s_inter.to(torch.uint8), w2_s_inter.to(torch.uint8)],
        fc1_expert_biases=w13_b,
        fc2_expert_biases=bias2.to(torch.bfloat16),
        swiglu_alpha=alpha,
        swiglu_beta=beta,
        swiglu_limit=limit,
        tp_size=1,
        tp_rank=0,
        ep_size=1,
        ep_rank=0,
        use_w4_group_scaling=True,
    )

    # Allow some mismatch due to MXFP4 quantization
    check_accuracy(ref, out, atol=0, rtol=0.3, percent=0.8)
```
**EN:** This pytest case verifies flashinfer cutlass mxfp4 fused MoE. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as topk, num_experts, num_tokens, intermediate_size. unsupported hardware, backend, or configuration combinations are skipped early. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 flashinfer cutlass mxfp4 fused MoE 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 topk、num_experts、num_tokens、intermediate_size 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

### Function `test_flashinfer_cutlass_mxfp4_mxfp8_fused_moe` (lines 839-1040)
```python
@pytest.mark.parametrize("topk", [1, 4])
@pytest.mark.parametrize("num_experts", [32])
@pytest.mark.parametrize("num_tokens", [1, 128])
@pytest.mark.parametrize("intermediate_size,hidden_size", [(3072, 3072)])
@pytest.mark.parametrize("alpha,beta,limit", [(1.0, 1.0, None), (1.702, 1.0, 7.0)])
@pytest.mark.skipif(
    not (
        current_platform.is_cuda()
        and current_platform.is_device_capability_family(100)
        and has_flashinfer()
    ),
    reason="NVIDIA GPU sm100 and flashinfer are required for this test",
)
def test_flashinfer_cutlass_mxfp4_mxfp8_fused_moe(
    topk: int,
    num_experts: int,
    num_tokens: int,
    intermediate_size: int,
    hidden_size: int,
    alpha: float | None,
    beta: float | None,
    limit: float | None,
):
    torch.manual_seed(42)
    device = "cuda:0"

    # Inputs
    hidden_states = torch.randn(
        num_tokens, hidden_size, device=device, dtype=torch.bfloat16
    )
    # Float weights in w13 format [w1; w3]
    w13 = (
        torch.randn(
            num_experts,
            2 * intermediate_size,
            hidden_size,
            device=device,
            dtype=torch.bfloat16,
        )
        / 10
# ... excerpt ...
        beta_t = None
    if limit is not None:
        limit_t = torch.full((num_experts,), limit, device=hidden_states.device)
    else:
        limit_t = None

    # Quant scales for SM100 MXFP8+MXFP4 path
    fake_input_scale = torch.ones(num_experts, device=device)
    quant_scales = [
        w13_scale_swapped.view(torch.int32),
        fake_input_scale,
        w2_scale.view(torch.int32),
        fake_input_scale,
    ]

    _ = flashinfer_cutlass_fused_moe(
        input=hidden_states_q,
        token_selected_experts=token_selected_experts,
        token_final_scales=token_final_scales,
        fc1_expert_weights=w13_q_swapped.contiguous().view(torch.long),
        fc2_expert_weights=w2_q.contiguous().view(torch.long),
        output_dtype=torch.bfloat16,
        output=out,
        quant_scales=quant_scales,
        fc1_expert_biases=w13_b,
        fc2_expert_biases=bias2.to(torch.bfloat16),
        swiglu_alpha=alpha_t,
        swiglu_beta=beta_t,
        swiglu_limit=limit_t,
        tp_size=1,
        tp_rank=0,
        ep_size=1,
        ep_rank=0,
        use_mxfp8_act_scaling=True,
        input_sf=hidden_states_sf,
    )

    # Allow some mismatch due to MXFP4 quantization
    check_accuracy(ref, out, atol=0, rtol=0.3, percent=0.8)
```
**EN:** This pytest case verifies flashinfer cutlass mxfp4 mxfp8 fused MoE. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as topk, num_experts, num_tokens, intermediate_size. unsupported hardware, backend, or configuration combinations are skipped early. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 flashinfer cutlass mxfp4 mxfp8 fused MoE 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 topk、num_experts、num_tokens、intermediate_size 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

### Function `test_trtllm_gen_mxfp8_block_scale_moe` (lines 1043-1207)
```python
@pytest.mark.parametrize("topk", [1, 4])
@pytest.mark.parametrize("num_experts", [32])
@pytest.mark.parametrize("num_tokens", [1, 128])
@pytest.mark.parametrize("intermediate_size,hidden_size", [(3072, 3072)])
@pytest.mark.parametrize("is_gated", [True], ids=["gated"])
@pytest.mark.skipif(
    not TRTLLM_GEN_MXFP8_AVAILABLE,
    reason="nvidia gpu and compute capability sm100 is required for this test",
)
def test_trtllm_gen_mxfp8_block_scale_moe(
    topk: int,
    num_experts: int,
    num_tokens: int,
    intermediate_size: int,
    hidden_size: int,
    is_gated: bool,
):
    torch.manual_seed(42)
    device = "cuda:0"

    inter_size = intermediate_size * (2 if is_gated else 1)

    hidden_states = (
        torch.randn(num_tokens, hidden_size, device=device, dtype=torch.bfloat16) / 20
    )
    w13 = (
        torch.randn(
            num_experts,
            inter_size,
            hidden_size,
            device=device,
            dtype=torch.bfloat16,
        )
        / 20
    )
    w2 = (
        torch.randn(
            num_experts,
            hidden_size,
            intermediate_size,
# ... excerpt ...
                w13_scale_interleaved.view(torch.uint8).reshape(w13_rows, -1),
                epilogue_tile_m,
            )
            .contiguous()
            .view(w13_scale.dtype)
        )
        gemm2_scales_shuffled.append(
            shuffle_matrix_sf_a(
                w2_scale[i].view(torch.uint8).reshape(hidden_size, -1), epilogue_tile_m
            )
            .contiguous()
            .view(w2_scale.dtype)
        )

    out = trtllm_fp8_block_scale_moe(
        routing_logits=router_logits_kernel,
        routing_bias=None,
        hidden_states=hidden_states_q,
        hidden_states_scale=hidden_states_scale,
        gemm1_weights=torch.stack(gemm1_weights_shuffled),
        gemm1_weights_scale=torch.stack(gemm1_scales_shuffled),
        gemm2_weights=torch.stack(gemm2_weights_shuffled),
        gemm2_weights_scale=torch.stack(gemm2_scales_shuffled),
        num_experts=num_experts,
        top_k=topk,
        n_group=None,
        topk_group=None,
        intermediate_size=intermediate_size,
        local_expert_offset=0,
        local_num_experts=num_experts,
        routed_scaling_factor=None,
        routing_method_type=1,  # renormalize routing
        use_shuffled_weight=True,
        weight_layout=0,  # MajorK
        fp8_quantization_type=Fp8QuantizationType.MxFp8,
    )

    # Block-scale MXFP8 kernels are approximate; require majority close.
    check_accuracy(ref, out, atol=0.1, rtol=0.85, percent=0.8)
```
**EN:** This pytest case verifies trtllm gen mxfp8 block scale MoE. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as topk, num_experts, num_tokens, intermediate_size. unsupported hardware, backend, or configuration combinations are skipped early. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 trtllm gen mxfp8 block scale MoE 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 topk、num_experts、num_tokens、intermediate_size 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

### Constants and module state (lines 1216-1245)
```python
ROCM_BACKEND_CONFIGS = {
    "TRITON": {
        "activation": "SWIGLUOAI",
        "rtol": 0.3,
        "percent": 0.95,
        "requires_aiter": False,
        "requires_gfx950": False,
    },
    "TRITON_UNFUSED": {
        "activation": "SWIGLUOAI",
        "rtol": 0.3,
        "percent": 0.95,
        "requires_aiter": False,
        "requires_gfx950": False,
    },
    "AITER_MXFP4_BF16": {
        "activation": "SILU",
        "rtol": 1.0,
        "percent": 0.7,
        "requires_aiter": True,
        "requires_gfx950": True,
    },
    "AITER_MXFP4_FP8": {
        "activation": "SWIGLUOAI",
        "rtol": 0.5,
        "percent": 0.9,
        "requires_aiter": True,
        "requires_gfx950": True,
    },
}
```
**EN:** This block centralizes shared constants and parameter grids, including ROCM_BACKEND_CONFIGS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 ROCM_BACKEND_CONFIGS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_rocm_mxfp4_moe_oracle` (lines 1248-1532)
```python
@pytest.mark.parametrize("backend_name", list(ROCM_BACKEND_CONFIGS.keys()))
@pytest.mark.parametrize("topk", [4])
@pytest.mark.parametrize("num_experts", [8])
@pytest.mark.parametrize("num_tokens,hidden_size,intermediate_size", [(16, 256, 256)])
@pytest.mark.skipif(
    not ROCM_AVAILABLE,
    reason="ROCm is required for this test",
)
@torch.inference_mode()
def test_rocm_mxfp4_moe_oracle(
    backend_name: str,
    topk: int,
    num_experts: int,
    num_tokens: int,
    hidden_size: int,
    intermediate_size: int,
):
    """
    Test ROCm MXFP4 MoE using oracle functions.

    This test validates that the oracle functions work end-to-end:
    - select_mxfp4_moe_backend() selects a valid backend
    - convert_to_mxfp4_moe_kernel_format() converts weights without error
    - make_mxfp4_moe_quant_config() builds a valid quant config
    - make_mxfp4_moe_kernel() creates a kernel that runs without error
    - The kernel output is within accuracy tolerance of reference
    """
    config = ROCM_BACKEND_CONFIGS[backend_name]

    # Check platform requirements
    if not ROCM_TRITON_KERNELS_AVAILABLE:
        pytest.skip("triton_kernels required for quantization")
    if config["requires_aiter"] and not ROCM_AITER_AVAILABLE:
        pytest.skip(f"Backend {backend_name} requires AITER")
    if config["requires_gfx950"] and not ROCM_GFX950:
        pytest.skip(f"Backend {backend_name} requires GFX950")

    from vllm.config import VllmConfig, set_current_vllm_config
    from vllm.model_executor.layers.fused_moe.activation import MoEActivation
    from vllm.model_executor.layers.fused_moe.oracle.mxfp4 import (
# ... excerpt ...
        w13_dq.to(torch.float32),
        w13_bias.to(torch.float32),
        w2_dq.to(torch.float32),
        w2_bias.to(torch.float32),
        alpha=1.702 if activation == MoEActivation.SWIGLUOAI else 1.0,
        beta=1.0 if activation == MoEActivation.SWIGLUOAI else 0.0,
        limit=7.0 if activation == MoEActivation.SWIGLUOAI else None,
        act_type="bf16",
        activation=act_name,
        use_interleaved_layout=use_interleaved,
    )

    # Compute and print accuracy statistics
    diff = (ref.float() - out.float()).abs()
    rel_diff = diff / (ref.float().abs() + 1e-6)

    print(f"\n[{backend_name}] Accuracy statistics:")
    print(
        f"  Reference: min={ref.min():.4f}, max={ref.max():.4f}, mean={ref.mean():.4f}"
    )
    print(
        f"  Output:    min={out.min():.4f}, max={out.max():.4f}, mean={out.mean():.4f}"
    )
    print(
        f"  Abs diff:  min={diff.min():.4f}, max={diff.max():.4f}, "
        f"mean={diff.mean():.4f}"
    )
    print(
        f"  Rel diff:  min={rel_diff.min():.4f}, max={rel_diff.max():.4f}, "
        f"mean={rel_diff.mean():.4f}"
    )

    # Check what percentage of values are within various tolerances
    for rtol in [0.1, 0.5, 1.0, 2.0]:
        within_tol = (diff <= rtol * out.float().abs()).float().mean()
        print(f"  Within rtol={rtol}: {within_tol * 100:.1f}%")

    # Check accuracy using per-backend thresholds
    check_accuracy(ref, out, atol=0.1, rtol=config["rtol"], percent=config["percent"])
```
**EN:** This pytest case verifies ROCm mxfp4 MoE oracle. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as backend_name, topk, num_experts, num_tokens. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 ROCm mxfp4 MoE oracle 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 backend_name、topk、num_experts、num_tokens 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。
- **Reusable fixtures / 可复用 fixture:** Fixtures package common setup so each test can focus on the scenario under study. / fixture 封装公共准备逻辑，让每个测试更专注于待验证场景。

## Dependencies / 依赖关系
- `importlib.metadata`
- `dataclasses -> dataclass`
- `importlib.util -> find_spec`
- `pytest`
- `torch`
- `packaging -> version`
- `vllm.platforms -> current_platform`
- `vllm.utils.flashinfer -> has_flashinfer`
