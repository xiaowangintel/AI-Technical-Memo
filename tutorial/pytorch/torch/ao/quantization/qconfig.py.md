# qconfig.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/quantization/qconfig.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements quantization observers, fake-quant modules, preparation/conversion passes, and backend-specific quantization helpers. This specific file centers on `qconfig.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior. Runtime configuration, environment overrides, or feature gating are important in this module.
- **Purpose (CN) / 用途（中文）**: 实现量化 observer、fake-quant 模块、prepare/convert pass 以及后端相关的量化辅助逻辑。 该文件具体围绕 `qconfig.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。 运行时配置、环境变量覆盖或特性开关是此模块的重要主题。
## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行
```python
# mypy: allow-untyped-defs
import copy
import warnings
from collections import namedtuple
from typing import Any
from typing_extensions import deprecated, TypeAliasType

import torch
import torch.nn as nn
from torch.ao.quantization.fake_quantize import (
    default_dynamic_fake_quant,
    default_embedding_fake_quant,
    default_embedding_fake_quant_4bit,
    default_fake_quant,
    default_fused_act_fake_quant,
    default_fused_per_channel_wt_fake_quant,
    default_fused_wt_fake_quant,
    default_per_channel_weight_fake_quant,
    default_weight_fake_quant,
    FakeQuantize,
    FakeQuantizeBase,
    fused_per_channel_wt_fake_quant_range_neg_127_to_127,
    fused_wt_fake_quant_range_neg_127_to_127,
    FusedMovingAvgObsFakeQuantize,
)
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch, torch.nn, torch.ao.quantization.fake_quantize:default_dynamic_fake_quant, torch.ao.quantization.fake_quantize:default_embedding_fake_quant; standard-library helpers such as copy, warnings, collections:namedtuple, typing:Any; external packages such as typing_extensions:deprecated, typing_extensions:TypeAliasType. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch, torch.nn, torch.ao.quantization.fake_quantize:default_dynamic_fake_quant, torch.ao.quantization.fake_quantize:default_embedding_fake_quant；标准库辅助模块，如 copy, warnings, collections:namedtuple, typing:Any；外部包，如 typing_extensions:deprecated, typing_extensions:TypeAliasType。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 27-47 / 第 27-47 行
```python
from .observer import (
    _PartialWrapper,
    default_debug_observer,
    default_dynamic_quant_observer,
    default_float_qparams_observer,
    default_float_qparams_observer_4bit,
    default_observer,
    default_per_channel_weight_observer,
    default_placeholder_observer,
    default_reuse_input_observer,
    default_weight_observer,
    HistogramObserver,
    MinMaxObserver,
    MovingAverageMinMaxObserver,
    NoopObserver,
    ObserverBase,
    per_channel_weight_observer_range_neg_127_to_127,
    PlaceholderObserver,
    ReuseInputObserver,
    weight_observer_range_neg_127_to_127,
)
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as .observer:_PartialWrapper, .observer:default_debug_observer, .observer:default_dynamic_quant_observer, .observer:default_float_qparams_observer. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 .observer:_PartialWrapper, .observer:default_debug_observer, .observer:default_dynamic_quant_observer, .observer:default_float_qparams_observer。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 50-81 / 第 50-81 行
```python
__all__ = [
    "QConfig",
    # TODO: deprecated, remove
    "QConfigDynamic",
    "default_qconfig",
    "default_debug_qconfig",
    "default_per_channel_qconfig",
    "default_dynamic_qconfig",
    "float16_dynamic_qconfig",
    "float16_static_qconfig",
    "per_channel_dynamic_qconfig",
    "float_qparams_weight_only_qconfig",
    "float_qparams_weight_only_qconfig_4bit",
    "default_quint8_weight_qconfig",
    "default_qat_qconfig",
    "default_dynamic_qat_qconfig",
    "default_weight_only_qconfig",
    "default_activation_only_qconfig",
    "default_qat_qconfig_v2",
    "default_reuse_input_qconfig",
    "default_symmetric_qnnpack_qconfig",
    "default_per_channel_symmetric_qnnpack_qconfig",
    "default_symmetric_qnnpack_qat_qconfig",
    "default_per_channel_symmetric_qnnpack_qat_qconfig",
    "default_embedding_qat_qconfig",
    "default_embedding_qat_qconfig_4bit",
    "get_default_qconfig",
    "get_default_qat_qconfig",
    "get_default_qconfig_dict",
    "get_default_qat_qconfig_dict",
    "QConfigAny",
    "qconfig_equals",
```
- **EN**: The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 82-107 / 第 82-107 行
```python
]


# pyrefly: ignore [invalid-inheritance]
class QConfig(namedtuple("QConfig", ["activation", "weight"])):
    """
    Describes how to quantize a layer or a part of the network by providing
    settings (observer classes) for activations and weights respectively.


    Note that QConfig needs to contain observer **classes** (like MinMaxObserver) or a callable that returns
    instances on invocation, not the concrete observer instances themselves.
    Quantization preparation function will instantiate observers multiple times for each of the layers.


    Observer classes have usually reasonable default arguments, but they can be overwritten with `with_args`
    method (that behaves like functools.partial)::

      my_qconfig = QConfig(
          activation=MinMaxObserver.with_args(dtype=torch.qint8),
          weight=default_observer.with_args(dtype=torch.qint8),
      )

    """

    __slots__ = ()
```
- **EN**: It introduces or extends class-level abstractions such as `QConfig`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `QConfig` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 109-139 / 第 109-139 行
```python
    def __new__(cls, activation, weight):
        # catch common mistakes
        if isinstance(activation, nn.Module) or isinstance(weight, nn.Module):
            raise ValueError(
                "QConfig received observer instance, please pass observer class instead. "
                + "Use MyObserver.with_args(x=1) to override arguments to constructor if needed"
            )
        return super().__new__(cls, activation, weight)


@deprecated(
    "`QConfigDynamic` is going to be deprecated in PyTorch 1.12, please use `QConfig` instead",
    category=FutureWarning,
)
# pyrefly: ignore [invalid-inheritance]
class QConfigDynamic(namedtuple("QConfigDynamic", ["activation", "weight"])):
    """
    Describes how to dynamically quantize a layer or a part of the network by providing
    settings (observer classes) for weights.

    It's like QConfig, but for dynamic quantization.

    Note that QConfigDynamic needs to contain observer **classes** (like MinMaxObserver) or a callable that returns
    instances on invocation, not the concrete observer instances themselves.
    Quantization function will instantiate observers multiple times for each of the layers.

    Observer classes have usually reasonable default arguments, but they can be overwritten with `with_args`
    method (that behaves like functools.partial)::

      my_qconfig = QConfigDynamic(weight=default_observer.with_args(dtype=torch.qint8))
    """
```
- **EN**: It introduces or extends class-level abstractions such as `QConfig`, `QConfigDynamic`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `QConfig`, `QConfigDynamic` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 141-170 / 第 141-170 行
```python
    __slots__ = ()

    def __new__(cls, activation=torch.nn.Identity, weight=torch.nn.Identity):
        # catch common mistakes
        if isinstance(weight, nn.Module):
            raise ValueError(
                "QConfigDynamic received observer instance, please pass observer class instead. "
                + "Use MyObserver.with_args(x=1) to override arguments to constructor if needed"
            )
        return super().__new__(cls, activation, weight)


default_qconfig = QConfig(activation=default_observer, weight=default_weight_observer)
"""
Default qconfig configuration.
"""

default_debug_qconfig = QConfig(
    weight=default_weight_observer, activation=default_debug_observer
)
"""
Default qconfig configuration for debugging.
"""

default_per_channel_qconfig = QConfig(
    activation=default_observer, weight=default_per_channel_weight_observer
)
"""
Default qconfig configuration for per channel weight quantization.
"""
```
- **EN**: It introduces or extends class-level abstractions such as `QConfigDynamic`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `QConfigDynamic` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 172-201 / 第 172-201 行
```python
default_dynamic_qconfig = QConfig(
    activation=default_dynamic_quant_observer, weight=default_weight_observer
)
"""
Default dynamic qconfig.
"""

float16_dynamic_qconfig = QConfig(
    activation=PlaceholderObserver.with_args(dtype=torch.float16, is_dynamic=True),
    weight=PlaceholderObserver.with_args(dtype=torch.float16),
)
"""
Dynamic qconfig with weights quantized to `torch.float16`.
"""

float16_static_qconfig = QConfig(
    activation=PlaceholderObserver.with_args(dtype=torch.float16),
    weight=PlaceholderObserver.with_args(dtype=torch.float16),
)
"""
Dynamic qconfig with both activations and weights quantized to `torch.float16`.
"""

per_channel_dynamic_qconfig = QConfig(
    activation=default_dynamic_quant_observer,
    weight=default_per_channel_weight_observer,
)
"""
Dynamic qconfig with weights quantized per channel.
"""
```
- **EN**: This chunk continues the implementation of `QConfigDynamic`, filling in the details of its control flow or data handling. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段延续了 `QConfigDynamic` 的实现，继续补充其控制流或数据处理细节。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 203-233 / 第 203-233 行
```python
float_qparams_weight_only_qconfig = QConfig(
    activation=default_placeholder_observer, weight=default_float_qparams_observer
)
"""
Dynamic qconfig with weights quantized with a floating point zero_point.
"""

float_qparams_weight_only_qconfig_4bit = QConfig(
    activation=default_placeholder_observer, weight=default_float_qparams_observer_4bit
)

default_qat_qconfig = QConfig(
    activation=default_fake_quant, weight=default_weight_fake_quant
)
"""
Default qconfig for QAT.
"""

default_dynamic_qat_qconfig = QConfig(
    activation=default_dynamic_fake_quant, weight=default_weight_fake_quant
)
"""
Default qconfig for dynamic QAT.
"""

default_weight_only_qconfig = QConfig(
    activation=torch.nn.Identity, weight=default_weight_fake_quant
)
"""
Default qconfig for quantizing weights only.
"""
```
- **EN**: This chunk continues the implementation of `QConfigDynamic`, filling in the details of its control flow or data handling. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段延续了 `QConfigDynamic` 的实现，继续补充其控制流或数据处理细节。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 235-265 / 第 235-265 行
```python
default_activation_only_qconfig = QConfig(
    activation=default_fake_quant, weight=torch.nn.Identity
)
"""
Default qconfig for quantizing activations only.
"""

# QAT config that uses a fused observer + fake quant modules for optimized training performance.
# to modify the activation/weight observers, the default entries in fake_quantize.py can be modified.
default_qat_qconfig_v2 = QConfig(
    activation=default_fused_act_fake_quant, weight=default_fused_wt_fake_quant
)
"""
Fused version of `default_qat_config`, has performance benefits.
"""

default_reuse_input_qconfig = QConfig(
    activation=default_reuse_input_observer, weight=NoopObserver
)
"""
Default qconfig for operators that reuse the observers from input Tensor, e.g. reshape
"""


def get_default_qconfig(backend="x86", version=0):
    """
    Returns the default PTQ qconfig for the specified backend.

    Args:
      * `backend` (str): a string representing the target backend. Currently supports
        `x86` (default), `fbgemm`, `qnnpack` and `onednn`.
```
- **EN**: Key callable entry points in this range include `get_default_qconfig`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_default_qconfig`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 267-298 / 第 267-298 行
```python
    Return:
        qconfig
    """
    supported_backends = ["fbgemm", "x86", "qnnpack", "onednn"]
    if backend not in supported_backends:
        raise AssertionError(
            "backend: "
            + str(backend)
            + f" not supported. backend must be one of {supported_backends}"
        )

    if version == 0:
        if backend == "fbgemm":
            qconfig = QConfig(
                activation=HistogramObserver.with_args(reduce_range=True),
                weight=default_per_channel_weight_observer,
            )
        elif backend == "qnnpack":
            # TODO: make this compatible with xnnpack constraints
            qconfig = QConfig(
                activation=HistogramObserver.with_args(reduce_range=False),
                weight=default_weight_observer,
            )
        elif backend == "onednn":
            if not torch.cpu._is_vnni_supported():
                warnings.warn(
                    "Default qconfig of oneDNN backend with reduce_range of false may have accuracy issues "
                    "on CPU without Vector Neural Network Instruction support.",
                    stacklevel=2,
                )
            qconfig = QConfig(
                activation=HistogramObserver.with_args(reduce_range=False),
```
- **EN**: Key callable entry points in this range include `get_default_qconfig`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_default_qconfig`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 299-329 / 第 299-329 行
```python
                weight=default_per_channel_weight_observer,
            )
        elif backend == "x86":
            qconfig = QConfig(
                activation=HistogramObserver.with_args(reduce_range=True),
                weight=default_per_channel_weight_observer,
            )
        else:
            # won't reach
            qconfig = default_qconfig
    else:
        raise AssertionError(
            "Version number: "
            + str(version)
            + " in get_default_qconfig is not supported. Version number must be 0"
        )

    return qconfig


"""
Default, symmetric PTQ qconfig for the specified backend. And a per_channel
variant of the same.

Symmetric here applies to signed weights with zero point = 0, and additional
value restrictions. The activations are also signed 8-bit integers with this
qconfig.

    * Once this change is merged [as of 3/17/22], with backend or qengine =
    'qnnpack', some quantized operators with this symmetric qconfig may use
    operators from xnnpack library.
```
- **EN**: Key callable entry points in this range include `get_default_qconfig`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_default_qconfig`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 331-357 / 第 331-357 行
```python
        ** Support to use xnnpack ops with `qnnpack` backed for asymmetric
        qconfig (returned by get_default_qconfig()) is not available yet.

    * This qconfig uses signed activations and weights. Weights have added
    restrictions such as zero point is forced to be 0, making the weights
    symmetric, hence the name. And the 8-bit quantized values are
    restricting to to [-127, +127], excluding -128.

    * xnnpack has a requantization scale value restriction, 0x1p-32 <=
    requantization_scale < 256.0 where, `requantization_scale = (input_scale
    * kernel_scale) / (output_scale)`. Using this eps (w/ assumed max value
    of 256) is to prevent requantization_scale to go below xnnpack lower
    threshold.
"""
default_symmetric_qnnpack_qconfig = QConfig(
    activation=HistogramObserver.with_args(
        dtype=torch.qint8, reduce_range=False, eps=2**-12
    ),
    weight=weight_observer_range_neg_127_to_127,
)

default_per_channel_symmetric_qnnpack_qconfig = QConfig(
    activation=HistogramObserver.with_args(
        dtype=torch.qint8, reduce_range=False, eps=2**-12
    ),
    weight=per_channel_weight_observer_range_neg_127_to_127,
)
```
- **EN**: This chunk continues the implementation of `get_default_qconfig`, filling in the details of its control flow or data handling. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段延续了 `get_default_qconfig` 的实现，继续补充其控制流或数据处理细节。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 359-381 / 第 359-381 行
```python
default_embedding_qat_qconfig = QConfig(
    activation=NoopObserver.with_args(dtype=torch.float32),
    weight=default_embedding_fake_quant,
)

default_embedding_qat_qconfig_4bit = QConfig(
    activation=NoopObserver.with_args(dtype=torch.float32),
    weight=default_embedding_fake_quant_4bit,
)

default_quint8_weight_qconfig = QConfig(
    activation=HistogramObserver, weight=MinMaxObserver
)


def get_default_qat_qconfig(backend="x86", version=1):
    """
    Returns the default QAT qconfig for the specified backend.

    Args:
      * `backend` (str): a string representing the target backend. Currently supports
        `x86` (default), `fbgemm`, `qnnpack` and `onednn`.
      * `version`: version, for backwards compatibility. Can be `None` or `1`.
```
- **EN**: Key callable entry points in this range include `get_default_qat_qconfig`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_default_qat_qconfig`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 383-414 / 第 383-414 行
```python
    Return:
        qconfig
    """
    supported_backends = ["fbgemm", "x86", "qnnpack", "onednn"]
    if backend not in supported_backends:
        raise AssertionError(
            "backend: "
            + str(backend)
            + f" not supported. backend must be one of {supported_backends}"
        )

    # Histogram observer is too slow for quantization aware training
    if version == 0:
        if backend == "fbgemm":
            qconfig = QConfig(
                activation=FakeQuantize.with_args(
                    observer=MovingAverageMinMaxObserver,
                    quant_min=0,
                    quant_max=255,
                    reduce_range=True,
                ),
                weight=default_per_channel_weight_fake_quant,
            )
        elif backend == "qnnpack":
            qconfig = QConfig(
                activation=FakeQuantize.with_args(
                    observer=MovingAverageMinMaxObserver,
                    quant_min=0,
                    quant_max=255,
                    reduce_range=False,
                ),
                weight=default_weight_fake_quant,
```
- **EN**: Key callable entry points in this range include `get_default_qat_qconfig`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_default_qat_qconfig`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 415-446 / 第 415-446 行
```python
            )
        elif backend == "onednn":
            qconfig = QConfig(
                activation=FakeQuantize.with_args(
                    observer=MovingAverageMinMaxObserver, quant_min=0, quant_max=255
                ),
                weight=default_per_channel_weight_fake_quant,
            )
        elif backend == "x86":
            qconfig = QConfig(
                activation=FakeQuantize.with_args(
                    observer=MovingAverageMinMaxObserver,
                    quant_min=0,
                    quant_max=255,
                    reduce_range=True,
                ),
                weight=default_per_channel_weight_fake_quant,
            )
        else:
            qconfig = default_qat_qconfig
    # Use the fused observe + fake_quant modules for doing QAT.
    elif version == 1:
        if backend == "fbgemm":
            qconfig = QConfig(
                activation=FusedMovingAvgObsFakeQuantize.with_args(
                    observer=MovingAverageMinMaxObserver,
                    quant_min=0,
                    quant_max=255,
                    reduce_range=True,
                ),
                weight=default_fused_per_channel_wt_fake_quant,
            )
```
- **EN**: Key callable entry points in this range include `get_default_qat_qconfig`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_default_qat_qconfig`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 447-478 / 第 447-478 行
```python
        elif backend == "qnnpack":
            # TODO: make this compatible with xnnpack constraints
            qconfig = QConfig(
                activation=FusedMovingAvgObsFakeQuantize.with_args(
                    observer=MovingAverageMinMaxObserver,
                    quant_min=0,
                    quant_max=255,
                    reduce_range=False,
                ),
                weight=default_fused_wt_fake_quant,
            )
        elif backend == "onednn":
            qconfig = QConfig(
                activation=FusedMovingAvgObsFakeQuantize.with_args(
                    observer=MovingAverageMinMaxObserver, quant_min=0, quant_max=255
                ),
                weight=default_fused_per_channel_wt_fake_quant,
            )
        elif backend == "x86":
            qconfig = QConfig(
                activation=FusedMovingAvgObsFakeQuantize.with_args(
                    observer=MovingAverageMinMaxObserver,
                    quant_min=0,
                    quant_max=255,
                    reduce_range=True,
                ),
                weight=default_fused_per_channel_wt_fake_quant,
            )
        else:
            qconfig = default_qat_qconfig_v2
    else:
        raise AssertionError(
```
- **EN**: Key callable entry points in this range include `get_default_qat_qconfig`. They package a focused unit of behavior behind named helpers or APIs. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_default_qat_qconfig`，它们把聚焦的行为封装成具名辅助函数或 API。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 479-500 / 第 479-500 行
```python
            "Version number: "
            + str(version)
            + "in get_default_qat_qconfig is not supported. Version number must be 0 or 1"
        )

    return qconfig


"""
Default symmetric QAT qconfig for qnnpack. And its per channel weight variant.
"""
default_symmetric_qnnpack_qat_qconfig = QConfig(
    activation=FusedMovingAvgObsFakeQuantize.with_args(
        observer=MovingAverageMinMaxObserver,
        quant_min=-128,
        quant_max=127,
        dtype=torch.qint8,
        reduce_range=False,
        eps=2**-12,
    ),
    weight=fused_wt_fake_quant_range_neg_127_to_127,
)
```
- **EN**: Key callable entry points in this range include `get_default_qat_qconfig`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_default_qat_qconfig`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 502-532 / 第 502-532 行
```python
default_per_channel_symmetric_qnnpack_qat_qconfig = QConfig(
    activation=FusedMovingAvgObsFakeQuantize.with_args(
        observer=MovingAverageMinMaxObserver,
        quant_min=-128,
        quant_max=127,
        dtype=torch.qint8,
        reduce_range=False,
        eps=2**-12,
    ),
    weight=fused_per_channel_wt_fake_quant_range_neg_127_to_127,
)

_default_fp32_placeholder_qconfig = QConfig(
    activation=PlaceholderObserver.with_args(dtype=torch.float32),
    weight=PlaceholderObserver.with_args(dtype=torch.float32),
)

_default_quint8_placeholder_qconfig = QConfig(
    activation=PlaceholderObserver.with_args(dtype=torch.quint8),
    # operators using this qconfig doesn't have weights
    weight=None,
)


@deprecated(
    "`torch.ao.quantization.get_default_qconfig_dict` is deprecated and will be removed in "
    "a future version. Please use `torch.ao.quantization.get_default_qconfig_mapping` instead.",
    category=FutureWarning,
)
def get_default_qconfig_dict(backend="x86", version=0):
    return torch.ao.quantization.get_default_qconfig_mapping(backend, version).to_dict()
```
- **EN**: Key callable entry points in this range include `get_default_qconfig_dict`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_default_qconfig_dict`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 535-566 / 第 535-566 行
```python
@deprecated(
    "`torch.ao.quantization.get_default_qat_qconfig_dict` is deprecated and will be removed in "
    "a future version. Please use `torch.ao.quantization.get_default_qat_qconfig_mapping` instead.",
    category=FutureWarning,
)
def get_default_qat_qconfig_dict(backend="x86", version=1):
    return torch.ao.quantization.get_default_qat_qconfig_mapping(
        backend, version
    ).to_dict()


def _assert_valid_qconfig(qconfig: QConfig | None, mod: torch.nn.Module) -> None:
    """
    Verifies that this `qconfig` is valid.
    """
    if qconfig is None:
        return
    is_conv_transpose_mod = isinstance(
        mod,
        (torch.nn.ConvTranspose1d, torch.nn.ConvTranspose2d, torch.nn.ConvTranspose3d),
    )
    if is_conv_transpose_mod:
        if qconfig.weight is None:
            # for now, we assume that any qconfig for ConvTranspose without a weight is valid
            return
        example_observer = qconfig.weight()
        is_per_channel = isinstance(
            example_observer,
            (
                torch.ao.quantization.PerChannelMinMaxObserver,
                torch.ao.quantization.MovingAveragePerChannelMinMaxObserver,
            ),
```
- **EN**: Key callable entry points in this range include `get_default_qat_qconfig_dict`, `_assert_valid_qconfig`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_default_qat_qconfig_dict`, `_assert_valid_qconfig`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 567-594 / 第 567-594 行
```python
        )
        if is_per_channel:
            raise AssertionError(
                "Per channel weight observer is not supported yet for ConvTranspose{n}d."
            )


QConfigAny = TypeAliasType("QConfigAny", QConfig | None)


def _add_module_to_qconfig_obs_ctr(
    qconfig: QConfigAny, module: nn.Module | None
) -> Any:
    r"""This is a helper function for use in quantization prepare that updates a qconfig so that
    the constructors stored in the qconfig will create observers on the same device that
    'module' is on. This is intended to be used when the qconfigs are propagated to each
    module in order to avoid potential device alignment issues.

    Args:
        qconfig: QConfig with obs constructors stored in activation and weight
        module: module which the qconfig is related to

    Return:
        qconfig: configured so that obs constructors set to construct on the same device as module
    """

    if module is None or qconfig is None or qconfig._fields != ("activation", "weight"):
        return qconfig
```
- **EN**: Key callable entry points in this range include `_assert_valid_qconfig`, `_add_module_to_qconfig_obs_ctr`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_assert_valid_qconfig`, `_add_module_to_qconfig_obs_ctr`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 596-626 / 第 596-626 行
```python
    def get_factory_kwargs_based_on_module_device():
        if not isinstance(module, torch.nn.Module):
            raise AssertionError("module must be an instance of torch.nn.Module")
        devices = {p.device for p in module.parameters()} | {
            p.device for p in module.buffers()
        }
        device = next(iter(devices)) if len(devices) > 0 else None
        return None if device is None else {"device": device}

    def configure_constructor_to_put_obs_on_module_device(original_constructor):
        try:
            # check if constructor can accept factory_kwargs
            check = original_constructor.with_args(factory_kwargs=None)
            check()
            return original_constructor.with_callable_args(
                factory_kwargs=get_factory_kwargs_based_on_module_device
            )
        except AttributeError:  # qconfig doesn't have activation or weight
            return original_constructor
        except TypeError:  # the class doesn't accept factory_kwargs argument
            return original_constructor

    activation = configure_constructor_to_put_obs_on_module_device(qconfig.activation)
    weight = configure_constructor_to_put_obs_on_module_device(qconfig.weight)

    return QConfig(activation, weight)


_ObserverOrFakeQuantizeConstructor = (
    _PartialWrapper | type[ObserverBase] | type[FakeQuantizeBase]
)
```
- **EN**: Key callable entry points in this range include `_add_module_to_qconfig_obs_ctr`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Structured exception handling keeps failure modes explicit instead of letting partial state escape silently. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_add_module_to_qconfig_obs_ctr`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 结构化异常处理让失败模式保持显式，而不是让部分状态悄悄泄漏出去。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 629-660 / 第 629-660 行
```python
def _obs_or_fq_ctr_equals(
    obs_or_fq1: _ObserverOrFakeQuantizeConstructor,
    obs_or_fq2: _ObserverOrFakeQuantizeConstructor,
):
    if isinstance(obs_or_fq1, _PartialWrapper) and isinstance(
        obs_or_fq2, _PartialWrapper
    ):
        return _partial_wrapper_equals(obs_or_fq1, obs_or_fq2)
    return obs_or_fq1 == obs_or_fq2


def _partial_wrapper_equals(obs_or_fq1: _PartialWrapper, obs_or_fq2: _PartialWrapper):
    """
    Return whether the two partial wrappers are equal,
    """
    # functools.partial has no __eq__ operator defined so '==' defaults to 'is'
    obs_or_fq1_keywords = copy.copy(obs_or_fq1.p.keywords)
    obs_or_fq2_keywords = copy.copy(obs_or_fq2.p.keywords)
    keywords_equal = True
    # compare observer constructor with _obs_or_fq_ctr_equals since direct compare would fail
    if "observer" in obs_or_fq1_keywords and "observer" in obs_or_fq2_keywords:
        keywords_equal = keywords_equal and _obs_or_fq_ctr_equals(
            obs_or_fq1_keywords["observer"], obs_or_fq2_keywords["observer"]
        )
        obs_or_fq1_keywords.pop("observer")
        obs_or_fq2_keywords.pop("observer")
    keywords_equal = keywords_equal and obs_or_fq1_keywords == obs_or_fq2_keywords
    return (
        obs_or_fq1.p.func == obs_or_fq2.p.func
        and obs_or_fq1.p.args == obs_or_fq2.p.args
        and keywords_equal
    )
```
- **EN**: Key callable entry points in this range include `_obs_or_fq_ctr_equals`, `_partial_wrapper_equals`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_obs_or_fq_ctr_equals`, `_partial_wrapper_equals`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 663-689 / 第 663-689 行
```python
def qconfig_equals(q1: QConfigAny, q2: QConfigAny):
    """
    Returns `True` if `q1` equals `q2`, and `False` otherwise.
    """
    if q1 is None or q2 is None:
        return q1 == q2
    else:
        if q1 is None or q2 is None:
            raise AssertionError(
                "Both q1 and q2 must be non-None for qconfig comparison"
            )
        try:
            # Qconfig weight and activation can be either a partial wrapper,
            # or an observer class. Special handling is required (above) for
            # comparing partial wrappers.
            activation_same = _obs_or_fq_ctr_equals(q1.activation, q2.activation)
            weight_same = _obs_or_fq_ctr_equals(q1.weight, q2.weight)
            return activation_same and weight_same
        except AttributeError:
            return q1 == q2


def _activation_is_memoryless(qconfig: QConfig):
    """
    Return whether the observer for activations defined in the given QConfig is memoryless.
    This means a MovingAverage observer with averaging constant equal to 1.
    """
```
- **EN**: Key callable entry points in this range include `qconfig_equals`, `_activation_is_memoryless`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Structured exception handling keeps failure modes explicit instead of letting partial state escape silently. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `qconfig_equals`, `_activation_is_memoryless`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 结构化异常处理让失败模式保持显式，而不是让部分状态悄悄泄漏出去。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 691-708 / 第 691-708 行
```python
    def _is_memoryless(observer):
        return (
            hasattr(observer, "averaging_constant") and observer.averaging_constant == 1
        )

    act = qconfig.activation()
    if isinstance(act, FakeQuantizeBase) and hasattr(act, "activation_post_process"):
        return _is_memoryless(act.activation_post_process)
    else:
        return _is_memoryless(act)


def _is_reuse_input_qconfig(qconfig: QConfig | None):
    return (
        qconfig is not None
        and isinstance(qconfig.activation(), ReuseInputObserver)
        and isinstance(qconfig.weight(), NoopObserver)
    )
```
- **EN**: Key callable entry points in this range include `_activation_is_memoryless`, `_is_reuse_input_qconfig`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_activation_is_memoryless`, `_is_reuse_input_qconfig`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

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
- **Fake quantization**
  - EN: Simulates quantized numerics during training while keeping tensors in differentiable forms.
  - CN: 在训练阶段模拟量化数值行为，同时保持张量处于可求导形式。
- **Configuration management**
  - EN: Centralizes feature flags, environment overrides, and runtime-config wiring.
  - CN: 集中管理特性开关、环境变量覆盖以及运行时配置接线。
- **QConfig**
  - EN: `QConfig` is one of the main classes that structures the file's behavior.
  - CN: `QConfig` 是组织该文件行为的核心类之一。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch.nn`, `torch.ao.quantization.fake_quantize:default_dynamic_fake_quant`, `torch.ao.quantization.fake_quantize:default_embedding_fake_quant`, `torch.ao.quantization.fake_quantize:default_embedding_fake_quant_4bit`, `torch.ao.quantization.fake_quantize:default_fake_quant`, `torch.ao.quantization.fake_quantize:default_fused_act_fake_quant`, `torch.ao.quantization.fake_quantize:default_fused_per_channel_wt_fake_quant`, `torch.ao.quantization.fake_quantize:default_fused_wt_fake_quant`, `torch.ao.quantization.fake_quantize:default_per_channel_weight_fake_quant`, `torch.ao.quantization.fake_quantize:default_weight_fake_quant`, `torch.ao.quantization.fake_quantize:FakeQuantize`, `torch.ao.quantization.fake_quantize:FakeQuantizeBase`, `torch.ao.quantization.fake_quantize:fused_per_channel_wt_fake_quant_range_neg_127_to_127`, `torch.ao.quantization.fake_quantize:fused_wt_fake_quant_range_neg_127_to_127`
- **Python standard library / Python 标准库**: `copy`, `warnings`, `collections:namedtuple`, `typing:Any`
- **Third-party packages / 第三方包**: `typing_extensions:deprecated`, `typing_extensions:TypeAliasType`
- **Explicit exports / 显式导出**: `QConfig`, `QConfigDynamic`, `default_qconfig`, `default_debug_qconfig`, `default_per_channel_qconfig`, `default_dynamic_qconfig`, `float16_dynamic_qconfig`, `float16_static_qconfig`, `per_channel_dynamic_qconfig`, `float_qparams_weight_only_qconfig`, `float_qparams_weight_only_qconfig_4bit`, `default_quint8_weight_qconfig`, `default_qat_qconfig`, `default_dynamic_qat_qconfig`, `default_weight_only_qconfig`
- **Primary symbols / 核心符号**: `QConfig`, `QConfigDynamic`, `get_default_qconfig`, `get_default_qat_qconfig`, `get_default_qconfig_dict`, `get_default_qat_qconfig_dict`, `_assert_valid_qconfig`, `_add_module_to_qconfig_obs_ctr`, `_obs_or_fq_ctr_equals`, `_partial_wrapper_equals`, `qconfig_equals`, `_activation_is_memoryless`, `_is_reuse_input_qconfig`
