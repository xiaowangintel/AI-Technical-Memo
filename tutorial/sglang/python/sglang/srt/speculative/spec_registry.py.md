# spec_registry.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/speculative/spec_registry.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Internal storage backing ``SpeculativeAlgorithm.register``. / 该文件实现了推测解码基础设施，例如 worker、元数据对象、CUDA Graph 运行器或共享工具函数。它通过协调草稿模型执行与目标模型校验来提升吞吐。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15: Module header, imports, and shared constants
```python
"""Internal storage backing ``SpeculativeAlgorithm.register``. Plugins
should use that classmethod API; do not import from this module directly.
"""

from __future__ import annotations

from typing import TYPE_CHECKING, Callable, Dict, Optional, Type

if TYPE_CHECKING:
    from sglang.srt.server_args import ServerArgs

WorkerFactory = Callable[["ServerArgs"], Type]
ServerArgsValidator = Callable[["ServerArgs"], None]
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 16-29: CustomSpecAlgo class declaration
```python
class CustomSpecAlgo:
    """A plugin-registered speculative algorithm. Duck-types
    ``SpeculativeAlgorithm`` enum values (same ``is_*()`` / ``create_worker``
    interface).

    Plugins may subclass this to override any ``is_*()`` / ``supports_*()`` /
    ``create_worker`` method (e.g. to integrate with builtin-specific
    branches like ``if spec_algorithm.is_eagle():`` in scheduler /
    model_runner). Pass the subclass via ``spec_class=...`` at registration.

    Defaults: all ``is_*()`` return ``False`` except ``is_speculative``;
    ``supports_spec_v2`` follows ``supports_overlap``.
    """
```
**EN:** This block declares the `CustomSpecAlgo` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `CustomSpecAlgo` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 30-41: CustomSpecAlgo initializer
```python
    def __init__(
        self,
        name: str,
        factory: WorkerFactory,
        *,
        supports_overlap: bool = False,
        validate_server_args: Optional[ServerArgsValidator] = None,
    ):
        self.name = name
        self.factory = factory
        self.supports_overlap = supports_overlap
        self.validate_server_args = validate_server_args
```
**EN:** This block initializes the `CustomSpecAlgo` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `CustomSpecAlgo` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 43-44: CustomSpecAlgo.__repr__ method
```python
    def __repr__(self) -> str:
        return f"CustomSpecAlgo({self.name!r})"
```
**EN:** This block uses `CustomSpecAlgo.__repr__` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `CustomSpecAlgo.__repr__` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 46-47: CustomSpecAlgo.is_none method
```python
    def is_none(self) -> bool:
        return False
```
**EN:** This block uses `CustomSpecAlgo.is_none` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `CustomSpecAlgo.is_none` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 49-50: CustomSpecAlgo.is_speculative method
```python
    def is_speculative(self) -> bool:
        return True
```
**EN:** This block uses `CustomSpecAlgo.is_speculative` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `CustomSpecAlgo.is_speculative` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 52-53: CustomSpecAlgo.is_eagle method
```python
    def is_eagle(self) -> bool:
        return False
```
**EN:** This block uses `CustomSpecAlgo.is_eagle` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `CustomSpecAlgo.is_eagle` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 55-56: CustomSpecAlgo.is_eagle3 method
```python
    def is_eagle3(self) -> bool:
        return False
```
**EN:** This block uses `CustomSpecAlgo.is_eagle3` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `CustomSpecAlgo.is_eagle3` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 58-59: CustomSpecAlgo.is_dflash method
```python
    def is_dflash(self) -> bool:
        return False
```
**EN:** This block uses `CustomSpecAlgo.is_dflash` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `CustomSpecAlgo.is_dflash` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 61-62: CustomSpecAlgo.is_standalone method
```python
    def is_standalone(self) -> bool:
        return False
```
**EN:** This block uses `CustomSpecAlgo.is_standalone` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `CustomSpecAlgo.is_standalone` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 64-65: CustomSpecAlgo.is_ngram method
```python
    def is_ngram(self) -> bool:
        return False
```
**EN:** This block uses `CustomSpecAlgo.is_ngram` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `CustomSpecAlgo.is_ngram` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 67-68: CustomSpecAlgo.supports_spec_v2 method
```python
    def supports_spec_v2(self) -> bool:
        return self.supports_overlap
```
**EN:** This block uses `CustomSpecAlgo.supports_spec_v2` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `CustomSpecAlgo.supports_spec_v2` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 70-75: CustomSpecAlgo.create_worker method
```python
    def create_worker(self, server_args: "ServerArgs") -> Type:
        if not server_args.disable_overlap_schedule and not self.supports_overlap:
            raise ValueError(
                f"Speculative algorithm {self.name} does not support overlap scheduling."
            )
        return self.factory(server_args)
```
**EN:** This block uses `CustomSpecAlgo.create_worker` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `CustomSpecAlgo.create_worker` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 76-85: Module-level supporting statements
```python


_REGISTRY: Dict[str, CustomSpecAlgo] = {}

# Builtin enum members + the NEXTN alias; plugins cannot shadow these.
_RESERVED_NAMES = frozenset(
    {"DFLASH", "EAGLE", "EAGLE3", "NEXTN", "STANDALONE", "NGRAM", "NONE"}
)
```
**EN:** This block contains supporting statements for the module, such as constants, attributes, registrations, or small helper expressions that shape the surrounding control flow.
**CN:** 该代码块包含模块的配套语句，例如常量、属性、注册逻辑或小型辅助表达式，它们共同塑造了周边控制流程。

### Lines 86-115: register algorithm function
```python
def register_algorithm(
    name: str,
    *,
    supports_overlap: bool = False,
    validate_server_args: Optional[ServerArgsValidator] = None,
    spec_class: Type[CustomSpecAlgo] = CustomSpecAlgo,
) -> Callable[[WorkerFactory], WorkerFactory]:
    """Return a decorator that registers a plugin algorithm under ``name``.

    Pass a ``spec_class`` subclass of ``CustomSpecAlgo`` to override any
    ``is_*()`` / ``supports_*()`` / ``create_worker`` method.
    """
    upper = name.upper()
    if upper in _RESERVED_NAMES:
        raise ValueError(
            f"'{upper}' is a reserved speculative algorithm name; cannot be re-registered."
        )
    if upper in _REGISTRY:
        raise ValueError(f"Speculative algorithm '{upper}' already registered.")

    def decorator(factory: WorkerFactory) -> WorkerFactory:
        _REGISTRY[upper] = spec_class(
            name=upper,
            factory=factory,
            supports_overlap=supports_overlap,
            validate_server_args=validate_server_args,
        )
        return factory

    return decorator
```
**EN:** This block uses `register_algorithm` to register components for later lookup. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `register_algorithm` 来注册组件以供后续查找。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 118-123: get spec function
```python
def get_spec(name: Optional[str]) -> Optional[CustomSpecAlgo]:
    """Return the registered spec for ``name``, or ``None`` for builtin /
    unknown names."""
    if name is None:
        return None
    return _REGISTRY.get(name.upper())
```
**EN:** This block uses `get_spec` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `get_spec` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- Registration and lookup / 注册与查找
- Speculative decoding / 推测解码

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.server_args`
### External / 外部
- `__future__`
- `typing` (stdlib)
