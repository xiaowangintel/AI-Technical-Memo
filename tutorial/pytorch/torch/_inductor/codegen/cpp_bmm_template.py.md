# cpp_bmm_template.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/cpp_bmm_template.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `CppBmmTemplate`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `CppBmmTemplate` 等类。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import contextlib
import itertools
from collections.abc import Callable
from typing import Any
from unittest.mock import patch

import sympy

from .. import ir
from ..select_algorithm import PartialRender
from ..virtualized import V
from .common import ArgName
from .cpp_gemm_template import CppGemmTemplate, GEMM_TEMPLATE
from .cpp_micro_gemm import LayoutType
from .cpp_template_kernel import CppTemplateKernel
from .cpp_utils import DTYPE_TO_CPP, GemmBlocking


# We pass all sizevars present in BY to the GEMM templates so variables are not renamed in the BMM definition
````
- **EN**: Imports dependencies such as `contextlib`, `itertools`, `collections.abc`, `typing`, `unittest.mock`, `sympy`, and `...+8` for the logic in this range. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里导入了 `contextlib`、`itertools`、`collections.abc`、`typing`、`unittest.mock`、`sympy`、`另有8项` 等依赖，为后续逻辑提供基础能力。借助符号表达式来推理索引、形状或代数性质。

### Lines 21-40 / 第 21-40 行
````python
GEMM_SINGLE_THREAD_MM_STUB = r"""
{{kernel.def_kernel(
    inputs={"X": X, "W": W},
    outputs={"Y": Y_2d},
    aliases=aliases,
    function_name=kernel_name+"_single_thread_mm",
    extra_sizevars=BY_sizevars + [b_index],
    placeholder="<SINGLE_THREAD_MM_DEF_FOR_BMM>")}}"""

GEMM_THREADED_MM_STUB = r"""
{{kernel.def_kernel(
    inputs={"X": X, "W": W},
    outputs={"Y": Y_2d},
    aliases=aliases,
    function_name=kernel_name+"_threaded_mm",
    extra_sizevars=BY_sizevars + [b_index],
    placeholder="<THREADED_MM_DEF_FOR_BMM>")}}"""

BMM_TEMPLATE = r"""
{{ template.codegen_microkernel_def() }}
````
- **EN**: Initializes or updates values such as `GEMM_SINGLE_THREAD_MM_STUB`, `inputs`, `outputs`, `aliases`, `function_name`, `extra_sizevars`, and `...+3`.
- **CN**: 初始化或更新了 `GEMM_SINGLE_THREAD_MM_STUB`、`inputs`、`outputs`、`aliases`、`function_name`、`extra_sizevars`、`另有3项` 等值。

### Lines 41-60 / 第 41-60 行
````python
{{ template.codegen_single_thread_gemm() }}
{{ template.codegen_multi_thread_gemm() }}

extern "C"
{{kernel.def_kernel(inputs={"X": BX, "W": BW}, outputs={"Y": BY}, aliases=aliases)}}
{
    const int64_t B = {{kernel.size(BY_2d, 0)}};
    {%- if num_threads > 1 %}
    constexpr int64_t num_threads = {{num_threads}};
    int64_t B_single_thread_block = (B / num_threads) * num_threads;

    {%- set use_dynamic_threads = ((config.cpp.threads < 1) and (num_threads == cpu_count)) or config.cpp.dynamic_threads %}
    {%- if use_dynamic_threads %}
    #pragma omp parallel for
    {%- else %}
    #pragma omp parallel for num_threads({{num_threads}})
    {%- endif %}
    {%- else %}
    int64_t B_single_thread_block = B;
    {%- endif %}
````
- **EN**: Implements the local expressions and calls needed for this part of the module.
- **CN**: 实现了该模块这一部分所需的局部表达式与调用逻辑。

### Lines 61-80 / 第 61-80 行
````python
    for (int64_t b_start = 0; b_start < B_single_thread_block; ++b_start) {
        {{template.get_gemm_function_call(
            kernel,
            kernel_name+"_single_thread_mm",
            "<SINGLE_THREAD_CALL_FOR_BMM>",
            b_index="b_start",
        )}}
    }
    for (int64_t b_start = B_single_thread_block; b_start < B; ++b_start) {
        {{template.get_gemm_function_call(
            kernel,
            kernel_name+"_threaded_mm",
            "<THREADED_MM_CALL_FOR_BMM>",
            b_index="b_start",
        )}}
    }
}
"""


````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `b_index`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `b_index` 等值。

### Lines 81-100 / 第 81-100 行
````python
class CppBmmTemplate(CppGemmTemplate):
    def __init__(
        self,
        input_nodes,
        layout: ir.Layout,
        num_threads: int,
        register_blocking: GemmBlocking,
        beta=1,
        alpha=1,
        has_bias=False,
        epilogue_creator: Callable[[ir.Buffer], ir.Pointwise] | None = None,
        should_block_weights: bool = False,
        name="bmm",
    ):
        """
        In order to simplify the implementation and increase code reuse, the BMM template implements
        two versions of the GEMM kernel: a single-threaded version and a multi-threaded version.
        GEMM kernels are called in a loop over the batch dimension, with single-threaded GEMM calls
        for all but the last (B % num_threads), which are handled by the multi-threaded GEMM kernel.

````
- **EN**: Introduces class `CppBmmTemplate`, function `__init__`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `layout`, `num_threads`, `register_blocking`, `beta`, `alpha`, `has_bias`, and `...+3`.
- **CN**: 这里定义了类`CppBmmTemplate`、函数`__init__`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。初始化或更新了 `layout`、`num_threads`、`register_blocking`、`beta`、`alpha`、`has_bias`、`另有3项` 等值。

### Lines 101-120 / 第 101-120 行
````python
        We use an extra sizevar `b_index` to index the batch dimension, which we pass into the GEMM
        template as a sympy.Symbol. This allows us to slice the 3D batch tensors in the GEMM template
        without any changes to the GEMM template itself.
        """
        super().__init__(
            input_nodes,
            layout,
            num_threads,
            register_blocking,
            beta=beta,
            alpha=alpha,
            has_bias=has_bias,
            epilogue_creator=epilogue_creator,
            should_block_weights=should_block_weights,
            name=name,
        )
        self.b_index = sympy.Symbol("s_b_index", integer=True, nonnegative=True)

    @staticmethod
    def get_padded_size(n, block_n, k, should_block_weight):
````
- **EN**: Introduces function `get_padded_size`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`get_padded_size`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 121-140 / 第 121-140 行
````python
        if should_block_weight:
            # Tensor is constant or not contiguous, so we will pad and block
            new_size, padded_n = CppGemmTemplate.get_padded_size(
                n, block_n, k, should_block_weight
            )
            # Add the new batch dimension
            new_size.insert(0, -1)
            return new_size, padded_n
        else:
            new_size = [-1, k, n]
            return new_size, n

    @staticmethod
    def check_if_block_weight(W, micro_gemm):
        assert isinstance(W, ir.IRNode)
        _, n = W.get_size()[-2:]
        result = (
            not W.get_layout().is_contiguous()
            or W.get_name() in V.graph.constants
            or (
````
- **EN**: Introduces function `check_if_block_weight`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`check_if_block_weight`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-160 / 第 141-160 行
````python
                n % micro_gemm.register_blocking.block_n != 0
                and micro_gemm.get_b_layout != LayoutType.NORMAL
            )
        )
        return result

    def get_gemm_function_call(
        self,
        kernel: CppTemplateKernel,
        function_name: str,
        placeholder: str,
        b_index: str,
    ) -> str:
        """
        Similar to 'def_kernel' in cpp_template_kernel, but instead of generating a function definition,
        generate a function call for the GEMM kernel.
        Args:
            placeholder: The string to replace the function call with
            b_index: The index for slicing the 3D batch tensors
        """
````
- **EN**: Introduces function `get_gemm_function_call`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `kernel`, `function_name`, `placeholder`, `b_index`, and `Args`.
- **CN**: 这里定义了函数`get_gemm_function_call`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `kernel`、`function_name`、`placeholder`、`b_index`、`Args` 等值。

### Lines 161-180 / 第 161-180 行
````python

        def hook():
            arg_defs, call_args, _, _ = kernel.args.python_argdefs()
            for i, buf in enumerate(call_args):
                if buf == self.b_index:
                    arg_defs[i] = ArgName(b_index)
            call = f"{function_name}({', '.join(x.full_name() for x in arg_defs)});"
            return call

        assert placeholder not in kernel.render_hooks
        kernel.render_hooks[placeholder] = hook
        return placeholder

    def get_default_reindexers(self, epilogue_nodes):
        def reindexer(args):
            # if epilogue nodes exist, they have 3D ranges but args are 2D, so add 0 index
            return [self.b_index] + args

        return [reindexer] * len(epilogue_nodes)

````
- **EN**: Introduces function `hook`, function `get_default_reindexers`, function `reindexer`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `call`.
- **CN**: 这里定义了函数`hook`、函数`get_default_reindexers`、函数`reindexer`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `call` 等值。

### Lines 181-200 / 第 181-200 行
````python
    def get_options(
        self,
        kernel: CppTemplateKernel,
        template_buffer_node: ir.CppTemplateBuffer | None = None,
        flag_template_buffer_has_other_users: bool | None = None,
        epilogue_nodes: list[ir.IRNode] | None = None,
        **kwargs,
    ) -> dict[str, Any]:
        options = super().get_options(
            kernel=kernel,
            template_buffer_node=template_buffer_node,
            flag_template_buffer_has_other_users=flag_template_buffer_has_other_users,
            epilogue_nodes=epilogue_nodes,
            **kwargs,
        )

        BX, BW, BY = options["X"], options["W"], options["Y"]
        options["BX"], options["BW"], options["BY"] = BX, BW, BY
        options["BY_2d"] = options["Y_2d"]
        for kword in ["X", "W", "GemmOut", "Y_2d"]:
````
- **EN**: Introduces function `get_options`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `kernel`, `template_buffer_node`, `flag_template_buffer_has_other_users`, `epilogue_nodes`, and `options`.
- **CN**: 这里定义了函数`get_options`。包含分支、循环或上下文管理等控制流。初始化或更新了 `kernel`、`template_buffer_node`、`flag_template_buffer_has_other_users`、`epilogue_nodes`、`options` 等值。

### Lines 201-220 / 第 201-220 行
````python
            options[kword] = kernel.select(options[kword], 0, self.b_index)
        for kword in ["X", "W", "Y_2d"]:
            options[kword + "_dtype"] = DTYPE_TO_CPP[options[kword].dtype]
        options["b_index"] = self.b_index
        options["BY_sizevars"] = [
            s
            for sym in itertools.chain(BY.get_size(), BY.get_stride())
            if isinstance(sym, sympy.Expr)
            for s in sym.free_symbols
        ]
        options["kernel_name"] = kernel.kernel_name

        return options

    def render(  # type: ignore[override, return]
        self,
        kernel: CppTemplateKernel,
        template_buffer_node: ir.CppTemplateBuffer | None = None,
        flag_template_buffer_has_other_users: bool | None = None,
        epilogue_nodes: list[ir.IRNode] | None = None,
````
- **EN**: Introduces function `render`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`render`。借助符号表达式来推理索引、形状或代数性质。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 221-240 / 第 221-240 行
````python
        **kwargs,
    ) -> str:
        options = self.get_options(
            kernel=kernel,
            template_buffer_node=template_buffer_node,
            flag_template_buffer_has_other_users=flag_template_buffer_has_other_users,
            epilogue_nodes=epilogue_nodes,
            **kwargs,
        )
        self.render_options = options

        with contextlib.ExitStack() as stack:
            for buf in options["fake_buffers"]:
                stack.enter_context(
                    patch.object(V.graph, "get_dtype", self._fake_get_dtype(buf))
                )
            result = self._template_from_string(BMM_TEMPLATE).render(**options)

            # Finalize the function definitions for the gemm routines
            sub_mm_hooks = {
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `options`, `kernel`, `template_buffer_node`, `flag_template_buffer_has_other_users`, `epilogue_nodes`, `result`, and `...+1`. This range continues the implementation of function `CppBmmTemplate.render`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `options`、`kernel`、`template_buffer_node`、`flag_template_buffer_has_other_users`、`epilogue_nodes`、`result`、`另有1项` 等值。这一段延续了函数`CppBmmTemplate.render` 的具体实现。

### Lines 241-260 / 第 241-260 行
````python
                name: hook
                for name, hook in kernel.render_hooks.items()
                if "FOR_BMM" in name
            }
            result = PartialRender(result, sub_mm_hooks).finalize_all()
            for name in sub_mm_hooks:
                del kernel.render_hooks[name]
            del kernel.args.sizevars[options["b_index"]]
            return result

    def codegen_single_thread_gemm(self):
        stub = self._template_from_string(GEMM_SINGLE_THREAD_MM_STUB).render(
            self.render_options
        )
        return stub + self._template_from_string(GEMM_TEMPLATE).render(
            {**self.render_options, "num_threads": 1}
        )

    def codegen_multi_thread_gemm(self):
        stub = self._template_from_string(GEMM_THREADED_MM_STUB).render(
````
- **EN**: Introduces function `codegen_single_thread_gemm`, function `codegen_multi_thread_gemm`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `name`, `result`, and `stub`.
- **CN**: 这里定义了函数`codegen_single_thread_gemm`、函数`codegen_multi_thread_gemm`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `name`、`result`、`stub` 等值。

### Lines 261-268 / 第 261-268 行
````python
            self.render_options
        )
        return stub + self._template_from_string(GEMM_TEMPLATE).render(
            self.render_options
        )

    def codegen_gemm_stub_def(self):
        return ""
````
- **EN**: Introduces function `codegen_gemm_stub_def`. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`codegen_gemm_stub_def`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `CppBmmTemplate`  
  **CN**: 主要类：`CppBmmTemplate`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `contextlib`, `itertools`, `collections.abc`, `typing`, `unittest.mock`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `..`, `..select_algorithm`, `..virtualized`, `.common`, `.cpp_gemm_template`, `.cpp_micro_gemm`, `.cpp_template_kernel`, `.cpp_utils`
