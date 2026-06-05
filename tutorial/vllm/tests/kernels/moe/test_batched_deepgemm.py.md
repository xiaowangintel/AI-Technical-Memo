# test_batched_deepgemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_batched_deepgemm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_batched_deepgemm, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_batched_deepgemm 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-22)
```python
import pytest
import torch

from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.config import fp8_w8a8_moe_quant_config
from vllm.model_executor.layers.fused_moe.experts.batched_deep_gemm_moe import (
    BatchedDeepGemmExperts,
)
from vllm.model_executor.layers.fused_moe.experts.fused_batched_moe import (
    BatchedTritonExperts,
)
from vllm.model_executor.layers.fused_moe.modular_kernel import FusedMoEKernel
from vllm.model_executor.layers.fused_moe.prepare_finalize.batched import (
    BatchedPrepareAndFinalize,
)
from vllm.utils.deep_gemm import calc_diff, is_deep_gemm_supported

from .test_deepgemm import make_block_quant_fp8_weights
from .utils import make_dummy_moe_config
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from .test_deepgemm, .utils; and vLLM components like vllm.model_executor.layers.fused_moe.activation, vllm.model_executor.layers.fused_moe.config, vllm.model_executor.layers.fused_moe.experts.batched_deep_gemm_moe, vllm.model_executor.layers.fused_moe.experts.fused_batched_moe.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 .test_deepgemm、.utils；vLLM 内部组件，例如 vllm.model_executor.layers.fused_moe.activation、vllm.model_executor.layers.fused_moe.config、vllm.model_executor.layers.fused_moe.experts.batched_deep_gemm_moe、vllm.model_executor.layers.fused_moe.experts.fused_batched_moe。

### Constants and module state (lines 24-24)
```python
BLOCK_SIZE = [128, 128]
```
**EN:** This block centralizes shared constants and parameter grids, including BLOCK_SIZE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 BLOCK_SIZE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_batched_deepgemm_vs_triton` (lines 27-124)
```python
@pytest.mark.skipif(not is_deep_gemm_supported(), reason="Requires deep_gemm kernels")
@pytest.mark.parametrize("E", [16, 32])  # number of experts
@pytest.mark.parametrize("T", [256, 512])  # tokens per expert
@pytest.mark.parametrize("K", [128, 256])  # hidden dim
@pytest.mark.parametrize("N", [512, 1024])  # intermediate dim per expert
@pytest.mark.parametrize("topk", [2, 4])
def test_batched_deepgemm_vs_triton(
    E: int, T: int, K: int, N: int, topk: int, monkeypatch, workspace_init
):
    """Compare BatchedDeepGemmExperts to BatchedTritonExperts."""

    monkeypatch.setenv("VLLM_USE_DEEP_GEMM", "1")

    device = "cuda"
    w1, w2, w1_s, w2_s = make_block_quant_fp8_weights(E, N, K, BLOCK_SIZE)

    M = E * T  # total tokens
    a = torch.randn(M, K, device=device, dtype=torch.bfloat16) / 10.0
    fp8_info = torch.finfo(torch.float8_e4m3fn)
    a.clamp_(fp8_info.min, fp8_info.max)

    # random router outputs → top-k indices / weights
    router_logits = torch.randn(M, E, device=device, dtype=torch.float32)
    topk_weights, topk_ids = torch.topk(router_logits, k=topk, dim=-1)
    topk_weights = torch.nn.functional.softmax(topk_weights, dim=-1)

    # token number for each expert
    cnt = torch.bincount(topk_ids.flatten(), minlength=E)
    max_cnt = int(cnt.max().item())
    # next power of 2 for max token number
    max_num_tokens = 1 << (max_cnt - 1).bit_length()

    prep_finalize = BatchedPrepareAndFinalize(
        max_num_tokens=max_num_tokens,
        num_local_experts=E,
        num_dispatchers=1,
        rank=0,
    )

    quant_config = fp8_w8a8_moe_quant_config(
# ... excerpt ...
    out_triton = mk_triton.apply(
        hidden_states=a,
        w1=w1,
        w2=w2,
        topk_weights=topk_weights,
        topk_ids=topk_ids,
        activation=MoEActivation.SILU,
        global_num_experts=E,
        expert_map=None,
        apply_router_weight_on_input=False,
    )

    # deepgemm
    deepgemm_experts = BatchedDeepGemmExperts(
        max_num_tokens=max_num_tokens,
        num_dispatchers=1,
        quant_config=quant_config,
        moe_config=make_dummy_moe_config(),
    )
    mk_deepgemm = FusedMoEKernel(
        prep_finalize,
        deepgemm_experts,
        inplace=False,
    )

    out_deepgemm = mk_deepgemm.apply(
        hidden_states=a,
        w1=w1,
        w2=w2,
        topk_weights=topk_weights,
        topk_ids=topk_ids,
        activation=MoEActivation.SILU,
        global_num_experts=E,
        expert_map=None,
        apply_router_weight_on_input=False,
    )

    diff = calc_diff(out_deepgemm, out_triton)
    assert diff < 1e-3, f"Output diff too large: {diff}"
```
**EN:** This pytest case verifies batched deepgemm vs triton. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as E, T, K, N. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 batched deepgemm vs triton 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 E、T、K、N 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm.model_executor.layers.fused_moe.activation -> MoEActivation`
- `vllm.model_executor.layers.fused_moe.config -> fp8_w8a8_moe_quant_config`
- `vllm.model_executor.layers.fused_moe.experts.batched_deep_gemm_moe -> BatchedDeepGemmExperts`
- `vllm.model_executor.layers.fused_moe.experts.fused_batched_moe -> BatchedTritonExperts`
- `vllm.model_executor.layers.fused_moe.modular_kernel -> FusedMoEKernel`
- `vllm.model_executor.layers.fused_moe.prepare_finalize.batched -> BatchedPrepareAndFinalize`
- `vllm.utils.deep_gemm -> calc_diff, is_deep_gemm_supported`
- `.test_deepgemm -> make_block_quant_fp8_weights`
- `.utils -> make_dummy_moe_config`
