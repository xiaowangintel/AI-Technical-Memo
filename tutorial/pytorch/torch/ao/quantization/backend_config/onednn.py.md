# onednn.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/quantization/backend_config/onednn.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements quantization observers, fake-quant modules, preparation/conversion passes, and backend-specific quantization helpers. This specific file centers on `onednn.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior. It also participates in graph/module transformation before or after quantized execution. Runtime configuration, environment overrides, or feature gating are important in this module.
- **Purpose (CN) / 用途（中文）**: 实现量化 observer、fake-quant 模块、prepare/convert pass 以及后端相关的量化辅助逻辑。 该文件具体围绕 `onednn.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。 它也参与量化执行前后的图/模块转换。 运行时配置、环境变量覆盖或特性开关是此模块的重要主题。
## Line-by-Line Analysis / 逐行分析

### Lines 1-31 / 第 1-31 行
```python
# mypy: allow-untyped-defs
import itertools
import operator

import torch
import torch.ao.nn.intrinsic as nni
import torch.ao.nn.quantized.reference as nnqr
import torch.nn as nn
import torch.nn.functional as F
from torch.ao.quantization.fuser_method_mappings import _sequential_wrapper2
from torch.ao.quantization.utils import MatchAllNode

from ._common_operator_config_utils import (
    _get_binary_op_configs,
    _get_bn_configs,
    _get_cat_config,
    _get_conv_configs,
    _get_default_op_configs,
    _get_embedding_op_configs,
    _get_fixed_qparams_op_configs,
    _get_linear_configs,
    _get_ln_configs,
    _get_rnn_op_configs,
    _get_share_qparams_op_configs,
)
from .backend_config import (
    BackendConfig,
    BackendPatternConfig,
    DTypeConfig,
    ObservationType,
)
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch, torch.ao.nn.intrinsic, torch.ao.nn.quantized.reference, torch.nn; standard-library helpers such as itertools, operator. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch, torch.ao.nn.intrinsic, torch.ao.nn.quantized.reference, torch.nn；标准库辅助模块，如 itertools, operator。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 34-62 / 第 34-62 行
```python
# ===================
# |  DTYPE CONFIGS  |
# ===================

onednn_weighted_op_int8_dtype_config = DTypeConfig(
    input_dtype=torch.quint8,
    output_dtype=torch.quint8,
    weight_dtype=torch.qint8,
    bias_dtype=torch.float,
)

onednn_op_quint8_dtype_config = DTypeConfig(
    input_dtype=torch.quint8,
    output_dtype=torch.quint8,
)

onednn_dynamic_int8_dtype_config = DTypeConfig(
    input_dtype=torch.quint8,
    output_dtype=torch.float,
    weight_dtype=torch.qint8,
    bias_dtype=torch.float,
    is_dynamic=True,
)

onednn_weight_only_qint8_dtype_config = DTypeConfig(
    input_dtype=torch.float,
    output_dtype=torch.float,
    weight_dtype=torch.qint8,
)
```
- **EN**: The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 64-94 / 第 64-94 行
```python
onednn_input_output_only_quint8_dtype_config = DTypeConfig(
    input_dtype=torch.quint8,
    output_dtype=torch.quint8,
    weight_dtype=torch.float,
    bias_dtype=torch.float,
)

# ===================
# |  FUSER METHODS  |
# ===================


def _fuse_linear_bn_leaky_relu(is_qat, linear, bn, leaky_relu):
    r"""Given the linear, bn and leaky_relu modules, fuses them and returns the fused module
    Args:
        is_qat: a flag for whether we are using quantization aware training fusion
                or post training quantization fusion
        linear: Module instance of type Linear
        bn: BatchNorm1d instance that needs to be fused with the linear layer
        leaky_relu: LeakyReLU instance that needs to be fused with the linear layer
    Examples::
        >>> # xdoctest: +SKIP(failing)
        >>> m1 = nn.Linear(20, 10)
        >>> b1 = nn.BatchNorm1d(10)
        >>> lr = nn.LeakyReLU(0.01)
        >>> m2 = _fuse_linear_bn_leaky_relu(m1, b1, lr)
    """
    if linear.training != bn.training or bn.training != leaky_relu.training:
        raise AssertionError(
            "Linear, BN and LeakyReLU all must be in the same mode (train or eval)."
        )
```
- **EN**: Key callable entry points in this range include `_fuse_linear_bn_leaky_relu`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_fuse_linear_bn_leaky_relu`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 96-123 / 第 96-123 行
```python
    if is_qat:
        raise NotImplementedError(
            f"Cannot fuse train modules: {(linear, bn, leaky_relu)}"
        )
    else:
        map_to_fused_module_eval = {
            nn.Linear: nni.LinearLeakyReLU,
        }
        fused_module = map_to_fused_module_eval.get(type(linear))
        if fused_module is not None:
            fused_linear = nn.utils.fusion.fuse_linear_bn_eval(linear, bn)
            fm = fused_module(fused_linear, leaky_relu)
            return fm
        else:
            raise NotImplementedError(
                f"Cannot fuse eval modules: {(linear, bn, leaky_relu)}"
            )


# ======================
# |  CONFIGS FOR CONV  |
# ======================
observation_type = ObservationType.OUTPUT_USE_DIFFERENT_OBSERVER_AS_INPUT

conv_dtype_configs = [onednn_weighted_op_int8_dtype_config]
conv_configs = _get_conv_configs(conv_dtype_configs)

# (1) Conv2d + Add
```
- **EN**: Key callable entry points in this range include `_fuse_linear_bn_leaky_relu`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_fuse_linear_bn_leaky_relu`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 125-149 / 第 125-149 行
```python
# conv2d   Y
#   \   /
#    add

# include:
# conv2d conv2d
#   \   /
#    add


def _fuse_conv_add_left(is_qat, add, conv, _):
    return nni.ConvAdd2d(conv, add)


def _conv_add_root_node_getter_left(pattern):
    _, conv, _ = pattern
    return conv


def _conv_add_extra_inputs_getter_left(pattern):
    """get inputs pattern for extra inputs, inputs for root node
    are assumed to be copied over from root node to the fused node
    """
    _, _conv, extra_input = pattern
    return [extra_input]
```
- **EN**: Key callable entry points in this range include `_fuse_conv_add_left`, `_conv_add_root_node_getter_left`, `_conv_add_extra_inputs_getter_left`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_fuse_conv_add_left`, `_conv_add_root_node_getter_left`, `_conv_add_extra_inputs_getter_left`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 152-179 / 第 152-179 行
```python
# conv2d
#  \
#  bn   Y
#   \   /
#    add


def _fuse_conv_bn_add_left(is_qat, add, bn_conv, _):
    bn, conv = bn_conv
    if is_qat:
        raise NotImplementedError(f"Cannot fuse train modules: {(conv, bn, add)}")
    else:
        fused_conv = nn.utils.fusion.fuse_conv_bn_eval(conv, bn)
        return nni.ConvAdd2d(fused_conv, add)


def _conv_bn_add_root_node_getter_left(add_pattern):
    _, bn_conv, _ = add_pattern
    _bn, conv = bn_conv
    return conv


def _conv_bn_add_extra_inputs_getter_left(add_pattern):
    """get inputs pattern for extra inputs, inputs for root node
    are assumed to be copied over from root node to the fused node
    """
    _, _bn_conv, extra_input = add_pattern
    return [extra_input]
```
- **EN**: Key callable entry points in this range include `_fuse_conv_bn_add_left`, `_conv_bn_add_root_node_getter_left`, `_conv_bn_add_extra_inputs_getter_left`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_fuse_conv_bn_add_left`, `_conv_bn_add_root_node_getter_left`, `_conv_bn_add_extra_inputs_getter_left`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 182-211 / 第 182-211 行
```python
conv_add_left_optioins = itertools.product(
    [True, False],  # with_bn
    [torch.add, operator.add],  # add_op
)

for with_bn, add_op in conv_add_left_optioins:
    if with_bn:
        conv_configs.append(
            BackendPatternConfig()
            ._set_pattern_complex_format(
                (add_op, (nn.BatchNorm2d, nn.Conv2d), MatchAllNode)
            )
            .set_observation_type(observation_type)
            .set_dtype_configs(conv_dtype_configs)
            .set_fuser_method(_fuse_conv_bn_add_left)
            ._set_root_node_getter(_conv_bn_add_root_node_getter_left)
            ._set_extra_inputs_getter(_conv_bn_add_extra_inputs_getter_left)
            .set_fused_module(nni.ConvAdd2d)
        )
    else:
        conv_configs.append(
            BackendPatternConfig()
            ._set_pattern_complex_format((add_op, nn.Conv2d, MatchAllNode))
            .set_observation_type(observation_type)
            .set_dtype_configs(conv_dtype_configs)
            .set_fuser_method(_fuse_conv_add_left)
            ._set_root_node_getter(_conv_add_root_node_getter_left)
            ._set_extra_inputs_getter(_conv_add_extra_inputs_getter_left)
            .set_fused_module(nni.ConvAdd2d)
        )
```
- **EN**: This chunk continues the implementation of `_conv_bn_add_extra_inputs_getter_left`, filling in the details of its control flow or data handling. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段延续了 `_conv_bn_add_extra_inputs_getter_left` 的实现，继续补充其控制流或数据处理细节。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 213-239 / 第 213-239 行
```python
#  Y   conv2d
#   \   /
#    add


def _fuse_conv_add_right(is_qat, add, _, conv):
    return nni.ConvAdd2d(conv, add)


def _conv_add_root_node_getter_right(pattern):
    _add, _, conv = pattern
    return conv


def _conv_add_extra_inputs_getter_right(pattern):
    """get inputs pattern for extra inputs, inputs for root node
    are assumed to be copied over from root node to the fused node
    """
    _, extra_input, _conv = pattern
    return [extra_input]


#      conv2d
#        /
#  Y    bn
#   \   /
#    add
```
- **EN**: Key callable entry points in this range include `_fuse_conv_add_right`, `_conv_add_root_node_getter_right`, `_conv_add_extra_inputs_getter_right`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_fuse_conv_add_right`, `_conv_add_root_node_getter_right`, `_conv_add_extra_inputs_getter_right`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 242-268 / 第 242-268 行
```python
def _fuse_conv_bn_add_right(is_qat, add, _, bn_conv):
    bn, conv = bn_conv
    if is_qat:
        raise NotImplementedError(f"Cannot fuse train modules: {(conv, bn, add)}")
    else:
        fused_conv = nn.utils.fusion.fuse_conv_bn_eval(conv, bn)
        return nni.ConvAdd2d(fused_conv, add)


def _conv_bn_add_root_node_getter_right(pattern):
    _add, _, bn_conv = pattern
    _bn, conv = bn_conv
    return conv


def _conv_bn_add_extra_inputs_getter_right(pattern):
    """get inputs pattern for extra inputs, inputs for root node
    are assumed to be copied over from root node to the fused node
    """
    _, extra_input, _bn_conv = pattern
    return [extra_input]


conv_add_optioins = itertools.product(
    [True, False],  # with_bn
    [torch.add, operator.add],  # add_op
)
```
- **EN**: Key callable entry points in this range include `_fuse_conv_bn_add_right`, `_conv_bn_add_root_node_getter_right`, `_conv_bn_add_extra_inputs_getter_right`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_fuse_conv_bn_add_right`, `_conv_bn_add_root_node_getter_right`, `_conv_bn_add_extra_inputs_getter_right`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 270-294 / 第 270-294 行
```python
for with_bn, add_op in conv_add_optioins:
    if with_bn:
        conv_configs.append(
            BackendPatternConfig()
            ._set_pattern_complex_format(
                (add_op, MatchAllNode, (nn.BatchNorm2d, nn.Conv2d))
            )
            .set_observation_type(observation_type)
            .set_dtype_configs(conv_dtype_configs)
            .set_fuser_method(_fuse_conv_bn_add_right)
            ._set_root_node_getter(_conv_bn_add_root_node_getter_right)
            ._set_extra_inputs_getter(_conv_bn_add_extra_inputs_getter_right)
            .set_fused_module(nni.ConvAdd2d)
        )
    else:
        conv_configs.append(
            BackendPatternConfig()
            ._set_pattern_complex_format((add_op, MatchAllNode, nn.Conv2d))
            .set_observation_type(observation_type)
            .set_dtype_configs(conv_dtype_configs)
            .set_fuser_method(_fuse_conv_add_right)
            ._set_root_node_getter(_conv_add_root_node_getter_right)
            ._set_extra_inputs_getter(_conv_add_extra_inputs_getter_right)
            .set_fused_module(nni.ConvAdd2d)
        )
```
- **EN**: This chunk continues the implementation of `_conv_bn_add_extra_inputs_getter_right`, filling in the details of its control flow or data handling. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段延续了 `_conv_bn_add_extra_inputs_getter_right` 的实现，继续补充其控制流或数据处理细节。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 296-321 / 第 296-321 行
```python
conv_configs.append(
    BackendPatternConfig(nni.ConvAdd2d)
    .set_observation_type(observation_type)
    .set_dtype_configs(conv_dtype_configs)
    .set_root_module(nn.Conv2d)
    .set_reference_quantized_module(nnqr.Conv2d)
)

# (2) Conv2d + Add + Relu

# conv2d Y
#   \   /
#    add
#     \
#     relu


def _fuse_conv_add_relu_left(is_qat, relu, add_pattern):
    add, conv, _ = add_pattern
    return nni.ConvAddReLU2d(conv, add, relu)


def _conv_add_relu_root_node_getter_left(pattern):
    _relu, add_pattern = pattern
    _, conv, _ = add_pattern
    return conv
```
- **EN**: Key callable entry points in this range include `_fuse_conv_add_relu_left`, `_conv_add_relu_root_node_getter_left`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_fuse_conv_add_relu_left`, `_conv_add_relu_root_node_getter_left`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 324-349 / 第 324-349 行
```python
def _conv_add_relu_extra_inputs_getter_left(pattern):
    """get inputs pattern for extra inputs, inputs for root node
    are assumed to be copied over from root node to the fused node
    """
    _relu, add_pattern = pattern
    _, _conv, extra_input = add_pattern
    return [extra_input]


# conv2d
#  \
#  bn   Y
#   \   /
#    add
#     \
#     relu


def _fuse_conv_bn_add_relu_left(is_qat, relu, add_pattern):
    add, bn_conv, _ = add_pattern
    bn, conv = bn_conv
    if is_qat:
        raise NotImplementedError(f"Cannot fuse train modules: {(conv, bn, add, relu)}")
    else:
        fused_conv = nn.utils.fusion.fuse_conv_bn_eval(conv, bn)
        return nni.ConvAddReLU2d(fused_conv, add, relu)
```
- **EN**: Key callable entry points in this range include `_conv_add_relu_extra_inputs_getter_left`, `_fuse_conv_bn_add_relu_left`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_conv_add_relu_extra_inputs_getter_left`, `_fuse_conv_bn_add_relu_left`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 352-383 / 第 352-383 行
```python
def _conv_bn_add_relu_root_node_getter_left(pattern):
    _relu, add_pattern = pattern
    _, bn_conv, _ = add_pattern
    _bn, conv = bn_conv
    return conv


def _conv_bn_add_relu_extra_inputs_getter_left(pattern):
    """get inputs pattern for extra inputs, inputs for root node
    are assumed to be copied over from root node to the fused node
    """
    _relu, add_pattern = pattern
    _, _bn_conv, extra_input = add_pattern
    return [extra_input]


conv_add_relu_left_optioins = itertools.product(
    [True, False],  # with_bn
    [torch.add, operator.add],  # add_op
)

for with_bn, add_op in conv_add_relu_left_optioins:
    if with_bn:
        conv_configs.append(
            BackendPatternConfig()
            ._set_pattern_complex_format(
                (nn.ReLU, (add_op, (nn.BatchNorm2d, nn.Conv2d), MatchAllNode))
            )
            .set_observation_type(observation_type)
            .set_dtype_configs(conv_dtype_configs)
            .set_fuser_method(_fuse_conv_bn_add_relu_left)
            ._set_root_node_getter(_conv_bn_add_relu_root_node_getter_left)
```
- **EN**: Key callable entry points in this range include `_conv_bn_add_relu_root_node_getter_left`, `_conv_bn_add_relu_extra_inputs_getter_left`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_conv_bn_add_relu_root_node_getter_left`, `_conv_bn_add_relu_extra_inputs_getter_left`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 384-414 / 第 384-414 行
```python
            ._set_extra_inputs_getter(_conv_bn_add_relu_extra_inputs_getter_left)
            .set_fused_module(nni.ConvAddReLU2d)
        )
    else:
        conv_configs.append(
            BackendPatternConfig()
            ._set_pattern_complex_format((nn.ReLU, (add_op, nn.Conv2d, MatchAllNode)))
            .set_observation_type(observation_type)
            .set_dtype_configs(conv_dtype_configs)
            .set_fuser_method(_fuse_conv_add_relu_left)
            ._set_root_node_getter(_conv_add_relu_root_node_getter_left)
            ._set_extra_inputs_getter(_conv_add_relu_extra_inputs_getter_left)
            .set_fused_module(nni.ConvAddReLU2d)
        )

#  Y   conv2d
#   \   /
#    add
#     \
#     relu


def _fuse_conv_add_relu_right(is_qat, relu, add_pattern):
    add, _, conv = add_pattern
    return nni.ConvAddReLU2d(conv, add, relu)


def _conv_add_relu_root_node_getter_right(pattern):
    _relu, add_pattern = pattern
    _, _extra_input, conv = add_pattern
    return conv
```
- **EN**: Key callable entry points in this range include `_fuse_conv_add_relu_right`, `_conv_add_relu_root_node_getter_right`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_fuse_conv_add_relu_right`, `_conv_add_relu_root_node_getter_right`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 417-442 / 第 417-442 行
```python
def _conv_add_relu_extra_inputs_getter_right(pattern):
    """get inputs pattern for extra inputs, inputs for root node
    are assumed to be copied over from root node to the fused node
    """
    _relu, add_pattern = pattern
    _, extra_input, _conv = add_pattern
    return [extra_input]


#      conv2d
#        /
#  Y    bn
#   \   /
#    add
#     \
#     relu


def _fuse_conv_bn_add_relu_right(is_qat, relu, add_pattern):
    add, _, bn_conv = add_pattern
    bn, conv = bn_conv
    if is_qat:
        raise NotImplementedError(f"Cannot fuse train modules: {(conv, bn, add, relu)}")
    else:
        fused_conv = nn.utils.fusion.fuse_conv_bn_eval(conv, bn)
        return nni.ConvAddReLU2d(fused_conv, add, relu)
```
- **EN**: Key callable entry points in this range include `_conv_add_relu_extra_inputs_getter_right`, `_fuse_conv_bn_add_relu_right`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_conv_add_relu_extra_inputs_getter_right`, `_fuse_conv_bn_add_relu_right`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 445-476 / 第 445-476 行
```python
def _conv_bn_add_relu_root_node_getter_right(pattern):
    _relu, add_pattern = pattern
    _, _, bn_conv = add_pattern
    _bn, conv = bn_conv
    return conv


def _conv_bn_add_relu_extra_inputs_getter_right(pattern):
    """get inputs pattern for extra inputs, inputs for root node
    are assumed to be copied over from root node to the fused node
    """
    _relu, add_pattern = pattern
    _, extra_input, _bn_conv = add_pattern
    return [extra_input]


conv_add_relu_left_optioins = itertools.product(
    [True, False],  # with_bn
    [torch.add, operator.add],  # add_op
)

for with_bn, add_op in conv_add_relu_left_optioins:
    if with_bn:
        conv_configs.append(
            BackendPatternConfig()
            ._set_pattern_complex_format(
                (nn.ReLU, (add_op, MatchAllNode, (nn.BatchNorm2d, nn.Conv2d)))
            )
            .set_observation_type(observation_type)
            .set_dtype_configs(conv_dtype_configs)
            .set_fuser_method(_fuse_conv_bn_add_relu_right)
            ._set_root_node_getter(_conv_bn_add_relu_root_node_getter_right)
```
- **EN**: Key callable entry points in this range include `_conv_bn_add_relu_root_node_getter_right`, `_conv_bn_add_relu_extra_inputs_getter_right`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_conv_bn_add_relu_root_node_getter_right`, `_conv_bn_add_relu_extra_inputs_getter_right`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 477-502 / 第 477-502 行
```python
            ._set_extra_inputs_getter(_conv_bn_add_relu_extra_inputs_getter_right)
            .set_fused_module(nni.ConvAddReLU2d)
        )
    else:
        conv_configs.append(
            BackendPatternConfig()
            ._set_pattern_complex_format((nn.ReLU, (add_op, MatchAllNode, nn.Conv2d)))
            .set_observation_type(observation_type)
            .set_dtype_configs(conv_dtype_configs)
            .set_fuser_method(_fuse_conv_add_relu_right)
            ._set_root_node_getter(_conv_add_relu_root_node_getter_right)
            ._set_extra_inputs_getter(_conv_add_relu_extra_inputs_getter_right)
            .set_fused_module(nni.ConvAddReLU2d)
        )

conv_configs.append(
    BackendPatternConfig(nni.ConvAddReLU2d)
    .set_observation_type(observation_type)
    .set_dtype_configs(conv_dtype_configs)
    .set_root_module(nn.Conv2d)
    .set_reference_quantized_module(nnqr.Conv2d)
)

# ========================
# |  CONFIGS FOR LINEAR  |
# ========================
```
- **EN**: This chunk continues the implementation of `_conv_bn_add_relu_extra_inputs_getter_right`, filling in the details of its control flow or data handling. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段延续了 `_conv_bn_add_relu_extra_inputs_getter_right` 的实现，继续补充其控制流或数据处理细节。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 504-535 / 第 504-535 行
```python
linear_dtype_configs = [
    onednn_weighted_op_int8_dtype_config,
    onednn_dynamic_int8_dtype_config,
]
linear_configs = _get_linear_configs(linear_dtype_configs)


def _add_eltwise_fusion_configs(
    configs,
    root_module,
    root_op,
    post_module,
    post_op,
    dtype_configs,
    fuser_method,
    fused_module,
    observation_type,
    ref_quant_module,
):
    # 1 base module + op module fusion config
    configs.append(
        BackendPatternConfig((root_module, post_module))
        .set_dtype_configs(dtype_configs)
        .set_fuser_method(fuser_method)
        .set_fused_module(fused_module)
    )
    # base module + functional post op
    configs.append(
        BackendPatternConfig((root_module, post_op))
        .set_dtype_configs(dtype_configs)
        .set_fuser_method(fuser_method)
        .set_fused_module(fused_module)
```
- **EN**: Key callable entry points in this range include `_add_eltwise_fusion_configs`. They package a focused unit of behavior behind named helpers or APIs. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_add_eltwise_fusion_configs`，它们把聚焦的行为封装成具名辅助函数或 API。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 536-557 / 第 536-557 行
```python
    )

    # 2 fused module configs
    configs.append(
        BackendPatternConfig(fused_module)
        .set_observation_type(observation_type)
        .set_dtype_configs(dtype_configs)
        .set_root_module(root_module)
        .set_reference_quantized_module(ref_quant_module)
    )

    # 3 functional base op + post op configs
    configs.append(
        BackendPatternConfig((root_op, post_module))
        .set_observation_type(observation_type)
        .set_dtype_configs(dtype_configs)
    )
    configs.append(
        BackendPatternConfig((root_op, post_op))
        .set_observation_type(observation_type)
        .set_dtype_configs(dtype_configs)
    )
```
- **EN**: Key callable entry points in this range include `_add_eltwise_fusion_configs`. They package a focused unit of behavior behind named helpers or APIs. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_add_eltwise_fusion_configs`，它们把聚焦的行为封装成具名辅助函数或 API。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 560-580 / 第 560-580 行
```python
# Configs for linear + leaky_relu fusion
_add_eltwise_fusion_configs(
    linear_configs,
    nn.Linear,
    F.linear,
    nn.LeakyReLU,
    F.leaky_relu,
    linear_dtype_configs,
    _sequential_wrapper2(nni.LinearLeakyReLU),
    nni.LinearLeakyReLU,
    observation_type,
    nnqr.Linear,
)

# Configs for linear module + batchnorm + leaky_relu
linear_configs.append(
    BackendPatternConfig((nn.Linear, nn.BatchNorm1d, nn.LeakyReLU))
    .set_dtype_configs(linear_dtype_configs)
    .set_fuser_method(_fuse_linear_bn_leaky_relu)
    .set_fused_module(nni.LinearLeakyReLU)
)
```
- **EN**: This chunk continues the implementation of `_add_eltwise_fusion_configs`, filling in the details of its control flow or data handling. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段延续了 `_add_eltwise_fusion_configs` 的实现，继续补充其控制流或数据处理细节。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 582-610 / 第 582-610 行
```python
# Configs for linear + tanh fusion
_add_eltwise_fusion_configs(
    linear_configs,
    nn.Linear,
    F.linear,
    nn.Tanh,
    torch.tanh,
    linear_dtype_configs,
    _sequential_wrapper2(nni.LinearTanh),
    nni.LinearTanh,
    observation_type,
    nnqr.Linear,
)

# ===========================
# |  CONFIGS FOR OTHER OPS  |
# ===========================

binary_op_dtype_configs = [onednn_op_quint8_dtype_config]
default_op_dtype_configs = [onednn_op_quint8_dtype_config]
fixed_qparams_op_dtype_configs = [onednn_op_quint8_dtype_config]
share_qparams_op_dtype_configs = [onednn_op_quint8_dtype_config]
rnn_op_dtype_configs = [onednn_dynamic_int8_dtype_config]
embedding_op_dtype_configs = [onednn_weight_only_qint8_dtype_config]
layer_norm_op_dtype_configs = [onednn_input_output_only_quint8_dtype_config]

# =====================
# |  BACKEND CONFIGS  |
# =====================
```
- **EN**: This chunk continues the implementation of `_add_eltwise_fusion_configs`, filling in the details of its control flow or data handling. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段延续了 `_add_eltwise_fusion_configs` 的实现，继续补充其控制流或数据处理细节。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 613-641 / 第 613-641 行
```python
def get_onednn_backend_config() -> BackendConfig:
    """
    Return the `BackendConfig` for PyTorch's native ONEDNN backend.
    """
    return (
        BackendConfig("onednn")
        .set_backend_pattern_configs(conv_configs)
        .set_backend_pattern_configs(linear_configs)
        .set_backend_pattern_configs(_get_binary_op_configs(binary_op_dtype_configs))
        .set_backend_pattern_config(_get_cat_config(default_op_dtype_configs))
        .set_backend_pattern_configs(_get_default_op_configs(default_op_dtype_configs))
        .set_backend_pattern_configs(
            _get_fixed_qparams_op_configs(fixed_qparams_op_dtype_configs)
        )
        .set_backend_pattern_configs(
            _get_share_qparams_op_configs(share_qparams_op_dtype_configs)
        )
        .set_backend_pattern_configs(_get_bn_configs(default_op_dtype_configs))
        .set_backend_pattern_configs(_get_ln_configs(layer_norm_op_dtype_configs))
        .set_backend_pattern_configs(_get_rnn_op_configs(rnn_op_dtype_configs))
        .set_backend_pattern_configs(
            _get_embedding_op_configs(embedding_op_dtype_configs)
        )
    )


__all__ = [
    "get_onednn_backend_config",
]
```
- **EN**: The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. Key callable entry points in this range include `get_onednn_backend_config`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 这一段的重要可调用入口包括 `get_onednn_backend_config`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements quantization observers, fake-quant modules, preparation/conversion passes, and backend-specific quantization helpers.
  - CN: 实现量化 observer、fake-quant 模块、prepare/convert pass 以及后端相关的量化辅助逻辑。
- **Quantization pipeline**
  - EN: Tracks calibration statistics, quantization parameters, or graph/module rewrites for lower-precision execution.
  - CN: 跟踪校准统计信息、量化参数，或面向低精度执行的图/模块改写。
- **Observer statistics**
  - EN: Collects runtime ranges or histograms so later code can derive quantization settings.
  - CN: 收集运行时范围或直方图，以便后续逻辑推导量化设置。
- **Graph rewriting**
  - EN: Rewrites module or FX graph structure so later execution follows a transformed pipeline.
  - CN: 改写模块或 FX 图结构，使后续执行遵循转换后的流水线。
- **Data loading**
  - EN: Coordinates dataset traversal, batching, multiprocessing workers, and sample collation.
  - CN: 协调数据集遍历、批处理、多进程 worker 以及样本拼装。
- **Configuration management**
  - EN: Centralizes feature flags, environment overrides, and runtime-config wiring.
  - CN: 集中管理特性开关、环境变量覆盖以及运行时配置接线。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch.ao.nn.intrinsic`, `torch.ao.nn.quantized.reference`, `torch.nn`, `torch.nn.functional`, `torch.ao.quantization.fuser_method_mappings:_sequential_wrapper2`, `torch.ao.quantization.utils:MatchAllNode`, `._common_operator_config_utils:_get_binary_op_configs`, `._common_operator_config_utils:_get_bn_configs`, `._common_operator_config_utils:_get_cat_config`, `._common_operator_config_utils:_get_conv_configs`, `._common_operator_config_utils:_get_default_op_configs`, `._common_operator_config_utils:_get_embedding_op_configs`, `._common_operator_config_utils:_get_fixed_qparams_op_configs`, `._common_operator_config_utils:_get_linear_configs`
- **Python standard library / Python 标准库**: `itertools`, `operator`
- **Explicit exports / 显式导出**: `get_onednn_backend_config`
- **Primary symbols / 核心符号**: `_fuse_linear_bn_leaky_relu`, `_fuse_conv_add_left`, `_conv_add_root_node_getter_left`, `_conv_add_extra_inputs_getter_left`, `_fuse_conv_bn_add_left`, `_conv_bn_add_root_node_getter_left`, `_conv_bn_add_extra_inputs_getter_left`, `_fuse_conv_add_right`, `_conv_add_root_node_getter_right`, `_conv_add_extra_inputs_getter_right`, `_fuse_conv_bn_add_right`, `_conv_bn_add_root_node_getter_right`, `_conv_bn_add_extra_inputs_getter_right`, `_fuse_conv_add_relu_left`, `_conv_add_relu_root_node_getter_left`
