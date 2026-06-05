# registry.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/registry.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module builds the model registry used by SGLang SRT to discover, validate, and instantiate supported model implementations. / 该模块构建 SGLang SRT 使用的模型注册表，用于发现、校验并实例化受支持的模型实现。

## Line-by-Line Analysis / 逐行分析

### Lines 5-14: Module imports
```python
import importlib
import logging
import pkgutil
from dataclasses import dataclass, field
from functools import lru_cache
from typing import AbstractSet, Dict, List, Optional, Tuple, Type, Union

import torch.nn as nn

from sglang.srt.environ import envs
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、SGLang 运行时组件。

### Lines 16-16: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 19-21: Class `_ModelRegistry` overview
```python
@dataclass
class _ModelRegistry:
    # Keyed by model_arch
```
**EN:** Defines `_ModelRegistry` as a reusable runtime type derived from no explicit base class. The class groups 6 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `_ModelRegistry`，其继承关系为 no explicit base class。这个类组织了 6 个方法，用于实现模型相关行为。

### Lines 22-22: Class `_ModelRegistry` attributes
```python
    models: Dict[str, Union[Type[nn.Module], str]] = field(default_factory=dict)
```
**EN:** Defines class-level attributes and metadata that shape how `_ModelRegistry` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `_ModelRegistry` 在运行时的行为。

### Lines 24-36: Method `_ModelRegistry.register`
```python
    def register(
        self, package_name: str, overwrite: bool = False, strict: bool = False
    ):
        new_models = import_model_classes(package_name, strict=strict)
        if overwrite:
            self.models.update(new_models)
        else:
            for arch, cls in new_models.items():
                if arch in self.models:
                    raise ValueError(
                        f"Model architecture {arch} already registered. Set overwrite=True to replace."
                    )
                self.models[arch] = cls
```
**EN:** This method implements `register(package_name: ..., overwrite: ...=..., strict: ...=...)` and registers metadata or implementation classes into a global lookup table.
**CN:** 这个方法实现了 `register(package_name: ..., overwrite: ...=..., strict: ...=...)`，其作用是把元数据或实现类注册到全局查找表中。

### Lines 38-39: Method `_ModelRegistry.get_supported_archs`
```python
    def get_supported_archs(self) -> AbstractSet[str]:
        return self.models.keys()
```
**EN:** This method implements `get_supported_archs()` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_supported_archs()`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 41-53: Method `_ModelRegistry._raise_for_unsupported`
```python
    def _raise_for_unsupported(self, architectures: List[str]):
        all_supported_archs = self.get_supported_archs()

        if any(arch in all_supported_archs for arch in architectures):
            raise ValueError(
                f"Model architectures {architectures} failed "
                "to be inspected. Please check the logs for more details."
            )

        raise ValueError(
            f"Model architectures {architectures} are not supported for now. "
            f"Supported architectures: {all_supported_archs}"
        )
```
**EN:** This method implements `_raise_for_unsupported(architectures: ...)` and implements a focused helper that supports the surrounding runtime flow inside `_ModelRegistry`.
**CN:** 这个方法实现了 `_raise_for_unsupported(architectures: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `_ModelRegistry` 内部调用。

### Lines 55-59: Method `_ModelRegistry._try_load_model_cls`
```python
    def _try_load_model_cls(self, model_arch: str) -> Optional[Type[nn.Module]]:
        if model_arch not in self.models:
            return None

        return self.models[model_arch]
```
**EN:** This method implements `_try_load_model_cls(model_arch: ...)` and implements a focused helper that supports the surrounding runtime flow inside `_ModelRegistry`.
**CN:** 这个方法实现了 `_try_load_model_cls(model_arch: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `_ModelRegistry` 内部调用。

### Lines 61-78: Method `_ModelRegistry._normalize_archs`
```python
    def _normalize_archs(
        self,
        architectures: Union[str, List[str]],
    ) -> List[str]:
        if isinstance(architectures, str):
            architectures = [architectures]
        if not architectures:
            logger.warning("No model architectures are specified")

        # filter out support architectures
        normalized_arch = list(
            filter(lambda model: model in self.models, architectures)
        )

        # make sure Transformers backend is put at the last as a fallback
        if len(normalized_arch) != len(architectures):
            normalized_arch.append("TransformersForCausalLM")
        return normalized_arch
```
**EN:** This method implements `_normalize_archs(architectures: ...)` and implements a focused helper that supports the surrounding runtime flow inside `_ModelRegistry`.
**CN:** 这个方法实现了 `_normalize_archs(architectures: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `_ModelRegistry` 内部调用。

### Lines 80-91: Method `_ModelRegistry.resolve_model_cls`
```python
    def resolve_model_cls(
        self,
        architectures: Union[str, List[str]],
    ) -> Tuple[Type[nn.Module], str]:
        architectures = self._normalize_archs(architectures)

        for arch in architectures:
            model_cls = self._try_load_model_cls(arch)
            if model_cls is not None:
                return (model_cls, arch)

        return self._raise_for_unsupported(architectures)
```
**EN:** This method implements `resolve_model_cls(architectures: ...)` and resolves runtime selections into concrete classes or configuration values.
**CN:** 这个方法实现了 `resolve_model_cls(architectures: ...)`，其作用是把运行时选择解析为具体类或配置值。

### Lines 94-127: Function `import_model_classes`
```python
@lru_cache()
def import_model_classes(package_name: str, strict: bool = False):
    model_arch_name_to_cls = {}
    package = importlib.import_module(package_name)
    for _, name, ispkg in pkgutil.iter_modules(package.__path__, package_name + "."):
        if not ispkg:
            if name.split(".")[-1] in envs.SGLANG_DISABLED_MODEL_ARCHS.get():
                logger.debug(f"Skip loading {name} due to SGLANG_DISABLED_MODEL_ARCHS")
                continue

            try:
                module = importlib.import_module(name)
            except Exception as e:
                if strict:
                    raise
                logger.warning(f"Ignore import error when loading {name}: {e}")
                continue
            if hasattr(module, "EntryClass"):
                entry = module.EntryClass
                if isinstance(
                    entry, list
                ):  # To support multiple model classes in one module
                    for tmp in entry:
                        assert (
                            tmp.__name__ not in model_arch_name_to_cls
                        ), f"Duplicated model implementation for {tmp.__name__}"
                        model_arch_name_to_cls[tmp.__name__] = tmp
                else:
                    assert (
                        entry.__name__ not in model_arch_name_to_cls
                    ), f"Duplicated model implementation for {entry.__name__}"
                    model_arch_name_to_cls[entry.__name__] = entry

    return model_arch_name_to_cls
```
**EN:** This function implements `import_model_classes(package_name: ..., strict: ...=...)` and implements a focused helper that supports the surrounding runtime flow Decorators: lru_cache().
**CN:** 这个函数实现了 `import_model_classes(package_name: ..., strict: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑 装饰器：lru_cache()。

### Lines 130-130: Top-level assign
```python
ModelRegistry = _ModelRegistry()
```
**EN:** Defines or updates ModelRegistry, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 ModelRegistry，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 131-131: Top-level expr
```python
ModelRegistry.register("sglang.srt.models")
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 133-134: Top-level if
```python
if external_pkg := envs.SGLANG_EXTERNAL_MODEL_PACKAGE.get():
    ModelRegistry.register(external_pkg, overwrite=True)
```
**EN:** Applies conditional top-level logic so the module can adapt to optional dependencies, feature flags, or environment-specific behavior.
**CN:** 应用条件化的顶层逻辑，使模块能够适配可选依赖、功能开关或特定环境行为。

## Key Concepts / 关键概念
- **EN:** Dynamic model registration / **CN:** 动态模型注册

## Dependencies / 依赖关系
- `importlib`
- `logging`
- `pkgutil`
- `dataclasses.dataclass`
- `dataclasses.field`
- `functools.lru_cache`
- `typing.AbstractSet`
- `typing.Dict`
- `typing.List`
- `typing.Optional`
- `typing.Tuple`
- `typing.Type`
- `typing.Union`
- `torch.nn`
- `sglang.srt.environ.envs`
