# test_rocm_attention_selector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/attention/test_rocm_attention_selector.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / attention / test_rocm_attention_selector, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / attention / test_rocm_attention_selector 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-10)
```python
import pytest
import torch

from vllm.config import AttentionConfig, VllmConfig, set_current_vllm_config
from vllm.platforms.rocm import RocmPlatform
from vllm.v1.attention.backends.registry import AttentionBackendEnum
from vllm.v1.attention.selector import _cached_get_attn_backend, get_attn_backend
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; and vLLM components like vllm.config, vllm.platforms.rocm, vllm.v1.attention.backends.registry, vllm.v1.attention.selector.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；vLLM 内部组件，例如 vllm.config、vllm.platforms.rocm、vllm.v1.attention.backends.registry、vllm.v1.attention.selector。

### Function `clear_cache` (lines 13-16)
```python
@pytest.fixture(autouse=True)
def clear_cache():
    """Clear lru cache to ensure each test case runs without caching."""
    _cached_get_attn_backend.cache_clear()
```
**EN:** This fixture prepares reusable state for clear cache. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该 fixture 为 clear cache 准备可复用的测试状态。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_selector` (lines 19-73)
```python
@pytest.mark.skip(reason="Skipped for now. Should be revisited.")
def test_selector(monkeypatch: pytest.MonkeyPatch):
    # Set the current platform to ROCm using monkeypatch
    monkeypatch.setattr("vllm.v1.attention.selector.current_platform", RocmPlatform())

    # Test standard ROCm attention
    attention_config = AttentionConfig(backend=AttentionBackendEnum.ROCM_ATTN)
    vllm_config = VllmConfig(attention_config=attention_config)

    with set_current_vllm_config(vllm_config):
        backend = get_attn_backend(16, torch.float16, torch.float16, 16, False)
        assert backend.get_name() == "ROCM_FLASH" or backend.get_name() == "TRITON_ATTN"

    # MLA test for deepseek related
    # Change the attention backend to triton MLA
    attention_config = AttentionConfig(backend=AttentionBackendEnum.TRITON_MLA)
    vllm_config = VllmConfig(attention_config=attention_config)

    with set_current_vllm_config(vllm_config):
        backend = get_attn_backend(576, torch.bfloat16, "auto", 16, False, use_mla=True)
        assert backend.get_name() == "TRITON_MLA"

    # If attention backend is None
    # If use_mla is true
    # The selected backend is triton MLA
    attention_config = AttentionConfig(backend=None)
    vllm_config = VllmConfig(attention_config=attention_config)

    with set_current_vllm_config(vllm_config):
        backend = get_attn_backend(576, torch.bfloat16, "auto", 16, False, use_mla=True)
        assert backend.get_name() == "TRITON_MLA"

    # Change the attention backend to AITER MLA
    attention_config = AttentionConfig(backend=AttentionBackendEnum.ROCM_AITER_MLA)
    vllm_config = VllmConfig(attention_config=attention_config)

    with set_current_vllm_config(vllm_config):
        backend = get_attn_backend(576, torch.bfloat16, "auto", 1, False, use_mla=True)
        assert backend.get_name() == "ROCM_AITER_MLA"

    # If attention backend is None
    # If use_mla is true
    # If VLLM_ROCM_USE_AITER is enabled
    # The selected backend is ROCM_AITER_MLA
    with monkeypatch.context() as m:
        m.setenv("VLLM_ROCM_USE_AITER", "1")

        attention_config = AttentionConfig(backend=None)
        vllm_config = VllmConfig(attention_config=attention_config)

        with set_current_vllm_config(vllm_config):
            backend = get_attn_backend(
                576, torch.bfloat16, "auto", 1, False, use_mla=True
            )
            assert backend.get_name() == "ROCM_AITER_MLA"
```
**EN:** This pytest case verifies selector. it consumes fixtures or inputs such as monkeypatch. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 selector 的行为。 它会使用诸如 monkeypatch 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

## Key Concepts / 关键概念
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。
- **Reusable fixtures / 可复用 fixture:** Fixtures package common setup so each test can focus on the scenario under study. / fixture 封装公共准备逻辑，让每个测试更专注于待验证场景。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm.config -> AttentionConfig, VllmConfig, set_current_vllm_config`
- `vllm.platforms.rocm -> RocmPlatform`
- `vllm.v1.attention.backends.registry -> AttentionBackendEnum`
- `vllm.v1.attention.selector -> _cached_get_attn_backend, get_attn_backend`
