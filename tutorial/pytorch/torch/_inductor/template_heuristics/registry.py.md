# registry.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/template_heuristics/registry.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It exposes functions such as `register_template_heuristic`, `get_template_heuristic`, `get_registered_heuristic_class`, `clear_registry`, and `override_template_heuristics`. Module note: Template heuristic registry system for PyTorch Inductor.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。同时提供 `register_template_heuristic`、`get_template_heuristic`、`get_registered_heuristic_class`、`clear_registry`、`override_template_heuristics` 等函数。模块文档字符串给出了额外背景说明。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
"""
Template heuristic registry system for PyTorch Inductor.

This module provides a centralized registration system for template heuristics,
allowing automatic registration based on device type and conditional registration
for CUDA vs ROCm based on torch.version.hip.
"""

from __future__ import annotations

import contextlib
import logging
from typing import Any, TYPE_CHECKING

````
- **EN**: Imports dependencies such as `__future__`, `contextlib`, `logging`, and `typing` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里导入了 `__future__`、`contextlib`、`logging`、`typing` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 15-28 / 第 15-28 行
````python
from .base import TemplateConfigHeuristics


if TYPE_CHECKING:
    from collections.abc import Iterator


# Module-wide registry for template heuristics
_TEMPLATE_HEURISTIC_REGISTRY: dict[
    tuple[str | None, ...], type[TemplateConfigHeuristics]
] = {}

# Manual cache for successful lookups only (fallback instances are not cached)
_HEURISTIC_CACHE: dict[tuple[str, str, str], TemplateConfigHeuristics] = {}
````
- **EN**: Imports dependencies such as `.base`, and `collections.abc` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里导入了 `.base`、`collections.abc` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 29-42 / 第 29-42 行
````python

log = logging.getLogger(__name__)


def register_template_heuristic(
    template_name: str,
    device_type: str | None,
    register: bool = True,
    op_name: str | None = None,
) -> Any:
    """
    Decorator to register template heuristic classes.

    Args:
````
- **EN**: Introduces function `register_template_heuristic`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `log`, `template_name`, `device_type`, `register`, `op_name`, and `Args`.
- **CN**: 这里定义了函数`register_template_heuristic`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `log`、`template_name`、`device_type`、`register`、`op_name`、`Args` 等值。

### Lines 43-56 / 第 43-56 行
````python
        template_name: Name of the template (e.g., "mm", "bmm", "scaled_mm")
        device_type: Device type ("cuda", "cpu", "xpu")
            Set this to None to indicate that the heuristic is applicable to all device types.
        register: Whether to register this heuristic. Caller should pass the condition directly.
        op_name: Name of the operator (e.g., "mm", "bmm", "scaled_mm"). This is optional
            and is only used when a template uses different heuristics for different ops

    Returns:
        Decorator function that registers the class if conditions are met.

    Example:
        @register_template_heuristic("mm", "cuda", register=torch.version.hip is None)
        class CUDAMMTemplateConfigHeuristic(MMTemplateConfigMixin, CUDAConfigHeuristic):
            pass
````
- **EN**: Introduces class `CUDAMMTemplateConfigHeuristic`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了类`CUDAMMTemplateConfigHeuristic`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 57-70 / 第 57-70 行
````python
    """

    def decorator(
        cls: type[TemplateConfigHeuristics],
    ) -> type[TemplateConfigHeuristics]:
        if register:
            key: tuple[str | None, ...] = (template_name, device_type, op_name)
            _TEMPLATE_HEURISTIC_REGISTRY[key] = cls
            log.info(
                f"Registered template heuristic: {cls.__name__} for '{template_name=}', '{device_type=}', '{op_name=}'"  # noqa: G004
            )
        return cls

    return decorator
````
- **EN**: Introduces function `decorator`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`decorator`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 71-84 / 第 71-84 行
````python


def get_template_heuristic(
    template_name: str, device_type: str, op_name: str
) -> TemplateConfigHeuristics:
    """
    Retrieve a template heuristic instance for the given template and device type.

    Args:
        template_name: Name of the template (e.g., "mm", "bmm", "scaled_mm")
        device_type: Device type ("cuda", "cpu", "xpu")
        op_name: Name of the operator (e.g., "mm", "bmm", "scaled_mm")

    Returns:
````
- **EN**: Introduces function `get_template_heuristic`. Handles device-specific behavior, capabilities, or dispatch decisions. Initializes or updates values such as `template_name`, `Args`, `device_type`, `op_name`, and `Returns`.
- **CN**: 这里定义了函数`get_template_heuristic`。处理与设备相关的行为、能力信息或派发决策。初始化或更新了 `template_name`、`Args`、`device_type`、`op_name`、`Returns` 等值。

### Lines 85-98 / 第 85-98 行
````python
        Template heuristic instance. If no specific heuristic is found,
        returns a fallback TemplateConfigHeuristics() instance (uncached).
    """
    # Check cache first
    cache_key = (template_name, device_type, op_name)
    if cache_key in _HEURISTIC_CACHE:
        return _HEURISTIC_CACHE[cache_key]

    heuristic_class = get_registered_heuristic_class(
        template_name, device_type, op_name
    )

    if heuristic_class is None:
        # Log error and return fallback instance (uncached)
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 99-112 / 第 99-112 行
````python
        log.error(
            "No template heuristic found - template_name=%s, device_type=%s, op_name=%s. "
            "Available combinations: %s. Using fallback TemplateConfigHeuristics instance.",
            template_name,
            device_type,
            op_name,
            list(_TEMPLATE_HEURISTIC_REGISTRY.keys()),
        )
        return TemplateConfigHeuristics()

    # Cache successful lookup and return
    instance = heuristic_class()
    _HEURISTIC_CACHE[cache_key] = instance
    return instance
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `instance`. This range continues the implementation of function `get_template_heuristic`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `instance` 等值。这一段延续了函数`get_template_heuristic` 的具体实现。

### Lines 113-126 / 第 113-126 行
````python


def get_registered_heuristic_class(
    template_name: str, device_type: str, op_name: str
) -> None | type[TemplateConfigHeuristics]:
    """
    Get the heuristic class registered for the given template/device/op combination.

    This is useful for creating custom heuristics that subclass the appropriate
    base class for a given template/device/op combination.

    Args:
        template_name: Name of the template (e.g., "mm", "bmm", "scaled_mm")
        device_type: Device type ("cuda", "cpu", "xpu")
````
- **EN**: Introduces function `get_registered_heuristic_class`. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `template_name`, `Args`, and `device_type`.
- **CN**: 这里定义了函数`get_registered_heuristic_class`。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `template_name`、`Args`、`device_type` 等值。

### Lines 127-140 / 第 127-140 行
````python
        op_name: Name of the operator (e.g., "mm", "bmm", "scaled_mm")

    Returns:
        The heuristic class if found, None otherwise.
    """
    keys = [
        # everything is specified
        (template_name, device_type, op_name),
        # heuristic is valid across all devices
        (template_name, None, op_name),
        # heuristic is valid across all ops for that device
        (template_name, device_type, None),
        # heuristic is always valid for that template
        (template_name, None, None),
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Initializes or updates values such as `op_name`, `Returns`, and `keys`. This range continues the implementation of function `get_registered_heuristic_class`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。初始化或更新了 `op_name`、`Returns`、`keys` 等值。这一段延续了函数`get_registered_heuristic_class` 的具体实现。

### Lines 141-154 / 第 141-154 行
````python
    ]
    for key in keys:
        if key in _TEMPLATE_HEURISTIC_REGISTRY:
            return _TEMPLATE_HEURISTIC_REGISTRY[key]

    return None


def clear_registry() -> None:
    """
    Clear all registered template heuristics.

    This is primarily useful for testing purposes to ensure a clean state.
    """
````
- **EN**: Introduces function `clear_registry`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`clear_registry`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 155-168 / 第 155-168 行
````python
    _TEMPLATE_HEURISTIC_REGISTRY.clear()
    _HEURISTIC_CACHE.clear()


@contextlib.contextmanager
def override_template_heuristics(
    device_type: str,
    template_op_pairs: list[tuple[str, str]],
    override_heuristic_class: type[TemplateConfigHeuristics] = TemplateConfigHeuristics,
) -> Iterator[None]:
    """
    Context manager to temporarily override template heuristics.

    This is useful for testing purposes, where we want to ensure a specific template/op pair
````
- **EN**: Introduces function `override_template_heuristics`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Initializes or updates values such as `device_type`, `template_op_pairs`, and `override_heuristic_class`.
- **CN**: 这里定义了函数`override_template_heuristics`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。初始化或更新了 `device_type`、`template_op_pairs`、`override_heuristic_class` 等值。

### Lines 169-182 / 第 169-182 行
````python
    uses a custom heuristic or returns no entries.

    Args:
        device_type: Device type ("cuda", "cpu", "xpu")
        template_op_pairs: List of (template_name, op_name) pairs to override.
        override_heuristic_class: Heuristic class to use for the override.
            Defaults to TemplateConfigHeuristics (which returns no entries).
    """
    # Save original entries to restore later
    original_entries = {}
    new_keys = []
    _HEURISTIC_CACHE.clear()
    try:
        for template_name, op_name in template_op_pairs:
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Args`, `device_type`, `template_op_pairs`, `override_heuristic_class`, `original_entries`, `new_keys`, and `...+1`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `Args`、`device_type`、`template_op_pairs`、`override_heuristic_class`、`original_entries`、`new_keys`、`另有1项` 等值。

### Lines 183-196 / 第 183-196 行
````python
            assert op_name is not None
            key = (template_name, device_type, op_name)
            if key in _TEMPLATE_HEURISTIC_REGISTRY:
                original_entries[key] = _TEMPLATE_HEURISTIC_REGISTRY[key]
            _TEMPLATE_HEURISTIC_REGISTRY[key] = override_heuristic_class
            new_keys.append(key)
        yield
    finally:
        # Restore original entries or remove if they didn't exist before
        for key in new_keys:
            _TEMPLATE_HEURISTIC_REGISTRY.pop(key, None)
            if key in original_entries:
                _TEMPLATE_HEURISTIC_REGISTRY[key] = original_entries[key]
        _HEURISTIC_CACHE.clear()
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `key`, and `finally`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `key`、`finally` 等值。

## Key Concepts / 关键概念
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Primary functions: `register_template_heuristic`, `get_template_heuristic`, `get_registered_heuristic_class`, `clear_registry`, and `override_template_heuristics`  
  **CN**: 主要函数：`register_template_heuristic`、`get_template_heuristic`、`get_registered_heuristic_class`、`clear_registry`、`override_template_heuristics`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `contextlib`, `logging`, `typing`, `collections.abc`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `.base`
