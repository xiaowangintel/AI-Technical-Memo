# choices.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/choices.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `Sortable`, `FusionScore`, and `InductorChoices`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `Sortable`、`FusionScore`、`InductorChoices` 等类。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
from __future__ import annotations

import dataclasses
import typing
from typing import Any, TYPE_CHECKING

import sympy

import torch
from torch._inductor.runtime.runtime_utils import next_power_of_2
from torch._inductor.scheduler import MixOrderReduction
from torch.utils._sympy.value_ranges import bound_sympy

from . import config
from .codecache import write_text
from .kernel_inputs import KernelInputs  # noqa: TC001
from .kernel_template_choice import make_ktc_generator
from .metrics import get_metric_table, is_metric_table_enabled
from .runtime.hints import DeviceProperties, ReductionHint
from .scheduler import BaseSchedulerNode, Scheduler, WhyNoFuse
````
- **EN**: Imports dependencies such as `__future__`, `dataclasses`, `typing`, `sympy`, `torch`, `torch._inductor.runtime.runtime_utils`, and `...+9` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里导入了 `__future__`、`dataclasses`、`typing`、`sympy`、`torch`、`torch._inductor.runtime.runtime_utils`、`另有9项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。借助符号表达式来推理索引、形状或代数性质。

### Lines 21-40 / 第 21-40 行
````python
from .select_algorithm import ExternKernelChoice
from .template_heuristics import get_template_heuristic
from .template_heuristics.triton import (
    BaseConfigHeuristic,
    CPUConfigHeuristic,
    CUDAConfigHeuristic,
    MTIAConfigHeuristic,
    ROCmConfigHeuristic,
    XPUConfigHeuristic,
)
from .utils import _use_autotune_backend
from .virtualized import V


if TYPE_CHECKING:
    from collections.abc import Generator
    from functools import partial

    from triton import Config as TritonConfig

````
- **EN**: Imports dependencies such as `.select_algorithm`, `.template_heuristics`, `.template_heuristics.triton`, `.utils`, `.virtualized`, `collections.abc`, and `...+2` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `.select_algorithm`、`.template_heuristics`、`.template_heuristics.triton`、`.utils`、`.virtualized`、`collections.abc`、`另有2项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含分支、循环或上下文管理等控制流。

### Lines 41-60 / 第 41-60 行
````python
    from .codegen.common import KernelTemplate
    from .codegen.simd_kernel_features import SIMDKernelFeatures
    from .codegen.triton import TritonKernel
    from .ir import ChoiceCaller
    from .kernel_template_choice import KernelTemplateChoice

    from torch.utils._ordered_set import OrderedSet  # isort: skip


class Sortable(typing.Protocol):
    """Anything that can be used as a list.sort() key (int/tuple/etc)"""

    def __lt__(self, other: typing.Self) -> bool: ...


@dataclasses.dataclass
class FusionScore:
    template_score: int
    node_type_score: bool
    memory_score: int
````
- **EN**: Imports dependencies such as `.codegen.common`, `.codegen.simd_kernel_features`, `.codegen.triton`, `.ir`, `.kernel_template_choice`, and `torch.utils._ordered_set` for the logic in this range. Introduces class `Sortable`, function `__lt__`, class `FusionScore`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields.
- **CN**: 这里导入了 `.codegen.common`、`.codegen.simd_kernel_features`、`.codegen.triton`、`.ir`、`.kernel_template_choice`、`torch.utils._ordered_set` 等依赖，为后续逻辑提供基础能力。这里定义了类`Sortable`、函数`__lt__`、类`FusionScore`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。

### Lines 61-80 / 第 61-80 行
````python
    buffer_overlap_score: int
    proximity_score: int

    def __lt__(self, other):
        """
        node_type_score has higher priority than memory_score unless
        the memory_score differs too much.

        buffer_overlap_score is prioritized below memory_score so that
        strict global memory savings (exact dep matches) are preferred
        over buffer overlap scoring (same buffer, different indexing).
        """
        threshold = 16
        if self.template_score != other.template_score:
            return self.template_score < other.template_score

        if (
            max(self.memory_score, other.memory_score)
            > min(self.memory_score, other.memory_score) * threshold
        ):
````
- **EN**: Introduces function `__lt__`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `buffer_overlap_score`, `proximity_score`, and `threshold`.
- **CN**: 这里定义了函数`__lt__`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `buffer_overlap_score`、`proximity_score`、`threshold` 等值。

### Lines 81-100 / 第 81-100 行
````python
            return self.memory_score < other.memory_score

        return (
            self.node_type_score,
            self.memory_score,
            self.buffer_overlap_score,
            self.proximity_score,
        ) < (
            other.node_type_score,
            other.memory_score,
            other.buffer_overlap_score,
            other.proximity_score,
        )


class InductorChoices:
    """
    This class contains a collection of default heuristics that affect performance of our generated
    code.  We try to not put correctness requirements in this file.

````
- **EN**: Introduces class `InductorChoices`. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`InductorChoices`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 101-120 / 第 101-120 行
````python
    You can override the choices made here by doing:

            class MyHeuristics(InductorChoices):
                ...

            torch._inductor.virtualized.V.set_choices_handler(MyHeuristics())

    Subclasses used with inductor_choices_class must implement uuid() for
    cache key computation.
    """

    def get_config_heuristics(
        self, device_type: str | None = "cuda"
    ) -> BaseConfigHeuristic:
        if device_type == "cuda":
            if torch.version.hip is None:
                return CUDAConfigHeuristic()
            else:
                return ROCmConfigHeuristic()
        elif device_type == "xpu":
````
- **EN**: Introduces class `MyHeuristics`, function `get_config_heuristics`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`MyHeuristics`、函数`get_config_heuristics`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 121-140 / 第 121-140 行
````python
            return XPUConfigHeuristic()
        elif device_type == "cpu":
            return CPUConfigHeuristic()
        elif device_type == "mtia":
            return MTIAConfigHeuristic()
        else:
            return BaseConfigHeuristic()

    # Conv configs
    def get_conv_configs(
        self, device_type: str | None = "cuda"
    ) -> partial[Generator[TritonConfig, None, None]]:
        conv_heuristics = self.get_config_heuristics(device_type)
        return conv_heuristics.get_conv_configs()

    def get_depthwise_conv_configs(self, device_type: str | None = "cuda") -> list[Any]:
        heuristics = self.get_config_heuristics(device_type)
        return heuristics.get_depthwise_conv_configs()

    # Flex attention configs
````
- **EN**: Introduces function `get_conv_configs`, function `get_depthwise_conv_configs`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_conv_configs`、函数`get_depthwise_conv_configs`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-160 / 第 141-160 行
````python
    # TODO(coconutruben): break out flexattention/decode configs into the new retrieval mechanism
    def get_flex_attention_fwd_configs(
        self, head_dim: int, dtype: torch.dtype, device_type: str | None = "cuda"
    ) -> list[Any]:
        flex_heuristics = self.get_config_heuristics(device_type)
        return flex_heuristics.get_flex_attn_fwd_configs(head_dim, dtype)

    def get_flex_attention_bwd_configs(
        self, head_dim: int, dtype: torch.dtype, device_type: str | None = "cuda"
    ) -> list[Any]:
        flex_heuristics = self.get_config_heuristics(device_type)
        return flex_heuristics.get_flex_attn_bwd_configs(head_dim, dtype)

    def get_flex_decode_configs(
        self, head_dim: int, dtype: torch.dtype, device_type: str | None = "cuda"
    ) -> list[Any]:
        flex_heuristics = self.get_config_heuristics(device_type)
        return flex_heuristics.get_flex_decode_configs(head_dim, dtype)

    def _finalize_template_configs(
````
- **EN**: Introduces function `get_flex_attention_fwd_configs`, function `get_flex_attention_bwd_configs`, function `get_flex_decode_configs`, function `_finalize_template_configs`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_flex_attention_fwd_configs`、函数`get_flex_attention_bwd_configs`、函数`get_flex_decode_configs`、函数`_finalize_template_configs`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 161-180 / 第 161-180 行
````python
        self,
        template_choices: dict[str, Generator[KernelTemplateChoice, None, None]],
        kernel_inputs: KernelInputs,
        templates: list[KernelTemplate | ExternKernelChoice],
        op_name: str,
        kwarg_overrides: dict[str, dict[str, Any]] | None = None,
    ) -> list[KernelTemplateChoice]:
        """
        This method can be subclassed to perform any override/modification of the choices.
        The incoming parameters are cheap (generators), so you can do any overrides without
        incurring too much cost. Override this method to customize the kernel template choices
        before they are converted to ChoiceCaller objects, which is expensive on template codegen.

        The full list of arguments are here to facilitate any overrides you may want to do,
        as they can be used to start from scratch for each template if so desired.

        Args:
            template_choices: Dictionary mapping template UIDs to generators of KernelTemplateChoice objects
            kernel_inputs: MMKernelInputs containing input tensor nodes and matrix indices
            templates: List of template objects (KernelTemplate or ExternKernelChoice) in use
````
- **EN**: Initializes or updates values such as `template_choices`, `kernel_inputs`, `templates`, `op_name`, `kwarg_overrides`, and `Args`. This range continues the implementation of function `InductorChoices._finalize_template_configs`.
- **CN**: 初始化或更新了 `template_choices`、`kernel_inputs`、`templates`、`op_name`、`kwarg_overrides`、`Args` 等值。这一段延续了函数`InductorChoices._finalize_template_configs` 的具体实现。

### Lines 181-200 / 第 181-200 行
````python
            op_name: Operation name (e.g., "bmm", "baddbmm", "addmm")
            kwarg_overrides: Optional dict of kwargs to override for each template heuristic

        Returns:
            Flattened list of KernelTemplateChoice objects across all templates
        """
        choices: list[KernelTemplateChoice] = []
        for choice_gen in template_choices.values():
            choices.extend(choice_gen)
        return choices

    def get_ktc(
        self,
        kernel_inputs: KernelInputs,
        template: KernelTemplate | ExternKernelChoice,
        op_name: str,
        kwarg_overrides: dict[str, Any] | None = None,
    ) -> Generator[KernelTemplateChoice, None, None]:
        """
        Utility to get the KernelTemplateChoice generator for a specific input.
````
- **EN**: Introduces function `get_ktc`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `op_name`, `kwarg_overrides`, `Returns`, `choices`, `kernel_inputs`, and `template`.
- **CN**: 这里定义了函数`get_ktc`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `op_name`、`kwarg_overrides`、`Returns`、`choices`、`kernel_inputs`、`template` 等值。

### Lines 201-220 / 第 201-220 行
````python

        This is a per template/op call, whereas get_template_configs is an op wide call (all templates).
        Consider when overriding/using at which level you need to make decisions
        """
        # Extract device_type from kernel_inputs
        device_type = kernel_inputs.device_type
        assert device_type is not None, "get_ktc requires a valid device type"
        # Extract template_name from the template object
        template_name = template.uid

        # Get the appropriate template-specific heuristic
        heuristic = get_template_heuristic(template_name, device_type, op_name)
        cs = heuristic.get_template_configs(
            kernel_inputs,
            op_name,
        )
        # adjust the kernel inputs to the template-specific heuristic, if needed
        # default here is to just return the kernel_inputs as is
        inputs_val = heuristic.adjust_kernel_inputs(kernel_inputs, op_name)
        extra_kwargs = heuristic.get_extra_kwargs(kernel_inputs, op_name)
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `device_type`, `template_name`, `heuristic`, `cs`, `inputs_val`, and `extra_kwargs`. This range continues the implementation of function `InductorChoices.get_ktc`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `device_type`、`template_name`、`heuristic`、`cs`、`inputs_val`、`extra_kwargs` 等值。这一段延续了函数`InductorChoices.get_ktc` 的具体实现。

### Lines 221-240 / 第 221-240 行
````python
        # Create KernelTemplateChoice generator using the moved function
        overrides = kwarg_overrides or {}
        return make_ktc_generator(
            template=template,
            cs=cs,
            extra_kwargs=extra_kwargs,
            overrides=overrides,
            layout=kernel_inputs.output_layout(),
            inputs=inputs_val,
        )

    def _need_to_fix_layout(
        self,
        adjusted_choices: list[KernelTemplateChoice],
        op_name: str,
    ) -> bool:
        """
        Check if we need to fix the layout instead of keeping it flexible

        Args:
````
- **EN**: Introduces function `_need_to_fix_layout`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `overrides`, `template`, `cs`, `extra_kwargs`, `layout`, `inputs`, and `...+3`.
- **CN**: 这里定义了函数`_need_to_fix_layout`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `overrides`、`template`、`cs`、`extra_kwargs`、`layout`、`inputs`、`另有3项` 等值。

### Lines 241-260 / 第 241-260 行
````python
            ktc: KernelTemplateChoice object

        Returns:
            True if we need to fix the layout, False otherwise
        """
        # TODO: debug and fix
        # NOTE: on mps, we see issues with flexible layouts on baddmm. This check just makes sure
        # that for mps, everything stays as it was before this optimization
        if len(adjusted_choices) > 0:
            if adjusted_choices[0].inputs.device_type == "mps" and op_name not in [
                "mm",
                "addmm",
            ]:
                return True

        # Since the following backends are not using get_mm_configs yet through the singular call,
        if not (config.max_autotune or config.max_autotune_gemm):
            # no danger of using other backends than ATEN
            if not config.max_autotune_allow_flexible_layouts and op_name not in [
                # The historical implementation for mm and addmm allowed had flexible layouts in the
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `ktc`, and `Returns`. This range continues the implementation of function `InductorChoices._need_to_fix_layout`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `ktc`、`Returns` 等值。这一段延续了函数`InductorChoices._need_to_fix_layout` 的具体实现。

### Lines 261-280 / 第 261-280 行
````python
                # not max-autotune world
                "mm",
                "addmm",
            ]:
                # TODO: deprecate this by migrating users to the new behavior
                return True
            return False

        if not config.max_autotune_allow_flexible_layouts:
            # we always need to fix the layout
            return True

        # Since the following backends are not using get_template_configs yet through the singular call,
        # we don't know if they are a valid choice or not. Instead, just skip the optimization
        # defensively.
        # TODO(coconutruben): remove this once CPP,CK,CUTLASS are supported
        if _use_autotune_backend("CUTLASS"):
            return True
        if _use_autotune_backend("CK") or _use_autotune_backend("CKTILE"):
            return True
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `InductorChoices._need_to_fix_layout`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了函数`InductorChoices._need_to_fix_layout` 的具体实现。

### Lines 281-300 / 第 281-300 行
````python
        if _use_autotune_backend("CPP"):
            return True
        return any(
            not isinstance(ktc.template, ExternKernelChoice) for ktc in adjusted_choices
        )

    def get_template_configs(
        self,
        kernel_inputs: KernelInputs,
        templates: list[KernelTemplate | ExternKernelChoice],
        op_name: str,
        kwarg_overrides: dict[str, dict[str, Any]] | None = None,
    ) -> list[ChoiceCaller]:
        """
        Get list of ChoiceCallers for MM templates using template-specific heuristics.

        Args:
            kernel_inputs: MMKernelInputs containing input tensor nodes and matrix indices
            layout: Output layout
            templates: List of template objects (KernelTemplate or ExternKernelChoice)
````
- **EN**: Introduces function `get_template_configs`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `kernel_inputs`, `templates`, `op_name`, `kwarg_overrides`, `Args`, and `layout`.
- **CN**: 这里定义了函数`get_template_configs`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `kernel_inputs`、`templates`、`op_name`、`kwarg_overrides`、`Args`、`layout` 等值。

### Lines 301-320 / 第 301-320 行
````python
            op_name: Operation name (e.g., "bmm", "baddbmm", "addmm", "mm_plus_mm")
            kwarg_overrides: Optional dict of kwargs to override for each template heuristic,
                             indexed by template.uid. These only override the per config kwargs, not the extra kwargs
        Returns:
            List of ChoiceCaller objects from the templates
        """
        if kwarg_overrides is None:
            kwarg_overrides = {}
        input_tensors = kernel_inputs.nodes()
        if len(input_tensors) < 2:
            raise ValueError(f"Need at least 2 input tensors, got {len(input_tensors)}")
        layout = kernel_inputs.output_layout()
        # First pass: Create dict of template.uid to generator of KernelTemplateChoice objects
        template_choices = {}
        for template in templates:
            template_choices[template.uid] = self.get_ktc(
                kernel_inputs,
                template,
                op_name,
                kwarg_overrides.get(template.uid, {}),
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `op_name`, `kwarg_overrides`, `Returns`, `input_tensors`, `layout`, and `template_choices`. This range continues the implementation of function `InductorChoices.get_template_configs`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `op_name`、`kwarg_overrides`、`Returns`、`input_tensors`、`layout`、`template_choices` 等值。这一段延续了函数`InductorChoices.get_template_configs` 的具体实现。

### Lines 321-340 / 第 321-340 行
````python
            )

        # Second pass: Adjust the template choices
        adjusted_choices = self._finalize_template_configs(
            template_choices,
            kernel_inputs,
            templates,
            op_name,
            kwarg_overrides,
        )
        # Layout optimization: if all choices are ExternKernelChoice and layout is FixedLayout, convert to FlexibleLayout
        if self._need_to_fix_layout(adjusted_choices, op_name):
            layout = kernel_inputs.output_layout(flexible=False)
            for ktc in adjusted_choices:
                ktc.layout = layout
                # for good measure, delete the cached ChoiceCaller from the ktc if it existed.
                # ExternKernelChoice are cheap to generate
                if hasattr(ktc, "_choice"):
                    del ktc._choice
        # Third pass: Convert to ChoiceCaller objects
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `adjusted_choices`, and `layout`. This range continues the implementation of function `InductorChoices.get_template_configs`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含分支、循环或上下文管理等控制流。初始化或更新了 `adjusted_choices`、`layout` 等值。这一段延续了函数`InductorChoices.get_template_configs` 的具体实现。

### Lines 341-360 / 第 341-360 行
````python
        return [ktc.choice for ktc in adjusted_choices if ktc.choice is not None]

    def triton_kernel_kwargs(
        self,
        kernel_cls: type[TritonKernel],
        features: SIMDKernelFeatures,
        groups: list[sympy.Expr],
        kernel_kwargs: dict[str, Any],
    ) -> dict[str, Any]:
        """Hook to change the kwargs passed to TritonKernel, used to apply fixed configurations"""
        return kernel_kwargs

    def override_best_choice(
        self,
        best_choice: ChoiceCaller,
        timings: dict[ChoiceCaller, float],
    ) -> ChoiceCaller:
        """Hook to override the autotuning best choice after benchmarking."""
        return best_choice

````
- **EN**: Introduces function `triton_kernel_kwargs`, function `override_best_choice`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `kernel_cls`, `features`, `groups`, `kernel_kwargs`, `best_choice`, and `timings`.
- **CN**: 这里定义了函数`triton_kernel_kwargs`、函数`override_best_choice`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `kernel_cls`、`features`、`groups`、`kernel_kwargs`、`best_choice`、`timings` 等值。

### Lines 361-380 / 第 361-380 行
````python
    def customize_fused_kernel_name(self, fused_name: str, src_code: str) -> str:
        """Hook to transform fused kernel names during codegen"""
        return fused_name

    @staticmethod
    def should_use_cooperative_reduction(
        device: torch.device, numel: sympy.Expr, reduction_numel: sympy.Expr
    ) -> bool:
        """Heuristic to decide if a cooperative reduction should be used."""
        if config.triton.force_cooperative_reductions:
            return True
        if not config.triton.cooperative_reductions or device.type == "cpu":
            return False

        xhint = V.graph.sizevars.optimization_hint(numel, fallback=2)
        if xhint <= 8:
            threshold = 32768 * xhint
        elif xhint <= 16:
            threshold = 2097152
        else:
````
- **EN**: Introduces function `customize_fused_kernel_name`, function `should_use_cooperative_reduction`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里定义了函数`customize_fused_kernel_name`、函数`should_use_cooperative_reduction`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 381-400 / 第 381-400 行
````python
            return False
        # TODO(jansel): should this default on for dynamic shapes?
        # TODO(laith) What if hint(reduction_numel) >= threshold ?
        # shall we compare hints instead
        return V.graph.sizevars.statically_known_geq(reduction_numel, threshold)

    @staticmethod
    def should_use_persistent_reduction(
        features: SIMDKernelFeatures, cooperative_reduction: bool
    ) -> bool:
        """
        Heuristic to decide if a persistent reduction should be used.
        """
        if not config.triton.persistent_reductions:
            return False
        threshold = {
            ReductionHint.INNER: 1024,
        }.get(features.get_reduction_hint(), 64)

        if features.get_reduction_hint() not in (
````
- **EN**: Introduces function `should_use_persistent_reduction`. Applies decorators to register behavior or alter how the following definition is constructed. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`should_use_persistent_reduction`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 401-420 / 第 401-420 行
````python
            ReductionHint.INNER,
            ReductionHint.OUTER_TINY,
        ):
            bounds = bound_sympy(features.reduction_numel)
            lower = bounds.lower
            upper = bounds.upper

            if not all(
                (
                    (isinstance(bound, int) or bound.is_constant())
                    and not torch.utils._sympy.numbers.is_infinite(bound)
                )
                for bound in (lower, upper)
            ):
                return False

            lower = next_power_of_2(int(lower))
            upper = next_power_of_2(int(upper))

            # If we are are coalescing on xblock (not ReductionHint.INNER) and this is not a tiny kernel
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `bounds`, `lower`, and `upper`. This range continues the implementation of function `InductorChoices.should_use_persistent_reduction`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `bounds`、`lower`、`upper` 等值。这一段延续了函数`InductorChoices.should_use_persistent_reduction` 的具体实现。

### Lines 421-440 / 第 421-440 行
````python
            # (not ReductionHint.OUTER_TINY), do not use persistent reduction if it induces tile
            # quantization. Persistent reduction forces rblock == rnumel, if the bounds between lower
            # and upper are large, for the lower values we will be masking off large % of read/writes,
            # when we could expand the coalescing xblock instead.
            if lower != upper:
                return False

        if cooperative_reduction:
            # The RSPLIT of cooperative reductions means each thread block is operating on fewer elements
            # The default fallback will be used if optimizations hint is not provided. The default fallback
            # is >> 32.
            threshold *= 32 // min(
                V.graph.sizevars.optimization_hint(features.numel), 32
            )

        # If multi_kernel is enabled, we do more aggressive persistent reduction.
        # This may result in some persistent reductions slower than the
        # corresponding non-persistent reductions. MultiKernel will do benchmarking
        # to pick the faster one.
        if config.triton.multi_kernel:
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `InductorChoices.should_use_persistent_reduction`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了函数`InductorChoices.should_use_persistent_reduction` 的具体实现。

### Lines 441-460 / 第 441-460 行
````python
            threshold *= 16

        return V.graph.sizevars.statically_known_leq(
            features.reduction_numel, threshold
        )  # type: ignore[arg-types]

    @staticmethod
    def reduction_split_factor(
        device: torch.device,
        reduction_numel_hint: int,
        numel_hint: int,
        inner_reduction: bool,
    ) -> int:
        """Heuristic to decide the RSPLIT used for split reductions.
        When a reduction has a small number of outputs there is not enough parallelism,
        so we will do the reduction in two phases."""
        props = DeviceProperties.create(device)
        num_sm = props.multi_processor_count
        warp_size = props.warp_size if props.warp_size is not None else 32
        max_threads_per_sm = (
````
- **EN**: Introduces function `reduction_split_factor`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`reduction_split_factor`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 461-480 / 第 461-480 行
````python
            props.max_threads_per_multi_processor
            if props.max_threads_per_multi_processor is not None
            else 2048
        )
        min_elements_per_thread = warp_size
        max_elements_per_thread = 512
        threads_per_sm = max_threads_per_sm
        min_elements_per_device = min_elements_per_thread * num_sm * threads_per_sm
        max_elements_per_device = max_elements_per_thread * num_sm * threads_per_sm
        num_warps = 8
        num_threads = warp_size * num_warps

        if inner_reduction:
            # do heuristics that's close to eager mode for split inner reduction
            # we leak reduction autotune configs here, and will need to refactor to avoid this later
            if numel_hint >= 2 * num_sm:  # don't split if there are enough outputs
                return 1
            if reduction_numel_hint <= 8192:
                return 1
            if reduction_numel_hint * numel_hint <= min_elements_per_device:
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `min_elements_per_thread`, `max_elements_per_thread`, `threads_per_sm`, `min_elements_per_device`, `max_elements_per_device`, `num_warps`, and `...+1`. This range continues the implementation of function `InductorChoices.reduction_split_factor`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `min_elements_per_thread`、`max_elements_per_thread`、`threads_per_sm`、`min_elements_per_device`、`max_elements_per_device`、`num_warps`、`另有1项` 等值。这一段延续了函数`InductorChoices.reduction_split_factor` 的具体实现。

### Lines 481-500 / 第 481-500 行
````python
                split_size = min_elements_per_thread
            elif reduction_numel_hint * numel_hint < max_elements_per_device:
                target_blocks = num_sm * threads_per_sm // (2 * num_threads)
                blocks_per_output = (target_blocks + numel_hint - 1) // numel_hint
                tmp_split_size = (
                    reduction_numel_hint + num_threads * blocks_per_output - 1
                ) // (num_threads * blocks_per_output)
                divisors = sympy.divisors(reduction_numel_hint)
                closest = min(divisors, key=lambda x: abs(x - tmp_split_size))
                if abs(closest - tmp_split_size) < 30:
                    # prefer even splits, but never smalle than min_elements_per_thread
                    split_size = max(closest, min_elements_per_thread)
                else:
                    split_size = tmp_split_size
            else:
                divisors = sympy.divisors(reduction_numel_hint)
                closest = min(divisors, key=lambda x: abs(x - max_elements_per_thread))
                if abs(closest - max_elements_per_thread) < 50:
                    # prefer even splits
                    split_size = closest
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `split_size`, `target_blocks`, `blocks_per_output`, `tmp_split_size`, `divisors`, `closest`, and `...+1`. This range continues the implementation of function `InductorChoices.reduction_split_factor`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含分支、循环或上下文管理等控制流。初始化或更新了 `split_size`、`target_blocks`、`blocks_per_output`、`tmp_split_size`、`divisors`、`closest`、`另有1项` 等值。这一段延续了函数`InductorChoices.reduction_split_factor` 的具体实现。

### Lines 501-520 / 第 501-520 行
````python
                else:
                    split_size = max_elements_per_thread
            return (reduction_numel_hint + split_size * num_threads - 1) // (
                split_size * num_threads
            )
        else:
            # TODO the best heuristic currently has XBLOCK (corresponding to numel_hint) 128
            # extend to even smaller number of outputs
            rvals_per_thread = 4  # comes from heuristics, refactor to not leak here
            xvals_per_block = 128
            xblocks = (numel_hint + xvals_per_block - 1) // xvals_per_block
            if reduction_numel_hint * numel_hint < min_elements_per_device:
                split_size = min_elements_per_thread
            elif reduction_numel_hint * numel_hint < max_elements_per_device:
                target_blocks = num_sm * threads_per_sm // (num_threads)
                target_blocks = (target_blocks + xblocks - 1) // xblocks
                tmp_split_size = (
                    reduction_numel_hint + rvals_per_thread * target_blocks - 1
                ) // (rvals_per_thread * target_blocks)
                divisors = sympy.divisors(reduction_numel_hint)
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `split_size`, `rvals_per_thread`, `xvals_per_block`, `xblocks`, `target_blocks`, and `...+2`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`split_size`、`rvals_per_thread`、`xvals_per_block`、`xblocks`、`target_blocks`、`另有2项` 等值。

### Lines 521-540 / 第 521-540 行
````python
                closest = min(divisors, key=lambda x: abs(x - tmp_split_size))
                if abs(tmp_split_size - closest) < 20:
                    split_size = max(closest, min_elements_per_thread)
                else:
                    split_size = tmp_split_size
            else:
                divisors = sympy.divisors(reduction_numel_hint)
                closest = min(divisors, key=lambda x: abs(x - max_elements_per_thread))
                if abs(closest - max_elements_per_thread) < 50:
                    # prefer even splits
                    split_size = closest
                else:
                    split_size = max_elements_per_thread

            return (reduction_numel_hint + rvals_per_thread * split_size - 1) // (
                rvals_per_thread * split_size
            )

    @staticmethod
    def can_fuse(
````
- **EN**: Introduces function `can_fuse`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`can_fuse`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 541-560 / 第 541-560 行
````python
        scheduler: Scheduler,
        node1: BaseSchedulerNode,
        node2: BaseSchedulerNode,
        shared_data_score: int,
    ) -> bool:
        """
        Heuristics to prevent fusion applied to both horizontal and vertical fusions.  Heuristics here should not
        be needed for correctness and tweaking them may yield additional performance.

        See also some related heuristics that can be changed via config:
            - config.triton.tiling_prevents_pointwise_fusion
            - config.triton.tiling_prevents_reduction_fusion
            - config.aggressive_fusion (will cause this function to be called more times)
        """
        if shared_data_score == 0 and (
            not config.aggressive_fusion or node1.is_reduction() or node2.is_reduction()
        ):
            if is_metric_table_enabled("fusion_failure_due_to_indexing_mismatch"):
                common_buf_names: OrderedSet[str] = (
                    node1.read_writes.buffer_names() & node2.read_writes.buffer_names()
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `scheduler`, `node1`, `node2`, `shared_data_score`, and `common_buf_names`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `scheduler`、`node1`、`node2`、`shared_data_score`、`common_buf_names` 等值。

### Lines 561-580 / 第 561-580 行
````python
                )
                if len(common_buf_names) > 0:
                    get_metric_table("fusion_failure_due_to_indexing_mismatch").add_row(
                        # pyrefly: ignore [bad-argument-type]
                        lambda: {
                            "pre_grad_graph_id": V.graph.graph_id,
                            "post_grad_graph_id": V.graph.post_grad_graph_id,
                            "node1_name": node1.get_name(),
                            "node2_name": node2.get_name(),
                            "node1_debug_str": write_text(node1.debug_str()),
                            "node2_debug_str": write_text(node2.debug_str()),
                            "common_buffer_names": list(common_buf_names),  # type: ignore[dict-item]
                            "failure_reason": scheduler.decide_fusion_fail_reason(
                                node1, node2, common_buf_names
                            ),
                        }
                    )

                    WhyNoFuse(node1, node2)("no shared data due to indexing mismatch")
                    return False
````
- **EN**: Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `lambda`.
- **CN**: 这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `lambda` 等值。

### Lines 581-600 / 第 581-600 行
````python
            WhyNoFuse(node1, node2)("no shared data")
            return False  # heuristic not needed for correctness

        if (
            not node1.is_foreach()
            and not node2.is_foreach()
            and len(node1.get_nodes()) + len(node2.get_nodes()) > config.max_fusion_size
        ):
            WhyNoFuse(node1, node2)("exceeds max fusion")
            return False  # heuristic not needed for correctness

        if scheduler.can_fusion_increase_peak_memory(node1, node2):
            WhyNoFuse(node1, node2)("Fusion will increase peak memory")
            return False

        if (
            config.max_fusion_unique_io_buffers is not None
            and scheduler.fusion_prevent_too_many_reads_and_writes(
                node1,
                node2,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `InductorChoices.can_fuse`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了函数`InductorChoices.can_fuse` 的具体实现。

### Lines 601-620 / 第 601-620 行
````python
                config.max_fusion_unique_io_buffers,
            )
        ):
            WhyNoFuse(node1, node2)("fusion_prevent_too_many_reads_and_writes")
            return False

        return True

    @staticmethod
    def can_fuse_vertical(
        scheduler: Scheduler,
        node1: BaseSchedulerNode,
        node2: BaseSchedulerNode,
        shared_data_score: int,
    ) -> bool:
        """Hook for heuristics to prevent vertical (producer/consumer) fusions"""
        return True

    @staticmethod
    def can_fuse_horizontal(
````
- **EN**: Introduces function `can_fuse_vertical`, function `can_fuse_horizontal`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `scheduler`, `node1`, `node2`, and `shared_data_score`.
- **CN**: 这里定义了函数`can_fuse_vertical`、函数`can_fuse_horizontal`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `scheduler`、`node1`、`node2`、`shared_data_score` 等值。

### Lines 621-640 / 第 621-640 行
````python
        scheduler: Scheduler,
        node1: BaseSchedulerNode,
        node2: BaseSchedulerNode,
        shared_data_score: int,
    ) -> bool:
        """Hook for heuristics to prevent horizontal (consumer/consumer) fusions"""
        if MixOrderReduction.can_fuse(node1, node2):
            # For mix order reduction, we disregard shared data or
            # distance.
            return True
        if shared_data_score < config.score_fusion_memory_threshold:
            WhyNoFuse(node1, node2)("score_fusion_memory_threshold")
            return False
        if scheduler.are_long_distant_nodes(node1, node2):
            WhyNoFuse(node1, node2)(
                "Nodes are too far away. Fusing them may increase peak memory."
            )
            return False
        return True

````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `scheduler`, `node1`, `node2`, and `shared_data_score`. This range continues the implementation of function `InductorChoices.can_fuse_horizontal`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `scheduler`、`node1`、`node2`、`shared_data_score` 等值。这一段延续了函数`InductorChoices.can_fuse_horizontal` 的具体实现。

### Lines 641-660 / 第 641-660 行
````python
    @staticmethod
    def score_fusion(
        scheduler: Scheduler,
        node1: BaseSchedulerNode,
        node2: BaseSchedulerNode,
    ) -> Sortable:
        """
        Assign a score (higher comes first) to the fusion of node1 and node2.
        When different fusions conflict with each other, this is the way we
        decide what order to run them in.

        Our current score is based on:
        - The type of fusion (template/reduction/etc)
        - Estimate of the saved memory operations
        - Fusions closer together in original graph order
        """

        memory_score, buffer_overlap_score, is_mix_order_reduction = (
            scheduler.score_fusion_memory(
                node1, node2, return_is_mix_order_reduction=True
````
- **EN**: Introduces function `score_fusion`. Applies decorators to register behavior or alter how the following definition is constructed. Initializes or updates values such as `scheduler`, `node1`, and `node2`.
- **CN**: 这里定义了函数`score_fusion`。使用装饰器来注册行为，或改变后续定义的构造方式。初始化或更新了 `scheduler`、`node1`、`node2` 等值。

### Lines 661-680 / 第 661-680 行
````python
            )
        )
        proximity_score = -max(
            abs(node1.min_order - node2.max_order),
            abs(node2.min_order - node1.max_order),
        )

        # prologue fusion always last
        if node2.is_template():
            template_score = 0
        else:
            template_score = 1 + (
                (node1.is_template() == config.epilogue_fusion_first)
                and memory_score > 0
            )

        type_score = node1.is_reduction() == node2.is_reduction() and memory_score > 0

        return FusionScore(
            template_score,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `proximity_score`, `template_score`, `else`, and `type_score`. This range continues the implementation of function `InductorChoices.score_fusion`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `proximity_score`、`template_score`、`else`、`type_score` 等值。这一段延续了函数`InductorChoices.score_fusion` 的具体实现。

### Lines 681-685 / 第 681-685 行
````python
            type_score,
            memory_score,
            buffer_overlap_score,
            proximity_score,
        )
````
- **EN**: This range continues the implementation of function `InductorChoices.score_fusion`.
- **CN**: 这一段延续了函数`InductorChoices.score_fusion` 的具体实现。

## Key Concepts / 关键概念
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
- **EN**: Scheduling decisions  
  **CN**: 调度决策
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Device capability modeling  
  **CN**: 设备能力建模

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`, `typing`, `collections.abc`, `functools`
- **Third-party / 第三方**: `sympy`, `triton`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor.runtime.runtime_utils`, `torch._inductor.scheduler`, `torch.utils._sympy.value_ranges`, `.`, `.codecache`, `.kernel_inputs`, `.kernel_template_choice`, `.metrics`, `.runtime.hints`, `.scheduler`, `.select_algorithm`, `.template_heuristics`, `.template_heuristics.triton`, `.utils`, `.virtualized`, `.codegen.common`, `.codegen.simd_kernel_features`, `.codegen.triton`, `.ir`, `...+1`
