# pynccl_allocator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/distributed/device_communicators/pynccl_allocator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed execution and communication part of the SRT runtime and implements logic centered on `pynccl_allocator`. It exposes primary entry points such as `is_symmetric_memory_enabled`, `set_graph_pool_id`, `disable_symmetric_memory_context`. / 该模块属于 SRT 运行时的分布式执行与通信部分，主要实现围绕 `pynccl_allocator` 的逻辑。 它对外提供的主要入口包括 `is_symmetric_memory_enabled`, `set_graph_pool_id`, `disable_symmetric_memory_context`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60: Module imports, constants, and setup (part 1/3)
```python
import ctypes
import logging
import os
import tempfile
import traceback
from contextlib import nullcontext

import torch
from torch.cuda.memory import (
    CUDAPluggableAllocator,
    _cuda_beginAllocateCurrentThreadToPool,
    _cuda_endAllocateToPool,
    _cuda_releasePool,
)

from sglang.srt.distributed.parallel_state import GroupCoordinator
from sglang.srt.environ import envs
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils.common import torch_release

after_2_8_0 = torch_release >= (2, 8)

# C++ source for the NCCL allocator plugin
# Key design:
# 1. nccl_alloc_plug: Allocates memory via ncclMemAlloc and TRACKS the segment
#    (ptr, size). Does NOT register with any comm at allocation time.
# 2. nccl_free_plug: Frees memory via ncclMemFree and UNTRACKS the segment.
#    Each segment is tracked only during its lifetime (from alloc to free).
# 3. Segment tracking uses thread-safe std::vector + unordered_map for O(1) operations.
# 4. Registration via nccl_allocator_register_segments_with_comm: Registers all
#    tracked segments with a given comm, using index-based tracking to avoid
#    re-registration. Registration state is maintained per-communicator in C++.
nccl_allocator_source = """

#include <cuda_runtime.h>
#include <mutex>
#include <vector>
#include <unordered_map>
#include <utility>

extern "C" {

// copy from https://github.com/NVIDIA/nccl/blob/master/src/nccl.h.in
typedef enum { ncclSuccess                 =  0,
               ncclUnhandledCudaError      =  1,
               ncclSystemError             =  2,
               ncclInternalError           =  3,
               ncclInvalidArgument         =  4,
               ncclInvalidUsage            =  5,
               ncclRemoteError             =  6,
               ncclInProgress              =  7,
               ncclNumResults              =  8 } ncclResult_t;

// NCCL symmetric memory window flags
#define NCCL_WIN_COLL_SYMMETRIC 0x01

typedef struct ncclComm* ncclComm_t;
typedef struct ncclWindow_vidmem* ncclWindow_t;

ncclResult_t  ncclMemAlloc(void** ptr, size_t size);
```
**EN:** This range organizes module-level state and shared setup. This chunk is part 1 of 3 for the same logical block. In this range it sets up imports and shared symbols; reads environment-driven configuration; records metrics or tracing signals.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 该片段是同一逻辑块的第 1/3 部分。 在这一范围内，它会建立导入关系并准备共享符号；读取环境变量驱动的配置；记录指标或追踪信号。

### Lines 61-120: Module-level constants and helpers (part 2/3)
```python
ncclResult_t  ncclMemFree(void *ptr);
ncclResult_t  ncclCommWindowRegister(ncclComm_t comm, void* buff, size_t size, ncclWindow_t* win, int winFlags);
const char*  ncclGetErrorString(ncclResult_t result);

#define NCCLCHECK(cmd) do {                                               \
  ncclResult_t res = cmd;                                                 \
  if (res != ncclSuccess) {                                               \
    fprintf(stderr, "ERROR: NCCL symmetric memory allocation failed. Most likely out of device memory. '%s'\\n", \
           ncclGetErrorString(res));                       \
    return NULL;                                                        \
  }                                                                       \
} while(0)

// Segment information structure
struct Segment {
    void* ptr;
    size_t size;
    Segment(void* p, size_t s) : ptr(p), size(s) {}
};

// Thread-safe segment tracking
// Segment tracking using std::vector for FIFO order.
// g_segments is maintained in insertion order (oldest first).
static std::vector<Segment> g_segments;
static std::mutex g_segment_mutex;

// Track which segments have been registered with each communicator.
// Key: comm_ptr, Value: the next segment index to register for this comm.
static std::unordered_map<uintptr_t, size_t> g_comm_registration_index;

// Add a segment to the tracking (appends to end, maintaining FIFO order)
static void track_segment(void* ptr, size_t size) {
    std::lock_guard<std::mutex> lock(g_segment_mutex);
    g_segments.emplace_back(ptr, size);
}

void* nccl_alloc_plug(size_t size, int device, void* stream) {
    void* ptr;
    NCCLCHECK(ncclMemAlloc(&ptr, size));

    // Track the segment but do NOT register with any comm
    // Registration will be done at context exit via register_segments_with_comm
    track_segment(ptr, size);

    return ptr;
}

void nccl_free_plug(void* ptr, size_t size, int device, void* stream) {
    ncclResult_t err = ncclMemFree(ptr);
    // NOTE: We assume that no individual allocation will be freed until the
    // entire memory pool is destroyed. If this assumption does not hold,
    // we will encounter asymmetry issues between GPUs. For now, we clear
    // all tracking state when the pool is destroyed.
    std::lock_guard<std::mutex> lock(g_segment_mutex);
    g_segments = std::vector<Segment>();
    g_comm_registration_index = std::unordered_map<uintptr_t, size_t>();
}

// Register all tracked segments with a communicator.
// Uses an index-based approach to avoid re-registering already-registered segments.
```
**EN:** This range organizes module-level state and shared setup. This chunk is part 2 of 3 for the same logical block. In this range it records metrics or tracing signals.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 该片段是同一逻辑块的第 2/3 部分。 在这一范围内，它会记录指标或追踪信号。

### Lines 121-159: Module imports, constants, and setup (part 3/3)
```python
// Returns 0 on success, non-zero on failure.
int nccl_allocator_register_segments_with_comm(uintptr_t comm_ptr) {
    std::lock_guard<std::mutex> lock(g_segment_mutex);

    ncclComm_t comm = reinterpret_cast<ncclComm_t>(comm_ptr);

    // Get the starting index for this communicator
    size_t start_index = g_comm_registration_index[comm_ptr];

    // Register all segments from start_index to the current end
    for (size_t i = start_index; i < g_segments.size(); ++i) {
        const Segment& seg = g_segments[i];
        ncclWindow_t win;
        ncclResult_t res = ncclCommWindowRegister(comm, seg.ptr, seg.size, &win, NCCL_WIN_COLL_SYMMETRIC);
        if (res != ncclSuccess) {
            fprintf(stderr, "ERROR: NCCL symmetric memory registration failed. '%s'\\n", ncclGetErrorString(res));
            return res;
        }
    }

    // Update the registration index for this communicator
    g_comm_registration_index[comm_ptr] = g_segments.size();

    return ncclSuccess;
}

}
"""

_allocator = None
_mem_pool = None
_graph_pool_id = None
_cur_device = None
_active_symmetric_memory_context = None

# Reference to the C registration function (with arg types set)
_register_func = None


```
**EN:** This range organizes module-level state and shared setup. This chunk is part 3 of 3 for the same logical block. In this range it sets up imports and shared symbols; records metrics or tracing signals; manages graph capture or replay logic.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 该片段是同一逻辑块的第 3/3 部分。 在这一范围内，它会建立导入关系并准备共享符号；记录指标或追踪信号；管理图捕获或回放逻辑。

### Lines 160-164: Function is_symmetric_memory_enabled
```python
def is_symmetric_memory_enabled():
    try:
        return get_global_server_args().enable_symm_mem
    except ValueError:
        return False
```
**EN:** This callable implements `is_symmetric_memory_enabled` and mainly implements is symmetric memory enabled. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `is_symmetric_memory_enabled`，主要用于实现 is symmetric memory enabled 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 167-169: Function set_graph_pool_id
```python
def set_graph_pool_id(graph_pool_id):
    global _graph_pool_id
    _graph_pool_id = graph_pool_id
```
**EN:** This callable implements `set_graph_pool_id`. It takes `graph_pool_id` and mainly applies configuration to mutable state. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `set_graph_pool_id`。它接收 `graph_pool_id`，主要用于将配置写入可变状态。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 172-177: Function disable_symmetric_memory_context
```python
def disable_symmetric_memory_context():
    if _active_symmetric_memory_context is None:
        return None
    saved_context = _active_symmetric_memory_context
    saved_context.__exit__(None, None, None)
    return saved_context
```
**EN:** This callable implements `disable_symmetric_memory_context` and mainly implements disable symmetric memory context. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `disable_symmetric_memory_context`，主要用于实现 disable symmetric memory context 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 180-182: Function restore_symmetric_memory_context
```python
def restore_symmetric_memory_context(saved_context):
    if saved_context is not None:
        saved_context.__enter__()
```
**EN:** This callable implements `restore_symmetric_memory_context`. It takes `saved_context` and mainly converts data into another representation. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `restore_symmetric_memory_context`。它接收 `saved_context`，主要用于将数据转换为另一种表示。 在这一范围内，它会记录指标或追踪信号。

### Lines 185-231: Function get_nccl_mem_pool
```python
def get_nccl_mem_pool() -> torch.cuda.MemPool:
    """
    Get the shared MemPool for all groups.

    All groups share the same pool to avoid memory fragmentation.
    Comm registration is handled at context exit time.
    """
    global _allocator, _mem_pool, _cur_device, _register_func
    if _allocator is None:
        import torch.utils.cpp_extension

        out_dir = os.path.join(tempfile.gettempdir(), "symm_allocator")
        os.makedirs(out_dir, exist_ok=True)
        # Make sure to clean up leftover pytorch lock files
        # from previous runs and synchronize across processes
        # right after
        try:
            os.remove(os.path.join(out_dir, "lock"))
        except FileNotFoundError:
            pass
        torch.distributed.barrier()

        nccl_allocator_libname = "nccl_allocator"
        lib_path = torch.utils.cpp_extension.load_inline(
            name=nccl_allocator_libname,
            cpp_sources=nccl_allocator_source,
            with_cuda=True,
            extra_ldflags=["-lnccl"],
            verbose=True,
            is_python_module=False,
            build_directory=out_dir,
        )
        nccl_allocator_lib = ctypes.CDLL(lib_path)
        _allocator = CUDAPluggableAllocator(
            f"{out_dir}/{nccl_allocator_libname}.so",
            "nccl_alloc_plug",
            "nccl_free_plug",
        ).allocator()
        _mem_pool = torch.cuda.MemPool(_allocator)
        _cur_device = torch.cuda.current_device()

        # Setup the C function for registration with correct arg types
        _register_func = nccl_allocator_lib.nccl_allocator_register_segments_with_comm
        _register_func.restype = ctypes.c_int
        _register_func.argtypes = [ctypes.c_uint64]

    return _mem_pool
```
**EN:** This callable implements `get_nccl_mem_pool` and mainly retrieves a value or derived view. The docstring states: "Get the shared MemPool for all groups." In this range it sets up imports and shared symbols; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `get_nccl_mem_pool`，主要用于获取某个值或派生视图。 在这一范围内，它会建立导入关系并准备共享符号；协调分布式通信。

### Lines 234-251: Class SymmetricMemoryContext
```python
class SymmetricMemoryContext:
    """
    Context manager for using symmetric memory with pynccl.

    To Utilize the symmetric memory feature in NCCL, the buffers need to be allocated
    by `ncclMemAlloc` and registered by `ncclCommWindowRegister`. Due to this, we introduce
    this context manager. All tensors created under this context will be correctly
    allocated and registered with a custom allocator.

    Key design:
    - All groups share a single MemPool to avoid memory fragmentation.
    - At allocation time, ptrs are tracked but NOT registered with any comm.
    - At context exit time, nccl_allocator_register_segments_with_comm is called
      to register all tracked segments with the current comm. The C++ layer
      tracks per-comm registration state using index-based tracking to avoid
      re-registration of already-registered segments.
    """

```
**EN:** This range introduces `SymmetricMemoryContext` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Context manager for using symmetric memory with pynccl." In this range it records metrics or tracing signals.
**CN:** 这一段引入 `SymmetricMemoryContext`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 252-263: Method SymmetricMemoryContext.__init__
```python
    def __init__(
        self,
        group_coordinator: GroupCoordinator,
    ):
        self.group_coordinator = group_coordinator
        self._pool_id = get_nccl_mem_pool().id
        self._device_index = torch.cuda.current_device()
        self.is_graph_capture = torch.cuda.is_current_stream_capturing()

        # Get comm ptr for tracking registrations
        # Use the comm pointer value as unique identifier
        self._comm_ptr = self.group_coordinator.pynccl_comm.comm.value
```
**EN:** This callable implements `SymmetricMemoryContext.__init__`. It takes `group_coordinator` and mainly initializes instance state and defaults. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `SymmetricMemoryContext.__init__`。它接收 `group_coordinator`，主要用于初始化实例状态与默认值。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 265-287: Method SymmetricMemoryContext.__enter__
```python
    def __enter__(self):
        assert (
            self.group_coordinator.pynccl_comm is not None
        ), f"Symmetric memory requires pynccl to be enabled in group '{self.group_coordinator.unique_name}'"

        if self.is_graph_capture:
            assert (
                _graph_pool_id is not None
            ), "graph_pool_id is not set under graph capture"
            # Pause graph memory pool to use symmetric memory with cuda graph
            if after_2_8_0:
                torch._C._cuda_endAllocateToPool(_cur_device, _graph_pool_id)
            else:
                torch._C._cuda_endAllocateCurrentStreamToPool(
                    _cur_device, _graph_pool_id
                )

        _cuda_beginAllocateCurrentThreadToPool(self._device_index, self._pool_id)

        global _active_symmetric_memory_context
        _active_symmetric_memory_context = self

        return self
```
**EN:** This callable implements `SymmetricMemoryContext.__enter__` and mainly implements enter. In this range it performs defensive checks on invalid state; records metrics or tracing signals; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `SymmetricMemoryContext.__enter__`，主要用于实现 enter 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；记录指标或追踪信号；管理图捕获或回放逻辑。

### Lines 289-305: Method SymmetricMemoryContext.__exit__
```python
    def __exit__(self, exc_type, exc_val, exc_tb):
        _cuda_endAllocateToPool(self._device_index, self._pool_id)
        _cuda_releasePool(self._device_index, self._pool_id)
        # Register all unregistered segments
        # with the current comm
        self._register_segments_for_comm()

        if self.is_graph_capture:
            if after_2_8_0:
                torch._C._cuda_beginAllocateCurrentThreadToPool(
                    _cur_device, _graph_pool_id
                )
            else:
                torch._C._cuda_beginAllocateToPool(_cur_device, _graph_pool_id)

        global _active_symmetric_memory_context
        _active_symmetric_memory_context = None
```
**EN:** This callable implements `SymmetricMemoryContext.__exit__`. It takes `exc_type`, `exc_val`, `exc_tb` and mainly implements exit. In this range it records metrics or tracing signals; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `SymmetricMemoryContext.__exit__`。它接收 `exc_type`, `exc_val`, `exc_tb`，主要用于实现 exit 相关逻辑。 在这一范围内，它会记录指标或追踪信号；管理图捕获或回放逻辑。

### Lines 307-322: Method SymmetricMemoryContext._register_segments_for_comm
```python
    def _register_segments_for_comm(self):
        """
        Register all tracked segments with the current comm.

        Delegates to C++ layer which handles:
        1. Tracking which segments have been registered with each comm
        2. Only registering new segments (avoiding re-registration)
        3. Thread-safe access to the segment registry
        """

        # Call C++ API to register all segments with this comm
        # C++ layer tracks per-comm registration state internally
        result = _register_func(self._comm_ptr)
        assert (
            result == 0
        ), f"nccl_allocator_register_segments_with_comm failed with return code: {result}"
```
**EN:** This callable implements `SymmetricMemoryContext._register_segments_for_comm` and mainly registers hooks, handlers, or operators. The docstring states: "Register all tracked segments with the current comm." In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `SymmetricMemoryContext._register_segments_for_comm`，主要用于注册钩子、处理器或算子。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 325-331: Function use_symmetric_memory
```python
def use_symmetric_memory(group_coordinator: GroupCoordinator, disabled: bool = False):
    disabled = (
        not is_symmetric_memory_enabled()
        or disabled
        or group_coordinator.world_size == 1
    )
    return SymmetricMemoryContext(group_coordinator) if not disabled else nullcontext()
```
**EN:** This callable implements `use_symmetric_memory`. It takes `group_coordinator`, `disabled` and mainly implements use symmetric memory. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `use_symmetric_memory`。它接收 `group_coordinator`, `disabled`，主要用于实现 use symmetric memory 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 332-339: Module-level constants and helpers
```python


# --- Debug mode for symmetric memory validation ---

_symm_mem_logger = logging.getLogger(__name__)
_debug_seen_traces: set = set()


```
**EN:** This range organizes module-level state and shared setup. In this range it emits logs for diagnostics; records metrics or tracing signals.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会输出日志以便诊断；记录指标或追踪信号。

### Lines 340-352: Function is_tensor_in_symmetric_mempool
```python
def is_tensor_in_symmetric_mempool(tensor: torch.Tensor) -> bool:
    """Check if a tensor's storage is allocated in the NCCL symmetric memory pool."""

    if _mem_pool is None:
        return False  # Pool not initialized

    data_ptr = tensor.untyped_storage().data_ptr()

    for segment in _mem_pool.snapshot():
        for block in segment["blocks"]:
            if block["address"] == data_ptr:
                return True
    return False
```
**EN:** This callable implements `is_tensor_in_symmetric_mempool`. It takes `tensor` and mainly implements is tensor in symmetric mempool. The docstring states: "Check if a tensor's storage is allocated in the NCCL symmetric memory pool." In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `is_tensor_in_symmetric_mempool`。它接收 `tensor`，主要用于实现 is tensor in symmetric mempool 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 355-406: Function debug_check_symmetric_mempool
```python
def debug_check_symmetric_mempool(
    group_coordinator: GroupCoordinator,
    tensors: dict,
    op_name: str,
) -> None:
    """
    Debug check: verify that tensors passed to communication ops are allocated
    in the NCCL symmetric memory pool.

    Enabled by setting SGLANG_DEBUG_SYMM_MEM=1.
    Only prints warnings on rank 0 and deduplicates identical stack traces.

    Args:
        tensors: dict mapping argument name to tensor
                 (e.g. {"input": t1, "output": t2})
        op_name: name of the communication operation being checked
    """
    if not envs.SGLANG_DEBUG_SYMM_MEM.get() or not is_symmetric_memory_enabled():
        return

    # Only print on rank 0
    if not group_coordinator.is_first_rank:
        return

    bad_names = []
    bad_details = []
    for name, tensor in tensors.items():
        if not is_tensor_in_symmetric_mempool(tensor):
            bad_names.append(name)
            bad_details.append(
                f"  - '{name}' (data_ptr=0x{tensor.storage().data_ptr():x}, "
                f"shape={list(tensor.shape)}, dtype={tensor.dtype})"
            )

    if bad_names:
        traces = traceback.format_stack()
        # Skip autotune stack traces
        if any("_flashinfer_autotune" in trace for trace in traces):
            return
        stack = "".join(traces[:-1])
        trace_key = f"{op_name}:{','.join(bad_names)}:{stack}"
        if trace_key not in _debug_seen_traces:
            _debug_seen_traces.add(trace_key)
            _symm_mem_logger.warning(
                "[SymmMem Debug] %s: %d tensor(s) are NOT in the "
                "NCCL symmetric memory pool:\n%s\n"
                "Stack trace:\n%s",
                op_name,
                len(bad_names),
                "\n".join(bad_details),
                stack,
            )
```
**EN:** This callable implements `debug_check_symmetric_mempool`. It takes `group_coordinator`, `tensors`, `op_name` and mainly checks preconditions and compatibility. The docstring states: "Debug check: verify that tensors passed to communication ops are allocated in the NCCL symmetric memory pool." In this range it emits logs for diagnostics; reads environment-driven configuration; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `debug_check_symmetric_mempool`。它接收 `group_coordinator`, `tensors`, `op_name`，主要用于检查前置条件与兼容性。 在这一范围内，它会输出日志以便诊断；读取环境变量驱动的配置；记录指标或追踪信号。

## Key Concepts / 关键概念
- `is_symmetric_memory_enabled`: implements is symmetric memory enabled / 实现 is symmetric memory enabled 相关逻辑
- `set_graph_pool_id`: applies configuration to mutable state / 将配置写入可变状态
- `disable_symmetric_memory_context`: implements disable symmetric memory context / 实现 disable symmetric memory context 相关逻辑
- `restore_symmetric_memory_context`: converts data into another representation / 将数据转换为另一种表示
- `get_nccl_mem_pool`: retrieves a value or derived view / 获取某个值或派生视图
- `SymmetricMemoryContext`: core class or state container / 核心类或状态容器
- `use_symmetric_memory`: implements use symmetric memory / 实现 use symmetric memory 相关逻辑
- `is_tensor_in_symmetric_mempool`: implements is tensor in symmetric mempool / 实现 is tensor in symmetric mempool 相关逻辑
- `debug_check_symmetric_mempool`: checks preconditions and compatibility / 检查前置条件与兼容性

## Dependencies / 依赖关系
- **Standard library / 标准库**: `ctypes`, `logging`, `os`, `tempfile`, `traceback`, `contextlib`
- **Third-party / 第三方**: `torch`, `torch.cuda.memory`, `torch.utils.cpp_extension`
- **Internal modules / 内部模块**: `sglang.srt.distributed.parallel_state`, `sglang.srt.environ`, `sglang.srt.server_args`, `sglang.srt.utils.common`
