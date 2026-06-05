# binary_folding.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/binary_folding.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It exposes functions such as `mark_mixed_dtype`, `mark_mixed_dtype_allowed_computation_ops`, `recover_original_precision_folded_computation_ops`, and `binary_folding_init`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。同时提供 `mark_mixed_dtype`、`mark_mixed_dtype_allowed_computation_ops`、`recover_original_precision_folded_computation_ops`、`binary_folding_init` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import functools
import itertools

import torch

from ..._dynamo.utils import counters
from .. import config
from ..pattern_matcher import Arg, CallFunction, KeywordArg
from .freezing_patterns import register_binary_folding_pattern


aten = torch.ops.aten
prims = torch.ops.prims


def mark_mixed_dtype(computation_node):
    computation_node_dtype = computation_node.meta["val"].dtype
    if computation_node_dtype not in (torch.float16, torch.bfloat16):
        return
````
- **EN**: Imports dependencies such as `functools`, `itertools`, `torch`, `..._dynamo.utils`, `..`, `..pattern_matcher`, and `...+1` for the logic in this range. Introduces function `mark_mixed_dtype`. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里导入了 `functools`、`itertools`、`torch`、`..._dynamo.utils`、`..`、`..pattern_matcher`、`另有1项` 等依赖，为后续逻辑提供基础能力。这里定义了函数`mark_mixed_dtype`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 21-40 / 第 21-40 行
````python

    if len(computation_node.users) != 1:
        return

    computation_node_user = next(iter(computation_node.users.keys()))
    if not isinstance(computation_node_user.meta["val"], torch.Tensor):
        return

    if computation_node_user.meta["val"].dtype != torch.float32:
        return

    while computation_node_user.target in _binary_ops:
        if len(computation_node_user.users) != 1:
            return

        computation_node_user = next(iter(computation_node_user.users.keys()))

    if computation_node_user.target != prims.convert_element_type.default:
        return

````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `computation_node_user`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `computation_node_user` 等值。

### Lines 41-60 / 第 41-60 行
````python
    computation_node.meta["_allow_mixed_dtype_folding"] = computation_node_dtype


def mark_mixed_dtype_allowed_computation_ops(gm):
    """
    Mark convolutions/linear which we will binary fold even with mixed precision constants. We constant fold in the higher precision
    for better accuracy and then recover the original precision after.
    """
    for target in [aten.convolution.default, aten.addmm.default, aten.mm.default]:
        for node in gm.graph.find_nodes(op="call_function", target=target):
            mark_mixed_dtype(node)


def recover_original_precision_folded_computation_ops(gm):
    """
    After binary folding conv/linear weights and biases to a higher dtype, recover the original precision they were in.
    """
    graph = gm.graph
    for target, idx in (
        (aten.convolution.default, (1, 2)),
````
- **EN**: Introduces function `mark_mixed_dtype_allowed_computation_ops`, function `recover_original_precision_folded_computation_ops`. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `graph`.
- **CN**: 这里定义了函数`mark_mixed_dtype_allowed_computation_ops`、函数`recover_original_precision_folded_computation_ops`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `graph` 等值。

### Lines 61-80 / 第 61-80 行
````python
        (aten.addmm.default, (0, 2)),
        (aten.mm.default, (1,)),
    ):
        for node in graph.find_nodes(op="call_function", target=target):
            orig_dtype = node.meta.get("_allow_mixed_dtype_folding", None)
            if orig_dtype is None:
                continue

            with graph.inserting_before(node):
                for i in idx:
                    old_input = node.args[i]
                    if old_input is None:
                        continue

                    new_input = graph.create_node(
                        "call_function",
                        prims.convert_element_type.default,
                        (old_input, orig_dtype),
                    )
                    node.replace_input_with(old_input, new_input)
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `orig_dtype`, `old_input`, and `new_input`. This range continues the implementation of function `recover_original_precision_folded_computation_ops`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `orig_dtype`、`old_input`、`new_input` 等值。这一段延续了函数`recover_original_precision_folded_computation_ops` 的具体实现。

### Lines 81-100 / 第 81-100 行
````python


_binary_ops = [aten.add.Tensor, aten.sub.Tensor, aten.mul.Tensor, aten.div.Tensor]


@functools.cache
def binary_folding_init():
    _conv_args = [Arg() for _ in range(9)]
    _addmm_args = [Arg() for _ in range(3)]
    _mm_args = [Arg() for _ in range(2)]
    _computation_ops = [aten.convolution.default, aten.addmm.default, aten.mm.default]
    _computation_calls = [
        CallFunction(aten.convolution.default, *_conv_args, _users=1),
        CallFunction(aten.addmm.default, *_addmm_args, _users=1),
        CallFunction(
            aten.reshape.default,
            CallFunction(aten.addmm.default, *_addmm_args, _users=1),
            Arg(),
            _users=1,
        ),
````
- **EN**: Introduces function `binary_folding_init`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Initializes or updates values such as `_binary_ops`, `_conv_args`, `_addmm_args`, `_mm_args`, `_computation_ops`, `_computation_calls`, and `...+1`.
- **CN**: 这里定义了函数`binary_folding_init`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。初始化或更新了 `_binary_ops`、`_conv_args`、`_addmm_args`、`_mm_args`、`_computation_ops`、`_computation_calls`、`另有1项` 等值。

### Lines 101-120 / 第 101-120 行
````python
        CallFunction(aten.mm.default, *_mm_args, _users=1),
        CallFunction(
            aten.reshape.default,
            CallFunction(aten.mm.default, *_mm_args, _users=1),
            Arg(),
            _users=1,
        ),
    ]

    """
    In order to fuse add/sub/mul/div with conv/linear, the dimensions of its
    constant tensor must satisfy the following:
    - with resizing, broadcast to w/ weight/bias tensor shape
    - broadcast to the conv/linear output shape
    It needs to have a shape that can resize to weight/bias
    tensor shape because we need to run the op with the conv/linear
    weights/bias without changing their sizes.
    It needs to broadcast to the conv/linear output shape so that we do
    accidentally change the shape of op output by pre-fusing it
    compared to eager.
````
- **EN**: Initializes or updates values such as `_users`. This range continues the implementation of function `binary_folding_init`.
- **CN**: 初始化或更新了 `_users` 等值。这一段延续了函数`binary_folding_init` 的具体实现。

### Lines 121-140 / 第 121-140 行
````python
    The only dimension value shared by weight, bias, and conv/linear output
    is they all contain a dim with value = channels-out. In the
    conv/linear output tensor, this is in the second dimension,
    so the pointwise op tensor may have a second dimension of
    value == channels-out, but all the other dimensions have to be 1
    """

    def _op_not_broadcasting_with_conv(weight_tensor, other_tensor):
        # According to opDoesNotBroadCastWithConv of frozen_conv_folding.cpp
        weight_shape = weight_tensor.shape
        other_shape = other_tensor.shape
        if len(weight_shape) < len(other_shape):
            return False
        if len(weight_shape) == len(other_shape) + 1:
            # weight shape is [o, i, *], other_shape is [o, 1...].
            for i in reversed(range(len(other_shape))):
                if i == 0 and weight_shape[0] == other_shape[i]:
                    continue
                if other_shape[i] != 1:
                    return False
````
- **EN**: Introduces function `_op_not_broadcasting_with_conv`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `value`, `weight_shape`, and `other_shape`.
- **CN**: 这里定义了函数`_op_not_broadcasting_with_conv`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `value`、`weight_shape`、`other_shape` 等值。

### Lines 141-160 / 第 141-160 行
````python
        else:
            # weight shape is [o, i, *], other_shape is [1, i, *]
            for i in reversed(range(len(other_shape))):
                if i == 1 and weight_shape[0] == other_shape[i]:
                    continue
                if other_shape[i] != 1:
                    return False
        return True

    def _op_not_broadcasting_with_linear(weight_tensor, other_tensor, has_reshape):
        weight_shape = weight_tensor.shape
        other_shape = other_tensor.shape
        other_shapes = [
            torch.Size(
                [
                    weight_shape[1],
                ]
            ),
            torch.Size([1, weight_shape[1]]),
            torch.Size(
````
- **EN**: Introduces function `_op_not_broadcasting_with_linear`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `weight_shape`, `other_shape`, and `other_shapes`.
- **CN**: 这里定义了函数`_op_not_broadcasting_with_linear`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`weight_shape`、`other_shape`、`other_shapes` 等值。

### Lines 161-180 / 第 161-180 行
````python
                [
                    1,
                ]
            ),
            torch.Size([1, 1]),
        ]
        if has_reshape:
            other_shapes.extend(
                [
                    torch.Size([1, 1, weight_shape[1]]),
                    torch.Size([1, 1, 1]),
                ]
            )
        return other_shape in other_shapes

    def _check_conv_and_broadcast_op(conv_node, other):
        # According to checkConvAndBroadcastingOpPreConditions of frozen_conv_folding.cpp.
        # conv.weight
        if conv_node.args[1].op != "get_attr":
            return False
````
- **EN**: Introduces function `_check_conv_and_broadcast_op`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_check_conv_and_broadcast_op`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 181-200 / 第 181-200 行
````python
        # conv.bias
        if conv_node.args[1] is not None and conv_node.args[1].op != "get_attr":
            return False
        if (
            not isinstance(other, int)
            and not isinstance(other, float)
            and other.op != "get_attr"
        ):
            return False

        if len(conv_node.args[1].users) != 1:
            return False

        weight_meta_value = conv_node.args[1].meta.get("val")
        if weight_meta_value is None:
            return False
        # Avoid fusing op that causes type promotion
        # restricting to float avoids int/float difficulties with scalar overload
        if not weight_meta_value.is_floating_point():
            return False
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `weight_meta_value`. This range continues the implementation of function `binary_folding_init._check_conv_and_broadcast_op`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `weight_meta_value` 等值。这一段延续了函数`binary_folding_init._check_conv_and_broadcast_op` 的具体实现。

### Lines 201-220 / 第 201-220 行
````python
        if isinstance(other, torch.fx.Node) and other.op == "get_attr":
            other_meta_value = other.meta.get("val")
            if not other_meta_value.is_floating_point():  # type: ignore[union-attr]
                return False
            if (
                torch.promote_types(other_meta_value.dtype, weight_meta_value.dtype)  # type: ignore[union-attr]
                != weight_meta_value.dtype
            ):
                if not conv_node.meta.get("_allow_mixed_dtype_folding", False):
                    return False

                if (
                    other_meta_value.dtype != torch.float  # type: ignore[union-attr]
                    and weight_meta_value.dtype not in (torch.float16, torch.bfloat16)
                ):
                    return False

            if not _op_not_broadcasting_with_conv(weight_meta_value, other_meta_value):
                return False
        elif not isinstance(other, float):
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `other_meta_value`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `other_meta_value` 等值。

### Lines 221-240 / 第 221-240 行
````python
            return False

        return True

    def _check_linear_and_broadcast_op(linear_node, other, has_reshape):
        weight_node = (
            linear_node.args[2]
            if linear_node.target is aten.addmm.default
            else linear_node.args[1]
        )
        bias_node = (
            linear_node.args[0] if linear_node.target is aten.addmm.default else None
        )
        if weight_node.op != "get_attr":
            return False
        if bias_node is not None and bias_node.op != "get_attr":
            return False
        if (
            not isinstance(other, int)
            and not isinstance(other, float)
````
- **EN**: Introduces function `_check_linear_and_broadcast_op`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `weight_node`, and `bias_node`.
- **CN**: 这里定义了函数`_check_linear_and_broadcast_op`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `weight_node`、`bias_node` 等值。

### Lines 241-260 / 第 241-260 行
````python
            and other.op != "get_attr"
        ):
            return False

        if len(weight_node.users) != 1:
            return False

        weight_meta_value = weight_node.meta.get("val")
        if weight_meta_value is None:
            return False
        # Avoid fusing op that causes type promotion
        # restricting to float avoids int/float difficulties with scalar overload
        if not weight_meta_value.is_floating_point():
            return False
        if isinstance(other, torch.fx.Node) and other.op == "get_attr":
            other_meta_value = other.meta.get("val")
            if not other_meta_value.is_floating_point():  # type: ignore[union-attr]
                return False
            if (
                torch.promote_types(other_meta_value.dtype, weight_meta_value.dtype)  # type: ignore[union-attr]
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `weight_meta_value`, and `other_meta_value`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `weight_meta_value`、`other_meta_value` 等值。

### Lines 261-280 / 第 261-280 行
````python
                != weight_meta_value.dtype
            ):
                if not linear_node.meta.get("_allow_mixed_dtype_folding", False):
                    return False

                if (
                    other_meta_value.dtype != torch.float  # type: ignore[union-attr]
                    and weight_meta_value.dtype not in (torch.float16, torch.bfloat16)
                ):
                    return False

            if not _op_not_broadcasting_with_linear(
                weight_meta_value, other_meta_value, has_reshape
            ):
                return False
        elif not isinstance(other, float):
            return False

        return True

````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `binary_folding_init._check_linear_and_broadcast_op`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了函数`binary_folding_init._check_linear_and_broadcast_op` 的具体实现。

### Lines 281-300 / 第 281-300 行
````python
    def _is_foldable_pattern(match):
        binary_node = match.output_node()
        has_reshape = False
        if binary_node.args[0].target in _computation_ops:
            computation_node = binary_node.args[0]
            other = binary_node.args[1]
        elif binary_node.args[0].target is aten.reshape.default:
            computation_node = binary_node.args[0].args[0]
            other = binary_node.args[1]
            has_reshape = True
        elif binary_node.args[1].target in _computation_ops:
            computation_node = binary_node.args[1]
            other = binary_node.args[0]
        else:
            computation_node = binary_node.args[1].args[0]
            other = binary_node.args[0]
            has_reshape = False
        if computation_node.target is aten.convolution.default:
            return _check_conv_and_broadcast_op(computation_node, other)
        elif computation_node.target in [aten.addmm.default, aten.mm.default]:
````
- **EN**: Introduces function `_is_foldable_pattern`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `binary_node`, `has_reshape`, `computation_node`, `other`, and `else`.
- **CN**: 这里定义了函数`_is_foldable_pattern`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `binary_node`、`has_reshape`、`computation_node`、`other`、`else` 等值。

### Lines 301-320 / 第 301-320 行
````python
            return (
                config.enable_linear_binary_folding
                and _check_linear_and_broadcast_op(computation_node, other, has_reshape)
            )

        return False

    def resize_scalar_or_tensor_to_shape(graph, other, shape, weight):
        if isinstance(other, float):
            with torch.utils._python_dispatch._disable_current_modes():
                other_tensor = torch.tensor(
                    other, dtype=weight.dtype, device=weight.device
                )
            graph.owning_module.register_buffer("other_tensor", other_tensor)
            res = graph.create_node("get_attr", "other_tensor")
            res = graph.create_node(
                "call_function",
                aten.reshape.default,
                (res, (1,)),
            )
````
- **EN**: Introduces function `resize_scalar_or_tensor_to_shape`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`resize_scalar_or_tensor_to_shape`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 321-340 / 第 321-340 行
````python
            res = graph.create_node(
                "call_function",
                aten.expand.default,
                (res, shape),
            )
        elif other.meta.get("val").numel() == 1:
            # expand errors if the shape input has less # dims than the tensor input
            res = graph.create_node(
                "call_function",
                aten.reshape.default,
                (other, (1,)),
            )
            res = graph.create_node(
                "call_function",
                aten.expand.default,
                (res, shape),
            )
        else:
            res = graph.create_node(
                "call_function",
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `res`, and `else`. This range continues the implementation of function `binary_folding_init.resize_scalar_or_tensor_to_shape`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `res`、`else` 等值。这一段延续了函数`binary_folding_init.resize_scalar_or_tensor_to_shape` 的具体实现。

### Lines 341-360 / 第 341-360 行
````python
                aten.reshape.default,
                (other, shape),
            )
        return res

    def _create_new_conv_node(graph, conv_node, binary_node, other):
        assert conv_node.target is aten.convolution.default
        conv_args = list(conv_node.args)
        weight_meta_value = conv_node.args[1].meta.get("val")
        bias = conv_args[2]
        if binary_node.target in [aten.add.Tensor, aten.sub.Tensor]:
            other_reshape = resize_scalar_or_tensor_to_shape(
                graph,
                other,
                (weight_meta_value.size(0),),
                weight_meta_value,
            )
            new_bias = graph.create_node(
                "call_function",
                binary_node.target,
````
- **EN**: Introduces function `_create_new_conv_node`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `conv_args`, `weight_meta_value`, `bias`, `other_reshape`, and `new_bias`.
- **CN**: 这里定义了函数`_create_new_conv_node`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `conv_args`、`weight_meta_value`、`bias`、`other_reshape`、`new_bias` 等值。

### Lines 361-380 / 第 361-380 行
````python
                (0 if bias is None else bias, other_reshape),
            )
            conv_args[2] = new_bias
        else:
            assert binary_node.target in [aten.mul.Tensor, aten.div.Tensor]
            weight_broadcast_shape = [1 for _ in range(len(weight_meta_value.shape))]
            weight_broadcast_shape[0] = weight_meta_value.size(0)
            other_reshape1 = resize_scalar_or_tensor_to_shape(
                graph,
                other,
                tuple(weight_broadcast_shape),
                weight_meta_value,
            )
            new_weight = graph.create_node(
                "call_function", binary_node.target, (conv_args[1], other_reshape1)
            )
            new_weight.meta.update(conv_args[1].meta)
            conv_args[1] = new_weight
            if bias is not None:
                other_reshape = resize_scalar_or_tensor_to_shape(
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `weight_broadcast_shape`, `other_reshape1`, `new_weight`, and `other_reshape`. This range continues the implementation of function `binary_folding_init._create_new_conv_node`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`weight_broadcast_shape`、`other_reshape1`、`new_weight`、`other_reshape` 等值。这一段延续了函数`binary_folding_init._create_new_conv_node` 的具体实现。

### Lines 381-400 / 第 381-400 行
````python
                    graph,
                    other,
                    (weight_meta_value.size(0),),
                    weight_meta_value,
                )
                new_bias = graph.create_node(
                    "call_function", binary_node.target, (bias, other_reshape)
                )
                new_bias.meta.update(bias.meta)
                conv_args[2] = new_bias
        return graph.create_node("call_function", conv_node.target, tuple(conv_args))

    def _create_new_linear_node(graph, linear_node, binary_node, other):
        assert linear_node.target in [aten.addmm.default, aten.mm.default]
        input_node = (
            linear_node.args[1]
            if linear_node.target is aten.addmm.default
            else linear_node.args[0]
        )
        weight_node = (
````
- **EN**: Introduces function `_create_new_linear_node`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `new_bias`, `input_node`, and `weight_node`.
- **CN**: 这里定义了函数`_create_new_linear_node`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `new_bias`、`input_node`、`weight_node` 等值。

### Lines 401-420 / 第 401-420 行
````python
            linear_node.args[2]
            if linear_node.target is aten.addmm.default
            else linear_node.args[1]
        )
        bias_node = (
            linear_node.args[0] if linear_node.target is aten.addmm.default else None
        )
        weight_meta_value = weight_node.meta.get("val")
        if binary_node.target in [aten.add.Tensor, aten.sub.Tensor]:
            other_reshape = resize_scalar_or_tensor_to_shape(
                graph,
                other,
                (weight_meta_value.size(1),),
                weight_meta_value,
            )
            new_bias_node = graph.create_node(
                "call_function",
                binary_node.target,
                (0 if bias_node is None else bias_node, other_reshape),
            )
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `bias_node`, `weight_meta_value`, `other_reshape`, and `new_bias_node`. This range continues the implementation of function `binary_folding_init._create_new_linear_node`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `bias_node`、`weight_meta_value`、`other_reshape`、`new_bias_node` 等值。这一段延续了函数`binary_folding_init._create_new_linear_node` 的具体实现。

### Lines 421-440 / 第 421-440 行
````python
            return graph.create_node(
                "call_function",
                aten.addmm.default,
                (new_bias_node, input_node, weight_node),
            )
        else:
            assert binary_node.target in [aten.mul.Tensor, aten.div.Tensor]
            weight_broadcast_shape = [1, weight_meta_value.size(1)]
            other_reshape1 = resize_scalar_or_tensor_to_shape(
                graph,
                other,
                tuple(weight_broadcast_shape),
                weight_meta_value,
            )
            new_weight_node = graph.create_node(
                "call_function", binary_node.target, (weight_node, other_reshape1)
            )
            new_weight_node.meta.update(weight_node.meta)
            if bias_node is not None:
                other_reshape = resize_scalar_or_tensor_to_shape(
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `weight_broadcast_shape`, `other_reshape1`, `new_weight_node`, and `other_reshape`. This range continues the implementation of function `binary_folding_init._create_new_linear_node`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`weight_broadcast_shape`、`other_reshape1`、`new_weight_node`、`other_reshape` 等值。这一段延续了函数`binary_folding_init._create_new_linear_node` 的具体实现。

### Lines 441-460 / 第 441-460 行
````python
                    graph,
                    other,
                    (weight_meta_value.size(1),),
                    weight_meta_value,
                )
                new_bias_node = graph.create_node(
                    "call_function", binary_node.target, (bias_node, other_reshape)
                )
                new_bias_node.meta.update(bias_node.meta)
                return graph.create_node(
                    "call_function",
                    linear_node.target,
                    (new_bias_node, input_node, new_weight_node),
                )
            else:
                return graph.create_node(
                    "call_function", linear_node.target, (input_node, new_weight_node)
                )

    for _computation_call, binary_op in itertools.product(
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `new_bias_node`, and `else`. This range continues the implementation of function `binary_folding_init._create_new_linear_node`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `new_bias_node`、`else` 等值。这一段延续了函数`binary_folding_init._create_new_linear_node` 的具体实现。

### Lines 461-480 / 第 461-480 行
````python
        _computation_calls, _binary_ops
    ):

        @register_binary_folding_pattern(
            CallFunction(binary_op, _computation_call, KeywordArg("other")),
            extra_check=_is_foldable_pattern,
        )
        def folded_op(match, *args, **kwargs):
            counters["inductor"]["binary_folding"] += 1
            other = kwargs.get("other")
            binary_node = match.output_node()
            reshape_node = None
            if binary_node.args[0].target in _computation_ops:
                computation_node = binary_node.args[0]
            elif binary_node.args[0].target is aten.reshape.default:
                computation_node = binary_node.args[0].args[0]
                reshape_node = binary_node.args[0]
            elif binary_node.args[1].target in _computation_ops:
                computation_node = binary_node.args[1]
            else:
````
- **EN**: Introduces function `folded_op`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`folded_op`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。

### Lines 481-500 / 第 481-500 行
````python
                computation_node = binary_node.args[1].args[0]
                reshape_node = binary_node.args[1]
            graph = match.graph
            with graph.inserting_before(reshape_node if reshape_node else binary_node):
                assert computation_node.target in _computation_ops
                if computation_node.target is aten.convolution.default:
                    counters["inductor"]["binary_folding_conv"] += 1
                    new_computation_node = _create_new_conv_node(
                        graph, computation_node, binary_node, other
                    )
                else:
                    new_computation_node = _create_new_linear_node(
                        graph, computation_node, binary_node, other
                    )
                new_computation_node.meta.update(computation_node.meta)
                if reshape_node:
                    assert reshape_node.target is aten.reshape.default
                    computation_node.replace_all_uses_with(new_computation_node)
                    binary_node.replace_all_uses_with(reshape_node)
                else:
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `computation_node`, `reshape_node`, `graph`, `new_computation_node`, and `else`. This range continues the implementation of function `binary_folding_init.folded_op`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `computation_node`、`reshape_node`、`graph`、`new_computation_node`、`else` 等值。这一段延续了函数`binary_folding_init.folded_op` 的具体实现。

### Lines 501-503 / 第 501-503 行
````python
                    binary_node.replace_all_uses_with(new_computation_node)
                graph.erase_node(binary_node)
                graph.erase_node(computation_node)
````
- **EN**: This range continues the implementation of function `binary_folding_init.folded_op`.
- **CN**: 这一段延续了函数`binary_folding_init.folded_op` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary functions: `mark_mixed_dtype`, `mark_mixed_dtype_allowed_computation_ops`, `recover_original_precision_folded_computation_ops`, and `binary_folding_init`  
  **CN**: 主要函数：`mark_mixed_dtype`、`mark_mixed_dtype_allowed_computation_ops`、`recover_original_precision_folded_computation_ops`、`binary_folding_init`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`, `itertools`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `..._dynamo.utils`, `..`, `..pattern_matcher`, `.freezing_patterns`
