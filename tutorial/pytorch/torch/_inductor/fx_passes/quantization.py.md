# quantization.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/quantization.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It defines classes such as `PostOpAttr`. It exposes functions such as `_get_pattern_output_dtype`, `_may_generate_pattern_with_dtype_convert`, `_may_generate_pattern_with_reshape`, `_generate_linear_t_pattern`, `_unary_fusion_pattern`, `get_dequantize_per_tensor_activation_pattern`, and `...+72`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。其中定义了 `PostOpAttr` 等类。同时提供 `_get_pattern_output_dtype`、`_may_generate_pattern_with_dtype_convert`、`_may_generate_pattern_with_reshape`、`_generate_linear_t_pattern`、`_unary_fusion_pattern`、`get_dequantize_per_tensor_activation_pattern`、`另有72项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-36 / 第 1-36 行
````python
# mypy: allow-untyped-decorators
# mypy: allow-untyped-defs
import copy
import functools
import itertools
import math
import operator
from typing import Any

import torch
from torch._dynamo.utils import counters
from torch.fx.experimental.symbolic_shapes import has_free_symbols
from torch.fx.node import map_arg

from .. import config
from ..lowering import lowerings as L, require_channels_last
from ..pattern_matcher import (
    Arg,
    CallFunction,
    filter_nodes,
    KeywordArg,
    ListOf,
    Match,
    stable_topological_sort,
)
from ..utils import pad_listlike
from .freezing_patterns import register_freezing_graph_pattern
from .post_grad import register_lowering_pattern


aten = torch.ops.aten
prims = torch.ops.prims
quantized_decomposed = torch.ops.quantized_decomposed
quantized = torch.ops.quantized

# Only for per tensor quant since permute may changes the channel idx
````
- **EN**: Imports dependencies such as `copy`, `functools`, `itertools`, `math`, `operator`, `typing`, and `...+10` for the logic in this range. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `aten`, `prims`, `quantized_decomposed`, and `quantized`.
- **CN**: 这里导入了 `copy`、`functools`、`itertools`、`math`、`operator`、`typing`、`另有10项` 等依赖，为后续逻辑提供基础能力。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `aten`、`prims`、`quantized_decomposed`、`quantized` 等值。

### Lines 37-72 / 第 37-72 行
````python
_PER_TENSOR_QUANTIZE_OPS = [
    quantized_decomposed.quantize_per_tensor.default,
    quantized_decomposed.quantize_per_tensor.tensor,
]

_VIEW_OPS = [
    aten.transpose.int,
    aten.permute.default,
    aten.view.default,
    aten.reshape.default,
]

"""
The quantization.py file primarily incorporates passes related to quantization fusion
in inductor, includes:
1. Dequant Promotion;
2. Conv/GEMM weight prepack with oneDNN Library;
3. Conv/GEMM quantization fusion with output quant node (if have);
4. Other pointwise operators' quantization fusion like: qmaxpool2d, qcat and more;

It also involves int8-mixed-fp32 and int8-mixed-bf16 quantization. The main difference
of patterns for int8-mixed-bf16, comparing with int8-mixed-fp32, is
1. There is to(dtype=torch.bfloat16) node at the inputs of activation and weight for Conv/GEMM.
2. There is to(dtype=torch.float32) node at the outputs of Conv/GEMM before inputs to next quant node.
Refer to: https://github.com/pytorch/pytorch/issues/111640 for detail design of int8-mixed-bf16
quantization.
"""


def _get_pattern_output_dtype(match: Match):
    """
    Get the pattern's output dtype from node's meta
    Assume only 1 output node in this matched pattern.
    """
    pattern_output_nodes = match.output_nodes()
    assert len(pattern_output_nodes) == 1
````
- **EN**: Introduces function `_get_pattern_output_dtype`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_get_pattern_output_dtype`。保留了指向设计说明、规范或厂商数据手册的注释引用。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 73-108 / 第 73-108 行
````python
    output_node = pattern_output_nodes[0]
    assert isinstance(output_node, torch.fx.Node)
    output_dtype = output_node.meta["val"].dtype
    assert output_dtype in [
        torch.int8,
        torch.uint8,
        torch.float32,
        torch.bfloat16,
        torch.float8_e4m3fn,
    ]
    return output_dtype


def _may_generate_pattern_with_dtype_convert(
    pattern, dtype=Arg(), with_dtype_convert=True, users=1
):
    if with_dtype_convert:
        return CallFunction(
            prims.convert_element_type.default,
            pattern,
            dtype,
            _users=users,
        )
    else:
        return pattern


def _may_generate_pattern_with_reshape(pattern, reshape_size=Arg(), with_reshape=True):
    if with_reshape:
        return CallFunction(
            torch.ops.aten.reshape.default,
            pattern,
            reshape_size,
        )
    else:
        return pattern
````
- **EN**: Introduces function `_may_generate_pattern_with_dtype_convert`, function `_may_generate_pattern_with_reshape`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_may_generate_pattern_with_dtype_convert`、函数`_may_generate_pattern_with_reshape`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 109-144 / 第 109-144 行
````python


def _generate_linear_t_pattern(
    _dequant_per_channel_pattern,
    dtype,
):
    assert dtype in [torch.float32, torch.bfloat16]
    t_pattern = CallFunction(
        aten.permute.default,
        _may_generate_pattern_with_dtype_convert(
            _dequant_per_channel_pattern,
            KeywordArg("autocast_wgt_dtype"),
            dtype == torch.bfloat16,
        ),
        KeywordArg("permute_axes"),
    )
    return t_pattern


def _unary_fusion_pattern(unary_fusion, call_fn, users, is_bf16):
    # only insert to_dtype if is_bf16 is True
    computation_call = _may_generate_pattern_with_dtype_convert(
        call_fn, dtype=KeywordArg("to_float"), with_dtype_convert=is_bf16, users=users
    )
    return unary_fusion(computation_call)


def get_dequantize_per_tensor_activation_pattern(is_tensor_overload=False):
    dequantize_per_tensor_activation_pattern = CallFunction(
        quantized_decomposed.dequantize_per_tensor.tensor
        if is_tensor_overload
        else quantized_decomposed.dequantize_per_tensor.default,
        KeywordArg("x"),
        KeywordArg("x_scale"),
        KeywordArg("x_zp"),
        KeywordArg("x_quant_min"),
````
- **EN**: Introduces function `_generate_linear_t_pattern`, function `_unary_fusion_pattern`, function `get_dequantize_per_tensor_activation_pattern`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_generate_linear_t_pattern`、函数`_unary_fusion_pattern`、函数`get_dequantize_per_tensor_activation_pattern`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 145-180 / 第 145-180 行
````python
        KeywordArg("x_quant_max"),
        KeywordArg("x_dq_dtype"),
    )
    return dequantize_per_tensor_activation_pattern


dequantize_per_channel_weight_pattern = CallFunction(
    quantized_decomposed.dequantize_per_channel.default,
    KeywordArg("q_weight"),
    KeywordArg("w_scale"),
    KeywordArg("w_zp"),
    KeywordArg("w_axis"),
    KeywordArg("w_quant_min"),
    KeywordArg("w_quant_max"),
    KeywordArg("w_dtype"),
)

dequantize_per_channel_to_bf16_weight_pattern = (
    _may_generate_pattern_with_dtype_convert(
        dequantize_per_channel_weight_pattern,
        KeywordArg("autocast_wgt_dtype"),
    )
)

dequantize_per_channel_clone_weight_pattern = CallFunction(
    aten.clone.default,
    dequantize_per_channel_weight_pattern,
    memory_format=KeywordArg("memory_format"),
)

dequantize_per_channel_to_bf16_clone_weight_pattern = CallFunction(
    aten.clone.default,
    dequantize_per_channel_to_bf16_weight_pattern,
    memory_format=KeywordArg("memory_format"),
)

````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `dequantize_per_channel_weight_pattern`, `dequantize_per_channel_to_bf16_weight_pattern`, `dequantize_per_channel_clone_weight_pattern`, `memory_format`, and `dequantize_per_channel_to_bf16_clone_weight_pattern`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `dequantize_per_channel_weight_pattern`、`dequantize_per_channel_to_bf16_weight_pattern`、`dequantize_per_channel_clone_weight_pattern`、`memory_format`、`dequantize_per_channel_to_bf16_clone_weight_pattern` 等值。

### Lines 181-216 / 第 181-216 行
````python

def get_qconv_pt2e_pattern(x_scale_zp_are_tensors=False, users=1):
    qconv_op = (
        torch.ops.onednn.qconv_pointwise.tensor
        if x_scale_zp_are_tensors
        else torch.ops.onednn.qconv_pointwise.default
    )
    return CallFunction(
        qconv_op,
        KeywordArg("x"),
        KeywordArg("x_scale"),
        KeywordArg("x_zp"),
        KeywordArg("packed_weight"),
        KeywordArg("w_scale"),
        KeywordArg("w_zp"),
        KeywordArg("b"),
        KeywordArg("stride"),
        KeywordArg("padding"),
        KeywordArg("dilation"),
        KeywordArg("groups"),
        KeywordArg("output_scale"),
        KeywordArg("output_zero_point"),
        KeywordArg("output_dtype"),
        KeywordArg("postop_name"),
        KeywordArg("postop_args"),
        KeywordArg("postop_algorithm"),
        _users=users,
    )


def get_qconv2d_binary_pt2e_pattern(x_scale_zp_are_tensors=False, users=1):
    qconv_op = (
        torch.ops.onednn.qconv2d_pointwise.binary_tensor
        if x_scale_zp_are_tensors
        else torch.ops.onednn.qconv2d_pointwise.binary
    )
````
- **EN**: Introduces function `get_qconv_pt2e_pattern`, function `get_qconv2d_binary_pt2e_pattern`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `qconv_op`, and `_users`.
- **CN**: 这里定义了函数`get_qconv_pt2e_pattern`、函数`get_qconv2d_binary_pt2e_pattern`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `qconv_op`、`_users` 等值。

### Lines 217-252 / 第 217-252 行
````python
    return CallFunction(
        qconv_op,
        KeywordArg("x"),
        KeywordArg("x_scale"),
        KeywordArg("x_zp"),
        KeywordArg("packed_weight"),
        KeywordArg("w_scale"),
        KeywordArg("w_zp"),
        KeywordArg("accum"),
        KeywordArg("b"),
        KeywordArg("stride"),
        KeywordArg("padding"),
        KeywordArg("dilation"),
        KeywordArg("groups"),
        KeywordArg("output_scale"),
        KeywordArg("output_zero_point"),
        KeywordArg("output_dtype"),
        KeywordArg("accum_scale"),
        KeywordArg("accum_zero_point"),
        KeywordArg("binary_op_name"),
        KeywordArg("alpha"),
        KeywordArg("unary_op_name"),
        KeywordArg("unary_op_args"),
        KeywordArg("unary_op_algorithm"),
        _users=users,
    )


def get_qlinear_pt2e_pattern(x_scale_zp_are_tensors, users=1):
    qlinear_op = (
        torch.ops.onednn.qlinear_pointwise.tensor
        if x_scale_zp_are_tensors
        else torch.ops.onednn.qlinear_pointwise.default
    )
    return CallFunction(
        qlinear_op,
````
- **EN**: Introduces function `get_qlinear_pt2e_pattern`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `_users`, and `qlinear_op`.
- **CN**: 这里定义了函数`get_qlinear_pt2e_pattern`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `_users`、`qlinear_op` 等值。

### Lines 253-288 / 第 253-288 行
````python
        KeywordArg("x"),
        KeywordArg("x_scale"),
        KeywordArg("x_zp"),
        KeywordArg("packed_weight"),
        KeywordArg("w_scale"),
        KeywordArg("w_zp"),
        KeywordArg("b"),
        KeywordArg("output_scale"),
        KeywordArg("output_zero_point"),
        KeywordArg("output_dtype"),
        KeywordArg("postop_name"),
        KeywordArg("postop_args"),
        KeywordArg("postop_algorithm"),
        _users=users,
    )


def get_qlinear_binary_pt2e_pattern(x_scale_zp_are_tensors, users=1):
    qlinear_op = (
        torch.ops.onednn.qlinear_pointwise.binary_tensor
        if x_scale_zp_are_tensors
        else torch.ops.onednn.qlinear_pointwise.binary
    )
    return CallFunction(
        qlinear_op,
        KeywordArg("x"),
        KeywordArg("x_scale"),
        KeywordArg("x_zp"),
        KeywordArg("packed_weight"),
        KeywordArg("w_scale"),
        KeywordArg("w_zp"),
        KeywordArg("x_2"),
        KeywordArg("b"),
        KeywordArg("output_scale"),
        KeywordArg("output_zero_point"),
        KeywordArg("output_dtype"),
````
- **EN**: Introduces function `get_qlinear_binary_pt2e_pattern`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `_users`, and `qlinear_op`.
- **CN**: 这里定义了函数`get_qlinear_binary_pt2e_pattern`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `_users`、`qlinear_op` 等值。

### Lines 289-324 / 第 289-324 行
````python
        KeywordArg("x2_scale"),
        KeywordArg("x2_zp"),
        KeywordArg("binary_op_name"),
        KeywordArg("alpha"),
        KeywordArg("unary_op_name"),
        KeywordArg("unary_op_args"),
        KeywordArg("unary_op_algorithm"),
        _users=users,
    )


dequantize_accum_pattern = CallFunction(
    quantized_decomposed.dequantize_per_tensor.default,
    KeywordArg("accum"),
    KeywordArg("accum_scale"),
    KeywordArg("accum_zp"),
    Arg(),
    Arg(),
    KeywordArg("accum_dq_dtype"),
)


def generate_pattern_with_binary(
    binary_post_op,
    computation_call,
    extra_input_pattern,
    dtype_convert=False,
    swap_inputs=False,
):
    binary_pattern = (
        CallFunction(
            binary_post_op,
            extra_input_pattern,
            computation_call,
        )
        if swap_inputs
````
- **EN**: Introduces function `generate_pattern_with_binary`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `_users`, `dequantize_accum_pattern`, `dtype_convert`, `swap_inputs`, and `binary_pattern`.
- **CN**: 这里定义了函数`generate_pattern_with_binary`。包含分支、循环或上下文管理等控制流。初始化或更新了 `_users`、`dequantize_accum_pattern`、`dtype_convert`、`swap_inputs`、`binary_pattern` 等值。

### Lines 325-360 / 第 325-360 行
````python
        else CallFunction(
            binary_post_op,
            computation_call,
            extra_input_pattern,
        )
    )
    return _may_generate_pattern_with_dtype_convert(
        binary_pattern,
        KeywordArg("convert_dtype_after_inplace_add"),
        dtype_convert,
    )


def generate_pattern_with_unary(computation_call, unary_post_op):
    if unary_post_op is not None:
        return CallFunction(
            unary_post_op,
            computation_call,
        )
    return computation_call


def generate_pattern_with_output_quant(computation_call, with_dtype_convert=False):
    quantized_op_output_pattern_pt2e = CallFunction(
        quantized_decomposed.quantize_per_tensor.default,
        _may_generate_pattern_with_dtype_convert(
            computation_call,
            Arg(),
            with_dtype_convert,
        ),
        KeywordArg("o_inv_scale"),
        KeywordArg("o_zp"),
        KeywordArg("o_qmin"),
        KeywordArg("o_qmax"),
        KeywordArg("o_dtype"),
    )
````
- **EN**: Introduces function `generate_pattern_with_unary`, function `generate_pattern_with_output_quant`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `quantized_op_output_pattern_pt2e`.
- **CN**: 这里定义了函数`generate_pattern_with_unary`、函数`generate_pattern_with_output_quant`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `quantized_op_output_pattern_pt2e` 等值。

### Lines 361-396 / 第 361-396 行
````python
    return quantized_op_output_pattern_pt2e


def _check_node_kwarg_arg_value(check_node, kwarg_name, args_index, expected_value):
    if kwarg_name in check_node.kwargs:
        actual_value = check_node.kwargs[kwarg_name]
        return actual_value == expected_value
    else:
        assert len(check_node.args) >= (args_index + 1)
        actual_value = check_node.args[args_index]
        return actual_value == expected_value


def _is_valid_quantized_conv_optimization_pattern():
    def fn(match):
        output_dtype = _get_pattern_output_dtype(match)
        if output_dtype in [torch.float32, torch.bfloat16]:
            # Only keep matched pattern with same output_dtype
            qconv_node_after_weight_prepack = filter_nodes(
                match.nodes, torch.ops.onednn.qconv_pointwise
            )[0]
            return _check_node_kwarg_arg_value(
                qconv_node_after_weight_prepack, "output_dtype", 13, output_dtype
            )
        return True

    return fn


def _is_valid_qconv_post_op_fusion_pattern(has_binary_post_op=False):
    return (
        _is_valid_qconv_binary_optimization_pattern()
        if has_binary_post_op
        else _is_valid_quantized_conv_optimization_pattern()
    )

````
- **EN**: Introduces function `_check_node_kwarg_arg_value`, function `_is_valid_quantized_conv_optimization_pattern`, function `fn`, function `_is_valid_qconv_post_op_fusion_pattern`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_check_node_kwarg_arg_value`、函数`_is_valid_quantized_conv_optimization_pattern`、函数`fn`、函数`_is_valid_qconv_post_op_fusion_pattern`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 397-432 / 第 397-432 行
````python

def _is_valid_qconv_lowering_pattern():
    def fn(match):
        if len(match.nodes) != 1:
            return False
        return match.nodes[0].target in (
            torch.ops.onednn.qconv_pointwise.default,
            torch.ops.onednn.qconv_pointwise.tensor,
            torch.ops.onednn.qconv2d_pointwise.binary,
            torch.ops.onednn.qconv2d_pointwise.binary_tensor,
        )

    return fn


def _register_quantized_conv_lowering(
    pattern,
    pass_number,
    computation_op,
):
    @register_lowering_pattern(
        pattern,
        extra_check=_is_valid_qconv_lowering_pattern(),
        pass_number=pass_number,
    )
    def qconv(match: Match, *args, **kwargs):
        # Activation QParams
        x, x_scale, x_zp = (
            kwargs["x"],
            kwargs["x_scale"],
            kwargs["x_zp"],
        )
        # Weight QParams
        packed_weight, w_scale, w_zp = (
            kwargs["packed_weight"],
            kwargs["w_scale"],
````
- **EN**: Introduces function `_is_valid_qconv_lowering_pattern`, function `fn`, function `_register_quantized_conv_lowering`, function `qconv`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_is_valid_qconv_lowering_pattern`、函数`fn`、函数`_register_quantized_conv_lowering`、函数`qconv`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 433-468 / 第 433-468 行
````python
            kwargs["w_zp"],
        )
        # Conv Params
        b, stride, padding, dilation, groups = (
            kwargs["b"],
            kwargs["stride"],
            kwargs["padding"],
            kwargs["dilation"],
            kwargs["groups"],
        )
        output_dtype = _get_pattern_output_dtype(match)
        assert output_dtype in [
            torch.int8,
            torch.uint8,
            torch.float8_e4m3fn,
            torch.float32,
            torch.bfloat16,
        ]
        # Output QParams
        o_inv_scale = kwargs["output_scale"]
        o_zero_point = kwargs["output_zero_point"]
        output_dtype = kwargs["output_dtype"]
        # post op
        postop_name = kwargs["postop_name"]
        postop_args = kwargs["postop_args"]
        postop_algorithm = kwargs["postop_algorithm"]

        computation_args = (
            x,
            x_scale,
            x_zp,
            packed_weight,
            w_scale,
            w_zp,
            b,
            stride,
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `output_dtype`, `o_inv_scale`, `o_zero_point`, `postop_name`, `postop_args`, `postop_algorithm`, and `...+1`. This range continues the implementation of function `_register_quantized_conv_lowering.qconv`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `output_dtype`、`o_inv_scale`、`o_zero_point`、`postop_name`、`postop_args`、`postop_algorithm`、`另有1项` 等值。这一段延续了函数`_register_quantized_conv_lowering.qconv` 的具体实现。

### Lines 469-504 / 第 469-504 行
````python
            padding,
            dilation,
            groups,
            o_inv_scale,
            o_zero_point,
            output_dtype,
            postop_name,
            postop_args,
            postop_algorithm,
        )
        counters["inductor"]["qconv_unary_lower_count"] += 1
        counters["inductor"]["qconv_unary_lower_nodes"] += len(match.nodes)
        return L[computation_op](*computation_args)

    return qconv


def _is_valid_quantized_linear_optimization_pattern():
    def fn(match):
        output_dtype = _get_pattern_output_dtype(match)
        if output_dtype in [torch.float32, torch.bfloat16]:
            # Only keep matched pattern with same output_dtype
            qlinear_node_after_weight_prepack = filter_nodes(
                match.nodes, torch.ops.onednn.qlinear_pointwise
            )[0]
            return _check_node_kwarg_arg_value(
                qlinear_node_after_weight_prepack, "output_dtype", 9, output_dtype
            )
        return True

    return fn


def _is_valid_qlinear_post_op_fusion_pattern(has_binary_post_op=False):
    return (
        _is_valid_qlinear_binary_optimization_pattern()
````
- **EN**: Introduces function `_is_valid_quantized_linear_optimization_pattern`, function `fn`, function `_is_valid_qlinear_post_op_fusion_pattern`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_is_valid_quantized_linear_optimization_pattern`、函数`fn`、函数`_is_valid_qlinear_post_op_fusion_pattern`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 505-540 / 第 505-540 行
````python
        if has_binary_post_op
        else _is_valid_quantized_linear_optimization_pattern()
    )


def _is_valid_qlinear_lowering_pattern():
    def fn(match):
        if len(match.nodes) != 1:
            return False
        return match.nodes[0].target in (
            torch.ops.onednn.qlinear_pointwise.default,
            torch.ops.onednn.qlinear_pointwise.tensor,
            torch.ops.onednn.qlinear_pointwise.binary,
            torch.ops.onednn.qlinear_pointwise.binary_tensor,
        )

    return fn


def _register_quantized_linear_unary_lowering(
    pattern,
    pass_number,
    computation_op,
):
    @register_lowering_pattern(
        pattern,
        extra_check=_is_valid_qlinear_lowering_pattern(),
        pass_number=pass_number,
    )
    def qlinear(match: Match, *args, **kwargs):
        output_dtype = _get_pattern_output_dtype(match)
        # Activation QParams
        x, x_scale, x_zp = (
            kwargs["x"],
            kwargs["x_scale"],
            kwargs["x_zp"],
````
- **EN**: Introduces function `_is_valid_qlinear_lowering_pattern`, function `fn`, function `_register_quantized_linear_unary_lowering`, function `qlinear`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_is_valid_qlinear_lowering_pattern`、函数`fn`、函数`_register_quantized_linear_unary_lowering`、函数`qlinear`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 541-576 / 第 541-576 行
````python
        )
        # Weight QParams
        packed_weight, w_scale, w_zp = (
            kwargs["packed_weight"],
            kwargs["w_scale"],
            kwargs["w_zp"],
        )

        # bias
        b = kwargs.get("b")

        # Output QParams
        o_inv_scale = kwargs["output_scale"]
        o_zero_point = kwargs["output_zero_point"]

        # post op
        postop_name = kwargs["postop_name"]
        postop_args = kwargs["postop_args"]
        postop_algorithm = kwargs["postop_algorithm"]

        computation_args = (
            x,
            x_scale,
            x_zp,
            packed_weight,
            w_scale,
            w_zp,
            b,
            o_inv_scale,
            o_zero_point,
            output_dtype,
            postop_name,
            postop_args,
            postop_algorithm,
        )
        counters["inductor"]["qlinear_unary_lower_count"] += 1
````
- **EN**: Initializes or updates values such as `b`, `o_inv_scale`, `o_zero_point`, `postop_name`, `postop_args`, `postop_algorithm`, and `...+1`. This range continues the implementation of function `_register_quantized_linear_unary_lowering.qlinear`.
- **CN**: 初始化或更新了 `b`、`o_inv_scale`、`o_zero_point`、`postop_name`、`postop_args`、`postop_algorithm`、`另有1项` 等值。这一段延续了函数`_register_quantized_linear_unary_lowering.qlinear` 的具体实现。

### Lines 577-612 / 第 577-612 行
````python
        counters["inductor"]["qlinear_unary_lower_nodes"] += len(match.nodes)
        return L[computation_op](*computation_args)

    return qlinear


def _register_quantized_linear_binary_lowering(
    pattern,
    pass_number,
    computation_op,
):
    @register_lowering_pattern(
        pattern,
        extra_check=_is_valid_qlinear_lowering_pattern(),
        pass_number=pass_number,
    )
    def qlinear_binary(match: Match, *args, **kwargs):
        output_dtype = _get_pattern_output_dtype(match)
        assert output_dtype is not None
        # Activation QParams
        x, x_scale, x_zp = (
            kwargs["x"],
            kwargs["x_scale"],
            kwargs["x_zp"],
        )
        x2 = kwargs["x_2"]
        x2_scale = kwargs["x2_scale"]
        x2_zp = kwargs["x2_zp"]
        # Weight QParams
        packed_weight, w_scale, w_zp = (
            kwargs["packed_weight"],
            kwargs["w_scale"],
            kwargs["w_zp"],
        )
        # bias
        b = kwargs.get("b")
````
- **EN**: Introduces function `_register_quantized_linear_binary_lowering`, function `qlinear_binary`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_register_quantized_linear_binary_lowering`、函数`qlinear_binary`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 613-648 / 第 613-648 行
````python
        # Output QParams
        o_inv_scale = kwargs["output_scale"]
        o_zero_point = kwargs["output_zero_point"]

        x2.realize()
        from .mkldnn_fusion import _qlinear_binary_can_be_inplace

        binary_op_name = kwargs["binary_op_name"]
        alpha = kwargs["alpha"]
        unary_op_name = kwargs["unary_op_name"]
        unary_op_args = kwargs["unary_op_args"]
        unary_op_algorithm = kwargs["unary_op_algorithm"]
        if (
            # TODO Ensure sum is safe and remove such check, i.e.,
            # x2 is not used by other operations
            # or current qlinear sum is the last user of x2.
            # This needs to be ensured when registering
            # the lowering pattern of quantized_linear_binary.
            binary_op_name == "sum" and (not _qlinear_binary_can_be_inplace(x2))
        ):
            binary_op_name = "add"

        computation_args = (
            x,
            x_scale,
            x_zp,
            packed_weight,
            w_scale,
            w_zp,
            x2,
            b,
            o_inv_scale,
            o_zero_point,
            output_dtype,
            x2_scale,
            x2_zp,
````
- **EN**: Imports dependencies such as `.mkldnn_fusion` for the logic in this range. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `o_inv_scale`, `o_zero_point`, `binary_op_name`, `alpha`, `unary_op_name`, `unary_op_args`, and `...+2`.
- **CN**: 这里导入了 `.mkldnn_fusion` 等依赖，为后续逻辑提供基础能力。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。初始化或更新了 `o_inv_scale`、`o_zero_point`、`binary_op_name`、`alpha`、`unary_op_name`、`unary_op_args`、`另有2项` 等值。

### Lines 649-684 / 第 649-684 行
````python
            binary_op_name,
            alpha,
            unary_op_name,
            unary_op_args,
            unary_op_algorithm,
        )
        counters["inductor"]["qlinear_binary_lower_count"] += 1
        counters["inductor"]["qlinear_binary_lower_nodes"] += len(match.nodes)
        return L[computation_op](*computation_args)

    return qlinear_binary


def _is_valid_qconv_binary_optimization_pattern():
    return _is_valid_quantized_op_binary_optimization_pattern(
        torch.ops.onednn.qconv_pointwise
    )


def _is_valid_qlinear_binary_optimization_pattern():
    return _is_valid_quantized_op_binary_optimization_pattern(
        torch.ops.onednn.qlinear_pointwise,
        # we don't insert q-dq for extra input due to accuracy issues
        extra_input_from_dequant=False,
    )


def _is_valid_quantized_op_binary_optimization_pattern(
    qop, extra_input_from_dequant=True
):
    # Check if it's a valid Binary Pattern for qconv2d and qlinear:
    # * qop_pointwise should only has one users
    # * If extra_input_from_dequant is True, extra input of binary node should come from dequant pattern
    # * the two inputs of binary node should have attribute "meta" and should be tensors
    # * the two inputs of binary node should have the same shape
    # * All users of the extra input in this pattern should be
````
- **EN**: Introduces function `_is_valid_qconv_binary_optimization_pattern`, function `_is_valid_qlinear_binary_optimization_pattern`, function `_is_valid_quantized_op_binary_optimization_pattern`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `extra_input_from_dequant`.
- **CN**: 这里定义了函数`_is_valid_qconv_binary_optimization_pattern`、函数`_is_valid_qlinear_binary_optimization_pattern`、函数`_is_valid_quantized_op_binary_optimization_pattern`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `extra_input_from_dequant` 等值。

### Lines 685-720 / 第 685-720 行
````python
    #   ancestor nodes of the compute node, except for the binary node
    #   connected to the compute node.
    def fn(match):
        output_dtype = _get_pattern_output_dtype(match)
        compute_node = filter_nodes(match.nodes, qop)[0]
        # qop_pointwise should only have one user
        if len(compute_node.users) != 1:
            return False
        binary_node_inputs = next(iter(compute_node.users)).args
        assert len(binary_node_inputs) == 2, "Expects binary node with 2 inputs"
        if output_dtype in [torch.float32, torch.bfloat16]:
            extra_input_of_binary_node = None
            for arg in binary_node_inputs:
                if arg != compute_node:
                    extra_input_of_binary_node = arg
                    break
            assert extra_input_of_binary_node is not None
            # Extra input of binary node comes from dequant pattern
            if extra_input_from_dequant and (
                (not isinstance(extra_input_of_binary_node, torch.fx.Node))
                or (
                    extra_input_of_binary_node.target
                    != quantized_decomposed.dequantize_per_tensor.default
                )
            ):
                return False

        # the two inputs of binary node should have attribute "meta" and should be tensors
        if not (
            hasattr(binary_node_inputs[0], "meta")
            and isinstance(binary_node_inputs[0].meta.get("val", None), torch.Tensor)  # type: ignore[union-attr]
        ) or not (
            hasattr(binary_node_inputs[1], "meta")
            and isinstance(binary_node_inputs[1].meta.get("val", None), torch.Tensor)  # type: ignore[union-attr]
        ):
            return False
````
- **EN**: Introduces function `fn`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`fn`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 721-756 / 第 721-756 行
````python
        # the two inputs of binary node should have the same shape
        if (
            binary_node_inputs[0].meta["val"].size()  # type: ignore[union-attr]
            != binary_node_inputs[1].meta["val"].size()  # type: ignore[union-attr]
        ):
            return False

        # All users of the extra input in this pattern should be
        # ancestor nodes of the compute node, except for the binary node
        # connected to the compute node.

        from .mkldnn_fusion import _get_remaining_users

        extra_input_of_pattern = (
            match.kwargs["other"]
            if "other" in match.kwargs
            else (
                match.kwargs["accum"]
                if (output_dtype in [torch.uint8, torch.int8])
                or (not extra_input_from_dequant)
                else match.kwargs["accum_after_dequant"]
            )
        )
        if (
            len(_get_remaining_users(extra_input_of_pattern, compute_node)) > 1
            or extra_input_of_pattern == compute_node.args[0]
        ):
            return False
        return True

    return fn


def _register_quantized_conv_binary_lowering(
    pattern,
    pass_number,
````
- **EN**: Imports dependencies such as `.mkldnn_fusion` for the logic in this range. Introduces function `_register_quantized_conv_binary_lowering`. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里导入了 `.mkldnn_fusion` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_register_quantized_conv_binary_lowering`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 757-792 / 第 757-792 行
````python
    computation_op,
):
    @register_lowering_pattern(
        pattern,
        extra_check=_is_valid_qconv_lowering_pattern(),
        pass_number=pass_number,
    )
    def qconv_binary(match: Match, *args, **kwargs):
        output_dtype = _get_pattern_output_dtype(match)
        assert output_dtype is not None
        x, x_scale, x_zp = kwargs["x"], kwargs["x_scale"], kwargs["x_zp"]
        accum = kwargs["accum"]
        accum_scale = kwargs["accum_scale"]
        accum_zp = kwargs["accum_zero_point"]
        packed_weight, w_scale, w_zp = (
            kwargs["packed_weight"],
            kwargs["w_scale"],
            kwargs["w_zp"],
        )
        b, stride, padding, dilation, groups = (
            kwargs["b"],
            kwargs["stride"],
            kwargs["padding"],
            kwargs["dilation"],
            kwargs["groups"],
        )
        # Output QParams
        output_scale = kwargs["output_scale"]
        output_zero_point = kwargs["output_zero_point"]

        # post ops
        binary_op_name = kwargs["binary_op_name"]
        alpha = kwargs["alpha"]
        unary_op_name = kwargs["unary_op_name"]
        unary_op_args = kwargs["unary_op_args"]
        unary_op_algorithm = kwargs["unary_op_algorithm"]
````
- **EN**: Introduces function `qconv_binary`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`qconv_binary`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 793-828 / 第 793-828 行
````python

        accum.realize()
        from .mkldnn_fusion import _can_be_inplace

        assert _can_be_inplace(accum), (
            "QConv Binary Inplace Fusion requires accum is not an alias or mutation."
        )

        computation_args = (
            x,
            x_scale,
            x_zp,
            packed_weight,
            w_scale,
            w_zp,
            accum,
            b,
            stride,
            padding,
            dilation,
            groups,
            output_scale,
            output_zero_point,
            output_dtype,
            accum_scale,
            accum_zp,
            binary_op_name,
            alpha,
            unary_op_name,
            unary_op_args,
            unary_op_algorithm,
        )
        counters["inductor"]["qconv2d_binary_lower_count"] += 1
        counters["inductor"]["qconv2d_binary_lower_nodes"] += len(match.nodes)
        return L[computation_op](*computation_args)

````
- **EN**: Imports dependencies such as `.mkldnn_fusion` for the logic in this range. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `computation_args`. This range continues the implementation of function `_register_quantized_conv_binary_lowering.qconv_binary`.
- **CN**: 这里导入了 `.mkldnn_fusion` 等依赖，为后续逻辑提供基础能力。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `computation_args` 等值。这一段延续了函数`_register_quantized_conv_binary_lowering.qconv_binary` 的具体实现。

### Lines 829-864 / 第 829-864 行
````python
    return qconv_binary


def _register_quantization_unary_lowering():
    # QConv2d
    for x_scale_zp_are_tensors, users in itertools.product([False, True], [1, 2]):
        qconv_pattern = get_qconv_pt2e_pattern(x_scale_zp_are_tensors, users)
        computation_op = (
            torch.ops.onednn.qconv_pointwise.tensor
            if x_scale_zp_are_tensors
            else torch.ops.onednn.qconv_pointwise.default
        )
        _register_quantized_conv_lowering(
            qconv_pattern,
            2,  # pass_number
            computation_op,
        )

    # QLinear
    for x_scale_zp_are_tensors in (False, True):
        qlinear_pattern = get_qlinear_pt2e_pattern(x_scale_zp_are_tensors)
        computation_op = (
            torch.ops.onednn.qlinear_pointwise.tensor
            if x_scale_zp_are_tensors
            else torch.ops.onednn.qlinear_pointwise.default
        )
        _register_quantized_linear_unary_lowering(
            qlinear_pattern,
            2,  # pass_number
            computation_op,
        )


def _register_quantization_binary_lowering():
    # QConv2d
    for x_scale_zp_are_tensors, users in itertools.product([False, True], [1, 2]):
````
- **EN**: Introduces function `_register_quantization_unary_lowering`, function `_register_quantization_binary_lowering`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_register_quantization_unary_lowering`、函数`_register_quantization_binary_lowering`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 865-900 / 第 865-900 行
````python
        qconv_pattern = get_qconv2d_binary_pt2e_pattern(x_scale_zp_are_tensors, users)
        computation_op = (
            torch.ops.onednn.qconv2d_pointwise.binary_tensor
            if x_scale_zp_are_tensors
            else torch.ops.onednn.qconv2d_pointwise.binary
        )
        _register_quantized_conv_binary_lowering(
            qconv_pattern,
            2,  # pass_number
            computation_op,
        )

    # QLinear
    for x_scale_zp_are_tensors in (False, True):
        qlinear_pattern = get_qlinear_binary_pt2e_pattern(x_scale_zp_are_tensors)
        computation_op = (
            torch.ops.onednn.qlinear_pointwise.binary_tensor
            if x_scale_zp_are_tensors
            else torch.ops.onednn.qlinear_pointwise.binary
        )
        _register_quantized_linear_binary_lowering(
            qlinear_pattern,
            2,  # pass_number
            computation_op,
        )


def _is_valid_quantized_maxpool2d_optimization_pattern():
    def fn(match):
        # Only match the pattern which max_pool2d_with_indices returns value
        # instead of indices.
        get_item_node = filter_nodes(match.nodes, operator.getitem)[0]
        return get_item_node.args[1] == 0

    return fn

````
- **EN**: Introduces function `_is_valid_quantized_maxpool2d_optimization_pattern`, function `fn`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_is_valid_quantized_maxpool2d_optimization_pattern`、函数`fn`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 901-936 / 第 901-936 行
````python

def _register_quantized_maxpool2d_lowering(
    pattern,
    computation_op,
):
    @register_lowering_pattern(
        pattern,
        extra_check=_is_valid_quantized_maxpool2d_optimization_pattern(),
    )
    def qmaxpool2d(match: Match, *args, **kwargs):
        x = kwargs["x"]
        kernel_size = kwargs["kernel_size"]
        stride = kwargs.get("stride")
        padding = kwargs.get("padding", 0)
        dilation = kwargs.get("dilation", 1)
        ceil_mode = kwargs.get("ceil_mode", False)

        if padding == 0:
            padding = [0, 0]
        if dilation == 1:
            dilation = [1, 1]
        if not stride:
            stride = kernel_size
        kernel_size = pad_listlike(kernel_size, 2)
        stride = pad_listlike(stride, 2)
        padding = pad_listlike(padding, 2)
        dilation = pad_listlike(dilation, 2)

        assert len(kernel_size) == 2
        assert len(stride) == 2
        assert len(padding) == 2
        assert len(dilation) == 2

        computation_args = (
            x,
            kernel_size,
````
- **EN**: Introduces function `_register_quantized_maxpool2d_lowering`, function `qmaxpool2d`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_register_quantized_maxpool2d_lowering`、函数`qmaxpool2d`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 937-972 / 第 937-972 行
````python
            stride,
            padding,
            dilation,
            ceil_mode,
        )
        computation_args, _ = require_channels_last(computation_op, *computation_args)
        counters["inductor"]["qmaxpool2d_matcher_count"] += 1
        counters["inductor"]["qmaxpool2d_matcher_nodes"] += len(match.nodes)
        return L[computation_op](*computation_args)

    return qmaxpool2d


def _register_quantization_maxpool2d():
    # Currently, the default parameters are not in FX Graph generated by Dynamo export.
    # So, if user defines nn.MaxPool2d with different assignment of default parameter,
    # it will generate graph with different number of input nodes and hence
    # different pattern to be matched.
    # Refer to the issue: https://github.com/pytorch/pytorch/issues/105901
    max_pool2d_args_list = [
        [
            KeywordArg("stride"),
        ],
        [
            KeywordArg("stride"),
            KeywordArg("padding"),
        ],
        [
            KeywordArg("stride"),
            KeywordArg("padding"),
            KeywordArg("dilation"),
        ],
        [
            KeywordArg("stride"),
            KeywordArg("padding"),
            KeywordArg("dilation"),
````
- **EN**: Introduces function `_register_quantization_maxpool2d`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_register_quantization_maxpool2d`。保留了指向设计说明、规范或厂商数据手册的注释引用。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 973-1008 / 第 973-1008 行
````python
            KeywordArg("ceil_mode"),
        ],
    ]
    for max_pool2d_args in max_pool2d_args_list:
        dequantize_maxpool2d_pattern = CallFunction(
            aten.max_pool2d_with_indices.default,
            get_dequantize_per_tensor_activation_pattern(),
            KeywordArg("kernel_size"),
            *max_pool2d_args,
        )
        dequantize_lowmem_maxpool2d_pattern = CallFunction(
            prims._low_memory_max_pool_with_offsets.default,
            get_dequantize_per_tensor_activation_pattern(),
            KeywordArg("kernel_size"),
            *max_pool2d_args,
            KeywordArg("offset_dtype"),
        )
        dequantize_maxpool2d_get_item_pattern = CallFunction(
            operator.getitem,
            dequantize_maxpool2d_pattern,
            Arg(),
        )
        dequantize_lowmem_maxpool2d_get_item_pattern = CallFunction(
            operator.getitem,
            dequantize_lowmem_maxpool2d_pattern,
            Arg(),
        )
        _register_quantized_maxpool2d_lowering(
            generate_pattern_with_output_quant(dequantize_maxpool2d_get_item_pattern),
            quantized.max_pool2d.default,
        )
        _register_quantized_maxpool2d_lowering(
            generate_pattern_with_output_quant(
                dequantize_lowmem_maxpool2d_get_item_pattern
            ),
            quantized.max_pool2d.default,
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `dequantize_maxpool2d_pattern`, `dequantize_lowmem_maxpool2d_pattern`, `dequantize_maxpool2d_get_item_pattern`, and `dequantize_lowmem_maxpool2d_get_item_pattern`. This range continues the implementation of function `_register_quantization_maxpool2d`.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。初始化或更新了 `dequantize_maxpool2d_pattern`、`dequantize_lowmem_maxpool2d_pattern`、`dequantize_maxpool2d_get_item_pattern`、`dequantize_lowmem_maxpool2d_get_item_pattern` 等值。这一段延续了函数`_register_quantization_maxpool2d` 的具体实现。

### Lines 1009-1044 / 第 1009-1044 行
````python
        )


def _is_input_output_same_scale_zp(check_node):
    def fn(match):
        # Ensure all the inputs and output has same scale and zero point
        # Step 1: Check inputs/output zero point
        # Get dequant nodes at input
        dequant_nodes = filter_nodes(
            match.nodes, quantized_decomposed.dequantize_per_tensor.default
        )
        zero_points = [node.args[2] for node in dequant_nodes]
        # Get quant nodes at output
        quant_nodes = filter_nodes(
            match.nodes, quantized_decomposed.quantize_per_tensor.default
        )
        assert len(quant_nodes) == 1, "expect only 1 add node at output quant pattern"
        zero_points.append(quant_nodes[0].args[2])
        if not all(zero_point == zero_points[0] for zero_point in zero_points):
            return False

        # Step 2: Check inputs/output scale
        scales = [node.args[1] for node in dequant_nodes]
        scales.append(quant_nodes[0].args[1])
        if not all(math.isclose(scale, scales[0], rel_tol=1e-5) for scale in scales):  # type: ignore[arg-type]
            return False

        return True

    return fn


def _register_quantized_cat_lowering(
    pattern,
    computation_op,
):
````
- **EN**: Introduces function `_is_input_output_same_scale_zp`, function `fn`, function `_register_quantized_cat_lowering`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_is_input_output_same_scale_zp`、函数`fn`、函数`_register_quantized_cat_lowering`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1045-1080 / 第 1045-1080 行
````python
    @register_lowering_pattern(
        pattern,
        extra_check=_is_input_output_same_scale_zp(aten.cat.default),
    )
    def qcat(match: Match, inputs, dim, **kwargs):
        # inputs is with format: [[x1, x1_dq_dtype, x1_zp, x1_scale], ...]
        uint8_inputs = [input[0] for input in inputs]
        counters["inductor"]["qcat_matcher_count"] += 1
        counters["inductor"]["qcat_matcher_nodes"] += len(match.nodes)
        return L[computation_op](uint8_inputs, dim)

    return qcat


_raw_dequantize_per_tensor_activation_pattern = CallFunction(
    quantized_decomposed.dequantize_per_tensor.default,
    Arg(),
    Arg(),
    Arg(),
    Arg(),
    Arg(),
    Arg(),
)


def _register_quantization_cat():
    dequantize_cat_pattern = CallFunction(
        aten.cat.default,
        ListOf(_raw_dequantize_per_tensor_activation_pattern),
        KeywordArg("dim"),
    )
    _register_quantized_cat_lowering(
        generate_pattern_with_output_quant(dequantize_cat_pattern),
        aten.cat,
    )

````
- **EN**: Introduces function `qcat`, function `_register_quantization_cat`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`qcat`、函数`_register_quantization_cat`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1081-1116 / 第 1081-1116 行
````python

def _register_quantized_reshape_lowering(
    pattern,
    computation_op,
):
    @register_lowering_pattern(
        pattern,
        extra_check=_is_input_output_same_scale_zp(aten.reshape.default),
    )
    def qreshape(match: Match, *args, **kwargs):
        qx = kwargs["x"]
        shape = kwargs["shape"]
        counters["inductor"]["qreshape_matcher_count"] += 1
        counters["inductor"]["qreshape_matcher_nodes"] += len(match.nodes)
        return L[computation_op](qx, shape)

    return qreshape


def _register_quantization_reshape():
    dequantize_reshape_pattern = CallFunction(
        torch.ops.aten.reshape.default,
        get_dequantize_per_tensor_activation_pattern(),
        KeywordArg("shape"),
    )
    _register_quantized_reshape_lowering(
        generate_pattern_with_output_quant(dequantize_reshape_pattern),
        aten.reshape,
    )


def _is_valid_concat_linear_int8_woq_optimization_pattern():
    def fn(match):
        if not config.cpp.enable_concat_linear:
            return False
        assert all(k in match.kwargs for k in ("x", "w1", "w2", "w3", "scales"))
````
- **EN**: Introduces function `_register_quantized_reshape_lowering`, function `qreshape`, function `_register_quantization_reshape`, function `_is_valid_concat_linear_int8_woq_optimization_pattern`, function `fn`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_register_quantized_reshape_lowering`、函数`qreshape`、函数`_register_quantization_reshape`、函数`_is_valid_concat_linear_int8_woq_optimization_pattern`、函数`fn`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1117-1152 / 第 1117-1152 行
````python
        if not all(
            hasattr(match.kwargs[key], "meta")
            for key in ["x", "w1", "w2", "w3", "scales"]
        ):
            return False
        x = match.kwargs["x"].meta["val"]
        w1 = match.kwargs["w1"].meta["val"]
        w2 = match.kwargs["w2"].meta["val"]
        w3 = match.kwargs["w3"].meta["val"]
        scales = match.kwargs["scales"].meta["val"]
        if len(match.kwargs["scales"].meta["val"].size()) > 1:
            return False
        num_scales = match.kwargs["scales"].meta["val"].numel()
        w1_cols = match.kwargs["w1"].meta["val"].size()[0]
        w2_cols = match.kwargs["w2"].meta["val"].size()[0]
        w3_cols = match.kwargs["w3"].meta["val"].size()[0]
        return (
            # For now, we only support woq mm kernels
            # with x.type=bfloat16 and w.type=int8
            x.dtype == torch.bfloat16
            and w1.dtype == torch.int8
            and w2.dtype == torch.int8
            and w3.dtype == torch.int8
            and scales.dtype == torch.bfloat16
            and x.device.type in ("cpu", "cuda")
            and x.device == w1.device
            and w1.device == w2.device
            and w2.device == w3.device
            and x.device == scales.device
            and num_scales == w1_cols + w2_cols + w3_cols
        )

    return fn


def _is_valid_woq_optimization_pattern():
````
- **EN**: Introduces function `_is_valid_woq_optimization_pattern`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_is_valid_woq_optimization_pattern`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1153-1188 / 第 1153-1188 行
````python
    def fn(match):
        assert all(k in match.kwargs for k in ("x", "weight", "scales"))
        if not all(
            hasattr(match.kwargs[key], "meta") for key in ["x", "weight", "scales"]
        ):
            return False
        x = match.kwargs["x"].meta["val"]
        weight = match.kwargs["weight"].meta["val"]
        scales = match.kwargs["scales"].meta["val"]
        return (
            # For now, we only support woq mm kernels
            # with x.type=bfloat16 and w.type=int8
            x.dtype == torch.bfloat16
            and weight.dtype == torch.int8
            and scales.dtype == torch.bfloat16
            and x.device.type in ("cpu", "cuda", "xpu")
            and x.device == weight.device
            and x.device == scales.device
        )

    return fn


def _register_concat_linear_int8_woq_lowering(
    pattern, computation_woq, computation_reshape
):
    @register_freezing_graph_pattern(
        pattern,
        extra_check=_is_valid_concat_linear_int8_woq_optimization_pattern(),
        pass_number=4,
    )
    def woq_int8(match: Match, *args, **kwargs):
        x = kwargs["x"]
        w1 = kwargs["w1"]
        w2 = kwargs["w2"]
        w3 = kwargs["w3"]
````
- **EN**: Introduces function `fn`, function `_register_concat_linear_int8_woq_lowering`, function `woq_int8`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`fn`、函数`_register_concat_linear_int8_woq_lowering`、函数`woq_int8`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 1189-1224 / 第 1189-1224 行
````python
        scales = kwargs["scales"]
        counters["inductor"]["woq_matcher_count"] += 1
        counters["inductor"]["woq_matcher_nodes"] += len(match.nodes)
        out_features = (
            w1.meta["val"].size()[0]
            + w2.meta["val"].size()[0]
            + w3.meta["val"].size()[0]
        )
        origin_x_size = tuple(x.meta["val"].size())
        x_shape = [-1, origin_x_size[-1]]
        out_shape = list(origin_x_size[:-1] + (out_features,))
        mm_node_of_x = None
        for candidate in iter(x.users.keys()):
            if (
                candidate.target is aten.mm.default
                and list(candidate._input_nodes)[1].target is aten.cat.default
            ):
                mm_node_of_x = candidate
                break
        assert mm_node_of_x is not None, "unable to find mm node"
        _, cat_wgt_node = mm_node_of_x._input_nodes
        scaling_node = next(iter(mm_node_of_x.users.keys()))
        user_of_scaling_node = next(iter(scaling_node.users.keys()))
        # Some other pass is making some changes that entails
        # adding a node before it's used, but it can only be found when
        # lint is run. stable_topological_sort() is being run before lint,
        # so that error was not being being discovered.
        # We call stable_topological_sort here as a workaround.
        stable_topological_sort(match.graph)
        with match.graph.inserting_before(user_of_scaling_node):
            new_cat_node = match.graph.call_function(
                aten.cat.default,
                args=([w1, w2, w3], 0),
            )
            x_reshape_node = match.graph.call_function(
                computation_reshape, args=(x, x_shape)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `scales`, `out_features`, `origin_x_size`, `x_shape`, `out_shape`, `mm_node_of_x`, and `...+5`. This range continues the implementation of function `_register_concat_linear_int8_woq_lowering.woq_int8`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `scales`、`out_features`、`origin_x_size`、`x_shape`、`out_shape`、`mm_node_of_x`、`另有5项` 等值。这一段延续了函数`_register_concat_linear_int8_woq_lowering.woq_int8` 的具体实现。

### Lines 1225-1260 / 第 1225-1260 行
````python
            )
            new_woq_node = match.graph.call_function(
                computation_woq,
                args=(x_reshape_node, new_cat_node, scales),
            )
            new_woq_node.meta = copy.copy(x.meta)
            output_reshape_node = match.graph.call_function(
                computation_reshape, args=(new_woq_node, out_shape)
            )
            scaling_node.replace_all_uses_with(output_reshape_node)
            match.graph.erase_node(scaling_node)
            match.graph.erase_node(mm_node_of_x)
            match.graph.erase_node(cat_wgt_node)
            match.graph.lint()

    return woq_int8


def _register_woq_lowering(pattern, computation_woq, computation_reshape):
    @register_lowering_pattern(
        pattern,
        extra_check=_is_valid_woq_optimization_pattern(),
    )
    def woq_int8(match: Match, *args, **kwargs):
        x = kwargs["x"]
        weight = kwargs["weight"]
        scales = kwargs["scales"]
        counters["inductor"]["woq_matcher_count"] += 1
        counters["inductor"]["woq_matcher_nodes"] += len(match.nodes)
        out_features = weight.get_size()[0]
        origin_x_size = x.get_size()
        x_shape = [-1, origin_x_size[-1]]
        out_shape = origin_x_size[:-1] + [
            out_features,
        ]
        func1 = L[computation_reshape](x, x_shape)
````
- **EN**: Introduces function `_register_woq_lowering`, function `woq_int8`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_register_woq_lowering`、函数`woq_int8`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1261-1296 / 第 1261-1296 行
````python
        func2 = L[computation_woq](func1, weight, scales)
        return L[computation_reshape](func2, out_shape)

    return woq_int8


def _register_woq_mm_int8_pattern1():
    # F.linear(x, weight.to(dtype=x.dtype)) * scales
    # case of dispatching to mm, with x reshape
    _woq_pattern = CallFunction(
        aten.mul.Tensor,
        CallFunction(
            aten.reshape.default,
            CallFunction(
                aten.mm.default,
                CallFunction(aten.reshape.default, KeywordArg("x"), Arg()),
                CallFunction(
                    aten.permute.default,
                    CallFunction(
                        prims.convert_element_type.default, KeywordArg("weight"), Arg()
                    ),
                    Arg(),
                ),
            ),
            Arg(),
        ),
        KeywordArg("scales"),
    )
    _register_woq_lowering(_woq_pattern, aten._weight_int8pack_mm.default, aten.reshape)


def _register_woq_mm_int8_pattern2():
    # F.linear(x, weight.to(dtype=x.dtype)) * scales
    # case of dispatching to mm, w/o x reshape
    _woq_pattern = CallFunction(
        aten.mul.Tensor,
````
- **EN**: Introduces function `_register_woq_mm_int8_pattern1`, function `_register_woq_mm_int8_pattern2`. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_register_woq_mm_int8_pattern1`、函数`_register_woq_mm_int8_pattern2`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1297-1332 / 第 1297-1332 行
````python
        CallFunction(
            aten.reshape.default,
            CallFunction(
                aten.mm.default,
                KeywordArg("x"),
                CallFunction(
                    aten.permute.default,
                    CallFunction(
                        prims.convert_element_type.default, KeywordArg("weight"), Arg()
                    ),
                    Arg(),
                ),
            ),
            Arg(),
        ),
        KeywordArg("scales"),
    )
    _register_woq_lowering(_woq_pattern, aten._weight_int8pack_mm.default, aten.reshape)


def _register_woq_mm_int8_pattern3():
    # F.linear(x, weight.to(dtype=x.dtype)) * scales
    # case of dispatching to bmm
    _woq_pattern = CallFunction(
        aten.mul.Tensor,
        CallFunction(
            aten.bmm.default,
            CallFunction(aten.expand.default, KeywordArg("x"), Arg()),
            CallFunction(
                aten.expand.default,
                CallFunction(
                    aten.permute.default,
                    CallFunction(
                        prims.convert_element_type.default, KeywordArg("weight"), Arg()
                    ),
                    Arg(),
````
- **EN**: Introduces function `_register_woq_mm_int8_pattern3`. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `_woq_pattern`.
- **CN**: 这里定义了函数`_register_woq_mm_int8_pattern3`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `_woq_pattern` 等值。

### Lines 1333-1368 / 第 1333-1368 行
````python
                ),
                Arg(),
            ),
        ),
        KeywordArg("scales"),
    )
    _register_woq_lowering(_woq_pattern, aten._weight_int8pack_mm.default, aten.reshape)


def _register_woq_mm_int8_pattern4():
    _woq_pattern = CallFunction(
        aten.mul.Tensor,
        CallFunction(
            aten.mm.default,
            KeywordArg("x"),
            CallFunction(
                prims.convert_element_type.default,
                CallFunction(
                    aten.permute.default,
                    KeywordArg("weight"),
                    Arg(),
                ),
                Arg(),
            ),
        ),
        KeywordArg("scales"),
    )
    _register_woq_lowering(_woq_pattern, aten._weight_int8pack_mm.default, aten.reshape)


def _register_int8_woq_concat_linear_pattern():
    def _create_wgt_node(wgt_node_name: str):
        return CallFunction(
            prims.convert_element_type.default,
            CallFunction(
                aten.permute.default,
````
- **EN**: Introduces function `_register_woq_mm_int8_pattern4`, function `_register_int8_woq_concat_linear_pattern`, function `_create_wgt_node`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `_woq_pattern`.
- **CN**: 这里定义了函数`_register_woq_mm_int8_pattern4`、函数`_register_int8_woq_concat_linear_pattern`、函数`_create_wgt_node`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `_woq_pattern` 等值。

### Lines 1369-1404 / 第 1369-1404 行
````python
                KeywordArg(wgt_node_name),
                Arg(),
            ),
            Arg(),
        )

    cat_wgt = CallFunction(
        aten.cat.default, [_create_wgt_node(wgt) for wgt in ["w1", "w2", "w3"]], 1
    )

    _woq_pattern = CallFunction(
        aten.mul.Tensor,
        CallFunction(aten.mm.default, KeywordArg("x"), cat_wgt),
        KeywordArg("scales"),
    )
    _register_concat_linear_int8_woq_lowering(
        _woq_pattern, aten._weight_int8pack_mm.default, aten.reshape
    )


def _register_quantization_lowerings():
    _register_quantization_unary_lowering()
    _register_quantization_binary_lowering()
    _register_quantization_maxpool2d()
    _register_quantization_cat()
    _register_quantization_reshape()


def _register_woq_lowerings():
    _register_woq_mm_int8_pattern1()
    _register_woq_mm_int8_pattern2()
    _register_woq_mm_int8_pattern3()
    _register_woq_mm_int8_pattern4()


def _is_valid_dequant_promotion_pattern(dtype=torch.float32):
````
- **EN**: Introduces function `_register_quantization_lowerings`, function `_register_woq_lowerings`, function `_is_valid_dequant_promotion_pattern`. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `cat_wgt`, and `_woq_pattern`.
- **CN**: 这里定义了函数`_register_quantization_lowerings`、函数`_register_woq_lowerings`、函数`_is_valid_dequant_promotion_pattern`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `cat_wgt`、`_woq_pattern` 等值。

### Lines 1405-1440 / 第 1405-1440 行
````python
    def _inner(match):
        assert dtype in [torch.float32, torch.bfloat16]
        dequant_pattern_end_node = match.output_node()
        if dequant_pattern_end_node.target not in [
            quantized_decomposed.dequantize_per_tensor.default,
            quantized_decomposed.dequantize_per_tensor.tensor,
            prims.convert_element_type.default,
            aten.reshape.default,
        ]:
            return False

        if dequant_pattern_end_node.target is aten.reshape.default:
            dequant_node = (
                dequant_pattern_end_node.args[
                    0
                ]  # pattern: linear <- reshape <- dequant
                if dtype == torch.float32
                else dequant_pattern_end_node.args[0].args[
                    0
                ]  # pattern: linear <- reshape <- to_bf16 <- dequant
            )
        else:
            dequant_node = (
                dequant_pattern_end_node  # pattern: linear <- dequant
                if dtype == torch.float32
                else dequant_pattern_end_node.args[
                    0
                ]  # pattern: linear <- to_bf16 <- dequant
            )

        if (
            dequant_node.target
            in [
                quantized_decomposed.dequantize_per_tensor.default,
                quantized_decomposed.dequantize_per_tensor.tensor,
            ]
````
- **EN**: Introduces function `_inner`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_inner`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1441-1476 / 第 1441-1476 行
````python
            and len(list(dequant_pattern_end_node.users)) > 1
        ):
            # If dequant pattern has more than 1 users, then do dequant promoted
            return True
        return False

    return _inner


def _register_dequant_promotion_pass(pattern, pass_number, dtype=torch.float32):
    @register_freezing_graph_pattern(
        pattern,
        extra_check=_is_valid_dequant_promotion_pattern(dtype),
        pass_number=pass_number,
    )
    def dequant_promotion(match: Match, *args, **kwargs):
        # Dequant_promotion will transform
        # graph 1:
        #            quant
        #      + - - - | - - - +
        #      |    dequant    |
        #      |    /     \    |
        #      |  node1  node2 |
        #      + - | - - - | - +
        #        quant   quant
        # into:
        # graph 2:
        #            quant
        #      + - - / - \ - - +
        #      |dequant dequant|
        #      |    |      |   |
        #      | node1 node2   |
        #      + - | - - - | - +
        #        quant   quant
        # In graph 1, the dequant node is shared by node1 and node2,
        # as a result, neither node1 nor node2 could form an int8
````
- **EN**: Introduces function `_register_dequant_promotion_pass`, function `dequant_promotion`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`_register_dequant_promotion_pass`、函数`dequant_promotion`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 1477-1512 / 第 1477-1512 行
````python
        # fusion pattern.
        # After this transformation, the graph 2 could hit the int8
        # fusion pattern: dequant-node-quant, respectively for
        # node1 and node2.
        assert dtype in [torch.float32, torch.bfloat16]

        def clone_to_new_node(graph, source_node, user_node):
            # Clone the source_node to a new node
            # Replace user_node's input from source_node to new_node
            assert source_node.op == "call_function", (
                "clone_to_new_node only support node.op call_function"
            )
            with graph.inserting_before(user_node):
                new_node = graph.call_function(
                    source_node.target,
                    args=source_node.args,
                    kwargs=source_node.kwargs,
                )
                new_node.meta = copy.copy(source_node.meta)
                user_node.replace_input_with(source_node, new_node)
            return new_node

        # Find the start node and end node of a dequant pattern
        # * End node should be the match.output_node()
        # * Start node should be the node of dequantize_per_tensor
        dequant_pattern_end_node = match.output_node()
        assert dequant_pattern_end_node.target in [
            quantized_decomposed.dequantize_per_tensor.default,
            quantized_decomposed.dequantize_per_tensor.tensor,
            prims.convert_element_type.default,
            aten.reshape.default,
        ]

        # For a dequant pattern, we should expect see the node list as:
        # * OPT(aten.reshape.default)
        # * OPT(prims.convert_element_type.default) (to_bf16)
````
- **EN**: Introduces function `clone_to_new_node`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`clone_to_new_node`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1513-1548 / 第 1513-1548 行
````python
        # * dequantize_per_tensor
        def _find_first_node_in_dequant_pattern(_node):
            if _node.target in [
                quantized_decomposed.dequantize_per_tensor.default,
                quantized_decomposed.dequantize_per_tensor.tensor,
            ]:
                # For a dequant pattern, we expect the start node is a dequantize_per_tensor node
                return _node
            else:
                assert len(_node.args) >= 1, (
                    "In in dequant pattern, each node should have more than 1 arg."
                )
                return _find_first_node_in_dequant_pattern(_node.args[0])

        dequant_pattern_start_node = _find_first_node_in_dequant_pattern(
            dequant_pattern_end_node
        )

        assert dequant_pattern_start_node.target in [
            quantized_decomposed.dequantize_per_tensor.default,
            quantized_decomposed.dequantize_per_tensor.tensor,
        ]

        # Clone the dequant pattern for each user node
        graph = match.graph
        user_node_list = list(dequant_pattern_end_node.users)
        for user_node in user_node_list[1:]:
            _source_node = dequant_pattern_end_node
            _user_node = user_node
            # pyrefly: ignore [bad-assignment, non-convergent-recursion]
            while _source_node != dequant_pattern_start_node.args[0]:
                _user_node = clone_to_new_node(graph, _source_node, _user_node)
                _source_node = _source_node.args[0]  # type: ignore[assignment]

        counters["inductor"]["dequant_promotion_matcher_count"] += 1
        counters["inductor"]["dequant_promotion_matcher_nodes"] += len(match.nodes)
````
- **EN**: Introduces function `_find_first_node_in_dequant_pattern`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `dequant_pattern_start_node`, `graph`, `user_node_list`, `_source_node`, and `_user_node`.
- **CN**: 这里定义了函数`_find_first_node_in_dequant_pattern`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`dequant_pattern_start_node`、`graph`、`user_node_list`、`_source_node`、`_user_node` 等值。

### Lines 1549-1584 / 第 1549-1584 行
````python


def _is_valid_dequant_conv_pattern(dtype, with_dtype_convert):
    def _inner(match):
        # Here we do some further check to ensure:
        # 1. It's a conv2d node with dim of 4, since we only support lowering of conv2d now.
        # 2. The dequant pattern has only 1 user of conv2d node.
        # If these conditions don't meet, we will not
        # insert weight prepack node into the matched pattern.
        conv_node = match.output_node()
        assert conv_node.target is aten.convolution.default
        input_meta_value = conv_node.args[0].meta.get("val")
        weight_meta_value = conv_node.args[1].meta.get("val")
        for meta_value in [input_meta_value, weight_meta_value]:
            if (
                meta_value is None
                or (meta_value.device.type != "cpu" and meta_value.device.type != "xpu")
                or meta_value.dim() not in [3, 4]
            ):
                # Only support conv1d/2d now
                return False

        assert dtype in [torch.float32, torch.bfloat16]

        if not with_dtype_convert:
            dequant_node = conv_node.args[0]
        else:
            convert_to_bf16 = conv_node.args[0]
            dequant_node = convert_to_bf16.args[0]

        if len(list(dequant_node.users)) != 1:
            # Ensure the dequant pattern only has 1 user
            # since we will delete the dequant pattern here
            return False
        return True

````
- **EN**: Introduces function `_is_valid_dequant_conv_pattern`, function `_inner`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_is_valid_dequant_conv_pattern`、函数`_inner`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1585-1620 / 第 1585-1620 行
````python
    return _inner


def _register_qconv_weight_prepack_pass(
    pattern, pass_number, dtype=torch.float32, with_dtype_convert=False
):
    @register_freezing_graph_pattern(
        pattern,
        extra_check=_is_valid_dequant_conv_pattern(dtype, with_dtype_convert),
        pass_number=pass_number,
    )
    def qconv_weight_prepack(match: Match, *args, **kwargs):
        """
        Match the pattern:
        int8 activation
          |
        dequant_per_tensor
          |
        Conv2d <- optional(aten.clone.default) <- dequant_per_channel <- int8_weight

        Insert weight prepack node and change the pattern to:
        int8 activation
          |
        onednn.qconv_pointwise <- onednn.qconv_prepack <- int8_weight
        """
        assert dtype in [torch.float32, torch.bfloat16]
        conv_node = match.output_node()
        assert conv_node.target is aten.convolution.default
        if not with_dtype_convert:
            dequant_node = conv_node.args[0]
        else:
            convert_to_bf16 = conv_node.args[0]
            dequant_node = convert_to_bf16.args[0]  # type: ignore[union-attr]
        has_clone_to_channel_last_node_in_pattern = (
            conv_node.args[1].target is aten.clone.default  # type: ignore[union-attr]
        )
````
- **EN**: Introduces function `_register_qconv_weight_prepack_pass`, function `qconv_weight_prepack`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`_register_qconv_weight_prepack_pass`、函数`qconv_weight_prepack`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 1621-1656 / 第 1621-1656 行
````python
        clone_node = (
            conv_node.args[1] if has_clone_to_channel_last_node_in_pattern else None
        )

        if dtype == torch.float32:
            dequant_per_channel = (
                clone_node.args[0]  # type: ignore[union-attr]
                if has_clone_to_channel_last_node_in_pattern
                else conv_node.args[1]
            )
        else:
            weight_to_bf16_node = (
                clone_node.args[0]  # type: ignore[union-attr]
                if has_clone_to_channel_last_node_in_pattern
                else conv_node.args[1]
            )
            dequant_per_channel = weight_to_bf16_node.args[0]  # type: ignore[union-attr]

        assert (
            dequant_per_channel.target  # type: ignore[union-attr]
            is quantized_decomposed.dequantize_per_channel.default
        )

        # Activation QParams
        qx, x_zp, x_scale = (
            kwargs["x"],
            kwargs["x_zp"],
            kwargs["x_scale"],
        )

        # Weight QParams
        qw, w_scale, w_zp = (
            kwargs["q_weight"],
            kwargs["w_scale"],
            kwargs["w_zp"],
        )
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `clone_node`, `dequant_per_channel`, `else`, and `weight_to_bf16_node`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `clone_node`、`dequant_per_channel`、`else`、`weight_to_bf16_node` 等值。

### Lines 1657-1692 / 第 1657-1692 行
````python

        # Conv Params
        bias, stride, padding, dilation, groups = (
            kwargs["b"],
            kwargs["stride"],
            kwargs["padding"],
            kwargs["dilation"],
            kwargs["groups"],
        )

        x_shape = qx.meta.get("tensor_meta").shape
        if has_free_symbols(x_shape):
            # For dynamic shape case, we can't get activation shape ahead of runtime.
            x_shape = None
        graph = match.graph
        with graph.inserting_before(conv_node):
            # Insert weight prepack node and the QConv node
            packed_weight_inputs = (
                qw,
                w_scale,
                x_scale,
                x_zp,
                stride,
                padding,
                dilation,
                groups,
                x_shape,
            )
            packed_weight_op = torch.ops.onednn.qconv_prepack
            prepack_weight_node = graph.call_function(
                packed_weight_op, args=packed_weight_inputs
            )

            new_args: tuple[Any, ...] = (
                qx,
                x_scale,
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `x_shape`, `graph`, `packed_weight_inputs`, `packed_weight_op`, `prepack_weight_node`, and `new_args`. This range continues the implementation of function `_register_qconv_weight_prepack_pass.qconv_weight_prepack`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `x_shape`、`graph`、`packed_weight_inputs`、`packed_weight_op`、`prepack_weight_node`、`new_args` 等值。这一段延续了函数`_register_qconv_weight_prepack_pass.qconv_weight_prepack` 的具体实现。

### Lines 1693-1728 / 第 1693-1728 行
````python
                x_zp,
                prepack_weight_node,
                w_scale,
                w_zp,
                bias,
                stride,
                padding,
                dilation,
                groups,
                1.0,  # output_scale
                0,  # output_zero_point
                dtype,  # output_dtype
                "none",  # attr
                [],  # scalars
                "",  # algorithm
            )
            new_conv_node = graph.call_function(
                torch.ops.onednn.qconv_pointwise.default, args=new_args
            )
            conv_node.replace_all_uses_with(new_conv_node)
            new_conv_node.meta.update(conv_node.meta)

            # Erase the original conv node
            graph.erase_node(conv_node)
            # Erase the dequant pattern
            if with_dtype_convert:
                graph.erase_node(convert_to_bf16)  # type: ignore[possibly-undefined, arg-type]
            graph.erase_node(dequant_node)  # type: ignore[arg-type]
            # Erase the dequant per channel pattern
            if clone_node is not None:
                graph.erase_node(clone_node)  # type: ignore[arg-type]
            if dtype == torch.bfloat16:
                graph.erase_node(weight_to_bf16_node)  # type: ignore[possibly-undefined, arg-type]
            graph.erase_node(dequant_per_channel)  # type: ignore[arg-type]
            counters["inductor"]["qconv_weight_prepack_matcher_count"] += 1
            counters["inductor"]["qconv_weight_prepack_matcher_nodes"] += len(
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `new_conv_node`. This range continues the implementation of function `_register_qconv_weight_prepack_pass.qconv_weight_prepack`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `new_conv_node` 等值。这一段延续了函数`_register_qconv_weight_prepack_pass.qconv_weight_prepack` 的具体实现。

### Lines 1729-1764 / 第 1729-1764 行
````python
                match.nodes
            )


def _generate_dequant_convolution_node_pattern(
    _dequant_per_channel_pattern, dtype=torch.float32, with_dtype_convert=False
):
    assert dtype in [torch.float32, torch.bfloat16]
    dequant_convolution_node_pattern = CallFunction(
        aten.convolution.default,
        _may_generate_pattern_with_dtype_convert(
            get_dequantize_per_tensor_activation_pattern(),
            KeywordArg("autocast_act_dtype"),
            with_dtype_convert,
        ),
        _dequant_per_channel_pattern,
        KeywordArg("b"),
        KeywordArg("stride"),
        KeywordArg("padding"),
        KeywordArg("dilation"),
        KeywordArg("is_transposed"),
        KeywordArg("out_padding"),
        KeywordArg("groups"),
    )
    return dequant_convolution_node_pattern


def _generate_qconv_weight_prepack_patterns(
    dtype=torch.float32, with_dtype_convert=False
):
    assert dtype in [torch.float32, torch.bfloat16]
    return (
        _generate_dequant_convolution_node_pattern(
            dequantize_per_channel_weight_pattern
            if dtype == torch.float32
            else dequantize_per_channel_to_bf16_weight_pattern,
````
- **EN**: Introduces function `_generate_dequant_convolution_node_pattern`, function `_generate_qconv_weight_prepack_patterns`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_generate_dequant_convolution_node_pattern`、函数`_generate_qconv_weight_prepack_patterns`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1765-1800 / 第 1765-1800 行
````python
            dtype,
            with_dtype_convert,
        ),
        # There is another pattern due to the pass of convert_conv_weights_to_channels_last
        # https://github.com/pytorch/pytorch/blob/07107919297db3f8ab37f11c12666b6d6d5f692e/torch/_inductor/freezing.py#L338-L362.
        # Depend on some heuristics, it may or may not insert to(channel_last) node
        # between convolution and dequant_per_channel node
        _generate_dequant_convolution_node_pattern(
            dequantize_per_channel_clone_weight_pattern
            if dtype == torch.float32
            else dequantize_per_channel_to_bf16_clone_weight_pattern,
            dtype,
            with_dtype_convert,
        ),
    )


def _get_linear_node(match, input_dim_exceeds_two, input_contiguous):
    output_reshape_node = None
    if input_dim_exceeds_two:
        if input_contiguous:
            output_reshape_node = match.output_node()
            assert output_reshape_node.target is aten.reshape.default
            linear_node = output_reshape_node.args[0]
        else:
            linear_nodes = filter_nodes(match.nodes, aten.bmm.default)
            assert len(linear_nodes) == 1
            linear_node = linear_nodes[0]
    else:
        linear_node = match.output_node()

    assert linear_node.target in (
        aten.addmm.default,
        aten.mm.default,
        aten.bmm.default,
    )
````
- **EN**: Introduces function `_get_linear_node`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_get_linear_node`。保留了指向设计说明、规范或厂商数据手册的注释引用。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1801-1836 / 第 1801-1836 行
````python
    return linear_node, output_reshape_node


def _get_linear_dq_node(
    linear_node,
    input_index,
    input_dim_exceeds_two,
    input_contiguous,
    with_dtype_convert,
):
    act_reshape_node = None
    activation_to_bf16_node = None
    act_expand_node = None
    if input_dim_exceeds_two:
        if input_contiguous:
            act_reshape_node = linear_node.args[input_index]
            assert act_reshape_node.target is aten.reshape.default
            if not with_dtype_convert:
                # pattern: linear -> reshape -> dequant
                dequant_node = act_reshape_node.args[0]
            else:
                # pattern: linear -> reshape -> to_bf16 -> dequant
                activation_to_bf16_node = act_reshape_node.args[0]
                dequant_node = activation_to_bf16_node.args[0]
        else:
            # bmm pattern decomposed from linear when input dim exceeds 2 and not contiguous
            act_expand_node = linear_node.args[input_index]
            assert act_expand_node.target is aten.expand.default
            if not with_dtype_convert:
                dequant_node = act_expand_node.args[0]
            else:
                activation_to_bf16_node = act_expand_node.args[0]
                dequant_node = activation_to_bf16_node.args[0]
    else:
        if not with_dtype_convert:
            # pattern: linear -> dequant
````
- **EN**: Introduces function `_get_linear_dq_node`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `act_reshape_node`, `activation_to_bf16_node`, `act_expand_node`, `dequant_node`, and `else`.
- **CN**: 这里定义了函数`_get_linear_dq_node`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `act_reshape_node`、`activation_to_bf16_node`、`act_expand_node`、`dequant_node`、`else` 等值。

### Lines 1837-1872 / 第 1837-1872 行
````python
            dequant_node = linear_node.args[input_index]
        else:
            # pattern: linear -> to_bf16 -> dequant
            activation_to_bf16_node = linear_node.args[input_index]
            dequant_node = activation_to_bf16_node.args[0]
    return dequant_node, act_reshape_node, activation_to_bf16_node, act_expand_node


def _is_valid_dequant_linear_pattern(
    dtype, input_dim_exceeds_two, input_contiguous, with_dtype_convert
):
    def _inner(match):
        # Check dequant pattern has only 1 user.
        (
            linear_node,
            _,
        ) = _get_linear_node(match, input_dim_exceeds_two, input_contiguous)

        input_index = 1 if linear_node.target is aten.addmm.default else 0
        assert dtype in [torch.float32, torch.bfloat16]
        (
            dequant_node,
            _,
            _,
            _,
        ) = _get_linear_dq_node(
            linear_node,
            input_index,
            input_dim_exceeds_two,
            input_contiguous,
            with_dtype_convert,
        )

        assert dequant_node.target in [
            quantized_decomposed.dequantize_per_tensor.default,
            quantized_decomposed.dequantize_per_tensor.tensor,
````
- **EN**: Introduces function `_is_valid_dequant_linear_pattern`, function `_inner`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_is_valid_dequant_linear_pattern`、函数`_inner`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1873-1908 / 第 1873-1908 行
````python
        ]

        if len(list(dequant_node.users)) != 1:
            # Ensure the dequant pattern only has 1 user
            # since we will delete the dequant pattern here
            return False

        # Extra check for bmm pattern
        if input_dim_exceeds_two and not input_contiguous:
            # Check for act
            # Act expand size should be exactly same as act size
            act_expand_size = match.kwargs["act_expand_size"]
            act_node = match.kwargs["x"]
            if not (
                hasattr(act_node, "meta")
                and isinstance(act_node.meta.get("val", None), torch.Tensor)
                and (act_node.meta["val"].size() == torch.Size(act_expand_size))
            ):
                return False

            # Check for wgt
            # wgt permute dims should be [1, 0]
            wgt_permute_dims = match.kwargs["permute_axes"]
            if wgt_permute_dims != [1, 0]:
                return False

            # Check below wgt size items:
            # wgt before expand should with dim 2
            # Expand size should with dim 3
            # Expand size[0] should same as act size[0]
            # Expand size[1] should same as wgt size[1]
            # Expand size[2] should same as wgt size[0]
            qweight_node = match.kwargs["q_weight"]
            wgt_expand_size = match.kwargs["wgt_expand_size"]
            if not (
                hasattr(qweight_node, "meta")
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `act_expand_size`, `act_node`, `wgt_permute_dims`, `qweight_node`, and `wgt_expand_size`. This range continues the implementation of function `_is_valid_dequant_linear_pattern._inner`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `act_expand_size`、`act_node`、`wgt_permute_dims`、`qweight_node`、`wgt_expand_size` 等值。这一段延续了函数`_is_valid_dequant_linear_pattern._inner` 的具体实现。

### Lines 1909-1944 / 第 1909-1944 行
````python
                and isinstance(qweight_node.meta.get("val", None), torch.Tensor)
                and len(qweight_node.meta["val"].size()) == 2
                and len(wgt_expand_size) == 3
                and wgt_expand_size[0] == act_node.meta["val"].size()[0]
                and wgt_expand_size[1] == qweight_node.meta["val"].size()[1]
                and wgt_expand_size[2] == qweight_node.meta["val"].size()[0]
            ):
                return False

        return True

    return _inner


def _register_qlinear_weight_prepack_pass(
    pattern,
    pass_number,
    dtype=torch.float32,
    input_dim_exceeds_two=False,
    input_contiguous=True,
    with_dtype_convert=False,
):
    @register_freezing_graph_pattern(
        pattern,
        extra_check=_is_valid_dequant_linear_pattern(
            dtype, input_dim_exceeds_two, input_contiguous, with_dtype_convert
        ),
        pass_number=pass_number,
    )
    def qlinear_weight_prepack(match: Match, *args, **kwargs):
        """
        Match the pattern:
        int8 activation
          |
        dequant_per_tensor
          |
````
- **EN**: Introduces function `_register_qlinear_weight_prepack_pass`, function `qlinear_weight_prepack`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`_register_qlinear_weight_prepack_pass`、函数`qlinear_weight_prepack`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 1945-1980 / 第 1945-1980 行
````python
        mm/addmm <- t <- dequant_per_channel <- int8_weight

        Insert weight prepack node and change the pattern to:
        int8 activation
          |
        onednn.qlinear_pointwise <- onednn.qlinear_prepack <- int8_weight
        """
        assert dtype in [torch.float32, torch.bfloat16]
        (
            linear_node,
            output_reshape_node,
        ) = _get_linear_node(match, input_dim_exceeds_two, input_contiguous)
        input_index = 1 if linear_node.target is aten.addmm.default else 0
        weight_index = input_index + 1

        (
            dequant_node,
            act_reshape_node,
            activation_to_bf16_node,
            act_expand_node,
        ) = _get_linear_dq_node(
            linear_node,
            input_index,
            input_dim_exceeds_two,
            input_contiguous,
            with_dtype_convert,
        )

        if input_dim_exceeds_two and not input_contiguous:
            wgt_expand_node = linear_node.args[weight_index]
            assert wgt_expand_node.target is aten.expand.default
            t_node = wgt_expand_node.args[0]
        else:
            t_node = linear_node.args[weight_index]

        if dtype == torch.float32:
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_index`, `weight_index`, `wgt_expand_node`, `t_node`, and `else`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `input_index`、`weight_index`、`wgt_expand_node`、`t_node`、`else` 等值。

### Lines 1981-2016 / 第 1981-2016 行
````python
            dequant_per_channel = t_node.args[0]
        else:
            weight_to_bf16_node = t_node.args[0]
            dequant_per_channel = weight_to_bf16_node.args[0]
        assert (
            dequant_per_channel.target
            is quantized_decomposed.dequantize_per_channel.default
        )

        # Activation QParams
        qx, x_zp, x_scale = (
            kwargs["x"],
            kwargs["x_zp"],
            kwargs["x_scale"],
        )

        # Weight QParams
        qw, w_scale, w_zp = (
            kwargs["q_weight"],
            kwargs["w_scale"],
            kwargs["w_zp"],
        )

        # Params
        bias = kwargs.get("b")

        x_shape = qx.meta.get("tensor_meta").shape
        if has_free_symbols(x_shape):
            # For dynamic shape case, we can't get activation shape ahead of runtime.
            x_shape = None
        graph = match.graph
        with graph.inserting_before(linear_node):
            # Insert weight prepack node and the qlinear node
            packed_weight_inputs = (
                qw,
                x_shape,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `dequant_per_channel`, `else`, `weight_to_bf16_node`, `bias`, `x_shape`, `graph`, and `...+1`. This range continues the implementation of function `_register_qlinear_weight_prepack_pass.qlinear_weight_prepack`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `dequant_per_channel`、`else`、`weight_to_bf16_node`、`bias`、`x_shape`、`graph`、`另有1项` 等值。这一段延续了函数`_register_qlinear_weight_prepack_pass.qlinear_weight_prepack` 的具体实现。

### Lines 2017-2052 / 第 2017-2052 行
````python
            )
            packed_weight_op = torch.ops.onednn.qlinear_prepack
            prepack_weight_node = graph.call_function(
                packed_weight_op, args=packed_weight_inputs
            )

            new_args: tuple[Any, ...] = (
                qx,
                x_scale,
                x_zp,
                prepack_weight_node,
                w_scale,
                w_zp,
                bias,
                1.0,  # output_scale
                0,  # output_zero_point
                dtype,  # output_dtype
                "none",  # post op name
                [],  # post op args
                "",  # post op algorithm
            )
            Node = torch.fx.node.Node
            if isinstance(x_scale, Node) and isinstance(x_zp, Node):
                new_linear_node = graph.call_function(
                    torch.ops.onednn.qlinear_pointwise.tensor, args=new_args
                )
            else:
                new_linear_node = graph.call_function(
                    torch.ops.onednn.qlinear_pointwise.default, args=new_args
                )
            if input_dim_exceeds_two:
                if input_contiguous:
                    output_reshape_node.replace_all_uses_with(new_linear_node)
                    new_linear_node.meta.update(output_reshape_node.meta)
                else:
                    if bias:
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `packed_weight_op`, `prepack_weight_node`, `new_args`, `Node`, `new_linear_node`, and `else`. This range continues the implementation of function `_register_qlinear_weight_prepack_pass.qlinear_weight_prepack`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `packed_weight_op`、`prepack_weight_node`、`new_args`、`Node`、`new_linear_node`、`else` 等值。这一段延续了函数`_register_qlinear_weight_prepack_pass.qlinear_weight_prepack` 的具体实现。

### Lines 2053-2088 / 第 2053-2088 行
````python
                        output_add_node_for_bias = match.output_node()
                        assert output_add_node_for_bias.target is aten.add.Tensor
                        output_add_node_for_bias.replace_all_uses_with(new_linear_node)
                        new_linear_node.meta.update(output_add_node_for_bias.meta)
                    else:
                        linear_node.replace_all_uses_with(new_linear_node)
                        new_linear_node.meta.update(linear_node.meta)
            else:
                linear_node.replace_all_uses_with(new_linear_node)
                new_linear_node.meta.update(linear_node.meta)

            # Erase the original linear node
            if input_dim_exceeds_two:
                if input_contiguous:
                    graph.erase_node(output_reshape_node)
                elif not input_contiguous and bias:
                    graph.erase_node(output_add_node_for_bias)  # type: ignore[possibly-undefined]
            graph.erase_node(linear_node)
            if input_dim_exceeds_two:
                if input_contiguous:
                    graph.erase_node(act_reshape_node)
                else:
                    graph.erase_node(act_expand_node)
                    graph.erase_node(wgt_expand_node)  # type: ignore[possibly-undefined]
            if with_dtype_convert:
                graph.erase_node(activation_to_bf16_node)
            # Erase the dequant pattern
            graph.erase_node(dequant_node)
            # Erase the dequant per channel pattern
            graph.erase_node(t_node)
            if dtype == torch.bfloat16:
                graph.erase_node(weight_to_bf16_node)  # type: ignore[possibly-undefined]
            graph.erase_node(dequant_per_channel)

            counters["inductor"]["qlinear_weight_prepack_matcher_count"] += 1
            counters["inductor"]["qlinear_weight_prepack_matcher_nodes"] += len(
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `output_add_node_for_bias`, and `else`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `output_add_node_for_bias`、`else` 等值。

### Lines 2089-2124 / 第 2089-2124 行
````python
                match.nodes
            )


def _generate_dequant_linear_node_pattern(
    _dequant_per_channel_pattern,
    dtype=torch.float32,
    input_dim_exceeds_two=False,
    is_tensor_overload=False,
    with_dtype_convert=False,
):
    assert dtype in [torch.float32, torch.bfloat16]
    t_pattern = _generate_linear_t_pattern(_dequant_per_channel_pattern, dtype)
    dequant_linear_bias_pattern = _may_generate_pattern_with_reshape(
        CallFunction(
            aten.addmm.default,
            KeywordArg("b"),
            _may_generate_pattern_with_reshape(
                _may_generate_pattern_with_dtype_convert(
                    get_dequantize_per_tensor_activation_pattern(is_tensor_overload),
                    KeywordArg("autocast_act_dtype"),
                    with_dtype_convert,
                ),
                KeywordArg("act_reshape_size"),
                input_dim_exceeds_two,
            ),
            t_pattern,
        ),
        KeywordArg("output_reshape_size"),
        input_dim_exceeds_two,
    )
    dequant_linear_no_bias_pattern = _may_generate_pattern_with_reshape(
        CallFunction(
            aten.mm.default,
            _may_generate_pattern_with_reshape(
                _may_generate_pattern_with_dtype_convert(
````
- **EN**: Introduces function `_generate_dequant_linear_node_pattern`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `dtype`, `input_dim_exceeds_two`, `is_tensor_overload`, `with_dtype_convert`, `t_pattern`, `dequant_linear_bias_pattern`, and `...+1`.
- **CN**: 这里定义了函数`_generate_dequant_linear_node_pattern`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `dtype`、`input_dim_exceeds_two`、`is_tensor_overload`、`with_dtype_convert`、`t_pattern`、`dequant_linear_bias_pattern`、`另有1项` 等值。

### Lines 2125-2160 / 第 2125-2160 行
````python
                    get_dequantize_per_tensor_activation_pattern(is_tensor_overload),
                    KeywordArg("autocast_act_dtype"),
                    with_dtype_convert,
                ),
                KeywordArg("act_reshape_size"),
                input_dim_exceeds_two,
            ),
            t_pattern,
        ),
        KeywordArg("output_reshape_size"),
        input_dim_exceeds_two,
    )
    return dequant_linear_bias_pattern, dequant_linear_no_bias_pattern


def _generate_dequant_bmm_node_pattern(
    _dequant_per_channel_pattern,
    dtype=torch.float32,
    with_bias=False,
    is_tensor_overload=False,
    with_dtype_convert=False,
):
    # When activation of linear dim exceed 2 and not contiguous
    t_pattern = _generate_linear_t_pattern(_dequant_per_channel_pattern, dtype)

    assert dtype in [torch.float32, torch.bfloat16]
    dequant_bmm_pattern = CallFunction(
        aten.bmm.default,
        CallFunction(
            aten.expand.default,
            _may_generate_pattern_with_dtype_convert(
                get_dequantize_per_tensor_activation_pattern(is_tensor_overload),
                KeywordArg("autocast_act_dtype"),
                with_dtype_convert,
            ),
            KeywordArg("act_expand_size"),
````
- **EN**: Introduces function `_generate_dequant_bmm_node_pattern`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `dtype`, `with_bias`, `is_tensor_overload`, `with_dtype_convert`, `t_pattern`, and `dequant_bmm_pattern`.
- **CN**: 这里定义了函数`_generate_dequant_bmm_node_pattern`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `dtype`、`with_bias`、`is_tensor_overload`、`with_dtype_convert`、`t_pattern`、`dequant_bmm_pattern` 等值。

### Lines 2161-2196 / 第 2161-2196 行
````python
        ),
        CallFunction(
            aten.expand.default,
            t_pattern,
            KeywordArg("wgt_expand_size"),
        ),
    )

    def _generate_pattern_with_output_add(_dequant_bmm_pattern, _with_bias):
        if _with_bias:
            return CallFunction(
                aten.add.Tensor,
                _dequant_bmm_pattern,
                KeywordArg("b"),
            )
        else:
            return _dequant_bmm_pattern

    return _generate_pattern_with_output_add(dequant_bmm_pattern, with_bias)


def _generate_qlinear_weight_prepack_patterns(
    dtype=torch.float32,
    input_dim_exceeds_two=False,
    input_contiguous=True,
    with_bias=False,
    is_tensor_overload=False,
    with_dtype_convert=False,
):
    if input_dim_exceeds_two and not input_contiguous:
        return _generate_dequant_bmm_node_pattern(
            dequantize_per_channel_weight_pattern,
            dtype,
            with_bias,
            is_tensor_overload,
            with_dtype_convert,
````
- **EN**: Introduces function `_generate_pattern_with_output_add`, function `_generate_qlinear_weight_prepack_patterns`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_generate_pattern_with_output_add`、函数`_generate_qlinear_weight_prepack_patterns`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 2197-2232 / 第 2197-2232 行
````python
        )
    else:
        return _generate_dequant_linear_node_pattern(
            dequantize_per_channel_weight_pattern,
            dtype,
            input_dim_exceeds_two,
            is_tensor_overload,
            with_dtype_convert,
        )


def _generate_linear_dynamic_fp16_pattern(
    _dequant_weight_pattern,
    input_dim_exceeds_two=False,
    input_contiguous=True,
    relu_fused=False,
):
    dtype = torch.float32
    t_pattern = _generate_linear_t_pattern(_dequant_weight_pattern, dtype)

    if input_dim_exceeds_two and not input_contiguous:
        # pattern is
        #                   x -> expand -> bmm (-> add) (-> relu)
        # w -> dequant -> permute -> expand /
        pattern_no_bias = CallFunction(
            aten.bmm.default,
            CallFunction(
                aten.expand.default,
                KeywordArg("x"),
                KeywordArg("act_expand_size"),
            ),
            CallFunction(
                aten.expand.default,
                t_pattern,
                KeywordArg("wgt_expand_size"),
            ),
````
- **EN**: Introduces function `_generate_linear_dynamic_fp16_pattern`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_generate_linear_dynamic_fp16_pattern`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 2233-2268 / 第 2233-2268 行
````python
        )
        pattern_with_bias = CallFunction(
            aten.add.Tensor,
            pattern_no_bias,
            KeywordArg("b"),
        )
        if relu_fused:
            pattern_with_bias = CallFunction(aten.relu.default, pattern_with_bias)
            pattern_no_bias = CallFunction(aten.relu.default, pattern_no_bias)
        return pattern_with_bias, pattern_no_bias

    x_pattern_with_reshape = _may_generate_pattern_with_reshape(
        KeywordArg("x"),
        KeywordArg("act_reshape_size"),
        input_dim_exceeds_two,
    )
    dequant_linear_bias_pattern = generate_pattern_with_unary(
        _may_generate_pattern_with_reshape(
            CallFunction(
                aten.addmm.default,
                KeywordArg("b"),
                x_pattern_with_reshape,
                t_pattern,
            ),
            KeywordArg("output_reshape_size"),
            input_dim_exceeds_two,
        ),
        aten.relu.default if relu_fused else None,
    )
    dequant_linear_no_bias_pattern = generate_pattern_with_unary(
        _may_generate_pattern_with_reshape(
            CallFunction(
                aten.mm.default,
                x_pattern_with_reshape,
                t_pattern,
            ),
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `pattern_with_bias`, `pattern_no_bias`, `x_pattern_with_reshape`, `dequant_linear_bias_pattern`, and `dequant_linear_no_bias_pattern`. This range continues the implementation of function `_generate_linear_dynamic_fp16_pattern`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `pattern_with_bias`、`pattern_no_bias`、`x_pattern_with_reshape`、`dequant_linear_bias_pattern`、`dequant_linear_no_bias_pattern` 等值。这一段延续了函数`_generate_linear_dynamic_fp16_pattern` 的具体实现。

### Lines 2269-2304 / 第 2269-2304 行
````python
            KeywordArg("output_reshape_size"),
            input_dim_exceeds_two,
        ),
        aten.relu.default if relu_fused else None,
    )
    return dequant_linear_bias_pattern, dequant_linear_no_bias_pattern


def _register_dequant_promotion():
    dequant_pattern_cases = itertools.product(
        [torch.float32, torch.bfloat16], [True, False], [True, False]
    )
    for dtype, input_dim_exceeds_two, is_tensor_overload in dequant_pattern_cases:
        # 4 dequantization patterns will be matched based on the dtype and input dimension size.
        # Case 1: int8-mixed-fp32, input dim size is 2
        # Case 2: int8-mixed-fp32, input dim size exceeds 2
        # Case 3: int8-mixed-bf16, input dim size is 2
        # Case 4: int8-mixed-bf16, input dim size exceeds 2
        #           quant
        #   + - - - - | - - - - +
        #   |      dequant      |
        #   |         |         |
        #   |    OPT(to_bf16)   |
        #   |         |         |
        #   |    OPT(reshape)   |
        #   |      /     \      |
        #   |    node1  node2   |
        #   + - - | - - - | - - +
        #  OPT(reshape) OPT(reshape)
        #   + - - | - - - | - - +
        #  OPT(to_fp32) OPT(to_fp32)
        #   + - - | - - - | - - +
        #       quant   quant
        _register_dequant_promotion_pass(
            _may_generate_pattern_with_reshape(
                _may_generate_pattern_with_dtype_convert(
````
- **EN**: Introduces function `_register_dequant_promotion`. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_register_dequant_promotion`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 2305-2340 / 第 2305-2340 行
````python
                    get_dequantize_per_tensor_activation_pattern(
                        is_tensor_overload=is_tensor_overload
                    ),
                    KeywordArg("autocast_act_dtype"),
                    dtype == torch.bfloat16,
                ),
                KeywordArg("act_reshape_size"),
                with_reshape=input_dim_exceeds_two,
            ),
            pass_number=0,
            dtype=dtype,
        )  # pass_number=0 to run before weight prepack


def _register_qconv_weight_prepack():
    for dtype, with_dtype_convert in itertools.product(
        [torch.float32, torch.bfloat16], [True, False]
    ):
        if dtype == torch.float32 and with_dtype_convert:
            continue
        weight_prepack_patterns = _generate_qconv_weight_prepack_patterns(
            dtype, with_dtype_convert
        )
        for weight_prepack_pattern in weight_prepack_patterns:
            # Register to pass_number 1, so we can do dequant promotion in pass_number 0.
            _register_qconv_weight_prepack_pass(
                weight_prepack_pattern,
                pass_number=1,
                dtype=dtype,
                with_dtype_convert=with_dtype_convert,
            )


def _register_qlinear_weight_prepack():
    # 6 Linear related patterns will be matched based on the dtype, input dimension size and input contiguous.
    # Then convert the pattern into a QLinear node with int8_fp32/bf16.
````
- **EN**: Introduces function `_register_qconv_weight_prepack`, function `_register_qlinear_weight_prepack`. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_register_qconv_weight_prepack`、函数`_register_qlinear_weight_prepack`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。

### Lines 2341-2376 / 第 2341-2376 行
````python
    # Case 1: int8-mixed-fp32, input dim size is 2
    # Case 2: int8-mixed-fp32, input dim size exceeds 2 and contiguous
    # Case 3: int8-mixed-bf16, input dim size is 2
    # Case 4: int8-mixed-bf16, input dim size exceeds 2 and contiguous

    #   + - - - - | - - - - - - | - - - - - +
    #   |    dq_per_tensor  dq_per_channel  |
    #   |         |              |          |
    #   |    OPT(to_bf16)    OPT(to_bf16)   |
    #   |         |              |          |
    #   |     OPT(reshape)   permute        |
    #   |            \        /             |
    #   |             addmm/mm              |
    #   |                |                  |
    #   |           OPT(reshape)            |

    # Case 5: int8-mixed-fp32, input dim size exceeds 2 and not contiguous
    # Case 6: int8-mixed-bf16, input dim size exceeds 2 and not contiguous

    #   + - - - - | - - - - - - | - - - - - +
    #   |    dq_per_tensor  dq_per_channel  |
    #   |         |              |          |
    #   |    OPT(to_bf16)    OPT(to_bf16)   |
    #   |         |              |          |
    #   |       expand       permute        |
    #   |          \             |          |
    #   |                    expand         |
    #   |                    /              |
    #   |               bmm                 |
    #   |                |                  |
    #   |            OPT(add)               |

    linear_weight_prepack_cases = itertools.product(
        [torch.float32, torch.bfloat16], [True, False], [True, False], [True, False]
    )

````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `linear_weight_prepack_cases`. This range continues the implementation of function `_register_qlinear_weight_prepack`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `linear_weight_prepack_cases` 等值。这一段延续了函数`_register_qlinear_weight_prepack` 的具体实现。

### Lines 2377-2412 / 第 2377-2412 行
````python
    # Step 1: register patterns from mm and addmm
    for (
        dtype,
        input_dim_exceeds_two,
        is_tensor_overload,
        with_dtype_convert,
    ) in linear_weight_prepack_cases:
        if dtype == torch.float32 and with_dtype_convert:
            continue
        weight_prepack_patterns = _generate_qlinear_weight_prepack_patterns(
            dtype,
            input_dim_exceeds_two,
            is_tensor_overload=is_tensor_overload,
            with_dtype_convert=with_dtype_convert,
        )
        for weight_prepack_pattern in weight_prepack_patterns:
            # Register to pass_number 1, so we can do dequant promotion in pass_number 0.
            _register_qlinear_weight_prepack_pass(
                weight_prepack_pattern,
                pass_number=1,
                dtype=dtype,
                input_dim_exceeds_two=input_dim_exceeds_two,
                with_dtype_convert=with_dtype_convert,
            )

    # Step 2: register patterns from bmm
    # Linear might be decomposed into bmm when input dim exceeds 2 and not contiguous
    # refer to:
    # https://github.com/pytorch/pytorch/blob/80c07df659362a95da7cd4f3ec367abfdace38c4/torch/_decomp/decompositions.py#L3965-L3968
    # in this case, we can convert it back to qlinear
    for (
        dtype,
        with_bias,
        is_tensor_overload,
        with_dtype_convert,
    ) in itertools.product(
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。

### Lines 2413-2448 / 第 2413-2448 行
````python
        [torch.float32, torch.bfloat16], [True, False], [True, False], [True, False]
    ):
        if dtype == torch.float32 and with_dtype_convert:
            continue
        bmm_pattern = _generate_qlinear_weight_prepack_patterns(
            dtype=dtype,
            input_dim_exceeds_two=True,
            input_contiguous=False,
            with_bias=with_bias,
            is_tensor_overload=is_tensor_overload,
            with_dtype_convert=with_dtype_convert,
        )
        _register_qlinear_weight_prepack_pass(
            bmm_pattern,
            pass_number=1
            if with_bias
            else 2,  # if with_bias, there is an output add, so we should try to match it firstly
            dtype=dtype,
            input_dim_exceeds_two=True,
            input_contiguous=False,
            with_dtype_convert=with_dtype_convert,
        )


def _register_linear_dynamic_fp16_weight_prepack_pass(
    pattern,
    pass_number,
    input_dim_exceeds_two=False,
    input_contiguous=True,
    relu_fused=False,
):
    def _extra_check_fn(match: Match):
        return match.kwargs["dtype_fp16"] == torch.float16

    @register_freezing_graph_pattern(
        pattern,
````
- **EN**: Introduces function `_register_linear_dynamic_fp16_weight_prepack_pass`, function `_extra_check_fn`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`_register_linear_dynamic_fp16_weight_prepack_pass`、函数`_extra_check_fn`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 2449-2484 / 第 2449-2484 行
````python
        extra_check=_extra_check_fn,
        pass_number=pass_number,
    )
    def linear_dynamic_fp16_weight_prepack(match: Match, *args, **kwargs):
        """
        Match the pattern:
        fp32 activation
          |
        mm/addmm <- t <- to_fp32 <- to_fp16 <- weight
          |
        (reshape) <- (relu)

        OR

        fp32 activation
          |
        expand
          |
         bmm <- expand <- t <- to_fp32 <- to_fp16 <- weight
          |
        (add) <- (relu)

        Insert weight prepack node and change the pattern to:
        fp32 activation
          |
        onednn.linear_dynamic_fp16 <- onednn.linear_prepack_fp16 <- weight
        (or onednn.linear_relu_dynamic_fp16)
        """
        # find params
        x = kwargs["x"]
        w = kwargs["w"]
        bias = kwargs.get("b")

        # find linear node
        nodes_to_find = [aten.addmm.default, aten.mm.default, aten.bmm.default]
        linear_nodes = []
````
- **EN**: Introduces function `linear_dynamic_fp16_weight_prepack`. Initializes or updates values such as `extra_check`, `pass_number`, `x`, `w`, `bias`, `nodes_to_find`, and `...+1`.
- **CN**: 这里定义了函数`linear_dynamic_fp16_weight_prepack`。初始化或更新了 `extra_check`、`pass_number`、`x`、`w`、`bias`、`nodes_to_find`、`另有1项` 等值。

### Lines 2485-2520 / 第 2485-2520 行
````python
        for node in nodes_to_find:
            linear_nodes.extend(filter_nodes(match.nodes, node))
        assert len(linear_nodes) == 1
        linear_node = linear_nodes[0]
        assert isinstance(linear_node, torch.fx.node.Node)
        input_index = 1 if linear_node.target is aten.addmm.default else 0
        weight_index = input_index + 1

        # find relu node
        relu_node = None
        if relu_fused:
            relu_node = match.output_node()
            assert isinstance(relu_node, torch.fx.node.Node)

        # find reshape node, expand node and add node
        (
            act_reshape_node,
            output_reshape_node,
            expand_x_node,
            expand_w_node,
            add_bias_node,
        ) = (None, None, None, None, None)
        t_node = None
        if input_dim_exceeds_two:
            if input_contiguous:
                act_reshape_node = linear_node.args[input_index]
                t_node = linear_node.args[weight_index]
                output_reshape_node = next(iter(linear_node.users))
                assert output_reshape_node.target is aten.reshape.default
            else:
                expand_x_node = linear_node.args[input_index]
                expand_w_node = linear_node.args[weight_index]
                assert isinstance(expand_w_node, torch.fx.node.Node)
                t_node = expand_w_node.args[0]
                if bias:
                    add_bias_node = next(iter(linear_node.users))
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `linear_node`, `input_index`, `weight_index`, `relu_node`, `t_node`, `act_reshape_node`, and `...+5`. This range continues the implementation of function `_register_linear_dynamic_fp16_weight_prepack_pass.linear_dynamic_fp16_weight_prepack`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `linear_node`、`input_index`、`weight_index`、`relu_node`、`t_node`、`act_reshape_node`、`另有5项` 等值。这一段延续了函数`_register_linear_dynamic_fp16_weight_prepack_pass.linear_dynamic_fp16_weight_prepack` 的具体实现。

### Lines 2521-2556 / 第 2521-2556 行
````python
                    assert add_bias_node.target is aten.add.Tensor
        else:
            t_node = linear_node.args[weight_index]
        assert isinstance(t_node, torch.fx.node.Node)

        w_to_fp32_node = t_node.args[0]
        assert (
            isinstance(w_to_fp32_node, torch.fx.node.Node)
            and w_to_fp32_node.target
            is quantized_decomposed.convert_element_type.no_fuse
        )
        w_to_fp16_node = w_to_fp32_node.args[0]
        assert (
            isinstance(w_to_fp16_node, torch.fx.node.Node)
            and w_to_fp16_node.target
            is quantized_decomposed.convert_element_type.no_fuse
        )

        x_shape = x.meta.get("tensor_meta").shape
        if has_free_symbols(x_shape):
            # For dynamic shape case, we can't get activation shape ahead of runtime.
            x_shape = None
        graph = match.graph
        with graph.inserting_before(linear_node):
            # Insert weight prepack node and the qlinear node
            packed_weight_inputs = (
                w,
                x_shape,
            )
            packed_weight_op = torch.ops.onednn.linear_prepack_fp16
            prepack_weight_node = graph.call_function(
                packed_weight_op, args=packed_weight_inputs
            )

            # create new linear node and insert on graph
            new_args: tuple[Any, ...] = (
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `t_node`, `w_to_fp32_node`, `w_to_fp16_node`, `x_shape`, `graph`, and `...+4`. This range continues the implementation of function `_register_linear_dynamic_fp16_weight_prepack_pass.linear_dynamic_fp16_weight_prepack`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`t_node`、`w_to_fp32_node`、`w_to_fp16_node`、`x_shape`、`graph`、`另有4项` 等值。这一段延续了函数`_register_linear_dynamic_fp16_weight_prepack_pass.linear_dynamic_fp16_weight_prepack` 的具体实现。

### Lines 2557-2592 / 第 2557-2592 行
````python
                x,
                prepack_weight_node,
                bias,
            )
            linear_op = (
                torch.ops.onednn.linear_relu_dynamic_fp16.default
                if relu_fused
                else torch.ops.onednn.linear_dynamic_fp16.default
            )
            new_linear_node = graph.call_function(linear_op, args=new_args)
            out_node = match.output_node()
            out_node.replace_all_uses_with(new_linear_node)

            # Erase the original nodes in the reverse order
            new_linear_node.meta.update(out_node.meta)
            if relu_node is not None:
                graph.erase_node(relu_node)
            if output_reshape_node is not None:
                graph.erase_node(output_reshape_node)
            if add_bias_node is not None:
                graph.erase_node(add_bias_node)
            graph.erase_node(linear_node)
            if act_reshape_node is not None:
                assert isinstance(act_reshape_node, torch.fx.node.Node)
                graph.erase_node(act_reshape_node)
            if expand_x_node is not None:
                assert isinstance(expand_x_node, torch.fx.node.Node)
                graph.erase_node(expand_x_node)
            if expand_w_node is not None:
                assert isinstance(expand_w_node, torch.fx.node.Node)
                graph.erase_node(expand_w_node)
            graph.erase_node(t_node)
            graph.erase_node(w_to_fp32_node)
            graph.erase_node(w_to_fp16_node)

            counters["inductor"]["qlinear_weight_prepack_matcher_count"] += 1
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `linear_op`, `new_linear_node`, and `out_node`. This range continues the implementation of function `_register_linear_dynamic_fp16_weight_prepack_pass.linear_dynamic_fp16_weight_prepack`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `linear_op`、`new_linear_node`、`out_node` 等值。这一段延续了函数`_register_linear_dynamic_fp16_weight_prepack_pass.linear_dynamic_fp16_weight_prepack` 的具体实现。

### Lines 2593-2628 / 第 2593-2628 行
````python
            counters["inductor"]["qlinear_weight_prepack_matcher_nodes"] += len(
                match.nodes
            )


def _register_linear_dynamic_fp16_weight_prepack():
    to_dtype_op = torch.ops.quantized_decomposed.convert_element_type.no_fuse
    weight_pattern = CallFunction(
        to_dtype_op,
        CallFunction(
            to_dtype_op,
            KeywordArg("w"),
            KeywordArg("dtype_fp16"),
        ),
        KeywordArg("dtype_fp32"),
    )
    cases = itertools.product(
        [False, True],  # input_dim_exceeds_two
        [True, False],  # input_contiguous
        [False, True],  # relu fused
    )
    for input_dim_exceeds_two, input_contiguous, relu_fused in cases:
        patterns = _generate_linear_dynamic_fp16_pattern(
            weight_pattern,
            input_dim_exceeds_two,
            input_contiguous,
            relu_fused,
        )
        for pattern in patterns:
            _register_linear_dynamic_fp16_weight_prepack_pass(
                pattern,
                pass_number=0 if relu_fused else 1,
                input_dim_exceeds_two=input_dim_exceeds_two,
                input_contiguous=input_contiguous,
                relu_fused=relu_fused,
            )
````
- **EN**: Introduces function `_register_linear_dynamic_fp16_weight_prepack`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `to_dtype_op`, `weight_pattern`, `cases`, `patterns`, `pass_number`, `input_dim_exceeds_two`, and `...+2`.
- **CN**: 这里定义了函数`_register_linear_dynamic_fp16_weight_prepack`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。初始化或更新了 `to_dtype_op`、`weight_pattern`、`cases`、`patterns`、`pass_number`、`input_dim_exceeds_two`、`另有2项` 等值。

### Lines 2629-2664 / 第 2629-2664 行
````python


def _register_smooth_quant_int_mm_pattern():
    """
    The pattern is:
      (no bias) reshape -> _int_mm -> convert_element_type -> (expand ->) mul -> mul -> reshape
    or
      (with bias) pattern_no_bias -> add (-> reshape -> reshape)
    """

    # When torch.compile'ing with dynamic=True, the expand node and the two tailing reshape nodes exist
    # When torch.compile'ing with dynamic=False, they don't exist
    def get_pattern_no_bias(expand_a_scale: bool, reshape_a: bool = True):
        return CallFunction(
            aten.mul.Tensor,
            CallFunction(
                aten.mul.Tensor,
                CallFunction(
                    prims.convert_element_type.default,
                    CallFunction(
                        aten._int_mm.default,
                        CallFunction(
                            aten.reshape.default,
                            KeywordArg("a"),
                            KeywordArg("in_shape"),
                        )
                        if reshape_a
                        else KeywordArg("a"),
                        KeywordArg("b"),
                    ),
                    KeywordArg("dtype"),
                ),
                (
                    CallFunction(
                        aten.expand.default,
                        KeywordArg("x_scale"),
````
- **EN**: Introduces function `_register_smooth_quant_int_mm_pattern`, function `get_pattern_no_bias`. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_register_smooth_quant_int_mm_pattern`、函数`get_pattern_no_bias`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 2665-2700 / 第 2665-2700 行
````python
                        Arg(),
                    )
                    if expand_a_scale
                    else KeywordArg("x_scale")
                ),
            ),
            KeywordArg("w_scale"),
        )

    def _with_outer_reshape(pattern):
        return CallFunction(
            aten.reshape.default, pattern, KeywordArg("out_shape_no_bias")
        )

    # for torch.compile(dynamic=False)
    pattern_no_bias_1 = _with_outer_reshape(get_pattern_no_bias(expand_a_scale=False))
    pattern_with_bias_1 = CallFunction(
        aten.add.Tensor,
        pattern_no_bias_1,
        KeywordArg("bias"),
    )
    # for torch.compile(dynamic=True)
    pattern_no_bias_2 = _with_outer_reshape(get_pattern_no_bias(expand_a_scale=True))
    pattern_with_bias_2 = CallFunction(
        aten.reshape.default,
        CallFunction(
            aten.reshape.default,
            CallFunction(
                aten.add.Tensor,
                pattern_no_bias_2,
                KeywordArg("bias"),
            ),
            Arg(),
        ),
        KeywordArg("out_shape_with_bias"),
    )
````
- **EN**: Introduces function `_with_outer_reshape`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `pattern_no_bias_1`, `pattern_with_bias_1`, `pattern_no_bias_2`, and `pattern_with_bias_2`.
- **CN**: 这里定义了函数`_with_outer_reshape`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `pattern_no_bias_1`、`pattern_with_bias_1`、`pattern_no_bias_2`、`pattern_with_bias_2` 等值。

### Lines 2701-2736 / 第 2701-2736 行
````python

    # The following patterns are for torchao int8_dynamic_activation_int8_weight linear,
    # when both activation and weights are symmetrically quantized.
    # In practice, though, they may also match smooth-quant pattern when a 2D input shape would be used.
    # Since add is not currently being used as a oneDNN post-op, but is unfused, we don't need these patterns with bias.
    # Ideally, we should add mul + add post-op support in ATen int8 oneDNN linear op.
    pattern1_with_no_outer_or_act_reshape = get_pattern_no_bias(
        expand_a_scale=False, reshape_a=False
    )
    pattern2_with_no_outer_or_act_reshape = get_pattern_no_bias(
        expand_a_scale=True, reshape_a=False
    )

    def _validate_pattern(match: Match):
        if len(match.nodes) not in [4, 5, 6, 7, 10]:
            return False
        # Make sure weight is a constant
        aten_int_mm_node = filter_nodes(match.nodes, aten._int_mm.default)[0]
        if not isinstance(aten_int_mm_node.args[1], torch.fx.node.Node):
            return False
        if aten_int_mm_node.args[1].op != "get_attr":
            return False

        if len(match.nodes) == 10:
            # Check the two tailing reshape nodes can be fused
            if match.nodes[9].args[1] != match.nodes[6].args[1]:
                return False
        if len(match.nodes) == 10 or (
            len(match.nodes) == 7 and match.nodes[6].target is aten.add.Tensor
        ):
            bias_idx = 7 if len(match.nodes) == 10 else 6
            # Check bias shape
            bias_node = match.nodes[bias_idx].args[1]
            if not isinstance(bias_node, torch.fx.node.Node):
                return False
            if len(bias_node.meta.get("tensor_meta").shape) != 1:  # type: ignore[union-attr]
````
- **EN**: Introduces function `_validate_pattern`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_validate_pattern`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 2737-2772 / 第 2737-2772 行
````python
                return False
        return True

    pattern_to_pass_number = {
        pattern_no_bias_2: 0,
        pattern_with_bias_2: 0,
        pattern_no_bias_1: 1,
        pattern_with_bias_1: 1,
        pattern1_with_no_outer_or_act_reshape: 2,
        pattern2_with_no_outer_or_act_reshape: 2,
    }
    for pattern, pass_number in pattern_to_pass_number.items():

        @register_freezing_graph_pattern(
            pattern,
            extra_check=_validate_pattern,
            pass_number=pass_number,
        )
        def _int_mm_weight_prepack(match: Match, *args, **kwargs):
            bias = kwargs.get("bias")
            x = kwargs["a"]
            weight = kwargs["b"]
            dtype = kwargs["dtype"]
            x_scale = kwargs["x_scale"]
            w_scale = kwargs["w_scale"]
            x_shape = x.meta.get("tensor_meta").shape
            if has_free_symbols(x_shape):
                # For dynamic shape case, we can't get activation shape ahead of runtime.
                x_shape = None

            out_node = match.output_node()
            with match.graph.inserting_before(out_node):
                transpose_node = match.graph.call_function(
                    aten.permute.default, args=(weight, [1, 0])
                )
                contig_node = match.graph.call_function(
````
- **EN**: Introduces function `_int_mm_weight_prepack`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`_int_mm_weight_prepack`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 2773-2808 / 第 2773-2808 行
````python
                    aten.contiguous.default, args=(transpose_node,)
                )
                packed_weight_inputs = (
                    contig_node,
                    x_shape,
                )
                packed_weight_op = torch.ops.onednn.qlinear_prepack
                prepack_weight_node = match.graph.call_function(
                    packed_weight_op, args=packed_weight_inputs
                )

                dummy_zp = None
                w_scale = match.graph.call_function(
                    prims.convert_element_type.default, args=(w_scale, torch.float32)
                )

                x_scale_shape = x_scale.meta.get("tensor_meta").shape
                x_scale_is_scalar = False
                if not has_free_symbols(x_scale_shape):
                    prod = 1
                    for d in x_scale_shape:
                        prod *= d
                    x_scale_is_scalar = prod == 1

                new_args: tuple[Any, ...]
                if x_scale_is_scalar:
                    # in this case, we can call onednn.qlinear directly
                    new_args = (
                        x,
                        x_scale,
                        dummy_zp,  # x_zp
                        prepack_weight_node,
                        w_scale,
                        dummy_zp,  # w_zp
                        bias,
                        1.0,  # output_scale
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `packed_weight_inputs`, `packed_weight_op`, `prepack_weight_node`, `dummy_zp`, `w_scale`, `x_scale_shape`, and `...+3`. This range continues the implementation of function `_register_smooth_quant_int_mm_pattern._int_mm_weight_prepack`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `packed_weight_inputs`、`packed_weight_op`、`prepack_weight_node`、`dummy_zp`、`w_scale`、`x_scale_shape`、`另有3项` 等值。这一段延续了函数`_register_smooth_quant_int_mm_pattern._int_mm_weight_prepack` 的具体实现。

### Lines 2809-2844 / 第 2809-2844 行
````python
                        0,  # output_zero_point
                        dtype,  # output_dtype
                        "none",  # post op name
                        [],  # post op args
                        "",  # post op algorithm
                    )
                    new_linear_node = match.graph.call_function(
                        torch.ops.onednn.qlinear_pointwise.tensor, args=new_args
                    )
                    out_node.replace_all_uses_with(new_linear_node)
                    new_linear_node.meta.update(out_node.meta)
                else:
                    # onednn.qlinear does not support per-channel quantization of x
                    # so in this case, we have to apply x scale and add bias ourselves after qlinear
                    in_shape = kwargs.get("in_shape")
                    if in_shape is None:
                        x_reshaped = x
                    else:
                        x_reshaped = match.graph.call_function(
                            aten.reshape.default, args=(x, in_shape)
                        )
                    new_args = (
                        x_reshaped,
                        1.0,  # x_scale
                        0,  # x_zp
                        prepack_weight_node,
                        w_scale,
                        dummy_zp,  # w_zp
                        None,  # bias
                        1.0,  # output_scale
                        0,  # output_zero_point
                        dtype,  # output_dtype
                        "none",  # post op name
                        [],  # post op args
                        "",  # post op algorithm
                    )
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `new_linear_node`, `else`, `in_shape`, `x_reshaped`, and `new_args`. This range continues the implementation of function `_register_smooth_quant_int_mm_pattern._int_mm_weight_prepack`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `new_linear_node`、`else`、`in_shape`、`x_reshaped`、`new_args` 等值。这一段延续了函数`_register_smooth_quant_int_mm_pattern._int_mm_weight_prepack` 的具体实现。

### Lines 2845-2880 / 第 2845-2880 行
````python
                    new_linear_node = match.graph.call_function(
                        torch.ops.onednn.qlinear_pointwise, args=new_args
                    )
                    # apply x scale
                    new_out_node = match.graph.call_function(
                        aten.mul.Tensor, args=(new_linear_node, x_scale)
                    )

                    # Add bias and reshape
                    has_outer_reshape = (
                        kwargs.get("out_shape_with_bias") is not None
                        or kwargs.get("out_shape_no_bias") is not None
                    )

                    if has_outer_reshape:
                        out_shape = kwargs.get(
                            "out_shape_with_bias", kwargs["out_shape_no_bias"]
                        )
                    if bias is not None:
                        new_out_node = match.graph.call_function(
                            aten.add.Tensor, args=(new_out_node, bias)
                        )
                        if has_outer_reshape:
                            new_out_node = match.graph.call_function(
                                aten.reshape.default,
                                args=(new_out_node, out_shape),  # type: ignore[possibly-undefined]
                            )
                    else:
                        if has_outer_reshape:
                            new_out_node = match.graph.call_function(
                                aten.reshape.default,
                                args=(new_out_node, out_shape),  # type: ignore[possibly-undefined]
                            )
                    out_node.replace_all_uses_with(new_out_node)
                    new_out_node.meta.update(out_node.meta)
                for node in reversed(match.nodes):
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `new_linear_node`, `new_out_node`, `has_outer_reshape`, `out_shape`, `args`, and `else`. This range continues the implementation of function `_register_smooth_quant_int_mm_pattern._int_mm_weight_prepack`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `new_linear_node`、`new_out_node`、`has_outer_reshape`、`out_shape`、`args`、`else` 等值。这一段延续了函数`_register_smooth_quant_int_mm_pattern._int_mm_weight_prepack` 的具体实现。

### Lines 2881-2916 / 第 2881-2916 行
````python
                    match.graph.erase_node(node)
                counters["inductor"]["qlinear_weight_prepack_matcher_count"] += 1
                counters["inductor"]["qlinear_weight_prepack_matcher_nodes"] += len(
                    match.nodes
                )


class PostOpAttr:
    def __init__(
        self,
        binary_op_name: str = "none",
        alpha=None,
        unary_op_name: str = "none",
        scalars_attr=None,
        algorithm_attr=None,
    ) -> None:
        self.binary_op_name = binary_op_name
        self.alpha = alpha if alpha else 1.0
        self.unary_op_name = unary_op_name
        self.scalars_attr = scalars_attr if scalars_attr else []
        self.algorithm_attr = algorithm_attr if algorithm_attr else ""


def _register_qconv_post_op_fusion_pass(
    pattern,
    pass_number,
    computation_op,
    post_op_attr,
):
    has_binary_post_op = post_op_attr.binary_op_name != "none"

    @register_freezing_graph_pattern(
        pattern,
        extra_check=_is_valid_qconv_post_op_fusion_pattern(has_binary_post_op),
        pass_number=pass_number,
    )
````
- **EN**: Introduces class `PostOpAttr`, function `__init__`, function `_register_qconv_post_op_fusion_pass`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `binary_op_name`, `alpha`, `unary_op_name`, `scalars_attr`, `algorithm_attr`, `has_binary_post_op`, and `...+2`.
- **CN**: 这里定义了类`PostOpAttr`、函数`__init__`、函数`_register_qconv_post_op_fusion_pass`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `binary_op_name`、`alpha`、`unary_op_name`、`scalars_attr`、`algorithm_attr`、`has_binary_post_op`、`另有2项` 等值。

### Lines 2917-2952 / 第 2917-2952 行
````python
    def qconv(match: Match, *args, **kwargs):
        # Activation QParams
        x, x_scale, x_zp = (
            kwargs["x"],
            kwargs["x_scale"],
            kwargs["x_zp"],
        )
        # Weight QParams
        packed_weight, w_scale, w_zp = (
            kwargs["packed_weight"],
            kwargs["w_scale"],
            kwargs["w_zp"],
        )
        # Conv Params
        b, stride, padding, dilation, groups = (
            kwargs["b"],
            kwargs["stride"],
            kwargs["padding"],
            kwargs["dilation"],
            kwargs["groups"],
        )
        output_dtype = _get_pattern_output_dtype(match)
        assert output_dtype in [torch.int8, torch.uint8, torch.float32, torch.bfloat16]
        # Output QParams
        o_inv_scale = (
            kwargs["o_inv_scale"]
            if (output_dtype == torch.uint8 or output_dtype == torch.int8)
            else 1.0
        )
        o_zero_point = (
            kwargs["o_zp"]
            if (output_dtype == torch.uint8 or output_dtype == torch.int8)
            else 0
        )
        assert (
            kwargs["postop_name"] == "none"
````
- **EN**: Introduces function `qconv`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`qconv`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 2953-2988 / 第 2953-2988 行
````python
        )  # Expected no post op fused in weight prepack phase
        if post_op_attr.unary_op_name == "hardtanh":
            min_value = kwargs.get("min_value")
            max_value = kwargs.get("max_value")
            post_op_attr.scalars_attr = [min_value, max_value]

        out_node = match.output_node()
        with match.graph.inserting_before(out_node):
            if not has_binary_post_op:
                computation_args: tuple[Any, ...] = (
                    x,
                    x_scale,
                    x_zp,
                    packed_weight,
                    w_scale,
                    w_zp,
                    b,
                    stride,
                    padding,
                    dilation,
                    groups,
                    o_inv_scale,
                    o_zero_point,
                    output_dtype,
                    post_op_attr.unary_op_name,
                    post_op_attr.scalars_attr,
                    post_op_attr.algorithm_attr,
                )
            else:
                accum = (
                    kwargs["accum"]
                    if output_dtype in [torch.uint8, torch.int8]
                    else kwargs["accum_after_dequant"]
                )
                accum_scale = (
                    kwargs["accum_scale"]
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `min_value`, `max_value`, `out_node`, `computation_args`, `else`, `accum`, and `...+1`. This range continues the implementation of function `_register_qconv_post_op_fusion_pass.qconv`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `min_value`、`max_value`、`out_node`、`computation_args`、`else`、`accum`、`另有1项` 等值。这一段延续了函数`_register_qconv_post_op_fusion_pass.qconv` 的具体实现。

### Lines 2989-3024 / 第 2989-3024 行
````python
                    if output_dtype in [torch.uint8, torch.int8]
                    else 1.0
                )
                accum_zp = (
                    kwargs["accum_zp"]
                    if output_dtype in [torch.uint8, torch.int8]
                    else 0
                )
                computation_args = (
                    x,
                    x_scale,
                    x_zp,
                    packed_weight,
                    w_scale,
                    w_zp,
                    accum,
                    b,
                    stride,
                    padding,
                    dilation,
                    groups,
                    o_inv_scale,
                    o_zero_point,
                    output_dtype,
                    accum_scale,
                    accum_zp,
                    post_op_attr.binary_op_name,
                    post_op_attr.alpha,
                    post_op_attr.unary_op_name,
                    post_op_attr.scalars_attr,
                    post_op_attr.algorithm_attr,
                )
            new_conv_node = match.graph.call_function(
                computation_op, args=computation_args
            )
            out_node.replace_all_uses_with(new_conv_node)
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `accum_zp`, `computation_args`, and `new_conv_node`. This range continues the implementation of function `_register_qconv_post_op_fusion_pass.qconv`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `accum_zp`、`computation_args`、`new_conv_node` 等值。这一段延续了函数`_register_qconv_post_op_fusion_pass.qconv` 的具体实现。

### Lines 3025-3060 / 第 3025-3060 行
````python
            new_conv_node.meta.update(out_node.meta)
            for node in reversed(match.nodes):
                match.graph.erase_node(node)
        count_key = (
            "qconv2d_binary_matcher_count"
            if has_binary_post_op
            else "qconv_unary_matcher_count"
        )
        nodes_key = (
            "qconv2d_binary_matcher_nodes"
            if has_binary_post_op
            else "qconv_unary_matcher_nodes"
        )
        counters["inductor"][count_key] += 1
        counters["inductor"][nodes_key] += len(match.nodes)

    return qconv


def _register_qconv_unary_fusion():
    from .mkldnn_fusion import _hardswish_fusion, _hardtanh_fusion, _silu_fusion

    for original_pattern_output_dtype in [torch.float32, torch.bfloat16]:
        # Priority 1 to match: QConv2d Unary pattern with int8 output
        # If a pattern1 is a sub-set of pattern2, we should try to match pattern2 firstly.
        # For example: pattern1 is qconv_fp32 -> relu, pattern2 is qconv_fp32 -> relu -> quant
        is_bf16 = original_pattern_output_dtype == torch.bfloat16
        conv_unary_replace_patterns = {
            PostOpAttr(
                "none", None, "none", [], ""
            ): generate_pattern_with_output_quant(
                get_qconv_pt2e_pattern(users=1),
            ),
            PostOpAttr(
                "none", None, "relu", [], ""
            ): generate_pattern_with_output_quant(
````
- **EN**: Imports dependencies such as `.mkldnn_fusion` for the logic in this range. Introduces function `_register_qconv_unary_fusion`. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里导入了 `.mkldnn_fusion` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_register_qconv_unary_fusion`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 3061-3096 / 第 3061-3096 行
````python
                generate_pattern_with_unary(
                    get_qconv_pt2e_pattern(users=1), aten.relu.default
                ),
            ),
            PostOpAttr(
                "none", None, "hardtanh", [], ""
            ): generate_pattern_with_output_quant(
                _unary_fusion_pattern(
                    _hardtanh_fusion,
                    get_qconv_pt2e_pattern(users=1),
                    1,
                    is_bf16,
                ),
                with_dtype_convert=is_bf16,
            ),
            PostOpAttr(
                "none", None, "hardswish", [], ""
            ): generate_pattern_with_output_quant(
                _unary_fusion_pattern(
                    _hardswish_fusion,
                    get_qconv_pt2e_pattern(users=1 if is_bf16 else 2),
                    2,
                    is_bf16,
                ),
                with_dtype_convert=is_bf16,
            ),
            PostOpAttr(
                "none", None, "swish", [], ""
            ): generate_pattern_with_output_quant(
                _unary_fusion_pattern(
                    _silu_fusion,
                    get_qconv_pt2e_pattern(users=1 if is_bf16 else 2),
                    2,
                    is_bf16,
                ),
                with_dtype_convert=is_bf16,
````
- **EN**: Initializes or updates values such as `with_dtype_convert`. This range continues the implementation of function `_register_qconv_unary_fusion`.
- **CN**: 初始化或更新了 `with_dtype_convert` 等值。这一段延续了函数`_register_qconv_unary_fusion` 的具体实现。

### Lines 3097-3132 / 第 3097-3132 行
````python
            ),
        }

        for unary_attr, patterns in conv_unary_replace_patterns.items():
            # Register qconv2d pattern for ExternKernel Lowering
            _register_qconv_post_op_fusion_pass(
                patterns,
                3,  # pass_number
                torch.ops.onednn.qconv_pointwise.default,  # computation_op
                unary_attr,  # unary_attr
            )

        # Priority 2 to match: QConv2d Unary pattern with fp32/bfloat16 output
        conv_unary_replace_float_out_patterns = {
            PostOpAttr("none", None, "relu", [], ""): generate_pattern_with_unary(
                get_qconv_pt2e_pattern(users=1), aten.relu.default
            ),
            PostOpAttr(
                "none", None, "hardtanh", [], ""
            ): _may_generate_pattern_with_dtype_convert(
                _unary_fusion_pattern(
                    _hardtanh_fusion,
                    get_qconv_pt2e_pattern(users=1),
                    1,
                    is_bf16,
                ),
                Arg(),
                is_bf16,
            ),
            PostOpAttr(
                "none", None, "hardswish", [], ""
            ): _may_generate_pattern_with_dtype_convert(
                _unary_fusion_pattern(
                    _hardswish_fusion,
                    get_qconv_pt2e_pattern(users=1 if is_bf16 else 2),
                    2,
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `conv_unary_replace_float_out_patterns`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。初始化或更新了 `conv_unary_replace_float_out_patterns` 等值。

### Lines 3133-3168 / 第 3133-3168 行
````python
                    is_bf16,
                ),
                Arg(),
                is_bf16,
            ),
            PostOpAttr(
                "none", None, "swish", [], ""
            ): _may_generate_pattern_with_dtype_convert(
                _unary_fusion_pattern(
                    _silu_fusion,
                    get_qconv_pt2e_pattern(users=1 if is_bf16 else 2),
                    2,
                    is_bf16,
                ),
                Arg(),
                is_bf16,
            ),
        }

        for unary_attr, patterns in conv_unary_replace_float_out_patterns.items():
            # Register qconv2d pattern for ExternKernel Lowering
            _register_qconv_post_op_fusion_pass(
                patterns,
                4,  # pass_number
                torch.ops.onednn.qconv_pointwise.default,  # computation_op
                unary_attr,  # unary_attr
            )


def _register_qconv_binary_fusion():
    for int8_mixed_bf16_with_inplace_add in [False, True]:
        # Priority 1 to match: QConv2d Binary or Binary-Unary pattern with int8 output
        swap_binary_inputs_list = [False, True]
        binary_replace_patterns = {}
        for swap_inputs in swap_binary_inputs_list:
            binary_replace_patterns.update(
````
- **EN**: Introduces function `_register_qconv_binary_fusion`. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_register_qconv_binary_fusion`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。

### Lines 3169-3204 / 第 3169-3204 行
````python
                {
                    PostOpAttr(
                        "sum", 1.0, "none", [], ""
                    ): generate_pattern_with_output_quant(
                        generate_pattern_with_binary(
                            aten.add.Tensor,
                            get_qconv_pt2e_pattern(users=1),
                            dequantize_accum_pattern,
                            int8_mixed_bf16_with_inplace_add,
                            swap_inputs=swap_inputs,
                        ),
                    ),
                    PostOpAttr(
                        "sum", 1.0, "relu", [], ""
                    ): generate_pattern_with_output_quant(
                        generate_pattern_with_unary(
                            generate_pattern_with_binary(
                                aten.add.Tensor,
                                get_qconv_pt2e_pattern(users=1),
                                dequantize_accum_pattern,
                                int8_mixed_bf16_with_inplace_add,
                                swap_inputs=swap_inputs,
                            ),
                            aten.relu.default,
                        ),
                    ),
                }
            )

        for binary_unary_attr, patterns in binary_replace_patterns.items():
            _register_qconv_post_op_fusion_pass(
                patterns,
                3,  # pass_number
                torch.ops.onednn.qconv2d_pointwise.binary,  # computation_op
                binary_unary_attr,  # binary_unary_attr
            )
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `swap_inputs`. This range continues the implementation of function `_register_qconv_binary_fusion`.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。初始化或更新了 `swap_inputs` 等值。这一段延续了函数`_register_qconv_binary_fusion` 的具体实现。

### Lines 3205-3240 / 第 3205-3240 行
````python

        # Priority 2 to match: QConv2d Binary-Unary pattern with fp32/bfloat16 output
        binary_replace_float_out_patterns = {}
        for swap_inputs in swap_binary_inputs_list:
            binary_replace_float_out_patterns.update(
                {
                    PostOpAttr("sum", 1.0, "relu", [], ""): generate_pattern_with_unary(
                        generate_pattern_with_binary(
                            aten.add.Tensor,
                            get_qconv_pt2e_pattern(users=1),
                            KeywordArg("accum_after_dequant"),
                            int8_mixed_bf16_with_inplace_add,
                            swap_inputs=swap_inputs,
                        ),
                        aten.relu.default,
                    )
                }
            )

        for (
            binary_unary_attr,
            patterns,
        ) in binary_replace_float_out_patterns.items():
            if int8_mixed_bf16_with_inplace_add:
                _register_qconv_post_op_fusion_pass(
                    patterns,
                    3,  # pass_number
                    torch.ops.onednn.qconv2d_pointwise.binary,  # computation_op
                    binary_unary_attr,  # binary_unary_attr
                )
            else:
                _register_qconv_post_op_fusion_pass(
                    patterns,
                    4,  # pass_number
                    torch.ops.onednn.qconv2d_pointwise.binary,  # computation_op
                    binary_unary_attr,  # binary_unary_attr
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `binary_replace_float_out_patterns`, `swap_inputs`, and `else`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。初始化或更新了 `binary_replace_float_out_patterns`、`swap_inputs`、`else` 等值。

### Lines 3241-3276 / 第 3241-3276 行
````python
                )

        # Priority 3: QConv2d Binary pattern with fp32/bfloat16 output
        binary_replace_float_out_patterns = {}
        for swap_inputs in swap_binary_inputs_list:
            binary_replace_float_out_patterns.update(
                {
                    PostOpAttr(
                        "sum", 1.0, "none", [], ""
                    ): generate_pattern_with_binary(
                        aten.add.Tensor,
                        get_qconv_pt2e_pattern(users=1),
                        KeywordArg("accum_after_dequant"),
                        int8_mixed_bf16_with_inplace_add,
                        swap_inputs=swap_inputs,
                    ),
                }
            )

        for (
            binary_unary_attr,
            patterns,
        ) in binary_replace_float_out_patterns.items():
            _register_qconv_post_op_fusion_pass(
                patterns,
                4 if int8_mixed_bf16_with_inplace_add else 5,  # pass_number
                torch.ops.onednn.qconv2d_pointwise.binary,  # computation_op
                binary_unary_attr,  # binary_unary_attr
            )


def _register_qlinear_post_op_fusion_pass(
    pattern,
    pass_number,
    computation_op,
    post_op_attr,
````
- **EN**: Introduces function `_register_qlinear_post_op_fusion_pass`. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_register_qlinear_post_op_fusion_pass`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。

### Lines 3277-3312 / 第 3277-3312 行
````python
):
    has_binary_post_op = post_op_attr.binary_op_name != "none"

    @register_freezing_graph_pattern(
        pattern,
        extra_check=_is_valid_qlinear_post_op_fusion_pattern(has_binary_post_op),
        pass_number=pass_number,
    )
    def qlinear_post_op_fusion(match: Match, *args, **kwargs):
        """
        Match the pattern:
        qlinear - post op
        """
        output_dtype = _get_pattern_output_dtype(match)
        # Activation QParams
        x, x_scale, x_zp = (
            kwargs["x"],
            kwargs["x_scale"],
            kwargs["x_zp"],
        )
        # Weight QParams
        packed_weight, w_scale, w_zp = (
            kwargs["packed_weight"],
            kwargs["w_scale"],
            kwargs["w_zp"],
        )

        # bias
        b = kwargs.get("b")

        # Output QParams
        o_inv_scale = (
            kwargs["o_inv_scale"]
            if (output_dtype in [torch.uint8, torch.int8])
            else 1.0
        )
````
- **EN**: Introduces function `qlinear_post_op_fusion`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`qlinear_post_op_fusion`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 3313-3348 / 第 3313-3348 行
````python
        o_zero_point = (
            kwargs["o_zp"] if (output_dtype in [torch.uint8, torch.int8]) else 0
        )
        assert (
            kwargs["postop_name"] == "none"
        )  # Expected no post op fused in weight prepack phase

        out_node = match.output_node()
        with match.graph.inserting_before(out_node):
            if not has_binary_post_op:
                computation_args: tuple[Any, ...] = (
                    x,
                    x_scale,
                    x_zp,
                    packed_weight,
                    w_scale,
                    w_zp,
                    b,
                    o_inv_scale,
                    o_zero_point,
                    output_dtype,
                    post_op_attr.unary_op_name,
                    post_op_attr.scalars_attr,
                    post_op_attr.algorithm_attr,
                )
            else:
                other = kwargs["other"] if "other" in kwargs else kwargs["accum"]
                x2_scale = 1.0
                x2_zp = 0
                computation_args = (
                    x,
                    x_scale,
                    x_zp,
                    packed_weight,
                    w_scale,
                    w_zp,
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `o_zero_point`, `out_node`, `computation_args`, `else`, `other`, `x2_scale`, and `...+1`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `o_zero_point`、`out_node`、`computation_args`、`else`、`other`、`x2_scale`、`另有1项` 等值。

### Lines 3349-3384 / 第 3349-3384 行
````python
                    other,
                    b,
                    o_inv_scale,
                    o_zero_point,
                    output_dtype,
                    x2_scale,
                    x2_zp,
                    post_op_attr.binary_op_name,
                    post_op_attr.alpha,
                    post_op_attr.unary_op_name,
                    post_op_attr.scalars_attr,
                    post_op_attr.algorithm_attr,
                )
            new_linear_node = match.graph.call_function(
                computation_op, args=computation_args
            )
            out_node.replace_all_uses_with(new_linear_node)
            new_linear_node.meta.update(out_node.meta)
            for node in reversed(match.nodes):
                match.graph.erase_node(node)
        count_key = (
            "qlinear_binary_matcher_count"
            if has_binary_post_op
            else "qlinear_unary_matcher_count"
        )
        nodes_key = (
            "qlinear_binary_matcher_nodes"
            if has_binary_post_op
            else "qlinear_unary_matcher_nodes"
        )
        counters["inductor"][count_key] += 1
        counters["inductor"][nodes_key] += len(match.nodes)


def _register_qlinear_unary_fusion():
    from .mkldnn_fusion import (
````
- **EN**: Imports dependencies such as `.mkldnn_fusion` for the logic in this range. Introduces function `_register_qlinear_unary_fusion`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `.mkldnn_fusion` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_register_qlinear_unary_fusion`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。

### Lines 3385-3420 / 第 3385-3420 行
````python
        _gelu_fusion_1 as _gelu_fusion_erf,
        _gelu_fusion_2 as _gelu_fusion_tanh,
    )

    for original_pattern_output_dtype in [torch.float32, torch.bfloat16]:
        is_bf16 = original_pattern_output_dtype == torch.bfloat16
        for x_scale_zp_are_tensors in (False, True):
            qlinear_pattern = get_qlinear_pt2e_pattern(x_scale_zp_are_tensors)
            computation_op = (
                torch.ops.onednn.qlinear_pointwise.tensor
                if x_scale_zp_are_tensors
                else torch.ops.onednn.qlinear_pointwise.default
            )
            # Priority 1 to match: QLinear Unary pattern with int8 output
            linear_unary_replace_patterns = {
                PostOpAttr(
                    "none", None, "none", [], ""
                ): generate_pattern_with_output_quant(
                    qlinear_pattern,
                ),
                PostOpAttr(
                    "none", None, "relu", [], ""
                ): generate_pattern_with_output_quant(
                    generate_pattern_with_unary(qlinear_pattern, aten.relu.default),
                ),
                PostOpAttr(
                    "none", None, "gelu", [], "none"
                ): generate_pattern_with_output_quant(
                    _unary_fusion_pattern(
                        _gelu_fusion_erf,
                        get_qlinear_pt2e_pattern(
                            x_scale_zp_are_tensors, 1 if is_bf16 else 2
                        ),
                        2,
                        is_bf16,
                    ),
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `is_bf16`, `qlinear_pattern`, `computation_op`, and `linear_unary_replace_patterns`. This range continues the implementation of function `_register_qlinear_unary_fusion`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `is_bf16`、`qlinear_pattern`、`computation_op`、`linear_unary_replace_patterns` 等值。这一段延续了函数`_register_qlinear_unary_fusion` 的具体实现。

### Lines 3421-3456 / 第 3421-3456 行
````python
                    with_dtype_convert=is_bf16,
                ),
                PostOpAttr(
                    "none", None, "gelu", [], "tanh"
                ): generate_pattern_with_output_quant(
                    _unary_fusion_pattern(
                        _gelu_fusion_tanh,
                        get_qlinear_pt2e_pattern(
                            x_scale_zp_are_tensors, 1 if is_bf16 else 4
                        ),
                        4,
                        is_bf16,
                    ),
                    with_dtype_convert=is_bf16,
                ),
            }

            for unary_attr, patterns in linear_unary_replace_patterns.items():
                _register_qlinear_post_op_fusion_pass(
                    patterns,
                    3,  # pass_number
                    computation_op,
                    unary_attr,  # unary_attr
                )

            # Priority 2 to match: QLinear Unary pattern with FP32/BF16 output
            linear_unary_replace_float_out_patterns = {
                PostOpAttr("none", None, "relu", [], ""): generate_pattern_with_unary(
                    qlinear_pattern, aten.relu.default
                ),
                PostOpAttr(
                    "none", None, "gelu", [], "none"
                ): _may_generate_pattern_with_dtype_convert(
                    _unary_fusion_pattern(
                        _gelu_fusion_erf,
                        get_qlinear_pt2e_pattern(
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `with_dtype_convert`, and `linear_unary_replace_float_out_patterns`. This range continues the implementation of function `_register_qlinear_unary_fusion`.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。初始化或更新了 `with_dtype_convert`、`linear_unary_replace_float_out_patterns` 等值。这一段延续了函数`_register_qlinear_unary_fusion` 的具体实现。

### Lines 3457-3492 / 第 3457-3492 行
````python
                            x_scale_zp_are_tensors, 1 if is_bf16 else 2
                        ),
                        2,
                        is_bf16,
                    ),
                    Arg(),
                    is_bf16,
                ),
                PostOpAttr(
                    "none", None, "gelu", [], "tanh"
                ): _may_generate_pattern_with_dtype_convert(
                    _unary_fusion_pattern(
                        _gelu_fusion_tanh,
                        get_qlinear_pt2e_pattern(
                            x_scale_zp_are_tensors, 1 if is_bf16 else 4
                        ),
                        4,
                        is_bf16,
                    ),
                    Arg(),
                    is_bf16,
                ),
            }

            for unary_attr, patterns in linear_unary_replace_float_out_patterns.items():
                _register_qlinear_post_op_fusion_pass(
                    patterns,
                    4,  # pass_number
                    computation_op,
                    unary_attr,  # unary_attr
                )


def _register_qlinear_binary_fusion():
    r"""
    Supported linear-binary(-unary) patterns
````
- **EN**: Introduces function `_register_qlinear_binary_fusion`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_register_qlinear_binary_fusion`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。

### Lines 3493-3528 / 第 3493-3528 行
````python

        linear(X)   extra input
               \   /
                Add
                 |
            Optional(relu)
                 |
                 Y

    1. int8-mixed-fp32
    +---+---------------+-----------+------------------------------+---------+
    | # | Add type      | Quant out | Pattern                      | Post op |
    +---+---------------+-----------+------------------------------+---------+
    | 1 | In-/out-place | Yes       | linear + fp32 -> (relu) -> q | add     |
    +---+---------------+-----------+------------------------------+---------+
    | 2 | In-/out-place | No        | linear + fp32 -> (relu)      | sum     |
    +---+---------------+-----------+------------------------------+---------+

    2. int8-mixed-bf16
    +---+----------+---------------+-----------+-----------------------------------------+---------+
    | # | X2 dtype | Add type      | Quant out | Pattern                                 | Post op |
    +---+----------+---------------+-----------+-----------------------------------------+---------+
    | 1 | BF16     | In-/out-place | Yes       | linear + bf16 -> (relu) -> q            | add     |
    +---+----------+---------------+-----------+-----------------------------------------+---------+
    | 2 | BF16     | In-/out-place | No        | linear + bf16 -> (relu)                 | sum     |
    +---+----------+---------------+-----------+-----------------------------------------+---------+
    | 3 | FP32     | Out-place     | Yes       | linear + fp32 -> (relu) -> q            | add     |
    |   |          | In-place right|           |                                         |         |
    +---+----------+---------------+-----------+-----------------------------------------+---------+
    | 4 | FP32     | Out-place     | No        | linear + fp32 -> (relu)                 | sum     |
    |   |          | In-place right|           |                                         |         |
    +---+----------+---------------+-----------+-----------------------------------------+---------+
    | 5 | FP32     | In-place left | Yes       | linear + fp32 -> to_bf16 -> (relu) -> q | add     |
    +---+----------+---------------+-----------+-----------------------------------------+---------+
    | 6 | FP32     | In-place left | No        | linear + fp32 -> to_bf16 -> (relu)      | add     |
    +---+----------+---------------+-----------+-----------------------------------------+---------+
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. This range continues the implementation of function `_register_qlinear_binary_fusion`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。这一段延续了函数`_register_qlinear_binary_fusion` 的具体实现。

### Lines 3529-3564 / 第 3529-3564 行
````python

    Note
    (1) The positions of linear and the extra input can be swapped.
    (2) we don't insert q-dq before the extra input of linear-add by recipe. But if q-dq is found at the
    extra input, we don't match that pattern because we cannot match all these patterns in 3 passes.
    """
    for x_scale_zp_are_tensors in (False, True):
        qlinear_binary_op = (
            torch.ops.onednn.qlinear_pointwise.binary_tensor
            if x_scale_zp_are_tensors
            else torch.ops.onednn.qlinear_pointwise.binary
        )
        unary_postop_list = ["none", "relu"]
        unary_postop_dict = {
            "none": None,
            "relu": aten.relu.default,
        }
        convert_dtype_after_binary_list = [False, True]

        # Priority 1 to match: QLinear Binary or Binary-Unary pattern with int8 output
        # Covers case (1) of int8-mixed-fp32 and case (1)(3)(5) of int8-mixed-bf16,
        # totally 3 patterns (2 are identical)
        swap_binary_inputs_list = [False, True]
        int8_mixed_bf16_list = [False, True]
        combinations = itertools.product(
            unary_postop_list,
            int8_mixed_bf16_list,
            swap_binary_inputs_list,
            convert_dtype_after_binary_list,
        )
        qlinear_binary_replace_patterns = {}
        for unary_op, int8_mixed_bf16, swap_inputs, cvt_dtype_binary in combinations:
            if not int8_mixed_bf16 and cvt_dtype_binary:
                # No convert node after binary node if dtypes are all fp32
                continue
            qlinear_binary_replace_patterns.update(
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `qlinear_binary_op`, `unary_postop_list`, `unary_postop_dict`, `convert_dtype_after_binary_list`, `swap_binary_inputs_list`, `int8_mixed_bf16_list`, and `...+2`. This range continues the implementation of function `_register_qlinear_binary_fusion`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `qlinear_binary_op`、`unary_postop_list`、`unary_postop_dict`、`convert_dtype_after_binary_list`、`swap_binary_inputs_list`、`int8_mixed_bf16_list`、`另有2项` 等值。这一段延续了函数`_register_qlinear_binary_fusion` 的具体实现。

### Lines 3565-3600 / 第 3565-3600 行
````python
                {
                    PostOpAttr(
                        "add", 1.0, unary_op, [], ""
                    ): generate_pattern_with_output_quant(
                        generate_pattern_with_unary(
                            generate_pattern_with_binary(
                                aten.add.Tensor,
                                get_qlinear_pt2e_pattern(x_scale_zp_are_tensors),
                                KeywordArg("other"),
                                # If fp32 extra input is inplace added to bf16 linear output,
                                # a to_bf16 node is inserted after binary
                                dtype_convert=cvt_dtype_binary,
                                swap_inputs=swap_inputs,
                            ),
                            unary_postop_dict[unary_op],
                        ),
                    )
                }
            )
        for binary_unary_attr, patterns in qlinear_binary_replace_patterns.items():
            _register_qlinear_post_op_fusion_pass(
                patterns,
                3,  # pass_number
                qlinear_binary_op,  # computation_op
                binary_unary_attr,
            )

        # Priority 2.1 to match: QLinear Binary-Unary pattern with fp32/bfloat16 output
        # Covers case (2) of int8-mixed-fp32 and case (2)(4) of int8-mixed-bf16,
        # totally 2 patterns (2 are identical)
        binary_replace_float_out_patterns = {}
        for swap_binary_inputs in swap_binary_inputs_list:
            binary_replace_float_out_patterns.update(
                {
                    PostOpAttr("sum", 1.0, "relu", [], ""): generate_pattern_with_unary(
                        generate_pattern_with_binary(
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `dtype_convert`, `swap_inputs`, and `binary_replace_float_out_patterns`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。初始化或更新了 `dtype_convert`、`swap_inputs`、`binary_replace_float_out_patterns` 等值。

### Lines 3601-3636 / 第 3601-3636 行
````python
                            aten.add.Tensor,
                            get_qlinear_pt2e_pattern(x_scale_zp_are_tensors),
                            KeywordArg("accum"),
                            dtype_convert=False,
                            swap_inputs=swap_binary_inputs,
                        ),
                        aten.relu.default,
                    ),
                }
            )
        for (
            binary_unary_attr,
            patterns,
        ) in binary_replace_float_out_patterns.items():
            _register_qlinear_post_op_fusion_pass(
                patterns,
                4,  # pass_number
                qlinear_binary_op,  # computation_op
                binary_unary_attr,
            )
        # Priority 2.2 to match: QLinear Binary-Unary pattern with fp32/bfloat16 output
        # Covers case (6) of int8-mixed-bf16
        binary_replace_float_out_patterns = {}
        for swap_binary_inputs in swap_binary_inputs_list:
            binary_replace_float_out_patterns.update(
                {
                    PostOpAttr("add", 1.0, "relu", [], ""): generate_pattern_with_unary(
                        generate_pattern_with_binary(
                            aten.add.Tensor,
                            get_qlinear_pt2e_pattern(x_scale_zp_are_tensors),
                            KeywordArg("other"),
                            dtype_convert=True,
                            swap_inputs=swap_binary_inputs,
                        ),
                        aten.relu.default,
                    ),
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `dtype_convert`, `swap_inputs`, and `binary_replace_float_out_patterns`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。初始化或更新了 `dtype_convert`、`swap_inputs`、`binary_replace_float_out_patterns` 等值。

### Lines 3637-3672 / 第 3637-3672 行
````python
                }
            )
        for (
            binary_unary_attr,
            patterns,
        ) in binary_replace_float_out_patterns.items():
            _register_qlinear_post_op_fusion_pass(
                patterns,
                4,  # pass_number
                qlinear_binary_op,  # computation_op
                binary_unary_attr,
            )

        # Priority 3.1: QLinear Binary pattern with fp32/bfloat16 output
        # Covers case (2) of int8-mixed-fp32 and case (2)(4) of int8-mixed-bf16,
        # totally 2 patterns (2 are identical)
        binary_replace_float_out_patterns = {}
        for swap_binary_inputs in swap_binary_inputs_list:
            binary_replace_float_out_patterns.update(
                {
                    PostOpAttr(
                        "sum", 1.0, "none", [], ""
                    ): generate_pattern_with_binary(
                        aten.add.Tensor,
                        get_qlinear_pt2e_pattern(x_scale_zp_are_tensors),
                        KeywordArg("accum"),
                        dtype_convert=False,
                        swap_inputs=swap_binary_inputs,
                    ),
                }
            )
        for (
            binary_unary_attr,
            patterns,
        ) in binary_replace_float_out_patterns.items():
            _register_qlinear_post_op_fusion_pass(
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `binary_replace_float_out_patterns`, `dtype_convert`, and `swap_inputs`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。初始化或更新了 `binary_replace_float_out_patterns`、`dtype_convert`、`swap_inputs` 等值。

### Lines 3673-3708 / 第 3673-3708 行
````python
                patterns,
                5,  # pass_number
                qlinear_binary_op,  # computation_op
                binary_unary_attr,
            )
        # Priority 3.2: QLinear Binary pattern with fp32/bfloat16 output
        # Covers (6) of int8-mixed-bf16
        binary_replace_float_out_patterns = {}
        for swap_binary_inputs in swap_binary_inputs_list:
            binary_replace_float_out_patterns.update(
                {
                    PostOpAttr(
                        "add", 1.0, "none", [], ""
                    ): generate_pattern_with_binary(
                        aten.add.Tensor,
                        get_qlinear_pt2e_pattern(x_scale_zp_are_tensors),
                        KeywordArg("other"),
                        dtype_convert=True,
                        swap_inputs=swap_binary_inputs,
                    ),
                }
            )
        for (
            binary_unary_attr,
            patterns,
        ) in binary_replace_float_out_patterns.items():
            _register_qlinear_post_op_fusion_pass(
                patterns,
                5,  # pass_number
                qlinear_binary_op,  # computation_op
                binary_unary_attr,
            )


@functools.cache
def _register_quantization_weight_pack_pass():
````
- **EN**: Introduces function `_register_quantization_weight_pack_pass`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`_register_quantization_weight_pack_pass`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 3709-3744 / 第 3709-3744 行
````python
    # Step 1: Dequant promotion for int8-mixed-fp32/bf16
    _register_dequant_promotion()

    # Step 2: QConv weight prepack
    _register_qconv_weight_prepack()

    # Step 3: QLinear weight prepack
    _register_qlinear_weight_prepack()
    _register_linear_dynamic_fp16_weight_prepack()

    # Step 4: weight prepack for SmoothQuant from Torchao
    _register_smooth_quant_int_mm_pattern()

    # Step 5: QLinear post op Fusion
    if not torch.ops.mkldnn._is_mkldnn_acl_supported():
        # skip fusion on ARM
        _register_qconv_unary_fusion()
        _register_qconv_binary_fusion()
        _register_qlinear_unary_fusion()
        _register_qlinear_binary_fusion()


def _is_valid_concat_linear_woq_int4_fusion(computation_nodes):
    computation_op = torch.ops.aten._weight_int4pack_mm_for_cpu.default
    act = computation_nodes[0].args[0]
    wgt = computation_nodes[0].args[1]
    in_feature_size = wgt.meta.get("val").size(1)  # type: ignore[union-attr]
    group_size = computation_nodes[0].args[2]
    return len(computation_nodes) >= 2 and all(
        (
            node.target == computation_op
            and node.args[0] == act  # share same activation
            and (
                node.args[1].meta.get("val").size(1) == in_feature_size
            )  # same in feature size
            and (node.args[1] != wgt or gemm_idx == 0)
````
- **EN**: Introduces function `_is_valid_concat_linear_woq_int4_fusion`. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_is_valid_concat_linear_woq_int4_fusion`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 3745-3780 / 第 3745-3780 行
````python
            and node.args[1].op == "get_attr"  # wgt are all constants
            and node.args[2] == group_size  # same group size
        )
        for gemm_idx, node in enumerate(computation_nodes)
    )


def concat_linear_woq_int4(gm: torch.fx.GraphModule):
    """
    Concat Linear optimization pass for WOQ int4
    This pass fuses the original pattern:
    def ...
        return (woq_int4(x, w1, group_size, scale_zp1), woq_int4(x, w2, group_size, scale_zp1) ...)
    into a single operation:
    def ...
        concat_res = woq_int4(x, concat_w, group_size, concat_scale_zp)
        return split(concat_res, split_size_list)
    """

    def concat_wgt(packed_wgts, scale_zps, group_size, act_dtype):
        # Concat the wgts and scale_zps, and repack the wgt
        unpacked_wgts = []
        for packed_wgt in packed_wgts:
            # Get the unpacked weight list
            # Same as https://github.com/pytorch/pytorch/pull/156174
            K = packed_wgt.size(1) * 2
            N = packed_wgt.size(0)
            x = torch.eye(K).to(dtype=act_dtype)
            qscales_and_zeros = (
                torch.tensor([1.0, 8.0])
                .to(dtype=act_dtype)
                .expand(K // group_size, N, 2)
                .contiguous()
            )
            unpacked_wgts.append(
                torch.ops.aten._weight_int4pack_mm_for_cpu(
````
- **EN**: Introduces function `concat_linear_woq_int4`, function `concat_wgt`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`concat_linear_woq_int4`、函数`concat_wgt`。保留了指向设计说明、规范或厂商数据手册的注释引用。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 3781-3816 / 第 3781-3816 行
````python
                    x,
                    packed_wgt,
                    group_size,
                    qscales_and_zeros,
                )
                .t()
                .contiguous()
                .to(torch.int32)  # N, K
            )
        concat_unpacked_wgt = torch.cat(unpacked_wgts, dim=0)
        repack_w = torch.ops.aten._convert_weight_to_int4pack_for_cpu(
            concat_unpacked_wgt, 1
        )
        concat_scale_zp = torch.cat(scale_zps, dim=1).contiguous()
        return repack_w, concat_scale_zp

    graph = gm.graph
    computation_op = torch.ops.aten._weight_int4pack_mm_for_cpu.default
    for node in graph.find_nodes(op="call_function", target=computation_op):
        if (
            not node._erased
            and isinstance(node.meta.get("val"), torch.Tensor)
            and node.meta["val"].device.type == "cpu"
        ):
            act = node.args[0]
            users = list(act.users)
            if _is_valid_concat_linear_woq_int4_fusion(users):
                with graph.inserting_before(node):
                    assert all(user.args[1].op == "get_attr" for user in users)
                    computation_node_0 = users[0]
                    packed_wgts = [getattr(gm, user.args[1].target) for user in users]
                    group_size = computation_node_0.args[2]
                    scale_zps = [getattr(gm, user.args[3].target) for user in users]
                    out_feature_size_list = [
                        packed_wgt.size(0) for packed_wgt in packed_wgts
                    ]
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `concat_unpacked_wgt`, `repack_w`, `concat_scale_zp`, `graph`, `computation_op`, `act`, and `...+6`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `concat_unpacked_wgt`、`repack_w`、`concat_scale_zp`、`graph`、`computation_op`、`act`、`另有6项` 等值。

### Lines 3817-3852 / 第 3817-3852 行
````python
                    repack_w, concat_scale_zp = concat_wgt(
                        packed_wgts, scale_zps, group_size, act.meta.get("val").dtype
                    )
                    repack_w_node_name = computation_node_0.args[1].target + "_concat"
                    concat_scale_zp_node_name = (
                        computation_node_0.args[3].target + "_concat"
                    )
                    gm.register_buffer(repack_w_node_name, repack_w)
                    setattr(gm, repack_w_node_name, repack_w)
                    gm.register_buffer(concat_scale_zp_node_name, concat_scale_zp)
                    setattr(gm, concat_scale_zp_node_name, concat_scale_zp)

                    repack_w_node = graph.create_node(
                        "get_attr", repack_w_node_name, (), {}
                    )
                    with graph.inserting_after(repack_w_node):
                        concat_scale_zp_node = graph.create_node(
                            "get_attr", concat_scale_zp_node_name, (), {}
                        )

                    with graph.inserting_after(concat_scale_zp_node):
                        concat_int4_gemm_node = graph.create_node(
                            "call_function",
                            computation_op,
                            (
                                act,
                                repack_w_node,
                                group_size,
                                concat_scale_zp_node,
                            ),
                        )
                    with graph.inserting_after(concat_int4_gemm_node):
                        split_node = graph.create_node(
                            "call_function",
                            torch.ops.aten.split_with_sizes.default,
                            (
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `repack_w_node_name`, `concat_scale_zp_node_name`, `repack_w_node`, `concat_scale_zp_node`, `concat_int4_gemm_node`, and `split_node`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。初始化或更新了 `repack_w_node_name`、`concat_scale_zp_node_name`、`repack_w_node`、`concat_scale_zp_node`、`concat_int4_gemm_node`、`split_node` 等值。

### Lines 3853-3888 / 第 3853-3888 行
````python
                                concat_int4_gemm_node,
                                out_feature_size_list,
                                1,  # split dim
                            ),
                        )
                    with graph.inserting_after(split_node):
                        for gemm_idx, user in enumerate(users):
                            assert user.target == computation_op
                            get_item = graph.create_node(
                                "call_function",
                                operator.getitem,
                                (
                                    split_node,
                                    gemm_idx,
                                ),
                            )
                            with graph.inserting_after(get_item):
                                clone_node = graph.create_node(
                                    "call_function",
                                    torch.ops.aten.clone.default,
                                    (get_item,),
                                    {"memory_format": torch.contiguous_format},
                                )
                                user.replace_all_uses_with(clone_node)
                                graph.erase_node(user)


def quant_lift_up(graph_module: torch.fx.GraphModule):
    """
    Lift up the quant node before view like nodes. It can benefit performance
    of Attention like block. For example, we have the pattern as:

             DQ
    DQ       LINEAR
    LINEAR   VIEW
    VIEW     PERMUTE
````
- **EN**: Introduces function `quant_lift_up`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `get_item`, and `clone_node`.
- **CN**: 这里定义了函数`quant_lift_up`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `get_item`、`clone_node` 等值。

### Lines 3889-3924 / 第 3889-3924 行
````python
    PERMUTE  TRANSPOSE
    Q        Q
    DQ       DQ
       Matmul
        DIV
        ADD
      SOFTMAX

    We want to lift up the quant nodes from matmul before view like nodes
    as the output of Linear node.

             DQ
    DQ       LINEAR
    LINEAR   Q
    Q        VIEW
    VIEW     PERMUTE
    PERMUTE  TRANSPOSE
    DQ       DQ
       Matmul
        DIV
        ADD
      SOFTMAX

    It produces a DQ->LINEAR->Q pattern which can be fused by backend.
    """

    def is_view_op(node):
        return node.op == "call_function" and node.target in _VIEW_OPS

    for node in graph_module.graph.nodes:
        # <TODO> Leslie: Here we verify that the quant node has exactly
        # one input FX node, with constant scalar value for scale and zero point.
        # For the case input of quant node has more than one input FX nodes,
        # extend the implementation to lift up all the connected nodes
        # before the view nodes to keep the topological order.
        if (
````
- **EN**: Introduces function `is_view_op`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`is_view_op`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 3925-3960 / 第 3925-3960 行
````python
            node.op == "call_function"
            and node.target in _PER_TENSOR_QUANTIZE_OPS
            and len(node.all_input_nodes) == 1
            and is_view_op(node.all_input_nodes[0])
        ):
            quant_node = node
            input_node_of_quant = quant_node.args[0]

            # Check the nodes along lift up path has only 1 user node
            # Propagate view like node to find where to insert the new quant node
            could_lift_up = True
            current_node = quant_node
            input_node = current_node.args[0]
            while is_view_op(input_node):
                if len(input_node.users) != 1:
                    could_lift_up = False
                    break
                current_node = input_node
                input_node = current_node.args[0]

            # Further check the input node of the first view node has only 1 user node
            if could_lift_up and len(input_node.users) == 1:
                counters["inductor"]["quant_lift_up_count"] += 1
                # Replace dequant's input from quant to quant's input
                quant_node.replace_all_uses_with(input_node_of_quant)
                # Insert the new quant node
                with graph_module.graph.inserting_before(current_node):
                    new_quant_node = graph_module.graph.node_copy(quant_node)
                    input_node.replace_all_uses_with(new_quant_node)

                    # Update inputs of new_quant_node
                    def maybe_replace_node(n: torch.fx.Node) -> torch.fx.Node:
                        if n == input_node_of_quant:
                            return input_node
                        else:
                            return n
````
- **EN**: Introduces function `maybe_replace_node`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `quant_node`, `input_node_of_quant`, `could_lift_up`, `current_node`, `input_node`, `new_quant_node`, and `...+1`.
- **CN**: 这里定义了函数`maybe_replace_node`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `quant_node`、`input_node_of_quant`、`could_lift_up`、`current_node`、`input_node`、`new_quant_node`、`另有1项` 等值。

### Lines 3961-3969 / 第 3961-3969 行
````python

                    new_args = map_arg(new_quant_node.args, maybe_replace_node)
                    new_kwargs = map_arg(new_quant_node.kwargs, maybe_replace_node)
                    new_quant_node.args = new_args  # type: ignore[assignment]
                    new_quant_node.kwargs = new_kwargs  # type: ignore[assignment]
                    graph_module.graph.erase_node(quant_node)

    graph_module.graph.lint()
    graph_module.recompile()
````
- **EN**: Initializes or updates values such as `new_args`, and `new_kwargs`. This range continues the implementation of function `quant_lift_up`.
- **CN**: 初始化或更新了 `new_args`、`new_kwargs` 等值。这一段延续了函数`quant_lift_up` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `PostOpAttr`  
  **CN**: 主要类：`PostOpAttr`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `copy`, `functools`, `itertools`, `math`, `operator`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._dynamo.utils`, `torch.fx.experimental.symbolic_shapes`, `torch.fx.node`, `..`, `..lowering`, `..pattern_matcher`, `..utils`, `.freezing_patterns`, `.post_grad`, `.mkldnn_fusion`
