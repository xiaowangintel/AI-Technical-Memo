# test_block_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_block_fp8.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_block_fp8, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_block_fp8 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-42)
```python
import pytest
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from tests.kernels.moe.utils import (
    make_dummy_moe_config,
    make_test_quant_config,
    make_test_weights,
    modular_triton_fused_moe,
)
from tests.kernels.quant_utils import (
    native_per_token_group_quant_fp8,
    native_w8a8_block_matmul,
)
from vllm.config import VllmConfig, set_current_vllm_config
from vllm.model_executor.layers.activation import SiluAndMul
from vllm.model_executor.layers.fused_moe import (
    fused_experts,
    fused_topk,
)
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.all2all_utils import (
    maybe_make_prepare_finalize,
)
from vllm.model_executor.layers.fused_moe.config import (
    fp8_w8a8_moe_quant_config,
)
from vllm.model_executor.layers.fused_moe.experts.deep_gemm_moe import (
    _valid_deep_gemm_shape,
)
from vllm.model_executor.layers.fused_moe.experts.triton_deep_gemm_moe import (
    TritonOrDeepGemmExperts,
)
from vllm.platforms import current_platform
from vllm.utils.deep_gemm import (
    get_mk_alignment_for_contiguous_layout,
    is_deep_gemm_e8m0_used,
)
from vllm.utils.import_utils import has_deep_gemm
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from tests.kernels.moe.utils, tests.kernels.quant_utils; and vLLM components like vllm.model_executor.layers.fused_moe.modular_kernel, vllm.config, vllm.model_executor.layers.activation, vllm.model_executor.layers.fused_moe.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 tests.kernels.moe.utils、tests.kernels.quant_utils；vLLM 内部组件，例如 vllm.model_executor.layers.fused_moe.modular_kernel、vllm.config、vllm.model_executor.layers.activation、vllm.model_executor.layers.fused_moe。

### Constants and module state (lines 44-44)
```python
dg_available = has_deep_gemm()
```
**EN:** This block prepares module-level state that later helpers and tests reuse.
**CN:** 该代码块准备了后续辅助函数和测试会复用的模块级状态。

### Top-level block starting at line 46 (lines 46-47)
```python
if current_platform.get_device_capability() < (9, 0):
    pytest.skip("FP8 Triton requires CUDA 9.0 or higher", allow_module_level=True)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 48 (lines 48-52)
```python
if current_platform.is_fp8_fnuz():
    pytest.skip(
        "Tests in this file require float8_e4m3fn and platform does not support",
        allow_module_level=True,
    )
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 54-105)
```python
vllm_config = VllmConfig()

# Test configurations
DTYPES = [torch.bfloat16]  # [torch.half, torch.bfloat16, torch.float32]
# Deepseek-V3's intermediate size 18432, so N is 18432*2/8=4608 at TP8
# and its hidden size is 7168.
MNK_FACTORS = [
    (1, 128, 128),
    (1, 128, 7168),
    (1, 1024, 7168),
    (1, 4608, 128),
    (1, 4608, 7168),
    (83, 128, 128),
    (83, 512, 512),
    (83, 4608, 512),
    (83, 4608, 7168),
    (128, 512, 512),
    (128, 1024, 7168),
    (128, 4608, 7168),
    (2048, 128, 128),
    (2048, 1024, 7168),
    (2048, 4608, 512),
    (2048, 4608, 7168),
    (8192, 128, 128),
    (8192, 128, 7168),
    (8192, 1024, 7168),
    (8192, 4608, 7168),
]

MNK_FACTORS_DG = [
    (128, 128, 128),
    (128, 128, 7168),
    (128, 1024, 7168),
    (128, 4608, 128),
    (128, 4608, 7168),
    (192, 512, 512),
    (192, 1024, 7168),
    (192, 4608, 7168),
    (1335, 128, 128),
    (1335, 1024, 7168),
    (1335, 4608, 512),
    (1335, 4608, 7168),
    (2048, 128, 128),
    (2048, 128, 7168),
    (2048, 1024, 7168),
    (2048, 4608, 7168),
]

BLOCK_SIZE = [[128, 128]]
E = [2, 8, 16]  # [128, 256]
TOP_KS = [1, 2, 6]
SEEDS = [0]
```
**EN:** This block centralizes shared constants and parameter grids, including DTYPES, MNK_FACTORS, MNK_FACTORS_DG, BLOCK_SIZE, E. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 DTYPES、MNK_FACTORS、MNK_FACTORS_DG、BLOCK_SIZE、E。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `torch_w8a8_block_fp8_moe` (lines 108-134)
```python
def torch_w8a8_block_fp8_moe(a, w1, w2, w1_s, w2_s, topk_weight, topk_ids, block_shape):
    """Fused moe with block-wise quantization using native torch."""
    B, D = a.shape
    topk = topk_ids.size(1)
    a = a.view(B, -1, D).repeat(1, topk, 1).reshape(-1, D)
    out = torch.zeros(B * topk, w2.shape[1], dtype=a.dtype, device=a.device)

    topk_weight = topk_weight.view(-1)
    topk_ids = topk_ids.view(-1)

    _, block_k = block_shape[0], block_shape[1]
    a_q, a_s = native_per_token_group_quant_fp8(a, block_k)
    a_q = a_q.to(torch.float32)
    for i in range(w1.shape[0]):
        mask = topk_ids == i
        if mask.sum():
            inter_out = native_w8a8_block_matmul(
                a_q[mask], w1[i], a_s[mask], w1_s[i], block_shape, output_dtype=a.dtype
            )
            act_out = SiluAndMul().forward_native(inter_out)
            act_out_q, act_out_s = native_per_token_group_quant_fp8(act_out, block_k)
            out[mask] = native_w8a8_block_matmul(
                act_out_q, w2[i], act_out_s, w2_s[i], block_shape, output_dtype=a.dtype
            )
    return (
        out.view(B, -1, w2.shape[1]) * topk_weight.view(B, -1, 1).to(out.dtype)
    ).sum(dim=1)
```
**EN:** This helper function implements the shared logic for torch w8a8 block FP8 MoE. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 torch w8a8 block FP8 MoE 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Top-level block starting at line 138 (lines 138-138)
```python
pytest.importorskip("torch.cuda")
```
**EN:** This top-level `Expr` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `Expr` 代码块执行周边测试所依赖的辅助逻辑。

### Function `setup_cuda` (lines 141-143)
```python
@pytest.fixture(autouse=True)
def setup_cuda():
    torch.set_default_device("cuda")
```
**EN:** This fixture prepares reusable state for setup CUDA. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该 fixture 为 setup CUDA 准备可复用的测试状态。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_w8a8_block_fp8_fused_moe` (lines 146-210)
```python
@pytest.mark.parametrize(("M", "N", "K"), MNK_FACTORS)
@pytest.mark.parametrize("E", E)
@pytest.mark.parametrize("topk", TOP_KS)
@pytest.mark.parametrize("block_size", BLOCK_SIZE)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("seed", SEEDS)
@torch.inference_mode()
def test_w8a8_block_fp8_fused_moe(
    M, N, K, E, topk, block_size, dtype, seed, monkeypatch, workspace_init
):
    if topk > E:
        pytest.skip(f"Skipping test; topk={topk} > E={E}")

    torch.manual_seed(seed)

    a = torch.randn((M, K), dtype=dtype) / 10
    score = torch.randn((M, E), dtype=dtype)

    w1, w2, quant_config = make_test_quant_config(
        E,
        N,
        K,
        dtype,
        quant_dtype=torch.float8_e4m3fn,
        per_act_token_quant=False,
        block_shape=block_size,
    )

    m_fused_moe = modular_triton_fused_moe(make_dummy_moe_config(), quant_config)

    topk_weights, topk_ids, _ = fused_topk(a, score.float(), topk, False)

    # Set the context to avoid lots of warning spam.
    with set_current_vllm_config(vllm_config):
        ref_out = torch_w8a8_block_fp8_moe(
            a,
            w1,
            w2,
            quant_config.w1_scale,
            quant_config.w2_scale,
            topk_weights,
            topk_ids,
            block_size,
        )

        out = fused_experts(
            a, w1, w2, topk_weights, topk_ids, quant_config=quant_config
        )

        m_out = m_fused_moe.apply(
            a,
            w1,
            w2,
            topk_weights,
            topk_ids,
            activation=MoEActivation.SILU,
            apply_router_weight_on_input=False,
            expert_map=None,
            global_num_experts=w1.shape[0],
        )

    # 0.039 only needed for M >= 8192
    tol = 0.035 if M < 8192 else 0.039
    torch.testing.assert_close(out, ref_out, atol=tol, rtol=tol)
    torch.testing.assert_close(m_out, ref_out, atol=tol, rtol=tol)
```
**EN:** This pytest case verifies w8a8 block FP8 fused MoE. It is parameterized across 6 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as M, N, K, E. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 w8a8 block FP8 fused MoE 的行为。 它通过 6 组参数化输入覆盖多种场景；它会使用诸如 M、N、K、E 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

### Function `test_w8a8_block_fp8_deep_gemm_fused_moe` (lines 213-318)
```python
@pytest.mark.parametrize(("M", "N", "K"), MNK_FACTORS_DG)
@pytest.mark.parametrize("E", E)
@pytest.mark.parametrize("topk", TOP_KS)
@pytest.mark.parametrize("seed", SEEDS)
@pytest.mark.skipif(not dg_available, reason="DeepGemm kernels not available.")
@pytest.mark.skipif(is_deep_gemm_e8m0_used(), reason="Not E8M0 scale MOE")
@torch.inference_mode()
def test_w8a8_block_fp8_deep_gemm_fused_moe(M, N, K, E, topk, seed, monkeypatch):
    if topk > E:
        pytest.skip(f"Skipping test: topk={topk} > E={E}")

    if not _valid_deep_gemm_shape(M, N, K):
        pytest.skip(f"Skipping test: invalid size m={M}, n={N}, k={K}")

    torch.manual_seed(seed)

    block_size = get_mk_alignment_for_contiguous_layout()
    dtype = torch.bfloat16

    a = torch.randn((M, K), dtype=dtype) / 10
    score = torch.randn((M, E), dtype=dtype)

    (_, w1, w1_s, _), (_, w2, w2_s, _) = make_test_weights(
        E,
        N,
        K,
        dtype,
        torch.float8_e4m3fn,
        per_out_ch_quant=False,
        block_shape=block_size,
    )

    # Note: for now use_compile will error out if the problem size is
    # large enough to trigger chunking. I'm leaving the flag and
    # setup code in case we are able to revisit this later.
    use_compile = False

    use_cudagraph = N >= 1024 and K >= 1024 and current_platform.is_cuda_alike()

    topk_weights, topk_ids, _ = fused_topk(a, score.float(), topk, False)
# ... excerpt ...
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            global_num_experts=E,
            activation=MoEActivation.SILU,
            apply_router_weight_on_input=False,
            expert_map=False,
        )

    # Set the context to avoid lots of warning spam.
    with set_current_vllm_config(vllm_config):
        ref_out = torch_w8a8_block_fp8_moe(
            a, w1, w2, w1_s, w2_s, topk_weights, topk_ids, block_size
        )

        if use_compile:
            deep_gemm_moe_fp8_fn = torch.compile(
                deep_gemm_moe_fp8, backend="inductor", fullgraph=True
            )
            torch._dynamo.mark_dynamic(a, 0)
            torch._dynamo.mark_dynamic(topk_weights, 0)
            torch._dynamo.mark_dynamic(topk_ids, 0)
        else:
            deep_gemm_moe_fp8_fn = deep_gemm_moe_fp8

        out = deep_gemm_moe_fp8_fn(a, w1, w2, w1_s, w2_s, topk_weights, topk_ids)

        if use_cudagraph:
            out.fill_(0)
            stream = torch.cuda.Stream()
            graph = torch.cuda.CUDAGraph()
            with torch.cuda.graph(graph, stream=stream):
                out = deep_gemm_moe_fp8_fn(
                    a, w1, w2, w1_s, w2_s, topk_weights, topk_ids
                )
            torch.accelerator.synchronize()
            graph.replay()
            torch.accelerator.synchronize()

    torch.testing.assert_close(out, ref_out, atol=0.035, rtol=0.035)
```
**EN:** This pytest case verifies w8a8 block FP8 deep gemm fused MoE. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as M, N, K, E. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 w8a8 block FP8 deep gemm fused MoE 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 M、N、K、E 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。
- **Reusable fixtures / 可复用 fixture:** Fixtures package common setup so each test can focus on the scenario under study. / fixture 封装公共准备逻辑，让每个测试更专注于待验证场景。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm.model_executor.layers.fused_moe.modular_kernel`
- `tests.kernels.moe.utils -> make_dummy_moe_config, make_test_quant_config, make_test_weights, modular_triton_fused_moe`
- `tests.kernels.quant_utils -> native_per_token_group_quant_fp8, native_w8a8_block_matmul`
- `vllm.config -> VllmConfig, set_current_vllm_config`
- `vllm.model_executor.layers.activation -> SiluAndMul`
- `vllm.model_executor.layers.fused_moe -> fused_experts, fused_topk`
- `vllm.model_executor.layers.fused_moe.activation -> MoEActivation`
- `vllm.model_executor.layers.fused_moe.all2all_utils -> maybe_make_prepare_finalize`
- `vllm.model_executor.layers.fused_moe.config -> fp8_w8a8_moe_quant_config`
- `vllm.model_executor.layers.fused_moe.experts.deep_gemm_moe -> _valid_deep_gemm_shape`
- `vllm.model_executor.layers.fused_moe.experts.triton_deep_gemm_moe -> TritonOrDeepGemmExperts`
- `vllm.platforms -> current_platform`
- `vllm.utils.deep_gemm -> get_mk_alignment_for_contiguous_layout, is_deep_gemm_e8m0_used`
- `vllm.utils.import_utils -> has_deep_gemm`
