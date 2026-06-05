# test_register.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/helion/test_register.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / helion / test_register, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / helion / test_register 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-8)
```python
"""
Unit tests for Helion kernel registration.

Tests ConfiguredHelionKernel, HelionKernelWrapper, and PresetConfigSearch
including config picker registration and custom autotuner integration.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 10-15)
```python
from unittest.mock import Mock, patch

import pytest
import torch

from vllm.utils.import_utils import has_helion
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as unittest.mock, pytest, torch; and vLLM components like vllm.utils.import_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 unittest.mock、pytest、torch；vLLM 内部组件，例如 vllm.utils.import_utils。

### Top-level block starting at line 17 (lines 17-21)
```python
if not has_helion():
    pytest.skip(
        "Helion is not installed. Install with: pip install vllm[helion]",
        allow_module_level=True,
    )
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Imports and shared setup (lines 23-37)
```python
import helion
import helion.language as hl

from tests.kernels.helion.helpers import dummy_kernel_registry
from vllm.kernels.helion.case_key import CaseKey
from vllm.kernels.helion.config_manager import ConfigManager
from vllm.kernels.helion.register import (
    _HOP_AVAILABLE,
    ConfiguredHelionKernel,
    HelionKernelWrapper,
    get_kernel_by_name,
    get_registered_kernels,
    register_kernel,
    validate_helion_settings,
)
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as helion, helion.language; shared test helpers from tests.kernels.helion.helpers; and vLLM components like vllm.kernels.helion.case_key, vllm.kernels.helion.config_manager, vllm.kernels.helion.register.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 helion、helion.language；共享测试辅助模块，例如 tests.kernels.helion.helpers；vLLM 内部组件，例如 vllm.kernels.helion.case_key、vllm.kernels.helion.config_manager、vllm.kernels.helion.register。

### Top-level block starting at line 39 (lines 39-44)
```python
if _HOP_AVAILABLE:
    from helion._compat import supports_torch_compile_fusion
    from helion._compiler._dynamo.higher_order_ops import (
        helion_kernel_wrapper_mutation,
    )
    from torch._inductor.utils import run_and_get_code
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Function `_add_kernel` (lines 47-51)
```python
def _add_kernel(x: torch.Tensor, y: torch.Tensor) -> torch.Tensor:
    out = torch.empty_like(x)
    for tile in hl.tile(x.size()):
        out[tile] = x[tile] + y[tile]
    return out
```
**EN:** This helper function implements the shared logic for add kernel. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 add kernel 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `sample_configs` (lines 54-78)
```python
@pytest.fixture
def sample_configs():
    """Create real Helion config objects for testing."""
    return {
        CaseKey({"batchsize": 32, "hiddensize": 4096}): helion.Config(
            block_sizes=[128],
            num_warps=4,
            num_stages=3,
        ),
        CaseKey({"batchsize": 64, "hiddensize": 4096}): helion.Config(
            block_sizes=[256],
            num_warps=8,
            num_stages=4,
        ),
        CaseKey({"batchsize": 128, "hiddensize": 4096}): helion.Config(
            block_sizes=[512],
            num_warps=16,
            num_stages=2,
        ),
        CaseKey.default(): helion.Config(
            block_sizes=[64],
            num_warps=2,
            num_stages=2,
        ),
    }
```
**EN:** This fixture prepares reusable state for sample configs. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该 fixture 为 sample configs 准备可复用的测试状态。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `sample_kernel` (lines 81-89)
```python
@pytest.fixture
def sample_kernel():
    """Create a simple test kernel function."""

    def test_kernel(x: torch.Tensor, y: torch.Tensor) -> torch.Tensor:
        """Simple test kernel that adds two tensors."""
        return x + y

    return test_kernel
```
**EN:** This fixture prepares reusable state for sample kernel. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该 fixture 为 sample kernel 准备可复用的测试状态。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `config_manager_with_test_configs` (lines 92-97)
```python
@pytest.fixture
def config_manager_with_test_configs(sample_configs):
    """Set up ConfigManager with test configs for nvidia_h200 platform."""
    mock_config_manager = Mock(spec=ConfigManager)
    mock_config_manager.get_platform_configs = Mock(return_value=sample_configs)
    return mock_config_manager
```
**EN:** This fixture prepares reusable state for config manager with test configs. it consumes fixtures or inputs such as sample_configs.
**CN:** 该 fixture 为 config manager with test configs 准备可复用的测试状态。 它会使用诸如 sample_configs 等 fixture 或输入。

### Function `configured_kernel` (lines 100-126)
```python
@pytest.fixture
def configured_kernel(sample_kernel, sample_configs, config_manager_with_test_configs):
    """Create a ConfiguredHelionKernel for testing."""

    def test_config_picker(args, config_keys):
        return None

    with (
        patch(
            "vllm.kernels.helion.config_manager.ConfigManager",
            return_value=config_manager_with_test_configs,
        ),
        patch(
            "vllm.kernels.helion.utils.get_canonical_gpu_name",
            return_value="nvidia_h200",
        ),
        patch("vllm.kernels.helion.register.helion.kernel") as mock_kernel,
    ):
        mock_decorated = Mock()
        mock_kernel.return_value = Mock(return_value=mock_decorated)

        return ConfiguredHelionKernel(
            op_name="test_kernel",
            config_picker=test_config_picker,
            raw_kernel_func=sample_kernel,
            helion_settings=None,
        )
```
**EN:** This fixture prepares reusable state for configured kernel. it consumes fixtures or inputs such as sample_kernel, sample_configs, config_manager_with_test_configs.
**CN:** 该 fixture 为 configured kernel 准备可复用的测试状态。 它会使用诸如 sample_kernel、sample_configs、config_manager_with_test_configs 等 fixture 或输入。

### Class `TestValidateHelionSettings` (lines 129-131)
```python
class TestValidateHelionSettings:
    """Test suite for validate_helion_settings utility function."""
```
**EN:** This helper class groups the state and behavior needed for TestValidateHelionSettings. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestValidateHelionSettings 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestValidateHelionSettings.test_accepts_none_settings` (lines 132-134)
```python
    def test_accepts_none_settings(self):
        """Test that None settings are accepted without error."""
        validate_helion_settings(None, "test_kernel")  # Should not raise
```
**EN:** This method on `TestValidateHelionSettings` checks accepts none settings. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestValidateHelionSettings` 中的这个方法用于检查 accepts none settings。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestValidateHelionSettings.test_accepts_valid_settings` (lines 136-141)
```python
    def test_accepts_valid_settings(self):
        """Test that valid settings without conflicts are accepted."""
        settings = helion.Settings()
        settings.static_shapes = False
        settings.print_output_code = True
        validate_helion_settings(settings, "test_kernel")  # Should not raise
```
**EN:** This method on `TestValidateHelionSettings` checks accepts valid settings. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestValidateHelionSettings` 中的这个方法用于检查 accepts valid settings。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestValidateHelionSettings.test_rejects_autotuner_fn` (lines 143-149)
```python
    def test_rejects_autotuner_fn(self):
        """Test that settings with custom autotuner_fn raise ValueError."""
        settings = helion.Settings()
        settings.autotuner_fn = lambda *args: None  # Set custom autotuner function

        with pytest.raises(ValueError, match="uses a custom autotuner"):
            validate_helion_settings(settings, "test_kernel")
```
**EN:** This method on `TestValidateHelionSettings` checks rejects autotuner fn. the expected failure path is asserted explicitly.
**CN:** `TestValidateHelionSettings` 中的这个方法用于检查 rejects autotuner fn。 代码会显式断言预期的失败路径。

### Method `TestValidateHelionSettings.test_warns_on_static_shapes_true` (lines 151-159)
```python
    def test_warns_on_static_shapes_true(self):
        """Test that static_shapes=True emits a warning about being overridden."""
        settings = helion.Settings()
        settings.static_shapes = True

        with patch("vllm.kernels.helion.register.logger") as mock_logger:
            validate_helion_settings(settings, "test_kernel")
            mock_logger.warning.assert_called_once()
            assert "overridden to False" in mock_logger.warning.call_args[0][0]
```
**EN:** This method on `TestValidateHelionSettings` checks warns on static shapes true. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestValidateHelionSettings` 中的这个方法用于检查 warns on static shapes true。 结尾处的断言会固定预期行为或计算图形态。

### Function `create_configured_kernel_with_configs` (lines 162-193)
```python
def create_configured_kernel_with_configs(
    op_name,
    config_picker,
    kernel_func,
    configs,
    platform="nvidia_h200",
    helion_settings=None,
):
    """Helper to create ConfiguredHelionKernel with real config objects."""
    mock_config_manager = Mock(spec=ConfigManager)
    mock_config_manager.get_platform_configs = Mock(return_value=configs)

    with (
        patch(
            "vllm.kernels.helion.config_manager.ConfigManager",
            return_value=mock_config_manager,
        ),
        patch(
            "vllm.kernels.helion.utils.get_canonical_gpu_name",
            return_value=platform,
        ),
        patch("vllm.kernels.helion.register.helion.kernel") as mock_kernel,
    ):
        mock_decorated = Mock()
        mock_kernel.return_value = Mock(return_value=mock_decorated)

        return ConfiguredHelionKernel(
            op_name=op_name,
            config_picker=config_picker,
            raw_kernel_func=kernel_func,
            helion_settings=helion_settings,
        )
```
**EN:** This helper function implements the shared logic for configured kernel with configs. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 configured kernel with configs 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestConfiguredHelionKernel` (lines 196-198)
```python
class TestConfiguredHelionKernel:
    """Test suite for ConfiguredHelionKernel."""
```
**EN:** This helper class groups the state and behavior needed for TestConfiguredHelionKernel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestConfiguredHelionKernel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestConfiguredHelionKernel.test_init_raises_without_picker` (lines 199-223)
```python
    def test_init_raises_without_picker(self, sample_kernel, sample_configs):
        """Test that __init__ raises when no picker registered."""
        configs: dict[CaseKey, helion.Config] = {
            CaseKey.default(): sample_configs[CaseKey.default()]
        }
        mock_config_manager = Mock(spec=ConfigManager)
        mock_config_manager.get_platform_configs = Mock(return_value=configs)

        with (
            patch(
                "vllm.kernels.helion.config_manager.ConfigManager",
                return_value=mock_config_manager,
            ),
            patch(
                "vllm.kernels.helion.utils.get_canonical_gpu_name",
                return_value="nvidia_h200",
            ),
            pytest.raises(RuntimeError, match="No config picker registered"),
        ):
            ConfiguredHelionKernel(
                op_name="test_kernel",
                config_picker=None,  # No picker registered
                raw_kernel_func=sample_kernel,
                helion_settings=None,
            )
```
**EN:** This method on `TestConfiguredHelionKernel` checks init raises without picker. it consumes fixtures or inputs such as sample_kernel, sample_configs. the expected failure path is asserted explicitly.
**CN:** `TestConfiguredHelionKernel` 中的这个方法用于检查 init raises without picker。 它会使用诸如 sample_kernel、sample_configs 等 fixture 或输入；代码会显式断言预期的失败路径。

### Method `TestConfiguredHelionKernel.test_config_selector_validates_picker_result` (lines 225-246)
```python
    def test_config_selector_validates_picker_result(
        self, sample_kernel, sample_configs
    ):
        """Test that config selector validates picker returns valid key."""

        def invalid_picker(args, config_keys):
            return {"invalid": 999}

        kernel = create_configured_kernel_with_configs(
            op_name="test_kernel",
            config_picker=invalid_picker,
            kernel_func=sample_kernel,
            configs=sample_configs,
        )

        key_computer = kernel._create_key_computer()
        selector = kernel._create_config_selector(key_computer)

        with pytest.raises(
            ValueError, match="Config picker returned invalid config key"
        ):
            selector((torch.randn(32, 4096),))
```
**EN:** This method on `TestConfiguredHelionKernel` checks config selector validates picker result. it consumes fixtures or inputs such as sample_kernel, sample_configs. the expected failure path is asserted explicitly.
**CN:** `TestConfiguredHelionKernel` 中的这个方法用于检查 config selector validates picker result。 它会使用诸如 sample_kernel、sample_configs 等 fixture 或输入；代码会显式断言预期的失败路径。

### Method `TestConfiguredHelionKernel.test_config_selector_handles_none_from_picker` (lines 248-267)
```python
    def test_config_selector_handles_none_from_picker(
        self, sample_kernel, sample_configs
    ):
        """Test that config selector falls back to 'default' on None."""

        def none_picker(args, config_keys):
            return None

        kernel = create_configured_kernel_with_configs(
            op_name="test_kernel",
            config_picker=none_picker,
            kernel_func=sample_kernel,
            configs=sample_configs,
        )

        key_computer = kernel._create_key_computer()
        selector = kernel._create_config_selector(key_computer)

        result = selector((torch.randn(32, 4096),))
        assert result is kernel.configs[CaseKey.default()]
```
**EN:** This method on `TestConfiguredHelionKernel` checks config selector handles none from picker. it consumes fixtures or inputs such as sample_kernel, sample_configs. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestConfiguredHelionKernel` 中的这个方法用于检查 config selector handles none from picker。 它会使用诸如 sample_kernel、sample_configs 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Method `TestConfiguredHelionKernel.test_create_decorated_kernel_passes_helion_settings` (lines 269-308)
```python
    def test_create_decorated_kernel_passes_helion_settings(
        self, sample_kernel, sample_configs
    ):
        """Test that _create_decorated_kernel passes helion_settings."""

        def default_picker(args, config_keys):
            return None

        settings = helion.Settings()
        settings.print_output_code = True

        mock_config_manager = Mock(spec=ConfigManager)
        mock_config_manager.get_platform_configs = Mock(return_value=sample_configs)

        with (
            patch("vllm.kernels.helion.register.helion.kernel") as mock_kernel,
            patch(
                "vllm.kernels.helion.config_manager.ConfigManager",
                return_value=mock_config_manager,
            ),
            patch(
                "vllm.kernels.helion.utils.get_canonical_gpu_name",
                return_value="nvidia_h200",
            ),
        ):
            mock_decorated = Mock()
            mock_kernel.return_value = Mock(return_value=mock_decorated)

            ConfiguredHelionKernel(
                op_name="test_kernel",
                config_picker=default_picker,
                raw_kernel_func=sample_kernel,
                helion_settings=settings,
            )

            call_kwargs = mock_kernel.call_args[1]
            assert "print_output_code" in call_kwargs
            assert call_kwargs["print_output_code"] is True
            # static_shapes is always forced to False by vLLM
            assert call_kwargs["static_shapes"] is False
```
**EN:** This method on `TestConfiguredHelionKernel` checks create decorated kernel passes helion settings. it consumes fixtures or inputs such as sample_kernel, sample_configs. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestConfiguredHelionKernel` 中的这个方法用于检查 create decorated kernel passes helion settings。 它会使用诸如 sample_kernel、sample_configs 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Method `TestConfiguredHelionKernel.test_key_and_config_selector_use_same_logic` (lines 310-360)
```python
    def test_key_and_config_selector_use_same_logic(
        self, sample_kernel, sample_configs
    ):
        """Test that key and config_selector produce identical results."""

        def tracking_picker(args, config_keys):
            x = args[0]
            batch_size = x.shape[0]
            if batch_size <= 32:
                return CaseKey({"batchsize": 32, "hiddensize": 4096})
            elif batch_size <= 64:
                return CaseKey({"batchsize": 64, "hiddensize": 4096})
            return CaseKey({"batchsize": 128, "hiddensize": 4096})

        mock_config_manager = Mock(spec=ConfigManager)
        mock_config_manager.get_platform_configs = Mock(return_value=sample_configs)

        with (
            patch("vllm.kernels.helion.register.helion.kernel") as mock_helion_kernel,
            patch(
                "vllm.kernels.helion.config_manager.ConfigManager",
                return_value=mock_config_manager,
            ),
            patch(
                "vllm.kernels.helion.utils.get_canonical_gpu_name",
                return_value="nvidia_h200",
            ),
        ):
            mock_decorated = Mock()
            mock_helion_kernel.return_value = Mock(return_value=mock_decorated)

            kernel = ConfiguredHelionKernel(
                op_name="test_kernel",
                config_picker=tracking_picker,
                raw_kernel_func=sample_kernel,
                helion_settings=None,
            )

            call_kwargs = mock_helion_kernel.call_args[1]
            key_fn = call_kwargs["key"]
            autotuner_fn = call_kwargs["autotuner_fn"]

            tensor = torch.randn(50, 4096)  # batch=50, should select batchsize_64

            key_result = key_fn(tensor)
            autotuner = autotuner_fn(None, (tensor,))
            config = autotuner.autotune()

            expected_key = CaseKey({"batchsize": 64, "hiddensize": 4096})
            assert key_result == str(expected_key)
            assert config is kernel.configs[expected_key]
```
**EN:** This method on `TestConfiguredHelionKernel` checks key and config selector use same logic. it consumes fixtures or inputs such as sample_kernel, sample_configs. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestConfiguredHelionKernel` 中的这个方法用于检查 key and config selector use same logic。 它会使用诸如 sample_kernel、sample_configs 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Class `TestHelionKernelWrapper` (lines 363-365)
```python
class TestHelionKernelWrapper:
    """Test suite for HelionKernelWrapper."""
```
**EN:** This helper class groups the state and behavior needed for TestHelionKernelWrapper. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestHelionKernelWrapper 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestHelionKernelWrapper.test_init_disables_on_missing_configs` (lines 366-401)
```python
    def test_init_disables_on_missing_configs(self, sample_kernel):
        """Test __init__ marks wrapper as disabled when configs are missing."""

        def fake_impl(*args, **kwargs):
            return torch.zeros_like(args[0])

        def default_picker(args, config_keys):
            return None

        mock_config_manager = Mock(spec=ConfigManager)
        mock_config_manager.get_platform_configs = Mock(
            return_value={}
        )  # Empty configs

        with (
            patch(
                "vllm.kernels.helion.config_manager.ConfigManager",
                return_value=mock_config_manager,
            ),
            patch(
                "vllm.kernels.helion.utils.get_canonical_gpu_name",
                return_value="nvidia_h200",
            ),
            patch("vllm.kernels.helion.register.helion.kernel") as mock_kernel,
        ):
            mock_kernel.return_value = Mock(return_value=sample_kernel)

            wrapper = HelionKernelWrapper(
                raw_kernel_func=sample_kernel,
                op_name="test_kernel",
                fake_impl=fake_impl,
                config_picker=default_picker,
            )

            assert wrapper._disabled is True
            assert "No configs available" in wrapper._disabled_reason
```
**EN:** This method on `TestHelionKernelWrapper` checks init disables on missing configs. it consumes fixtures or inputs such as sample_kernel. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestHelionKernelWrapper` 中的这个方法用于检查 init disables on missing configs。 它会使用诸如 sample_kernel 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Method `TestHelionKernelWrapper.test_disabled_wrapper_raises_on_call` (lines 403-436)
```python
    def test_disabled_wrapper_raises_on_call(self, sample_kernel):
        """Test __call__ raises RuntimeError on a disabled wrapper."""

        def fake_impl(*args, **kwargs):
            return torch.zeros_like(args[0])

        def default_picker(args, config_keys):
            return None

        mock_config_manager = Mock(spec=ConfigManager)
        mock_config_manager.get_platform_configs = Mock(return_value={})

        with (
            patch(
                "vllm.kernels.helion.config_manager.ConfigManager",
                return_value=mock_config_manager,
            ),
            patch(
                "vllm.kernels.helion.utils.get_canonical_gpu_name",
                return_value="nvidia_h200",
            ),
            patch("vllm.kernels.helion.register.helion.kernel") as mock_kernel,
        ):
            mock_kernel.return_value = Mock(return_value=sample_kernel)

            wrapper = HelionKernelWrapper(
                raw_kernel_func=sample_kernel,
                op_name="test_kernel",
                fake_impl=fake_impl,
                config_picker=default_picker,
            )

        with pytest.raises(RuntimeError, match="is disabled"):
            wrapper(torch.randn(4, 4), torch.randn(4, 4))
```
**EN:** This method on `TestHelionKernelWrapper` checks disabled wrapper raises on call. it consumes fixtures or inputs such as sample_kernel. the expected failure path is asserted explicitly.
**CN:** `TestHelionKernelWrapper` 中的这个方法用于检查 disabled wrapper raises on call。 它会使用诸如 sample_kernel 等 fixture 或输入；代码会显式断言预期的失败路径。

### Method `TestHelionKernelWrapper.test_disabled_wrapper_get_configured_op_raises` (lines 438-471)
```python
    def test_disabled_wrapper_get_configured_op_raises(self, sample_kernel):
        """Test get_configured_op raises RuntimeError on a disabled wrapper."""

        def fake_impl(*args, **kwargs):
            return torch.zeros_like(args[0])

        def default_picker(args, config_keys):
            return None

        mock_config_manager = Mock(spec=ConfigManager)
        mock_config_manager.get_platform_configs = Mock(return_value={})

        with (
            patch(
                "vllm.kernels.helion.config_manager.ConfigManager",
                return_value=mock_config_manager,
            ),
            patch(
                "vllm.kernels.helion.utils.get_canonical_gpu_name",
                return_value="nvidia_h200",
            ),
            patch("vllm.kernels.helion.register.helion.kernel") as mock_kernel,
        ):
            mock_kernel.return_value = Mock(return_value=sample_kernel)

            wrapper = HelionKernelWrapper(
                raw_kernel_func=sample_kernel,
                op_name="test_kernel",
                fake_impl=fake_impl,
                config_picker=default_picker,
            )

        with pytest.raises(RuntimeError, match="is disabled"):
            wrapper.get_configured_op()
```
**EN:** This method on `TestHelionKernelWrapper` checks disabled wrapper get configured op raises. it consumes fixtures or inputs such as sample_kernel. the expected failure path is asserted explicitly.
**CN:** `TestHelionKernelWrapper` 中的这个方法用于检查 disabled wrapper get configured op raises。 它会使用诸如 sample_kernel 等 fixture 或输入；代码会显式断言预期的失败路径。

### Method `TestHelionKernelWrapper.test_disabled_wrapper_supports_get_inputs` (lines 473-511)
```python
    def test_disabled_wrapper_supports_get_inputs(self, sample_kernel):
        """Test get_inputs works on a disabled wrapper."""

        def fake_impl(*args, **kwargs):
            return torch.zeros_like(args[0])

        def default_picker(args, config_keys):
            return None

        expected_inputs = {"key1": (torch.randn(4),)}
        input_gen = Mock(return_value=expected_inputs)

        mock_config_manager = Mock(spec=ConfigManager)
        mock_config_manager.get_platform_configs = Mock(return_value={})

        with (
            patch(
                "vllm.kernels.helion.config_manager.ConfigManager",
                return_value=mock_config_manager,
            ),
            patch(
                "vllm.kernels.helion.utils.get_canonical_gpu_name",
                return_value="nvidia_h200",
            ),
            patch("vllm.kernels.helion.register.helion.kernel") as mock_kernel,
        ):
            mock_kernel.return_value = Mock(return_value=sample_kernel)

            wrapper = HelionKernelWrapper(
                raw_kernel_func=sample_kernel,
                op_name="test_kernel",
                fake_impl=fake_impl,
                config_picker=default_picker,
                input_generator=input_gen,
            )

        assert wrapper._disabled is True
        result = wrapper.get_inputs()
        assert result is expected_inputs
```
**EN:** This method on `TestHelionKernelWrapper` checks disabled wrapper supports get inputs. it consumes fixtures or inputs such as sample_kernel. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestHelionKernelWrapper` 中的这个方法用于检查 disabled wrapper supports get inputs。 它会使用诸如 sample_kernel 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Method `TestHelionKernelWrapper.test_disabled_wrapper_supports_run_autotune` (lines 513-558)
```python
    def test_disabled_wrapper_supports_run_autotune(self, sample_kernel):
        """Test run_autotune works on a disabled wrapper."""

        def fake_impl(*args, **kwargs):
            return torch.zeros_like(args[0])

        def default_picker(args, config_keys):
            return None

        mock_config_manager = Mock(spec=ConfigManager)
        mock_config_manager.get_platform_configs = Mock(return_value={})

        mock_config = Mock()

        with (
            patch(
                "vllm.kernels.helion.config_manager.ConfigManager",
                return_value=mock_config_manager,
            ),
            patch(
                "vllm.kernels.helion.utils.get_canonical_gpu_name",
                return_value="nvidia_h200",
            ),
            patch("vllm.kernels.helion.register.helion.kernel") as mock_kernel,
        ):
            mock_kernel.return_value = Mock(return_value=sample_kernel)

            wrapper = HelionKernelWrapper(
                raw_kernel_func=sample_kernel,
                op_name="test_kernel",
                fake_impl=fake_impl,
                config_picker=default_picker,
            )

        assert wrapper._disabled is True

        with patch(
            "vllm.kernels.helion.register.create_helion_decorated_kernel"
        ) as mock_create:
            mock_autotune_kernel = Mock()
            mock_autotune_kernel.autotune.return_value = mock_config
            mock_create.return_value = mock_autotune_kernel

            inputs = (torch.randn(4, 4),)
            result = wrapper.run_autotune(inputs)
            assert result is mock_config
```
**EN:** This method on `TestHelionKernelWrapper` checks disabled wrapper supports run autotune. it consumes fixtures or inputs such as sample_kernel. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestHelionKernelWrapper` 中的这个方法用于检查 disabled wrapper supports run autotune。 它会使用诸如 sample_kernel 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Method `TestHelionKernelWrapper.test_init_caches_configured_kernel` (lines 560-595)
```python
    def test_init_caches_configured_kernel(self, sample_kernel, sample_configs):
        """Test __init__ eagerly builds and caches ConfiguredHelionKernel."""

        def fake_impl(*args, **kwargs):
            return torch.zeros_like(args[0])

        def default_picker(args, config_keys):
            return None

        mock_config_manager = Mock(spec=ConfigManager)
        mock_config_manager.get_platform_configs = Mock(return_value=sample_configs)

        with (
            patch(
                "vllm.kernels.helion.config_manager.ConfigManager",
                return_value=mock_config_manager,
            ),
            patch(
                "vllm.kernels.helion.utils.get_canonical_gpu_name",
                return_value="nvidia_h200",
            ),
            patch("vllm.kernels.helion.register.helion.kernel") as mock_kernel,
        ):
            mock_kernel.return_value = Mock(return_value=sample_kernel)

            wrapper = HelionKernelWrapper(
                raw_kernel_func=sample_kernel,
                op_name="test_kernel",
                fake_impl=fake_impl,
                config_picker=default_picker,
            )

            assert wrapper._configured_kernel is not None
            result1 = wrapper.get_configured_op()
            result2 = wrapper.get_configured_op()
            assert result1 is result2
```
**EN:** This method on `TestHelionKernelWrapper` checks init caches configured kernel. it consumes fixtures or inputs such as sample_kernel, sample_configs. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestHelionKernelWrapper` 中的这个方法用于检查 init caches configured kernel。 它会使用诸如 sample_kernel、sample_configs 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Method `TestHelionKernelWrapper.test_init_eagerly_initializes_hop_path` (lines 597-630)
```python
    @pytest.mark.skipif(
        not _HOP_AVAILABLE, reason="HOP path only used when HOP available"
    )
    def test_init_eagerly_initializes_hop_path(self):
        """Test that register_kernel eagerly builds the configured kernel
        on the HOP path (no custom op registration needed)."""
        from vllm.kernels.helion.utils import get_canonical_gpu_name

        configs: dict[CaseKey, helion.Config] = {
            CaseKey.default(): helion.Config(block_sizes=[4, 4])
        }
        with (
            dummy_kernel_registry(configs=configs) as register,
            patch(
                "vllm.kernels.helion.utils.get_canonical_gpu_name",
                wraps=get_canonical_gpu_name,
            ) as mock_gpu,
        ):
            wrapper = register(
                config_picker=lambda args, keys: None,
            )(_add_kernel)

            mock_gpu.assert_called_once()
            assert wrapper._configured_kernel is not None

        with patch(
            "vllm.kernels.helion.utils.get_canonical_gpu_name",
            side_effect=AssertionError("get_canonical_gpu_name called during __call__"),
        ):
            x = torch.randn(4, 4, device="cuda")
            y = torch.randn(4, 4, device="cuda")
            result = wrapper(x, y)
            expected = x + y
            assert torch.allclose(result, expected)
```
**EN:** This method on `TestHelionKernelWrapper` checks init eagerly initializes hop path. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestHelionKernelWrapper` 中的这个方法用于检查 init eagerly initializes hop path。 不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Method `TestHelionKernelWrapper.test_init_eagerly_initializes` (lines 632-654)
```python
    @pytest.mark.skipif(
        _HOP_AVAILABLE, reason="CustomOp path not used when HOP available"
    )
    def test_init_eagerly_initializes(self):
        """Test that register_kernel eagerly loads configs and detects GPU
        during construction so __call__ needs no further initialization."""
        from vllm.kernels.helion.utils import get_canonical_gpu_name

        with (
            dummy_kernel_registry() as register,
            patch(
                "vllm.kernels.helion.utils.get_canonical_gpu_name",
                wraps=get_canonical_gpu_name,
            ) as mock_gpu,
        ):
            wrapper = register(
                config_picker=lambda args, keys: None,
            )(_add_kernel)

            # Init must have detected GPU and built the kernel
            mock_gpu.assert_called_once()
            assert wrapper._configured_kernel is not None
            assert hasattr(torch.ops.vllm_helion, wrapper.op_name)
```
**EN:** This method on `TestHelionKernelWrapper` checks init eagerly initializes. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestHelionKernelWrapper` 中的这个方法用于检查 init eagerly initializes。 不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Method `TestHelionKernelWrapper.test_get_or_register_custom_op_returns_cached_op` (lines 656-697)
```python
    @pytest.mark.skipif(
        _HOP_AVAILABLE, reason="CustomOp path not used when HOP available"
    )
    def test_get_or_register_custom_op_returns_cached_op(
        self, sample_kernel, sample_configs
    ):
        def fake_impl(*args, **kwargs):
            return torch.zeros_like(args[0])

        def default_picker(args, config_keys):
            return None

        mock_config_manager = Mock(spec=ConfigManager)
        mock_config_manager.get_platform_configs = Mock(return_value=sample_configs)

        existing_op = Mock()
        mock_namespace = Mock()
        mock_namespace.test_kernel = existing_op

        with (
            patch(
                "vllm.kernels.helion.config_manager.ConfigManager",
                return_value=mock_config_manager,
            ),
            patch(
                "vllm.kernels.helion.utils.get_canonical_gpu_name",
                return_value="nvidia_h200",
            ),
            patch.object(torch.ops, "vllm_helion", mock_namespace),
            patch("vllm.kernels.helion.register.helion.kernel") as mock_kernel,
        ):
            mock_decorated = Mock()
            mock_kernel.return_value = Mock(return_value=mock_decorated)

            wrapper = HelionKernelWrapper(
                raw_kernel_func=sample_kernel,
                op_name="test_kernel",
                fake_impl=fake_impl,
                config_picker=default_picker,
            )
            result = wrapper._get_or_register_custom_op()
            assert result is existing_op
```
**EN:** This method on `TestHelionKernelWrapper` checks get or register custom op returns cached op. it consumes fixtures or inputs such as sample_kernel, sample_configs. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestHelionKernelWrapper` 中的这个方法用于检查 get or register custom op returns cached op。 它会使用诸如 sample_kernel、sample_configs 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Method `TestHelionKernelWrapper.test_get_or_register_custom_op_registers_new_op` (lines 699-757)
```python
    @pytest.mark.skipif(
        _HOP_AVAILABLE, reason="CustomOp path not used when HOP available"
    )
    def test_get_or_register_custom_op_registers_new_op(
        self, sample_kernel, sample_configs
    ):
        def fake_impl(*args, **kwargs):
            return torch.zeros_like(args[0])

        def default_picker(args, config_keys):
            return None

        mock_config_manager = Mock(spec=ConfigManager)
        mock_config_manager.get_platform_configs = Mock(return_value=sample_configs)

        new_op = Mock()
        registered_ops: dict[str, Mock] = {}

        class MockNamespace:
            def __getattr__(self, name):
                if name in registered_ops:
                    return registered_ops[name]
                raise AttributeError(name)

        mock_namespace = MockNamespace()

        def register_side_effect(op_name, op_func, **kwargs):
            registered_ops[op_name] = new_op

        with (
            patch(
                "vllm.kernels.helion.config_manager.ConfigManager",
                return_value=mock_config_manager,
            ),
            patch(
                "vllm.kernels.helion.utils.get_canonical_gpu_name",
                return_value="nvidia_h200",
            ),
            patch.object(torch.ops, "vllm_helion", mock_namespace),
            patch(
                "vllm.kernels.helion.register.direct_register_custom_op",
                side_effect=register_side_effect,
            ) as mock_register,
            patch("vllm.kernels.helion.register.helion.kernel") as mock_kernel,
        ):
            mock_decorated = Mock()
            mock_kernel.return_value = Mock(return_value=mock_decorated)

            wrapper = HelionKernelWrapper(
                raw_kernel_func=sample_kernel,
                op_name="test_kernel",
                fake_impl=fake_impl,
                config_picker=default_picker,
            )
            result = wrapper._get_or_register_custom_op()

            mock_register.assert_called_once()
            assert result is new_op
            assert mock_register.call_args[1]["op_func"] is mock_decorated
```
**EN:** This method on `TestHelionKernelWrapper` checks get or register custom op registers new op. it consumes fixtures or inputs such as sample_kernel, sample_configs. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestHelionKernelWrapper` 中的这个方法用于检查 get or register custom op registers new op。 它会使用诸如 sample_kernel、sample_configs 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Class `TestKernelRegistry` (lines 760-762)
```python
class TestKernelRegistry:
    """Test suite for kernel registry functionality."""
```
**EN:** This helper class groups the state and behavior needed for TestKernelRegistry. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestKernelRegistry 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestKernelRegistry.setup_method` (lines 763-768)
```python
    def setup_method(self):
        """Save and clear the registry before each test."""
        from vllm.kernels.helion.register import _REGISTERED_KERNELS

        self._saved_registry = dict(_REGISTERED_KERNELS)
        _REGISTERED_KERNELS.clear()
```
**EN:** This method on `TestKernelRegistry` implements setup method. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestKernelRegistry` 中的这个方法实现了 setup method。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestKernelRegistry.teardown_method` (lines 770-775)
```python
    def teardown_method(self):
        """Restore the registry after each test."""
        from vllm.kernels.helion.register import _REGISTERED_KERNELS

        _REGISTERED_KERNELS.clear()
        _REGISTERED_KERNELS.update(self._saved_registry)
```
**EN:** This method on `TestKernelRegistry` implements teardown method. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestKernelRegistry` 中的这个方法实现了 teardown method。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestKernelRegistry.test_get_registered_kernels_returns_copy` (lines 777-785)
```python
    def test_get_registered_kernels_returns_copy(self):
        """Test get_registered_kernels returns copy of registry."""
        result1 = get_registered_kernels()
        result2 = get_registered_kernels()

        # Should be separate objects
        assert result1 is not result2
        # Should have same content
        assert result1 == result2
```
**EN:** This method on `TestKernelRegistry` checks get registered kernels returns copy. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestKernelRegistry` 中的这个方法用于检查 get registered kernels returns copy。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestKernelRegistry.test_get_kernel_by_name_returns_kernel` (lines 787-799)
```python
    def test_get_kernel_by_name_returns_kernel(self):
        """Test get_kernel_by_name returns registered kernel."""
        with dummy_kernel_registry() as register:
            wrapper = register("test_kernel", config_picker=lambda args, keys: None)(
                _add_kernel
            )

        from vllm.kernels.helion.register import _REGISTERED_KERNELS

        _REGISTERED_KERNELS["test_kernel"] = wrapper

        result = get_kernel_by_name("test_kernel")
        assert result is wrapper
```
**EN:** This method on `TestKernelRegistry` checks get kernel by name returns kernel. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestKernelRegistry` 中的这个方法用于检查 get kernel by name returns kernel。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestKernelRegistry.test_get_kernel_by_name_returns_none_for_missing` (lines 801-804)
```python
    def test_get_kernel_by_name_returns_none_for_missing(self):
        """Test get_kernel_by_name returns None for missing kernel."""
        result = get_kernel_by_name("nonexistent")
        assert result is None
```
**EN:** This method on `TestKernelRegistry` checks get kernel by name returns none for missing. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestKernelRegistry` 中的这个方法用于检查 get kernel by name returns none for missing。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestKernelRegistry.test_register_kernel_auto_generates_fake_impl` (lines 806-819)
```python
    def test_register_kernel_auto_generates_fake_impl(self):
        """Test register_kernel auto-generates fake_impl when not provided."""
        with (
            dummy_kernel_registry() as register,
            patch("vllm.kernels.helion.register.infer_fake_impl") as mock_infer,
        ):
            mock_fake = Mock()
            mock_infer.return_value = mock_fake
            wrapper = register(
                config_picker=lambda args, keys: None,
            )(_add_kernel)

        mock_infer.assert_called_once_with(_add_kernel, None)
        assert wrapper._fake_impl is mock_fake
```
**EN:** This method on `TestKernelRegistry` checks register kernel auto generates fake impl. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestKernelRegistry` 中的这个方法用于检查 register kernel auto generates fake impl。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestKernelRegistry.test_register_kernel_creates_wrapper` (lines 821-830)
```python
    def test_register_kernel_creates_wrapper(self):
        """Test register_kernel creates HelionKernelWrapper."""
        with dummy_kernel_registry() as register:
            result = register("test_name", config_picker=lambda args, keys: None)(
                _add_kernel
            )

        assert isinstance(result, HelionKernelWrapper)
        assert result.op_name == "test_name"
        assert result.raw_kernel_func is _add_kernel
```
**EN:** This method on `TestKernelRegistry` checks register kernel creates wrapper. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestKernelRegistry` 中的这个方法用于检查 register kernel creates wrapper。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestKernelRegistry.test_register_kernel_auto_detects_name` (lines 832-837)
```python
    def test_register_kernel_auto_detects_name(self):
        """Test register_kernel uses function name when no name provided."""
        with dummy_kernel_registry() as register:
            wrapper = register(config_picker=lambda args, keys: None)(_add_kernel)

        assert wrapper.op_name == "_add_kernel"
```
**EN:** This method on `TestKernelRegistry` checks register kernel auto detects name. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestKernelRegistry` 中的这个方法用于检查 register kernel auto detects name。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestKernelRegistry.test_register_kernel_registers_in_global_registry` (lines 839-848)
```python
    def test_register_kernel_registers_in_global_registry(self):
        """Test register_kernel adds wrapper to global registry."""
        with dummy_kernel_registry() as register:
            wrapper = register("test_kernel", config_picker=lambda args, keys: None)(
                _add_kernel
            )

        registered_kernels = get_registered_kernels()
        assert "test_kernel" in registered_kernels
        assert registered_kernels["test_kernel"] is wrapper
```
**EN:** This method on `TestKernelRegistry` checks register kernel registers in global registry. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestKernelRegistry` 中的这个方法用于检查 register kernel registers in global registry。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestKernelRegistry.test_register_kernel_passes_helion_settings` (lines 850-862)
```python
    def test_register_kernel_passes_helion_settings(self):
        """Test register_kernel passes helion_settings to wrapper."""
        settings = helion.Settings()
        settings.print_output_code = True

        with dummy_kernel_registry() as register:
            result = register(
                "test_name",
                config_picker=lambda args, keys: None,
                helion_settings=settings,
            )(_add_kernel)

        assert result.helion_settings is settings
```
**EN:** This method on `TestKernelRegistry` checks register kernel passes helion settings. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestKernelRegistry` 中的这个方法用于检查 register kernel passes helion settings。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestKernelRegistry.test_register_kernel_supports_decorator_syntax` (lines 864-876)
```python
    def test_register_kernel_supports_decorator_syntax(self):
        """Test register_kernel works with decorator arguments."""
        mock_fake = Mock()

        with dummy_kernel_registry() as register:
            result = register(
                "custom_name",
                config_picker=lambda args, keys: None,
                fake_impl=mock_fake,
            )(_add_kernel)

        assert result.op_name == "custom_name"
        assert result._fake_impl is mock_fake
```
**EN:** This method on `TestKernelRegistry` checks register kernel supports decorator syntax. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestKernelRegistry` 中的这个方法用于检查 register kernel supports decorator syntax。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestKernelRegistry.test_register_kernel_raises_on_duplicate_registration` (lines 878-888)
```python
    def test_register_kernel_raises_on_duplicate_registration(self):
        """Test register_kernel raises error on duplicate names."""
        with dummy_kernel_registry() as register:
            register("duplicate_name", config_picker=lambda args, keys: None)(
                _add_kernel
            )

            with pytest.raises(ValueError, match="already registered"):
                register("duplicate_name", config_picker=lambda args, keys: None)(
                    _add_kernel
                )
```
**EN:** This method on `TestKernelRegistry` checks register kernel raises on duplicate registration. the expected failure path is asserted explicitly.
**CN:** `TestKernelRegistry` 中的这个方法用于检查 register kernel raises on duplicate registration。 代码会显式断言预期的失败路径。

### Method `TestKernelRegistry.test_register_kernel_rejects_autotuner_fn_in_settings` (lines 890-903)
```python
    def test_register_kernel_rejects_autotuner_fn_in_settings(self):
        """Test register_kernel rejects conflicting autotuner_fn."""
        mock_settings = Mock()
        mock_settings.to_dict.return_value = {"autotuner_fn": Mock()}

        with pytest.raises(ValueError, match="uses a custom autotuner"):

            @register_kernel(
                "test",
                config_picker=lambda args, keys: None,
                helion_settings=mock_settings,
            )
            def test_kernel(x):
                return x
```
**EN:** This method on `TestKernelRegistry` checks register kernel rejects autotuner fn in settings. the expected failure path is asserted explicitly.
**CN:** `TestKernelRegistry` 中的这个方法用于检查 register kernel rejects autotuner fn in settings。 代码会显式断言预期的失败路径。

### Method `TestKernelRegistry.test_register_kernel_no_warning_with_static_shapes_false` (lines 905-920)
```python
    def test_register_kernel_no_warning_with_static_shapes_false(self):
        """Test register_kernel doesn't warn with static_shapes=False."""
        mock_settings = Mock()
        mock_settings.to_dict.return_value = {"static_shapes": False}

        with (
            dummy_kernel_registry() as register,
            patch("vllm.kernels.helion.register.logger") as mock_logger,
        ):
            register(
                "test",
                config_picker=lambda args, keys: None,
                helion_settings=mock_settings,
            )(_add_kernel)

        mock_logger.warning.assert_not_called()
```
**EN:** This method on `TestKernelRegistry` checks register kernel no warning with static shapes false. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestKernelRegistry` 中的这个方法用于检查 register kernel no warning with static shapes false。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestKernelRegistry.test_disabled_kernel_appears_in_registry` (lines 922-953)
```python
    def test_disabled_kernel_appears_in_registry(self):
        """Test that a disabled wrapper is still in the global registry."""

        def fake_impl(*args, **kwargs):
            return torch.zeros_like(args[0])

        mock_config_manager = Mock(spec=ConfigManager)
        mock_config_manager.get_platform_configs = Mock(return_value={})

        with (
            patch(
                "vllm.kernels.helion.config_manager.ConfigManager",
                return_value=mock_config_manager,
            ),
            patch(
                "vllm.kernels.helion.utils.get_canonical_gpu_name",
                return_value="nvidia_h200",
            ),
            patch("vllm.kernels.helion.register.helion.kernel") as mock_kernel,
        ):
            mock_kernel.return_value = Mock(return_value=_add_kernel)

            wrapper = register_kernel(
                "disabled_kernel",
                config_picker=lambda args, keys: None,
                fake_impl=fake_impl,
            )(_add_kernel)

        assert wrapper._disabled is True
        registered = get_registered_kernels()
        assert "disabled_kernel" in registered
        assert registered["disabled_kernel"] is wrapper
```
**EN:** This method on `TestKernelRegistry` checks disabled kernel appears in registry. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestKernelRegistry` 中的这个方法用于检查 disabled kernel appears in registry。 结尾处的断言会固定预期行为或计算图形态。

### Class `TestTorchCompileHOP` (lines 956-959)
```python
@pytest.mark.skipif(not _HOP_AVAILABLE, reason="Requires PyTorch >= 2.11 for HOP")
class TestTorchCompileHOP:
    """Test that HelionKernelWrapper emits the correct HOP under torch.compile."""
```
**EN:** This helper class groups the state and behavior needed for TestTorchCompileHOP. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestTorchCompileHOP 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestTorchCompileHOP.test_compiled_graph_contains_helion_hop` (lines 960-1012)
```python
    def test_compiled_graph_contains_helion_hop(self):
        """Verify torch.compile on a HelionKernelWrapper emits a
        helion_kernel_wrapper_mutation HOP node in the FX graph."""
        configs: dict[CaseKey, helion.Config] = {
            CaseKey.default(): helion.Config(block_sizes=[4, 4])
        }

        with dummy_kernel_registry(configs=configs) as register:
            add_helion_kernel = register(
                op_name="test_torch_compile_add_kernel",
                config_picker=lambda args, keys: None,
            )(_add_kernel)

        captured_graph: torch.fx.GraphModule | None = None

        def capturing_backend(gm, example_inputs):
            nonlocal captured_graph
            assert captured_graph is None, "Backend called multiple times"
            captured_graph = gm
            return gm.forward

        def f(x, y):
            return add_helion_kernel(x, y)

        torch._dynamo.reset()
        compiled_f = torch.compile(f, backend=capturing_backend, fullgraph=True)

        x = torch.randn(4, 4, device="cuda")
        y = torch.randn(4, 4, device="cuda")

        # Run compiled version and capture graph
        compiled_result = compiled_f(x, y)

        assert captured_graph is not None
        hop_nodes = [
            node
            for node in captured_graph.graph.nodes
            if node.op == "call_function"
            and node.target is helion_kernel_wrapper_mutation
        ]
        assert len(hop_nodes) > 0, (
            "Expected helion_kernel_wrapper_mutation HOP node in compiled graph, "
            f"but found none. Graph nodes: "
            f"{[(n.op, n.target) for n in captured_graph.graph.nodes]}"
        )

        # Verify compiled result matches eager execution
        eager_result = f(x, y)  # Run in eager mode

        assert torch.allclose(compiled_result, eager_result, atol=1e-5, rtol=1e-5), (
            "Compiled execution result doesn't match eager execution. "
            f"Max difference: {torch.max(torch.abs(compiled_result - eager_result))}"
        )
```
**EN:** This method on `TestTorchCompileHOP` checks compiled graph contains helion hop. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestTorchCompileHOP` 中的这个方法用于检查 compiled graph contains helion hop。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestTorchCompileHOP.test_inductor_backend_compiles_helion_hop` (lines 1014-1060)
```python
    @pytest.mark.skipif(
        not (_HOP_AVAILABLE and supports_torch_compile_fusion()),
        reason="Requires PyTorch with Helion inductor fusion support",
    )
    def test_inductor_backend_compiles_helion_hop(self):
        """Test torch.compile with inductor backend and Helion fusion enabled."""

        configs: dict[CaseKey, helion.Config] = {
            CaseKey.default(): helion.Config(block_sizes=[4, 4])
        }

        with dummy_kernel_registry(configs=configs) as register:
            add_helion_kernel = register(
                op_name="test_inductor_add_kernel",
                config_picker=lambda args, keys: None,
                helion_settings=helion.Settings(
                    torch_compile_fusion=True, static_shapes=False
                ),
            )(_add_kernel)

        def f(x, y):
            x = x * 2.0
            y = y + 1.0
            out = add_helion_kernel(x, y)
            return out.relu()

        torch._dynamo.reset()
        compiled_f = torch.compile(f, backend="inductor", fullgraph=True)

        x = torch.randn(4, 4, device="cuda")
        y = torch.randn(4, 4, device="cuda")

        compiled_result, source_codes = run_and_get_code(compiled_f, x, y)
        eager_result = f(x, y)

        assert torch.allclose(compiled_result, eager_result, atol=1e-5, rtol=1e-5), (
            "Inductor-compiled result doesn't match eager execution. "
            f"Max difference: {torch.max(torch.abs(compiled_result - eager_result))}"
        )

        # With fusion enabled, prologue/epilogue ops should be fused into
        # a single triton kernel rather than generating separate kernels.
        kernel_count = sum(code.count("@triton.jit") for code in source_codes)
        assert kernel_count == 1, (
            f"Expected 1 fused triton kernel, got {kernel_count}. "
            "Prologue/epilogue ops were not fused into the Helion kernel."
        )
```
**EN:** This method on `TestTorchCompileHOP` checks inductor backend compiles helion hop. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestTorchCompileHOP` 中的这个方法用于检查 inductor backend compiles helion hop。 不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

## Key Concepts / 关键概念
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。
- **Reusable fixtures / 可复用 fixture:** Fixtures package common setup so each test can focus on the scenario under study. / fixture 封装公共准备逻辑，让每个测试更专注于待验证场景。

## Dependencies / 依赖关系
- `unittest.mock -> Mock, patch`
- `pytest`
- `torch`
- `vllm.utils.import_utils -> has_helion`
- `helion`
- `helion.language`
- `tests.kernels.helion.helpers -> dummy_kernel_registry`
- `vllm.kernels.helion.case_key -> CaseKey`
- `vllm.kernels.helion.config_manager -> ConfigManager`
- `vllm.kernels.helion.register -> _HOP_AVAILABLE, ConfiguredHelionKernel, HelionKernelWrapper, get_kernel_by_name, get_registered_kernels, register_kernel, validate_helion_settings`
