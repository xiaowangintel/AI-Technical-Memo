# kernel.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/cutlass/kernel.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `LayoutArg`, `CUTLASSKernel`, `CUTLASSTemplateKernel`, and `CUTLASSTemplateCaller`. It exposes functions such as `_normalize_idx`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `LayoutArg`、`CUTLASSKernel`、`CUTLASSTemplateKernel`、`CUTLASSTemplateCaller` 等类。同时提供 `_normalize_idx` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import functools
import itertools
import logging
from collections import defaultdict
from collections.abc import Callable
from dataclasses import dataclass
from typing import Any, Literal, TYPE_CHECKING

from sympy import Expr, symbols

import torch._inductor.config as config
from torch import dtype as torch_dtype
from torch._inductor.codegen.common import get_device_op_overrides
from torch._inductor.codegen.cpp_wrapper_cpu import CppWrapperCpu
from torch._inductor.scheduler import BaseSchedulerNode
from torch._inductor.utils import do_bench_using_profiling, OrderedSet, Placeholder
from torch.utils._sympy.value_ranges import ValueRanges

from .utils import DTYPE_TO_CUTLASS_TYPE
````
- **EN**: Imports dependencies such as `functools`, `itertools`, `logging`, `collections`, `collections.abc`, `dataclasses`, and `...+10` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里导入了 `functools`、`itertools`、`logging`、`collections`、`collections.abc`、`dataclasses`、`另有10项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。借助符号表达式来推理索引、形状或代数性质。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 21-40 / 第 21-40 行
````python


if TYPE_CHECKING:
    from .template import ArgInfo

from ...autotune_process import CUTLASSBenchmarkRequest
from ...ir import (
    Buffer,
    ChoiceCaller,
    CUTLASSTemplateBuffer,
    IRNode,
    Layout,
    PrimitiveInfoType,
    TensorBox,
)
from ...utils import sympy_product
from ...virtualized import V
from ..common import (
    CSEVariable,
    IndentedBuffer,
````
- **EN**: Imports dependencies such as `.template`, `...autotune_process`, `...ir`, `...utils`, `...virtualized`, and `..common` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `.template`、`...autotune_process`、`...ir`、`...utils`、`...virtualized`、`..common` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。包含分支、循环或上下文管理等控制流。

### Lines 41-60 / 第 41-60 行
````python
    Kernel,
    OpOverrides,
    WorkspaceArg,
    WorkspaceZeroMode,
)
from ..cpp_utils import CppPrinter, DTYPE_TO_CPP


if TYPE_CHECKING:
    from torch._inductor.codegen.cutlass.template import CUTLASSTemplate

log = logging.getLogger(__name__)

cexpr = CppPrinter().doprint


def _normalize_idx(index: int, total_length: int) -> int:
    return index if index >= 0 else index + total_length


````
- **EN**: Imports dependencies such as `..cpp_utils`, and `torch._inductor.codegen.cutlass.template` for the logic in this range. Introduces function `_normalize_idx`. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `..cpp_utils`、`torch._inductor.codegen.cutlass.template` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_normalize_idx`。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 61-80 / 第 61-80 行
````python
ValidLayoutSymbols = Literal["M", "N", "K", "B", "lda", "ldb", "ldc", "ldd"]
ValidLayoutAttrs = Literal["size", "stride"]


@dataclass(frozen=True)
class LayoutArg:
    node: IRNode
    symbol: ValidLayoutSymbols
    attr: ValidLayoutAttrs
    dim: int

    def matches(self, node, attr, dim) -> bool:
        return self.node == node and self.attr == attr and self.dim == dim


class CUTLASSKernel(Kernel):
    """
    Baseclass for Cutlass based Kernels
    """

````
- **EN**: Introduces class `LayoutArg`, function `matches`, class `CUTLASSKernel`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`LayoutArg`、函数`matches`、类`CUTLASSKernel`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 81-100 / 第 81-100 行
````python
    overrides = OpOverrides  # type: ignore[assignment]

    def __init__(self, *args, **kwargs) -> None:
        super().__init__(*args, **kwargs)
        self.layout_args: dict[str, list[LayoutArg]] = defaultdict(list)
        self.size_args: list[Expr | int] = []
        # Mapping from arg name to IRNode.
        self.named_nodes: dict[str, IRNode] = {}

    def find_symbol(self, node: IRNode, attr: ValidLayoutAttrs, dim: int) -> str | None:
        arg = self.find_layout_arg(node, attr, dim)
        return arg.symbol if arg else None

    def find_layout_arg(
        self, node: IRNode, attr: ValidLayoutAttrs, dim: int
    ) -> LayoutArg | None:
        matches = [
            arg
            for arg in itertools.chain.from_iterable(self.layout_args.values())
            if arg.matches(node, attr, dim)
````
- **EN**: Introduces function `__init__`, function `find_symbol`, function `find_layout_arg`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`__init__`、函数`find_symbol`、函数`find_layout_arg`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 101-120 / 第 101-120 行
````python
        ]
        if len(matches) >= 1:
            # Verify all matches have the same node, attribute, and dimension
            # And if they come from the same node, whichever symbol we use is fine.
            # if in runtime the logic changes, this would trigger guard
            first_match = matches[0]
            if not all(
                match.node == first_match.node
                and match.attr == first_match.attr
                and match.dim == first_match.dim
                for match in matches
            ):
                raise AssertionError("All matching layout args should be identical")
            return first_match
        attr_values = node.get_size() if attr == "size" else node.get_stride()
        if dim >= len(attr_values):
            return None
        expr = attr_values[dim]
        fallback_matches = []
        for arg in itertools.chain.from_iterable(self.layout_args.values()):
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `first_match`, `attr_values`, `expr`, and `fallback_matches`. This range continues the implementation of function `CUTLASSKernel.find_layout_arg`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `first_match`、`attr_values`、`expr`、`fallback_matches` 等值。这一段延续了函数`CUTLASSKernel.find_layout_arg` 的具体实现。

### Lines 121-140 / 第 121-140 行
````python
            if arg.attr != attr:
                continue
            if arg.node.get_name() != node.get_name():
                continue
            arg_values = (
                arg.node.get_size() if arg.attr == "size" else arg.node.get_stride()
            )
            if arg.dim >= len(arg_values):
                continue
            if arg_values[arg.dim] == expr:
                fallback_matches.append(arg)
        if fallback_matches:
            return fallback_matches[0]
        return None

    def add_layout_arg(
        self, symbol: ValidLayoutSymbols, node: IRNode, attr: ValidLayoutAttrs, dim: int
    ):
        arg = LayoutArg(node, symbol, attr, dim)
        self.layout_args[symbol].append(arg)
````
- **EN**: Introduces function `add_layout_arg`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `arg_values`, and `arg`.
- **CN**: 这里定义了函数`add_layout_arg`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `arg_values`、`arg` 等值。

### Lines 141-160 / 第 141-160 行
````python

    def init_layout_args(self) -> None:
        X = self.named_nodes["X"]
        W = self.named_nodes["W"]
        Y = self.named_nodes["Y"]
        Bias = self.named_nodes.get("Bias", None)
        x_mdim = _normalize_idx(-2, len(X.get_size()))
        x_kdim = _normalize_idx(-1, len(X.get_size()))
        w_kdim = _normalize_idx(-2, len(W.get_size()))
        w_ndim = _normalize_idx(-1, len(W.get_size()))
        y_mdim = _normalize_idx(-2, len(Y.get_size()))
        y_ndim = _normalize_idx(-1, len(Y.get_size()))
        self.add_layout_arg("M", X, "size", x_mdim)
        self.add_layout_arg("K", X, "size", x_kdim)
        self.add_layout_arg("K", W, "size", w_kdim)
        self.add_layout_arg("N", W, "size", w_ndim)
        self.add_layout_arg("M", Y, "size", y_mdim)
        self.add_layout_arg("N", Y, "size", y_ndim)
        if len(X.get_size()) > 2:
            self.add_layout_arg("B", X, "size", 0)
````
- **EN**: Introduces function `init_layout_args`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `X`, `W`, `Y`, `Bias`, `x_mdim`, `x_kdim`, and `...+4`.
- **CN**: 这里定义了函数`init_layout_args`。包含分支、循环或上下文管理等控制流。初始化或更新了 `X`、`W`、`Y`、`Bias`、`x_mdim`、`x_kdim`、`另有4项` 等值。

### Lines 161-180 / 第 161-180 行
````python

        lda_dim = self.find_ld_idx(X)
        ldb_dim = self.find_ld_idx(W)
        ldc_dim = self.find_ld_idx(Bias) if Bias else None
        ldd_dim = self.find_ld_idx(Y)
        self.add_layout_arg("lda", X, "stride", lda_dim)
        self.add_layout_arg("ldb", W, "stride", ldb_dim)
        if Bias is not None and ldc_dim is not None:
            self.add_layout_arg("ldc", Bias, "stride", ldc_dim)
        self.add_layout_arg("ldd", Y, "stride", ldd_dim)

    def get_layout_args(self) -> tuple[Expr | int, ...]:
        X = self.named_nodes["X"]
        W = self.named_nodes["W"]
        Y = self.named_nodes["Y"]
        Bias = self.named_nodes.get("Bias", None)
        mdim = _normalize_idx(-2, len(X.get_size()))
        ndim = _normalize_idx(-1, len(W.get_size()))
        kdim = _normalize_idx(-1, len(X.get_size()))

````
- **EN**: Introduces function `get_layout_args`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `lda_dim`, `ldb_dim`, `ldc_dim`, `ldd_dim`, `X`, `W`, and `...+5`.
- **CN**: 这里定义了函数`get_layout_args`。借助符号表达式来推理索引、形状或代数性质。包含分支、循环或上下文管理等控制流。初始化或更新了 `lda_dim`、`ldb_dim`、`ldc_dim`、`ldd_dim`、`X`、`W`、`另有5项` 等值。

### Lines 181-200 / 第 181-200 行
````python
        def get_ld(node) -> Expr | int:
            dim = self.find_ld_idx(node)
            return node.get_stride()[dim]

        M = X.get_size()[mdim]
        N = W.get_size()[ndim]
        K = X.get_size()[kdim]
        B = X.get_size()[0] if len(X.get_size()) > 2 else 1
        LDA = get_ld(X)
        LDB = get_ld(W)
        LDC = get_ld(Bias) if Bias else 0
        LDD = get_ld(Y)
        return (M, N, K, B, LDA, LDB, LDC, LDD)

    def get_dynamic_shape_args(self) -> list[Expr | int]:
        return [*self.get_layout_args(), *self.size_args]

    def get_offset_args(self) -> list[Expr]:
        return [node.get_layout().offset for node in self.named_nodes.values()]

````
- **EN**: Introduces function `get_ld`, function `get_dynamic_shape_args`, function `get_offset_args`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `dim`, `M`, `N`, `K`, `B`, `LDA`, and `...+3`.
- **CN**: 这里定义了函数`get_ld`、函数`get_dynamic_shape_args`、函数`get_offset_args`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `dim`、`M`、`N`、`K`、`B`、`LDA`、`另有3项` 等值。

### Lines 201-220 / 第 201-220 行
````python
    @staticmethod
    def find_ld_idx(node: IRNode) -> int:
        strides = node.get_stride()
        # Handle 1D tensor case
        if V.graph.sizevars.statically_known_equals(strides[-1], 1):
            return _normalize_idx(-2, len(strides))

        assert V.graph.sizevars.statically_known_equals(strides[-2], 1), strides[-2]
        return _normalize_idx(-1, len(strides))


class CUTLASSTemplateKernel(CUTLASSKernel):
    """
    Template kernels defined by Cutlass in C++.
    """

    def __init__(
        self,
        kernel_name: str,
        runtime_arg_info: list["ArgInfo"],
````
- **EN**: Introduces function `find_ld_idx`, class `CUTLASSTemplateKernel`, function `__init__`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`find_ld_idx`、类`CUTLASSTemplateKernel`、函数`__init__`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 221-240 / 第 221-240 行
````python
        runtime_arg_values: list[Any],
        device_type: str = "cuda",  # type: ignore[assignment]
    ) -> None:
        """
        Initializes a new instance of the CUTLASSTemplateKernel class.

        Args:
            kernel_name (str): The name of the kernel.
        """
        super().__init__()
        self.kernel_name = kernel_name
        self.runtime_arg_info = runtime_arg_info
        self.runtime_arg_values = runtime_arg_values
        self.device_type = device_type
        self.device_codegen = get_device_op_overrides(self.device_type)
        self._EXTRA_CPP_ARGS = f"size_t* workspace_size, uint8_t* workspace, {self.device_codegen.cpp_stream_type()} stream"

    def check_not_null(self, node: IRNode) -> str:
        """
        Generates code to check that a node is not null.
````
- **EN**: Introduces function `check_not_null`. Handles device-specific behavior, capabilities, or dispatch decisions. Initializes or updates values such as `runtime_arg_values`, `device_type`, and `Args`.
- **CN**: 这里定义了函数`check_not_null`。处理与设备相关的行为、能力信息或派发决策。初始化或更新了 `runtime_arg_values`、`device_type`、`Args` 等值。

### Lines 241-260 / 第 241-260 行
````python
        """
        if node is None:
            return ""

        size_str = self.size(node, 0, -1)
        name_str = self.arg_name(node)
        if name_str is None:
            return ""

        res = IndentedBuffer(initial_indent=2)
        res.tabwidth = 1
        res.splice(
            f"""
            {{
              if (!{name_str}) {{
                int64_t {name_str}_size = {size_str};
                if ({name_str}_size > 0) {{
                  throw std::runtime_error("input {name_str} is null but size is not 0!");
                }}
              }}
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `size_str`, `name_str`, and `res`. This range continues the implementation of function `CUTLASSTemplateKernel.check_not_null`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `size_str`、`name_str`、`res` 等值。这一段延续了函数`CUTLASSTemplateKernel.check_not_null` 的具体实现。

### Lines 261-280 / 第 261-280 行
````python
            }}
            """
        )
        return res.getvalue()

    def get_signature(self) -> str:
        return self.signature

    def _collect_unbound_layout_free_symbols(self, node: IRNode) -> OrderedSet[Expr]:
        free_symbols: OrderedSet[Expr] = OrderedSet()
        for attr_name, values in (
            ("size", node.get_size()),
            ("stride", node.get_stride()),
        ):
            attr = attr_name  # help mypy narrow the Literal argument below
            for dim, expr in enumerate(values):
                if not isinstance(expr, Expr):
                    continue
                if self.find_layout_arg(node, attr, dim) is not None:
                    continue
````
- **EN**: Introduces function `get_signature`, function `_collect_unbound_layout_free_symbols`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_signature`、函数`_collect_unbound_layout_free_symbols`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 281-300 / 第 281-300 行
````python
                for symbol in expr.free_symbols:
                    free_symbols.add(symbol)  # type: ignore[arg-type]
        return free_symbols

    def def_kernel(
        self,
        inputs: list[IRNode],
        outputs: list[IRNode],
        names_str: str = "",
        input_reorder: list[int] | None = None,
    ) -> str:
        """
        Hook called from template code to generate function definition and
        needed args.

        Args:
            inputs: List of input IRNodes
            outputs: List of output IRNodes
            names_str: Comma separated list of input + output argument names.
            input_reorder: The actual order of input nodes.
````
- **EN**: Introduces function `def_kernel`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `inputs`, `outputs`, `names_str`, `input_reorder`, and `Args`.
- **CN**: 这里定义了函数`def_kernel`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `inputs`、`outputs`、`names_str`、`input_reorder`、`Args` 等值。

### Lines 301-320 / 第 301-320 行
````python
                           e.g. The template might have input argument defined as [X, W, Bias],
                           and the actual input passed into this template could be [Bias, X, W].
                           In this case, the `input_reorder` would be [2, 0, 1].
            additional_size_args: Additional size arguments for epilogue inputs
        """
        # NB: name order matters here, it's used to match up offsets
        names = [x.strip() for x in names_str.strip().split(",")]
        if len(inputs) + len(outputs) != len(names):
            raise RuntimeError(
                f"{len(inputs) + len(outputs)=} != {len(names)=}, {inputs=}, {outputs=}, {names=}"
            )

        if input_reorder is not None:
            assert len(inputs) == len(input_reorder)
        else:
            input_reorder = list(range(len(inputs)))

        for idx in input_reorder:
            name = names[idx]
            node = inputs[idx]
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `additional_size_args`, `names`, `else`, `input_reorder`, `name`, and `node`. This range continues the implementation of function `CUTLASSTemplateKernel.def_kernel`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `additional_size_args`、`names`、`else`、`input_reorder`、`name`、`node` 等值。这一段延续了函数`CUTLASSTemplateKernel.def_kernel` 的具体实现。

### Lines 321-340 / 第 321-340 行
````python
            if node is not None:
                self.named_nodes[name] = node
                self.args.input_buffers[node.get_name()] = name

        for name, node in zip(names[len(inputs) : len(inputs) + len(outputs)], outputs):
            if node is not None:
                # NB: named nodes must be populated in the order of names
                self.named_nodes[name] = node
                self.args.output_buffers[node.get_name()] = name

        arg_defs, *_ = self.args.cpp_argdefs(DTYPE_TO_CUTLASS_TYPE)

        self.init_layout_args()
        free_symbols: OrderedSet[Expr] = OrderedSet()
        for node in self.named_nodes.values():
            free_symbols |= self._collect_unbound_layout_free_symbols(node)
        size_vars = ["M", "N", "K", "B", "lda", "ldb", "ldc", "ldd"]
        size_vars.extend(str(s) for s in free_symbols)
        self.size_args.extend(free_symbols)
        size_args = [f"const int {s}" for s in size_vars]
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `free_symbols`, `size_vars`, and `size_args`. This range continues the implementation of function `CUTLASSTemplateKernel.def_kernel`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含分支、循环或上下文管理等控制流。初始化或更新了 `free_symbols`、`size_vars`、`size_args` 等值。这一段延续了函数`CUTLASSTemplateKernel.def_kernel` 的具体实现。

### Lines 341-360 / 第 341-360 行
````python
        offset_args = [f"const int {name}_offset" for name in self.named_nodes]
        runtime_arg_decls = ",".join(
            [f"{arg.ty} {arg.name}" for arg in self.runtime_arg_info]
        )
        if runtime_arg_decls:
            runtime_arg_decls += ", "

        signature = (
            f"int {self.kernel_name}({', '.join(arg_defs + size_args + offset_args)},\
 {runtime_arg_decls}{self._EXTRA_CPP_ARGS})"
        )
        self.signature = signature
        return signature

    def call_kernel(
        self,
        name: str,
        node: "CUTLASSTemplateBuffer",  # type: ignore[name-defined]
    ) -> None:
        """
````
- **EN**: Introduces function `call_kernel`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `offset_args`, `runtime_arg_decls`, `signature`, `name`, and `node`.
- **CN**: 这里定义了函数`call_kernel`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `offset_args`、`runtime_arg_decls`、`signature`、`name`、`node` 等值。

### Lines 361-380 / 第 361-380 行
````python
        Generates code to call the kernel through V.graph.wrapper_code.
        used from within torch._inductor.wrapper.PythonWrapperCodegen

        name: Name of kernel function.
        node: The CUTLASSTemplateBuffer node which contains information about the kernel, it's fused epilogue nodes
        as well as all required inputs and outputs.
        """
        wrapper = V.graph.wrapper_code

        arg_types: list[Any]
        if V.graph.cpp_wrapper:
            # Make sure we initialize these kernels since they're exported as
            # C-style symbol names.
            assert isinstance(wrapper, CppWrapperCpu)
            wrapper.initialized_kernels[name] = self
            # We always originally initialize name with "KERNEL_NAME". So, we
            # we replace with the real kernel name passed as an arg to this function.
            self.signature = self.signature.replace(str(Placeholder.KERNEL_NAME), name)
            _, call_args, arg_types = self.args.cpp_argdefs(DTYPE_TO_CUTLASS_TYPE)
        else:
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `name`, `node`, `wrapper`, `arg_types`, and `else`. This range continues the implementation of function `CUTLASSTemplateKernel.call_kernel`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `name`、`node`、`wrapper`、`arg_types`、`else` 等值。这一段延续了函数`CUTLASSTemplateKernel.call_kernel` 的具体实现。

### Lines 381-400 / 第 381-400 行
````python
            _, call_args, _, arg_types = self.args.python_argdefs()

        dynamic_shape_args = self.get_dynamic_shape_args()
        offset_args = self.get_offset_args()
        call_args.extend(dynamic_shape_args)  # type: ignore[arg-type]
        call_args.extend(offset_args)  # type: ignore[arg-type]
        for arg in self.runtime_arg_values:
            call_args.append(str(arg))
        arg_types.extend("const int" for _ in dynamic_shape_args)
        arg_types.extend("const int" for _ in offset_args)
        for arg in self.runtime_arg_info:
            arg_types.append(arg.ty)
        # dynamo wraps unspec variable as 0d CPU tensor, need convert to scalar
        for i in range(len(call_args)):
            if V.graph.is_unspec_arg(call_args[i]):
                call_args[i] = call_args[i] + ".item()"
            elif isinstance(arg_types[i], torch_dtype):
                call_args[i] = (
                    call_args[i]
                    if V.graph.cpp_wrapper
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `dynamic_shape_args`, and `offset_args`. This range continues the implementation of function `CUTLASSTemplateKernel.call_kernel`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `dynamic_shape_args`、`offset_args` 等值。这一段延续了函数`CUTLASSTemplateKernel.call_kernel` 的具体实现。

### Lines 401-420 / 第 401-420 行
````python
                    else f"c_void_p({call_args[i]}.data_ptr())"
                )

        # workspace_size ptr is NULL to mark this call is not intended for retrieving workspace_size.
        # workspace_size should have already been retrieved prior to this call.
        # workspace_size is here.
        call_args.append("nullptr" if V.graph.cpp_wrapper else "None")
        if V.graph.cpp_wrapper:
            arg_types.append("size_t*")

        if node.get_workspace_size() > 0:
            ws = WorkspaceArg(
                count=node.get_workspace_size(),
                device=V.graph.get_current_device_or_throw(),
                zero_mode=WorkspaceZeroMode.UNINITIALIZED,
                outer_name=WorkspaceArg.unique_name(),
            )
            wrapper.generate_workspace_allocation(ws)
            workspace = str(ws.outer_name)
            call_args.append(
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `ws`, `count`, `device`, `zero_mode`, `outer_name`, and `workspace`. This range continues the implementation of function `CUTLASSTemplateKernel.call_kernel`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `ws`、`count`、`device`、`zero_mode`、`outer_name`、`workspace` 等值。这一段延续了函数`CUTLASSTemplateKernel.call_kernel` 的具体实现。

### Lines 421-440 / 第 421-440 行
````python
                workspace
                if V.graph.cpp_wrapper
                else f"c_void_p({workspace}.data_ptr())"
            )
        else:
            ws = None
            call_args.append("nullptr" if V.graph.cpp_wrapper else "None")
        if V.graph.cpp_wrapper:
            arg_types.append("uint8_t*")

        wrapper.generate_kernel_call(
            name,
            call_args,
            triton=False,
            arg_types=arg_types,
        )
        if ws:
            wrapper.generate_workspace_deallocation(ws)

    def dtype(self, node: IRNode) -> str | None:
````
- **EN**: Introduces function `dtype`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`dtype`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。

### Lines 441-460 / 第 441-460 行
````python
        """
        Generates code which represents dtype of a given node.
        """

        if node is None:
            return "void"
        return DTYPE_TO_CPP.get(node.get_layout().dtype)

    def cutlass_dtype(self, node: IRNode, default_dtype="void") -> str | None:
        # Helper method, called into from CUTLASSGemmTemplate
        if node is None:
            return default_dtype
        from torch._inductor.codegen.cutlass.template import CUTLASSTemplate

        return CUTLASSTemplate._DTYPE_TO_CUTLASS[node.get_layout().dtype]

    def max_valid_index(self, node: IRNode, default=-1):
        # Helper method, called into from CUTLASSGemmTemplate
        if node is None:
            return default
````
- **EN**: Imports dependencies such as `torch._inductor.codegen.cutlass.template` for the logic in this range. Introduces function `cutlass_dtype`, function `max_valid_index`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `torch._inductor.codegen.cutlass.template` 等依赖，为后续逻辑提供基础能力。这里定义了函数`cutlass_dtype`、函数`max_valid_index`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 461-480 / 第 461-480 行
````python
        max_valid_offset = 0
        for i in range(len(node.get_size())):
            max_valid_offset += (node.get_size()[i] - 1) * node.get_stride()[i]
        return max_valid_offset

    def ptr(self, node: IRNode) -> str:
        """
        Generates code which represents pointer of a given node.
        """

        if node is None:
            return "nullptr"
        arg_name = self.arg_name(node)
        if arg_name is None:
            return "nullptr"
        return f"{arg_name} + {arg_name}_offset"

    def size(
        self,
        node: IRNode,
````
- **EN**: Introduces function `ptr`, function `size`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `max_valid_offset`, `arg_name`, and `node`.
- **CN**: 这里定义了函数`ptr`、函数`size`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `max_valid_offset`、`arg_name`、`node` 等值。

### Lines 481-500 / 第 481-500 行
````python
        start_index: int,
        end_index: int | None = None,
        default_value: int = 0,
    ) -> str:
        """
        Hook called from template code to get the size of an arg.
        Generates code which represents size of a given node in [start_index, end_index).
        If node is None, returns default_value.

        TODO: Will add needed args to pass it in if it is dynamic.
        """

        if node is None:
            return str(default_value)

        start_index = _normalize_idx(start_index, len(node.get_size()))
        if end_index is None:
            end_index = start_index
        end_index = _normalize_idx(end_index, len(node.get_size()))
        sizes = [
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `start_index`, `end_index`, `default_value`, `TODO`, and `sizes`. This range continues the implementation of function `CUTLASSTemplateKernel.size`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `start_index`、`end_index`、`default_value`、`TODO`、`sizes` 等值。这一段延续了函数`CUTLASSTemplateKernel.size` 的具体实现。

### Lines 501-520 / 第 501-520 行
````python
            self.find_symbol(node, "size", dim=i) or node.get_size()[i]
            for i in range(start_index, end_index + 1)
        ]
        if len(sizes) == 0:
            return str(default_value)

        sizes = [symbols(v) if isinstance(v, str) else v for v in sizes]
        val = sympy_product(sizes)
        return val

    def stride(self, node: IRNode, index: int, default_value: int = 0) -> str:
        """
        Hook called from template code to get the stride of an arg.
        Generates code which represents stride of a given node at index.
        If node is None, returns default_value.

        TODO: Will add needed args to pass it in if it is dynamic.
        """

        if node is None:
````
- **EN**: Introduces function `stride`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `sizes`, `val`, and `TODO`.
- **CN**: 这里定义了函数`stride`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `sizes`、`val`、`TODO` 等值。

### Lines 521-540 / 第 521-540 行
````python
            return str(default_value)

        index = _normalize_idx(index, len(node.get_size()))
        if index < 0:
            return str(default_value)

        stride = node.get_stride()[index]
        if V.graph.sizevars.statically_known_leq(stride, 1):
            return str(stride)
        return self.find_symbol(node, "stride", dim=index) or str(stride)

    def batch_stride(self, node: IRNode, default_value: int = 0) -> str:
        """
        Hook called from template code to get the batch stride of an arg.
        Returns 0 if batch dim is not present.

        This method assumes that batch stride is the largest stride.
        """

        if node is None:
````
- **EN**: Introduces function `batch_stride`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `index`, and `stride`.
- **CN**: 这里定义了函数`batch_stride`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `index`、`stride` 等值。

### Lines 541-560 / 第 541-560 行
````python
            return str(default_value)

        if len(node.get_size()) < 3:
            return str(default_value)

        batch_stride = node.get_stride()[0]
        if V.graph.sizevars.statically_known_leq(batch_stride, 1):
            return str(batch_stride)

        return "{}*{}".format(
            self.find_symbol(node, "size", dim=1) or node.get_size()[1],
            self.find_symbol(node, "size", dim=2) or node.get_size()[2],
        )

    def row_or_column_stride(self, node: IRNode, default_value: int = 0) -> str:
        """
        Hook called from template code to get the row or column stride of an arg.
        This is required by some CUTLASS 2.X APIs.
        If the node is in row_major, it returns stride[-2].
        If the node is in column_major, it returns stride[-1].
````
- **EN**: Introduces function `row_or_column_stride`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `batch_stride`.
- **CN**: 这里定义了函数`row_or_column_stride`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `batch_stride` 等值。

### Lines 561-580 / 第 561-580 行
````python

        TODO: Will add needed args to pass it in if it is dynamic.
        """

        if node is None or len(node.get_stride()) < 2:
            return str(default_value)

        stride0 = node.get_stride()[-1]
        stride1 = node.get_stride()[-2]
        if stride0 == 1:
            return cexpr(self.rename_indexing(stride1))
        elif stride1 == 1:
            return cexpr(self.rename_indexing(stride0))
        else:
            raise RuntimeError(
                f"At least 1 stride should be 1. Strides: {node.get_stride()=}"
            )

    def load(self, name: str, index: Expr, mode: Any = None) -> CSEVariable:
        """
````
- **EN**: Introduces function `load`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`load`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 581-600 / 第 581-600 行
````python
        Mock load function for memory planning to optimize allocations properly.
        """
        return self.create_cse_var(name, bounds=ValueRanges.unknown())

    def store(self, name: str, index: Expr, value: Any, mode: Any = None) -> None:
        """
        Mock store function for memory planning to optimize allocations properly.
        """
        self.store_buffer_names.add(name)


class CUTLASSTemplateCaller(ChoiceCaller):
    """
    CUTLASSTemplateCaller

    This class represents a caller for CUTLASS template kernels. It is a subclass of ChoiceCaller.
    Attributes:
        name (str): The name of the caller.
        category (str): The category of the caller.
        bmreq (CUTLASSBenchmarkRequest): The benchmark request for the caller.
````
- **EN**: Introduces function `store`, class `CUTLASSTemplateCaller`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Processes profiling, FLOP, or benchmark information to characterize execution cost. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`store`、类`CUTLASSTemplateCaller`。借助符号表达式来推理索引、形状或代数性质。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 601-620 / 第 601-620 行
````python
        template_buffer (CUTLASSTemplateBuffer): The template buffer for the caller.
    """

    def __init__(
        self,
        name: str,
        category: str,
        input_nodes: list[Buffer],
        layout: Layout,
        make_kernel_render: Callable[
            [CUTLASSTemplateBuffer, list[BaseSchedulerNode] | None],
            tuple[CUTLASSTemplateKernel, functools.partial[str]],
        ],
        bmreq: CUTLASSBenchmarkRequest,
        supports_epilogue_fusion: bool,
        template: "CUTLASSTemplate",  # type: ignore[name-defined]
        info_kwargs: dict[str, PrimitiveInfoType | list[PrimitiveInfoType]] | None,  # type: ignore[type-arg]
        description: str,
    ) -> None:
        super().__init__(name, input_nodes, layout, description)
````
- **EN**: Introduces function `__init__`. Initializes or updates values such as `name`, `category`, `input_nodes`, `layout`, `make_kernel_render`, `bmreq`, and `...+4`.
- **CN**: 这里定义了函数`__init__`。初始化或更新了 `name`、`category`、`input_nodes`、`layout`、`make_kernel_render`、`bmreq`、`另有4项` 等值。

### Lines 621-640 / 第 621-640 行
````python
        self.category = category
        self.make_kernel_render = make_kernel_render
        self.bmreq = bmreq
        self.supports_epilogue_fusion = supports_epilogue_fusion
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

    def __str__(self) -> str:
        return f"CUTLASSTemplateCaller(source_file={self.bmreq.source_file})"
````
- **EN**: Introduces function `precompile`, function `benchmark`, function `__str__`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`precompile`、函数`benchmark`、函数`__str__`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 641-660 / 第 641-660 行
````python

    def call_name(self) -> str:
        return f"cutlass_template_kernels.{self.name}"

    def kernel_hash_key(self) -> str:
        """
        Return kernel hash key that does not depend on swizzle.
        """
        return "-".join(
            [
                self.category,
                self.bmreq.hash_key,
            ]
        )

    def hash_key(self) -> str:
        """
        Return kernel hash key that does not depend on swizzle.
        """
        swizzle_str: str = (
````
- **EN**: Introduces function `call_name`, function `kernel_hash_key`, function `hash_key`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `swizzle_str`.
- **CN**: 这里定义了函数`call_name`、函数`kernel_hash_key`、函数`hash_key`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `swizzle_str` 等值。

### Lines 661-680 / 第 661-680 行
````python
            str(self.info_kwargs.get("swizzle"))
            if isinstance(self.info_kwargs, dict)
            else "None"
        )
        return "-".join(
            [
                self.category,
                self.bmreq.hash_key,
                swizzle_str,
            ]
        )

    def info_dict(self) -> dict[str, PrimitiveInfoType | list[PrimitiveInfoType]]:
        """
        Information returned here is logged to the autotune log file when that is enabled.

        In general, we should avoid calling this function as it is expensive to compute,
        and can add up very fast.
        """
        if self.info_kwargs is not None and "op" in self.info_kwargs:
````
- **EN**: Introduces function `info_dict`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`info_dict`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 681-700 / 第 681-700 行
````python
            op: Any = self.info_kwargs["op"]
            return {
                "backend": "CUDA",
                "op_type": type(op).__name__,
                "op_conf_name": str(op.configuration_name()),
                "op_arch": str(op.arch),
                "tile_shape": str(op.tile_description.tile_shape),
                "epilogue_schedule": str(op.epilogue_schedule),
                "kernel_schedule": str(op.kernel_schedule),
                "element_accumulator": str(op.accumulator_type()),
                "op_name": str(op.procedural_name()),
                "instruction_shape": str(
                    op.tile_description.math_instruction.instruction_shape
                ),
                "swizzle": str(self.info_kwargs["swizzle"]),
            }
        else:
            return {"backend": "CUDA", "op_type": "unknown"}

    def output_node(self) -> TensorBox:
````
- **EN**: Introduces function `output_node`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`output_node`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 701-713 / 第 701-713 行
````python
        self.bmreq.update_workspace_size()
        buffer = CUTLASSTemplateBuffer(
            layout=self.layout,
            inputs=self.input_nodes,
            make_kernel_render=self.make_kernel_render,
            workspace_size=self.bmreq.workspace_size,
            supports_epilogue_fusion=self.supports_epilogue_fusion,
            template=self.template,
        )
        # Pass KTC annotation to the buffer for encoding
        if "ktc" in self.annotations:
            buffer.annotations["ktc"] = self.annotations["ktc"]
        return TensorBox.create(buffer)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `buffer`, `layout`, `inputs`, `make_kernel_render`, `workspace_size`, `supports_epilogue_fusion`, and `...+1`. This range continues the implementation of function `CUTLASSTemplateCaller.output_node`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `buffer`、`layout`、`inputs`、`make_kernel_render`、`workspace_size`、`supports_epilogue_fusion`、`另有1项` 等值。这一段延续了函数`CUTLASSTemplateCaller.output_node` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Scheduling decisions  
  **CN**: 调度决策
- **EN**: Profiling data handling  
  **CN**: 性能画像数据处理
- **EN**: Device capability modeling  
  **CN**: 设备能力建模

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`, `itertools`, `logging`, `collections`, `collections.abc`, `dataclasses`, `typing`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch._inductor.config`, `torch`, `torch._inductor.codegen.common`, `torch._inductor.codegen.cpp_wrapper_cpu`, `torch._inductor.scheduler`, `torch._inductor.utils`, `torch.utils._sympy.value_ranges`, `.utils`, `...autotune_process`, `...ir`, `...utils`, `...virtualized`, `..common`, `..cpp_utils`, `.template`, `torch._inductor.codegen.cutlass.template`
