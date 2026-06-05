# registry.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_dynamo/backends/registry.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Connects captured graphs to compiler backends and post-processing pipelines.
- **Purpose (CN)**: 将捕获到的图连接到编译后端及后处理流水线。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```python
"""
This module implements TorchDynamo's backend registry system for managing compiler backends.

The registry provides a centralized way to register, discover and manage different compiler
backends that can be used with torch.compile(). It handles:

- Backend registration and discovery through decorators and entry points
- Lazy loading of backend implementations
- Lookup and validation of backend names
- Categorization of backends using tags (debug, experimental, etc.)

Key components:
- CompilerFn: Type for backend compiler functions that transform FX graphs
- _BACKENDS: Registry mapping backend names to entry points
- _COMPILER_FNS: Registry mapping backend names to loaded compiler functions
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 17-32
```python
Example usage:
    @register_backend
    def my_compiler(fx_graph, example_inputs):
        # Transform FX graph into optimized implementation
        return compiled_fn

    # Use registered backend
    torch.compile(model, backend="my_compiler")

The registry also supports discovering backends through setuptools entry points
in the "torch_dynamo_backends" group. Example:
```
setup.py
---
from setuptools import setup
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 33-47
```python
setup(
    name='my_torch_backend',
    version='0.1',
    packages=['my_torch_backend'],
    entry_points={
        'torch_dynamo_backends': [
            # name = path to entry point of backend implementation
            'my_compiler = my_torch_backend.compiler:my_compiler_function',
        ],
    },
)
```
```
my_torch_backend/compiler.py
---
```
- **EN**: This module-level block helps hand work to a compiler/backend pipeline.
- **CN**: 这个模块级代码块用于将工作移交给编译器或后端流水线。

### Lines 48-60
```python
def my_compiler_function(fx_graph, example_inputs):
    # Transform FX graph into optimized implementation
    return compiled_fn
```
Using `my_compiler` backend:
```
import torch

model = ...  # Your PyTorch model
optimized_model = torch.compile(model, backend="my_compiler")
```
"""
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果。

### Lines 61-77
```python
import functools
import logging
from collections.abc import Callable, Sequence
from importlib.metadata import EntryPoint
from typing import Any, Protocol

import torch
from torch import fx


log = logging.getLogger(__name__)


class CompiledFn(Protocol):
    def __call__(self, *args: torch.Tensor) -> tuple[torch.Tensor, ...]: ...
```
- **EN**: This module-level block helps hand work to a compiler/backend pipeline.
- **CN**: 这个模块级代码块用于将工作移交给编译器或后端流水线。

### Lines 78-95
```python
CompilerFn = Callable[[fx.GraphModule, list[torch.Tensor]], CompiledFn]

_BACKENDS: dict[str, EntryPoint | None] = {}
_COMPILER_FNS: dict[str, CompilerFn] = {}
_default_backend: str | CompilerFn = "inductor"


def register_backend(
    compiler_fn: CompilerFn | None = None,
    name: str | None = None,
    tags: Sequence[str] = (),
) -> Callable[..., Any]:
    """
    Decorator to add a given compiler to the registry to allow calling
    `torch.compile` with string shorthand.  Note: for projects not
    imported by default, it might be easier to pass a function directly
    as a backend and not use a string.
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。

### Lines 96-113
```python
    Args:
        compiler_fn: Callable taking a FX graph and fake tensor inputs
        name: Optional name, defaults to `compiler_fn.__name__`
        tags: Optional set of string tags to categorize backend with
    """
    if compiler_fn is None:
        # @register_backend(name="") syntax
        return functools.partial(register_backend, name=name, tags=tags)  # type: ignore[return-value]
    assert callable(compiler_fn)
    name = name or compiler_fn.__name__
    assert name not in _COMPILER_FNS, f"duplicate name: {name}"
    if compiler_fn not in _BACKENDS:
        _BACKENDS[name] = None
    _COMPILER_FNS[name] = compiler_fn
    compiler_fn._tags = tuple(tags)  # type: ignore[attr-defined]
    return compiler_fn
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 114-129
```python
register_debug_backend = functools.partial(register_backend, tags=("debug",))
register_experimental_backend = functools.partial(
    register_backend, tags=("experimental",)
)


def lookup_backend(compiler_fn: str | CompilerFn) -> CompilerFn:
    """Expand backend strings to functions"""
    if isinstance(compiler_fn, str):
        if compiler_fn not in _BACKENDS:
            _lazy_import()
        if compiler_fn not in _BACKENDS:
            from ..exc import InvalidBackend

            raise InvalidBackend(name=compiler_fn)
```
- **EN**: This module-level block helps hand work to a compiler/backend pipeline. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于将工作移交给编译器或后端流水线。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 130-147
```python
        if compiler_fn not in _COMPILER_FNS:
            entry_point = _BACKENDS[compiler_fn]
            if entry_point is not None:
                register_backend(compiler_fn=entry_point.load(), name=compiler_fn)
        compiler_fn = _COMPILER_FNS[compiler_fn]
    return compiler_fn


# NOTE: can't type this due to public api mismatch; follow up with dev team
def list_backends(exclude_tags=("debug", "experimental")) -> list[str]:  # type: ignore[no-untyped-def]
    """
    Return valid strings that can be passed to:

        torch.compile(..., backend="name")
    """
    _lazy_import()
    exclude_tags_set = set(exclude_tags or ())
```
- **EN**: This module-level block helps hand work to a compiler/backend pipeline. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于将工作移交给编译器或后端流水线。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 148-165
```python
    backends = [
        name
        for name in _BACKENDS
        if name not in _COMPILER_FNS
        or not exclude_tags_set.intersection(_COMPILER_FNS[name]._tags)  # type: ignore[attr-defined]
    ]
    return sorted(backends)


@functools.cache
def _lazy_import() -> None:
    from .. import backends
    from ..utils import import_submodule

    import_submodule(backends)

    from ..repro.after_dynamo import dynamo_minifier_backend
```
- **EN**: These decorators register or transform the following definition so it can hand work to a compiler/backend pipeline.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将工作移交给编译器或后端流水线。

### Lines 166-183
```python
    assert dynamo_minifier_backend is not None

    _discover_entrypoint_backends()


@functools.cache
def _discover_entrypoint_backends() -> None:
    # importing here so it will pick up the mocked version in test_backends.py
    from importlib.metadata import entry_points

    group_name = "torch_dynamo_backends"
    eps = entry_points(group=group_name)
    # pyrefly: ignore [bad-index]
    eps_dict = {name: eps[name] for name in eps.names}
    for backend_name in eps_dict:
        _BACKENDS[backend_name] = eps_dict[backend_name]
```
- **EN**: These decorators register or transform the following definition so it can hand work to a compiler/backend pipeline.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将工作移交给编译器或后端流水线。

### Lines 184-195
```python
def _is_registered_backend(compiler_fn: CompilerFn) -> bool:
    """
    Check if the given compiler function is a registered backend.
    Custom backends (user-provided callables not in the registry) return False.
    """
    # Ensure backends are loaded
    _lazy_import()

    # Check if it's directly a registered backend function
    if compiler_fn in _COMPILER_FNS.values():
        return True
```
- **EN**: Defines the `_is_registered_backend` function; this block introduces logic that hand work to a compiler/backend pipeline.
- **CN**: 定义`_is_registered_backend` 函数；该代码块引入了用于将工作移交给编译器或后端流水线的逻辑。

### Lines 196-212
```python
    # Check for _TorchCompileInductorWrapper or _TorchCompileWrapper
    # These have a compiler_name attribute that identifies the backend
    if hasattr(compiler_fn, "compiler_name"):
        compiler_name = compiler_fn.compiler_name
        if compiler_name in _BACKENDS or compiler_name in _COMPILER_FNS:
            return True

    # Check if the wrapper has a compiler_fn attribute (e.g., _TorchCompileWrapper)
    if hasattr(compiler_fn, "compiler_fn"):
        return compiler_fn.compiler_fn in _COMPILER_FNS.values()

    return False


def set_default_backend(backend: str | CompilerFn | None) -> None:
    """Set the default backend used by torch.compile when no backend is explicitly specified.
```
- **EN**: This module-level block helps hand work to a compiler/backend pipeline. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于将工作移交给编译器或后端流水线。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 213-224
```python
    Pass None to reset to the default ("inductor").
    """
    global _default_backend
    if backend is None:
        _default_backend = "inductor"
        return
    if not isinstance(backend, str) and not callable(backend):
        raise TypeError(f"backend must be a string or callable, got {type(backend)}")
    _default_backend = backend


def get_default_backend() -> str | CompilerFn:
```
- **EN**: This module-level block helps hand work to a compiler/backend pipeline. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于将工作移交给编译器或后端流水线。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 225-226
```python
    """Return the current default backend for torch.compile."""
    return _default_backend
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

## Key Concepts / 关键概念

- **Graph capture / 图捕获**
  - EN: The file participates in converting eager Python execution into compilable graph form.
  - CN: 该文件参与把即时执行的 Python 程序转换为可编译的图形式。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Compiler handoff / 编译器交接**
  - EN: The implementation prepares captured work for backend compilation or execution.
  - CN: 该实现会把捕获到的工作整理后交给后端编译或执行。
- **Program transforms / 程序变换**
  - EN: The implementation rewrites existing program structure into a more useful form.
  - CN: 该实现会把现有程序结构改写成更有用的形式。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `..`, `..utils`, `..repro.after_dynamo`, `..exc`
- **Standard library / 标准库**: `functools`, `logging`, `collections.abc`, `importlib.metadata`, `typing`
- **Primary symbols / 核心符号**: `CompiledFn`, `_BACKENDS`, `_COMPILER_FNS`, `register_backend`, `lookup_backend`, `list_backends`, `_lazy_import`, `_discover_entrypoint_backends`, `_is_registered_backend`, `set_default_backend`, `get_default_backend`
- **Relationship summary / 关系总结**: Dependencies center on graph capture, guard checking, bytecode analysis, and backend handoff. / 依赖关系主要围绕图捕获、守卫检查、字节码分析和后端交接展开。
