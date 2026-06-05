# _tensorify_python_scalars.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/passes/_tensorify_python_scalars.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Collects graph-to-graph optimization, rewriting, and analysis passes for FX IR.
- **Purpose (CN)**: 汇集针对 FX 中间表示的图到图优化、改写与分析 passes。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24
```python
from __future__ import annotations

import logging
import os
from typing import Any, TYPE_CHECKING

from sympy import Integer, Number, Symbol
from sympy.logic.boolalg import BooleanAtom

import torch
import torch.fx as fx
from torch._dynamo.exc import TensorifyScalarRestartAnalysis
from torch._dynamo.symbolic_convert import TensorifyState
from torch._dynamo.utils import get_metrics_context
from torch._prims_common import get_computation_dtype
from torch._subclasses.fake_tensor import FakeTensor
from torch._utils_internal import justknobs_check
from torch.fx._utils import lazy_format_graph_code
from torch.fx.experimental.symbolic_shapes import (
    guard_scalar,
    has_free_symbols,
    ShapeEnv,
)
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。

### Lines 25-42
```python
# TODO: refactor
from torch.fx.passes.runtime_assert import _get_sym_val
from torch.fx.proxy import MetaProxy
from torch.utils._sympy.interp import _run_sympy_handler, sympy_interp
from torch.utils._sympy.reference import TensorReferenceAnalysis
from torch.utils._sympy.symbol import symbol_is_type, SymT


if TYPE_CHECKING:
    from torch._subclasses import fake_tensor
    from torch.fx.graph_module import GraphModule


__all__: list[str] = []

log = logging.getLogger(__name__)
graph_code_log = torch._logging.getArtifactLogger(__name__, "graph_code_verbose")
```
- **EN**: This typing-only branch imports symbols for static analysis without affecting runtime behavior.
- **CN**: 这个仅用于类型检查的分支会导入静态分析所需符号，而不会影响运行时行为。

### Lines 43-60
```python
# The general shape of this transformation is to look for Tensor operations
# that take a backed SymFloat as an argument, and then redo them as tensor
# compute (with ints and tensors as inputs). For example, add(Tensor, Scalar)
# can be translated into add(Tensor, Tensor). Because Dynamo has already
# arranged for floats to be Tensor inputs to the graph, for typical float
# compute you can entirely translate the Python float operations into Tensor
# operations with only Tensor inputs.
#
# This pass is also responsible for doing CSE on the fly as we do this, since
# you don't want to keep recomputing the same quantity over and over again if
# it's used multiple times.
#
# This pass runs on the JOINT graph produced by AOT Autograd, prior to partitioning.
# The primary goal of this pass is to eliminate floats by replacing TensorScalar
# operations with TensorTensor operations and then Dead Code Elimination (DCE) of
# the item calls, which effectively removes the floats.
#
# This needs to happen before partitioning because it influences partitioning decisions,
```
- **EN**: This comment block records intent, caveats, or maintainers’ notes for the code that follows.
- **CN**: 这一注释块记录了后续代码的设计意图、注意事项或维护者说明。

### Lines 61-82
```python
# specifically by ensuring that we don't need to save floats across partitions.
# Additionally, there is a separate pass that changes which device computations
# occur on. That pass must be run after this one, but still before partitioning.
#
# HISTORY NOTE: Originally, I wanted to formulate this pass as pushing item()
# calls down, transforming float compute into int compute as we went. If you
# manage to eliminate all float compute, this ends up being equivalent, but
# there is a critical difference when some floats cannot be eliminated: when
# we call item() on them, what should it's SymFloat be? Ideally, it would
# be the same backed SymFloat we had before. But without symbolic expression
# propagation on tensor quantities, repropagating would instead give you an
# unbacked SymFloat. Maybe it is a good idea to implement symbolic propagation
# on 0d scalar tensors, but I decided to go for something simpler to start.
#
# The boring stuff:
#
# * What operators can I Tensor-ify? (Anything with a Scalar argument)
# * How do I Tensor-ify a SymFloat sympy expression (Sympy -> Op Handler -> Tensor)
#
# TODO: make sure this runs before CPU->CUDA pass for cudagraph friendliness
```
- **EN**: This comment block records intent, caveats, or maintainers’ notes for the code that follows.
- **CN**: 这一注释块记录了后续代码的设计意图、注意事项或维护者说明。

### Lines 83-105
```python
SUPPORTED_OPS = {
    torch.ops.aten.mul.Tensor: torch.ops.aten.mul.Tensor,
    torch.ops.aten.add.Tensor: torch.ops.aten.add.Tensor,
    torch.ops.aten.sub.Tensor: torch.ops.aten.sub.Tensor,
    torch.ops.aten.div.Tensor: torch.ops.aten.div.Tensor,
    torch.ops.aten.gt.Scalar: torch.ops.aten.gt.Tensor,
    torch.ops.aten.lt.Scalar: torch.ops.aten.lt.Tensor,
    torch.ops.aten.ge.Scalar: torch.ops.aten.ge.Tensor,
    torch.ops.aten.le.Scalar: torch.ops.aten.le.Tensor,
    torch.ops.aten.eq.Scalar: torch.ops.aten.eq.Tensor,
    torch.ops.aten.ne.Scalar: torch.ops.aten.ne.Tensor,
}


@torch.fx._compatibility.compatibility(is_backward_compatible=False)
def tensorify_python_scalars(
    gm: GraphModule, shape_env: ShapeEnv, fake_mode: fake_tensor.FakeTensorMode
) -> None:
    """
    Converts Python scalar operations into Tensor operations within the graph. This pass looks for
    Tensor operations that involve SymFloat arguments and transforms them into equivalent operations
    that use only Tensor inputs.
```
- **EN**: These decorators register or transform the following definition so it can track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够跟踪符号形状约束以及依赖形状的推理。

### Lines 106-131
```python
    Args:
        gm: The FX graph module representing the computation graph.
        shape_env: The shape environment responsible for symbolic shape tracking and propagation
        during graph transformations.

    Returns:
        None
    """
    import sympy

    knob = True
    if (env := os.getenv("TENSORIFY_PYTHON_SCALARS")) is not None:
        if env in ("0", "FALSE"):
            knob = False
    else:
        knob = justknobs_check("pytorch/compiler:tensorify_python_scalars")
    if not knob:
        return None

    graph = gm.graph
    tracer = fx.proxy.GraphAppendingTracer(graph)
    expr_to_sym_proxy: dict[sympy.Expr, MetaProxy] = {}
    expr_to_tensor_proxy: dict[sympy.Expr, MetaProxy] = {}
    tensorified_symbols: set[sympy.Symbol] = set()
    should_restart = False
```
- **EN**: This block continues `tensorify_python_scalars` and works to track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `tensorify_python_scalars`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 132-154
```python
    first_non_placeholder = None
    placeholders = set()
    for node in graph.nodes:
        if node.op != "placeholder":
            first_non_placeholder = node
            break
        else:
            placeholders.add(node)

    Analysis = TensorReferenceAnalysis

    def _sympy_interp(expr: sympy.Expr) -> MetaProxy:
        # sympy_interp() with hash consing, and special handling for
        # generating constants correctly

        # hash cons
        if isinstance(expr, Symbol) and expr not in expr_to_tensor_proxy:
            # This is guaranteed to be populated by invariant established by
            # insert_deferred_runtime_asserts
            expr_to_tensor_proxy[expr] = torch.ops.aten.scalar_tensor.default(
                expr_to_sym_proxy[expr]
            )
```
- **EN**: Defines the `tensorify_python_scalars` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`tensorify_python_scalars` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 155-172
```python
        # cache constants, why not
        if isinstance(expr, (Integer, Number, BooleanAtom)):
            dtype = None
            c: bool | int | float
            if isinstance(expr, BooleanAtom):
                dtype = torch.bool
                c = bool(expr)
            elif isinstance(expr, sympy.Integer):
                dtype = torch.int64
                c = int(expr)
            elif isinstance(expr, sympy.Number):
                dtype = torch.float64
                c = float(expr)

            node = graph.call_function(
                torch.ops.aten.scalar_tensor.default,
                # pyrefly: ignore [unbound-name]
                (c,),
```
- **EN**: This block continues `tensorify_python_scalars._sympy_interp` and works to build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `tensorify_python_scalars._sympy_interp`，用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理。

### Lines 173-199
```python
                {"dtype": dtype},
            )
            with fake_mode:
                # pyrefly: ignore [unbound-name]
                node.meta["val"] = torch.ops.aten.scalar_tensor.default(c, dtype=dtype)
            expr_to_tensor_proxy[expr] = MetaProxy(
                node,
                tracer=tracer,
                fake_mode=fake_mode,
            )

        if expr in expr_to_tensor_proxy:
            return expr_to_tensor_proxy[expr]

        # don't cache
        if isinstance(expr, Symbol):
            return sympy_interp(Analysis, expr_to_tensor_proxy, expr)  # type: ignore[arg-type]

        # hash cons on arguments, run expr handler
        expr_to_tensor_proxy[expr] = _run_sympy_handler(
            Analysis,
            [_sympy_interp(arg) for arg in expr.args],  # type: ignore[arg-type]
            expr,
        )

        return expr_to_tensor_proxy[expr]
```
- **EN**: This block continues `tensorify_python_scalars` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `tensorify_python_scalars`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 200-223
```python
    failed_tensorify_ops: set[str] = set()
    nodes = list(graph.nodes)
    for i, node in enumerate(nodes[:-1]):
        with graph.inserting_before(
            nodes[i + 1] if node not in placeholders else first_non_placeholder
        ):
            # Look for tensor.item() calls on placeholders
            if (
                node is not None
                and node.op == "call_function"
                and node.target is torch.ops.aten._local_scalar_dense.default
            ):
                source_tensor = node.args[0].meta["val"]
                dtype = source_tensor.dtype

                if not isinstance(node.args[0], fx.Node):
                    raise AssertionError(f"Expected fx.Node, got {node.args[0]}")

                s = node.meta["val"].node.expr

                expr_to_sym_proxy[s] = MetaProxy(
                    node, tracer=tracer, fake_mode=fake_mode
                )
```
- **EN**: This block continues `tensorify_python_scalars` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `tensorify_python_scalars`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 224-244
```python
                # only tensorify if the dtype is floating point
                if not dtype.is_floating_point:
                    continue

                expr_to_tensor_proxy[s] = MetaProxy(
                    node.args[0], tracer=tracer, fake_mode=fake_mode
                )
                if len(source_tensor.shape) != 0:
                    # .item() always produces a scalar value, even when it is
                    # called on a size-1 tensor with rank > 0. Preserve that 0-d
                    # semantics before tensorifying the scalar expression so
                    # later tensor math and autograd tangents do not keep an
                    # accidental length-1 dimension.
                    expr_to_tensor_proxy[s] = torch.ops.aten.reshape.default(
                        expr_to_tensor_proxy[s], []
                    )
                # Upcast the float tensor to torch.float64 to avoid precision problem
                expr_to_tensor_proxy[s] = torch.ops.prims.convert_element_type.default(
                    expr_to_tensor_proxy[s], torch.float64
                )
```
- **EN**: This block continues `tensorify_python_scalars` and works to track symbolic shape constraints and shape-dependent reasoning. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `tensorify_python_scalars`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会根据运行时条件分支处理。

### Lines 245-262
```python
            # pyrefly: ignore [bad-argument-type]
            elif (sym_expr := _get_sym_val(node)) is not None:
                if sym_expr not in expr_to_sym_proxy and not isinstance(
                    sym_expr, (sympy.Number, sympy.logic.boolalg.BooleanAtom)
                ):
                    expr_to_sym_proxy[sym_expr] = MetaProxy(
                        # pyrefly: ignore [bad-argument-type]
                        node,
                        tracer=tracer,
                        fake_mode=fake_mode,
                    )

            # Specialize all dimensions that contain symfloats. Here's
            # an example test that requires this:
            # PYTORCH_OPINFO_SAMPLE_INPUT_INDEX=4 python test/inductor/test_torchinductor_opinfo.py TestInductorOpInfoCUDA.test_comprehensive_nn_functional_interpolate_bicubic_cuda_float32

            val = node.meta.get("val")
            if isinstance(val, FakeTensor):
```
- **EN**: This block continues `tensorify_python_scalars` and works to build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `tensorify_python_scalars`，用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理。

### Lines 263-287
```python
                for dim in val.shape:
                    if isinstance(dim, torch.SymInt):
                        for s in dim.node.expr.free_symbols:
                            name = str(s)
                            if symbol_is_type(
                                s, SymT.FLOAT
                            ) and not TensorifyState.should_specialize(name):
                                # In principle, we could support float input that
                                # is used to do size compute. The problem is that
                                # we don't actually want to tensorify the compute
                                # in this case, which means we need codegen support for
                                # all symfloats.
                                TensorifyState.specialize(name)
                                should_restart = True

            # Look for functions to convert

            if node.op == "call_function" and (
                replacement_op := SUPPORTED_OPS.get(node.target)
            ):
                args: list[Any] = []
                transform = False

                compute_dtype = get_computation_dtype(node.meta["val"].dtype)
```
- **EN**: This block continues `tensorify_python_scalars` and works to track symbolic shape constraints and shape-dependent reasoning. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `tensorify_python_scalars`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 288-314
```python
                for a in node.args:
                    if (
                        isinstance(a, fx.Node)
                        and "val" in a.meta
                        and isinstance(zf := a.meta["val"], torch.SymFloat)
                    ):
                        transform = True
                        try:
                            proxy = _sympy_interp(zf.node.expr)
                        except NotImplementedError:
                            transform = False
                            break

                        # We use _expr instead of expr b/c we want the symbol not the replacement
                        tensorified_symbols.add(a.meta["val"].node._expr)

                        # The upcasting is irrelevant when the compute dtype is bool. This happens
                        # in cases where we are tensorifying a comparison operator such as
                        # torch.ops.aten.gt.Tensor
                        if (
                            compute_dtype != torch.bool
                            and proxy.node.meta["val"].dtype != compute_dtype
                        ):
                            proxy = torch.ops.prims.convert_element_type.default(
                                proxy, compute_dtype
                            )
```
- **EN**: This block continues `tensorify_python_scalars` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `tensorify_python_scalars`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 315-335
```python
                        args.append(proxy)
                    elif isinstance(a, fx.Node):
                        args.append(MetaProxy(a, tracer=tracer, fake_mode=fake_mode))
                    else:
                        args.append(a)

                if transform:
                    replacement_proxy = replacement_op(*args)

                    if compute_dtype != node.meta["val"].dtype:
                        replacement_proxy = (
                            torch.ops.prims.convert_element_type.default(
                                replacement_proxy,
                                node.meta["val"].dtype,
                            )
                        )

                    node.replace_all_uses_with(replacement_proxy.node)

                    graph.erase_node(node)
```
- **EN**: This block continues `tensorify_python_scalars` and works to build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `tensorify_python_scalars`，用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理。

### Lines 336-362
```python
                    metrics_context = get_metrics_context()
                    if metrics_context.in_progress():
                        metrics_context.set(
                            "tensorify_float_success", True, overwrite=True
                        )
            else:
                for a in node.args:
                    if (
                        isinstance(a, fx.Node)
                        and "val" in a.meta
                        and isinstance(zf := a.meta["val"], torch.SymFloat)
                    ):
                        failed_tensorify_ops.update(str(node.target))

                        log.info("Failed to tensorify %s", node.target)

    # Now do one more pass that specializes all symfloats we didn't manage
    # to tensorify away.
    for node in reversed(graph.nodes):
        if node.op == "output" or node.op == "placeholder":
            continue

        with graph.inserting_before(node):
            if len(node.users) == 0 and not node.is_impure():
                graph.erase_node(node)
                continue
```
- **EN**: This block continues `tensorify_python_scalars` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `tensorify_python_scalars`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 363-381
```python
            if isinstance(
                (val := node.meta.get("val")),
                (torch.SymFloat, torch.SymInt, torch.SymBool),
            ):
                if has_free_symbols(val.node.expr) and all(
                    symbol_is_type(s, SymT.FLOAT) for s in val.node.expr.free_symbols
                ):
                    # If all symbols are backed symfloats, we can just specialize the whole node
                    # and get more precise guards. eg.
                    #
                    # zf = a.item()
                    # zf2 = zf // 2
                    # op(.. zf2 ..)
                    #
                    # It's better to guard on zf // 2 == 2.0 than zf == 5.0

                    node.replace_all_uses_with(guard_scalar(val))
                    graph.erase_node(node)
```
- **EN**: This block continues `tensorify_python_scalars` and works to track symbolic shape constraints and shape-dependent reasoning. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `tensorify_python_scalars`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会根据运行时条件分支处理。

### Lines 382-399
```python
    # Sometimes by the time we get to tensorify, there have already been
    # specializations, eg. in python_arg_parser.h. In these cases,
    # placeholder nodes no longer have a reference to their original
    # symfloat and thus we need to deduce specializations have happened
    # via shape_env.replacements. NB: there's an important invariant here
    # that symfloats keep consistent names across restarts.
    for k, v in shape_env.backed_var_to_val.items():
        if symbol_is_type(k, SymT.FLOAT) and isinstance(v, sympy.core.numbers.Float):
            name = str(k)
            if (
                not TensorifyState.should_specialize(name)
                and k not in tensorified_symbols
            ):
                TensorifyState.specialize(name)
                should_restart = True

    if should_restart:
        # Sledgehammer time. Restart dynamo analysis, keeping track of which input sources
```
- **EN**: This block continues `tensorify_python_scalars` and works to track symbolic shape constraints and shape-dependent reasoning. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `tensorify_python_scalars`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 400-412
```python
        # are no longer needed and should be specialized. Restarting analysis is necessary
        # because we need to instruct Dynamo to NOT make these as inputs.
        metrics_context = get_metrics_context()
        if metrics_context.in_progress():
            metrics_context.set(
                "tensorify_float_failure", failed_tensorify_ops, overwrite=True
            )
            metrics_context.set("tensorify_float_success", True, overwrite=True)
        raise TensorifyScalarRestartAnalysis

    graph_code_log.debug(
        "%s", lazy_format_graph_code("tensorify_python_scalars", gm, colored=True)
    )
```
- **EN**: This block continues `tensorify_python_scalars` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `tensorify_python_scalars`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

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

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.fx`, `torch._dynamo.exc`, `torch._dynamo.symbolic_convert`, `torch._dynamo.utils`, `torch._prims_common`, `torch._subclasses.fake_tensor`, `torch._utils_internal`, `torch.fx._utils`, `torch.fx.experimental.symbolic_shapes`, `torch.fx.passes.runtime_assert`, `torch.fx.proxy`, `torch.utils._sympy.interp`, `torch.utils._sympy.reference`, `torch.utils._sympy.symbol`
- **Standard library / 标准库**: `__future__`, `logging`, `os`, `typing`
- **Third-party packages / 第三方包**: `sympy`, `sympy.logic.boolalg`
- **Primary symbols / 核心符号**: `__all__`, `SUPPORTED_OPS`, `tensorify_python_scalars`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
