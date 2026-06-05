# cuda.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/platforms/cuda.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for cuda in the multimodal generation stack. Key symbols include `device_id_to_physical_device_id`, `with_nvml_context`, `CudaPlatformBase`. / 该模块包含多模态生成体系中与 cuda 相关的运行时支持代码。 关键符号包括 `device_id_to_physical_device_id`, `with_nvml_context`, `CudaPlatformBase`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-37: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
# Adapted from vllm: https://github.com/vllm-project/vllm/blob/v0.7.3/vllm/platforms/cuda.py
"""Code inside this file can safely assume cuda platform, e.g. importing
pynvml. However, it should not initialize cuda context.
"""

import os
from collections.abc import Callable
from functools import lru_cache, wraps
from typing import Any, TypeVar

import psutil
# ...

# pytorch 2.5 uses cudnn sdpa by default, which will cause crash on some models
# see https://github.com/huggingface/diffusers/issues/9704 for details
torch.backends.cuda.enable_cudnn_sdp(False)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 40-56: Function `device_id_to_physical_device_id` / 函数 `device_id_to_physical_device_id`
```python
def device_id_to_physical_device_id(device_id: int) -> int:
    if "CUDA_VISIBLE_DEVICES" in os.environ:
        device_ids = os.environ["CUDA_VISIBLE_DEVICES"].split(",")
        if device_ids == [""]:
            msg = (
                "CUDA_VISIBLE_DEVICES is set to empty string, which means"
                " GPU support is disabled. If you are using ray, please unset"
                " the environment variable `CUDA_VISIBLE_DEVICES` inside the"
                " worker/actor. "
                "Check https://github.com/vllm-project/vllm/issues/8402 for"
                " more information."
            )
            raise RuntimeError(msg)
        physical_device_id = device_ids[device_id]
        return int(physical_device_id)
    else:
        return device_id
```
**EN:** This function drives `device_id_to_physical_device_id` with inputs such as `device_id`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `device_id_to_physical_device_id`，主要处理 `device_id` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 59-68: Function `with_nvml_context` / 函数 `with_nvml_context`
```python
def with_nvml_context(fn: Callable[_P, _R]) -> Callable[_P, _R]:
    @wraps(fn)
    def wrapper(*args: _P.args, **kwargs: _P.kwargs) -> _R:
        pynvml.nvmlInit()
        try:
            return fn(*args, **kwargs)
        finally:
            pynvml.nvmlShutdown()

    return wrapper
```
**EN:** This function drives `with_nvml_context` with inputs such as `fn`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `with_nvml_context`，主要处理 `fn` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 71-463: Class `CudaPlatformBase` / 类 `CudaPlatformBase`
```python
class CudaPlatformBase(Platform):
    _enum = PlatformEnum.CUDA
    device_name: str = "cuda"
    device_type: str = "cuda"
    dispatch_key: str = "CUDA"
    device_control_env_var: str = "CUDA_VISIBLE_DEVICES"

    @classmethod
    def get_local_torch_device(cls) -> torch.device:
        return torch.device(f"cuda:{envs.LOCAL_RANK}")

    @classmethod
    def get_device_capability(cls, device_id: int = 0) -> DeviceCapability | None:
        raise NotImplementedError
# ...

    @classmethod
    def get_device_communicator_cls(cls) -> str:
        return "sglang.multimodal_gen.runtime.distributed.device_communicators.cuda_communicator.CudaCommunicator"  # noqa
```
**EN:** This class models `CudaPlatformBase` as a specialization of `Platform`. Important methods include `get_local_torch_device`, `get_device_capability`, `get_device_name`, `get_device_total_memory`.
**CN:** 该类实现 `CudaPlatformBase`，并继承/扩展 `Platform`。 其中较重要的方法包括 `get_local_torch_device`, `get_device_capability`, `get_device_name`, `get_device_total_memory`。

### Lines 470-568: Class `NvmlCudaPlatform` / 类 `NvmlCudaPlatform`
```python
class NvmlCudaPlatform(CudaPlatformBase):
    @classmethod
    @lru_cache(maxsize=8)
    @with_nvml_context
    def get_device_capability(cls, device_id: int = 0) -> DeviceCapability | None:
        try:
            physical_device_id = device_id_to_physical_device_id(device_id)
            handle = pynvml.nvmlDeviceGetHandleByIndex(physical_device_id)
            major, minor = pynvml.nvmlDeviceGetCudaComputeCapability(handle)
            return DeviceCapability(major=major, minor=minor)
        except RuntimeError:
            return None

    @classmethod
# ...
                    " make sure to set `CUDA_DEVICE_ORDER=PCI_BUS_ID` to "
                    "avoid unexpected behavior.",
                    ", ".join(device_names),
                )
```
**EN:** This class models `NvmlCudaPlatform` as a specialization of `CudaPlatformBase`. Important methods include `get_device_capability`, `has_device_capability`, `get_device_name`, `get_device_uuid`.
**CN:** 该类实现 `NvmlCudaPlatform`，并继承/扩展 `CudaPlatformBase`。 其中较重要的方法包括 `get_device_capability`, `has_device_capability`, `get_device_name`, `get_device_uuid`。

### Lines 571-593: Class `NonNvmlCudaPlatform` / 类 `NonNvmlCudaPlatform`
```python
class NonNvmlCudaPlatform(CudaPlatformBase):
    @classmethod
    def get_device_capability(cls, device_id: int = 0) -> DeviceCapability:
        major, minor = torch.cuda.get_device_capability(device_id)
        return DeviceCapability(major=major, minor=minor)

    @classmethod
    def get_device_name(cls, device_id: int = 0) -> str:
        return str(torch.cuda.get_device_name(device_id))

    @classmethod
    @lru_cache(maxsize=1)
    def get_device_total_memory(cls, device_id: int = 0) -> int:
        device_props = torch.cuda.get_device_properties(device_id)
# ...
            "NVLink detection not possible, as context support was"
            " not found. Assuming no NVLink available."
        )
        return False
```
**EN:** This class models `NonNvmlCudaPlatform` as a specialization of `CudaPlatformBase`. Important methods include `get_device_capability`, `get_device_name`, `get_device_total_memory`, `is_full_nvlink`.
**CN:** 该类实现 `NonNvmlCudaPlatform`，并继承/扩展 `CudaPlatformBase`。 其中较重要的方法包括 `get_device_capability`, `get_device_name`, `get_device_total_memory`, `is_full_nvlink`。

### Lines 594-618: Imports and module setup / 导入与模块初始化
```python


# Autodetect either NVML-enabled or non-NVML platform
# based on whether NVML is available.
nvml_available = False
try:
    try:
        pynvml.nvmlInit()
        nvml_available = True
    except Exception:
        # On Jetson, NVML is not supported.
        nvml_available = False
finally:
    if nvml_available:
# ...
    if not isinstance(pynvml, _MockModule):
        CudaPlatform.log_warnings()
except ModuleNotFoundError:
    CudaPlatform.log_warnings()
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen`, `sglang.multimodal_gen.runtime.platforms.interface`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.utils`, `sglang.multimodal_gen.runtime.layers.attention.backends.flash_attn`, `sglang.multimodal_gen.runtime.layers.attention.backends.sliding_tile_attn`, `sglang.multimodal_gen.runtime.layers.attention.backends.sage_attn`, `sglang.multimodal_gen.runtime.layers.attention.backends.sage_attn3`
- **External / 外部**: `psutil`, `torch`, `typing_extensions`, `sphinx.ext.autodoc.mock`, `flashinfer`, `sgl_kernel`, `torch.distributed`, `st_attn`
- **Stdlib / 标准库**: `os`, `collections.abc`, `functools`, `typing`
