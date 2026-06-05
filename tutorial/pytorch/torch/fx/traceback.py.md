# traceback.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/traceback.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Implements torch.fx symbolic tracing, graph IR manipulation, and transformation utilities. Key symbols exposed here include `_P`, `_R`, `__all__`, `GRADIENT_ACC_SPECIAL_STACK`.
- **Purpose (CN)**: 实现 torch.fx 的符号跟踪、图中间表示操作与变换工具。 这里暴露的关键符号包括 `_P`, `_R`, `__all__`, `GRADIENT_ACC_SPECIAL_STACK`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-22
```python
import copy
import logging
import traceback
from collections import defaultdict
from collections.abc import Callable, Iterator
from contextlib import contextmanager
from enum import Enum
from typing import Any, Optional, ParamSpec, TypeVar, Union

from torch._utils_internal import signpost_event

from ._compatibility import compatibility
from .graph import Graph
from .graph_module import GraphModule
from .node import Node


_P = ParamSpec("_P")
_R = TypeVar("_R")

log = logging.getLogger(__name__)
```
- **EN**: This range initializes module-level constants or registries that later code reuses to build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这一段初始化模块级常量或注册表，供后续代码重复使用以构建、遍历或改写图结构及其元数据。

### Lines 23-47
```python
__all__ = [
    "annotate",
    "annotate_fn",
    "preserve_node_meta",
    "has_preserved_node_meta",
    "set_stack_trace",
    "set_grad_fn_seq_nr",
    "reset_grad_fn_seq_nr",
    "format_stack",
    "set_current_meta",
    "get_current_meta",
    "NodeSource",
    "NodeSourceAction",
    "get_graph_provenance_json",
    "set_current_replay_node",
    "get_current_replay_node",
]

current_meta: dict[str, Any] = {}
current_replay_node: Node | None = None
# Preserve the node meta fields in torch.fx.proxy._COPY_META_FIELDS
should_preserve_node_meta = False
# Preserve the "seq_nr" node meta field
_should_preserve_node_meta = False
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 48-73
```python
GRADIENT_ACC_SPECIAL_STACK = (
    "Gradient addition node due to multiple use of tensor around:"
)
# =============================================================================
# FX Metadata Registry for Memory Profiler
# =============================================================================
# Global in-memory registry for FX metadata
# Maps module_name -> metadata dict containing lineno_map and node_metadata
_FX_METADATA_REGISTRY: dict[str, dict[str, Any]] = {}


def _register_fx_metadata(module_name: str, metadata: dict[str, Any]) -> None:
    """
    Register FX metadata in the global in-memory registry.

    This is called automatically during graph module compilation to store metadata
    for later use by memory profiler augmentation.

    Args:
        module_name: The module identifier (content-addressed filename)
        metadata: Metadata dict containing lineno_map, node_metadata, and source_code
    """
    # TODO: add logging to tlparse
    _FX_METADATA_REGISTRY[module_name] = metadata
```
- **EN**: This range initializes module-level constants or registries that later code reuses to build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这一段初始化模块级常量或注册表，供后续代码重复使用以构建、遍历或改写图结构及其元数据。

### Lines 74-99
```python
@compatibility(is_backward_compatible=False)
class NodeSourceAction(Enum):
    CREATE = "create"
    REPLACE = "replace"


@compatibility(is_backward_compatible=False)
class NodeSource:
    """
    NodeSource is a data structure that contains the provenance information of a node.
    If node `a` is created from node `b`, then `a.meta["from_node"]` may contain NodeSource(b).
    """

    class NodeInfo:
        def __init__(self, name: str, target: str, graph_id: int) -> None:
            self.name = name
            self.target = target
            self.graph_id = graph_id

    pass_name: str
    action: list["NodeSourceAction"]
    from_node: list["NodeSource"]
    node_info: Optional["NodeInfo"]
    _dict: dict[str, Any] | None
    _action_string: str | None
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 100-117
```python
    def __init__(
        self,
        node: Node | None,
        pass_name: str = "",
        action: Union["NodeSourceAction", list["NodeSourceAction"]] | None = None,
    ) -> None:
        self.pass_name = pass_name

        if action is None:
            action = []
        elif not isinstance(action, list):
            action = [action]
        for a in action:
            if not isinstance(a, NodeSourceAction):
                raise AssertionError(f"Expected NodeSourceAction, got {type(a)}")
        self.action = action
        if node:
            self.node_info = self.NodeInfo(
```
- **EN**: Defines the `NodeSource.__init__` method; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`NodeSource.__init__` 方法；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 118-144
```python
                name=node.name, target=str(node.target), graph_id=id(node.graph)
            )
            self.from_node = (
                copy.deepcopy(node.meta["from_node"])
                if "from_node" in node.meta
                else []
            )
        else:
            self.node_info = None
            self.from_node = []

        # cache the action string and dict representation for performance.
        self._action_string: str | None = None
        self._dict: dict[str, Any] | None = None

    @property
    def name(self) -> str:
        return self.node_info.name if self.node_info else ""

    @property
    def target(self) -> str:
        return self.node_info.target if self.node_info else ""

    @property
    def graph_id(self) -> int:
        return self.node_info.graph_id if self.node_info else -1
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 145-165
```python
    def __repr__(self) -> str:
        return self.print_readable()

    def _get_action_string(self) -> str:
        if self._action_string is None:
            self._action_string = "+".join([a.name.lower() for a in self.action])
        return self._action_string

    def print_readable(self, indent: int = 0) -> str:
        if indent > 9:
            return ""
        result = ""
        action_string = self._get_action_string()
        result += (
            " " * indent * 4
            + f"(name={self.name}, pass_name={self.pass_name}, action={action_string}, graph_id={self.graph_id})\n"
        )
        for item in self.from_node:
            result += item.print_readable(indent + 1)
        return result
```
- **EN**: Declares `NodeSource`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `NodeSource`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 166-187
```python
    def to_dict(self) -> dict[str, Any]:
        if self._dict is None:
            # Convert the object to a dictionary
            action_string = self._get_action_string()
            self._dict = {
                "name": self.name,
                "target": self.target,
                "graph_id": self.graph_id,
                "pass_name": self.pass_name,
                "action": action_string,
                "from_node": [node.to_dict() for node in self.from_node],
            }

        if self._dict is None:
            raise AssertionError("_dict is None after initialization")
        return self._dict

    def __eq__(self, other: object) -> bool:
        if not isinstance(other, NodeSource):
            return False
        return self.to_dict() == other.to_dict()
```
- **EN**: Declares `NodeSource`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `NodeSource`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 188-213
```python
    def __hash__(self) -> int:
        # Create a hash based on the dictionary representation
        # We need to convert the dict to a hashable form
        def _make_hashable(obj: Any) -> Any:
            if isinstance(obj, dict):
                return tuple(sorted((k, _make_hashable(v)) for k, v in obj.items()))
            elif isinstance(obj, list):
                return tuple(_make_hashable(item) for item in obj)
            else:
                return obj

        return hash(_make_hashable(self.to_dict()))

    @classmethod
    def _from_dict(cls, d: dict[str, Any] | None) -> Optional["NodeSource"]:
        """
        Recursively deserialize from_node metadata from dictionary data.
        It is used to deserialize the from_node field from serialized metadata.
        Please use constructor NodeSource(node, ...) to create a NodeSource object.
        """
        if d is None:
            return None

        if not isinstance(d, dict):
            raise AssertionError(f"Expected a dict, got {type(d)}")
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 214-235
```python
        # Create a NodeSource object directly without going through the constructor
        # to avoid issues with graph ID and node creation
        node_source = NodeSource.__new__(NodeSource)

        # Reset the cached properties
        node_source._action_string = None
        node_source._dict = None

        # Set the basic attributes
        node_source.pass_name = d.get("pass_name", "")

        # Parse action string back to NodeSourceAction enum list
        action_str = d.get("action", "")
        actions = []
        if action_str:
            for action_name in action_str.split("+"):
                if action_name.upper() == "CREATE":
                    actions.append(NodeSourceAction.CREATE)
                elif action_name.upper() == "REPLACE":
                    actions.append(NodeSourceAction.REPLACE)
        node_source.action = actions
```
- **EN**: This block continues `NodeSource._from_dict` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `NodeSource._from_dict`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 236-256
```python
        # Create the NodeInfo object directly
        if "name" in d and "target" in d and "graph_id" in d:
            node_info = NodeSource.NodeInfo(
                d.get("name", ""), d.get("target", ""), d.get("graph_id", -1)
            )
            node_source.node_info = node_info
        else:
            node_source.node_info = None

        # Recursively deserialize nested from_node
        if d.get("from_node", None) is not None:
            node_source.from_node = [
                result
                for fn in d.get("from_node", [])
                if (result := cls._from_dict(fn)) is not None
            ]
        else:
            node_source.from_node = []
        return node_source
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 257-281
```python
@compatibility(is_backward_compatible=False)
@contextmanager
def preserve_node_meta(enable: bool = True) -> Iterator[None]:
    global should_preserve_node_meta
    global current_meta
    saved_should_preserve_node_meta = should_preserve_node_meta
    # Shallow copy is OK since fields of current_meta are not mutated
    saved_current_meta = current_meta.copy()
    try:
        should_preserve_node_meta = enable
        yield
    finally:
        should_preserve_node_meta = saved_should_preserve_node_meta
        current_meta = saved_current_meta


@contextmanager
def _preserve_node_seq_nr(preserve_seq_nr: bool = True) -> Iterator[None]:
    """
    Temporarily enables or disables the preservation of node.meta["seq_nr"] in the
    tracing context.
    """
    global _should_preserve_node_meta
    saved = _should_preserve_node_meta
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 282-301
```python
    try:
        _should_preserve_node_meta = preserve_seq_nr
        yield
    finally:
        _should_preserve_node_meta = saved


@compatibility(is_backward_compatible=False)
def set_stack_trace(stack: list[str]) -> None:
    global current_meta

    if should_preserve_node_meta:
        if stack:
            current_meta["stack_trace"] = "".join(stack)
        else:
            # when the stack is empty, we explicitly clear the stack_trace to avoid
            # propagating it to future node.˙
            current_meta.pop("stack_trace", None)
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 302-325
```python
@compatibility(is_backward_compatible=False)
@contextmanager
def annotate(annotation_dict: dict[str, Any]) -> Iterator[None]:
    """
    Temporarily adds custom annotations to the current tracing context.
    The fx_node produced from this tracing context will have the
    custom annotations in node.metadata["custom"] field.

    This context manager allows you to insert arbitrary metadata into the PT2
    tracing system by updating the global `current_meta["custom"]` dictionary.
    The annotations are automatically reverted after the context exits.

    Gradient accumulation nodes will not be annotated.

    This is intended for advanced users who need to attach additional metadata to the fx nodes
    (e.g., for debugging, analysis, or external tooling) during export tracing.

    Note:
        This API is **not backward compatible** and may evolve in future releases.

    Note:
        This API is not compatible with fx.symbolic_trace or jit.trace. It's intended
        to be used with PT2 family of tracers, e.g. torch.export and dynamo.
```
- **EN**: These decorators register or transform the following definition so it can track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够跟踪符号形状约束以及依赖形状的推理。

### Lines 326-345
```python
    Args:
        annotation_dict (dict): A dictionary of custom key-value pairs to inject
            into the FX trace metadata.

    Example:
        After exiting the context, custom annotations are removed.

        >>> with annotate({"source": "custom_pass", "tag": 42}):
        ...     pass  # Your computation here
    """

    global current_meta

    has_custom = "custom" in current_meta
    old_custom = copy.copy(current_meta.get("custom", {}))

    try:
        if not has_custom:
            current_meta["custom"] = dict[str, Any]()
```
- **EN**: This block continues `annotate` and works to trace Python execution into an intermediate graph representation. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `annotate`，用于将 Python 执行过程跟踪为中间图表示。 同时它还会根据运行时条件分支处理。

### Lines 346-371
```python
        # Update with all key-value pairs from the input dict
        current_meta["custom"].update(annotation_dict)
        yield
    finally:
        if has_custom:
            # Restore the original custom dict
            current_meta["custom"] = old_custom
        else:
            del current_meta["custom"]


@compatibility(is_backward_compatible=False)
def annotate_fn(
    annotation_dict: dict[str, Any],
) -> Callable[[Callable[_P, _R]], Callable[_P, _R]]:
    """
    A decorator that wraps a function with the annotate context manager.
    Use this when you want to annotate an entire function instead of a specific code block.

    Note:
        This API is **not backward compatible** and may evolve in future releases.

    Note:
        This API is not compatible with fx.symbolic_trace or jit.trace. It's intended
        to be used with PT2 family of tracers, e.g. torch.export and dynamo.
```
- **EN**: These decorators register or transform the following definition so it can track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够跟踪符号形状约束以及依赖形状的推理。

### Lines 372-397
```python
    Args:
        annotation_dict (dict): A dictionary of custom key-value pairs to inject
            into the FX trace metadata for all operations in the function.

    Example:
        All operations in my_function will have {"pp_stage": 1} in their metadata.

        >>> @annotate_fn({"pp_stage": 1})
        ... def my_function(x):
        ...     return x + 1
    """
    from functools import wraps

    def decorator(func: Callable[_P, _R]) -> Callable[_P, _R]:
        @wraps(func)
        # NB: Do not annotate with _P.args/_P.kwargs here. Dynamo guards on
        # the identity of ParamSpec annotation objects, causing guard failures.
        def wrapper(*args: Any, **kwargs: Any) -> Any:
            with annotate(annotation_dict):
                return func(*args, **kwargs)

        return wrapper  # type: ignore[return-value]

    return decorator
```
- **EN**: These decorators register or transform the following definition so it can enforce guards that validate whether cached compiled code can be reused.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够实施守卫检查以判断缓存的编译代码能否复用。

### Lines 398-422
```python
@contextmanager
def _set_autograd_backward(enable: bool = True) -> Iterator[None]:
    global current_meta

    had_autograd_backward = "autograd_backward" in current_meta
    old_autograd_backward = current_meta.get("autograd_backward", False)

    if enable:
        _mark_autograd_backward()
    try:
        yield
    finally:
        if had_autograd_backward:
            current_meta["autograd_backward"] = old_autograd_backward
        else:
            _reset_autograd_backward()


@compatibility(is_backward_compatible=False)
def _mark_autograd_backward() -> None:
    global current_meta

    current_meta["autograd_backward"] = True
```
- **EN**: These decorators register or transform the following definition so it can connect execution with differentiation-aware logic.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将执行过程与可微分逻辑连接起来。

### Lines 423-441
```python
@compatibility(is_backward_compatible=False)
def _reset_autograd_backward() -> None:
    global current_meta

    current_meta.pop("autograd_backward", None)


@compatibility(is_backward_compatible=False)
def set_grad_fn_seq_nr(seq_nr: int) -> None:
    global current_meta

    if should_preserve_node_meta:
        # The seq_nr is captured by eager mode in the grad_fn during forward
        current_meta["grad_fn_seq_nr"] = current_meta.get("grad_fn_seq_nr", []) + [
            seq_nr
        ]
        current_meta["in_grad_fn"] = current_meta.get("in_grad_fn", 0) + 1
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 442-467
```python
@compatibility(is_backward_compatible=False)
def reset_grad_fn_seq_nr() -> None:
    # NB: reset state properly, this would be helpful towards supporting
    #     reentrant autograd if we actually wanted to do that.
    global current_meta
    if should_preserve_node_meta:
        current_level = current_meta.get("in_grad_fn", 0)
        if current_level <= 0:
            raise AssertionError(f"Expected current_level > 0, got {current_level}")
        if current_level == 1:
            del current_meta["in_grad_fn"]
            del current_meta["grad_fn_seq_nr"]
        else:
            current_meta["in_grad_fn"] = current_level - 1
            current_meta["grad_fn_seq_nr"] = current_meta["grad_fn_seq_nr"][:-1]


@compatibility(is_backward_compatible=False)
def format_stack() -> list[str]:
    if should_preserve_node_meta:
        return [current_meta.get("stack_trace", "")]
    else:
        # fallback to traceback.format_stack()
        return traceback.format_list(traceback.extract_stack()[:-1])
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 468-485
```python
@compatibility(is_backward_compatible=False)
def has_preserved_node_meta() -> bool:
    return should_preserve_node_meta


def _is_preserving_node_seq_nr() -> bool:
    return _should_preserve_node_meta


@compatibility(is_backward_compatible=False)
@contextmanager
def set_current_meta(node: Node, pass_name: str = "") -> Iterator[None]:
    global current_meta
    if should_preserve_node_meta and node.meta:
        saved_meta = current_meta
        try:
            current_meta = node.meta.copy()
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 486-504
```python
            # Update the "from_node" field in current_meta for provenance tracking.
            # Instead of appending, overwrite the "from_node" field because current_meta
            # will be assigned to the new node. The new NodeSource(node, ...) will
            # include the information from the previous current_meta["from_node"].
            current_meta["from_node"] = [
                NodeSource(node, pass_name, NodeSourceAction.CREATE)
            ]
            yield
        finally:
            current_meta = saved_meta
    else:
        yield


@compatibility(is_backward_compatible=False)
def get_current_meta() -> dict[str, Any]:
    return current_meta
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 505-529
```python
@compatibility(is_backward_compatible=False)
@contextmanager
def set_current_replay_node(node: Node | None) -> Iterator[None]:
    """
    Set the currently replay node. If `current_replay_node` is not None,
    then we're re-generating the `current_replay_node` in FunctionalTensorMode.
    """
    # See [Note] annotation for more details.
    global current_replay_node
    saved_current_replay_node = current_replay_node
    try:
        current_replay_node = node
        yield
    finally:
        current_replay_node = saved_current_replay_node


@compatibility(is_backward_compatible=False)
def get_current_replay_node() -> Node | None:
    """
    Get the currently replay node
    """
    return current_replay_node
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 530-547
```python
@compatibility(is_backward_compatible=False)
def get_graph_provenance_json(graph: Graph) -> dict[str, Any]:
    """
    Given an fx.Graph, return a json that contains the provenance information of each node.
    """
    try:
        provenance_tracking_json = {}
        for node in graph.nodes:
            if node.op == "call_function":
                provenance_tracking_json[node.name] = (
                    [source.to_dict() for source in node.meta["from_node"]]
                    if "from_node" in node.meta
                    else []
                )
        return provenance_tracking_json
    except Exception as e:
        # Since this is just debugging, it should never interfere with regular
        # program execution, so we use this try-except to guard against any error
```
- **EN**: These decorators register or transform the following definition so it can enforce guards that validate whether cached compiled code can be reused.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够实施守卫检查以判断缓存的编译代码能否复用。

### Lines 548-565
```python
        signpost_event(
            "inductor",
            "provenance_tracking_error",
            {
                "function": "get_graph_provenance_json",
                "error_msg": str(e),
                "stack_trace": traceback.format_exc(),
            },
        )
        return {}


def _get_custom_metadata(gm: GraphModule) -> str:
    if not isinstance(gm, GraphModule):
        raise AssertionError(f"Expected GraphModule, got {type(gm)}")

    def helper(gm: GraphModule) -> list[Any]:
        custom_metadata = []
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 566-591
```python
        for node in gm.graph.nodes:
            if hasattr(node, "meta") and node.meta.get("custom", None):
                custom_metadata.append((node.op, node.name, node.meta["custom"]))
            if node.op == "get_attr" and isinstance(
                getattr(gm, node.target), GraphModule
            ):
                custom_metadata.append(
                    # pyrefly: ignore[bad-argument-type]
                    helper(getattr(gm, node.target))
                )
        return custom_metadata

    return "\n".join(str(x) for x in helper(gm))


def _get_ordered_seq_nr_groups(
    gm: GraphModule | list[GraphModule],
) -> list[list[str]]:
    """
    Group call_function nodes by seq_nr, order by seq_nr value,
    and return a list of lists of node names (sorted alphabetically).

    Args:
        gm: A single GraphModule or a list of GraphModules to process.
            When a list is provided, nodes from all graphs are grouped together.
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 592-609
```python
    Returns:
        A list of lists, where each inner list contains node names that share the same seq_nr,
        sorted alphabetically. The outer list is ordered by seq_nr value.
    """
    # Normalize input to a list
    if isinstance(gm, GraphModule):
        gms = [gm]
    else:
        gms = gm

    seq_nr_dict: dict[int, list[str]] = defaultdict(list)
    for graph_module in gms:
        for node in graph_module.graph.nodes:
            if node.op == "call_function":
                seq_nr = node.meta.get("seq_nr")
                if seq_nr is not None:
                    seq_nr_dict[seq_nr].append(node.name)
    # Sort by seq_nr and return list of sorted lists
```
- **EN**: This block continues `_get_ordered_seq_nr_groups` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_get_ordered_seq_nr_groups`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 610-610
```python
    return [sorted(seq_nr_dict[k]) for k in sorted(seq_nr_dict.keys())]
```
- **EN**: This block continues `_get_ordered_seq_nr_groups` and works to manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `_get_ordered_seq_nr_groups`，用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
- **Guards / 守卫**
  - EN: Runtime predicates preserve correctness when cached compiled graphs are reused.
  - CN: 运行时谓词用于在复用缓存编译图时保持正确性。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Tracing / 跟踪**
  - EN: The file captures Python-level operations and turns them into analyzable graph structures.
  - CN: 该文件会捕获 Python 层操作，并将其转换为可分析的图结构。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch._utils_internal`, `._compatibility`, `.graph`, `.graph_module`, `.node`
- **Standard library / 标准库**: `copy`, `logging`, `traceback`, `collections`, `collections.abc`, `contextlib`, `enum`, `typing`, `functools`
- **Primary symbols / 核心符号**: `_P`, `_R`, `__all__`, `GRADIENT_ACC_SPECIAL_STACK`, `_FX_METADATA_REGISTRY`, `_register_fx_metadata`, `NodeSourceAction`, `NodeSource`, `preserve_node_meta`, `_preserve_node_seq_nr`, `set_stack_trace`, `annotate`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
