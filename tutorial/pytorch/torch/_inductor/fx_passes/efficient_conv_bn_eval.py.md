# efficient_conv_bn_eval.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/efficient_conv_bn_eval.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It exposes functions such as `efficient_conv_bn_eval`, `efficient_conv_bn_eval_decomposed`, `efficient_conv_bn_eval_graph_transform_inlined`, `efficient_conv_bn_eval_graph_transform_decomposed`, and `efficient_conv_bn_eval_graph_transform`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。同时提供 `efficient_conv_bn_eval`、`efficient_conv_bn_eval_decomposed`、`efficient_conv_bn_eval_graph_transform_inlined`、`efficient_conv_bn_eval_graph_transform_decomposed`、`efficient_conv_bn_eval_graph_transform` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import torch
import torch.nn as nn
from torch._dynamo.utils import counters
from torch._inductor import config as inductor_config
from torch.func import functional_call

from ..pattern_matcher import (
    CallFunctionVarArgs,
    CallModuleVarArgs,
    Match,
    register_graph_pattern,
)
from .pre_grad import efficient_conv_bn_eval_pass


def efficient_conv_bn_eval(
    bn: nn.modules.batchnorm._BatchNorm, conv: nn.modules.conv._ConvNd, x: torch.Tensor
):
    """
````
- **EN**: Imports dependencies such as `torch`, `torch.nn`, `torch._dynamo.utils`, `torch._inductor`, `torch.func`, `..pattern_matcher`, and `...+1` for the logic in this range. Introduces function `efficient_conv_bn_eval`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `bn`.
- **CN**: 这里导入了 `torch`、`torch.nn`、`torch._dynamo.utils`、`torch._inductor`、`torch.func`、`..pattern_matcher`、`另有1项` 等依赖，为后续逻辑提供基础能力。这里定义了函数`efficient_conv_bn_eval`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `bn` 等值。

### Lines 21-40 / 第 21-40 行
````python
    Implementation based on https://arxiv.org/abs/2305.11624
    "Efficient ConvBN Blocks for Transfer Learning and Beyond"
    It leverages the associative law between convolution and affine transform,
    i.e., normalize (weight conv feature) = (normalize weight) conv feature.
    It works for Eval mode of ConvBN blocks during validation, and can be used
    for **training** as well, but only if one sets `bn.training=False`. It
     reduces memory footprint and computation cost, at the cost of slightly
     reduced numerical stability.
    Args:
        bn (nn.modules.batchnorm._BatchNorm): a BatchNorm module.
        conv (nn.modules.conv._ConvNd): a conv module
        x (torch.Tensor): Input feature map.
    """

    assert bn.running_var is not None
    assert bn.running_mean is not None

    # These lines of code are designed to deal with various cases
    # like bn without affine transform, and conv without bias
    weight_on_the_fly = conv.weight
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Args`, and `weight_on_the_fly`.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `Args`、`weight_on_the_fly` 等值。

### Lines 41-60 / 第 41-60 行
````python
    if conv.bias is not None:
        bias_on_the_fly = conv.bias
    else:
        bias_on_the_fly = torch.zeros_like(bn.running_var)

    if bn.weight is not None:
        bn_weight = bn.weight
    else:
        bn_weight = torch.ones_like(bn.running_var)

    if bn.bias is not None:
        bn_bias = bn.bias
    else:
        bn_bias = torch.zeros_like(bn.running_var)

    # shape of [C_out, 1, 1, 1] in Conv2d
    target_shape = [-1] + [1] * (conv.weight.ndim - 1)
    if isinstance(conv, nn.modules.conv._ConvTransposeNd):
        # for transposed conv, the C_out dimension should at index 1.
        target_shape[:2] = [target_shape[1], target_shape[0]]
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `bias_on_the_fly`, `else`, `bn_weight`, `bn_bias`, and `target_shape`. This range continues the implementation of function `efficient_conv_bn_eval`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `bias_on_the_fly`、`else`、`bn_weight`、`bn_bias`、`target_shape` 等值。这一段延续了函数`efficient_conv_bn_eval` 的具体实现。

### Lines 61-80 / 第 61-80 行
````python
    weight_coeff = torch.rsqrt(bn.running_var + bn.eps).reshape(target_shape)
    # shape of [C_out, 1, 1, 1] in Conv2d
    coefff_on_the_fly = bn_weight.view_as(weight_coeff) * weight_coeff

    # shape of [C_out, C_in, k, k] in Conv2d
    weight_on_the_fly = weight_on_the_fly * coefff_on_the_fly
    # shape of [C_out] in Conv2d
    bias_on_the_fly = bn_bias + coefff_on_the_fly.flatten() * (
        bias_on_the_fly - bn.running_mean
    )

    input = x
    params = {"weight": weight_on_the_fly, "bias": bias_on_the_fly}
    output = functional_call(conv, params, input)
    return output


def efficient_conv_bn_eval_decomposed(
    bn_weight,
    bn_bias,
````
- **EN**: Introduces function `efficient_conv_bn_eval_decomposed`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `weight_coeff`, `coefff_on_the_fly`, `weight_on_the_fly`, `bias_on_the_fly`, `input`, `params`, and `...+1`.
- **CN**: 这里定义了函数`efficient_conv_bn_eval_decomposed`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `weight_coeff`、`coefff_on_the_fly`、`weight_on_the_fly`、`bias_on_the_fly`、`input`、`params`、`另有1项` 等值。

### Lines 81-100 / 第 81-100 行
````python
    bn_running_mean,
    bn_running_var,
    bn_eps,
    conv: torch._ops.OpOverload,
    conv_weight,
    conv_bias,
    x,
    conv_remaining_args,
):
    """
    Implementation based on https://arxiv.org/abs/2305.11624
    "Efficient ConvBN Blocks for Transfer Learning and Beyond"
    It leverages the associative law between convolution and affine transform,
    i.e., normalize (weight conv feature) = (normalize weight) conv feature.
    It works for Eval mode of ConvBN blocks during validation, and can be used
    for **training** as well, but only if one sets `bn.training=False`. It
     reduces memory footprint and computation cost, at the cost of slightly
     reduced numerical stability.
    Args:
    """
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `conv`, and `Args`. This range continues the implementation of function `efficient_conv_bn_eval_decomposed`.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。包含分支、循环或上下文管理等控制流。初始化或更新了 `conv`、`Args` 等值。这一段延续了函数`efficient_conv_bn_eval_decomposed` 的具体实现。

### Lines 101-120 / 第 101-120 行
````python
    assert bn_running_var is not None

    # These lines of code are designed to deal with various cases
    # like bn without affine transform, and conv without bias
    weight_on_the_fly = conv_weight
    if conv_bias is not None:
        bias_on_the_fly = conv_bias
    else:
        bias_on_the_fly = torch.zeros_like(bn_running_var)

    if bn_weight is None:
        bn_weight = torch.ones_like(bn_running_var)

    if bn_bias is None:
        bn_bias = torch.zeros_like(bn_running_var)

    # shape of [C_out, 1, 1, 1] in Conv2d
    target_shape = [-1] + [1] * (conv_weight.ndim - 1)
    if "conv_transpose" in conv.__str__():
        # for transposed conv, the C_out dimension should at index 1.
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `weight_on_the_fly`, `bias_on_the_fly`, `else`, `bn_weight`, `bn_bias`, and `target_shape`. This range continues the implementation of function `efficient_conv_bn_eval_decomposed`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `weight_on_the_fly`、`bias_on_the_fly`、`else`、`bn_weight`、`bn_bias`、`target_shape` 等值。这一段延续了函数`efficient_conv_bn_eval_decomposed` 的具体实现。

### Lines 121-140 / 第 121-140 行
````python
        target_shape[:2] = [target_shape[1], target_shape[0]]
    weight_coeff = torch.rsqrt(bn_running_var + bn_eps).reshape(target_shape)
    # shape of [C_out, 1, 1, 1] in Conv2d
    coefff_on_the_fly = bn_weight.view_as(weight_coeff) * weight_coeff

    # shape of [C_out, C_in, k, k] in Conv2d
    weight_on_the_fly = weight_on_the_fly * coefff_on_the_fly
    # shape of [C_out] in Conv2d
    bias_on_the_fly = bn_bias + coefff_on_the_fly.flatten() * (
        bias_on_the_fly - bn_running_mean
    )

    input = x
    return conv(*((input, weight_on_the_fly, bias_on_the_fly) + conv_remaining_args))


@register_graph_pattern(
    CallFunctionVarArgs(
        [
            torch.nn.functional.batch_norm,
````
- **EN**: Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `weight_coeff`, `coefff_on_the_fly`, `weight_on_the_fly`, `bias_on_the_fly`, and `input`.
- **CN**: 使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `weight_coeff`、`coefff_on_the_fly`、`weight_on_the_fly`、`bias_on_the_fly`、`input` 等值。

### Lines 141-160 / 第 141-160 行
````python
        ]
    ),
    # pyrefly: ignore [bad-argument-type]
    pass_dict=efficient_conv_bn_eval_pass,
    extra_check=lambda match: not inductor_config.freezing
    and inductor_config.efficient_conv_bn_eval_fx_passes,
)
def efficient_conv_bn_eval_graph_transform_inlined(match: Match, *args, **kwargs):
    bn_node = match.nodes[0]
    graph = match.graph
    assert len(bn_node.args) == 8

    # We can only use efficient conv-bn for eval mode with track_running_stats
    # bn_node.args is `training`
    if bn_node.args[-3]:
        return

    # Check if the input is Conv
    input_node = bn_node.args[0]

````
- **EN**: Introduces function `efficient_conv_bn_eval_graph_transform_inlined`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `pass_dict`, `extra_check`, `bn_node`, `graph`, and `input_node`.
- **CN**: 这里定义了函数`efficient_conv_bn_eval_graph_transform_inlined`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `pass_dict`、`extra_check`、`bn_node`、`graph`、`input_node` 等值。

### Lines 161-180 / 第 161-180 行
````python
    if input_node.op != "call_function":  # type: ignore[union-attr]
        return

    input_fn = input_node.target  # type: ignore[arg-type, union-attr]
    supported_convs = [
        torch._C._nn.linear,
        torch.conv1d,
        torch.conv2d,
        torch.conv3d,
        torch.conv_transpose1d,
        torch.conv_transpose2d,
        torch.conv_transpose3d,
    ]

    if not any(input_fn is cls for cls in supported_convs):
        return

    conv_node = input_node
    # Output of conv is used by other nodes, cannot optimize
    if len(conv_node.users) > 1:  # type: ignore[union-attr]
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_fn`, `supported_convs`, and `conv_node`. This range continues the implementation of function `efficient_conv_bn_eval_graph_transform_inlined`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `input_fn`、`supported_convs`、`conv_node` 等值。这一段延续了函数`efficient_conv_bn_eval_graph_transform_inlined` 的具体实现。

### Lines 181-200 / 第 181-200 行
````python
        return

    counters["inductor"]["efficient_conv_bn_eval"] += 1

    with graph.inserting_before(bn_node):
        # prepare args for the fused function
        bn_running_mean = bn_node.args[1]
        bn_running_var = bn_node.args[2]
        bn_weight = bn_node.args[3]
        bn_bias = bn_node.args[4]
        bn_eps = bn_node.args[7]
        assert len(conv_node.args) >= 2  # type: ignore[union-attr]
        conv_input = conv_node.args[0]  # type: ignore[union-attr]
        conv_weight = conv_node.args[1]  # type: ignore[union-attr]
        conv_bias = conv_node.args[2] if len(conv_node.args) >= 3 else None  # type: ignore[union-attr]
        conv_remaining_args = conv_node.args[3:]  # type: ignore[union-attr]
        args = (
            bn_weight,
            bn_bias,
            bn_running_mean,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `bn_running_mean`, `bn_running_var`, `bn_weight`, `bn_bias`, `bn_eps`, `conv_input`, and `...+4`. This range continues the implementation of function `efficient_conv_bn_eval_graph_transform_inlined`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `bn_running_mean`、`bn_running_var`、`bn_weight`、`bn_bias`、`bn_eps`、`conv_input`、`另有4项` 等值。这一段延续了函数`efficient_conv_bn_eval_graph_transform_inlined` 的具体实现。

### Lines 201-220 / 第 201-220 行
````python
            bn_running_var,
            bn_eps,
            conv_node.target,  # type: ignore[union-attr]
            conv_weight,
            conv_bias,
            conv_input,
            conv_remaining_args,
        )

        # create a new node
        new_node = graph.create_node(
            op="call_function",
            target=efficient_conv_bn_eval_decomposed,
            args=args,  # type: ignore[arg-type]
            name="efficient_conv_bn_eval",
        )

    # this node replaces the original conv + bn, and therefore
    # should replace the uses of bn_node
    bn_node.replace_all_uses_with(new_node)
````
- **EN**: Initializes or updates values such as `new_node`, `op`, `target`, `args`, and `name`. This range continues the implementation of function `efficient_conv_bn_eval_graph_transform_inlined`.
- **CN**: 初始化或更新了 `new_node`、`op`、`target`、`args`、`name` 等值。这一段延续了函数`efficient_conv_bn_eval_graph_transform_inlined` 的具体实现。

### Lines 221-240 / 第 221-240 行
````python
    # take care of the deletion order:
    # delete bn_node first, and then conv_node
    graph.erase_node(bn_node)
    graph.erase_node(conv_node)  # type: ignore[arg-type]

    return


@register_graph_pattern(
    CallFunctionVarArgs(
        [
            torch.ops.aten.batch_norm.default,
        ]
    ),
    # pyrefly: ignore [bad-argument-type]
    pass_dict=efficient_conv_bn_eval_pass,
    extra_check=lambda match: not inductor_config.freezing
    and inductor_config.efficient_conv_bn_eval_fx_passes,
)
def efficient_conv_bn_eval_graph_transform_decomposed(match: Match, *args, **kwargs):
````
- **EN**: Introduces function `efficient_conv_bn_eval_graph_transform_decomposed`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`efficient_conv_bn_eval_graph_transform_decomposed`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 241-260 / 第 241-260 行
````python
    bn_node = match.nodes[0]
    graph = match.graph
    assert len(bn_node.args) == 9

    # We can only use efficient conv-bn for eval mode with track_running_stats
    # bn_node.args is `training`
    if bn_node.args[-4]:
        return

    # Check if the input is Conv
    input_node = bn_node.args[0]

    if input_node.op != "call_function":  # type: ignore[union-attr]
        return

    input_fn = input_node.target  # type: ignore[arg-type, union-attr]
    supported_convs = [
        torch.ops.aten.linear.default,
        torch.ops.aten.conv1d.default,
        torch.ops.aten.conv2d.default,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `bn_node`, `graph`, `input_node`, `input_fn`, and `supported_convs`. This range continues the implementation of function `efficient_conv_bn_eval_graph_transform_decomposed`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `bn_node`、`graph`、`input_node`、`input_fn`、`supported_convs` 等值。这一段延续了函数`efficient_conv_bn_eval_graph_transform_decomposed` 的具体实现。

### Lines 261-280 / 第 261-280 行
````python
        torch.ops.aten.conv3d.default,
        torch.ops.aten.conv_transpose1d.default,
        torch.ops.aten.conv_transpose2d.input,
        torch.ops.aten.conv_transpose3d.input,
    ]

    if not any(input_fn is cls for cls in supported_convs):
        return

    conv_node = input_node
    # Output of conv is used by other nodes, cannot optimize
    if len(conv_node.users) > 1:  # type: ignore[union-attr]
        return

    counters["inductor"]["efficient_conv_bn_eval"] += 1

    with graph.inserting_before(bn_node):
        # prepare args for the fused function
        bn_weight = bn_node.args[1]
        bn_bias = bn_node.args[2]
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `conv_node`, `bn_weight`, and `bn_bias`. This range continues the implementation of function `efficient_conv_bn_eval_graph_transform_decomposed`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `conv_node`、`bn_weight`、`bn_bias` 等值。这一段延续了函数`efficient_conv_bn_eval_graph_transform_decomposed` 的具体实现。

### Lines 281-300 / 第 281-300 行
````python
        bn_running_mean = bn_node.args[3]
        bn_running_var = bn_node.args[4]
        bn_eps = bn_node.args[7]
        assert len(conv_node.args) >= 2  # type: ignore[union-attr]
        conv_input = conv_node.args[0]  # type: ignore[union-attr]
        conv_weight = conv_node.args[1]  # type: ignore[union-attr]
        conv_bias = conv_node.args[2] if len(conv_node.args) >= 3 else None  # type: ignore[union-attr]
        conv_remaining_args = conv_node.args[3:]  # type: ignore[union-attr]
        args = (
            bn_weight,
            bn_bias,
            bn_running_mean,
            bn_running_var,
            bn_eps,
            conv_node.target,  # type: ignore[union-attr]
            conv_weight,
            conv_bias,
            conv_input,
            conv_remaining_args,
        )
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `bn_running_mean`, `bn_running_var`, `bn_eps`, `conv_input`, `conv_weight`, `conv_bias`, and `...+2`. This range continues the implementation of function `efficient_conv_bn_eval_graph_transform_decomposed`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `bn_running_mean`、`bn_running_var`、`bn_eps`、`conv_input`、`conv_weight`、`conv_bias`、`另有2项` 等值。这一段延续了函数`efficient_conv_bn_eval_graph_transform_decomposed` 的具体实现。

### Lines 301-320 / 第 301-320 行
````python

        # create a new node
        new_node = graph.create_node(
            op="call_function",
            target=efficient_conv_bn_eval_decomposed,
            args=args,  # type: ignore[arg-type]
            name="efficient_conv_bn_eval",
        )

    # this node replaces the original conv + bn, and therefore
    # should replace the uses of bn_node
    bn_node.replace_all_uses_with(new_node)
    # take care of the deletion order:
    # delete bn_node first, and then conv_node
    graph.erase_node(bn_node)
    graph.erase_node(conv_node)  # type: ignore[arg-type]

    return


````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `new_node`, `op`, `target`, `args`, and `name`. This range continues the implementation of function `efficient_conv_bn_eval_graph_transform_decomposed`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `new_node`、`op`、`target`、`args`、`name` 等值。这一段延续了函数`efficient_conv_bn_eval_graph_transform_decomposed` 的具体实现。

### Lines 321-340 / 第 321-340 行
````python
@register_graph_pattern(
    CallModuleVarArgs(
        [
            nn.modules.batchnorm._BatchNorm,
            nn.BatchNorm1d,
            nn.BatchNorm2d,
            nn.BatchNorm3d,
            nn.SyncBatchNorm,
        ],
    ),
    # pyrefly: ignore [bad-argument-type]
    pass_dict=efficient_conv_bn_eval_pass,
    extra_check=lambda match: not inductor_config.freezing
    and inductor_config.efficient_conv_bn_eval_fx_passes,
)
def efficient_conv_bn_eval_graph_transform(match: Match, *args, **kwargs):
    # We matched a BN node
    bn_node = match.nodes[0]
    graph = match.graph
    gm = graph.owning_module
````
- **EN**: Introduces function `efficient_conv_bn_eval_graph_transform`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `pass_dict`, `extra_check`, `bn_node`, `graph`, and `gm`.
- **CN**: 这里定义了函数`efficient_conv_bn_eval_graph_transform`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `pass_dict`、`extra_check`、`bn_node`、`graph`、`gm` 等值。

### Lines 341-360 / 第 341-360 行
````python
    bn_mod = getattr(gm, bn_node.target)  # type: ignore[arg-type]

    # We can only use efficient conv-bn for eval mode with track_running_stats
    if not bn_mod.track_running_stats or bn_mod.training:
        return

    # Check if the input is Conv
    if bn_node.args:
        input_node = bn_node.args[0]
    else:
        input_node = bn_node.kwargs["input"]
    if input_node.op != "call_module":  # type: ignore[union-attr]
        return
    if not hasattr(gm, input_node.target):  # type: ignore[arg-type, union-attr]
        return
    input_mod = getattr(gm, input_node.target)  # type: ignore[arg-type, union-attr]
    supported_convs = [
        nn.Linear,
        nn.Conv1d,
        nn.Conv2d,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `bn_mod`, `input_node`, `else`, `input_mod`, and `supported_convs`. This range continues the implementation of function `efficient_conv_bn_eval_graph_transform`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `bn_mod`、`input_node`、`else`、`input_mod`、`supported_convs` 等值。这一段延续了函数`efficient_conv_bn_eval_graph_transform` 的具体实现。

### Lines 361-380 / 第 361-380 行
````python
        nn.Conv3d,
        nn.ConvTranspose1d,
        nn.ConvTranspose2d,
        nn.ConvTranspose3d,
    ]
    if not any(isinstance(input_mod, cls) for cls in supported_convs):
        return
    conv_node = input_node
    # Output of conv is used by other nodes, cannot optimize
    if len(conv_node.users) > 1:  # type: ignore[union-attr]
        return

    # Find a pair of conv and bn computation nodes to optimize.
    counters["inductor"]["efficient_conv_bn_eval"] += 1

    with graph.inserting_before(conv_node):  # type: ignore[arg-type]
        # create `get_attr` node to access modules
        # note that we directly call `create_node` to fill the `name`
        # argument. `graph.get_attr` and
        # `graph.call_function` does not allow the `name` argument.
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `conv_node`. This range continues the implementation of function `efficient_conv_bn_eval_graph_transform`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `conv_node` 等值。这一段延续了函数`efficient_conv_bn_eval_graph_transform` 的具体实现。

### Lines 381-400 / 第 381-400 行
````python
        conv_get_node = graph.create_node(
            op="get_attr",
            target=conv_node.target,  # type: ignore[union-attr]
            name="get_conv",
        )
        bn_get_node = graph.create_node(
            op="get_attr", target=bn_node.target, name="get_bn"
        )
        if conv_node.args:  # type: ignore[union-attr]
            conv_input = conv_node.args[0]  # type: ignore[union-attr]
        else:
            conv_input = conv_node.kwargs["input"]  # type: ignore[union-attr]
        # prepare args for the fused function
        args = (bn_get_node, conv_get_node, conv_input)
        # create a new node
        new_node = graph.create_node(
            op="call_function",
            target=efficient_conv_bn_eval,
            args=args,
            name="efficient_conv_bn_eval",
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `conv_get_node`, `op`, `target`, `name`, `bn_get_node`, `conv_input`, and `...+3`. This range continues the implementation of function `efficient_conv_bn_eval_graph_transform`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `conv_get_node`、`op`、`target`、`name`、`bn_get_node`、`conv_input`、`另有3项` 等值。这一段延续了函数`efficient_conv_bn_eval_graph_transform` 的具体实现。

### Lines 401-408 / 第 401-408 行
````python
        )
    # this node replaces the original conv + bn, and therefore
    # should replace the uses of bn_node
    bn_node.replace_all_uses_with(new_node)
    # take care of the deletion order:
    # delete bn_node first, and then conv_node
    graph.erase_node(bn_node)
    graph.erase_node(conv_node)  # type: ignore[arg-type]
````
- **EN**: This range continues the implementation of function `efficient_conv_bn_eval_graph_transform`.
- **CN**: 这一段延续了函数`efficient_conv_bn_eval_graph_transform` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Primary functions: `efficient_conv_bn_eval`, `efficient_conv_bn_eval_decomposed`, `efficient_conv_bn_eval_graph_transform_inlined`, `efficient_conv_bn_eval_graph_transform_decomposed`, and `efficient_conv_bn_eval_graph_transform`  
  **CN**: 主要函数：`efficient_conv_bn_eval`、`efficient_conv_bn_eval_decomposed`、`efficient_conv_bn_eval_graph_transform_inlined`、`efficient_conv_bn_eval_graph_transform_decomposed`、`efficient_conv_bn_eval_graph_transform`

## Dependencies / 依赖关系
- **Standard library / 标准库**: None / 无
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.nn`, `torch._dynamo.utils`, `torch._inductor`, `torch.func`, `..pattern_matcher`, `.pre_grad`
