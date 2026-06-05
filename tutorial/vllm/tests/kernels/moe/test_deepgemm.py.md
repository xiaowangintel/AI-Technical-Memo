# test_deepgemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_deepgemm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_deepgemm, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_deepgemm 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-6)
```python
"""
Unit-test DeepGEMM FP8 and FP4 kernels (no DeepEP).
Compare DeepGEMM path against the Triton fallback inside vLLM's fused_experts.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 8-39)
```python
import importlib
import math

import pytest
import torch

# vLLM fused-expert reference (Triton fallback + DeepGEMM option)
import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from tests.kernels.moe.utils import make_dummy_moe_config
from vllm.model_executor.layers.fused_moe.activation import (
    MoEActivation,
)
from vllm.model_executor.layers.fused_moe.all2all_utils import (
    maybe_make_prepare_finalize,
)
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEQuantConfig,
    FusedMoEQuantDesc,
    fp8_w8a8_moe_quant_config,
)
from vllm.model_executor.layers.fused_moe.experts.triton_deep_gemm_moe import (
    TritonOrDeepGemmExperts,
)
from vllm.model_executor.layers.fused_moe.fused_moe import fused_experts
from vllm.model_executor.layers.quantization.utils.fp8_utils import (
    per_token_group_quant_fp8,
)
from vllm.utils.deep_gemm import (
    calc_diff,
    is_deep_gemm_supported,
    per_block_cast_to_fp8,
)
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as importlib, math, pytest, torch; shared test helpers from tests.kernels.moe.utils; and vLLM components like vllm.model_executor.layers.fused_moe.modular_kernel, vllm.model_executor.layers.fused_moe.activation, vllm.model_executor.layers.fused_moe.all2all_utils, vllm.model_executor.layers.fused_moe.config.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 importlib、math、pytest、torch；共享测试辅助模块，例如 tests.kernels.moe.utils；vLLM 内部组件，例如 vllm.model_executor.layers.fused_moe.modular_kernel、vllm.model_executor.layers.fused_moe.activation、vllm.model_executor.layers.fused_moe.all2all_utils、vllm.model_executor.layers.fused_moe.config。

### Constants and module state (lines 41-41)
```python
BLOCK_SIZE = [128, 128]
```
**EN:** This block centralizes shared constants and parameter grids, including BLOCK_SIZE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 BLOCK_SIZE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `make_block_quant_fp8_weights` (lines 44-88)
```python
def make_block_quant_fp8_weights(
    e: int,
    n: int,
    k: int,
    block_size: list[int],
):
    """
    Generate (w1, w2) expert weights and their per-block scale tensors
    in FP8 block-quantized format.

      w1 shape: (E, 2N, K)
      w2 shape: (E, K, N)
    """
    dtype = torch.bfloat16
    fp8_max, fp8_min = (
        torch.finfo(torch.float8_e4m3fn).max,
        torch.finfo(torch.float8_e4m3fn).min,
    )

    # bf16 reference weights
    w1_bf16 = torch.randn(e, 2 * n, k, device="cuda", dtype=dtype) / 10
    w2_bf16 = torch.randn(e, k, n, device="cuda", dtype=dtype) / 10
    w1_bf16.clamp_(fp8_min, fp8_max)
    w2_bf16.clamp_(fp8_min, fp8_max)

    block_n, block_k = block_size
    n_tiles_w1 = math.ceil((2 * n) / block_n)
    k_tiles_w1 = math.ceil(k / block_k)
    n_tiles_w2 = math.ceil(k / block_n)
    k_tiles_w2 = math.ceil(n / block_k)

    w1 = torch.empty_like(w1_bf16, dtype=torch.float8_e4m3fn)
    w2 = torch.empty_like(w2_bf16, dtype=torch.float8_e4m3fn)
    w1_s = torch.empty(e, n_tiles_w1, k_tiles_w1, device="cuda", dtype=torch.float32)
    w2_s = torch.empty(e, n_tiles_w2, k_tiles_w2, device="cuda", dtype=torch.float32)

    for i in range(e):
        w1[i], w1_s[i] = per_block_cast_to_fp8(
            w1_bf16[i], block_size=block_size, use_ue8m0=True
        )
        w2[i], w2_s[i] = per_block_cast_to_fp8(
            w2_bf16[i], block_size=block_size, use_ue8m0=True
        )

    return w1, w2, w1_s, w2_s
```
**EN:** This helper function implements the shared logic for block quant FP8 weights. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 block quant FP8 weights 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `run_single_case` (lines 91-156)
```python
def run_single_case(m, n, k, topk, num_experts, block_size):
    """
    Run one (M,N,K) configuration on a single GPU and assert DeepGEMM ==
    Triton baseline within tolerance.
    """
    tokens_bf16 = (
        torch.randn(m, k, device="cuda", dtype=torch.bfloat16)
        .clamp_min_(-1)
        .clamp_max_(1)
    )
    _, a1_scale = per_token_group_quant_fp8(tokens_bf16, block_size[1])

    # expert weight tensors
    w1, w2, w1_s, w2_s = make_block_quant_fp8_weights(num_experts, n, k, block_size)

    router_logits = torch.randn(m, num_experts, device="cuda", dtype=torch.float32)
    topk_weights, topk_ids = torch.topk(router_logits, k=topk, dim=-1)
    topk_weights = torch.nn.functional.softmax(topk_weights, dim=-1)

    quant_config = fp8_w8a8_moe_quant_config(
        w1_scale=w1_s,
        w2_scale=w2_s,
        a1_scale=a1_scale,
        block_shape=block_size,
    )
    moe_config = make_dummy_moe_config()

    deep_gemm_experts = mk.FusedMoEKernel(
        prepare_finalize=maybe_make_prepare_finalize(
            moe=moe_config,
            quant_config=quant_config,
            allow_new_interface=True,
            use_monolithic=False,
        ),
        fused_experts=TritonOrDeepGemmExperts(
            moe_config=moe_config,
            quant_config=quant_config,
        ),
        inplace=False,
    )

    # triton reference
    out_triton = fused_experts(
        hidden_states=tokens_bf16,
        w1=w1,
        w2=w2,
        topk_weights=topk_weights,
        topk_ids=topk_ids,
        inplace=False,
        quant_config=quant_config,
    )

    # DeepGemm
    out_deepgemm = deep_gemm_experts.apply(
        hidden_states=tokens_bf16,
        w1=w1,
        w2=w2,
        topk_weights=topk_weights,
        topk_ids=topk_ids,
        global_num_experts=num_experts,
        activation=MoEActivation.SILU,
        apply_router_weight_on_input=False,
        expert_map=None,
    )
    diff = calc_diff(out_deepgemm, out_triton)
    assert diff < 0.001, f"Diff exceeded 1%: {diff}"
```
**EN:** This helper function implements the shared logic for run single case. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 run single case 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Constants and module state (lines 160-168)
```python
MNKs = [
    (1024, 768, 128),
    (2048, 768, 512),
    (512, 1024, 1024),
    (4096, 4096, 1024),
]

TOPKS = [2, 6]
NUM_EXPERTS = [32]
```
**EN:** This block centralizes shared constants and parameter grids, including TOPKS, NUM_EXPERTS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 TOPKS、NUM_EXPERTS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_deepgemm_vs_triton` (lines 171-208)
```python
@pytest.mark.parametrize(("m", "n", "k"), MNKs)
@pytest.mark.parametrize("topk", TOPKS)
@pytest.mark.parametrize("num_experts", NUM_EXPERTS)
@pytest.mark.skipif(not is_deep_gemm_supported(), reason="Requires deep_gemm kernels")
def test_deepgemm_vs_triton(m, n, k, topk, num_experts, monkeypatch, workspace_init):
    with monkeypatch.context() as mp:
        mp.setenv("VLLM_USE_DEEP_GEMM", "1")

        _DeepGemmExperts = importlib.import_module(
            "vllm.model_executor.layers.fused_moe.experts.deep_gemm_moe"
        ).DeepGemmExperts

        call_counter = {"cnt": 0}

        orig_fn = _DeepGemmExperts.apply

        def _spy_apply(*args, **kwargs):
            call_counter["cnt"] += 1
            return orig_fn(*args, **kwargs)

        monkeypatch.setattr(_DeepGemmExperts, "apply", _spy_apply)
        if topk > num_experts:
            pytest.skip(f"topk={topk} > num_experts={num_experts}")

        run_single_case(
            m=m,
            n=n,
            k=k,
            topk=topk,
            num_experts=num_experts,
            block_size=BLOCK_SIZE,
        )

        # ensure that the DeepGEMM path was indeed taken.
        assert call_counter["cnt"] == 1, (
            f"DeepGEMM path was not executed during the test. "
            f"Call counter: {call_counter['cnt']}"
        )
```
**EN:** This pytest case verifies deepgemm vs triton. It is parameterized across 3 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, n, k, topk. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 deepgemm vs triton 的行为。 它通过 3 组参数化输入覆盖多种场景；它会使用诸如 m、n、k、topk 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Function `make_mxfp4_weights` (lines 216-257)
```python
def make_mxfp4_weights(
    e: int,
    n: int,
    k: int,
):
    """
    Generate (w1, w2) expert weights in MXFP4 packed format with float32 scales,
    plus BF16 reference weights for validation.

      w1 shape: (E, 2N, K//2) uint8    — packed FP4
      w2 shape: (E, K, N//2)  uint8    — packed FP4
      w1_s shape: (E, 2N, K//32) float32  — per-row block-32 scales
      w2_s shape: (E, K, N//32)  float32  — per-row block-32 scales
      w1_bf16: (E, 2N, K)   — original BF16 for reference
      w2_bf16: (E, K, N)    — original BF16 for reference
    """
    from deep_gemm.utils.math import per_token_cast_to_fp4

    dtype = torch.bfloat16
    gran_k = 32  # MXFP4 block size

    # bf16 reference weights — scale by 1/sqrt(dim) for numerical stability
    w1_bf16 = torch.randn(e, 2 * n, k, device="cuda", dtype=dtype) * (k**-0.5)
    w2_bf16 = torch.randn(e, k, n, device="cuda", dtype=dtype) * (n**-0.5)

    # Quantize per-expert to FP4
    w1 = torch.empty(e, 2 * n, k // 2, device="cuda", dtype=torch.uint8)
    w2 = torch.empty(e, k, n // 2, device="cuda", dtype=torch.uint8)
    w1_s = torch.empty(
        e, 2 * n, math.ceil(k / gran_k), device="cuda", dtype=torch.float32
    )
    w2_s = torch.empty(e, k, math.ceil(n / gran_k), device="cuda", dtype=torch.float32)

    for i in range(e):
        w1[i], w1_s[i] = per_token_cast_to_fp4(
            w1_bf16[i].float(), use_ue8m0=True, gran_k=gran_k
        )
        w2[i], w2_s[i] = per_token_cast_to_fp4(
            w2_bf16[i].float(), use_ue8m0=True, gran_k=gran_k
        )

    return w1, w2, w1_s, w2_s, w1_bf16, w2_bf16
```
**EN:** This helper function implements the shared logic for mxfp4 weights. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 mxfp4 weights 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `_bf16_moe_reference` (lines 260-279)
```python
def _bf16_moe_reference(x, w1, w2, topk_weights, topk_ids):
    """BF16 token-loop MoE reference for correctness testing."""
    import torch.nn.functional as F

    num_tokens, hidden_size = x.shape
    intermediate = w1.shape[1] // 2
    top_k = topk_ids.shape[1]

    output = torch.zeros(num_tokens, hidden_size, dtype=torch.float32, device=x.device)
    for t in range(num_tokens):
        for kk in range(top_k):
            e = topk_ids[t, kk].item()
            w = topk_weights[t, kk].item()
            fc1 = x[t : t + 1].float() @ w1[e].float().T
            linear = fc1[:, :intermediate]
            gate = fc1[:, intermediate:]
            act = F.silu(gate) * linear
            fc2 = act @ w2[e].float().T
            output[t] += w * fc2[0]
    return output.to(torch.bfloat16)
```
**EN:** This helper function implements the shared logic for BF16 MoE reference. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 BF16 MoE reference 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `run_single_fp4_case` (lines 282-347)
```python
def run_single_fp4_case(m, n, k, topk, num_experts):
    """
    Run one (M,N,K) configuration with FP4 weights on DeepGEMM and assert
    DeepGEMM FP4 == BF16 reference within tolerance.
    """
    tokens_bf16 = torch.randn(m, k, device="cuda", dtype=torch.bfloat16) * (k**-0.5)

    # FP4 expert weight tensors + BF16 originals for reference
    w1, w2, w1_s, w2_s, w1_bf16, w2_bf16 = make_mxfp4_weights(num_experts, n, k)

    router_logits = torch.randn(m, num_experts, device="cuda", dtype=torch.float32)
    topk_weights, topk_ids = torch.topk(router_logits, k=topk, dim=-1)
    topk_weights = torch.nn.functional.softmax(topk_weights, dim=-1)

    from vllm.model_executor.layers.quantization.utils.quant_utils import (
        GroupShape,
    )
    from vllm.platforms import current_platform

    _fp8_dtype = current_platform.fp8_dtype()
    _block_shape = GroupShape(128, 128)
    quant_config = FusedMoEQuantConfig(
        _a1=FusedMoEQuantDesc(_fp8_dtype, _block_shape, None, None, None, None),
        _a2=FusedMoEQuantDesc(_fp8_dtype, _block_shape, None, None, None, None),
        _w1=FusedMoEQuantDesc("mxfp4", None, w1_s, None, None, None),
        _w2=FusedMoEQuantDesc("mxfp4", None, w2_s, None, None, None),
    )
    moe_config = make_dummy_moe_config()

    from vllm.model_executor.layers.fused_moe.experts.deep_gemm_moe import (
        DeepGemmFP4Experts,
    )

    deep_gemm_fp4_experts = mk.FusedMoEKernel(
        prepare_finalize=maybe_make_prepare_finalize(
            moe=moe_config,
            quant_config=quant_config,
            allow_new_interface=True,
            use_monolithic=False,
        ),
        fused_experts=DeepGemmFP4Experts(
            moe_config=moe_config,
            quant_config=quant_config,
        ),
        inplace=False,
    )

    # DeepGEMM FP4 path
    out_deepgemm_fp4 = deep_gemm_fp4_experts.apply(
        hidden_states=tokens_bf16,
        w1=w1,
        w2=w2,
        topk_weights=topk_weights,
        topk_ids=topk_ids,
        global_num_experts=num_experts,
        activation=MoEActivation.SILU,
        apply_router_weight_on_input=False,
        expert_map=None,
    )

    # BF16 reference using the same original weights
    out_ref = _bf16_moe_reference(tokens_bf16, w1_bf16, w2_bf16, topk_weights, topk_ids)

    # FP4 vs BF16 reference: quantization error from FP4 weights + FP8 activations
    diff = calc_diff(out_deepgemm_fp4, out_ref)
    assert diff < 0.05, f"FP4 diff exceeded 5%: {diff}"
```
**EN:** This helper function implements the shared logic for run single fp4 case. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 run single fp4 case 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Constants and module state (lines 352-358)
```python
FP4_MNKs = [
    (128, 4096, 4096),  # DeepSeek V4 shape
    (256, 2048, 2048),  # Half-size variant
]

FP4_TOPKS = [2]
FP4_NUM_EXPERTS = [8]
```
**EN:** This block centralizes shared constants and parameter grids, including FP4_TOPKS, FP4_NUM_EXPERTS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 FP4_TOPKS、FP4_NUM_EXPERTS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_deepgemm_fp4_vs_triton` (lines 361-400)
```python
@pytest.mark.parametrize(("m", "n", "k"), FP4_MNKs)
@pytest.mark.parametrize("topk", FP4_TOPKS)
@pytest.mark.parametrize("num_experts", FP4_NUM_EXPERTS)
@pytest.mark.skipif(not is_deep_gemm_supported(), reason="Requires deep_gemm kernels")
def test_deepgemm_fp4_vs_triton(
    m, n, k, topk, num_experts, monkeypatch, workspace_init
):
    pytest.importorskip("deep_gemm.utils.math")
    with monkeypatch.context() as mp:
        mp.setenv("VLLM_USE_DEEP_GEMM", "1")

        _DeepGemmFP4Experts = importlib.import_module(
            "vllm.model_executor.layers.fused_moe.experts.deep_gemm_moe"
        ).DeepGemmFP4Experts

        call_counter = {"cnt": 0}

        orig_fn = _DeepGemmFP4Experts.apply

        def _spy_apply(*args, **kwargs):
            call_counter["cnt"] += 1
            return orig_fn(*args, **kwargs)

        monkeypatch.setattr(_DeepGemmFP4Experts, "apply", _spy_apply)
        if topk > num_experts:
            pytest.skip(f"topk={topk} > num_experts={num_experts}")

        run_single_fp4_case(
            m=m,
            n=n,
            k=k,
            topk=topk,
            num_experts=num_experts,
        )

        # ensure that the DeepGEMM FP4 path was indeed taken.
        assert call_counter["cnt"] == 1, (
            f"DeepGEMM FP4 path was not executed during the test. "
            f"Call counter: {call_counter['cnt']}"
        )
```
**EN:** This pytest case verifies deepgemm fp4 vs triton. It is parameterized across 3 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, n, k, topk. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 deepgemm fp4 vs triton 的行为。 它通过 3 组参数化输入覆盖多种场景；它会使用诸如 m、n、k、topk 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `importlib`
- `math`
- `pytest`
- `torch`
- `vllm.model_executor.layers.fused_moe.modular_kernel`
- `tests.kernels.moe.utils -> make_dummy_moe_config`
- `vllm.model_executor.layers.fused_moe.activation -> MoEActivation`
- `vllm.model_executor.layers.fused_moe.all2all_utils -> maybe_make_prepare_finalize`
- `vllm.model_executor.layers.fused_moe.config -> FusedMoEQuantConfig, FusedMoEQuantDesc, fp8_w8a8_moe_quant_config`
- `vllm.model_executor.layers.fused_moe.experts.triton_deep_gemm_moe -> TritonOrDeepGemmExperts`
- `vllm.model_executor.layers.fused_moe.fused_moe -> fused_experts`
- `vllm.model_executor.layers.quantization.utils.fp8_utils -> per_token_group_quant_fp8`
- `vllm.utils.deep_gemm -> calc_diff, is_deep_gemm_supported, per_block_cast_to_fp8`
