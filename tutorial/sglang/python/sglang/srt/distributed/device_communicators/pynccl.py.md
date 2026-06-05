# pynccl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/distributed/device_communicators/pynccl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed execution and communication part of the SRT runtime and implements logic centered on `pynccl`. It exposes primary entry points such as `PyNcclCommunicator`. / 该模块属于 SRT 运行时的分布式执行与通信部分，主要实现围绕 `pynccl` 的逻辑。 它对外提供的主要入口包括 `PyNcclCommunicator`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Module imports, constants, and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/v0.6.4.post1/vllm/distributed/device_communicators/pynccl.py

import logging
from contextlib import contextmanager
from typing import Optional, Union

# ===================== import region =====================
import torch
import torch.distributed as dist
from torch.distributed import ProcessGroup, ReduceOp

from sglang.srt.distributed.device_communicators.pynccl_wrapper import (
    NCCLLibrary,
    buffer_type,
    cudaStream_t,
    ncclComm_t,
    ncclDataTypeEnum,
    ncclRedOpTypeEnum,
    ncclUniqueId,
)
from sglang.srt.distributed.utils import StatelessProcessGroup
from sglang.srt.utils.common import get_current_device_stream_fast

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；协调分布式通信。

### Lines 29-30: Class PyNcclCommunicator
```python
class PyNcclCommunicator:

```
**EN:** This range introduces `PyNcclCommunicator` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `PyNcclCommunicator`，并定义其后续方法依赖的结构或元数据。

### Lines 31-90: Method PyNcclCommunicator.__init__ (part 1/2)
```python
    def __init__(
        self,
        group: Union[ProcessGroup, StatelessProcessGroup],
        device: Union[int, str, torch.device],
        library_path: Optional[str] = None,
    ):
        """
        Args:
            group: the process group to work on. If None, it will use the
                default process group.
            device: the device to bind the PyNcclCommunicator to. If None,
                it will be bind to f"cuda:{local_rank}".
            library_path: the path to the NCCL library. If None, it will
                use the default library path.
        It is the caller's responsibility to make sure each communicator
        is bind to a unique device.
        """
        if not isinstance(group, StatelessProcessGroup):
            assert dist.is_initialized()
            assert (
                dist.get_backend(group) != dist.Backend.NCCL
            ), "PyNcclCommunicator should be attached to a non-NCCL group."
            # note: this rank is the rank in the group
            self.rank = dist.get_rank(group)
            self.world_size = dist.get_world_size(group)
        else:
            self.rank = group.rank
            self.world_size = group.world_size

        self.group = group

        # if world_size == 1, no need to create communicator
        if self.world_size == 1:
            self.available = False
            self.disabled = True
            return
        try:
            self.nccl = NCCLLibrary(library_path)
        except Exception:
            # disable because of missing NCCL library
            # e.g. in a non-GPU environment
            self.available = False
            self.disabled = True
            return

        self.available = True
        self.disabled = False

        self.nccl_version = self.nccl.ncclGetRawVersion()
        if self.rank == 0:
            logger.info("sglang is using nccl==%s", self.nccl.ncclGetVersion())

        if self.rank == 0:
            # get the unique id from NCCL
            self.unique_id = self.nccl.ncclGetUniqueId()
        else:
            # construct an empty unique id
            self.unique_id = ncclUniqueId()

        if not isinstance(group, StatelessProcessGroup):
```
**EN:** This callable implements `PyNcclCommunicator.__init__`. It takes `group`, `device`, `library_path` and mainly initializes instance state and defaults. The docstring states: "Args: group: the process group to work on." This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `PyNcclCommunicator.__init__`。它接收 `group`, `device`, `library_path`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；输出日志以便诊断。

### Lines 91-126: Method PyNcclCommunicator.__init__ (part 2/2)
```python
            tensor = torch.ByteTensor(list(self.unique_id.internal))
            ranks = dist.get_process_group_ranks(group)
            # arg `src` in `broadcast` is the global rank
            dist.broadcast(tensor, src=ranks[0], group=group)
            byte_list = tensor.tolist()
            for i, byte in enumerate(byte_list):
                self.unique_id.internal[i] = byte
        else:
            self.unique_id = group.broadcast_obj(self.unique_id, src=0)
        if isinstance(device, int):
            device = torch.device(f"cuda:{device}")
        elif isinstance(device, str):
            device = torch.device(device)
        # now `device` is a `torch.device` object
        assert isinstance(device, torch.device)
        self.device = device
        # nccl communicator and stream will use this device
        # `torch.cuda.device` is a context manager that changes the
        # current cuda device to the specified one
        with torch.cuda.device(device):
            self.comm: ncclComm_t = self.nccl.ncclCommInitRank(
                self.world_size, self.unique_id, self.rank
            )
            warmup_stream = torch.cuda.Stream()

            # A small all_reduce for warmup.
            with torch.cuda.stream(warmup_stream):
                data = torch.zeros(1, device=device)
                self.all_reduce(data)
            warmup_stream.synchronize()
            del data

        # by default it is disabled, e.g. in profiling models and prefill phase.
        # to use it, use under `with obj.change_state(enable=True)`, usually
        # when we are using CUDA graph.
        self.disabled = True
```
**EN:** This callable implements `PyNcclCommunicator.__init__`. It takes `group`, `device`, `library_path` and mainly initializes instance state and defaults. The docstring states: "Args: group: the process group to work on." This chunk is part 2 of 2 for the same logical block. In this range it performs defensive checks on invalid state; coordinates distributed communication; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `PyNcclCommunicator.__init__`。它接收 `group`, `device`, `library_path`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信；管理图捕获或回放逻辑。

### Lines 128-130: Method PyNcclCommunicator._resolve_stream
```python
    def _resolve_stream(self) -> torch.cuda.Stream:
        """Return the current device stream used for NCCL calls."""
        return get_current_device_stream_fast()
```
**EN:** This callable implements `PyNcclCommunicator._resolve_stream` and mainly implements resolve stream. The docstring states: "Return the current device stream used for NCCL calls."
**CN:** 这一可调用对象实现了 `PyNcclCommunicator._resolve_stream`，主要用于实现 resolve stream 相关逻辑。

### Lines 132-151: Method PyNcclCommunicator.all_reduce
```python
    def all_reduce(self, tensor: torch.Tensor, op: ReduceOp = ReduceOp.SUM):
        if self.disabled:
            return
        # nccl communicator created on a specific device
        # will only work on tensors on the same device
        # otherwise it will cause "illegal memory access"
        assert tensor.device == self.device, (
            f"this nccl communicator is created to work on {self.device}, "
            f"but the input tensor is on {tensor.device}"
        )
        stream = self._resolve_stream()
        self.nccl.ncclAllReduce(
            buffer_type(tensor.data_ptr()),
            buffer_type(tensor.data_ptr()),
            tensor.numel(),
            ncclDataTypeEnum.from_torch(tensor.dtype),
            ncclRedOpTypeEnum.from_torch(op),
            self.comm,
            cudaStream_t(stream.cuda_stream),
        )
```
**EN:** This callable implements `PyNcclCommunicator.all_reduce`. It takes `tensor`, `op` and mainly reduces or aggregates values. In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `PyNcclCommunicator.all_reduce`。它接收 `tensor`, `op`，主要用于归约或聚合数值。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 153-179: Method PyNcclCommunicator.outplace_all_reduce
```python
    def outplace_all_reduce(
        self,
        in_tensor: torch.Tensor,
        out_tensor: Optional[torch.Tensor] = None,
        op: ReduceOp = ReduceOp.SUM,
    ) -> Optional[torch.Tensor]:
        if self.disabled:
            return None
        assert in_tensor.device == self.device, (
            f"this nccl communicator is created to work on {self.device}, "
            f"but the input tensor is on {in_tensor.device}"
        )

        if out_tensor is None:
            out_tensor = torch.empty_like(in_tensor)

        stream = self._resolve_stream()
        self.nccl.ncclAllReduce(
            buffer_type(in_tensor.data_ptr()),  # sendbuff
            buffer_type(out_tensor.data_ptr()),  # recvbuff - DIFFERENT pointer
            in_tensor.numel(),
            ncclDataTypeEnum.from_torch(in_tensor.dtype),
            ncclRedOpTypeEnum.from_torch(op),
            self.comm,
            cudaStream_t(stream.cuda_stream),
        )
        return out_tensor
```
**EN:** This callable implements `PyNcclCommunicator.outplace_all_reduce`. It takes `in_tensor`, `out_tensor`, `op` and mainly reduces or aggregates values. In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `PyNcclCommunicator.outplace_all_reduce`。它接收 `in_tensor`, `out_tensor`, `op`，主要用于归约或聚合数值。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 181-223: Method PyNcclCommunicator.all_gather
```python
    def all_gather(
        self,
        output_tensor: torch.Tensor,
        input_tensor: torch.Tensor,
        sizes: Optional[list[int]] = None,
    ):
        if self.disabled:
            return
        # nccl communicator created on a specific device
        # will only work on tensors on the same device
        # otherwise it will cause "illegal memory access"
        assert input_tensor.device == self.device, (
            f"this nccl communicator is created to work on {self.device}, "
            f"but the input tensor is on {input_tensor.device}"
        )
        stream = self._resolve_stream()

        if sizes is not None:
            split_offset = 0

            self.nccl.ncclGroupStart()
            for root, split_size in enumerate(sizes):
                dst_slice = output_tensor[split_offset : split_offset + split_size]
                self.nccl.ncclBroadcast(
                    buffer_type(input_tensor.data_ptr()),
                    buffer_type(dst_slice.data_ptr()),
                    dst_slice.numel(),
                    ncclDataTypeEnum.from_torch(input_tensor.dtype),
                    root,
                    self.comm,
                    cudaStream_t(stream.cuda_stream),
                )
                split_offset += split_size
            self.nccl.ncclGroupEnd()
        else:
            self.nccl.ncclAllGather(
                buffer_type(input_tensor.data_ptr()),
                buffer_type(output_tensor.data_ptr()),
                input_tensor.numel(),
                ncclDataTypeEnum.from_torch(input_tensor.dtype),
                self.comm,
                cudaStream_t(stream.cuda_stream),
            )
```
**EN:** This callable implements `PyNcclCommunicator.all_gather`. It takes `output_tensor`, `input_tensor`, `sizes` and mainly implements all gather. In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `PyNcclCommunicator.all_gather`。它接收 `output_tensor`, `input_tensor`, `sizes`，主要用于实现 all gather 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 225-250: Method PyNcclCommunicator.cp_all_gather_into_tensor
```python
    def cp_all_gather_into_tensor(
        self,
        output_tensor: torch.Tensor,
        input_tensor: torch.Tensor,
        stream: torch.cuda.Stream,
        sizes: Optional[list[int]] = None,
    ):
        """
        Currently, it is mainly used in context parallelism,
        primarily leveraging pynccl to implement non-blocking allgather communication.
        """
        # nccl communicator created on a specific device
        # will only work on tensors on the same device
        # otherwise it will cause "illegal memory access"
        assert input_tensor.device == self.device, (
            f"this nccl communicator is created to work on {self.device}, "
            f"but the input tensor is on {input_tensor.device}"
        )
        self.nccl.ncclAllGather(
            buffer_type(input_tensor.data_ptr()),
            buffer_type(output_tensor.data_ptr()),
            input_tensor.numel(),
            ncclDataTypeEnum.from_torch(input_tensor.dtype),
            self.comm,
            cudaStream_t(stream.cuda_stream),
        )
```
**EN:** This callable implements `PyNcclCommunicator.cp_all_gather_into_tensor`. It takes `output_tensor`, `input_tensor`, `stream`, `sizes` and mainly converts data into another representation. The docstring states: "Currently, it is mainly used in context parallelism, primarily leveraging pynccl to implement non-blocking allgather communication." In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `PyNcclCommunicator.cp_all_gather_into_tensor`。它接收 `output_tensor`, `input_tensor`, `stream`, `sizes`，主要用于将数据转换为另一种表示。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 252-297: Method PyNcclCommunicator.reduce_scatter
```python
    def reduce_scatter(
        self,
        output_tensor: torch.Tensor,
        input_tensor: torch.Tensor,
        op: ReduceOp = ReduceOp.SUM,
        sizes: Optional[list[int]] = None,
    ):
        if self.disabled:
            return
        # nccl communicator created on a specific device
        # will only work on tensors on the same device
        # otherwise it will cause "illegal memory access"
        assert input_tensor.device == self.device, (
            f"this nccl communicator is created to work on {self.device}, "
            f"but the input tensor is on {input_tensor.device}"
        )
        stream = self._resolve_stream()

        if sizes is not None:
            split_offset = 0
            self.nccl.ncclGroupStart()
            for root, split_size in enumerate(sizes):
                chunk = input_tensor[split_offset : split_offset + split_size, ...]

                self.nccl.ncclReduce(
                    buffer_type(chunk.data_ptr()),
                    buffer_type(output_tensor.data_ptr()),
                    chunk.numel(),
                    ncclDataTypeEnum.from_torch(input_tensor.dtype),
                    ncclRedOpTypeEnum.from_torch(op),
                    root,
                    self.comm,
                    cudaStream_t(stream.cuda_stream),
                )
                split_offset += split_size
            self.nccl.ncclGroupEnd()
        else:
            self.nccl.ncclReduceScatter(
                buffer_type(input_tensor.data_ptr()),
                buffer_type(output_tensor.data_ptr()),
                output_tensor.numel(),
                ncclDataTypeEnum.from_torch(input_tensor.dtype),
                ncclRedOpTypeEnum.from_torch(op),
                self.comm,
                cudaStream_t(stream.cuda_stream),
            )
```
**EN:** This callable implements `PyNcclCommunicator.reduce_scatter`. It takes `output_tensor`, `input_tensor`, `op`, `sizes` and mainly reduces or aggregates values. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `PyNcclCommunicator.reduce_scatter`。它接收 `output_tensor`, `input_tensor`, `op`, `sizes`，主要用于归约或聚合数值。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 299-314: Method PyNcclCommunicator.send
```python
    def send(self, tensor: torch.Tensor, dst: int):
        if self.disabled:
            return
        assert tensor.device == self.device, (
            f"this nccl communicator is created to work on {self.device}, "
            f"but the input tensor is on {tensor.device}"
        )
        stream = self._resolve_stream()
        self.nccl.ncclSend(
            buffer_type(tensor.data_ptr()),
            tensor.numel(),
            ncclDataTypeEnum.from_torch(tensor.dtype),
            dst,
            self.comm,
            cudaStream_t(stream.cuda_stream),
        )
```
**EN:** This callable implements `PyNcclCommunicator.send`. It takes `tensor`, `dst` and mainly sends data to another component. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `PyNcclCommunicator.send`。它接收 `tensor`, `dst`，主要用于向其他组件发送数据。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 316-331: Method PyNcclCommunicator.recv
```python
    def recv(self, tensor: torch.Tensor, src: int):
        if self.disabled:
            return
        assert tensor.device == self.device, (
            f"this nccl communicator is created to work on {self.device}, "
            f"but the input tensor is on {tensor.device}"
        )
        stream = self._resolve_stream()
        self.nccl.ncclRecv(
            buffer_type(tensor.data_ptr()),
            tensor.numel(),
            ncclDataTypeEnum.from_torch(tensor.dtype),
            src,
            self.comm,
            cudaStream_t(stream.cuda_stream),
        )
```
**EN:** This callable implements `PyNcclCommunicator.recv`. It takes `tensor`, `src` and mainly receives data from another component. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `PyNcclCommunicator.recv`。它接收 `tensor`, `src`，主要用于从其他组件接收数据。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 333-357: Method PyNcclCommunicator.broadcast
```python
    def broadcast(self, tensor: torch.Tensor, src: int):
        if self.disabled:
            return
        assert tensor.device == self.device, (
            f"this nccl communicator is created to work on {self.device}, "
            f"but the input tensor is on {tensor.device}"
        )
        stream = self._resolve_stream()

        if src == self.rank:
            sendbuff = buffer_type(tensor.data_ptr())
            # NCCL requires the sender also to have a receive buffer
            recvbuff = buffer_type(tensor.data_ptr())
        else:
            sendbuff = buffer_type()
            recvbuff = buffer_type(tensor.data_ptr())
        self.nccl.ncclBroadcast(
            sendbuff,
            recvbuff,
            tensor.numel(),
            ncclDataTypeEnum.from_torch(tensor.dtype),
            src,
            self.comm,
            cudaStream_t(stream.cuda_stream),
        )
```
**EN:** This callable implements `PyNcclCommunicator.broadcast`. It takes `tensor`, `src` and mainly implements broadcast. In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `PyNcclCommunicator.broadcast`。它接收 `tensor`, `src`，主要用于实现 broadcast 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 359-360: Method PyNcclCommunicator.register_comm_window_raw
```python
    def register_comm_window_raw(self, ptr: int, size: int):
        return self.nccl.ncclCommWindowRegister(self.comm, buffer_type(ptr), size, 1)
```
**EN:** This callable implements `PyNcclCommunicator.register_comm_window_raw`. It takes `ptr`, `size` and mainly registers hooks, handlers, or operators.
**CN:** 这一可调用对象实现了 `PyNcclCommunicator.register_comm_window_raw`。它接收 `ptr`, `size`，主要用于注册钩子、处理器或算子。

### Lines 362-363: Method PyNcclCommunicator.deregister_comm_window
```python
    def deregister_comm_window(self, window):
        return self.nccl.ncclCommWindowDeregister(self.comm, window)
```
**EN:** This callable implements `PyNcclCommunicator.deregister_comm_window`. It takes `window` and mainly registers hooks, handlers, or operators.
**CN:** 这一可调用对象实现了 `PyNcclCommunicator.deregister_comm_window`。它接收 `window`，主要用于注册钩子、处理器或算子。

### Lines 365-366: Method PyNcclCommunicator.group_start
```python
    def group_start(self):
        self.nccl.ncclGroupStart()
```
**EN:** This callable implements `PyNcclCommunicator.group_start` and mainly implements group start.
**CN:** 这一可调用对象实现了 `PyNcclCommunicator.group_start`，主要用于实现 group start 相关逻辑。

### Lines 368-369: Method PyNcclCommunicator.group_end
```python
    def group_end(self):
        self.nccl.ncclGroupEnd()
```
**EN:** This callable implements `PyNcclCommunicator.group_end` and mainly implements group end.
**CN:** 这一可调用对象实现了 `PyNcclCommunicator.group_end`，主要用于实现 group end 相关逻辑。

### Lines 371-385: Method PyNcclCommunicator.change_state
```python
    @contextmanager
    def change_state(self, enable: Optional[bool] = None):
        """
        A context manager to change the enabled state of the communicator.
        """
        if enable is None:
            # guess a default value when not specified
            enable = self.available

        old_disable = self.disabled
        self.disabled = not enable
        try:
            yield
        finally:
            self.disabled = old_disable
```
**EN:** This callable implements `PyNcclCommunicator.change_state`. It takes `enable` and mainly implements change state. The docstring states: "A context manager to change the enabled state of the communicator."
**CN:** 这一可调用对象实现了 `PyNcclCommunicator.change_state`。它接收 `enable`，主要用于实现 change state 相关逻辑。

## Key Concepts / 关键概念
- `PyNcclCommunicator`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `contextlib`, `typing`
- **Third-party / 第三方**: `torch`, `torch.distributed`
- **Internal modules / 内部模块**: `sglang.srt.distributed.device_communicators.pynccl_wrapper`, `sglang.srt.distributed.utils`, `sglang.srt.utils.common`
