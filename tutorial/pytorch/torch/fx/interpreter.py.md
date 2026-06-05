# interpreter.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/interpreter.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Executes or simulates FX graphs node by node for analysis and transformations.
- **Purpose (CN)**: 按节点执行或模拟 FX 图，以支持分析与变换。

## Line-by-Line Analysis / 逐行分析

### Lines 1-26
```python
import inspect
import logging
from collections.abc import Iterator
from contextlib import contextmanager
from typing import Any

import torch
import torch.fx.traceback as fx_traceback
from torch._logging import LazyString, trace_structured
from torch.hub import tqdm

from . import config
from ._compatibility import compatibility
from ._lazy_graph_module import _make_graph_module
from ._symbolic_trace import Tracer
from .graph import Graph
from .graph_module import GraphModule
from .node import Argument, map_aggregate, map_arg, Node, Target
from .proxy import Proxy


log = logging.getLogger(__name__)

__all__ = ["Interpreter", "Transformer"]
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 27-50
```python
def _format_fx_node(n: Node) -> str:
    """
    Format a torch.fx.Node into a human-readable string for debug logging.

    Args:
        n (torch.fx.Node): The FX node being executed.

    Returns:
        str: A formatted string describing the node operation, including its
        name, target, positional arguments, and keyword arguments.
    """
    module_prefix = getattr(n.target, "__module__", "")
    module_prefix = f"{module_prefix}." if module_prefix else ""

    # Handle positional and keyword arguments
    args = ", ".join(map(str, n.args))
    kwargs = ", ".join(f"{k}={v}" for k, v in n.kwargs.items())
    joined = ", ".join(filter(None, [args, kwargs]))

    return (
        f"{n.name} = {module_prefix}{getattr(n.target, '__name__', n.target)}({joined})"
    )
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果。

### Lines 51-76
```python
@compatibility(is_backward_compatible=True)
class Interpreter:
    """
    An Interpreter executes an FX graph Node-by-Node. This pattern
    can be useful for many things, including writing code
    transformations as well as analysis passes.

    Methods in the Interpreter class can be overridden to customize
    the behavior of execution. The map of overridable methods
    in terms of call hierarchy::

        run()
            +-- run_node
                +-- placeholder()
                +-- get_attr()
                +-- call_function()
                +-- call_method()
                +-- call_module()
                +-- output()

    Example:

        Suppose we want to swap all instances of ``torch.neg`` with
        ``torch.sigmoid`` and vice versa (including their ``Tensor``
        method equivalents). We could subclass Interpreter like so::
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 77-100
```python
            class NegSigmSwapInterpreter(Interpreter):
                def call_function(
                    self, target: Target, args: Tuple, kwargs: Dict
                ) -> Any:
                    if target is torch.sigmoid:
                        return torch.neg(*args, **kwargs)
                    return super().call_function(target, args, kwargs)

                def call_method(self, target: Target, args: Tuple, kwargs: Dict) -> Any:
                    if target == "neg":
                        call_self, *args_tail = args
                        return call_self.sigmoid(*args_tail, **kwargs)
                    return super().call_method(target, args, kwargs)


            def fn(x):
                return torch.sigmoid(x).neg()


            gm = torch.fx.symbolic_trace(fn)
            input = torch.randn(3, 4)
            result = NegSigmSwapInterpreter(gm).run(input)
            torch.testing.assert_close(result, torch.neg(input).sigmoid())
```
- **EN**: Declares `Interpreter`; this class packages state and methods that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 声明 `Interpreter`；该类封装了状态与方法，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 101-118
```python
    Args:
        module (torch.nn.Module): The module to be executed
        garbage_collect_values (bool): Whether to delete values after their last
            use within the Module's execution. This ensures optimal memory usage during
            execution. This can be disabled to, for example, examine all of the intermediate
            values in the execution by looking at the ``Interpreter.env`` attribute.
        graph (Optional[Graph]): If passed, the interpreter will execute this
            graph instead of `module.graph`, using the provided `module`
            argument to satisfy any requests for state.
    """

    @compatibility(is_backward_compatible=True)
    def __init__(
        self,
        module: torch.nn.Module,
        garbage_collect_values: bool = True,
        graph: Graph | None = None,
    ) -> None:
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 119-142
```python
        self.module = module
        self.submodules = dict(self.module.named_modules())
        if graph is not None:
            self.graph = graph
        else:
            self.graph = self.module.graph  # type: ignore[assignment]
        self.env: dict[Node, Any] = {}
        self.name = "Interpreter"
        self.garbage_collect_values = garbage_collect_values
        self.extra_traceback = True

        if self.garbage_collect_values:
            # Run through reverse nodes and record the first instance of a use
            # of a given node. This represents the *last* use of the node in the
            # execution order of the program, which we will use to free unused
            # values
            node_to_last_use: dict[Node, Node] = {}
            self.user_to_last_uses: dict[Node, list[Node]] = {}

            def register_last_uses(n: Node, user: Node) -> None:
                if n not in node_to_last_use:
                    node_to_last_use[n] = user
                    self.user_to_last_uses.setdefault(user, []).append(n)
```
- **EN**: Defines the `Interpreter.__init__` method; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`Interpreter.__init__` 方法；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 143-165
```python
            for node in reversed(self.graph.nodes):
                for n in node._input_nodes:
                    register_last_uses(n, node)

    @compatibility(is_backward_compatible=True)
    def run(
        self,
        *args: Any,
        initial_env: dict[Node, Any] | None = None,
        enable_io_processing: bool = True,
    ) -> Any:
        """
        Run `module` via interpretation and return the result.

        Args:
            *args: The arguments to the Module to run, in positional order
            initial_env (Optional[Dict[Node, Any]]): An optional starting environment for execution.
                This is a dict mapping `Node` to any value. This can be used, for example, to
                pre-populate results for certain `Nodes` so as to do only partial evaluation within
                the interpreter.
            enable_io_processing (bool): If true, we process the inputs and outputs with graph's process_inputs and
                process_outputs function first before using them.
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 166-186
```python
        Returns:
            Any: The value returned from executing the Module
        """
        self.env = initial_env if initial_env is not None else {}

        # Positional function args are consumed left-to-right by
        # `placeholder` nodes. Use an iterator to keep track of
        # position and extract those values.
        if enable_io_processing:
            args = self.graph.process_inputs(*args)
        self.args_iter: Iterator[Any] = iter(args)
        pbar = tqdm(
            total=len(self.graph.nodes),
            desc=f"{self.name}: {str(list(self.graph.nodes)) if config.verbose_progress else ''}",
            initial=0,
            position=0,
            leave=True,
            disable=config.disable_progress,
            delay=0,
        )
```
- **EN**: This block continues `Interpreter.run` and works to build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `Interpreter.run`，用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理。

### Lines 187-204
```python
        for node in self.graph.nodes:
            pbar.update(1)
            if node in self.env:
                # Short circuit if we have this value. This could
                # be used, for example, for partial evaluation
                # where the caller has pre-populated `env` with
                # values for a subset of the program.
                continue

            try:
                self.env[node] = self.run_node(node)
            except Exception as e:
                if self.extra_traceback:
                    msg = f"While executing {node.format_node()}"
                    msg = f"{e.args[0]}\n\n{msg}" if e.args else str(msg)
                    msg += f"\nOriginal traceback:\n{node.stack_trace}"
                    if (
                        isinstance(self.module, GraphModule)
```
- **EN**: This block continues `Interpreter.run` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `Interpreter.run`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 205-230
```python
                        and self.module.graph is not None
                        and isinstance(self.module.graph, torch.fx.Graph)
                    ):
                        trace_structured(
                            "artifact",
                            metadata_fn=lambda: {
                                "name": "fx_interpreter_error",
                                "encoding": "string",
                            },
                            payload_fn=lambda: (
                                f"{msg}\nGraphModule: "
                                f"{self.module.print_readable(print_output=False, include_stride=True)}"  # type: ignore[operator]
                            ),
                        )

                    msg += "\nUse tlparse to see full graph. "
                    msg += "(https://github.com/pytorch/tlparse?tab=readme-ov-file#tlparse-parse-structured-pt2-logs)"
                    e.args = (msg,) + e.args[1:]
                    if isinstance(e, KeyError):
                        raise RuntimeError(*e.args) from e
                raise

            if self.garbage_collect_values:
                for to_delete in self.user_to_last_uses.get(node, []):
                    del self.env[to_delete]
```
- **EN**: This block continues `Interpreter.run` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `Interpreter.run`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 231-248
```python
            if node.op == "output":
                output_val = self.env[node]
                return (
                    self.graph.process_outputs(output_val)
                    if enable_io_processing
                    else output_val
                )

    @compatibility(is_backward_compatible=True)
    def boxed_run(self, args_list: list[Any]) -> Any:
        """
        Run `module` via interpretation and return the result.  This uses the "boxed"
        calling convention, where you pass a list of arguments, which will be cleared
        by the interpreter.  This ensures that input tensors are promptly deallocated.
        """
        # Collect placeholder nodes first
        placeholder_nodes = [n for n in self.graph.nodes if n.op == "placeholder"]
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 249-272
```python
        # Check argument count
        if len(args_list) != len(placeholder_nodes):
            detail = (
                "extra arguments"
                if len(args_list) > len(placeholder_nodes)
                else "missing arguments"
            )
            raise RuntimeError(
                f"Interpreter.boxed_run expected {len(placeholder_nodes)} arguments for placeholders "
                f"but received {len(args_list)} ({detail})"
            )

        # Assign arguments to placeholders
        env = dict(zip(placeholder_nodes, args_list))
        args_list.clear()
        return self.run(initial_env=env)

    @contextmanager
    def _set_current_node(self, node: Node) -> Iterator[None]:
        with fx_traceback.set_current_meta(
            node, f"Interpreter_{self.__class__.__name__}"
        ):
            yield
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 273-295
```python
    @compatibility(is_backward_compatible=True)
    def run_node(self, n: Node) -> Any:
        """
        Run a specific node ``n`` and return the result.
        Calls into placeholder, get_attr, call_function,
        call_method, call_module, or output depending
        on ``node.op``

        Args:
            n (Node): The Node to execute

        Returns:
            Any: The result of executing ``n``
        """
        log.debug("run_node %s", LazyString(lambda: _format_fx_node(n)))
        with self._set_current_node(n):
            args, kwargs = self.fetch_args_kwargs_from_env(n)
            if not isinstance(args, tuple):
                raise AssertionError(f"Expected args to be tuple, got {type(args)}")
            if not isinstance(kwargs, dict):
                raise AssertionError(f"Expected kwargs to be dict, got {type(kwargs)}")
            return getattr(self, n.op)(n.target, args, kwargs)
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 296-313
```python
    # Main Node running APIs
    @compatibility(is_backward_compatible=True)
    def placeholder(
        self, target: "Target", args: tuple[Argument, ...], kwargs: dict[str, Any]
    ) -> Any:
        """
        Execute a ``placeholder`` node. Note that this is stateful:
        ``Interpreter`` maintains an internal iterator over
        arguments passed to ``run`` and this method returns
        next() on that iterator.

        Args:
            target (Target): The call target for this node. See
                `Node <https://pytorch.org/docs/main/fx.html#torch.fx.Node>`__ for
                details on semantics
            args (Tuple): Tuple of positional args for this invocation
            kwargs (Dict): Dict of keyword arguments for this invocation
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 314-333
```python
        Returns:
            Any: The argument value that was retrieved.
        """
        if not isinstance(target, str):
            raise AssertionError(f"Expected target to be str, got {type(target)}")
        if target.startswith("*"):
            # For a starred parameter e.g. `*args`, retrieve all
            # remaining values from the args list.
            return list(self.args_iter)
        else:
            try:
                return next(self.args_iter)
            except StopIteration as si:
                if len(args) > 0:
                    return args[0]
                else:
                    raise RuntimeError(
                        f"Expected positional argument for parameter {target}, but one was not passed in!"
                    ) from si
```
- **EN**: This block continues `Interpreter` and works to organize reusable module behavior and state. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `Interpreter`，用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 334-355
```python
    @compatibility(is_backward_compatible=True)
    def get_attr(
        self, target: "Target", args: tuple[Argument, ...], kwargs: dict[str, Any]
    ) -> Any:
        """
        Execute a ``get_attr`` node. Will retrieve an attribute
        value from the ``Module`` hierarchy of ``self.module``.

        Args:
            target (Target): The call target for this node. See
                `Node <https://pytorch.org/docs/main/fx.html#torch.fx.Node>`__ for
                details on semantics
            args (Tuple): Tuple of positional args for this invocation
            kwargs (Dict): Dict of keyword arguments for this invocation

        Return:
            Any: The value of the attribute that was retrieved
        """
        if not isinstance(target, str):
            raise AssertionError(f"Expected target to be str, got {type(target)}")
        return self.fetch_attr(target)
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 356-378
```python
    @compatibility(is_backward_compatible=True)
    def call_function(
        self, target: "Target", args: tuple[Argument, ...], kwargs: dict[str, Any]
    ) -> Any:
        """
        Execute a ``call_function`` node and return the result.

        Args:
            target (Target): The call target for this node. See
                `Node <https://pytorch.org/docs/main/fx.html#torch.fx.Node>`__ for
                details on semantics
            args (Tuple): Tuple of positional args for this invocation
            kwargs (Dict): Dict of keyword arguments for this invocation

        Return
            Any: The value returned by the function invocation
        """
        if isinstance(target, str):
            raise AssertionError("target should not be a string for call_function")

        # Execute the function and return the result
        return target(*args, **kwargs)
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 379-403
```python
    @compatibility(is_backward_compatible=True)
    def call_method(
        self, target: "Target", args: tuple[Argument, ...], kwargs: dict[str, Any]
    ) -> Any:
        """
        Execute a ``call_method`` node and return the result.

        Args:
            target (Target): The call target for this node. See
                `Node <https://pytorch.org/docs/main/fx.html#torch.fx.Node>`__ for
                details on semantics
            args (Tuple): Tuple of positional args for this invocation
            kwargs (Dict): Dict of keyword arguments for this invocation

        Return
            Any: The value returned by the method invocation
        """
        # args[0] is the `self` object for this method call
        self_obj, *args_tail = args

        # Execute the method and return the result
        if not isinstance(target, str):
            raise AssertionError(f"Expected target to be str, got {type(target)}")
        return getattr(self_obj, target)(*args_tail, **kwargs)
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 404-429
```python
    @compatibility(is_backward_compatible=True)
    def call_module(
        self, target: "Target", args: tuple[Argument, ...], kwargs: dict[str, Any]
    ) -> Any:
        """
        Execute a ``call_module`` node and return the result.

        Args:
            target (Target): The call target for this node. See
                `Node <https://pytorch.org/docs/main/fx.html#torch.fx.Node>`__ for
                details on semantics
            args (Tuple): Tuple of positional args for this invocation
            kwargs (Dict): Dict of keyword arguments for this invocation

        Return
            Any: The value returned by the module invocation
        """
        # Retrieve executed args and kwargs values from the environment

        # Execute the method and return the result
        if not isinstance(target, str):
            raise AssertionError(f"Expected target to be str, got {type(target)}")
        submod = self.fetch_attr(target)

        return submod(*args, **kwargs)
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 430-455
```python
    @compatibility(is_backward_compatible=True)
    def output(
        self, target: "Target", args: tuple[Argument, ...], kwargs: dict[str, Any]
    ) -> Any:
        """
        Execute an ``output`` node. This really just retrieves
        the value referenced by the ``output`` node and returns it.

        Args:
            target (Target): The call target for this node. See
                `Node <https://pytorch.org/docs/main/fx.html#torch.fx.Node>`__ for
                details on semantics
            args (Tuple): Tuple of positional args for this invocation
            kwargs (Dict): Dict of keyword arguments for this invocation

        Return:
            Any: The return value referenced by the output node
        """
        return args[0]

    # Helper methods
    @compatibility(is_backward_compatible=True)
    def fetch_attr(self, target: str) -> Any:
        """
        Fetch an attribute from the ``Module`` hierarchy of ``self.module``.
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 456-482
```python
        Args:
            target (str): The fully-qualified name of the attribute to fetch

        Return:
            Any: The value of the attribute.
        """
        target_atoms = target.split(".")
        attr_itr = self.module
        for i, atom in enumerate(target_atoms):
            if not hasattr(attr_itr, atom):
                raise RuntimeError(
                    f"Node referenced nonexistent target {'.'.join(target_atoms[: i + 1])}"
                )
            attr_itr = getattr(attr_itr, atom)
        return attr_itr

    @compatibility(is_backward_compatible=True)
    def fetch_args_kwargs_from_env(
        self, n: Node
    ) -> tuple[tuple[Any, ...], dict[str, Any]]:
        """
        Fetch the concrete values of ``args`` and ``kwargs`` of node ``n``
        from the current execution environment.

        Args:
            n (Node): The node for which ``args`` and ``kwargs`` should be fetched.
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 483-505
```python
        Return:
            Tuple[Tuple, Dict]: ``args`` and ``kwargs`` with concrete values for ``n``.
        """
        args = self.map_nodes_to_values(n.args, n)
        if not isinstance(args, tuple):
            raise AssertionError(f"Expected args to be tuple, got {type(args)}")
        kwargs = self.map_nodes_to_values(n.kwargs, n)
        if not isinstance(kwargs, dict):
            raise AssertionError(f"Expected kwargs to be dict, got {type(kwargs)}")
        return args, kwargs

    @compatibility(is_backward_compatible=True)
    def map_nodes_to_values(self, args: Argument, n: Node) -> Argument:
        """
        Recursively descend through ``args`` and look up the concrete value
        for each ``Node`` in the current execution environment.

        Args:
            args (Argument): Data structure within which to look up concrete values

            n (Node): Node to which ``args`` belongs. This is only used for error reporting.
        """
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 506-531
```python
        def load_arg(n_arg: Node) -> Any:
            if n_arg not in self.env:
                raise RuntimeError(
                    f"Node {n} referenced nonexistent value {n_arg}! Run Graph.lint() "
                    f"to diagnose such issues"
                )
            return self.env[n_arg]

        return map_arg(args, load_arg)


@compatibility(is_backward_compatible=True)
class Transformer(Interpreter):
    """
    ``Transformer`` is a special type of interpreter that produces a
    new ``Module``. It exposes a ``transform()`` method that returns
    the transformed ``Module``. ``Transformer`` does not require
    arguments to run, as ``Interpreter`` does. ``Transformer`` works
    entirely symbolically.

    Example:

        Suppose we want to swap all instances of ``torch.neg`` with
        ``torch.sigmoid`` and vice versa (including their ``Tensor``
        method equivalents). We could subclass ``Transformer`` like so::
```
- **EN**: These decorators register or transform the following definition so it can track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够跟踪符号形状约束以及依赖形状的推理。

### Lines 532-558
```python
            class NegSigmSwapXformer(Transformer):
                def call_function(
                    self,
                    target: "Target",
                    args: Tuple[Argument, ...],
                    kwargs: Dict[str, Any],
                ) -> Any:
                    if target is torch.sigmoid:
                        return torch.neg(*args, **kwargs)
                    return super().call_function(target, args, kwargs)

                def call_method(
                    self,
                    target: "Target",
                    args: Tuple[Argument, ...],
                    kwargs: Dict[str, Any],
                ) -> Any:
                    if target == "neg":
                        call_self, *args_tail = args
                        return call_self.sigmoid(*args_tail, **kwargs)
                    return super().call_method(target, args, kwargs)


            def fn(x):
                return torch.sigmoid(x).neg()
```
- **EN**: Declares `Transformer(Interpreter)`; this class packages state and methods that manipulate FX graphs, tracing helpers, or transformation utilities.
- **CN**: 声明 `Transformer(Interpreter)`；该类封装了状态与方法，用于操作 FX 图、跟踪辅助逻辑或变换工具。

### Lines 559-583
```python
            gm = torch.fx.symbolic_trace(fn)

            transformed: torch.nn.Module = NegSigmSwapXformer(gm).transform()
            input = torch.randn(3, 4)
            torch.testing.assert_close(transformed(input), torch.neg(input).sigmoid())

    Args:
        module (GraphModule): The ``Module`` to be transformed.
    """

    @compatibility(is_backward_compatible=True)
    def __init__(self, module: GraphModule) -> None:
        super().__init__(module)
        self.new_graph = Graph()
        self.new_graph.set_codegen(module.graph._codegen)

        class TransformerTracer(Tracer):
            def __init__(self, graph: Graph) -> None:
                super().__init__()
                self.graph = graph
                self.tensor_attrs: dict[torch.Tensor, str] = {}  # type: ignore[assignment]

            def is_leaf_module(self, _: torch.nn.Module, __: str) -> bool:
                return True
```
- **EN**: These decorators register or transform the following definition so it can track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够跟踪符号形状约束以及依赖形状的推理。

### Lines 584-609
```python
        self.tracer = TransformerTracer(self.new_graph)
        self.tracer.root = module

    @compatibility(is_backward_compatible=True)
    def placeholder(
        self, target: "Target", args: tuple[Argument, ...], kwargs: dict[str, Any]
    ) -> Proxy:
        """
        Execute a ``placeholder`` node. In ``Transformer``, this is
        overridden to insert a new ``placeholder`` into the output
        graph.

        Args:
            target (Target): The call target for this node. See
                `Node <https://pytorch.org/docs/main/fx.html#torch.fx.Node>`__ for
                details on semantics
            args (Tuple): Tuple of positional args for this invocation
            kwargs (Dict): Dict of keyword arguments for this invocation
        """
        if not isinstance(target, str):
            raise AssertionError(f"Expected target to be str, got {type(target)}")
        default_value = next(iter(args)) if args else inspect.Signature.empty
        return Proxy(
            self.new_graph.placeholder(target, default_value=default_value), self.tracer
        )
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 610-629
```python
    @compatibility(is_backward_compatible=True)
    def get_attr(
        self, target: "Target", args: tuple[Argument, ...], kwargs: dict[str, Any]
    ) -> Proxy:
        """
        Execute a ``get_attr`` node. In ``Transformer``, this is
        overridden to insert a new ``get_attr`` node into the output
        graph.

        Args:
            target (Target): The call target for this node. See
                `Node <https://pytorch.org/docs/main/fx.html#torch.fx.Node>`__ for
                details on semantics
            args (Tuple): Tuple of positional args for this invocation
            kwargs (Dict): Dict of keyword arguments for this invocation
        """
        if not isinstance(target, str):
            raise AssertionError(f"Expected target to be str, got {type(target)}")
        return self.tracer.create_proxy("get_attr", target, args, kwargs)
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 630-656
```python
    @compatibility(is_backward_compatible=True)
    def call_module(
        self, target: "Target", args: tuple[Argument, ...], kwargs: dict[str, Any]
    ) -> Any:
        # Override so that the leaf module policy from `self.tracer` is respected.
        if not isinstance(target, str):
            raise AssertionError(f"Expected target to be str, got {type(target)}")
        submod = self.fetch_attr(target)
        return self.tracer.call_module(submod, submod.forward, args, kwargs)

    @compatibility(is_backward_compatible=True)
    def call_function(
        self, target: "Target", args: tuple[Argument, ...], kwargs: dict[str, Any]
    ) -> Any:
        # Override so that functions that were wrapped are still wrapped.
        return self.tracer.create_proxy("call_function", target, args, kwargs)

    @compatibility(is_backward_compatible=True)
    def transform(self) -> GraphModule:
        """
        Transform ``self.module`` and return the transformed
        ``GraphModule``.
        """
        with fx_traceback.preserve_node_meta():
            result = super().run(enable_io_processing=False)
        if result is not None:
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 657-670
```python
            def strip_proxy(a: Argument | Proxy) -> Any:
                return a.node if isinstance(a, Proxy) else a

            new_output_node = self.new_graph.output(map_aggregate(result, strip_proxy))
            # also preserve the metadata from the old output node, if it exists
            old_output_node = list(self.graph.nodes)[-1]
            if old_output_node.op != "output":
                raise AssertionError(
                    f"Expected output node, got op={old_output_node.op}"
                )
            for k, v in old_output_node.meta.items():
                new_output_node.meta[k] = v

        return _make_graph_module(self.module, self.new_graph)
```
- **EN**: Defines the `Transformer.transform` method; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`Transformer.transform` 方法；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
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
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.fx.traceback`, `torch._logging`, `torch.hub`, `.`, `._compatibility`, `._lazy_graph_module`, `._symbolic_trace`, `.graph`, `.graph_module`, `.node`, `.proxy`
- **Standard library / 标准库**: `inspect`, `logging`, `collections.abc`, `contextlib`, `typing`
- **Primary symbols / 核心符号**: `__all__`, `_format_fx_node`, `Interpreter`, `Transformer`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
