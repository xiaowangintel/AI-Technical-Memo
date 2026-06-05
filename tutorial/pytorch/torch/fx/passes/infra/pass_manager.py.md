# pass_manager.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/passes/infra/pass_manager.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Collects graph-to-graph optimization, rewriting, and analysis passes for FX IR.
- **Purpose (CN)**: 汇集针对 FX 中间表示的图到图优化、改写与分析 passes。

## Line-by-Line Analysis / 逐行分析

### Lines 1-26
```python
import inspect
import logging
from collections.abc import Callable
from functools import wraps
from queue import Queue
from typing import Any

import torch.nn as nn
from torch.fx._compatibility import compatibility
from torch.fx.graph_module import GraphModule
from torch.fx.passes.infra.pass_base import PassResult


logger = logging.getLogger(__name__)
logger.setLevel(logging.WARNING)

__all__ = ["pass_result_wrapper", "this_before_that_pass_constraint", "PassManager"]


@compatibility(is_backward_compatible=False)
def pass_result_wrapper(fn: Callable[..., Any]) -> Callable[..., PassResult | None]:
    """
    Wrapper for passes which currently do not return a PassResult.
    This wrapper makes them return a PassResult containing the modified object
    and True for the "modified" flag.
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 27-52
```python
    Args:
        fn (Callable[Module, Any])

    Returns:
        wrapped_fn (Callable[Module, PassResult])
    """
    if fn is None:
        # pyrefly: ignore [bad-return]
        return None

    @wraps(fn)
    def wrapped_fn(gm: nn.Module) -> PassResult | None:
        res = fn(gm)
        if res is None:
            return PassResult(gm, True)
        if isinstance(res, PassResult):
            return res
        elif isinstance(res, nn.Module):
            return PassResult(res, True)

    if not inspect.isfunction(fn):
        wrapped_fn.__name__ = type(fn).__name__

    return wrapped_fn
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 53-75
```python
def _validate_pass_schedule_constraint(
    constraint: Callable[[Callable[..., Any], Callable[..., Any]], bool],
    passes: list[Callable[..., Any]],
) -> None:
    for i, a in enumerate(passes):
        for j, b in enumerate(passes[i + 1 :]):
            if constraint(a, b):
                continue
            raise RuntimeError(
                f"pass schedule constraint violated. Expected {a} before {b}"
                f" but found {a} at index {i} and {b} at index{j} in pass"
                f" list."
            )


def _topological_sort_passes(
    passes: list[Callable[..., Any]], constraints: list[Callable[..., Any]]
) -> list[Callable[..., Any]]:
    """
    Args
        passes: Passes that we are ordering
        constraints: Constraints applied on these passes
```
- **EN**: This module-level block helps validate invariants and surface meaningful failures. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于校验不变量并给出有意义的失败信息。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 76-102
```python
    Returns
        A sorted list of callables and a boolean of if a circular dependency
        existed
    """
    if len(constraints) == 0:
        return passes

    # Construct a graph mapping nodes to a list of their users
    graph: dict[Callable[..., Any], list[Callable[..., Any]]] = {p: [] for p in passes}
    indegree_map: dict[Callable[..., Any], int] = dict.fromkeys(passes, 0)
    candidates: Queue[Callable[..., Any]] = Queue()
    for a in passes:
        for b in passes:
            if a == b:
                continue

            for constraint in constraints:
                if not constraint(a, b):
                    graph[b].append(a)
                    indegree_map[a] += 1

        if indegree_map[a] == 0:
            candidates.put(a)

    visited: dict[Callable[..., Any], bool] = dict.fromkeys(passes, False)
    sorted_passes: list[Callable[..., Any]] = []
```
- **EN**: This block continues `_topological_sort_passes` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_topological_sort_passes`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 103-124
```python
    while not candidates.empty():
        p = candidates.get()
        sorted_passes.append(p)
        visited[p] = True

        for n in graph[p]:
            if not visited[n]:
                indegree_map[n] -= 1
                if indegree_map[n] == 0:
                    candidates.put(n)

    # Check if there are unvisited nodes (aka cycles in the graph)
    cycle_passes = list(filter(lambda p: indegree_map[p] != 0, indegree_map.keys()))
    if len(cycle_passes) != 0:
        error = (
            f"Circular dependency detected within the following passes: {cycle_passes}"
        )
        raise RuntimeError(error)

    return sorted_passes
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 125-151
```python
@compatibility(is_backward_compatible=False)
def this_before_that_pass_constraint(
    this: Callable[..., Any], that: Callable[..., Any]
) -> Callable[[Callable[..., Any], Callable[..., Any]], bool]:
    """
    Defines a partial order ('depends on' function) where ``this`` must occur
    before ``that``.

    For example, the following pass list and constraint list would be invalid::

        passes = [pass_b, pass_a]

        constraints = [this_before_that_pass_constraint(pass_a, pass_b)]

    Args:
        this (Callable): pass which should occur first
        that (Callable): pass which should occur later

    Returns:
        depends_on (Callable[[Object, Object], bool])
    """

    def depends_on(a: Callable[..., Any], b: Callable[..., Any]) -> bool:
        return a != that or b != this

    return depends_on
```
- **EN**: These decorators register or transform the following definition so it can connect execution with differentiation-aware logic.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将执行过程与可微分逻辑连接起来。

### Lines 152-174
```python

@compatibility(is_backward_compatible=False)
class PassManager:
    """
    Construct a PassManager.

    Collects passes and constraints. This defines the pass schedule, manages
    pass constraints and pass execution.

    Args:
        passes (Optional[List[Callable]]): List of passes. A pass is a
            callable which modifies an object and returns a PassResult
        constraint (Optional[List[Callable]]): List of constraints. A
            constraint is a callable which takes two passes (A, B) and returns
            True if A depends on B and False otherwise. See implementation of
            `this_before_that_pass_constraint` for example.
        steps (int): Max number of times we run the passes (default = 1).
        run_checks_after_each_pass (bool): Whether to run checks and linting
            after each pass
        suppress_check_failures (bool): Whether to raise errors when running
            checks
    """
```
- **EN**: These decorators register or transform the following definition so it can connect execution with differentiation-aware logic.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将执行过程与可微分逻辑连接起来。

### Lines 175-196
```python
    passes: list[Callable[..., PassResult | None]]
    constraints: list[Callable[[Callable[..., Any], Callable[..., Any]], bool]]
    _validated: bool = False
    steps: int = 1

    def __init__(
        self,
        passes: list[Callable[..., PassResult | None]] | None = None,
        constraints: list[Callable[[Callable[..., Any], Callable[..., Any]], bool]]
        | None = None,
        steps: int | None = None,
        run_checks_after_each_pass: bool = False,
        suppress_check_failures: bool = False,
    ) -> None:
        self.passes = passes or []
        self.constraints = constraints or []
        if steps:
            self.steps = steps

        self.run_checks_after_each_pass = run_checks_after_each_pass
        self.suppress_check_failures = suppress_check_failures
```
- **EN**: Declares `PassManager`; this class packages state and methods that initialize learnable tensors and related state.
- **CN**: 声明 `PassManager`；该类封装了状态与方法，用于初始化可学习张量及相关状态。

### Lines 197-223
```python
    def add_pass(self, _pass: Callable[..., PassResult | None]) -> None:
        """
        Adds a pass into the current list of passes.
        """
        self.passes.append(_pass)
        self._validated = False

    def add_constraint(
        self, constraint: Callable[[Callable[..., Any], Callable[..., Any]], bool]
    ) -> None:
        """
        Adds a constraint into the current list of constraints.
        """
        self.constraints.append(constraint)
        self._validated = False

    def validate_constraints(self) -> None:
        """
        Validates that current pass schedule defined by `self.passes` is valid
        according to all constraints in `self.constraints`
        """
        if self._validated:
            return
        for constraint in self.constraints:
            _validate_pass_schedule_constraint(constraint, self.passes)
        self._validated = True
```
- **EN**: Declares `PassManager`; this class packages state and methods that manipulate FX graphs, tracing helpers, or transformation utilities.
- **CN**: 声明 `PassManager`；该类封装了状态与方法，用于操作 FX 图、跟踪辅助逻辑或变换工具。

### Lines 224-250
```python
    def solve_constraints(self) -> None:
        """
        Finds a valid traversal order based on the given constraints and orders
        the passes based on this order.

        If a circular dependency exists between the constraints and steps = 1,
        then we will raise an error because if steps != 1 this means that we
        will re-run the passes, allowing for circular dependencies.
        """
        self.passes = _topological_sort_passes(self.passes, self.constraints)
        self._validated = True

    def add_checks(self, check: Callable[[nn.Module], None]) -> None:
        """
        Adds a function which takes runs various checks on a given graph module.
        This function is run before and after each pass if the
        `run_checks_after_each_pass` flag is enabled.
        """
        sig = inspect.signature(check)

        if len(list(sig.parameters.values())) != 1:
            raise TypeError(
                "PassManager check function should only take in one variable, a module"
            )

        setattr(self, "check", check)  # noqa: B010
```
- **EN**: Declares `PassManager`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `PassManager`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 251-275
```python
    def check(self, module: nn.Module) -> None:
        pass

    def __call__(self, module: nn.Module) -> PassResult:
        """
        Runs a list of passes in the order based on `self.passes` on the given
        graph module. Each time a pass is run, checks and linting will be run on
        the graph module if `run_checks_after_each_pass` is set.

        If the module is a graph module, we will run the list of passes until
        the graph stops changing, or until `steps` number of times.
        """
        # Order the passes based on the constraints
        if not self._validated:
            self.solve_constraints()

        # Check graph invariants
        self.check(module)

        # Run the set of passes `steps` number of times or until the graph stops
        # changing
        overall_modified = False
        for _ in range(self.steps):
            modified = False
```
- **EN**: Declares `PassManager`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `PassManager`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 276-299
```python
            # Run the set of passes on the graph module
            for i, fn in enumerate(self.passes):
                fn_name = fn.__name__ if inspect.isfunction(fn) else type(fn).__name__
                logger.debug("Running pass '%s'", fn_name)

                try:
                    res = fn(module)

                    if not isinstance(res, PassResult) and not hasattr(
                        res, "graph_module"
                    ):
                        raise TypeError(
                            f"The result of the pass {fn_name} should be type PassResult."
                            + "Please wrap it with pass_result_wrapper()"
                        )
                    # pyrefly: ignore[missing-attribute]
                    module = res.graph_module
                    # pyrefly: ignore[missing-attribute]
                    modified = modified or res.modified

                    if isinstance(module, GraphModule):
                        logger.debug("Graph after pass '%s': %s", fn_name, module.graph)
                        module.recompile()
```
- **EN**: This block continues `PassManager.__call__` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `PassManager.__call__`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 300-317
```python
                    # Check graph invariants
                    if self.run_checks_after_each_pass:
                        self.check(module)

                except Exception as e:
                    prev_pass_names = [
                        p.__name__ if inspect.isfunction(p) else type(p).__name__
                        for p in self.passes[:i]
                    ]
                    msg = f"An error occurred when running the '{fn_name}' pass after the following passes: {prev_pass_names}"
                    raise Exception(msg) from e  # noqa: TRY002

            # If the graph no longer changes, then we can stop running these passes
            overall_modified = overall_modified or modified
            if not modified:
                break

        return PassResult(module, overall_modified)
```
- **EN**: This block continues `PassManager.__call__` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `PassManager.__call__`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Compiler handoff / 编译器交接**
  - EN: The implementation prepares captured work for backend compilation or execution.
  - CN: 该实现会把捕获到的工作整理后交给后端编译或执行。
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
- **Program transforms / 程序变换**
  - EN: The implementation rewrites existing program structure into a more useful form.
  - CN: 该实现会把现有程序结构改写成更有用的形式。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch.nn`, `torch.fx._compatibility`, `torch.fx.graph_module`, `torch.fx.passes.infra.pass_base`
- **Standard library / 标准库**: `inspect`, `logging`, `collections.abc`, `functools`, `queue`, `typing`
- **Primary symbols / 核心符号**: `__all__`, `pass_result_wrapper`, `_validate_pass_schedule_constraint`, `_topological_sort_passes`, `this_before_that_pass_constraint`, `PassManager`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
