# cuda_combined_scheduling.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/cuda_combined_scheduling.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `CUDACombinedScheduling`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `CUDACombinedScheduling` 等类。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
# mypy: allow-untyped-defs
from __future__ import annotations

from typing import Any, TYPE_CHECKING

from ..scheduler import (
    BaseSchedulerNode,
    BaseScheduling,
    FusedSchedulerNode,
    Scheduler,
    SchedulerNode,
)
from .cutedsl.cutedsl_scheduling import CuteDSLScheduling
from .cutlass.scheduling import CUTLASSScheduling
````
- **EN**: Imports dependencies such as `__future__`, `typing`, `..scheduler`, `.cutedsl.cutedsl_scheduling`, and `.cutlass.scheduling` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis.
- **CN**: 这里导入了 `__future__`、`typing`、`..scheduler`、`.cutedsl.cutedsl_scheduling`、`.cutlass.scheduling` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。

### Lines 15-28 / 第 15-28 行
````python
from .nv_universal_gemm.nv_universal_gemm_scheduling import NVUniversalGemmScheduling
from .rocm.rocm_cpp_scheduling import ROCmCPPScheduling
from .triton import TritonScheduling


if TYPE_CHECKING:
    from collections.abc import Sequence
    from typing import TypeAlias

    from sympy import Expr

    import torch
    from torch.utils._ordered_set import OrderedSet

````
- **EN**: Imports dependencies such as `.nv_universal_gemm.nv_universal_gemm_scheduling`, `.rocm.rocm_cpp_scheduling`, `.triton`, `collections.abc`, `typing`, `sympy`, and `...+2` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里导入了 `.nv_universal_gemm.nv_universal_gemm_scheduling`、`.rocm.rocm_cpp_scheduling`、`.triton`、`collections.abc`、`typing`、`sympy`、`另有2项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 29-42 / 第 29-42 行
````python
    from .common import BackendFeature

    _IntLike: TypeAlias = int | Expr


class CUDACombinedScheduling(BaseScheduling):
    """
    Scheduler for CUDA Kernels, which delegates calls as appropriate
    to the CUDA-C++ and Triton Schedulers, which both work for CUDA devices
    and use a unified-wrapper for codegen.

    If Scheduling code needs to be specialized for the case of mixed Triton / CUDA C++ code,
    this would also be the place to do it.
    """
````
- **EN**: Imports dependencies such as `.common` for the logic in this range. Introduces class `CUDACombinedScheduling`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里导入了 `.common` 等依赖，为后续逻辑提供基础能力。这里定义了类`CUDACombinedScheduling`。借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 43-56 / 第 43-56 行
````python

    def __init__(self, scheduler: Scheduler | None) -> None:
        super().__init__(scheduler)
        self._triton_scheduling = TritonScheduling(scheduler)
        self._cutlass_scheduling = CUTLASSScheduling(scheduler)
        self._rocm_cpp_scheduling = ROCmCPPScheduling(scheduler)
        self._cutedsl_scheduling = CuteDSLScheduling(scheduler)
        self._nv_universal_gemm_scheduling = NVUniversalGemmScheduling(scheduler)

    def get_backend_features(self, device: torch.device) -> OrderedSet[BackendFeature]:
        return self._triton_scheduling.get_backend_features(device)

    def choose_node_backend(self, node: BaseSchedulerNode) -> BaseScheduling:
        if self._cutlass_scheduling.is_cutlass_template(node):
````
- **EN**: Introduces function `__init__`, function `get_backend_features`, function `choose_node_backend`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`__init__`、函数`get_backend_features`、函数`choose_node_backend`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 57-70 / 第 57-70 行
````python
            return self._cutlass_scheduling
        if self._rocm_cpp_scheduling.is_rocm_cpp_template(node):
            return self._rocm_cpp_scheduling
        if self._cutedsl_scheduling.is_cutedsl_template(node):
            return self._cutedsl_scheduling
        if self._nv_universal_gemm_scheduling.is_nv_universal_gemm_template(node):
            return self._nv_universal_gemm_scheduling
        return self._triton_scheduling

    def can_fuse_vertical(
        self, node1: BaseSchedulerNode, node2: BaseSchedulerNode
    ) -> bool:
        if self._cutlass_scheduling.can_fuse_vertical(node1, node2):
            return True
````
- **EN**: Introduces function `can_fuse_vertical`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`can_fuse_vertical`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 71-84 / 第 71-84 行
````python
        elif self._cutlass_scheduling.is_cutlass_template(
            node1
        ) or self._cutlass_scheduling.is_cutlass_template(node2):
            return False
        # CuteDSL doesn't support vertical fusion currently
        elif self._cutedsl_scheduling.is_cutedsl_template(
            node1
        ) or self._cutedsl_scheduling.is_cutedsl_template(node2):
            return False
        # NVIDIA Universal GEMM doesn't support vertical fusion currently
        elif self._nv_universal_gemm_scheduling.is_nv_universal_gemm_template(
            node1
        ) or self._nv_universal_gemm_scheduling.is_nv_universal_gemm_template(node2):
            return False
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `CUDACombinedScheduling.can_fuse_vertical`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了函数`CUDACombinedScheduling.can_fuse_vertical` 的具体实现。

### Lines 85-98 / 第 85-98 行
````python
        return self._triton_scheduling.can_fuse_vertical(node1, node2)

    def can_fuse_horizontal(
        self, node1: BaseSchedulerNode, node2: BaseSchedulerNode
    ) -> bool:
        for node in (node1, node2):
            if self._cutlass_scheduling.is_cutlass_template(node):
                return self._cutlass_scheduling.can_fuse_horizontal(
                    node1, node2
                )  # always False at the moment
            if self._cutedsl_scheduling.is_cutedsl_template(node):
                return self._cutedsl_scheduling.can_fuse_horizontal(
                    node1, node2
                )  # always False at the moment
````
- **EN**: Introduces function `can_fuse_horizontal`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`can_fuse_horizontal`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 99-112 / 第 99-112 行
````python
            if self._nv_universal_gemm_scheduling.is_nv_universal_gemm_template(node):
                return self._nv_universal_gemm_scheduling.can_fuse_horizontal(
                    node1, node2
                )  # always False at the moment
        return self._triton_scheduling.can_fuse_horizontal(node1, node2)

    def group_fn(
        self, sizes: Sequence[Sequence[_IntLike]]
    ) -> tuple[tuple[_IntLike, ...], ...]:
        return self._triton_scheduling.group_fn(sizes)

    def codegen_template(
        self,
        template_node: BaseSchedulerNode,
````
- **EN**: Introduces function `group_fn`, function `codegen_template`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `template_node`.
- **CN**: 这里定义了函数`group_fn`、函数`codegen_template`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `template_node` 等值。

### Lines 113-126 / 第 113-126 行
````python
        epilogue_nodes: Sequence[BaseSchedulerNode],
        prologue_nodes: Sequence[BaseSchedulerNode],
    ) -> str | None:
        if self._cutlass_scheduling.is_cutlass_template(template_node):
            assert not prologue_nodes
            return self._cutlass_scheduling.codegen_template(
                template_node, epilogue_nodes, prologue_nodes
            )
        elif self._rocm_cpp_scheduling.is_rocm_cpp_template(template_node):
            assert not epilogue_nodes
            assert not prologue_nodes
            return self._rocm_cpp_scheduling.codegen_template(
                template_node, epilogue_nodes, prologue_nodes
            )
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `epilogue_nodes`, and `prologue_nodes`. This range continues the implementation of function `CUDACombinedScheduling.codegen_template`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `epilogue_nodes`、`prologue_nodes` 等值。这一段延续了函数`CUDACombinedScheduling.codegen_template` 的具体实现。

### Lines 127-140 / 第 127-140 行
````python
        elif self._cutedsl_scheduling.is_cutedsl_template(template_node):
            # TODO remove this when we add epilogue support
            assert not epilogue_nodes
            assert not prologue_nodes
            return self._cutedsl_scheduling.codegen_template(
                template_node, epilogue_nodes, prologue_nodes
            )
        elif self._nv_universal_gemm_scheduling.is_nv_universal_gemm_template(
            template_node
        ):
            # NVIDIA Universal GEMM doesn't support epilogue/prologue fusion yet
            assert not epilogue_nodes
            assert not prologue_nodes
            return self._nv_universal_gemm_scheduling.codegen_template(
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `CUDACombinedScheduling.codegen_template`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了函数`CUDACombinedScheduling.codegen_template` 的具体实现。

### Lines 141-154 / 第 141-154 行
````python
                template_node, epilogue_nodes, prologue_nodes
            )
        else:
            return self._triton_scheduling.codegen_template(
                template_node, epilogue_nodes, prologue_nodes
            )

    def codegen_mix_order_reduction(self, node):
        return self._triton_scheduling.codegen_mix_order_reduction(node)

    def codegen_node(self, node: FusedSchedulerNode | SchedulerNode) -> None:
        return self._triton_scheduling.codegen_node(node)

    def codegen_sync(self) -> None:
````
- **EN**: Introduces function `codegen_mix_order_reduction`, function `codegen_node`, function `codegen_sync`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`.
- **CN**: 这里定义了函数`codegen_mix_order_reduction`、函数`codegen_node`、函数`codegen_sync`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。

### Lines 155-168 / 第 155-168 行
````python
        return self._triton_scheduling.codegen_sync()

    def flush(self) -> None:
        return self._triton_scheduling.flush()

    def codegen_combo_kernel(self, *args: Any, **kwargs: Any) -> None:
        return self._triton_scheduling.codegen_combo_kernel(*args, **kwargs)

    def benchmark_fused_nodes(
        self, nodes: Sequence[BaseSchedulerNode]
    ) -> tuple[float, str]:
        return self._triton_scheduling.benchmark_fused_nodes(nodes)

    def benchmark_codegened_module(self, module):
````
- **EN**: Introduces function `flush`, function `codegen_combo_kernel`, function `benchmark_fused_nodes`, function `benchmark_codegened_module`. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`flush`、函数`codegen_combo_kernel`、函数`benchmark_fused_nodes`、函数`benchmark_codegened_module`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 169-182 / 第 169-182 行
````python
        return self._triton_scheduling.benchmark_codegened_module(module)

    def generate_kernel_code_from_nodes(
        self,
        nodes: Sequence[Any],
        benchmark_kernel: bool = False,
        hint_override: int | None = None,
    ) -> str:
        return self._triton_scheduling.generate_kernel_code_from_nodes(
            nodes, benchmark_kernel, hint_override=hint_override
        )

    def benchmark_combo_kernel(
        self, node_list: Sequence[BaseSchedulerNode], node_benchmark_results
````
- **EN**: Introduces function `generate_kernel_code_from_nodes`, function `benchmark_combo_kernel`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `nodes`, `benchmark_kernel`, and `hint_override`.
- **CN**: 这里定义了函数`generate_kernel_code_from_nodes`、函数`benchmark_combo_kernel`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `nodes`、`benchmark_kernel`、`hint_override` 等值。

### Lines 183-186 / 第 183-186 行
````python
    ) -> tuple[float, float, list[str | None]]:
        return self._triton_scheduling.benchmark_combo_kernel(
            node_list, node_benchmark_results
        )
````
- **EN**: Includes returns or checks that define the contract of this code path. This range continues the implementation of function `CUDACombinedScheduling.benchmark_combo_kernel`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。这一段延续了函数`CUDACombinedScheduling.benchmark_combo_kernel` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Scheduling decisions  
  **CN**: 调度决策
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Primary classes: `CUDACombinedScheduling`  
  **CN**: 主要类：`CUDACombinedScheduling`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `typing`, `collections.abc`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `..scheduler`, `.cutedsl.cutedsl_scheduling`, `.cutlass.scheduling`, `.nv_universal_gemm.nv_universal_gemm_scheduling`, `.rocm.rocm_cpp_scheduling`, `.triton`, `torch`, `torch.utils._ordered_set`, `.common`
