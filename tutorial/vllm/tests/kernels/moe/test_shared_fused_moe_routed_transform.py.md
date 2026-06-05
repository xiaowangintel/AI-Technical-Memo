# test_shared_fused_moe_routed_transform.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_shared_fused_moe_routed_transform.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_shared_fused_moe_routed_transform, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_shared_fused_moe_routed_transform 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-8)
```python
"""
Tests for FusedMoE with routed_input_transform.

Verifies that applying routed_input_transform inside FusedMoE
produces the same results as applying the transform manually outside.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 10-18)
```python
import pytest
import torch
import torch.nn as nn

from vllm.config import VllmConfig, set_current_vllm_config
from vllm.forward_context import set_forward_context
from vllm.model_executor.layers.fused_moe import FusedMoE
from vllm.platforms import current_platform
from vllm.utils.torch_utils import is_torch_equal_or_newer, set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch, torch.nn; and vLLM components like vllm.config, vllm.forward_context, vllm.model_executor.layers.fused_moe, vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch、torch.nn；vLLM 内部组件，例如 vllm.config、vllm.forward_context、vllm.model_executor.layers.fused_moe、vllm.platforms。

### Class `SimpleLinear` (lines 21-23)
```python
class SimpleLinear(nn.Module):
    """A simple linear transform mimicking latent projection in latent MoE."""
```
**EN:** This helper class groups the state and behavior needed for SimpleLinear. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 SimpleLinear 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `SimpleLinear.__init__` (lines 24-28)
```python
    def __init__(self, in_features: int, out_features: int, dtype: torch.dtype):
        super().__init__()
        self.weight = nn.Parameter(
            torch.randn(out_features, in_features, device="cuda", dtype=dtype) / 10
        )
```
**EN:** This method implements the initialization for `SimpleLinear`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `SimpleLinear` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `SimpleLinear.forward` (lines 30-31)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return nn.functional.linear(x, self.weight)
```
**EN:** This method on `SimpleLinear` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `SimpleLinear` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `SimpleSharedExperts` (lines 34-36)
```python
class SimpleSharedExperts(nn.Module):
    """A simple 2-layer MLP mimicking shared experts."""
```
**EN:** This helper class groups the state and behavior needed for SimpleSharedExperts. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 SimpleSharedExperts 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `SimpleSharedExperts.__init__` (lines 37-47)
```python
    def __init__(self, hidden_size: int, intermediate_size: int, dtype: torch.dtype):
        super().__init__()
        self.up = nn.Linear(
            hidden_size, intermediate_size * 2, bias=False, device="cuda", dtype=dtype
        )
        self.down = nn.Linear(
            intermediate_size, hidden_size, bias=False, device="cuda", dtype=dtype
        )
        with torch.no_grad():
            self.up.weight.div_(10)
            self.down.weight.div_(10)
```
**EN:** This method implements the initialization for `SimpleSharedExperts`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `SimpleSharedExperts` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `SimpleSharedExperts.forward` (lines 49-52)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        gate_up = self.up(x)
        gate, up = gate_up.chunk(2, dim=-1)
        return self.down(nn.functional.silu(gate) * up)
```
**EN:** This method on `SimpleSharedExperts` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `SimpleSharedExperts` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `_assert_close` (lines 55-106)
```python
def _assert_close(
    actual: torch.Tensor,
    expected: torch.Tensor,
    atol: float,
    rtol: float,
    label: str,
) -> None:
    """assert_close that prints diff diagnostics on both success and failure."""
    actual_nans = int(actual.isnan().sum().item())
    expected_nans = int(expected.isnan().sum().item())
    actual_zeros = int((actual == 0).sum().item())
    expected_zeros = int((expected == 0).sum().item())
    n_total = actual.numel()

    diff = (actual - expected).abs()
    max_diff = diff.max().item()
    mean_diff = diff.mean().item()
    n_exceed = int((diff > atol).sum().item())
    pct_exceed = n_exceed / n_total * 100

    print(
        f"[{label}] "
        f"shape={list(actual.shape)}, "
        f"max_diff={max_diff:.6e}, "
        f"mean_diff={mean_diff:.6e}, "
        f"exceed_atol({atol})={n_exceed}/{n_total} ({pct_exceed:.2f}%), "
        f"actual=[{actual.min().item():.4f}, {actual.max().item():.4f}], "
        f"expected=[{expected.min().item():.4f}, {expected.max().item():.4f}], "
        f"nan(actual/expected)={actual_nans}/{expected_nans}, "
        f"zeros(actual/expected)={actual_zeros}/{expected_zeros}"
    )

    assert actual_nans == 0, (
        f"{label}: actual has {actual_nans}/{n_total} NaN values "
        f"(expected has {expected_nans}). "
        f"This indicates a kernel bug, not a precision issue."
    )
    assert expected_nans == 0, (
        f"{label}: expected has {expected_nans}/{n_total} NaN values. "
        f"This indicates a kernel bug, not a precision issue."
    )

    torch.testing.assert_close(
        actual,
        expected,
        atol=atol,
        rtol=rtol,
        msg=(
            f"{label}: max_diff={max_diff:.6e}, mean_diff={mean_diff:.6e}, "
            f"exceed_atol({atol})={n_exceed}/{n_total} ({pct_exceed:.2f}%)"
        ),
    )
```
**EN:** This helper function implements the shared logic for assert close. numeric results are compared against a reference with explicit tolerances.
**CN:** 该辅助函数实现了 assert close 所需的共享逻辑。 数值结果会在显式容差下与参考结果进行比较。

### Function `setup_cuda` (lines 109-113)
```python
@pytest.fixture(autouse=True)
def setup_cuda():
    if not torch.cuda.is_available():
        pytest.skip("CUDA not available")
    torch.set_default_device("cuda")
```
**EN:** This fixture prepares reusable state for setup CUDA. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 fixture 为 setup CUDA 准备可复用的测试状态。 不支持的硬件、后端或配置组合会被提前跳过。

### Function `test_routed_input_transform_inside_vs_outside` (lines 116-229)
```python
@pytest.mark.parametrize("num_tokens", [1, 32])
@pytest.mark.parametrize("hidden_size,latent_size", [(256, 128), (128, 64)])
@pytest.mark.parametrize("dtype", [torch.bfloat16])
@pytest.mark.parametrize(
    "use_rocm_aiter", [True, False] if current_platform.is_rocm() else [False]
)
@pytest.mark.skipif(
    is_torch_equal_or_newer("2.10.0"),
    reason="Test fails with PyTorch 2.10.0 see: https://github.com/vllm-project/vllm/issues/33995",
)
def test_routed_input_transform_inside_vs_outside(
    num_tokens: int,
    hidden_size: int,
    latent_size: int,
    dtype: torch.dtype,
    use_rocm_aiter: bool,
    dist_init,
    workspace_init,
    monkeypatch,
):
    """Compare FusedMoE with transform inside vs manually applying outside.
    Method A (inside): FusedMoE with routed_input_transform
    Method B (outside): Manually transform, then FusedMoE without transform
    """
    if current_platform.is_rocm():
        monkeypatch.setenv("VLLM_ROCM_USE_AITER", "1" if use_rocm_aiter else "0")
        monkeypatch.setenv("VLLM_ROCM_USE_AITER_MOE", "1" if use_rocm_aiter else "0")
        from vllm._aiter_ops import rocm_aiter_ops

        rocm_aiter_ops.refresh_env_variables()

    set_random_seed(42)

    num_experts = 8
    top_k = 2
    intermediate_size = hidden_size * 2

    vllm_config = VllmConfig()
    vllm_config.compilation_config.static_forward_context = dict()

# ... excerpt ...
        )

        # Weights are created via torch.empty (uninitialized).
        # Initialize with seeded random values for reproducibility.
        with torch.no_grad():
            moe_with_transform.w13_weight.normal_()
            moe_with_transform.w13_weight.div_(10)
            moe_with_transform.w2_weight.normal_()
            moe_with_transform.w2_weight.div_(10)
            moe_without_transform.w13_weight.copy_(moe_with_transform.w13_weight)
            moe_without_transform.w2_weight.copy_(moe_with_transform.w2_weight)

        moe_with_transform.quant_method.process_weights_after_loading(
            moe_with_transform
        )
        moe_without_transform.quant_method.process_weights_after_loading(
            moe_without_transform
        )

        hidden_states = torch.randn(num_tokens, hidden_size, device="cuda", dtype=dtype)
        router_logits = torch.randn(num_tokens, num_experts, device="cuda", dtype=dtype)

        with set_forward_context(None, vllm_config, num_tokens=num_tokens):
            # Method A: combined output (shared + routed)
            combined_A = moe_with_transform(hidden_states, router_logits)

            # Method B: manually transform, get routed output, add shared
            transformed_hidden = routed_transform(hidden_states)
            routed_out_B = moe_without_transform(transformed_hidden, router_logits)
            shared_out_B = shared_experts(hidden_states)
            combined_B = shared_out_B + routed_out_B

        torch.testing.assert_close(
            combined_A,
            combined_B,
            atol=1e-3,
            rtol=1e-3,
            msg="Combined output should match: transform inside vs outside",
        )
```
**EN:** This pytest case verifies routed input transform inside vs outside. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, hidden_size, latent_size, dtype. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 routed input transform inside vs outside 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、hidden_size、latent_size、dtype 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。
- **Reusable fixtures / 可复用 fixture:** Fixtures package common setup so each test can focus on the scenario under study. / fixture 封装公共准备逻辑，让每个测试更专注于待验证场景。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `torch.nn`
- `vllm.config -> VllmConfig, set_current_vllm_config`
- `vllm.forward_context -> set_forward_context`
- `vllm.model_executor.layers.fused_moe -> FusedMoE`
- `vllm.platforms -> current_platform`
- `vllm.utils.torch_utils -> is_torch_equal_or_newer, set_random_seed`
