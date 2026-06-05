# rewriter.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/experimental/rewriter.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Holds experimental tracing, shape reasoning, and transformation infrastructure built on FX.
- **Purpose (CN)**: 存放基于 FX 的实验性跟踪、形状推理与变换基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```python
import ast
import copy
import functools
import inspect
import textwrap
from collections.abc import Callable
from types import FunctionType
from typing import Any, cast

import torch
from torch._sources import normalize_source_lines
from torch.fx._symbolic_trace import Tracer
from torch.fx.graph import Graph
```
- **EN**: This import block pulls in the dependencies used by later definitions, including `ast`, `copy`, `functools`, `inspect`.
- **CN**: 这一导入块引入后续定义所需的依赖，其中包括 `ast`, `copy`, `functools`, `inspect`。

### Lines 16-31
```python
class AST_Rewriter(ast.NodeTransformer):
    """
    Take a FunctionType object representing a `forward` method, then
    perform an AST rewrite to swap out nodes that are not symbolically
    traceable with a callsite to the FX alternative.

    To support swapping out an AST node, define a new `visit` method on
    that node. For more details, see:
    https://docs.python.org/3/library/ast.html#ast.NodeTransformer
    """

    # This function checks for new keys added in the globals dict. TorchDynamo
    # can insert new keys in the global dict and upset the check. Therefore, put
    # a disable here. This function is an optimization pass and not really
    # suitable for dynamo tracing anyways.
    @torch._dynamo.disable
```
- **EN**: These decorators register or transform the following definition so it can track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够跟踪符号形状约束以及依赖形状的推理。

### Lines 32-43
```python
    def rewrite(self, fn: FunctionType) -> FunctionType:
        # Normalize the source lines
        sourcelines, _ = inspect.getsourcelines(fn)
        sourcelines = normalize_source_lines(sourcelines)
        source = "".join(sourcelines)
        normalized_str = textwrap.dedent(source)

        # Rewrite the original AST
        source_ast = ast.parse(normalized_str)
        dest_ast = ast.fix_missing_locations(self.visit(source_ast))

        # Pull out the compiled function from the newly-created Module
```
- **EN**: Defines the `AST_Rewriter.rewrite` method; this block introduces logic that hand work to a compiler/backend pipeline.
- **CN**: 定义`AST_Rewriter.rewrite` 方法；该代码块引入了用于将工作移交给编译器或后端流水线的逻辑。

### Lines 44-55
```python
        code = compile(dest_ast, "", "exec")
        globals_dict = copy.copy(fn.__globals__)
        keys_before = set(globals_dict.keys())
        exec(code, globals_dict)
        new_keys = list(set(globals_dict.keys()) - keys_before)
        if len(new_keys) != 1:
            raise AssertionError(f"Expected 1 new key, got {len(new_keys)}")
        fn_compiled = globals_dict[new_keys[0]]

        # return the compiled function with the original globals
        def change_func_globals(
            f: FunctionType, globals: dict[str, object]
```
- **EN**: Defines the `AST_Rewriter.rewrite` method; this block introduces logic that hand work to a compiler/backend pipeline.
- **CN**: 定义`AST_Rewriter.rewrite` 方法；该代码块引入了用于将工作移交给编译器或后端流水线的逻辑。

### Lines 56-73
```python
        ) -> FunctionType:
            """Based on https://stackoverflow.com/a/13503277/2988730 (@unutbu)"""
            # __globals__ is a private member of the function class
            # so we have to copy the function, f, all of its member, except f.__globals__
            g = FunctionType(
                f.__code__,
                globals,
                name=f.__name__,
                argdefs=f.__defaults__,
                closure=f.__closure__,
            )
            g = functools.update_wrapper(g, f)
            g.__kwdefaults__ = copy.copy(f.__kwdefaults__)  # type:ignore[attr-defined]
            return g  # pyrefly: ignore [bad-return]

        # Return the correct FunctionType object
        return change_func_globals(fn_compiled, globals=fn.__globals__)
```
- **EN**: This block continues `AST_Rewriter` and works to hand work to a compiler/backend pipeline. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `AST_Rewriter`，用于将工作移交给编译器或后端流水线。 同时它还会计算并返回中间值或结果。

### Lines 74-91
```python
    def visit_Assert(self, node: ast.Assert) -> ast.Expr:
        """
        Swap out the Assert node (Python's `assert`) with a callsite to the
        symbolically-traceable torch._assert function
        """
        # Create the Call node
        n = ast.parse("torch._assert()", mode="eval")
        if not isinstance(n, ast.Expression):
            raise AssertionError(f"Expected ast.Expression, got {type(n)}")
        call_node = n.body
        if not isinstance(call_node, ast.Call):
            raise AssertionError(f"Expected ast.Call, got {type(call_node)}")
        msg = node.msg if node.msg else ast.Constant(value="", kind=None)
        call_node.args = [node.test, msg]

        # Ensure that the new node conforms to the Python AST grammar
        expr_wrapper = ast.Expr(value=call_node)
```
- **EN**: Defines the `AST_Rewriter.visit_Assert` method; this block introduces logic that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 定义`AST_Rewriter.visit_Assert` 方法；该代码块引入了用于跟踪符号形状约束以及依赖形状的推理的逻辑。

### Lines 92-103
```python
        # Return the new Call node to signify that we want to use it as
        # a replacement for the original _assert node
        return ast.copy_location(expr_wrapper, node)

    def visit_AnnAssign(self, node: ast.AnnAssign) -> ast.Assign:
        """
        Swap out Python's AnnAssign with an Assign node where the annotation function is called.
        Example:
             Original:
             y: Tensor_Type(1,2,3, Dyn) = f2(x)
            Output:
             y = annotate(f2(x),Tensor_Type((1,2,3,Dyn)))
```
- **EN**: Declares `AST_Rewriter(ast.NodeTransformer)`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `AST_Rewriter(ast.NodeTransformer)`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 104-116
```python
        """
        return ast.Assign(
            targets=[node.target],
            value=ast.Call(
                func=ast.Name(id="annotate", ctx=ast.Load()),
                # pyrefly: ignore [bad-argument-type]
                args=[node.value, node.annotation],
                keywords=[],
            ),
        )


class RewritingTracer(Tracer):
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 117-133
```python
    def trace(
        self,
        root: torch.nn.Module | Callable[..., Any],
        concrete_args: dict[str, Any] | None = None,
    ) -> Graph:
        return super().trace(_rewrite(root), concrete_args)


def _rewrite(
    fn: torch.nn.Module | Callable[..., Any],
) -> torch.nn.Module | Callable[..., Any]:
    if isinstance(fn, torch.nn.Module):
        # Rewrite this module's `forward` as well as the `forward`s of
        # all of this module's recursive descendents. Return the new,
        # rewritten module hierarchy.
        def rewrite_module(m: torch.nn.Module) -> torch.nn.Module:
            class RewrittenModule(torch.nn.Module):
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 134-146
```python
                def __init__(self, orig: torch.nn.Module) -> None:
                    super().__init__()
                    for k, v in orig.__dict__.items():
                        if isinstance(v, torch.nn.Module):
                            self.__dict__[k] = copy.copy(rewrite_module(v))
                        else:
                            self.__dict__[k] = copy.copy(v)

            RewrittenModule.forward = AST_Rewriter().rewrite(
                cast(FunctionType, m.forward)
            )
            return RewrittenModule(m)
```
- **EN**: Defines the `_rewrite` function; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`_rewrite` 函数；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

### Lines 147-150
```python
        return rewrite_module(fn)
    else:
        # Rewrite this single free function
        return AST_Rewriter().rewrite(cast(FunctionType, fn))
```
- **EN**: This block continues `_rewrite` and works to organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `_rewrite`，用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

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
- **Compiler handoff / 编译器交接**
  - EN: The implementation prepares captured work for backend compilation or execution.
  - CN: 该实现会把捕获到的工作整理后交给后端编译或执行。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch._sources`, `torch.fx._symbolic_trace`, `torch.fx.graph`
- **Standard library / 标准库**: `ast`, `copy`, `functools`, `inspect`, `textwrap`, `collections.abc`, `types`, `typing`
- **Primary symbols / 核心符号**: `AST_Rewriter`, `RewritingTracer`, `_rewrite`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
