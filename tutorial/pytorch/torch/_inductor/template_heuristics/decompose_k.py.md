# decompose_k.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/template_heuristics/decompose_k.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `EmptyDecomposeKConfigHeuristics`, and `DecomposeKConfigHeuristics`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `EmptyDecomposeKConfigHeuristics`、`DecomposeKConfigHeuristics` 等类。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10 / 第 1-10 行
````python
from __future__ import annotations

from typing import Any, TYPE_CHECKING

import sympy

from ..ir import get_free_symbols
from ..kernel.mm import decompose_k_subgraph_template
from ..kernel_inputs import KernelInputs, MMKernelInputs
from ..utils import get_k_splits
````
- **EN**: Imports dependencies such as `__future__`, `typing`, `sympy`, `..ir`, `..kernel.mm`, `..kernel_inputs`, and `...+1` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里导入了 `__future__`、`typing`、`sympy`、`..ir`、`..kernel.mm`、`..kernel_inputs`、`另有1项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。借助符号表达式来推理索引、形状或代数性质。

### Lines 11-20 / 第 11-20 行
````python
from ..virtualized import V
from .base import TemplateConfigHeuristics
from .gemm import GemmMaxAutotuneTemplateConfigHeuristics
from .registry import register_template_heuristic


if TYPE_CHECKING:
    from collections.abc import Generator


````
- **EN**: Imports dependencies such as `..virtualized`, `.base`, `.gemm`, `.registry`, and `collections.abc` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `..virtualized`、`.base`、`.gemm`、`.registry`、`collections.abc` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。

### Lines 21-30 / 第 21-30 行
````python
@register_template_heuristic(decompose_k_subgraph_template.uid, None, op_name="mm")
class EmptyDecomposeKConfigHeuristics(TemplateConfigHeuristics):
    """empty heuristics to skip decompose k on anything not cuda"""


@register_template_heuristic(
    decompose_k_subgraph_template.uid,
    "xpu",
    op_name="mm",
)
````
- **EN**: Introduces class `EmptyDecomposeKConfigHeuristics`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了类`EmptyDecomposeKConfigHeuristics`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 31-40 / 第 31-40 行
````python
# Register on CUDA (both NVIDIA and ROCm/HIP)
# Runtime enablement is controlled by config.triton.num_decompose_k_splits (0 disables)
@register_template_heuristic(
    decompose_k_subgraph_template.uid,
    "cuda",
    op_name="mm",
)
# TODO(coconutruben): enable decompose k on other devices (xpu, cpu, mps, mtia)
# by either adding specific register_template_heuristic tags, or setting the
# device to None (enabled on all devices)
````
- **EN**: Applies decorators to register behavior or alter how the following definition is constructed. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 使用装饰器来注册行为，或改变后续定义的构造方式。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 41-50 / 第 41-50 行
````python
class DecomposeKConfigHeuristics(GemmMaxAutotuneTemplateConfigHeuristics):
    def _get_template_configs_impl(
        self,
        kernel_inputs: KernelInputs,
        op_name: str,
    ) -> Generator[dict[str, Any], None, None]:
        """
        Get all the valid k_splits for the given m, n, k.
        """
        assert isinstance(kernel_inputs, MMKernelInputs), (
````
- **EN**: Introduces class `DecomposeKConfigHeuristics`, function `_get_template_configs_impl`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `kernel_inputs`, and `op_name`.
- **CN**: 这里定义了类`DecomposeKConfigHeuristics`、函数`_get_template_configs_impl`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `kernel_inputs`、`op_name` 等值。

### Lines 51-60 / 第 51-60 行
````python
            f"{self.__class__.__name__} requires MMKernelInputs"
        )

        # Check for unbacked symbols - if found, yield nothing
        unbacked_symbols = any(
            len(get_free_symbols(itr, unbacked_only=True)) > 0
            for itr in (
                *kernel_inputs.shapes_symbolic(),
                *kernel_inputs.strides_symbolic(),
            )
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `unbacked_symbols`. This range continues the implementation of function `DecomposeKConfigHeuristics._get_template_configs_impl`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `unbacked_symbols` 等值。这一段延续了函数`DecomposeKConfigHeuristics._get_template_configs_impl` 的具体实现。

### Lines 61-70 / 第 61-70 行
````python
        )
        if unbacked_symbols:
            return

        m, n, k = kernel_inputs.mnk_symbolic()
        k_splits = get_k_splits(m, n, k)
        for k_split in k_splits:
            if not V.graph.sizevars.statically_known_true(
                sympy.Eq(sympy.Mod(k, k_split), 0)
            ):
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `k_splits`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `k_splits` 等值。

### Lines 71-72 / 第 71-72 行
````python
                continue
            yield {"k_split": k_split}
````
- **EN**: Includes returns or checks that define the contract of this code path. This range continues the implementation of function `DecomposeKConfigHeuristics._get_template_configs_impl`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。这一段延续了函数`DecomposeKConfigHeuristics._get_template_configs_impl` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Autotuning support  
  **CN**: 自动调优支持
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Primary classes: `EmptyDecomposeKConfigHeuristics`, and `DecomposeKConfigHeuristics`  
  **CN**: 主要类：`EmptyDecomposeKConfigHeuristics`、`DecomposeKConfigHeuristics`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `typing`, `collections.abc`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `..ir`, `..kernel.mm`, `..kernel_inputs`, `..utils`, `..virtualized`, `.base`, `.gemm`, `.registry`
