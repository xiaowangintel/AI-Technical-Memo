# all_reduce_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/device_communicators/all_reduce_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements distributed device-communication helpers or backend adapters. / 实现分布式设备通信辅助逻辑或后端适配器。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化
```python
import ctypes
import json
import os
import pickle
import subprocess
import sys
import tempfile
from collections.abc import Sequence
from itertools import product
from typing import Any

import torch
import torch.distributed as dist
import torch.multiprocessing as mp

import vllm.envs as envs
from vllm.distributed.device_communicators.cuda_wrapper import CudaRTLibrary
from vllm.logger import init_logger
from vllm.platforms import current_platform
from vllm.utils.system_utils import update_environment_variables
```
**EN:** This block imports `ctypes`, `json`, `os`, `pickle`, `subprocess`, `sys` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `ctypes`, `json`, `os`, `pickle`, `subprocess`, `sys`，为后续实现准备运行时、类型与辅助 API。

### Module constants / 模块常量
```python
logger = init_logger(__name__)

KiB = 1024
MiB = 1024 * 1024
# Max size for each world size in case symmetric memory is available
# For different SM architectures
CUSTOM_ALL_REDUCE_MAX_SIZES = {
    "9.0": {
        2: 64 * MiB,  # 64 MB
        4: 32 * MiB,  # 32 MB
        6: MiB // 2,  # 512 KB
        8: MiB // 4,  # 256 KB
    },
    "10.0": {
        2: 2 * MiB,  # 2 MB
        4: 2 * MiB,  # 2 MB
        6: 1 * MiB,  # 1 MB
        8: 1 * MiB,  # 1 MB
    },
    "10.3": {
        2: 4 * MiB,  # 4 MB
        4: 4 * MiB,  # 4 MB
        6: 8 * MiB,  # 8 MB
        8: 4 * MiB,  # 4 MB
    },
}

SYMM_MEM_ALL_REDUCE_MAX_SIZES = {
    "9.0": {
        2: 64 * MiB,  # 64 MB
        4: 32 * MiB,  # 32 MB
        6: 64 * MiB,  # 64 MB
        8: 64 * MiB,  # 64 MB
    },
    "10.0": {
        2: 8 * MiB,  # 8 MB
        4: 32 * MiB,  # 32 MB
        6: 128 * MiB,  # 128 MB
        8: 128 * MiB,  # 128 MB
    },
    "10.3": {
        2: 4 * MiB,  # 4 MB
        4: 32 * MiB,  # 32 MB
        6: 32 * MiB,  # 32 MB
        8: 64 * MiB,  # 64 MB
# ... truncated for analysis ...
NCCL_SYMM_MEM_ALL_REDUCE_CONFIG: dict[str, Any] = {
    "min_world_size": 4,
    # Ranges where custom_AR outperforms NCCL symm_mem: (lower_bound, upper_bound)
    # NCCL symm_mem will NOT be used for sizes in range: lower < size < upper
    "custom_ar_preferred_ranges": {
        4: (16 * KiB, 512 * KiB),  # custom_AR wins for 32K-256K
        8: (16 * KiB, 128 * KiB),  # custom_AR wins for 32K-64K
    },
    "always_use_above_world_size": 8,  # Always use symm mem for world_size > 8
}
```
**EN:** This section defines module-level aliases, constants, or shared state such as `logger`, `KiB`, `MiB`, `CUSTOM_ALL_REDUCE_MAX_SIZES`, `SYMM_MEM_ALL_REDUCE_MAX_SIZES`, `NCCL_SYMM_MEM_ALL_REDUCE_CONFIG`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `logger`, `KiB`, `MiB`, `CUSTOM_ALL_REDUCE_MAX_SIZES`, `SYMM_MEM_ALL_REDUCE_MAX_SIZES`, `NCCL_SYMM_MEM_ALL_REDUCE_CONFIG`，供后续代码复用。

### Function `should_nccl_symm_mem_allreduce` / 函数 `should_nccl_symm_mem_allreduce`
```python
def should_nccl_symm_mem_allreduce(world_size: int, input_tensor: torch.Tensor) -> bool:
    """
    Determine if NCCL symmetric memory allreduce should be used.

    Based on H100 and GB200 benchmarks, NCCL symm_mem is preferred for:
    - Small tensors (≤16K): Lower latency than custom_AR
    - Large tensors (≥128K for 8 GPUs, ≥512K for 4 GPUs): Better bandwidth

    Custom_AR is preferred for mid-range sizes where its P2P approach
    has lower overhead than the symm_mem copy-in/copy-out pattern.
    """
    from vllm.distributed.device_communicators.pynccl_allocator import (
        is_symmetric_memory_enabled,
    )

    if envs.VLLM_BATCH_INVARIANT:
        return False

    if not is_symmetric_memory_enabled():
        return False

    if world_size < NCCL_SYMM_MEM_ALL_REDUCE_CONFIG["min_world_size"]:
        return False

    tensor_size = input_tensor.nbytes
    custom_ar_range = NCCL_SYMM_MEM_ALL_REDUCE_CONFIG["custom_ar_preferred_ranges"].get(
        world_size
    )

    if custom_ar_range is not None:
        lower_bound, upper_bound = custom_ar_range
        # Use symm_mem for small sizes (≤ lower_bound) and large sizes (≥ upper_bound)
        # Use custom_AR (not symm_mem) for mid-range sizes
        return tensor_size <= lower_bound or tensor_size >= upper_bound
    return world_size > NCCL_SYMM_MEM_ALL_REDUCE_CONFIG["always_use_above_world_size"]
```
**EN:** `should_nccl_symm_mem_allreduce` implements a focused helper routine for this module. The docstring frames it as: Determine if NCCL symmetric memory allreduce should be used. It primarily works with arguments like `world_size`, `input_tensor`. Key calls include `NCCL_SYMM_MEM_ALL_REDUCE_CONFIG.get`, `is_symmetric_memory_enabled`.
**CN:** `should_nccl_symm_mem_allreduce` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `world_size`, `input_tensor` 这样的参数。 关键调用包括 `NCCL_SYMM_MEM_ALL_REDUCE_CONFIG.get`, `is_symmetric_memory_enabled`。

### Function `producer` / 函数 `producer`
```python
def producer(
    batch_src: Sequence[int],
    producer_queue,
    consumer_queue,
    result_queue,
    cuda_visible_devices: str | None = None,
):
    if cuda_visible_devices is not None:
        update_environment_variables({"CUDA_VISIBLE_DEVICES": cuda_visible_devices})

    lib = CudaRTLibrary()
    for i in batch_src:
        lib.cudaSetDevice(i)
        pointer = lib.cudaMalloc(1024)
        lib.cudaMemset(pointer, 1, 1024)
        lib.cudaDeviceSynchronize()
        handle = lib.cudaIpcGetMemHandle(pointer)
        producer_queue.put(handle)
        open_success = consumer_queue.get()
        if open_success:
            # use two queues to simulate barrier
            producer_queue.put(0)
            consumer_queue.get()
            # check if the memory is modified
            host_data = (ctypes.c_char * 1024)()
            lib.cudaMemcpy(host_data, pointer, 1024)  # type: ignore
            for i in range(1024):
                if ord(host_data[i]) != 2:
                    open_success = False
                    break
        result_queue.put(open_success)
        lib.cudaDeviceReset()
```
**EN:** `producer` implements a focused helper routine for this module. It primarily works with arguments like `batch_src`, `producer_queue`, `consumer_queue`, `result_queue`. Key calls include `CudaRTLibrary`, `update_environment_variables`, `lib.cudaSetDevice`.
**CN:** `producer` 实现了一个面向当前模块的辅助例程。 它主要处理诸如 `batch_src`, `producer_queue`, `consumer_queue`, `result_queue` 这样的参数。 关键调用包括 `CudaRTLibrary`, `update_environment_variables`, `lib.cudaSetDevice`。

### Function `consumer` / 函数 `consumer`
```python
def consumer(
    batch_tgt: Sequence[int],
    producer_queue,
    consumer_queue,
    result_queue,
    cuda_visible_devices: str | None = None,
):
    if cuda_visible_devices is not None:
        update_environment_variables({"CUDA_VISIBLE_DEVICES": cuda_visible_devices})

    lib = CudaRTLibrary()
    for j in batch_tgt:
        lib.cudaSetDevice(j)
        handle = producer_queue.get()
        open_success = False
        try:
            pointer = lib.cudaIpcOpenMemHandle(handle)  # type: ignore
            open_success = True
        except RuntimeError:
            # cannot error out here, because the producer process
            # is still waiting for the response.
            pass
        consumer_queue.put(open_success)
        if open_success:
            # modify the memory
            lib.cudaMemset(pointer, 2, 1024)
            lib.cudaDeviceSynchronize()
            # use two queues to simulate barrier
            producer_queue.get()
            consumer_queue.put(0)
            # check if the memory is modified
            host_data = (ctypes.c_char * 1024)()
            lib.cudaMemcpy(host_data, pointer, 1024)  # type: ignore
            for i in range(1024):
                if ord(host_data[i]) != 2:
                    open_success = False
                    break
        result_queue.put(open_success)
        lib.cudaDeviceReset()
```
**EN:** `consumer` implements a focused helper routine for this module. It primarily works with arguments like `batch_tgt`, `producer_queue`, `consumer_queue`, `result_queue`. Key calls include `CudaRTLibrary`, `update_environment_variables`, `lib.cudaSetDevice`.
**CN:** `consumer` 实现了一个面向当前模块的辅助例程。 它主要处理诸如 `batch_tgt`, `producer_queue`, `consumer_queue`, `result_queue` 这样的参数。 关键调用包括 `CudaRTLibrary`, `update_environment_variables`, `lib.cudaSetDevice`。

### Function `can_actually_p2p` / 函数 `can_actually_p2p`
```python
def can_actually_p2p(
    batch_src: Sequence[int],
    batch_tgt: Sequence[int],
) -> Sequence[bool]:
    """
    Usually, checking if P2P access is enabled can be done by
    `torch.cuda.can_device_access_peer(src, tgt)`. However, sometimes
    the driver might be broken, and `torch.cuda.can_device_access_peer(src, tgt)`
    returns `True` even if P2P access is not actually possible.
    See https://github.com/vllm-project/vllm/issues/2728 and
    https://forums.developer.nvidia.com/t/direct-gpu-gpu-communication-does-not-seem-to-work-properly/283264/10
    Therefore, we have to perform a real P2P access to check if it is actually
    possible.

    Note on p2p and cuda IPC:
    Usually, one process uses one GPU:
    GPU src --> cuda context src --> tensor src --> process src

    We need to combine p2p and cuda IPC, so that:
    GPU src --> cuda context src --> tensor src --> process src
                                      |shared|
    GPU tgt --> cuda context tgt --> tensor tgt --> process tgt
    That is to say, process src creates a tensor in GPU src, passes IPC handle to
    process tgt, and process tgt accesses the tensor in GPU tgt. Any operation on the
    tensor in process tgt will be reflected in the tensor in process src, because
    they are the same memory segment.
    It is important to note that process tgt accesses the tensor in GPU tgt, not
    GPU src. That's why we need p2p access.

    The most time-consuming part is the process creation. To avoid creating
    processes for every pair of GPUs, we use batched testing. We create two
    processes for testing all pairs of GPUs in batch. The trick is to reset
    the device after each test (which is not available in PyTorch).
    """  # noqa
    cuda_visible_devices = envs.CUDA_VISIBLE_DEVICES
    # pass the CUDA_VISIBLE_DEVICES to the child process
    # to make sure they see the same set of GPUs

    # make sure the processes are spawned
    smp = mp.get_context("spawn")
    producer_queue = smp.Queue()
    consumer_queue = smp.Queue()
    result_queue = smp.Queue()
    p_src = smp.Process(
        target=producer,
# ... truncated for analysis ...
            logger.warning(
                "Two processes do not agree on the P2P access"
                " status on %d -> %d, treat as disabled.",
                src,
                tgt,
            )
            result.append(False)
        else:
            result.append(a)
    return result
```
**EN:** `can_actually_p2p` implements a focused helper routine for this module. The docstring frames it as: Usually, checking if P2P access is enabled can be done by `torch.cuda.can_device_access_peer(src, tgt)`. However, sometimes the driver.... It primarily works with arguments like `batch_src`, `batch_tgt`. Key calls include `mp.get_context`, `smp.Queue`, `smp.Process`.
**CN:** `can_actually_p2p` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `batch_src`, `batch_tgt` 这样的参数。 关键调用包括 `mp.get_context`, `smp.Queue`, `smp.Process`。

### Module constants / 模块常量
```python
_gpu_p2p_access_cache: dict[str, bool] | None = None
```
**EN:** This section defines module-level aliases, constants, or shared state such as `_gpu_p2p_access_cache`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `_gpu_p2p_access_cache`，供后续代码复用。

### Function `gpu_p2p_access_check` / 函数 `gpu_p2p_access_check`
```python
def gpu_p2p_access_check(src: int, tgt: int) -> bool:
    """Check if GPU src can access GPU tgt."""

    # if the cache variable is already calculated,
    # read from the cache instead of checking it again
    global _gpu_p2p_access_cache
    if _gpu_p2p_access_cache is not None:
        return _gpu_p2p_access_cache[f"{src}->{tgt}"]

    is_distributed = dist.is_initialized()

    num_dev = current_platform.device_count()
    cuda_visible_devices = envs.CUDA_VISIBLE_DEVICES
    if cuda_visible_devices is None:
        cuda_visible_devices = ",".join(str(i) for i in range(num_dev))

    path = os.path.join(
        envs.VLLM_CACHE_ROOT, f"gpu_p2p_access_cache_for_{cuda_visible_devices}.json"
    )
    os.makedirs(os.path.dirname(path), exist_ok=True)
    from vllm.distributed.parallel_state import get_world_group

    if (not is_distributed or get_world_group().local_rank == 0) and (
        not os.path.exists(path)
    ):
        # only the local master process (with local_rank == 0) can
        #  enter this block to calculate the cache
        logger.info("generating GPU P2P access cache in %s", path)
        cache: dict[str, bool] = {}
        ids = list(range(num_dev))
        # batch of all pairs of GPUs
        batch_src, batch_tgt = zip(*list(product(ids, ids)))
        # NOTE: we use `subprocess` rather than `multiprocessing` here
        # because the caller might not have `if __name__ == "__main__":`,
        # in that case we cannot use spawn method in multiprocessing.
        # However, `can_actually_p2p` requires spawn method.
        # The fix is, we use `subprocess` to call the function,
        # where we have `if __name__ == "__main__":` in this file.

        # use a temporary file to store the result
        # we don't use the output of the subprocess directly,
        # because the subprocess might produce logging output
        with tempfile.NamedTemporaryFile() as output_file:
            input_bytes = pickle.dumps((batch_src, batch_tgt, output_file.name))
            returned = subprocess.run(
# ... truncated for analysis ...
            cache[f"{_i}->{_j}"] = r
        with open(path, "w") as f:
            json.dump(cache, f, indent=4)
    if is_distributed:
        get_world_group().barrier()
    logger.info("reading GPU P2P access cache from %s", path)
    with open(path) as f:
        cache = json.load(f)
    _gpu_p2p_access_cache = cache
    return _gpu_p2p_access_cache[f"{src}->{tgt}"]
```
**EN:** `gpu_p2p_access_check` implements a focused helper routine for this module. The docstring frames it as: Check if GPU src can access GPU tgt. It primarily works with arguments like `src`, `tgt`. Key calls include `dist.is_initialized`, `current_platform.device_count`, `os.path.join`.
**CN:** `gpu_p2p_access_check` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `src`, `tgt` 这样的参数。 关键调用包括 `dist.is_initialized`, `current_platform.device_count`, `os.path.join`。

### Module constants / 模块常量
```python
__all__ = ["gpu_p2p_access_check"]
```
**EN:** This section defines module-level aliases, constants, or shared state such as `__all__`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `__all__`，供后续代码复用。

### Conditional block / 条件代码块
```python
if __name__ == "__main__":
    batch_src, batch_tgt, output_file = pickle.loads(sys.stdin.buffer.read())
    result = can_actually_p2p(batch_src, batch_tgt)
    with open(output_file, "wb") as f:
        f.write(pickle.dumps(result))
```
**EN:** This top-level conditional adapts module behavior to a runtime capability, configuration flag, or platform detail.
**CN:** 该顶层条件分支会根据运行时能力、配置开关或平台细节调整模块行为。

## Key Concepts / 关键概念
- Collective communication backends / collective 通信后端
- `should_nccl_symm_mem_allreduce`: module-level helper or API entry / `should_nccl_symm_mem_allreduce`：模块级辅助函数或 API 入口
- `producer`: module-level helper or API entry / `producer`：模块级辅助函数或 API 入口
- `consumer`: module-level helper or API entry / `consumer`：模块级辅助函数或 API 入口
- `can_actually_p2p`: module-level helper or API entry / `can_actually_p2p`：模块级辅助函数或 API 入口
- `gpu_p2p_access_check`: module-level helper or API entry / `gpu_p2p_access_check`：模块级辅助函数或 API 入口

## Dependencies / 依赖关系
- **Standard library / 标准库**: `ctypes`, `json`, `os`, `pickle`, `subprocess`, `sys`, `tempfile`, `collections.abc`, `itertools`, `typing`
- **Third-party / 第三方**: `torch`, `torch.distributed`, `torch.multiprocessing`
- **Internal modules / 内部模块**: `vllm.envs`, `vllm.distributed.device_communicators.cuda_wrapper`, `vllm.logger`, `vllm.platforms`, `vllm.utils.system_utils`, `vllm.distributed.device_communicators.pynccl_allocator`, `vllm.distributed.parallel_state`
