# registration.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/onnx/_internal/torchscript_exporter/registration.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal ONNX exporter logic for registration, covering graph capture, translation, validation, or serialization.
- 用途 (CN): 实现与 registration 相关的 ONNX 导出器内部逻辑，涵盖图捕获、转换、校验或序列化。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```python
# mypy: allow-untyped-defs
"""Module for handling symbolic function registration."""

import warnings
from collections.abc import Callable, Collection, Sequence
from typing import Generic, TypeVar
from typing_extensions import ParamSpec
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: none; external imports: `warnings`, `collections.abc`, `typing`, `typing_extensions`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：无；外部导入：`warnings`, `collections.abc`, `typing`, `typing_extensions`。

### Lines 8-14
```python

from torch.onnx import _constants, errors


OpsetVersion = int


```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch.onnx`; external imports: none.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch.onnx`；外部导入：无。

### Lines 15-23
```python
def _dispatch_opset_version(
    target: OpsetVersion, registered_opsets: Collection[OpsetVersion]
) -> OpsetVersion | None:
    """Finds the registered opset given a target opset version and the available opsets.

    Args:
        target: The target opset version.
        registered_opsets: The available opsets.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_dispatch_opset_version`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_dispatch_opset_version`。

### Lines 24-33
```python
    Returns:
        The registered opset version.
    """
    if not registered_opsets:
        return None

    descending_registered_versions = sorted(registered_opsets, reverse=True)
    # Linear search for the opset version, which is fine since the number of opset
    # versions is small.

```
- EN: This block advances ONNX export translation; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 34-40
```python
    if target >= _constants.ONNX_BASE_OPSET:
        # Always look down toward opset 1 when the target is >= ONNX_BASE_OPSET (opset 9).
        # When a custom op is register at opset 1, we want to be able to discover it as a
        # fallback for all opsets >= ONNX_BASE_OPSET.
        for version in descending_registered_versions:
            if version <= target:
                return version
```
- EN: This block advances ONNX export translation; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 41-47
```python
        return None

    # target < opset 9. This is the legacy behavior to support opset 7 and opset 8.
    # for caffe2 support. We search up toward opset 9.
    for version in reversed(descending_registered_versions):
        # Count back up until _constants.ONNX_BASE_OPSET
        if target <= version <= _constants.ONNX_BASE_OPSET:
```
- EN: This block advances ONNX export translation; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 48-57
```python
            return version

    return None


_K = TypeVar("_K")
_V = TypeVar("_V")
_R = TypeVar("_R")
_P = ParamSpec("_P")

```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 58-65
```python

class OverrideDict(Collection[_K], Generic[_K, _V]):
    """A dictionary that merges built-in and custom symbolic functions.

    It supports overriding and un-overriding built-in symbolic functions with custom
    ones.
    """

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `OverrideDict`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`OverrideDict`。

### Lines 66-73
```python
    def __init__(self) -> None:
        self._base: dict[_K, _V] = {}
        self._overrides: dict[_K, _V] = {}
        self._merged: dict[_K, _V] = {}

    def set_base(self, key: _K, value: _V) -> None:
        self._base[key] = value
        if key not in self._overrides:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__init__`, `set_base`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__init__`, `set_base`。

### Lines 74-80
```python
            self._merged[key] = value

    def in_base(self, key: _K) -> bool:
        """Checks if a key is in the base dictionary."""
        return key in self._base

    def override(self, key: _K, value: _V) -> None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `in_base`, `override`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`in_base`, `override`。

### Lines 81-89
```python
        """Overrides a base key-value with a new pair."""
        self._overrides[key] = value
        self._merged[key] = value

    def remove_override(self, key: _K) -> None:
        """Un-overrides a key-value pair."""
        self._overrides.pop(key, None)  # type: ignore[arg-type]
        self._merged.pop(key, None)  # type: ignore[arg-type]
        if key in self._base:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `remove_override`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`remove_override`。

### Lines 90-96
```python
            self._merged[key] = self._base[key]

    def overridden(self, key: _K) -> bool:
        """Checks if a key-value pair is overridden."""
        return key in self._overrides

    def __getitem__(self, key: _K) -> _V:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `overridden`, `__getitem__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`overridden`, `__getitem__`。

### Lines 97-103
```python
        return self._merged[key]

    def get(self, key: _K, default: _V | None = None):
        return self._merged.get(key, default)

    def __contains__(self, key: object) -> bool:
        return key in self._merged
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `get`, `__contains__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`get`, `__contains__`。

### Lines 104-110
```python

    def __iter__(self):
        return iter(self._merged)

    def __len__(self) -> int:
        return len(self._merged)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__iter__`, `__len__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__iter__`, `__len__`。

### Lines 111-117
```python
    def __repr__(self) -> str:
        return f"OverrideDict(base={self._base}, overrides={self._overrides})"

    def __bool__(self) -> bool:
        return bool(self._merged)


```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__repr__`, `__bool__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__repr__`, `__bool__`。

### Lines 118-125
```python
class _SymbolicFunctionGroup:
    """Different versions of symbolic functions registered to the same name.

    O(number of registered versions of an op) search is performed to find the most
    recent version of the op.

    The registration is delayed until op is used to improve startup time.

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `_SymbolicFunctionGroup`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`_SymbolicFunctionGroup`。

### Lines 126-134
```python
    Function overloads with different arguments are not allowed.
    Custom op overrides are supported.
    """

    def __init__(self, name: str) -> None:
        self._name = name
        # A dictionary of functions, keyed by the opset version.
        self._functions: OverrideDict[OpsetVersion, Callable] = OverrideDict()

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__init__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__init__`。

### Lines 135-142
```python
    def __repr__(self) -> str:
        return f"_SymbolicFunctionGroup({self._name}, registered={self._functions})"

    def __getitem__(self, key: OpsetVersion) -> Callable:
        result = self.get(key)
        if result is None:
            raise KeyError(key)
        return result
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__repr__`, `__getitem__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__repr__`, `__getitem__`。

### Lines 143-149
```python

    # TODO(justinchuby): Add @functools.lru_cache(maxsize=None) if lookup time becomes
    # a problem.
    def get(self, opset: OpsetVersion) -> Callable | None:
        """Find the most recent version of the function."""
        version = _dispatch_opset_version(opset, self._functions)
        if version is None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `get`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`get`。

### Lines 150-156
```python
            return None

        return self._functions[version]

    def add(self, func: Callable, opset: OpsetVersion) -> None:
        """Adds a symbolic function.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `add`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`add`。

### Lines 157-170
```python
        Args:
            func: The function to add.
            opset: The opset version of the function to add.
        """
        if self._functions.in_base(opset):
            warnings.warn(
                f"Symbolic function '{self._name}' already registered for opset {opset}. "
                f"Replacing the existing function with new function. This is unexpected. "
                f"Please report it on {_constants.PYTORCH_GITHUB_ISSUES_URL}.",
                errors.OnnxExporterWarning,
                stacklevel=2,
            )
        self._functions.set_base(opset, func)

```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 171-179
```python
    def add_custom(self, func: Callable, opset: OpsetVersion) -> None:
        """Adds a custom symbolic function.

        Args:
            func: The symbolic function to register.
            opset: The corresponding opset version.
        """
        self._functions.override(opset, func)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `add_custom`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`add_custom`。

### Lines 180-186
```python
    def remove_custom(self, opset: OpsetVersion) -> None:
        """Removes a custom symbolic function.

        Args:
            opset: The opset version of the custom function to remove.
        """
        if not self._functions.overridden(opset):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `remove_custom`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`remove_custom`。

### Lines 187-193
```python
            warnings.warn(
                f"No custom function registered for '{self._name}' opset {opset}",
                stacklevel=2,
            )
            return
        self._functions.remove_override(opset)

```
- EN: This block advances ONNX export translation; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 194-201
```python
    def get_min_supported(self) -> OpsetVersion:
        """Returns the lowest built-in opset version supported by the function."""
        return min(self._functions)


class SymbolicRegistry:
    """Registry for symbolic functions.

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `get_min_supported`, `SymbolicRegistry`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`get_min_supported`, `SymbolicRegistry`。

### Lines 202-209
```python
    The registry maintains a mapping from qualified names to symbolic functions.
    It is used to register new symbolic functions and to dispatch calls to
    the appropriate function.
    """

    def __init__(self) -> None:
        self._registry: dict[str, _SymbolicFunctionGroup] = {}

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__init__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__init__`。

### Lines 210-221
```python
    def register(
        self, name: str, opset: OpsetVersion, func: Callable, custom: bool = False
    ) -> None:
        """Registers a symbolic function.

        Args:
            name: The qualified name of the function to register. In the form of 'domain::op'.
                E.g. 'aten::add'.
            opset: The opset version of the function to register.
            func: The symbolic function to register.
            custom: Whether the function is a custom function that overrides existing ones.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `register`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`register`。

### Lines 222-232
```python
        Raises:
            ValueError: If the separator '::' is not in the name.
        """
        if "::" not in name:
            raise ValueError(
                f"The name must be in the form of 'domain::op', not '{name}'"
            )
        symbolic_functions = self._registry.setdefault(
            name, _SymbolicFunctionGroup(name)
        )
        if custom:
```
- EN: This block advances ONNX export translation; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 233-239
```python
            symbolic_functions.add_custom(func, opset)
        else:
            symbolic_functions.add(func, opset)

    def unregister(self, name: str, opset: OpsetVersion) -> None:
        """Unregisters a symbolic function.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `unregister`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`unregister`。

### Lines 240-247
```python
        Args:
            name: The qualified name of the function to unregister.
            opset: The opset version of the function to unregister.
        """
        if name not in self._registry:
            return
        self._registry[name].remove_custom(opset)

```
- EN: This block advances ONNX export translation; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流。关键符号：无明显局部符号。

### Lines 248-255
```python
    def get_function_group(self, name: str) -> _SymbolicFunctionGroup | None:
        """Returns the function group for the given name."""
        return self._registry.get(name)

    def is_registered_op(self, name: str, version: int) -> bool:
        """Returns whether the given op is registered for the given opset version."""
        functions = self.get_function_group(name)
        if functions is None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `get_function_group`, `is_registered_op`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`get_function_group`, `is_registered_op`。

### Lines 256-262
```python
            return False
        return functions.get(version) is not None

    def all_functions(self) -> set[str]:
        """Returns the set of all registered function names."""
        return set(self._registry)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `all_functions`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`all_functions`。

### Lines 263-271
```python

def onnx_symbolic(
    name: str,
    opset: OpsetVersion | Sequence[OpsetVersion],
    decorate: Sequence[Callable] | None = None,
    custom: bool = False,
) -> Callable:
    """Registers a symbolic function.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `onnx_symbolic`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`onnx_symbolic`。

### Lines 272-280
```python
    Usage::

    ```
    @onnx_symbolic(
        "aten::symbolic_b",
        opset=10,
        decorate=[quantized_aten_handler(scale=1 / 128, zero_point=0)],
    )
    @symbolic_helper.parse_args("v", "v", "b")
```
- EN: This block advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 281-290
```python
    def symbolic_b(g: _C.Graph, x: _C.Value, y: _C.Value, arg1: bool) -> _C.Value: ...
    ```

    Args:
        name: The qualified name of the function in the form of 'domain::op'.
            E.g. 'aten::add'.
        opset: The opset versions of the function to register at.
        decorate: A sequence of decorators to apply to the function.
        custom: Whether the function is a custom symbolic function.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `symbolic_b`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`symbolic_b`。

### Lines 291-297
```python
    Raises:
        ValueError: If the separator '::' is not in the name.
    """

    def wrapper(func: Callable[_P, _R]) -> Callable[_P, _R]:
        decorated = func
        if decorate is not None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `wrapper`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`wrapper`。

### Lines 298-305
```python
            for decorate_func in decorate:
                decorated = decorate_func(decorated)

        global registry
        nonlocal opset
        if isinstance(opset, OpsetVersion):
            opset = (opset,)
        for opset_version in opset:
```
- EN: This block advances ONNX export translation; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 306-312
```python
            registry.register(name, opset_version, decorated, custom=custom)

        # Return the original function because the decorators in "decorate" are only
        # specific to the instance being registered.
        return func

    return wrapper
```
- EN: This block advances ONNX export translation; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 313-321
```python


def custom_onnx_symbolic(
    name: str,
    opset: OpsetVersion | Sequence[OpsetVersion],
    decorate: Sequence[Callable] | None = None,
) -> Callable:
    """Registers a custom symbolic function.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `custom_onnx_symbolic`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`custom_onnx_symbolic`。

### Lines 322-329
```python
    Args:
        name: the qualified name of the function.
        opset: the opset version of the function.
        decorate: a sequence of decorators to apply to the function.

    Returns:
        The decorator.

```
- EN: This block advances ONNX export translation. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：无明显局部符号。

### Lines 330-337
```python
    Raises:
        ValueError: If the separator '::' is not in the name.
    """
    return onnx_symbolic(name, opset, decorate, custom=True)


# The registry for all symbolic functions.
registry = SymbolicRegistry()
```
- EN: This block advances ONNX export translation; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: ONNX export pipeline — the code translates PyTorch programs into ONNX-friendly representations.
  CN: ONNX 导出流水线——代码把 PyTorch 程序转换为适合 ONNX 的表示。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Interop boundary — the implementation connects PyTorch semantics with ONNX model representation requirements.
  CN: 互操作边界——实现负责连接 PyTorch 语义与 ONNX 模型表示需求。
- EN: Module integration — the implementation coordinates neighboring Python modules and utility layers.
  CN: 模块集成——该实现会协调相邻的 Python 模块与工具层。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch.onnx`
- External imports / 外部导入: `warnings`, `collections.abc`, `typing`, `typing_extensions`
- Representative symbols / 代表性符号: `_dispatch_opset_version`, `_K`, `_V`, `_R`, `_P`, `OverrideDict`, `_SymbolicFunctionGroup`, `SymbolicRegistry`, `onnx_symbolic`, `custom_onnx_symbolic`
