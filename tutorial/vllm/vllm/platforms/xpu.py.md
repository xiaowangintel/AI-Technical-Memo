# xpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/platforms/xpu.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines platform-specific behavior and capability checks for hardware backends. / 定义硬件后端的平台特定行为与能力检查逻辑。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-29)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import contextlib
import os
from typing import TYPE_CHECKING

import torch

# import custom ops, trigger op registration
import vllm_xpu_kernels._C  # noqa
import vllm_xpu_kernels._moe_C  # noqa
import vllm_xpu_kernels._xpu_C  # noqa

import vllm.envs as envs
from vllm.logger import init_logger
from vllm.utils.torch_utils import supports_xpu_graph
from vllm.v1.attention.backends.registry import AttentionBackendEnum

from .interface import DeviceCapability, Platform, PlatformEnum

if TYPE_CHECKING:
    from vllm.config import VllmConfig
    from vllm.config.kernel import IrOpPriorityConfig
    from vllm.v1.attention.selector import AttentionSelectorConfig
else:
    VllmConfig = None

logger = init_logger(__name__)
```
**EN:** Sets up the module with standard-library support such as `contextlib`, `os`, `typing`, external packages such as `torch`, `vllm_xpu_kernels._C`, `vllm_xpu_kernels._moe_C`, vLLM modules such as `vllm.envs`, `vllm.logger`, `vllm.utils.torch_utils`. It prepares the symbols later used by `XPUPlatform`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.envs`, `vllm.logger`, `vllm.utils.torch_utils` 等 vLLM 内部依赖。 这些准备工作为后续的 `XPUPlatform` 提供上下文。

### XPUPlatform overview (lines 32-411)
```python
class XPUPlatform(Platform):
    _enum = PlatformEnum.XPU
    device_name: str = "xpu"
    device_type: str = "xpu"
    dispatch_key: str = "XPU"
    # Intel XPU's device key is "GPU" for Ray.
    # see https://github.com/ray-project/ray/blob/6a5eb5865eeb9ccf058a79b44f107e327e360673/python/ray/_private/accelerators/intel_gpu.py#L20 # noqa: E501
    ray_device_key: str = "GPU"
    dist_backend: str = "xccl"  # xccl only
    device_control_env_var: str = "ZE_AFFINITY_MASK"

    @classmethod
    def import_kernels(cls) -> None:
        # Do not import vllm._C
        with contextlib.suppress(ImportError):
            import vllm._moe_C  # noqa: F401

    @classmethod
    def get_attn_backend_cls(
        cls,
        selected_backend: "AttentionBackendEnum",
        attn_selector_config: "AttentionSelectorConfig",
        num_heads: int | None = None,
    ) -> str:
        from vllm.v1.attention.backends.utils import set_kv_cache_layout

        set_kv_cache_layout("NHD")
    # ...
```
**EN:** Defines the `XPUPlatform` class used by this module. It extends `Platform`. Key methods include `import_kernels`, `get_attn_backend_cls`, `get_supported_vit_attn_backends`, `get_vit_attn_backend`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `XPUPlatform` 是该文件中的核心类，用于封装与 `XPUPlatform` 相关的状态和行为。 它继承自 `Platform`。 关键方法包括 `import_kernels`, `get_attn_backend_cls`, `get_supported_vit_attn_backends`, `get_vit_attn_backend`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### XPUPlatform.check_and_update_config (lines 173-253)
```python
    def check_and_update_config(cls, vllm_config: VllmConfig) -> None:
        parallel_config = vllm_config.parallel_config

        # lazy import to avoid circular import
        from vllm.config import CUDAGraphMode

        compilation_config = vllm_config.compilation_config
        if compilation_config.compile_sizes is None:
            compilation_config.compile_sizes = []

        attention_config = vllm_config.attention_config
        if attention_config.backend is None:
            attention_config.backend = AttentionBackendEnum.FLASH_ATTN
        if not supports_xpu_graph():
            compilation_config.cudagraph_mode = CUDAGraphMode.NONE
            logger.warning(
                "XPU Graph is not supported in the current PyTorch version, "
                "disabling cudagraph_mode."
            )
        elif not envs.VLLM_XPU_ENABLE_XPU_GRAPH:
    # ...
        # ref. https://openucx.readthedocs.io/en/master/faq.html
        os.environ["UCX_MEMTYPE_CACHE"] = "n"

        # spawn is the only supported multiprocessing method on XPU
        if "VLLM_WORKER_MULTIPROC_METHOD" not in os.environ:
            os.environ["VLLM_WORKER_MULTIPROC_METHOD"] = "spawn"
```
**EN:** `check_and_update_config` checks whether a runtime condition is satisfied. It mainly works with `vllm_config`. Inside the body, it relies on `fusion_passes_to_disable.items`, `supports_xpu_graph`, `logger.warning` to complete the main steps.
**CN:** `check_and_update_config` 负责检查运行时条件是否满足。 它主要处理 `vllm_config` 等参数。 实现过程中会调用 `fusion_passes_to_disable.items`, `supports_xpu_graph`, `logger.warning` 等函数完成关键步骤。

### XPUPlatform.update_block_size_for_backend (lines 256-304)
```python
    def update_block_size_for_backend(cls, vllm_config: "VllmConfig") -> None:
        super().update_block_size_for_backend(vllm_config)
        from vllm.config.vllm import get_layers_from_vllm_config
        from vllm.model_executor.layers.attention_layer_base import (
            AttentionLayerBase,
        )
        from vllm.utils.math_utils import cdiv

        cache_config = vllm_config.cache_config
        # special fix for GDN since kernel only supports block size dividable by 64
        attn_layers = get_layers_from_vllm_config(
            vllm_config,
            AttentionLayerBase,  # type: ignore[type-abstract]
        )

        kernel_block_size = None
        for layer in attn_layers.values():
            b = layer.get_attn_backend()
            if b.get_name() == "GDN_ATTN":
                kernel_block_size = 64
    # ...
            "set mamba_page_size_padded to %d bytes accordingly, before was %d bytes.",
            new_block_size,
            kernel_block_size,
            cache_config.mamba_page_size_padded,
            original_mamba_page_size_padded,
        )
```
**EN:** `update_block_size_for_backend` updates module state according to new inputs. It mainly works with `vllm_config`. Inside the body, it relies on `super.update_block_size_for_backend`, `get_layers_from_vllm_config`, `attn_layers.values` to complete the main steps.
**CN:** `update_block_size_for_backend` 负责根据新输入更新模块状态。 它主要处理 `vllm_config` 等参数。 实现过程中会调用 `super.update_block_size_for_backend`, `get_layers_from_vllm_config`, `attn_layers.values` 等函数完成关键步骤。

### XPUPlatform.get_attn_backend_cls (lines 50-96)
```python
    def get_attn_backend_cls(
        cls,
        selected_backend: "AttentionBackendEnum",
        attn_selector_config: "AttentionSelectorConfig",
        num_heads: int | None = None,
    ) -> str:
        from vllm.v1.attention.backends.utils import set_kv_cache_layout

        set_kv_cache_layout("NHD")
        logger.info(
            "Setting VLLM_KV_CACHE_LAYOUT to 'NHD' for XPU; "
            "only NHD layout is supported by XPU attention kernels."
        )

        # TurboQuant KV cache: route directly to TQ backend
        kv_cache_dtype = attn_selector_config.kv_cache_dtype
        if kv_cache_dtype is not None and kv_cache_dtype.startswith("turboquant_"):
            logger.info_once("Using TurboQuant attention backend.")
            return AttentionBackendEnum.TURBOQUANT.get_path()

    # ...
                f"Invalid attention backend for {cls.device_name}, "
                f"with use_mla: {attn_selector_config.use_mla}"
            )

        logger.info("Using Flash Attention backend.")
        return AttentionBackendEnum.FLASH_ATTN.get_path()
```
**EN:** `get_attn_backend_cls` retrieves data or state needed by the pipeline. It mainly works with `selected_backend`, `attn_selector_config`, `num_heads`. Inside the body, it relies on `set_kv_cache_layout`, `logger.info`, `AttentionBackendEnum.FLASH_ATTN.get_path` to complete the main steps.
**CN:** `get_attn_backend_cls` 负责获取流水线所需的数据或状态。 它主要处理 `selected_backend`, `attn_selector_config`, `num_heads` 等参数。 实现过程中会调用 `set_kv_cache_layout`, `logger.info`, `AttentionBackendEnum.FLASH_ATTN.get_path` 等函数完成关键步骤。

### XPUPlatform.get_vit_attn_backend (lines 107-125)
```python
    def get_vit_attn_backend(
        cls,
        head_size: int,
        dtype: torch.dtype,
        backend: "AttentionBackendEnum | None" = None,
    ) -> "AttentionBackendEnum":
        if backend is not None:
            assert backend in cls.get_supported_vit_attn_backends(), (
                f"Backend {backend} is not supported for vit attention. "
                f"Supported backends are: "
                f"{cls.get_supported_vit_attn_backends()}."
            )
            logger.info_once(f"Using backend {backend} for vit attention")
            return backend

        logger.info_once(
            f"Using backend {AttentionBackendEnum.FLASH_ATTN} for vit attention"
        )
        return AttentionBackendEnum.FLASH_ATTN
```
**EN:** `get_vit_attn_backend` retrieves data or state needed by the pipeline. It mainly works with `head_size`, `dtype`, `backend`. Inside the body, it relies on `logger.info_once`, `cls.get_supported_vit_attn_backends` to complete the main steps.
**CN:** `get_vit_attn_backend` 负责获取流水线所需的数据或状态。 它主要处理 `head_size`, `dtype`, `backend` 等参数。 实现过程中会调用 `logger.info_once`, `cls.get_supported_vit_attn_backends` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`XPUPlatform`**: Core class that organizes module behavior. / **`XPUPlatform`**：组织模块行为的核心类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: contextlib, os, typing
- **Third-party / 第三方**: torch, vllm_xpu_kernels._C, vllm_xpu_kernels._moe_C, vllm_xpu_kernels._xpu_C
- **Internal vLLM / vLLM 内部依赖**: vllm.envs, vllm.logger, vllm.utils.torch_utils, vllm.v1.attention.backends.registry, .interface, vllm.config, vllm.config.kernel, vllm.v1.attention.selector, vllm.v1.attention.backends.utils, vllm.config.vllm, vllm.model_executor.layers.attention_layer_base, vllm.utils.math_utils, vllm.config.compilation, vllm._moe_C
