# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/platforms/__init__.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for init in the multimodal generation stack. Key symbols include `cuda_platform_plugin`, `mps_platform_plugin`, `cpu_platform_plugin`. / 该模块包含多模态生成体系中与 init 相关的运行时支持代码。 关键符号包括 `cuda_platform_plugin`, `mps_platform_plugin`, `cpu_platform_plugin`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
# Adapted from vllm: https://github.com/vllm-project/vllm/blob/v0.7.3/vllm/platforms/__init__.py

import traceback
from typing import TYPE_CHECKING

# imported by other files, do not remove
from sglang.multimodal_gen.runtime.platforms.interface import (  # noqa: F401
    AttentionBackendEnum,
    Platform,
    PlatformEnum,
)
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.multimodal_gen.utils import resolve_obj_by_qualname

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 21-58: Function `cuda_platform_plugin` / 函数 `cuda_platform_plugin`
```python
def cuda_platform_plugin() -> str | None:
    is_cuda = False

    try:
        from sglang.multimodal_gen.utils import import_pynvml

        pynvml = import_pynvml()  # type: ignore[no-untyped-call]
        pynvml.nvmlInit()
        try:
            # NOTE: Edge case: sgl_diffusion cpu build on a GPU machine.
            # Third-party pynvml can be imported in cpu build,
            # we need to check if sgl_diffusion is built with cpu too.
            # Otherwise, sgl_diffusion will always activate cuda plugin
            # on a GPU machine, even if in a cpu build.
# ...

    return (
        "sglang.multimodal_gen.runtime.platforms.cuda.CudaPlatform" if is_cuda else None
    )
```
**EN:** This function drives `cuda_platform_plugin`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `cuda_platform_plugin`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 61-74: Function `mps_platform_plugin` / 函数 `mps_platform_plugin`
```python
def mps_platform_plugin() -> str | None:
    """Detect if MPS (Metal Performance Shaders) is available on macOS."""
    is_mps = False

    try:
        import torch

        if torch.backends.mps.is_available():
            is_mps = True
            logger.debug("MPS (Metal Performance Shaders) is available")
    except Exception as e:
        logger.debug("MPS detection failed: %s", e)

    return "sglang.multimodal_gen.runtime.platforms.mps.MpsPlatform" if is_mps else None
```
**EN:** This function drives `mps_platform_plugin`. Detect if MPS (Metal Performance Shaders) is available on macOS.
**CN:** 这个函数负责 `mps_platform_plugin`。 文档字符串说明：Detect if MPS (Metal Performance Shaders) is available on macOS.

### Lines 77-80: Function `cpu_platform_plugin` / 函数 `cpu_platform_plugin`
```python
def cpu_platform_plugin() -> str | None:
    """Detect if CPU platform should be used."""
    # CPU is always available as a fallback
    return "sglang.multimodal_gen.runtime.platforms.cpu.CpuPlatform"
```
**EN:** This function drives `cpu_platform_plugin`. Detect if CPU platform should be used.
**CN:** 这个函数负责 `cpu_platform_plugin`。 文档字符串说明：Detect if CPU platform should be used.

### Lines 83-101: Function `rocm_platform_plugin` / 函数 `rocm_platform_plugin`
```python
def rocm_platform_plugin() -> str | None:
    is_rocm = False

    try:
        import amdsmi

        amdsmi.amdsmi_init()
        try:
            if len(amdsmi.amdsmi_get_processor_handles()) > 0:
                is_rocm = True
                logger.debug("ROCm platform is available")
        finally:
            amdsmi.amdsmi_shut_down()
    except Exception as e:
        logger.debug("ROCm platform is unavailable: %s", e)

    return (
        "sglang.multimodal_gen.runtime.platforms.rocm.RocmPlatform" if is_rocm else None
    )
```
**EN:** This function drives `rocm_platform_plugin`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `rocm_platform_plugin`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 104-119: Function `npu_platform_plugin` / 函数 `npu_platform_plugin`
```python
def npu_platform_plugin() -> str | None:
    is_npu = False

    try:
        import torch

        if torch.npu.is_available():
            is_npu = True
            logger.debug("NPU is available")
    except Exception as e:
        logger.debug("NPU detection failed: %s", e)
    return (
        "sglang.multimodal_gen.runtime.platforms.npu.NPUPlatformBase"
        if is_npu
        else None
    )
```
**EN:** This function drives `npu_platform_plugin`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `npu_platform_plugin`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 122-138: Function `musa_platform_plugin` / 函数 `musa_platform_plugin`
```python
def musa_platform_plugin() -> str | None:
    is_musa = False

    try:
        import pymtml

        pymtml.mtmlLibraryInit()
        try:
            is_musa = pymtml.mtmlLibraryCountDevice() > 0
        finally:
            pymtml.mtmlLibraryShutDown()
    except Exception as e:
        logger.debug("MUSA platform is unavailable: %s", e)

    return (
        "sglang.multimodal_gen.runtime.platforms.musa.MusaPlatform" if is_musa else None
    )
```
**EN:** This function drives `musa_platform_plugin`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `musa_platform_plugin`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 141-159: Function `xpu_platform_plugin` / 函数 `xpu_platform_plugin`
```python
def xpu_platform_plugin() -> str | None:
    """Detect if Intel XPU platform is available."""
    is_xpu = False

    try:
        import torch

        # Check if Intel Extension for PyTorch is available and XPU devices exist
        if hasattr(torch, "xpu") and torch.xpu.is_available():
            device_count = torch.xpu.device_count()
            if device_count > 0:
                is_xpu = True
                logger.info(
                    "Intel XPU platform is available with %d device(s)", device_count
                )
    except Exception as e:
        logger.info("Intel XPU platform is unavailable: %s", e)

    return "sglang.multimodal_gen.runtime.platforms.xpu.XpuPlatform" if is_xpu else None
```
**EN:** This function drives `xpu_platform_plugin`. Detect if Intel XPU platform is available.
**CN:** 这个函数负责 `xpu_platform_plugin`。 文档字符串说明：Detect if Intel XPU platform is available.

### Lines 160-170: Top-level configuration / 顶层配置
```python


builtin_platform_plugins = {
    "cuda": cuda_platform_plugin,
    "rocm": rocm_platform_plugin,
    "xpu": xpu_platform_plugin,
    "mps": mps_platform_plugin,
    "cpu": cpu_platform_plugin,
    "npu": npu_platform_plugin,
    "musa": musa_platform_plugin,
}
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 173-212: Function `resolve_current_platform_cls_qualname` / 函数 `resolve_current_platform_cls_qualname`
```python
def resolve_current_platform_cls_qualname() -> str:
    # TODO(will): if we need to support other platforms, we should consider if
    # vLLM's plugin architecture is suitable for our needs.

    # Try MPS first on macOS
    platform_cls_qualname = mps_platform_plugin()
    if platform_cls_qualname is not None:
        return platform_cls_qualname

    # Try Intel XPU
    platform_cls_qualname = xpu_platform_plugin()
    if platform_cls_qualname is not None:
        return platform_cls_qualname

# ...
    if platform_cls_qualname is not None:
        return platform_cls_qualname

    raise RuntimeError("No platform plugin found. Please check your " "installation.")
```
**EN:** This function drives `resolve_current_platform_cls_qualname`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `resolve_current_platform_cls_qualname`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 213-218: Top-level configuration / 顶层配置
```python


_current_platform: Platform | None = None
_init_trace: str = ""

current_platform: Platform
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 221-238: Function `__getattr__` / 函数 `__getattr__`
```python
def __getattr__(name: str):
    if name == "current_platform":
        # lazy init current_platform.
        # 1. out-of-tree platform plugins need `from sglang.multimodal_gen.runtime.platforms import
        #    Platform` so that they can inherit `Platform` class. Therefore,
        #    we cannot resolve `current_platform` during the import of
        #    `sglang.multimodal_gen.runtime.platforms`.
        global _current_platform
        if _current_platform is None:
            platform_cls_qualname = resolve_current_platform_cls_qualname()
            _current_platform = resolve_obj_by_qualname(platform_cls_qualname)()
            global _init_trace
            _init_trace = "".join(traceback.format_stack())
        return _current_platform
    elif name in globals():
        return globals()[name]
    else:
        raise AttributeError(f"No attribute named '{name}' exists in {__name__}.")
```
**EN:** This function drives `__getattr__` with inputs such as `name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `__getattr__`，主要处理 `name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 239-241: Registration and exports / 注册与导出
```python


__all__ = ["Platform", "PlatformEnum", "current_platform", "_init_trace"]
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Video generation flow / 视频生成流程
- Symbol `cuda_platform_plugin` anchors the module API / 符号 `cuda_platform_plugin` 构成该模块的核心 API
- Symbol `mps_platform_plugin` anchors the module API / 符号 `mps_platform_plugin` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.platforms.interface`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.utils`
- **External / 外部**: `torch`, `amdsmi`, `pymtml`
- **Stdlib / 标准库**: `traceback`, `typing`, `os`
