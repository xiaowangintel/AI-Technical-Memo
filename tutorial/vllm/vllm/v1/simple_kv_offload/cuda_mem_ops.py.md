# cuda_mem_ops.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/simple_kv_offload/cuda_mem_ops.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Low-level CUDA/HIP memory helpers: pinning and batch DMA transfers. / 该模块位于 `simple_kv_offload` 子系统，主要围绕 `pin_tensor`, `_CUmemLocation`, `_CUmemcpyAttributes` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""Low-level CUDA/HIP memory helpers: pinning and batch DMA transfers."""

import ctypes
from typing import Any, NamedTuple

import numpy as np
import torch

from vllm.logger import init_logger
from vllm.platforms import current_platform

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `pin_tensor` function / `pin_tensor` 函数
```python
def pin_tensor(tensor: torch.Tensor) -> None:
    """Pin a CPU tensor via cudaHostRegister.

    This bypasses PyTorch's CUDACachingHostAllocator which rounds
    every ``pin_memory=True`` allocation up to the next power of 2
    (e.g. 100 GB becomes 128 GB).
    """
    err = torch.cuda.cudart().cudaHostRegister(tensor.data_ptr(), tensor.nbytes, 0)
    if err.value != 0:
        raise RuntimeError(f"cudaHostRegister failed: {err}")
```
**EN:** This function implements `pin_tensor` within the module. The docstring frames it as: Pin a CPU tensor via cudaHostRegister. Key calls include `cudaHostRegister`, `data_ptr`, `RuntimeError`, `cudart`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `pin_tensor`，其作用域位于the module。 关键调用包括 `cudaHostRegister`, `data_ptr`, `RuntimeError`, `cudart`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_CUmemLocation` class / `_CUmemLocation` 类
```python
class _CUmemLocation(ctypes.Structure):
    _fields_ = [("type", ctypes.c_uint), ("id", ctypes.c_int)]
```
**EN:** Introduces the `_CUmemLocation` class on top of `ctypes.Structure`. Core methods include its methods defined below.
**CN:** 这里定义 `_CUmemLocation` 类，其基类包括 `ctypes.Structure`。核心方法包括 下方定义的方法。

### `_CUmemcpyAttributes` class / `_CUmemcpyAttributes` 类
```python
class _CUmemcpyAttributes(ctypes.Structure):
    _fields_ = [
        ("srcAccessOrder", ctypes.c_uint),
        ("srcLocHint", _CUmemLocation),
        ("dstLocHint", _CUmemLocation),
        ("flags", ctypes.c_uint),
    ]
```
**EN:** Introduces the `_CUmemcpyAttributes` class on top of `ctypes.Structure`. Core methods include its methods defined below.
**CN:** 这里定义 `_CUmemcpyAttributes` 类，其基类包括 `ctypes.Structure`。核心方法包括 下方定义的方法。

### Module constants / 模块常量
```python
_BATCH_MEMCPY_FUNC_TYPE = ctypes.CFUNCTYPE(
    ctypes.c_uint,  # CUresult / hipError_t
    ctypes.c_void_p,
    ctypes.c_void_p,
    ctypes.c_void_p,
    ctypes.c_size_t,
    ctypes.c_void_p,
    ctypes.c_void_p,
    ctypes.c_size_t,
    ctypes.c_void_p,
    ctypes.c_void_p,
)

# Resolved lazily on first use.
_batch_memcpy_fn: Any = None
```
**EN:** Defines module-level constants or aliases such as `_BATCH_MEMCPY_FUNC_TYPE`, `_batch_memcpy_fn`, which are reused by later definitions.
**CN:** 定义 `_BATCH_MEMCPY_FUNC_TYPE`, `_batch_memcpy_fn` 等模块级常量或别名，供后续定义复用。

### `_resolve_batch_memcpy` function / `_resolve_batch_memcpy` 函数
```python
def _resolve_batch_memcpy():
    """Resolve the platform batch-memcpy entry point (one-time).

    * CUDA: ``cuMemcpyBatchAsync`` via ``cuGetProcAddress`` (uses
      srcAccessOrder=STREAM via one attributes entry).
    * ROCm: ``hipMemcpyBatchAsync`` from libamdhip64 (ROCm 7.1+). ROCm
      7.2.1 or 7.2.2 rejects any call with ``numAttrs > 0``
      (see ROCm/clr @ rocm-7.2.1 hipamd/src/hip_memory.cpp:2819-2822), so
      we call with ``numAttrs=0``.

    Raises ``RuntimeError`` if the symbol is unavailable (older CUDA
    driver, ROCm < 7.1, unusual install). The connector requires the
    batch API.
    """
    if current_platform.is_rocm():
        try:
            lib = ctypes.CDLL("libamdhip64.so", mode=ctypes.RTLD_GLOBAL)
            fn = lib.hipMemcpyBatchAsync
        except (OSError, AttributeError) as e:
            raise RuntimeError(
                "hipMemcpyBatchAsync is unavailable in this ROCm install; "
                "SimpleCPUOffloadConnector requires ROCm 7.1+."
            ) from e
        fn.restype = ctypes.c_uint
        fn.argtypes = [
            ctypes.c_void_p,  # dsts
            ctypes.c_void_p,  # srcs
            ctypes.c_void_p,  # sizes
            ctypes.c_size_t,  # count
            ctypes.c_void_p,  # attrs
            ctypes.c_void_p,  # attrIdxs
            ctypes.c_size_t,  # numAttrs
            ctypes.c_void_p,  # failIdx
            ctypes.c_void_p,  # stream
        ]
        return fn

    from cuda.bindings import driver as drv

    err, ptr, _ = drv.cuGetProcAddress(b"cuMemcpyBatchAsync", 12080, 0)
    if err != drv.CUresult.CUDA_SUCCESS:
        raise RuntimeError(f"cuGetProcAddress(cuMemcpyBatchAsync) failed: {err}")
    return _BATCH_MEMCPY_FUNC_TYPE(ptr)
```
**EN:** This function implements `_resolve_batch_memcpy` within the module. The docstring frames it as: Resolve the platform batch-memcpy entry point (one-time). Key calls include `is_rocm`, `cuGetProcAddress`, `_BATCH_MEMCPY_FUNC_TYPE`, `RuntimeError`, `CDLL`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_resolve_batch_memcpy`，其作用域位于the module。 关键调用包括 `is_rocm`, `cuGetProcAddress`, `_BATCH_MEMCPY_FUNC_TYPE`, `RuntimeError`, `CDLL`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `BatchMemcpyParams` class / `BatchMemcpyParams` 类
```python
class BatchMemcpyParams(NamedTuple):
    src_bases: np.ndarray  # [num_layers] uint64 — data_ptr per layer
    dst_bases: np.ndarray  # [num_layers] uint64
    bpb: np.ndarray  # [num_layers] uint64 — bytes per block
    num_layers: int
    # CUDA only: one attributes entry with srcAccessOrder=ANY. Unused on
    # ROCm (7.2.1 or 7.2.2) because the current runtime rejects numAttrs > 0.
    attrs: _CUmemcpyAttributes
    attrs_idx: ctypes.c_size_t
    # NOTE: cuMemcpyBatchAsync_v2() removed fail_idx field, but we use
    # cuMemcpyBatchAsync() with fail_idx for backward compatibility
    fail_idx: ctypes.c_size_t
    stream_handle: int  # raw cudaStream_t / CUstream
```
**EN:** Introduces the `BatchMemcpyParams` class on top of `NamedTuple`. Core methods include its methods defined below.
**CN:** 这里定义 `BatchMemcpyParams` 类，其基类包括 `NamedTuple`。核心方法包括 下方定义的方法。

### `build_params` function / `build_params` 函数
```python
def build_params(
    src_caches: dict[str, torch.Tensor],
    dst_caches: dict[str, torch.Tensor],
    stream: torch.cuda.Stream,
) -> BatchMemcpyParams:
    global _batch_memcpy_fn
    if _batch_memcpy_fn is None:
        _batch_memcpy_fn = _resolve_batch_memcpy()

    assert list(src_caches.keys()) == list(dst_caches.keys())
    src_tensors = list(src_caches.values())
    dst_tensors = list(dst_caches.values())

    src_bases, dst_bases, bpb = [], [], []
    for s, d in zip(src_tensors, dst_tensors):
        s_bpb = s.stride(0) * s.element_size()
        assert s_bpb == d.stride(0) * d.element_size()
        src_bases.append(s.data_ptr())
        dst_bases.append(d.data_ptr())
        bpb.append(s_bpb)

    # ``srcAccessOrder=3`` == CU_MEMCPY_SRC_ACCESS_ORDER_ANY /
    # hipMemcpySrcAccessOrderAny. See
    # https://docs.nvidia.com/cuda/cuda-driver-api/group__CUDA__MEM.html#group__CUDA__MEM_1g6f1ff58e3065df3eb4b573dba77ad31f  # noqa: E501
    attrs = _CUmemcpyAttributes(srcAccessOrder=3)

    return BatchMemcpyParams(
        src_bases=np.array(src_bases, dtype=np.uint64),
        dst_bases=np.array(dst_bases, dtype=np.uint64),
        bpb=np.array(bpb, dtype=np.uint64),
        num_layers=len(src_tensors),
        attrs=attrs,
        attrs_idx=ctypes.c_size_t(0),
        fail_idx=ctypes.c_size_t(0),
        stream_handle=stream.cuda_stream,
    )
```
**EN:** This function builds derived structures within the module. Key calls include `list`, `zip`, `_CUmemcpyAttributes`, `BatchMemcpyParams`, `_resolve_batch_memcpy`, `values`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会构建派生结构，其作用域位于the module。 关键调用包括 `list`, `zip`, `_CUmemcpyAttributes`, `BatchMemcpyParams`, `_resolve_batch_memcpy`, `values`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `copy_blocks` function / `copy_blocks` 函数
```python
def copy_blocks(
    src_block_ids: list[int],
    dst_block_ids: list[int],
    params: BatchMemcpyParams,
) -> None:
    """Copy blocks via cuMemcpyBatchAsync / hipMemcpyBatchAsync."""
    n = len(src_block_ids)
    if n == 0:
        return

    src_ids = np.array(src_block_ids, dtype=np.uint64)
    dst_ids = np.array(dst_block_ids, dtype=np.uint64)

    src_all = (
        params.src_bases[:, None] + src_ids[None, :] * params.bpb[:, None]
    ).ravel()
    dst_all = (
        params.dst_bases[:, None] + dst_ids[None, :] * params.bpb[:, None]
    ).ravel()
    sz_all = np.repeat(params.bpb, n)
    total = n * params.num_layers

    # ROCm 7.2.1/7.2.2 rejects any call with numAttrs>0 (hipMemcpyBatchAsync
    # hipamd/src/hip_memory.cpp:2819-2822); CUDA uses one attrs entry so
    # srcAccessOrder is honored. attrs / attrsIdxs are ignored when
    # numAttrs==0, so we pass the same values from both paths.
    num_attrs = 0 if current_platform.is_rocm() else 1
    err = _batch_memcpy_fn(
        dst_all.ctypes.data,
        src_all.ctypes.data,
        sz_all.ctypes.data,
        total,
        ctypes.addressof(params.attrs),
        ctypes.byref(params.attrs_idx),
        num_attrs,
        ctypes.byref(params.fail_idx),
        params.stream_handle,
    )
    if err != 0:
        raise RuntimeError(
            f"batch memcpy failed: err={err} failIdx={params.fail_idx.value}"
        )
```
**EN:** This function implements `copy_blocks` within the module. The docstring frames it as: Copy blocks via cuMemcpyBatchAsync / hipMemcpyBatchAsync. Key calls include `len`, `array`, `ravel`, `repeat`, `_batch_memcpy_fn`, `is_rocm`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `copy_blocks`，其作用域位于the module。 关键调用包括 `len`, `array`, `ravel`, `repeat`, `_batch_memcpy_fn`, `is_rocm`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `pin_tensor`: top-level helper or orchestration entry point. / `pin_tensor`：顶层辅助函数或编排入口。
- `_CUmemLocation`: central class or interface in this module. / `_CUmemLocation`：本模块中的核心类或接口。
- `_CUmemcpyAttributes`: central class or interface in this module. / `_CUmemcpyAttributes`：本模块中的核心类或接口。
- `_resolve_batch_memcpy`: top-level helper or orchestration entry point. / `_resolve_batch_memcpy`：顶层辅助函数或编排入口。
- `BatchMemcpyParams`: central class or interface in this module. / `BatchMemcpyParams`：本模块中的核心类或接口。
- `build_params`: top-level helper or orchestration entry point. / `build_params`：顶层辅助函数或编排入口。
- `copy_blocks`: top-level helper or orchestration entry point. / `copy_blocks`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `ctypes`, `typing`
- External / 外部依赖: `numpy`, `torch`, `cuda`
- Internal vLLM / 内部依赖: `vllm.logger`, `vllm.platforms`
