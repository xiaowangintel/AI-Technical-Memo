# cpp_template_kernel.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/cpp_template_kernel.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `CppTemplateKernel`, and `CppTemplateCaller`. It exposes functions such as `parse_expr_with_index_symbols`, and `wrap_with_tensorbox`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `CppTemplateKernel`、`CppTemplateCaller` 等类。同时提供 `parse_expr_with_index_symbols`、`wrap_with_tensorbox` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import itertools
from collections.abc import Callable, Iterable
from typing import Any
from unittest.mock import patch

import sympy
from sympy.parsing.sympy_parser import parse_expr

import torch
from torch._inductor.utils import do_bench_using_profiling
from torch.utils._ordered_set import OrderedSet
from torch.utils._sympy.symbol import SymT

from .. import config, cpp_builder, ir, lowering as L
from ..autotune_process import CppBenchmarkRequest
from ..loop_body import LoopBody
from ..select_algorithm import PartialRender
from ..utils import sympy_index_symbol, sympy_index_symbol_with_prefix
from ..virtualized import V
````
- **EN**: Imports dependencies such as `itertools`, `collections.abc`, `typing`, `unittest.mock`, `sympy`, `sympy.parsing.sympy_parser`, and `...+10` for the logic in this range. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里导入了 `itertools`、`collections.abc`、`typing`、`unittest.mock`、`sympy`、`sympy.parsing.sympy_parser`、`另有10项` 等依赖，为后续逻辑提供基础能力。借助符号表达式来推理索引、形状或代数性质。

### Lines 21-40 / 第 21-40 行
````python
from .common import REMOVED
from .cpp import CppKernel, CppKernelProxy, KernelGroup, ParallelDepth
from .cpp_utils import cexpr_index, DTYPE_TO_CPP, LocalBufferContext


def parse_expr_with_index_symbols(expr):
    if isinstance(expr, sympy.Expr):
        return expr
    elif isinstance(expr, (list, tuple)):
        return [parse_expr_with_index_symbols(e) for e in expr]
    else:
        expr = parse_expr(str(expr))
        int_symbols = {sym: sympy_index_symbol(sym.name) for sym in expr.free_symbols}
        return expr.subs(int_symbols)


def wrap_with_tensorbox(node) -> ir.TensorBox:
    return (
        ir.TensorBox.create(node) if isinstance(node, ir.Buffer) else ir.TensorBox(node)
    )
````
- **EN**: Imports dependencies such as `.common`, `.cpp`, and `.cpp_utils` for the logic in this range. Introduces function `parse_expr_with_index_symbols`, function `wrap_with_tensorbox`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `.common`、`.cpp`、`.cpp_utils` 等依赖，为后续逻辑提供基础能力。这里定义了函数`parse_expr_with_index_symbols`、函数`wrap_with_tensorbox`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 41-60 / 第 41-60 行
````python


class CppTemplateKernel(CppKernel):
    def __init__(self, kernel_name, num_threads):
        super().__init__(None, num_threads)
        self.kernel_name = kernel_name
        self.render_hooks = {}
        self.local_buffers = {}

    def render(self, template, **kwargs):
        return PartialRender(
            template.render(kernel=self, **kwargs), self.render_hooks
        ).finalize_all()

    def def_kernel(
        self,
        inputs: dict[str, ir.Buffer],
        outputs: dict[str, ir.Buffer],
        aliases: dict[str, str] | None = None,
        function_name: str = "",
````
- **EN**: Introduces class `CppTemplateKernel`, function `__init__`, function `render`, function `def_kernel`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `inputs`, `outputs`, `aliases`, and `function_name`.
- **CN**: 这里定义了类`CppTemplateKernel`、函数`__init__`、函数`render`、函数`def_kernel`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `inputs`、`outputs`、`aliases`、`function_name` 等值。

### Lines 61-80 / 第 61-80 行
````python
        extra_sizevars: list[sympy.Expr] | None = None,
        placeholder: str = "<DEF_KERNEL>",
    ) -> str:
        if len(function_name) == 0:
            function_name = str(self.kernel_name)
        for name, inp in inputs.items():
            if inp is not None:
                self.args.input_buffers[inp.get_name()] = name
        for name, out in outputs.items():
            self.args.output_buffers[out.get_name()] = name
        if aliases is not None:
            for alias, orig in aliases.items():
                if orig in self.args.input_buffers:
                    self.args.input_buffers[alias] = self.args.input_buffers[orig]
                if orig in self.args.output_buffers:
                    self.args.output_buffers[alias] = self.args.output_buffers[orig]

        unique_sizevars = OrderedSet(
            s
            for input in inputs.values()
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `extra_sizevars`, `placeholder`, `function_name`, and `unique_sizevars`. This range continues the implementation of function `CppTemplateKernel.def_kernel`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含分支、循环或上下文管理等控制流。初始化或更新了 `extra_sizevars`、`placeholder`、`function_name`、`unique_sizevars` 等值。这一段延续了函数`CppTemplateKernel.def_kernel` 的具体实现。

### Lines 81-100 / 第 81-100 行
````python
            if input is not None
            for sym in itertools.chain(input.get_size(), input.get_stride())
            if isinstance(sym, sympy.Expr)
            for s in sym.free_symbols
        )
        unique_sizevars.update(
            s
            for sym in extra_sizevars or []
            if isinstance(sym, sympy.Expr)
            for s in sym.free_symbols
        )
        unique_sizevars.update(
            s
            for output in outputs.values()
            for sym in itertools.chain(output.get_size(), output.get_stride())
            if isinstance(sym, sympy.Expr)
            for s in sym.free_symbols
        )
        sizevars = sorted(unique_sizevars, key=str)
        for sizevar in sizevars:
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `sizevars`. This range continues the implementation of function `CppTemplateKernel.def_kernel`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含分支、循环或上下文管理等控制流。初始化或更新了 `sizevars` 等值。这一段延续了函数`CppTemplateKernel.def_kernel` 的具体实现。

### Lines 101-120 / 第 101-120 行
````python
            self.args.sizevars[sizevar] = f"k{sizevar}"

        def hook():
            # remove all aliases before generate function definition
            if aliases is not None:
                for alias in aliases:
                    if alias in self.args.input_buffers:
                        raise AssertionError(
                            f"input_buffers cannot be removed: {alias}"
                        )
                    if alias in self.args.output_buffers:
                        self.args.output_buffers[alias] = REMOVED
            cpp_argdefs, _, _ = self.args.cpp_argdefs()
            return f"void {function_name}({', '.join(cpp_argdefs)})"

        assert placeholder not in self.render_hooks
        self.render_hooks[placeholder] = hook
        return placeholder

    def call_kernel(self, name: str, node: ir.CppTemplateBuffer):
````
- **EN**: Introduces function `hook`, function `call_kernel`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`hook`、函数`call_kernel`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 121-140 / 第 121-140 行
````python
        wrapper = V.graph.wrapper_code
        _, call_args, arg_types = self.args.cpp_argdefs()
        wrapper.generate_kernel_call(name, call_args, triton=False, arg_types=arg_types)

    def dtype(self, node: ir.Buffer) -> str:
        return DTYPE_TO_CPP[node.get_dtype()]

    def acc_dtype(self, node: ir.Buffer) -> str:
        if node.get_dtype() in [torch.float32, torch.bfloat16, torch.half]:
            return "float"
        else:
            raise NotImplementedError(f"Unsupported dtype: {node.get_dtype()}")

    def size(self, node: ir.Buffer, dim: int) -> str:
        return cexpr_index(self.rename_indexing(node.get_size()[dim]))

    def stride(self, node: ir.Buffer, dim: int) -> str:
        return cexpr_index(self.rename_indexing(node.get_stride()[dim]))

    def index(self, node: ir.Buffer, indices: list[Any]) -> str:
````
- **EN**: Introduces function `dtype`, function `acc_dtype`, function `size`, function `stride`, function `index`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`dtype`、函数`acc_dtype`、函数`size`、函数`stride`、函数`index`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 141-160 / 第 141-160 行
````python
        indexer = node.get_layout().as_fixed().make_indexer()
        index = indexer(parse_expr_with_index_symbols(indices))
        index = self.rename_indexing(index)
        outer_name = node.get_name()
        inner_name = (
            outer_name
            if outer_name in self.local_buffers
            else self.args.input(node.get_name())
        )
        return f"{inner_name}[{cexpr_index(index)}]"

    def slice_nd(self, node, ranges: list[tuple[Any, Any]]) -> ir.ReinterpretView:
        """
        Slice the given node with a list of ranges (start and end) corresponding to its dims.
        The dim is not sliced if the corresponding range is empty.
        """
        assert len(ranges) == len(node.get_size()), f"{ranges=}, {node=}"
        sliced = wrap_with_tensorbox(node)
        for dim, _range in enumerate(ranges):
            if len(_range) == 0:
````
- **EN**: Introduces function `slice_nd`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `indexer`, `index`, `outer_name`, `inner_name`, and `sliced`.
- **CN**: 这里定义了函数`slice_nd`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `indexer`、`index`、`outer_name`、`inner_name`、`sliced` 等值。

### Lines 161-180 / 第 161-180 行
````python
                continue
            assert len(_range) == 2
            start, end = parse_expr_with_index_symbols(_range)
            sliced = L.slice_(sliced, dim, start, end, clamp=False)
        assert isinstance(sliced, ir.TensorBox)
        assert isinstance(sliced.data, ir.ReinterpretView), sliced.data
        return sliced.data

    def select(self, node, dim: int, idx: int) -> ir.ReinterpretView:
        # We avoid using L.select here because we need clamp=False so the dim after slicing
        # is 1 instead of a sympy expression of symbol - dim_size.
        node = wrap_with_tensorbox(node)
        idx = ir.View.handle_negative_index(idx, node.get_size()[dim])
        sliced = L.squeeze(L.slice_(node, dim, idx, idx + 1, clamp=False), dim)
        assert isinstance(sliced.data, ir.ReinterpretView), sliced.data
        return sliced.data

    def view(self, node, sizes: list[Any]) -> ir.IRNode:
        node = wrap_with_tensorbox(node)
        sizes = parse_expr_with_index_symbols(sizes)
````
- **EN**: Introduces function `select`, function `view`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `sliced`, `node`, `idx`, and `sizes`.
- **CN**: 这里定义了函数`select`、函数`view`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `sliced`、`node`、`idx`、`sizes` 等值。

### Lines 181-200 / 第 181-200 行
````python
        return L.view(node, sizes).data  # type: ignore[arg-type]

    def permute(self, node, dims):
        node = wrap_with_tensorbox(node)
        permuted = L.permute(node, dims).data
        assert isinstance(permuted, ir.ReinterpretView)
        return permuted

    def maybe_codegen_profile(self) -> str:
        if config.cpp.enable_kernel_profile:
            graph_id = V.graph.graph_id
            prefix = "graph_" + str(graph_id) + "_" if graph_id is not None else ""
            handle_str = (
                "torch::aot_inductor::RAIIAtenRecordFunctionHandle "
                f'record_{prefix}{self.kernel_name}_("{prefix}{self.kernel_name}", nullptr);'
            )
            return handle_str
        else:
            return ""

````
- **EN**: Introduces function `permute`, function `maybe_codegen_profile`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `node`, `permuted`, `graph_id`, `prefix`, `handle_str`, and `else`.
- **CN**: 这里定义了函数`permute`、函数`maybe_codegen_profile`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `node`、`permuted`、`graph_id`、`prefix`、`handle_str`、`else` 等值。

### Lines 201-220 / 第 201-220 行
````python
    def unroll_pragma(self, unroll):
        if cpp_builder.is_gcc():
            return f"#pragma GCC unroll {unroll}"
        else:
            return f"#pragma unroll {unroll}"

    def define_buffer(self, name, sizes: list[Any], dtype=torch.float) -> str:
        """Define kernel local buffer"""
        sizes = parse_expr_with_index_symbols(sizes)
        buf = ir.Buffer(
            name=name, layout=ir.FixedLayout(torch.device("cpu"), dtype, sizes)
        )
        self.local_buffers[name] = buf
        ctype = f"{DTYPE_TO_CPP[dtype]}"
        numel = f"{cexpr_index(buf.get_numel())}"
        return f"auto _{name} = std::make_unique<{ctype}[]>({numel}); auto {name} = _{name}.get();"

    def define_stack_allocated_buffer(
        self, name, sizes: list[Any], dtype=torch.float
    ) -> str:
````
- **EN**: Introduces function `unroll_pragma`, function `define_buffer`, function `define_stack_allocated_buffer`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`unroll_pragma`、函数`define_buffer`、函数`define_stack_allocated_buffer`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 221-240 / 第 221-240 行
````python
        """Define stack-allocated buffer"""
        sizes = parse_expr_with_index_symbols(sizes)
        buf = ir.Buffer(
            name=name, layout=ir.FixedLayout(torch.device("cpu"), dtype, sizes)
        )
        self.local_buffers[name] = buf
        ctype = f"{DTYPE_TO_CPP[dtype]}"
        numel = f"{cexpr_index(buf.get_numel())}"
        return f"alignas(64) {ctype} _{name}[{numel}]; {ctype}* {name} = _{name};"

    def reinit_buffer_if_null(self, name):
        """Reinit the previously defined local buffer if it is null"""
        assert name in self.local_buffers
        buf = self.local_buffers[name]
        ctype = f"{DTYPE_TO_CPP[buf.layout.dtype]}"
        numel = f"{cexpr_index(buf.get_numel())}"
        return f"if (_{name} == nullptr) {{ _{name} = std::make_unique<{ctype}[]>({numel}); {name} = _{name}.get(); }}"

    def release_buffer(self, name):
        """Codegen the code to release the ownership of a local buffer to others"""
````
- **EN**: Introduces function `reinit_buffer_if_null`, function `release_buffer`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`reinit_buffer_if_null`、函数`release_buffer`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 241-260 / 第 241-260 行
````python
        assert name in self.local_buffers
        return f"_{name}.release()"

    def store_pointwise_nodes(
        self,
        dst: ir.Buffer,
        nodes: list[ir.IRNode],
        offsets: list[sympy.Expr] | None = None,
        reindexers: list[Callable[[list[Any]], list[Any]] | None] | None = None,
    ) -> str:
        var_sizes = (tuple(dst.get_size()), ())
        var_ranges = {
            sympy_index_symbol_with_prefix(SymT.INDEX, i): sz
            for i, sz in enumerate(var_sizes[0])
        }
        if not offsets:
            offsets = [sympy.S.Zero] * len(var_sizes[0])
        if not reindexers:
            reindexers = [None] * len(nodes)
        assert len(offsets) == len(var_sizes[0])
````
- **EN**: Introduces function `store_pointwise_nodes`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`store_pointwise_nodes`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 261-280 / 第 261-280 行
````python
        output_index = dst.get_layout().make_indexer()([*var_ranges.keys()])
        kernel_group = KernelGroup()
        kernel_group.args = self.args
        cpp_kernel_proxy = CppKernelProxy(kernel_group)
        bodies = []
        var_sizes_list = []
        for i, node in enumerate(nodes):
            output_name = node.get_name() if i < len(nodes) - 1 else dst.get_name()
            node = node.data if isinstance(node, ir.ComputedBuffer) else node
            assert isinstance(node, ir.Pointwise), node

            def fn(*args):
                assert len(args) == 2
                assert len(args[0]) == len(var_sizes[0])
                assert len(args[1]) == 0
                new_args = [arg + offset for arg, offset in zip(args[0], offsets)]  # type: ignore[arg-type]
                if reindexers[i] is not None:
                    new_args = reindexers[i](new_args)  # type: ignore[misc]
                V.ops.store(
                    output_name,
````
- **EN**: Introduces function `fn`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `output_index`, `kernel_group`, `cpp_kernel_proxy`, `bodies`, `var_sizes_list`, `output_name`, and `...+2`.
- **CN**: 这里定义了函数`fn`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `output_index`、`kernel_group`、`cpp_kernel_proxy`、`bodies`、`var_sizes_list`、`output_name`、`另有2项` 等值。

### Lines 281-300 / 第 281-300 行
````python
                    output_index,
                    node.make_loader()(new_args).value,
                )

            body = LoopBody(
                fn,
                (list(var_ranges.keys()), ()),
                var_ranges,
                list(var_ranges.keys()),
                tuple(),
            )
            bodies.append(body)
            var_sizes_list.append(var_sizes)

        cpp_kernel_proxy.codegen_loop_bodies(bodies, var_sizes_list)

        def max_parallel_depth():
            return ParallelDepth(parallel_depth=0, start_depth=0)

        # This loop is not parallelized since it is not the outermost loop.
````
- **EN**: Introduces function `max_parallel_depth`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `body`.
- **CN**: 这里定义了函数`max_parallel_depth`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `body` 等值。

### Lines 301-320 / 第 301-320 行
````python
        with patch.object(
            cpp_kernel_proxy.loop_nest, "max_parallel_depth", max_parallel_depth
        ):
            kernel_group.finalize_kernel(cpp_kernel_proxy, [])
        return kernel_group.loops_code.getvalue()

    def store_grouped_gemm_pointwise_nodes(
        self,
        dst: tuple[ir.Buffer],
        nodes: list[ir.IRNode],
        offsets: list[sympy.Expr],
        reindexers: list[Callable[[list[Any]], list[Any]] | None],
        output_names: list[str],
    ) -> str:
        ref_dst = dst[0]
        var_sizes = (tuple(ref_dst.get_size()), ())
        var_ranges = {
            sympy_index_symbol_with_prefix(SymT.INDEX, i): sz
            for i, sz in enumerate(var_sizes[0])
        }
````
- **EN**: Introduces function `store_grouped_gemm_pointwise_nodes`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`store_grouped_gemm_pointwise_nodes`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 321-340 / 第 321-340 行
````python
        assert offsets, "offsets should be set outside"
        assert all(len(offset) == len(var_sizes[0]) for offset in offsets)
        output_index = ref_dst.get_layout().make_indexer()([*var_ranges.keys()])
        kernel_group = KernelGroup()
        kernel_group.args = self.args
        cpp_kernel_proxy = CppKernelProxy(kernel_group)
        bodies = []
        var_sizes_list = []
        for i, node in enumerate(nodes):
            output_name = output_names[i]
            node = node.data if isinstance(node, ir.ComputedBuffer) else node
            assert isinstance(node, ir.Pointwise), node

            def fn(*args):
                assert len(args) == 2
                assert len(args[0]) == len(var_sizes[0])
                assert len(args[1]) == 0
                new_args = [arg + offset for arg, offset in zip(args[0], offsets[i])]  # type: ignore[arg-type]
                if reindexers[i] is not None:
                    new_args = reindexers[i](new_args)  # type: ignore[misc]
````
- **EN**: Introduces function `fn`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `output_index`, `kernel_group`, `cpp_kernel_proxy`, `bodies`, `var_sizes_list`, `output_name`, and `...+2`.
- **CN**: 这里定义了函数`fn`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `output_index`、`kernel_group`、`cpp_kernel_proxy`、`bodies`、`var_sizes_list`、`output_name`、`另有2项` 等值。

### Lines 341-360 / 第 341-360 行
````python
                V.ops.store(
                    output_name,
                    output_index,
                    node.make_loader()(new_args).value,
                )

            body = LoopBody(
                fn,
                (list(var_ranges.keys()), ()),
                var_ranges,
                list(var_ranges.keys()),
                tuple(),
            )
            bodies.append(body)
            var_sizes_list.append(var_sizes)

        cpp_kernel_proxy.codegen_loop_bodies(bodies, var_sizes_list)

        def max_parallel_depth():
            return ParallelDepth(parallel_depth=0, start_depth=0)
````
- **EN**: Introduces function `max_parallel_depth`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `body`.
- **CN**: 这里定义了函数`max_parallel_depth`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `body` 等值。

### Lines 361-380 / 第 361-380 行
````python

        # This loop is not parallelized since it is not the outermost loop.
        with patch.object(
            cpp_kernel_proxy.loop_nest, "max_parallel_depth", max_parallel_depth
        ):
            kernel_group.finalize_kernel(cpp_kernel_proxy, [])
        return kernel_group.loops_code.getvalue()

    def store_output(
        self,
        dst: ir.Buffer,
        src: ir.Buffer,
        orig_src: ir.Buffer | None = None,
        epilogue_nodes: list[ir.IRNode] | None = None,
        offsets: list[Any] | None = None,
        reindexers: list[Callable[[list[Any]], list[Any]] | None] | None = None,
    ):
        """
        Store the `src` buffer to the `dst` buffer. The size of `src` and `dst` should match.
        If `epilogue_nodes` is provided, the `src` buffer is firstly computed with the epilogues
````
- **EN**: Introduces function `store_output`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `dst`, `src`, `orig_src`, `epilogue_nodes`, `offsets`, and `reindexers`.
- **CN**: 这里定义了函数`store_output`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `dst`、`src`、`orig_src`、`epilogue_nodes`、`offsets`、`reindexers` 等值。

### Lines 381-400 / 第 381-400 行
````python
        before stored to `dst`. The `epilogues_nodes` are all pointwise.

        Notes:
        1. `src` and `dst` buffer could be the same buffer in which case we are doing in-place compute
           and stores. In case `epilogue_nodes` are not provided, we do nothing.
        2. The `epilogue_nodes`, if exist, have computations on `src` before storing to `dst` but since
           they come form the original Inductor IR, they might need to be adjusted before working with
           `src` and `dst` as outlined below:
           a) `src` or `dst` buffer could be a sub-slice of the ranges the `epilogue_nodes`work on.
              In this case, the `offsets` could be provided to adjust the indices passed to
              `epilogue_nodes` during codegen and the data ranges are also configured according to
              the sizes of `src` and `dst`.
           b) `dst` might be indexed in a different way as the `epilogue_nodes`, hence a `reindexer` is
              needed on the indices to `epilogue_nodes` to match the indexing of `dst`.
           c) If `src` is local, we need to add a local buffer for it and localize the `orig_src` buffer
              in `epilogue_nodes` with `src`.
        """
        assert isinstance(dst, (ir.Buffer, ir.ReinterpretView))
        assert dst.get_size() == src.get_size(), f"{dst=}, {src=}"
        if offsets:
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Notes`. This range continues the implementation of function `CppTemplateKernel.store_output`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `Notes` 等值。这一段延续了函数`CppTemplateKernel.store_output` 的具体实现。

### Lines 401-420 / 第 401-420 行
````python
            offsets = parse_expr_with_index_symbols(offsets)
        if epilogue_nodes:
            with LocalBufferContext(self.args) as scope:
                assert orig_src is not None
                if orig_src.get_name() != src.get_name():
                    scope.add_local_buffer(
                        src,
                        [
                            orig_src,
                        ],
                    )
                    epilogue_nodes = scope.localize_nodes(epilogue_nodes)
                return self.store_pointwise_nodes(
                    dst,
                    epilogue_nodes,  # type: ignore[arg-type]
                    offsets,
                    reindexers,
                )
        else:
            if dst.get_name() != src.get_name():
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `offsets`, `epilogue_nodes`, and `else`. This range continues the implementation of function `CppTemplateKernel.store_output`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `offsets`、`epilogue_nodes`、`else` 等值。这一段延续了函数`CppTemplateKernel.store_output` 的具体实现。

### Lines 421-440 / 第 421-440 行
````python
                # src is local
                copy = L.copy(dst, src).data.data
                with LocalBufferContext(self.args) as scope:
                    scope.add_local_buffer(src)

                    return self.store_pointwise_nodes(dst, [copy])
            else:
                assert dst.layout == src.layout, f"{dst=}, {src=}"
                return ""

    def store_outputs(
        self,
        dst: tuple[ir.Buffer],
        src: tuple[ir.IRNode],
        orig_src: tuple[ir.IRNode] | None = None,
        epilogue_nodes: list[ir.IRNode] | None = None,
        offsets: list[Any] | None = None,
        reindexers: list[Callable[[list[Any]], list[Any]] | None] | None = None,
        multi_output_buffers: tuple[ir.MultiOutput, ...] | None = None,
    ):
````
- **EN**: Introduces function `store_outputs`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `copy`, `else`, `dst`, `src`, `orig_src`, `epilogue_nodes`, and `...+3`.
- **CN**: 这里定义了函数`store_outputs`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `copy`、`else`、`dst`、`src`、`orig_src`、`epilogue_nodes`、`另有3项` 等值。

### Lines 441-460 / 第 441-460 行
````python
        assert isinstance(dst, Iterable)
        assert all(_dst.get_size() == _src.get_size() for _src, _dst in zip(src, dst))
        if offsets:
            offsets = parse_expr_with_index_symbols(offsets)
        gemm_num = len(src)
        final_offsets = []
        output_names = []
        if epilogue_nodes:
            if not reindexers:
                reindexers = [None] * len(epilogue_nodes)
            with LocalBufferContext(self.args) as scope:
                assert orig_src is not None
                localize_epilogue_nodes = []
                all_read_names = []
                for epilogue in epilogue_nodes:
                    all_read_names.extend(list(epilogue.get_read_names()))
                localize_epilogue_nodes.extend(scope.localize_nodes(epilogue_nodes))
                final_offsets.extend([offsets] * len(localize_epilogue_nodes))
                output_names.extend(
                    [node.get_name() for node in localize_epilogue_nodes]
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `offsets`, `gemm_num`, `final_offsets`, `output_names`, `reindexers`, `localize_epilogue_nodes`, and `...+1`. This range continues the implementation of function `CppTemplateKernel.store_outputs`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `offsets`、`gemm_num`、`final_offsets`、`output_names`、`reindexers`、`localize_epilogue_nodes`、`另有1项` 等值。这一段延续了函数`CppTemplateKernel.store_outputs` 的具体实现。

### Lines 461-480 / 第 461-480 行
````python
                )
                for gemm_idx in range(gemm_num):
                    if orig_src[gemm_idx].get_name() != src[gemm_idx].get_name():
                        if orig_src[gemm_idx].get_name() in all_read_names or (
                            multi_output_buffers
                            and multi_output_buffers[gemm_idx].get_name()
                            in all_read_names
                        ):
                            # If any of the Epilogue nodes use this GEMM output, let's localize the GEMM output
                            global_buffers = [orig_src[gemm_idx]]
                            if (
                                multi_output_buffers
                                and multi_output_buffers[gemm_idx].get_name()
                                in all_read_names
                                and orig_src[gemm_idx].get_name() not in all_read_names
                            ):
                                # Epilogue might directly read the MultiOutput, Locallize MultiOutput to the local Buffer
                                # if this MultiOutput has not been stored by in-template epilogue
                                # otherwise, use the cse store cache if it will be stored before used
                                global_buffers.append(multi_output_buffers[gemm_idx])
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `global_buffers`. This range continues the implementation of function `CppTemplateKernel.store_outputs`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含分支、循环或上下文管理等控制流。初始化或更新了 `global_buffers` 等值。这一段延续了函数`CppTemplateKernel.store_outputs` 的具体实现。

### Lines 481-500 / 第 481-500 行
````python
                            scope.add_local_buffer(
                                src[gemm_idx],
                                global_buffers,
                            )
                        else:
                            scope.add_local_buffer(src[gemm_idx])
                            localize_epilogue_nodes.extend(
                                [L.copy(dst[gemm_idx], src[gemm_idx]).data.data]
                            )
                            reindexers.append(None)
                            output_names.append(dst[gemm_idx].get_name())
                            final_offsets.append(
                                [sympy.S.Zero] * len(dst[gemm_idx].get_size())
                            )
                res = self.store_grouped_gemm_pointwise_nodes(
                    dst,
                    localize_epilogue_nodes,
                    final_offsets,
                    reindexers,
                    output_names=output_names,
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `res`, and `output_names`. This range continues the implementation of function `CppTemplateKernel.store_outputs`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`res`、`output_names` 等值。这一段延续了函数`CppTemplateKernel.store_outputs` 的具体实现。

### Lines 501-520 / 第 501-520 行
````python
                )
                for gemm_idx in range(gemm_num):
                    if (
                        multi_output_buffers
                        and multi_output_buffers[gemm_idx].get_name() in all_read_names
                    ):
                        # If the MultiOutput is used in the Epilogue, let's remove it from args
                        multi_output_name = multi_output_buffers[gemm_idx].get_name()
                        if (
                            multi_output_name in self.args.output_buffers
                            and self.args.output_buffers[multi_output_name]
                            is not REMOVED
                        ):
                            self.remove_buffer(multi_output_name)
                return res
        else:
            if dst[0].get_name() != src[0].get_name():
                copy_list = []
                with LocalBufferContext(self.args) as scope:
                    for _src, _dst in zip(src, dst):
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `multi_output_name`, `else`, and `copy_list`. This range continues the implementation of function `CppTemplateKernel.store_outputs`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `multi_output_name`、`else`、`copy_list` 等值。这一段延续了函数`CppTemplateKernel.store_outputs` 的具体实现。

### Lines 521-540 / 第 521-540 行
````python
                        copy_list.extend([L.copy(_dst, _src).data.data])
                        scope.add_local_buffer(_src)
                        output_names.append(_dst.get_name())
                        final_offsets.append([sympy.S.Zero] * len(_dst.get_size()))
                    reindexers = [None] * len(copy_list)
                    return self.store_grouped_gemm_pointwise_nodes(
                        dst,
                        nodes=copy_list,
                        offsets=final_offsets,
                        reindexers=reindexers,
                        output_names=output_names,
                    )
            else:
                assert all(
                    _src.get_name() == _dst.get_name() for _src, _dst in zip(src, dst)
                )
                assert all(
                    _src.get_layout() == _dst.get_layout()
                    for _src, _dst in zip(src, dst)
                )
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `reindexers`, `nodes`, `offsets`, `output_names`, and `else`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `reindexers`、`nodes`、`offsets`、`output_names`、`else` 等值。

### Lines 541-560 / 第 541-560 行
````python
                return ""

    def check_bounds(self, expr, size, lower, upper):
        # CppTemplateKernel does not need codegen related operations
        return


class CppTemplateCaller(ir.ChoiceCaller):
    """
    CppTemplateCaller

    This class represents a caller for CPP template kernels. It is a subclass of ir.ChoiceCaller.
    Attributes:
        name (str): The name of the caller.
        category (str): The category of the caller.
        bmreq (CppBenchmarkRequest): The benchmark request for the caller.
        template_buffer (ir.CppTemplateBuffer): The template buffer for the caller.
    """

    def __init__(
````
- **EN**: Introduces function `check_bounds`, class `CppTemplateCaller`, function `__init__`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `Attributes`.
- **CN**: 这里定义了函数`check_bounds`、类`CppTemplateCaller`、函数`__init__`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `Attributes` 等值。

### Lines 561-580 / 第 561-580 行
````python
        self,
        name: str,
        category: str,
        input_nodes: list[ir.Buffer],
        layout: ir.Layout,
        make_kernel_render: Callable[
            [
                ir.CppTemplateBuffer,
                bool,
                list[ir.IRNode] | None,
            ],
            str,
        ],
        bmreq: CppBenchmarkRequest,
        template: "CppTemplate",  # type: ignore[name-defined]  # noqa: F821
        info_kwargs: dict[str, ir.PrimitiveInfoType | list[ir.PrimitiveInfoType]]
        | None = None,
    ):
        super().__init__(name, input_nodes, layout, description="")
        self.category = category
````
- **EN**: Initializes or updates values such as `name`, `category`, `input_nodes`, `layout`, `make_kernel_render`, `bmreq`, and `...+2`. This range continues the implementation of function `CppTemplateCaller.__init__`.
- **CN**: 初始化或更新了 `name`、`category`、`input_nodes`、`layout`、`make_kernel_render`、`bmreq`、`另有2项` 等值。这一段延续了函数`CppTemplateCaller.__init__` 的具体实现。

### Lines 581-600 / 第 581-600 行
````python
        self.make_kernel_render = make_kernel_render
        self.bmreq = bmreq
        self.template = template
        self.info_kwargs = info_kwargs

    def precompile(self) -> None:
        assert self.bmreq is not None
        self.bmreq.precompile()

    def benchmark(self, *args, out) -> float:
        assert self.bmreq is not None
        if config.profile_bandwidth_with_do_bench_using_profiling:
            algo = self.bmreq.make_run_fn(*args, out=out)
            return do_bench_using_profiling(algo)
        return self.bmreq.benchmark(*args, out=out)

    def hash_key(self) -> str:
        return "-".join(
            [
                self.category,
````
- **EN**: Introduces function `precompile`, function `benchmark`, function `hash_key`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`precompile`、函数`benchmark`、函数`hash_key`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 601-620 / 第 601-620 行
````python
                self.bmreq.hash_key,
            ]
        )

    def info_dict(
        self,
    ) -> dict[str, ir.PrimitiveInfoType | list[ir.PrimitiveInfoType]]:
        return {"backend": "CPP", "op_type": "unknown"}

    def output_node(self) -> ir.TensorBox:
        buffer = ir.CppTemplateBuffer(
            layout=self.layout,
            inputs=self.input_nodes,
            make_kernel_render=self.make_kernel_render,
            template=self.template,
            choice=self,
        )
        # Pass KTC annotation to the buffer for encoding
        if "ktc" in self.annotations:
            buffer.annotations["ktc"] = self.annotations["ktc"]
````
- **EN**: Introduces function `info_dict`, function `output_node`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `buffer`, `layout`, `inputs`, `make_kernel_render`, `template`, and `choice`.
- **CN**: 这里定义了函数`info_dict`、函数`output_node`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `buffer`、`layout`、`inputs`、`make_kernel_render`、`template`、`choice` 等值。

### Lines 621-621 / 第 621-621 行
````python
        return ir.TensorBox.create(buffer)
````
- **EN**: Includes returns or checks that define the contract of this code path. This range continues the implementation of function `CppTemplateCaller.output_node`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。这一段延续了函数`CppTemplateCaller.output_node` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Profiling data handling  
  **CN**: 性能画像数据处理
- **EN**: Device capability modeling  
  **CN**: 设备能力建模

## Dependencies / 依赖关系
- **Standard library / 标准库**: `itertools`, `collections.abc`, `typing`, `unittest.mock`
- **Third-party / 第三方**: `sympy`, `sympy.parsing.sympy_parser`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor.utils`, `torch.utils._ordered_set`, `torch.utils._sympy.symbol`, `..`, `..autotune_process`, `..loop_body`, `..select_algorithm`, `..utils`, `..virtualized`, `.common`, `.cpp`, `.cpp_utils`
