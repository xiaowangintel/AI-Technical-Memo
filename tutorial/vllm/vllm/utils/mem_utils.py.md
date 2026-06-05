# mem_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/utils/mem_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides reusable utility helpers shared across vLLM components. / 提供 vLLM 各组件复用的通用工具函数。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-16)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import contextlib
import gc
import time
from collections.abc import Generator
from dataclasses import dataclass, field
from functools import cache

import psutil
import torch
import torch.types

from vllm.platforms import current_platform

from .mem_constants import GiB_bytes, KiB_bytes, MiB_bytes
```
**EN:** Sets up the module with standard-library support such as `contextlib`, `gc`, `time`, external packages such as `psutil`, `torch`, `torch.types`, vLLM modules such as `vllm.platforms`, `.mem_constants`, `vllm`. It prepares the symbols later used by `DeviceMemoryProfiler`, `MemorySnapshot`, `format_kib`, `format_mib`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.platforms`, `.mem_constants`, `vllm` 等 vLLM 内部依赖。 这些准备工作为后续的 `DeviceMemoryProfiler`, `MemorySnapshot`, `format_kib`, `format_mib` 提供上下文。

### format_kib (lines 19-20)
```python
def format_kib(b: int) -> str:
    return f"{round(b / KiB_bytes, 2)}"
```
**EN:** `format_kib` formats information for logging or display. It mainly works with `b`. Inside the body, it relies on `round` to complete the main steps.
**CN:** `format_kib` 负责格式化信息用于日志或显示。 它主要处理 `b` 等参数。 实现过程中会调用 `round` 等函数完成关键步骤。

### format_mib (lines 23-24)
```python
def format_mib(b: int) -> str:
    return f"{round(b / MiB_bytes, 2)}"
```
**EN:** `format_mib` formats information for logging or display. It mainly works with `b`. Inside the body, it relies on `round` to complete the main steps.
**CN:** `format_mib` 负责格式化信息用于日志或显示。 它主要处理 `b` 等参数。 实现过程中会调用 `round` 等函数完成关键步骤。

### format_gib (lines 27-28)
```python
def format_gib(b: int) -> str:
    return f"{round(b / GiB_bytes, 2)}"
```
**EN:** `format_gib` formats information for logging or display. It mainly works with `b`. Inside the body, it relies on `round` to complete the main steps.
**CN:** `format_gib` 负责格式化信息用于日志或显示。 它主要处理 `b` 等参数。 实现过程中会调用 `round` 等函数完成关键步骤。

### get_max_shared_memory_bytes (lines 32-40)
```python
def get_max_shared_memory_bytes(gpu: int = 0) -> int:
    """Returns the maximum shared memory per thread block in bytes."""
    from vllm import _custom_ops as ops

    max_shared_mem = ops.get_max_shared_memory_per_block_device_attribute(gpu)
    # value 0 will cause MAX_SEQ_LEN become negative and test_attention.py
    # will fail
    assert max_shared_mem > 0, "max_shared_mem cannot be zero"
    return int(max_shared_mem)
```
**EN:** `get_max_shared_memory_bytes`: Returns the maximum shared memory per thread block in bytes. It mainly works with `gpu`. Inside the body, it relies on `ops.get_max_shared_memory_per_block_device_attribute` to complete the main steps.
**CN:** `get_max_shared_memory_bytes` 负责获取流水线所需的数据或状态。 它主要处理 `gpu` 等参数。 实现过程中会调用 `ops.get_max_shared_memory_per_block_device_attribute` 等函数完成关键步骤。

### get_cpu_memory (lines 43-45)
```python
def get_cpu_memory() -> int:
    """Returns the total CPU memory of the node in bytes."""
    return psutil.virtual_memory().total
```
**EN:** `get_cpu_memory`: Returns the total CPU memory of the node in bytes. Inside the body, it relies on `psutil.virtual_memory` to complete the main steps.
**CN:** `get_cpu_memory` 负责获取流水线所需的数据或状态。 实现过程中会调用 `psutil.virtual_memory` 等函数完成关键步骤。

### DeviceMemoryProfiler (lines 48-67)
```python
class DeviceMemoryProfiler:
    def __init__(self, device: torch.types.Device | None = None):
        self.device = device

    def current_memory_usage(self) -> float:
        # Return the memory usage in bytes.
        gc.collect()
        return current_platform.get_current_memory_usage(self.device)

    def __enter__(self):
        self.initial_memory = self.current_memory_usage()
        # This allows us to call methods of the context manager if needed
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.final_memory = self.current_memory_usage()
        self.consumed_memory = self.final_memory - self.initial_memory

        # Force garbage collection
        gc.collect()
```
**EN:** Defines the `DeviceMemoryProfiler` class used by this module. Key methods include `__init__`, `current_memory_usage`.
**CN:** `DeviceMemoryProfiler` 是该文件中的核心类，用于封装与 `DeviceMemoryProfiler` 相关的状态和行为。 关键方法包括 `__init__`, `current_memory_usage`。

### MemorySnapshot overview (lines 71-157)
```python
class MemorySnapshot:
    """Memory snapshot."""

    torch_peak: int = 0
    free_memory: int = 0
    total_memory: int = 0
    cuda_memory: int = 0
    torch_memory: int = 0
    non_torch_memory: int = 0
    timestamp: float = 0.0

    device: torch.types.Device = None
    auto_measure: bool = True

    def __post_init__(self) -> None:
        if self.device is None:
            device_fn = current_platform.current_device
            assert device_fn is not None
            self.device_ = torch.device(device_fn())
        else:
            self.device_ = torch.device(self.device)

        if self.auto_measure:
            self.measure()

    def measure(self) -> None:
        device = self.device_
    # ...
```
**EN:** `MemorySnapshot`: Memory snapshot. Key methods include `measure`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `MemorySnapshot` 是该文件中的核心类，用于封装与 `MemorySnapshot` 相关的状态和行为。 关键方法包括 `measure`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### MemorySnapshot.measure (lines 96-126)
```python
    def measure(self) -> None:
        device = self.device_

        # we measure the torch peak memory usage via allocated_bytes,
        # rather than `torch.accelerator.memory_reserved()` .
        # After `torch.accelerator.reset_peak_memory_stats()`,
        # `torch.accelerator.memory_reserved()` will keep growing, and only shrink
        # when we call `torch.accelerator.empty_cache()` or OOM happens.
        self.torch_peak = torch.accelerator.memory_stats(device).get(
            "allocated_bytes.all.peak", 0
        )

        self.free_memory, self.total_memory = current_platform.mem_get_info(device)
        if current_platform.is_integrated_gpu(device.index):
            # On UMA (Unified Memory Architecture) platforms where CPU and
            # GPU share physical memory (e.g. GH200, DGX Spark, Jetson Orin),
            # cudaMemGetInfo underreports free memory because it does not
            # account for reclaimable OS memory (page cache, buffers).
            # Use psutil to get the true available memory.
            # https://docs.nvidia.com/cuda/cuda-for-tegra-appnote/#estimating-total-allocatable-device-memory-on-an-integrated-gpu-device
    # ...
        # PyTorch gets from cuda (by calling cudaMalloc, etc.)
        # this is used to measure the non-torch memory usage
        self.torch_memory = torch.accelerator.memory_reserved(device)

        self.non_torch_memory = self.cuda_memory - self.torch_memory
        self.timestamp = time.time()
```
**EN:** `measure` implements helper logic used by this module. Inside the body, it relies on `torch.accelerator.memory_stats.get`, `current_platform.mem_get_info`, `current_platform.is_integrated_gpu` to complete the main steps.
**CN:** `measure` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `torch.accelerator.memory_stats.get`, `current_platform.mem_get_info`, `current_platform.is_integrated_gpu` 等函数完成关键步骤。

### MemorySnapshot.__sub__ (lines 128-145)
```python
    def __sub__(self, other: "MemorySnapshot") -> "MemorySnapshot":
        if self.device_ != other.device_:
            raise ValueError(
                "The two snapshots should be from the same device! "
                f"Found: {self.device_} vs. {other.device_}"
            )

        return MemorySnapshot(
            torch_peak=self.torch_peak - other.torch_peak,
            free_memory=self.free_memory - other.free_memory,
            total_memory=self.total_memory - other.total_memory,
            cuda_memory=self.cuda_memory - other.cuda_memory,
            torch_memory=self.torch_memory - other.torch_memory,
            non_torch_memory=self.non_torch_memory - other.non_torch_memory,
            timestamp=self.timestamp - other.timestamp,
            device=self.device_,
            auto_measure=False,
        )
```
**EN:** `__sub__` implements helper logic used by this module. It mainly works with `other`. Inside the body, it relies on `MemorySnapshot`, `ValueError` to complete the main steps.
**CN:** `__sub__` 负责实现本模块使用的辅助逻辑。 它主要处理 `other` 等参数。 实现过程中会调用 `MemorySnapshot`, `ValueError` 等函数完成关键步骤。

### MemorySnapshot.__repr__ (lines 147-157)
```python
    def __repr__(self) -> str:
        return (
            f"torch_peak={format_gib(self.torch_peak)}GiB, "
            f"free_memory={format_gib(self.free_memory)}GiB, "
            f"total_memory={format_gib(self.total_memory)}GiB, "
            f"{current_platform.device_name}_memory={format_gib(self.cuda_memory)}GiB, "
            f"torch_memory={format_gib(self.torch_memory)}GiB, "
            f"non_torch_memory={format_gib(self.non_torch_memory)}GiB, "
            f"timestamp={self.timestamp}, "
            f"auto_measure={self.auto_measure}"
        )
```
**EN:** `__repr__` implements helper logic used by this module. Inside the body, it relies on `format_gib` to complete the main steps.
**CN:** `__repr__` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `format_gib` 等函数完成关键步骤。

### MemorySnapshot.__post_init__ (lines 85-94)
```python
    def __post_init__(self) -> None:
        if self.device is None:
            device_fn = current_platform.current_device
            assert device_fn is not None
            self.device_ = torch.device(device_fn())
        else:
            self.device_ = torch.device(self.device)

        if self.auto_measure:
            self.measure()
```
**EN:** `__post_init__` implements helper logic used by this module. Inside the body, it relies on `torch.device`, `self.measure`, `device_fn` to complete the main steps.
**CN:** `__post_init__` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `torch.device`, `self.measure`, `device_fn` 等函数完成关键步骤。

### MemoryProfilingResult (lines 161-187)
```python
class MemoryProfilingResult:
    """Memory profiling result. All numbers are in bytes."""

    non_kv_cache_memory: int = 0
    torch_peak_increase: int = 0
    non_torch_increase: int = 0
    weights_memory: int = 0
    before_create: MemorySnapshot = field(default_factory=MemorySnapshot)
    profile_time: float = 0.0

    def __post_init__(self) -> None:
        device = self.before_create.device_

        self.before_profile = MemorySnapshot(device=device, auto_measure=False)
        self.after_profile = MemorySnapshot(device=device, auto_measure=False)

    def __repr__(self) -> str:
        return (
            f"Memory profiling takes {self.profile_time:.2f} seconds. "
            f"Total non KV cache memory: "
            f"{format_gib(self.non_kv_cache_memory)}GiB; "
            f"torch peak memory increase: "
            f"{format_gib(self.torch_peak_increase)}GiB; "
            f"non-torch forward increase memory: "
            f"{format_gib(self.non_torch_increase)}GiB; "
            f"weights memory: {format_gib(self.weights_memory)}GiB."
        )
```
**EN:** `MemoryProfilingResult`: Memory profiling result.
**CN:** `MemoryProfilingResult` 是该文件中的核心类，用于封装与 `MemoryProfilingResult` 相关的状态和行为。

### memory_profiling (lines 191-275)
```python
def memory_profiling(
    baseline_snapshot: MemorySnapshot,
    weights_memory: int = 0,
) -> Generator[MemoryProfilingResult, None, None]:
    """
    Memory profiling context manager.

    baseline_snapshot: the memory snapshot before the current vLLM instance.
    weights_memory: memory used by PyTorch when loading the model weights.
        Note that, before loading the model weights, we also initialize the device
        and distributed environment, which may consume some memory. This part is not
        included in the weights_memory because PyTorch does not control it.

    The memory in one GPU can be classified into 3 categories:
    1. memory used by anything other than the current vLLM instance.
    2. memory used by torch in the current vLLM instance.
    3. memory used in the current vLLM instance, but not by torch.

    A quantitive example:

    Before creating the current vLLM instance:
        category 1: 1 GiB
        category 2: 0 GiB
        category 3: 0 GiB
    # ...

    non_torch_memory = result.non_torch_increase
    peak_activation_memory = result.torch_peak_increase
    result.non_kv_cache_memory = (
        non_torch_memory + peak_activation_memory + result.weights_memory
    )
```
**EN:** `memory_profiling`: Memory profiling context manager. It mainly works with `baseline_snapshot`, `weights_memory`. Inside the body, it relies on `gc.collect`, `torch.accelerator.empty_cache`, `torch.accelerator.reset_peak_memory_stats` to complete the main steps.
**CN:** `memory_profiling` 负责实现本模块使用的辅助逻辑。 它主要处理 `baseline_snapshot`, `weights_memory` 等参数。 实现过程中会调用 `gc.collect`, `torch.accelerator.empty_cache`, `torch.accelerator.reset_peak_memory_stats` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`DeviceMemoryProfiler`**: Core class that organizes module behavior. / **`DeviceMemoryProfiler`**：组织模块行为的核心类。
- **`MemorySnapshot`**: Core class that organizes module behavior. / **`MemorySnapshot`**：组织模块行为的核心类。
- **`MemoryProfilingResult`**: Core class that organizes module behavior. / **`MemoryProfilingResult`**：组织模块行为的核心类。
- **`format_kib`**: Key helper or entry point in this file. / **`format_kib`**：本文件中的关键辅助函数或入口。
- **`format_mib`**: Key helper or entry point in this file. / **`format_mib`**：本文件中的关键辅助函数或入口。
- **`format_gib`**: Key helper or entry point in this file. / **`format_gib`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: contextlib, gc, time, collections.abc, dataclasses, functools
- **Third-party / 第三方**: psutil, torch, torch.types
- **Internal vLLM / vLLM 内部依赖**: vllm.platforms, .mem_constants, vllm
