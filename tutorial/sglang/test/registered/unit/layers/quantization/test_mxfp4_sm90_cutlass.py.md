# test_mxfp4_sm90_cutlass.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/layers/quantization/test_mxfp4_sm90_cutlass.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates mxfp4 sm90 cutlass behavior in SGLang's unit / layers / quantization area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 层 / quantization 领域中与 mxfp4 sm90 cutlass 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: supporting statements / 辅助语句
```python
"""Unit test for the SM90 cutlass MXFP4 path in :class:`Mxfp4MoEMethod`.

Builds a single-layer GPT-OSS-style MoE with random MXFP4 weights, drives the
SGLang plumbing (``_process_weights_for_sm90_cutlass`` + ``_apply_sm90_cutlass``)
and compares against a direct FlashInfer ``cutlass_fused_moe`` call with the
same inputs. Both paths invoke the same SM90 kernel from FlashInfer PR #3084,
so outputs must be bit-exact.

Run on H100/H200:

    python -m pytest test/registered/unit/layers/quantization/test_mxfp4_sm90_cutlass.py -v
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 14-21: module imports and dependencies / 模块导入与依赖
```python
from __future__ import annotations

from contextlib import nullcontext

import pytest
import torch

from sglang.test.ci.ci_register import register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `__future__`, `contextlib`, `pytest`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `__future__`, `contextlib`, `pytest`, `torch`。

### Lines 23-34: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=120, stage="base-b", runner_config="1-gpu-large")

flashinfer_fused_moe = pytest.importorskip("flashinfer.fused_moe")

if not hasattr(flashinfer_fused_moe, "interleave_moe_weights_for_sm90_mixed_gemm"):
    pytest.skip(
        "FlashInfer build does not include PR #3084 SM90 mixed-input helpers",
        allow_module_level=True,
    )

if not torch.cuda.is_available():
    pytest.skip("CUDA required", allow_module_level=True)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, importorskip, hasattr.
**CN:** 该代码块通过 register_cuda_ci, importorskip, hasattr 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 36-36: module imports and dependencies / 模块导入与依赖
```python
from sglang.srt.utils import is_sm90_supported, is_sm100_supported
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.srt.utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.srt.utils`。

### Lines 38-42: supporting statements / 辅助语句
```python
if not is_sm90_supported() or is_sm100_supported():
    pytest.skip(
        "SM90-only path; require Hopper without SM100 promotion",
        allow_module_level=True,
    )
```
**EN:** This block performs supporting work through calls such as `is_sm100_supported`, `skip`, `is_sm90_supported`, preparing state for nearby definitions.
**CN:** 该代码块通过 `is_sm100_supported`, `skip`, `is_sm90_supported` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 44-49: module imports and dependencies / 模块导入与依赖
```python
from flashinfer.fused_moe import (
    cutlass_fused_moe,
    interleave_moe_scales_for_sm90_mixed_gemm,
    interleave_moe_weights_for_sm90_mixed_gemm,
)
from flashinfer.fused_moe.core import ActivationType
```
**EN:** This block imports the modules needed by the rest of the file, including `flashinfer.fused_moe`, `flashinfer.fused_moe.core`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `flashinfer.fused_moe`, `flashinfer.fused_moe.core`。

### Lines 51-51: module-level constants and configuration / 模块级常量与配置
```python
GROUP_SIZE = 32  # MXFP4 block size
```
**EN:** This block defines shared names such as `GROUP_SIZE`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `GROUP_SIZE` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 54-54: class _MockLayer declaration / 类 _MockLayer 声明
```python
class _MockLayer:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 55-59: supporting statements / 辅助语句
```python
    """Stand-in for ``FusedMoE`` carrying the attributes the SM90 helpers read.

    We construct one by hand so the test stays out of SGLang's distributed init
    path (``get_tp_group`` etc.).
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 62-62: class _MockTopKOutput declaration / 类 _MockTopKOutput 声明
```python
class _MockTopKOutput:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 63-65: method init / 方法 init
```python
    def __init__(self, weights, ids):
        self.topk_weights = weights
        self.topk_ids = ids
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 68-116: function make random mxfp4 / 函数 make random mxfp4
```python
def _make_random_mxfp4(num_experts, hidden, inter, seed=0):
    g = torch.Generator(device="cuda").manual_seed(seed)
    w13 = torch.randint(
        0,
        256,
        (num_experts, 2 * inter, hidden // 2),
        dtype=torch.uint8,
        device="cuda",
        generator=g,
    )
    w2 = torch.randint(
        0,
        256,
        (num_experts, hidden, inter // 2),
        dtype=torch.uint8,
        device="cuda",
        generator=g,
    )
    # E8M0 scales centered around 127 (= 2^0); narrow band keeps dequant values
    # in a sane range so SwiGLU clamp doesn't dominate.
    w13_s = torch.randint(
        125,
        130,
        (num_experts, 2 * inter, hidden // GROUP_SIZE),
        dtype=torch.uint8,
        device="cuda",
        generator=g,
    )
    w2_s = torch.randint(
        125,
        130,
        (num_experts, hidden, inter // GROUP_SIZE),
        dtype=torch.uint8,
        device="cuda",
        generator=g,
    )
    w13_b = (
        torch.randn(
            num_experts, 2 * inter, dtype=torch.float32, device="cuda", generator=g
        ).to(torch.bfloat16)
        * 0.01
    )
    w2_b = (
        torch.randn(
            num_experts, hidden, dtype=torch.float32, device="cuda", generator=g
        ).to(torch.bfloat16)
        * 0.01
    )
    return w13, w2, w13_s, w2_s, w13_b, w2_b
```
**EN:** This block implements `_make_random_mxfp4` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_random_mxfp4`，承担模块行为中的一个聚焦逻辑片段。

### Lines 119-126: function make topk / 函数 make topk
```python
def _make_topk(tokens, num_experts, top_k, seed=1):
    g = torch.Generator(device="cuda").manual_seed(seed)
    logits = torch.randn(
        tokens, num_experts, dtype=torch.float32, device="cuda", generator=g
    )
    weights, ids = torch.topk(torch.softmax(logits, dim=-1), top_k, dim=-1)
    weights = weights / weights.sum(dim=-1, keepdim=True)
    return weights.to(torch.float32), ids.to(torch.int32)
```
**EN:** This block implements `_make_topk` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_topk`，承担模块行为中的一个聚焦逻辑片段。

### Lines 129-142: function build mock layer / 函数 build mock layer
```python
def _build_mock_layer(num_experts, hidden, inter, w13, w2, w13_s, w2_s, w13_b, w2_b):
    layer = _MockLayer()
    layer.w13_weight = torch.nn.Parameter(w13.clone(), requires_grad=False)
    layer.w2_weight = torch.nn.Parameter(w2.clone(), requires_grad=False)
    layer.w13_weight_scale = torch.nn.Parameter(w13_s.clone(), requires_grad=False)
    layer.w2_weight_scale = torch.nn.Parameter(w2_s.clone(), requires_grad=False)
    layer.w13_weight_bias = torch.nn.Parameter(w13_b.clone(), requires_grad=False)
    layer.w2_weight_bias = torch.nn.Parameter(w2_b.clone(), requires_grad=False)
    layer.num_local_experts = num_experts  # tests run with EP size = 1
    layer.moe_tp_size = 1
    layer.moe_tp_rank = 0
    layer.moe_ep_size = 1
    layer.moe_ep_rank = 0
    return layer
```
**EN:** This block implements `_build_mock_layer` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_build_mock_layer`，承担模块行为中的一个聚焦逻辑片段。

### Lines 145-146: function round up / 函数 round up
```python
def _round_up(x, base):
    return ((x + base - 1) // base) * base
```
**EN:** This block implements `_round_up` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_round_up`，承担模块行为中的一个聚焦逻辑片段。

### Lines 149-163: function build method / 函数 build method
```python
def _build_method(num_experts, hidden, inter):
    from sglang.srt.layers.quantization.mxfp4 import Mxfp4MoEMethod

    method = Mxfp4MoEMethod.__new__(Mxfp4MoEMethod)
    method._fi_kernel = "cutlass_sm90"
    method.num_experts = num_experts
    # The new SM90 cutlass path tracks padded sizes in dedicated attrs;
    # ``hidden_size`` / ``intermediate_size_per_partition`` keep the unpadded
    # values to mirror what ``create_weights`` records.
    method.hidden_size = hidden
    method.intermediate_size_per_partition = inter
    method._padded_hidden = _round_up(hidden, 128)
    method._padded_intermediate = _round_up(inter, 128)
    method.use_flashinfer = True
    return method
```
**EN:** This block implements `_build_method` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_build_method`，承担模块行为中的一个聚焦逻辑片段。

### Lines 166-201: function expected w13 processed / 函数 expected w13 processed
```python
def _expected_w13_processed(w13_un, w13_s_un, w13_b_un, N_pad, K_pad, group_size):
    """Replicate ``_process_weights_for_sm90_cutlass`` for w13: de-interleave
    HF's pair-wise ``[g_0, u_0, g_1, u_1, ...]`` layout into halved
    ``[up; gate]``, pad each half along its row dim from ``N_un -> N_pad``
    and last dim from ``K_un -> K_pad`` with zeros, then run the FlashInfer
    SM90 byte / scale interleave helpers."""
    E, two_n_un, last_un_w = w13_un.shape
    N_un = two_n_un // 2
    K_un = last_un_w * 2  # packed 4-bit -> *2 for raw K

    def _split_and_pad(unpadded, last_pad, last_un, dtype):
        gate = unpadded[:, 0::2, :]
        up = unpadded[:, 1::2, :]
        out = torch.zeros(E, 2 * N_pad, last_pad, dtype=dtype, device=unpadded.device)
        out[:, :N_un, :last_un] = up
        out[:, N_pad : N_pad + N_un, :last_un] = gate
        return out

    w13_pad = _split_and_pad(
        w13_un.view(torch.uint8), K_pad // 2, K_un // 2, w13_un.dtype
    )
    w13_s_pad = _split_and_pad(
        w13_s_un, K_pad // group_size, K_un // group_size, w13_s_un.dtype
    )

    gate_b = w13_b_un[:, 0::2]
    up_b = w13_b_un[:, 1::2]
    w13_b_pad = torch.zeros(E, 2 * N_pad, dtype=w13_b_un.dtype, device=w13_b_un.device)
    w13_b_pad[:, :N_un] = up_b
    w13_b_pad[:, N_pad : N_pad + N_un] = gate_b

    w13_il = interleave_moe_weights_for_sm90_mixed_gemm(w13_pad, "fp4")
    w13_s_il = interleave_moe_scales_for_sm90_mixed_gemm(
        w13_s_pad, group_size=group_size
    )
    return w13_il, w13_s_il, w13_b_pad
```
**EN:** Replicate ``_process_weights_for_sm90_cutlass`` for w13: de-interleave HF's pair-wise ``[g_0, u_0, g_1, u_1, ...]`` layout into halved ``[up; gate]``, pad each half along its row dim from ``N_un -> N_pad`` and last dim from ``K_un -> K_pad`` with zeros, then run the FlashInfer SM90 byte / scale interleave helpers. This block implements `_expected_w13_processed` and captures one focused piece of the module's behavior.
**CN:** Replicate ``_process_weights_for_sm90_cutlass`` for w13: de-interleave HF's pair-wise ``[g_0, u_0, g_1, u_1, ...]`` layout into halved ``[up; gate]``, pad each half along its row dim from ``N_un -> N_pad`` and last dim from ``K_un -> K_pad`` with zeros, then run the FlashInfer SM90 byte / scale interleave helpers. 该代码块实现 `_expected_w13_processed`，承担模块行为中的一个聚焦逻辑片段。

### Lines 204-223: function expected w2 processed / 函数 expected w2 processed
```python
def _expected_w2_processed(w2_un, w2_s_un, w2_b_un, N_pad, K_pad, group_size):
    """w2 needs padding only (no halving / no de-interleave)."""
    E, K_un, last_un_w = w2_un.shape
    N_un = last_un_w * 2

    def _pad(unpadded, last_pad, last_un):
        out = torch.zeros(
            E, K_pad, last_pad, dtype=unpadded.dtype, device=unpadded.device
        )
        out[:, :K_un, :last_un] = unpadded
        return out

    w2_pad = _pad(w2_un.view(torch.uint8), N_pad // 2, N_un // 2)
    w2_s_pad = _pad(w2_s_un, N_pad // group_size, N_un // group_size)
    w2_b_pad = torch.zeros(E, K_pad, dtype=w2_b_un.dtype, device=w2_b_un.device)
    w2_b_pad[:, :K_un] = w2_b_un

    w2_il = interleave_moe_weights_for_sm90_mixed_gemm(w2_pad, "fp4")
    w2_s_il = interleave_moe_scales_for_sm90_mixed_gemm(w2_s_pad, group_size=group_size)
    return w2_il, w2_s_il, w2_b_pad
```
**EN:** w2 needs padding only (no halving / no de-interleave). This block implements `_expected_w2_processed` and captures one focused piece of the module's behavior.
**CN:** w2 needs padding only (no halving / no de-interleave). 该代码块实现 `_expected_w2_processed`，承担模块行为中的一个聚焦逻辑片段。

### Lines 226-280: test case process weights matches direct interleave / 测试用例 process weights matches direct interleave
```python
@pytest.mark.parametrize(
    "num_experts,hidden,inter",
    [
        # Aligned shapes (no padding needed).
        (4, 256, 256),
        (8, 768, 384),
        (8, 1024, 1024),
        # Non-aligned shapes (exercise the de-interleave + pad path).
        # 192 % 128 = 64, so N_pad = K_pad = 256 (round_up(192, 128)).
        (4, 192, 192),
        # GPT-OSS-20B-like: hidden=2880, inter=2880 -> padded to 2944.
        # Use smaller E to keep memory bounded.
        (4, 2880, 2880),
    ],
)
def test_process_weights_matches_direct_interleave(num_experts, hidden, inter):
    """``_process_weights_for_sm90_cutlass`` must produce the same bytes as
    a manual de-interleave + pad + halved-swap + interleave reference."""
    w13, w2, w13_s, w2_s, w13_b, w2_b = _make_random_mxfp4(num_experts, hidden, inter)

    layer = _build_mock_layer(
        num_experts, hidden, inter, w13, w2, w13_s, w2_s, w13_b, w2_b
    )
    method = _build_method(num_experts, hidden, inter)
    method._process_weights_for_sm90_cutlass(layer)

    N_pad = _round_up(inter, 128)
    K_pad = _round_up(hidden, 128)
    ref_w13, ref_w13_s, ref_w13_b = _expected_w13_processed(
        w13, w13_s, w13_b, N_pad, K_pad, GROUP_SIZE
    )
    ref_w2, ref_w2_s, ref_w2_b = _expected_w2_processed(
        w2, w2_s, w2_b, N_pad, K_pad, GROUP_SIZE
    )

    assert torch.equal(layer.w13_weight.data, ref_w13)
    assert torch.equal(layer.w2_weight.data, ref_w2)
    assert torch.equal(layer.w13_weight_scale.data, ref_w13_s)
    assert torch.equal(layer.w2_weight_scale.data, ref_w2_s)
    assert torch.equal(layer.w13_weight_bias.data, ref_w13_b)
    assert torch.equal(layer.w2_weight_bias.data, ref_w2_b)

    # SwiGLU per-expert scalars seeded with GPT-OSS defaults.
    assert torch.allclose(
        layer.swiglu_alpha,
        torch.full((num_experts,), 1.702, dtype=torch.float32, device="cuda"),
    )
    assert torch.allclose(
        layer.swiglu_beta,
        torch.full((num_experts,), 1.0, dtype=torch.float32, device="cuda"),
    )
    assert torch.allclose(
        layer.swiglu_limit,
        torch.full((num_experts,), 7.0, dtype=torch.float32, device="cuda"),
    )
```
**EN:** ``_process_weights_for_sm90_cutlass`` must produce the same bytes as a manual de-interleave + pad + halved-swap + interleave reference. This test exercises `test_process_weights_matches_direct_interleave` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** ``_process_weights_for_sm90_cutlass`` must produce the same bytes as a manual de-interleave + pad + halved-swap + interleave reference. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_process_weights_matches_direct_interleave`。

### Lines 283-362: test case apply sm90 cutlass matches flashinfer direct (part 1/2) / 测试用例 apply sm90 cutlass matches flashinfer direct（第 1/2 部分）
```python
@pytest.mark.parametrize(
    "tokens,num_experts,hidden,inter,top_k",
    [
        # Aligned shapes (no padding).
        (4, 4, 256, 256, 2),
        (16, 8, 768, 384, 2),
        (32, 8, 1024, 1024, 4),
        # Non-aligned (exercises pad x + trim output).
        (8, 4, 192, 192, 2),
    ],
)
def test_apply_sm90_cutlass_matches_flashinfer_direct(
    tokens, num_experts, hidden, inter, top_k, monkeypatch
):
    """End-to-end: SGLang's ``_apply_sm90_cutlass`` must produce the same
    output as a direct FlashInfer ``cutlass_fused_moe`` call fed with the
    same processed weights / scales / biases. The processing pipeline is
    covered separately by ``test_process_weights_matches_direct_interleave``;
    here we just verify that ``apply`` calls the kernel with the right
    arguments (incl. input padding + output trim)."""
    import sglang.srt.layers.quantization.mxfp4 as mxfp4_mod

    # Bypass symmetric-memory / TP-group: not relevant to numerics.
    monkeypatch.setattr(
        mxfp4_mod, "use_symmetric_memory", lambda *a, **kw: nullcontext()
    )
    monkeypatch.setattr(mxfp4_mod, "is_allocation_symmetric", lambda: False)
    monkeypatch.setattr(mxfp4_mod, "get_tp_group", lambda: None)

    w13, w2, w13_s, w2_s, w13_b, w2_b = _make_random_mxfp4(num_experts, hidden, inter)
    x = torch.randn(tokens, hidden, dtype=torch.bfloat16, device="cuda") * 0.1
    topk_w, topk_i = _make_topk(tokens, num_experts, top_k)

    # ---- SGLang path ----
    layer = _build_mock_layer(
        num_experts, hidden, inter, w13, w2, w13_s, w2_s, w13_b, w2_b
    )
    method = _build_method(num_experts, hidden, inter)
    method._process_weights_for_sm90_cutlass(layer)

    out_sglang = method._apply_sm90_cutlass(
        layer, x.clone(), _MockTopKOutput(topk_w, topk_i)
    ).hidden_states

    # ---- FlashInfer-direct reference using the same processed weights ----
    K_pad = method._padded_hidden
    if K_pad != hidden:
        x_padded = torch.nn.functional.pad(
            x.clone(), (0, K_pad - hidden), mode="constant", value=0.0
        )
    else:
        x_padded = x.clone()

    out_ref_padded = torch.empty(tokens, K_pad, dtype=torch.bfloat16, device="cuda")
    cutlass_fused_moe(
        input=x_padded,
        token_selected_experts=topk_i.to(torch.int),
        token_final_scales=topk_w,
        fc1_expert_weights=layer.w13_weight,
        fc2_expert_weights=layer.w2_weight,
        output_dtype=torch.bfloat16,
        quant_scales=[
            layer.w13_weight_scale.view(torch.int32),
            layer.w2_weight_scale.view(torch.int32),
        ],
        fc1_expert_biases=layer.w13_weight_bias,
        fc2_expert_biases=layer.w2_weight_bias,
        swiglu_alpha=layer.swiglu_alpha,
        swiglu_beta=layer.swiglu_beta,
        swiglu_limit=layer.swiglu_limit,
        use_w4_group_scaling=True,
        activation_type=ActivationType.Swiglu,
        output=out_ref_padded,
    )
    out_ref = (
        out_ref_padded[:, :hidden].contiguous() if K_pad != hidden else out_ref_padded
    )

    assert torch.equal(out_sglang, out_ref), (
        f"SGLang vs FlashInfer-direct mismatch; "
```
**EN:** End-to-end: SGLang's ``_apply_sm90_cutlass`` must produce the same output as a direct FlashInfer ``cutlass_fused_moe`` call fed with the same processed weights / scales / biases. This test exercises `test_apply_sm90_cutlass_matches_flashinfer_direct` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** End-to-end: SGLang's ``_apply_sm90_cutlass`` must produce the same output as a direct FlashInfer ``cutlass_fused_moe`` call fed with the same processed weights / scales / biases. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_apply_sm90_cutlass_matches_flashinfer_direct`。 这一段对应同一逻辑块的第 1 部分。

### Lines 363-364: test case apply sm90 cutlass matches flashinfer direct (part 2/2) / 测试用例 apply sm90 cutlass matches flashinfer direct（第 2/2 部分）
```python
        f"max abs diff = {(out_sglang.float() - out_ref.float()).abs().max().item():.4g}"
    )
```
**EN:** End-to-end: SGLang's ``_apply_sm90_cutlass`` must produce the same output as a direct FlashInfer ``cutlass_fused_moe`` call fed with the same processed weights / scales / biases. This test exercises `test_apply_sm90_cutlass_matches_flashinfer_direct` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** End-to-end: SGLang's ``_apply_sm90_cutlass`` must produce the same output as a direct FlashInfer ``cutlass_fused_moe`` call fed with the same processed weights / scales / biases. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_apply_sm90_cutlass_matches_flashinfer_direct`。 这一段对应同一逻辑块的第 2 部分。

### Lines 365-373: supporting source context / 辅助源码上下文
```python


# =============================================================================
# DeepSeek-V4 path: Mxfp4FlashinferCutlassMoEMethod (sibling of Marlin /
# trtllm-gen). Wired into fp8.py's get_quant_method when SM90 +
# is_flashinfer_mxfp4 + is_fp4_experts.
# =============================================================================


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 374-415: function make random dsv4 mxfp4 / 函数 make random dsv4 mxfp4
```python
def _make_random_dsv4_mxfp4(num_experts, hidden, inter, seed=0):
    """Mirrors the fp8 base method's allocation for fp4 experts: int8-packed
    4-bit weights, fp32 scales (containing 2**e values, not raw E8M0 bytes)."""
    g = torch.Generator(device="cuda").manual_seed(seed)
    # int8 storage (signed) -- matches Fp8MoEMethod.create_weights for fp4_experts.
    w13 = torch.randint(
        -128,
        128,
        (num_experts, 2 * inter, hidden // 2),
        dtype=torch.int8,
        device="cuda",
        generator=g,
    )
    w2 = torch.randint(
        -128,
        128,
        (num_experts, hidden, inter // 2),
        dtype=torch.int8,
        device="cuda",
        generator=g,
    )
    # fp32 scales whose bit pattern after .to(float8_e8m0fnu).view(uint8) lands
    # in a sane E8M0 band -- generate exponents around 0 (= 2**0).
    raw_e = torch.randint(
        125,
        130,
        (num_experts, 2 * inter, hidden // GROUP_SIZE),
        dtype=torch.uint8,
        device="cuda",
        generator=g,
    )
    raw_e2 = torch.randint(
        125,
        130,
        (num_experts, hidden, inter // GROUP_SIZE),
        dtype=torch.uint8,
        device="cuda",
        generator=g,
    )
    w13_s = raw_e.view(torch.float8_e8m0fnu).to(torch.float32)
    w2_s = raw_e2.view(torch.float8_e8m0fnu).to(torch.float32)
    return w13, w2, w13_s, w2_s
```
**EN:** Mirrors the fp8 base method's allocation for fp4 experts: int8-packed 4-bit weights, fp32 scales (containing 2**e values, not raw E8M0 bytes). This block implements `_make_random_dsv4_mxfp4` and captures one focused piece of the module's behavior.
**CN:** Mirrors the fp8 base method's allocation for fp4 experts: int8-packed 4-bit weights, fp32 scales (containing 2**e values, not raw E8M0 bytes). 该代码块实现 `_make_random_dsv4_mxfp4`，承担模块行为中的一个聚焦逻辑片段。

### Lines 418-497: test case dsv4 apply matches flashinfer direct (part 1/2) / 测试用例 dsv4 apply matches flashinfer direct（第 1/2 部分）
```python
@pytest.mark.parametrize(
    "tokens,num_experts,hidden,inter,top_k",
    [
        (4, 4, 256, 256, 2),
        (16, 8, 768, 384, 2),
        (256, 8, 1024, 1024, 4),
    ],
)
def test_dsv4_apply_matches_flashinfer_direct(
    tokens, num_experts, hidden, inter, top_k, monkeypatch
):
    """End-to-end: SGLang's DSv4 ``Mxfp4FlashinferCutlassMoEMethod.apply``
    output must match a direct FlashInfer ``cutlass_fused_moe`` call with
    the equivalent reorder + scale-cast + interleave applied manually."""
    from types import SimpleNamespace

    import sglang.srt.layers.quantization.mxfp4_flashinfer_cutlass_moe as ds_mod
    from sglang.srt.layers.quantization.utils import reorder_w1w3_to_w3w1

    # Bypass symmetric-memory / TP-group stack -- not relevant to numerics.
    monkeypatch.setattr(ds_mod, "use_symmetric_memory", lambda *a, **kw: nullcontext())
    monkeypatch.setattr(ds_mod, "is_allocation_symmetric", lambda: False)
    monkeypatch.setattr(ds_mod, "get_tp_group", lambda: None)

    w13, w2, w13_s, w2_s = _make_random_dsv4_mxfp4(num_experts, hidden, inter)
    x = torch.randn(tokens, hidden, dtype=torch.bfloat16, device="cuda") * 0.1
    topk_w, topk_i = _make_topk(tokens, num_experts, top_k)

    # ---- SGLang DSv4 path ----
    method = ds_mod.Mxfp4FlashinferCutlassMoEMethod.__new__(
        ds_mod.Mxfp4FlashinferCutlassMoEMethod
    )
    method._fp8 = SimpleNamespace(
        process_weights_after_loading=lambda layer: None,
    )
    method.prefix = "test"
    # plain SiLU * up — all three SwiGLU scalars None (no clamp configured).
    method._swiglu_alpha_tensor = None
    method._swiglu_beta_tensor = None
    method._swiglu_limit_tensor = None

    layer = _MockLayer()
    layer.w13_weight = torch.nn.Parameter(w13.clone(), requires_grad=False)
    layer.w2_weight = torch.nn.Parameter(w2.clone(), requires_grad=False)
    layer.w13_weight_scale_inv = torch.nn.Parameter(w13_s.clone(), requires_grad=False)
    layer.w2_weight_scale_inv = torch.nn.Parameter(w2_s.clone(), requires_grad=False)
    layer.num_local_experts = num_experts
    layer.moe_tp_size = 1
    layer.moe_tp_rank = 0
    layer.moe_ep_size = 1
    layer.moe_ep_rank = 0

    method.process_weights_after_loading(layer)

    out_sglang = method.apply(
        layer, _MockDispatchOutput(x.clone(), topk_w, topk_i)
    ).hidden_states

    # ---- Direct FlashInfer reference ----
    w13_re, w13_s_re = reorder_w1w3_to_w3w1(w13, w13_s)
    w13_s_u8 = w13_s_re.to(torch.float8_e8m0fnu).view(torch.uint8).contiguous()
    w2_s_u8 = w2_s.to(torch.float8_e8m0fnu).view(torch.uint8).contiguous()
    ref_w13 = interleave_moe_weights_for_sm90_mixed_gemm(
        w13_re.view(torch.uint8).contiguous(), "fp4"
    )
    ref_w2 = interleave_moe_weights_for_sm90_mixed_gemm(
        w2.view(torch.uint8).contiguous(), "fp4"
    )
    ref_w13_s = interleave_moe_scales_for_sm90_mixed_gemm(
        w13_s_u8, group_size=GROUP_SIZE
    )
    ref_w2_s = interleave_moe_scales_for_sm90_mixed_gemm(w2_s_u8, group_size=GROUP_SIZE)

    out_ref = torch.empty(tokens, hidden, dtype=torch.bfloat16, device="cuda")
    cutlass_fused_moe(
        input=x.clone(),
        token_selected_experts=topk_i,
        token_final_scales=topk_w,
        fc1_expert_weights=ref_w13,
        fc2_expert_weights=ref_w2,
```
**EN:** End-to-end: SGLang's DSv4 ``Mxfp4FlashinferCutlassMoEMethod.apply`` output must match a direct FlashInfer ``cutlass_fused_moe`` call with the equivalent reorder + scale-cast + interleave applied manually. This test exercises `test_dsv4_apply_matches_flashinfer_direct` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** End-to-end: SGLang's DSv4 ``Mxfp4FlashinferCutlassMoEMethod.apply`` output must match a direct FlashInfer ``cutlass_fused_moe`` call with the equivalent reorder + scale-cast + interleave applied manually. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dsv4_apply_matches_flashinfer_direct`。 这一段对应同一逻辑块的第 1 部分。

### Lines 498-514: test case dsv4 apply matches flashinfer direct (part 2/2) / 测试用例 dsv4 apply matches flashinfer direct（第 2/2 部分）
```python
        output_dtype=torch.bfloat16,
        quant_scales=[ref_w13_s.view(torch.int32), ref_w2_s.view(torch.int32)],
        fc1_expert_biases=None,
        fc2_expert_biases=None,
        swiglu_alpha=None,
        swiglu_beta=None,
        swiglu_limit=None,
        use_w4_group_scaling=True,
        activation_type=ActivationType.Swiglu,
        output=out_ref,
    )

    assert torch.equal(out_sglang, out_ref), (
        f"DSv4 SGLang vs FlashInfer-direct mismatch; "
        f"max abs diff = "
        f"{(out_sglang.float() - out_ref.float()).abs().max().item():.4g}"
    )
```
**EN:** End-to-end: SGLang's DSv4 ``Mxfp4FlashinferCutlassMoEMethod.apply`` output must match a direct FlashInfer ``cutlass_fused_moe`` call with the equivalent reorder + scale-cast + interleave applied manually. This test exercises `test_dsv4_apply_matches_flashinfer_direct` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** End-to-end: SGLang's DSv4 ``Mxfp4FlashinferCutlassMoEMethod.apply`` output must match a direct FlashInfer ``cutlass_fused_moe`` call with the equivalent reorder + scale-cast + interleave applied manually. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dsv4_apply_matches_flashinfer_direct`。 这一段对应同一逻辑块的第 2 部分。

### Lines 517-517: class _MockDispatchOutput declaration / 类 _MockDispatchOutput 声明
```python
class _MockDispatchOutput:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 518-520: supporting statements / 辅助语句
```python
    """Stand-in for StandardDispatchOutput. ``topk_output`` is a real
    ``StandardTopKOutput`` so ``TopKOutputChecker.format_is_standard``
    (an isinstance check) returns True without distributed init."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 522-538: method init / 方法 init
```python
    def __init__(self, hidden_states, topk_weights, topk_ids):
        from sglang.srt.layers.moe.topk import StandardTopKOutput

        self.hidden_states = hidden_states
        # router_logits is unused by Mxfp4FlashinferCutlassMoEMethod.apply;
        # supply a placeholder of the right shape to keep the NamedTuple happy.
        router_logits = torch.zeros(
            topk_ids.shape[0],
            int(topk_ids.max().item()) + 1 if topk_ids.numel() else 1,
            dtype=torch.float32,
            device=topk_ids.device,
        )
        self.topk_output = StandardTopKOutput(
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            router_logits=router_logits,
        )
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 541-544: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    import sys

    sys.exit(pytest.main([__file__, "-v"]))
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_MockLayer`: Stand-in for ``FusedMoE`` carrying the attributes the SM90 helpers read. / 用于组织相关测试、夹具或辅助方法。
- `_MockTopKOutput`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_make_random_mxfp4`: This block implements `_make_random_mxfp4` and captures one focused piece of the module's behavior. / 该代码块实现 `_make_random_mxfp4`，承担模块行为中的一个聚焦逻辑片段。
- `_make_topk`: This block implements `_make_topk` and captures one focused piece of the module's behavior. / 该代码块实现 `_make_topk`，承担模块行为中的一个聚焦逻辑片段。
- `_build_mock_layer`: This block implements `_build_mock_layer` and captures one focused piece of the module's behavior. / 该代码块实现 `_build_mock_layer`，承担模块行为中的一个聚焦逻辑片段。
- `_round_up`: This block implements `_round_up` and captures one focused piece of the module's behavior. / 该代码块实现 `_round_up`，承担模块行为中的一个聚焦逻辑片段。
- `_build_method`: This block implements `_build_method` and captures one focused piece of the module's behavior. / 该代码块实现 `_build_method`，承担模块行为中的一个聚焦逻辑片段。
- `_expected_w13_processed`: Replicate ``_process_weights_for_sm90_cutlass`` for w13: de-interleave HF's pair-wise ``[g_0, u_0, g_1, u_1, ...]`` layout into halved ``[up; gate]``, pad each half along its row dim from ``N_un -> N_pad`` and last dim from ``K_un -> K_pad`` with zeros, then run the FlashInfer SM90 byte / scale interleave helpers. / 该代码块实现 `_expected_w13_processed`，承担模块行为中的一个聚焦逻辑片段。
- `_MockTopKOutput.__init__`: This block implements `__init__` and captures one focused piece of the module's behavior. / 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。
- `_MockDispatchOutput.__init__`: This block implements `__init__` and captures one focused piece of the module's behavior. / 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `contextlib`
- **Third-party modules / 第三方模块**: `pytest`, `torch`, `flashinfer.fused_moe`, `flashinfer.fused_moe.core`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.srt.utils`

- **Total lines / 总行数**: 544
