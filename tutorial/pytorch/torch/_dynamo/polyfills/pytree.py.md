# pytree.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_dynamo/polyfills/pytree.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Implements torch.compile front-end capture, guard management, and graph/lowering helpers in TorchDynamo. The module docstring emphasizes: Python polyfills for torch.utils.pytree
- **Purpose (CN)**: 实现 TorchDynamo 中的 torch.compile 前端捕获、守卫管理与图/降级辅助逻辑。 模块文档字符串重点说明了该实现的职责。

## Line-by-Line Analysis / 逐行分析

### Lines 1-41
```python
# Owner(s): ["module: pytree"]

"""
Python polyfills for torch.utils.pytree
"""

from __future__ import annotations

from collections import deque
from dataclasses import dataclass, field
from typing import Any, TYPE_CHECKING, TypeVar

import optree
import optree._C
import optree.utils
from optree import (
    is_namedtuple,
    is_namedtuple_class,
    is_namedtuple_instance,
    is_structseq,
    is_structseq_class,
    is_structseq_instance,
    namedtuple_fields,
    structseq_fields,
)

import torch.utils._cxx_pytree as cxx_pytree  # noqa: F401
import torch.utils._pytree as python_pytree
from torch.utils._pytree import BUILTIN_TYPES, STANDARD_DICT_TYPES

from ..decorators import substitute_in_graph


if TYPE_CHECKING:
    import builtins
    from collections.abc import Callable, Iterable, Mapping
    from typing_extensions import Self, TypeIs

    from torch.utils._cxx_pytree import PyTree
```
- **EN**: This typing-only branch imports symbols for static analysis without affecting runtime behavior.
- **CN**: 这个仅用于类型检查的分支会导入静态分析所需符号，而不会影响运行时行为。

### Lines 42-81
```python
__all__ = [
    "is_namedtuple",
    "is_namedtuple_class",
    "is_namedtuple_instance",
    "is_structseq",
    "is_structseq_class",
    "is_structseq_instance",
    "namedtuple_fields",
    "structseq_fields",
    "treespec_leaf",
    "treespec_tuple",
    "treespec_dict",
    "tree_is_leaf",
    "tree_iter",
    "tree_leaves",
    "tree_flatten",
    "tree_flatten_with_path",
    "tree_structure",
    "tree_unflatten",
]


_T = TypeVar("_T")
_KT = TypeVar("_KT")
_VT = TypeVar("_VT")


@substitute_in_graph(
    optree._C.is_dict_insertion_ordered,
    can_constant_fold_through=True,
)
def _(*args: Any, **kwargs: Any) -> bool:
    # In namespace 'torch', the dictionary is always traversed in insertion order.
    # This function returns True.
    raise ValueError(
        "Should not be called directly "
        "because the original function will be called in the constant fold path."
    )
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 82-116
```python
__name = ""
for __name, __func in (
    ("is_namedtuple", is_namedtuple),
    ("is_namedtuple_class", is_namedtuple_class),
    ("is_namedtuple_instance", is_namedtuple_instance),
    ("is_structseq", is_structseq),
    ("is_structseq_class", is_structseq_class),
    ("is_structseq_instance", is_structseq_instance),
    ("namedtuple_fields", namedtuple_fields),
    ("structseq_fields", structseq_fields),
):
    globals()[__name] = substitute_in_graph(
        __func,  # type: ignore[arg-type]
        can_constant_fold_through=True,
    )(__func.__python_implementation__)  # type: ignore[attr-defined]
    del __func
del __name


@substitute_in_graph(optree.tree_is_leaf, can_constant_fold_through=True)  # type: ignore[arg-type]
def tree_is_leaf(
    tree: PyTree,
    /,
    is_leaf: Callable[[PyTree], bool] | None = None,
    *,
    none_is_leaf: bool = False,
    namespace: str = "",
) -> bool:
    if (tree is None and none_is_leaf) or (is_leaf is not None and is_leaf(tree)):
        return True
    if optree.register_pytree_node.get(type(tree), namespace=namespace) is None:
        return True
    return False
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 117-146
```python
@substitute_in_graph(optree.tree_iter, can_constant_fold_through=False)  # type: ignore[arg-type]
def tree_iter(
    tree: PyTree,
    /,
    is_leaf: Callable[[PyTree], bool] | None = None,
    *,
    none_is_leaf: bool = False,
    namespace: str = "",
) -> Iterable[Any]:
    stack = [tree]
    while stack:
        node = stack.pop()
        if tree_is_leaf(
            node,
            is_leaf=is_leaf,
            none_is_leaf=none_is_leaf,
            namespace=namespace,
        ):
            yield node
            continue

        children, *_ = optree.tree_flatten_one_level(
            node,
            is_leaf=is_leaf,
            none_is_leaf=none_is_leaf,
            namespace=namespace,
        )
        stack.extend(reversed(children))
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 147-183
```python
@substitute_in_graph(optree.tree_leaves, can_constant_fold_through=True)  # type: ignore[arg-type]
def tree_leaves(
    tree: PyTree,
    /,
    is_leaf: Callable[[PyTree], bool] | None = None,
    *,
    none_is_leaf: bool = False,
    namespace: str = "",
) -> list[Any]:
    return list(
        tree_iter(
            tree,
            is_leaf=is_leaf,
            none_is_leaf=none_is_leaf,
            namespace=namespace,
        )
    )


class _Asterisk(str):
    __slots__ = ()

    def __new__(cls) -> Self:
        return super().__new__(cls, "*")

    def __repr__(self) -> str:
        return "*"  # no quotes


_asterisk = _Asterisk()
del _Asterisk


@dataclass(frozen=True, slots=True)
class PyTreeSpec:
    """Analog for :class:`optree.PyTreeSpec` in Python."""
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 184-223
```python
    _children: tuple[PyTreeSpec, ...]
    _type: builtins.type | None
    _metadata: Any
    _entries: tuple[Any, ...]
    _unflatten_func: Callable[[Any | None, Iterable[PyTree]], PyTree] | None
    none_is_leaf: bool
    namespace: str

    num_nodes: int = field(init=False)
    num_leaves: int = field(init=False)
    num_children: int = field(init=False)

    def __post_init__(self, /) -> None:
        if self._type is None:
            assert len(self._children) == 0
            assert self._metadata is None
            assert self._entries == ()
            assert self._unflatten_func is None
            num_nodes = 1
            num_leaves = 1
            num_children = 0
        else:
            assert callable(self._unflatten_func)
            num_nodes = 1
            num_leaves = 0
            for child in self._children:
                num_nodes += child.num_nodes
                num_leaves += child.num_leaves
            num_children = len(self._children)

        object.__setattr__(self, "num_nodes", num_nodes)
        object.__setattr__(self, "num_leaves", num_leaves)
        object.__setattr__(self, "num_children", num_children)

    def __repr__(self, /) -> str:
        def helper(treespec: PyTreeSpec) -> str:
            if treespec.is_leaf():
                assert treespec.type is None
                return _asterisk
```
- **EN**: Declares `PyTreeSpec`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `PyTreeSpec`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 224-260
```python
            assert treespec.type is not None
            assert callable(treespec._unflatten_func)
            children_representations = [
                helper(subspec) for subspec in treespec._children
            ]
            if (
                treespec.type in BUILTIN_TYPES
                or (treespec.type is type(None) and not self.none_is_leaf)
                or optree.is_namedtuple_class(treespec.type)
                or optree.is_structseq_class(treespec.type)
            ):
                return treespec._unflatten_func(
                    treespec._metadata,
                    children_representations,
                )
            return (
                f"CustomTreeNode({treespec.type.__name__}[{treespec._metadata!r}], "
                f"[{', '.join(children_representations)}])"
            )

        inner = [
            str(helper(self)),
            *(["NoneIsLeaf"] if self.none_is_leaf else []),
            f"namespace={self.namespace!r}",
        ]
        return f"PyTreeSpec({', '.join(inner)})"

    def __len__(self, /) -> int:
        return self.num_leaves

    @property
    def type(self, /) -> builtins.type | None:
        return self._type

    def is_leaf(self, /) -> bool:
        return self.num_nodes == 1 and self.num_leaves == 1
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 261-295
```python
    def paths(self, /) -> list[tuple[Any, ...]]:
        def helper(treespec: PyTreeSpec, path_prefix: list[Any]) -> None:
            if treespec.is_leaf():
                paths.append(path_prefix)
                return

            for entry, subspec in zip(
                treespec._entries,
                treespec._children,
                strict=True,
            ):
                helper(subspec, path_prefix + [entry])

        paths: list[list[Any]] = []
        helper(self, [])
        return [tuple(path) for path in paths]

    def accessors(self, /) -> list[optree.PyTreeAccessor]:
        def helper(
            treespec: PyTreeSpec,
            entry_path_prefix: list[optree.PyTreeEntry],
        ) -> None:
            if treespec.is_leaf():
                entry_paths.append(entry_path_prefix)
                return

            node_type = treespec.type
            assert node_type is not None
            handler = optree.register_pytree_node.get(
                node_type, namespace=treespec.namespace
            )
            assert handler is not None
            kind: optree.PyTreeKind = handler.kind
            path_entry_type: type[optree.PyTreeEntry] = handler.path_entry_type
```
- **EN**: Declares `PyTreeSpec`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `PyTreeSpec`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 296-331
```python
            for entry, subspec in zip(
                treespec._entries,
                treespec._children,
                strict=True,
            ):
                helper(
                    subspec,
                    entry_path_prefix + [path_entry_type(entry, node_type, kind)],
                )

        entry_paths: list[list[optree.PyTreeEntry]] = []
        helper(self, [])
        return [optree.PyTreeAccessor(path) for path in entry_paths]

    def children(self, /) -> list[PyTreeSpec]:
        return list(self._children)

    def child(self, index: int, /) -> PyTreeSpec:
        return self._children[index]

    def entries(self, /) -> list[Any]:
        return list(self._entries)

    def entry(self, index: int, /) -> Any:
        return self._entries[index]

    def flatten_up_to(self, tree: PyTree, /) -> list[PyTree]:
        def helper(
            treespec: PyTreeSpec,
            node: PyTree,
            subtrees: list[PyTree],
        ) -> None:
            if treespec.is_leaf():
                subtrees.append(node)
                return
```
- **EN**: Declares `PyTreeSpec`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `PyTreeSpec`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 332-372
```python
            node_type = type(node)
            if treespec.type not in BUILTIN_TYPES:
                # Always require custom node types to match exactly
                if node_type != treespec.type:
                    raise ValueError(
                        f"Type mismatch; "
                        f"expected {treespec.type!r}, but got {node_type!r}.",
                    )

                children, metadata, *_ = optree.tree_flatten_one_level(
                    node,
                    none_is_leaf=self.none_is_leaf,
                    namespace=self.namespace,
                )
                if len(children) != treespec.num_children:
                    raise ValueError(
                        f"Node arity mismatch; "
                        f"expected {treespec.num_children}, but got {len(children)}.",
                    )
                if metadata != treespec._metadata:
                    raise ValueError(
                        f"Node context mismatch for custom node type {treespec.type!r}.",
                    )
            else:
                # For builtin dictionary types, we allow some flexibility
                # Otherwise, we require exact matches
                both_standard_dict = (
                    treespec.type in STANDARD_DICT_TYPES
                    and node_type in STANDARD_DICT_TYPES
                )
                if not both_standard_dict and node_type != treespec.type:
                    raise ValueError(
                        f"Node type mismatch; "
                        f"expected {treespec.type!r}, but got {node_type!r}.",
                    )
                if len(node) != treespec.num_children:
                    raise ValueError(
                        f"Node arity mismatch; "
                        f"expected {treespec.num_children}, but got {len(node)}.",
                    )
```
- **EN**: This block continues `PyTreeSpec.flatten_up_to.helper` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `PyTreeSpec.flatten_up_to.helper`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 373-409
```python
                if both_standard_dict:
                    # dictionary types are compatible with each other
                    expected_keys = treespec.entries()
                    got_key_set = set(node)
                    expected_key_set = set(expected_keys)
                    if got_key_set != expected_key_set:
                        missing_keys = expected_key_set.difference(got_key_set)
                        extra_keys = got_key_set.difference(expected_key_set)
                        message = ""
                        if missing_keys:
                            message += f"; missing key(s): {missing_keys}"
                        if extra_keys:
                            message += f"; extra key(s): {extra_keys}"
                        raise ValueError(f"Node keys mismatch{message}.")
                    children = [node[key] for key in expected_keys]
                else:
                    # node_type is treespec.type
                    children, metadata, *_ = optree.tree_flatten_one_level(
                        node,
                        none_is_leaf=self.none_is_leaf,
                        namespace=self.namespace,
                    )
                    if (
                        node_type is not deque  # ignore mismatch of `maxlen` for deque
                    ) and metadata != treespec._metadata:
                        raise ValueError(
                            f"Node metadata mismatch for node type {treespec.type!r}; "
                            f"expected {treespec._metadata!r}, but got {metadata!r}.",  # namedtuple type mismatch
                        )

            for subtree, subspec in zip(children, treespec._children, strict=True):
                helper(subspec, subtree, subtrees)

        subtrees: list[PyTree] = []
        helper(self, tree, subtrees)
        return subtrees
```
- **EN**: This block continues `PyTreeSpec` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `PyTreeSpec`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 410-438
```python
    def unflatten(self, leaves: Iterable[Any], /) -> PyTree:
        if not isinstance(leaves, (list, tuple)):
            leaves = list(leaves)
        if len(leaves) != self.num_leaves:
            raise ValueError(
                f"treespec.unflatten(leaves): `leaves` has length {len(leaves)} "
                f"but the spec refers to a pytree that holds {self.num_leaves} "
                f"items ({self}).",
            )
        if self.is_leaf():
            return leaves[0]

        # Recursively unflatten the children
        start = 0
        end = 0
        subtrees = []
        for subspec in self._children:
            end += subspec.num_leaves
            subtrees.append(subspec.unflatten(leaves[start:end]))
            start = end

        assert callable(self._unflatten_func)
        return self._unflatten_func(self._metadata, subtrees)


def _is_pytreespec_instance(obj: Any, /) -> TypeIs[PyTreeSpec | python_pytree.TreeSpec]:
    return isinstance(obj, (PyTreeSpec, python_pytree.TreeSpec))
```
- **EN**: This module-level block helps validate invariants and surface meaningful failures. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于校验不变量并给出有意义的失败信息。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 439-466
```python
@substitute_in_graph(  # type: ignore[arg-type]
    optree.treespec_leaf,
    # We need to disable constant folding here because we want the function to reference the
    # PyTreeSpec class defined above, not the one in the C++ module.
    can_constant_fold_through=False,
)
def treespec_leaf(
    *,
    none_is_leaf: bool = False,
    namespace: str = "",  # unused
) -> PyTreeSpec:
    return PyTreeSpec(
        (),
        None,
        None,
        (),
        None,
        none_is_leaf=none_is_leaf,
        namespace="",
    )


@substitute_in_graph(  # type: ignore[arg-type]
    optree.treespec_tuple,
    # We need to disable constant folding here because we want the function to reference the
    # PyTreeSpec class defined above, not the one in the C++ module.
    can_constant_fold_through=False,
)
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 467-499
```python
def treespec_tuple(
    iterable: Iterable[PyTreeSpec] = (),
    /,
    *,
    none_is_leaf: bool = False,
    namespace: str = "",
) -> PyTreeSpec:
    children = tuple(iterable)
    if any(not _is_pytreespec_instance(child) for child in children):
        raise ValueError(f"Expected a tuple of PyTreeSpecs, got: {children!r}.")
    if any(child.none_is_leaf != none_is_leaf for child in children):
        raise ValueError(
            "All children PyTreeSpecs must have the same `none_is_leaf` value "
            f"as the parent; expected {none_is_leaf}, got: {children!r}.",
        )
    if any(child.namespace not in (namespace, "") for child in children):
        raise ValueError(
            "All children PyTreeSpecs must have the same `namespace` value "
            f"as the parent; expected {namespace!r}, got: {children!r}.",
        )
    handler = optree.register_pytree_node.get(tuple, namespace=namespace)
    assert handler is not None
    return PyTreeSpec(
        tuple(children),
        tuple,
        None,
        tuple(range(len(children))),
        handler.unflatten_func,
        none_is_leaf=none_is_leaf,
        namespace=namespace,
    )
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 500-527
```python
@substitute_in_graph(  # type: ignore[arg-type]
    optree.treespec_dict,
    # We need to disable constant folding here because we want the function to reference the
    # PyTreeSpec class defined above, not the one in the C++ module.
    can_constant_fold_through=False,
)
def treespec_dict(
    mapping: Mapping[Any, PyTreeSpec] | Iterable[tuple[Any, PyTreeSpec]] = (),
    /,
    *,
    none_is_leaf: bool = False,
    namespace: str = "",
    **kwargs: PyTreeSpec,
) -> PyTreeSpec:
    dct = dict(mapping, **kwargs)
    if any(not _is_pytreespec_instance(child) for child in dct.values()):
        raise ValueError(f"Expected a dictionary of TreeSpecs, got: {dct!r}.")
    if any(child.none_is_leaf != none_is_leaf for child in dct.values()):
        raise ValueError(
            "All children PyTreeSpecs must have the same `none_is_leaf` value "
            f"as the parent; expected {none_is_leaf}, got: {dct!r}.",
        )
    if any(child.namespace not in (namespace, "") for child in dct.values()):
        raise ValueError(
            "All children PyTreeSpecs must have the same `namespace` value "
            f"as the parent; expected {namespace!r}, got: {dct!r}.",
        )
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 528-562
```python
    (
        children,
        metadata,
        entries,
        unflatten_func,
    ) = optree.tree_flatten_one_level(  # type: ignore[assignment,var-annotated]
        dct,  # type: ignore[arg-type]
        none_is_leaf=none_is_leaf,
        namespace=namespace,
    )
    return PyTreeSpec(
        tuple(children),  # type: ignore[arg-type]
        dict,
        metadata,
        entries,
        unflatten_func,  # type: ignore[arg-type]
        none_is_leaf=none_is_leaf,
        namespace=namespace,
    )


@substitute_in_graph(  # type: ignore[arg-type]
    optree.tree_flatten,
    # We need to disable constant folding here because we want the function to reference the
    # PyTreeSpec class defined above, not the one in the C++ module.
    can_constant_fold_through=False,
)
def tree_flatten(
    tree: PyTree,
    /,
    is_leaf: Callable[[PyTree], bool] | None = None,
    *,
    none_is_leaf: bool = False,
    namespace: str = "",
) -> tuple[list[Any], PyTreeSpec]:
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 563-604
```python
    def helper(node: PyTree, leaves: list[Any]) -> PyTreeSpec:
        if tree_is_leaf(
            node,
            is_leaf=is_leaf,
            none_is_leaf=none_is_leaf,
            namespace=namespace,
        ):
            leaves.append(node)
            return PyTreeSpec(
                (),
                None,
                None,
                (),
                None,
                none_is_leaf=none_is_leaf,
                namespace=namespace,
            )

        (
            children,
            metadata,
            entries,
            unflatten_func,
        ) = optree.tree_flatten_one_level(
            node,
            is_leaf=is_leaf,
            none_is_leaf=none_is_leaf,
            namespace=namespace,
        )

        # Recursively flatten the children
        subspecs = tuple(helper(child, leaves) for child in children)
        return PyTreeSpec(
            subspecs,
            type(node),
            metadata,
            entries,
            unflatten_func,  # type: ignore[arg-type]
            none_is_leaf=none_is_leaf,
            namespace=namespace,
        )  # type: ignore[arg-type]
```
- **EN**: Defines the `tree_flatten` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`tree_flatten` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 605-636
```python
    leaves: list[Any] = []
    treespec = helper(tree, leaves)
    return leaves, treespec


@substitute_in_graph(  # type: ignore[arg-type]
    optree._C.flatten,
    # We need to disable constant folding here because we want the function to reference the
    # PyTreeSpec class defined above, not the one in the C++ module.
    can_constant_fold_through=False,
)
def _C_flatten(
    tree: PyTree,
    /,
    leaf_predicate: Callable[[PyTree], bool] | None = None,
    none_is_leaf: bool = False,
    namespace: str = "",
) -> tuple[list[Any], PyTreeSpec]:
    return tree_flatten(  # type: ignore[return-value]
        tree,
        is_leaf=leaf_predicate,
        none_is_leaf=none_is_leaf,
        namespace=namespace,
    )


@substitute_in_graph(  # type: ignore[arg-type]
    optree.tree_flatten_with_path,
    # We need to disable constant folding here because we want the function to reference the
    # PyTreeSpec class defined above, not the one in the C++ module.
    can_constant_fold_through=False,
)
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 637-674
```python
def tree_flatten_with_path(
    tree: PyTree,
    /,
    is_leaf: Callable[[PyTree], bool] | None = None,
    *,
    none_is_leaf: bool = False,
    namespace: str = "",
) -> tuple[list[tuple[Any, ...]], list[Any], PyTreeSpec]:
    leaves, treespec = tree_flatten(
        tree,
        is_leaf=is_leaf,
        none_is_leaf=none_is_leaf,
        namespace=namespace,
    )
    return treespec.paths(), leaves, treespec  # type: ignore[return-value]


@substitute_in_graph(  # type: ignore[arg-type]
    optree._C.flatten_with_path,
    # We need to disable constant folding here because we want the function to reference the
    # PyTreeSpec class defined above, not the one in the C++ module.
    can_constant_fold_through=False,
)
def _C_flatten_with_path(
    tree: PyTree,
    /,
    leaf_predicate: Callable[[PyTree], bool] | None = None,
    none_is_leaf: bool = False,
    namespace: str = "",
) -> tuple[list[tuple[Any, ...]], list[Any], PyTreeSpec]:
    return tree_flatten_with_path(  # type: ignore[return-value]
        tree,
        is_leaf=leaf_predicate,
        none_is_leaf=none_is_leaf,
        namespace=namespace,
    )
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 675-715
```python
@substitute_in_graph(  # type: ignore[arg-type]
    optree.tree_structure,
    # We need to disable constant folding here because we want the function to reference the
    # PyTreeSpec class defined above, not the one in the C++ module.
    can_constant_fold_through=False,
)
def tree_structure(
    tree: PyTree,
    /,
    is_leaf: Callable[[PyTree], bool] | None = None,
    *,
    none_is_leaf: bool = False,
    namespace: str = "",
) -> PyTreeSpec:
    return tree_flatten(  # type: ignore[return-value]
        tree,
        is_leaf=is_leaf,
        none_is_leaf=none_is_leaf,
        namespace=namespace,
    )[1]


@substitute_in_graph(  # type: ignore[arg-type]
    optree.tree_unflatten,
    # We need to disable constant folding here because we want the function to reference the
    # PyTreeSpec class defined above, not the one in the C++ module.
    can_constant_fold_through=False,
)
def tree_unflatten(treespec: PyTreeSpec, leaves: Iterable[Any]) -> PyTree:
    if not _is_pytreespec_instance(treespec):
        raise TypeError(
            f"Expected `treespec` to be an instance of "
            f"PyTreeSpec but got item of type {type(treespec)}."
        )
    return treespec.unflatten(leaves)


_none_registration = optree.register_pytree_node.get(type(None))
assert _none_registration is not None
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 716-745
```python
@substitute_in_graph(  # type: ignore[arg-type]
    _none_registration.unflatten_func,
    can_constant_fold_through=True,
    skip_signature_check=True,
)
def none_unflatten(_: None, children: Iterable[_T], /) -> None:
    if len(list(children)) != 0:
        raise ValueError("Expected no children.")
    return None


with optree.dict_insertion_ordered(False, namespace="torch"):
    _dict_registration = optree.register_pytree_node.get(dict)
    assert _dict_registration is not None


@substitute_in_graph(  # type: ignore[arg-type]
    _dict_registration.flatten_func,
    can_constant_fold_through=True,
    skip_signature_check=True,
)
def dict_flatten(
    dct: dict[_KT, _VT], /
) -> tuple[list[_VT], tuple[list[_KT], list[_KT]], tuple[_KT, ...]]:
    sorted_keys = optree.utils.total_order_sorted(dct)
    values = [dct[key] for key in sorted_keys]
    original_keys = list(dct)
    return values, (original_keys, sorted_keys), tuple(sorted_keys)
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 746-759
```python
@substitute_in_graph(  # type: ignore[arg-type]
    _dict_registration.unflatten_func,
    can_constant_fold_through=True,
    skip_signature_check=True,
)
def dict_unflatten(
    metadata: tuple[list[_KT], list[_KT]],
    values: Iterable[_VT],
    /,
) -> dict[_KT, _VT]:
    original_keys, sorted_keys = metadata
    d = dict.fromkeys(original_keys)
    d.update(zip(sorted_keys, values, strict=True))
    return d  # type: ignore[return-value]
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

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
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch.utils._cxx_pytree`, `torch.utils._pytree`, `..decorators`
- **Standard library / 标准库**: `__future__`, `collections`, `dataclasses`, `typing`, `builtins`, `collections.abc`, `typing_extensions`
- **Third-party packages / 第三方包**: `optree`, `optree._C`, `optree.utils`
- **Primary symbols / 核心符号**: `__all__`, `_T`, `_KT`, `_VT`, `_`, `tree_is_leaf`, `tree_iter`, `tree_leaves`, `_Asterisk`, `PyTreeSpec`, `_is_pytreespec_instance`, `treespec_leaf`
- **Relationship summary / 关系总结**: Dependencies center on graph capture, guard checking, bytecode analysis, and backend handoff. / 依赖关系主要围绕图捕获、守卫检查、字节码分析和后端交接展开。
