# test_triton_moe_no_act_mul.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_triton_moe_no_act_mul.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_triton_moe_no_act_mul, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_triton_moe_no_act_mul 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-8)
```python
"""Tests for MoE with non-gated activations (*_no_mul).

These tests verify that MoE layers work correctly with activations like
silu_no_mul, gelu_no_mul, relu2_no_mul where the activation output dimension
equals N (not N // 2 like gated activations).
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 10-19)
```python
import pytest
import torch

from tests.kernels.moe.utils import make_dummy_moe_config
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.config import (
    FUSED_MOE_UNQUANTIZED_CONFIG,
)
from vllm.model_executor.layers.fused_moe.experts.triton_moe import TritonExperts
from vllm.platforms import current_platform
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from tests.kernels.moe.utils; and vLLM components like vllm.model_executor.layers.fused_moe.activation, vllm.model_executor.layers.fused_moe.config, vllm.model_executor.layers.fused_moe.experts.triton_moe, vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 tests.kernels.moe.utils；vLLM 内部组件，例如 vllm.model_executor.layers.fused_moe.activation、vllm.model_executor.layers.fused_moe.config、vllm.model_executor.layers.fused_moe.experts.triton_moe、vllm.platforms。

### Constants and module state (lines 22-31)
```python
M_SIZES = [1, 16, 64]
N_SIZES = [128, 256]
K_SIZES = [64, 128]
TOPK_VALUES = [1, 2]
NUM_EXPERTS = 8
NO_MUL_ACTIVATIONS = [
    MoEActivation.SILU_NO_MUL,
    MoEActivation.GELU_NO_MUL,
    MoEActivation.RELU2_NO_MUL,
]
```
**EN:** This block centralizes shared constants and parameter grids, including M_SIZES, N_SIZES, K_SIZES, TOPK_VALUES, NUM_EXPERTS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 M_SIZES、N_SIZES、K_SIZES、TOPK_VALUES、NUM_EXPERTS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `make_test_tensors` (lines 34-58)
```python
def make_test_tensors(
    m: int,
    n: int,
    k: int,
    num_experts: int,
    topk: int,
    dtype: torch.dtype = torch.bfloat16,
    device: str = "cuda",
):
    """Create test tensors for MoE with non-gated activation.

    For non-gated activations (*_no_mul):
    - w1: (E, N, K) - projects from K to N
    - w2: (E, K, N) - projects from N back to K (note: N, not N//2)
    """
    hidden_states = torch.randn(m, k, dtype=dtype, device=device)

    # For non-gated: w1 projects K -> N, w2 projects N -> K
    w1 = torch.randn(num_experts, n, k, dtype=dtype, device=device) * 0.1
    w2 = torch.randn(num_experts, k, n, dtype=dtype, device=device) * 0.1

    topk_weights = torch.ones(m, topk, dtype=torch.float32, device=device) / topk
    topk_ids = torch.randint(0, num_experts, (m, topk), device=device)

    return hidden_states, w1, w2, topk_weights, topk_ids
```
**EN:** This helper function implements the shared logic for test tensors. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 test tensors 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_triton_experts_no_mul_activation` (lines 61-144)
```python
@pytest.mark.skipif(
    not current_platform.has_device_capability(80),
    reason="Requires compute capability >= 8.0",
)
@pytest.mark.parametrize("m", M_SIZES)
@pytest.mark.parametrize("n", N_SIZES)
@pytest.mark.parametrize("k", K_SIZES)
@pytest.mark.parametrize("topk", TOPK_VALUES)
@pytest.mark.parametrize("activation", NO_MUL_ACTIVATIONS)
@torch.inference_mode()
def test_triton_experts_no_mul_activation(
    m: int,
    n: int,
    k: int,
    topk: int,
    activation: MoEActivation,
):
    hidden_states, w1, w2, topk_weights, topk_ids = make_test_tensors(
        m, n, k, NUM_EXPERTS, topk
    )

    experts = TritonExperts(
        moe_config=make_dummy_moe_config(),
        quant_config=FUSED_MOE_UNQUANTIZED_CONFIG,
    )

    ws1_shape, ws2_shape, out_shape = experts.workspace_shapes(
        M=m,
        N=n,
        K=k,
        topk=topk,
        global_num_experts=NUM_EXPERTS,
        local_num_experts=NUM_EXPERTS,
        expert_tokens_meta=None,
        activation=activation,
    )

    # Verify workspace shapes are correct for no_mul activation
    # workspace1 should handle activation_out_dim = N (not N//2)
    assert ws1_shape == (m, topk, max(n, k)), (
# ... excerpt ...
    )
    assert out_shape == (m, k), (
        f"output shape mismatch: expected {(m, k)}, got {out_shape}"
    )

    workspace1 = torch.empty(
        ws1_shape[0] * ws1_shape[1] * ws1_shape[2],
        dtype=hidden_states.dtype,
        device=hidden_states.device,
    )
    workspace2 = torch.empty(
        ws2_shape[0] * ws2_shape[1] * ws2_shape[2],
        dtype=hidden_states.dtype,
        device=hidden_states.device,
    )
    output = torch.zeros(m, k, dtype=hidden_states.dtype, device=hidden_states.device)

    experts.apply(
        output=output,
        hidden_states=hidden_states,
        w1=w1,
        w2=w2,
        topk_weights=topk_weights,
        topk_ids=topk_ids,
        activation=activation,
        global_num_experts=NUM_EXPERTS,
        expert_map=None,
        a1q_scale=None,
        a2_scale=None,
        workspace13=workspace1,
        workspace2=workspace2,
        expert_tokens_meta=None,
        apply_router_weight_on_input=False,
    )

    assert output.shape == (m, k), f"Expected shape {(m, k)}, got {output.shape}"
    assert not torch.isnan(output).any(), "Output contains NaN"
    assert not torch.isinf(output).any(), "Output contains Inf"
    assert output.abs().sum() > 0, "Output is all zeros"
```
**EN:** This pytest case verifies triton experts no mul activation. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, n, k, topk. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 triton experts no mul activation 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 m、n、k、topk 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。 由于该代码块较大，这里只展示关键片段。

### Function `test_workspace_shapes_no_mul_vs_gated` (lines 147-185)
```python
@pytest.mark.skipif(
    not current_platform.has_device_capability(80),
    reason="Requires compute capability >= 8.0",
)
@torch.inference_mode()
def test_workspace_shapes_no_mul_vs_gated():
    """Test that workspace shapes differ correctly between gated and non-gated."""
    from vllm.model_executor.layers.fused_moe.experts.triton_moe import TritonExperts

    M, N, K, topk = 64, 256, 128, 2

    experts = TritonExperts(
        moe_config=make_dummy_moe_config(),
        quant_config=FUSED_MOE_UNQUANTIZED_CONFIG,
    )

    ws1_no_mul, _, out_no_mul = experts.workspace_shapes(
        M, N, K, topk, 8, 8, None, MoEActivation.SILU_NO_MUL
    )

    ws1_gated, _, out_gated = experts.workspace_shapes(
        M, N, K, topk, 8, 8, None, MoEActivation.SILU
    )

    # For no_mul: activation_out_dim = N
    # For gated: activation_out_dim = N // 2
    # workspace1 should use max(activation_out_dim, K)
    activation_out_dim_no_mul = N
    activation_out_dim_gated = N // 2

    assert ws1_no_mul[2] == max(activation_out_dim_no_mul, K), (
        f"no_mul workspace1 last dim should be max({activation_out_dim_no_mul}, {K})"
    )
    assert ws1_gated[2] == max(activation_out_dim_gated, K), (
        f"gated workspace1 last dim should be max({activation_out_dim_gated}, {K})"
    )

    # Output shapes should be the same
    assert out_no_mul == out_gated == (M, K)
```
**EN:** This pytest case verifies workspace shapes no mul vs gated. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 workspace shapes no mul vs gated 的行为。 不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Function `test_adjust_n_for_activation` (lines 188-211)
```python
@pytest.mark.skipif(
    not current_platform.has_device_capability(80),
    reason="Requires compute capability >= 8.0",
)
@torch.inference_mode()
def test_adjust_n_for_activation():
    """Test the adjust_N_for_activation method."""
    from vllm.model_executor.layers.fused_moe.experts.triton_moe import TritonExperts

    experts = TritonExperts(
        moe_config=make_dummy_moe_config(),
        quant_config=FUSED_MOE_UNQUANTIZED_CONFIG,
    )

    N = 256

    # Gated activations should return N // 2
    assert experts.adjust_N_for_activation(N, MoEActivation.SILU) == N // 2
    assert experts.adjust_N_for_activation(N, MoEActivation.GELU) == N // 2

    # Non-gated activations should return N
    assert experts.adjust_N_for_activation(N, MoEActivation.SILU_NO_MUL) == N
    assert experts.adjust_N_for_activation(N, MoEActivation.GELU_NO_MUL) == N
    assert experts.adjust_N_for_activation(N, MoEActivation.RELU2_NO_MUL) == N
```
**EN:** This pytest case verifies adjust n for activation. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 adjust n for activation 的行为。 不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `tests.kernels.moe.utils -> make_dummy_moe_config`
- `vllm.model_executor.layers.fused_moe.activation -> MoEActivation`
- `vllm.model_executor.layers.fused_moe.config -> FUSED_MOE_UNQUANTIZED_CONFIG`
- `vllm.model_executor.layers.fused_moe.experts.triton_moe -> TritonExperts`
- `vllm.platforms -> current_platform`
