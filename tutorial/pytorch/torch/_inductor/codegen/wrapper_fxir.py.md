# wrapper_fxir.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/wrapper_fxir.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `SymbolBuffer`, `TritonKernel`, `WrapperFxCodegen`, and `FxConverter`. It exposes functions such as `replace_floor_div`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `SymbolBuffer`、`TritonKernel`、`WrapperFxCodegen`、`FxConverter` 等类。同时提供 `replace_floor_div` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
import dataclasses
import functools
import logging
import operator
import textwrap
from collections import Counter
from collections.abc import Callable, Iterable, Sequence
from typing import Any

import sympy

import torch
from torch._export.passes._node_metadata_hook import (
    _node_metadata_hook,
    _set_node_metadata_hook,
)
from torch._higher_order_ops.triton_kernel_wrap import (
    TraceableTritonKernelWrapper,
    tracing_triton_hopifier_singleton,
    triton_kernel_wrapper_mutation,
)
from torch._inductor.codecache import LambdaFuture, PyCodeCache
from torch._inductor.runtime.triton_heuristics import CachingAutotuner
from torch._inductor.select_algorithm import extern_kernels  # noqa: F401
from torch._inductor.utils import convert_to_symint
from torch._inductor.virtualized import V
from torch._library.triton import wrap_triton
from torch.fx import GraphModule
````
- **EN**: Imports dependencies such as `dataclasses`, `functools`, `logging`, `operator`, `textwrap`, `collections`, and `...+13` for the logic in this range. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里导入了 `dataclasses`、`functools`、`logging`、`operator`、`textwrap`、`collections`、`另有13项` 等依赖，为后续逻辑提供基础能力。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 29-56 / 第 29-56 行
````python
from torch.fx.experimental.symbolic_shapes import (
    CallMethodKey,
    ConvertIntKey,
    DivideByKey,
)
from torch.utils import _pytree as pytree
from torch.utils._ordered_set import OrderedSet
from torch.utils._sympy.functions import FloorDiv
from torch.utils._sympy.interp import _run_sympy_handler, sympy_interp
from torch.utils._sympy.reference import OptimizedPythonReferenceAnalysis
from torch.utils._sympy.solve import try_solve

from .. import config, ir
from ..runtime.triton_compat import Config
from ..utils import cache_property_on_self, LineContext, ValueWithLineMap
from .common import (
    CodegenSymbol,
    FileBackedGraphModule,
    WorkspaceArg,
    WorkspaceZeroMode,
)
from .wrapper import (
    AllocateLine,
    BufferLike,
    CommentLine,
    ConditionalLine,
    DynamicScalarLine,
    EnterDeviceContextManagerLine,
````
- **EN**: Imports dependencies such as `torch.fx.experimental.symbolic_shapes`, `torch.utils`, `torch.utils._ordered_set`, `torch.utils._sympy.functions`, `torch.utils._sympy.interp`, `torch.utils._sympy.reference`, and `...+6` for the logic in this range. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里导入了 `torch.fx.experimental.symbolic_shapes`、`torch.utils`、`torch.utils._ordered_set`、`torch.utils._sympy.functions`、`torch.utils._sympy.interp`、`torch.utils._sympy.reference`、`另有6项` 等依赖，为后续逻辑提供基础能力。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 57-84 / 第 57-84 行
````python
    EnterSubgraphLine,
    ExitDeviceContextManagerLine,
    ExitSubgraphLine,
    ExternKernelAllocLine,
    ExternKernelOutLine,
    FreeIfNotReusedLine,
    FreeLine,
    IndexPutFallbackLine,
    KernelCallLine,
    KernelDefinitionLine,
    Line,
    MultiOutputLine,
    NullLine,
    PythonWrapperCodegen,
    ReinterpretLine,
    ReuseLine,
    ScatterFallbackLine,
    SubgraphPythonWrapperCodegen,
    SymbolicCallArg,
    SymbolicCallArgLine,
    UnbackedSymbolDefsLine,
    WrapperLine,
)


aten = torch.ops.aten
log = logging.getLogger(__name__)

````
- **EN**: Initializes or updates values such as `aten`, and `log`.
- **CN**: 初始化或更新了 `aten`、`log` 等值。

### Lines 85-112 / 第 85-112 行
````python

@dataclasses.dataclass
class SymbolBuffer(CodegenSymbol):
    """
    Represents a sympy.Symbol graph input.
    """

    symbol: sympy.Symbol

    def get_name(self) -> str:
        return str(self.symbol)

    def get_example(self) -> torch.Tensor | torch.SymInt:
        sym_int = convert_to_symint(self.symbol)
        assert isinstance(sym_int, torch.SymInt)
        return sym_int


CodegenBuffer = BufferLike | SymbolBuffer


@dataclasses.dataclass
class TritonKernel:
    """
    Stores metadata about Triton kernels for use in FX.
    """

    tuner: CachingAutotuner
````
- **EN**: Introduces class `SymbolBuffer`, function `get_name`, function `get_example`, class `TritonKernel`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里定义了类`SymbolBuffer`、函数`get_name`、函数`get_example`、类`TritonKernel`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。借助符号表达式来推理索引、形状或代数性质。

### Lines 113-140 / 第 113-140 行
````python
    wrapped: TraceableTritonKernelWrapper


def replace_floor_div(expr: sympy.Expr) -> sympy.Expr:
    """
    Replace sympy.floor with FloorDiv.
    """

    def replace(expr: sympy.Expr) -> sympy.Expr:
        expr = sympy.together(expr)

        # Division is represented as a Mul with a Rational factor or a Pow with negative
        # exponent. We convert floor(Mul(...)) to FloorDiv(numerator, denominator) by
        # partitioning factors into the numerator and denominator.
        (numerator, denominator) = (sympy.S.One,) * 2
        for arg in sympy.Mul.make_args(expr):
            if isinstance(arg, sympy.Rational):
                numerator *= arg.numerator
                denominator *= arg.denominator
            elif isinstance(arg, sympy.Pow) and arg.exp.is_negative:
                denominator *= arg.base**-arg.exp
            else:
                numerator *= arg

        return FloorDiv(numerator, denominator)

    return expr.replace(sympy.floor, replace)

````
- **EN**: Introduces function `replace_floor_div`, function `replace`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`replace_floor_div`、函数`replace`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-168 / 第 141-168 行
````python

class WrapperFxCodegen(PythonWrapperCodegen):
    """
    Backend to generate wrapper code as an FX IR graph.
    """

    supports_caching = False

    def __init__(self, *args: Any, **kwargs: Any):
        super().__init__(*args, **kwargs)
        self.subgms: dict[str, torch.fx.GraphModule] = {}

    def codegen_inputs(self) -> None:
        """
        This would generate code for symbolic input shapes, strides, etc.
        Since the FX converter handles this, do nothing here.
        """

    def codegen_conditional(self, conditional: ir.Conditional) -> None:
        """
        Conditional codegen normally emits a number of different wrapper lines.
        Instead, FX conversion uses a dedicated line for the whole conditional.
        """
        self.writeline(ConditionalLine(self, conditional))
        for subgraph in (conditional.true_subgraph, conditional.false_subgraph):
            self.codegen_subgraph_common(subgraph)

    def define_subgraph_launcher_fn(
````
- **EN**: Introduces class `WrapperFxCodegen`, function `__init__`, function `codegen_inputs`, function `codegen_conditional`, function `define_subgraph_launcher_fn`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `supports_caching`.
- **CN**: 这里定义了类`WrapperFxCodegen`、函数`__init__`、函数`codegen_inputs`、函数`codegen_conditional`、函数`define_subgraph_launcher_fn`。包含分支、循环或上下文管理等控制流。初始化或更新了 `supports_caching` 等值。

### Lines 169-196 / 第 169-196 行
````python
        self, name: str, subgraph_code: ValueWithLineMap | FileBackedGraphModule
    ) -> None:
        """
        Record subgms as they're generated.
        """
        assert isinstance(subgraph_code, FileBackedGraphModule)
        self.subgms[name] = subgraph_code.gm

    @property
    @cache_property_on_self
    def is_subgraph(self) -> bool:
        return isinstance(self, SubgraphPythonWrapperCodegen)

    def get_fx_graph_inputs(
        self,
    ) -> dict[str, ir.TensorBox | ir.TorchBindObject | sympy.Expr | None]:
        """
        Get the input nodes corresponding to FX graph placeholders.
        """

        if V.aot_compilation and not self.is_subgraph:
            # AOT graphs must match the signature of the input module.
            return {
                node.name: V.graph.graph_inputs.get(node.name)
                for node in V.graph.module.graph.find_nodes(op="placeholder")  # type: ignore[operator, union-attr]
            }

        return self.get_graph_inputs()
````
- **EN**: Introduces function `is_subgraph`, function `get_fx_graph_inputs`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里定义了函数`is_subgraph`、函数`get_fx_graph_inputs`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。借助符号表达式来推理索引、形状或代数性质。

### Lines 197-224 / 第 197-224 行
````python

    def _generate(self, is_inference: bool) -> tuple[FileBackedGraphModule, None]:
        self.run_wrapper_ir_passes(is_inference)

        prologue = "\n".join(
            [
                self.imports.getvalue(),
                self.header.getvalue(),
            ]
        )
        gm = FxConverter(
            lines=self.lines,
            prologue=prologue,
            graph_inputs=self.get_fx_graph_inputs(),
            graph_outputs=self.get_graph_outputs(),
            subgms=self.subgms,
            is_subgraph=self.is_subgraph,
        ).generate()

        compiled_fn = self.compile_graph(gm)

        return FileBackedGraphModule(gm, compiled_fn), None

    def compile_graph(self, gm: GraphModule) -> Callable[..., Any]:
        """
        Converts the graph module into a runnable function. The default implementation
        is simply an interpreter calling kernels in eager mode. Derived backends can
        override this to do further compilation.
````
- **EN**: Introduces function `_generate`, function `compile_graph`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `prologue`, `gm`, `lines`, `graph_inputs`, `graph_outputs`, `subgms`, and `...+2`.
- **CN**: 这里定义了函数`_generate`、函数`compile_graph`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `prologue`、`gm`、`lines`、`graph_inputs`、`graph_outputs`、`subgms`、`另有2项` 等值。

### Lines 225-252 / 第 225-252 行
````python
        """
        return gm.forward

    def write_header(self) -> None:
        """
        Python subgraphs normally lack headers.
        Override this behavior to generate prologues for FX subgraphs.
        """
        PythonWrapperCodegen.write_header(self)

    def register_alignment_check_inputs(self) -> None:
        """FXIR does not emit deferred alignment copies.
        Alignment is handled by the runtime wrapper."""

    def codegen_deferred_alignment_copies(self, input_names: Iterable[str]) -> None:
        """FXIR does not emit deferred alignment copies."""

    @classmethod
    def create(
        cls: type["WrapperFxCodegen"],
        is_subgraph: bool,
        subgraph_name: str | None,
        parent_wrapper: PythonWrapperCodegen | None,
        partition_signatures: ir.GraphPartitionSignature | None = None,
    ) -> "WrapperFxCodegen":
        if is_subgraph:
            assert subgraph_name is not None
            assert parent_wrapper is not None
````
- **EN**: Introduces function `write_header`, function `register_alignment_check_inputs`, function `codegen_deferred_alignment_copies`, function `create`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`write_header`、函数`register_alignment_check_inputs`、函数`codegen_deferred_alignment_copies`、函数`create`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 253-280 / 第 253-280 行
````python

            # Subgraphs override some methods of PythonWrapperCodegen.
            # Apply these overrides to the user-provided class, with priority given to
            # user-provided methods.
            class SubgraphFxWrapperCodegen(cls, SubgraphPythonWrapperCodegen):  # type: ignore[misc,valid-type]
                def compile_graph(self, gm: GraphModule) -> Callable[..., Any]:
                    """
                    Skip graph compilation for subgraphs.
                    """

                    def crash_if_run(*args: Any) -> None:
                        raise NotImplementedError("Cannot run a subgraph in isolation!")

                    return crash_if_run

            return SubgraphFxWrapperCodegen(
                subgraph_name, parent_wrapper, partition_signatures
            )

        return cls()


@dataclasses.dataclass
class FxConverter:
    """
    Generates FX IR from Wrapper IR. As each instance is only meant to be used once, the
    input and output code are stored as attributes.
    """
````
- **EN**: Introduces class `SubgraphFxWrapperCodegen`, function `compile_graph`, function `crash_if_run`, class `FxConverter`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`SubgraphFxWrapperCodegen`、函数`compile_graph`、函数`crash_if_run`、类`FxConverter`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 281-308 / 第 281-308 行
````python

    lines: list[Line]
    prologue: str
    graph_inputs: dict[str, ir.TensorBox | ir.TorchBindObject | sympy.Expr | None]
    graph_outputs: list[ir.IRNode]
    subgms: dict[str, torch.fx.GraphModule]
    is_subgraph: bool

    def __post_init__(self) -> None:
        graph = torch.fx.Graph()
        self.gm = GraphModule({}, graph)  # Wrapper FX IR.
        self.buffer_to_node: dict[
            str | None, torch.fx.Node
        ] = {}  # Symbol table for codegen.
        self.kernels: dict[str, TritonKernel] = {}  # Table to store Triton kernels.
        self._unique_symbol_ids: Counter[str] = Counter()
        self.tracer = torch.fx.proxy.GraphAppendingTracer(graph)
        self.expr_to_proxy: dict[sympy.Expr, torch.fx.Proxy] = {}

    def _import_kernel(self, code: str, kernel_name: str) -> CachingAutotuner:
        """
        Imports a kernel from source, possibly autotuning block parameters.
        """
        module_code = "\n".join([self.prologue, code])
        mod = PyCodeCache.load(module_code)
        kernel = getattr(mod, kernel_name)

        if isinstance(kernel, LambdaFuture):
````
- **EN**: Introduces function `__post_init__`, function `_import_kernel`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里定义了函数`__post_init__`、函数`_import_kernel`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 309-336 / 第 309-336 行
````python
            kernel = kernel.result()

        if not isinstance(kernel, CachingAutotuner):
            raise NotImplementedError(
                textwrap.dedent(f"""
                Unsupported type for kernel {kernel_name}: {type(kernel)}.
                FX conversion only supports Triton kernels.
            """)
            )

        return kernel

    def _create_as_strided(
        self,
        input_node: torch.fx.Node,
        size: tuple[Any, ...],
        stride: tuple[Any, ...],
        offset: int | sympy.Expr,
    ) -> torch.fx.Node:
        return self.gm.graph.call_function(
            torch.as_strided,
            args=(
                input_node,
                self._generate_sym_nodes(size),
                self._generate_sym_nodes(stride),
                self._generate_sym_node(offset),
            ),
        )
````
- **EN**: Introduces function `_create_as_strided`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_create_as_strided`。借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 337-364 / 第 337-364 行
````python

    def _record_allocation(self, buffer: CodegenBuffer, node: torch.fx.Node) -> None:
        """
        Updates the symbol table to record that an Inductor buffer maps to the result of
        an FX node.
        """
        assert node not in self.buffer_to_node
        self.buffer_to_node[buffer.get_name()] = node

    def _free(self, buffer: CodegenBuffer | ir.TorchBindObject) -> None:
        """
        Removes the buffer from the symbol table.
        """
        name = buffer.get_name()
        del self.buffer_to_node[name]

    def _lookup_args(self, args: tuple[Any, ...]) -> tuple[Any, ...]:
        """
        Maps call args back to FX nodes.
        """
        return tuple(
            self.buffer_to_node[arg]
            if isinstance(arg, str)
            else arg.inner_expr
            if isinstance(arg, SymbolicCallArg)
            else arg
            for arg in args
        )
````
- **EN**: Introduces function `_record_allocation`, function `_free`, function `_lookup_args`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `name`.
- **CN**: 这里定义了函数`_record_allocation`、函数`_free`、函数`_lookup_args`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `name` 等值。

### Lines 365-392 / 第 365-392 行
````python

    def _get_buffer(self, node: ir.IRNode) -> CodegenBuffer:
        """
        Extract buffer data from an IR node.
        """
        if isinstance(node, (ir.Buffer, WorkspaceArg)):
            return node
        elif isinstance(node, (ir.BaseView, ir.MutableBox)):
            return self._get_buffer(node.data)
        elif isinstance(node, sympy.Symbol):
            return SymbolBuffer(node)
        else:
            raise NotImplementedError(f"Unable to extract buffer from node: {node}")

    def _generate_size_proxy(
        self, node: torch.fx.Node, expr: sympy.Expr
    ) -> torch.fx.Proxy:
        proxy = torch.fx.Proxy(node, tracer=self.tracer)
        self.expr_to_proxy[expr] = proxy
        return proxy

    def _generate_graph_inputs(self) -> None:
        """
        Converts graph inputs to FX placeholders.
        """

        for name, ir_node in self.graph_inputs.items():
            if ir_node is None:
````
- **EN**: Introduces function `_get_buffer`, function `_generate_size_proxy`, function `_generate_graph_inputs`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_get_buffer`、函数`_generate_size_proxy`、函数`_generate_graph_inputs`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 393-420 / 第 393-420 行
````python
                # Create dummy input nodes to match the input signature
                self.gm.graph.placeholder(name)
                continue

            # Introduce a new symbol for constant inputs.
            is_constant = isinstance(ir_node, (int, float, sympy.Integer, sympy.Float))
            buffer = (
                SymbolBuffer(sympy.Symbol(name, is_integer=True))
                if is_constant
                else self._get_buffer(ir_node)
            )
            placeholder_node = self.gm.graph.placeholder(buffer.get_name())
            placeholder_node.meta["val"] = (
                ir_node if is_constant else buffer.get_example()
            )
            self._record_allocation(buffer, placeholder_node)

            # Record symbol definitions for dynamic shapes.
            if isinstance(ir_node, sympy.Symbol):
                self._generate_size_proxy(placeholder_node, ir_node)

    def _generate_graph_input_shapes(self) -> None:
        """
        Generate nodes creating symints that are part of graph input
        shape/strides.
        """

        def _codegen_symbol(
````
- **EN**: Introduces function `_generate_graph_input_shapes`, function `_codegen_symbol`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `is_constant`, `buffer`, and `placeholder_node`.
- **CN**: 这里定义了函数`_generate_graph_input_shapes`、函数`_codegen_symbol`。借助符号表达式来推理索引、形状或代数性质。包含分支、循环或上下文管理等控制流。初始化或更新了 `is_constant`、`buffer`、`placeholder_node` 等值。

### Lines 421-448 / 第 421-448 行
````python
            sym_or_exp: sympy.Symbol | sympy.Expr,
            base_node: torch.fx.Node,
            target: torch._ops.OpOverload,
            dim: int,
        ) -> None:
            def codegen_proxy() -> torch.fx.Proxy:
                size_node = self.gm.graph.call_function(target, (base_node, dim))
                size_proxy = self._generate_size_proxy(size_node, sym_or_exp)
                return size_proxy

            if isinstance(sym_or_exp, sympy.Symbol):
                if sym_or_exp in self.expr_to_proxy:
                    return
                codegen_proxy()

            elif isinstance(sym_or_exp, sympy.Integer):
                return

            elif isinstance(sym_or_exp, sympy.Expr):
                # Check if we need to solve for an undefined symbol.
                undefined_symbols = [
                    sym
                    for sym in sym_or_exp.free_symbols
                    if sym not in self.expr_to_proxy
                ]
                if len(undefined_symbols) == 0:
                    self._sympy_interp(sym_or_exp)
                    return
````
- **EN**: Introduces function `codegen_proxy`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`codegen_proxy`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 449-476 / 第 449-476 行
````python
                elif len(undefined_symbols) > 1:
                    raise ValueError(f"Underdetermined input expression: {sym_or_exp}")

                # Define a new symbol for the input size.
                size_proxy = codegen_proxy()
                size_symbol = sympy.Symbol(
                    size_proxy.node.name, integer=True, nonnegative=True
                )
                self.expr_to_proxy[size_symbol] = size_proxy

                # Solve for the undefined symbol.
                undefined_symbol = undefined_symbols[0]
                solution = try_solve(
                    sympy.Eq(sym_or_exp, size_symbol), undefined_symbol
                )
                if solution is None:
                    raise ValueError(f"Cannot solve input expression: {sym_or_exp}")

                # Since the symbol is a size, it must be an integer.
                # Therefore, we can convert division to FloorDiv.
                undefined_symbol_expr = solution[1]
                if undefined_symbol.is_integer:
                    undefined_symbol_expr = replace_floor_div(
                        sympy.floor(undefined_symbol_expr)
                    )

                # Generate FX for the symbol.
                self._sympy_interp(undefined_symbol_expr)
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `size_proxy`, `size_symbol`, `undefined_symbol`, `solution`, and `undefined_symbol_expr`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `size_proxy`、`size_symbol`、`undefined_symbol`、`solution`、`undefined_symbol_expr` 等值。

### Lines 477-504 / 第 477-504 行
````python
                self.expr_to_proxy[undefined_symbol] = self.expr_to_proxy[
                    undefined_symbol_expr
                ]

        for ir_node in self.graph_inputs.values():
            if isinstance(ir_node, ir.TensorBox):
                buffer = self._get_buffer(ir_node)
                placeholder_node = self.buffer_to_node[buffer.get_name()]

                for dim, size in enumerate(ir_node.get_size()):
                    _codegen_symbol(
                        size, placeholder_node, torch.ops.aten.sym_size.int, dim
                    )
                for dim, stride in enumerate(ir_node.get_stride()):
                    _codegen_symbol(
                        stride, placeholder_node, torch.ops.aten.sym_stride.int, dim
                    )

    def _generate_graph_constants(self) -> None:
        for name, value in V.graph.constants.items():
            node = self.gm.graph.get_attr(name)
            node.meta["val"] = value
            setattr(self.gm, name, value)
            self.buffer_to_node[name] = node

    def _generate_buffer(self, node: ir.IRNode) -> torch.fx.Node | None:
        """
        Generates FX IR for transformations on a buffer, such as ReinterpretView.
````
- **EN**: Introduces function `_generate_graph_constants`, function `_generate_buffer`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `buffer`, `placeholder_node`, and `node`.
- **CN**: 这里定义了函数`_generate_graph_constants`、函数`_generate_buffer`。包含分支、循环或上下文管理等控制流。初始化或更新了 `buffer`、`placeholder_node`、`node` 等值。

### Lines 505-532 / 第 505-532 行
````python
        Does nothing if no such transformations are present.
        """

        if isinstance(node, ir.ShapeAsConstantBuffer):
            # Generate FX nodes to compute the shape expression.
            return self._sympy_interp(node.expr).node

        def generate_to_buffer(node: ir.IRNode) -> BufferLike | None:
            if isinstance(node, (ir.Buffer, WorkspaceArg)):
                return node
            elif isinstance(node, ir.NoneAsConstantBuffer):
                return None
            elif isinstance(node, ir.MutableBox):
                return generate_to_buffer(node.data)
            elif isinstance(node, ir.ReinterpretView):
                # We need to introduce a new symbol if the output is a ReinterpretView.
                # Use a WorkspaceArg for this.
                buffer = self._get_buffer(node.data)
                assert isinstance(buffer, (ir.Buffer, WorkspaceArg))
                unique_name = self.gm.graph._graph_namespace.create_name(
                    f"{buffer.get_name()}_view", None
                )
                device = buffer.get_device()
                assert device
                reused_as = WorkspaceArg(
                    count=buffer.get_size(),
                    zero_mode=WorkspaceZeroMode.UNINITIALIZED,
                    device=device,
````
- **EN**: Introduces function `generate_to_buffer`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`generate_to_buffer`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 533-560 / 第 533-560 行
````python
                    outer_name=unique_name,
                    dtype=buffer.get_dtype(),
                )

                # Generate FX IR for the view.
                self._generate_reinterpret_helper(buffer, reused_as, node.layout)

                return reused_as
            else:
                raise NotImplementedError(f"Unrecognized buffer/view node: {node}")

        buffer = generate_to_buffer(node)
        return self.buffer_to_node[buffer.get_name()] if buffer is not None else None

    def _generate_outputs(
        self,
    ) -> torch.fx.Node | None | list[torch.fx.Node | None]:
        """
        Generate FX IR for graph outputs.
        """
        output_nodes = [
            self._generate_buffer(node) for idx, node in enumerate(self.graph_outputs)
        ]

        # Parent graphs with single return elements don't use a tuple.
        output_value = (
            output_nodes[0]
            if len(output_nodes) == 1 and not self.is_subgraph
````
- **EN**: Introduces function `_generate_outputs`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_generate_outputs`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 561-588 / 第 561-588 行
````python
            else output_nodes
        )

        return output_value

    def _generate_subgm_getattrs(self) -> None:
        """
        Generate getattr nodes for subgms.
        """

        def generate_getattr(name: str, subgm: torch.fx.GraphModule) -> torch.fx.Node:
            self.gm.add_submodule(name, subgm)
            node = self.gm.graph.get_attr(name)
            node.meta["val"] = subgm
            return node

        self.subgm_getattrs = {
            name: generate_getattr(name, subgm) for name, subgm in self.subgms.items()
        }

    def _get_subgm_attr(self, subgraph: ir.Subgraph) -> torch.fx.Node:
        """
        Look up the getattr node for a subgraph.
        """
        graph = subgraph.graph
        assert graph is not None
        return self.subgm_getattrs[graph.name]

````
- **EN**: Introduces function `_generate_subgm_getattrs`, function `generate_getattr`, function `_get_subgm_attr`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `node`, `name`, and `graph`.
- **CN**: 这里定义了函数`_generate_subgm_getattrs`、函数`generate_getattr`、函数`_get_subgm_attr`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `node`、`name`、`graph` 等值。

### Lines 589-616 / 第 589-616 行
````python
    def generate(self) -> torch.fx.GraphModule:
        """
        Main entrypoint for FX codegen.
        """
        self._generate_graph_inputs()
        self._generate_graph_constants()
        self._generate_subgm_getattrs()

        with _set_node_metadata_hook(
            self.gm,
            functools.partial(_node_metadata_hook, fake_mode=V.fake_mode),
        ):
            self._generate_graph_input_shapes()

            # Generate FX IR from Wrapper IR lines.
            for line in self.lines:
                if isinstance(line, WrapperLine):
                    line.codegen_fx(self)(line)
                elif isinstance(line, LineContext):
                    # Ignore line context in FX IR.
                    pass
                else:
                    raise NotImplementedError(
                        textwrap.dedent(
                            f"""
                        Found line of unrecognized type '{type(line)}':
                            '{line}'

````
- **EN**: Introduces function `generate`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`.
- **CN**: 这里定义了函数`generate`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。

### Lines 617-644 / 第 617-644 行
````python
                        FX conversion only supports Wrapper IR lines.
                        """
                        )
                    )

            output = self._generate_outputs()

        self.gm.graph.output(output)
        self.gm.recompile()
        return self.gm

    def _sympy_interp(self, expr: sympy.Expr) -> torch.fx.Proxy:
        # hash cons
        if expr in self.expr_to_proxy:
            return self.expr_to_proxy[expr]
        # base cases, don't cache
        if isinstance(
            expr,
            (
                sympy.Integer,
                sympy.Number,
                sympy.Symbol,
                sympy.logic.boolalg.BooleanAtom,
            ),
        ):
            return sympy_interp(
                OptimizedPythonReferenceAnalysis, self.expr_to_proxy, expr
            )
````
- **EN**: Introduces function `_sympy_interp`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_sympy_interp`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 645-672 / 第 645-672 行
````python

        # hash cons on arguments, run expr handler
        self.expr_to_proxy[expr] = _run_sympy_handler(
            OptimizedPythonReferenceAnalysis,
            [self._sympy_interp(arg) for arg in expr.args],
            expr,
        )
        return self.expr_to_proxy[expr]

    def _generate_sym_node(self, s: int | sympy.Expr) -> int | torch.fx.Node:
        if isinstance(s, (int, sympy.Integer)):
            return int(s)
        elif isinstance(s, sympy.Symbol):
            assert s in self.expr_to_proxy, (
                f"Could not find a node corresponding to the symbol {s}"
            )
            return self.expr_to_proxy[s].node
        elif isinstance(s, sympy.Expr):
            return self._sympy_interp(s).node

        elif isinstance(s, torch.fx.Node):
            return s

        else:
            raise ValueError(f"{s} of type {type(s)} is not a valid input")

    def _generate_sym_nodes(
        self, shape: Sequence[sympy.Expr]
````
- **EN**: Introduces function `_generate_sym_node`, function `_generate_sym_nodes`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_generate_sym_node`、函数`_generate_sym_nodes`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 673-700 / 第 673-700 行
````python
    ) -> list[int | torch.fx.Node]:
        return [self._generate_sym_node(s) for s in shape]

    def _generate_allocate(self, line: WrapperLine) -> None:
        assert isinstance(line, AllocateLine)
        buffer = line.node
        name = buffer.get_name()
        assert name not in V.graph.removed_buffers

        device = buffer.get_device()
        assert device
        dtype = buffer.get_dtype()
        shape = self._generate_sym_nodes(buffer.get_size())
        stride = self._generate_sym_nodes(buffer.get_stride())

        node = self.gm.graph.call_function(
            torch.empty_strided,
            args=(shape, stride),
            kwargs={"dtype": dtype, "device": device.type},
        )
        assert name
        node.name = name
        self._record_allocation(buffer, node)

    def _generate_conditional(self, line: WrapperLine) -> None:
        assert isinstance(line, ConditionalLine)

        def get_subgm_attr(subgraph: ir.Subgraph | None) -> torch.fx.Node:
````
- **EN**: Introduces function `_generate_allocate`, function `_generate_conditional`, function `get_subgm_attr`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_generate_allocate`、函数`_generate_conditional`、函数`get_subgm_attr`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 701-728 / 第 701-728 行
````python
            assert subgraph is not None
            return self._get_subgm_attr(subgraph)

        # Access the subgraphs as getattrs.
        ir_node = line.node
        (true_subgm, false_subgm) = [
            get_subgm_attr(subgraph)
            for subgraph in (ir_node.true_subgraph, ir_node.false_subgraph)
        ]

        def generate_buffer(node: ir.IRNode | None) -> torch.fx.Node | None:
            assert node is not None
            return self._generate_buffer(node)

        predicate = generate_buffer(ir_node.predicate)
        assert ir_node.operands is not None
        operands = tuple(generate_buffer(arg) for arg in ir_node.operands)
        fx_node = self.gm.graph.call_function(
            torch.ops.higher_order.cond,
            args=(predicate, true_subgm, false_subgm, operands),
        )
        self._record_allocation(ir_node, fx_node)

    def _generate_assert_size_stride(self, line: WrapperLine) -> None:
        pass

    def _generate_comment(self, line: WrapperLine) -> None:
        assert isinstance(line, CommentLine)
````
- **EN**: Introduces function `generate_buffer`, function `_generate_assert_size_stride`, function `_generate_comment`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `ir_node`, `predicate`, `operands`, `fx_node`, and `args`.
- **CN**: 这里定义了函数`generate_buffer`、函数`_generate_assert_size_stride`、函数`_generate_comment`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `ir_node`、`predicate`、`operands`、`fx_node`、`args` 等值。

### Lines 729-756 / 第 729-756 行
````python
        # We ignore comments in FX IR.

    def _generate_dynamic_scalar(self, line: WrapperLine) -> None:
        assert isinstance(line, DynamicScalarLine)

        ir_node = line.node
        (input_ir_node,) = ir_node.inputs
        assert isinstance(input_ir_node, ir.IRNode)
        input_fx_node = self._generate_buffer(input_ir_node)
        keypath = ir_node.keypath
        graph = self.gm.graph

        def generate_item(x: torch.fx.Node | None) -> torch.fx.Node:
            assert x is not None
            return graph.call_function(
                aten.item.default,
                args=(x,),
            )

        if len(keypath) == 0:
            result_fx_node = generate_item(input_fx_node)
        elif len(keypath) == 1 and isinstance(keypath[0], ConvertIntKey):
            where_fx_node = graph.call_function(
                aten.where.Scalar,
                args=(input_fx_node, 1, 0),
            )
            result_fx_node = generate_item(where_fx_node)
        else:
````
- **EN**: Introduces function `_generate_dynamic_scalar`, function `generate_item`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `ir_node`, `input_fx_node`, `keypath`, `graph`, `args`, `result_fx_node`, and `...+2`.
- **CN**: 这里定义了函数`_generate_dynamic_scalar`、函数`generate_item`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `ir_node`、`input_fx_node`、`keypath`、`graph`、`args`、`result_fx_node`、`另有2项` 等值。

### Lines 757-784 / 第 757-784 行
````python
            raise NotImplementedError(f"Unsupported keypath: {keypath}")

        result_symbol = ir_node.sym
        result_buffer = SymbolBuffer(result_symbol)
        self._record_allocation(result_buffer, result_fx_node)
        self._generate_size_proxy(result_fx_node, result_symbol)

    def _generate_enter_device_context_manager(self, line: WrapperLine) -> None:
        assert isinstance(line, EnterDeviceContextManagerLine)
        # We ignore the device context in FX IR.

    def _generate_exit_device_context_manager(self, line: WrapperLine) -> None:
        assert isinstance(line, ExitDeviceContextManagerLine)
        # We ignore the device context in FX IR.

    def _generate_enter_subgraph(self, line: WrapperLine) -> None:
        assert isinstance(line, EnterSubgraphLine)
        # We ignore memory planning lines in FX IR.

    def _generate_exit_subgraph(self, line: WrapperLine) -> None:
        assert isinstance(line, ExitSubgraphLine)
        # We ignore memory planning lines in FX IR.

    def _generate_free(self, line: WrapperLine) -> None:
        assert isinstance(line, FreeLine)

        buf = line.node

````
- **EN**: Introduces function `_generate_enter_device_context_manager`, function `_generate_exit_device_context_manager`, function `_generate_enter_subgraph`, function `_generate_exit_subgraph`, function `_generate_free`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `result_symbol`, `result_buffer`, and `buf`.
- **CN**: 这里定义了函数`_generate_enter_device_context_manager`、函数`_generate_exit_device_context_manager`、函数`_generate_enter_subgraph`、函数`_generate_exit_subgraph`、函数`_generate_free`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `result_symbol`、`result_buffer`、`buf` 等值。

### Lines 785-812 / 第 785-812 行
````python
        # No need to free placeholders.
        if self.buffer_to_node[buf.get_name()].op == "placeholder":
            return

        self._free(buf)

    def _generate_free_if_not_reused(self, line: WrapperLine) -> None:
        assert isinstance(line, FreeIfNotReusedLine)
        buf = line.node
        assert buf.get_name() not in V.graph.removed_buffers
        if not line.is_reused:
            self._free(buf)

    def _generate_line_context(self, line: WrapperLine) -> None:
        assert isinstance(line, LineContext)
        # We ignore line context in FX IR.

    def _generate_reinterpret(self, line: WrapperLine) -> None:
        assert isinstance(line, ReinterpretLine)
        self._generate_reinterpret_helper(line.node, line.reused_as, line.layout)

    def _generate_reinterpret_helper(
        self, input_buffer: BufferLike, result_buffer: BufferLike, layout: ir.Layout
    ) -> None:
        input_node = self.buffer_to_node[input_buffer.get_name()]

        # Look up output metadata.
        name = result_buffer.get_name()
````
- **EN**: Introduces function `_generate_free_if_not_reused`, function `_generate_line_context`, function `_generate_reinterpret`, function `_generate_reinterpret_helper`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `buf`, `input_node`, and `name`.
- **CN**: 这里定义了函数`_generate_free_if_not_reused`、函数`_generate_line_context`、函数`_generate_reinterpret`、函数`_generate_reinterpret_helper`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `buf`、`input_node`、`name` 等值。

### Lines 813-840 / 第 813-840 行
````python
        assert name
        size = tuple(layout.size)
        stride = tuple(layout.stride)
        if isinstance(layout, ir.NonOwningLayout):
            # Look up the view's layout.
            view = layout.view
            assert isinstance(view, ir.ReinterpretView), (
                f"unexpected type: {type(view)}"
            )
            layout = view.layout
        offset = input_buffer.get_offset() + layout.offset

        # Map ReinterpretView to as_strided.
        result_node = self._create_as_strided(input_node, size, stride, offset)
        result_node.name = name
        self._record_allocation(result_buffer, result_node)

    def _generate_reuse(self, line: WrapperLine) -> None:
        assert isinstance(line, ReuseLine)
        old = line.node
        new = line.reused_as
        assert not any(buf.get_name() in V.graph.removed_buffers for buf in (old, new))
        assert old.get_dtype() == new.get_dtype()

        old_node = self.buffer_to_node[old.get_name()]
        result_node = old_node

        # Change shape and stride.
````
- **EN**: Introduces function `_generate_reuse`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `size`, `stride`, `view`, `layout`, `offset`, `result_node`, and `...+3`.
- **CN**: 这里定义了函数`_generate_reuse`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `size`、`stride`、`view`、`layout`、`offset`、`result_node`、`另有3项` 等值。

### Lines 841-868 / 第 841-868 行
````python
        size = tuple(new.get_size())
        stride = tuple(new.get_stride())
        offset = new.get_offset()
        if (
            tuple(old.get_size()) != size
            or tuple(old.get_stride()) != stride
            or old.get_offset() != offset
        ):
            result_node = self._create_as_strided(old_node, size, stride, offset)

        self._record_allocation(new, result_node)

        # Free the old buffer, if we allocated a new tensor.
        if (
            old.get_name() not in V.graph.get_output_names()
            and line.delete_old
            and result_node is not old_node
        ):
            self._free(old)

    def _generate_multi_output(self, line: WrapperLine) -> None:
        assert isinstance(line, MultiOutputLine)

        arg_node = self.buffer_to_node[line.arg_name]

        # For non-tuple / non-list outputs, map the
        # output to the same node as the input.
        if len(line.indices) == 0:
````
- **EN**: Introduces function `_generate_multi_output`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `size`, `stride`, `offset`, `result_node`, and `arg_node`.
- **CN**: 这里定义了函数`_generate_multi_output`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `size`、`stride`、`offset`、`result_node`、`arg_node` 等值。

### Lines 869-896 / 第 869-896 行
````python
            self.buffer_to_node[line.result_name] = arg_node
            return

        # Extract the index for tuple access.
        inds = line.indices[0][1:]
        assert len(inds) == 1, f"Cannot convert {inds} to an index."
        idx = inds[0]

        node = self.gm.graph.call_function(operator.getitem, args=(arg_node, idx))
        node.name = line.result_name
        self.buffer_to_node[line.result_name] = node

    def _generate_fallback_call(
        self,
        ir_node: ir.ExternKernel,
        args: tuple[Any, ...] | None = None,
        kwargs: dict[str, Any] | None = None,
    ) -> None:
        fx_node = self.gm.graph.call_function(
            ir_node.op_overload,  # type: ignore[arg-type]
            args=args,
            kwargs=kwargs,
        )
        result_buffer = ir_node.codegen_reference()
        self.buffer_to_node[result_buffer] = fx_node
        # For in-place mutation ops (e.g., scatter_reduce_, index_put_),
        # update the buffer mapping for mutated inputs so downstream
        # references to the mutated buffer see the post-mutation node.
````
- **EN**: Introduces function `_generate_fallback_call`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `inds`, `idx`, `node`, `ir_node`, `args`, `kwargs`, and `...+2`.
- **CN**: 这里定义了函数`_generate_fallback_call`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `inds`、`idx`、`node`、`ir_node`、`args`、`kwargs`、`另有2项` 等值。

### Lines 897-924 / 第 897-924 行
````python
        for mutated_name in ir_node.get_mutation_names():
            self.buffer_to_node[mutated_name] = fx_node

    def _generate_index_put_fallback(self, line: WrapperLine) -> None:
        assert isinstance(line, IndexPutFallbackLine)
        ir_node = line.node

        def generate_buffer_or_none(
            x: ir.IRNode | Sequence[ir.IRNode] | None,
        ) -> torch.fx.Node | None:
            """
            Handles None before calling _generate_buffer.
            """
            if x is None:
                return None

            assert isinstance(x, ir.IRNode)
            return self._generate_buffer(x)

        (x, values) = [generate_buffer_or_none(t) for t in ir_node.inputs[:2]]
        indices = tuple(generate_buffer_or_none(t) for t in line.indices)
        accumulate = ir_node.constant_args[0]
        args = (x, indices, values, accumulate)
        self._generate_fallback_call(ir_node, args)

    def _generate_scatter_fallback(self, line: WrapperLine) -> None:
        assert isinstance(line, ScatterFallbackLine)
        ir_node = line.node
````
- **EN**: Introduces function `_generate_index_put_fallback`, function `generate_buffer_or_none`, function `_generate_scatter_fallback`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `ir_node`, `x`, `indices`, `accumulate`, and `args`.
- **CN**: 这里定义了函数`_generate_index_put_fallback`、函数`generate_buffer_or_none`、函数`_generate_scatter_fallback`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `ir_node`、`x`、`indices`、`accumulate`、`args` 等值。

### Lines 925-952 / 第 925-952 行
````python
        assert ir.is_node_sequence(ir_node.inputs)
        (x, index, src) = [self._generate_buffer(t) for t in ir_node.inputs] + (
            [] if ir_node.src_is_tensor else [ir_node.constant_args[1]]
        )
        args = (x, ir_node.constant_args[0], index, src)
        kwargs = {}
        if reduce := ir_node.kwargs.get("reduce"):
            kwargs["reduce"] = reduce
        # Only pass kwargs that the op's schema actually accepts, since
        # ScatterFallback stores both reduce and include_self for all
        # scatter variants, but not all ops support them (e.g.,
        # scatter_.value has no kwargs, scatter_reduce_.two has both).
        assert isinstance(ir_node.op_overload, torch._ops.OpOverload)
        schema_arg_names = OrderedSet(
            [a.name for a in ir_node.op_overload._schema.arguments]
        )
        kwargs = {k: v for k, v in ir_node.kwargs.items() if k in schema_arg_names}

        self._generate_fallback_call(ir_node, args, kwargs)

    def _generate_null(self, line: WrapperLine) -> None:
        assert isinstance(line, NullLine)
        # Does nothing.

    def _generate_comm_buffer_allocate(self, line: WrapperLine) -> None:
        assert isinstance(line, AllocateLine) and line.comm_buffer
        raise NotImplementedError("Comm buffer allocation is not yet supported")

````
- **EN**: Introduces function `_generate_null`, function `_generate_comm_buffer_allocate`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `args`, `kwargs`, and `schema_arg_names`.
- **CN**: 这里定义了函数`_generate_null`、函数`_generate_comm_buffer_allocate`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `args`、`kwargs`、`schema_arg_names` 等值。

### Lines 953-980 / 第 953-980 行
````python
    def _generate_comm_buffer_free(self, line: WrapperLine) -> None:
        assert isinstance(line, FreeIfNotReusedLine) and line.comm_buffer
        self._free(line.node)

    def _generate_triton_call(self, line: WrapperLine) -> None:
        assert isinstance(line, KernelCallLine)

        # Collect all kwargs, including autotuned block sizes.
        call_args = self._lookup_args(line.call_args)
        kernel = self.kernels[line.kernel_name]
        tuner = kernel.tuner

        def tune_kernel(tuner: CachingAutotuner, call_args: Sequence[Any]) -> None:
            from triton.runtime import driver

            log.info("Autotuning Triton kernel %s at compile time.", kernel_name)

            device = driver.active.get_current_device()

            stream = driver.active.get_current_stream(device)

            def node_to_tuning_arg(arg: Any) -> Any:
                """
                Create real tensors for autotuning arguments, substituting size hints
                for dynamic shapes.
                """

                def to_size_hint_sympy_int(arg: sympy.Expr | int) -> int:
````
- **EN**: Imports dependencies such as `triton.runtime` for the logic in this range. Introduces function `_generate_comm_buffer_free`, function `_generate_triton_call`, function `tune_kernel`, function `node_to_tuning_arg`, function `to_size_hint_sympy_int`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里导入了 `triton.runtime` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_generate_comm_buffer_free`、函数`_generate_triton_call`、函数`tune_kernel`、函数`node_to_tuning_arg`、函数`to_size_hint_sympy_int`。借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 981-1008 / 第 981-1008 行
````python
                    return V.graph.sizevars.optimization_hint(arg)

                def to_size_hint_list(arg: list[torch.SymInt | int]) -> list[int]:
                    args_sympy = [
                        x.node.expr if isinstance(x, torch.SymInt) else x for x in arg
                    ]
                    return pytree.tree_map(to_size_hint_sympy_int, args_sympy)

                if not isinstance(arg, torch.fx.Node):
                    return to_size_hint_sympy_int(arg)

                fake = arg.meta["val"]
                return torch.empty_strided(
                    to_size_hint_list(fake.shape),
                    to_size_hint_list(fake.stride()),
                    dtype=fake.dtype,
                    device=device,
                ).zero_()

            # call args can be fx nodes or sympy expressions or integers!
            arg_values = [node_to_tuning_arg(arg) for arg in call_args]
            tuner.run(*arg_values, stream=stream)

        # Optionally autotune the kernels.
        # The FX backend currently only supports compile-time tuning.
        kernel_name = tuner.fn.__name__
        if config.triton.autotune_at_compile_time:
            # Skip compile-time autotuning if any unbacked symbol lacks a user-provided
````
- **EN**: Introduces function `to_size_hint_list`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`to_size_hint_list`。借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。

### Lines 1009-1036 / 第 1009-1036 行
````python
            # optimization hint — autotuning with the generic fallback would
            # produce meaningless results.
            hinted = V.graph.sizevars.all_unbacked_explicitly_hinted
            can_tune = True
            for arg in call_args:
                if isinstance(arg, torch.fx.Node):
                    fake = arg.meta["val"]
                    if not hinted(list(fake.shape) + list(fake.stride())):
                        can_tune = False
                        break
                elif not hinted(arg):
                    can_tune = False
                    break
            if can_tune:
                tune_kernel(tuner, call_args)
            else:
                log.info(
                    "Detected unhinted unbacked symints. Skipping compile-time autotuning for kernel %s.",
                    kernel_name,
                )
        else:
            log.info(
                "Skipping autotuning for kernel %s. Set config.triton.autotune_at_compile_time = True to enable.",
                kernel_name,
            )

        triton_meta = tuner.triton_meta
        signature = triton_meta["signature"]
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `hinted`, `can_tune`, `fake`, `else`, `triton_meta`, and `signature`. This range continues the implementation of function `FxConverter._generate_triton_call`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含分支、循环或上下文管理等控制流。初始化或更新了 `hinted`、`can_tune`、`fake`、`else`、`triton_meta`、`signature` 等值。这一段延续了函数`FxConverter._generate_triton_call` 的具体实现。

### Lines 1037-1064 / 第 1037-1064 行
````python

        def add_constants_to_call_args(
            call_args: Sequence[Any], cfg: Config
        ) -> tuple[Any, ...]:
            """
            Add constant kwargs to the arg list.
            """
            # Add args from the proper Triton signature.
            # Exclude constants and config kwargs, as those are tracked separately.
            new_call_args = []
            constants = triton_meta["constants"]
            call_kwargs = {
                key: val
                for key, val in zip(signature, call_args)
                # pyrefly: ignore [missing-attribute]
                if key not in constants and key not in cfg.kwargs
            }

            # Add constants stored as Triton metadata, in signature order.
            call_kwargs |= constants
            new_call_args = [
                call_kwargs[key]
                for key in signature
                # pyrefly: ignore [missing-attribute]
                if key not in cfg.kwargs
            ]

            # Add Inductor's extra launcher args to the end.
````
- **EN**: Introduces function `add_constants_to_call_args`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `call_args`, `new_call_args`, `constants`, `call_kwargs`, and `key`.
- **CN**: 这里定义了函数`add_constants_to_call_args`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含分支、循环或上下文管理等控制流。初始化或更新了 `call_args`、`new_call_args`、`constants`、`call_kwargs`、`key` 等值。

### Lines 1065-1092 / 第 1065-1092 行
````python
            if extra_launcher_args := tuner.inductor_meta.get("extra_launcher_args"):
                new_call_args.extend(
                    call_args[len(call_args) - len(extra_launcher_args) :]
                )

            return tuple(new_call_args)

        kernel_config = tuner.compile_results[0].config
        extra_options = getattr(kernel_config, "extra_options", None)
        call_args = add_constants_to_call_args(call_args, kernel_config)
        call_args, grid = tuner._interpret_args_grid(call_args, kernel_config)
        call_kwargs = dict(zip(signature, call_args))
        # pyrefly: ignore [missing-attribute]
        assert not any(kwarg in kernel_config.kwargs for kwarg in call_kwargs), (
            f"kwargs overlap config: {call_kwargs}"
        )
        # pyrefly: ignore [missing-attribute]
        call_kwargs.update(kernel_config.kwargs)

        # Replace sympy.floor with FloorDiv, to make the expression traceable.
        grid = [replace_floor_div(x) if isinstance(x, sympy.Expr) else x for x in grid]
        wrapper_grid = [tuple(self._generate_sym_nodes(grid))]
        call_kwargs = {
            name: self._generate_sym_node(val) for name, val in call_kwargs.items()
        }

        # Store non-graphable kwargs in the side table.
        (
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `kernel_config`, `extra_options`, `call_args`, `call_kwargs`, `grid`, `wrapper_grid`, and `...+1`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `kernel_config`、`extra_options`、`call_args`、`call_kwargs`、`grid`、`wrapper_grid`、`另有1项` 等值。

### Lines 1093-1120 / 第 1093-1120 行
````python
            call_kwargs,
            constant_args_idx,
        ) = tracing_triton_hopifier_singleton.store_non_graphable_args(call_kwargs)

        triton_node = self.gm.graph.call_function(
            triton_kernel_wrapper_mutation,
            kwargs={
                "kernel_idx": kernel.wrapped.kernel_idx,
                "constant_args_idx": constant_args_idx,
                "grid": wrapper_grid,
                "tma_descriptor_metadata": {},
                "kwargs": call_kwargs,
            },
        )
        if extra_options:
            triton_node.meta["extra_options"] = extra_options

    def _generate_extern_kernel_alloc(self, line: WrapperLine) -> None:
        assert isinstance(line, ExternKernelAllocLine)
        node = line.node
        self._generate_extern_kernel_common(node, node)

    def _generate_extern_kernel_out(
        self,
        line: WrapperLine,
    ) -> None:
        assert isinstance(line, ExternKernelOutLine)
        node = line.node
````
- **EN**: Introduces function `_generate_extern_kernel_alloc`, function `_generate_extern_kernel_out`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `triton_node`, `kwargs`, `node`, and `line`.
- **CN**: 这里定义了函数`_generate_extern_kernel_alloc`、函数`_generate_extern_kernel_out`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `triton_node`、`kwargs`、`node`、`line` 等值。

### Lines 1121-1148 / 第 1121-1148 行
````python
        out_node = node.output_view if node.output_view else node
        self._generate_extern_kernel_common(node, out_node)

    def _generate_extern_kernel_common(
        self, kernel: ir.ExternKernel, out_ir_node: ir.IRNode
    ) -> None:
        """
        Generates FX IR from either ExternKernelAlloc or ExternKernelOut.
        """

        # Get FX nodes corresponding to the call args.
        assert ir.is_node_sequence(kernel.inputs)
        tensor_nodes = tuple(self._generate_buffer(arg) for arg in kernel.inputs)
        if hasattr(kernel, "unflatten_args"):
            args, _ = kernel.unflatten_args(tensor_nodes, kernel.constant_args)
        else:
            args = tensor_nodes + tuple(kernel.constant_args)

        # Get the result buffer.
        # Some kernels write to a pre-existing output tensor via the "out" kwarg.
        # Materialize any IR nodes in kwargs to FX nodes (e.g., TensorBox -> Tensor).
        kwargs = {
            k: self._generate_buffer(v) if isinstance(v, ir.IRNode) else v
            for k, v in kernel.kwargs.items()
        }

        result_buffer: str | None = None
        if isinstance(kernel, ir.ExternKernelOut):
````
- **EN**: Introduces function `_generate_extern_kernel_common`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `out_node`, `tensor_nodes`, `else`, `args`, `kwargs`, `k`, and `...+1`.
- **CN**: 这里定义了函数`_generate_extern_kernel_common`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `out_node`、`tensor_nodes`、`else`、`args`、`kwargs`、`k`、`另有1项` 等值。

### Lines 1149-1176 / 第 1149-1176 行
````python
            kwargs["out"] = self.buffer_to_node[out_ir_node.codegen_reference()]
        elif isinstance(kernel.layout, (ir.Layout, ir.MultiOutputLayout)):
            result_buffer = kernel.get_name()
        elif isinstance(kernel.layout, ir.NoneLayout):
            pass
        else:
            raise NotImplementedError(f"Unrecognized output layout: {kernel.layout}")

        fx_node = self.gm.graph.call_function(
            kernel.op_overload,  # type: ignore[arg-type]
            args=args,
            kwargs=kwargs,
        )

        # Assign the result to the given name.
        if result_buffer:
            assert "out" not in kwargs, (
                f"Extern kernel '{kernel}' has both result and out kwarg. Expected only one."
            )
            fx_node.name = result_buffer
            self.buffer_to_node[result_buffer] = fx_node

    def _generate_kernel_call(self, line: WrapperLine) -> None:
        assert isinstance(line, KernelCallLine)
        if not line.triton:
            raise NotImplementedError("FX conversion only supports Triton kernels.")

        self._generate_triton_call(line)
````
- **EN**: Introduces function `_generate_kernel_call`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_generate_kernel_call`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1177-1204 / 第 1177-1204 行
````python

    def _generate_kernel_definition(self, line: WrapperLine) -> None:
        assert isinstance(line, KernelDefinitionLine)

        # Generate code for the kernel.
        kernel_code = PythonWrapperCodegen._format_kernel_definition(
            line.kernel_name, line.kernel_body, metadata=line.metadata
        )

        # Import the module and store the JIT kernel.
        tuner = self._import_kernel(kernel_code, line.kernel_name)
        wrapped = wrap_triton(tuner.fn)
        self.kernels[line.kernel_name] = TritonKernel(tuner, wrapped)

    def _generate_symbolic_call_arg(self, line: WrapperLine) -> None:
        assert isinstance(line, SymbolicCallArgLine)
        # Store the arg: expr mapping for later use.
        arg = line.arg

        inner_expr_proxy = self._sympy_interp(arg.inner_expr)
        self.expr_to_proxy[arg.inner] = inner_expr_proxy

    def _generate_unbacked_symbol_defs(self, line: WrapperLine) -> None:
        assert isinstance(line, UnbackedSymbolDefsLine)
        graph = self.gm.graph

        def convert_key(node: torch.fx.Node, path: pytree.KeyPath) -> torch.fx.Node:
            """
````
- **EN**: Introduces function `_generate_kernel_definition`, function `_generate_symbolic_call_arg`, function `_generate_unbacked_symbol_defs`, function `convert_key`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `kernel_code`, `tuner`, `wrapped`, `arg`, `inner_expr_proxy`, and `graph`.
- **CN**: 这里定义了函数`_generate_kernel_definition`、函数`_generate_symbolic_call_arg`、函数`_generate_unbacked_symbol_defs`、函数`convert_key`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `kernel_code`、`tuner`、`wrapped`、`arg`、`inner_expr_proxy`、`graph` 等值。

### Lines 1205-1232 / 第 1205-1232 行
````python
            Generate FX IR for each key entry.
            """
            # Base case.
            if len(path) == 0:
                return node

            # Process the first entry and recurse.
            entry = path[0]
            if isinstance(entry, CallMethodKey):
                target = {
                    "size": aten.sym_size.int,
                    "stride": aten.sym_stride.int,
                    "storage_offset": aten.sym_storage_offset,
                }[entry.name]
                assert callable(target)
                node = graph.call_function(
                    target,
                    args=(
                        (node, path[1].idx)
                        if len(path) > 1 and isinstance(path[1], pytree.SequenceKey)
                        else (node,)
                    ),
                )
                return convert_key(node, path[1 + len(node.args) :])
            elif isinstance(entry, pytree.SequenceKey):
                node = graph.call_function(operator.getitem, args=(node, entry.idx))
                return convert_key(node, path[1:])
            elif isinstance(entry, DivideByKey):
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `entry`, `target`, `node`, and `args`. This range continues the implementation of function `FxConverter._generate_unbacked_symbol_defs.convert_key`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `entry`、`target`、`node`、`args` 等值。这一段延续了函数`FxConverter._generate_unbacked_symbol_defs.convert_key` 的具体实现。

### Lines 1233-1251 / 第 1233-1251 行
````python
                node = graph.call_function(
                    operator.floordiv, args=(node, entry.divisor)
                )
                return convert_key(node, path[1:])
            else:
                raise NotImplementedError(f"Unrecognized entry type: {type(entry)}")

        root_node = self.buffer_to_node[line.output_name]
        unbacked_bindings = line.unbacked_bindings
        assert unbacked_bindings is not None
        for s, keypath in unbacked_bindings.items():
            # Check if we already generated this symbol.
            if s.name in self.buffer_to_node:
                continue

            node = convert_key(root_node, keypath)
            out_buffer = SymbolBuffer(s)
            self._record_allocation(out_buffer, node)
            self._generate_size_proxy(node, s)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `node`, `else`, `root_node`, `unbacked_bindings`, and `out_buffer`. This range continues the implementation of function `FxConverter._generate_unbacked_symbol_defs`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `node`、`else`、`root_node`、`unbacked_bindings`、`out_buffer` 等值。这一段延续了函数`FxConverter._generate_unbacked_symbol_defs` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Autotuning support  
  **CN**: 自动调优支持

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `functools`, `logging`, `operator`, `textwrap`, `collections`, `collections.abc`, `typing`
- **Third-party / 第三方**: `sympy`, `triton.runtime`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._export.passes._node_metadata_hook`, `torch._higher_order_ops.triton_kernel_wrap`, `torch._inductor.codecache`, `torch._inductor.runtime.triton_heuristics`, `torch._inductor.select_algorithm`, `torch._inductor.utils`, `torch._inductor.virtualized`, `torch._library.triton`, `torch.fx`, `torch.fx.experimental.symbolic_shapes`, `torch.utils`, `torch.utils._ordered_set`, `torch.utils._sympy.functions`, `torch.utils._sympy.interp`, `torch.utils._sympy.reference`, `torch.utils._sympy.solve`, `..`, `..runtime.triton_compat`, `..utils`, `...+2`
