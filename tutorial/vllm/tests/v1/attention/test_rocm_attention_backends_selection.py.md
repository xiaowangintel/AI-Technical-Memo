# test_rocm_attention_backends_selection.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/attention/test_rocm_attention_backends_selection.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Tests for attention backend selectors. / 该文件的文档字符串表明其用途：`tests for 注意力 后端 selectors`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (line 3)
```python
"""Tests for attention backend selectors."""
```
**EN:** Module docstring that declares the scope of the file: Tests for attention backend selectors.
**CN:** 模块文档字符串直接说明了文件范围：`tests for 注意力 后端 selectors`。

### Imports and setup / 导入与设置 (lines 5-12)
```python
from unittest.mock import MagicMock, patch

import pytest
import torch

from vllm.platforms import current_platform
from vllm.v1.attention.backends.registry import AttentionBackendEnum
from vllm.v1.attention.selector import AttentionSelectorConfig
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm.platforms, vllm.v1.attention.backends.registry, vllm.v1.attention.selector, vllm.envs, vllm.platforms.rocm`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm.platforms, vllm.v1.attention.backends.registry, vllm.v1.attention.selector, vllm.envs, vllm.platforms.rocm`。

### Module state / 模块级状态 (lines 15-17)
```python
pytestmark = pytest.mark.skipif(
    not current_platform.is_rocm(), reason="ROCm-specific tests"
)
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `pytestmark`. It also sets pytest marks that scope the whole file. Shared setup calls include `mark.skipif, current_platform.is_rocm`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`pytestmark`。 该块还设置了作用于整个文件的 pytest 标记。 共享初始化调用包括 `mark.skipif, current_platform.is_rocm`。

### mock_vllm_config (lines 21-27)
```python
def mock_vllm_config():
    """Create a mock VllmConfig for testing."""
    config = MagicMock()
    config.model_config.dtype = torch.float16
    config.model_config.hf_config.architectures = ["LlamaForCausalLM"]
    config.cache_config.block_size = 16
    return config
```
**EN:** Fixture/helper `mock_vllm_config` prepares reusable state for downstream tests. Key calls include `MagicMock`.
**CN:** `mock_vllm_config` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `MagicMock`。

### mock_on_gfx9 (lines 31-34)
```python
def mock_on_gfx9():
    """Mock gfx9 arch detection to return True."""
    with patch("vllm.platforms.rocm.on_gfx9", return_value=True):
        yield
```
**EN:** Fixture/helper `mock_on_gfx9` prepares reusable state for downstream tests. Key calls include `patch`.
**CN:** `mock_on_gfx9` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `patch`。

### mock_on_mi3xx (lines 38-41)
```python
def mock_on_mi3xx():
    """Mock mi3xx arch detection to return True."""
    with patch("vllm.platforms.rocm.on_mi3xx", return_value=True):
        yield
```
**EN:** Fixture/helper `mock_on_mi3xx` prepares reusable state for downstream tests. Key calls include `patch`.
**CN:** `mock_on_mi3xx` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `patch`。

### test_standard_attention_backend_selection (lines 109-153)
```python
def test_standard_attention_backend_selection(
    env_vars,
    selected_backend,
    expected_backend_path,
    mock_vllm_config,
    mock_on_gfx9,
    mock_on_mi3xx,
    monkeypatch,
):
    """Test standard attention backend selection with various configurations."""
    # Set environment variables
    for key, value in env_vars.items():
        monkeypatch.setenv(key, value)

    # Import after setting env vars to ensure they're picked up
    # Reload envs to pick up new environment variables
    import importlib
    # ... excerpt omitted for brevity ...
    backend_path = RocmPlatform.get_attn_backend_cls(
        selected_backend=backend_enum, attn_selector_config=attn_selector_config
    )
    assert backend_path == expected_backend_path
```
**EN:** Parameterized test covering `standard attention backend selection`. Parameter axes: `env_vars, selected_backend, expected_backend_path`. Inputs/fixtures: `env_vars, selected_backend, expected_backend_path, mock_vllm_config, mock_on_gfx9, mock_on_mi3xx, monkeypatch`. It exercises `mark.parametrize, env_vars.items, importlib.reload, AttentionSelectorConfig, RocmPlatform.get_attn_backend_cls, monkeypatch.setenv`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `standard 注意力 后端 selection` 的测试用例。 参数维度：`env_vars, selected_backend, expected_backend_path`。 输入或 fixture：`env_vars, selected_backend, expected_backend_path, mock_vllm_config, mock_on_gfx9, mock_on_mi3xx, monkeypatch`。 该测试会调用 `mark.parametrize, env_vars.items, importlib.reload, AttentionSelectorConfig, RocmPlatform.get_attn_backend_cls, monkeypatch.setenv`。 代码主体包含 1 个显式断言。

### test_mla_backend_selection (lines 226-304)
```python
def test_mla_backend_selection(
    env_vars,
    selected_backend,
    block_size,
    expected_backend_path,
    should_raise,
    mock_vllm_config,
    monkeypatch,
):
    """Test MLA backend selection with various configurations."""
    # Set environment variables
    for key, value in env_vars.items():
        monkeypatch.setenv(key, value)

    # Import after setting env vars
    # Reload envs
    import importlib
    # ... excerpt omitted for brevity ...
    mock_rocm_ops = MagicMock()
    mock_aiter_module = MagicMock()
            with pytest.raises(ValueError):
            backend_path = RocmPlatform.get_attn_backend_cls(
                selected_backend=backend_enum, attn_selector_config=attn_selector_config
            )
            assert backend_path == expected_backend_path
```
**EN:** Parameterized test covering `MLA backend selection`. Parameter axes: `env_vars, selected_backend, block_size, expected_backend_path, should_raise`. Inputs/fixtures: `env_vars, selected_backend, block_size, expected_backend_path, should_raise, mock_vllm_config, monkeypatch`. It exercises `mark.parametrize, env_vars.items, importlib.reload, MagicMock, monkeypatch.setenv, env_vars.get`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `mla 后端 selection` 的测试用例。 参数维度：`env_vars, selected_backend, block_size, expected_backend_path, should_raise`。 输入或 fixture：`env_vars, selected_backend, block_size, expected_backend_path, should_raise, mock_vllm_config, monkeypatch`。 该测试会调用 `mark.parametrize, env_vars.items, importlib.reload, MagicMock, monkeypatch.setenv, env_vars.get`。 代码主体包含 1 个显式断言。

### test_aiter_fa_requires_mi3xx (lines 307-332)
```python
def test_aiter_fa_requires_mi3xx(mock_vllm_config):
    """Test that ROCM_AITER_FA requires mi3xx architecture."""
    from vllm.platforms.rocm import RocmPlatform

    # Mock on_mi3xx to return False (used by supports_compute_capability)
    with (
        patch("vllm.platforms.rocm.on_mi3xx", return_value=False),
        pytest.raises(
            ValueError,
            match="compute capability not supported",
        ),
    ):
        attn_selector_config = AttentionSelectorConfig(
            head_size=128,
            dtype=torch.float16,
            kv_cache_dtype="auto",
            block_size=16,
            use_mla=False,
            has_sink=False,
            use_sparse=False,
        )

        RocmPlatform.get_attn_backend_cls(
            selected_backend=AttentionBackendEnum.ROCM_AITER_FA,
            attn_selector_config=attn_selector_config,
        )
```
**EN:** Test case covering `aiter fa requires mi3xx`. Inputs/fixtures: `mock_vllm_config`. It exercises `patch, pytest.raises, AttentionSelectorConfig, RocmPlatform.get_attn_backend_cls`. Validation is mainly performed through expected exception checks.
**CN:** 该代码块是覆盖 `aiter fa requires mi3xx` 的测试用例。 输入或 fixture：`mock_vllm_config`。 该测试会调用 `patch, pytest.raises, AttentionSelectorConfig, RocmPlatform.get_attn_backend_cls`。 主要通过预期异常检查来完成验证。

### test_sparse_not_supported (lines 335-355)
```python
def test_sparse_not_supported(mock_vllm_config):
    """Test that sparse MLA without use_mla flag raises an error."""
    from vllm.platforms.rocm import RocmPlatform

    with pytest.raises(
        ValueError,
        match="No valid attention backend found",
    ):
        attn_selector_config = AttentionSelectorConfig(
            head_size=128,
            dtype=torch.float16,
            kv_cache_dtype="auto",
            block_size=16,
            use_mla=False,
            has_sink=False,
            use_sparse=True,
        )

        RocmPlatform.get_attn_backend_cls(
            selected_backend=None, attn_selector_config=attn_selector_config
        )
```
**EN:** Test case covering `sparse not supported`. Inputs/fixtures: `mock_vllm_config`. It exercises `pytest.raises, AttentionSelectorConfig, RocmPlatform.get_attn_backend_cls`. Validation is mainly performed through expected exception checks.
**CN:** 该代码块是覆盖 `sparse not supported` 的测试用例。 输入或 fixture：`mock_vllm_config`。 该测试会调用 `pytest.raises, AttentionSelectorConfig, RocmPlatform.get_attn_backend_cls`。 主要通过预期异常检查来完成验证。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Attention backend and metadata coverage
- **CN:** 注意力后端与元数据覆盖

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm.platforms, vllm.v1.attention.backends.registry, vllm.v1.attention.selector, vllm.envs, vllm.platforms.rocm`.
- **CN:** 被测试的 vLLM 模块：`vllm.platforms, vllm.v1.attention.backends.registry, vllm.v1.attention.selector, vllm.envs, vllm.platforms.rocm`。
- **EN:** Standard-library support: `unittest.mock, importlib`.
- **CN:** 标准库支持：`unittest.mock, importlib`。
