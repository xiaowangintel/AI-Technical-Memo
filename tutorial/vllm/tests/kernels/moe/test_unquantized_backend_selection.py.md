# test_unquantized_backend_selection.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_unquantized_backend_selection.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_unquantized_backend_selection, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_unquantized_backend_selection 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-12)
```python
from unittest.mock import patch

import pytest

from tests.kernels.moe.utils import make_dummy_moe_config
from vllm.model_executor.layers.fused_moe.oracle.unquantized import (
    UnquantizedMoeBackend,
    select_unquantized_moe_backend,
)
from vllm.platforms import current_platform
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as unittest.mock, pytest; shared test helpers from tests.kernels.moe.utils; and vLLM components like vllm.model_executor.layers.fused_moe.oracle.unquantized, vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 unittest.mock、pytest；共享测试辅助模块，例如 tests.kernels.moe.utils；vLLM 内部组件，例如 vllm.model_executor.layers.fused_moe.oracle.unquantized、vllm.platforms。

### Constants and module state (lines 14-17)
```python
skipif_not_cuda_rocm = pytest.mark.skipif(
    not (current_platform.is_cuda() or current_platform.is_rocm()),
    reason="Only supported on CUDA/ROCm platforms.",
)
```
**EN:** This block prepares module-level state that later helpers and tests reuse.
**CN:** 该代码块准备了后续辅助函数和测试会复用的模块级状态。

### Function `test_select_default_backend_by_platform` (lines 20-84)
```python
@pytest.mark.parametrize(
    "platform_method,expected_backend",
    [
        ("is_cuda", UnquantizedMoeBackend.TRITON),  # Default CUDA without FlashInfer
        ("is_rocm", UnquantizedMoeBackend.TRITON),  # ROCm without AITER
        ("is_cpu", UnquantizedMoeBackend.CPU),
        ("is_xpu", UnquantizedMoeBackend.XPU),
        ("is_tpu", UnquantizedMoeBackend.TPU),
        ("is_out_of_tree", UnquantizedMoeBackend.OOT),
    ],
)
@patch(
    "vllm.utils.flashinfer.has_flashinfer",
    return_value=False,
)
@patch(
    "vllm.model_executor.layers.fused_moe.oracle.unquantized.rocm_aiter_ops.is_fused_moe_enabled",
    return_value=False,
)
def test_select_default_backend_by_platform(
    mock_aiter_enabled,
    mock_has_flashinfer,
    monkeypatch,
    platform_method,
    expected_backend,
):
    """Test default backend selection per platform with all optional
    accelerators (FlashInfer, AITER) disabled."""
    with patch(
        "vllm.model_executor.layers.fused_moe.oracle.unquantized.current_platform"
    ) as mock_platform:
        # Set all platform checks to False
        mock_platform.is_cuda.return_value = False
        mock_platform.is_rocm.return_value = False
        mock_platform.is_cpu.return_value = False
        mock_platform.is_xpu.return_value = False
        mock_platform.is_tpu.return_value = False
        mock_platform.is_out_of_tree.return_value = False

        # Set only the specified platform to True
        getattr(mock_platform, platform_method).return_value = True

    with (
        patch.object(current_platform, "is_cuda", return_value=False),
        patch.object(current_platform, "is_rocm", return_value=False),
        patch.object(current_platform, "is_cpu", return_value=False),
        patch.object(current_platform, "is_xpu", return_value=False),
        patch.object(current_platform, "is_tpu", return_value=False),
        patch.object(current_platform, "is_out_of_tree", return_value=False),
        patch.object(current_platform, platform_method, return_value=True),
    ):
        moe_config = make_dummy_moe_config()
        selected_backend, expert_cls = select_unquantized_moe_backend(
            moe_config=moe_config
        )

        assert selected_backend == expected_backend
        if expected_backend in [
            UnquantizedMoeBackend.CPU,
            UnquantizedMoeBackend.OOT,
            UnquantizedMoeBackend.TPU,
        ]:
            assert expert_cls is None
        else:
            assert expert_cls is not None
```
**EN:** This pytest case verifies select default backend by platform. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as mock_aiter_enabled, mock_has_flashinfer, monkeypatch, platform_method. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 select default backend by platform 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 mock_aiter_enabled、mock_has_flashinfer、monkeypatch、platform_method 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Function `test_select_rocm_aiter_backend` (lines 87-116)
```python
@patch(
    "vllm.utils.flashinfer.has_flashinfer",
    return_value=False,
)
@patch(
    "vllm.model_executor.layers.fused_moe.oracle.unquantized.rocm_aiter_ops.is_fused_moe_enabled",
    return_value=True,
)
@pytest.mark.skipif(
    not current_platform.is_rocm(), reason="ROCm-specific backend selection test"
)
def test_select_rocm_aiter_backend(mock_aiter_enabled, mock_has_flashinfer):
    """Test ROCm backend selection when AITER is available."""
    with patch(
        "vllm.model_executor.layers.fused_moe.oracle.unquantized.current_platform"
    ) as mock_platform:
        mock_platform.is_cuda.return_value = False
        mock_platform.is_rocm.return_value = True
        mock_platform.is_cpu.return_value = False
        mock_platform.is_xpu.return_value = False
        mock_platform.is_tpu.return_value = False
        mock_platform.is_out_of_tree.return_value = False

        moe_config = make_dummy_moe_config()
        selected_backend, expert_cls = select_unquantized_moe_backend(
            moe_config=moe_config,
        )

        assert selected_backend == UnquantizedMoeBackend.AITER
        assert expert_cls is not None
```
**EN:** This pytest case verifies select ROCm aiter backend. it consumes fixtures or inputs such as mock_aiter_enabled, mock_has_flashinfer. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 select ROCm aiter backend 的行为。 它会使用诸如 mock_aiter_enabled、mock_has_flashinfer 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Function `test_select_cuda_flashinfer_trtllm_backend` (lines 119-149)
```python
@patch(
    "vllm.model_executor.layers.fused_moe.experts.trtllm_bf16_moe.TrtLlmBf16Experts.is_supported_config",
    return_value=(True, None),
)
@pytest.mark.skipif(
    not current_platform.is_cuda(), reason="Only supported on NVIDIA platforms."
)
def test_select_cuda_flashinfer_trtllm_backend(mock_is_supported_trtllm, monkeypatch):
    """Test CUDA backend selection when FlashInfer TRTLLM is available and enabled."""
    with (
        patch.object(current_platform, "is_cuda", return_value=True),
        patch.object(current_platform, "is_rocm", return_value=False),
        patch.object(current_platform, "is_cpu", return_value=False),
        patch.object(current_platform, "is_xpu", return_value=False),
        patch.object(current_platform, "is_tpu", return_value=False),
        patch.object(current_platform, "is_out_of_tree", return_value=False),
        patch.object(current_platform, "has_device_capability", return_value=True),
    ):
        monkeypatch.setenv("VLLM_USE_FLASHINFER_MOE_FP16", "1")

        moe_config = make_dummy_moe_config()
        # TRTLLM requires EP and does not support DP
        moe_config.moe_parallel_config.use_ep = True
        moe_config.moe_parallel_config.use_dp = False

        selected_backend, experts_cls = select_unquantized_moe_backend(
            moe_config=moe_config
        )

        assert selected_backend == UnquantizedMoeBackend.FLASHINFER_TRTLLM
        assert experts_cls is not None
```
**EN:** This pytest case verifies select CUDA flashinfer trtllm backend. it consumes fixtures or inputs such as mock_is_supported_trtllm, monkeypatch. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 select CUDA flashinfer trtllm backend 的行为。 它会使用诸如 mock_is_supported_trtllm、monkeypatch 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Function `test_select_cuda_flashinfer_cutlass_backend` (lines 152-197)
```python
@patch(
    "vllm.utils.flashinfer.has_flashinfer",
    return_value=True,
)
@patch(
    "vllm.model_executor.layers.fused_moe.experts.trtllm_bf16_moe.TrtLlmBf16Experts.is_supported_config",
    return_value=(False, None),
)
@patch(
    "vllm.model_executor.layers.fused_moe.experts.flashinfer_cutlass_moe.FlashInferExperts.is_supported_config",
    return_value=(True, None),
)
@pytest.mark.skipif(
    not current_platform.is_cuda(), reason="Only supported on NVIDIA platforms."
)
def test_select_cuda_flashinfer_cutlass_backend(
    mock_has_flashinfer,
    mock_is_supported_trtllm,
    mock_is_supported_cutlass,
    monkeypatch,
):
    """Test CUDA backend selection when FlashInfer TRTLLM is not available
    and FlashInfer CUTLASS is available."""
    with (
        patch.object(current_platform, "is_cuda", return_value=True),
        patch.object(current_platform, "is_rocm", return_value=False),
        patch.object(current_platform, "is_cpu", return_value=False),
        patch.object(current_platform, "is_xpu", return_value=False),
        patch.object(current_platform, "is_tpu", return_value=False),
        patch.object(current_platform, "is_out_of_tree", return_value=False),
        patch.object(current_platform, "has_device_capability", return_value=True),
    ):
        # Enable FlashInfer via env var
        monkeypatch.setenv("VLLM_USE_FLASHINFER_MOE_FP16", "1")

        moe_config = make_dummy_moe_config()
        # CUTLASS requires EP and does not support DP
        moe_config.moe_parallel_config.use_ep = True
        moe_config.moe_parallel_config.use_dp = False

        selected_backend, experts_cls = select_unquantized_moe_backend(
            moe_config=moe_config
        )

        assert selected_backend == UnquantizedMoeBackend.FLASHINFER_CUTLASS
        assert experts_cls is not None
```
**EN:** This pytest case verifies select CUDA flashinfer cutlass backend. it consumes fixtures or inputs such as mock_has_flashinfer, mock_is_supported_trtllm, mock_is_supported_cutlass, monkeypatch. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 select CUDA flashinfer cutlass backend 的行为。 它会使用诸如 mock_has_flashinfer、mock_is_supported_trtllm、mock_is_supported_cutlass、monkeypatch 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Function `test_select_lora_backend_prefers_triton` (lines 200-210)
```python
@skipif_not_cuda_rocm
def test_select_lora_backend_prefers_triton():
    """LoRA-enabled unquantized MoE should select Triton backend."""
    moe_config = make_dummy_moe_config()
    moe_config.is_lora_enabled = True
    selected_backend, experts_cls = select_unquantized_moe_backend(
        moe_config=moe_config
    )

    assert selected_backend == UnquantizedMoeBackend.TRITON
    assert experts_cls is not None
```
**EN:** This pytest case verifies select lora backend prefers triton. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 select lora backend prefers triton 的行为。 不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Function `test_select_lora_explicit_non_triton_backend` (lines 213-227)
```python
@skipif_not_cuda_rocm
def test_select_lora_explicit_non_triton_backend():
    """LoRA should override explicit non-Triton backend to Triton."""
    moe_config = make_dummy_moe_config()
    moe_config.is_lora_enabled = True

    # Use string from mapping in function map_unquantized_backend()
    moe_config.moe_backend = "flashinfer_cutlass"

    selected_backend, experts_cls = select_unquantized_moe_backend(
        moe_config=moe_config
    )

    assert selected_backend == UnquantizedMoeBackend.TRITON
    assert experts_cls is not None
```
**EN:** This pytest case verifies select lora explicit non triton backend. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 select lora explicit non triton backend 的行为。 不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Function `test_select_explicit_triton_backend` (lines 230-243)
```python
@skipif_not_cuda_rocm
@pytest.mark.parametrize("is_lora_enabled", [False, True])
def test_select_explicit_triton_backend(is_lora_enabled):
    """Explicit triton backend selection should return Triton."""
    moe_config = make_dummy_moe_config()
    moe_config.is_lora_enabled = is_lora_enabled
    moe_config.moe_backend = "triton"

    selected_backend, experts_cls = select_unquantized_moe_backend(
        moe_config=moe_config
    )

    assert selected_backend == UnquantizedMoeBackend.TRITON
    assert experts_cls is not None
```
**EN:** This pytest case verifies select explicit triton backend. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as is_lora_enabled. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 select explicit triton backend 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 is_lora_enabled 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Function `test_select_explicit_triton_ignores_flashinfer_env` (lines 246-261)
```python
@skipif_not_cuda_rocm
def test_select_explicit_triton_ignores_flashinfer_env(monkeypatch):
    """Explicit triton backend should override FlashInfer env selection."""
    monkeypatch.setenv("VLLM_USE_FLASHINFER_MOE_FP16", "1")
    monkeypatch.setenv("VLLM_FLASHINFER_MOE_BACKEND", "throughput")

    moe_config = make_dummy_moe_config()
    moe_config.is_lora_enabled = False
    moe_config.moe_backend = "triton"

    selected_backend, experts_cls = select_unquantized_moe_backend(
        moe_config=moe_config
    )

    assert selected_backend == UnquantizedMoeBackend.TRITON
    assert experts_cls is not None
```
**EN:** This pytest case verifies select explicit triton ignores flashinfer env. it consumes fixtures or inputs such as monkeypatch. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 select explicit triton ignores flashinfer env 的行为。 它会使用诸如 monkeypatch 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Function `test_select_lora_ignores_flashinfer_env` (lines 264-277)
```python
@skipif_not_cuda_rocm
def test_select_lora_ignores_flashinfer_env(monkeypatch):
    """LoRA path should still choose Triton even if FlashInfer env is on."""
    monkeypatch.setenv("VLLM_USE_FLASHINFER_MOE_FP16", "1")
    monkeypatch.setenv("VLLM_FLASHINFER_MOE_BACKEND", "throughput")

    moe_config = make_dummy_moe_config()
    moe_config.is_lora_enabled = True
    selected_backend, experts_cls = select_unquantized_moe_backend(
        moe_config=moe_config
    )

    assert selected_backend == UnquantizedMoeBackend.TRITON
    assert experts_cls is not None
```
**EN:** This pytest case verifies select lora ignores flashinfer env. it consumes fixtures or inputs such as monkeypatch. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 select lora ignores flashinfer env 的行为。 它会使用诸如 monkeypatch 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `unittest.mock -> patch`
- `pytest`
- `tests.kernels.moe.utils -> make_dummy_moe_config`
- `vllm.model_executor.layers.fused_moe.oracle.unquantized -> UnquantizedMoeBackend, select_unquantized_moe_backend`
- `vllm.platforms -> current_platform`
