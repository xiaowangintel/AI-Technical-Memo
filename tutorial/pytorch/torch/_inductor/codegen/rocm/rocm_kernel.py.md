# rocm_kernel.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/rocm/rocm_kernel.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `ROCmKernel`, `ROCmTemplateKernel`, and `ROCmTemplateCaller`. It exposes functions such as `_normalize_idx`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `ROCmKernel`、`ROCmTemplateKernel`、`ROCmTemplateCaller` 等类。同时提供 `_normalize_idx` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import logging
from collections.abc import Callable, Sequence
from typing import Any, TYPE_CHECKING

import torch._inductor.config as config
from torch._inductor.codegen.cpp_wrapper_cpu import CppWrapperCpu
from torch._inductor.utils import do_bench_using_profiling

from ...ir import Buffer, ChoiceCaller, IRNode, Layout, PrimitiveInfoType, TensorBox
from ...virtualized import V
from ..common import Kernel, OpOverrides, WorkspaceArg, WorkspaceZeroMode
from ..cpp_utils import CppPrinter
from .rocm_benchmark_request import ROCmBenchmarkRequest
from .rocm_template_buffer import ROCmTemplateBuffer
from .rocm_utils import DTYPE_TO_ROCM_TYPE


if TYPE_CHECKING:
    from torch._inductor.codegen.rocm.rocm_template import ArgInfo, ROCmTemplate
````
- **EN**: Imports dependencies such as `logging`, `collections.abc`, `typing`, `torch._inductor.config`, `torch._inductor.codegen.cpp_wrapper_cpu`, `torch._inductor.utils`, and `...+8` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `logging`、`collections.abc`、`typing`、`torch._inductor.config`、`torch._inductor.codegen.cpp_wrapper_cpu`、`torch._inductor.utils`、`另有8项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。包含分支、循环或上下文管理等控制流。

### Lines 21-40 / 第 21-40 行
````python

log = logging.getLogger(__name__)

cexpr = CppPrinter().doprint


def _normalize_idx(index: int, total_length: int) -> int:
    return index if index >= 0 else index + total_length


class ROCmKernel(Kernel):
    """
    Baseclass for ROCm based Kernels
    """

    overrides = OpOverrides  # type: ignore[assignment]


class ROCmTemplateKernel(ROCmKernel):
    """
````
- **EN**: Introduces function `_normalize_idx`, class `ROCmKernel`, class `ROCmTemplateKernel`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `log`, `cexpr`, and `overrides`.
- **CN**: 这里定义了函数`_normalize_idx`、类`ROCmKernel`、类`ROCmTemplateKernel`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `log`、`cexpr`、`overrides` 等值。

### Lines 41-60 / 第 41-60 行
````python
    Template kernels defined by ROCm in C++.
    """

    _EXTRA_CPP_ARGS = "size_t* workspace_size, uint8_t* workspace, hipStream_t stream"

    def __init__(
        self,
        kernel_name: str,
        runtime_arg_info: list["ArgInfo"],
        runtime_arg_values: list[Any],
    ) -> None:
        """
        Initializes a new instance of the ROCmTemplateKernel class.

        Args:
            kernel_name (str): The name of the kernel.
        """
        super().__init__()
        self.kernel_name = kernel_name
        # Mapping from arg name to IRNode.
````
- **EN**: Introduces function `__init__`. Initializes or updates values such as `_EXTRA_CPP_ARGS`, `kernel_name`, `runtime_arg_info`, `runtime_arg_values`, and `Args`.
- **CN**: 这里定义了函数`__init__`。初始化或更新了 `_EXTRA_CPP_ARGS`、`kernel_name`、`runtime_arg_info`、`runtime_arg_values`、`Args` 等值。

### Lines 61-80 / 第 61-80 行
````python
        self.named_nodes: dict[str, IRNode] = {}
        self.runtime_arg_info = runtime_arg_info
        self.runtime_arg_values = runtime_arg_values

    def get_signature(self):
        return self.signature

    def def_kernel(
        self,
        inputs: list[IRNode],
        outputs: list[IRNode],
        size_args: list[str],
        names_str: str = "",
        input_reorder: list[int] | None = None,
    ) -> str:
        """
        Hook called from template code to generate function definition and
        needed args.

        Args:
````
- **EN**: Introduces function `get_signature`, function `def_kernel`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `inputs`, `outputs`, `size_args`, `names_str`, `input_reorder`, and `Args`.
- **CN**: 这里定义了函数`get_signature`、函数`def_kernel`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `inputs`、`outputs`、`size_args`、`names_str`、`input_reorder`、`Args` 等值。

### Lines 81-100 / 第 81-100 行
````python
            inputs: List of input IRNodes
            outputs: List of output IRNodes
            names_str: Comma separated list of input + output argument names.
            input_reorder: The actual order of input nodes.
                           e.g. The template might have input argument defined as [X, W, Bias],
                           and the actual input passed into this template could be [Bias, X, W].
                           In this case, the `input_reorder` would be [2, 0, 1].
        """
        names = [x.strip() for x in names_str.strip().split(",")]
        if len(inputs) + len(outputs) != len(names):
            raise RuntimeError(
                f"{len(inputs) + len(outputs)=} != {len(names)=}, {inputs=}, {outputs=}, {names=}"
            )

        if input_reorder == [2, 0, 1]:
            input_reorder = [4, 0, 1, 2, 3]

        if input_reorder is not None:
            assert len(inputs) == len(input_reorder)
        else:
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `inputs`, `outputs`, `names_str`, `input_reorder`, `names`, and `else`. This range continues the implementation of function `ROCmTemplateKernel.def_kernel`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `inputs`、`outputs`、`names_str`、`input_reorder`、`names`、`else` 等值。这一段延续了函数`ROCmTemplateKernel.def_kernel` 的具体实现。

### Lines 101-120 / 第 101-120 行
````python
            input_reorder = list(range(len(inputs)))

        for idx in input_reorder:
            name = names[idx]
            node = inputs[idx]
            if node is not None:
                self.named_nodes[name] = node
                self.args.input_buffers[node.get_name()] = name

        for name, node in zip(names[len(inputs) : len(inputs) + len(outputs)], outputs):
            if node is not None:
                self.named_nodes[name] = node
                self.args.output_buffers[node.get_name()] = name

        arg_defs, *_ = self.args.cpp_argdefs(DTYPE_TO_ROCM_TYPE)

        runtime_arg_defs = [f"{arg.ty} {arg.name}" for arg in self.runtime_arg_info]

        signature = f"int {self.kernel_name}({', '.join(arg_defs + size_args + runtime_arg_defs)},{self._EXTRA_CPP_ARGS})"
        self.signature = signature
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_reorder`, `name`, `node`, `runtime_arg_defs`, and `signature`. This range continues the implementation of function `ROCmTemplateKernel.def_kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `input_reorder`、`name`、`node`、`runtime_arg_defs`、`signature` 等值。这一段延续了函数`ROCmTemplateKernel.def_kernel` 的具体实现。

### Lines 121-140 / 第 121-140 行
````python
        return signature

    def call_kernel(
        self,
        name: str,
        node: "ROCmTemplateBuffer",  # type: ignore[name-defined]
    ) -> None:
        """
        Generates code to call the kernel through V.graph.wrapper_code.
        used from within torch._inductor.wrapper.PythonWrapperCodegen

        name: Name of kernel function.
        node: The ROCmTemplateBuffer node which contains information about the kernel, it's fused epilogue nodes
        as well as all required inputs and outputs.
        """
        wrapper = V.graph.wrapper_code

        arg_types: list[Any]
        if V.graph.cpp_wrapper:
            # Make sure we initialize these kernels since they're exported as
````
- **EN**: Introduces function `call_kernel`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `name`, `node`, `wrapper`, and `arg_types`.
- **CN**: 这里定义了函数`call_kernel`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `name`、`node`、`wrapper`、`arg_types` 等值。

### Lines 141-160 / 第 141-160 行
````python
            # C-style symbol names.
            assert isinstance(wrapper, CppWrapperCpu)
            wrapper.initialized_kernels[name] = self
            # Kinda hacky because we always originally initialize name with "KERNEL_NAME"
            # So, we replace with the real kernel name passed as an arg to this function.
            self.signature = self.signature.replace("KERNEL_NAME", name)
            _, call_args, arg_types = self.args.cpp_argdefs(DTYPE_TO_ROCM_TYPE)
        else:
            _, call_args, _, arg_types = self.args.python_argdefs()

        kernel_args = []
        for arg in call_args:
            # dynamo wraps unspec variable as 0d CPU tensor, need convert to scalar
            if V.graph.is_unspec_arg(arg):
                arg = arg + ".item()"
            else:
                if not V.graph.cpp_wrapper:
                    arg = f"c_void_p({arg}.data_ptr())"
            kernel_args.append(arg)

````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `kernel_args`, and `arg`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`kernel_args`、`arg` 等值。

### Lines 161-180 / 第 161-180 行
````python
        # add size args
        size_args = [
            f"{V.graph.sizevars.simplify(sarg)}" for sarg in node.template.size_args()
        ]

        if V.graph.cpp_wrapper:
            kernel_args.extend(size_args)
        else:
            kernel_args.extend(f"c_int({sarg})" for sarg in size_args)

        if V.graph.cpp_wrapper:
            arg_types.extend(["int"] * len(node.template.size_args()))

        # the runtime args come right after the size args
        kernel_args.extend(self.runtime_arg_values)
        for arg in self.runtime_arg_info:
            arg_types.append(arg.ty)

        # workspace_size ptr is NULL to mark this call is not intended for retrieving workspace_size.
        # workspace_size should have already been retrieved prior to this call.
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `size_args`, and `else`. This range continues the implementation of function `ROCmTemplateKernel.call_kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `size_args`、`else` 等值。这一段延续了函数`ROCmTemplateKernel.call_kernel` 的具体实现。

### Lines 181-200 / 第 181-200 行
````python
        kernel_args.append("nullptr" if V.graph.cpp_wrapper else "None")
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
            data_ptr = f"{ws.outer_name}.data_ptr()"
            kernel_args.append(
                data_ptr if V.graph.cpp_wrapper else f"c_void_p({data_ptr})"
            )
        else:
            ws = None
            kernel_args.append("nullptr" if V.graph.cpp_wrapper else "None")
        if V.graph.cpp_wrapper:
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `ws`, `count`, `device`, `zero_mode`, `outer_name`, `data_ptr`, and `...+1`. This range continues the implementation of function `ROCmTemplateKernel.call_kernel`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `ws`、`count`、`device`、`zero_mode`、`outer_name`、`data_ptr`、`另有1项` 等值。这一段延续了函数`ROCmTemplateKernel.call_kernel` 的具体实现。

### Lines 201-220 / 第 201-220 行
````python
            arg_types.append("uint8_t*")
        wrapper.generate_kernel_call(
            name,
            kernel_args,
            triton=False,
            arg_types=arg_types,
        )
        if ws:
            wrapper.generate_workspace_deallocation(ws)


class ROCmTemplateCaller(ChoiceCaller):
    """
    ROCmTemplateCaller

    This class represents a caller for ROCm template kernels. It is a subclass of ChoiceCaller.
    Attributes:
        name (str): The name of the caller.
        category (str): The category of the caller.
        bmreq (ROCmBenchmarkRequest): The benchmark request for the caller.
````
- **EN**: Introduces class `ROCmTemplateCaller`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Processes profiling, FLOP, or benchmark information to characterize execution cost. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了类`ROCmTemplateCaller`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。包含分支、循环或上下文管理等控制流。

### Lines 221-240 / 第 221-240 行
````python
        template_buffer (ROCmTemplateBuffer): The template buffer for the caller.
    """

    def __init__(
        self,
        name: str,
        category: str,
        input_nodes: list[Buffer],
        layout: Layout,
        make_kernel_render: Callable[
            [ROCmTemplateBuffer, Sequence[IRNode] | None], str
        ],
        bmreq: ROCmBenchmarkRequest,
        template: "ROCmTemplate",  # type: ignore[name-defined]
        info_kwargs: dict[str, PrimitiveInfoType | list[PrimitiveInfoType]] | None,  # type: ignore[type-arg]
    ) -> None:
        super().__init__(name, input_nodes, layout, description="")
        self.category = category
        self.make_kernel_render = make_kernel_render
        self.bmreq = bmreq
````
- **EN**: Introduces function `__init__`. Initializes or updates values such as `name`, `category`, `input_nodes`, `layout`, `make_kernel_render`, `bmreq`, and `...+2`.
- **CN**: 这里定义了函数`__init__`。初始化或更新了 `name`、`category`、`input_nodes`、`layout`、`make_kernel_render`、`bmreq`、`另有2项` 等值。

### Lines 241-260 / 第 241-260 行
````python
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
        return f"ROCmTemplateCaller(source_file={self.bmreq.source_file}, {self.info_dict()})"

    def call_name(self) -> str:
        return f"rocm_template_kernels.{self.name}"

````
- **EN**: Introduces function `precompile`, function `benchmark`, function `__str__`, function `call_name`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`precompile`、函数`benchmark`、函数`__str__`、函数`call_name`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 261-280 / 第 261-280 行
````python
    def hash_key(self) -> str:
        return "-".join(
            [
                self.category,
                self.bmreq.hash_key,
            ]
        )

    def info_dict(self) -> dict[str, PrimitiveInfoType | list[PrimitiveInfoType]]:
        """Information returned here is logged to the autotune log file when that is enabled."""
        return {
            "backend": "ROCm",
            "name": self.name,
            **dict(self.info_kwargs["op"].dict_items()),  # type: ignore[union-attr, index]
        }

    def output_node(self) -> TensorBox:
        self.bmreq.update_workspace_size()
        buffer = ROCmTemplateBuffer(
            layout=self.layout,
````
- **EN**: Introduces function `hash_key`, function `info_dict`, function `output_node`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `buffer`, and `layout`.
- **CN**: 这里定义了函数`hash_key`、函数`info_dict`、函数`output_node`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `buffer`、`layout` 等值。

### Lines 281-289 / 第 281-289 行
````python
            inputs=self.input_nodes,
            make_kernel_render=self.make_kernel_render,
            workspace_size=self.bmreq.workspace_size,
            template=self.template,
        )
        # Pass KTC annotation to the buffer for encoding
        if "ktc" in self.annotations:
            buffer.annotations["ktc"] = self.annotations["ktc"]
        return TensorBox.create(buffer)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `inputs`, `make_kernel_render`, `workspace_size`, and `template`. This range continues the implementation of function `ROCmTemplateCaller.output_node`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `inputs`、`make_kernel_render`、`workspace_size`、`template` 等值。这一段延续了函数`ROCmTemplateCaller.output_node` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Profiling data handling  
  **CN**: 性能画像数据处理
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Autotuning support  
  **CN**: 自动调优支持
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `collections.abc`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch._inductor.config`, `torch._inductor.codegen.cpp_wrapper_cpu`, `torch._inductor.utils`, `...ir`, `...virtualized`, `..common`, `..cpp_utils`, `.rocm_benchmark_request`, `.rocm_template_buffer`, `.rocm_utils`, `torch._inductor.codegen.rocm.rocm_template`
