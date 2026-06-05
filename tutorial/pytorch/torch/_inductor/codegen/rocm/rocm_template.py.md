# rocm_template.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/rocm/rocm_template.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `ArgInfo`, and `ROCmTemplate`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `ArgInfo`、`ROCmTemplate` 等类。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
# mypy: allow-untyped-defs
import functools
import itertools
import logging
from collections.abc import Sequence
from dataclasses import dataclass
from typing import Any
from unittest.mock import patch

from ...autotune_process import TensorMeta
from ...ir import Buffer, IRNode, Layout
from ...utils import IndentedBuffer, unique
from ...virtualized import V
from ..common import KernelTemplate
````
- **EN**: Imports dependencies such as `functools`, `itertools`, `logging`, `collections.abc`, `dataclasses`, `typing`, and `...+6` for the logic in this range.
- **CN**: 这里导入了 `functools`、`itertools`、`logging`、`collections.abc`、`dataclasses`、`typing`、`另有6项` 等依赖，为后续逻辑提供基础能力。

### Lines 15-28 / 第 15-28 行
````python
from .rocm_benchmark_request import ROCmBenchmarkRequest
from .rocm_kernel import ROCmTemplateCaller, ROCmTemplateKernel
from .rocm_template_buffer import ROCmTemplateBuffer
from .rocm_utils import DTYPE_TO_ROCM_TYPE


log = logging.getLogger(__name__)


# FIXME: unify with the CUDA version
@dataclass(frozen=True)
class ArgInfo:
    name: str
    ty: str
````
- **EN**: Imports dependencies such as `.rocm_benchmark_request`, `.rocm_kernel`, `.rocm_template_buffer`, and `.rocm_utils` for the logic in this range. Introduces class `ArgInfo`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields.
- **CN**: 这里导入了 `.rocm_benchmark_request`、`.rocm_kernel`、`.rocm_template_buffer`、`.rocm_utils` 等依赖，为后续逻辑提供基础能力。这里定义了类`ArgInfo`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。

### Lines 29-42 / 第 29-42 行
````python


class ROCmTemplate(KernelTemplate):
    index_counter = itertools.count()
    gfx9_threads_per_warp = 64

    def __init__(
        self,
        name: str,
        input_nodes: list[Buffer],
        layout: Layout,
        input_reorder: list[int] | None = None,
    ) -> None:
        """
````
- **EN**: Introduces class `ROCmTemplate`, function `__init__`. Initializes or updates values such as `index_counter`, `gfx9_threads_per_warp`, `name`, `input_nodes`, `layout`, and `input_reorder`.
- **CN**: 这里定义了类`ROCmTemplate`、函数`__init__`。初始化或更新了 `index_counter`、`gfx9_threads_per_warp`、`name`、`input_nodes`、`layout`、`input_reorder` 等值。

### Lines 43-56 / 第 43-56 行
````python

        Baseclass for ROCm C++ Templates, derived from KernelTemplate. Not to be instantiated directly.

        Args:
            name (str): The name of the ROCmTemplate object.
            input_nodes (List[IRNode]): A list of input IRNodes.
            layout (Layout): The layout of the output buffer / tensor.
            input_reorder (Optional[List[int]]): An optional list that specifies the order of the input nodes.

        """
        super().__init__(name)
        self.input_nodes = input_nodes
        self.output_node: Buffer = Buffer(name="buf_out", layout=layout)
        self.input_reorder = input_reorder
````
- **EN**: Initializes or updates values such as `Args`. This range continues the implementation of function `ROCmTemplate.__init__`.
- **CN**: 初始化或更新了 `Args` 等值。这一段延续了函数`ROCmTemplate.__init__` 的具体实现。

### Lines 57-70 / 第 57-70 行
````python
        self.layout = layout

    def generate(  # type: ignore[override]
        self,
        **kwargs,
    ) -> ROCmTemplateCaller:
        """
        Generates the ROCm template caller object for the given GEMM template and operation. This ROCmTemplateCaller
        may be used to call and benchmark the generated ROCm kernel in a standalone manner to enable Autotuning.

        Args:
            kwargs: Additional keyword arguments.

        Returns:
````
- **EN**: Introduces function `generate`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Initializes or updates values such as `Args`, `kwargs`, and `Returns`.
- **CN**: 这里定义了函数`generate`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。初始化或更新了 `Args`、`kwargs`、`Returns` 等值。

### Lines 71-84 / 第 71-84 行
````python
            A ROCmTemplateCaller object representing the generated ROCm template caller.
        """
        kernel_name = f"rocm_{self.name}"
        kernel_hash_name = f"rocm_{self.name}_{next(self.index_counter)}"
        with (
            patch.object(V.graph, "get_dtype", self._fake_get_dtype(self.output_node)),
            ROCmTemplateKernel(
                kernel_name=kernel_name,
                runtime_arg_info=self.get_runtime_arg_info(),
                runtime_arg_values=self.get_runtime_arg_values(**kwargs),
            ) as kernel,
        ):
            code = self.render(kernel=kernel, **kwargs)
            _, call_args, _, _ = kernel.args.python_argdefs()
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `kernel_name`, `kernel_hash_name`, `runtime_arg_info`, `runtime_arg_values`, and `code`. This range continues the implementation of function `ROCmTemplate.generate`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `kernel_name`、`kernel_hash_name`、`runtime_arg_info`、`runtime_arg_values`、`code` 等值。这一段延续了函数`ROCmTemplate.generate` 的具体实现。

### Lines 85-98 / 第 85-98 行
````python
            log.debug("Autotune key: %s, Generated Code:\n%s", kernel_hash_name, code)
            log.debug(
                "Args: cpp_argdefs: %s, python_argdefs: %s",
                kernel.args.cpp_argdefs(DTYPE_TO_ROCM_TYPE),
                kernel.args.python_argdefs(),
            )

        input_reorder = (
            self.input_reorder
            if self.input_reorder is not None
            else list(range(len(self.input_nodes)))
        )
        expected_args = list(
            unique(self.input_nodes[idx].get_name() for idx in input_reorder)
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_reorder`, and `expected_args`. This range continues the implementation of function `ROCmTemplate.generate`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `input_reorder`、`expected_args` 等值。这一段延续了函数`ROCmTemplate.generate` 的具体实现。

### Lines 99-112 / 第 99-112 行
````python
        )
        expected_args.extend([self.output_node.get_name()])
        assert list(call_args)[: len(expected_args)] == expected_args, (
            call_args,
            expected_args,
        )

        size_args = (
            self.size_args() if hasattr(self, "size_args") else ()
        )  # subclass should define def size_args()
        # Resolve symbolic sizes to concrete ints for benchmarking only.
        size_args_ints = list(V.graph.sizevars.optimization_hints(size_args))
        # The runtime args come right after the size args
        runtime_args = self.get_runtime_arg_values(**kwargs)
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `size_args`, `size_args_ints`, and `runtime_args`. This range continues the implementation of function `ROCmTemplate.generate`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `size_args`、`size_args_ints`、`runtime_args` 等值。这一段延续了函数`ROCmTemplate.generate` 的具体实现。

### Lines 113-126 / 第 113-126 行
````python
        extra_args = size_args_ints + runtime_args
        bmreq = ROCmBenchmarkRequest(
            kernel_name=kernel_name,
            input_tensor_meta=TensorMeta.from_irnodes(self.input_nodes),
            output_tensor_meta=TensorMeta.from_irnodes(self.output_node),
            extra_args=extra_args,
            source_code=code,
        )

        def make_kernel_render(
            template_node: ROCmTemplateBuffer,
            epilogue_nodes: Sequence[IRNode] | None = None,
        ):
            kernel = ROCmTemplateKernel(
````
- **EN**: Introduces function `make_kernel_render`. Initializes or updates values such as `extra_args`, `bmreq`, `kernel_name`, `input_tensor_meta`, `output_tensor_meta`, `source_code`, and `...+3`.
- **CN**: 这里定义了函数`make_kernel_render`。初始化或更新了 `extra_args`、`bmreq`、`kernel_name`、`input_tensor_meta`、`output_tensor_meta`、`source_code`、`另有3项` 等值。

### Lines 127-140 / 第 127-140 行
````python
                kernel_name="KERNEL_NAME",
                runtime_arg_info=self.get_runtime_arg_info(),
                runtime_arg_values=self.get_runtime_arg_values(**kwargs),
            )
            render = functools.partial(
                self.render,
                kernel=kernel,
                template_buffer_node=template_node,
                epilogue_nodes=epilogue_nodes,
                **kwargs,  # includes "op" argument in case of CUTLASSGemmTemplate
            )
            return kernel, render

        return ROCmTemplateCaller(
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `kernel_name`, `runtime_arg_info`, `runtime_arg_values`, `render`, `kernel`, `template_buffer_node`, and `...+1`. This range continues the implementation of function `ROCmTemplate.generate.make_kernel_render`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `kernel_name`、`runtime_arg_info`、`runtime_arg_values`、`render`、`kernel`、`template_buffer_node`、`另有1项` 等值。这一段延续了函数`ROCmTemplate.generate.make_kernel_render` 的具体实现。

### Lines 141-154 / 第 141-154 行
````python
            kernel_hash_name,
            self.name,
            self.input_nodes,
            self.output_node.get_layout(),
            make_kernel_render,
            bmreq,
            self,
            kwargs,
        )

    def header(self) -> IndentedBuffer:
        res = IndentedBuffer()
        res.splice(
            """
````
- **EN**: Introduces function `header`. Initializes or updates values such as `res`.
- **CN**: 这里定义了函数`header`。初始化或更新了 `res` 等值。

### Lines 155-168 / 第 155-168 行
````python
                #include <exception>
                #include <iostream>
                #include <memory>
                #include <random>
                #include <vector>
            """
        )
        return res

    def globals(self) -> IndentedBuffer:
        res = IndentedBuffer()
        res.splice(
            """
                // We compile all models with -fvisibility=hidden. Any symbols that need to be
````
- **EN**: Introduces function `globals`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `res`.
- **CN**: 这里定义了函数`globals`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `res` 等值。

### Lines 169-182 / 第 169-182 行
````python
                // exposed in the final shared library must be declared with PT_EXPORT to make
                // them visible.
                #ifdef __GNUC__ // Applies to any compiler with GNU extensions (clang and g++)
                #define PT_EXPORT __attribute__((__visibility__("default")))
                #else
                #ifdef _WIN32
                #define PT_EXPORT __declspec(dllexport)
                #else
                #define PT_EXPORT
                #endif
                #endif
            """
        )
        return res
````
- **EN**: Includes returns or checks that define the contract of this code path. This range continues the implementation of function `ROCmTemplate.globals`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。这一段延续了函数`ROCmTemplate.globals` 的具体实现。

### Lines 183-191 / 第 183-191 行
````python

    def render(self, **kwargs) -> str:
        raise NotImplementedError

    def get_runtime_arg_info(self) -> list[ArgInfo]:
        return []

    def get_runtime_arg_values(self, **kwargs) -> list[Any]:
        return []
````
- **EN**: Introduces function `render`, function `get_runtime_arg_info`, function `get_runtime_arg_values`. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`render`、函数`get_runtime_arg_info`、函数`get_runtime_arg_values`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Autotuning support  
  **CN**: 自动调优支持
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `ArgInfo`, and `ROCmTemplate`  
  **CN**: 主要类：`ArgInfo`、`ROCmTemplate`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`, `itertools`, `logging`, `collections.abc`, `dataclasses`, `typing`, `unittest.mock`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `...autotune_process`, `...ir`, `...utils`, `...virtualized`, `..common`, `.rocm_benchmark_request`, `.rocm_kernel`, `.rocm_template_buffer`, `.rocm_utils`
