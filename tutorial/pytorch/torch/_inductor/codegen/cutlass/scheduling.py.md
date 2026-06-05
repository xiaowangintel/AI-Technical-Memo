# scheduling.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/cutlass/scheduling.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `WhyNoFuseNames`, and `CUTLASSScheduling`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `WhyNoFuseNames`、`CUTLASSScheduling` 等类。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import hashlib
import logging
from collections.abc import Sequence
from typing import cast, TypeGuard

from torch._inductor.codegen.cutlass.python_evt import (
    CutlassEVTCodegen,
    MockCutlassHandler,
)
from torch._inductor.utils import Placeholder
from torch.utils._ordered_set import OrderedSet

from ...._dynamo.utils import counters
from ... import config
from ...codecache import code_hash, get_path
from ...ir import Buffer, ComputedBuffer, CUTLASSTemplateBuffer, Pointwise
from ...scheduler import (
    BaseSchedulerNode,
    BaseScheduling,
````
- **EN**: Imports dependencies such as `hashlib`, `logging`, `collections.abc`, `typing`, `torch._inductor.codegen.cutlass.python_evt`, `torch._inductor.utils`, and `...+6` for the logic in this range. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里导入了 `hashlib`、`logging`、`collections.abc`、`typing`、`torch._inductor.codegen.cutlass.python_evt`、`torch._inductor.utils`、`另有6项` 等依赖，为后续逻辑提供基础能力。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 21-40 / 第 21-40 行
````python
    FusedSchedulerNode,
    SchedulerNode,
    WhyNoFuse,
)
from ...utils import get_fused_kernel_name, get_kernel_metadata, sympy_product
from ...virtualized import V
from ..common import BackendFeature, IndentedBuffer


log = logging.getLogger(__name__)


class WhyNoFuseNames(WhyNoFuse):
    def __init__(self, name1: str, name2: str) -> None:
        self.name1 = name1
        self.name2 = name2


class CUTLASSScheduling(BaseScheduling):
    """
````
- **EN**: Imports dependencies such as `...utils`, `...virtualized`, and `..common` for the logic in this range. Introduces class `WhyNoFuseNames`, function `__init__`, class `CUTLASSScheduling`. Initializes or updates values such as `log`.
- **CN**: 这里导入了 `...utils`、`...virtualized`、`..common` 等依赖，为后续逻辑提供基础能力。这里定义了类`WhyNoFuseNames`、函数`__init__`、类`CUTLASSScheduling`。初始化或更新了 `log` 等值。

### Lines 41-60 / 第 41-60 行
````python
    Partial Scheduling implementation for cutlass C++ Kernels.
    This class is intended to be used in combination with TritonScheduling,
    and delegated to by CUDACombinedScheduling/XPUCombinedScheduling.

    It handles fusion decisions and cutlass C++ specific template code generation.
    """

    @classmethod
    def get_backend_features(cls, device) -> OrderedSet[BackendFeature]:
        return OrderedSet()

    def group_fn(self, sizes):
        return tuple(V.graph.sizevars.simplify(sympy_product(s)) for s in sizes)

    @staticmethod
    def is_cutlass_template(node: BaseSchedulerNode) -> TypeGuard[SchedulerNode]:
        return isinstance(node, SchedulerNode) and isinstance(
            node.node, CUTLASSTemplateBuffer
        )

````
- **EN**: Introduces function `get_backend_features`, function `group_fn`, function `is_cutlass_template`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_backend_features`、函数`group_fn`、函数`is_cutlass_template`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 61-80 / 第 61-80 行
````python
    def is_cutlass_fused_template(self, node: BaseSchedulerNode) -> bool:
        return isinstance(node, FusedSchedulerNode) and self.is_cutlass_template(node)

    def can_fuse_vertical(
        self, node1: BaseSchedulerNode, node2: BaseSchedulerNode
    ) -> bool:
        if self.is_cutlass_template(node1) and isinstance(node2, BaseSchedulerNode):
            assert node1.node, "node1.node should not be None"
            return self._can_fuse_epilogue_impl(
                cast(CUTLASSTemplateBuffer, node1.node),
                [],
                node2,  # type: ignore[arg-type]
            )
        elif self.is_cutlass_fused_template(node1) and isinstance(
            node2, BaseSchedulerNode
        ):
            assert node1.node, "node1.node should not be None"
            assert node2.node, "node2.node should not be None"
            fnode1 = cast(FusedSchedulerNode, node1)
            return self._can_fuse_epilogue_impl(
````
- **EN**: Introduces function `is_cutlass_fused_template`, function `can_fuse_vertical`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `fnode1`.
- **CN**: 这里定义了函数`is_cutlass_fused_template`、函数`can_fuse_vertical`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `fnode1` 等值。

### Lines 81-100 / 第 81-100 行
````python
                fnode1.get_template_node(),  # type: ignore[arg-type]
                self._unwrap_epilogue_nodes(fnode1),
                node2,  # type: ignore[arg-type]
            )

        return False

    def define_kernel(self, src_code: str, node_schedule) -> str:
        wrapper = V.graph.wrapper_code
        if src_code in wrapper.src_to_kernel:
            kernel_name = wrapper.src_to_kernel[src_code]
        else:
            fused_name = (
                get_fused_kernel_name(node_schedule, config.triton.descriptive_names)
                if config.triton.descriptive_names
                else ""
            )

            # use the original src_code as the key
            kernel_hash = hashlib.sha256(src_code.encode("utf-8")).hexdigest()[:8]
````
- **EN**: Introduces function `define_kernel`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`define_kernel`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 101-120 / 第 101-120 行
````python
            if fused_name == "fused":
                # no EVT kernel, use the original kernel name
                kernel_name = f"cutlass_{kernel_hash}"
            else:
                kernel_name = f"cutlass_{fused_name}_{kernel_hash}"
            wrapper.src_to_kernel[src_code] = kernel_name
            src_code = src_code.replace(str(Placeholder.KERNEL_NAME), kernel_name)

            _, _, kernel_path = get_path(code_hash(src_code), "py")

            compile_wrapper = IndentedBuffer()
            compile_wrapper.writeline(f"async_compile.{V.graph.device_type}(r'''")
            compile_wrapper.splice(src_code, strip=True)
            compile_wrapper.writeline(
                f"''', 'so', aot_compile={str(V.graph.aot_mode)})"
            )

            metadata_comment = f"# kernel path: {kernel_path}"
            origins, detailed_origins = get_kernel_metadata(node_schedule, wrapper)
            metadata_comment += "\n" + origins + "\n" + detailed_origins
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `kernel_name`, `else`, `src_code`, `compile_wrapper`, and `metadata_comment`. This range continues the implementation of function `CUTLASSScheduling.define_kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `kernel_name`、`else`、`src_code`、`compile_wrapper`、`metadata_comment` 等值。这一段延续了函数`CUTLASSScheduling.define_kernel` 的具体实现。

### Lines 121-140 / 第 121-140 行
````python
            wrapper.define_kernel(
                kernel_name, compile_wrapper.getvalue(), metadata_comment
            )
        return kernel_name

    def codegen_template(
        self,
        template_node: BaseSchedulerNode,
        epilogue_nodes: Sequence[BaseSchedulerNode],
        prologue_nodes: Sequence[BaseSchedulerNode],
    ):
        """
        Codegen a cutlass template, possibly with fused epilogues
        """
        counters["inductor"]["cutlass_epilogue_fusion_counter"] += len(epilogue_nodes)
        assert self.is_cutlass_template(template_node), (
            "Template node passed to CUTLASSScheduling.codegen_template must be a SchedulerNode that wraps a CUTLASSTemplateBuffer"
        )
        _, (_numel, rnumel) = template_node.group
        assert rnumel == 1
````
- **EN**: Introduces function `codegen_template`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `template_node`, `epilogue_nodes`, and `prologue_nodes`.
- **CN**: 这里定义了函数`codegen_template`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `template_node`、`epilogue_nodes`、`prologue_nodes` 等值。

### Lines 141-160 / 第 141-160 行
````python
        ctb: CUTLASSTemplateBuffer = cast(CUTLASSTemplateBuffer, template_node.node)
        epilogue_ir_nodes: list[Buffer] = [n.node for n in epilogue_nodes]  # type: ignore[misc]
        assert all(isinstance(n, ComputedBuffer) for n in epilogue_ir_nodes), (
            "Epilogue nodes must all be instances of ir.ComputedBuffer"
        )
        kernel, render = ctb.make_kernel_render(  # type: ignore[misc]
            ctb, epilogue_nodes=epilogue_nodes
        )
        with kernel:
            for node in [template_node, *epilogue_nodes]:
                node.mark_run()

            # typically there is a codegen pass which runs after mark_run
            # for this kernel we've already generated the C++ code, but we still
            # need to let the kernel know about loads/stores that occur in the fused
            # kernel for memory planning to properly optimize allocations
            ctb.emulate_store_fn()
            for node in epilogue_ir_nodes:
                with V.set_ops_handler(MockCutlassHandler(V.get_ops_handler())):
                    assert isinstance(
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `ctb`, and `epilogue_ir_nodes`. This range continues the implementation of function `CUTLASSScheduling.codegen_template`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `ctb`、`epilogue_ir_nodes` 等值。这一段延续了函数`CUTLASSScheduling.codegen_template` 的具体实现。

### Lines 161-180 / 第 161-180 行
````python
                        node, ComputedBuffer
                    )  # Not sure why we need to do this again
                    node.get_store_function()(CutlassEVTCodegen.get_index_vars(node))

        with V.set_kernel_handler(kernel):
            src_code = render()
            node_schedule = [template_node, *epilogue_nodes]
            kernel_name = self.define_kernel(src_code, node_schedule)

        # debug printing values of intermediate tensors
        _, call_args, arg_signatures, _ = kernel.args.python_argdefs()
        debug_printer_manager = V.graph.wrapper_code.debug_printer
        debug_printer_manager.set_printer_args(
            call_args, kernel_name, arg_signatures, kernel
        )
        with debug_printer_manager:
            self.codegen_comment(node_schedule, kernel_name)
            kernel.call_kernel(kernel_name, ctb)

        V.graph.removed_buffers |= kernel.removed_buffers
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `src_code`, `node_schedule`, `kernel_name`, and `debug_printer_manager`. This range continues the implementation of function `CUTLASSScheduling.codegen_template`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `src_code`、`node_schedule`、`kernel_name`、`debug_printer_manager` 等值。这一段延续了函数`CUTLASSScheduling.codegen_template` 的具体实现。

### Lines 181-200 / 第 181-200 行
````python
        self.free_buffers_in_scheduler()

    @staticmethod
    def _unwrap_epilogue_nodes(
        fused_node: FusedSchedulerNode,
    ) -> list[BaseSchedulerNode]:
        nodes = fused_node.get_nodes()
        template_node = fused_node.get_template_node()
        assert all(n.node is not None for n in nodes), (
            "All epilogue nodes should have an IRNode"
        )
        # pyrefly: ignore [redundant-cast]
        return cast(
            list[BaseSchedulerNode], [n for n in nodes if n.node is not template_node]
        )

    def _can_fuse_epilogue_impl(
        self,
        cutlass_template_buffer: CUTLASSTemplateBuffer,
        existing_epilogue_nodes: list[BaseSchedulerNode],
````
- **EN**: Introduces function `_unwrap_epilogue_nodes`, function `_can_fuse_epilogue_impl`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `fused_node`, `nodes`, `template_node`, `cutlass_template_buffer`, and `existing_epilogue_nodes`.
- **CN**: 这里定义了函数`_unwrap_epilogue_nodes`、函数`_can_fuse_epilogue_impl`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `fused_node`、`nodes`、`template_node`、`cutlass_template_buffer`、`existing_epilogue_nodes` 等值。

### Lines 201-220 / 第 201-220 行
````python
        node_to_fuse: BaseSchedulerNode,
    ) -> bool:
        """
        Check if the given node can be fused with the epilogue. At the moment, Kernels
        support fusion with Pointwise operations, wrapped in (named) ComputedBuffer nodes.

        Args:
            cutlass_template_buffer : A CUTLASSTemplateBuffer object representing the CUTLASS template and it's result buffer
            existing_epilogue_nodes : List[SchedulerNode]: The list of already fused epilogue nodes.
            node_to_fuse: The SchedulerNode node to be checked if it can be fused with the epilogue.
        Returns:
        - bool: True if the given node can be fused with the epilogue, False otherwise.

        """
        why = WhyNoFuseNames(
            cutlass_template_buffer.get_name(), node_to_fuse.get_name()
        )

        scheduler_nodes_to_fuse = node_to_fuse.get_nodes()

````
- **EN**: Initializes or updates values such as `node_to_fuse`, `Args`, `cutlass_template_buffer`, `existing_epilogue_nodes`, `Returns`, `why`, and `...+1`. This range continues the implementation of function `CUTLASSScheduling._can_fuse_epilogue_impl`.
- **CN**: 初始化或更新了 `node_to_fuse`、`Args`、`cutlass_template_buffer`、`existing_epilogue_nodes`、`Returns`、`why`、`另有1项` 等值。这一段延续了函数`CUTLASSScheduling._can_fuse_epilogue_impl` 的具体实现。

### Lines 221-240 / 第 221-240 行
````python
        assert isinstance(cutlass_template_buffer, CUTLASSTemplateBuffer)

        # Checks on constituent nodes
        for s_node in scheduler_nodes_to_fuse:
            node = s_node.node

            if not isinstance(node, ComputedBuffer):
                why(f"{node} is not a ComputedBuffer")
                return False
            elif not isinstance(node.data, Pointwise):
                why(f"{node} is not a Pointwise op")
                return False
            elif not node.get_computed_buffer_name():  # type: ignore[attr-defined]
                why(f"{node} does not have a computed buffer name")
                return False

            name = node.get_computed_buffer_name()  # type: ignore[attr-defined]
            # dtype can differ, and strides can differ as long as they are broadcastable
            if node.get_size() != cutlass_template_buffer.get_size():
                why(
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `node`, and `name`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `node`、`name` 等值。

### Lines 241-260 / 第 241-260 行
````python
                    f"{name}'s size: {node.get_size()} differs from {cutlass_template_buffer.get_name()}'s \
size: {cutlass_template_buffer.get_size()}"
                )
                return False

        assert len(
            existing_epilogue_nodes
        ) or cutlass_template_buffer.get_name() in OrderedSet(
            [rd.name for rd in node_to_fuse.read_writes.reads]
        ), "First epilogue node must read from cutlass template buffer"

        if node_to_fuse.has_aliasing_or_mutation():
            why(f"{node_to_fuse.get_name()} has aliasing or mutation")
            return False
        elif node_to_fuse.is_reduction():
            why(
                f"{node_to_fuse.get_name()} is a reduction which is not yet supported by EVT"
            )
            return False
        elif (
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `size`. This range continues the implementation of function `CUTLASSScheduling._can_fuse_epilogue_impl`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `size` 等值。这一段延续了函数`CUTLASSScheduling._can_fuse_epilogue_impl` 的具体实现。

### Lines 261-280 / 第 261-280 行
````python
            not config.cutlass.cutlass_epilogue_fusion_enabled
            or not config.epilogue_fusion
        ):
            why("cutlass epilogue fusion is not enabled")
            return False
        elif not cutlass_template_buffer.supports_epilogue_fusion:
            why("epilogue fusion is only supported for TMA-enabled gemm ops")
            return False

        try:
            from torch._inductor.codegen.cutlass.python_evt import CutlassEVTCodegen

            CutlassEVTCodegen.ir_to_evt_python_code(
                cutlass_template_buffer.get_name(),
                existing_epilogue_nodes + list(node_to_fuse.get_nodes()),
                OrderedSet(),
            )

        except NotImplementedError as e:
            not_implemented_op = str(e)
````
- **EN**: Imports dependencies such as `torch._inductor.codegen.cutlass.python_evt` for the logic in this range. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `try`, and `not_implemented_op`.
- **CN**: 这里导入了 `torch._inductor.codegen.cutlass.python_evt` 等依赖，为后续逻辑提供基础能力。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `try`、`not_implemented_op` 等值。

### Lines 281-295 / 第 281-295 行
````python
            if not_implemented_op.startswith("_op_"):
                not_implemented_op = not_implemented_op[4:]
                why(
                    f"Cannot fuse epilogue node {node_to_fuse} into {cutlass_template_buffer.name}, \
likely due to unsupported operation: {not_implemented_op}"
                )
                return False
            else:  # Likely due to unsupported dtype.
                why(
                    f"Cannot fuse epilogue node {node_to_fuse} into {cutlass_template_buffer.name}. \
Reason: {not_implemented_op}"
                )
                return False

        return True
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `not_implemented_op`, `else`, and `Reason`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `not_implemented_op`、`else`、`Reason` 等值。

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
- **EN**: Scheduling decisions  
  **CN**: 调度决策
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发

## Dependencies / 依赖关系
- **Standard library / 标准库**: `hashlib`, `logging`, `collections.abc`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch._inductor.codegen.cutlass.python_evt`, `torch._inductor.utils`, `torch.utils._ordered_set`, `...._dynamo.utils`, `...`, `...codecache`, `...ir`, `...scheduler`, `...utils`, `...virtualized`, `..common`
