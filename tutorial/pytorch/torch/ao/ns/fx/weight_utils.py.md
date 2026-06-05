# weight_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/ns/fx/weight_utils.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements numeric-suite tooling used to compare floating-point and quantized model behavior. This specific file centers on `weight_utils.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior. It also participates in graph/module transformation before or after quantized execution.
- **Purpose (CN) / 用途（中文）**: 实现数值套件工具，用于比较浮点模型与量化模型的行为。 该文件具体围绕 `weight_utils.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。 它也参与量化执行前后的图/模块转换。
## Line-by-Line Analysis / 逐行分析

### Lines 1-23 / 第 1-23 行
```python
from collections.abc import Callable

import torch
import torch.ao.nn.intrinsic as nni
import torch.ao.nn.intrinsic.qat as nniqat
import torch.ao.nn.intrinsic.quantized as nniq
import torch.ao.nn.qat as nnqat
import torch.ao.nn.quantized as nnq
import torch.ao.nn.quantized.dynamic as nnqd
import torch.nn as nn
import torch.nn.functional as F
from torch.fx import GraphModule
from torch.fx.graph import Node

from .ns_types import NSSingleResultType, NSSingleResultValuesType
from .utils import get_target_type_str, getattr_from_fqn, return_first_non_observer_node


toq = torch.ops.quantized


def mod_weight_detach(mod: nn.Module) -> torch.Tensor:
    return mod.weight.detach()  # type: ignore[operator]
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch, torch.ao.nn.intrinsic, torch.ao.nn.intrinsic.qat, torch.ao.nn.intrinsic.quantized; standard-library helpers such as collections.abc:Callable. Key callable entry points in this range include `mod_weight_detach`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch, torch.ao.nn.intrinsic, torch.ao.nn.intrinsic.qat, torch.ao.nn.intrinsic.quantized；标准库辅助模块，如 collections.abc:Callable。 这一段的重要可调用入口包括 `mod_weight_detach`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 26-48 / 第 26-48 行
```python
def mod_0_weight_detach(mod: nn.Module) -> torch.Tensor:
    return mod[0].weight.detach()  # type: ignore[index]


def mod_weight_bias_0(mod: nn.Module) -> torch.Tensor:
    return mod._weight_bias()[0]  # type: ignore[operator]


def get_lstm_weight(mod: nn.Module) -> list[torch.Tensor]:
    res = []
    for idx, param_name in enumerate(mod._flat_weights_names):  # type: ignore[arg-type]
        if "weight_ih_l" in param_name or "weight_hh_l" in param_name:
            param_value = mod._flat_weights[idx].detach()  # type: ignore[index,union-attr]
            res.append(param_value)
    return res


def get_qlstm_weight(mod: nn.Module) -> list[torch.Tensor]:
    res = []
    for weight_value in mod._all_weight_values:  # type: ignore[union-attr]
        res.append(weight_value.param.__getstate__()[0][4][0].__getstate__()[0][0])
        res.append(weight_value.param.__getstate__()[0][4][1].__getstate__()[0][0])
    return res
```
- **EN**: Key callable entry points in this range include `mod_0_weight_detach`, `mod_weight_bias_0`, `get_lstm_weight`, `get_qlstm_weight`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `mod_0_weight_detach`, `mod_weight_bias_0`, `get_lstm_weight`, `get_qlstm_weight`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 51-66 / 第 51-66 行
```python
def get_conv_mod_weight(mod: nn.Module) -> torch.Tensor:
    if isinstance(mod, (nn.Conv1d, nn.Conv2d, nn.Conv3d)):
        return mod.weight.detach()
    elif isinstance(mod, (nni.ConvReLU1d, nni.ConvReLU2d, nni.ConvReLU3d)):
        return mod[0].weight.detach()  # type: ignore[operator]
    else:
        return mod._weight_bias()[0]  # type: ignore[operator]


def get_linear_mod_weight(mod: nn.Module) -> torch.Tensor:
    if isinstance(mod, nn.Linear):
        return mod.weight.detach()
    elif isinstance(mod, nni.LinearReLU):
        return mod[0].weight.detach()  # type: ignore[operator]
    else:
        return mod._weight_bias()[0]  # type: ignore[operator]
```
- **EN**: Key callable entry points in this range include `get_conv_mod_weight`, `get_linear_mod_weight`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `get_conv_mod_weight`, `get_linear_mod_weight`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 69-89 / 第 69-89 行
```python
def get_lstm_mod_weights(mod: nn.Module) -> list[torch.Tensor]:
    # TODO(future PR): make more generic, handle everything
    if isinstance(mod, nn.LSTM):
        res = []
        for idx, param_name in enumerate(mod._flat_weights_names):
            if "weight_ih_l" in param_name or "weight_hh_l" in param_name:
                param_value = mod._flat_weights[idx].detach()  # type: ignore[index,union-attr]
                res.append(param_value)
        return res
    else:
        if not isinstance(mod, nnqd.LSTM):
            raise AssertionError(f"type {type(mod)} not handled yet")
        res = []
        for weight_value in mod._all_weight_values:
            res.append(
                weight_value.param.__getstate__()[0][4][0].__getstate__()[0][0]  # type: ignore[index]
            )
            res.append(
                weight_value.param.__getstate__()[0][4][1].__getstate__()[0][0]  # type: ignore[index]
            )
        return res
```
- **EN**: Key callable entry points in this range include `get_lstm_mod_weights`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `get_lstm_mod_weights`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 92-114 / 第 92-114 行
```python
def get_conv_fun_weight(node: Node, gm: GraphModule) -> torch.Tensor:
    # traverse backwards from the weight arg, accounting for any observers
    weight_arg_node = node.args[1]
    if not isinstance(weight_arg_node, Node):
        raise AssertionError(f"Expected Node, got {type(weight_arg_node)}")
    weight_node = return_first_non_observer_node(weight_arg_node, gm)
    if not isinstance(weight_node, Node):
        raise AssertionError(f"Expected Node, got {type(weight_node)}")
    if weight_node.op != "get_attr":
        raise AssertionError(f"Expected get_attr, got {weight_node.op}")
    weight = getattr_from_fqn(gm, weight_node.target)  # type: ignore[arg-type]
    return weight.detach()


def get_qconv_fun_weight(node: Node, gm: GraphModule) -> torch.Tensor:
    # qconv state is arg 1
    qconv_state_node = node.args[1]
    if not isinstance(qconv_state_node, Node):
        raise AssertionError(f"Expected Node, got {type(qconv_state_node)}")
    if qconv_state_node.op != "get_attr":
        raise AssertionError(f"Expected get_attr, got {qconv_state_node.op}")
    qconv_state_obj = getattr_from_fqn(gm, qconv_state_node.target)  # type: ignore[arg-type]
    return qconv_state_obj.weight()
```
- **EN**: Key callable entry points in this range include `get_conv_fun_weight`, `get_qconv_fun_weight`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `get_conv_fun_weight`, `get_qconv_fun_weight`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 117-140 / 第 117-140 行
```python
def get_linear_fun_weight(node: Node, gm: GraphModule) -> torch.Tensor:
    # traverse backwards from the weight arg, accounting for any observers
    # supported patterns:
    # weight -> obs -> linear
    # weight -> to(torch.float16) -> dequantize -> linear
    linear_second_arg = node.args[1]
    if not isinstance(linear_second_arg, Node):
        raise AssertionError(f"Expected Node, got {type(linear_second_arg)}")

    if linear_second_arg.op == "call_module":
        # weight -> obs -> linear
        weight_arg_node = node.args[1]
        if not isinstance(weight_arg_node, Node):
            raise AssertionError(f"Expected Node, got {type(weight_arg_node)}")
        weight_node = weight_arg_node.args[0]
        if not isinstance(weight_node, Node):
            raise AssertionError(f"Expected Node, got {type(weight_node)}")
        if weight_node.op != "get_attr":
            raise AssertionError(f"Expected get_attr, got {weight_node.op}")
        weight = getattr_from_fqn(gm, weight_node.target)  # type: ignore[arg-type]
        return weight.detach()
    elif linear_second_arg.op == "call_method":
        # weight -> to(torch.float16) -> dequantize -> linear
        if linear_second_arg.op != "call_method":
```
- **EN**: Key callable entry points in this range include `get_linear_fun_weight`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_linear_fun_weight`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 141-162 / 第 141-162 行
```python
            raise AssertionError(f"Expected call_method, got {linear_second_arg.op}")
        dequant_node = node.args[1]
        if not isinstance(dequant_node, Node):
            raise AssertionError(f"Expected Node, got {type(dequant_node)}")
        to_fp16_node = dequant_node.args[0]
        if not isinstance(to_fp16_node, Node):
            raise AssertionError(f"Expected Node, got {type(to_fp16_node)}")
        # extract the dtype, so we can cast to it before returning
        target_dtype = to_fp16_node.args[1]
        weight_node = to_fp16_node.args[0]
        if not isinstance(weight_node, Node):
            raise AssertionError(f"Expected Node, got {type(weight_node)}")
        if weight_node.op != "get_attr":
            raise AssertionError(f"Expected get_attr, got {weight_node.op}")
        weight = getattr_from_fqn(gm, weight_node.target)  # type: ignore[arg-type]
        # return the weight with fp16 cast
        return weight.detach().to(target_dtype)
    else:
        if linear_second_arg.op != "get_attr":
            raise AssertionError(f"Expected get_attr, got {linear_second_arg.op}")
        weight = getattr_from_fqn(gm, linear_second_arg.target)  # type: ignore[arg-type]
        return weight.detach()
```
- **EN**: Key callable entry points in this range include `get_linear_fun_weight`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_linear_fun_weight`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 165-188 / 第 165-188 行
```python
def get_qlinear_fun_weight(node: Node, gm: GraphModule) -> torch.Tensor:
    # packed weight is arg 1
    packed_weight_node = node.args[1]
    if not isinstance(packed_weight_node, Node):
        raise AssertionError(f"Expected Node, got {type(packed_weight_node)}")
    if packed_weight_node.op != "get_attr":
        raise AssertionError(f"Expected get_attr, got {packed_weight_node.op}")
    packed_weight = getattr_from_fqn(gm, packed_weight_node.target)  # type: ignore[arg-type]
    # TODO(future PR): why does packed_weight.unpack() not work?
    (weight, _bias), _name = packed_weight.__getstate__()
    return weight


def get_op_to_type_to_weight_extraction_fn() -> dict[str, dict[Callable, Callable]]:
    op_to_type_to_weight_extraction_fn: dict[str, dict[Callable, Callable]] = {
        "call_module": {
            # Conv1d
            nn.Conv1d: mod_weight_detach,
            nni.ConvReLU1d: mod_0_weight_detach,
            nnq.Conv1d: mod_weight_bias_0,
            nnqat.Conv1d: mod_weight_detach,
            nniqat.ConvBn1d: mod_weight_detach,
            nniqat.ConvBnReLU1d: mod_weight_detach,
            nniqat.ConvReLU1d: mod_weight_detach,
```
- **EN**: Key callable entry points in this range include `get_qlinear_fun_weight`, `get_op_to_type_to_weight_extraction_fn`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `get_qlinear_fun_weight`, `get_op_to_type_to_weight_extraction_fn`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 189-212 / 第 189-212 行
```python
            nniq.ConvReLU1d: mod_weight_bias_0,
            # Conv2d
            nn.Conv2d: mod_weight_detach,
            nni.ConvReLU2d: mod_0_weight_detach,
            nnq.Conv2d: mod_weight_bias_0,
            nnqat.Conv2d: mod_weight_detach,
            nniqat.ConvBn2d: mod_weight_detach,
            nniqat.ConvBnReLU2d: mod_weight_detach,
            nniqat.ConvReLU2d: mod_weight_detach,
            nniq.ConvReLU2d: mod_weight_bias_0,
            # Conv3d
            nn.Conv3d: mod_weight_detach,
            nni.ConvReLU3d: mod_0_weight_detach,
            nnq.Conv3d: mod_weight_bias_0,
            nnqat.Conv3d: mod_weight_detach,
            nniqat.ConvBn3d: mod_weight_detach,
            nniqat.ConvBnReLU3d: mod_weight_detach,
            nniqat.ConvReLU3d: mod_weight_detach,
            nniq.ConvReLU3d: mod_weight_bias_0,
            # Linear
            nn.Linear: mod_weight_detach,
            nnq.Linear: mod_weight_bias_0,
            nni.LinearReLU: mod_0_weight_detach,
            nniq.LinearReLU: mod_weight_bias_0,
```
- **EN**: Key callable entry points in this range include `get_op_to_type_to_weight_extraction_fn`. They package a focused unit of behavior behind named helpers or APIs.
- **CN**: 这一段的重要可调用入口包括 `get_op_to_type_to_weight_extraction_fn`，它们把聚焦的行为封装成具名辅助函数或 API。

### Lines 213-236 / 第 213-236 行
```python
            nnqat.Linear: mod_weight_detach,
            nnqd.Linear: mod_weight_bias_0,
            nniqat.LinearReLU: mod_weight_detach,
            nniqat.LinearBn1d: mod_weight_detach,
            nn.modules.linear.NonDynamicallyQuantizableLinear: mod_weight_detach,
            # LSTM
            nn.LSTM: get_lstm_weight,
            nnqd.LSTM: get_qlstm_weight,
        },
        "call_function": {
            # Conv
            F.conv1d: get_conv_fun_weight,
            F.conv2d: get_conv_fun_weight,
            F.conv3d: get_conv_fun_weight,
            toq.conv1d: get_qconv_fun_weight,
            toq.conv2d: get_qconv_fun_weight,
            toq.conv3d: get_qconv_fun_weight,
            toq.conv1d_relu: get_qconv_fun_weight,
            toq.conv2d_relu: get_qconv_fun_weight,
            toq.conv3d_relu: get_qconv_fun_weight,
            # Linear
            F.linear: get_linear_fun_weight,
            toq.linear: get_qlinear_fun_weight,
            toq.linear_relu: get_qlinear_fun_weight,
```
- **EN**: Key callable entry points in this range include `get_op_to_type_to_weight_extraction_fn`. They package a focused unit of behavior behind named helpers or APIs. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_op_to_type_to_weight_extraction_fn`，它们把聚焦的行为封装成具名辅助函数或 API。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 237-258 / 第 237-258 行
```python
        },
    }

    return op_to_type_to_weight_extraction_fn


def extract_weight_from_node(
    node: Node,
    gm: GraphModule,
    op_to_type_to_weight_extraction_fn: dict[str, dict[Callable, Callable]]
    | None = None,
) -> NSSingleResultType | None:
    res_type = NSSingleResultValuesType.WEIGHT.value

    # Not all graphmodules have _node_name_to_scope, so only fill it
    # out if it exists.
    fqn = None
    if hasattr(gm, "_node_name_to_scope"):
        fqn = gm._node_name_to_scope[node.name][0]  # type: ignore[index]

    if op_to_type_to_weight_extraction_fn is None:
        op_to_type_to_weight_extraction_fn = get_op_to_type_to_weight_extraction_fn()
```
- **EN**: Key callable entry points in this range include `get_op_to_type_to_weight_extraction_fn`, `extract_weight_from_node`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `get_op_to_type_to_weight_extraction_fn`, `extract_weight_from_node`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 260-279 / 第 260-279 行
```python
    ref_node_type = get_target_type_str(node, gm)
    # for extracting weights, these are always the same
    prev_node_type = ref_node_type

    if node.op == "call_function":
        function_mapping = op_to_type_to_weight_extraction_fn["call_function"]
        for target_fn_type, weight_extraction_fn in function_mapping.items():
            if node.target == target_fn_type:
                weight = weight_extraction_fn(node, gm)
                return {
                    "type": res_type,
                    "values": [weight],
                    "prev_node_name": node.name,
                    "prev_node_target_type": prev_node_type,
                    "ref_node_name": node.name,
                    "ref_node_target_type": ref_node_type,
                    "index_within_arg": 0,
                    "index_of_arg": 0,
                    "fqn": fqn,
                }
```
- **EN**: Key callable entry points in this range include `extract_weight_from_node`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `extract_weight_from_node`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 281-302 / 第 281-302 行
```python
    elif node.op == "call_module":
        # for call_module, we need to look up the modules to do the type check
        if not isinstance(node.target, str):
            raise AssertionError(f"Expected str, got {type(node.target)}")
        mod = getattr_from_fqn(gm, node.target)
        module_mapping = op_to_type_to_weight_extraction_fn["call_module"]
        for target_mod_type, weight_extraction_fn in module_mapping.items():
            if type(mod) is target_mod_type:
                weight = weight_extraction_fn(mod)
                return {
                    "type": res_type,
                    "values": [weight],
                    "prev_node_name": node.name,
                    "prev_node_target_type": prev_node_type,
                    "ref_node_name": node.name,
                    "ref_node_target_type": ref_node_type,
                    "index_within_arg": 0,
                    "index_of_arg": 0,
                    "fqn": fqn,
                }

    return None
```
- **EN**: Key callable entry points in this range include `extract_weight_from_node`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `extract_weight_from_node`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

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
- **mod_weight_detach**
  - EN: `mod_weight_detach` is a representative function that exposes or coordinates an important action in this module.
  - CN: `mod_weight_detach` 是一个具有代表性的函数，用于暴露或协调此模块中的关键动作。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch.ao.nn.intrinsic`, `torch.ao.nn.intrinsic.qat`, `torch.ao.nn.intrinsic.quantized`, `torch.ao.nn.qat`, `torch.ao.nn.quantized`, `torch.ao.nn.quantized.dynamic`, `torch.nn`, `torch.nn.functional`, `torch.fx:GraphModule`, `torch.fx.graph:Node`, `.ns_types:NSSingleResultType`, `.ns_types:NSSingleResultValuesType`, `.utils:get_target_type_str`, `.utils:getattr_from_fqn`
- **Python standard library / Python 标准库**: `collections.abc:Callable`
- **Primary symbols / 核心符号**: `mod_weight_detach`, `mod_0_weight_detach`, `mod_weight_bias_0`, `get_lstm_weight`, `get_qlstm_weight`, `get_conv_mod_weight`, `get_linear_mod_weight`, `get_lstm_mod_weights`, `get_conv_fun_weight`, `get_qconv_fun_weight`, `get_linear_fun_weight`, `get_qlinear_fun_weight`, `get_op_to_type_to_weight_extraction_fn`, `extract_weight_from_node`
