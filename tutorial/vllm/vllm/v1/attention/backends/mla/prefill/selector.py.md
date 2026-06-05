# selector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/mla/prefill/selector.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Selector for MLA prefill backends. / 该模块位于 `attention/backends/mla/prefill` 子系统，主要围绕 `MLAPrefillSelectorConfig`, `is_deepseek_r1_mla_compatible`, `_get_mla_prefill_backend_priorities` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""Selector for MLA prefill backends.

This module provides functions for selecting the appropriate MLA prefill
backend based on device capabilities and configuration.
"""

from functools import cache
from typing import TYPE_CHECKING, NamedTuple

import torch

from vllm.logger import init_logger
from vllm.platforms.interface import DeviceCapability
from vllm.v1.attention.backends.mla.prefill.registry import MLAPrefillBackendEnum

if TYPE_CHECKING:
    from vllm.config import VllmConfig
    from vllm.v1.attention.backends.mla.prefill.base import MLAPrefillBackend

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `MLAPrefillSelectorConfig` class / `MLAPrefillSelectorConfig` 类
```python
class MLAPrefillSelectorConfig(NamedTuple):
    """Hashable configuration for MLA prefill backend selection.

    This is analogous to AttentionSelectorConfig and contains model-specific
    configuration needed to select an MLA prefill backend, extracted from
    VllmConfig into a hashable form for caching.
    """

    dtype: torch.dtype
    is_r1_compatible: bool
```
**EN:** Introduces the `MLAPrefillSelectorConfig` class on top of `NamedTuple`. Core methods include its methods defined below. Docstring signal: Hashable configuration for MLA prefill backend selection.
**CN:** 这里定义 `MLAPrefillSelectorConfig` 类，其基类包括 `NamedTuple`。核心方法包括 下方定义的方法。

### `is_deepseek_r1_mla_compatible` function / `is_deepseek_r1_mla_compatible` 函数
```python
def is_deepseek_r1_mla_compatible(vllm_config: "VllmConfig") -> bool:
    """Check if model has DeepSeek R1 compatible MLA dimensions.

    DeepSeek R1 MLA dimensions are:
    - qk_nope_head_dim = 128
    - qk_rope_head_dim = 64
    - v_head_dim = 128
    """
    if vllm_config.model_config is None:
        return False
    hf_text_config = vllm_config.model_config.hf_text_config
    qk_nope_head_dim = getattr(hf_text_config, "qk_nope_head_dim", 1)
    qk_rope_head_dim = getattr(hf_text_config, "qk_rope_head_dim", 1)
    v_head_dim = getattr(hf_text_config, "v_head_dim", 1)
    return qk_nope_head_dim == 128 and qk_rope_head_dim == 64 and v_head_dim == 128
```
**EN:** This function answers a boolean capability check within the module. The docstring frames it as: Check if model has DeepSeek R1 compatible MLA dimensions. Key calls include `getattr`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会回答布尔能力判断，其作用域位于the module。 关键调用包括 `getattr`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_get_mla_prefill_backend_priorities` function / `_get_mla_prefill_backend_priorities` 函数
```python
def _get_mla_prefill_backend_priorities(
    device_capability: DeviceCapability,
) -> list[MLAPrefillBackendEnum]:
    """Get MLA prefill backend priorities based on device capability.

    Args:
        device_capability: The device's compute capability.

    Returns:
        List of backends in priority order (highest priority first).
    """
    if device_capability.major == 10:  # Blackwell
        return [
            MLAPrefillBackendEnum.FLASH_ATTN,
            MLAPrefillBackendEnum.TRTLLM_RAGGED,
            MLAPrefillBackendEnum.FLASHINFER,
            MLAPrefillBackendEnum.TOKENSPEED_MLA,
        ]
    else:  # Hopper (SM90) and older
        return [
            MLAPrefillBackendEnum.FLASH_ATTN,
        ]
```
**EN:** This function implements `_get_mla_prefill_backend_priorities` within the module. The docstring frames it as: Get MLA prefill backend priorities based on device capability. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_get_mla_prefill_backend_priorities`，其作用域位于the module。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `get_mla_prefill_backend` function / `get_mla_prefill_backend` 函数
```python
def get_mla_prefill_backend(
    vllm_config: "VllmConfig",
) -> "type[MLAPrefillBackend]":
    """Select the MLA prefill backend based on configuration and device.

    This function first checks for explicit user preferences via
    mla_prefill_backend in AttentionConfig, then falls back to automatic
    priority-based selection.

    Args:
        vllm_config: The vLLM configuration.

    Returns:
        The selected prefill backend class.
    """
    from vllm.platforms import current_platform

    device_capability = current_platform.get_device_capability()
    if device_capability is None:
        logger.info_once(
            "Device capability not available, using FlashAttention MLA prefill backend."
        )
        return MLAPrefillBackendEnum.FLASH_ATTN.get_class()

    attention_config = vllm_config.attention_config

    selector_config = MLAPrefillSelectorConfig(
        dtype=vllm_config.model_config.dtype,
        is_r1_compatible=is_deepseek_r1_mla_compatible(vllm_config),
    )

    if attention_config.mla_prefill_backend is not None:
        selected_backend = attention_config.mla_prefill_backend
        backend_cls: type[MLAPrefillBackend] | None = None
        try:
            backend_cls = selected_backend.get_class()
            invalid_reasons = backend_cls.validate_configuration(
                device_capability, selector_config
            )
        except ImportError:
            invalid_reasons = ["ImportError"]
        if invalid_reasons:
            raise ValueError(
                f"Selected MLA prefill backend {selected_backend.name} "
                f"is not valid for this configuration. "
                f"Reason: {invalid_reasons}"
            )
        assert backend_cls is not None
        logger.info("Using %s MLA prefill backend.", selected_backend.name)
        return backend_cls

    return _auto_select_mla_prefill_backend(
        device_capability,
        selector_config,
    )
```
**EN:** This function returns or derives a value within the module. The docstring frames it as: Select the MLA prefill backend based on configuration and device. Key calls include `get_device_capability`, `MLAPrefillSelectorConfig`, `_auto_select_mla_prefill_backend`, `info_once`, `get_class`, `info`. The control flow contains 4 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `get_device_capability`, `MLAPrefillSelectorConfig`, `_auto_select_mla_prefill_backend`, `info_once`, `get_class`, `info`。 控制流包含 4 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_auto_select_mla_prefill_backend` function / `_auto_select_mla_prefill_backend` 函数
```python
@cache
def _auto_select_mla_prefill_backend(
    device_capability: DeviceCapability,
    selector_config: MLAPrefillSelectorConfig,
) -> "type[MLAPrefillBackend]":
    """Auto-select the best available MLA prefill backend.

    Args:
        device_capability: The device's compute capability.
        selector_config: Hashable configuration for backend selection.

    Returns:
        The selected prefill backend class.
    """
    priorities = _get_mla_prefill_backend_priorities(device_capability)
    all_invalid_reasons: dict[str, list[str]] = {}

    for backend_enum in priorities:
        backend_cls: type[MLAPrefillBackend] | None = None
        try:
            backend_cls = backend_enum.get_class()
            invalid_reasons = backend_cls.validate_configuration(
                device_capability, selector_config
            )
        except ImportError:
            invalid_reasons = ["ImportError"]
        if not invalid_reasons:
            assert backend_cls is not None
            logger.info_once("Using %s MLA prefill backend.", backend_enum.name)
            return backend_cls
        all_invalid_reasons[backend_enum.name] = invalid_reasons

    reasons_str = (
        "{"
        + ", ".join(
            f"{name}: [{', '.join(reasons)}]"
            for name, reasons in all_invalid_reasons.items()
        )
        + "}"
    )
    config_str = repr(selector_config)
    logger.debug_once(
        "Some MLA prefill backends are not valid with %s. Reasons: %s.",
        config_str,
        reasons_str,
    )

    raise ValueError(
        f"No valid MLA prefill backend found with {config_str}. Reasons: {reasons_str}."
    )
```
**EN:** This function implements `_auto_select_mla_prefill_backend` within the module. The docstring frames it as: Auto-select the best available MLA prefill backend. Key calls include `_get_mla_prefill_backend_priorities`, `repr`, `debug_once`, `ValueError`, `get_class`, `validate_configuration`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_auto_select_mla_prefill_backend`，其作用域位于the module。 关键调用包括 `_get_mla_prefill_backend_priorities`, `repr`, `debug_once`, `ValueError`, `get_class`, `validate_configuration`。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `MLAPrefillSelectorConfig`: central class or interface in this module. / `MLAPrefillSelectorConfig`：本模块中的核心类或接口。
- `is_deepseek_r1_mla_compatible`: top-level helper or orchestration entry point. / `is_deepseek_r1_mla_compatible`：顶层辅助函数或编排入口。
- `_get_mla_prefill_backend_priorities`: top-level helper or orchestration entry point. / `_get_mla_prefill_backend_priorities`：顶层辅助函数或编排入口。
- `get_mla_prefill_backend`: top-level helper or orchestration entry point. / `get_mla_prefill_backend`：顶层辅助函数或编排入口。
- `_auto_select_mla_prefill_backend`: top-level helper or orchestration entry point. / `_auto_select_mla_prefill_backend`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `functools`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.logger`, `vllm.platforms.interface`, `vllm.v1.attention.backends.mla.prefill.registry`, `vllm.config`, `vllm.v1.attention.backends.mla.prefill.base`, `vllm.platforms`
