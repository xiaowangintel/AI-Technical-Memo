# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/model_loader/reload/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements utils support for the `reload` portion of vLLM. / 为 vLLM 的 `reload` 子目录实现与 工具 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-8)
```python
from inspect import BoundArguments

import torch

from .types import LayerReloadingInfo, LayerTensors
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 10-16)
```python
__all__ = [
    "get_layer_tensors",
    "get_layer_params_buffers",
    "get_layer_size",
    "has_device_tensors",
    "get_info_size",
]
```
**EN:** This constant/configuration block defines `__all__`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `__all__`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Function `get_layer_tensors` (lines 19-22)
```python
def get_layer_tensors(layer: torch.nn.Module) -> dict[str, torch.Tensor]:
    """Get all parameters and buffers from a module as a dict."""
    params, buffers = get_layer_params_buffers(layer)
    return params | buffers
```
**EN:** Function `get_layer_tensors` provides a reusable helper around the module's main workflow. The docstring highlights: Get all parameters and buffers from a module as a dict. Key calls such as `get_layer_params_buffers` show the concrete execution path.
**CN:** Function `get_layer_tensors` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Get all parameters and buffers from a module as a dict. 像 `get_layer_params_buffers` 这样的关键调用展示了该代码块的具体执行路径。

### Function `get_layer_params_buffers` (lines 25-30)
```python
def get_layer_params_buffers(layer: torch.nn.Module) -> LayerTensors:
    """Get all parameters and buffers of a module as a tuple of dicts."""
    return (
        {name: param for name, param in layer._parameters.items() if param is not None},
        {name: buffer for name, buffer in layer._buffers.items() if buffer is not None},
    )
```
**EN:** Function `get_layer_params_buffers` provides a reusable helper around the module's main workflow. The docstring highlights: Get all parameters and buffers of a module as a tuple of dicts. Key calls such as `layer._parameters.items`, `layer._buffers.items` show the concrete execution path.
**CN:** Function `get_layer_params_buffers` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Get all parameters and buffers of a module as a tuple of dicts. 像 `layer._parameters.items`, `layer._buffers.items` 这样的关键调用展示了该代码块的具体执行路径。

### Function `get_layer_size` (lines 33-45)
```python
def get_layer_size(layer: torch.nn.Module) -> int:
    """Calculate total number of elements across loadable tensors in a layer.

    Excludes SKIP_TENSORS (e.g. _expert_map) which are never moved to meta
    device and never loaded via weight_loader during layerwise reload.
    """
    from .meta import SKIP_TENSORS

    return sum(
        tensor.numel()
        for name, tensor in get_layer_tensors(layer).items()
        if name not in SKIP_TENSORS
    )
```
**EN:** Function `get_layer_size` provides a reusable helper around the module's main workflow. The docstring highlights: Calculate total number of elements across loadable tensors in a layer. Key calls such as `sum`, `tensor.numel`, `get_layer_tensors(layer).items`, `get_layer_tensors` show the concrete execution path.
**CN:** Function `get_layer_size` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Calculate total number of elements across loadable tensors in a layer. 像 `sum`, `tensor.numel`, `get_layer_tensors(layer).items`, `get_layer_tensors` 这样的关键调用展示了该代码块的具体执行路径。

### Function `has_device_tensors` (lines 48-58)
```python
def has_device_tensors(bound_args: BoundArguments) -> bool:
    """
    Return True if the loaded weights exist on an accelerator device

    :param bound_args: args to load weights
    :return: True if weights are on accelerator device
    """
    return any(
        isinstance(value, torch.Tensor) and value.device.type not in ("meta", "cpu")
        for value in bound_args.arguments.values()
    )
```
**EN:** Function `has_device_tensors` provides a reusable helper around the module's main workflow. The docstring highlights: Return True if the loaded weights exist on an accelerator device :param bound_args: args to load weights :return: True if weights are on accelerator device Key calls such as `any`, `isinstance`, `bound_args.arguments.values` show the concrete execution path.
**CN:** Function `has_device_tensors` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Return True if the loaded weights exist on an accelerator device :param bound_args: args to load weights :return: True if weights are on accelerator device 像 `any`, `isinstance`, `bound_args.arguments.values` 这样的关键调用展示了该代码块的具体执行路径。

### Function `get_info_size` (lines 61-73)
```python
def get_info_size(info: LayerReloadingInfo) -> int:
    """
    Calculate the number of bytes used by loaded weights for a given layer

    :param info: layerwise info to get size of
    :return: number of bytes used by loaded weights
    """
    return sum(
        value.nbytes
        for _, args in info.loaded_weights
        for value in args.arguments.values()
        if isinstance(value, torch.Tensor) and value.device.type not in ("meta", "cpu")
    )
```
**EN:** Function `get_info_size` provides a reusable helper around the module's main workflow. The docstring highlights: Calculate the number of bytes used by loaded weights for a given layer :param info: layerwise info to get size of :return: number of bytes used by loaded weights Key calls such as `sum`, `args.arguments.values`, `isinstance` show the concrete execution path.
**CN:** Function `get_info_size` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Calculate the number of bytes used by loaded weights for a given layer :param info: layerwise info to get size of :return: number of bytes used by loaded weights 像 `sum`, `args.arguments.values`, `isinstance` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from inspect import BoundArguments`
- **Third-party / 第三方**: `import torch`
- **vLLM internal / vLLM 内部依赖**: `from .types import LayerReloadingInfo, LayerTensors`
