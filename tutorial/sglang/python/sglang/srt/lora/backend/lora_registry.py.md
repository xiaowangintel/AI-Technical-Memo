# lora_registry.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/lora/backend/lora_registry.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a LoRA execution backend or backend interface for SGLang, defining how adapter weights are applied during model execution. It also exposes the data paths needed by CUDA-graph-friendly LoRA kernels. / 该文件实现了 SGLang 的 LoRA 执行后端或后端接口，定义了在模型执行期间如何应用适配器权重。它也提供了兼容 CUDA Graph 的 LoRA 内核所需的数据路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10: Module header, imports, and shared constants
```python
import logging
from typing import Type

from sglang.srt.lora.backend.base_backend import BaseLoRABackend

logger = logging.getLogger(__name__)

LORA_SUPPORTED_BACKENDS = {}
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 11-16: register lora backend function
```python
def register_lora_backend(name):
    def decorator(fn):
        LORA_SUPPORTED_BACKENDS[name] = fn
        return fn

    return decorator
```
**EN:** This block uses `register_lora_backend` to register components for later lookup. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `register_lora_backend` 来注册组件以供后续查找。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 19-23: create triton backend function
```python
@register_lora_backend("triton")
def create_triton_backend():
    from sglang.srt.lora.backend.triton_backend import TritonLoRABackend

    return TritonLoRABackend
```
**EN:** This block uses `create_triton_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `create_triton_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 26-30: create triton csgmv backend function
```python
@register_lora_backend("csgmv")
def create_triton_csgmv_backend():
    from sglang.srt.lora.backend.chunked_backend import ChunkedSgmvLoRABackend

    return ChunkedSgmvLoRABackend
```
**EN:** This block uses `create_triton_csgmv_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `create_triton_csgmv_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 33-37: create ascend backend function
```python
@register_lora_backend("ascend")
def create_ascend_backend():
    from sglang.srt.lora.backend.ascend_backend import AscendLoRABackend

    return AscendLoRABackend
```
**EN:** This block uses `create_ascend_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `create_ascend_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 40-44: create torch native backend function
```python
@register_lora_backend("torch_native")
def create_torch_native_backend():
    from sglang.srt.lora.backend.torch_backend import TorchNativeLoRABackend

    return TorchNativeLoRABackend
```
**EN:** This block uses `create_torch_native_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `create_torch_native_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 47-51: create flashinfer backend function
```python
@register_lora_backend("flashinfer")
def create_flashinfer_backend():
    raise ValueError(
        "FlashInfer LoRA backend has been deprecated, please use `triton` instead."
    )
```
**EN:** This block uses `create_flashinfer_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `create_flashinfer_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 54-61: get backend from name function
```python
def get_backend_from_name(name: str) -> Type[BaseLoRABackend]:
    """
    Get corresponding backend class from backend's name
    """
    if name not in LORA_SUPPORTED_BACKENDS:
        raise ValueError(f"Invalid backend: {name}")
    lora_backend = LORA_SUPPORTED_BACKENDS[name]()
    return lora_backend
```
**EN:** This block uses `get_backend_from_name` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `get_backend_from_name` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- LoRA adapter execution / LoRA 适配器执行
- Backend abstraction / 后端抽象
- Registration and lookup / 注册与查找

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.lora.backend.ascend_backend`
- `sglang.srt.lora.backend.base_backend`
- `sglang.srt.lora.backend.chunked_backend`
- `sglang.srt.lora.backend.torch_backend`
- `sglang.srt.lora.backend.triton_backend`
### External / 外部
- `logging` (stdlib)
- `typing` (stdlib)
