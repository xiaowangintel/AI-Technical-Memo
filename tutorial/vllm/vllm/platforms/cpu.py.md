# cpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/platforms/cpu.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines platform-specific behavior and capability checks for hardware backends. / 定义硬件后端的平台特定行为与能力检查逻辑。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-29)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import glob
import os
import platform
import subprocess
import sys
from typing import TYPE_CHECKING

import torch

from vllm.logger import init_logger
from vllm.utils.cpu_resource_utils import (
    DEVICE_CONTROL_ENV_VAR,
    get_memory_node_info,
)
from vllm.utils.mem_constants import GiB_bytes
from vllm.v1.attention.backends.registry import AttentionBackendEnum

from .interface import CpuArchEnum, Platform, PlatformEnum

logger = init_logger(__name__)

if TYPE_CHECKING:
    from vllm.config import VllmConfig
    from vllm.v1.attention.selector import AttentionSelectorConfig
else:
    VllmConfig = None
```
**EN:** Sets up the module with standard-library support such as `glob`, `os`, `platform`, external packages such as `torch`, vLLM modules such as `vllm.logger`, `vllm.utils.cpu_resource_utils`, `vllm.utils.mem_constants`. It prepares the symbols later used by `CpuPlatform`, `get_max_threads`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.logger`, `vllm.utils.cpu_resource_utils`, `vllm.utils.mem_constants` 等 vLLM 内部依赖。 这些准备工作为后续的 `CpuPlatform`, `get_max_threads` 提供上下文。

### get_max_threads (lines 32-38)
```python
def get_max_threads(pid=0):
    if hasattr(os, "sched_getaffinity"):
        return len(os.sched_getaffinity(pid))
    elif platform.system() == "Darwin":
        return os.cpu_count()
    else:
        raise NotImplementedError("Unsupported OS")
```
**EN:** `get_max_threads` retrieves data or state needed by the pipeline. It mainly works with `pid`. Inside the body, it relies on `os.sched_getaffinity`, `platform.system`, `os.cpu_count` to complete the main steps.
**CN:** `get_max_threads` 负责获取流水线所需的数据或状态。 它主要处理 `pid` 等参数。 实现过程中会调用 `os.sched_getaffinity`, `platform.system`, `os.cpu_count` 等函数完成关键步骤。

### CpuPlatform overview (lines 41-483)
```python
class CpuPlatform(Platform):
    _enum = PlatformEnum.CPU
    device_name: str = "cpu"
    device_type: str = "cpu"
    dispatch_key: str = "CPU"
    dist_backend: str = "gloo"
    device_control_env_var = DEVICE_CONTROL_ENV_VAR

    @property
    def supported_dtypes(self) -> list[torch.dtype]:
        if self.get_cpu_architecture() == CpuArchEnum.POWERPC:
            return [torch.bfloat16, torch.float32]
        elif self.get_cpu_architecture() == CpuArchEnum.ARM and sys.platform.startswith(
            "darwin"
        ):
            if (
                subprocess.check_output(
                    ["sysctl -n hw.optional.arm.FEAT_BF16"], shell=True
                ).strip()
                == b"1"
            ):
                return [torch.bfloat16, torch.float16, torch.float32]
            return [torch.float16, torch.float32]
        elif self.get_cpu_architecture() == CpuArchEnum.RISCV:
            return [torch.bfloat16, torch.float16, torch.float32]
        # x86/aarch64 CPU has supported both bf16 and fp16 natively.
        return [torch.bfloat16, torch.float16, torch.float32]
    # ...
```
**EN:** Defines the `CpuPlatform` class used by this module. It extends `Platform`. Key methods include `supported_dtypes`, `get_device_name`, `get_attn_backend_cls`, `get_device_total_memory`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `CpuPlatform` 是该文件中的核心类，用于封装与 `CpuPlatform` 相关的状态和行为。 它继承自 `Platform`。 关键方法包括 `supported_dtypes`, `get_device_name`, `get_attn_backend_cls`, `get_device_total_memory`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### CpuPlatform.check_and_update_config (lines 110-310)
```python
    def check_and_update_config(cls, vllm_config: VllmConfig) -> None:
        model_config = vllm_config.model_config

        if model_config is not None:
            model_config.disable_cascade_attn = True

        cache_config = vllm_config.cache_config

        if not cache_config.user_specified_block_size:
            cache_config.block_size = 128

        if cache_config.block_size % 32 != 0:
            logger.warning(
                "CPU backend prefers block_size is multiples of 32, "
                "otherwise the performance is not optimized."
            )

        # Lagecy setting
        env_key = "VLLM_CPU_KVCACHE_SPACE"
        if env_key in os.environ and os.environ[env_key] != "":
    # ...
            )
            vllm_config.scheduler_config.enable_chunked_prefill = False
            vllm_config.scheduler_config.max_num_batched_tokens = max(
                vllm_config.model_config.max_model_len,
                vllm_config.scheduler_config.DEFAULT_MAX_NUM_BATCHED_TOKENS,
            )
```
**EN:** `check_and_update_config` checks whether a runtime condition is satisfied. It mainly works with `vllm_config`. Inside the body, it relies on `torch.cpu._is_amx_tile_supported`, `os.getenv`, `Platform.get_cpu_architecture` to complete the main steps.
**CN:** `check_and_update_config` 负责检查运行时条件是否满足。 它主要处理 `vllm_config` 等参数。 实现过程中会调用 `torch.cpu._is_amx_tile_supported`, `os.getenv`, `Platform.get_cpu_architecture` 等函数完成关键步骤。

### CpuPlatform.discover_numa_topology (lines 326-377)
```python
    def discover_numa_topology(cls) -> list[list[int]]:
        """
        Discover NUMA topology and keep the last physical core of each numa
        into one core group list for nixl start_kv_load()
        """
        SYS_NODE = "/sys/devices/system/node"
        SYS_CPU = "/sys/devices/system/cpu"

        if not (os.path.exists(SYS_NODE) and os.path.exists(SYS_CPU)):
            return []

        core_rsv_for_kv = []
        for node in os.listdir(SYS_NODE):
            if not node.startswith("node") or not node[4:].isdigit():
                continue
            node_path = f"{SYS_NODE}/{node}"

            seen_phys = set()
            for cpu in os.listdir(node_path):
                if not cpu.startswith("cpu") or not cpu[3:].isdigit():
    # ...
                    seen_phys.add(phys)

            if len(seen_phys) > 0:
                core_rsv_for_kv.append(list(seen_phys))

        return core_rsv_for_kv
```
**EN:** `discover_numa_topology`: Discover NUMA topology and keep the last physical core of each numa into one core group list for nixl start_kv_load(). Inside the body, it relies on `os.listdir`, `os.path.exists`, `min` to complete the main steps.
**CN:** `discover_numa_topology` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `os.listdir`, `os.path.exists`, `min` 等函数完成关键步骤。

### CpuPlatform.pack_kv_cache (lines 442-483)
```python
    def pack_kv_cache(
        cls,
        key: torch.Tensor,
        value: torch.Tensor,
        key_cache: torch.Tensor,
        value_cache: torch.Tensor,
        block_ids: list[int],
        indices: torch.Tensor,
    ) -> None:
        """
        Rewrite the kv cache shape for the current platform.
        """
        # Import lazily: cpu_attn pulls in _custom_ops, which needs a fully
        # initialized vllm.platforms (avoid circular import while CpuPlatform loads).
        from vllm._custom_ops import cpu_attn_reshape_and_cache
        from vllm.v1.attention.backends.cpu_attn import _get_attn_isa

        dtype = key.dtype
        # For CPU_ATTN, the shape is [N, num_kv_heads, block_size, head_size]
        _, _, block_size, head_size = key_cache.shape
    # ...
            value,
            key_cache,
            value_cache,
            slot_mapping,
            isa,
        )
```
**EN:** `pack_kv_cache`: Rewrite the kv cache shape for the current platform. It mainly works with `key`, `value`, `key_cache`, `value_cache`. Inside the body, it relies on `key.permute.flatten`, `value.permute.flatten`, `_get_attn_isa` to complete the main steps.
**CN:** `pack_kv_cache` 负责实现本模块使用的辅助逻辑。 它主要处理 `key`, `value`, `key_cache`, `value_cache` 等参数。 实现过程中会调用 `key.permute.flatten`, `value.permute.flatten`, `_get_attn_isa` 等函数完成关键步骤。

### CpuPlatform.get_device_communicator_cls (lines 388-392)
```python
    def get_device_communicator_cls(cls) -> str:
        """
        Get device specific communicator class for distributed communication.
        """
        return "vllm.distributed.device_communicators.cpu_communicator.CpuCommunicator"  # noqa
```
**EN:** `get_device_communicator_cls`: Get device specific communicator class for distributed communication.
**CN:** `get_device_communicator_cls` 负责获取流水线所需的数据或状态。

## Key Concepts / 关键概念
- **`CpuPlatform`**: Core class that organizes module behavior. / **`CpuPlatform`**：组织模块行为的核心类。
- **`get_max_threads`**: Key helper or entry point in this file. / **`get_max_threads`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: glob, os, platform, subprocess, sys, typing
- **Third-party / 第三方**: torch
- **Internal vLLM / vLLM 内部依赖**: vllm.logger, vllm.utils.cpu_resource_utils, vllm.utils.mem_constants, vllm.v1.attention.backends.registry, .interface, vllm.config, vllm.v1.attention.selector, vllm._custom_ops, vllm.v1.attention.backends.cpu_attn, vllm._C, vllm._C_AVX2, vllm._C_AVX512
