# registry.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/registry.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for registry within the multimodal runtime. Key symbols include `_parse_aliases_from_ast`, `_discover_and_register_models`, `_ModelInfo`. / 该模块实现多模态运行时中与 registry 相关的模型构件。 关键符号包括 `_parse_aliases_from_ast`, `_discover_and_register_models`, `_ModelInfo`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-41: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
# Adapted from vllm: https://github.com/vllm-project/vllm/blob/v0.7.3/vllm/model_executor/models/registry.py

import ast
import importlib
import os
import pickle
import subprocess
import sys
import tempfile
from abc import ABC, abstractmethod
from collections.abc import Callable, Set
# ...
}

# Global alias mapping: external_path -> canonical_class_name
_ALIAS_TO_MODEL: dict[str, str] = {}
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 44-51: Function `_parse_aliases_from_ast` / 函数 `_parse_aliases_from_ast`
```python
def _parse_aliases_from_ast(value_node: ast.expr) -> list[str]:
    """Parse _aliases list from AST node."""
    aliases = []
    if isinstance(value_node, (ast.List, ast.Tuple)):
        for elt in value_node.elts:
            if isinstance(elt, ast.Constant) and isinstance(elt.value, str):
                aliases.append(elt.value)
    return aliases
```
**EN:** This function drives `_parse_aliases_from_ast` with inputs such as `value_node`. Parse _aliases list from AST node.
**CN:** 这个函数负责 `_parse_aliases_from_ast`，主要处理 `value_node` 等输入。 文档字符串说明：Parse _aliases list from AST node.

### Lines 54-151: Function `_discover_and_register_models` / 函数 `_discover_and_register_models`
```python
@lru_cache(maxsize=None)
def _discover_and_register_models() -> dict[str, tuple[str, str, str]]:
    discovered_models = dict(_IMAGE_ENCODER_MODELS)

    # Collect class definitions with their _aliases
    class_aliases: dict[str, list[str]] = {}

    for component in COMPONENT_DIRS:
        component_path = os.path.join(MODELS_PATH, component)
        for filename in os.listdir(component_path):
            if not filename.endswith(".py"):
                continue

            mod_relname = filename[:-3]
# ...
                )
            _ALIAS_TO_MODEL[alias] = class_name

    return discovered_models
```
**EN:** This function drives `_discover_and_register_models`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_discover_and_register_models`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 152-162: Registration and exports / 注册与导出
```python


_SGLANG_DIFFUSION_MODELS = _discover_and_register_models()

_SUBPROCESS_COMMAND = [
    sys.executable,
    "-m",
    "sglang.multimodal_gen.runtime.models.dits.registry",
]

_T = TypeVar("_T")
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 165-173: Class `_ModelInfo` / 类 `_ModelInfo`
```python
@dataclass(frozen=True)
class _ModelInfo:
    architecture: str

    @staticmethod
    def from_model_cls(model: type[nn.Module]) -> "_ModelInfo":
        return _ModelInfo(
            architecture=model.__name__,
        )
```
**EN:** This class models `_ModelInfo`. Important methods include `from_model_cls`.
**CN:** 该类实现 `_ModelInfo`。 其中较重要的方法包括 `from_model_cls`。

### Lines 176-184: Class `_BaseRegisteredModel` / 类 `_BaseRegisteredModel`
```python
class _BaseRegisteredModel(ABC):

    @abstractmethod
    def inspect_model_cls(self) -> _ModelInfo:
        raise NotImplementedError

    @abstractmethod
    def load_model_cls(self) -> type[nn.Module]:
        raise NotImplementedError
```
**EN:** This class models `_BaseRegisteredModel` as a specialization of `ABC`. Important methods include `inspect_model_cls`, `load_model_cls`.
**CN:** 该类实现 `_BaseRegisteredModel`，并继承/扩展 `ABC`。 其中较重要的方法包括 `inspect_model_cls`, `load_model_cls`。

### Lines 187-207: Class `_RegisteredModel` / 类 `_RegisteredModel`
```python
@dataclass(frozen=True)
class _RegisteredModel(_BaseRegisteredModel):
    """
    Represents a model that has already been imported in the main process.
    """

    interfaces: _ModelInfo
    model_cls: type[nn.Module]

    @staticmethod
    def from_model_cls(model_cls: type[nn.Module]):
        return _RegisteredModel(
            interfaces=_ModelInfo.from_model_cls(model_cls),
            model_cls=model_cls,
# ...
        return self.interfaces

    def load_model_cls(self) -> type[nn.Module]:
        return self.model_cls
```
**EN:** This class models `_RegisteredModel` as a specialization of `_BaseRegisteredModel`. Represents a model that has already been imported in the main process. Important methods include `from_model_cls`, `inspect_model_cls`, `load_model_cls`.
**CN:** 该类实现 `_RegisteredModel`，并继承/扩展 `_BaseRegisteredModel`。 文档字符串指出：Represents a model that has already been imported in the main process. 其中较重要的方法包括 `from_model_cls`, `inspect_model_cls`, `load_model_cls`。

### Lines 210-235: Function `_run_in_subprocess` / 函数 `_run_in_subprocess`
```python
def _run_in_subprocess(fn: Callable[[], _T]) -> _T:
    # NOTE: We use a temporary directory instead of a temporary file to avoid
    # issues like https://stackoverflow.com/questions/23212435/permission-denied-to-write-to-my-temporary-file
    with tempfile.TemporaryDirectory() as tempdir:
        output_filepath = os.path.join(tempdir, "registry_output.tmp")

        # `cloudpickle` allows pickling lambda functions directly
        input_bytes = cloudpickle.dumps((fn, output_filepath))

        # cannot use `sys.executable __file__` here because the script
        # contains relative imports
        returned = subprocess.run(
            _SUBPROCESS_COMMAND, input=input_bytes, capture_output=True
        )
# ...
            ) from e

        with open(output_filepath, "rb") as f:
            return cast(_T, pickle.load(f))
```
**EN:** This function drives `_run_in_subprocess` with inputs such as `fn`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_run_in_subprocess`，主要处理 `fn` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 238-256: Class `_LazyRegisteredModel` / 类 `_LazyRegisteredModel`
```python
@dataclass(frozen=True)
class _LazyRegisteredModel(_BaseRegisteredModel):
    """
    Represents a model that has not been imported in the main process.
    """

    module_name: str
    component_name: str
    class_name: str

    # Performed in another process to avoid initializing CUDA
    def inspect_model_cls(self) -> _ModelInfo:
        return _run_in_subprocess(
            lambda: _ModelInfo.from_model_cls(self.load_model_cls())
        )

    def load_model_cls(self) -> type[nn.Module]:
        mod = importlib.import_module(self.module_name)
        return cast(type[nn.Module], getattr(mod, self.class_name))
```
**EN:** This class models `_LazyRegisteredModel` as a specialization of `_BaseRegisteredModel`. Represents a model that has not been imported in the main process. Important methods include `inspect_model_cls`, `load_model_cls`.
**CN:** 该类实现 `_LazyRegisteredModel`，并继承/扩展 `_BaseRegisteredModel`。 文档字符串指出：Represents a model that has not been imported in the main process. 其中较重要的方法包括 `inspect_model_cls`, `load_model_cls`。

### Lines 259-271: Function `_try_load_model_cls` / 函数 `_try_load_model_cls`
```python
@lru_cache(maxsize=128)
def _try_load_model_cls(
    model_arch: str,
    model: _BaseRegisteredModel,
) -> type[nn.Module] | None:
    from sglang.multimodal_gen.runtime.platforms import current_platform

    current_platform.verify_model_arch(model_arch)
    try:
        return model.load_model_cls()
    except Exception:
        logger.exception("Ignore import error when loading '%s'", model_arch)
        return None
```
**EN:** This function drives `_try_load_model_cls` with inputs such as `model_arch`, `model`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_try_load_model_cls`，主要处理 `model_arch`, `model` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 274-283: Function `_try_inspect_model_cls` / 函数 `_try_inspect_model_cls`
```python
@lru_cache(maxsize=128)
def _try_inspect_model_cls(
    model_arch: str,
    model: _BaseRegisteredModel,
) -> _ModelInfo | None:
    try:
        return model.inspect_model_cls()
    except Exception:
        logger.exception("Error in inspecting model architecture '%s'", model_arch)
        return None
```
**EN:** This function drives `_try_inspect_model_cls` with inputs such as `model_arch`, `model`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_try_inspect_model_cls`，主要处理 `model_arch`, `model` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 286-406: Class `_ModelRegistry` / 类 `_ModelRegistry`
```python
@dataclass
class _ModelRegistry:
    # Keyed by model_arch
    registered_models: dict[str, _BaseRegisteredModel] = field(default_factory=dict)

    def get_supported_archs(self) -> Set[str]:
        return self.registered_models.keys()

    def resolve_by_alias(self, alias: str) -> type[nn.Module] | None:
        """Resolve a model class by its alias (external module path)."""
        if alias in _ALIAS_TO_MODEL:
            canonical_name = _ALIAS_TO_MODEL[alias]
            return self._try_load_model_cls(canonical_name)
        return None
# ...
            if model_cls is not None:
                return (model_cls, arch)

        return self._raise_for_unsupported(architectures)
```
**EN:** This class models `_ModelRegistry`. Important methods include `get_supported_archs`, `resolve_by_alias`, `register_model`, `_raise_for_unsupported`.
**CN:** 该类实现 `_ModelRegistry`。 其中较重要的方法包括 `get_supported_archs`, `resolve_by_alias`, `register_model`, `_raise_for_unsupported`。

### Lines 407-422: Registration and exports / 注册与导出
```python


ModelRegistry = _ModelRegistry(
    {
        model_arch: _LazyRegisteredModel(
            module_name=f"sglang.multimodal_gen.runtime.models.{component_name}.{mod_relname}",
            component_name=component_name,
            class_name=cls_name,
        )
        for model_arch, (
            component_name,
            mod_relname,
            cls_name,
        ) in _SGLANG_DIFFUSION_MODELS.items()
    }
)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Caching strategy / 缓存策略
- Command-line interface / 命令行接口
- Image generation flow / 图像生成流程
- Video generation flow / 视频生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.runtime.platforms`
- **External / 外部**: `ast`, `cloudpickle`, `torch`
- **Stdlib / 标准库**: `importlib`, `os`, `pickle`, `subprocess`, `sys`, `tempfile`, `abc`, `collections.abc`
