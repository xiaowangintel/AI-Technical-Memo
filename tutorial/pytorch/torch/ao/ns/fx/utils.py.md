# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/ns/fx/utils.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements numeric-suite tooling used to compare floating-point and quantized model behavior. This specific file centers on `utils.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior. It also participates in graph/module transformation before or after quantized execution.
- **Purpose (CN) / 用途（中文）**: 实现数值套件工具，用于比较浮点模型与量化模型的行为。 该文件具体围绕 `utils.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。 它也参与量化执行前后的图/模块转换。
## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行
```python
# mypy: allow-untyped-decorators
# mypy: allow-untyped-defs
import enum
import operator
from collections.abc import Callable

import torch
import torch.ao.nn.intrinsic.quantized as nniq
import torch.ao.nn.quantized as nnq
import torch.nn as nn
from torch.ao.quantization import FakeQuantizeBase, ObserverBase
from torch.ao.quantization.observer import _is_activation_post_process
from torch.ao.quantization.utils import getattr_from_fqn
from torch.fx import GraphModule
from torch.fx.graph import Node

from .ns_types import NSNodeTargetType, NSResultsType


toq = torch.ops.quantized
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch, torch.ao.nn.intrinsic.quantized, torch.ao.nn.quantized, torch.nn; standard-library helpers such as enum, operator, collections.abc:Callable. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch, torch.ao.nn.intrinsic.quantized, torch.ao.nn.quantized, torch.nn；标准库辅助模块，如 enum, operator, collections.abc:Callable。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 23-52 / 第 23-52 行
```python
# TODO(future PR): consider deleting this enum and using the torch types
# directly.  This might be tricky because it is not a one to one mapping.
class NodeInputOrOutputType(enum.Enum):
    FP32 = enum.auto()  # torch.float
    INT8 = enum.auto()  # torch.qint8 or torch.quint8
    FP16 = enum.auto()  # torch.float16
    UNKNOWN = enum.auto()  # we cannot determine input/output dtype
    # TODO(future PR): while these functions can support multiple dtypes,
    #   for the purposes of numerical debugging we want to get the actual
    #   dtype used in the model. We will likely need some kind of dtype
    #   propagation to estimate this.
    FP32_OR_INT8 = enum.auto()  # either torch.float or torch.quint8 or torch.qint8
    # TODO(future PRs): dynamic quant, fake quant, etc


def get_node_first_input_and_output_type(
    node: Node,
    gm: GraphModule,
    logger_cls: Callable,
    node_type_to_io_type_map: dict[str, set[NSNodeTargetType]],
) -> tuple[NodeInputOrOutputType, NodeInputOrOutputType]:
    # TODO(future PR): clean this up
    FUNS_IO_TYPE_FP32 = node_type_to_io_type_map["funs_io_type_fp32"]
    FUNS_IO_TYPE_FP16 = node_type_to_io_type_map["funs_io_type_fp16"]
    FUNS_IO_TYPE_INT8 = node_type_to_io_type_map["funs_io_type_int8"]
    FUNS_IO_TYPE_FP32_OR_INT8 = node_type_to_io_type_map["funs_io_type_fp32_or_int8"]
    MODS_IO_TYPE_FP32 = node_type_to_io_type_map["mods_io_type_fp32"]
    MODS_IO_TYPE_INT8 = node_type_to_io_type_map["mods_io_type_int8"]
    MODS_IO_TYPE_FP32_OR_INT8 = node_type_to_io_type_map["mods_io_type_fp32_or_int8"]
    METHS_IO_TYPE_FP32_OR_INT8 = node_type_to_io_type_map["meths_io_type_fp32_or_int8"]
```
- **EN**: It introduces or extends class-level abstractions such as `NodeInputOrOutputType`, which organize state and behavior for this subsystem. Key callable entry points in this range include `get_node_first_input_and_output_type`. They package a focused unit of behavior behind named helpers or APIs. Named constants such as `FP32`, `INT8`, `FP16`, `UNKNOWN` centralize shared configuration or sentinel values. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `NodeInputOrOutputType` 等类级抽象，用于组织该子系统的状态与行为。 这一段的重要可调用入口包括 `get_node_first_input_and_output_type`，它们把聚焦的行为封装成具名辅助函数或 API。 `FP32, INT8, FP16, UNKNOWN` 等具名常量把共享配置或哨兵值集中定义在一起。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 54-79 / 第 54-79 行
```python
    if node.op == "call_function":
        if node.target in FUNS_IO_TYPE_FP32:
            return (NodeInputOrOutputType.FP32, NodeInputOrOutputType.FP32)
        if node.target in FUNS_IO_TYPE_FP16:
            return (NodeInputOrOutputType.FP16, NodeInputOrOutputType.FP16)
        elif node.target in FUNS_IO_TYPE_INT8:
            return (NodeInputOrOutputType.INT8, NodeInputOrOutputType.INT8)
        elif node.target in FUNS_IO_TYPE_FP32_OR_INT8:
            first_arg = get_normalized_nth_input(node, gm, 0)
            if not isinstance(first_arg, Node):
                raise AssertionError(f"Expected Node, got {type(first_arg)}")
            (
                _prev_node_input_type,
                prev_node_output_type,
            ) = get_node_first_input_and_output_type(
                first_arg, gm, logger_cls, node_type_to_io_type_map
            )
            return (prev_node_output_type, prev_node_output_type)
        else:
            return (NodeInputOrOutputType.UNKNOWN, NodeInputOrOutputType.UNKNOWN)

    elif node.op == "call_module":
        if node.op != "call_module":
            raise AssertionError(f"Expected call_module, got '{node.op}'")
        if not isinstance(node.target, str):
            raise AssertionError(f"Expected str, but got {type(node.target)}")
```
- **EN**: Key callable entry points in this range include `get_node_first_input_and_output_type`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `get_node_first_input_and_output_type`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 81-112 / 第 81-112 行
```python
        mod = getattr_from_fqn(gm, node.target)
        is_known_fp32_or_int8_input_module = any(
            isinstance(mod, target_type)  # type: ignore[arg-type]
            for target_type in MODS_IO_TYPE_FP32_OR_INT8
        )
        if (
            isinstance(mod, (logger_cls, ObserverBase, FakeQuantizeBase))  # type: ignore[arg-type]
            or is_known_fp32_or_int8_input_module
        ):
            # A logger or observer's input and output type is the output
            # type of the preceding node.
            first_arg = get_normalized_nth_input(node, gm, 0)
            if not isinstance(first_arg, Node):
                raise AssertionError(f"Expected Node, got {type(first_arg)}")
            (
                _prev_node_input_type,
                prev_node_output_type,
            ) = get_node_first_input_and_output_type(
                first_arg, gm, logger_cls, node_type_to_io_type_map
            )
            return (prev_node_output_type, prev_node_output_type)
        is_known_fp32_input_module = any(
            isinstance(mod, target_type)  # type: ignore[arg-type]
            for target_type in MODS_IO_TYPE_FP32
        )
        is_known_int8_input_module = any(
            isinstance(mod, target_type)  # type: ignore[arg-type]
            for target_type in MODS_IO_TYPE_INT8
        )
        if is_known_fp32_input_module:
            return (NodeInputOrOutputType.FP32, NodeInputOrOutputType.FP32)
        elif is_known_int8_input_module:
```
- **EN**: Key callable entry points in this range include `get_node_first_input_and_output_type`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_node_first_input_and_output_type`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 113-144 / 第 113-144 行
```python
            return (NodeInputOrOutputType.INT8, NodeInputOrOutputType.INT8)
        else:
            return (NodeInputOrOutputType.UNKNOWN, NodeInputOrOutputType.UNKNOWN)

    elif node.op == "call_method":
        if node.target == "dequantize":
            # Dequantize is a special node because it allows multiple input types.
            # So, we look up the output type of the previous node and return that
            # as the input type of this node instance.
            prev_node = get_normalized_nth_input(node, gm, 0)
            if not isinstance(prev_node, Node):
                raise AssertionError(f"Expected Node, got {type(prev_node)}")
            (
                _prev_node_input_type,
                prev_node_output_type,
            ) = get_node_first_input_and_output_type(
                prev_node, gm, logger_cls, node_type_to_io_type_map
            )
            return (prev_node_output_type, NodeInputOrOutputType.FP32)

        elif node.target == "to":
            # to is a special node because it allows multiple input types.
            # So, we look up the output type of the previous node and return that
            # as the input type of this node instance. We also look up the target
            # of to and return the correct output type.
            prev_node = get_normalized_nth_input(node, gm, 0)
            if not isinstance(prev_node, Node):
                raise AssertionError(f"Expected Node, got {type(prev_node)}")
            (
                _prev_node_input_type,
                prev_node_output_type,
            ) = get_node_first_input_and_output_type(
```
- **EN**: Key callable entry points in this range include `get_node_first_input_and_output_type`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_node_first_input_and_output_type`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 145-170 / 第 145-170 行
```python
                prev_node, gm, logger_cls, node_type_to_io_type_map
            )

            cur_node_dtype_target = get_normalized_nth_input(node, gm, 1)
            if cur_node_dtype_target is not torch.float16:
                raise AssertionError(
                    f"{cur_node_dtype_target} handling needs to be added"
                )

            return (prev_node_output_type, NodeInputOrOutputType.FP16)

        elif node.target in METHS_IO_TYPE_FP32_OR_INT8:
            first_arg = get_normalized_nth_input(node, gm, 0)
            if not isinstance(first_arg, Node):
                raise AssertionError(f"Expected Node, got {type(first_arg)}")
            (
                _prev_node_input_type,
                prev_node_output_type,
            ) = get_node_first_input_and_output_type(
                first_arg, gm, logger_cls, node_type_to_io_type_map
            )
            return (prev_node_output_type, prev_node_output_type)

        return (NodeInputOrOutputType.UNKNOWN, NodeInputOrOutputType.UNKNOWN)
    else:
        return (NodeInputOrOutputType.UNKNOWN, NodeInputOrOutputType.UNKNOWN)
```
- **EN**: Key callable entry points in this range include `get_node_first_input_and_output_type`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `get_node_first_input_and_output_type`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 173-202 / 第 173-202 行
```python
def get_node_input_qparams(
    node: Node,
    gm: GraphModule,
    node_type_to_io_type_map: dict[str, set[NSNodeTargetType]],
) -> tuple[torch.Tensor | float, torch.Tensor | int] | None:
    """
    Returns the qparams (scale, zero_point) of the first input to `node`,
    if they can be inferred from the graph.
    """
    prev_node = get_normalized_nth_input(node, gm, 0)

    if not isinstance(prev_node, Node):
        return None

    MODS_IO_TYPE_FP32_OR_INT8 = node_type_to_io_type_map["mods_io_type_fp32_or_int8"]

    def _get_scale_zp_from_function_args(node, gm, scale_arg_idx, zp_arg_idx):
        scale_node = get_normalized_nth_input(node, gm, scale_arg_idx)
        zp_node = get_normalized_nth_input(node, gm, zp_arg_idx)
        if not isinstance(scale_node, Node):
            raise AssertionError(f"Expected Node, got {type(scale_node)}")
        if not isinstance(scale_node.target, str):
            raise AssertionError(f"Expected str, got {type(scale_node.target)}")
        if not isinstance(zp_node, Node):
            raise AssertionError(f"Expected Node, got {type(zp_node)}")
        if not isinstance(zp_node.target, str):
            raise AssertionError(f"Expected str, got {type(zp_node.target)}")
        scale_obj = getattr_from_fqn(gm, scale_node.target)
        zp_obj = getattr_from_fqn(gm, zp_node.target)
        return (scale_obj, zp_obj)
```
- **EN**: Key callable entry points in this range include `get_node_input_qparams`. They package a focused unit of behavior behind named helpers or APIs. Named constants such as `MODS_IO_TYPE_FP32_OR_INT8` centralize shared configuration or sentinel values. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `get_node_input_qparams`，它们把聚焦的行为封装成具名辅助函数或 API。 `MODS_IO_TYPE_FP32_OR_INT8` 等具名常量把共享配置或哨兵值集中定义在一起。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 204-235 / 第 204-235 行
```python
    if prev_node.op == "call_function":
        # quantize - read the args directly
        if prev_node.target is torch.quantize_per_tensor:
            return _get_scale_zp_from_function_args(prev_node, gm, 1, 2)
        elif prev_node.target in (toq.add, toq.add_relu, toq.mul, toq.mul_relu):
            return _get_scale_zp_from_function_args(prev_node, gm, 2, 3)

        return None
        # TODO(future PR): handle more functionals
        # TODO(future PR): handle functional ops which inherit qparams from input

    elif prev_node.op == "call_module":
        # get type of the module
        if not isinstance(prev_node.target, str):
            raise AssertionError(f"Expected str, got {type(prev_node.target)}")
        module_obj = getattr_from_fqn(gm, prev_node.target)
        if isinstance(
            module_obj,
            (
                nnq.Linear,
                nnq.Conv1d,
                nnq.Conv2d,
                nniq.ConvReLU2d,
                nnq.Conv3d,
                nnq.BatchNorm2d,
                nnq.BatchNorm3d,
                nnq.ConvTranspose1d,
                nnq.ConvTranspose2d,
                nnq.ELU,
                nnq.GroupNorm,
                nnq.InstanceNorm1d,
                nnq.InstanceNorm2d,
```
- **EN**: Key callable entry points in this range include `get_node_input_qparams`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_node_input_qparams`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 236-258 / 第 236-258 行
```python
                nnq.InstanceNorm3d,
                nnq.LayerNorm,
                nnq.Hardswish,
                nnq.LeakyReLU,
                nnq.ReLU6,
                nniq.BNReLU2d,
                nniq.BNReLU3d,
                nniq.ConvReLU1d,
                nniq.ConvReLU2d,
                nniq.ConvReLU3d,
                nniq.LinearReLU,
            ),
        ):
            return (module_obj.scale, module_obj.zero_point)  # type: ignore[return-value]

        is_known_fp32_or_int8_input_module = any(
            isinstance(module_obj, target_type)  # type: ignore[arg-type]
            for target_type in MODS_IO_TYPE_FP32_OR_INT8
        )
        if is_known_fp32_or_int8_input_module:
            return get_node_input_qparams(prev_node, gm, node_type_to_io_type_map)

    return None
```
- **EN**: Key callable entry points in this range include `get_node_input_qparams`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `get_node_input_qparams`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 261-292 / 第 261-292 行
```python
def return_first_non_observer_node(
    node: Node,
    gm: GraphModule,
) -> Node:
    """
    If node is not an observer, returns it.  If node is an observer,
    navigates up the graph and returns the first parent which is not an
    observer.  For example,

    graph: (node_non_obs), node = node_non_obs : returns node_non_obs
    graph: (node_non_obs -> obs0), node = obs0 : returns node_non_obs
    graph: (node_non_obs -> obs0 -> fq0), node = fq0 : returns node_non_obs
    """
    if node.op == "call_module":
        node_obj = getattr_from_fqn(gm, node.target)  # type: ignore[arg-type]
        if _is_activation_post_process(node_obj):
            if len(node.args) != 1:
                raise AssertionError(
                    f"Expected node.args to have length 1, got {len(node.args)}"
                )
            if not isinstance(node.args[0], Node):
                raise AssertionError(f"Expected Node, got {type(node.args[0])}")
            node = node.args[0]
            # code duplication intended, not worth refactoring
            if not isinstance(node.target, str):
                raise AssertionError(f"Expected str, got {type(node.target)}")
            node_obj = getattr_from_fqn(gm, node.target)
            if _is_activation_post_process(node_obj):
                if len(node.args) != 1:
                    raise AssertionError(
                        f"Expected node.args to have length 1, got {len(node.args)}"
                    )
```
- **EN**: Key callable entry points in this range include `return_first_non_observer_node`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code.
- **CN**: 这一段的重要可调用入口包括 `return_first_non_observer_node`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。

### Lines 293-322 / 第 293-322 行
```python
                if not isinstance(node.args[0], Node):
                    raise AssertionError(f"Expected Node, got {type(node.args[0])}")
                node = node.args[0]
    return node


def get_number_of_non_param_args(
    node: Node,
    gm: GraphModule,
) -> int:
    """
    Assumes that all non-param args occur first. Returns the number of
    non-param args expected for a node.  For example, for

      F.linear(x, weight, bias)

    Returns 1, because x is a non-param arg and weight and bias are params.
    For

      lstm_mod(x, hid)

    Returns 2, because both x and hid are non-param args.
    """
    if node.op == "call_module":
        node_obj = getattr_from_fqn(gm, node.target)  # type: ignore[arg-type]
        if isinstance(node_obj, nn.LSTM):
            return 2

    # default is 1
    return 1
```
- **EN**: Key callable entry points in this range include `return_first_non_observer_node`, `get_number_of_non_param_args`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `return_first_non_observer_node`, `get_number_of_non_param_args`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 325-346 / 第 325-346 行
```python
def get_arg_indices_of_inputs_to_log(node: Node) -> list[int]:
    """
    Returns the indices of args of the node which we should attach
    loggers to, if input logging is enabled.

    For example,
    * for (x + y), returns [0, 1]
    * for (1 + y), returns [1]
    * for (x + 1), returns [0]
    * for (linear(x, w, b)) returns [0]
    * by default, returns [0]
    """
    if len(node.args) == 0:
        return []
    if node.op == "call_function" and (
        # TODO(future PR): use relationship map instead of hardcoding
        node.target in (torch.add, torch.ops.quantized.add, operator.add)
        or node.target in (torch.mul, torch.ops.quantized.mul, operator.mul)
    ):
        result = [i for i in range(2) if type(node.args[i]) is Node]
        return result
    return [0]
```
- **EN**: Key callable entry points in this range include `get_arg_indices_of_inputs_to_log`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_arg_indices_of_inputs_to_log`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 349-378 / 第 349-378 行
```python
def get_target_type_str(node: Node, gm: GraphModule) -> str:
    """
    Returns a string representation of the type of the function or module
    pointed to by this node, or '' for other node types.
    """
    target_type = ""
    if node.op in ("call_function", "call_method"):
        target_type = torch.typename(node.target)
    elif node.op == "call_module":
        if not isinstance(node.target, str):
            raise AssertionError(f"Expected str, got {type(node.target)}")
        target_mod = getattr_from_fqn(gm, node.target)
        target_type = torch.typename(target_mod)
    return target_type


def rekey_logger_info_on_node_name_of_model(
    results: NSResultsType,
    model_name: str,
) -> NSResultsType:
    """
    Rekeys the layer name of a results dictionary to use node names
    from `model_name`.

    For example, transforms

        {'base_op_1_0': {'node_output': {'model_a':
          [{'ref_node_name': 'linear1', ...}]}}}

    into
```
- **EN**: Key callable entry points in this range include `get_target_type_str`, `rekey_logger_info_on_node_name_of_model`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `get_target_type_str`, `rekey_logger_info_on_node_name_of_model`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 380-408 / 第 380-408 行
```python
        {'linear1': {'node_output': {'model_a':
          [{'ref_node_name': 'linear1', ...}]}}}

    Note: we cannot use these node names directly because they are not
    guaranteed to be consistent across models. This is why we extract
    the results first and rekey afterwards.
    """
    new_results = {}
    for old_layer_name, result_type_to_results in results.items():
        new_layer_name = None
        for model_name_to_results in result_type_to_results.values():
            for cur_model_name, list_of_results in model_name_to_results.items():
                if cur_model_name == model_name:
                    if len(list_of_results) == 0:
                        raise AssertionError("Expected list_of_results to be not empty")
                    new_layer_name = list_of_results[0]["ref_node_name"]
                else:
                    continue
        if new_layer_name is not None:
            new_results[new_layer_name] = result_type_to_results
        else:
            new_results[old_layer_name] = result_type_to_results
    return new_results


def maybe_add_missing_fqns(results: NSResultsType) -> None:
    """
    If `fqn` entries are filled in for one of the models in `results`, copies
    them over to any models which do not have them filled out.
```
- **EN**: Key callable entry points in this range include `rekey_logger_info_on_node_name_of_model`, `maybe_add_missing_fqns`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `rekey_logger_info_on_node_name_of_model`, `maybe_add_missing_fqns`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 410-437 / 第 410-437 行
```python
    A common use case benefitting from this is comparing a model prepared by
    quantization to a quantized model. In this case, the model prepared by
    quantization would have `fqn` entries, and the quantized model would not.
    """

    # Check in the first result to find any model with fqn entries defined.
    model_name_with_fqns = None
    for result_type_to_results in results.values():
        for model_name_to_results in result_type_to_results.values():
            for model_name, model_results in model_name_to_results.items():
                if len(model_results) > 0:
                    if model_results[0]["fqn"] is not None:
                        model_name_with_fqns = model_name
                        break
            break
        break

    if model_name_with_fqns:
        for result_type_to_results in results.values():
            for model_name_to_results in result_type_to_results.values():
                ref_model_results = model_name_to_results[model_name_with_fqns]
                for model_name, model_results in model_name_to_results.items():
                    if model_name == model_name_with_fqns:
                        continue

                    for i in range(len(model_results)):
                        fqn = ref_model_results[i]["fqn"]
                        model_results[i]["fqn"] = fqn
```
- **EN**: Key callable entry points in this range include `maybe_add_missing_fqns`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `maybe_add_missing_fqns`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 440-466 / 第 440-466 行
```python
def maybe_dequantize_first_two_tensor_args_and_handle_tuples(f):
    def inner(*args, **kwargs):
        a0, a1, *a_other = args

        if (isinstance(a0, tuple) and isinstance(a1, tuple)) or (
            isinstance(a0, list) and isinstance(a1, list)
        ):
            results = []
            for el0, el1 in zip(a0, a1):
                new_args = (el0, el1, *a_other)
                results.append(inner(*new_args, **kwargs))
            return results

        elif isinstance(a0, torch.Tensor) and isinstance(a1, torch.Tensor):
            if a0.is_quantized:
                a0 = a0.dequantize()
            if a1.is_quantized:
                a1 = a1.dequantize()

        # for the purposes of this util, only handle floats
        if a0.dtype != torch.float or a1.dtype != torch.float:
            return None

        new_args = (a0, a1, *a_other)
        return f(*new_args, **kwargs)

    return inner
```
- **EN**: Key callable entry points in this range include `maybe_dequantize_first_two_tensor_args_and_handle_tuples`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `maybe_dequantize_first_two_tensor_args_and_handle_tuples`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 469-499 / 第 469-499 行
```python
@maybe_dequantize_first_two_tensor_args_and_handle_tuples
def compute_sqnr(x: torch.Tensor, y: torch.Tensor) -> torch.Tensor:
    """
    Computes the SQNR between `x` and `y`.

    Args:
        x: Tensor or tuple of tensors
        y: Tensor or tuple of tensors

    Return:
        float or tuple of floats
    """
    Ps = torch.norm(x)
    Pn = torch.norm(x - y)
    return 20 * torch.log10(Ps / Pn)


@maybe_dequantize_first_two_tensor_args_and_handle_tuples
def compute_normalized_l2_error(x: torch.Tensor, y: torch.Tensor) -> torch.Tensor:
    """
    Computes the normalized L2 error between `x` and `y`.

    Args:
        x: Tensor or tuple of tensors
        y: Tensor or tuple of tensors

    Return:
        float or tuple of floats
    """

    return torch.sqrt(((x - y) ** 2).sum() / (x**2).sum())
```
- **EN**: Key callable entry points in this range include `compute_sqnr`, `compute_normalized_l2_error`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `compute_sqnr`, `compute_normalized_l2_error`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 502-533 / 第 502-533 行
```python
@maybe_dequantize_first_two_tensor_args_and_handle_tuples
def compute_cosine_similarity(x: torch.Tensor, y: torch.Tensor) -> torch.Tensor:
    """
    Computes the cosine similarity between `x` and `y`.

    Args:
        x: Tensor or tuple of tensors
        y: Tensor or tuple of tensors

    Return:
        float or tuple of floats
    """
    # For convolutions, the shape of the quantized weight has one additional
    # dimension compared to the shape of the fp32 weight. Match the shapes
    # to enable cosine similarity comparison.
    x = x.reshape(1, -1)
    y = y.reshape(1, -1)
    return torch.nn.functional.cosine_similarity(x, y)


def op_type_supports_shadowing(node: Node) -> bool:
    if node.op == "call_function":
        if node.target in (
            torch.add,
            torch.mul,
            operator.add,
            operator.mul,
            torch.cat,
            torch.stack,
        ):
            # shadowing for ops with multiple tensor inputs is not implemented yet
            return False
```
- **EN**: Key callable entry points in this range include `compute_cosine_similarity`, `op_type_supports_shadowing`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `compute_cosine_similarity`, `op_type_supports_shadowing`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 534-565 / 第 534-565 行
```python
    return True


def get_normalized_nth_input(node: Node, gm: GraphModule, idx: int) -> Node:
    """
    Given a node, gets the n'th input to that node, normalizing
    args and kwargs to the best of its ability.
    """
    try:
        norm_args_and_kwargs = node.normalized_arguments(
            gm, normalize_to_only_use_kwargs=True
        )
        if norm_args_and_kwargs is not None:
            norm_args, norm_kwargs = norm_args_and_kwargs
            if len(norm_args) + len(norm_kwargs) <= idx:
                raise AssertionError(
                    f"Index {idx} out of range: total = {len(norm_args) + len(norm_kwargs)}"
                )
            if idx < len(norm_args):
                return norm_args[idx]
            else:
                # note: in Python 3.7+ dicts are ordered
                return list(norm_kwargs.values())[idx]
        else:
            if len(node.args) + len(node.kwargs) <= idx:
                raise AssertionError(
                    f"Index {idx} out of range: total = {len(node.args) + len(node.kwargs)}"
                )
            if idx < len(node.args):
                return node.args[idx]  # type: ignore[return-value]
            else:
                kwargs_idx = idx + len(node.args)
```
- **EN**: Key callable entry points in this range include `op_type_supports_shadowing`, `get_normalized_nth_input`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Structured exception handling keeps failure modes explicit instead of letting partial state escape silently. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `op_type_supports_shadowing`, `get_normalized_nth_input`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 结构化异常处理让失败模式保持显式，而不是让部分状态悄悄泄漏出去。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 566-579 / 第 566-579 行
```python
                return list(node.kwargs.values())[kwargs_idx]  # type: ignore[return-value]
    except RuntimeError:
        # this RuntimeError happens when node argument normalization
        # requires typehints to proceed, such as for torch.add where
        # either the first, second or both arguments could be tensors
        if len(node.args) + len(node.kwargs) <= idx:
            raise AssertionError(
                f"Index {idx} out of range: total = {len(node.args) + len(node.kwargs)}"
            ) from None
        if idx < len(node.args):
            return node.args[idx]  # type: ignore[return-value]
        else:
            kwargs_idx = idx + len(node.args)
            return list(node.kwargs.values())[kwargs_idx]  # type: ignore[return-value]
```
- **EN**: Key callable entry points in this range include `get_normalized_nth_input`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Structured exception handling keeps failure modes explicit instead of letting partial state escape silently. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `get_normalized_nth_input`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 结构化异常处理让失败模式保持显式，而不是让部分状态悄悄泄漏出去。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements numeric-suite tooling used to compare floating-point and quantized model behavior.
  - CN: 实现数值套件工具，用于比较浮点模型与量化模型的行为。
- **Quantization pipeline**
  - EN: Tracks calibration statistics, quantization parameters, or graph/module rewrites for lower-precision execution.
  - CN: 跟踪校准统计信息、量化参数，或面向低精度执行的图/模块改写。
- **Observer statistics**
  - EN: Collects runtime ranges or histograms so later code can derive quantization settings.
  - CN: 收集运行时范围或直方图，以便后续逻辑推导量化设置。
- **Fake quantization**
  - EN: Simulates quantized numerics during training while keeping tensors in differentiable forms.
  - CN: 在训练阶段模拟量化数值行为，同时保持张量处于可求导形式。
- **Graph rewriting**
  - EN: Rewrites module or FX graph structure so later execution follows a transformed pipeline.
  - CN: 改写模块或 FX 图结构，使后续执行遵循转换后的流水线。
- **Numeric comparison**
  - EN: Helps compare reference and transformed executions to diagnose quantization effects.
  - CN: 帮助比较参考执行与转换后执行，从而诊断量化带来的影响。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch.ao.nn.intrinsic.quantized`, `torch.ao.nn.quantized`, `torch.nn`, `torch.ao.quantization:FakeQuantizeBase`, `torch.ao.quantization:ObserverBase`, `torch.ao.quantization.observer:_is_activation_post_process`, `torch.ao.quantization.utils:getattr_from_fqn`, `torch.fx:GraphModule`, `torch.fx.graph:Node`, `.ns_types:NSNodeTargetType`, `.ns_types:NSResultsType`
- **Python standard library / Python 标准库**: `enum`, `operator`, `collections.abc:Callable`
- **Primary symbols / 核心符号**: `NodeInputOrOutputType`, `get_node_first_input_and_output_type`, `get_node_input_qparams`, `return_first_non_observer_node`, `get_number_of_non_param_args`, `get_arg_indices_of_inputs_to_log`, `get_target_type_str`, `rekey_logger_info_on_node_name_of_model`, `maybe_add_missing_fqns`, `maybe_dequantize_first_two_tensor_args_and_handle_tuples`, `compute_sqnr`, `compute_normalized_l2_error`, `compute_cosine_similarity`, `op_type_supports_shadowing`, `get_normalized_nth_input`
