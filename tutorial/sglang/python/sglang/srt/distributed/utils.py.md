# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/distributed/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed execution and communication part of the SRT runtime and implements logic centered on `utils`. It exposes primary entry points such as `set_global_tcp_store`, `get_global_tcp_store`, `ensure_divisibility`. / 该模块属于 SRT 运行时的分布式执行与通信部分，主要实现围绕 `utils` 的逻辑。 它对外提供的主要入口包括 `set_global_tcp_store`, `get_global_tcp_store`, `ensure_divisibility`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Module imports, constants, and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/v0.6.4.post1/vllm/distributed/utils.py

# Copyright 2023 The vLLM team.
# Adapted from
# https://github.com/NVIDIA/Megatron-LM/blob/main/megatron/core/tensor_parallel/utils.py
# Copyright (c) 2022, NVIDIA CORPORATION. All rights reserved.
import dataclasses
import logging
import os
import pickle
import time
from collections import deque
from typing import Any, Deque, Dict, Optional, Sequence, Tuple

import torch
from torch.distributed import TCPStore

logger = logging.getLogger(__name__)

# Global TCPStore that is created during distributed initialization
# This is the single shared store that all components should use
_global_tcp_store: Optional[TCPStore] = None


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；协调分布式通信。

### Lines 27-35: Function set_global_tcp_store
```python
def set_global_tcp_store(store: TCPStore) -> None:
    """Set the global TCPStore instance.

    This should be called during distributed initialization to make
    the store available to all components that need it.
    """
    global _global_tcp_store
    _global_tcp_store = store
    logger.info("Global TCPStore has been set")
```
**EN:** This callable implements `set_global_tcp_store`. It takes `store` and mainly converts data into another representation. The docstring states: "Set the global TCPStore instance." In this range it emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `set_global_tcp_store`。它接收 `store`，主要用于将数据转换为另一种表示。 在这一范围内，它会输出日志以便诊断。

### Lines 38-56: Function get_global_tcp_store
```python
def get_global_tcp_store() -> Optional[TCPStore]:
    """Get the existing global TCPStore.

    This function provides access to the shared TCPStore instance that was
    created during distributed initialization. All components (like NIXL buffers)
    should use this same store for coordination.

    Returns:
        The global TCPStore instance, or None if not initialized yet.
    """
    global _global_tcp_store

    if _global_tcp_store is None:
        logger.warning(
            "Global TCPStore not found. Make sure init_distributed_environment "
            "was called with a tcp:// init method."
        )

    return _global_tcp_store
```
**EN:** This callable implements `get_global_tcp_store` and mainly converts data into another representation. The docstring states: "Get the existing global TCPStore." In this range it emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `get_global_tcp_store`，主要用于将数据转换为另一种表示。 在这一范围内，它会输出日志以便诊断。

### Lines 59-63: Function ensure_divisibility
```python
def ensure_divisibility(numerator, denominator):
    """Ensure that numerator is divisible by the denominator."""
    assert numerator % denominator == 0, "{} is not divisible by {}".format(
        numerator, denominator
    )
```
**EN:** This callable implements `ensure_divisibility`. It takes `numerator`, `denominator` and mainly ensures required state exists. The docstring states: "Ensure that numerator is divisible by the denominator." In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ensure_divisibility`。它接收 `numerator`, `denominator`，主要用于确保所需状态已经存在。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 66-70: Function divide
```python
def divide(numerator, denominator):
    """Ensure that numerator is divisible by the denominator and return
    the division value."""
    ensure_divisibility(numerator, denominator)
    return numerator // denominator
```
**EN:** This callable implements `divide`. It takes `numerator`, `denominator` and mainly implements divide. The docstring states: "Ensure that numerator is divisible by the denominator and return the division value."
**CN:** 这一可调用对象实现了 `divide`。它接收 `numerator`, `denominator`，主要用于实现 divide 相关逻辑。

### Lines 73-98: Function split_tensor_along_last_dim
```python
def split_tensor_along_last_dim(
    tensor: torch.Tensor,
    num_partitions: int,
    contiguous_split_chunks: bool = False,
) -> Sequence[torch.Tensor]:
    """Split a tensor along its last dimension.

    Arguments:
        tensor: input tensor.
        num_partitions: number of partitions to split the tensor
        contiguous_split_chunks: If True, make each chunk contiguous
                                 in memory.

    Returns:
        A list of Tensors
    """
    # Get the size and dimension.
    last_dim = tensor.dim() - 1
    last_dim_size = divide(tensor.size()[last_dim], num_partitions)
    # Split.
    tensor_list = torch.split(tensor, last_dim_size, dim=last_dim)
    # NOTE: torch.split does not create contiguous tensors by default.
    if contiguous_split_chunks:
        return tuple(chunk.contiguous() for chunk in tensor_list)

    return tensor_list
```
**EN:** This callable implements `split_tensor_along_last_dim`. It takes `tensor`, `num_partitions`, `contiguous_split_chunks` and mainly implements split tensor along last dim. The docstring states: "Split a tensor along its last dimension."
**CN:** 这一可调用对象实现了 `split_tensor_along_last_dim`。它接收 `tensor`, `num_partitions`, `contiguous_split_chunks`，主要用于实现 split tensor along last dim 相关逻辑。

### Lines 101-137: Function get_pp_indices
```python
def get_pp_indices(
    num_hidden_layers: int, pp_rank: int, pp_size: int
) -> Tuple[int, int]:
    """Try to evenly distribute layers across partitions.
    If the number of layers is not divisible by the number of partitions,
    the last N partitions will have one extra layer, where N = remainder.
    """
    # partition_list_str can be set to None in sglang
    partition_list_str = os.getenv("SGLANG_PP_LAYER_PARTITION", None)
    if partition_list_str is not None:
        try:
            partitions = [int(layer) for layer in partition_list_str.split(",")]
        except ValueError as err:
            raise ValueError(
                "Invalid partition string: {}".format(partition_list_str)
            ) from err
        if len(partitions) != pp_size:
            raise ValueError(f"{len(partitions)=} does not match {pp_size=}.")
        if sum(partitions) != num_hidden_layers:
            raise ValueError(f"{sum(partitions)=} does not match {num_hidden_layers=}.")
        start_layer = sum(partitions[:pp_rank])
        end_layer = start_layer + partitions[pp_rank]
    else:
        base_layers = num_hidden_layers // pp_size
        remainder = num_hidden_layers % pp_size
        # Distribute the extra layers to the last 'remainder' partitions
        if pp_rank >= pp_size - remainder:
            partitions_without_extra_layer = pp_size - remainder
            # This partition gets one extra layer
            start_layer = pp_rank * (base_layers + 1) - partitions_without_extra_layer
            end_layer = start_layer + (base_layers + 1)
        else:
            # This partition gets only base layers
            start_layer = pp_rank * base_layers
            end_layer = start_layer + base_layers

    return (start_layer, end_layer)
```
**EN:** This callable implements `get_pp_indices`. It takes `num_hidden_layers`, `pp_rank`, `pp_size` and mainly retrieves a value or derived view. The docstring states: "Try to evenly distribute layers across partitions." In this range it sets up imports and shared symbols; performs defensive checks on invalid state; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `get_pp_indices`。它接收 `num_hidden_layers`, `pp_rank`, `pp_size`，主要用于获取某个值或派生视图。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；读取环境变量驱动的配置。

### Lines 140-161: Class StatelessProcessGroup
```python
@dataclasses.dataclass
class StatelessProcessGroup:
    """A dataclass to hold a metadata store, and the rank, world_size of the
    group. Only use it to communicate metadata between processes.
    For data-plane communication, create NCCL-related objects.
    """

    rank: int
    world_size: int
    store: torch._C._distributed_c10d.Store
    data_expiration_seconds: int = 3600  # 1 hour

    # dst rank -> counter
    send_dst_counter: Dict[int, int] = dataclasses.field(default_factory=dict)
    # src rank -> counter
    recv_src_counter: Dict[int, int] = dataclasses.field(default_factory=dict)
    broadcast_send_counter: int = 0
    broadcast_recv_src_counter: Dict[int, int] = dataclasses.field(default_factory=dict)

    # A deque to store the data entries, with key and timestamp.
    entries: Deque[Tuple[str, float]] = dataclasses.field(default_factory=deque)

```
**EN:** This range introduces `StatelessProcessGroup` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "A dataclass to hold a metadata store, and the rank, world_size of the group." In this range it coordinates distributed communication.
**CN:** 这一段引入 `StatelessProcessGroup`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会协调分布式通信。

### Lines 162-166: Method StatelessProcessGroup.__post_init__
```python
    def __post_init__(self):
        assert self.rank < self.world_size
        self.send_dst_counter = {i: 0 for i in range(self.world_size)}
        self.recv_src_counter = {i: 0 for i in range(self.world_size)}
        self.broadcast_recv_src_counter = {i: 0 for i in range(self.world_size)}
```
**EN:** This callable implements `StatelessProcessGroup.__post_init__` and mainly implements post init. In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `StatelessProcessGroup.__post_init__`，主要用于实现 post init 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 168-174: Method StatelessProcessGroup.send_obj
```python
    def send_obj(self, obj: Any, dst: int):
        """Send an object to a destination rank."""
        self.expire_data()
        key = f"send_to/{dst}/{self.send_dst_counter[dst]}"
        self.store.set(key, pickle.dumps(obj))
        self.send_dst_counter[dst] += 1
        self.entries.append((key, time.perf_counter()))
```
**EN:** This callable implements `StatelessProcessGroup.send_obj`. It takes `obj`, `dst` and mainly sends data to another component. The docstring states: "Send an object to a destination rank."
**CN:** 这一可调用对象实现了 `StatelessProcessGroup.send_obj`。它接收 `obj`, `dst`，主要用于向其他组件发送数据。

### Lines 176-185: Method StatelessProcessGroup.expire_data
```python
    def expire_data(self):
        """Expire data that is older than `data_expiration_seconds` seconds."""
        while self.entries:
            # check the oldest entry
            key, timestamp = self.entries[0]
            if time.perf_counter() - timestamp > self.data_expiration_seconds:
                self.store.delete_key(key)
                self.entries.popleft()
            else:
                break
```
**EN:** This callable implements `StatelessProcessGroup.expire_data` and mainly implements expire data. The docstring states: "Expire data that is older than `data_expiration_seconds` seconds."
**CN:** 这一可调用对象实现了 `StatelessProcessGroup.expire_data`，主要用于实现 expire data 相关逻辑。

### Lines 187-193: Method StatelessProcessGroup.recv_obj
```python
    def recv_obj(self, src: int) -> Any:
        """Receive an object from a source rank."""
        obj = pickle.loads(
            self.store.get(f"send_to/{self.rank}/{self.recv_src_counter[src]}")
        )
        self.recv_src_counter[src] += 1
        return obj
```
**EN:** This callable implements `StatelessProcessGroup.recv_obj`. It takes `src` and mainly receives data from another component. The docstring states: "Receive an object from a source rank." In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `StatelessProcessGroup.recv_obj`。它接收 `src`，主要用于从其他组件接收数据。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 195-211: Method StatelessProcessGroup.broadcast_obj
```python
    def broadcast_obj(self, obj: Optional[Any], src: int) -> Any:
        """Broadcast an object from a source rank to all other ranks.
        It does not clean up after all ranks have received the object.
        Use it for limited times, e.g., for initialization.
        """
        if self.rank == src:
            self.expire_data()
            key = f"broadcast_from/{src}/" f"{self.broadcast_send_counter}"
            self.store.set(key, pickle.dumps(obj))
            self.broadcast_send_counter += 1
            self.entries.append((key, time.perf_counter()))
            return obj
        else:
            key = f"broadcast_from/{src}/" f"{self.broadcast_recv_src_counter[src]}"
            recv_obj = pickle.loads(self.store.get(key))
            self.broadcast_recv_src_counter[src] += 1
            return recv_obj
```
**EN:** This callable implements `StatelessProcessGroup.broadcast_obj`. It takes `obj`, `src` and mainly implements broadcast obj. The docstring states: "Broadcast an object from a source rank to all other ranks." In this range it sets up imports and shared symbols; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `StatelessProcessGroup.broadcast_obj`。它接收 `obj`, `src`，主要用于实现 broadcast obj 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；协调分布式通信。

### Lines 213-223: Method StatelessProcessGroup.all_gather_obj
```python
    def all_gather_obj(self, obj: Any) -> list[Any]:
        """All gather an object from all ranks."""
        gathered_objs = []
        for i in range(self.world_size):
            if i == self.rank:
                gathered_objs.append(obj)
                self.broadcast_obj(obj, src=self.rank)
            else:
                recv_obj = self.broadcast_obj(None, src=i)
                gathered_objs.append(recv_obj)
        return gathered_objs
```
**EN:** This callable implements `StatelessProcessGroup.all_gather_obj`. It takes `obj` and mainly implements all gather obj. The docstring states: "All gather an object from all ranks." In this range it sets up imports and shared symbols; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `StatelessProcessGroup.all_gather_obj`。它接收 `obj`，主要用于实现 all gather obj 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；协调分布式通信。

### Lines 225-231: Method StatelessProcessGroup.barrier
```python
    def barrier(self):
        """A barrier to synchronize all ranks."""
        for i in range(self.world_size):
            if i == self.rank:
                self.broadcast_obj(None, src=self.rank)
            else:
                self.broadcast_obj(None, src=i)
```
**EN:** This callable implements `StatelessProcessGroup.barrier` and mainly implements barrier. The docstring states: "A barrier to synchronize all ranks." In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `StatelessProcessGroup.barrier`，主要用于实现 barrier 相关逻辑。 在这一范围内，它会协调分布式通信。

### Lines 233-268: Method StatelessProcessGroup.create
```python
    @staticmethod
    def create(
        host: str,
        port: int,
        rank: int,
        world_size: int,
        data_expiration_seconds: int = 3600,
    ) -> "StatelessProcessGroup":
        """A replacement for `torch.distributed.init_process_group` that does not
        pollute the global state.

        If we have process A and process B called `torch.distributed.init_process_group`
        to form a group, and then we want to form another group with process A, B, C,
        D, it is not possible in PyTorch, because process A and process B have already
        formed a group, and process C and process D cannot join that group. This
        function is a workaround for this issue.

        `torch.distributed.init_process_group` is a global call, while this function
        is a stateless call. It will return a `StatelessProcessGroup` object that can be
        used for exchanging metadata. With this function, process A and process B
        can call `StatelessProcessGroup.create` to form a group, and then process A, B,
        C, and D can call `StatelessProcessGroup.create` to form another group.
        """  # noqa
        store = TCPStore(
            host_name=host,
            port=port,
            world_size=world_size,
            is_master=(rank == 0),
        )

        return StatelessProcessGroup(
            rank=rank,
            world_size=world_size,
            store=store,
            data_expiration_seconds=data_expiration_seconds,
        )
```
**EN:** This callable implements `StatelessProcessGroup.create`. It takes `host`, `port`, `rank`, `world_size` and mainly constructs new objects or contexts. The docstring states: "A replacement for `torch.distributed.init_process_group` that does not pollute the global state." In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `StatelessProcessGroup.create`。它接收 `host`, `port`, `rank`, `world_size`，主要用于构造新的对象或上下文。 在这一范围内，它会协调分布式通信。

## Key Concepts / 关键概念
- `set_global_tcp_store`: converts data into another representation / 将数据转换为另一种表示
- `get_global_tcp_store`: converts data into another representation / 将数据转换为另一种表示
- `ensure_divisibility`: ensures required state exists / 确保所需状态已经存在
- `divide`: implements divide / 实现 divide 相关逻辑
- `split_tensor_along_last_dim`: implements split tensor along last dim / 实现 split tensor along last dim 相关逻辑
- `get_pp_indices`: retrieves a value or derived view / 获取某个值或派生视图
- `StatelessProcessGroup`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `logging`, `os`, `pickle`, `time`, `collections`, `typing`
- **Third-party / 第三方**: `torch`, `torch.distributed`
