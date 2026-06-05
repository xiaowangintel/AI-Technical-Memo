# linear_attn_model_registry.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/linear_attn_model_registry.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module provides core runtime configuration logic for linear attn model registry, including shared helpers, loading paths, or registry behavior. / 该模块提供与 linear attn model registry 相关的核心运行时配置逻辑，包括通用辅助函数、加载流程或注册表行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16: Documents the scope
```python
"""Registry for linear attention hybrid models (softmax + linear attention).

External models can register themselves without modifying SGLang core files:

    from sglang.srt.configs.linear_attn_model_registry import (
        register_linear_attn_model, LinearAttnModelSpec,
    )

    register_linear_attn_model(LinearAttnModelSpec(
        config_class=MyLinearAttnConfig,
        backend_class_name="sglang.srt.layers.attention.linear.kda_backend.KDAAttnBackend",
        arch_names=["MyLinearAttnForCausalLM"],
        uses_mamba_radix_cache=True,
        support_mamba_cache=True,
    ))
"""
```
**EN:** This string literal serves as documentation for the module, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 模块 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 17-17: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 18-23: Imports dependencies
```python
from __future__ import annotations

import importlib
import logging
from dataclasses import dataclass, field
from typing import Any, Optional
```
**EN:** This block groups related imports for the module, including __future__.annotations, importlib, logging, dataclasses.dataclass, dataclasses.field, and 2 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 __future__.annotations, importlib, logging, dataclasses.dataclass, dataclasses.field 等 2 项，为后续代码准备所需名称。

### Lines 24-24: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 25-25: Declares logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This statement initializes logger in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 logger。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 26-27: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 28-29: Declares class LinearAttnModelSpec
```python
@dataclass
class LinearAttnModelSpec:
```
**EN:** This block introduces class `LinearAttnModelSpec` as a reusable abstraction inside the module. Specification for a hybrid (softmax + linear attention) model.
**CN:** 该代码块声明类 `LinearAttnModelSpec`，作为模块中的可复用抽象。 文档字符串摘要：Specification for a hybrid (softmax + linear attention) model.

### Lines 30-30: Documents the scope
```python
    """Specification for a hybrid (softmax + linear attention) model."""
```
**EN:** This string literal serves as documentation for the LinearAttnModelSpec, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 LinearAttnModelSpec 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 31-31: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the LinearAttnModelSpec, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 LinearAttnModelSpec 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 32-38: Declares config_class, backend_class_name, arch_names, uses_mamba_radix_cache, support_mamba_cache, and 2 more
```python
    config_class: type
    backend_class_name: str  # fully-qualified class name, lazily imported
    arch_names: list[str] = field(default_factory=list)
    uses_mamba_radix_cache: bool = True
    support_mamba_cache: bool = True
    support_mamba_cache_extra_buffer: bool = False
    unwrap_text_config: bool = False  # call get_text_config() before isinstance check
```
**EN:** This block initializes a related set of values in the LinearAttnModelSpec, including config_class, backend_class_name, arch_names, uses_mamba_radix_cache, support_mamba_cache, and 2 more. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 LinearAttnModelSpec 中初始化一组相关值，包括 config_class, backend_class_name, arch_names, uses_mamba_radix_cache, support_mamba_cache 等 2 项。将这些赋值集中在一起有助于理解周边配置。

### Lines 39-40: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 41-41: Declares _LINEAR_ATTN_MODEL_REGISTRY
```python
_LINEAR_ATTN_MODEL_REGISTRY: list[LinearAttnModelSpec] = []
```
**EN:** This statement initializes _LINEAR_ATTN_MODEL_REGISTRY in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 _LINEAR_ATTN_MODEL_REGISTRY。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 42-43: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 44-51: Defines function register_linear_attn_model
```python
def register_linear_attn_model(spec: LinearAttnModelSpec) -> None:
    _LINEAR_ATTN_MODEL_REGISTRY.append(spec)
    logger.info(
        "Registered linear attn model: config=%s, backend=%s, archs=%s",
        spec.config_class.__name__,
        spec.backend_class_name.rsplit(".", 1)[-1],
        spec.arch_names,
    )
```
**EN:** This block defines function `register_linear_attn_model`. Parameters: spec.
**CN:** 该代码块定义函数 `register_linear_attn_model`。 参数包括 spec。

### Lines 52-53: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 54-59: Defines function get_linear_attn_config
```python
def get_linear_attn_config(hf_config: Any) -> Optional[tuple[LinearAttnModelSpec, Any]]:
    for spec in _LINEAR_ATTN_MODEL_REGISTRY:
        config = hf_config.get_text_config() if spec.unwrap_text_config else hf_config
        if isinstance(config, spec.config_class):
            return spec, config
    return None
```
**EN:** This block defines function `get_linear_attn_config`. Parameters: hf_config.
**CN:** 该代码块定义函数 `get_linear_attn_config`。 参数包括 hf_config。

### Lines 60-61: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 62-66: Defines function get_linear_attn_spec_by_arch
```python
def get_linear_attn_spec_by_arch(arch_name: str) -> Optional[LinearAttnModelSpec]:
    for spec in _LINEAR_ATTN_MODEL_REGISTRY:
        if arch_name in spec.arch_names:
            return spec
    return None
```
**EN:** This block defines function `get_linear_attn_spec_by_arch`. Parameters: arch_name.
**CN:** 该代码块定义函数 `get_linear_attn_spec_by_arch`。 参数包括 arch_name。

### Lines 67-68: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 69-72: Defines function import_backend_class
```python
def import_backend_class(dotted_name: str) -> type:
    module_path, class_name = dotted_name.rsplit(".", 1)
    module = importlib.import_module(module_path)
    return getattr(module, class_name)
```
**EN:** This block defines function `import_backend_class`. Parameters: dotted_name.
**CN:** 该代码块定义函数 `import_backend_class`。 参数包括 dotted_name。

## Key Concepts / 关键概念
- **Classes / 类**: `LinearAttnModelSpec`
- **Functions / 函数**: `register_linear_attn_model`, `get_linear_attn_config`, `get_linear_attn_spec_by_arch`, `import_backend_class`
- **Constants / 常量**: `_LINEAR_ATTN_MODEL_REGISTRY`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `__future__`, `dataclasses`, `importlib`, `logging`, `typing`
