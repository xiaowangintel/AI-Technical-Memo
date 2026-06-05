# opaque_object.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_library/opaque_object.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Python helpers for torch.library registration, library fragments, and operator definitions.
- **Purpose (CN)**: 实现 torch.library 注册、库片段以及算子定义相关的 Python 辅助逻辑。
## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
````python
"""
Note [Opaque Objects]

Opaque objects are the way we allow custom operators to accept a user-defined
"black box" object as an input.

There are two kinds of opaque types: VALUE type and REFERENCE type.
The distinction determines how torch.compile handles the object.

REFERENCE TYPES (default):

Reference-typed opaque objects represent mutable stateful objects and are
treated as black boxes. In torch.compile, since torch.compile cannot optimize
the anything (including tensors) within the object, the object must be an
input to the graph.

You can register a custom class as being a reference-based opaque object class
through `register_opaque_type(MyClass, typ="reference")`.
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 20-36 / 第 20-36 行
````python
VALUE TYPES:

Value-typed opaque objects represent constant values.
In torch.compile, the graph specializes on the object like how other constants
are. Therefore there are a couple of methods on the class that must be
implemented before registering it as a value-typed opaque object class:
  - __eq__: torch.compile will create guards based on the equality of this
  object, meaning that a recompilation will happen if __eq__ returns False.
  - __hash__: This must be implemented for Fake Tensor caching
  - __fx_repr__: This must be implemented to provide an evaluable representation
    for FX graph codegen. It should return a tuple of (repr_string, dict[str, type])
    where repr_string can reconstruct the object and the dict maps names used in
    repr_string to their corresponding types.

You can register a custom class as being a reference-based opaque object class
through `register_opaque_type(MyClass, typ="value")`.
"""
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 38-54 / 第 38-54 行
````python
import logging
from collections.abc import Callable
from dataclasses import dataclass
from enum import Enum
from typing import Any, Literal, NewType, TYPE_CHECKING, TypeAlias
from typing_extensions import TypeIs
from weakref import WeakKeyDictionary

import torch
from torch._opaque_base import OpaqueBase, OpaqueBaseMeta


if TYPE_CHECKING:
    from torch.fx import Proxy
    from torch.fx.experimental.proxy_tensor import PythonKeyTracer

from .fake_class_registry import register_fake_class
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch._opaque_base, torch.fx, ...; standard-library helpers such as logging, collections.abc, dataclasses, ...; other helper packages such as typing_extensions. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch._opaque_base、torch.fx、...；标准库辅助模块，如 logging、collections.abc、dataclasses、...；其他辅助包，如 typing_extensions。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 57-75 / 第 57-75 行
````python
log = logging.getLogger(__name__)


class MemberType(Enum):
    """
    Defines how a member (attribute/property/method) of an opaque object is handled
    during torch.compile tracing.
    """

    # Reads/calls the member at trace time with the real object and bakes the result as a constant
    USE_REAL = "use_real"
    # Inlines/traces the member
    INLINED = "inlined"


@register_fake_class("aten::OpaqueObject")
class FakeOpaqueObject:
    def __init__(self) -> None:
        pass
````
- **EN**: It introduces or extends `MemberType`, `FakeOpaqueObject`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration. Decorators such as `register_fake_class` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 它引入或扩展了 `MemberType`、`FakeOpaqueObject`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 像 `register_fake_class` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 77-96 / 第 77-96 行
````python
    @classmethod
    def __obj_unflatten__(cls, flattened_ctx: dict[str, Any]) -> None:
        raise RuntimeError(
            "FakeOpaqueObject should not be created through __obj_unflatten__ "
            "and should be special handled. Please file an issue to Github."
        )


OpaqueTypeStr = "__torch__.torch.classes.aten.OpaqueObject"

OpaqueType = NewType("OpaqueType", torch._C.ScriptObject)

# Type for reconstruct_fn: called by PythonKeyTracer.create_arg when make_fx
# encounters an untracked opaque reference (e.g. a backward closure capture).
# Should derive the object from existing graph inputs or return None to fall
# back to get_attr.  Args: (obj, get_tracked_proxy, tracer).
ReconstructFn: TypeAlias = Callable[
    [OpaqueBase, Callable[[OpaqueBase], "Proxy | None"], "PythonKeyTracer"],
    "Proxy | None",
]
````
- **EN**: This chunk defines `__obj_unflatten__`, which implements a focused helper used by the surrounding module. Decorators such as `classmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__obj_unflatten__`，其作用是实现周边模块使用的关键辅助逻辑。 像 `classmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 99-114 / 第 99-114 行
````python
@dataclass
class _OpaqueTypeInfo:
    class_name: str
    opaque_typ: Literal["reference", "value"]
    guard_fn: Callable[
        [Any], list[Any]
    ]  # Callable that takes the object and returns list of values to guard on
    members: dict[str, MemberType]  # Maps member name to how it should be handled
    hoist: bool
    reconstruct_fn: ReconstructFn | None


# Mapping of type -> (string name, reference/value type)
_OPAQUE_TYPES: WeakKeyDictionary[Any, _OpaqueTypeInfo] = WeakKeyDictionary()
# Mapping of class_name -> (type, reference/value type)
_OPAQUE_TYPES_BY_NAME: dict[str, _OpaqueTypeInfo] = {}
````
- **EN**: Typing-related scaffolding keeps the Python API explicit about expected tensors, literals, or callable shapes. It introduces or extends `_OpaqueTypeInfo`, which hold the main object-oriented state for this portion of the file. Decorators such as `dataclass` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks.
- **CN**: 与类型相关的脚手架让 Python API 更明确地描述期望的张量、字面量或可调用对象形状。 它引入或扩展了 `_OpaqueTypeInfo`，这些类承载了本段涉及的主要面向对象状态。 像 `dataclass` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。

### Lines 117-135 / 第 117-135 行
````python
def _resolve_opaque_type_info(cls: Any) -> _OpaqueTypeInfo | None:
    if cls in _OPAQUE_TYPES:
        return _OPAQUE_TYPES[cls]
    if not isinstance(cls, type):
        return None

    # Allow subclasses too
    for parent in cls.__mro__[1:]:
        if parent in _OPAQUE_TYPES:
            return _OPAQUE_TYPES[parent]
    return None


def get_opaque_type_name(cls: Any) -> str:
    """
    Gets the registered opaque type name for a given class.

    Args:
        cls (type): The class to get the type name for.
````
- **EN**: This chunk defines `get_opaque_type_name`, which retrieves runtime state and exposes it through a Python-friendly accessor. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_opaque_type_name`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 137-149 / 第 137-149 行
````python
    Returns:
        str: The registered type name for the class.

    Raises:
        ValueError: If the class is not registered as an opaque type.
    """
    info = _resolve_opaque_type_info(cls)
    if info is None:
        raise ValueError(
            f"Class {cls} is not registered as an opaque type. "
            f"Call register_opaque_type({cls.__name__}) first."
        )
    return info.class_name
````
- **EN**: This chunk continues `get_opaque_type_name` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `get_opaque_type_name`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 152-166 / 第 152-166 行
````python
def register_opaque_type(
    cls: Any,
    *,
    typ: str,
    hoist=False,
    guard_fn: Any = None,
    members: dict[str, MemberType] | None = None,
    reconstruct_fn: ReconstructFn | None = None,
) -> None:
    """
    Registers the given type as an opaque type which allows this to be consumed
    by a custom operator.

    The type name will be automatically generated from the class's fully
    qualified name (ex. my_module.MyClass).
````
- **EN**: This chunk defines `register_opaque_type`, which registers a hook, schema, operator, or callback with surrounding infrastructure.
- **CN**: 这一段定义了 `register_opaque_type`，其作用是向周边基础设施注册钩子、schema、算子或回调。

### Lines 168-187 / 第 168-187 行
````python
    Args:
        cls (type): The class to register as an opaque type.
        typ (str): Either "reference" or "value". See Note [Opaque Objects] for
            more details.
        hoist (bool): Only applies to value types. A hoist=True value type
            object is lifted as an input to the torch.compile'd graph, instead
            of being a constant baked into the graph. This is useful to
            improve compilation times in hierarchical compilation
            (e.g., change your custom ops to use hoisted strings to avoid
            baking the string into the Dynamo/AOTAutograd/FX graphs).
            This flag does nothing for reference types.
        guard_fn (callable | None): A function that takes an instance of the opaque
            object and returns a list of values to guard on. These values will be compared
            for equality on each function call, triggering recompilation if they change.
            Only applicable for reference types.
            Example: lambda obj: [obj.x, obj.y]
        members (dict[str, MemberType] | None): Dictionary mapping member names
            (attributes, properties, or methods) to their MemberType, which controls
            how they are handled during torch.compile tracing:
            - MemberType.USE_REAL: Evaluates with the real object at compile time and
````
- **EN**: This chunk continues `register_opaque_type` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `register_opaque_type`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 188-204 / 第 188-204 行
````python
              bakes the result as a constant
            - MemberType.INLINED: Inlines the method call into the trace
    """
    import torch.utils._pytree as pytree

    # Prevent registration of built-in types (int, str, list, dict, etc.) and torch.Tensor
    if cls.__module__ == "builtins" or cls is torch.Tensor:
        raise ValueError(
            f"Unable to register built-in type {cls} as an opaque type. "
            "Please wrap it in a custom class and register the custom class as opaque."
        )

    if cls in pytree.SUPPORTED_NODES:
        raise ValueError(
            f"{cls} cannot be registered as an opaque object as it has been "
            "registered as a pytree. Opaque objects must be pytree leaves."
        )
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.utils._pytree. This chunk continues `register_opaque_type` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.utils._pytree。 这一段延续了 `register_opaque_type`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 206-220 / 第 206-220 行
````python
    # Value types store the real object directly during tracing (no
    # FakeScriptObject wrapper), so they don't need OpaqueBaseMeta.
    if typ != "value" and not isinstance(cls, OpaqueBaseMeta):
        raise TypeError(
            f"Opaque type {cls} must subclass torch._opaque_base.OpaqueBase "
            "or 'metaclass=torch._opaque_base.OpaqueBaseMeta'. "
            "This is required so that FakeScriptObject can be registered "
            "as a virtual subclass, allowing isinstance() checks to work "
            "during torch.compile tracing. "
        )

    if typ not in ["reference", "value"]:
        raise AssertionError(
            f"Opaque type must be either 'reference' or 'value', got {typ!r}"
        )
````
- **EN**: This chunk continues `register_opaque_type` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `register_opaque_type`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 222-240 / 第 222-240 行
````python
    if typ == "value":
        # Enums use identity-based equality (singletons), which is fine for guarding.
        if not issubclass(cls, Enum) and cls.__eq__ is object.__eq__:  # type: ignore[comparison-overlap]
            raise TypeError(
                f"Value-type opaque object of type {cls} is "
                "expected to have a non-default `__eq__` "
                "implementation as we will use this in torch.compile "
                "to guard on the equality of objects."
            )

        # Class with a custom `__eq__` without `__hash__` won't inherit the default
        # `__hash__` from object; see https://stackoverflow.com/a/1608907.
        if cls.__hash__ is None:  # type: ignore[comparison-overlap]
            raise TypeError(
                f"Value-type opaque object of type {cls} is "
                "expected to have a non-default `__hash__` "
                "implementation as we will use this in torch.compile "
                "for FakeTensor caching."
            )
````
- **EN**: This chunk continues `register_opaque_type` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `register_opaque_type`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 242-260 / 第 242-260 行
````python
        # Enums are special-cased in get_opaque_obj_repr.
        if not issubclass(cls, Enum) and not hasattr(cls, "__fx_repr__"):
            raise TypeError(
                f"Value-type opaque object of type {cls} is "
                "expected to have a `__fx_repr__` method "
                "implementation as we will use this to reconstruct "
                "the object in the FX codegen. __fx_repr__ should return "
                "a tuple of (repr_string, dict[str, type])."
            )

        if guard_fn is not None:
            raise TypeError(
                "No need to specify `guard_fn` for "
                f"value-type opaque class {cls} as it will be guarded based "
                "on `__eq__`."
            )

    # Generate a fully qualified name by combining module and qualname
    name = f"{cls.__module__}.{cls.__qualname__}"
````
- **EN**: This chunk continues `register_opaque_type` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `register_opaque_type`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 262-278 / 第 262-278 行
````python
    type_info = _OpaqueTypeInfo(
        name, typ, guard_fn, members or {}, hoist, reconstruct_fn
    )
    _OPAQUE_TYPES[cls] = type_info
    _OPAQUE_TYPES_BY_NAME[name] = type_info

    torch._C._register_opaque_type(name)


# Enums are always opaque value types.
register_opaque_type(Enum, typ="value")


def is_opaque_value(value: object) -> TypeIs[OpaqueType]:
    if is_opaque_type(type(value)):
        return True
    from torch._library.fake_class_registry import FakeScriptObject
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._library.fake_class_registry. This chunk defines `is_opaque_value`, which checks a capability or invariant before later code relies on it. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._library.fake_class_registry。 这一段定义了 `is_opaque_value`，其作用是检查某项能力或不变量，供后续逻辑依赖。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 280-296 / 第 280-296 行
````python
    if isinstance(value, FakeScriptObject):
        return is_opaque_type(type(value.real_obj))
    return False


def should_hoist(cls: Any) -> bool:
    info = _resolve_opaque_type_info(cls)
    if info is None:
        return False
    return info.hoist


def get_reconstruct_fn(cls: type[OpaqueBase]) -> ReconstructFn | None:
    info = _resolve_opaque_type_info(cls)
    if info is None:
        return None
    return info.reconstruct_fn
````
- **EN**: This chunk defines `get_reconstruct_fn`, which retrieves runtime state and exposes it through a Python-friendly accessor. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_reconstruct_fn`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 299-316 / 第 299-316 行
````python
def has_members(cls: Any) -> bool:
    info = _resolve_opaque_type_info(cls)
    if info is None:
        return False
    return len(info.members) > 0


def is_opaque_type(cls: type[Any] | str) -> bool:
    """
    Checks if the given type is an opaque type.
    Also returns True for subclasses of registered opaque types.
    """
    if isinstance(cls, str):
        return torch._C._is_opaque_type_registered(cls)

    if not isinstance(cls, type):
        log.warning("Passed invalid type `%s` to is_opaque_type, returning False", cls)
        return False
````
- **EN**: This chunk defines `is_opaque_type`, which checks a capability or invariant before later code relies on it. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `is_opaque_type`，其作用是检查某项能力或不变量，供后续逻辑依赖。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 318-334 / 第 318-334 行
````python
    info = _resolve_opaque_type_info(cls)
    if info is None:
        return False

    return torch._C._is_opaque_type_registered(info.class_name)


def is_opaque_value_type(cls: type[Any] | str) -> bool:
    """
    Checks if the given type is an opaque **value** type.
    See Note [Opaque Objects] for more information.
    """
    if not is_opaque_type(cls):
        return False

    if isinstance(cls, str):
        return _OPAQUE_TYPES_BY_NAME[cls].opaque_typ == "value"
````
- **EN**: This chunk defines `is_opaque_value_type`, which checks a capability or invariant before later code relies on it. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `is_opaque_value_type`，其作用是检查某项能力或不变量，供后续逻辑依赖。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 336-351 / 第 336-351 行
````python
    info = _resolve_opaque_type_info(cls)
    if info is None:
        return False
    return info.opaque_typ == "value"


def is_opaque_reference_type(cls: Any) -> bool:
    """
    Checks if the given type is an opaque **reference** type.
    See Note [Opaque Objects] for more information.
    """
    if not is_opaque_type(cls):
        return False

    if isinstance(cls, str):
        return _OPAQUE_TYPES_BY_NAME[cls].opaque_typ == "reference"
````
- **EN**: This chunk defines `is_opaque_reference_type`, which checks a capability or invariant before later code relies on it. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `is_opaque_reference_type`，其作用是检查某项能力或不变量，供后续逻辑依赖。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 353-371 / 第 353-371 行
````python
    info = _resolve_opaque_type_info(cls)
    if info is None:
        return False
    return info.opaque_typ == "reference"


def get_opaque_obj_repr(obj: Any) -> tuple[str, dict[str, type]]:
    """
    Get the FX-evaluable repr for an opaque object and collect required globals.

    Objects must implement __fx_repr__() which should return:
        (repr_string, dict_mapping_name_to_type)

    where repr_string is an evaluable string representation and
    dict_mapping_name_to_type maps the names used in repr_string to their types.

    For example, if repr_string is "Foo(bar=Bar(1))", the dict should be:
        {"Foo": Foo, "Bar": Bar}
    """
````
- **EN**: This chunk defines `get_opaque_obj_repr`, which retrieves runtime state and exposes it through a Python-friendly accessor. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_opaque_obj_repr`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 373-387 / 第 373-387 行
````python
    # Enums are special cased
    if isinstance(obj, Enum):
        cls = type(obj)
        return f"{cls.__name__}.{obj.name}", {cls.__name__: cls}

    if not hasattr(obj, "__fx_repr__"):
        raise TypeError(
            f"Value-type opaque object of type {obj} is "
            "expected to have a `__fx_repr__` method "
            "implementation as we will use this to reconstruct "
            "the object in the FX codegen. __fx_repr__ should return "
            "a tuple of (repr_string, dict[str, type])."
        )

    repr_str, globals_dict = obj.__fx_repr__()
````
- **EN**: This chunk continues `get_opaque_obj_repr` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `get_opaque_obj_repr`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 389-406 / 第 389-406 行
````python
    if not isinstance(repr_str, str):
        raise TypeError(
            f"__fx_repr__ for {type(obj).__name__} must return a string as the "
            f"first element, got {type(repr_str).__name__}"
        )

    if not isinstance(globals_dict, dict):
        raise TypeError(
            f"__fx_repr__ for {type(obj).__name__} must return a dict as the "
            f"second element, got {type(globals_dict).__name__}"
        )

    return repr_str, globals_dict


def get_opaque_obj_info(cls: Any) -> _OpaqueTypeInfo | None:
    if not is_opaque_type(cls):
        return None
````
- **EN**: This chunk defines `get_opaque_obj_info`, which retrieves runtime state and exposes it through a Python-friendly accessor. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_opaque_obj_info`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 408-420 / 第 408-420 行
````python
    if isinstance(cls, str):
        return _OPAQUE_TYPES_BY_NAME[cls]

    return _resolve_opaque_type_info(cls)


def get_member_type(cls: Any, member_name: str) -> MemberType | None:
    """
    Get the MemberType for a specific member of an opaque object class.

    Args:
        cls: The opaque object class (or its string name)
        member_name: The name of the member to query
````
- **EN**: This chunk defines `get_member_type`, which retrieves runtime state and exposes it through a Python-friendly accessor. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_member_type`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 422-428 / 第 422-428 行
````python
    Returns:
        MemberType if the member is registered, None otherwise
    """
    info = get_opaque_obj_info(cls)
    if info is None:
        return None
    return info.members.get(member_name)
````
- **EN**: This chunk continues `get_member_type` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `get_member_type`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Library registration**
  - EN: Builds Python-side operator/library registrations that feed the dispatcher.
  - CN: 构建 Python 侧的算子/库注册，并将其接入 dispatcher。
- **MemberType**
  - EN: `MemberType` is one of the main symbols declared or implemented in this file.
  - CN: `MemberType` 是本文件声明或实现的主要符号之一。
- **FakeOpaqueObject**
  - EN: `FakeOpaqueObject` is one of the main symbols declared or implemented in this file.
  - CN: `FakeOpaqueObject` 是本文件声明或实现的主要符号之一。
- **Graph handling**
  - EN: The implementation manipulates captured graphs, graph metadata, or graph-scoped execution state.
  - CN: 实现会处理捕获图、图元数据或图作用域的执行状态。
- **Gradient semantics**
  - EN: The implementation preserves gradient correctness and explains how backward/JVP state is tracked.
  - CN: 实现会维护梯度正确性，并说明如何跟踪 backward/JVP 状态。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch._opaque_base`, `torch.fx`, `torch.fx.experimental.proxy_tensor`, `.fake_class_registry`, `torch.utils._pytree`, `torch._library.fake_class_registry`
- **Standard library / 标准库**: `logging`, `collections.abc`, `dataclasses`, `enum`, `typing`, `weakref`
- **Other helper packages / 其他辅助包**: `typing_extensions`
- **Primary symbols in this file / 本文件核心符号**: `MemberType`, `FakeOpaqueObject`, `_OpaqueTypeInfo`, `_OPAQUE_TYPES`, `_OPAQUE_TYPES_BY_NAME`, `_resolve_opaque_type_info`, `get_opaque_type_name`, `register_opaque_type`, `is_opaque_value`, `should_hoist`
