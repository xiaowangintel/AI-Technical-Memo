# register.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/kernels/helion/register.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Registers Helion kernels and maps preset configs into runtime dispatch. / 注册 Helion 内核，并把预设配置接入运行时分派。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 3-37)
```python
"""
vLLM Helion kernel registration with pre-tuned config selection.

This module leverages Helion's internal config selection infrastructure to use
pre-tuned configs instead of runtime autotuning.

How Helion Normally Works
-------------------------
For each kernel invocation, Helion:
1. Computes a cache key from input arguments
2. Looks up the key in its internal compilation cache
3. On cache miss, runs autotuning to find the best config
4. Compiles and caches the kernel with that config

How We Override It
------------------
We override two Helion hooks to use pre-tuned configs:

1. **key**: We provide a key function (derived from config_picker) that
   computes cache keys matching our pre-tuned config keys. This ensures Helion's
   internal cache uses keys that correspond to configs we've prepared.

2. **autotuner_fn**: We provide PresetConfigSearch which, instead of autotuning,
   simply returns the pre-tuned config for the computed key. On cache miss,
   Helion calls our autotuner which returns the author-prepared config.

Both hooks use the same config_picker logic to ensure the cache key computed
by key matches the config returned by the autotuner.

Key Classes
-----------
- HelionKernelWrapper: Wraps raw kernel + config_picker, creates configured kernels
- ConfiguredHelionKernel: Platform-specific kernel with pre-tuned configs
- PresetConfigSearch: Custom autotuner that returns pre-tuned configs
"""
```
**EN:** The opening docstring explains the file's role and design intent, giving readers context before the concrete kernel code starts.
**CN:** 开头的文档字符串说明了文件职责与设计意图，让读者在进入具体内核实现前先获得整体上下文。

### Imports (lines 39-50)
```python
from __future__ import annotations

from collections.abc import Callable
from typing import Any

import torch
from torch.library import Library

from vllm.kernels.helion.case_key import CaseKey
from vllm.logger import init_logger
from vllm.utils.import_utils import has_helion
from vllm.utils.torch_utils import direct_register_custom_op
```
**EN:** This import block loads `__future__`, `collections.abc`, `typing`, `torch`, `torch.library`, `vllm.kernels.helion.case_key`, ..., establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `register.py`.
**CN:** 该导入代码块加载了 `__future__`, `collections.abc`, `typing`, `torch`, `torch.library`, `vllm.kernels.helion.case_key`, ...，为 `register.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Conditional setup (lines 52-56)
```python
if not has_helion():
    raise ImportError(
        "register module requires helion to be installed. "
        "Install it with: pip install helion"
    )
```
**EN:** This conditional gate checks `not has_helion()` and switches behavior based on optional dependencies or runtime capability.
**CN:** 该条件分支检查 `not has_helion()`，并根据可选依赖或运行时能力切换行为。

### Imports (lines 58-61)
```python
import helion
from helion.autotuner.base_search import BaseAutotuner
from helion.runtime.config import Config
from helion.runtime.settings import default_autotuner_fn
```
**EN:** This import block loads `helion`, `helion.autotuner.base_search`, `helion.runtime.config`, `helion.runtime.settings`, establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `register.py`.
**CN:** 该导入代码块加载了 `helion`, `helion.autotuner.base_search`, `helion.runtime.config`, `helion.runtime.settings`，为 `register.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Constants / assignments (lines 67-67)
```python
_HOP_AVAILABLE = False
```
**EN:** This block defines module constants (`_HOP_AVAILABLE`) that encode defaults, feature flags, or operator metadata used later in the file.
**CN:** 该代码块定义了模块常量（`_HOP_AVAILABLE`），用于表示默认值、特性开关或后续逻辑要使用的算子元数据。

### Conditional setup (lines 69-75)
```python
if _HOP_AVAILABLE:
    from helion._compat import supports_torch_compile_fusion
    from helion._compiler._dynamo.higher_order_ops import helion_kernel_side_table
    from helion._compiler._dynamo.variables import HelionKernelVariable
    from helion.runtime.kernel import Kernel
    from torch._dynamo.guards import GuardBuilder
    from torch._dynamo.variables.builder import VariableBuilder
```
**EN:** This conditional gate checks `_HOP_AVAILABLE` and switches behavior based on optional dependencies or runtime capability.
**CN:** 该条件分支检查 `_HOP_AVAILABLE`，并根据可选依赖或运行时能力切换行为。

### Constants / assignments (lines 78-82)
```python
logger = init_logger(__name__)

vllm_helion_lib = Library("vllm_helion", "FRAGMENT")  # noqa

ConfigPicker = Callable[[tuple[Any, ...], list[CaseKey]], CaseKey | None]
```
**EN:** This assignment block initializes `logger`, `vllm_helion_lib`, `ConfigPicker`, capturing shared state or configuration consumed by later helper functions and kernel classes.
**CN:** 该赋值代码块初始化了 `logger`, `vllm_helion_lib`, `ConfigPicker`，保存后续辅助函数和内核类会消费的共享状态或配置。

### Function `validate_helion_settings` (lines 85-110)
```python
def validate_helion_settings(
    helion_settings: helion.Settings | None, op_name: str
) -> None:
    if helion_settings is None:
        return

    settings_dict = helion_settings.to_dict()

    if (
        "autotuner_fn" in settings_dict
        and settings_dict["autotuner_fn"] is not None
        and settings_dict["autotuner_fn"] is not default_autotuner_fn
    ):
        raise ValueError(
            f"HelionKernelWrapper for '{op_name}' uses a custom autotuner via "
            f"config picker. Remove 'autotuner_fn' from helion_settings and use "
            f"register_kernel(..., config_picker=...) instead."
        )

    if settings_dict.get("static_shapes") is True:
        logger.warning(
            "Kernel '%s' has static_shapes=True in helion_settings, "
            "which will be overridden to False. vLLM requires dynamic "
            "shapes for variable batch sizes and sequence lengths.",
            op_name,
        )
```
**EN:** This validation helper implements `validate_helion_settings`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `to_dict`, `ValueError`, `get`, `warning`.
**CN:** 该函数 `validate_helion_settings` 封装了此模块中的一段关键运行时逻辑，重点处理 validate helion settings 相关工作。 它内部会调用 `to_dict`, `ValueError`, `get`, `warning` 等例程。

### Function `create_helion_decorated_kernel` (lines 113-128)
```python
def create_helion_decorated_kernel(
    raw_kernel_func: Callable,
    helion_settings: helion.Settings | None = None,
    extra_kwargs: dict[str, Any] | None = None,
) -> Any:
    kernel_kwargs: dict[str, Any] = {}
    if helion_settings:
        kernel_kwargs.update(helion_settings.to_dict())

    # vLLM requires dynamic shapes for variable batch sizes and sequence lengths
    kernel_kwargs["static_shapes"] = False

    if extra_kwargs:
        kernel_kwargs.update(extra_kwargs)

    return helion.kernel(**kernel_kwargs)(raw_kernel_func)
```
**EN:** This factory helper implements `create_helion_decorated_kernel`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `update`, `kernel`, `to_dict`.
**CN:** 该函数 `create_helion_decorated_kernel` 封装了此模块中的一段关键运行时逻辑，重点处理 create helion decorated kernel 相关工作。 它内部会调用 `update`, `kernel`, `to_dict` 等例程。

### Class `PresetConfigSearch` (lines 131-143)
```python
class PresetConfigSearch(BaseAutotuner):
    """Custom autotuner that uses a preset config selector instead of autotuning."""

    def __init__(
        self,
        args: tuple[Any, ...],
        config_selector: Callable[[tuple[Any, ...]], Config],
    ):
        self.args = args
        self.config_selector = config_selector

    def autotune(self, *, skip_cache: bool = False) -> Config:
        return self.config_selector(self.args)
```
**EN:** This class defines `PresetConfigSearch`. Custom autotuner that uses a preset config selector instead of autotuning. It inherits from `BaseAutotuner`. Key methods include `__init__`, `autotune`.
**CN:** 该类定义了 `PresetConfigSearch`。 它主要负责与 `PresetConfigSearch` 对应的数据组织、接口约束或执行流程。 它继承自 `BaseAutotuner`。 关键方法包括 `__init__`, `autotune`。

### Method `PresetConfigSearch.__init__` (lines 134-140)
```python
    def __init__(
        self,
        args: tuple[Any, ...],
        config_selector: Callable[[tuple[Any, ...]], Config],
    ):
        self.args = args
        self.config_selector = config_selector
```
**EN:** This method implements `PresetConfigSearch.__init__`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `PresetConfigSearch.__init__` 封装了此模块中的一段关键运行时逻辑，重点处理 init 相关工作。

### Method `PresetConfigSearch.autotune` (lines 142-143)
```python
    def autotune(self, *, skip_cache: bool = False) -> Config:
        return self.config_selector(self.args)
```
**EN:** This method implements `PresetConfigSearch.autotune`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `config_selector`.
**CN:** 该方法 `PresetConfigSearch.autotune` 封装了此模块中的一段关键运行时逻辑，重点处理 autotune 相关工作。 它内部会调用 `config_selector` 等例程。

### Class `ConfiguredHelionKernel` (lines 146-251)
```python
class ConfiguredHelionKernel:
    """A configured Helion kernel bound to a specific platform."""

    def __init__(
        self,
        op_name: str,
        config_picker: ConfigPicker | None,
        raw_kernel_func: Callable,
        helion_settings: helion.Settings | None = None,
    ):
        self.op_name = op_name
        self.config_picker = config_picker
        self.raw_kernel_func = raw_kernel_func
        self.helion_settings = helion_settings
        self._decorated_kernel = self._create_decorated_kernel()

    def __call__(self, *args, **kwargs):
        return self._decorated_kernel(*args, **kwargs)

    def _create_key_computer(self):
        """
        Create a key computer function derived from the config picker.

        The returned function receives kernel arguments unpacked (*args) to match
        Helion's key signature (called as self._key_fn(*args)).
        """
        if self.config_picker is None:
            raise RuntimeError(
                f"No config picker registered for kernel '{self.op_name}'. "
                f"A config_picker must be provided to register_kernel()."
            )

        picker = self.config_picker
        all_keys = list(self.configs.keys())
        default = CaseKey.default()
        has_default = default in self.configs

        def key_computer(*args):
            selected = picker(args, all_keys)
            if selected is not None:
                return str(selected)
            if has_default:
                return str(default)
            return None

        return key_computer

    def _create_config_selector(self, key_computer):
        str_to_key = {str(k): k for k in self.configs}

        def config_selector(args):
            selected_str = key_computer(*args)

            if selected_str is None:
                raise ValueError(
                    f"Config picker returned None for kernel "
                    f"'{self.op_name}' with available config keys: "
                    f"{list(self.configs.keys())}"
                )

            config_key = str_to_key.get(selected_str)
            if config_key is None:
                raise ValueError(
                    f"Config picker returned invalid config key "
                    f"'{selected_str}' for kernel "
                    f"'{self.op_name}'. "
                    f"Available keys: {list(self.configs.keys())}"
                )

            return self.configs[config_key]

        return config_selector

    def _load_platform_configs(self) -> None:
        from vllm.kernels.helion.config_manager import ConfigManager
        from vllm.kernels.helion.utils import get_canonical_gpu_name

        self.platform = get_canonical_gpu_name()
        config_manager = ConfigManager()
        self.configs = config_manager.get_platform_configs(self.op_name, self.platform)

        if not self.configs:
            raise ValueError(
                f"No configs available for kernel '{self.op_name}' "
                f"on platform '{self.platform}'"
            )

    def _create_decorated_kernel(self) -> Callable[..., Any]:
        self._load_platform_configs()

        key_computer = self._create_key_computer()
        config_selector = self._create_config_selector(key_computer)

        extra_kwargs = {
            "autotuner_fn": lambda _, args: PresetConfigSearch(args, config_selector),
            "key": key_computer,
        }

        logger.debug(
            "Creating decorated kernel %s with custom autotuner on platform %s",
            self.op_name,
            self.platform,
        )
        return create_helion_decorated_kernel(
            self.raw_kernel_func, self.helion_settings, extra_kwargs
        )
```
**EN:** This kernel class defines `ConfiguredHelionKernel`. A configured Helion kernel bound to a specific platform. Key methods include `__init__`, `__call__`, `_create_key_computer`, `_create_config_selector`, `_load_platform_configs`, ....
**CN:** 该内核类定义了 `ConfiguredHelionKernel`。 它主要负责与 `ConfiguredHelionKernel` 对应的数据组织、接口约束或执行流程。 关键方法包括 `__init__`, `__call__`, `_create_key_computer`, `_create_config_selector`, `_load_platform_configs`, ...。

### Method `ConfiguredHelionKernel.__init__` (lines 149-160)
```python
    def __init__(
        self,
        op_name: str,
        config_picker: ConfigPicker | None,
        raw_kernel_func: Callable,
        helion_settings: helion.Settings | None = None,
    ):
        self.op_name = op_name
        self.config_picker = config_picker
        self.raw_kernel_func = raw_kernel_func
        self.helion_settings = helion_settings
        self._decorated_kernel = self._create_decorated_kernel()
```
**EN:** This method implements `ConfiguredHelionKernel.__init__`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `_create_decorated_kernel`.
**CN:** 该方法 `ConfiguredHelionKernel.__init__` 封装了此模块中的一段关键运行时逻辑，重点处理 init 相关工作。 它内部会调用 `_create_decorated_kernel` 等例程。

### Method `ConfiguredHelionKernel.__call__` (lines 162-163)
```python
    def __call__(self, *args, **kwargs):
        return self._decorated_kernel(*args, **kwargs)
```
**EN:** This method implements `ConfiguredHelionKernel.__call__`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `_decorated_kernel`.
**CN:** 该方法 `ConfiguredHelionKernel.__call__` 封装了此模块中的一段关键运行时逻辑，重点处理 call 相关工作。 它内部会调用 `_decorated_kernel` 等例程。

### Method `ConfiguredHelionKernel._create_key_computer` (lines 165-191)
```python
    def _create_key_computer(self):
        """
        Create a key computer function derived from the config picker.

        The returned function receives kernel arguments unpacked (*args) to match
        Helion's key signature (called as self._key_fn(*args)).
        """
        if self.config_picker is None:
            raise RuntimeError(
                f"No config picker registered for kernel '{self.op_name}'. "
                f"A config_picker must be provided to register_kernel()."
            )

        picker = self.config_picker
        all_keys = list(self.configs.keys())
        default = CaseKey.default()
        has_default = default in self.configs

        def key_computer(*args):
            selected = picker(args, all_keys)
            if selected is not None:
                return str(selected)
            if has_default:
                return str(default)
            return None

        return key_computer
```
**EN:** This method implements `ConfiguredHelionKernel._create_key_computer`. Create a key computer function derived from the config picker. Internally it relies on calls such as `list`, `default`, `RuntimeError`, `keys`.
**CN:** 该方法 `ConfiguredHelionKernel._create_key_computer` 封装了此模块中的一段关键运行时逻辑，重点处理 create key computer 相关工作。 它内部会调用 `list`, `default`, `RuntimeError`, `keys` 等例程。

### Method `ConfiguredHelionKernel._create_config_selector` (lines 193-217)
```python
    def _create_config_selector(self, key_computer):
        str_to_key = {str(k): k for k in self.configs}

        def config_selector(args):
            selected_str = key_computer(*args)

            if selected_str is None:
                raise ValueError(
                    f"Config picker returned None for kernel "
                    f"'{self.op_name}' with available config keys: "
                    f"{list(self.configs.keys())}"
                )

            config_key = str_to_key.get(selected_str)
            if config_key is None:
                raise ValueError(
                    f"Config picker returned invalid config key "
                    f"'{selected_str}' for kernel "
                    f"'{self.op_name}'. "
                    f"Available keys: {list(self.configs.keys())}"
                )

            return self.configs[config_key]

        return config_selector
```
**EN:** This method implements `ConfiguredHelionKernel._create_config_selector`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `str`, `key_computer`, `get`, `ValueError`.
**CN:** 该方法 `ConfiguredHelionKernel._create_config_selector` 封装了此模块中的一段关键运行时逻辑，重点处理 create config selector 相关工作。 它内部会调用 `str`, `key_computer`, `get`, `ValueError` 等例程。

### Method `ConfiguredHelionKernel._load_platform_configs` (lines 219-231)
```python
    def _load_platform_configs(self) -> None:
        from vllm.kernels.helion.config_manager import ConfigManager
        from vllm.kernels.helion.utils import get_canonical_gpu_name

        self.platform = get_canonical_gpu_name()
        config_manager = ConfigManager()
        self.configs = config_manager.get_platform_configs(self.op_name, self.platform)

        if not self.configs:
            raise ValueError(
                f"No configs available for kernel '{self.op_name}' "
                f"on platform '{self.platform}'"
            )
```
**EN:** This method implements `ConfiguredHelionKernel._load_platform_configs`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `get_canonical_gpu_name`, `ConfigManager`, `get_platform_configs`, `ValueError`.
**CN:** 该方法 `ConfiguredHelionKernel._load_platform_configs` 封装了此模块中的一段关键运行时逻辑，重点处理 load platform configs 相关工作。 它内部会调用 `get_canonical_gpu_name`, `ConfigManager`, `get_platform_configs`, `ValueError` 等例程。

### Method `ConfiguredHelionKernel._create_decorated_kernel` (lines 233-251)
```python
    def _create_decorated_kernel(self) -> Callable[..., Any]:
        self._load_platform_configs()

        key_computer = self._create_key_computer()
        config_selector = self._create_config_selector(key_computer)

        extra_kwargs = {
            "autotuner_fn": lambda _, args: PresetConfigSearch(args, config_selector),
            "key": key_computer,
        }

        logger.debug(
            "Creating decorated kernel %s with custom autotuner on platform %s",
            self.op_name,
            self.platform,
        )
        return create_helion_decorated_kernel(
            self.raw_kernel_func, self.helion_settings, extra_kwargs
        )
```
**EN:** This method implements `ConfiguredHelionKernel._create_decorated_kernel`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `_load_platform_configs`, `_create_key_computer`, `_create_config_selector`, `debug`.
**CN:** 该方法 `ConfiguredHelionKernel._create_decorated_kernel` 封装了此模块中的一段关键运行时逻辑，重点处理 create decorated kernel 相关工作。 它内部会调用 `_load_platform_configs`, `_create_key_computer`, `_create_config_selector`, `debug` 等例程。

### Class `HelionKernelWrapper` (lines 254-364)
```python
class HelionKernelWrapper:
    """Wrapper for Helion kernels with pre-tuned config selection and HOP support."""

    def __init__(
        self,
        raw_kernel_func: Callable,
        op_name: str,
        fake_impl: Callable,
        config_picker: ConfigPicker,
        helion_settings: helion.Settings | None = None,
        input_generator: (Callable[[], dict[CaseKey, tuple[Any, ...]]] | None) = None,
    ):
        # Validate helion_settings doesn't conflict with our custom autotuner
        validate_helion_settings(helion_settings, op_name)

        self.raw_kernel_func = raw_kernel_func
        self.op_name = op_name
        self._fake_impl = fake_impl
        self.helion_settings = helion_settings
        self._config_picker = config_picker
        self._input_generator = input_generator
        self._configured_kernel: ConfiguredHelionKernel | None = None
        # TODO(@gmagogsfm): Remove this disable flag once integrated with vLLM IR,
        # which handles op enablement/disablement.
        self._disabled = False
        self._disabled_reason: str | None = None

        try:
            if not _HOP_AVAILABLE:
                self._get_or_register_custom_op()
            else:
                self.get_configured_op()
        except ValueError as e:
            self._disabled = True
            self._disabled_reason = str(e)
            logger.warning(
                "Helion kernel '%s' is disabled: %s",
                op_name,
                self._disabled_reason,
            )

    def __call__(self, *args, **kwargs):
        if self._disabled:
            raise RuntimeError(
                f"Helion kernel '{self.op_name}' is disabled: {self._disabled_reason}"
            )
        if not _HOP_AVAILABLE:
            op = getattr(torch.ops.vllm_helion, self.op_name)
            return op(*args, **kwargs)
        assert self._configured_kernel is not None, (
            f"Kernel '{self.op_name}' was not initialized. "
            "Please open an issue on GitHub."
        )

        # During Dynamo tracing, this call will be intercepted by our custom
        # HelionKernelWrapperVariable and handled via proper HOP emission.
        # During eager execution, call the kernel directly.
        return self._configured_kernel(*args, **kwargs)

    def get_inputs(self) -> dict[CaseKey, tuple[Any, ...]]:
        if self._input_generator is None:
            raise NotImplementedError(
                f"No input generator registered for kernel '{self.op_name}'. "
                f"Use register_kernel(..., input_generator=...) to register one."
            )
        return self._input_generator()

    def run_autotune(
        self,
        inputs: tuple[Any, ...],
        autotune_effort: str = "quick",
    ) -> Config:
        """Run autotuning for a single input configuration."""
        extra_kwargs = {
            "autotune_effort": autotune_effort,
            "autotune_ignore_errors": True,
        }
        autotune_kernel = create_helion_decorated_kernel(
            self.raw_kernel_func, self.helion_settings, extra_kwargs
        )
        return autotune_kernel.autotune(inputs)

    def get_configured_op(self) -> ConfiguredHelionKernel:
        if self._disabled:
            raise RuntimeError(
                f"Helion kernel '{self.op_name}' is disabled: {self._disabled_reason}"
            )
        if self._configured_kernel is None:
            self._configured_kernel = ConfiguredHelionKernel(
                op_name=self.op_name,
                config_picker=self._config_picker,
                raw_kernel_func=self.raw_kernel_func,
                helion_settings=self.helion_settings,
            )
        return self._configured_kernel

    def _get_or_register_custom_op(self) -> Any:
        if hasattr(torch.ops.vllm_helion, self.op_name):
            return getattr(torch.ops.vllm_helion, self.op_name)

        configured_kernel = self.get_configured_op()

        logger.info("Registering op: vllm_helion::%s", self.op_name)
        direct_register_custom_op(
            op_name=self.op_name,
            op_func=configured_kernel._decorated_kernel,
            mutates_args=None,
            fake_impl=self._fake_impl,
            target_lib=vllm_helion_lib,
        )
        return getattr(torch.ops.vllm_helion, self.op_name)
```
**EN:** This kernel class defines `HelionKernelWrapper`. Wrapper for Helion kernels with pre-tuned config selection and HOP support. Key methods include `__init__`, `__call__`, `get_inputs`, `run_autotune`, `get_configured_op`, ....
**CN:** 该内核类定义了 `HelionKernelWrapper`。 它主要负责与 `HelionKernelWrapper` 对应的数据组织、接口约束或执行流程。 关键方法包括 `__init__`, `__call__`, `get_inputs`, `run_autotune`, `get_configured_op`, ...。

### Method `HelionKernelWrapper.__init__` (lines 257-293)
```python
    def __init__(
        self,
        raw_kernel_func: Callable,
        op_name: str,
        fake_impl: Callable,
        config_picker: ConfigPicker,
        helion_settings: helion.Settings | None = None,
        input_generator: (Callable[[], dict[CaseKey, tuple[Any, ...]]] | None) = None,
    ):
        # Validate helion_settings doesn't conflict with our custom autotuner
        validate_helion_settings(helion_settings, op_name)

        self.raw_kernel_func = raw_kernel_func
        self.op_name = op_name
        self._fake_impl = fake_impl
        self.helion_settings = helion_settings
        self._config_picker = config_picker
        self._input_generator = input_generator
        self._configured_kernel: ConfiguredHelionKernel | None = None
        # TODO(@gmagogsfm): Remove this disable flag once integrated with vLLM IR,
        # which handles op enablement/disablement.
        self._disabled = False
        self._disabled_reason: str | None = None

        try:
            if not _HOP_AVAILABLE:
                self._get_or_register_custom_op()
            else:
                self.get_configured_op()
        except ValueError as e:
            self._disabled = True
            self._disabled_reason = str(e)
            logger.warning(
                "Helion kernel '%s' is disabled: %s",
                op_name,
                self._disabled_reason,
            )
```
**EN:** This method implements `HelionKernelWrapper.__init__`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `validate_helion_settings`, `_get_or_register_custom_op`, `get_configured_op`, `str`.
**CN:** 该方法 `HelionKernelWrapper.__init__` 封装了此模块中的一段关键运行时逻辑，重点处理 init 相关工作。 它内部会调用 `validate_helion_settings`, `_get_or_register_custom_op`, `get_configured_op`, `str` 等例程。

### Method `HelionKernelWrapper.__call__` (lines 295-311)
```python
    def __call__(self, *args, **kwargs):
        if self._disabled:
            raise RuntimeError(
                f"Helion kernel '{self.op_name}' is disabled: {self._disabled_reason}"
            )
        if not _HOP_AVAILABLE:
            op = getattr(torch.ops.vllm_helion, self.op_name)
            return op(*args, **kwargs)
        assert self._configured_kernel is not None, (
            f"Kernel '{self.op_name}' was not initialized. "
            "Please open an issue on GitHub."
        )

        # During Dynamo tracing, this call will be intercepted by our custom
        # HelionKernelWrapperVariable and handled via proper HOP emission.
        # During eager execution, call the kernel directly.
        return self._configured_kernel(*args, **kwargs)
```
**EN:** This method implements `HelionKernelWrapper.__call__`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `_configured_kernel`, `RuntimeError`, `getattr`, `op`.
**CN:** 该方法 `HelionKernelWrapper.__call__` 封装了此模块中的一段关键运行时逻辑，重点处理 call 相关工作。 它内部会调用 `_configured_kernel`, `RuntimeError`, `getattr`, `op` 等例程。

### Method `HelionKernelWrapper.get_inputs` (lines 313-319)
```python
    def get_inputs(self) -> dict[CaseKey, tuple[Any, ...]]:
        if self._input_generator is None:
            raise NotImplementedError(
                f"No input generator registered for kernel '{self.op_name}'. "
                f"Use register_kernel(..., input_generator=...) to register one."
            )
        return self._input_generator()
```
**EN:** This method implements `HelionKernelWrapper.get_inputs`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `_input_generator`, `NotImplementedError`.
**CN:** 该方法 `HelionKernelWrapper.get_inputs` 封装了此模块中的一段关键运行时逻辑，重点处理 get inputs 相关工作。 它内部会调用 `_input_generator`, `NotImplementedError` 等例程。

### Method `HelionKernelWrapper.run_autotune` (lines 321-334)
```python
    def run_autotune(
        self,
        inputs: tuple[Any, ...],
        autotune_effort: str = "quick",
    ) -> Config:
        """Run autotuning for a single input configuration."""
        extra_kwargs = {
            "autotune_effort": autotune_effort,
            "autotune_ignore_errors": True,
        }
        autotune_kernel = create_helion_decorated_kernel(
            self.raw_kernel_func, self.helion_settings, extra_kwargs
        )
        return autotune_kernel.autotune(inputs)
```
**EN:** This method implements `HelionKernelWrapper.run_autotune`. Run autotuning for a single input configuration. Internally it relies on calls such as `create_helion_decorated_kernel`, `autotune`.
**CN:** 该方法 `HelionKernelWrapper.run_autotune` 封装了此模块中的一段关键运行时逻辑，重点处理 run autotune 相关工作。 它内部会调用 `create_helion_decorated_kernel`, `autotune` 等例程。

### Method `HelionKernelWrapper.get_configured_op` (lines 336-348)
```python
    def get_configured_op(self) -> ConfiguredHelionKernel:
        if self._disabled:
            raise RuntimeError(
                f"Helion kernel '{self.op_name}' is disabled: {self._disabled_reason}"
            )
        if self._configured_kernel is None:
            self._configured_kernel = ConfiguredHelionKernel(
                op_name=self.op_name,
                config_picker=self._config_picker,
                raw_kernel_func=self.raw_kernel_func,
                helion_settings=self.helion_settings,
            )
        return self._configured_kernel
```
**EN:** This method implements `HelionKernelWrapper.get_configured_op`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `RuntimeError`, `ConfiguredHelionKernel`.
**CN:** 该方法 `HelionKernelWrapper.get_configured_op` 封装了此模块中的一段关键运行时逻辑，重点处理 get configured op 相关工作。 它内部会调用 `RuntimeError`, `ConfiguredHelionKernel` 等例程。

### Method `HelionKernelWrapper._get_or_register_custom_op` (lines 350-364)
```python
    def _get_or_register_custom_op(self) -> Any:
        if hasattr(torch.ops.vllm_helion, self.op_name):
            return getattr(torch.ops.vllm_helion, self.op_name)

        configured_kernel = self.get_configured_op()

        logger.info("Registering op: vllm_helion::%s", self.op_name)
        direct_register_custom_op(
            op_name=self.op_name,
            op_func=configured_kernel._decorated_kernel,
            mutates_args=None,
            fake_impl=self._fake_impl,
            target_lib=vllm_helion_lib,
        )
        return getattr(torch.ops.vllm_helion, self.op_name)
```
**EN:** This method implements `HelionKernelWrapper._get_or_register_custom_op`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `hasattr`, `get_configured_op`, `info`, `direct_register_custom_op`.
**CN:** 该方法 `HelionKernelWrapper._get_or_register_custom_op` 封装了此模块中的一段关键运行时逻辑，重点处理 get or register custom op 相关工作。 它内部会调用 `hasattr`, `get_configured_op`, `info`, `direct_register_custom_op` 等例程。

### Constants / assignments (lines 368-368)
```python
_REGISTERED_KERNELS: dict[str, HelionKernelWrapper] = {}
```
**EN:** This block defines module constants (`_REGISTERED_KERNELS`) that encode defaults, feature flags, or operator metadata used later in the file.
**CN:** 该代码块定义了模块常量（`_REGISTERED_KERNELS`），用于表示默认值、特性开关或后续逻辑要使用的算子元数据。

### Function `get_registered_kernels` (lines 371-372)
```python
def get_registered_kernels() -> dict[str, HelionKernelWrapper]:
    return _REGISTERED_KERNELS.copy()
```
**EN:** This lookup helper implements `get_registered_kernels`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `copy`.
**CN:** 该函数 `get_registered_kernels` 封装了此模块中的一段关键运行时逻辑，重点处理 get registered kernels 相关工作。 它内部会调用 `copy` 等例程。

### Function `get_kernel_by_name` (lines 375-376)
```python
def get_kernel_by_name(kernel_name: str) -> HelionKernelWrapper | None:
    return _REGISTERED_KERNELS.get(kernel_name)
```
**EN:** This lookup helper implements `get_kernel_by_name`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `get`.
**CN:** 该函数 `get_kernel_by_name` 封装了此模块中的一段关键运行时逻辑，重点处理 get kernel by name 相关工作。 它内部会调用 `get` 等例程。

### Function `infer_fake_impl` (lines 379-397)
```python
def infer_fake_impl(
    kernel_func: Callable,
    helion_settings: helion.Settings | None = None,
) -> Callable:
    def helion_fake_kernel(*args, **kwargs):
        kernel_kwargs = {}
        if helion_settings:
            kernel_kwargs.update(helion_settings.to_dict())

        temp_decorated_kernel = helion.kernel(**kernel_kwargs)(kernel_func)

        # Bind with args to get config_spec, then get a valid default config
        bound = temp_decorated_kernel.bind(args)
        default_config = bound.config_spec.default_config()
        compiled_runner = bound.compile_config(default_config)

        return compiled_runner(*args, **kwargs, _launcher=lambda *a, **kw: None)

    return helion_fake_kernel
```
**EN:** This helper implements `infer_fake_impl`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `bind`, `default_config`, `compile_config`, `compiled_runner`.
**CN:** 该函数 `infer_fake_impl` 封装了此模块中的一段关键运行时逻辑，重点处理 infer fake impl 相关工作。 它内部会调用 `bind`, `default_config`, `compile_config`, `compiled_runner` 等例程。

### Function `register_kernel` (lines 400-471)
```python
def register_kernel(
    op_name: str | None = None,
    *,
    config_picker: ConfigPicker,
    fake_impl: Callable | None = None,
    helion_settings: helion.Settings | None = None,
    input_generator: (Callable[[], dict[CaseKey, tuple[Any, ...]]] | None) = None,
) -> Callable[[Callable], HelionKernelWrapper]:
    """Register a Helion kernel with pre-tuned config selection.

    Args:
        config_picker: Required. Receives ``(args, config_keys)``
            where each config key is a ``dict[str, Any]`` mapping
            parameter names to values.  Return the best-matching
            dict, or ``None`` to fall back to the default config.

            Example::

                def pick_config(args, config_keys):
                    x = args[0]
                    best = min(config_keys, key=lambda k: abs(k["size"] - x.shape[0]))
                    return best

        input_generator: Optional. Returns ``dict[str, tuple]`` where
            each key is a serialized config key and each value is a
            tuple of arguments to pass to the kernel.

            Example::

                def generate_inputs():
                    return {
                        "4096": (torch.randn(4096, device="cuda"), 0.5),
                        "8192": (torch.randn(8192, device="cuda"), 0.5),
                    }
    """

    def decorator(kernel_func: Callable) -> HelionKernelWrapper:
        final_op_name = op_name if op_name else kernel_func.__name__

        if final_op_name in _REGISTERED_KERNELS:
            raise ValueError(
                f"Helion kernel '{final_op_name}' is already registered. "
                f"Use a different op_name or check for duplicate registrations."
            )

        final_fake_impl = fake_impl
        if final_fake_impl is None:
            final_fake_impl = infer_fake_impl(kernel_func, helion_settings)
            logger.debug(
                "Auto-generated fake_impl for Helion kernel '%s'",
                kernel_func.__name__,
            )

        kernel_wrapper = HelionKernelWrapper(
            raw_kernel_func=kernel_func,
            op_name=final_op_name,
            fake_impl=final_fake_impl,
            config_picker=config_picker,
            helion_settings=helion_settings,
            input_generator=input_generator,
        )

        _REGISTERED_KERNELS[final_op_name] = kernel_wrapper

        logger.info(
            "Registered Helion kernel '%s' as HelionKernelWrapper",
            kernel_func.__name__,
        )

        return kernel_wrapper

    return decorator
```
**EN:** This registration helper implements `register_kernel`. Register a Helion kernel with pre-tuned config selection. Internally it relies on calls such as `HelionKernelWrapper`, `info`, `ValueError`, `infer_fake_impl`.
**CN:** 该函数 `register_kernel` 封装了此模块中的一段关键运行时逻辑，重点处理 register kernel 相关工作。 它内部会调用 `HelionKernelWrapper`, `info`, `ValueError`, `infer_fake_impl` 等例程。

### Conditional setup (lines 475-501)
```python
if _HOP_AVAILABLE:

    def _register_vllm_helion_dynamo_variable():
        """Register HelionKernelWrapper with Dynamo's VariableBuilder.

        When Dynamo encounters a HelionKernelWrapper during tracing, this
        extracts the underlying Helion Kernel and delegates to Helion's own
        registered Kernel handler, which handles HOP emission, side table
        registration, and inductor lowering setup.
        """

        def wrap_helion_kernel_wrapper(
            builder: VariableBuilder, value: HelionKernelWrapper
        ):
            kernel = value.get_configured_op()._decorated_kernel
            if supports_torch_compile_fusion():
                helion_handler = VariableBuilder._type_dispatch()[Kernel]
                return helion_handler(builder, kernel)
            kernel_idx = helion_kernel_side_table.add_kernel(kernel)
            builder.install_guards(GuardBuilder.ID_MATCH)
            return HelionKernelVariable(kernel, kernel_idx, source=builder.source)

        dispatch = VariableBuilder._type_dispatch()
        dispatch[HelionKernelWrapper] = wrap_helion_kernel_wrapper

    # Register immediately when the module is imported
    _register_vllm_helion_dynamo_variable()
```
**EN:** This conditional gate checks `_HOP_AVAILABLE` and switches behavior based on optional dependencies or runtime capability.
**CN:** 该条件分支检查 `_HOP_AVAILABLE`，并根据可选依赖或运行时能力切换行为。

## Key Concepts / 关键概念
- **Helion integration / Helion 集成**
  - **EN:** The file plugs vLLM into Helion compilation, registration, or preset configuration flows.
  - **CN:** 该文件把 vLLM 接入 Helion 的编译、注册或预设配置流程。
- **Kernel registration / 内核注册**
  - **EN:** The file registers operators, backends, or preset configurations with PyTorch/vLLM.
  - **CN:** 该文件向 PyTorch/vLLM 注册算子、后端或预设配置。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `vllm.kernels.helion.case_key`, `vllm.logger`, `vllm.utils.import_utils`, `vllm.utils.torch_utils`
- **External / 外部依赖**: `__future__`, `collections.abc`, `typing`, `torch`, `torch.library`, `helion`, `helion.autotuner.base_search`, `helion.runtime.config`, `helion.runtime.settings`
