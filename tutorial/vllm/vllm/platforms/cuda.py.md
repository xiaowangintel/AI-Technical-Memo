# cuda.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/platforms/cuda.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Code inside this file can safely assume cuda platform, e.g / 该模块围绕 `cuda` 相关逻辑组织实现，提供与源文件相匹配的核心功能。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-49)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""Code inside this file can safely assume cuda platform, e.g. importing
pynvml. However, it should not initialize cuda context.
"""

from __future__ import annotations

import os
from collections.abc import Callable
from datetime import timedelta
from functools import cache, lru_cache, wraps
from typing import TYPE_CHECKING, TypeVar

import torch
from torch.distributed import PrefixStore, ProcessGroup
from torch.distributed.distributed_c10d import is_nccl_available
from typing_extensions import ParamSpec

# import custom ops, trigger op registration
import vllm._C  # noqa
import vllm._C_stable_libtorch  # noqa
import vllm.envs as envs
from vllm.logger import init_logger
    # ...

pynvml = import_pynvml()

# pytorch 2.5 uses cudnn sdpa by default, which will cause crash on some models
# see https://github.com/huggingface/diffusers/issues/9704 for details
torch.backends.cuda.enable_cudnn_sdp(False)
```
**EN:** Sets up the module with standard-library support such as `__future__`, `os`, `collections.abc`, external packages such as `torch`, `torch.distributed`, `torch.distributed.distributed_c10d`, vLLM modules such as `vllm._C`, `vllm._C_stable_libtorch`, `vllm.envs`. It prepares the symbols later used by `CudaPlatformBase`, `NvmlCudaPlatform`, `_cuda_device_count_stateless`, `_get_backend_priorities`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm._C`, `vllm._C_stable_libtorch`, `vllm.envs` 等 vLLM 内部依赖。 这些准备工作为后续的 `CudaPlatformBase`, `NvmlCudaPlatform`, `_cuda_device_count_stateless`, `_get_backend_priorities` 提供上下文。

### _cuda_device_count_stateless (lines 53-75)
```python
def _cuda_device_count_stateless(cuda_visible_devices: str | None = None) -> int:
    """Get number of CUDA devices, caching based on the value of CUDA_VISIBLE_DEVICES
    at the time of call.

    This should be used instead of torch.accelerator.device_count() unless
    CUDA_VISIBLE_DEVICES has already been set to the desired value.

    # This can be removed and simply replaced with torch.cuda.get_device_count
    # after https://github.com/pytorch/pytorch/pull/122815 is released."""
    # Note: cuda_visible_devices is not used, but we keep it as an argument for
    # LRU Cache purposes.

    # Code below is based on
    # https://github.com/pytorch/pytorch/blob/
    # c1cd946818442aca8c7f812b16d187ce1586c3bc/
    # torch/cuda/__init__.py#L831C1-L831C17
    import torch.cuda

    if not torch.cuda._is_compiled():
        return 0
    raw_count = torch.cuda._device_count_nvml()
    r = torch._C._cuda_getDeviceCount() if raw_count < 0 else raw_count
    return r
```
**EN:** `_cuda_device_count_stateless`: Get number of CUDA devices, caching based on the value of CUDA_VISIBLE_DEVICES at the time of call. It mainly works with `cuda_visible_devices`. Inside the body, it relies on `lru_cache`, `torch.cuda._device_count_nvml`, `torch.cuda._is_compiled` to complete the main steps.
**CN:** `_cuda_device_count_stateless` 负责实现本模块使用的辅助逻辑。 它主要处理 `cuda_visible_devices` 等参数。 实现过程中会调用 `lru_cache`, `torch.cuda._device_count_nvml`, `torch.cuda._is_compiled` 等函数完成关键步骤。

### _get_backend_priorities (lines 79-147)
```python
def _get_backend_priorities(
    use_mla: bool,
    device_capability: DeviceCapability,
    num_heads: int | None = None,
    kv_cache_dtype: CacheDType | None = None,
) -> list[AttentionBackendEnum]:
    """Get backend priorities with lazy import to avoid circular dependency."""
    if use_mla:
        if device_capability.major == 10:
            # Sparse MLA backend priorities
            # See https://github.com/vllm-project/vllm/issues/35807 for
            # benchmark results
            if kv_cache_dtype is not None and is_quantized_kv_cache(kv_cache_dtype):
                # Prefer FlashInfer for fp8 kv cache
                sparse_backends = [
                    AttentionBackendEnum.FLASHINFER_MLA_SPARSE,
                    AttentionBackendEnum.FLASHMLA_SPARSE,
                ]
            else:
                # BF16 KV Cache
                # Prefer FlashInfer at low head counts (FlashMLA uses padding)
                if num_heads is not None and num_heads <= 16:
                    sparse_backends = [
                        AttentionBackendEnum.FLASHINFER_MLA_SPARSE,
    # ...
                AttentionBackendEnum.FLASH_ATTN,
                AttentionBackendEnum.FLASHINFER,
                AttentionBackendEnum.TRITON_ATTN,
                AttentionBackendEnum.FLEX_ATTENTION,
                AttentionBackendEnum.TURBOQUANT,
            ]
```
**EN:** `_get_backend_priorities`: Get backend priorities with lazy import to avoid circular dependency. It mainly works with `use_mla`, `device_capability`, `num_heads`, `kv_cache_dtype`. Inside the body, it relies on `is_quantized_kv_cache` to complete the main steps.
**CN:** `_get_backend_priorities` 负责获取流水线所需的数据或状态。 它主要处理 `use_mla`, `device_capability`, `num_heads`, `kv_cache_dtype` 等参数。 实现过程中会调用 `is_quantized_kv_cache` 等函数完成关键步骤。

### with_nvml_context (lines 150-159)
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
**EN:** `with_nvml_context` implements helper logic used by this module. It mainly works with `fn`. Inside the body, it relies on `wraps`, `pynvml.nvmlInit`, `fn` to complete the main steps.
**CN:** `with_nvml_context` 负责实现本模块使用的辅助逻辑。 它主要处理 `fn` 等参数。 实现过程中会调用 `wraps`, `pynvml.nvmlInit`, `fn` 等函数完成关键步骤。

### CudaPlatformBase overview (lines 162-587)
```python
class CudaPlatformBase(Platform):
    _enum = PlatformEnum.CUDA
    device_name: str = "cuda"
    device_type: str = "cuda"
    dispatch_key: str = "CUDA"
    ray_device_key: str = "GPU"
    dist_backend: str = "nccl"
    device_control_env_var: str = "CUDA_VISIBLE_DEVICES"
    ray_noset_device_env_vars: list[str] = [
        "RAY_EXPERIMENTAL_NOSET_CUDA_VISIBLE_DEVICES",
    ]

    @property
    def supported_dtypes(self) -> list[torch.dtype]:
        if self.has_device_capability(80):
            # Ampere and Hopper or later NVIDIA GPUs.
            return [torch.bfloat16, torch.float16, torch.float32]
        if self.has_device_capability(60):
            # Pascal, Volta and Turing NVIDIA GPUs, BF16 is not supported
            return [torch.float16, torch.float32]
        # Kepler and Maxwell NVIDIA GPUs, only FP32 is supported,
        # though vLLM doesn't support these GPUs.
        return [torch.float32]

    @classmethod
    def set_device(cls, device: torch.device) -> None:
        """
    # ...
```
**EN:** Defines the `CudaPlatformBase` class used by this module. It extends `Platform`. Key methods include `supported_dtypes`, `set_device`, `manual_seed_all`, `get_device_capability`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `CudaPlatformBase` 是该文件中的核心类，用于封装与 `CudaPlatformBase` 相关的状态和行为。 它继承自 `Platform`。 关键方法包括 `supported_dtypes`, `set_device`, `manual_seed_all`, `get_device_capability`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### CudaPlatformBase.get_attn_backend_cls (lines 287-378)
```python
    def get_attn_backend_cls(
        cls,
        selected_backend: AttentionBackendEnum | None,
        attn_selector_config: AttentionSelectorConfig,
        num_heads: int | None = None,
    ) -> str:
        device_capability = cls.get_device_capability()
        assert device_capability is not None

        # First try checking just the selected backend, if there is one.
        if selected_backend is not None:
            try:
                backend_class = selected_backend.get_class()
                invalid_reasons = backend_class.validate_configuration(
                    device_capability=device_capability,
                    **attn_selector_config._asdict(),
                )
            except ImportError:
                invalid_reasons = ["ImportError"]
            if invalid_reasons:
    # ...
            "Using %s attention backend out of potential backends: %s.",
            selected_backend.name,
            "[" + ", ".join(f"'{b[0].name}'" for b in valid_backends_priorities) + "]",
        )

        return selected_backend.get_path()
```
**EN:** `get_attn_backend_cls` retrieves data or state needed by the pipeline. It mainly works with `selected_backend`, `attn_selector_config`, `num_heads`. Inside the body, it relies on `cls.get_device_capability`, `cls.get_valid_backends`, `attn_selector_config.__repr__` to complete the main steps.
**CN:** `get_attn_backend_cls` 负责获取流水线所需的数据或状态。 它主要处理 `selected_backend`, `attn_selector_config`, `num_heads` 等参数。 实现过程中会调用 `cls.get_device_capability`, `cls.get_valid_backends`, `attn_selector_config.__repr__` 等函数完成关键步骤。

### CudaPlatformBase.get_vit_attn_backend (lines 398-434)
```python
    def get_vit_attn_backend(
        cls,
        head_size: int,
        dtype: torch.dtype,
        backend: AttentionBackendEnum | None = None,
    ) -> AttentionBackendEnum:
        if backend is not None:
            assert backend in cls.get_supported_vit_attn_backends(), (
                f"Backend {backend} is not supported for vit attention. "
                f"Supported backends are: {cls.get_supported_vit_attn_backends()}"
            )
            logger.info_once(f"Using backend {backend} for vit attention")
            return backend

        cc = cls.get_device_capability()
        for vit_attn_backend in cls.get_supported_vit_attn_backends():
            if vit_attn_backend == AttentionBackendEnum.TORCH_SDPA:
                return vit_attn_backend
            try:
                backend_class = vit_attn_backend.get_class()
    # ...
                    )
                    return vit_attn_backend
            except ImportError:
                pass

        return AttentionBackendEnum.TORCH_SDPA
```
**EN:** `get_vit_attn_backend` retrieves data or state needed by the pipeline. It mainly works with `head_size`, `dtype`, `backend`. Inside the body, it relies on `cls.get_device_capability`, `cls.get_supported_vit_attn_backends`, `logger.info_once` to complete the main steps.
**CN:** `get_vit_attn_backend` 负责获取流水线所需的数据或状态。 它主要处理 `head_size`, `dtype`, `backend` 等参数。 实现过程中会调用 `cls.get_device_capability`, `cls.get_supported_vit_attn_backends`, `logger.info_once` 等函数完成关键步骤。

### CudaPlatformBase.get_valid_backends (lines 252-284)
```python
    def get_valid_backends(
        cls,
        device_capability: DeviceCapability,
        attn_selector_config: AttentionSelectorConfig,
        num_heads: int | None = None,
    ) -> tuple[
        list[tuple[AttentionBackendEnum, int]],
        dict[AttentionBackendEnum, tuple[int, list[str]]],
    ]:
        valid_backends_priorities = []
        invalid_reasons: dict[AttentionBackendEnum, tuple[int, list[str]]] = {}

        backend_priorities = _get_backend_priorities(
            attn_selector_config.use_mla,
            device_capability,
            num_heads,
            attn_selector_config.kv_cache_dtype,
        )
        for priority, backend in enumerate(backend_priorities):
            try:
    # ...
            if invalid_reasons_i:
                invalid_reasons[backend] = (priority, invalid_reasons_i)
            else:
                valid_backends_priorities.append((backend, priority))

        return valid_backends_priorities, invalid_reasons
```
**EN:** `get_valid_backends` retrieves data or state needed by the pipeline. It mainly works with `device_capability`, `attn_selector_config`, `num_heads`. Inside the body, it relies on `_get_backend_priorities`, `backend.get_class`, `backend_class.validate_configuration` to complete the main steps.
**CN:** `get_valid_backends` 负责获取流水线所需的数据或状态。 它主要处理 `device_capability`, `attn_selector_config`, `num_heads` 等参数。 实现过程中会调用 `_get_backend_priorities`, `backend.get_class`, `backend_class.validate_configuration` 等函数完成关键步骤。

### CudaPlatformBase.check_and_update_config (lines 222-241)
```python
    def check_and_update_config(cls, vllm_config: VllmConfig) -> None:
        parallel_config = vllm_config.parallel_config
        model_config = vllm_config.model_config

        if parallel_config.worker_cls == "auto":
            parallel_config.worker_cls = "vllm.v1.worker.gpu_worker.Worker"

        scheduler_config = vllm_config.scheduler_config
        # Note: model_config may be None during testing
        if (
            model_config is not None
            and model_config.is_mm_prefix_lm
            and scheduler_config.is_multimodal_model
            and not scheduler_config.disable_chunked_mm_input
        ):
            logger.warning(
                "Forcing --disable_chunked_mm_input for models "
                "with multimodal-bidirectional attention."
            )
            scheduler_config.disable_chunked_mm_input = True
```
**EN:** `check_and_update_config` checks whether a runtime condition is satisfied. It mainly works with `vllm_config`. Inside the body, it relies on `logger.warning` to complete the main steps.
**CN:** `check_and_update_config` 负责检查运行时条件是否满足。 它主要处理 `vllm_config` 等参数。 实现过程中会调用 `logger.warning` 等函数完成关键步骤。

### NvmlCudaPlatform overview (lines 594-812)
```python
class NvmlCudaPlatform(CudaPlatformBase):
    @classmethod
    @cache
    @with_nvml_context
    def get_device_capability(cls, device_id: int = 0) -> DeviceCapability | None:
        try:
            physical_device_id = cls.device_id_to_physical_device_id(device_id)
            handle = pynvml.nvmlDeviceGetHandleByIndex(physical_device_id)
            major, minor = pynvml.nvmlDeviceGetCudaComputeCapability(handle)
            return DeviceCapability(major=major, minor=minor)
        except RuntimeError:
            return None

    @classmethod
    @with_nvml_context
    def has_device_capability(
        cls,
        capability: tuple[int, int] | int,
        device_id: int = 0,
    ) -> bool:
        try:
            return super().has_device_capability(capability, device_id)
        except RuntimeError:
            return False

    @classmethod
    @with_nvml_context
    # ...
```
**EN:** Defines the `NvmlCudaPlatform` class used by this module. It extends `CudaPlatformBase`. Key methods include `get_device_capability`, `has_device_capability`, `get_device_name`, `get_device_uuid`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `NvmlCudaPlatform` 是该文件中的核心类，用于封装与 `NvmlCudaPlatform` 相关的状态和行为。 它继承自 `CudaPlatformBase`。 关键方法包括 `get_device_capability`, `has_device_capability`, `get_device_name`, `get_device_uuid`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### NvmlCudaPlatform.get_device_numa_node (lines 672-707)
```python
    def get_device_numa_node(cls, device_id: int = 0) -> int | None:
        """Get the NUMA node ID for a GPU device."""
        physical_device_id = cls.device_id_to_physical_device_id(device_id)
        handle = pynvml.nvmlDeviceGetHandleByIndex(physical_device_id)

        try:
            numa_node = pynvml.nvmlDeviceGetNumaNodeId(handle)
            if cls._numa_node_has_cpus(numa_node):
                return numa_node
            # On non-CDMM Grace-Blackwell systems (e.g. GB200), each GPU's HBM
            # is a separate NUMA node with no CPUs.  Fall through to
            # CPU-affinity-based detection to find the nearest CPU node.
            logger.debug(
                "NUMA node %d for GPU %d has no CPUs (non-CDMM topology), "
                "falling back to CPU-affinity-based detection",
                numa_node,
                device_id,
            )
        except Exception:
            pass
    # ...
                    )
                    return numa_node
        except Exception as e:
            logger.warning("Failed to get NUMA node for GPU %d: %s", device_id, e)

        return None
```
**EN:** `get_device_numa_node`: Get the NUMA node ID for a GPU device. It mainly works with `device_id`. Inside the body, it relies on `cls.device_id_to_physical_device_id`, `pynvml.nvmlDeviceGetHandleByIndex`, `pynvml.nvmlDeviceGetNumaNodeId` to complete the main steps.
**CN:** `get_device_numa_node` 负责获取流水线所需的数据或状态。 它主要处理 `device_id` 等参数。 实现过程中会调用 `cls.device_id_to_physical_device_id`, `pynvml.nvmlDeviceGetHandleByIndex`, `pynvml.nvmlDeviceGetNumaNodeId` 等函数完成关键步骤。

### NvmlCudaPlatform.get_all_device_numa_nodes (lines 778-795)
```python
    def get_all_device_numa_nodes(cls) -> list[int] | None:
        """Get NUMA nodes for all visible GPU devices."""
        try:
            numa_nodes = []
            for device_id in range(cls.device_count()):
                numa_node = cls.get_device_numa_node(device_id)
                if numa_node is None:
                    logger.warning(
                        "Could not detect NUMA node for GPU %d, "
                        "disabling automatic NUMA binding",
                        device_id,
                    )
                    return None
                numa_nodes.append(numa_node)
            return numa_nodes
        except Exception as e:
            logger.warning("Failed to get NUMA nodes for GPUs: %s", e)
            return None
```
**EN:** `get_all_device_numa_nodes`: Get NUMA nodes for all visible GPU devices. Inside the body, it relies on `cls.device_count`, `cls.get_device_numa_node`, `numa_nodes.append` to complete the main steps.
**CN:** `get_all_device_numa_nodes` 负责获取流水线所需的数据或状态。 实现过程中会调用 `cls.device_count`, `cls.get_device_numa_node`, `numa_nodes.append` 等函数完成关键步骤。

### Additional repeated patterns / 其他重复模式
```python
# Large file omitted for brevity
# ...
```
**EN:** The remaining code mostly expands the same abstractions with more wrappers, constants, or schema variants. The analysis above focuses on the control points that shape overall behavior.
**CN:** 剩余代码主要在相同抽象之上继续展开，补充更多包装函数、常量或模式变体。上面的分析聚焦于决定整体行为的关键控制点。

## Key Concepts / 关键概念
- **`CudaPlatformBase`**: Core class that organizes module behavior. / **`CudaPlatformBase`**：组织模块行为的核心类。
- **`NvmlCudaPlatform`**: Core class that organizes module behavior. / **`NvmlCudaPlatform`**：组织模块行为的核心类。
- **`NonNvmlCudaPlatform`**: Core class that organizes module behavior. / **`NonNvmlCudaPlatform`**：组织模块行为的核心类。
- **`_cuda_device_count_stateless`**: Key helper or entry point in this file. / **`_cuda_device_count_stateless`**：本文件中的关键辅助函数或入口。
- **`_get_backend_priorities`**: Key helper or entry point in this file. / **`_get_backend_priorities`**：本文件中的关键辅助函数或入口。
- **`with_nvml_context`**: Key helper or entry point in this file. / **`with_nvml_context`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: __future__, os, collections.abc, datetime, functools, typing, pathlib
- **Third-party / 第三方**: torch, torch.distributed, torch.distributed.distributed_c10d, typing_extensions, torch.cuda
- **Internal vLLM / vLLM 内部依赖**: vllm._C, vllm._C_stable_libtorch, vllm.envs, vllm.logger, vllm.utils.import_utils, vllm.utils.torch_utils, vllm.v1.attention.backends.registry, .interface, vllm.config, vllm.config.cache, vllm.config.kernel, vllm.v1.attention.selector, vllm.config.compilation
