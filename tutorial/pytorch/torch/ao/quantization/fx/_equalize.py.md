# _equalize.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/quantization/fx/_equalize.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements quantization observers, fake-quant modules, preparation/conversion passes, and backend-specific quantization helpers. This specific file centers on `_equalize.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior. It also participates in graph/module transformation before or after quantized execution.
- **Purpose (CN) / 用途（中文）**: 实现量化 observer、fake-quant 模块、prepare/convert pass 以及后端相关的量化辅助逻辑。 该文件具体围绕 `_equalize.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。 它也参与量化执行前后的图/模块转换。
## Line-by-Line Analysis / 逐行分析

### Lines 1-42 / 第 1-42 行
```python
# mypy: allow-untyped-defs
import operator
import warnings
from collections import namedtuple
from typing import Any
from typing_extensions import TypeIs

import torch
import torch.ao.nn.intrinsic as nni
import torch.nn as nn
import torch.nn.functional as F
from torch.ao.quantization.fx.graph_module import _get_observed_graph_module_attr
from torch.ao.quantization.observer import (
    _with_args,
    ObserverBase,
    PerChannelMinMaxObserver,
)
from torch.ao.quantization.utils import _parent_name, check_min_max_valid
from torch.fx import GraphModule
from torch.fx.graph import Node

from .utils import (
    get_new_attr_name_with_prefix,
    maybe_get_next_module,
    node_arg_is_weight,
)


CUSTOM_MODULE_SUPP_LIST: list[Any] = []


def reshape_scale(scale: torch.Tensor, axis: int, input: torch.Tensor) -> torch.Tensor:
    """Reshapes the scale so that we can multiply it to the input by the given axis."""
    new_shape = [1] * input.ndim
    new_shape[axis] = input.size(axis)
    return scale.view(new_shape)


qsheme_mapping_per_tensor_to_per_channel = {
    torch.per_tensor_affine: torch.per_channel_affine,
    torch.per_tensor_symmetric: torch.per_channel_symmetric,
}
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch, torch.ao.nn.intrinsic, torch.nn, torch.nn.functional; standard-library helpers such as operator, warnings, collections:namedtuple, typing:Any; external packages such as typing_extensions:TypeIs. Key callable entry points in this range include `reshape_scale`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch, torch.ao.nn.intrinsic, torch.nn, torch.nn.functional；标准库辅助模块，如 operator, warnings, collections:namedtuple, typing:Any；外部包，如 typing_extensions:TypeIs。 这一段的重要可调用入口包括 `reshape_scale`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 45-78 / 第 45-78 行
```python
class _InputEqualizationObserver(nn.Module):
    r"""Observer for tracking the running min/max values of input columns, and
    computing the quantization parameters for the overall min/max input values.

    Args:
        dtype: Quantized data type
        qscheme: Quantization scheme
        quant_min: Minimum quantization value. If unspecified, it will
            follow the 8-bit setup.
        quant_max: Maximum quantization value. If unspecified, it will
            follow the 8-bit setup.

    The running minimum/maximum :math:`x_\text{min/max}` are computed in the
    same way as :class:`~torch.ao.quantization.observer.PerChannelMinMaxObserver`,
    with the difference that the running min/max values are stored per column.
    This observer is intended to be used along with a WeightEqualizationObserver
    to calculate the equalization scale.
    """

    def __init__(
        self,
        dtype=torch.quint8,
        qscheme=torch.per_tensor_affine,
        quant_min=None,
        quant_max=None,
        factory_kwargs=None,
    ) -> None:
        super().__init__()

        if qscheme not in {torch.per_tensor_affine, torch.per_tensor_symmetric}:
            raise TypeError("Input qscheme must be per-tensor")

        self.dtype = dtype
        self.qscheme = qscheme
```
- **EN**: It introduces or extends class-level abstractions such as `_InputEqualizationObserver`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `_InputEqualizationObserver` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 80-115 / 第 80-115 行
```python
        per_channel_qscheme = qsheme_mapping_per_tensor_to_per_channel[qscheme]
        self.input_obs = PerChannelMinMaxObserver(
            ch_axis=1,
            dtype=dtype,
            qscheme=per_channel_qscheme,
            quant_min=quant_min,
            quant_max=quant_max,
            factory_kwargs=factory_kwargs,
        )

        self.equalization_scale = torch.tensor(1)
        self.equalization_shape: list[int] = []

    def forward(self, x_orig):
        if x_orig.ndim < 2 or x_orig.ndim > 5:
            raise ValueError(
                "InputEqualizationObserver only supports Linear and Conv layers"
            )

        # Calculate the shape needed to reshape the equalization scale later (needed for Conv layers)
        self.equalization_shape = [1] * x_orig.ndim
        self.equalization_shape[1] = x_orig.size(1)

        return self.input_obs(x_orig)

    def get_input_minmax(self):
        return (self.input_obs.min_val, self.input_obs.max_val)

    def set_equalization_scale(self, equalization_scale):
        # Reshape the equalization scale along axis=1 so that it can be
        # multiplied with the input along axis=1
        if equalization_scale.nelement() == 1 and equalization_scale == torch.tensor(1):
            return
        self.equalization_scale = torch.reshape(
            equalization_scale, self.equalization_shape
        )
```
- **EN**: It introduces or extends class-level abstractions such as `_InputEqualizationObserver`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `_InputEqualizationObserver` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 117-155 / 第 117-155 行
```python
    def calculate_scaled_minmax(self):
        r"""Returns the scaled min/max inputs"""
        if (
            self.equalization_scale.nelement() == 1
            and self.equalization_scale == torch.tensor(1)
        ):
            warnings.warn(
                "Must call calculate_equalization_scale before calling calculate_scaled_minmax. "
                + "Will not scale the next quantization observer.",
                stacklevel=2,
            )
            return None, None

        # Calculate qparams for the scaled min/max inputs
        # Scale the input by the equalization scale located at the same column
        # index
        (min_inputs, max_inputs) = self.get_input_minmax()
        equalization_scale_reshaped = reshape_scale(
            self.equalization_scale, 0, min_inputs
        )
        min_input_scaled = torch.min(torch.mul(min_inputs, equalization_scale_reshaped))
        max_input_scaled = torch.max(torch.mul(max_inputs, equalization_scale_reshaped))

        return min_input_scaled, max_input_scaled

    with_args = classmethod(_with_args)


class _WeightEqualizationObserver(nn.Module):
    r"""Observer for tracking the running min/max values of weight columns and
    rows, and computing the quantization parameters for the weight rows.

    Args:
        dtype: Quantized data type
        qscheme: Quantization scheme
        quant_min: Minimum quantization value. If unspecified, it will
            follow the 8-bit setup.
        quant_max: Maximum quantization value. If unspecified, it will
            follow the 8-bit setup.
```
- **EN**: It introduces or extends class-level abstractions such as `_InputEqualizationObserver`, `_WeightEqualizationObserver`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `_InputEqualizationObserver`, `_WeightEqualizationObserver` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 157-198 / 第 157-198 行
```python
    This observer is made up of 1 PerChannelMinMaxObserver `weight_col_obs` used
    to record the running minimum and maximum of columns of incoming weight
    tensors. This observer is intended to be used along with an
    InputEqualizationObserver to calculate the equalization scale.

    The running minimum/maximum :math:`w_\text{min/max}` are computed in the
    same way as :class:`~torch.ao.quantization.observer.PerChannelMinMaxObserver`.
    """

    def __init__(
        self,
        dtype=torch.qint8,
        qscheme=torch.per_tensor_affine,
        quant_min=None,
        quant_max=None,
        factory_kwargs=None,
    ) -> None:
        super().__init__()

        self.dtype = dtype
        self.qscheme = qscheme
        self.ch_axis = 1

        per_channel_qscheme = qscheme
        if qscheme in {torch.per_tensor_affine, torch.per_tensor_symmetric}:
            per_channel_qscheme = qsheme_mapping_per_tensor_to_per_channel[qscheme]
        self.weight_col_obs = PerChannelMinMaxObserver(
            ch_axis=1,
            dtype=dtype,
            qscheme=per_channel_qscheme,
            quant_min=quant_min,
            quant_max=quant_max,
            factory_kwargs=factory_kwargs,
        )

        self.equalization_scale = torch.tensor(1)

    def forward(self, w_orig):
        if w_orig.ndim < 2 or w_orig.ndim > 5:
            raise ValueError(
                "InputEqualizationObserver only supports Linear and Conv layers"
            )
```
- **EN**: It introduces or extends class-level abstractions such as `_WeightEqualizationObserver`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `_WeightEqualizationObserver` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 200-240 / 第 200-240 行
```python
        return self.weight_col_obs(w_orig)

    def get_weight_col_minmax(self):
        return (self.weight_col_obs.min_val, self.weight_col_obs.max_val)

    def set_equalization_scale(self, equalization_scale):
        self.equalization_scale = equalization_scale

    with_args = classmethod(_with_args)


def calculate_equalization_scale(
    input_obs: _InputEqualizationObserver, weight_obs: _WeightEqualizationObserver
) -> torch.Tensor:
    r"""Calculates the equalization scale and sets the equalization_scale value
    in the observers.

    Args:
        input_obs: Observer that tracks the ranges for the input columns
        weight_obs: Observer that tracks the ranges for the weight columns
    """

    (min_inputs, max_inputs) = input_obs.get_input_minmax()
    (min_weights, max_weights) = weight_obs.get_weight_col_minmax()

    if not (
        check_min_max_valid(min_inputs, max_inputs)
        and check_min_max_valid(min_weights, max_weights)
    ):
        warnings.warn(
            "Must run observer before calling calculate_equalization_scale. "
            + "Returning default equalization scale torch.tensor(1).",
            stacklevel=2,
        )
        return torch.tensor(1)

    if min_inputs.shape != min_weights.shape:
        raise ValueError(
            "Input and Weight must have the same column dimension. "
            + f"Found {min_inputs.shape} and {min_weights.shape} shapes instead."
        )
```
- **EN**: It introduces or extends class-level abstractions such as `_WeightEqualizationObserver`, which organize state and behavior for this subsystem. Key callable entry points in this range include `calculate_equalization_scale`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `_WeightEqualizationObserver` 等类级抽象，用于组织该子系统的状态与行为。 这一段的重要可调用入口包括 `calculate_equalization_scale`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 242-282 / 第 242-282 行
```python
    equalization_scale = torch.sqrt(
        (max_weights - min_weights) / (max_inputs - min_inputs)
    )
    # Replace all 'inf', 'nan', 0's with 1s to prevent errors
    equalization_scale[equalization_scale == 0.0] = 1
    equalization_scale = torch.nan_to_num(equalization_scale, nan=1, posinf=1, neginf=1)
    return equalization_scale


class EqualizationQConfig(
    # pyrefly: ignore [invalid-inheritance]
    namedtuple("EqualizationQConfig", ["input_activation", "weight"])
):
    """
    Describes how to quantize a layer or a part of the network specifically for
    input-weight equalization by providing settings (observer classes) for
    inputs, outputs, and weights.

    Note that EqualizationQConfig needs to contain observer **classes** (like
    MinMaxObserver) or a callable that returns instances on invocation, not the
    concrete observer instances themselves.
    Quantization function will instantiate observers multiple times for each of
    the layers.

    Observer classes have usually reasonable default arguments, but they can be
    overwritten with `with_args` method (that behaves like functools.partial):

    my_qconfig = EqualizationQConfig(input_activation=_InputEqualizationObserver.with_args(dtype=torch.qint8),
                                    weight=_WeightEqualizationObserver.with_args(dtype=torch.qint8))
    """

    __slots__ = ()

    def __new__(cls, input_activation=torch.nn.Identity, weight=torch.nn.Identity):
        if isinstance(input_activation, nn.Module) or isinstance(weight, nn.Module):
            raise ValueError(
                "EqualizationQConfig received observer instance, please pass observer class instead. "
                + "Use MyObserver.with_args(x=1) to override arguments to constructor if needed"
            )
        self = super().__new__(cls, input_activation, weight)
        return self
```
- **EN**: It introduces or extends class-level abstractions such as `EqualizationQConfig`, which organize state and behavior for this subsystem. Key callable entry points in this range include `calculate_equalization_scale`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `EqualizationQConfig` 等类级抽象，用于组织该子系统的状态与行为。 这一段的重要可调用入口包括 `calculate_equalization_scale`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 285-313 / 第 285-313 行
```python
input_equalization_observer = _InputEqualizationObserver.with_args(
    dtype=torch.quint8, qscheme=torch.per_tensor_symmetric
)
weight_equalization_observer = _WeightEqualizationObserver.with_args(
    dtype=torch.qint8, qscheme=torch.per_channel_symmetric
)
default_equalization_qconfig = EqualizationQConfig(
    input_activation=input_equalization_observer, weight=weight_equalization_observer
)


def fused_module_supports_equalization(module) -> bool:
    """Checks if the fused node supports equalization."""
    return type(module) in [
        nni.LinearReLU,
        nni.ConvReLU1d,
        nni.ConvReLU2d,
        nni.ConvReLU3d,
    ]


def nn_module_supports_equalization(module) -> bool:
    """Checks if the torch.nn node supports equalization."""
    return type(module) in [nn.Linear, nn.Conv1d, nn.Conv2d, nn.Conv3d]


def custom_module_supports_equalization(module) -> bool:
    """Checks if the custom node supports equalization."""
    return type(module) in CUSTOM_MODULE_SUPP_LIST
```
- **EN**: Key callable entry points in this range include `fused_module_supports_equalization`, `nn_module_supports_equalization`, `custom_module_supports_equalization`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `fused_module_supports_equalization`, `nn_module_supports_equalization`, `custom_module_supports_equalization`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 316-352 / 第 316-352 行
```python
def node_supports_equalization(node: Node, modules) -> bool:
    """Checks if the current node supports equalization
    Currently we only support nn.Linear/F.Linear and nn.Conv/F.conv layers
    """
    if node.op == "call_module":
        return (
            nn_module_supports_equalization(modules[str(node.target)])
            or fused_module_supports_equalization(modules[str(node.target)])
            or custom_module_supports_equalization(modules[str(node.target)])
        )
    elif node.op == "call_function":
        return node.target in [F.linear, F.conv1d, F.conv2d, F.conv3d]
    return False


def is_equalization_observer(
    observer: nn.Module,
) -> TypeIs[_InputEqualizationObserver | _WeightEqualizationObserver]:
    return isinstance(
        observer, (_InputEqualizationObserver, _WeightEqualizationObserver)
    )


###############################################################################
# Functions for equalization during convert                                   #
###############################################################################


def get_op_node_and_weight_eq_obs(
    input_eq_obs_node: Node, model: GraphModule, modules: dict[str, nn.Module]
) -> tuple[Node | None, _WeightEqualizationObserver | None]:
    """Gets the following weight equalization observer. There should always
    exist a weight equalization observer after an input equalization observer.

    Returns the operation node that follows the input equalization observer node
    and the weight equalization observer
    """
```
- **EN**: Key callable entry points in this range include `node_supports_equalization`, `is_equalization_observer`, `get_op_node_and_weight_eq_obs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `node_supports_equalization`, `is_equalization_observer`, `get_op_node_and_weight_eq_obs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 354-388 / 第 354-388 行
```python
    # Find the op node that comes directly after the input equalization observer
    op_node = None
    for user in input_eq_obs_node.users:
        if node_supports_equalization(user, modules):
            op_node = user
            break

    if op_node is None:
        raise AssertionError(
            "Expected an operation node after the input equalization observer"
        )
    if op_node.op == "call_module":
        # If the op_node is a nn.Linear layer, then it must have a
        # WeightEqualizationObserver configuration
        maybe_equalization_node_name_to_config = _get_observed_graph_module_attr(
            model, "equalization_node_name_to_qconfig"
        )
        if maybe_equalization_node_name_to_config is None:
            raise AssertionError(
                "Expected 'equalization_node_name_to_qconfig' attribute in observed graph module"
            )
        equalization_node_name_to_qconfig: dict[str, Any] = (
            maybe_equalization_node_name_to_config  # type: ignore[assignment]
        )
        if equalization_node_name_to_qconfig.get(op_node.name) is None:
            raise AssertionError(
                f"No equalization qconfig found for op node {op_node.name}"
            )
        weight_eq_obs = equalization_node_name_to_qconfig.get(op_node.name).weight()  # type: ignore[union-attr]

        if not isinstance(weight_eq_obs, _WeightEqualizationObserver):
            raise AssertionError(
                "Expected weight equalization observer to be a _WeightEqualizationObserver"
            )
        return op_node, weight_eq_obs
```
- **EN**: Key callable entry points in this range include `get_op_node_and_weight_eq_obs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_op_node_and_weight_eq_obs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 390-432 / 第 390-432 行
```python
    elif op_node.op == "call_function":
        weight_node = maybe_get_weight_eq_obs_node(op_node, modules)
        if weight_node is not None:
            weight_eq_obs = modules[str(weight_node.target)]
            if not isinstance(weight_eq_obs, _WeightEqualizationObserver):
                raise AssertionError(
                    "Expected weight equalization observer to be a _WeightEqualizationObserver"
                )
            return op_node, weight_eq_obs

    return None, None


def maybe_get_weight_eq_obs_node(
    op_node: Node, modules: dict[str, nn.Module]
) -> Node | None:
    """Gets the weight equalization observer node if it exists."""
    if op_node.op != "call_function":
        raise AssertionError(
            "maybe_get_weight_eq_obs_node expects a call_function op_node"
        )
    for node_arg in op_node.args:
        if node_arg_is_weight(op_node, node_arg):
            if (
                isinstance(node_arg, Node)
                and node_arg.op == "call_module"
                and isinstance(
                    modules[str(node_arg.target)], _WeightEqualizationObserver
                )
            ):
                return node_arg
    return None


def maybe_get_next_input_eq_obs(
    node: Node, modules: dict[str, nn.Module]
) -> _InputEqualizationObserver | None:
    """Gets the following input equalization observer if it exists.

    For example, in the case of connecting linear layers:
        x -> inp_obs1 -> eq_obs1 -> linear1 -> out_obs1 -> eq_obs2 -> linear2 -> out_obs2
    If the node being passed in is the linear1 node, then we want to return eq_obs2,
    the following equalization observer for linear2.
```
- **EN**: Key callable entry points in this range include `get_op_node_and_weight_eq_obs`, `maybe_get_weight_eq_obs_node`, `maybe_get_next_input_eq_obs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_op_node_and_weight_eq_obs`, `maybe_get_weight_eq_obs_node`, `maybe_get_next_input_eq_obs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 434-475 / 第 434-475 行
```python
    However, if there are no connecting layers:
        x -> inp_obs1 -> eq_obs1 -> linear1 -> out_obs1 -> add
    Then we want to return None.

    In the case of an unfused linear-relu layer with a connecting linear layer:
        linear1 -> relu -> out_obs1 -> eq_obs2 -> linear2 -> out_obs2
    Since it is unfused, we want to skip over the relu layer and return eq_obs2,
    the following equalization observer for linear2.
    """

    if not node_supports_equalization(node, modules):
        raise AssertionError("Node does not support equalization")

    # Locate the following nn.ReLU or F.relu node if it exists
    maybe_relu_node = maybe_get_next_module(node, modules, nn.ReLU)
    if maybe_relu_node is None:
        maybe_relu_node = maybe_get_next_module(
            node, modules, target_functional_type=F.relu
        )

    # Locate the following output observer if it exists.
    # We will skip the relu node if it exists.
    maybe_obs_node = (
        maybe_get_next_module(node, modules, ObserverBase)
        if maybe_relu_node is None
        else maybe_get_next_module(maybe_relu_node, modules, ObserverBase)
    )
    if maybe_obs_node is None:
        return None

    maybe_eq_obs_node = maybe_get_next_module(
        maybe_obs_node, modules, _InputEqualizationObserver
    )
    if maybe_eq_obs_node is None:
        return None

    maybe_eq_obs = modules[str(maybe_eq_obs_node)]
    if not isinstance(maybe_eq_obs, _InputEqualizationObserver):
        raise AssertionError(
            "Expected the following equalization observer to be an _InputEqualizationObserver"
        )
    return maybe_eq_obs
```
- **EN**: Key callable entry points in this range include `maybe_get_next_input_eq_obs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `maybe_get_next_input_eq_obs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 478-519 / 第 478-519 行
```python
def maybe_get_next_equalization_scale(
    node: Node, modules: dict[str, nn.Module]
) -> torch.Tensor | None:
    """If the next next node is an InputEqualizationObserver then we want to
    return its equalization scale, else we return 1

    This is used in the case where there are two connecting linear layers:
        linear1 -> LinearOutObs -> InputEqObs -> linear2
    In this case, the node given is linear1 and we want to locate the InputEqObs.
    """
    next_inp_eq_obs = maybe_get_next_input_eq_obs(node, modules)

    if next_inp_eq_obs:
        if (
            next_inp_eq_obs.equalization_scale.nelement() == 1
            and next_inp_eq_obs.equalization_scale == torch.tensor(1)
        ):
            return None
        return next_inp_eq_obs.equalization_scale
    return None


def scale_input_observer(node: Node, modules: dict[str, nn.Module]) -> None:
    """Scales the following input quantization observer's min/max values by
    updating the values with the scaled min/max values calculated by the input
    equalization observer
    """
    input_eq_obs = modules[str(node.target)]
    if not isinstance(input_eq_obs, _InputEqualizationObserver):
        raise AssertionError(
            "Expected the module at node.target to be an _InputEqualizationObserver"
        )

    input_quant_obs_node = node.args[0]
    if not isinstance(input_quant_obs_node, Node):
        raise AssertionError(
            "Expected the input quantization observer node to be a Node"
        )

    input_quant_obs = modules[str(input_quant_obs_node.target)]
    if not isinstance(input_quant_obs, ObserverBase):
        return
```
- **EN**: Key callable entry points in this range include `maybe_get_next_equalization_scale`, `scale_input_observer`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `maybe_get_next_equalization_scale`, `scale_input_observer`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 521-562 / 第 521-562 行
```python
    min_input_scaled, max_input_scaled = input_eq_obs.calculate_scaled_minmax()
    if min_input_scaled is None and max_input_scaled is None:
        return
    input_quant_obs.min_val = min_input_scaled
    input_quant_obs.max_val = max_input_scaled


def scale_weight_node(
    node: Node,
    modules: dict[str, nn.Module],
    equalization_scale: torch.Tensor,
    next_equalization_scale: torch.Tensor | None,
) -> None:
    """Scale the weights for input-weight equalization by multiplying the
    weight by 1/equalization_scale and next_equalization_scale

    Args:
        node: Current node whose weights we want to scale
        equalization_scale: Current node's calculated equalization scale
        next_equalization_scale: Next node's calculated equalization scale if
           the following node needs to be equalized, 1 otherwise
    """
    if equalization_scale is None:
        return

    if fused_module_supports_equalization(modules[str(node.target)]):
        op_module = modules[str(node.target)][0]  # type: ignore[index]
    else:
        op_module = modules[str(node.target)]
    if not (
        nn_module_supports_equalization(op_module)
        or custom_module_supports_equalization(op_module)
    ):
        raise AssertionError(
            "Expected operation module to support equalization (nn or custom)"
        )

    # Scale the weights for input-weight equalization
    # If the following layer needs to be equalized then we will multiply its scale
    weight = op_module.weight
    if not isinstance(weight, torch.Tensor):
        raise AssertionError("Expected op_module.weight to be a torch.Tensor")
```
- **EN**: Key callable entry points in this range include `scale_input_observer`, `scale_weight_node`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `scale_input_observer`, `scale_weight_node`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 564-602 / 第 564-602 行
```python
    # Scale the weights by the reciprocal of the equalization scale
    # Reshape the equalization scale so that we can multiply it to the weight along axis=1
    equalization_scale_reshaped = reshape_scale(equalization_scale, 1, weight)
    scaled_weight = torch.mul(weight, torch.reciprocal(equalization_scale_reshaped))

    if next_equalization_scale is None:
        op_module.weight = nn.Parameter(scaled_weight)
        return

    # Multiply the weights row wise by the next equalization scale
    # Reshape the equalization scale so that we can multiply it to the weight along axis=0
    next_equalization_scale_reshaped = reshape_scale(next_equalization_scale, 0, weight)
    scaled_weight = torch.mul(scaled_weight, next_equalization_scale_reshaped)

    op_module.weight = nn.Parameter(scaled_weight)

    # Multiply the bias element wise by the next equalization scale
    bias = op_module.bias
    if bias is None:
        return
    if not isinstance(bias, torch.Tensor):
        raise AssertionError("Expected op_module.bias to be a torch.Tensor")

    # Reshape the equalization scale so that we can multiply it element-wise to the bias
    next_equalization_scale_reshaped = reshape_scale(next_equalization_scale, 0, bias)
    scaled_bias = torch.mul(bias, next_equalization_scale_reshaped)
    op_module.bias = nn.Parameter(scaled_bias)


def scale_weight_functional(
    op_node: Node,
    model: GraphModule,
    modules: dict[str, nn.Module],
    equalization_scale: torch.Tensor,
    next_equalization_scale: torch.Tensor | None,
) -> None:
    """Scales the weight value for functional layers"""
    if equalization_scale is None:
        return
```
- **EN**: Key callable entry points in this range include `scale_weight_node`, `scale_weight_functional`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `scale_weight_node`, `scale_weight_functional`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 604-645 / 第 604-645 行
```python
    # From the given op_node, the path looks like:
    #   get_attr(weight) -> weight_quant_obs -> weight_eq_obs -> op_node
    # So we want to trace back from the op_node to get the equalization observer
    # node, then the quantization observer node, and then finally the weight
    # node which contains the weight values.

    # Get the equalization observer node
    weight_eq_obs_node = maybe_get_weight_eq_obs_node(op_node, modules)
    if weight_eq_obs_node is None:
        return

    # Get the quantization observer node
    weight_quant_obs_node = weight_eq_obs_node.args[0]
    if weight_quant_obs_node is None:
        return
    if not (
        isinstance(weight_quant_obs_node, Node)
        and isinstance(modules[str(weight_quant_obs_node.target)], ObserverBase)
    ):
        raise AssertionError(
            "Expected weight_quant_obs_node to be a Node whose module is an ObserverBase"
        )

    # Get the get_attr(weight) node
    weight_node = weight_quant_obs_node.args[0]
    if weight_node is None:
        return
    if not (isinstance(weight_node, Node) and weight_node.op == "get_attr"):
        raise AssertionError("Expected weight node to be a 'get_attr' Node")

    weight_parent_name, weight_name = _parent_name(weight_node.target)
    weight = getattr(modules[weight_parent_name], weight_name)

    # Scale the weights for input-weight equalization
    # If the following layer needs to be equalized then we will multiply its scale
    # Reshape the equalization scale so that we can multiply it to the weight along axis=1
    equalization_scale_reshaped = reshape_scale(equalization_scale, 1, weight)
    scaled_weight = torch.mul(weight, torch.reciprocal(equalization_scale_reshaped))

    if next_equalization_scale is None:
        setattr(modules[weight_parent_name], weight_name, scaled_weight)
        return
```
- **EN**: Key callable entry points in this range include `scale_weight_functional`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `scale_weight_functional`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 647-689 / 第 647-689 行
```python
    # Multiply the weights row wise by the next equalization scale
    # Reshape the equalization scale so that we can multiply it to the weight along axis=1
    next_equalization_scale_reshaped = reshape_scale(
        next_equalization_scale, 0, scaled_weight
    )
    scaled_weight = torch.mul(scaled_weight, next_equalization_scale_reshaped)

    setattr(modules[weight_parent_name], weight_name, scaled_weight)
    if not torch.allclose(model.get_buffer(str(weight_node.target)), scaled_weight):
        raise AssertionError("Model buffer for weight does not match the scaled weight")

    # Multiply the bias element wise by the next equalization scale
    bias_node = None
    for node in op_node.args:
        # Find the node containing the weight values
        if isinstance(node, Node) and node.op == "get_attr" and "bias" in node.name:
            bias_node = node
            break
    if bias_node is None:
        return

    bias_parent_name, bias_name = _parent_name(bias_node.target)
    bias = getattr(modules[bias_parent_name], bias_name)

    # Reshape the equalization scale so that we can multiply it element-wise to the bias
    next_equalization_scale_reshaped = reshape_scale(next_equalization_scale, 0, bias)
    scaled_bias = torch.mul(bias, next_equalization_scale_reshaped)
    setattr(modules[bias_parent_name], bias_name, scaled_bias)


def clear_weight_quant_obs_node(op_node: Node, modules: dict[str, nn.Module]) -> None:
    """Given the operation node, we want find the corresponding quantization
    observer and reset its min/max values
    """
    weight_eq_obs_node = maybe_get_weight_eq_obs_node(op_node, modules)
    if weight_eq_obs_node is None:
        return

    weight_quant_obs_node = weight_eq_obs_node.args[0]
    if weight_quant_obs_node is None:
        return
    if not isinstance(weight_quant_obs_node, Node):
        raise AssertionError("Expected weight_quant_obs_node to be a Node")
```
- **EN**: Key callable entry points in this range include `scale_weight_functional`, `clear_weight_quant_obs_node`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `scale_weight_functional`, `clear_weight_quant_obs_node`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 691-719 / 第 691-719 行
```python
    weight_quant_obs = modules[str(weight_quant_obs_node.target)]
    if not isinstance(modules[str(weight_quant_obs_node.target)], ObserverBase):
        raise AssertionError(
            "Expected the module at weight_quant_obs_node to be an ObserverBase"
        )
    weight_quant_obs.reset_min_max_vals()  # type: ignore[operator]


def remove_node(model: GraphModule, node: Node, prev_node: Node):
    """Removes the given node from the model by replacing all of its users with
    the given previous node
    """
    # For all of the current node's users, replace the current node with
    # the input quantization observer node
    orig_users = list(node.users.keys())
    for user_node in orig_users:
        user_node.replace_input_with(node, prev_node)

    # Erase the InputEqualizationObserver node
    model.graph.erase_node(node)


def update_obs_for_equalization(
    model: GraphModule, modules: dict[str, nn.Module]
) -> dict[str, _WeightEqualizationObserver]:
    """Update all of the observer's equalization scale. For each
    InputEqualizationObserver, we will find the location of the next
    WeightEqualizationObserver, create it, and calculate the equalization scale
    based on the two observers.
```
- **EN**: Key callable entry points in this range include `clear_weight_quant_obs_node`, `remove_node`, `update_obs_for_equalization`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `clear_weight_quant_obs_node`, `remove_node`, `update_obs_for_equalization`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 721-761 / 第 721-761 行
```python
    We will then return a dictionary mapping operation node names to
    the corresponding WeightEqualizationObservers for that operation.
    """
    weight_eq_obs_dict = {}
    for node in model.graph.nodes:
        if node.op == "call_module" and isinstance(
            modules[node.target], _InputEqualizationObserver
        ):
            input_eq_obs = modules[node.target]
            if not isinstance(input_eq_obs, _InputEqualizationObserver):
                raise AssertionError(
                    "Expected module at node.target to be an _InputEqualizationObserver"
                )
            op_node, weight_eq_obs = get_op_node_and_weight_eq_obs(node, model, modules)

            if op_node is None or weight_eq_obs is None:
                continue

            if op_node.op == "call_module":
                # Calibrate the weight equalization observer since it has just
                # been created
                if fused_module_supports_equalization(modules[str(op_node.target)]):
                    module = modules[str(op_node.target)][0]  # type: ignore[index]
                    if not nn_module_supports_equalization(module):
                        raise AssertionError(
                            "Expected fused module to support equalization"
                        )
                    weight_eq_obs(module.weight)
                else:
                    weight_eq_obs(modules[str(op_node.target)].weight)

            # Calculate and set the equalization scale values
            equalization_scale = calculate_equalization_scale(
                input_eq_obs, weight_eq_obs
            )
            input_eq_obs.set_equalization_scale(equalization_scale)
            weight_eq_obs.set_equalization_scale(equalization_scale)

            weight_eq_obs_dict[op_node.name] = weight_eq_obs

    return weight_eq_obs_dict
```
- **EN**: Key callable entry points in this range include `update_obs_for_equalization`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `update_obs_for_equalization`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 764-805 / 第 764-805 行
```python
def convert_eq_obs(
    model: GraphModule,
    modules: dict[str, nn.Module],
    weight_eq_obs_dict: dict[str, _WeightEqualizationObserver],
) -> None:
    """Converts the equalization operations and updates the other nodes in the
    following way:
        - Removes the input equalization observers and inserts a mul operator
          along with an equalization scale node wherever applicable (we do not
          want to insert a mul operator between connecting linear layers).
        - Updates the input quantization observers with the scaled input min/max
          values.
        - Scales the weights by the current and next equalization scales.
        - Removes the weight equalization observer node if it exists.

    Before (after prepare):
                                    weight values
                                          |
                                    WeightQuantObs
                                          |
                                      WeightEqObs
                                          |
        x -> InpQuantObs -> InpEqObs -> linear -> OutQuantObs

    After this function:
                                              scaled weight values
                                                      |
       equalization scale                       WeightQuantObs
              |                                       |
        x -> mul -> InpQuantObs (scaled min/max) -> linear -> OutQuantObs

    After convert:
       equalization scale                 scaled weight values
              |                                    |
        x -> mul -> quantize_per_tensor -> quantized::linear

    Note that although the equalization observer appeared after the quantization
    observer after prepare_fx, the mul node appears before the quantization node
    after convert_fx. This is because placing the equalization observer after
    the quantization observer in prepare_fx would allow us to keep the invariant
    that the graph before the current node inserts its observers is not
    modified.
```
- **EN**: Key callable entry points in this range include `convert_eq_obs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `convert_eq_obs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 807-841 / 第 807-841 行
```python
    Having the equalization observer before the quantization observer would also
    cause some inconsistences between the ordering of the quantization and
    equalization observers.
    For example, a single linear layer would look like:
        x -> InpEqObs1 -> InpQuantObs1 -> linear1 -> OutQuantObs1
    But between two connected linear layers, it would look like:
        linear1 -> OutQuantObs1 -> InpEqObs2 -> linear2 -> OutQuantObs2
    """
    for node in model.graph.nodes:
        if node.op == "call_module" and isinstance(
            modules[node.target], _InputEqualizationObserver
        ):
            inp_quant_obs_node = node.args[0]
            prev_node = inp_quant_obs_node.args[0]

            # If the previous node is a layer that needs to be equalized, then
            # we will remove the current node because we do not need to add any
            # equalization nodes between two layers that need to be equalized

            # Before: linear1/relu (prev_node) -> output_quant_obs1 (inp_quant_obs_node) -> input_eq_obs2 (node) -> linear2
            # After: linear1/relu (prev_node) -> output_quant_obs1 (inp_quant_obs_node) -> linear2
            if (
                node_supports_equalization(prev_node, modules)
                or "relu" in prev_node.name
            ):
                remove_node(model, node, inp_quant_obs_node)
                continue

            # Update the following input quantization observer's min/max values
            scale_input_observer(node, modules)

            # Remove the InputEqualization node and add a mul operator before
            # the quantization observer node that appears before the equalization node
            # Before: x -> input_quant_obs -> input_eq_obs -> linear
            # After: x -> mul -> input_quant_obs -> linear
```
- **EN**: Key callable entry points in this range include `convert_eq_obs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `convert_eq_obs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 843-877 / 第 843-877 行
```python
            # Create a node containing the equalization scale
            with model.graph.inserting_before(inp_quant_obs_node):
                get_new_eq_scale_name = get_new_attr_name_with_prefix(
                    prev_node.name + "_equalization_scale"
                )
                name = get_new_eq_scale_name(modules)
                setattr(model, name, modules[node.target].equalization_scale)
                eq_scale_node = model.graph.create_node("get_attr", name)

            # Create a node multiplying the input with the equalization scale
            with model.graph.inserting_after(eq_scale_node):
                inputs = (prev_node, eq_scale_node)
                mul_node = model.graph.create_node("call_function", torch.mul, inputs)

            # Set the mul nod to be the input_quant_obs_node's input instead of
            # the previous node
            inp_quant_obs_node.replace_input_with(prev_node, mul_node)
            remove_node(model, node, inp_quant_obs_node)

        elif weight_eq_obs_dict.get(node.name) is not None:
            weight_eq_obs = weight_eq_obs_dict.get(node.name)
            if not isinstance(weight_eq_obs, _WeightEqualizationObserver):
                raise AssertionError(
                    "Expected weight equalization observer to be a _WeightEqualizationObserver"
                )
            equalization_scale = weight_eq_obs.equalization_scale

            if (
                equalization_scale.nelement() == 1
                and equalization_scale == torch.tensor(1)
            ):
                equalization_scale = None  # type: ignore[assignment]
            maybe_next_equalization_scale = maybe_get_next_equalization_scale(
                node, modules
            )
```
- **EN**: Key callable entry points in this range include `convert_eq_obs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `convert_eq_obs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 879-919 / 第 879-919 行
```python
            # Scale the weight nodes
            if node.op == "call_module":
                scale_weight_node(
                    node,
                    modules,
                    # pyrefly: ignore [bad-argument-type]
                    equalization_scale,
                    maybe_next_equalization_scale,
                )
            elif node.op == "call_function":
                scale_weight_functional(
                    node,
                    model,
                    modules,
                    # pyrefly: ignore [bad-argument-type]
                    equalization_scale,
                    maybe_next_equalization_scale,
                )

                weight_eq_obs_node = maybe_get_weight_eq_obs_node(node, modules)
                if weight_eq_obs_node is None:
                    return
                if not isinstance(
                    modules[str(weight_eq_obs_node.target)], _WeightEqualizationObserver
                ):
                    raise AssertionError(
                        "Expected weight equalization observer to be a _WeightEqualizationObserver"
                    )

                # Clear the quantization observer's min/max values so that they
                # can get updated later based on the new scale values
                clear_weight_quant_obs_node(node, modules)

                # Erase the weight equalization observer node
                prev_node = weight_eq_obs_node.args[0]
                remove_node(model, weight_eq_obs_node, prev_node)  # type: ignore[arg-type]
            else:
                raise ValueError(
                    "Expected operation node to be 'call_module' or 'call_function"
                    + f"Instead got node {node.name} as '{node.op}'."
                )
```
- **EN**: Key callable entry points in this range include `convert_eq_obs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `convert_eq_obs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 922-961 / 第 922-961 行
```python
def _convert_equalization_ref(model: GraphModule):
    """Reference function which applies changes needed for equalization, but
    does not quantize the nodes
    """
    modules = dict(model.named_modules(remove_duplicate=False))

    # Calculate the equalization scale, update the observers with the scaled
    # inputs, and scale the weight
    weight_eq_obs_dict = update_obs_for_equalization(model, modules)
    convert_eq_obs(model, modules, weight_eq_obs_dict)

    return GraphModule(model, model.graph)


###############################################################################
# Functions for running the equalized model on the Numeric Suite              #
###############################################################################


def get_layer_sqnr_dict(
    model_a: nn.Module, model_b: nn.Module, x: torch.Tensor
) -> dict[str, float]:
    """Runs the Numeric Suite on model_a and model_b and returns a dictionary
    containing the SQNR between layers in model_a and model_b.

    Note: In order to support equalized models, this function has a hacky fix in
    which we do not match any torch.mul operators. This is because equalized
    models contain extra mul operators to scale the input by the equalization
    scale, but this edge case has not been resolved yet within the numeric suite code.

    Args:
        model_a: A float model
        model_b: A quantized model
        x: Inputs to use during calibration
    """
    import torch.ao.ns._numeric_suite_fx as ns
    from torch.ao.ns.fx.mappings import get_unmatchable_types_map

    unmatchable_types_map = get_unmatchable_types_map()
    unmatchable_types_map["funs_unmatchable"].add(torch.mul)
```
- **EN**: Key callable entry points in this range include `_convert_equalization_ref`, `get_layer_sqnr_dict`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_convert_equalization_ref`, `get_layer_sqnr_dict`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 963-1001 / 第 963-1001 行
```python
    model_a_ns, model_b_ns = ns.add_loggers(
        "fp32",
        model_a,
        "int8",
        model_b,
        ns.OutputLogger,
        unmatchable_types_map=unmatchable_types_map,
    )

    model_a_ns(x)
    model_b_ns(x)

    activation_comparison_dict = ns.extract_logger_info(
        model_a_ns, model_b_ns, ns.OutputLogger, "int8"
    )
    ns.extend_logger_results_with_comparison(
        activation_comparison_dict,
        "fp32",
        "int8",
        torch.ao.ns.fx.utils.compute_sqnr,
        "sqnr",
    )

    # Construct a dictionary mapping layer names to the SQNR values
    layer_sqnr_dict = {}
    for key in activation_comparison_dict:
        layer = activation_comparison_dict[key]["node_output"]["int8"][0]["fqn"]
        sqnr = activation_comparison_dict[key]["node_output"]["int8"][0]["sqnr"][0]
        layer_sqnr_dict[layer] = sqnr

    return layer_sqnr_dict


def get_equalization_qconfig_dict(
    layer_sqnr_dict: dict[str, float], num_layers_to_equalize: int
) -> Any:
    """Given the layer to SQNR dictionary, find the layers with the highest
    quantization errors, and return an equalization_qconfig_dict
    specifying to only equalize those top layers.
```
- **EN**: Key callable entry points in this range include `get_layer_sqnr_dict`, `get_equalization_qconfig_dict`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_layer_sqnr_dict`, `get_equalization_qconfig_dict`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 1003-1021 / 第 1003-1021 行
```python
    Args:
        layer_sqnr_dict: Dictionary mapping layer names to SQNR values (found
            when comparing an equalized model against a float model)
        num_layers_to_equalize: Number of layers with the highest quantization
           errors to equalize
    """

    # Sort the layer_sqnr_dictionary values and get the layers with the lowest
    # SQNR values (aka highest quantization errors)
    layer_sqnr_sorted = sorted(layer_sqnr_dict.items(), key=operator.itemgetter(1))
    layers_to_equalize = layer_sqnr_sorted[:num_layers_to_equalize]

    # Constructs an equalization_qconfig_dict that specifies to only equalize
    # the layers with the highest quantization errors
    module_to_qconfig_list = [
        (item[0], default_equalization_qconfig) for item in layers_to_equalize
    ]
    equalization_qconfig_dict = {"module_name": module_to_qconfig_list}
    return equalization_qconfig_dict
```
- **EN**: Key callable entry points in this range include `get_equalization_qconfig_dict`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_equalization_qconfig_dict`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

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
- **_InputEqualizationObserver**
  - EN: `_InputEqualizationObserver` is one of the main classes that structures the file's behavior.
  - CN: `_InputEqualizationObserver` 是组织该文件行为的核心类之一。
- **_WeightEqualizationObserver**
  - EN: `_WeightEqualizationObserver` is one of the main classes that structures the file's behavior.
  - CN: `_WeightEqualizationObserver` 是组织该文件行为的核心类之一。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch.ao.nn.intrinsic`, `torch.nn`, `torch.nn.functional`, `torch.ao.quantization.fx.graph_module:_get_observed_graph_module_attr`, `torch.ao.quantization.observer:_with_args`, `torch.ao.quantization.observer:ObserverBase`, `torch.ao.quantization.observer:PerChannelMinMaxObserver`, `torch.ao.quantization.utils:_parent_name`, `torch.ao.quantization.utils:check_min_max_valid`, `torch.fx:GraphModule`, `torch.fx.graph:Node`, `.utils:get_new_attr_name_with_prefix`, `.utils:maybe_get_next_module`, `.utils:node_arg_is_weight`
- **Python standard library / Python 标准库**: `operator`, `warnings`, `collections:namedtuple`, `typing:Any`
- **Third-party packages / 第三方包**: `typing_extensions:TypeIs`
- **Primary symbols / 核心符号**: `_InputEqualizationObserver`, `_WeightEqualizationObserver`, `EqualizationQConfig`, `reshape_scale`, `calculate_equalization_scale`, `fused_module_supports_equalization`, `nn_module_supports_equalization`, `custom_module_supports_equalization`, `node_supports_equalization`, `is_equalization_observer`, `get_op_node_and_weight_eq_obs`, `maybe_get_weight_eq_obs_node`, `maybe_get_next_input_eq_obs`, `maybe_get_next_equalization_scale`, `scale_input_observer`
