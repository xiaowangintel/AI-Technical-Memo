# normalize.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/experimental/normalize.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Holds experimental tracing, shape reasoning, and transformation infrastructure built on FX.
- **Purpose (CN)**: 存放基于 FX 的实验性跟踪、形状推理与变换基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
import operator
from collections.abc import Callable
from typing import Any

import torch
import torch.fx
import torch.fx as fx
from torch.fx import Proxy, Transformer
from torch.fx.node import Argument, map_aggregate, Node, Target
from torch.fx.operator_schemas import (
    create_type_hint,
    normalize_function,
    normalize_module,
)

from .schema_type_annotation import AnnotateTypesWithSchema
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。

### Lines 19-36
```python
class NormalizeArgs(Transformer):
    """
    Normalize arguments to Python targets. This means that
    `args/kwargs` will be matched up to the module/functional's
    signature and rewritten to exclusively kwargs in positional order
    if `normalize_to_only_use_kwargs` is true. Also populates default
    values. Does not support positional-only parameters or varargs
    parameters (*args, **kwargs).

    If the nodes have 'type' metadata, it will use it to disambiguate
    overloads. Otherwise, it will throw an error.

    Example usage:
        m = torchvision.models.resnet18()
        traced = torch.fx.symbolic_trace(m)
        traced = NormalizeArgs(traced).transform()
    """
```
- **EN**: Declares `NormalizeArgs(Transformer)`; this class packages state and methods that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 声明 `NormalizeArgs(Transformer)`；该类封装了状态与方法，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 37-51
```python
    def __init__(
        self, module: torch.fx.GraphModule, normalize_to_only_use_kwargs: bool = True
    ) -> None:
        super().__init__(module)
        self.node_map: dict[Proxy, Node] = {}
        self.normalize_to_only_use_kwargs = normalize_to_only_use_kwargs

    def run_node(self, n: Node) -> Any:
        args, kwargs = self.fetch_args_kwargs_from_env(n)

        def get_type(arg: object) -> Any:
            if isinstance(arg, fx.Node):
                return n.meta.get("type")
            return type(arg)
```
- **EN**: Declares `NormalizeArgs(Transformer)`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `NormalizeArgs(Transformer)`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 52-66
```python
        arg_types = map_aggregate(n.args, get_type)
        if not isinstance(arg_types, tuple):
            raise AssertionError(f"Expected tuple, got {type(arg_types)}")
        arg_types = tuple(create_type_hint(i) for i in arg_types)
        kwarg_types = {k: get_type(v) for k, v in kwargs.items()}
        if n.op == "call_function":
            out = self.call_function(n.target, args, kwargs, arg_types, kwarg_types)
        else:
            out = super().run_node(n)
        if n.op != "output":
            self.node_map[out] = n
            out.node.meta = n.meta
            out.node.type = n.type
        return out
```
- **EN**: This block continues `NormalizeArgs` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `NormalizeArgs`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 67-78
```python
    def call_function(
        self,
        target: Target,
        args: tuple[Argument, ...],
        kwargs: dict[str, Any],
        arg_types: tuple[Any, ...] | None = None,
        kwarg_types: dict[str, Any] | None = None,
    ) -> Proxy:
        if not callable(target):
            raise AssertionError(f"Expected callable target, got {type(target)}")
        new_args_and_kwargs = normalize_function(
            target,
```
- **EN**: Defines the `NormalizeArgs.call_function` method; this block introduces logic that proxy tensor-like values through symbolic execution helpers.
- **CN**: 定义`NormalizeArgs.call_function` 方法；该代码块引入了用于借助符号执行辅助逻辑代理类张量值的逻辑。

### Lines 79-92
```python
            args,  # type: ignore[arg-type]
            kwargs,
            arg_types,  # type: ignore[arg-type]
            kwarg_types,
            self.normalize_to_only_use_kwargs,
        )
        if new_args_and_kwargs:
            new_args, new_kwargs = new_args_and_kwargs
            return self.tracer.create_proxy(
                "call_function", target, new_args, new_kwargs
            )
        else:
            return super().call_function(target, args, kwargs)
```
- **EN**: This block continues `NormalizeArgs` and works to trace Python execution into an intermediate graph representation. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `NormalizeArgs`，用于将 Python 执行过程跟踪为中间图表示。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 93-110
```python
    def call_module(
        self, target: Target, args: tuple[Argument, ...], kwargs: dict[str, Any]
    ) -> Proxy:
        if not isinstance(target, str):
            raise AssertionError(f"Expected str target, got {type(target)}")
        new_args_and_kwargs = normalize_module(
            self.module,
            target,
            args,  # type: ignore[arg-type]
            kwargs,
            self.normalize_to_only_use_kwargs,
        )
        if new_args_and_kwargs:
            new_args, new_kwargs = new_args_and_kwargs
            return super().call_module(target, new_args, new_kwargs)
        else:
            return super().call_module(target, args, kwargs)
```
- **EN**: This module-level block helps proxy tensor-like values through symbolic execution helpers. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于借助符号执行辅助逻辑代理类张量值。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 111-126
```python

class NormalizeOperators(AnnotateTypesWithSchema):
    """
    Normalize callsites that are different ways of "spelling" the same
    invocation into a single, canonical call. Currently supports:

    1. Normalize operators (e.g. operator.add) to the `torch` ops they
       ultimately invoke (e.g. torch.add) when it is possible to statically
       reason that

    Example usage:

        m = torchvision.models.resnet18()

        traced = torch.fx.symbolic_trace(m)
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。

### Lines 127-138
```python
        traced = NormalizeOperators(traced).transform()
    """

    binary_magic_method_remap: dict[
        Callable[[Any, Any], Any], Callable[[Any, Any], Any]
    ] = {
        torch.add: operator.add,
        torch.mul: operator.mul,
        torch.sub: operator.sub,
        torch.div: operator.truediv,
        torch.floor_divide: operator.floordiv,
        torch.remainder: operator.mod,
```
- **EN**: This block continues `NormalizeOperators` and works to trace Python execution into an intermediate graph representation.
- **CN**: 该代码块继续实现 `NormalizeOperators`，用于将 Python 执行过程跟踪为中间图表示。

### Lines 139-155
```python
        torch.eq: operator.eq,
        torch.ne: operator.ne,
        torch.lt: operator.lt,
        torch.le: operator.le,
        torch.gt: operator.gt,
        torch.ge: operator.ge,
    }

    def call_function(
        self, target: Target, args: tuple[Argument, ...], kwargs: dict[str, Any]
    ) -> Proxy:
        # Normalize operators according to the magic methods implemented on tensors here:
        # https://github.com/pytorch/pytorch/blob/28c5d90b679c6b38bf4183ec99f16d933c2f1bcd/tools/autograd/templates/python_variable_methods.cpp#L1137

        if not callable(target):
            raise AssertionError(f"Expected callable target, got {type(target)}")
```
- **EN**: Declares `NormalizeOperators(AnnotateTypesWithSchema)`; this class packages state and methods that proxy tensor-like values through symbolic execution helpers.
- **CN**: 声明 `NormalizeOperators(AnnotateTypesWithSchema)`；该类封装了状态与方法，用于借助符号执行辅助逻辑代理类张量值。

### Lines 156-167
```python
        if target in self.binary_magic_method_remap:
            if len(args) != 2:
                return super().call_function(target, args, kwargs)
            lhs, rhs = args

            return super().call_function(
                target=self.binary_magic_method_remap[target],
                args=(lhs, rhs),
                kwargs={},
            )

        return super().call_function(target, args, kwargs)
```
- **EN**: This block continues `NormalizeOperators.call_function` and works to manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `NormalizeOperators.call_function`，用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

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

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.fx`, `torch.fx.node`, `torch.fx.operator_schemas`, `.schema_type_annotation`
- **Standard library / 标准库**: `operator`, `collections.abc`, `typing`
- **Primary symbols / 核心符号**: `NormalizeArgs`, `NormalizeOperators`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
