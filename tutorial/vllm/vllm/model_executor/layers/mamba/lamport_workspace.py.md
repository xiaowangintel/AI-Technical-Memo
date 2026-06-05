# lamport_workspace.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/mamba/lamport_workspace.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `IpcBuffer`, `LamportWorkspace` for Mamba/state-space layers and kernels. / 实现 `IpcBuffer`, `LamportWorkspace`，用于Mamba/状态空间模型层与内核。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 5-11)
```python
import array
import contextlib
import struct
import sys
import threading

import torch
```
**EN:** This opening block pulls in external dependencies such as `array`, `contextlib`, `struct`, `sys`, `threading`, `torch` and internal modules such as no internal imports. That import mix shows the file is part of the Mamba/state-space layers and kernels stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `array`, `contextlib`, `struct`, `sys`, `threading`, `torch`）以及内部模块（如 no internal imports）。这些导入关系表明该文件属于Mamba/状态空间模型层与内核栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 18-264)
```python
_ALIGN = 1 << 21  # 2 MiB — CUDA IPC allocation alignment


# ---------------------------------------------------------------------------
# CUDA helpers
# ---------------------------------------------------------------------------


def _check(error):
    """Raise on CUDA runtime error."""
    success = getattr(cudart.cudaError_t, "cudaSuccess", None) or cudart.cudaError_t(0)
    if error != success:
        raise RuntimeError(f"CUDA runtime error: {error}")


def _cuda_malloc(size: int):
    aligned = ((size + _ALIGN - 1) >> 21) << 21
    err, ptr = cudart.cudaMalloc(aligned)
    _check(err)
    return ptr, aligned


def _cuda_free(ptr: int):
    if ptr:
        _check(cudart.cudaFree(ptr)[0])


def _cuda_memset_zero(ptr: int, size: int):
    _check(cudart.cudaMemset(ptr, 0, size)[0])


def _cuda_memcpy_d2d(dst: int, src: int, size: int):
    _check(
        cudart.cudaMemcpy(
            dst, src, size, cudart.cudaMemcpyKind.cudaMemcpyDeviceToDevice
        )[0]
    )


# ---------------------------------------------------------------------------
# IPC buffer
# ---------------------------------------------------------------------------


class IpcBuffer:
# ... truncated for analysis ...
        return (
            f"LamportWorkspace(rank={self.rank}, world_size={self.world_size}, "
            f"comm_size={self.comm_size})"
        )


# ---------------------------------------------------------------------------
# Cached convenience function (mirrors TRT-LLM's get_allreduce_workspace)
# ---------------------------------------------------------------------------

_cache_lock = threading.Lock()
_workspace_cache: dict = {}
```
**EN:** This block defines module-level metadata or constants such as `_ALIGN`, `_cache_lock`, `_workspace_cache`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the Mamba/state-space layers and kernels pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `_ALIGN`, `_cache_lock`, `_workspace_cache`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在Mamba/状态空间模型层与内核流程中复用。

### Function `_check` (lines 26-30)
```python
def _check(error):
    """Raise on CUDA runtime error."""
    success = getattr(cudart.cudaError_t, "cudaSuccess", None) or cudart.cudaError_t(0)
    if error != success:
        raise RuntimeError(f"CUDA runtime error: {error}")
```
**EN:** Defines function `_check` with signature `_check(error)`. It mainly works with `error`; implements one step of the Mamba/SSM execution path. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `getattr`, `cudart.cudaError_t`, `RuntimeError`.
**CN:** 定义函数 `_check`，其签名为 `_check(error)`。它主要围绕 `error` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `getattr`, `cudart.cudaError_t`, `RuntimeError`。

### Function `_cuda_malloc` (lines 33-37)
```python
def _cuda_malloc(size: int):
    aligned = ((size + _ALIGN - 1) >> 21) << 21
    err, ptr = cudart.cudaMalloc(aligned)
    _check(err)
    return ptr, aligned
```
**EN:** Defines function `_cuda_malloc` with signature `_cuda_malloc(size: int)`. It mainly works with `size`; implements one step of the Mamba/SSM execution path. The body uses tensor/kernel operations. Key calls include `cudart.cudaMalloc`, `_check`.
**CN:** 定义函数 `_cuda_malloc`，其签名为 `_cuda_malloc(size: int)`。它主要围绕 `size` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含张量或内核操作。关键调用包括 `cudart.cudaMalloc`, `_check`。

### Function `_cuda_free` (lines 40-42)
```python
def _cuda_free(ptr: int):
    if ptr:
        _check(cudart.cudaFree(ptr)[0])
```
**EN:** Defines function `_cuda_free` with signature `_cuda_free(ptr: int)`. It mainly works with `ptr`; implements one step of the Mamba/SSM execution path. The body uses branching, tensor/kernel operations. Key calls include `_check`, `cudart.cudaFree`.
**CN:** 定义函数 `_cuda_free`，其签名为 `_cuda_free(ptr: int)`。它主要围绕 `ptr` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断、张量或内核操作。关键调用包括 `_check`, `cudart.cudaFree`。

### Function `_cuda_memset_zero` (lines 45-46)
```python
def _cuda_memset_zero(ptr: int, size: int):
    _check(cudart.cudaMemset(ptr, 0, size)[0])
```
**EN:** Defines function `_cuda_memset_zero` with signature `_cuda_memset_zero(ptr: int, size: int)`. It mainly works with `ptr`, `size`; implements one step of the Mamba/SSM execution path. The body uses tensor/kernel operations. Key calls include `_check`, `cudart.cudaMemset`.
**CN:** 定义函数 `_cuda_memset_zero`，其签名为 `_cuda_memset_zero(ptr: int, size: int)`。它主要围绕 `ptr`, `size` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含张量或内核操作。关键调用包括 `_check`, `cudart.cudaMemset`。

### Function `_cuda_memcpy_d2d` (lines 49-54)
```python
def _cuda_memcpy_d2d(dst: int, src: int, size: int):
    _check(
        cudart.cudaMemcpy(
            dst, src, size, cudart.cudaMemcpyKind.cudaMemcpyDeviceToDevice
        )[0]
    )
```
**EN:** Defines function `_cuda_memcpy_d2d` with signature `_cuda_memcpy_d2d(dst: int, src: int, size: int)`. It mainly works with `dst`, `src`, `size`; implements one step of the Mamba/SSM execution path. The body uses tensor/kernel operations. Key calls include `_check`, `cudart.cudaMemcpy`.
**CN:** 定义函数 `_cuda_memcpy_d2d`，其签名为 `_cuda_memcpy_d2d(dst: int, src: int, size: int)`。它主要围绕 `dst`, `src`, `size` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含张量或内核操作。关键调用包括 `_check`, `cudart.cudaMemcpy`。

### Function `_lamport_fill_neg_zero` (lines 135-147)
```python
def _lamport_fill_neg_zero(device_ptr: int, size_bytes: int):
    """
    Fill device memory with IEEE-754 negative zero (-0.0f = 0x80000000).
    This is the "slot empty" sentinel for the Lamport protocol: the kernel
    spin-waits until a value is *not* negative zero.
    """
    if size_bytes == 0 or device_ptr == 0:
        return
    n_floats = size_bytes // 4
    # torch preserves -0.0 in IEEE-754
    fill = torch.full((n_floats,), -0.0, dtype=torch.float32, device="cuda")
    _cuda_memcpy_d2d(device_ptr, fill.data_ptr(), size_bytes)
    del fill
```
**EN:** Defines function `_lamport_fill_neg_zero` with signature `_lamport_fill_neg_zero(device_ptr: int, size_bytes: int)`. It mainly works with `device_ptr`, `size_bytes`; implements one step of the Mamba/SSM execution path. The body uses branching, tensor/kernel operations. Key calls include `torch.full`, `_cuda_memcpy_d2d`, `fill.data_ptr`.
**CN:** 定义函数 `_lamport_fill_neg_zero`，其签名为 `_lamport_fill_neg_zero(device_ptr: int, size_bytes: int)`。它主要围绕 `device_ptr`, `size_bytes` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断、张量或内核操作。关键调用包括 `torch.full`, `_cuda_memcpy_d2d`, `fill.data_ptr`。

### Function `get_allreduce_workspace` (lines 267-302)
```python
def get_allreduce_workspace(
    rank: int,
    world_size: int,
    comm_size: int | None = None,
    max_tokens: int = 16384,
    process_group=None,
) -> torch.Tensor:
    """
    Return a cached workspace tensor for the given (rank, world_size) pair.

    On first call the workspace is allocated and IPC handles are exchanged;
    subsequent calls with the same arguments return the cached tensor.

    Parameters
    ----------
    rank, world_size : int
        TP rank and TP size.
    comm_size : int, optional
        Explicit slot size in bytes.  If ``None``, computed automatically
        from ``max_tokens`` and ``world_size`` (fused Q+K path).
    max_tokens : int
        Maximum number of tokens per batch (used when ``comm_size is None``).
    process_group : optional
        ``torch.distributed`` process group.
    """
    if comm_size is None:
        comm_size = LamportWorkspace.compute_comm_size_for_minimax(
            max_tokens, world_size, fused_qk=True
        )
    pg_id = id(process_group) if process_group is not None else 0
    key = (rank, world_size, comm_size, pg_id)
    with _cache_lock:
        if key not in _workspace_cache:
            ws = LamportWorkspace(rank, world_size, comm_size, process_group)
            _workspace_cache[key] = ws
        return _workspace_cache[key].workspace
```
**EN:** Defines function `get_allreduce_workspace` with signature `get_allreduce_workspace(rank: int, world_size: int, comm_size: int | None=None, max_tokens: int=16384, process_group=None) -> torch.Tensor`. It mainly works with `rank`, `world_size`, `comm_size`, `max_tokens`, `process_group`; returns a derived property or capability check. The body uses branching, context-managed execution. Key calls include `LamportWorkspace.compute_comm_size_for_minimax`, `id`, `LamportWorkspace`.
**CN:** 定义函数 `get_allreduce_workspace`，其签名为 `get_allreduce_workspace(rank: int, world_size: int, comm_size: int | None=None, max_tokens: int=16384, process_group=None) -> torch.Tensor`。它主要围绕 `rank`, `world_size`, `comm_size`, `max_tokens`, `process_group` 展开；返回派生属性或能力判断结果。函数体包含分支判断、上下文管理。关键调用包括 `LamportWorkspace.compute_comm_size_for_minimax`, `id`, `LamportWorkspace`。

### Class `IpcBuffer` overview (lines 62-127)
```python
class IpcBuffer:
    """
    Allocates CUDA device memory and exchanges IPC handles with all ranks
    so that every rank holds a valid device pointer to every other rank's buffer.
    """

    def __init__(self, rank: int, world_size: int, size: int, process_group=None):
        self.rank = rank
        self.world_size = world_size
        self.peer_ptrs: list[int] = [0] * world_size
        self.local_ptr: int = 0
        self._alive = False

        if size <= 0:
            return

        self.local_ptr, _ = _cuda_malloc(size)
        _cuda_memset_zero(self.local_ptr, size)
        self._alive = True

        # --- exchange IPC handles via torch.distributed ---
        err, local_handle = cudart.cudaIpcGetMemHandle(self.local_ptr)
        _check(err)

        all_handles: list[bytes | None] = [None] * world_size
```
**EN:** Defines class `IpcBuffer` with base classes no explicit base class and decorators none. It acts as a Mamba/SSM module building block and exposes 4 direct methods, with notable entries `__init__`, `serialize`, `cleanup`, `__del__`. Its docstring says: Allocates CUDA device memory and exchanges IPC handles with all ranks so that every rank holds a valid device pointer to every other rank's buffer.
**CN:** 定义类 `IpcBuffer`，其基类为 无显式基类，装饰器为 无。它在整体实现中充当Mamba/状态空间模块构件，并直接暴露 4 个方法，较重要的包括 `__init__`, `serialize`, `cleanup`, `__del__`。 文档字符串进一步说明了该类的定位。

### Method `IpcBuffer.__init__` (lines 68-101)
```python
    def __init__(self, rank: int, world_size: int, size: int, process_group=None):
        self.rank = rank
        self.world_size = world_size
        self.peer_ptrs: list[int] = [0] * world_size
        self.local_ptr: int = 0
        self._alive = False

        if size <= 0:
            return

        self.local_ptr, _ = _cuda_malloc(size)
        _cuda_memset_zero(self.local_ptr, size)
        self._alive = True

        # --- exchange IPC handles via torch.distributed ---
        err, local_handle = cudart.cudaIpcGetMemHandle(self.local_ptr)
        _check(err)

        all_handles: list[bytes | None] = [None] * world_size
        torch.distributed.all_gather_object(
            all_handles, bytes(local_handle.reserved), group=process_group
        )

        for r in range(world_size):
            if r == rank:
                self.peer_ptrs[r] = self.local_ptr
            else:
                handle = cudart.cudaIpcMemHandle_t()
                handle.reserved = all_handles[r]
                err, ptr = cudart.cudaIpcOpenMemHandle(
                    handle, cudart.cudaIpcMemLazyEnablePeerAccess
                )
                _check(err)
                self.peer_ptrs[r] = ptr
```
**EN:** Defines function `IpcBuffer.__init__` with signature `__init__(self, rank: int, world_size: int, size: int, process_group=None)`. It mainly works with `rank`, `world_size`, `size`, `process_group`; initializes the object state and cached resources. The body uses branching, iteration, tensor/kernel operations. Key calls include `_cuda_malloc`, `_cuda_memset_zero`, `cudart.cudaIpcGetMemHandle`, `_check`, `torch.distributed.all_gather_object`, `range`.
**CN:** 定义函数 `IpcBuffer.__init__`，其签名为 `__init__(self, rank: int, world_size: int, size: int, process_group=None)`。它主要围绕 `rank`, `world_size`, `size`, `process_group` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、循环处理、张量或内核操作。关键调用包括 `_cuda_malloc`, `_cuda_memset_zero`, `cudart.cudaIpcGetMemHandle`, `_check`, `torch.distributed.all_gather_object`, `range`。

### Method `IpcBuffer.serialize` (lines 103-108)
```python
    def serialize(self) -> list[int]:
        """Return peer pointers as a list of int64 values (one per rank)."""
        raw = b""
        for ptr in self.peer_ptrs:
            raw += struct.pack("P", ptr)
        return array.array("Q", raw).tolist()
```
**EN:** Defines function `IpcBuffer.serialize` with signature `serialize(self) -> list[int]`. It mainly works with object context only; implements one step of the Mamba/SSM execution path. The body uses iteration. Key calls include `array.array.tolist`, `struct.pack`, `array.array`.
**CN:** 定义函数 `IpcBuffer.serialize`，其签名为 `serialize(self) -> list[int]`。它主要围绕 仅依赖对象上下文 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含循环处理。关键调用包括 `array.array.tolist`, `struct.pack`, `array.array`。

### Method `IpcBuffer.cleanup` (lines 110-123)
```python
    def cleanup(self):
        if not self._alive:
            return
        self._alive = False
        for r in range(self.world_size):
            if self.peer_ptrs[r] == 0:
                continue
            if r == self.rank:
                _cuda_free(self.peer_ptrs[r])
            else:
                with contextlib.suppress(RuntimeError):
                    _check(cudart.cudaIpcCloseMemHandle(self.peer_ptrs[r])[0])
            self.peer_ptrs[r] = 0
        self.local_ptr = 0
```
**EN:** Defines function `IpcBuffer.cleanup` with signature `cleanup(self)`. It mainly works with object context only; implements one step of the Mamba/SSM execution path. The body uses branching, iteration, context-managed execution, tensor/kernel operations. Key calls include `range`, `_cuda_free`, `contextlib.suppress`, `_check`, `cudart.cudaIpcCloseMemHandle`.
**CN:** 定义函数 `IpcBuffer.cleanup`，其签名为 `cleanup(self)`。它主要围绕 仅依赖对象上下文 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断、循环处理、上下文管理、张量或内核操作。关键调用包括 `range`, `_cuda_free`, `contextlib.suppress`, `_check`, `cudart.cudaIpcCloseMemHandle`。

### Method `IpcBuffer.__del__` (lines 125-127)
```python
    def __del__(self):
        if not sys.is_finalizing():
            self.cleanup()
```
**EN:** Defines function `IpcBuffer.__del__` with signature `__del__(self)`. It mainly works with object context only; implements one step of the Mamba/SSM execution path. The body uses branching. Key calls include `sys.is_finalizing`, `self.cleanup`.
**CN:** 定义函数 `IpcBuffer.__del__`，其签名为 `__del__(self)`。它主要围绕 仅依赖对象上下文 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断。关键调用包括 `sys.is_finalizing`, `self.cleanup`。

### Class `LamportWorkspace` overview (lines 155-256)
```python
class LamportWorkspace:
    """
    Self-contained workspace for Lamport-based cross-GPU AllReduce.

    Parameters
    ----------
    rank : int
        Local rank (0-based).
    world_size : int
        Total number of ranks in the TP group.
    comm_size : int
        Size in bytes of *one* Lamport buffer slot. The total IPC allocation
        per rank is ``3 * comm_size`` (triple-buffering). Must be large enough
        to hold the per-slot data written by the kernel.  Use
        ``compute_comm_size_for_minimax()`` for a safe default.
    process_group : optional
        ``torch.distributed`` process group for IPC handle exchange.
        ``None`` uses the default group.
    """

    def __init__(self, rank: int, world_size: int, comm_size: int, process_group=None):
        assert world_size >= 2, "Lamport workspace requires at least 2 ranks"
        assert comm_size > 0, "comm_size must be positive"

        self.rank = rank
```
**EN:** Defines class `LamportWorkspace` with base classes no explicit base class and decorators none. It acts as a workspace/buffer manager and exposes 6 direct methods, with notable entries `__init__`, `workspace`, `compute_comm_size_for_minimax`, `cleanup`, `__del__`, `__repr__`. Its docstring says: Self-contained workspace for Lamport-based cross-GPU AllReduce.
**CN:** 定义类 `LamportWorkspace`，其基类为 无显式基类，装饰器为 无。它在整体实现中充当工作区/缓冲区管理器，并直接暴露 6 个方法，较重要的包括 `__init__`, `workspace`, `compute_comm_size_for_minimax`, `cleanup`, `__del__`, `__repr__`。 文档字符串进一步说明了该类的定位。

### Method `LamportWorkspace.__init__` (lines 175-210)
```python
    def __init__(self, rank: int, world_size: int, comm_size: int, process_group=None):
        assert world_size >= 2, "Lamport workspace requires at least 2 ranks"
        assert comm_size > 0, "comm_size must be positive"

        self.rank = rank
        self.world_size = world_size
        self.comm_size = comm_size

        # 1) Lamport triple-buffer (the only IPC memory the kernel reads/writes)
        lamport_total = 3 * comm_size
        self._lamport = IpcBuffer(rank, world_size, lamport_total, process_group)
        _lamport_fill_neg_zero(self._lamport.local_ptr, lamport_total)

        # 2) flag_buffer on device: int32[3] = {counter, unused, lamport_flag}
        #    counter  — used for block-level sync inside the kernel
        #    unused   — reserved (index 1)
        #    lamport_flag — triple-buffer rotation index (0 → 1 → 2 → 0 …)
        self._flag_buf = torch.zeros(3, dtype=torch.int32, device="cuda")

        # 3) layout_buffer on device: int64[2] = {clear_size, comm_size}
        #    clear_size — bytes to clear from *previous* slot (set by kernel)
        #    comm_size  — size of one triple-buffer slot
        self._layout_buf = torch.tensor(
            [0, comm_size], dtype=torch.int64, device="cuda"
        )

        # 4) Assemble device-side void* pointer array
        N = world_size
        ptrs: list[int] = []
        ptrs += [0] * N  # [0   .. N-1]   ipc_buffers  (placeholder)
        ptrs += [0] * N  # [N   .. 2N-1]  ipc_barriers (placeholder)
        ptrs += self._lamport.serialize()  # [2N  .. 3N-1]  lamport peer ptrs
        ptrs.append(self._flag_buf.data_ptr())  # [3N]           flag_buffer
        ptrs.append(self._layout_buf.data_ptr())  # [3N+1]       layout_buffer

        self._workspace = torch.tensor(ptrs, dtype=torch.int64, device="cuda")
```
**EN:** Defines function `LamportWorkspace.__init__` with signature `__init__(self, rank: int, world_size: int, comm_size: int, process_group=None)`. It mainly works with `rank`, `world_size`, `comm_size`, `process_group`; initializes the object state and cached resources. The body uses validation/error handling, tensor/kernel operations. Key calls include `IpcBuffer`, `_lamport_fill_neg_zero`, `torch.zeros`, `torch.tensor`, `self._lamport.serialize`, `ptrs.append`.
**CN:** 定义函数 `LamportWorkspace.__init__`，其签名为 `__init__(self, rank: int, world_size: int, comm_size: int, process_group=None)`。它主要围绕 `rank`, `world_size`, `comm_size`, `process_group` 展开；负责初始化对象状态和缓存资源。函数体包含校验或报错逻辑、张量或内核操作。关键调用包括 `IpcBuffer`, `_lamport_fill_neg_zero`, `torch.zeros`, `torch.tensor`, `self._lamport.serialize`, `ptrs.append`。

### Method `LamportWorkspace.workspace` (lines 213-216)
```python
    def workspace(self) -> torch.Tensor:
        """Device tensor (int64) that can be passed to the kernel
        as ``void** workspace``."""
        return self._workspace
```
**EN:** Defines function `LamportWorkspace.workspace` with signature `workspace(self) -> torch.Tensor`. It mainly works with object context only; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `LamportWorkspace.workspace`，其签名为 `workspace(self) -> torch.Tensor`。它主要围绕 仅依赖对象上下文 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `LamportWorkspace.compute_comm_size_for_minimax` (lines 223-242)
```python
    def compute_comm_size_for_minimax(
        max_tokens: int,
        world_size: int,
        fused_qk: bool = True,
    ) -> int:
        """
        Return a safe ``comm_size`` (in bytes) for MiniMaxReduceRMSKernel.

        The kernel stores per-token variance scalars in the Lamport buffer:
          - single-matrix path: ``world_size × max_tokens × 4`` bytes per slot
          - fused Q+K path: ``world_size × 2 × ceil(max_tokens/4) × 16`` bytes per slot

        The returned value is rounded up to 2 MiB alignment.
        """
        if fused_qk:
            groups = (max_tokens + 3) // 4
            slot_bytes = world_size * 2 * groups * 16  # 16 = sizeof(float4)
        else:
            slot_bytes = world_size * max_tokens * 4  # 4  = sizeof(float)
        return ((slot_bytes + _ALIGN - 1) >> 21) << 21
```
**EN:** Defines function `LamportWorkspace.compute_comm_size_for_minimax` with signature `compute_comm_size_for_minimax(max_tokens: int, world_size: int, fused_qk: bool=True) -> int`. It mainly works with `max_tokens`, `world_size`, `fused_qk`; computes derived values used by later kernels or layers. The body uses branching. Key calls include no notable helper calls.
**CN:** 定义函数 `LamportWorkspace.compute_comm_size_for_minimax`，其签名为 `compute_comm_size_for_minimax(max_tokens: int, world_size: int, fused_qk: bool=True) -> int`。它主要围绕 `max_tokens`, `world_size`, `fused_qk` 展开；计算后续层或内核要使用的派生值。函数体包含分支判断。关键调用包括 无显著辅助调用。

### Method `LamportWorkspace.cleanup` (lines 244-246)
```python
    def cleanup(self):
        if hasattr(self, "_lamport"):
            self._lamport.cleanup()
```
**EN:** Defines function `LamportWorkspace.cleanup` with signature `cleanup(self)`. It mainly works with object context only; implements one step of the Mamba/SSM execution path. The body uses branching. Key calls include `hasattr`, `self._lamport.cleanup`.
**CN:** 定义函数 `LamportWorkspace.cleanup`，其签名为 `cleanup(self)`。它主要围绕 仅依赖对象上下文 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断。关键调用包括 `hasattr`, `self._lamport.cleanup`。

### Method `LamportWorkspace.__del__` (lines 248-250)
```python
    def __del__(self):
        if not sys.is_finalizing():
            self.cleanup()
```
**EN:** Defines function `LamportWorkspace.__del__` with signature `__del__(self)`. It mainly works with object context only; implements one step of the Mamba/SSM execution path. The body uses branching. Key calls include `sys.is_finalizing`, `self.cleanup`.
**CN:** 定义函数 `LamportWorkspace.__del__`，其签名为 `__del__(self)`。它主要围绕 仅依赖对象上下文 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断。关键调用包括 `sys.is_finalizing`, `self.cleanup`。

### Method `LamportWorkspace.__repr__` (lines 252-256)
```python
    def __repr__(self):
        return (
            f"LamportWorkspace(rank={self.rank}, world_size={self.world_size}, "
            f"comm_size={self.comm_size})"
        )
```
**EN:** Defines function `LamportWorkspace.__repr__` with signature `__repr__(self)`. It mainly works with object context only; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `LamportWorkspace.__repr__`，其签名为 `__repr__(self)`。它主要围绕 仅依赖对象上下文 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

## Key Concepts / 关键概念
- **EN:** The file belongs to the Mamba/SSM path, so recurrent state updates, scan-style kernels, or short convolutions are recurring themes.
  **CN:** 该文件位于 Mamba/状态空间模型路径中，因此循环状态更新、scan 风格内核或短卷积是反复出现的主题。
- **EN:** Top-level classes include `IpcBuffer`, `LamportWorkspace`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `IpcBuffer`, `LamportWorkspace`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `_check`, `_cuda_malloc`, `_cuda_free`, `_cuda_memset_zero`, `_cuda_memcpy_d2d` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `_check`, `_cuda_malloc`, `_cuda_free`, `_cuda_memset_zero`, `_cuda_memcpy_d2d` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `array`, `contextlib`, `struct`, `sys`, `threading`, `torch`
- **Internal / 内部**: None / 无
