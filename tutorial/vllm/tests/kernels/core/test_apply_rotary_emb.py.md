# test_apply_rotary_emb.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/core/test_apply_rotary_emb.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / core / test_apply_rotary_emb, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / core / test_apply_rotary_emb 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-12)
```python
"""
Tests for ApplyRotaryEmb CustomOp dispatch behavior.

This test ensures that RotaryEmbedding classes correctly call the appropriate
ApplyRotaryEmb methods based on the calling context:

1. RotaryEmbedding.forward_native() -> ApplyRotaryEmb.forward_native()
2. RotaryEmbedding.forward_cuda() -> ApplyRotaryEmb.forward() (auto-dispatch)
3. RotaryEmbedding.forward_hip() -> ApplyRotaryEmb.forward() (auto-dispatch)
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 14-25)
```python
from dataclasses import dataclass

import pytest
import torch

from vllm.config import (
    CompilationConfig,
    VllmConfig,
    get_cached_compilation_config,
    set_current_vllm_config,
)
from vllm.platforms import current_platform
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as dataclasses, pytest, torch; and vLLM components like vllm.config, vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 dataclasses、pytest、torch；vLLM 内部组件，例如 vllm.config、vllm.platforms。

### Constants and module state (lines 27-27)
```python
CUDA_DEVICES = ["cuda:0"]
```
**EN:** This block centralizes shared constants and parameter grids, including CUDA_DEVICES. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 CUDA_DEVICES。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Class `RotaryEmbeddingTestCase` (lines 30-40)
```python
@dataclass
class RotaryEmbeddingTestCase:
    """Test case configuration for RotaryEmbedding dispatch tests."""

    name: str
    rope_class: type
    rope_kwargs: dict
    method_name: str  # forward_native, forward_cuda, forward
    positions_shape: tuple  # (num_tokens,) or (3, num_tokens) or (4, num_tokens)
    expect_forward_native: bool  # Should call ApplyRotaryEmb.forward_native()
    expect_forward: bool  # Should call ApplyRotaryEmb.forward()
```
**EN:** This dataclass packages the fields needed to describe RotaryEmbeddingTestCase. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 RotaryEmbeddingTestCase 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Function `get_test_cases` (lines 43-104)
```python
def get_test_cases() -> list[RotaryEmbeddingTestCase]:
    """Generate test cases for all RotaryEmbedding classes."""
    from vllm.model_executor.layers.rotary_embedding.ernie45_vl_rope import (
        Ernie4_5_VLRotaryEmbedding,
    )
    from vllm.model_executor.layers.rotary_embedding.mrope import MRotaryEmbedding
    from vllm.model_executor.layers.rotary_embedding.xdrope import XDRotaryEmbedding

    common_kwargs = {
        "head_size": 128,
        "rotary_dim": 128,
        "max_position_embeddings": 4096,
        "base": 10000,
        "is_neox_style": True,
        "dtype": torch.bfloat16,
    }

    return [
        # MRotaryEmbedding tests
        RotaryEmbeddingTestCase(
            name="MRotaryEmbedding.forward_native",
            rope_class=MRotaryEmbedding,
            rope_kwargs={**common_kwargs, "mrope_section": [16, 24, 24]},
            method_name="forward_native",
            positions_shape=(3, 32),  # 2D for multimodal
            expect_forward_native=True,
            expect_forward=False,
        ),
        RotaryEmbeddingTestCase(
            name="MRotaryEmbedding.forward_cuda_1d",
            rope_class=MRotaryEmbedding,
            rope_kwargs={**common_kwargs, "mrope_section": [16, 24, 24]},
            method_name="forward_cuda",
            positions_shape=(32,),  # 1D triggers apply_rotary_emb path
            expect_forward_native=False,
            expect_forward=True,
        ),
        # XDRotaryEmbedding tests
        RotaryEmbeddingTestCase(
            name="XDRotaryEmbedding.forward",
            rope_class=XDRotaryEmbedding,
            rope_kwargs={
                **common_kwargs,
                "scaling_alpha": 1.0,
                "xdrope_section": [16, 16, 16, 16],
            },
            method_name="forward",
            positions_shape=(4, 32),  # 4D for P/W/H/T
            expect_forward_native=False,
            expect_forward=True,
        ),
        # Ernie4_5_VLRotaryEmbedding tests
        RotaryEmbeddingTestCase(
            name="Ernie4_5_VLRotaryEmbedding.forward_native",
            rope_class=Ernie4_5_VLRotaryEmbedding,
            rope_kwargs={**common_kwargs, "mrope_section": [22, 22, 20]},
            method_name="forward_native",
            positions_shape=(3, 32),  # 2D for multimodal
            expect_forward_native=True,
            expect_forward=False,
        ),
    ]
```
**EN:** This helper function implements the shared logic for test cases. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 test cases 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `run_dispatch_test` (lines 107-185)
```python
def run_dispatch_test(
    test_case: RotaryEmbeddingTestCase,
    device: str,
):
    """Run a dispatch test for a RotaryEmbedding class."""
    vllm_config = VllmConfig(
        compilation_config=CompilationConfig(custom_ops=["all", "+apply_rotary_emb"])
    )
    get_cached_compilation_config.cache_clear()

    with set_current_vllm_config(vllm_config):
        rope = test_case.rope_class(**test_case.rope_kwargs).to(device=device)

        apply_rotary_emb = rope.apply_rotary_emb

        # Verify custom op is enabled
        if test_case.expect_forward_native:
            assert (
                apply_rotary_emb._forward_method != apply_rotary_emb.forward_native
            ), "Test setup error: ApplyRotaryEmb custom op should be enabled"

        # Setup call tracking
        call_tracker = {"forward_native_called": False, "forward_called": False}
        original_forward_native = apply_rotary_emb.forward_native
        original_forward = apply_rotary_emb.forward

        def tracked_forward_native(*args, **kwargs):
            call_tracker["forward_native_called"] = True
            return original_forward_native(*args, **kwargs)

        def tracked_forward(*args, **kwargs):
            call_tracker["forward_called"] = True
            return original_forward(*args, **kwargs)

        apply_rotary_emb.forward_native = tracked_forward_native
        apply_rotary_emb.forward = tracked_forward

        try:
            num_tokens = test_case.positions_shape[-1]
            num_q_heads = 8
            num_kv_heads = 2
            head_size = test_case.rope_kwargs["head_size"]
            max_position = test_case.rope_kwargs["max_position_embeddings"]

            positions = torch.randint(
                0, max_position // 4, test_case.positions_shape, device=device
            )
            query = torch.randn(
                num_tokens, num_q_heads * head_size, dtype=torch.bfloat16, device=device
            )
            key = torch.randn(
                num_tokens,
                num_kv_heads * head_size,
                dtype=torch.bfloat16,
                device=device,
            )

            # Call the method under test
            method = getattr(rope, test_case.method_name)
            method(positions, query.clone(), key.clone())

            # Verify expectations
            if test_case.expect_forward_native:
                assert call_tracker["forward_native_called"], (
                    f"{test_case.name} should call ApplyRotaryEmb.forward_native()"
                )
            if not test_case.expect_forward:
                assert not call_tracker["forward_called"], (
                    f"{test_case.name} should NOT call ApplyRotaryEmb.forward(). "
                    "Bug: when +apply_rotary_emb is enabled, forward_native() "
                    "incorrectly dispatches to CUDA/HIP kernels."
                )
            if test_case.expect_forward:
                assert call_tracker["forward_called"], (
                    f"{test_case.name} should call ApplyRotaryEmb.forward()"
                )
        finally:
            apply_rotary_emb.forward_native = original_forward_native
            apply_rotary_emb.forward = original_forward
```
**EN:** This helper function implements the shared logic for run dispatch test. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 run dispatch test 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `test_rotary_embedding_dispatch` (lines 188-203)
```python
@pytest.mark.skipif(
    not current_platform.is_cuda_alike(), reason="Skipping CUDA/ROCm only tests."
)
@pytest.mark.parametrize("test_case", get_test_cases(), ids=lambda tc: tc.name)
@pytest.mark.parametrize("device", CUDA_DEVICES)
def test_rotary_embedding_dispatch(
    test_case: RotaryEmbeddingTestCase,
    device: str,
):
    """
    Test that RotaryEmbedding classes dispatch to the correct ApplyRotaryEmb method.

    - forward_native methods should call ApplyRotaryEmb.forward_native()
    - forward_cuda/forward methods should call ApplyRotaryEmb.forward()
    """
    run_dispatch_test(test_case, device)
```
**EN:** This pytest case verifies rotary embedding dispatch. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as test_case, device. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 rotary embedding dispatch 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 test_case、device 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `dataclasses -> dataclass`
- `pytest`
- `torch`
- `vllm.config -> CompilationConfig, VllmConfig, get_cached_compilation_config, set_current_vllm_config`
- `vllm.platforms -> current_platform`
