# test_rocm_aiter_topk.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_rocm_aiter_topk.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_rocm_aiter_topk, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_rocm_aiter_topk 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 12-17)
```python
import importlib.util

import pytest
import torch

from vllm.platforms import current_platform
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as importlib.util, pytest, torch; and vLLM components like vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 importlib.util、pytest、torch；vLLM 内部组件，例如 vllm.platforms。

### Top-level block starting at line 19 (lines 19-20)
```python
if not current_platform.is_rocm():
    pytest.skip("This test can only run on ROCm.", allow_module_level=True)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Imports and shared setup (lines 23-23)
```python
import vllm.model_executor.layers.fused_moe.experts.rocm_aiter_moe  # noqa: F401
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in and vLLM components like vllm.model_executor.layers.fused_moe.experts.rocm_aiter_moe  # noqa: F401.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括vLLM 内部组件，例如 vllm.model_executor.layers.fused_moe.experts.rocm_aiter_moe  # noqa: F401。

### Constants and module state (lines 27-27)
```python
aiter_available = importlib.util.find_spec("aiter") is not None
```
**EN:** This block prepares module-level state that later helpers and tests reuse.
**CN:** 该代码块准备了后续辅助函数和测试会复用的模块级状态。

### Top-level block starting at line 29 (lines 29-30)
```python
if not aiter_available:
    pytest.skip("These tests require AITER to run.", allow_module_level=True)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Function `test_rocm_aiter_biased_grouped_topk_custom_op_registration` (lines 33-39)
```python
def test_rocm_aiter_biased_grouped_topk_custom_op_registration():
    """Test that the custom op is correctly registered."""
    # Check if the op exists in torch.ops.vllm
    assert hasattr(torch.ops.vllm, "rocm_aiter_biased_grouped_topk")

    # Check if the op is callable
    assert callable(torch.ops.vllm.rocm_aiter_biased_grouped_topk)
```
**EN:** This pytest case verifies ROCm aiter biased grouped topk custom op registration. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 ROCm aiter biased grouped topk custom op registration 的行为。 结尾处的断言会固定预期行为或计算图形态。

### Function `test_rocm_aiter_grouped_topk_custom_op_registration` (lines 42-48)
```python
def test_rocm_aiter_grouped_topk_custom_op_registration():
    """Test that the custom op is correctly registered."""
    # Check if the op exists in torch.ops.vllm
    assert hasattr(torch.ops.vllm, "rocm_aiter_grouped_topk")

    # Check if the op is callable
    assert callable(torch.ops.vllm.rocm_aiter_grouped_topk)
```
**EN:** This pytest case verifies ROCm aiter grouped topk custom op registration. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 ROCm aiter grouped topk custom op registration 的行为。 结尾处的断言会固定预期行为或计算图形态。

### Function `test_rocm_aiter_biased_grouped_topk_torch_compile_compatibility` (lines 51-137)
```python
def test_rocm_aiter_biased_grouped_topk_torch_compile_compatibility():
    """Test that the op can be used with torch.compile."""
    # Create test tensors
    token = 64
    expert = 256
    num_expert_group = 8
    topk = 8
    topk_group = 4
    renormalize = True
    scale_factor = 1.0

    gating_output = torch.randn((token, expert), dtype=torch.bfloat16, device="cuda")
    e_score_correction_bias = torch.randn(
        (expert,), dtype=torch.bfloat16, device="cuda"
    )

    device = gating_output.device
    topk_ids = torch.empty((token, topk), dtype=torch.int32, device=device)
    topk_weights = torch.empty((token, topk), dtype=torch.float32, device=device)

    # Define a function that uses the op
    def biased_grouped_topk_fn(
        gating_output, e_score_correction_bias, topk_weights, topk_ids
    ):
        return torch.ops.vllm.rocm_aiter_biased_grouped_topk(
            gating_output,
            e_score_correction_bias,
            topk_weights,
            topk_ids,
            num_expert_group,
            topk_group,
            renormalize,
            scale_factor,
        )

    # Verify the op's fake implementation
    torch.library.opcheck(
        torch.ops.vllm.rocm_aiter_biased_grouped_topk,
        (gating_output, e_score_correction_bias, topk_weights, topk_ids),
        kwargs={
# ... excerpt ...
    # Compile the function with appropriate settings
    compiled_fn = torch.compile(
        biased_grouped_topk_fn,
        fullgraph=True,
        backend="inductor",
        mode="reduce-overhead",
        dynamic=False,
    )

    topk_weights_original = torch.empty(
        (token, topk), dtype=torch.float32, device=device
    )
    topk_ids_original = torch.empty((token, topk), dtype=torch.int32, device=device)

    topk_weights_compiled = torch.empty(
        (token, topk), dtype=torch.float32, device=device
    )
    topk_ids_compiled = torch.empty((token, topk), dtype=torch.int32, device=device)

    # Run both compiled (V1 graph mode) and uncompiled versions (V1 eager mode)
    biased_grouped_topk_fn(
        gating_output, e_score_correction_bias, topk_weights_original, topk_ids_original
    )
    compiled_fn(
        gating_output, e_score_correction_bias, topk_weights_compiled, topk_ids_compiled
    )

    # Sort the results for comparison since the order might not be deterministic
    topk_ids_original, indices_original = torch.sort(topk_ids_original)
    topk_weights_original = torch.gather(topk_weights_original, 1, indices_original)

    topk_ids_compiled, indices_compiled = torch.sort(topk_ids_compiled)
    topk_weights_compiled = torch.gather(topk_weights_compiled, 1, indices_compiled)

    # Verify results match
    assert torch.allclose(
        topk_weights_original, topk_weights_compiled, rtol=1e-2, atol=1e-2
    )
    assert torch.allclose(topk_ids_original, topk_ids_compiled)
```
**EN:** This pytest case verifies ROCm aiter biased grouped topk torch compile compatibility. assertions at the end lock in the intended behavior or graph shape. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 ROCm aiter biased grouped topk torch compile compatibility 的行为。 结尾处的断言会固定预期行为或计算图形态。 由于该代码块较大，这里只展示关键片段。

### Function `test_rocm_aiter_grouped_topk_torch_compile_compatibility` (lines 140-221)
```python
def test_rocm_aiter_grouped_topk_torch_compile_compatibility():
    """Test that the op can be used with torch.compile."""
    # Create test tensors
    token = 64
    expert = 256
    num_expert_group = 8
    topk = 8
    topk_group = 4
    renormalize = True
    scoring_func = "softmax"
    scale_factor = 1.0

    gating_output = torch.randn((token, expert), dtype=torch.bfloat16, device="cuda")

    device = gating_output.device
    topk_ids = torch.empty((token, topk), dtype=torch.int32, device=device)
    topk_weights = torch.empty((token, topk), dtype=torch.float32, device=device)

    # Define a function that uses the op
    def grouped_topk_fn(gating_output, topk_weights, topk_ids, scoring_func):
        return torch.ops.vllm.rocm_aiter_grouped_topk(
            gating_output,
            topk_weights,
            topk_ids,
            num_expert_group,
            topk_group,
            renormalize,
            scoring_func,
            scale_factor,
        )

    # Verify the op's fake implementation
    torch.library.opcheck(
        torch.ops.vllm.rocm_aiter_grouped_topk,
        (gating_output, topk_weights, topk_ids),
        kwargs={
            "num_expert_group": num_expert_group,
            "topk_group": topk_group,
            "need_renorm": renormalize,
            "scoring_func": scoring_func,
# ... excerpt ...
    )

    # Compile the function with appropriate settings
    compiled_fn = torch.compile(
        grouped_topk_fn,
        fullgraph=True,
        backend="inductor",
        mode="reduce-overhead",
        dynamic=False,
    )

    topk_weights_original = torch.empty(
        (token, topk), dtype=torch.float32, device=device
    )
    topk_ids_original = torch.empty((token, topk), dtype=torch.int32, device=device)

    topk_weights_compiled = torch.empty(
        (token, topk), dtype=torch.float32, device=device
    )
    topk_ids_compiled = torch.empty((token, topk), dtype=torch.int32, device=device)

    # Run both compiled (V1 graph mode) and uncompiled versions (V1 eager mode)
    grouped_topk_fn(
        gating_output, topk_weights_original, topk_ids_original, scoring_func
    )
    compiled_fn(gating_output, topk_weights_compiled, topk_ids_compiled, scoring_func)

    # Sort the results for comparison since the order might not be deterministic
    topk_ids_original, indices_original = torch.sort(topk_ids_original)
    topk_weights_original = torch.gather(topk_weights_original, 1, indices_original)

    topk_ids_compiled, indices_compiled = torch.sort(topk_ids_compiled)
    topk_weights_compiled = torch.gather(topk_weights_compiled, 1, indices_compiled)

    # Verify results match
    assert torch.allclose(
        topk_weights_original, topk_weights_compiled, rtol=1e-2, atol=1e-2
    )
    assert torch.allclose(topk_ids_original, topk_ids_compiled)
```
**EN:** This pytest case verifies ROCm aiter grouped topk torch compile compatibility. assertions at the end lock in the intended behavior or graph shape. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 ROCm aiter grouped topk torch compile compatibility 的行为。 结尾处的断言会固定预期行为或计算图形态。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `importlib.util`
- `pytest`
- `torch`
- `vllm.platforms -> current_platform`
- `vllm.model_executor.layers.fused_moe.experts.rocm_aiter_moe`
