# quantization_mappings.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/quantization/quantization_mappings.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements quantization observers, fake-quant modules, preparation/conversion passes, and backend-specific quantization helpers. This specific file centers on `quantization_mappings.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior. Runtime configuration, environment overrides, or feature gating are important in this module.
- **Purpose (CN) / 用途（中文）**: 实现量化 observer、fake-quant 模块、prepare/convert pass 以及后端相关的量化辅助逻辑。 该文件具体围绕 `quantization_mappings.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。 运行时配置、环境变量覆盖或特性开关是此模块的重要主题。
## Line-by-Line Analysis / 逐行分析

### Lines 1-15 / 第 1-15 行
```python
import copy
from collections.abc import Callable
from typing import Any

import torch
import torch.ao.nn as ao_nn
import torch.ao.nn.intrinsic as nni
import torch.ao.nn.intrinsic.qat as nniqat
import torch.ao.nn.intrinsic.quantized as nniq
import torch.ao.nn.intrinsic.quantized.dynamic as nniqd
import torch.ao.nn.qat as nnqat
import torch.ao.nn.qat.dynamic as nnqatd
import torch.ao.nn.quantized as nnq
import torch.ao.nn.quantized.dynamic as nnqd
import torch.ao.nn.quantized.reference as nnqr
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch, torch.ao.nn, torch.ao.nn.intrinsic, torch.ao.nn.intrinsic.qat; standard-library helpers such as copy, collections.abc:Callable, typing:Any. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch, torch.ao.nn, torch.ao.nn.intrinsic, torch.ao.nn.intrinsic.qat；标准库辅助模块，如 copy, collections.abc:Callable, typing:Any。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 17-40 / 第 17-40 行
```python
# Because `torch.ao.nn` uses lazy imports, we need to make
# sure we import the contents explicitly here.
import torch.ao.nn.sparse
import torch.nn.functional as F
from torch import nn
from torch.ao.quantization.fake_quantize import (
    default_fixed_qparams_range_0to1_fake_quant,
    default_fixed_qparams_range_neg1to1_fake_quant,
)
from torch.ao.quantization.stubs import DeQuantStub, QuantStub
from torch.ao.quantization.utils import get_combined_dict
from torch.nn.utils.parametrize import type_before_parametrizations


__all__ = [
    "DEFAULT_REFERENCE_STATIC_QUANT_MODULE_MAPPINGS",
    "DEFAULT_STATIC_QUANT_MODULE_MAPPINGS",
    "DEFAULT_QAT_MODULE_MAPPINGS",
    "DEFAULT_DYNAMIC_QUANT_MODULE_MAPPINGS",
    "DEFAULT_FLOAT_TO_QUANTIZED_OPERATOR_MAPPINGS",
    "DEFAULT_MODULE_TO_ACT_POST_PROCESS",
    "DEFAULT_STATIC_SPARSE_QUANT_MODULE_MAPPINGS",
    "DEFAULT_DYNAMIC_SPARSE_QUANT_MODULE_MAPPINGS",
    "no_observer_set",
```
- **EN**: The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 41-55 / 第 41-55 行
```python
    "get_default_static_quant_module_mappings",
    "get_default_static_quant_reference_module_mappings",
    "get_embedding_static_quant_module_mappings",
    "get_default_static_sparse_quant_module_mappings",
    "get_static_quant_module_class",
    "get_dynamic_quant_module_class",
    "get_default_qat_module_mappings",
    "get_embedding_qat_module_mappings",
    "get_default_dynamic_quant_module_mappings",
    "get_default_dynamic_sparse_quant_module_mappings",
    "get_default_qconfig_propagation_list",
    "get_default_compare_output_module_list",
    "get_default_float_to_quantized_operator_mappings",
    "get_quantized_operator",
]
```
- **EN**: The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 57-74 / 第 57-74 行
```python
# Default map for swapping float module to reference quantized modules
DEFAULT_REFERENCE_STATIC_QUANT_MODULE_MAPPINGS: dict[Callable, Any] = {
    QuantStub: nnq.Quantize,
    DeQuantStub: nnq.DeQuantize,
    nn.Linear: nnqr.Linear,
    nn.Conv1d: nnqr.Conv1d,
    nn.Conv2d: nnqr.Conv2d,
    nn.Conv3d: nnqr.Conv3d,
    nn.ConvTranspose1d: nnqr.ConvTranspose1d,
    nn.ConvTranspose2d: nnqr.ConvTranspose2d,
    nn.ConvTranspose3d: nnqr.ConvTranspose3d,
    nn.Embedding: nnqr.Embedding,
    nn.EmbeddingBag: nnqr.EmbeddingBag,
    nn.GRUCell: nnqr.GRUCell,
    nn.LSTMCell: nnqr.LSTMCell,
    nn.RNNCell: nnqr.RNNCell,
    nn.LSTM: nnqr.LSTM,
}
```
- **EN**: The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 76-99 / 第 76-99 行
```python
# Default map for swapping float module to quantized ones
DEFAULT_STATIC_QUANT_MODULE_MAPPINGS: dict[Callable, Any] = {
    QuantStub: nnq.Quantize,
    DeQuantStub: nnq.DeQuantize,
    nn.BatchNorm2d: nnq.BatchNorm2d,
    nn.BatchNorm3d: nnq.BatchNorm3d,
    nn.Dropout: nnq.Dropout,
    nn.Conv1d: nnq.Conv1d,
    nn.Conv2d: nnq.Conv2d,
    nn.Conv3d: nnq.Conv3d,
    nn.ConvTranspose1d: nnq.ConvTranspose1d,
    nn.ConvTranspose2d: nnq.ConvTranspose2d,
    nn.ConvTranspose3d: nnq.ConvTranspose3d,
    nn.ELU: nnq.ELU,
    nn.Embedding: nnq.Embedding,
    nn.EmbeddingBag: nnq.EmbeddingBag,
    nn.GroupNorm: nnq.GroupNorm,
    nn.Hardswish: nnq.Hardswish,
    nn.InstanceNorm1d: nnq.InstanceNorm1d,
    nn.InstanceNorm2d: nnq.InstanceNorm2d,
    nn.InstanceNorm3d: nnq.InstanceNorm3d,
    nn.LayerNorm: nnq.LayerNorm,
    nn.LeakyReLU: nnq.LeakyReLU,
    nn.modules.linear.NonDynamicallyQuantizableLinear: nnq.Linear,
```
- **EN**: The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 100-123 / 第 100-123 行
```python
    nn.Linear: nnq.Linear,
    nn.ReLU6: nnq.ReLU6,
    nn.PReLU: nnq.PReLU,
    # Wrapper Modules:
    nnq.FloatFunctional: nnq.QFunctional,
    # Intrinsic modules:
    nni.BNReLU2d: nniq.BNReLU2d,
    nni.BNReLU3d: nniq.BNReLU3d,
    nni.ConvReLU1d: nniq.ConvReLU1d,
    nni.ConvReLU2d: nniq.ConvReLU2d,
    nni.ConvReLU3d: nniq.ConvReLU3d,
    nni.ConvAdd2d: nniq.ConvAdd2d,
    nni.ConvAddReLU2d: nniq.ConvAddReLU2d,
    nni.LinearReLU: nniq.LinearReLU,
    nni.LinearLeakyReLU: nniq.LinearLeakyReLU,
    nni.LinearTanh: nniq.LinearTanh,
    nniqat.ConvBn1d: nnq.Conv1d,
    nniqat.ConvBn2d: nnq.Conv2d,
    nniqat.ConvBn3d: nnq.Conv3d,
    nniqat.ConvBnReLU1d: nniq.ConvReLU1d,
    nniqat.ConvBnReLU2d: nniq.ConvReLU2d,
    nniqat.ConvBnReLU3d: nniq.ConvReLU3d,
    nniqat.ConvReLU2d: nniq.ConvReLU2d,
    nniqat.ConvReLU3d: nniq.ConvReLU3d,
```
- **EN**: The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 124-147 / 第 124-147 行
```python
    nniqat.LinearReLU: nniq.LinearReLU,
    nniqat.LinearBn1d: nnq.Linear,
    # QAT modules:
    nnqat.Linear: nnq.Linear,
    nnqat.Conv2d: nnq.Conv2d,
    nnqat.Conv3d: nnq.Conv3d,
}

# Default map for swapping float module to qat modules
DEFAULT_QAT_MODULE_MAPPINGS: dict[Callable, Any] = {
    nn.Conv2d: nnqat.Conv2d,
    nn.Conv3d: nnqat.Conv3d,
    nn.Linear: nnqat.Linear,
    nn.modules.linear.NonDynamicallyQuantizableLinear: nnqat.Linear,
    # Intrinsic modules:
    nni.ConvBn1d: nniqat.ConvBn1d,
    nni.ConvBn2d: nniqat.ConvBn2d,
    nni.ConvBn3d: nniqat.ConvBn3d,
    nni.ConvBnReLU1d: nniqat.ConvBnReLU1d,
    nni.ConvBnReLU2d: nniqat.ConvBnReLU2d,
    nni.ConvBnReLU3d: nniqat.ConvBnReLU3d,
    nni.ConvReLU2d: nniqat.ConvReLU2d,
    nni.ConvReLU3d: nniqat.ConvReLU3d,
    nni.LinearReLU: nniqat.LinearReLU,
```
- **EN**: The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 148-171 / 第 148-171 行
```python
    nni.LinearBn1d: nniqat.LinearBn1d,
}

# Default map for swapping dynamic modules
DEFAULT_DYNAMIC_QUANT_MODULE_MAPPINGS: dict[Callable, Any] = {
    nn.GRUCell: nnqd.GRUCell,
    nn.Linear: nnqd.Linear,
    nnqatd.Linear: nnqd.Linear,
    nn.modules.linear.NonDynamicallyQuantizableLinear: nnqd.Linear,
    nn.LSTM: nnqd.LSTM,
    nn.GRU: nnqd.GRU,
    nn.LSTMCell: nnqd.LSTMCell,
    nn.RNNCell: nnqd.RNNCell,
    nni.LinearReLU: nniqd.LinearReLU,
    nn.EmbeddingBag: nnq.EmbeddingBag,
    nn.Embedding: nnq.Embedding,
    # Don't want to enable these by default because the numerical
    # accuracy is poor compared to other dynamic ops
    # nn.Conv1d: nnqd.Conv1d,
    # nn.Conv2d: nnqd.Conv2d,
    # nn.Conv3d: nnqd.Conv3d,
    # nn.ConvTranspose1d: nnqd.ConvTranspose1d,
    # nn.ConvTranspose2d: nnqd.ConvTranspose2d,
    # nn.ConvTranspose3d: nnqd.ConvTranspose3d,
```
- **EN**: The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 172-188 / 第 172-188 行
```python
}

# Allowlist for propagating the qconfig
_INCLUDE_QCONFIG_PROPAGATE_LIST: set[Callable] = {
    nn.Sequential,
}

# Default mapping from floating point function or torch ops to quantized ops
# TODO: merge with default static mapping
DEFAULT_FLOAT_TO_QUANTIZED_OPERATOR_MAPPINGS: dict[Callable | str, Callable] = {
    F.elu: torch.ops.quantized.elu,
    F.hardswish: torch.ops.quantized.hardswish,
    F.instance_norm: torch.ops.quantized.instance_norm,
    F.layer_norm: torch.ops.quantized.layer_norm,
    F.leaky_relu: torch.ops.quantized.leaky_relu,
    F.dropout: torch.ops.quantized.dropout,
}
```
- **EN**: The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 190-212 / 第 190-212 行
```python
# mapping from module to output activation post process class
DEFAULT_MODULE_TO_ACT_POST_PROCESS: dict[Callable, Callable] = {
    nn.Hardsigmoid: default_fixed_qparams_range_0to1_fake_quant,
    nn.Sigmoid: default_fixed_qparams_range_0to1_fake_quant,
    nn.Softmax: default_fixed_qparams_range_0to1_fake_quant,
    nn.Tanh: default_fixed_qparams_range_neg1to1_fake_quant,
}

# Default map for swapping float module to static sparse quantized ones
DEFAULT_STATIC_SPARSE_QUANT_MODULE_MAPPINGS: dict[Callable, Any] = {
    nn.Linear: ao_nn.sparse.quantized.Linear
}

# Default map for swapping float module to dynamic sparse quantized ones
DEFAULT_DYNAMIC_SPARSE_QUANT_MODULE_MAPPINGS: dict[Callable, Any] = {
    nn.Linear: ao_nn.sparse.quantized.dynamic.Linear
}


def no_observer_set() -> set[Any]:
    r"""These modules cannot have observers inserted by default."""
    no_observers = {nn.quantizable.LSTM, nn.quantizable.MultiheadAttention}
    return no_observers
```
- **EN**: Key callable entry points in this range include `no_observer_set`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `no_observer_set`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 215-235 / 第 215-235 行
```python
def get_default_static_quant_module_mappings() -> dict[Callable, Any]:
    """Get module mapping for post training static quantization"""
    return copy.deepcopy(DEFAULT_STATIC_QUANT_MODULE_MAPPINGS)


def get_default_static_quant_reference_module_mappings() -> dict[Callable, Any]:
    """Get reference module mapping for post training static quantization"""
    return copy.deepcopy(DEFAULT_REFERENCE_STATIC_QUANT_MODULE_MAPPINGS)


def get_embedding_static_quant_module_mappings() -> dict[Callable, Any]:
    """Get module mapping, including mapping for embedding QAT"""
    mapping = copy.deepcopy(DEFAULT_STATIC_QUANT_MODULE_MAPPINGS)
    mapping[nnqat.EmbeddingBag] = nnq.EmbeddingBag
    mapping[nnqat.Embedding] = nnq.Embedding
    return mapping


def get_default_static_sparse_quant_module_mappings() -> dict[Callable, Any]:
    """Get module mapping for post training static sparse quantization"""
    return copy.deepcopy(DEFAULT_STATIC_SPARSE_QUANT_MODULE_MAPPINGS)
```
- **EN**: Key callable entry points in this range include `get_default_static_quant_module_mappings`, `get_default_static_quant_reference_module_mappings`, `get_embedding_static_quant_module_mappings`, `get_default_static_sparse_quant_module_mappings`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_default_static_quant_module_mappings`, `get_default_static_quant_reference_module_mappings`, `get_embedding_static_quant_module_mappings`, `get_default_static_sparse_quant_module_mappings`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 238-260 / 第 238-260 行
```python
def get_static_quant_module_class(
    float_module_class: Callable,
    additional_static_quant_mapping: dict[Callable, Any] | None = None,
    is_reference: bool = False,
) -> Any:
    r"""n Get the statically quantized module class corresponding to
    the floating point module class
    """
    if additional_static_quant_mapping is None:
        additional_static_quant_mapping = {}
    all_mappings = get_combined_dict(
        DEFAULT_REFERENCE_STATIC_QUANT_MODULE_MAPPINGS
        if is_reference
        else DEFAULT_STATIC_QUANT_MODULE_MAPPINGS,
        additional_static_quant_mapping,
    )
    static_quant_module_class = all_mappings.get(float_module_class, None)
    if static_quant_module_class is None:
        raise AssertionError(
            f"Floating point module class {str(float_module_class)}"
            + " does not have a corresponding quantized module class"
        )
    return copy.deepcopy(static_quant_module_class)
```
- **EN**: Key callable entry points in this range include `get_static_quant_module_class`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_static_quant_module_class`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 263-281 / 第 263-281 行
```python
def get_dynamic_quant_module_class(
    float_module_class: Callable,
    additional_dynamic_quant_mapping: dict[Callable, Any] | None = None,
) -> Any:
    r"""n Get the dynamically quantized module class corresponding to
    the floating point module class
    """
    if additional_dynamic_quant_mapping is None:
        additional_dynamic_quant_mapping = {}
    all_mappings = get_combined_dict(
        DEFAULT_DYNAMIC_QUANT_MODULE_MAPPINGS, additional_dynamic_quant_mapping
    )
    dynamic_quant_module_class = all_mappings.get(float_module_class, None)
    if dynamic_quant_module_class is None:
        raise AssertionError(
            f"Floating point module class {str(float_module_class)}"
            + " does not have a corresponding quantized module class"
        )
    return copy.deepcopy(dynamic_quant_module_class)
```
- **EN**: Key callable entry points in this range include `get_dynamic_quant_module_class`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_dynamic_quant_module_class`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 284-302 / 第 284-302 行
```python
def get_default_qat_module_mappings() -> dict[Callable, Any]:
    """Get default module mapping for quantization aware training"""
    return copy.deepcopy(DEFAULT_QAT_MODULE_MAPPINGS)


def get_embedding_qat_module_mappings() -> dict[Callable, Any]:
    """Get module mapping for quantization aware training
    This is includes default values in addition to
    enabling qat for embeddings.
    """
    mapping = copy.deepcopy(DEFAULT_QAT_MODULE_MAPPINGS)
    mapping[nn.EmbeddingBag] = nnqat.EmbeddingBag
    mapping[nn.Embedding] = nnqat.Embedding
    return mapping


def get_default_dynamic_quant_module_mappings() -> dict[Callable, Any]:
    """Get module mapping for post training dynamic quantization"""
    return DEFAULT_DYNAMIC_QUANT_MODULE_MAPPINGS
```
- **EN**: Key callable entry points in this range include `get_default_qat_module_mappings`, `get_embedding_qat_module_mappings`, `get_default_dynamic_quant_module_mappings`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_default_qat_module_mappings`, `get_embedding_qat_module_mappings`, `get_default_dynamic_quant_module_mappings`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 305-320 / 第 305-320 行
```python
def get_default_dynamic_sparse_quant_module_mappings() -> dict[Callable, Any]:
    """Get module mapping for post training dynamic sparse quantization"""
    return DEFAULT_DYNAMIC_SPARSE_QUANT_MODULE_MAPPINGS


def get_default_qconfig_propagation_list() -> set[Callable]:
    """Get the default list of module types that we'll attach qconfig
    attribute to in prepare
    """
    QCONFIG_PROPAGATE_MODULE_CLASS_LIST = (
        set(DEFAULT_STATIC_QUANT_MODULE_MAPPINGS.keys())
        | set(DEFAULT_QAT_MODULE_MAPPINGS.keys())
        | set(DEFAULT_DYNAMIC_QUANT_MODULE_MAPPINGS.keys())
        | _INCLUDE_QCONFIG_PROPAGATE_LIST
    )
    return copy.deepcopy(QCONFIG_PROPAGATE_MODULE_CLASS_LIST)
```
- **EN**: Key callable entry points in this range include `get_default_dynamic_sparse_quant_module_mappings`, `get_default_qconfig_propagation_list`. They package a focused unit of behavior behind named helpers or APIs. Named constants such as `QCONFIG_PROPAGATE_MODULE_CLASS_LIST` centralize shared configuration or sentinel values. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_default_dynamic_sparse_quant_module_mappings`, `get_default_qconfig_propagation_list`，它们把聚焦的行为封装成具名辅助函数或 API。 `QCONFIG_PROPAGATE_MODULE_CLASS_LIST` 等具名常量把共享配置或哨兵值集中定义在一起。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 323-342 / 第 323-342 行
```python
def get_default_compare_output_module_list() -> set[Callable]:
    """Get list of module class types that we will record output
    in numeric suite
    """
    NUMERIC_SUITE_COMPARE_MODEL_OUTPUT_MODULE_LIST = (
        set(DEFAULT_STATIC_QUANT_MODULE_MAPPINGS.values())
        | set(DEFAULT_QAT_MODULE_MAPPINGS.values())
        | set(DEFAULT_DYNAMIC_QUANT_MODULE_MAPPINGS.values())
        | set(DEFAULT_STATIC_QUANT_MODULE_MAPPINGS.keys())
        | set(DEFAULT_QAT_MODULE_MAPPINGS.keys())
        | set(DEFAULT_DYNAMIC_QUANT_MODULE_MAPPINGS.keys())
        | _INCLUDE_QCONFIG_PROPAGATE_LIST
    )
    return copy.deepcopy(NUMERIC_SUITE_COMPARE_MODEL_OUTPUT_MODULE_LIST)


def get_default_float_to_quantized_operator_mappings() -> dict[
    Callable | str, Callable
]:
    return copy.deepcopy(DEFAULT_FLOAT_TO_QUANTIZED_OPERATOR_MAPPINGS)
```
- **EN**: Key callable entry points in this range include `get_default_compare_output_module_list`, `get_default_float_to_quantized_operator_mappings`. They package a focused unit of behavior behind named helpers or APIs. Named constants such as `NUMERIC_SUITE_COMPARE_MODEL_OUTPUT_MODULE_LIST` centralize shared configuration or sentinel values. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_default_compare_output_module_list`, `get_default_float_to_quantized_operator_mappings`，它们把聚焦的行为封装成具名辅助函数或 API。 `NUMERIC_SUITE_COMPARE_MODEL_OUTPUT_MODULE_LIST` 等具名常量把共享配置或哨兵值集中定义在一起。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 345-367 / 第 345-367 行
```python
# TODO: merge with get_static_quant_module_class
def get_quantized_operator(float_op: Callable | str) -> Callable:
    """Get the quantized operator corresponding to the float operator"""
    quantized_op = DEFAULT_FLOAT_TO_QUANTIZED_OPERATOR_MAPPINGS.get(float_op)
    if quantized_op is None:
        raise AssertionError(
            f"Operator {str(float_op)} does not have corresponding quantized op"
        )
    return quantized_op


def _get_special_act_post_process(module: torch.nn.Module) -> Callable | None:
    r"""Get the special activation post process for `module`, this has
    higher priority than the activation post process in `qconfig`
    e.g.
    input: torch.nn.Sigmoid
    output: default_affine_fixed_qparam_fake_quant
    """
    return DEFAULT_MODULE_TO_ACT_POST_PROCESS.get(type_before_parametrizations(module))


def _has_special_act_post_process(module: torch.nn.Module) -> bool:
    return module.training and type(module) in DEFAULT_MODULE_TO_ACT_POST_PROCESS
```
- **EN**: Key callable entry points in this range include `get_quantized_operator`, `_get_special_act_post_process`, `_has_special_act_post_process`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_quantized_operator`, `_get_special_act_post_process`, `_has_special_act_post_process`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

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
- **Pruning and sparsity**
  - EN: Controls parameter masking, sparse layouts, or pruning schedules that reduce effective model size.
  - CN: 控制参数掩码、稀疏布局或剪枝计划，以降低模型的有效规模。
- **Numeric comparison**
  - EN: Helps compare reference and transformed executions to diagnose quantization effects.
  - CN: 帮助比较参考执行与转换后执行，从而诊断量化带来的影响。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch.ao.nn`, `torch.ao.nn.intrinsic`, `torch.ao.nn.intrinsic.qat`, `torch.ao.nn.intrinsic.quantized`, `torch.ao.nn.intrinsic.quantized.dynamic`, `torch.ao.nn.qat`, `torch.ao.nn.qat.dynamic`, `torch.ao.nn.quantized`, `torch.ao.nn.quantized.dynamic`, `torch.ao.nn.quantized.reference`, `torch.ao.nn.sparse`, `torch.nn.functional`, `torch:nn`, `torch.ao.quantization.fake_quantize:default_fixed_qparams_range_0to1_fake_quant`
- **Python standard library / Python 标准库**: `copy`, `collections.abc:Callable`, `typing:Any`
- **Explicit exports / 显式导出**: `DEFAULT_REFERENCE_STATIC_QUANT_MODULE_MAPPINGS`, `DEFAULT_STATIC_QUANT_MODULE_MAPPINGS`, `DEFAULT_QAT_MODULE_MAPPINGS`, `DEFAULT_DYNAMIC_QUANT_MODULE_MAPPINGS`, `DEFAULT_FLOAT_TO_QUANTIZED_OPERATOR_MAPPINGS`, `DEFAULT_MODULE_TO_ACT_POST_PROCESS`, `DEFAULT_STATIC_SPARSE_QUANT_MODULE_MAPPINGS`, `DEFAULT_DYNAMIC_SPARSE_QUANT_MODULE_MAPPINGS`, `no_observer_set`, `get_default_static_quant_module_mappings`, `get_default_static_quant_reference_module_mappings`, `get_embedding_static_quant_module_mappings`, `get_default_static_sparse_quant_module_mappings`, `get_static_quant_module_class`, `get_dynamic_quant_module_class`
- **Primary symbols / 核心符号**: `no_observer_set`, `get_default_static_quant_module_mappings`, `get_default_static_quant_reference_module_mappings`, `get_embedding_static_quant_module_mappings`, `get_default_static_sparse_quant_module_mappings`, `get_static_quant_module_class`, `get_dynamic_quant_module_class`, `get_default_qat_module_mappings`, `get_embedding_qat_module_mappings`, `get_default_dynamic_quant_module_mappings`, `get_default_dynamic_sparse_quant_module_mappings`, `get_default_qconfig_propagation_list`, `get_default_compare_output_module_list`, `get_default_float_to_quantized_operator_mappings`, `get_quantized_operator`
