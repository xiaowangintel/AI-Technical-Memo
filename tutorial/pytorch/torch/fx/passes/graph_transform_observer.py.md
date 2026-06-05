# graph_transform_observer.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/passes/graph_transform_observer.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Collects graph-to-graph optimization, rewriting, and analysis passes for FX IR.
- **Purpose (CN)**: 汇集针对 FX 中间表示的图到图优化、改写与分析 passes。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
```python
import os
from collections.abc import Callable
from types import TracebackType
from typing import TypeVar

from torch.fx import Graph, Node
from torch.fx._compatibility import compatibility
from torch.fx.graph_module import GraphModule
from torch.fx.traceback import NodeSource, NodeSourceAction


T = TypeVar("T")


from .graph_drawer import FxGraphDrawer
```
- **EN**: This range initializes module-level constants or registries that later code reuses to build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这一段初始化模块级常量或注册表，供后续代码重复使用以构建、遍历或改写图结构及其元数据。

### Lines 18-29
```python
__all__ = ["GraphTransformObserver"]


@compatibility(is_backward_compatible=False)
class GraphTransformObserver:
    __pass_count = 0

    def __init__(
        self,
        gm: GraphModule,
        passname: str,
        subsystem: str | None = None,
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 30-45
```python
        log_url: str | None = None,
    ) -> None:
        """
        log_url is inferred to be torch._inductor.config.trace.log_url_for_graph_xform unless otherwise specified
        """
        from torch._inductor import config as inductor_config

        self.gm = gm
        self.passname = passname
        self.subsystem = subsystem

        if log_url is None:
            log_url = inductor_config.trace.log_url_for_graph_xform

        self.log_url = log_url
```
- **EN**: This block continues `GraphTransformObserver.__init__` and works to build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `GraphTransformObserver.__init__`，用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理。

### Lines 46-62
```python
        self.active = (
            self.log_url is not None
            or inductor_config.trace.provenance_tracking_level == 1
        )

        if self.active:
            self.erased_nodes: set[str] = set()
            self.created_nodes: set[str] = set()
            self.name_to_node: dict[str, Node] = {}
            # record graph modules deepcopied from self.gm, so we can remove hooks on them when exiting the context
            self.copied_gms: list[GraphModule] = []

            self._node_creation_hook = self.get_node_creation_hook()
            self._node_erase_hook = self.get_node_erase_hook()
            self._node_replace_hook = self.get_node_replace_hook()
            self._deepcopy_hook = self.get_deepcopy_hook()
```
- **EN**: This block continues `GraphTransformObserver.__init__` and works to build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `GraphTransformObserver.__init__`，用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理。

### Lines 63-78
```python
        # If log_url is None, we don't log anything
        if self.log_url is None:
            return
        GraphTransformObserver.__pass_count += 1

        self.input_dot_graph = FxGraphDrawer(
            self.gm,
            self.passname,
            ignore_getattr=True,
            ignore_parameters_and_buffers=True,
        ).get_dot_graph()

    @classmethod
    def get_current_pass_count(cls) -> int:
        return cls.__pass_count
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 79-94
```python
    def apply_gm_pass(self, pass_fn: Callable[[GraphModule], T]) -> T | None:
        from torch._dynamo.utils import dynamo_timed

        with self:
            if self._check_disable_pass():
                return None
            with dynamo_timed(
                f"pass.{self.subsystem}.{self.passname}"
                if self.subsystem
                else f"pass.{self.passname}"
            ):
                return pass_fn(self.gm)

    def apply_graph_pass(self, pass_fn: Callable[[Graph], T]) -> T | None:
        from torch._dynamo.utils import dynamo_timed
```
- **EN**: Declares `GraphTransformObserver`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `GraphTransformObserver`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 95-110
```python
        with self:
            if self._check_disable_pass():
                return None
            with dynamo_timed(
                f"pass.{self.subsystem}.{self.passname}"
                if self.subsystem
                else f"pass.{self.passname}"
            ):
                return pass_fn(self.gm.graph)

    def _check_disable_pass(self) -> bool:
        from torch._inductor import config as inductor_config

        if self.passname.upper() in inductor_config.disabled_passes.upper():
            return True
```
- **EN**: Declares `GraphTransformObserver`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `GraphTransformObserver`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 111-128
```python
        if self.subsystem is None:
            return False

        debug_info = lambda: self.passname  # noqa: E731
        from torch._inductor.compiler_bisector import CompilerBisector

        return CompilerBisector.disable_subsystem(
            "inductor", self.subsystem, debug_info
        )

    def __enter__(self) -> "GraphTransformObserver":
        if not self.active:
            return self
        self.gm._register_create_node_hook(self._node_creation_hook)
        self.gm._register_erase_node_hook(self._node_erase_hook)
        self.gm._register_replace_node_hook(self._node_replace_hook)
        self.gm._register_deepcopy_hook(self._deepcopy_hook)
```
- **EN**: Declares `GraphTransformObserver`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `GraphTransformObserver`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 129-140
```python
        self.erased_nodes.clear()
        self.created_nodes.clear()
        self.name_to_node.clear()
        self.copied_gms.clear()

        for node in self.gm.graph.nodes:
            self.name_to_node[node.name] = node

        return self

    def __exit__(
        self,
```
- **EN**: Declares `GraphTransformObserver`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `GraphTransformObserver`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 141-155
```python
        type: type[BaseException] | None,
        value: BaseException | None,
        tb: TracebackType | None,
    ) -> None:
        if not self.active:
            return
        for gm in self.copied_gms + [self.gm]:
            gm._unregister_create_node_hook(self._node_creation_hook)
            gm._unregister_erase_node_hook(self._node_erase_hook)
            gm._unregister_replace_node_hook(self._node_replace_hook)
            gm._unregister_deepcopy_hook(self._deepcopy_hook)

        if self.log_url is None:
            return
```
- **EN**: This block continues `GraphTransformObserver.__exit__` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `GraphTransformObserver.__exit__`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 156-170
```python
        if len(self.created_nodes) > 0 or len(self.erased_nodes) > 0:
            for e in self.input_dot_graph.get_node_list():
                if e.get_name() in self.erased_nodes:
                    e.obj_dict["attributes"]["fillcolor"] = "yellow"
                else:
                    e.obj_dict["attributes"]["fillcolor"] = "grey"
            if self.log_url is None:
                raise AssertionError("log_url is not set")
            self.input_dot_graph.write(
                os.path.join(
                    self.log_url,
                    f"pass_{GraphTransformObserver.__pass_count}_{self.passname}_input_graph.dot",
                )
            )
```
- **EN**: This block continues `GraphTransformObserver.__exit__` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `GraphTransformObserver.__exit__`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 171-188
```python
            output_dot_graph = FxGraphDrawer(
                self.gm,
                self.passname,
                ignore_getattr=True,
                ignore_parameters_and_buffers=True,
            ).get_dot_graph()
            for e in output_dot_graph.get_node_list():
                if e.get_name() in self.created_nodes:
                    e.obj_dict["attributes"]["fillcolor"] = "yellow"
                else:
                    e.obj_dict["attributes"]["fillcolor"] = "grey"
            output_dot_graph.write(
                os.path.join(
                    self.log_url,
                    f"pass_{GraphTransformObserver.__pass_count}_{self.passname}_output_graph.dot",
                )
            )
```
- **EN**: This block continues `GraphTransformObserver` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `GraphTransformObserver`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 189-202
```python
    def get_node_creation_hook(self) -> Callable[[Node], None]:
        # We have to return a function instead of using a class method directly
        # to avoid max recursion issue when deepcopy a graph module within the context manager.
        def on_node_creation(node: Node) -> None:
            self.created_nodes.add(node.name)
            self.name_to_node[node.name] = node
            source = NodeSource(None, self.passname, NodeSourceAction.CREATE)
            if "from_node" not in node.meta:
                node.meta["from_node"] = [source]
            else:
                node.meta["from_node"].append(source)

        return on_node_creation
```
- **EN**: Declares `GraphTransformObserver`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `GraphTransformObserver`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 203-220
```python
    def get_node_erase_hook(self) -> Callable[[Node], None]:
        def on_node_erase(node: Node) -> None:
            self.erased_nodes.add(node.name)
            self.name_to_node.pop(node.name, None)

        return on_node_erase

    def get_node_replace_hook(self) -> Callable[[Node, str, Node], None]:
        def on_node_replace(old: Node, new: str, user: Node) -> None:
            # Update node meta when replacing old node with new node
            new_node = self.name_to_node.get(new, None)

            if not new_node:
                return

            if not isinstance(new_node, Node):
                raise AssertionError(f"Expected Node, got {type(new_node)}")
```
- **EN**: Declares `GraphTransformObserver`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `GraphTransformObserver`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 221-235
```python
            # replace hook is called once for each user of old
            # this avoids adding duplicated source nodes
            added_nodes = {s.name for s in new_node.meta.get("from_node", [])}
            if old.name in added_nodes:
                return

            action = [NodeSourceAction.REPLACE]
            if new_node.name in self.created_nodes:
                action.append(NodeSourceAction.CREATE)

            def created_this_pass(source: NodeSource) -> bool:
                return source.pass_name == self.passname and source.action == [
                    NodeSourceAction.CREATE
                ]
```
- **EN**: Defines the `GraphTransformObserver.get_node_replace_hook.on_node_replace` method; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`GraphTransformObserver.get_node_replace_hook.on_node_replace` 方法；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 236-252
```python
            # remove redundant source added on node creation
            new_from_node = new_node.meta.get("from_node", [])
            new_from_node = [
                source for source in new_from_node if not created_this_pass(source)
            ]

            # add new source
            new_node_source = NodeSource(old, self.passname, action)
            new_from_node.append(new_node_source)
            new_node.meta["from_node"] = new_from_node

        return on_node_replace

    def get_deepcopy_hook(self) -> Callable[[GraphModule], None]:
        def on_deepcopy(gm: GraphModule) -> None:
            self.copied_gms.append(gm)
```
- **EN**: Declares `GraphTransformObserver`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `GraphTransformObserver`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 253-253
```python
        return on_deepcopy
```
- **EN**: This block continues `GraphTransformObserver.get_deepcopy_hook` and works to manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `GraphTransformObserver.get_deepcopy_hook`，用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Tracing / 跟踪**
  - EN: The file captures Python-level operations and turns them into analyzable graph structures.
  - CN: 该文件会捕获 Python 层操作，并将其转换为可分析的图结构。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Compiler handoff / 编译器交接**
  - EN: The implementation prepares captured work for backend compilation or execution.
  - CN: 该实现会把捕获到的工作整理后交给后端编译或执行。
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch.fx`, `torch.fx._compatibility`, `torch.fx.graph_module`, `torch.fx.traceback`, `.graph_drawer`, `torch._inductor`, `torch._dynamo.utils`, `torch._inductor.compiler_bisector`
- **Standard library / 标准库**: `os`, `collections.abc`, `types`, `typing`
- **Primary symbols / 核心符号**: `T`, `__all__`, `GraphTransformObserver`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
