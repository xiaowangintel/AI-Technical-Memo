# xpu_combined_scheduling.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/xpu/xpu_combined_scheduling.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `XPUCombinedScheduling`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `XPUCombinedScheduling` 等类。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
# mypy: allow-untyped-defs
from __future__ import annotations

from typing import Any, TYPE_CHECKING

from torch._inductor.scheduler import (
    BaseSchedulerNode,
    BaseScheduling,
    FusedSchedulerNode,
    Scheduler,
    SchedulerNode,
)

from ..cutlass.scheduling import CUTLASSScheduling
````
- **EN**: Imports dependencies such as `__future__`, `typing`, `torch._inductor.scheduler`, and `..cutlass.scheduling` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis.
- **CN**: 这里导入了 `__future__`、`typing`、`torch._inductor.scheduler`、`..cutlass.scheduling` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。

### Lines 15-28 / 第 15-28 行
````python
from ..triton import TritonScheduling


if TYPE_CHECKING:
    from collections.abc import Sequence
    from typing import TypeAlias

    from sympy import Expr

    import torch
    from torch.utils._ordered_set import OrderedSet

    from ..common import BackendFeature

````
- **EN**: Imports dependencies such as `..triton`, `collections.abc`, `typing`, `sympy`, `torch`, `torch.utils._ordered_set`, and `...+1` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里导入了 `..triton`、`collections.abc`、`typing`、`sympy`、`torch`、`torch.utils._ordered_set`、`另有1项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 29-42 / 第 29-42 行
````python
    _IntLike: TypeAlias = int | Expr


class XPUCombinedScheduling(BaseScheduling):
    """
    Scheduler for XPU Kernels, which delegates calls as appropriate
    to the SYCL-C++ and Triton Schedulers, which both work for XPU devices
    and use a unified-wrapper for codegen.

    If Scheduling code needs to be specialized for the case of mixed Triton / SYCL C++ code,
    this would also be the place to do it.
    """

    def __init__(self, scheduler: Scheduler | None) -> None:
````
- **EN**: Introduces class `XPUCombinedScheduling`, function `__init__`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Initializes or updates values such as `_IntLike`.
- **CN**: 这里定义了类`XPUCombinedScheduling`、函数`__init__`。借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。初始化或更新了 `_IntLike` 等值。

### Lines 43-56 / 第 43-56 行
````python
        super().__init__(scheduler)
        self._triton_scheduling = TritonScheduling(scheduler)
        self._cutlass_scheduling = CUTLASSScheduling(scheduler)

    def get_backend_features(self, device: torch.device) -> OrderedSet[BackendFeature]:
        return self._triton_scheduling.get_backend_features(device)

    def choose_node_backend(self, node: BaseSchedulerNode) -> BaseScheduling:
        if self._cutlass_scheduling.is_cutlass_template(node):
            return self._cutlass_scheduling
        return self._triton_scheduling

    def can_fuse_vertical(
        self, node1: BaseSchedulerNode, node2: BaseSchedulerNode
````
- **EN**: Introduces function `get_backend_features`, function `choose_node_backend`, function `can_fuse_vertical`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_backend_features`、函数`choose_node_backend`、函数`can_fuse_vertical`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 57-70 / 第 57-70 行
````python
    ) -> bool:
        if self._cutlass_scheduling.can_fuse_vertical(node1, node2):
            return True
        elif self._cutlass_scheduling.is_cutlass_template(
            node1
        ) or self._cutlass_scheduling.is_cutlass_template(node2):
            return False
        return self._triton_scheduling.can_fuse_vertical(node1, node2)

    def can_fuse_horizontal(
        self, node1: BaseSchedulerNode, node2: BaseSchedulerNode
    ) -> bool:
        for node in (node1, node2):
            if self._cutlass_scheduling.is_cutlass_template(node):
````
- **EN**: Introduces function `can_fuse_horizontal`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`can_fuse_horizontal`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 71-84 / 第 71-84 行
````python
                return self._cutlass_scheduling.can_fuse_horizontal(
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
        epilogue_nodes: Sequence[BaseSchedulerNode],
````
- **EN**: Introduces function `group_fn`, function `codegen_template`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `template_node`, and `epilogue_nodes`.
- **CN**: 这里定义了函数`group_fn`、函数`codegen_template`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `template_node`、`epilogue_nodes` 等值。

### Lines 85-98 / 第 85-98 行
````python
        prologue_nodes: Sequence[BaseSchedulerNode],
    ) -> str | None:
        if self._cutlass_scheduling.is_cutlass_template(template_node):
            assert not prologue_nodes
            return self._cutlass_scheduling.codegen_template(
                template_node, epilogue_nodes, prologue_nodes
            )
        else:
            return self._triton_scheduling.codegen_template(
                template_node, epilogue_nodes, prologue_nodes
            )

    def codegen_mix_order_reduction(self, node):
        return self._triton_scheduling.codegen_mix_order_reduction(node)
````
- **EN**: Introduces function `codegen_mix_order_reduction`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `prologue_nodes`, and `else`.
- **CN**: 这里定义了函数`codegen_mix_order_reduction`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `prologue_nodes`、`else` 等值。

### Lines 99-112 / 第 99-112 行
````python

    def codegen_node(self, node: FusedSchedulerNode | SchedulerNode) -> None:
        return self._triton_scheduling.codegen_node(node)

    def codegen_sync(self) -> None:
        return self._triton_scheduling.codegen_sync()

    def flush(self) -> None:
        return self._triton_scheduling.flush()

    def codegen_combo_kernel(self, *args: Any, **kwargs: Any) -> None:
        return self._triton_scheduling.codegen_combo_kernel(*args, **kwargs)

    def benchmark_fused_nodes(
````
- **EN**: Introduces function `codegen_node`, function `codegen_sync`, function `flush`, function `codegen_combo_kernel`, function `benchmark_fused_nodes`. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`codegen_node`、函数`codegen_sync`、函数`flush`、函数`codegen_combo_kernel`、函数`benchmark_fused_nodes`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 113-126 / 第 113-126 行
````python
        self, nodes: Sequence[BaseSchedulerNode]
    ) -> tuple[float, str]:
        return self._triton_scheduling.benchmark_fused_nodes(nodes)

    def benchmark_codegened_module(self, module):
        return self._triton_scheduling.benchmark_codegened_module(module)

    def generate_kernel_code_from_nodes(
        self,
        nodes: Sequence[Any],
        benchmark_kernel: bool = False,
        hint_override: int | None = None,
    ) -> str:
        return self._triton_scheduling.generate_kernel_code_from_nodes(
````
- **EN**: Introduces function `benchmark_codegened_module`, function `generate_kernel_code_from_nodes`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `nodes`, `benchmark_kernel`, and `hint_override`.
- **CN**: 这里定义了函数`benchmark_codegened_module`、函数`generate_kernel_code_from_nodes`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `nodes`、`benchmark_kernel`、`hint_override` 等值。

### Lines 127-135 / 第 127-135 行
````python
            nodes, benchmark_kernel, hint_override=hint_override
        )

    def benchmark_combo_kernel(
        self, node_list: Sequence[BaseSchedulerNode], node_benchmark_results
    ) -> tuple[float, float, list[str | None]]:
        return self._triton_scheduling.benchmark_combo_kernel(
            node_list, node_benchmark_results
        )
````
- **EN**: Introduces function `benchmark_combo_kernel`. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`benchmark_combo_kernel`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

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
- **EN**: Primary classes: `XPUCombinedScheduling`  
  **CN**: 主要类：`XPUCombinedScheduling`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `typing`, `collections.abc`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch._inductor.scheduler`, `..cutlass.scheduling`, `..triton`, `torch`, `torch.utils._ordered_set`, `..common`
