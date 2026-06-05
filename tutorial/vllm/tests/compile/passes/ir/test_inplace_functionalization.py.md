# test_inplace_functionalization.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/passes/ir/test_inplace_functionalization.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / passes / ir / test_inplace_functionalization, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / passes / ir / test_inplace_functionalization 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-8)
```python
"""
Tests for IR inplace functionalization pass integration.

This test suite verifies that the inplace functionalization pass, lowering pass,
and clone cleanup pass work together correctly with donated buffer tracking.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 10-31)
```python
from collections.abc import Callable

import pytest
import torch
import torch._dynamo.exc
from torch import nn

import vllm.kernels  # noqa: F401 to register kernels
from vllm.compilation.passes.inductor_pass import InductorPass, get_pass_context
from vllm.compilation.passes.ir.clone_elimination import (
    UnsafeCloneEliminationPass,
)
from vllm.compilation.passes.ir.inplace_functionalization import (
    VllmIRInplaceFunctionalizationPass,
)
from vllm.compilation.passes.ir.lowering_pass import VllmIRLoweringPass
from vllm.config import VllmConfig
from vllm.ir import ops
from vllm.platforms import current_platform
from vllm.triton_utils import HAS_TRITON, tl, triton

from ...backend import TestBackend
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as collections.abc, pytest, torch, torch._dynamo.exc; shared test helpers from ...backend; and vLLM components like vllm.kernels  # noqa: F401 to register kernels, vllm.compilation.passes.inductor_pass, vllm.compilation.passes.ir.clone_elimination, vllm.compilation.passes.ir.inplace_functionalization.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 collections.abc、pytest、torch、torch._dynamo.exc；共享测试辅助模块，例如 ...backend；vLLM 内部组件，例如 vllm.kernels  # noqa: F401 to register kernels、vllm.compilation.passes.inductor_pass、vllm.compilation.passes.ir.clone_elimination、vllm.compilation.passes.ir.inplace_functionalization。

### Class `StoreDonationInfoPass` (lines 34-34)
```python
class StoreDonationInfoPass(InductorPass):
```
**EN:** This helper class groups the state and behavior needed for StoreDonationInfoPass. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 StoreDonationInfoPass 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `StoreDonationInfoPass.__init__` (lines 35-36)
```python
    def __init__(self):
        self.donated_input_ids_sets: list[set[int]] = []
```
**EN:** This method implements the initialization for `StoreDonationInfoPass`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `StoreDonationInfoPass` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `StoreDonationInfoPass.__call__` (lines 38-40)
```python
    def __call__(self, *args, **kwargs):
        ctx = get_pass_context()
        self.donated_input_ids_sets += [ctx.donated_input_ids]
```
**EN:** This method implements the call entry for `StoreDonationInfoPass`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `StoreDonationInfoPass` 的调用入口。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `MaybeInplaceModel` (lines 43-45)
```python
class MaybeInplaceModel(nn.Module):
    """Model using only maybe_inplace variants."""
```
**EN:** This helper class groups the state and behavior needed for MaybeInplaceModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 MaybeInplaceModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `MaybeInplaceModel.__init__` (lines 46-49)
```python
    def __init__(self, hidden_size=16):
        super().__init__()
        self.weight1 = nn.Parameter(torch.ones(hidden_size, dtype=torch.bfloat16))
        self.weight2 = nn.Parameter(torch.ones(hidden_size, dtype=torch.bfloat16))
```
**EN:** This method implements the initialization for `MaybeInplaceModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `MaybeInplaceModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `MaybeInplaceModel.forward` (lines 51-62)
```python
    def forward(
        self, x: torch.Tensor, residual1: torch.Tensor, residual2: torch.Tensor
    ):
        # First maybe_inplace - x & residual1 are donated
        x_normed1, residual_out1 = ops.fused_add_rms_norm.maybe_inplace(
            x, residual1, self.weight1, 1e-5
        )
        # Second maybe_inplace - residual2 is donated
        x_normed2, residual_out2 = ops.fused_add_rms_norm.maybe_inplace(
            x_normed1, residual2, self.weight2, 1e-5
        )
        return x_normed2, residual_out1, residual_out2
```
**EN:** This method on `MaybeInplaceModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `MaybeInplaceModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `FunctionalModel` (lines 65-67)
```python
class FunctionalModel(nn.Module):
    """Model using only functional (default) variants."""
```
**EN:** This helper class groups the state and behavior needed for FunctionalModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 FunctionalModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `FunctionalModel.__init__` (lines 68-71)
```python
    def __init__(self, hidden_size=16):
        super().__init__()
        self.weight1 = nn.Parameter(torch.ones(hidden_size, dtype=torch.bfloat16))
        self.weight2 = nn.Parameter(torch.ones(hidden_size, dtype=torch.bfloat16))
```
**EN:** This method implements the initialization for `FunctionalModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `FunctionalModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `FunctionalModel.forward` (lines 73-84)
```python
    def forward(
        self, x: torch.Tensor, residual1: torch.Tensor, residual2: torch.Tensor
    ):
        # First functional - no donation
        x_normed1, residual_out1 = ops.fused_add_rms_norm(
            x, residual1, self.weight1, 1e-5
        )
        # Second functional - no donation
        x_normed2, residual_out2 = ops.fused_add_rms_norm(
            x_normed1, residual2, self.weight2, 1e-5
        )
        return x_normed2, residual_out1, residual_out2
```
**EN:** This method on `FunctionalModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `FunctionalModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `MixedModel` (lines 87-89)
```python
class MixedModel(nn.Module):
    """Model mixing maybe_inplace and functional variants."""
```
**EN:** This helper class groups the state and behavior needed for MixedModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 MixedModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `MixedModel.__init__` (lines 90-93)
```python
    def __init__(self, hidden_size=16):
        super().__init__()
        self.weight1 = nn.Parameter(torch.ones(hidden_size, dtype=torch.bfloat16))
        self.weight2 = nn.Parameter(torch.ones(hidden_size, dtype=torch.bfloat16))
```
**EN:** This method implements the initialization for `MixedModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `MixedModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `MixedModel.forward` (lines 95-107)
```python
    def forward(
        self, x: torch.Tensor, residual1: torch.Tensor, residual2: torch.Tensor
    ):
        # First maybe_inplace - x & residual1 are donated
        x_normed1, residual_out1 = ops.fused_add_rms_norm.maybe_inplace(
            x, residual1, self.weight1, 1e-5
        )
        # Second functional - no donation, x_normed1 must be preserved as it's returned
        x_normed2, residual_out2 = ops.fused_add_rms_norm(
            x_normed1, residual2, self.weight2, 1e-5
        )
        # Return both to prevent x_normed1 from being optimized away
        return x_normed1, x_normed2, residual_out1, residual_out2
```
**EN:** This method on `MixedModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `MixedModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `ModelWithTritonAfterMaybeInplace` (lines 110-115)
```python
class ModelWithTritonAfterMaybeInplace(nn.Module):
    """
    Model using maybe_inplace followed by a Triton kernel.
    Test clone elimination can handle Triton in the graph
    """
```
**EN:** This helper class groups the state and behavior needed for ModelWithTritonAfterMaybeInplace. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 ModelWithTritonAfterMaybeInplace 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `ModelWithTritonAfterMaybeInplace.__init__` (lines 116-143)
```python
    def __init__(self, hidden_size=16):
        super().__init__()
        self.weight = nn.Parameter(torch.ones(hidden_size, dtype=torch.bfloat16))

        @triton.jit
        def _triton_add_kernel(
            x_ptr,
            y_ptr,
            n_elements,
            BLOCK_SIZE: tl.constexpr,
        ):
            pid = tl.program_id(axis=0)
            block_start = pid * BLOCK_SIZE
            offsets = block_start + tl.arange(0, BLOCK_SIZE)
            mask = offsets < n_elements
            x = tl.load(x_ptr + offsets, mask=mask)
            y = x + 0.1
            tl.store(y_ptr + offsets, y, mask=mask)

        def triton_add(x: torch.Tensor) -> torch.Tensor:
            """Simple Triton add kernel."""
            y = torch.empty_like(x)
            n_elements = x.numel()
            grid = (triton.cdiv(n_elements, 256),)
            _triton_add_kernel[grid](x, y, n_elements, BLOCK_SIZE=256)
            return y

        self.triton_add = triton_add
```
**EN:** This method implements the initialization for `ModelWithTritonAfterMaybeInplace`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `ModelWithTritonAfterMaybeInplace` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `ModelWithTritonAfterMaybeInplace.forward` (lines 145-156)
```python
    def forward(self, x: torch.Tensor, residual: torch.Tensor, residual2: torch.Tensor):
        x_normed, residual_out = ops.fused_add_rms_norm.maybe_inplace(
            x, residual, self.weight, 1e-5
        )

        x_processed = self.triton_add(x_normed)

        # x_processed does not need to be cloned, residual2 does
        x_normed2, residual_out2 = ops.fused_add_rms_norm(
            x_processed, residual2, self.weight, 1e-5
        )
        return x_normed2, residual_out2
```
**EN:** This method on `ModelWithTritonAfterMaybeInplace` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `ModelWithTritonAfterMaybeInplace` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Constants and module state (lines 159-159)
```python
skipif_no_triton = pytest.mark.skipif(not HAS_TRITON, reason="Requires Triton")
```
**EN:** This block prepares module-level state that later helpers and tests reuse.
**CN:** 该代码块准备了后续辅助函数和测试会复用的模块级状态。

### Function `test_inplace_functionalization` (lines 162-246)
```python
@pytest.mark.skipif(
    not current_platform.is_cuda_alike(),
    reason="Only test on cuda and rocm platform",
)
@pytest.mark.parametrize(
    "model_class,expected_functionalized,expected_donated,expected_clones",
    [
        # 2 inplace calls, all activations donated, all clones eliminated
        (MaybeInplaceModel, 2, 3, 0),
        # No inplace calls, no donations, 3 clones (one eliminated)
        (FunctionalModel, 0, 0, 3),
        # One inplace call, two donated activations, 2 clones
        (MixedModel, 1, 2, 2),
        # One inplace call, two donated, 1 clone remaining
        pytest.param(ModelWithTritonAfterMaybeInplace, 1, 2, 1, marks=skipif_no_triton),
    ],
)
def test_inplace_functionalization(
    default_vllm_config: VllmConfig,
    model_class,
    expected_functionalized: int,
    expected_clones: int,
    expected_donated: int,
):
    """Test inplace functionalization, lowering, and clone cleanup."""
    torch.set_default_device(current_platform.device_type)

    # Use vllm_c so inplace path is triggered
    default_vllm_config.kernel_config.ir_op_priority.fused_add_rms_norm = [
        "vllm_c",
        "native",
    ]

    # Create passes in order they run during compilation
    functionalization_pass = VllmIRInplaceFunctionalizationPass(default_vllm_config)
    lowering_pass = VllmIRLoweringPass(default_vllm_config)
    donated_info_pass = StoreDonationInfoPass()
    cleanup_pass = UnsafeCloneEliminationPass(default_vllm_config)

    # Set up backend with pre-grad pass
# ... excerpt ...
    residual2 = torch.randn(8, 16, dtype=torch.bfloat16)

    with default_vllm_config.kernel_config.ir_op_priority.set_priority():
        # Reference output without optimization
        ref_output = model(x.clone(), residual1.clone(), residual2.clone())

        # Compile with inplace optimization
        compiled_model = torch.compile(model, backend=backend, fullgraph=True)
        output = compiled_model(x.clone(), residual1.clone(), residual2.clone())

    # Verify correctness (relaxed tolerance for bfloat16)
    for i in range(len(ref_output)):
        torch.testing.assert_close(output[i], ref_output[i], rtol=1e-2, atol=1e-2)

    # Verify expected number of ops were functionalized
    func_ops = functionalization_pass.functionalized_ops
    assert len(func_ops) == int(bool(expected_functionalized))
    if expected_functionalized > 0:
        assert "fused_add_rms_norm" in func_ops
        assert func_ops["fused_add_rms_norm"] == expected_functionalized

    # Verify lowering happened (2 ops in all cases)
    assert "fused_add_rms_norm" in lowering_pass.selected_impls
    assert len(lowering_pass.selected_impls["fused_add_rms_norm"]) == 2
    assert all(
        provider == "vllm_c"
        for node, provider in lowering_pass.selected_impls["fused_add_rms_norm"].items()
    ), lowering_pass.selected_impls

    # Verify correct number of donated IDs
    assert len(donated_info_pass.donated_input_ids_sets) == 1
    assert len(donated_info_pass.donated_input_ids_sets[0]) == expected_donated

    # Verify expected number of clones after cleanup
    actual_clones = backend.op_count(torch.ops.aten.clone.default, before=False)
    assert actual_clones == expected_clones, (
        f"Expected {expected_clones} clones, got {actual_clones}:"
        f"{backend.print_graphs()}"
    )
```
**EN:** This pytest case verifies inplace functionalization. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as default_vllm_config, model_class, expected_functionalized, expected_clones. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 inplace functionalization 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 default_vllm_config、model_class、expected_functionalized、expected_clones 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

### Function `test_donated_buffer_context_propagation` (lines 249-282)
```python
@pytest.mark.skipif(
    not current_platform.is_cuda_alike(),
    reason="Only test on cuda and rocm platform",
)
def test_donated_buffer_context_propagation(default_vllm_config):
    """Test that donated_input_ids propagates correctly through pass_context."""
    torch.set_default_device(current_platform.device_type)

    # Create a custom backend that inspects pass_context in cleanup pass
    functionalization_pass = VllmIRInplaceFunctionalizationPass(default_vllm_config)
    lowering_pass = VllmIRLoweringPass(default_vllm_config)

    donation_info_pass = StoreDonationInfoPass()
    cleanup_pass = UnsafeCloneEliminationPass(default_vllm_config)

    backend = TestBackend(lowering_pass, donation_info_pass, cleanup_pass)
    backend.inductor_config["pre_grad_custom_pass"] = functionalization_pass

    model = MaybeInplaceModel()
    x = torch.randn(8, 16, dtype=torch.bfloat16)
    residual1 = torch.randn(8, 16, dtype=torch.bfloat16)
    residual2 = torch.randn(8, 16, dtype=torch.bfloat16)

    compiled_model = torch.compile(model, backend=backend, fullgraph=True)
    compiled_model(x.clone(), residual1.clone(), residual2.clone())

    donated_ids_seen = donation_info_pass.donated_input_ids_sets
    # Verify donated_input_ids was set and propagated
    assert len(donated_ids_seen) == 1
    # Should have donated inputs (exact indices depend on AOTAutograd)
    assert len(donated_ids_seen[0]) == 3
    # All donated ids should be valid non-negative integers
    for idx in donated_ids_seen[0]:
        assert isinstance(idx, int) and idx >= 0, f"Invalid donated index: {idx}"
```
**EN:** This pytest case verifies donated buffer context propagation. it consumes fixtures or inputs such as default_vllm_config. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 donated buffer context propagation 的行为。 它会使用诸如 default_vllm_config 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Function `test_maybe_inplace_reuse_error` (lines 285-325)
```python
@pytest.mark.skipif(
    not current_platform.is_cuda_alike(),
    reason="Only test on cuda and rocm platform",
)
def test_maybe_inplace_reuse_error(default_vllm_config):
    """Test that reusing a donated activation input raises ValueError."""
    torch.set_default_device(current_platform.device_type)

    class ReuseModel(nn.Module):
        """Model that incorrectly reuses a donated activation input."""

        def __init__(self, hidden_size=16):
            super().__init__()
            self.weight = nn.Parameter(torch.ones(hidden_size, dtype=torch.bfloat16))

        def forward(self, x: torch.Tensor, residual: torch.Tensor):
            # x is donated to maybe_inplace
            x_normed, residual_out = ops.fused_add_rms_norm.maybe_inplace(
                x, residual, self.weight, 1e-5
            )
            # ERROR: x is used again after being donated
            return x_normed + x  # This should raise ValueError

    functionalization_pass = VllmIRInplaceFunctionalizationPass(default_vllm_config)
    lowering_pass = VllmIRLoweringPass(default_vllm_config)
    cleanup_pass = UnsafeCloneEliminationPass(default_vllm_config)

    backend = TestBackend(lowering_pass, cleanup_pass)
    backend.inductor_config["pre_grad_custom_pass"] = functionalization_pass

    model = ReuseModel()
    x = torch.randn(8, 16, dtype=torch.bfloat16)
    residual = torch.randn(8, 16, dtype=torch.bfloat16)

    # Compilation should raise BackendCompilerFailed wrapping ValueError
    with pytest.raises(
        torch._dynamo.exc.BackendCompilerFailed,
        match="is used again after the node",
    ):
        compiled_model = torch.compile(model, backend=backend, fullgraph=True)
        compiled_model(x.clone(), residual.clone())
```
**EN:** This pytest case verifies maybe inplace reuse error. it consumes fixtures or inputs such as default_vllm_config. unsupported hardware, backend, or configuration combinations are skipped early. the expected failure path is asserted explicitly.
**CN:** 该 pytest 用例验证 maybe inplace reuse error 的行为。 它会使用诸如 default_vllm_config 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；代码会显式断言预期的失败路径。

### Function `test_split_marker` (lines 331-334)
```python
@torch.library.custom_op("vllm::test_split_marker", mutates_args=())
def test_split_marker(x: torch.Tensor) -> torch.Tensor:
    """Identity op that marks a split point for piecewise compilation."""
    return x.clone()
```
**EN:** This pytest case verifies split marker. it consumes fixtures or inputs such as x.
**CN:** 该 pytest 用例验证 split marker 的行为。 它会使用诸如 x 等 fixture 或输入。

### Function `_fake_split_marker` (lines 337-339)
```python
@test_split_marker.register_fake
def _fake_split_marker(x: torch.Tensor) -> torch.Tensor:
    return torch.empty_like(x)
```
**EN:** This helper function implements the shared logic for fake split marker. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 fake split marker 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TransformerBlockWithSplits` (lines 342-344)
```python
class TransformerBlockWithSplits(nn.Module):
    """Transformer block with explicit split points for piecewise compilation."""
```
**EN:** This helper class groups the state and behavior needed for TransformerBlockWithSplits. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TransformerBlockWithSplits 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TransformerBlockWithSplits.__init__` (lines 345-372)
```python
    def __init__(self, hidden_size=32, intermediate_size=128):
        super().__init__()
        self.hidden_size = hidden_size
        self.intermediate_size = intermediate_size

        # Attention-like projection
        self.attn_proj = nn.Linear(
            hidden_size, hidden_size, bias=False, dtype=torch.bfloat16
        )

        # Post-attention norm
        self.post_attn_norm = nn.Parameter(
            torch.ones(hidden_size, dtype=torch.bfloat16)
        )

        # MLP
        self.gate_proj = nn.Linear(
            hidden_size, intermediate_size, bias=False, dtype=torch.bfloat16
        )
        self.up_proj = nn.Linear(
            hidden_size, intermediate_size, bias=False, dtype=torch.bfloat16
        )
        self.down_proj = nn.Linear(
            intermediate_size, hidden_size, bias=False, dtype=torch.bfloat16
        )

        # Post-MLP norm
        self.post_mlp_norm = nn.Parameter(torch.ones(hidden_size, dtype=torch.bfloat16))
```
**EN:** This method implements the initialization for `TransformerBlockWithSplits`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TransformerBlockWithSplits` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TransformerBlockWithSplits.forward` (lines 374-397)
```python
    def forward(self, x: torch.Tensor):
        # Attention block with residual
        residual1 = x
        attn_out = self.attn_proj(x)

        # Fused add + norm (maybe_inplace: residual1 is donated)
        normed1, residual1 = ops.fused_add_rms_norm.maybe_inplace(
            attn_out, residual1, self.post_attn_norm, 1e-5
        )

        # Force a graph split here
        normed1 = torch.ops.vllm.test_split_marker(normed1)

        # MLP block
        gate = self.gate_proj(normed1)
        up = self.up_proj(normed1)
        mlp_out = self.down_proj(gate * torch.nn.functional.silu(up))

        # Fused add + norm (maybe_inplace: residual1 is donated)
        normed2, residual2 = ops.fused_add_rms_norm.maybe_inplace(
            mlp_out, residual1, self.post_mlp_norm, 1e-5
        )

        return normed2, residual2
```
**EN:** This method on `TransformerBlockWithSplits` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TransformerBlockWithSplits` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `with_dyn_arg` (lines 400-405)
```python
def with_dyn_arg(fn: Callable, arg_index: int, dim_index: int):
    def inner(*args):
        torch._dynamo.mark_dynamic(args[arg_index], dim_index)
        return fn(*args)

    return inner
```
**EN:** This helper function implements the shared logic for with dyn arg. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 with dyn arg 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_piecewise_compilation_with_donated_buffers` (lines 408-465)
```python
@pytest.mark.skipif(
    not current_platform.is_cuda_alike(),
    reason="Only test on cuda and rocm platform",
)
def test_piecewise_compilation_with_donated_buffers(monkeypatch, fresh_vllm_cache):
    """
    Test piecewise compilation with donated buffers across graph splits.
    Utilizes a custom splitting op. Uses fresh cache to avoid compilation caching.
    """
    torch.set_default_device(current_platform.device_type)

    # Disable compilation cache to avoid serialization issues
    monkeypatch.setenv("VLLM_DISABLE_COMPILE_CACHE", "1")

    from vllm.compilation.backends import VllmBackend
    from vllm.config import CompilationConfig, VllmConfig

    # Create config with custom splitting op
    store_donation_info = StoreDonationInfoPass()
    vllm_config = VllmConfig(
        compilation_config=CompilationConfig(
            custom_ops=["all"],
            splitting_ops=["vllm::test_split_marker"],
            inductor_compile_config={"post_grad_custom_post_pass": store_donation_info},
        )
    )

    backend = VllmBackend(vllm_config)

    model = TransformerBlockWithSplits()
    x = torch.randn(8, 32, dtype=torch.bfloat16)

    # Reference output
    ref_output = with_dyn_arg(model, 0, 0)(x.clone())

    # Compile with piecewise compilation (graph will split at split_marker)
    compiled_model = torch.compile(model, backend=backend, fullgraph=False)
    output = with_dyn_arg(compiled_model, 0, 0)(x.clone())

    # Verify correctness (relaxed tolerance for bfloat16)
    torch.testing.assert_close(output[0], ref_output[0], rtol=1e-2, atol=1e-2)
    torch.testing.assert_close(output[1], ref_output[1], rtol=1e-2, atol=1e-2)

    # Verify the model was split into multiple submodules
    assert hasattr(backend, "split_gm"), "Backend should have split graph module"

    # Should have at least 2 submodules (split by test_split_marker op)
    submodules = list(backend.split_gm.named_children())
    num_submodules = len(submodules)
    assert num_submodules >= 2, (
        f"Expected at least 2 submodules (split), got {num_submodules}"
    )

    # Check that donation info was propagated correctly
    donated_inputs_sets = store_donation_info.donated_input_ids_sets
    assert len(donated_inputs_sets) == 2
    assert len(donated_inputs_sets[0]) == 1
    assert len(donated_inputs_sets[1]) == 1
```
**EN:** This pytest case verifies piecewise compilation with donated buffers. it consumes fixtures or inputs such as monkeypatch, fresh_vllm_cache. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 piecewise compilation with donated buffers 的行为。 它会使用诸如 monkeypatch、fresh_vllm_cache 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `collections.abc -> Callable`
- `pytest`
- `torch`
- `torch._dynamo.exc`
- `torch -> nn`
- `vllm.kernels`
- `vllm.compilation.passes.inductor_pass -> InductorPass, get_pass_context`
- `vllm.compilation.passes.ir.clone_elimination -> UnsafeCloneEliminationPass`
- `vllm.compilation.passes.ir.inplace_functionalization -> VllmIRInplaceFunctionalizationPass`
- `vllm.compilation.passes.ir.lowering_pass -> VllmIRLoweringPass`
- `vllm.config -> VllmConfig`
- `vllm.ir -> ops`
- `vllm.platforms -> current_platform`
- `vllm.triton_utils -> HAS_TRITON, tl, triton`
- `...backend -> TestBackend`
