# graph_gradual_typechecker.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/experimental/graph_gradual_typechecker.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Holds experimental tracing, shape reasoning, and transformation infrastructure built on FX.
- **Purpose (CN)**: 存放基于 FX 的实验性跟踪、形状推理与变换基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28
```python
import itertools
import operator
from collections.abc import Callable
from functools import reduce
from typing import Any, TypeVar
from typing_extensions import ParamSpec

import sympy

import torch
from torch.fx.experimental.refinement_types import Equality
from torch.fx.experimental.unification import Var  # type: ignore[attr-defined]
from torch.fx.node import Node, Target
from torch.fx.tensor_type import Dyn, is_consistent, is_more_precise, TensorType
from torch.nn.modules.batchnorm import BatchNorm2d
from torch.nn.modules.conv import Conv2d


_T = TypeVar("_T")
_P = ParamSpec("_P")

_INFERENCE_RULES: dict[Target, Callable[..., Any]] = {}
_REFINEMENT_RULES: dict[Target, Callable[..., Any]] = {}
_RULES: dict[Target, Callable[..., Any]] = {}

__all__ = [
    "GraphTypeChecker",
    "Refine",
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 29-61
```python
    "adaptiveavgpool2d_check",
    "adaptiveavgpool2d_inference_rule",
    "add_inference_rule",
    "all_eq",
    "bn2d_inference_rule",
    "broadcast_types",
    "calculate_out_dimension",
    "conv2d_inference_rule",
    "conv_refinement_rule",
    "conv_rule",
    "element_wise_eq",
    "expand_to_tensor_dim",
    "first_two_eq",
    "flatten_check",
    "flatten_inference_rule",
    "flatten_refinement_rule",
    "get_attr_inference_rule",
    "get_greatest_upper_bound",
    "get_parameter",
    "linear_check",
    "linear_inference_rule",
    "linear_refinement_rule",
    "maxpool2d_check",
    "maxpool2d_inference_rule",
    "register_algebraic_expressions_inference_rule",
    "register_inference_rule",
    "register_refinement_rule",
    "relu_inference_rule",
    "reshape_inference_rule",
    "transpose_inference_rule",
]
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。

### Lines 62-100
```python
# TODO: narrow t to TensorType | _DynType once Node.type is narrowed
def expand_to_tensor_dim(t: Any, n: int) -> TensorType:
    """
    Expand a type to the desired tensor dimension if possible
    Raise an error otherwise.
    - t is the given type
    - n is a number of dimensions to expand to
    """
    if t == Dyn:
        dims = [Dyn] * n
        return TensorType(tuple(dims))
    elif isinstance(t, TensorType):
        if len(t.__args__) != n:
            raise TypeError(
                f"Cannot extend tensor. Tensor {t} has rank {len(t.__args__)}. It should have rank {n}"
            )
        return t
    else:
        raise TypeError(f"Cannot match the type {t}")


def broadcast_types(t1: Any, t2: Any) -> tuple[Any, Any]:
    """
    Applies broadcasting to both given types such that they
    become consistent with each other and returns two new
    resulting types
    """

    # if either type is Dyn, do nothing since the types are already consistent
    if t1 == Dyn or t2 == Dyn or isinstance(t1, Var) or isinstance(t2, Var):
        return t1, t2

    if isinstance(t1, TensorType) and isinstance(t2, TensorType):
        s1 = len(t1.__args__)
        s2 = len(t2.__args__)

        new_t1 = list(t1.__args__)
        new_t2 = list(t2.__args__)
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 101-139
```python
        # We make the types the same length which is the first requirement
        # for consistency
        if s1 > s2:
            for _ in range(s1 - s2):
                new_t2.insert(0, 1)

        elif s2 > s1:
            for _ in range(s2 - s1):
                new_t1.insert(0, 1)

        # we replace occurrences of "1" with each tensor with
        # the corresponding type from the other tensor
        for i, (x, y) in enumerate(zip(new_t1, new_t2)):
            if x == 1:
                new_t1[i] = y
            elif y == 1:
                new_t2[i] = x

        # at this point our tensors should be consistent
        # and we can apply the element-wise operation and find the right dimension
        # for the output of the operation
        (t1, t2) = TensorType(tuple(new_t1)), TensorType(tuple(new_t2))
        return (t1, t2)
    else:
        raise TypeError(f"Cannot broadcast types {t1} and {t2}")


def register_inference_rule(
    call_target: Target,
) -> Callable[[Callable[_P, _T]], Callable[_P, _T]]:
    def register(fn: Callable[_P, _T]) -> Callable[_P, _T]:
        if call_target in _INFERENCE_RULES:
            raise RuntimeError(f"Inference rule already registered for {call_target}!")
        _INFERENCE_RULES[call_target] = fn
        return fn

    return register
```
- **EN**: This module-level block helps validate invariants and surface meaningful failures. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于校验不变量并给出有意义的失败信息。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 140-174
```python
def register_refinement_rule(
    call_target: Target,
) -> Callable[[Callable[_P, _T]], Callable[_P, _T]]:
    def register(fn: Callable[_P, _T]) -> Callable[_P, _T]:
        if call_target in _REFINEMENT_RULES:
            raise RuntimeError(f"Refinement rule already registered for {call_target}!")
        _REFINEMENT_RULES[call_target] = fn
        return fn

    return register


def register_algebraic_expressions_inference_rule(
    call_target: Target,
) -> Callable[[Callable[_P, _T]], Callable[_P, _T]]:
    def register(fn: Callable[_P, _T]) -> Callable[_P, _T]:
        if call_target in _RULES:
            raise RuntimeError(f"Rule already registered for {call_target}!")
        _RULES[call_target] = fn
        return fn

    return register


@register_inference_rule(torch.add)
@register_inference_rule(operator.add)
def add_inference_rule(n: Node) -> Any:
    """
    Apply the addition inference rule. This includes:
    - scalar addition
    - broadcasting semantics

    Note that we always return the least precise type between
    the operands (after applying broadcasting) to be the final type of the operation
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 175-211
```python
    Note that we do not modify the operand types themselves after applying broadcasting
    to them. We only use them to calculate the final type
    """
    if not isinstance(n.args[0], Node):
        raise AssertionError(f"Expected Node, got {type(n.args[0])}")
    if not isinstance(n.args[1], Node):
        raise AssertionError(f"Expected Node, got {type(n.args[1])}")
    t1 = n.args[0].type
    t2 = n.args[1].type

    # handle scalar addition
    if t1 is int and isinstance(t2, TensorType):
        n.type = t2
        return n.type

    # handle scalar addition
    elif t2 is int and isinstance(t1, TensorType):
        n.type = t1
        return n.type

    # we bring the new types to the point where
    # we can check for consistency
    # any inconsistency would not have been caused
    # by broadcasting at this point
    (new_t1, new_t2) = broadcast_types(t1, t2)

    if new_t1 != t1 or new_t2 != t2:
        n.meta["broadcast"] = True
        n.meta[str(n.args[0])] = new_t1
        n.meta[str(n.args[1])] = new_t2

    else:
        n.meta["broadcast"] = False

    new_t1 = t1 if not n.meta["broadcast"] else new_t1
    new_t2 = t2 if not n.meta["broadcast"] else new_t2
```
- **EN**: This block continues `add_inference_rule` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `add_inference_rule`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 212-248
```python
    # we check for consistency between the new types
    if is_consistent(new_t1, new_t2):
        # we return the less precise type because
        # broadcasting may have happened
        # for operands with shape [1,2,Dyn] and [1,2,1]
        # we have to assign the node [1,2,Dyn]
        if is_more_precise(new_t1, new_t2):
            n.type = new_t2
        else:
            n.type = new_t1
        return n.type
    else:
        raise TypeError(
            f"Cannot add arguments {n.args[0]} ({n.args[0].type}) and {n.args[1]} ({n.args[1].type}) in node {n}."
            f" Types should match "
        )


@register_inference_rule(getattr)
def get_attr_inference_rule(n: Node, traced: Any) -> Any:
    """
    The current getattr rule only handles the shape attribute
    Can be extended to other attributes
    The most representative type we have is "Dyn" but the system
    can be extended with more types, such as a type to represent shapes
    """
    attr_name = n.args[1]

    if attr_name == "shape":
        n.type = Dyn
    else:
        raise TypeError("Not yet implemented")

    # TODO. We leave it like this till we add a type to represent tensor sizes
    return n.type
```
- **EN**: These decorators register or transform the following definition so it can track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够跟踪符号形状约束以及依赖形状的推理。

### Lines 249-286
```python
@register_inference_rule(torch.transpose)
def transpose_inference_rule(n: Node) -> Any:
    """
    We check that dimensions for the transpose operations
    are within range of the tensor type of the node
    """
    if n.target is torch.transpose:
        if not isinstance(n.args[0], Node):
            raise AssertionError(f"Expected Node, got {type(n.args[0])}")
        t = n.args[0].type

        if not isinstance(n.args[1], int):
            raise AssertionError(f"Expected int, got {type(n.args[1])}")
        if not isinstance(n.args[2], int):
            raise AssertionError(f"Expected int, got {type(n.args[2])}")
        dim1, dim2 = n.args[1], n.args[2]

        if t == Dyn:
            n.type = Dyn
            return n.type

        elif isinstance(t, TensorType):
            if 0 <= dim1 < len(t.__args__) and 0 <= dim2 < len(t.__args__):
                new_type = list(t.__args__)
                new_type[dim1], new_type[dim2] = new_type[dim2], new_type[dim1]
                final = TensorType(new_type)
                n.type = get_greatest_upper_bound(n.type, final)
                return n.type
            else:
                raise TypeError(
                    f"Cannot transpose {dim1} and {dim2} in type {t} for node {n}"
                )
        else:
            raise TypeError(
                f"Cannot transpose {dim1} and {dim2} in type {t} for node {n}"
            )
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 287-328
```python
@register_inference_rule(torch.reshape)
def reshape_inference_rule(n: Node) -> TensorType:
    """
    Without dynamism, the rule checks that the
    product of the elements of the argument tensor
    type is equal to the product of the elements
    of the required shape. We gradualize this rule
    by adding a case to handle fully dynamic input
    as well as input where some of the tensor dimensions
    are unknown. In this case we check for divisibility
    """
    if not isinstance(n.args[0], Node):
        raise AssertionError(f"Expected Node, got {type(n.args[0])}")
    t1 = n.args[0].type

    if not isinstance(n.args[1], list):
        raise AssertionError(f"Expected list, got {type(n.args[1])}")
    t2 = n.args[1]
    t2_type = TensorType([Dyn if elem == -1 else elem for elem in t2])

    # if we do not know the original tensor dimension,
    # we return the required dimension
    if t1 == Dyn:
        n.type = t2_type
        return t2_type

    # if any of the dimensions are unknown,
    # we check for divisibility
    elif isinstance(t1, TensorType):
        if not isinstance(t1, TensorType):
            raise AssertionError(f"Expected TensorType, got {type(t1)}")
        a = [e if e != Dyn else 1 for e in t1.__args__]
        p1 = reduce(operator.mul, a)
        p2 = reduce(operator.mul, t2)
        if p1 % p2 == 0 or p2 % p1 == 0:
            n.type = t2_type
            return t2_type
        else:
            raise TypeError(f"Cannot reshape in node {n} from {t1} to {t2_type}")
    else:
        raise TypeError(f"Cannot reshape in node {n} from {t1} to {t2_type}")
```
- **EN**: These decorators register or transform the following definition so it can track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够跟踪符号形状约束以及依赖形状的推理。

### Lines 329-366
```python

@register_inference_rule(BatchNorm2d)
def bn2d_inference_rule(n: Node, module_instance: Any) -> Any:
    """
    Given a BatchNorm2D instance and a node check the following conditions:
    - the input type can be expanded to a size 4 tensor: t =  (x_1, x_2, x_3, x_4)
    - the current node type can be expanded to a size 4 tensor: t' =  (x_1', x_2', x_3', x_4')
    - t is consistent with t'
    - x_2 is consistent with the module's num_features
    - x_2' is consistent with the module's num_features
    output type: the more precise type of t and t'
    """
    if not isinstance(n.args[0], Node):
        raise AssertionError(f"Expected Node, got {type(n.args[0])}")
    n.args[0].type = expand_to_tensor_dim(n.args[0].type, 4)
    arg_type = n.args[0].type
    n.type = expand_to_tensor_dim(n.type, 4)

    # we check the conditions on the incoming argument
    # and any existing annotation
    # we also check for consistency between both annotations
    if (
        is_consistent(arg_type.__args__[1], module_instance.num_features)
        and is_consistent(n.type.__args__[1], module_instance.num_features)
        and is_consistent(arg_type, n.type)
    ):
        # we choose the more precise type
        # to be the node type
        # so if an incoming argument has more type information
        # we set this node's type to be the argument type
        n.type = get_greatest_upper_bound(arg_type, n.type)
        return n.type
    else:
        raise TypeError(
            f"Cannot apply {module_instance} with input type {arg_type} and existing type {n.type} on {n}"
        )
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 367-407
```python
def calculate_out_dimension(d_in: Any, module_instance: Any, index: int) -> Any:
    """
    For calculating h_in and w_out according to the conv2D documentation
    """
    padding = (
        (module_instance.padding, module_instance.padding)
        if isinstance(module_instance.padding, int)
        else module_instance.padding
    )
    kernel_size = (
        (module_instance.kernel_size, module_instance.kernel_size)
        if isinstance(module_instance.kernel_size, int)
        else module_instance.kernel_size
    )
    stride = (
        (module_instance.stride, module_instance.stride)
        if isinstance(module_instance.stride, int)
        else module_instance.stride
    )
    dilation = (
        (module_instance.dilation, module_instance.dilation)
        if isinstance(module_instance.dilation, int)
        else module_instance.dilation
    )

    DIMENSION_TYPES = (int, sympy.Symbol)

    if d_in == Dyn:
        return Dyn

    elif isinstance(d_in, DIMENSION_TYPES):
        n = d_in + 2 * padding[index] - dilation[index] * (kernel_size[index] - 1) - 1

        return (n // stride[0]) + 1

    else:
        raise TypeError(
            f"{d_in} in {module_instance} must be a number or Dyn. Received {type(d_in)}"
        )
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 408-442
```python
# TODO: narrow params/return to TensorType | _DynType once Node.type is narrowed
def get_greatest_upper_bound(type1: Any, type2: Any) -> Any:
    """
    Get the most precise type that's consistent with the given types
    """
    if type1 == Dyn:
        return type2
    elif type2 == Dyn:
        return type1
    elif isinstance(type1, TensorType) and isinstance(type2, TensorType):
        if not is_consistent(type1, type2):
            raise TypeError(f"Inconsistent types {type1}, {type2}")
        gub = [
            t1 if is_more_precise(t1, t2) else t2
            for (t1, t2) in zip(type1.__args__, type2.__args__)
        ]
        return TensorType(tuple(gub))


@register_inference_rule(Conv2d)
def conv2d_inference_rule(n: Node, module_instance: Any) -> Any:
    """
    Given a Conv2D instance and a node check the following conditions:
    - the input type can be expanded to a size 4 tensor: t =  (x_1, x_2, H, W)
    - the current node type can be expanded to a size 4 tensor: t' =  (x_1', x_2', x_3', x_4')
    - x_2 is consistent with the module's in_channels
    - let o = (x_1, out_channels, H_out, W_out)
    then the output is the greatest upper bound of o and the existing node type t'.
    """
    if not isinstance(n.args[0], Node):
        raise AssertionError(f"Expected Node, got {type(n.args[0])}")
    n.args[0].type = expand_to_tensor_dim(n.args[0].type, 4)
    arg_type = n.args[0].type
    curr_node_type = expand_to_tensor_dim(n.type, 4)
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 443-475
```python
    if is_consistent(arg_type.__args__[1], module_instance.in_channels):
        w_in = arg_type.__args__[3]
        h_in = arg_type.__args__[2]
        h_out = calculate_out_dimension(h_in, module_instance, 0)
        w_out = calculate_out_dimension(w_in, module_instance, 1)
        new_type = TensorType(
            (arg_type.__args__[0], module_instance.out_channels, h_out, w_out)
        )
        gub = get_greatest_upper_bound(new_type, curr_node_type)
        n.type = gub
        return n.type
    else:
        raise TypeError(
            f"Cannot apply {module_instance} with input type {arg_type} and existing type {n.type} on {n}"
        )


@register_inference_rule(torch.nn.ReLU)
def relu_inference_rule(n: Node, module_instance: Any) -> Any:
    """
    Input and output shapes should be equal.
    """
    if not isinstance(n.args[0], Node):
        raise AssertionError(f"Expected Node, got {type(n.args[0])}")

    if n.args[0].type == Dyn and isinstance(n.type, TensorType):
        n.args[0].type = expand_to_tensor_dim(n.args[0].type, len(n.type.__args__))

    if isinstance(n.args[0].type, TensorType):
        n.type = get_greatest_upper_bound(n.args[0].type, n.type)
    return n.type
```
- **EN**: These decorators register or transform the following definition so it can track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够跟踪符号形状约束以及依赖形状的推理。

### Lines 476-517
```python
def maxpool2d_check(typ: Any, module_instance: Any) -> TensorType:
    """
    Applies the maxpool2d shape information to the input
    this affects the last two dimensions
    """
    new_type_list = list(typ.__args__)
    if len(new_type_list) == 4 or len(new_type_list) == 3:
        w_in = new_type_list[-1]
        h_in = new_type_list[-2]

        h_out = calculate_out_dimension(h_in, module_instance, 0)
        w_out = calculate_out_dimension(w_in, module_instance, 1)

        new_type_list[-1] = w_out
        new_type_list[-2] = h_out
        return TensorType(tuple(new_type_list))

    else:
        raise TypeError(f"Wrong size {typ} for {module_instance}")


@register_inference_rule(torch.nn.MaxPool2d)
def maxpool2d_inference_rule(n: Node, module_instance: Any) -> Any:
    """
    Given a MaxPool2D instance and a node check the following conditions:
    - Input size matches size 3 or 4
    - Current node type is consistent with the output type we will calculate
    - Input size matches output size and the last two dimensions of the output
      are w_out and h_out. The remaining dimensions are the same as the input
    - Our final result is the greatest upper bound of the output we calculate
      and the current node type.
    """
    if not isinstance(n.args[0], Node):
        raise AssertionError(f"Expected Node, got {type(n.args[0])}")

    if n.args[0].type == Dyn and isinstance(n.type, TensorType):
        n.args[0].type = expand_to_tensor_dim(n.args[0].type, len(n.type.__args__))
    if isinstance(n.args[0].type, TensorType):
        output = maxpool2d_check(n.args[0].type, module_instance)
        n.type = get_greatest_upper_bound(output, n.type)
    return n.type
```
- **EN**: These decorators register or transform the following definition so it can track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够跟踪符号形状约束以及依赖形状的推理。

### Lines 518-552
```python

def linear_check(tensor_type: Any, module_instance: Any) -> TensorType:
    """
    Checks that an input tensor type satisfies the conditions for linear operation
    and returns the output type based on in and out features given by module_instance
    """
    if len(tensor_type.__args__) >= 2:
        if is_consistent(module_instance.in_features, tensor_type.__args__[-1]):
            new_type_args = list(tensor_type.__args__)
            new_type_args[-1] = module_instance.out_features
            return TensorType(tuple(new_type_args))
        else:
            raise TypeError(
                f"Inconsistent {module_instance.in_features} and {tensor_type.__args__[-1]} in {module_instance}"
            )
    else:
        raise TypeError(f"Type {tensor_type} must have rank 2 or more.")


@register_inference_rule(torch.nn.Linear)
def linear_inference_rule(n: Node, module_instance: Any) -> Any:
    """
    Applies the shape information to the input then gets the greatest upper bound
    of the resulting type and the existing type
    """
    if not isinstance(n.args[0], Node):
        raise AssertionError(f"Expected Node, got {type(n.args[0])}")
    if n.args[0].type == Dyn and isinstance(n.type, TensorType):
        n.args[0].type = expand_to_tensor_dim(n.args[0].type, len(n.type.__args__))
    if isinstance(n.args[0].type, TensorType):
        output_type = linear_check(n.args[0].type, module_instance)
        n.type = get_greatest_upper_bound(output_type, n.type)
    return n.type
```
- **EN**: These decorators register or transform the following definition so it can track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够跟踪符号形状约束以及依赖形状的推理。

### Lines 553-594
```python
def adaptiveavgpool2d_check(tensor_type: Any, module_instance: Any) -> TensorType:
    output_size = module_instance.output_size
    if isinstance(output_size, int):
        output_size = [output_size, output_size]
    elif isinstance(output_size, tuple):
        output_size = list(output_size)
        if output_size[0] is None:
            output_size[0] = output_size[1]
        if output_size[1] is None:
            output_size[1] = output_size[0]

    new_type_list = list(tensor_type.__args__)

    if len(tensor_type.__args__) == 4 or len(tensor_type.__args__) == 3:
        new_type_list[-1] = output_size[1]
        new_type_list[-2] = output_size[0]

        return TensorType(tuple(new_type_list))

    else:
        raise TypeError(f"Tensor ranks must be 3 or 4. Got {tensor_type}")


@register_inference_rule(torch.nn.AdaptiveAvgPool2d)
def adaptiveavgpool2d_inference_rule(n: Node, module_instance: Any) -> Any:
    """
    The input and output sizes should be the same except for the last
    two dimensions taken from the input, which represent width and height
    """
    if not isinstance(n.args[0], Node):
        raise AssertionError(f"Expected Node, got {type(n.args[0])}")
    if n.args[0].type == Dyn and isinstance(n.type, TensorType):
        n.args[0].type = expand_to_tensor_dim(n.args[0].type, len(n.type.__args__))
    if isinstance(n.args[0].type, TensorType):
        output_type = adaptiveavgpool2d_check(n.args[0].type, module_instance)
        n.type = get_greatest_upper_bound(n.type, output_type)
    return n.type


def flatten_check(tensor_type: Any, start_dim: int, end_dim: int) -> TensorType:
    l = len(tensor_type.__args__)
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 595-632
```python
    start_dim = l if start_dim == -1 else abs(start_dim)
    end_dim = l + end_dim + 1 if end_dim < 0 else end_dim + 1

    if 0 <= start_dim <= (l - 1) and 0 <= end_dim <= l and start_dim < end_dim:
        my_args = list(tensor_type.__args__)
        lhs = my_args[0:start_dim]
        rhs = my_args[end_dim:]
        mid = my_args[start_dim:end_dim]
        if Dyn in mid:
            mid = [Dyn]
        else:
            mid = [reduce(operator.mul, my_args[start_dim:end_dim])]
        new_type_list = lhs + mid + rhs
        return TensorType(tuple(new_type_list))
    else:
        raise TypeError(
            f"Incompatible dimensions {start_dim}, {end_dim - 1} in type {tensor_type}"
        )


@register_inference_rule(torch.flatten)
def flatten_inference_rule(n: Node) -> Any:
    """
    Applies the flatten shape information to the input then gets the
    greatest upper bound of the resulting type and the existing type
    """
    if not isinstance(n.args[0], Node):
        raise AssertionError(f"Expected Node, got {type(n.args[0])}")

    # set the default start and end dims
    start_dim = 1
    end_dim = -1

    if len(n.args) > 1:
        if not isinstance(n.args[1], int):
            raise AssertionError(f"Expected int, got {type(n.args[1])}")
        start_dim = n.args[1]
```
- **EN**: These decorators register or transform the following definition so it can track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够跟踪符号形状约束以及依赖形状的推理。

### Lines 633-666
```python
    if len(n.args) > 2:
        if not isinstance(n.args[2], int):
            raise AssertionError(f"Expected int, got {type(n.args[2])}")
        end_dim = n.args[2]

    if n.args[0].type == Dyn and isinstance(n.type, TensorType):
        n.args[0].type = expand_to_tensor_dim(n.args[0].type, len(n.type.__args__))

    if isinstance(n.args[0].type, TensorType):
        output_type = flatten_check(n.args[0].type, start_dim, end_dim)
        n.type = get_greatest_upper_bound(output_type, n.type)

    return n.type


class GraphTypeChecker:
    def __init__(self, env: dict[str, Any], traced: torch.fx.GraphModule) -> None:
        self.env = env
        self.traced = traced

    def type_check(self) -> bool:
        """
        A gradual type checker for graphs
        Effect: every node's field type will be
        populated with a type after type-checking is done
        """
        graph = self.traced.graph

        # type check every node with gradual type rules
        # if any node does not type check return false
        for n in graph.nodes:
            self.type_check_node(n)
        return True
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 667-706
```python
    def type_check_node(self, n: Node) -> Any:
        """
        Type check a given fx node.
        Current operations:
        - Reshape
        - Transpose
        - Add
        - Relu
        - conv2d
        - batchnorm2d
        - flatten
        - maxpool2d
        - adaptiveavgpool2d
        - linear
        """
        if n.type is None:
            n.type = Dyn

        if n.op == "placeholder":
            return n.type

        elif n.op == "get_attr":
            t = get_parameter(self.traced, n.target)  # type: ignore[arg-type]
            if isinstance(t.data, torch.Tensor):
                n.type = TensorType(t.data.shape)
            return n.type

        elif n.op == "call_function":
            if n.target is getattr:
                if getattr not in _INFERENCE_RULES:
                    raise AssertionError("getattr not in _INFERENCE_RULES")
                return _INFERENCE_RULES[n.target](n, self.traced)

            elif n.target in _INFERENCE_RULES:
                return _INFERENCE_RULES[n.target](n)
            else:
                raise RuntimeError(
                    f"No inference rule registered for target {n.target}!"
                )
```
- **EN**: Defines the `GraphTypeChecker.type_check_node` method; this block introduces logic that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 定义`GraphTypeChecker.type_check_node` 方法；该代码块引入了用于跟踪符号形状约束以及依赖形状的推理的逻辑。

### Lines 707-743
```python
        elif n.op == "call_module":
            # pyrefly: ignore[bad-argument-type]
            module_instance = self.traced.get_submodule(n.target)
            if type(module_instance) in _INFERENCE_RULES:
                return _INFERENCE_RULES[type(module_instance)](n, module_instance)
            else:
                raise RuntimeError(
                    f"No inference rule registered for class {type(module_instance)}!"
                )

        elif n.op == "output":

            def get_node_type(a: Any) -> Any:
                return a.type

            n.type = torch.fx.node.map_arg(n.args[0], get_node_type)
            return n.type

        else:
            raise NotImplementedError(f"Method {n.op} not yet implemented")


@register_refinement_rule(Conv2d)
def conv_refinement_rule(n: Node) -> list[Any] | None:
    """
    The equality constraints are between the first dimension of
    the input and output
    """
    res: list[Any] = []
    if not isinstance(n.args[0], Node):
        raise AssertionError(f"Expected Node, got {type(n.args[0])}")
    arg_type = n.args[0].type
    if isinstance(arg_type, TensorType) and isinstance(n.type, TensorType):
        res = [Equality(arg_type.__args__[0], n.type.__args__[0])]
        return res
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 744-775
```python
@register_refinement_rule(torch.nn.Linear)
def linear_refinement_rule(n: Node) -> list[Any]:
    """
    The equality constraints are between the first dimension of
    the input and output
    """
    res: list[Any] = []
    if not isinstance(n.args[0], Node):
        raise AssertionError(f"Expected Node, got {type(n.args[0])}")
    arg_type = n.args[0].type
    if isinstance(arg_type, TensorType) and isinstance(n.type, TensorType):
        res = [Equality(arg_type.__args__[0], n.type.__args__[0])]
    return res


@register_refinement_rule(BatchNorm2d)
@register_refinement_rule(torch.nn.ReLU)
def all_eq(n: Node) -> list[Any]:
    """
    For operations where the input shape is equal to the output shape
    """
    res: list[Any] = []
    if not isinstance(n.args[0], Node):
        raise AssertionError(f"Expected Node, got {type(n.args[0])}")
    arg_type = n.args[0].type
    if isinstance(arg_type, TensorType) and isinstance(n.type, TensorType):
        args1 = arg_type.__args__
        args2 = n.type.__args__
        res = [Equality(args1[i], args2[i]) for i in range(len(args1))]
    return res
```
- **EN**: These decorators register or transform the following definition so it can track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够跟踪符号形状约束以及依赖形状的推理。

### Lines 776-805
```python
@register_refinement_rule(torch.nn.AdaptiveAvgPool2d)
@register_refinement_rule(torch.nn.MaxPool2d)
def first_two_eq(n: Node) -> list[Any]:
    """
    For operations where the first two dimensions of the input and output shape
    are equal
    """
    res: list[Any] = []
    if not isinstance(n.args[0], Node):
        raise AssertionError(f"Expected Node, got {type(n.args[0])}")
    arg_type = n.args[0].type
    if isinstance(arg_type, TensorType) and isinstance(n.type, TensorType):
        args1 = arg_type.__args__
        args2 = n.type.__args__
        res = [Equality(args1[0], args2[0]), Equality(args1[1], args2[1])]
    return res


@register_refinement_rule(torch.add)
@register_refinement_rule(operator.add)
def element_wise_eq(n: Node) -> list[Any]:
    """
    For element-wise operations and handles broadcasting.
    Note that after applying broadcasting to the arguments
    we are able to determine if certain dimensions have not been broadcast
    if they are symbolicallu equal.

    in this case, we can establish equality between those dimensions and the
    corresponding output dimensions.
```
- **EN**: These decorators register or transform the following definition so it can track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够跟踪符号形状约束以及依赖形状的推理。

### Lines 806-846
```python
    Note that it takes two iterations for this result. One iteration to establish
    equality between certain dimensions of the operands (requiring the whole solver
    including unification) and another iteration to establish equality between the operands
    and the resulting type, requiring another round of constraint generation and unificaiton.
    """
    res: list[Any] = []
    if isinstance(n.args[0], Node) and isinstance(n.args[1], Node):
        arg_type1 = n.args[0].type
        arg_type2 = n.args[1].type
        if (
            isinstance(arg_type1, TensorType)
            and isinstance(arg_type2, TensorType)
            and isinstance(n.type, TensorType)
        ):
            args1, args2 = broadcast_types(arg_type1, arg_type2)
            # by this point, we know that args1 and args2 are the same size.
            a1 = args1.__args__
            a2 = args2.__args__
            a3 = n.type.__args__

            # we would be here in the second iteration where we establish equality
            # between operand type dimensions and the resulting type dimensions
            r = []
            for x, y, z in zip(a1, a2, a3):
                if x == y:
                    r.append(Equality(x, z))
            res = r
    return res


@register_refinement_rule(torch.flatten)
def flatten_refinement_rule(n: Node) -> list[Any]:
    """
    Generates equality constraints between the dimensions of the input and output
    that will not be involved in the flatten operation
    """
    if not isinstance(n.args[0], Node):
        raise AssertionError(f"Expected Node, got {type(n.args[0])}")

    eq_const = []
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 847-874
```python
    start_dim = 1
    end_dim = -1

    if len(n.args) > 1:
        if not isinstance(n.args[1], int):
            raise AssertionError(f"Expected int, got {type(n.args[1])}")
        start_dim = n.args[1]

    if len(n.args) > 2:
        if not isinstance(n.args[2], int):
            raise AssertionError(f"Expected int, got {type(n.args[2])}")
        end_dim = n.args[2]

    if isinstance(n.type, TensorType) and isinstance(n.args[0].type, TensorType):
        l = len(n.type.__args__)
        arg_type = n.args[0].type
        start_dim = l if start_dim == -1 else start_dim
        end_dim = l + end_dim + 1 if end_dim < 0 else end_dim + 1

        for t1, t2 in zip(n.type.__args__[0:start_dim], arg_type.__args__[0:start_dim]):
            eq_const.append(Equality(t1, t2))

        for t1, t2 in zip(n.type.__args__[end_dim:], arg_type.__args__[end_dim:]):
            eq_const.append(Equality(t1, t2))
    return eq_const


@register_algebraic_expressions_inference_rule(Conv2d)
```
- **EN**: These decorators register or transform the following definition so it can validate invariants and surface meaningful failures.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够校验不变量并给出有意义的失败信息。

### Lines 875-915
```python
def conv_rule(n: Node, module_instance: Any) -> TensorType | None:
    """
    Represents the output in terms of an algrbraic expression w.r.t
    the input when possible
    """
    if not isinstance(n.args[0], Node):
        raise AssertionError(f"Expected Node, got {type(n.args[0])}")
    arg_type = n.args[0].type
    if isinstance(arg_type, TensorType) and isinstance(n.type, TensorType):
        w_in = arg_type.__args__[3]
        h_in = arg_type.__args__[2]
        h_out = calculate_out_dimension(h_in, module_instance, 0)
        w_out = calculate_out_dimension(w_in, module_instance, 1)
        new_type = TensorType((n.type.__args__[0], n.type.__args__[1], h_out, w_out))
        n.type = new_type
        return new_type


class Refine:
    """
    Symbolic shape inference.
    Generates constraints over type variables.
    Currently all constraints are equality constraints.
    """

    def __init__(self, traced: Any) -> None:
        self.constraints = []
        self.traced = traced
        self.symbol_iter = itertools.count(start=0, step=1)

    def refine(self) -> bool:
        """
        Generates constraints for
        every node in the graph based on
        the operation.
        """
        graph = self.traced.graph
        for n in graph.nodes:
            self.refine_node(n)
        return True
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 916-957
```python
    def symbolic_relations(self) -> bool:
        """
        Infers algebraic relations
        """
        graph = self.traced.graph
        for n in graph.nodes:
            self.infer_symbolic_relations(n)
        return True

    def replace_dyn_with_fresh_var(self, typ: Any) -> Any:
        """
        Replace all unknown types with fresh type variables.
        """
        if typ == Dyn:
            new_symbol = Var(next(self.symbol_iter))
            return new_symbol
        elif isinstance(typ, TensorType):
            new_args = [self.replace_dyn_with_fresh_var(a) for a in typ.__args__]
            return TensorType(tuple(new_args))
        elif isinstance(typ, list):
            return [self.replace_dyn_with_fresh_var(t) for t in typ]
        elif isinstance(typ, tuple):
            return (self.replace_dyn_with_fresh_var(t) for t in typ)
        else:
            return typ

    def convert_to_sympy_symbols(self, typ: Any) -> Any:
        """
        Replace all unknown types with fresh type variables.
        """
        if isinstance(typ, Var):
            return sympy.symbols(str(typ))
        elif isinstance(typ, TensorType):
            new_args = [self.convert_to_sympy_symbols(a) for a in typ.__args__]
            return TensorType(tuple(new_args))
        elif isinstance(typ, list):
            return [self.convert_to_sympy_symbols(t) for t in typ]
        elif isinstance(typ, tuple):
            return (self.convert_to_sympy_symbols(t) for t in typ)
        else:
            return typ
```
- **EN**: Declares `Refine`; this class packages state and methods that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 声明 `Refine`；该类封装了状态与方法，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 958-998
```python
    def refine_node(self, n: Node) -> Any:
        """
        Returns a list of equality constraints for
        call_module and call_function nodes.
        Models the relation between input and output dimensions
        using constraints in case they are both tensors.
        All operations used in resnet50 are defined.
        """
        if n.type is None:
            n.type = Dyn

        n.type = self.replace_dyn_with_fresh_var(n.type)

        if n.op == "call_function":
            if n.target in _REFINEMENT_RULES:
                self.constraints += _REFINEMENT_RULES[n.target](n)

        if n.op == "call_module":
            module_instance = self.traced.get_submodule(n.target)
            if type(module_instance) in _REFINEMENT_RULES:
                self.constraints += _REFINEMENT_RULES[type(module_instance)](n)

        if n.op == "output":

            def get_node_type(a: Any) -> Any:
                return a.type

            n.type = torch.fx.node.map_arg(n.args[0], get_node_type)
            return n.type

    def infer_symbolic_relations(self, n: Node) -> Any:
        n.type = self.convert_to_sympy_symbols(n.type)
        if n.op == "call_function":
            if n.target in _RULES:
                return _RULES[n.target](n)

        if n.op == "call_module":
            module_instance = self.traced.get_submodule(n.target)
            if type(module_instance) in _RULES:
                return _RULES[type(module_instance)](n, module_instance)
```
- **EN**: Declares `Refine`; this class packages state and methods that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 声明 `Refine`；该类封装了状态与方法，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 999-1038
```python
        if n.op == "output":

            def get_node_type(a: Any) -> Any:
                return a.type

            n.type = torch.fx.node.map_arg(n.args[0], get_node_type)
            return n.type


def get_parameter(traced: Any, target: str) -> torch.nn.Parameter:
    """
    Returns the parameter given by ``target`` if it exists,
    otherwise throws an error.

    See the docstring for ``get_submodule`` for a more detailed
    explanation of this method's functionality as well as how to
    correctly specify ``target``.

    Args:
        target: The fully-qualified string name of the Parameter
            to look for. (See ``get_submodule`` for how to specify a
            fully-qualified string.)

    Returns:
        torch.nn.Parameter: The Parameter referenced by ``target``

    Raises:
        AttributeError: If the target string references an invalid
            path or resolves to something that is not an
            ``nn.Parameter``
    """
    module_path, _, param_name = target.rpartition(".")

    mod: torch.nn.Module = traced.get_submodule(module_path)

    if not hasattr(mod, param_name):
        raise AttributeError(mod._get_name() + " has no attribute `" + param_name + "`")

    param: torch.nn.Parameter = getattr(mod, param_name)
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 1039-1039
```python
    return param
```
- **EN**: This block continues `get_parameter` and works to manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `get_parameter`，用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Tracing / 跟踪**
  - EN: The file captures Python-level operations and turns them into analyzable graph structures.
  - CN: 该文件会捕获 Python 层操作，并将其转换为可分析的图结构。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.fx.experimental.refinement_types`, `torch.fx.experimental.unification`, `torch.fx.node`, `torch.fx.tensor_type`, `torch.nn.modules.batchnorm`, `torch.nn.modules.conv`
- **Standard library / 标准库**: `itertools`, `operator`, `collections.abc`, `functools`, `typing`, `typing_extensions`
- **Third-party packages / 第三方包**: `sympy`
- **Primary symbols / 核心符号**: `_T`, `_P`, `_INFERENCE_RULES`, `_REFINEMENT_RULES`, `_RULES`, `__all__`, `expand_to_tensor_dim`, `broadcast_types`, `register_inference_rule`, `register_refinement_rule`, `register_algebraic_expressions_inference_rule`, `add_inference_rule`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
