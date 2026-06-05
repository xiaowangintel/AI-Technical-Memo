# _cxx_pytree.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/utils/_cxx_pytree.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements utility logic that supports broader PyTorch execution, packaging, or developer workflows. This specific file centers on `_cxx_pytree.py`. Key abstractions such as `_Asterisk, LeafSpecMeta` shape the public surface of the file.
- **Purpose (CN) / 用途（中文）**: 实现支撑更广泛 PyTorch 执行、打包或开发者工作流的辅助逻辑。 该文件具体围绕 `_cxx_pytree.py` 展开。 `_Asterisk, LeafSpecMeta` 等关键抽象塑造了该文件的公开接口。
## Line-by-Line Analysis / 逐行分析

### Lines 1-42 / 第 1-42 行
```python
# Owner(s): ["module: pytree"]

"""
Contains utility functions for working with nested python data structures.

A *pytree* is a Python nested data structure. It is a tree in the sense that
nodes are Python collections (e.g., list, tuple, dict) and the leaves are
Python values. Furthermore, a pytree should not contain reference cycles.

pytrees are useful for working with nested collections of Tensors. For example,
one can use `tree_map` to map a function over all Tensors inside some nested
collection of Tensors and `tree_leaves` to get a flat list of all Tensors
inside some nested collection. pytrees are helpful for implementing nested
collection support for PyTorch APIs.
"""

import functools
import sys
import types
from collections.abc import Callable, Iterable, Mapping
from typing import Any, overload, TypeAlias, TypeVar
from typing_extensions import deprecated, Self, TypeIs

import torch.utils._pytree as python_pytree
from torch.torch_version import TorchVersion as _TorchVersion
from torch.utils._pytree import (
    Context,
    DumpableContext,
    FlattenFn,
    FlattenWithKeysFn,
    FromDumpableContextFn,
    is_namedtuple,
    is_namedtuple_class,
    is_namedtuple_instance,
    is_structseq,
    is_structseq_class,
    is_structseq_instance,
    KeyPath,
    PyTree,
    ToDumpableContextFn,
    UnflattenFn,
)
```
- **EN**: The opening docstring frames the module contract, telling readers what subsystem responsibilities the rest of the file implements. This block establishes the module dependencies, pulling in internal torch modules such as torch.utils._pytree, torch.torch_version:TorchVersion, torch.utils._pytree:Context, torch.utils._pytree:DumpableContext; standard-library helpers such as functools, sys, types, collections.abc:Callable; external packages such as typing_extensions:deprecated, typing_extensions:Self, typing_extensions:TypeIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 开头的模块文档字符串给出了整体契约，说明后续代码要实现的子系统职责。 这一段建立模块依赖，引入了内部 torch 模块，如 torch.utils._pytree, torch.torch_version:TorchVersion, torch.utils._pytree:Context, torch.utils._pytree:DumpableContext；标准库辅助模块，如 functools, sys, types, collections.abc:Callable；外部包，如 typing_extensions:deprecated, typing_extensions:Self, typing_extensions:TypeIs。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 45-88 / 第 45-88 行
```python
# Do not try to import `optree` package if the static version check already fails.
if not python_pytree._cxx_pytree_dynamo_traceable:
    raise ImportError(
        f"{__name__} depends on `optree>={python_pytree._optree_minimum_version}`, "
        "which is an optional dependency of PyTorch. "
        "To use it, please upgrade your optree package via "
        "`python3 -m pip install --upgrade optree`"
    )


import optree
from optree import PyTreeSpec  # direct import for type annotations


__all__ = [
    "PyTree",
    "Context",
    "FlattenFn",
    "UnflattenFn",
    "DumpableContext",
    "ToDumpableContextFn",
    "FromDumpableContextFn",
    "PyTreeSpec",
    "TreeSpec",
    "LeafSpec",
    "keystr",
    "key_get",
    "register_pytree_node",
    "tree_is_leaf",
    "tree_flatten",
    "tree_flatten_with_path",
    "tree_unflatten",
    "tree_iter",
    "tree_leaves",
    "tree_leaves_with_path",
    "tree_structure",
    "tree_map",
    "tree_map_with_path",
    "tree_map_",
    "tree_map_only",
    "tree_map_only_",
    "tree_all",
    "tree_any",
    "tree_all_only",
```
- **EN**: The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 89-128 / 第 89-128 行
```python
    "tree_any_only",
    "treespec_dumps",
    "treespec_loads",
    "treespec_pprint",
    "is_namedtuple",
    "is_namedtuple_class",
    "is_namedtuple_instance",
    "is_structseq",
    "is_structseq_class",
    "is_structseq_instance",
]


# In-tree installation may have VCS-based versioning. Update the previous static version.
python_pytree._optree_version = _TorchVersion(optree.__version__)  # type: ignore[attr-defined]

__TORCH_DICT_SESSION = optree.dict_insertion_ordered(True, namespace="torch")
__TORCH_DICT_SESSION.__enter__()  # enable globally and permanently


T = TypeVar("T")
S = TypeVar("S")
U = TypeVar("U")
R = TypeVar("R")

TreeSpec: TypeAlias = PyTreeSpec
OpTreeUnflattenFn = Callable[[Context, Iterable[Any]], PyTree]

# Keep deprecated alias for backward compatibility
FlattenFunc = FlattenFn  # deprecated
UnflattenFunc = UnflattenFn  # deprecated
FlattenWithKeysFunc = FlattenWithKeysFn  # deprecated


def _reverse_args(func: UnflattenFn) -> OpTreeUnflattenFn:
    @functools.wraps(func)
    def wrapped(*args: Any, **kwargs: Any) -> Any:
        return func(*reversed(args), **kwargs)

    return wrapped
```
- **EN**: Key callable entry points in this range include `_reverse_args`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Named constants such as `__TORCH_DICT_SESSION`, `T`, `S`, `U` centralize shared configuration or sentinel values. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `_reverse_args`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 `__TORCH_DICT_SESSION, T, S, U` 等具名常量把共享配置或哨兵值集中定义在一起。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 131-173 / 第 131-173 行
```python
def register_pytree_node(
    cls: type[Any],
    flatten_fn: FlattenFn,
    unflatten_fn: UnflattenFn,
    *,
    serialized_type_name: str | None = None,
    to_dumpable_context: ToDumpableContextFn | None = None,
    from_dumpable_context: FromDumpableContextFn | None = None,
    flatten_with_keys_fn: FlattenWithKeysFn | None = None,
) -> None:
    """Register a container-like type as pytree node.

    Args:
        cls (type): A Python type to treat as an internal pytree node.
        flatten_fn (callable): A function to be used during flattening, taking an instance of
            ``cls`` and returning a pair, with (1) an iterable for the children to be flattened
            recursively, and (2) some hashable auxiliary data to be stored in the treespec and to be
            passed to the ``unflatten_fn``.
        unflatten_fn (callable): A function taking two arguments: the auxiliary data that was
            returned by ``flatten_fn`` and stored in the treespec, and the unflattened children.
            The function should return an instance of ``cls``.
        serialized_type_name (str, optional): A keyword argument used to specify the fully
            qualified name used when serializing the tree spec.
        to_dumpable_context (callable, optional): An optional keyword argument to custom specify how
            to convert the context of the pytree to a custom json dumpable representation. This is
            used for json serialization, which is being used in :mod:`torch.export` right now.
        from_dumpable_context (callable, optional): An optional keyword argument to custom specify
            how to convert the custom json dumpable representation of the context back to the
            original context. This is used for json deserialization, which is being used in
            :mod:`torch.export` right now.

    Example::

        >>> # xdoctest: +SKIP
        >>> # Register a Python type with lambda functions
        >>> register_pytree_node(
        ...     set,
        ...     lambda s: (sorted(s), None, None),
        ...     lambda children, _: set(children),
        ... )
    """
    if flatten_with_keys_fn is not None:
        raise NotImplementedError("KeyPaths are not yet supported in cxx_pytree.")
```
- **EN**: Key callable entry points in this range include `register_pytree_node`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `register_pytree_node`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 175-213 / 第 175-213 行
```python
    _private_register_pytree_node(
        cls,
        flatten_fn,
        unflatten_fn,
        serialized_type_name=serialized_type_name,
        to_dumpable_context=to_dumpable_context,
        from_dumpable_context=from_dumpable_context,
    )

    python_pytree._private_register_pytree_node(
        cls,
        flatten_fn,
        unflatten_fn,
        serialized_type_name=serialized_type_name,
        to_dumpable_context=to_dumpable_context,
        from_dumpable_context=from_dumpable_context,
    )


@deprecated(
    "`torch.utils._cxx_pytree._register_pytree_node` is deprecated. "
    "Please use `torch.utils._cxx_pytree.register_pytree_node` instead.",
    category=FutureWarning,
)
def _register_pytree_node(
    cls: type[Any],
    flatten_fn: FlattenFn,
    unflatten_fn: UnflattenFn,
    *,
    serialized_type_name: str | None = None,
    to_dumpable_context: ToDumpableContextFn | None = None,
    from_dumpable_context: FromDumpableContextFn | None = None,
) -> None:
    """Register a container-like type as pytree node for the C++ pytree only.

    The ``namespace`` argument is used to avoid collisions that occur when different libraries
    register the same Python type with different behaviors. It is recommended to add a unique prefix
    to the namespace to avoid conflicts with other libraries. Namespaces can also be used to specify
    the same class in different namespaces for different use cases.
```
- **EN**: Key callable entry points in this range include `register_pytree_node`, `_register_pytree_node`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `register_pytree_node`, `_register_pytree_node`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 215-247 / 第 215-247 行
```python
    .. warning::
        For safety reasons, a ``namespace`` must be specified while registering a custom type. It is
        used to isolate the behavior of flattening and unflattening a pytree node type. This is to
        prevent accidental collisions between different libraries that may register the same type.

    Args:
        cls (type): A Python type to treat as an internal pytree node.
        flatten_fn (callable): A function to be used during flattening, taking an instance of
            ``cls`` and returning a pair, with (1) an iterable for the children to be flattened
            recursively, and (2) some hashable auxiliary data to be stored in the treespec and to be
            passed to the ``unflatten_fn``.
        unflatten_fn (callable): A function taking two arguments: the auxiliary data that was
            returned by ``flatten_fn`` and stored in the treespec, and the unflattened children.
            The function should return an instance of ``cls``.
        serialized_type_name (str, optional): A keyword argument used to specify the fully
            qualified name used when serializing the tree spec.
        to_dumpable_context (callable, optional): An optional keyword argument to custom specify how
            to convert the context of the pytree to a custom json dumpable representation. This is
            used for json serialization, which is being used in :mod:`torch.export` right now.
        from_dumpable_context (callable, optional): An optional keyword argument to custom specify
            how to convert the custom json dumpable representation of the context back to the
            original context. This is used for json deserialization, which is being used in
            :mod:`torch.export` right now.
    """

    _private_register_pytree_node(
        cls,
        flatten_fn,
        unflatten_fn,
        serialized_type_name=serialized_type_name,
        to_dumpable_context=to_dumpable_context,
        from_dumpable_context=from_dumpable_context,
    )
```
- **EN**: Key callable entry points in this range include `_register_pytree_node`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `_register_pytree_node`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 250-292 / 第 250-292 行
```python
def _private_register_pytree_node(
    cls: type[Any],
    flatten_fn: FlattenFn,
    unflatten_fn: UnflattenFn,
    *,
    serialized_type_name: str | None = None,
    to_dumpable_context: ToDumpableContextFn | None = None,
    from_dumpable_context: FromDumpableContextFn | None = None,
) -> None:
    """This is an internal function that is used to register a pytree node type
    for the C++ pytree only. End-users should use :func:`register_pytree_node`
    instead.
    """
    # TODO(XuehaiPan): remove this condition when we make Python pytree out-of-box support
    # PyStructSequence types
    if not optree.is_structseq_class(cls):
        optree.register_pytree_node(
            cls,
            flatten_fn,
            _reverse_args(unflatten_fn),
            namespace="torch",
        )


def _is_pytreespec_instance(
    obj: Any,
    /,
) -> TypeIs[TreeSpec | python_pytree.PyTreeSpec]:
    if isinstance(obj, (TreeSpec, python_pytree.PyTreeSpec)):
        return True
    if "torch._dynamo.polyfills.pytree" in sys.modules:
        # The PyTorch Dynamo pytree module is not always available, so we check if it is loaded.
        # If the PyTorch Dynamo pytree module is loaded, we can check if the treespec
        # is an instance of the PyTorch Dynamo TreeSpec class.
        import torch._dynamo.polyfills.pytree as dynamo_pytree

        return isinstance(obj, dynamo_pytree.PyTreeSpec)
    return False


def treespec_leaf() -> TreeSpec:
    """Make a treespec representing a leaf node."""
    return optree.treespec_leaf(none_is_leaf=True, namespace="torch")
```
- **EN**: Key callable entry points in this range include `_private_register_pytree_node`, `_is_pytreespec_instance`, `treespec_leaf`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `_private_register_pytree_node`, `_is_pytreespec_instance`, `treespec_leaf`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 295-331 / 第 295-331 行
```python
def treespec_tuple(iterable: Iterable[TreeSpec] = (), /) -> TreeSpec:
    """Make a tuple treespec from an iterable of child treespecs."""
    return optree.treespec_tuple(iterable, none_is_leaf=True, namespace="torch")


def treespec_dict(
    mapping: Mapping[Any, TreeSpec] | Iterable[tuple[Any, TreeSpec]] = (),
    /,
    **kwargs: TreeSpec,
) -> TreeSpec:
    """Make a dict treespec from a dict of child treespecs."""
    return optree.treespec_dict(
        mapping,
        **kwargs,
        none_is_leaf=True,
        namespace="torch",
    )


def tree_is_leaf(
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> bool:
    """Check if a pytree is a leaf.

    >>> tree_is_leaf(1)
    True
    >>> tree_is_leaf(None)
    True
    >>> tree_is_leaf([1, 2, 3])
    False
    >>> tree_is_leaf((1, 2, 3), is_leaf=lambda x: isinstance(x, tuple))
    True
    >>> tree_is_leaf({"a": 1, "b": 2, "c": 3})
    False
    >>> tree_is_leaf({"a": 1, "b": 2, "c": None})
    False
```
- **EN**: Key callable entry points in this range include `treespec_tuple`, `treespec_dict`, `tree_is_leaf`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `treespec_tuple`, `treespec_dict`, `tree_is_leaf`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 333-373 / 第 333-373 行
```python
    Args:
        tree (pytree): A pytree to check if it is a leaf node.
        is_leaf (callable, optional): An extra leaf predicate function that will be called at each
            flattening step. The function should have a single argument with signature
            ``is_leaf(node) -> bool``. If it returns :data:`True`, the whole subtree being treated
            as a leaf. Otherwise, the default pytree registry will be used to determine a node is a
            leaf or not. If the function is not specified, the default pytree registry will be used.

    Returns:
        A boolean indicating if the pytree is a leaf node.
    """
    return optree.tree_is_leaf(
        tree,
        is_leaf=is_leaf,
        none_is_leaf=True,
        namespace="torch",
    )


def tree_flatten(
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> tuple[list[Any], TreeSpec]:
    """Flatten a pytree.

    See also :func:`tree_unflatten`.

    The flattening order (i.e., the order of elements in the output list) is deterministic,
    corresponding to a left-to-right depth-first tree traversal.

    >>> tree = {"b": (2, [3, 4]), "a": 1, "c": None, "d": 5}
    >>> tree_flatten(tree)
    ([2, 3, 4, 1, None, 5], PyTreeSpec({'b': (*, [*, *]), 'a': *, 'c': *, 'd': *}, NoneIsLeaf, namespace='torch'))
    >>> tree_flatten(1)
    ([1], PyTreeSpec(*, NoneIsLeaf, namespace='torch'))
    >>> tree_flatten(None)
    ([None], PyTreeSpec(*, NoneIsLeaf, namespace='torch'))
    >>> from collections import OrderedDict
    >>> tree = OrderedDict([("b", (2, [3, 4])), ("a", 1), ("c", None), ("d", 5)])
    >>> tree_flatten(tree)
    ([2, 3, 4, 1, None, 5], PyTreeSpec(OrderedDict({'b': (*, [*, *]), 'a': *, 'c': *, 'd': *}), NoneIsLeaf, namespace='torch'))
```
- **EN**: Key callable entry points in this range include `tree_is_leaf`, `tree_flatten`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `tree_is_leaf`, `tree_flatten`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 375-408 / 第 375-408 行
```python
    Args:
        tree (pytree): A pytree to flatten.
        is_leaf (callable, optional): An extra leaf predicate function that will be called at each
            flattening step. The function should have a single argument with signature
            ``is_leaf(node) -> bool``. If it returns :data:`True`, the whole subtree being treated
            as a leaf. Otherwise, the default pytree registry will be used to determine a node is a
            leaf or not. If the function is not specified, the default pytree registry will be used.

    Returns:
        A pair ``(leaves, treespec)`` where the first element is a list of leaf values and the
        second element is a treespec representing the structure of the pytree.
    """
    return optree.tree_flatten(  # type: ignore[return-value]
        tree,
        is_leaf=is_leaf,
        none_is_leaf=True,
        namespace="torch",
    )


def tree_unflatten(leaves: Iterable[Any], treespec: TreeSpec) -> PyTree:
    """Reconstruct a pytree from the treespec and the leaves.

    The inverse of :func:`tree_flatten`.

    >>> tree = {"b": (2, [3, 4]), "a": 1, "c": None, "d": 5}
    >>> leaves, treespec = tree_flatten(tree)
    >>> tree == tree_unflatten(leaves, treespec)
    True

    Args:
        leaves (iterable): The list of leaves to use for reconstruction. The list must match the
            number of leaves of the treespec.
        treespec (TreeSpec): The treespec to reconstruct.
```
- **EN**: Key callable entry points in this range include `tree_flatten`, `tree_unflatten`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `tree_flatten`, `tree_unflatten`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 410-448 / 第 410-448 行
```python
    Returns:
        The reconstructed pytree, containing the ``leaves`` placed in the structure described by
        ``treespec``.
    """
    if not _is_pytreespec_instance(treespec):
        if not _is_pytreespec_instance(leaves):
            raise TypeError(
                f"Expected `treespec` to be an instance of "
                f"PyTreeSpec but got item of type {type(treespec)}."
            )
        # Allow passing the PyTreeSpec instance as the first argument
        # pyrefly: ignore [bad-assignment]
        leaves, treespec = treespec, leaves
    return treespec.unflatten(leaves)


def tree_iter(
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> Iterable[Any]:
    """Get an iterator over the leaves of a pytree.

    See also :func:`tree_flatten`.

    >>> tree = {"b": (2, [3, 4]), "a": 1, "c": None, "d": 5}
    >>> list(tree_iter(tree))
    [2, 3, 4, 1, None, 5]
    >>> list(tree_iter(1))
    [1]
    >>> list(tree_iter(None))
    [None]

    Args:
        tree (pytree): A pytree to flatten.
        is_leaf (callable, optional): An extra leaf predicate function that will be called at each
            flattening step. The function should have a single argument with signature
            ``is_leaf(node) -> bool``. If it returns :data:`True`, the whole subtree being treated
            as a leaf. Otherwise, the default pytree registry will be used to determine a node is a
            leaf or not. If the function is not specified, the default pytree registry will be used.
```
- **EN**: Key callable entry points in this range include `tree_unflatten`, `tree_iter`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `tree_unflatten`, `tree_iter`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 450-483 / 第 450-483 行
```python
    Returns:
        An iterator over the leaf values.
    """
    return optree.tree_iter(
        tree,
        is_leaf=is_leaf,
        none_is_leaf=True,
        namespace="torch",
    )


def tree_leaves(
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> list[Any]:
    """Get the leaves of a pytree.

    See also :func:`tree_flatten`.

    >>> tree = {"b": (2, [3, 4]), "a": 1, "c": None, "d": 5}
    >>> tree_leaves(tree)
    [2, 3, 4, 1, None, 5]
    >>> tree_leaves(1)
    [1]
    >>> tree_leaves(None)
    [None]

    Args:
        tree (pytree): A pytree to flatten.
        is_leaf (callable, optional): An extra leaf predicate function that will be called at each
            flattening step. The function should have a single argument with signature
            ``is_leaf(node) -> bool``. If it returns :data:`True`, the whole subtree being treated
            as a leaf. Otherwise, the default pytree registry will be used to determine a node is a
            leaf or not. If the function is not specified, the default pytree registry will be used.
```
- **EN**: Key callable entry points in this range include `tree_iter`, `tree_leaves`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `tree_iter`, `tree_leaves`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 485-518 / 第 485-518 行
```python
    Returns:
        A list of leaf values.
    """
    return optree.tree_leaves(
        tree,
        is_leaf=is_leaf,
        none_is_leaf=True,
        namespace="torch",
    )


def tree_structure(
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> TreeSpec:
    """Get the treespec for a pytree.

    See also :func:`tree_flatten`.

    >>> tree = {"b": (2, [3, 4]), "a": 1, "c": None, "d": 5}
    >>> tree_structure(tree)
    PyTreeSpec({'b': (*, [*, *]), 'a': *, 'c': *, 'd': *}, NoneIsLeaf, namespace='torch')
    >>> tree_structure(1)
    PyTreeSpec(*, NoneIsLeaf, namespace='torch')
    >>> tree_structure(None)
    PyTreeSpec(*, NoneIsLeaf, namespace='torch')

    Args:
        tree (pytree): A pytree to flatten.
        is_leaf (callable, optional): An extra leaf predicate function that will be called at each
            flattening step. The function should have a single argument with signature
            ``is_leaf(node) -> bool``. If it returns :data:`True`, the whole subtree being treated
            as a leaf. Otherwise, the default pytree registry will be used to determine a node is a
            leaf or not. If the function is not specified, the default pytree registry will be used.
```
- **EN**: Key callable entry points in this range include `tree_leaves`, `tree_structure`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `tree_leaves`, `tree_structure`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 520-550 / 第 520-550 行
```python
    Returns:
        A treespec object representing the structure of the pytree.
    """
    return optree.tree_structure(  # type: ignore[return-value]
        tree,
        is_leaf=is_leaf,
        none_is_leaf=True,
        namespace="torch",
    )


def tree_map(
    func: Callable[..., Any],
    tree: PyTree,
    *rests: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> PyTree:
    """Map a multi-input function over pytree args to produce a new pytree.

    See also :func:`tree_map_`.

    >>> tree_map(lambda x: x + 1, {"x": 7, "y": (42, 64)})
    {'x': 8, 'y': (43, 65)}
    >>> tree_map(lambda x: x is None, {"x": 7, "y": (42, 64), "z": None})
    {'x': False, 'y': (False, False), 'z': True}

    If multiple inputs are given, the structure of the tree is taken from the first input;
    subsequent inputs need only have ``tree`` as a prefix:

    >>> tree_map(lambda x, y: [x] + y, [5, 6], [[7, 9], [1, 2]])
    [[5, 7, 9], [6, 1, 2]]
```
- **EN**: Key callable entry points in this range include `tree_structure`, `tree_map`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `tree_structure`, `tree_map`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 552-588 / 第 552-588 行
```python
    Args:
        func (callable): A function that takes ``1 + len(rests)`` arguments, to be applied at the
            corresponding leaves of the pytrees.
        tree (pytree): A pytree to be mapped over, with each leaf providing the first positional
            argument to function ``func``.
        rests (tuple of pytree): A tuple of pytrees, each of which has the same structure as
            ``tree`` or has ``tree`` as a prefix.
        is_leaf (callable, optional): An extra leaf predicate function that will be called at each
            flattening step. The function should have a single argument with signature
            ``is_leaf(node) -> bool``. If it returns :data:`True`, the whole subtree being treated
            as a leaf. Otherwise, the default pytree registry will be used to determine a node is a
            leaf or not. If the function is not specified, the default pytree registry will be used.

    Returns:
        A new pytree with the same structure as ``tree`` but with the value at each leaf given by
        ``func(x, *xs)`` where ``x`` is the value at the corresponding leaf in ``tree`` and ``xs``
        is the tuple of values at corresponding nodes in ``rests``.
    """
    return optree.tree_map(
        func,
        tree,
        *rests,
        is_leaf=is_leaf,
        none_is_leaf=True,
        namespace="torch",
    )


def tree_map_(
    func: Callable[..., Any],
    tree: PyTree,
    *rests: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> PyTree:
    """Like :func:`tree_map`, but do an inplace call on each leaf and return the original tree.

    See also :func:`tree_map`.
```
- **EN**: Key callable entry points in this range include `tree_map`, `tree_map_`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `tree_map`, `tree_map_`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 590-627 / 第 590-627 行
```python
    Args:
        func (callable): A function that takes ``1 + len(rests)`` arguments, to be applied at the
            corresponding leaves of the pytrees.
        tree (pytree): A pytree to be mapped over, with each leaf providing the first positional
            argument to function ``func``.
        rests (tuple of pytree): A tuple of pytrees, each of which has the same structure as
            ``tree`` or has ``tree`` as a prefix.
        is_leaf (callable, optional): An extra leaf predicate function that will be called at each
            flattening step. The function should have a single argument with signature
            ``is_leaf(node) -> bool``. If it returns :data:`True`, the whole subtree being treated
            as a leaf. Otherwise, the default pytree registry will be used to determine a node is a
            leaf or not. If the function is not specified, the default pytree registry will be used.

    Returns:
        The original ``tree`` with the value at each leaf is given by the side-effect of function
        ``func(x, *xs)`` (not the return value) where ``x`` is the value at the corresponding leaf
        in ``tree`` and ``xs`` is the tuple of values at values at corresponding nodes in ``rests``.
    """
    return optree.tree_map_(
        func,
        tree,
        *rests,
        is_leaf=is_leaf,
        none_is_leaf=True,
        namespace="torch",
    )


Type2 = tuple[type[T], type[S]]
Type3 = tuple[type[T], type[S], type[U]]
TypeAny = type[Any] | tuple[type[Any], ...] | types.UnionType

Fn2 = Callable[[T | S], R]
Fn3 = Callable[[T | S | U], R]
Fn = Callable[[T], R]
FnAny = Callable[[Any], R]

MapOnlyFn = Callable[[T], Callable[[Any], Any]]
```
- **EN**: Key callable entry points in this range include `tree_map_`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `tree_map_`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 630-670 / 第 630-670 行
```python
# These specializations help with type inference on the lambda passed to this
# function
@overload
def map_only(type_or_types_or_pred: type[T], /) -> MapOnlyFn[Fn[T, Any]]: ...


@overload
def map_only(type_or_types_or_pred: Type2[T, S], /) -> MapOnlyFn[Fn2[T, S, Any]]: ...


@overload
def map_only(
    type_or_types_or_pred: Type3[T, S, U], /
) -> MapOnlyFn[Fn3[T, S, U, Any]]: ...


# This specialization is needed for the implementations below that call
@overload
def map_only(type_or_types_or_pred: TypeAny, /) -> MapOnlyFn[FnAny[Any]]: ...


@overload
def map_only(
    type_or_types_or_pred: Callable[[Any], bool], /
) -> MapOnlyFn[FnAny[Any]]: ...


def map_only(
    type_or_types_or_pred: TypeAny | Callable[[Any], bool], /
) -> MapOnlyFn[FnAny[Any]]:
    """
    Suppose you are writing a tree_map over tensors, leaving everything
    else unchanged.  Ordinarily you would have to write:

        def go(t):
            if isinstance(t, Tensor):
                return ...
            else:
                return t

    With this function, you only need to write:
```
- **EN**: Key callable entry points in this range include `map_only`, `map_only`, `map_only`, `map_only`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `map_only`, `map_only`, `map_only`, `map_only`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 672-707 / 第 672-707 行
```python
        @map_only(Tensor)
        def go(t):
            return ...

    You can also directly use 'tree_map_only'
    """
    if isinstance(type_or_types_or_pred, (type, tuple, types.UnionType)):

        def pred(x: Any) -> bool:
            return isinstance(x, type_or_types_or_pred)  # type: ignore[arg-type]

    elif callable(type_or_types_or_pred):
        pred = type_or_types_or_pred  # type: ignore[assignment]
    else:
        raise TypeError("Argument must be a type, a tuple of types, or a callable.")

    def wrapper(func: Callable[[T], Any]) -> Callable[[Any], Any]:
        @functools.wraps(func)
        def wrapped(x: T) -> Any:
            if pred(x):
                return func(x)
            return x

        return wrapped

    return wrapper


@overload
def tree_map_only(
    type_or_types_or_pred: type[T],
    /,
    func: Fn[T, Any],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> PyTree: ...
```
- **EN**: Key callable entry points in this range include `map_only`, `tree_map_only`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `map_only`, `tree_map_only`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 710-747 / 第 710-747 行
```python
@overload
def tree_map_only(
    type_or_types_or_pred: Type2[T, S],
    /,
    func: Fn2[T, S, Any],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> PyTree: ...


@overload
def tree_map_only(
    type_or_types_or_pred: Type3[T, S, U],
    /,
    func: Fn3[T, S, U, Any],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> PyTree: ...


@overload
def tree_map_only(
    type_or_types_or_pred: TypeAny,
    /,
    func: FnAny[Any],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> PyTree: ...


@overload
def tree_map_only(
    type_or_types_or_pred: Callable[[Any], bool],
    /,
    func: FnAny[Any],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> PyTree: ...
```
- **EN**: Key callable entry points in this range include `tree_map_only`, `tree_map_only`, `tree_map_only`, `tree_map_only`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `tree_map_only`, `tree_map_only`, `tree_map_only`, `tree_map_only`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 750-787 / 第 750-787 行
```python
def tree_map_only(
    type_or_types_or_pred: TypeAny | Callable[[Any], bool],
    /,
    func: FnAny[Any],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> PyTree:
    return tree_map(map_only(type_or_types_or_pred)(func), tree, is_leaf=is_leaf)


@overload
def tree_map_only_(
    type_or_types_or_pred: type[T],
    /,
    func: Fn[T, Any],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> PyTree: ...


@overload
def tree_map_only_(
    type_or_types_or_pred: Type2[T, S],
    /,
    func: Fn2[T, S, Any],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> PyTree: ...


@overload
def tree_map_only_(
    type_or_types_or_pred: Type3[T, S, U],
    /,
    func: Fn3[T, S, U, Any],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> PyTree: ...
```
- **EN**: Key callable entry points in this range include `tree_map_only`, `tree_map_only_`, `tree_map_only_`, `tree_map_only_`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `tree_map_only`, `tree_map_only_`, `tree_map_only_`, `tree_map_only_`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 790-826 / 第 790-826 行
```python
@overload
def tree_map_only_(
    type_or_types_or_pred: TypeAny,
    /,
    func: FnAny[Any],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> PyTree: ...


@overload
def tree_map_only_(
    type_or_types_or_pred: Callable[[Any], bool],
    /,
    func: FnAny[Any],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> PyTree: ...


def tree_map_only_(
    type_or_types_or_pred: TypeAny | Callable[[Any], bool],
    /,
    func: FnAny[Any],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> PyTree:
    return tree_map_(map_only(type_or_types_or_pred)(func), tree, is_leaf=is_leaf)


def tree_all(
    pred: Callable[[Any], bool],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> bool:
    flat_args = tree_iter(tree, is_leaf=is_leaf)
    return all(map(pred, flat_args))
```
- **EN**: Key callable entry points in this range include `tree_map_only_`, `tree_map_only_`, `tree_map_only_`, `tree_all`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `tree_map_only_`, `tree_map_only_`, `tree_map_only_`, `tree_all`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 829-865 / 第 829-865 行
```python
def tree_any(
    pred: Callable[[Any], bool],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> bool:
    flat_args = tree_iter(tree, is_leaf=is_leaf)
    return any(map(pred, flat_args))


@overload
def tree_all_only(
    type_or_types: type[T],
    /,
    pred: Fn[T, bool],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> bool: ...


@overload
def tree_all_only(
    type_or_types: Type2[T, S],
    /,
    pred: Fn2[T, S, bool],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> bool: ...


@overload
def tree_all_only(
    type_or_types: Type3[T, S, U],
    /,
    pred: Fn3[T, S, U, bool],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> bool: ...
```
- **EN**: Key callable entry points in this range include `tree_any`, `tree_all_only`, `tree_all_only`, `tree_all_only`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `tree_any`, `tree_all_only`, `tree_all_only`, `tree_all_only`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 868-906 / 第 868-906 行
```python
def tree_all_only(
    type_or_types: TypeAny,
    /,
    pred: FnAny[bool],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> bool:
    flat_args = tree_iter(tree, is_leaf=is_leaf)
    return all(pred(x) for x in flat_args if isinstance(x, type_or_types))


@overload
def tree_any_only(
    type_or_types: type[T],
    /,
    pred: Fn[T, bool],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> bool: ...


@overload
def tree_any_only(
    type_or_types: Type2[T, S],
    /,
    pred: Fn2[T, S, bool],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> bool: ...


@overload
def tree_any_only(
    type_or_types: Type3[T, S, U],
    /,
    pred: Fn3[T, S, U, bool],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> bool: ...
```
- **EN**: Key callable entry points in this range include `tree_all_only`, `tree_any_only`, `tree_any_only`, `tree_any_only`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `tree_all_only`, `tree_any_only`, `tree_any_only`, `tree_any_only`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 909-945 / 第 909-945 行
```python
def tree_any_only(
    type_or_types: TypeAny,
    /,
    pred: FnAny[bool],
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> bool:
    flat_args = tree_iter(tree, is_leaf=is_leaf)
    return any(pred(x) for x in flat_args if isinstance(x, type_or_types))


def broadcast_prefix(
    prefix_tree: PyTree,
    full_tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> list[Any]:
    """Return a list of broadcasted leaves in ``prefix_tree`` to match the number of leaves in ``full_tree``.

    If a ``prefix_tree`` is a prefix of a ``full_tree``, this means the ``full_tree`` can be
    constructed by replacing the leaves of ``prefix_tree`` with appropriate **subtrees**.

    This function returns a list of leaves with the same size as ``full_tree``. The leaves are
    replicated from ``prefix_tree``. The number of replicas is determined by the corresponding
    subtree in ``full_tree``.

    >>> broadcast_prefix(1, [1, 2, 3])
    [1, 1, 1]
    >>> broadcast_prefix([1, 2, 3], [1, 2, 3])
    [1, 2, 3]
    >>> broadcast_prefix([1, 2, 3], [1, 2, 3, 4])
    Traceback (most recent call last):
        ...
    ValueError: list arity mismatch; expected: 3, got: 4; list: [1, 2, 3, 4].
    >>> broadcast_prefix([1, 2, 3], [1, 2, (3, 4)])
    [1, 2, 3, 3]
    >>> broadcast_prefix([1, 2, 3], [1, 2, {"a": 3, "b": 4, "c": (None, 5)}])
    [1, 2, 3, 3, 3, 3]
```
- **EN**: Key callable entry points in this range include `tree_any_only`, `broadcast_prefix`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `tree_any_only`, `broadcast_prefix`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 947-990 / 第 947-990 行
```python
    Args:
        prefix_tree (pytree): A pytree with the same structure as a prefix of ``full_tree``.
        full_tree (pytree): A pytree with the same structure as a suffix of ``prefix_tree``.
        is_leaf (callable, optional): An extra leaf predicate function that will be called at each
            flattening step. The function should have a single argument with signature
            ``is_leaf(node) -> bool``. If it returns :data:`True`, the whole subtree being treated
            as a leaf. Otherwise, the default pytree registry will be used to determine a node is a
            leaf or not. If the function is not specified, the default pytree registry will be used.

    Returns:
        A list of leaves in ``prefix_tree`` broadcasted to match the number of leaves in ``full_tree``.
    """
    result: list[Any] = []

    def add_leaves(x: Any, subtree: PyTree) -> None:
        subtreespec = tree_structure(subtree, is_leaf=is_leaf)
        result.extend([x] * subtreespec.num_leaves)

    tree_map_(
        add_leaves,
        prefix_tree,
        full_tree,
        is_leaf=is_leaf,
    )
    return result


# Broadcasts a pytree to the provided TreeSpec and returns the flattened
# values. If this is not possible, then this function returns None.
#
# For example, given pytree=0 and spec=TreeSpec(list, None, [LeafSpec(), LeafSpec()]),
# would return [0, 0]. This is useful for part of the vmap implementation:
# a user can pass in vmap(fn, in_dims)(*inputs). `in_dims` should be
# broadcastable to the tree structure of `inputs` and we use
# _broadcast_to_and_flatten to check this.
def _broadcast_to_and_flatten(
    tree: PyTree,
    treespec: TreeSpec,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> list[Any] | None:
    if not _is_pytreespec_instance(treespec):
        raise TypeError(
            f"Expected `treespec` to be an instance of "
            f"PyTreeSpec but got item of type {type(treespec)}."
```
- **EN**: Key callable entry points in this range include `broadcast_prefix`, `_broadcast_to_and_flatten`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `broadcast_prefix`, `_broadcast_to_and_flatten`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 991-1031 / 第 991-1031 行
```python
        )
    full_tree = tree_unflatten([0] * treespec.num_leaves, treespec)
    try:
        return broadcast_prefix(tree, full_tree, is_leaf=is_leaf)
    except ValueError:
        return None


def treespec_dumps(treespec: TreeSpec, protocol: int | None = None) -> str:
    """Serialize a treespec to a JSON string."""
    if not _is_pytreespec_instance(treespec):
        raise TypeError(
            f"Expected `treespec` to be an instance of "
            f"PyTreeSpec but got item of type {type(treespec)}."
        )

    dummy_tree = tree_unflatten([0] * treespec.num_leaves, treespec)
    orig_treespec = python_pytree.tree_structure(dummy_tree)
    return python_pytree.treespec_dumps(orig_treespec, protocol=protocol)


@functools.lru_cache
def treespec_loads(serialized: str) -> TreeSpec:
    """Deserialize a treespec from a JSON string."""
    orig_treespec = python_pytree.treespec_loads(serialized)
    dummy_tree = python_pytree.tree_unflatten(
        [0] * orig_treespec.num_leaves,
        orig_treespec,
    )
    treespec = tree_structure(dummy_tree)
    return treespec


class _Asterisk(str):
    __slots__ = ()

    def __new__(cls) -> Self:
        return super().__new__(cls, "*")

    def __repr__(self) -> str:
        return "*"  # no quotes
```
- **EN**: It introduces or extends class-level abstractions such as `_Asterisk`, which organize state and behavior for this subsystem. Key callable entry points in this range include `_broadcast_to_and_flatten`, `treespec_dumps`, `treespec_loads`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Structured exception handling keeps failure modes explicit instead of letting partial state escape silently. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 它引入或扩展了 `_Asterisk` 等类级抽象，用于组织该子系统的状态与行为。 这一段的重要可调用入口包括 `_broadcast_to_and_flatten`, `treespec_dumps`, `treespec_loads`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 结构化异常处理让失败模式保持显式，而不是让部分状态悄悄泄漏出去。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 1034-1062 / 第 1034-1062 行
```python
_asterisk = _Asterisk()
del _Asterisk


def treespec_pprint(treespec: TreeSpec) -> str:
    dummy_tree = tree_unflatten([_asterisk] * treespec.num_leaves, treespec)
    return repr(dummy_tree)


class LeafSpecMeta(type(TreeSpec)):  # type: ignore[misc]
    def __instancecheck__(self, instance: object) -> bool:
        return _is_pytreespec_instance(instance) and instance.is_leaf()


@deprecated(
    "`isinstance(treespec, LeafSpec)` is deprecated, "
    "use `isinstance(treespec, TreeSpec)` and `treespec.is_leaf()` instead.",
    category=FutureWarning,
)
class LeafSpec(TreeSpec, metaclass=LeafSpecMeta):  # type: ignore[misc,final]
    def __new__(cls) -> Self:
        return treespec_leaf()  # type: ignore[return-value]


def tree_flatten_with_path(
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> tuple[list[tuple[KeyPath, Any]], TreeSpec]:
    """Flattens a pytree like :func:`tree_flatten`, but also returns each leaf's key path.
```
- **EN**: It introduces or extends class-level abstractions such as `LeafSpecMeta`, `LeafSpec`, which organize state and behavior for this subsystem. Key callable entry points in this range include `treespec_pprint`, `tree_flatten_with_path`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 它引入或扩展了 `LeafSpecMeta`, `LeafSpec` 等类级抽象，用于组织该子系统的状态与行为。 这一段的重要可调用入口包括 `treespec_pprint`, `tree_flatten_with_path`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 1064-1099 / 第 1064-1099 行
```python
    Args:
        tree: a pytree to flatten. If it contains a custom type, that type must be
            registered with an appropriate `tree_flatten_with_path_fn` when registered
            with :func:`register_pytree_node`.
        is_leaf: An extra leaf predicate function that will be called at each
            flattening step. The function should have a single argument with signature
            ``is_leaf(node) -> bool``. If it returns :data:`True`, the whole subtree being treated
            as a leaf. Otherwise, the default pytree registry will be used to determine a node is a
            leaf or not. If the function is not specified, the default pytree registry will be used.
    Returns:
        A tuple where the first element is a list of (key path, leaf) pairs, and the
        second element is a :class:`TreeSpec` representing the structure of the flattened
        tree.
    """
    raise NotImplementedError("KeyPaths are not yet supported in cxx_pytree.")


def tree_leaves_with_path(
    tree: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> list[tuple[KeyPath, Any]]:
    """Gets the leaves of a pytree like ``tree_leaves`` and returns each leaf's key path.

    Args:
        tree: a pytree. If it contains a custom type, that type must be
            registered with an appropriate `tree_flatten_with_path_fn` when registered
            with :func:`register_pytree_node`.
        is_leaf: An extra leaf predicate function that will be called at each
            flattening step. The function should have a single argument with signature
            ``is_leaf(node) -> bool``. If it returns :data:`True`, the whole subtree being treated
            as a leaf. Otherwise, the default pytree registry will be used to determine a node is a
            leaf or not. If the function is not specified, the default pytree registry will be used.
    Returns:
        A list of (key path, leaf) pairs.
    """
    raise NotImplementedError("KeyPaths are not yet supported in cxx_pytree.")
```
- **EN**: Key callable entry points in this range include `tree_flatten_with_path`, `tree_leaves_with_path`. They package a focused unit of behavior behind named helpers or APIs. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `tree_flatten_with_path`, `tree_leaves_with_path`，它们把聚焦的行为封装成具名辅助函数或 API。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 1102-1141 / 第 1102-1141 行
```python
def tree_map_with_path(
    func: Callable[..., Any],
    tree: PyTree,
    *rests: PyTree,
    is_leaf: Callable[[PyTree], bool] | None = None,
) -> PyTree:
    """Like :func:`tree_map`, but the provided callable takes an additional key path argument.

    Args:
        func: A function that takes ``2 + len(rests)`` arguments, to be applied at the
            corresponding leaves of the pytrees. The first positional argument
            to ``func`` is the key path of the leaf in question. The second
            positional argument is the value of the leaf.
        tree: A pytree to be mapped over, with each leaf providing the first positional
            argument to function ``func``.
        rests: A tuple of pytrees, each of which has the same structure as
            ``tree`` or has ``tree`` as a prefix.
        is_leaf: An extra leaf predicate function that will be called at each
            flattening step. The function should have a single argument with signature
            ``is_leaf(node) -> bool``. If it returns :data:`True`, the whole subtree being treated
            as a leaf. Otherwise, the default pytree registry will be used to determine a node is a
            leaf or not. If the function is not specified, the default pytree registry will be used.

    Returns
        A new pytree with the same structure as ``tree`` but with the value at each leaf given by
        ``func(keypath, x, *xs)`` where ``keypath`` is the key path at the
        corresponding leaf in ``tree``, ``x`` is the value at that leaf, and
        ``xs`` is the tuple of values at corresponding nodes in ``rests``.
    """
    raise NotImplementedError("KeyPaths are not yet supported in cxx_pytree.")


def keystr(kp: KeyPath) -> str:
    """Given a key path, return a pretty-printed representation."""
    raise NotImplementedError("KeyPaths are not yet supported in cxx_pytree.")


def key_get(obj: Any, kp: KeyPath) -> Any:
    """Given an object and a key path, return the value at the key path."""
    raise NotImplementedError("KeyPaths are not yet supported in cxx_pytree.")
```
- **EN**: Key callable entry points in this range include `tree_map_with_path`, `keystr`, `key_get`. They package a focused unit of behavior behind named helpers or APIs. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段的重要可调用入口包括 `tree_map_with_path`, `keystr`, `key_get`，它们把聚焦的行为封装成具名辅助函数或 API。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 1144-1151 / 第 1144-1151 行
```python
with python_pytree._NODE_REGISTRY_LOCK:
    # pyrefly: ignore [bad-assignment]
    python_pytree._cxx_pytree_imported = True
    args, kwargs = (), {}  # type: ignore[var-annotated]
    for args, kwargs in python_pytree._cxx_pytree_pending_imports:
        _private_register_pytree_node(*args, **kwargs)
    python_pytree._cxx_pytree_pending_imports.clear()
    del args, kwargs
```
- **EN**: This chunk continues the implementation of `key_get`, filling in the details of its control flow or data handling. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 这一段延续了 `key_get` 的实现，继续补充其控制流或数据处理细节。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 该实现推理的是嵌套容器结构，而不只是单个张量值。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements utility logic that supports broader PyTorch execution, packaging, or developer workflows.
  - CN: 实现支撑更广泛 PyTorch 执行、打包或开发者工作流的辅助逻辑。
- **Graph rewriting**
  - EN: Rewrites module or FX graph structure so later execution follows a transformed pipeline.
  - CN: 改写模块或 FX 图结构，使后续执行遵循转换后的流水线。
- **Pytree processing**
  - EN: Manipulates nested Python container structure while preserving leaves and shape metadata.
  - CN: 在保留叶子节点与结构元数据的同时处理嵌套 Python 容器。
- **TensorBoard integration**
  - EN: Exports model, scalar, and graph information into TensorBoard-consumable formats.
  - CN: 把模型、标量与图信息导出为 TensorBoard 可消费的格式。
- **Serialization safety**
  - EN: Keeps persisted data exchange bounded, predictable, and easier to validate.
  - CN: 让持久化数据交换保持有界、可预测且更易验证。
- **_Asterisk**
  - EN: `_Asterisk` is one of the main classes that structures the file's behavior.
  - CN: `_Asterisk` 是组织该文件行为的核心类之一。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch.utils._pytree`, `torch.torch_version:TorchVersion`, `torch.utils._pytree:Context`, `torch.utils._pytree:DumpableContext`, `torch.utils._pytree:FlattenFn`, `torch.utils._pytree:FlattenWithKeysFn`, `torch.utils._pytree:FromDumpableContextFn`, `torch.utils._pytree:is_namedtuple`, `torch.utils._pytree:is_namedtuple_class`, `torch.utils._pytree:is_namedtuple_instance`, `torch.utils._pytree:is_structseq`, `torch.utils._pytree:is_structseq_class`, `torch.utils._pytree:is_structseq_instance`, `torch.utils._pytree:KeyPath`, `torch.utils._pytree:PyTree`
- **Python standard library / Python 标准库**: `functools`, `sys`, `types`, `collections.abc:Callable`, `collections.abc:Iterable`, `collections.abc:Mapping`, `typing:Any`, `typing:overload`, `typing:TypeAlias`, `typing:TypeVar`
- **Third-party packages / 第三方包**: `typing_extensions:deprecated`, `typing_extensions:Self`, `typing_extensions:TypeIs`, `optree`, `optree:PyTreeSpec`
- **Explicit exports / 显式导出**: `PyTree`, `Context`, `FlattenFn`, `UnflattenFn`, `DumpableContext`, `ToDumpableContextFn`, `FromDumpableContextFn`, `PyTreeSpec`, `TreeSpec`, `LeafSpec`, `keystr`, `key_get`, `register_pytree_node`, `tree_is_leaf`, `tree_flatten`
- **Primary symbols / 核心符号**: `_Asterisk`, `LeafSpecMeta`, `LeafSpec`, `_reverse_args`, `register_pytree_node`, `_register_pytree_node`, `_private_register_pytree_node`, `_is_pytreespec_instance`, `treespec_leaf`, `treespec_tuple`, `treespec_dict`, `tree_is_leaf`, `tree_flatten`, `tree_unflatten`, `tree_iter`
