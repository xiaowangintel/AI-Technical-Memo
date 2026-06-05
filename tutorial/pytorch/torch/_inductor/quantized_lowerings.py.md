# quantized_lowerings.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/quantized_lowerings.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It exposes functions such as `register_quantized_ops`, and `register_woq_mm_ops`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。同时提供 `register_quantized_ops`、`register_woq_mm_ops` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
import logging
from typing import Any

import torch
from torch._inductor.kernel.mm_common import mm_args

from . import config, lowering
from .codegen.cpp_gemm_template import CppGemmTemplate, CppWoqInt4GemmTemplate
from .codegen.cpp_utils import create_epilogue_with_attr
from .lowering import expand, register_lowering
from .mkldnn_ir import WeightInt4PackMatmul
from .select_algorithm import (
    autotune_select_algorithm,
    ExternKernelChoice,
````
- **EN**: Imports dependencies such as `logging`, `typing`, `torch`, `torch._inductor.kernel.mm_common`, `.`, `.codegen.cpp_gemm_template`, and `...+4` for the logic in this range. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里导入了 `logging`、`typing`、`torch`、`torch._inductor.kernel.mm_common`、`.`、`.codegen.cpp_gemm_template`、`另有4项` 等依赖，为后续逻辑提供基础能力。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 15-28 / 第 15-28 行
````python
    realize_inputs,
)
from .utils import use_aten_gemm_kernels, use_cpp_gemm_template
from .virtualized import V


log = logging.getLogger(__name__)

aten__weight_int8pack_mm = ExternKernelChoice(
    torch._weight_int8pack_mm, "at::_weight_int8pack_mm", has_out_variant=False
)

aten__weight_int4pack_mm_cpu = ExternKernelChoice(
    torch.ops.quantized.int4mm_packed_weight_cpu,
````
- **EN**: Imports dependencies such as `.utils`, and `.virtualized` for the logic in this range. Initializes or updates values such as `log`, `aten__weight_int8pack_mm`, and `aten__weight_int4pack_mm_cpu`.
- **CN**: 这里导入了 `.utils`、`.virtualized` 等依赖，为后续逻辑提供基础能力。初始化或更新了 `log`、`aten__weight_int8pack_mm`、`aten__weight_int4pack_mm_cpu` 等值。

### Lines 29-42 / 第 29-42 行
````python
    "at::native::_weight_int4pack_mm_cpu_tensor",
    has_out_variant=False,
    kernel_creator=WeightInt4PackMatmul.create,
)

quantized = torch.ops.quantized
_quantized = torch.ops._quantized
aten = torch.ops.aten


def register_quantized_ops() -> None:
    lowering.add_needs_realized_inputs(
        [
            quantized.max_pool2d,
````
- **EN**: Introduces function `register_quantized_ops`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `has_out_variant`, `kernel_creator`, `quantized`, `_quantized`, and `aten`.
- **CN**: 这里定义了函数`register_quantized_ops`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `has_out_variant`、`kernel_creator`、`quantized`、`_quantized`、`aten` 等值。

### Lines 43-56 / 第 43-56 行
````python
            _quantized.wrapped_fbgemm_pack_gemm_matrix_fp16,
            _quantized.wrapped_fbgemm_linear_fp16_weight,
        ]
    )
    lowering.make_fallback(quantized.max_pool2d)
    lowering.make_fallback(_quantized.wrapped_fbgemm_pack_gemm_matrix_fp16)
    lowering.make_fallback(_quantized.wrapped_fbgemm_linear_fp16_weight)


def register_woq_mm_ops() -> None:
    @register_lowering(aten._weight_int8pack_mm, type_promotion_kind=None)  # type: ignore[misc]
    def int8pack_mm(
        input: torch.Tensor,
        weight: torch.Tensor,
````
- **EN**: Introduces function `register_woq_mm_ops`, function `int8pack_mm`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `input`, and `weight`.
- **CN**: 这里定义了函数`register_woq_mm_ops`、函数`int8pack_mm`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `input`、`weight` 等值。

### Lines 57-70 / 第 57-70 行
````python
        scale: torch.Tensor,
        *,
        layout: Any = None,
    ) -> Any:
        _, _, _, layout, mat1, mat2 = mm_args(
            input, weight, layout=layout, mat2_transposed=True
        )
        assert (
            mat1.get_dtype() in [torch.bfloat16, torch.float16, torch.float]
            and mat2.get_dtype() == torch.int8
        )
        aten_layout = layout

        # options to tune from
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `scale`, `layout`, and `aten_layout`. This range continues the implementation of function `register_woq_mm_ops.int8pack_mm`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `scale`、`layout`、`aten_layout` 等值。这一段延续了函数`register_woq_mm_ops.int8pack_mm` 的具体实现。

### Lines 71-84 / 第 71-84 行
````python
        choices = (
            [aten__weight_int8pack_mm.bind((mat1, mat2, scale), aten_layout)]
            if use_aten_gemm_kernels()
            else []
        )

        # scale is applied as an epilogue, and the scale tensor is expanded (with a view op)
        # for broadcasting, as it's 1D.
        def _mul_epilogue(buf: torch.Tensor) -> Any:
            return create_epilogue_with_attr(
                buf, "mul", other=realize_inputs(expand(scale, layout.size))
            )

        if use_cpp_gemm_template(aten_layout, mat1, mat2, mat2_transposed=True):
````
- **EN**: Introduces function `_mul_epilogue`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `choices`.
- **CN**: 这里定义了函数`_mul_epilogue`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `choices` 等值。

### Lines 85-98 / 第 85-98 行
````python
            CppGemmTemplate.add_choices(
                choices,
                aten_layout,
                [mat1, mat2, scale],
                trans_w=True,
                epilogue_creator=_mul_epilogue,  # type: ignore[arg-type]
            )

        node, _ = autotune_select_algorithm(
            "_weight_int8pack_mm", choices, [mat1, mat2, scale], aten_layout
        )
        return node

    @register_lowering(aten._weight_int4pack_mm_for_cpu, type_promotion_kind=None)  # type: ignore[misc]
````
- **EN**: Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `trans_w`, and `epilogue_creator`.
- **CN**: 使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `trans_w`、`epilogue_creator` 等值。

### Lines 99-112 / 第 99-112 行
````python
    def int4pack_mm_cpu(
        input: torch.Tensor,
        weight: torch.Tensor,
        qGroupSize: int,
        qScaleAndZeros: torch.Tensor,
        *,
        layout: Any = None,
    ) -> Any:
        _, _, _, layout, mat1, mat2 = mm_args(
            input, weight, layout=layout, use_4x2_dim=True, mat2_transposed=True
        )
        assert (
            mat1.get_dtype() in [torch.bfloat16, torch.float16, torch.float]
            and mat2.get_dtype() == torch.uint8
````
- **EN**: Introduces function `int4pack_mm_cpu`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `input`, `weight`, `qGroupSize`, `qScaleAndZeros`, and `layout`.
- **CN**: 这里定义了函数`int4pack_mm_cpu`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `input`、`weight`、`qGroupSize`、`qScaleAndZeros`、`layout` 等值。

### Lines 113-126 / 第 113-126 行
````python
        )
        group_size = V.graph.add_tensor_constant(
            torch.tensor(qGroupSize, dtype=torch.int64), name=None
        )
        aten_layout = layout

        # options to tune from
        choices = (
            [
                aten__weight_int4pack_mm_cpu.bind(
                    (mat1, mat2, group_size, qScaleAndZeros), aten_layout
                )
            ]
            if use_aten_gemm_kernels()
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `group_size`, `aten_layout`, and `choices`. This range continues the implementation of function `register_woq_mm_ops.int4pack_mm_cpu`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `group_size`、`aten_layout`、`choices` 等值。这一段延续了函数`register_woq_mm_ops.int4pack_mm_cpu` 的具体实现。

### Lines 127-140 / 第 127-140 行
````python
            else []
        )
        if (
            (config.max_autotune or config.max_autotune_gemm)
            and use_cpp_gemm_template(
                aten_layout,
                mat1,
                mat2,
                mat2_transposed=True,
                is_woq_int4=True,
                q_group_size=qGroupSize,
            )
            and mat2.get_layout().is_contiguous()
        ):
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `mat2_transposed`, `is_woq_int4`, and `q_group_size`. This range continues the implementation of function `register_woq_mm_ops.int4pack_mm_cpu`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `mat2_transposed`、`is_woq_int4`、`q_group_size` 等值。这一段延续了函数`register_woq_mm_ops.int4pack_mm_cpu` 的具体实现。

### Lines 141-154 / 第 141-154 行
````python
            # pyrefly: ignore [bad-specialization, missing-attribute, not-a-type]
            CppWoqInt4GemmTemplate[qGroupSize].add_choices(
                choices,
                aten_layout,
                [mat1, mat2, group_size, qScaleAndZeros],
            )

        # define functions to generate example inputs for weight and group size
        # otherwise, autotuner generates example inputs of all zeros for them
        def get_example_weight(x: torch._inductor.ir.IRNode) -> torch.Tensor:
            assert x.get_layout().is_contiguous()
            shape = x.get_size()
            device = x.get_device()
            return torch.randint(0, 255, shape, dtype=torch.uint8, device=device)
````
- **EN**: Introduces function `get_example_weight`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_example_weight`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 155-168 / 第 155-168 行
````python

        input_gen_fns = {
            1: get_example_weight,  # packed weight
            2: lambda x: V.graph.constants[x.get_name()],  # group size
        }

        node, _ = autotune_select_algorithm(
            "_weight_int4pack_mm_for_cpu",
            choices,
            [mat1, mat2, group_size, qScaleAndZeros],
            aten_layout,
            input_gen_fns=input_gen_fns,
        )
        return node
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `input_gen_fns`. This range continues the implementation of function `register_woq_mm_ops.int4pack_mm_cpu`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `input_gen_fns` 等值。这一段延续了函数`register_woq_mm_ops.int4pack_mm_cpu` 的具体实现。

### Lines 169-171 / 第 169-171 行
````python

    lowering.make_fallback(aten._dyn_quant_matmul_4bit)
    lowering.make_fallback(aten._dyn_quant_pack_4bit_weight)
````
- **EN**: This range continues the implementation of function `register_woq_mm_ops`.
- **CN**: 这一段延续了函数`register_woq_mm_ops` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Autotuning support  
  **CN**: 自动调优支持
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary functions: `register_quantized_ops`, and `register_woq_mm_ops`  
  **CN**: 主要函数：`register_quantized_ops`、`register_woq_mm_ops`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor.kernel.mm_common`, `.`, `.codegen.cpp_gemm_template`, `.codegen.cpp_utils`, `.lowering`, `.mkldnn_ir`, `.select_algorithm`, `.utils`, `.virtualized`
