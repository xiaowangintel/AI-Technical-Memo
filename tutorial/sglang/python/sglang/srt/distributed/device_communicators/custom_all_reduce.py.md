# custom_all_reduce.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/distributed/device_communicators/custom_all_reduce.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed execution and communication part of the SRT runtime and implements logic centered on `custom_all_reduce`. It exposes primary entry points such as `CustomAllreduce`, `dispatch_custom_allreduce`, `_use_amd_deterministic_impl`. / 该模块属于 SRT 运行时的分布式执行与通信部分，主要实现围绕 `custom_all_reduce` 的逻辑。 它对外提供的主要入口包括 `CustomAllreduce`, `dispatch_custom_allreduce`, `_use_amd_deterministic_impl`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-37: Module imports, constants, and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/v0.6.4.post1/vllm/distributed/device_communicators/custom_all_reduce.py

import ctypes
import logging
from contextlib import contextmanager
from functools import partial
from typing import Any, List, Optional, Union

import torch
import torch.distributed as dist
from torch.distributed import ProcessGroup

import sglang.srt.distributed.device_communicators.custom_all_reduce_ops as ops
from sglang.srt.compilation.piecewise_context_manager import is_in_piecewise_cuda_graph
from sglang.srt.distributed.device_communicators.cuda_wrapper import CudaRTLibrary
from sglang.srt.distributed.device_communicators.custom_all_reduce_utils import (
    can_use_custom_all_reduce_with_nvlink,
    is_weak_contiguous,
)
from sglang.srt.environ import envs
from sglang.srt.utils import (
    get_bool_env_var,
    is_cuda,
    is_hip,
    is_musa,
    log_info_on_rank0,
)

_is_cuda = is_cuda()
_is_hip = is_hip()
_is_musa = is_musa()

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；协调分布式通信。

### Lines 38-48: Class CustomAllreduce
```python
class CustomAllreduce:
    _SUPPORTED_WORLD_SIZES = [2, 4, 6, 8]
    _MAX_CAR_SIZE = 8192 * 1024
    if _is_hip:
        # crossover is at 16MB buffer size for ROCm
        _MAX_CAR_SIZE = 2 * 8192 * 1024
    if _is_musa:
        # crossover is at 128MB buffer size for MUSA
        _MAX_CAR_SIZE = 16 * 8196 * 1024

    # max_size: max supported allreduce size
```
**EN:** This range introduces `CustomAllreduce` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `CustomAllreduce`，并定义其后续方法依赖的结构或元数据。

### Lines 49-108: Method CustomAllreduce.__init__ (part 1/2)
```python
    def __init__(
        self,
        group: ProcessGroup,
        device: Union[int, str, torch.device],
        max_size=_MAX_CAR_SIZE,
    ) -> None:
        """
        Args:
            group: the process group to work on. If None, it will use the
                default process group.
            device: the device to bind the CustomAllreduce to. If None,
                it will be bind to f"cuda:{local_rank}".
        It is the caller's responsibility to make sure each communicator
        is bind to a unique device, and all communicators in this group
        are in the same node.
        """
        self._IS_CAPTURING = False
        self.disabled = True  # This can be modified in-place by context manager in piecewise cuda graph runner
        self.original_disabled = True  # To store the original state
        self.use_amd_deterministic_impl = _use_amd_deterministic_impl()

        if not ops.IS_CUSTOM_AR_AVAILABLE:
            # disable because of missing custom allreduce library
            # e.g. in a non-cuda environment
            return

        rank = dist.get_rank(group=group)
        world_size = dist.get_world_size(group=group)

        if isinstance(device, int):
            device = torch.device(f"cuda:{device}")
        elif isinstance(device, str):
            device = torch.device(device)
        # now `device` is a `torch.device` object
        assert isinstance(device, torch.device)
        self.device = device
        full_nvlink = can_use_custom_all_reduce_with_nvlink(
            group=group,
            device=device,
            supported_world_size=self._SUPPORTED_WORLD_SIZES,
            cls_name="CustomAllreduce",
        )
        if full_nvlink is None:
            return  # fail to get nvlink status

        self.group = group
        self.max_size = max_size
        self.rank = rank
        self.world_size = world_size
        self.full_nvlink = full_nvlink

        if not _is_hip:
            # Buffers memory are owned by this Python class and passed to C++.
            # Meta data composes of two parts: meta data for synchronization and a
            # temporary buffer for storing intermediate allreduce results.
            self.meta_ptrs = self.create_shared_buffer(
                ops.meta_size() + max_size, group=group
            )
            # This is a pre-registered IPC buffer. In eager mode, input tensors
            # are first copied into this buffer before allreduce is performed
```
**EN:** This callable implements `CustomAllreduce.__init__`. It takes `group`, `device`, `max_size` and mainly initializes instance state and defaults. The docstring states: "Args: group: the process group to work on." This chunk is part 1 of 2 for the same logical block. In this range it performs defensive checks on invalid state; coordinates distributed communication; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CustomAllreduce.__init__`。它接收 `group`, `device`, `max_size`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信；管理图捕获或回放逻辑。

### Lines 109-142: Method CustomAllreduce.__init__ (part 2/2)
```python
            self.buffer_ptrs = self.create_shared_buffer(max_size, group=group)
            # This is a buffer for storing the tuples of pointers pointing to
            # IPC buffers from all ranks. Each registered tuple has size of
            # 8*world_size bytes where world_size is at most 8. Allocating 8MB
            # is enough for 131072 such tuples. The largest model I've seen only
            # needs less than 10000 of registered tuples.
            self.rank_data = torch.empty(
                max_size, dtype=torch.uint8, device=self.device
            )
            self._ptr = ops.init_custom_ar(
                self.meta_ptrs, self.rank_data, rank, self.full_nvlink
            )
            ops.register_buffer(self._ptr, self.buffer_ptrs)
        else:
            # meta data buffers need to be "uncached" for signal on MI200
            self.meta = ops.allocate_meta_buffer(ops.meta_size() + max_size)
            self.buffer = torch.empty(max_size, dtype=torch.uint8, device=self.device)
            handle = ops.get_meta_buffer_ipc_handle(self.meta)
            shard_data = (
                bytes(handle),  # ipc handle to base ptr
                0,  # offset of base ptr
            )
            handles, offsets = self._gather_ipc_meta(shard_data)
            self.rank_data = torch.empty(
                max_size, dtype=torch.uint8, device=self.device
            )
            self._ptr = ops.init_custom_ar(
                self.meta, self.rank_data, handles, offsets, rank, self.full_nvlink
            )
            self.register_buffer(self.buffer)

        self.disabled = False
        self.original_disabled = False  # Ensure original_disabled == disabled
        self.tms_cudagraph = envs.SGLANG_MEMORY_SAVER_CUDA_GRAPH.get()
```
**EN:** This callable implements `CustomAllreduce.__init__`. It takes `group`, `device`, `max_size` and mainly initializes instance state and defaults. The docstring states: "Args: group: the process group to work on." This chunk is part 2 of 2 for the same logical block. In this range it sets up imports and shared symbols; reads environment-driven configuration; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CustomAllreduce.__init__`。它接收 `group`, `device`, `max_size`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；读取环境变量驱动的配置；管理图捕获或回放逻辑。

### Lines 144-169: Method CustomAllreduce.create_shared_buffer
```python
    @staticmethod
    def create_shared_buffer(
        size_in_bytes: int, group: Optional[ProcessGroup] = None
    ) -> List[int]:
        """
        Creates a shared buffer and returns a list of pointers
        representing the buffer on all processes in the group.
        """
        lib = CudaRTLibrary()
        pointer = lib.cudaMalloc(size_in_bytes)
        if _is_musa:
            lib.cudaMemset(pointer, 0, size_in_bytes)
        handle = lib.cudaIpcGetMemHandle(pointer)
        world_size = dist.get_world_size(group=group)
        rank = dist.get_rank(group=group)
        handles = [None] * world_size
        dist.all_gather_object(handles, handle, group=group)

        pointers: List[int] = []
        for i, h in enumerate(handles):
            if i == rank:
                pointers.append(pointer.value)  # type: ignore
            else:
                pointers.append(lib.cudaIpcOpenMemHandle(h).value)  # type: ignore

        return pointers
```
**EN:** This callable implements `CustomAllreduce.create_shared_buffer`. It takes `size_in_bytes`, `group` and mainly constructs new objects or contexts. The docstring states: "Creates a shared buffer and returns a list of pointers representing the buffer on all processes in the group." In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `CustomAllreduce.create_shared_buffer`。它接收 `size_in_bytes`, `group`，主要用于构造新的对象或上下文。 在这一范围内，它会协调分布式通信。

### Lines 171-177: Method CustomAllreduce.free_shared_buffer
```python
    @staticmethod
    def free_shared_buffer(
        pointers: List[int], group: Optional[ProcessGroup] = None
    ) -> None:
        rank = dist.get_rank(group=group)
        lib = CudaRTLibrary()
        lib.cudaFree(ctypes.c_void_p(pointers[rank]))
```
**EN:** This callable implements `CustomAllreduce.free_shared_buffer`. It takes `pointers`, `group` and mainly implements free shared buffer. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `CustomAllreduce.free_shared_buffer`。它接收 `pointers`, `group`，主要用于实现 free shared buffer 相关逻辑。 在这一范围内，它会协调分布式通信。

### Lines 179-192: Method CustomAllreduce.capture
```python
    @contextmanager
    def capture(self):
        """
        The main responsibility of this context manager is the
        `register_graph_buffers` call at the end of the context.
        It records all the buffer addresses used in the CUDA graph.
        """
        try:
            self._IS_CAPTURING = True
            yield
        finally:
            self._IS_CAPTURING = False
            if not self.disabled:
                self.register_graph_buffers()
```
**EN:** This callable implements `CustomAllreduce.capture` and mainly implements capture. The docstring states: "The main responsibility of this context manager is the `register_graph_buffers` call at the end of the context." In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CustomAllreduce.capture`，主要用于实现 capture 相关逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 194-202: Method CustomAllreduce._get_ipc_meta
```python
    def _get_ipc_meta(self, inp: torch.Tensor):
        # _share_cuda_() doesn't accept meta buffer not allocated from
        # PyTorch cache allocator, use direct HIP call to get IPC handle
        handle = ops.get_meta_buffer_ipc_handle(inp)
        shard_data = (
            bytes(handle),  # ipc handle to base ptr
            0,  # offset of base ptr
        )
        return self._gather_ipc_meta(shard_data)
```
**EN:** This callable implements `CustomAllreduce._get_ipc_meta`. It takes `inp` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `CustomAllreduce._get_ipc_meta`。它接收 `inp`，主要用于获取某个值或派生视图。

### Lines 204-226: Method CustomAllreduce._gather_ipc_meta
```python
    def _gather_ipc_meta(self, shard_data):
        # Note: don't use `[[None]] * self.world_size` here
        # because it will create a list of the same reference
        all_data: List[Optional[Any]] = [[None] for i in range(self.world_size)]
        all_data[self.rank][0] = shard_data

        ranks = dist.get_process_group_ranks(group=self.group)
        ranks.sort()
        for i, rank in enumerate(ranks):
            dist.broadcast_object_list(
                all_data[i], src=rank, group=self.group, device="cpu"
            )

        # we cannot directly use `dist.all_gather_object` here
        # because it is incompatible with `gloo` backend under inference mode.
        # see https://github.com/pytorch/pytorch/issues/126032 for details.

        handles = []
        offsets = []
        for i in range(len(all_data)):
            handles.append(all_data[i][0][0])  # type: ignore
            offsets.append(all_data[i][0][1])  # type: ignore
        return handles, offsets
```
**EN:** This callable implements `CustomAllreduce._gather_ipc_meta`. It takes `shard_data` and mainly implements gather ipc meta. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `CustomAllreduce._gather_ipc_meta`。它接收 `shard_data`，主要用于实现 gather ipc meta 相关逻辑。 在这一范围内，它会协调分布式通信。

### Lines 228-230: Method CustomAllreduce.register_buffer
```python
    def register_buffer(self, inp: torch.Tensor):
        handles, offsets = self._get_ipc_meta(inp)
        ops.register_buffer(self._ptr, inp, handles, offsets)
```
**EN:** This callable implements `CustomAllreduce.register_buffer`. It takes `inp` and mainly registers hooks, handlers, or operators.
**CN:** 这一可调用对象实现了 `CustomAllreduce.register_buffer`。它接收 `inp`，主要用于注册钩子、处理器或算子。

### Lines 232-256: Method CustomAllreduce.register_graph_buffers
```python
    def register_graph_buffers(self):
        if _is_hip:
            handle, offset = ops.get_graph_buffer_ipc_meta(self._ptr)
            handles, offsets = self._gather_ipc_meta((bytes(handle), offset))
            log_info_on_rank0(logger, f"Registering {len(offset)} cuda graph addresses")
            ops.register_graph_buffers(self._ptr, handles, offsets)
        else:
            handle, offset = ops.get_graph_buffer_ipc_meta(self._ptr)
            log_info_on_rank0(logger, f"Registering {len(offset)} cuda graph addresses")
            # We cannot directly use `dist.all_gather_object` here
            # because it is incompatible with `gloo` backend under inference mode.
            # see https://github.com/pytorch/pytorch/issues/126032 for details.
            all_data = [
                [None, None] for _ in range(dist.get_world_size(group=self.group))
            ]
            all_data[self.rank] = [handle, offset]
            ranks = sorted(dist.get_process_group_ranks(group=self.group))
            for i, rank in enumerate(ranks):
                dist.broadcast_object_list(
                    all_data[i], src=rank, group=self.group, device="cpu"
                )
            # Unpack list of tuples to tuple of lists.
            handles = [d[0] for d in all_data]  # type: ignore
            offsets = [d[1] for d in all_data]  # type: ignore
            ops.register_graph_buffers(self._ptr, handles, offsets)
```
**EN:** This callable implements `CustomAllreduce.register_graph_buffers` and mainly registers hooks, handlers, or operators. In this range it coordinates distributed communication; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CustomAllreduce.register_graph_buffers`，主要用于注册钩子、处理器或算子。 在这一范围内，它会协调分布式通信；管理图捕获或回放逻辑。

### Lines 258-281: Method CustomAllreduce.should_custom_ar
```python
    def should_custom_ar(self, inp: torch.Tensor):
        if self.disabled:
            return False
        inp_size = inp.numel() * inp.element_size()
        # custom allreduce requires input byte size to be multiples of 16
        if inp_size % 16 != 0:
            return False
        if not is_weak_contiguous(inp):
            return False
        # for 4 or more non NVLink-capable GPUs, custom allreduce provides
        # little performance improvement over NCCL.
        if not _is_hip:
            if self.world_size == 2 or self.full_nvlink:
                return inp_size <= self.max_size
            return False

        if _is_hip:
            if self.use_amd_deterministic_impl:
                return True
            if self.full_nvlink:
                return inp_size <= self.max_size
            return False

        return False
```
**EN:** This callable implements `CustomAllreduce.should_custom_ar`. It takes `inp` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `CustomAllreduce.should_custom_ar`。它接收 `inp`，主要用于将数据转换为另一种表示。

### Lines 283-305: Method CustomAllreduce._all_reduce_impl
```python
    def _all_reduce_impl(self, inp: torch.Tensor, registered: bool):
        out = torch.empty_like(inp)
        if not _is_hip:  # CUDA-like
            if registered:
                ops.all_reduce(self._ptr, inp, out, 0, 0)
            else:
                ops.all_reduce(
                    self._ptr, inp, out, self.buffer_ptrs[self.rank], self.max_size
                )
        elif self.use_amd_deterministic_impl:
            inp_size = inp.numel() * inp.element_size()
            if inp_size < self.max_size:
                reg_buffer = self.buffer.view(inp.dtype)[: inp.numel()]
                ops.deterministic_all_reduce_unreg(self._ptr, inp, reg_buffer, out)
            else:
                self.register_buffer(inp)
                ops.deterministic_all_reduce_reg(self._ptr, inp, out)
        else:  # normal AMD ROCm path
            if registered:
                ops.all_reduce_reg(self._ptr, inp, out)
            else:
                ops.all_reduce_unreg(self._ptr, inp, self.buffer, out)
        return out
```
**EN:** This callable implements `CustomAllreduce._all_reduce_impl`. It takes `inp`, `registered` and mainly reduces or aggregates values. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `CustomAllreduce._all_reduce_impl`。它接收 `inp`, `registered`，主要用于归约或聚合数值。 在这一范围内，它会协调分布式通信。

### Lines 307-327: Method CustomAllreduce.custom_all_reduce
```python
    def custom_all_reduce(self, input: torch.Tensor) -> Optional[torch.Tensor]:
        """The main allreduce API that provides support for cuda graph."""
        # When custom allreduce is disabled, this will be None.
        if self.disabled or not self.should_custom_ar(input):
            return None
        if self._IS_CAPTURING:
            if torch.cuda.is_current_stream_capturing():
                return self._all_reduce_impl(input, registered=not self.tms_cudagraph)
            else:
                # Could be warmup OR piecewise cuda graph split op execution.
                # In piecewise cuda graph, split ops run eagerly outside the graph
                # but _IS_CAPTURING is still True. We need to do real all-reduce.
                if is_in_piecewise_cuda_graph():
                    # Split op execution - do real all-reduce
                    return self._all_reduce_impl(input, registered=False)
                else:
                    # True warmup - mimic the allocation pattern since custom
                    # allreduce is out-of-place.
                    return torch.zeros_like(input)
        else:
            return self._all_reduce_impl(input, registered=False)
```
**EN:** This callable implements `CustomAllreduce.custom_all_reduce`. It takes `input` and mainly converts data into another representation. The docstring states: "The main allreduce API that provides support for cuda graph." In this range it coordinates distributed communication; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CustomAllreduce.custom_all_reduce`。它接收 `input`，主要用于将数据转换为另一种表示。 在这一范围内，它会协调分布式通信；管理图捕获或回放逻辑。

### Lines 329-335: Method CustomAllreduce.close
```python
    def close(self):
        if not self.disabled and self._ptr:
            ops.dispose(self._ptr)
            if _is_cuda:
                self.free_shared_buffer(self.meta_ptrs)
                self.free_shared_buffer(self.buffer_ptrs)
            self._ptr = 0
```
**EN:** This callable implements `CustomAllreduce.close` and mainly implements close.
**CN:** 这一可调用对象实现了 `CustomAllreduce.close`，主要用于实现 close 相关逻辑。

### Lines 337-338: Method CustomAllreduce.__del__
```python
    def __del__(self):
        self.close()
```
**EN:** This callable implements `CustomAllreduce.__del__` and mainly implements del.
**CN:** 这一可调用对象实现了 `CustomAllreduce.__del__`，主要用于实现 del 相关逻辑。

### Lines 341-400: Function dispatch_custom_allreduce (part 1/2)
```python
def dispatch_custom_allreduce():
    """Return the CustomAllreduce class to use (aiter on ROCm if enabled).

    On AMD with 1-stage AR enabled, use sglang's CustomAllreduce.
    Otherwise use AiterCustomAllreduce if available.

    On CUDA, the JIT-compiled v2 implementation is used by default.
    Set SGLANG_OPT_USE_CUSTOM_ALL_REDUCE_V2=0 to fall back to the legacy CustomAllreduce.
    Note: ``ServerArgs._handle_environment_variables`` forces this env to "0" when
    ``nnodes > 1`` since custom AR is intra-node only.
    """
    if _is_cuda and envs.SGLANG_OPT_USE_CUSTOM_ALL_REDUCE_V2.get():
        from .custom_all_reduce_v2 import CustomAllReduceV2

        logger.debug("[AR] Using CustomAllReduceV2 (JIT-compiled)")
        return CustomAllReduceV2

    if _is_cuda or _is_musa:
        return CustomAllreduce

    assert _is_hip

    if envs.SGLANG_USE_1STAGE_ALLREDUCE.is_set():
        if envs.SGLANG_USE_1STAGE_ALLREDUCE.get():
            logger.debug(
                "[AR] All-reduce: 1-stage kernel (SGLANG_USE_1STAGE_ALLREDUCE=1)"
            )
        else:
            logger.debug("[AR] All-reduce: default (SGLANG_USE_1STAGE_ALLREDUCE=0)")
    elif envs.SGLANG_ENABLE_DETERMINISTIC_INFERENCE.get():
        logger.debug(
            "[AR] All-reduce: 1-stage kernel (deterministic inference enabled)"
        )
    else:
        logger.debug("[AR] All-reduce: default")

    # On AMD with 1-stage AR, use sglang's CustomAllreduce
    # (AiterCustomAllreduce doesn't have deterministic_all_reduce method)
    if _use_amd_deterministic_impl():
        return CustomAllreduce

    if get_bool_env_var("SGLANG_USE_AITER_AR", default="true"):
        try:
            from aiter.dist.device_communicators.custom_all_reduce import (
                CustomAllreduce as AiterCustomAllreduce,
            )

            logger.info("[AR] Using AiterCustomAllreduce (AMD default)")
            tms_cudagraph = envs.SGLANG_MEMORY_SAVER_CUDA_GRAPH.get()
            return partial(
                AiterCustomAllreduce,
                enable_register_for_capturing=not tms_cudagraph,
            )
        except ImportError as e:
            logger.warning(
                "[AR] Aiter custom all-reduce not available; "
                "falling back to sglang CustomAllreduce. Details: %s",
                e,
            )
            return CustomAllreduce
```
**EN:** This callable implements `dispatch_custom_allreduce` and mainly converts data into another representation. The docstring states: "Return the CustomAllreduce class to use (aiter on ROCm if enabled)." This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `dispatch_custom_allreduce`，主要用于将数据转换为另一种表示。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；输出日志以便诊断。

### Lines 401-402: Function dispatch_custom_allreduce (part 2/2)
```python

    return CustomAllreduce
```
**EN:** This callable implements `dispatch_custom_allreduce` and mainly converts data into another representation. The docstring states: "Return the CustomAllreduce class to use (aiter on ROCm if enabled)." This chunk is part 2 of 2 for the same logical block.
**CN:** 这一可调用对象实现了 `dispatch_custom_allreduce`，主要用于将数据转换为另一种表示。 该片段是同一逻辑块的第 2/2 部分。

### Lines 405-411: Function _use_amd_deterministic_impl
```python
def _use_amd_deterministic_impl() -> bool:
    if not _is_hip:  # CUDA is always deterministic
        return False
    if envs.SGLANG_USE_1STAGE_ALLREDUCE.is_set():
        return envs.SGLANG_USE_1STAGE_ALLREDUCE.get()
    else:
        return envs.SGLANG_ENABLE_DETERMINISTIC_INFERENCE.get()
```
**EN:** This callable implements `_use_amd_deterministic_impl` and mainly implements use amd deterministic impl. In this range it reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `_use_amd_deterministic_impl`，主要用于实现 use amd deterministic impl 相关逻辑。 在这一范围内，它会读取环境变量驱动的配置。

## Key Concepts / 关键概念
- `CustomAllreduce`: core class or state container / 核心类或状态容器
- `dispatch_custom_allreduce`: converts data into another representation / 将数据转换为另一种表示
- `_use_amd_deterministic_impl`: implements use amd deterministic impl / 实现 use amd deterministic impl 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `ctypes`, `logging`, `contextlib`, `functools`, `typing`
- **Third-party / 第三方**: `torch`, `torch.distributed`, `aiter.dist.device_communicators.custom_all_reduce`
- **Internal modules / 内部模块**: `sglang.srt.distributed.device_communicators.custom_all_reduce_ops`, `sglang.srt.compilation.piecewise_context_manager`, `sglang.srt.distributed.device_communicators.cuda_wrapper`, `sglang.srt.distributed.device_communicators.custom_all_reduce_utils`, `sglang.srt.environ`, `sglang.srt.utils`, `.custom_all_reduce_v2`
