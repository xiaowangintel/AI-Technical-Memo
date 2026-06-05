# musa.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/platforms/musa.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for musa in the multimodal generation stack. Key symbols include `device_id_to_physical_device_id`, `with_mtml_context`, `MusaPlatformBase`. / 该模块包含多模态生成体系中与 musa 相关的运行时支持代码。 关键符号包括 `device_id_to_physical_device_id`, `with_mtml_context`, `MusaPlatformBase`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: Imports and module setup / 导入与模块初始化
```python
"""
This file is a platform abstraction for MThreads (MUSA) GPUs,
adjusted to match the structure and interface of `cuda.py`.
"""

import os
from collections.abc import Callable
from functools import lru_cache, wraps
from typing import Any, TypeVar

import psutil
import pymtml

# isort: off
# ...
logger = init_logger(__name__)

_P = ParamSpec("_P")
_R = TypeVar("_R")
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 36-52: Function `device_id_to_physical_device_id` / 函数 `device_id_to_physical_device_id`
```python
def device_id_to_physical_device_id(device_id: int) -> int:
    if "MUSA_VISIBLE_DEVICES" in os.environ:
        device_ids = os.environ["MUSA_VISIBLE_DEVICES"].split(",")
        if device_ids == [""]:
            msg = (
                "MUSA_VISIBLE_DEVICES is set to empty string, which means"
                " GPU support is disabled. If you are using ray, please unset"
                " the environment variable `MUSA_VISIBLE_DEVICES` inside the"
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

### Lines 55-64: Function `with_mtml_context` / 函数 `with_mtml_context`
```python
def with_mtml_context(fn: Callable[_P, _R]) -> Callable[_P, _R]:
    @wraps(fn)
    def wrapper(*args: _P.args, **kwargs: _P.kwargs) -> _R:
        pymtml.nvmlInit()
        try:
            return fn(*args, **kwargs)
        finally:
            pymtml.nvmlShutdown()

    return wrapper
```
**EN:** This function drives `with_mtml_context` with inputs such as `fn`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `with_mtml_context`，主要处理 `fn` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 67-233: Class `MusaPlatformBase` / 类 `MusaPlatformBase`
```python
class MusaPlatformBase(Platform):
    _enum = PlatformEnum.MUSA
    device_name: str = "musa"
    device_type: str = "musa"
    dispatch_key: str = "MUSA"
    device_control_env_var: str = "MUSA_VISIBLE_DEVICES"

    @classmethod
    @lru_cache(maxsize=1)
    def is_float64_supported(cls) -> bool:
        return False

    @classmethod
    def get_local_torch_device(cls) -> torch.device:
# ...

    @classmethod
    def get_device_communicator_cls(cls) -> str:
        return "sglang.multimodal_gen.runtime.distributed.device_communicators.cuda_communicator.CudaCommunicator"  # noqa
```
**EN:** This class models `MusaPlatformBase` as a specialization of `Platform`. Important methods include `is_float64_supported`, `get_local_torch_device`, `get_device_capability`, `get_device_name`.
**CN:** 该类实现 `MusaPlatformBase`，并继承/扩展 `Platform`。 其中较重要的方法包括 `is_float64_supported`, `get_local_torch_device`, `get_device_capability`, `get_device_name`。

### Lines 240-335: Class `MtmlMusaPlatform` / 类 `MtmlMusaPlatform`
```python
class MtmlMusaPlatform(MusaPlatformBase):
    @classmethod
    @lru_cache(maxsize=8)
    @with_mtml_context
    def get_device_capability(cls, device_id: int = 0) -> DeviceCapability | None:
        try:
            physical_device_id = device_id_to_physical_device_id(device_id)
            handle = pymtml.nvmlDeviceGetHandleByIndex(physical_device_id)
            major, minor = pymtml.nvmlDeviceGetCudaComputeCapability(handle)
            return DeviceCapability(major=major, minor=minor)
        except RuntimeError:
            return None

    @classmethod
# ...
                    " make sure to set `MUSA_DEVICE_ORDER=PCI_BUS_ID` to "
                    "avoid unexpected behavior.",
                    ", ".join(device_names),
                )
```
**EN:** This class models `MtmlMusaPlatform` as a specialization of `MusaPlatformBase`. Important methods include `get_device_capability`, `has_device_capability`, `get_device_name`, `get_device_uuid`.
**CN:** 该类实现 `MtmlMusaPlatform`，并继承/扩展 `MusaPlatformBase`。 其中较重要的方法包括 `get_device_capability`, `has_device_capability`, `get_device_name`, `get_device_uuid`。

### Lines 338-360: Class `NonMtmlMusaPlatform` / 类 `NonMtmlMusaPlatform`
```python
class NonMtmlMusaPlatform(MusaPlatformBase):
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
            "MTLink detection not possible, as context support was"
            " not found. Assuming no MTLink available."
        )
        return False
```
**EN:** This class models `NonMtmlMusaPlatform` as a specialization of `MusaPlatformBase`. Important methods include `get_device_capability`, `get_device_name`, `get_device_total_memory`, `is_full_mtlink`.
**CN:** 该类实现 `NonMtmlMusaPlatform`，并继承/扩展 `MusaPlatformBase`。 其中较重要的方法包括 `get_device_capability`, `get_device_name`, `get_device_total_memory`, `is_full_mtlink`。

### Lines 361-393: Imports and module setup / 导入与模块初始化
```python


# Autodetect either MTML-enabled or non-MTML platform
# based on whether MTML is available.
mtml_available = False

if "MUSA_DISABLE_MTML" not in os.environ:
    try:
        try:
            pymtml.nvmlInit()
            mtml_available = True
        except Exception:
            mtml_available = False
    finally:
# ...
    print(MusaPlatform.get_device_name())
    print(MusaPlatform.get_device_capability())
    print(MusaPlatform.get_device_total_memory())
    print(MusaPlatform.is_full_mtlink([0, 1, 2, 3, 4, 5, 6, 7]))
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行
- Symbol `device_id_to_physical_device_id` anchors the module API / 符号 `device_id_to_physical_device_id` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen`, `sglang.multimodal_gen.runtime.platforms.interface`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.runtime.layers.attention.backends.flash_attn`, `sglang.multimodal_gen.runtime.layers.attention.backends.sage_attn`
- **External / 外部**: `psutil`, `pymtml`, `torch`, `torchada`, `typing_extensions`, `sphinx.ext.autodoc.mock`, `torch.distributed`, `sageattention`
- **Stdlib / 标准库**: `os`, `collections.abc`, `functools`, `typing`
