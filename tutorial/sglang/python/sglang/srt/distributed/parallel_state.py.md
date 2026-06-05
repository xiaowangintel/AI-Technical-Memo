# parallel_state.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/distributed/parallel_state.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed execution and communication part of the SRT runtime and implements logic centered on `parallel_state`. The module docstring frames it as: "Distributed state." / 该模块属于 SRT 运行时的分布式执行与通信部分，主要实现围绕 `parallel_state` 的逻辑。 它对外提供的主要入口包括 `get_torch_distributed_pg_options`, `GraphCaptureContext`, `P2PWork`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60: Module imports, constants, and setup (part 1/2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/v0.6.4.post1/vllm/distributed/parallel_state.py

# Copyright 2023 The vLLM team.
# Adapted from
# https://github.com/NVIDIA/Megatron-LM/blob/main/megatron/core/parallel_state.py
# Copyright (c) 2022, NVIDIA CORPORATION. All rights reserved.
"""Distributed state.
It takes over the control of the distributed environment from PyTorch.
The typical workflow is:

- call `init_distributed_environment` to initialize the distributed environment.
- call `initialize_model_parallel` or `ensure_model_parallel_initialized` to
 initialize the model parallel groups.

- any code dealing with the distributed stuff

- call `destroy_model_parallel` to destroy the model parallel groups.
- call `destroy_distributed_environment` to destroy the distributed environment.

If you only need to use the distributed environment without model/pipeline
 parallelism, you can skip the model parallel initialization and destruction
 steps.
"""

import contextlib
import gc
import logging
import os
import pickle
import weakref
from collections import namedtuple
from contextlib import contextmanager, nullcontext
from dataclasses import dataclass
from datetime import timedelta
from multiprocessing import shared_memory
from typing import Any, Callable, Dict, List, Optional, Tuple, Union
from unittest.mock import patch

import torch
import torch.distributed
from torch.distributed import Backend, ProcessGroup

from sglang.srt.compilation.compilation_config import register_split_op
from sglang.srt.compilation.piecewise_context_manager import is_in_piecewise_cuda_graph
from sglang.srt.distributed.utils import set_global_tcp_store
from sglang.srt.environ import envs
from sglang.srt.utils import (
    get_current_device_stream_fast,
    get_int_env_var,
    is_cpu,
    is_cuda_alike,
    is_hip,
    is_musa,
    is_npu,
    is_shm_available,
    is_xpu,
)
from sglang.srt.utils.custom_op import register_custom_op
```
**EN:** This range organizes module-level state and shared setup. This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols; coordinates distributed communication; reads environment-driven configuration.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；协调分布式通信；读取环境变量驱动的配置。

### Lines 61-77: Module imports, constants, and setup (part 2/2)
```python
from sglang.srt.utils.network import get_local_ip_auto

_is_npu = is_npu()
_is_cpu = is_cpu()
_is_xpu = is_xpu()
_is_musa = is_musa()

TensorMetadata = namedtuple("TensorMetadata", ["device", "dtype", "size"])

# use int value instead of ReduceOp.SUM to support torch compile
REDUCE_OP_SUM = int(torch.distributed.ReduceOp.SUM)

# Reuse the user-provided distributed timeout for model-parallel subgroup
# creation so runtime collectives do not silently fall back to backend defaults.
_MODEL_PARALLEL_GROUP_TIMEOUT: Optional[timedelta] = None


```
**EN:** This range organizes module-level state and shared setup. This chunk is part 2 of 2 for the same logical block. In this range it sets up imports and shared symbols; coordinates distributed communication; prepares compilation-related behavior.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；协调分布式通信；处理与编译相关的行为。

### Lines 78-93: Function get_torch_distributed_pg_options
```python
def get_torch_distributed_pg_options(group_name=None):
    if not _is_npu:
        return None

    # Only create HCCL options for default group or MoE-related groups
    if group_name is not None and "moe" not in group_name:
        return None

    import torch_npu

    options = torch_npu._C._distributed_c10d.ProcessGroupHCCL.Options()
    hccl_buffer_size = int(
        os.environ.get("DEEPEP_HCCL_BUFFSIZE") or os.environ.get("HCCL_BUFFSIZE") or 200
    )
    options.hccl_config = {"hccl_buffer_size": hccl_buffer_size}
    return options
```
**EN:** This callable implements `get_torch_distributed_pg_options`. It takes `group_name` and mainly converts data into another representation. In this range it sets up imports and shared symbols; coordinates distributed communication; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `get_torch_distributed_pg_options`。它接收 `group_name`，主要用于将数据转换为另一种表示。 在这一范围内，它会建立导入关系并准备共享符号；协调分布式通信；读取环境变量驱动的配置。

### Lines 96-98: Class GraphCaptureContext
```python
@dataclass
class GraphCaptureContext:
    stream: torch.get_device_module().Stream
```
**EN:** This range introduces `GraphCaptureContext` and defines the structure or metadata that its methods rely on. In this range it manages graph capture or replay logic.
**CN:** 这一段引入 `GraphCaptureContext`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 101-104: Class P2PWork
```python
@dataclass
class P2PWork:
    work: Optional[torch.distributed.Work]
    payload: Optional[torch.Tensor]
```
**EN:** This range introduces `P2PWork` and defines the structure or metadata that its methods rely on. In this range it coordinates distributed communication.
**CN:** 这一段引入 `P2PWork`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会协调分布式通信。

### Lines 107-130: Function _split_tensor_dict
```python
def _split_tensor_dict(
    tensor_dict: Dict[str, Union[torch.Tensor, Any]],
) -> Tuple[List[Tuple[str, Any]], List[torch.Tensor]]:
    """Split the tensor dictionary into two parts:
    1. A list of (key, value) pairs. If the value is a tensor, it is replaced
         by its metadata.
    2. A list of tensors.
    """
    metadata_list: List[Tuple[str, Any]] = []
    tensor_list: List[torch.Tensor] = []
    for key, value in tensor_dict.items():
        if isinstance(value, torch.Tensor):
            # Note: we cannot use `value.device` here,
            # because it contains not only the device type but also the device
            # index (e.g. "cuda:0"). We only need the device type.
            # receiving side will set the device index.
            device = value.device.type
            metadata_list.append(
                (key, TensorMetadata(device, value.dtype, value.size()))
            )
            tensor_list.append(value)
        else:
            metadata_list.append((key, value))
    return metadata_list, tensor_list
```
**EN:** This callable implements `_split_tensor_dict`. It takes `tensor_dict` and mainly implements split tensor dict. The docstring states: "Split the tensor dictionary into two parts: 1."
**CN:** 这一可调用对象实现了 `_split_tensor_dict`。它接收 `tensor_dict`，主要用于实现 split tensor dict 相关逻辑。

### Lines 131-135: Module-level constants and helpers
```python


_group_name_counter: Dict[str, int] = {}


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 136-146: Function _get_unique_name
```python
def _get_unique_name(name: str) -> str:
    """Get a unique name for the group.
    Example:
    _get_unique_name("tp") -> "tp:0"
    _get_unique_name("tp") -> "tp:1"
    """
    if name not in _group_name_counter:
        _group_name_counter[name] = 0
    newname = f"{name}:{_group_name_counter[name]}"
    _group_name_counter[name] += 1
    return newname
```
**EN:** This callable implements `_get_unique_name`. It takes `name` and mainly retrieves a value or derived view. The docstring states: "Get a unique name for the group."
**CN:** 这一可调用对象实现了 `_get_unique_name`。它接收 `name`，主要用于获取某个值或派生视图。

### Lines 147-151: Module-level constants and helpers
```python


_groups: Dict[str, Callable[[], Optional["GroupCoordinator"]]] = {}


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 152-153: Function _register_group
```python
def _register_group(group: "GroupCoordinator") -> None:
    _groups[group.unique_name] = weakref.ref(group)
```
**EN:** This callable implements `_register_group`. It takes `group` and mainly registers hooks, handlers, or operators.
**CN:** 这一可调用对象实现了 `_register_group`。它接收 `group`，主要用于注册钩子、处理器或算子。

### Lines 156-163: Function inplace_all_reduce
```python
@register_custom_op(mutates_args=["tensor"])
@register_split_op()
def inplace_all_reduce(tensor: torch.Tensor, group_name: str) -> None:
    assert group_name in _groups, f"Group {group_name} is not found."
    group = _groups[group_name]()
    if group is None:
        raise ValueError(f"Group {group_name} is destroyed.")
    group._all_reduce_in_place(tensor)
```
**EN:** This callable implements `inplace_all_reduce`. It takes `tensor`, `group_name` and mainly reduces or aggregates values. In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `inplace_all_reduce`。它接收 `tensor`, `group_name`，主要用于归约或聚合数值。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 166-174: Function outplace_all_reduce
```python
@register_custom_op(out_shape="tensor")
def outplace_all_reduce(
    tensor: torch.Tensor, group_name: str, outplace_all_reduce_method: str
) -> torch.Tensor:
    assert group_name in _groups, f"Group {group_name} is not found."
    group = _groups[group_name]()
    if group is None:
        raise ValueError(f"Group {group_name} is destroyed.")
    return group._all_reduce_out_place(tensor, outplace_all_reduce_method)
```
**EN:** This callable implements `outplace_all_reduce`. It takes `tensor`, `group_name`, `outplace_all_reduce_method` and mainly reduces or aggregates values. In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `outplace_all_reduce`。它接收 `tensor`, `group_name`, `outplace_all_reduce_method`，主要用于归约或聚合数值。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 177-185: Function reg_all_gather_into_tensor
```python
@register_custom_op(mutates_args=["output"])
def reg_all_gather_into_tensor(
    output: torch.Tensor, input: torch.Tensor, group_name: str
) -> None:
    assert group_name in _groups, f"Group {group_name} is not found."
    group = _groups[group_name]()
    if group is None:
        raise ValueError(f"Group {group_name} is destroyed.")
    group._all_gather_into_tensor(output, input)
```
**EN:** This callable implements `reg_all_gather_into_tensor`. It takes `output`, `input`, `group_name` and mainly converts data into another representation. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `reg_all_gather_into_tensor`。它接收 `output`, `input`, `group_name`，主要用于将数据转换为另一种表示。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 188-196: Function reg_reduce_scatter_tensor
```python
@register_custom_op(mutates_args=["output"])
def reg_reduce_scatter_tensor(
    output: torch.Tensor, input: torch.Tensor, group_name: str
) -> None:
    assert group_name in _groups, f"Group {group_name} is not found."
    group = _groups[group_name]()
    if group is None:
        raise ValueError(f"Group {group_name} is destroyed.")
    group._reduce_scatter_tensor(output, input)
```
**EN:** This callable implements `reg_reduce_scatter_tensor`. It takes `output`, `input`, `group_name` and mainly reduces or aggregates values. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `reg_reduce_scatter_tensor`。它接收 `output`, `input`, `group_name`，主要用于归约或聚合数值。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 199-239: Class GroupCoordinator
```python
class GroupCoordinator:
    """
    PyTorch ProcessGroup wrapper for a group of processes.
    PyTorch ProcessGroup is bound to one specific communication backend,
        e.g. NCCL, Gloo, MPI, etc.
    GroupCoordinator takes charge of all the communication operations among
        the processes in the group. It can route the communication to
        a specific implementation (e.g. switch allreduce implementation
        based on the tensor size and cuda graph mode).
    """

    # available attributes:
    rank: int  # global rank
    ranks: List[int]  # global ranks in the group
    world_size: int  # size of the group
    # difference between `local_rank` and `rank_in_group`:
    # if we have a group of size 4 across two nodes:
    # Process | Node | Rank | Local Rank | Rank in Group
    #   0     |   0  |  0   |     0      |       0
    #   1     |   0  |  1   |     1      |       1
    #   2     |   1  |  2   |     0      |       2
    #   3     |   1  |  3   |     1      |       3
    local_rank: int  # local rank used to assign devices
    rank_in_group: int  # rank inside the group
    cpu_group: ProcessGroup  # group for CPU communication
    device_group: ProcessGroup  # group for device communication
    use_pynccl: bool  # a hint of whether to use PyNccl
    use_pymscclpp: bool  # a hint of whether to use PyMsccl
    use_custom_allreduce: bool  # a hint of whether to use CustomAllreduce
    use_torch_symm_mem_all_reduce: (
        bool  # a hint of whether to use TorchSymmMemAllReduce
    )
    use_message_queue_broadcaster: (
        bool  # a hint of whether to use message queue broadcaster
    )
    # communicators are only created for world size > 1
    pynccl_comm: Optional[Any]  # PyNccl communicator
    ca_comm: Optional[Any]  # Custom allreduce communicator
    torch_symm_mem_comm: Optional[Any]  # Torch symm mem communicator
    mq_broadcaster: Optional[Any]  # shared memory broadcaster

```
**EN:** This range introduces `GroupCoordinator` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "PyTorch ProcessGroup wrapper for a group of processes." In this range it coordinates distributed communication; manages graph capture or replay logic.
**CN:** 这一段引入 `GroupCoordinator`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会协调分布式通信；管理图捕获或回放逻辑。

### Lines 240-299: Method GroupCoordinator.__init__ (part 1/4)
```python
    def __init__(
        self,
        group_ranks: List[List[int]],
        local_rank: int,
        torch_distributed_backend: Union[str, Backend],
        use_pynccl: bool,
        use_pymscclpp: bool,
        use_custom_allreduce: bool,
        use_torch_symm_mem_all_reduce: bool,
        use_hpu_communicator: bool,
        use_xpu_communicator: bool,
        use_npu_communicator: bool,
        use_message_queue_broadcaster: bool = False,
        group_name: Optional[str] = None,
        gloo_timeout: timedelta = timedelta(seconds=120 * 60),
        recovered_rank: bool = False,
    ):
        # Set group info
        group_name = group_name or "anonymous"
        self.unique_name = _get_unique_name(group_name)
        _register_group(self)

        # Set rank info
        self.rank = torch.distributed.get_rank()
        self.local_rank = local_rank
        self.device_group = None
        self.cpu_group = None
        self.local_size = get_int_env_var("LOCAL_SIZE", 0)

        if is_cuda_alike():
            device_id = (
                0 if envs.SGLANG_ONE_VISIBLE_DEVICE_PER_PROCESS.get() else local_rank
            )
            self.device = torch.device(f"cuda:{device_id}")
        elif _is_npu:
            self.device = torch.device(f"npu:{local_rank}")
        elif _is_xpu:
            self.device = torch.device(f"xpu:{local_rank}")
        elif _is_musa:
            self.device = torch.device(f"musa:{local_rank}")
        else:
            self.device = torch.device("cpu")
        self.device_module = torch.get_device_module(self.device)

        for ranks in group_ranks:
            active_ranks = torch.ones(len(ranks), dtype=torch.int32, device=self.device)
            active_ranks_cpu = torch.ones(len(ranks), dtype=torch.int32)
            subgroup_timeout = _MODEL_PARALLEL_GROUP_TIMEOUT
            if "mooncake" in torch_distributed_backend:
                from mooncake.ep import MooncakeBackendOptions

                device_group = torch.distributed.new_group(
                    ranks,
                    backend="mooncake",
                    pg_options=MooncakeBackendOptions(active_ranks, recovered_rank),
                    timeout=subgroup_timeout,
                )
                cpu_group = torch.distributed.new_group(
                    ranks,
                    backend="mooncake-cpu",
```
**EN:** This callable implements `GroupCoordinator.__init__`. It takes `group_ranks`, `local_rank`, `torch_distributed_backend`, `use_pynccl` and mainly initializes instance state and defaults. This chunk is part 1 of 4 for the same logical block. In this range it sets up imports and shared symbols; coordinates distributed communication; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `GroupCoordinator.__init__`。它接收 `group_ranks`, `local_rank`, `torch_distributed_backend`, `use_pynccl`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 1/4 部分。 在这一范围内，它会建立导入关系并准备共享符号；协调分布式通信；读取环境变量驱动的配置。

### Lines 300-359: Method GroupCoordinator.__init__ (part 2/4)
```python
                    pg_options=MooncakeBackendOptions(active_ranks_cpu, recovered_rank),
                    timeout=subgroup_timeout,
                )
            else:
                pg_options = get_torch_distributed_pg_options(group_name)
                device_group = torch.distributed.new_group(
                    ranks,
                    backend=torch_distributed_backend,
                    pg_options=pg_options,
                    timeout=subgroup_timeout,
                )
                # a group with `gloo` backend, to allow direct coordination
                # between processes through the CPU.
                cpu_group = torch.distributed.new_group(
                    ranks, backend="gloo", timeout=gloo_timeout
                )
            if self.rank in ranks:
                self.ranks = ranks
                self.world_size = len(ranks)
                self.rank_in_group = ranks.index(self.rank)
                self.device_group = device_group
                self.cpu_group = cpu_group
                self.active_ranks = active_ranks
                self.active_ranks_cpu = active_ranks_cpu

        assert self.cpu_group is not None
        assert self.device_group is not None

        # Import communicators
        self.use_pynccl = use_pynccl
        self.use_pymscclpp = use_pymscclpp
        self.use_custom_allreduce = use_custom_allreduce
        self.use_torch_symm_mem_all_reduce = use_torch_symm_mem_all_reduce
        self.use_hpu_communicator = use_hpu_communicator
        self.use_xpu_communicator = use_xpu_communicator
        self.use_npu_communicator = use_npu_communicator
        self.use_message_queue_broadcaster = use_message_queue_broadcaster

        # Lazy import to avoid documentation build error
        from sglang.srt.distributed.device_communicators.custom_all_reduce import (
            dispatch_custom_allreduce,
        )
        from sglang.srt.distributed.device_communicators.pymscclpp import (
            PyMscclppCommunicator,
        )
        from sglang.srt.distributed.device_communicators.pynccl import (
            PyNcclCommunicator,
        )
        from sglang.srt.distributed.device_communicators.pynccl_allocator import (
            debug_check_symmetric_mempool,
            is_symmetric_memory_enabled,
            use_symmetric_memory,
        )
        from sglang.srt.distributed.device_communicators.torch_symm_mem import (
            TorchSymmMemCommunicator,
        )
        from sglang.srt.layers.dp_attention import is_allocation_symmetric

        self.is_symmetric_memory_enabled = is_symmetric_memory_enabled
        self.use_symmetric_memory = use_symmetric_memory
```
**EN:** This callable implements `GroupCoordinator.__init__`. It takes `group_ranks`, `local_rank`, `torch_distributed_backend`, `use_pynccl` and mainly initializes instance state and defaults. This chunk is part 2 of 4 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.__init__`。它接收 `group_ranks`, `local_rank`, `torch_distributed_backend`, `use_pynccl`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 2/4 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；协调分布式通信。

### Lines 360-419: Method GroupCoordinator.__init__ (part 3/4)
```python
        self.is_allocation_symmetric = is_allocation_symmetric
        self.debug_check_symmetric_mempool = debug_check_symmetric_mempool
        if is_hip():
            from sglang.srt.distributed.device_communicators.quick_all_reduce import (
                QuickAllReduce,
                qr_rocm_arch_available,
            )

        self.pynccl_comm: Optional[PyNcclCommunicator] = None
        if use_pynccl and self.world_size > 1:
            self.pynccl_comm = PyNcclCommunicator(
                group=self.cpu_group,
                device=self.device,
            )

        self.pymscclpp_comm: Optional[PyMscclppCommunicator] = None
        if use_pymscclpp and self.world_size > 1:
            self.pymscclpp_comm = PyMscclppCommunicator(
                group=self.cpu_group,
                device=self.device,
            )

        self.ca_comm: Optional[Any] = None
        self.qr_comm: Optional[QuickAllReduce] = None
        if use_custom_allreduce and self.world_size > 1:
            # Initialize a custom fast all-reduce implementation.
            try:
                CAClass = dispatch_custom_allreduce()
                self.ca_comm = CAClass(
                    group=self.cpu_group,
                    device=self.device,
                )
            except Exception as e:
                logger.warning(
                    f"Setup Custom allreduce failed with {e}. To silence this "
                    "warning, specify --disable-custom-all-reduce explicitly."
                )

            if is_hip():
                try:
                    # Initialize a custom quick all-reduce implementation for AMD
                    # when rocm >= gfx942. Quick reduce is designed as a
                    # complement to custom allreduce.
                    # Based on quickreduce (https://github.com/mk1-project/quickreduce).
                    if qr_rocm_arch_available():
                        self.qr_comm = QuickAllReduce(
                            group=self.cpu_group, device=self.device
                        )
                except Exception as e:
                    logger.warning(f"Failed to initialize QuickAllReduce: {e}")
        elif self.world_size > 1 and is_hip():
            logger.info("[AR] All-reduce call path: NCCL (custom AR disabled)")

        self.torch_symm_mem_comm: Optional[TorchSymmMemCommunicator] = None
        if self.use_torch_symm_mem_all_reduce and self.world_size > 1:
            self.torch_symm_mem_comm = TorchSymmMemCommunicator(
                group=self.cpu_group,
                device=self.device,
            )

```
**EN:** This callable implements `GroupCoordinator.__init__`. It takes `group_ranks`, `local_rank`, `torch_distributed_backend`, `use_pynccl` and mainly initializes instance state and defaults. This chunk is part 3 of 4 for the same logical block. In this range it sets up imports and shared symbols; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.__init__`。它接收 `group_ranks`, `local_rank`, `torch_distributed_backend`, `use_pynccl`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 3/4 部分。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；协调分布式通信。

### Lines 420-453: Method GroupCoordinator.__init__ (part 4/4)
```python
        # Create communicator for other hardware backends
        from sglang.srt.distributed.device_communicators.hpu_communicator import (
            HpuCommunicator,
        )
        from sglang.srt.distributed.device_communicators.npu_communicator import (
            NpuCommunicator,
        )
        from sglang.srt.distributed.device_communicators.xpu_communicator import (
            XpuCommunicator,
        )

        self.hpu_communicator: Optional[HpuCommunicator] = None
        if use_hpu_communicator and self.world_size > 1:
            self.hpu_communicator = HpuCommunicator(group=self.device_group)

        self.xpu_communicator: Optional[XpuCommunicator] = None
        if use_xpu_communicator and self.world_size > 1:
            self.xpu_communicator = XpuCommunicator(group=self.device_group)

        self.npu_communicator: Optional[NpuCommunicator] = None
        if use_npu_communicator and self.world_size > 1:
            self.npu_communicator = NpuCommunicator(group=self.device_group)

        # Create message queue
        from sglang.srt.distributed.device_communicators.shm_broadcast import (
            MessageQueue,
        )

        self.mq_broadcaster: Optional[MessageQueue] = None
        if use_message_queue_broadcaster and self.world_size > 1 and not recovered_rank:
            # Recovered ranks create their mq_broadcaster in elastic_ep.py
            self.mq_broadcaster = MessageQueue.create_from_process_group(
                self.cpu_group, 1 << 22, 6
            )
```
**EN:** This callable implements `GroupCoordinator.__init__`. It takes `group_ranks`, `local_rank`, `torch_distributed_backend`, `use_pynccl` and mainly initializes instance state and defaults. This chunk is part 4 of 4 for the same logical block. In this range it sets up imports and shared symbols; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.__init__`。它接收 `group_ranks`, `local_rank`, `torch_distributed_backend`, `use_pynccl`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 4/4 部分。 在这一范围内，它会建立导入关系并准备共享符号；协调分布式通信。

### Lines 455-460: Method GroupCoordinator.__repr__
```python
    def __repr__(self):
        return (
            f"ranks={self.ranks} rank={self.rank} local_rank={self.local_rank} use_pynccl={self.use_pynccl} "
            f"device_group={self.device_group} cpu_group={self.cpu_group} unique_name={self.unique_name} "
            f"world_size={self.world_size} rank_in_group={self.rank_in_group}"
        )
```
**EN:** This callable implements `GroupCoordinator.__repr__` and mainly implements repr.
**CN:** 这一可调用对象实现了 `GroupCoordinator.__repr__`，主要用于实现 repr 相关逻辑。

### Lines 462-465: Method GroupCoordinator.first_rank
```python
    @property
    def first_rank(self):
        """Return the global rank of the first process in the group"""
        return self.ranks[0]
```
**EN:** This callable implements `GroupCoordinator.first_rank` and mainly implements first rank. The docstring states: "Return the global rank of the first process in the group"
**CN:** 这一可调用对象实现了 `GroupCoordinator.first_rank`，主要用于实现 first rank 相关逻辑。

### Lines 467-470: Method GroupCoordinator.last_rank
```python
    @property
    def last_rank(self):
        """Return the global rank of the last process in the group"""
        return self.ranks[-1]
```
**EN:** This callable implements `GroupCoordinator.last_rank` and mainly implements last rank. The docstring states: "Return the global rank of the last process in the group"
**CN:** 这一可调用对象实现了 `GroupCoordinator.last_rank`，主要用于实现 last rank 相关逻辑。

### Lines 472-475: Method GroupCoordinator.is_first_rank
```python
    @property
    def is_first_rank(self):
        """Return whether the caller is the first process in the group"""
        return self.rank == self.first_rank
```
**EN:** This callable implements `GroupCoordinator.is_first_rank` and mainly implements is first rank. The docstring states: "Return whether the caller is the first process in the group"
**CN:** 这一可调用对象实现了 `GroupCoordinator.is_first_rank`，主要用于实现 is first rank 相关逻辑。

### Lines 477-480: Method GroupCoordinator.is_last_rank
```python
    @property
    def is_last_rank(self):
        """Return whether the caller is the last process in the group"""
        return self.rank == self.last_rank
```
**EN:** This callable implements `GroupCoordinator.is_last_rank` and mainly implements is last rank. The docstring states: "Return whether the caller is the last process in the group"
**CN:** 这一可调用对象实现了 `GroupCoordinator.is_last_rank`，主要用于实现 is last rank 相关逻辑。

### Lines 482-487: Method GroupCoordinator.next_rank
```python
    @property
    def next_rank(self):
        """Return the global rank of the process that follows the caller"""
        rank_in_group = self.rank_in_group
        world_size = self.world_size
        return self.ranks[(rank_in_group + 1) % world_size]
```
**EN:** This callable implements `GroupCoordinator.next_rank` and mainly implements next rank. The docstring states: "Return the global rank of the process that follows the caller"
**CN:** 这一可调用对象实现了 `GroupCoordinator.next_rank`，主要用于实现 next rank 相关逻辑。

### Lines 489-494: Method GroupCoordinator.prev_rank
```python
    @property
    def prev_rank(self):
        """Return the global rank of the process that precedes the caller"""
        rank_in_group = self.rank_in_group
        world_size = self.world_size
        return self.ranks[(rank_in_group - 1) % world_size]
```
**EN:** This callable implements `GroupCoordinator.prev_rank` and mainly implements prev rank. The docstring states: "Return the global rank of the process that precedes the caller"
**CN:** 这一可调用对象实现了 `GroupCoordinator.prev_rank`，主要用于实现 prev rank 相关逻辑。

### Lines 496-555: Method GroupCoordinator.graph_capture (part 1/2)
```python
    @contextmanager
    def graph_capture(
        self,
        graph_capture_context: Optional[GraphCaptureContext] = None,
        stream: Optional[torch.cuda.Stream] = None,
    ):
        if graph_capture_context is None:
            if stream is None:
                stream = self.device_module.Stream()
            graph_capture_context = GraphCaptureContext(stream)
        else:
            stream = graph_capture_context.stream
        # We don't need the context of custom quick allreduce because the ipc access
        # is already collected in init() and we can capture the quick allreduce directly.
        ca_comm = self.ca_comm
        maybe_ca_context = nullcontext() if ca_comm is None else ca_comm.capture()

        # ensure all initialization operations complete before attempting to
        # capture the graph on another stream
        curr_stream = get_current_device_stream_fast()
        if curr_stream != stream:
            stream.wait_stream(curr_stream)

        with self.device_module.stream(stream), maybe_ca_context:
            # In graph mode, we have to be very careful about the collective
            # operations. The current status is:
            #     allreduce \ Mode   |  Eager  |  Graph  |
            # --------------------------------------------
            # quick allreduce        | enabled | enabled |
            # custom allreduce       | enabled | enabled |
            # PyNccl                 | disabled| enabled |
            # PyMscclpp              | disabled| enabled |
            # TorchSymmMem           | disabled| enabled |
            # torch.distributed      | enabled | disabled|
            #
            # Note: When custom quick allreduce is enabled, a runtime check
            #  will be performed. If the tensor size is too small, it will
            #  automatically fall back to the next available option.
            # Note that custom allreduce will have a runtime check, if the
            #  tensor size is too large, it will fallback to the next
            #  available option.
            # Note that the PyMsccl needs to register the tensor in ahead,
            #  which will introduce large overhead in the eager case,
            #  therefore it is only supported in the graph case.
            # In summary: We select the appropriate allreduce method for
            #  each mode based on the algorithm order in the table and
            #  their usage conditions.
            pynccl_comm = self.pynccl_comm
            maybe_pynccl_context: Any
            if not pynccl_comm:
                maybe_pynccl_context = nullcontext()
            else:
                maybe_pynccl_context = pynccl_comm.change_state(enable=True)

            pymscclpp_comm = self.pymscclpp_comm
            maybe_pymscclpp_context: Any
            if not pymscclpp_comm:
                maybe_pymscclpp_context = nullcontext()
            else:
                maybe_pymscclpp_context = pymscclpp_comm.change_state(enable=True)
```
**EN:** This callable implements `GroupCoordinator.graph_capture`. It takes `graph_capture_context`, `stream` and mainly implements graph capture. This chunk is part 1 of 2 for the same logical block. In this range it coordinates distributed communication; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `GroupCoordinator.graph_capture`。它接收 `graph_capture_context`, `stream`，主要用于实现 graph capture 相关逻辑。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会协调分布式通信；管理图捕获或回放逻辑。

### Lines 556-557: Method GroupCoordinator.graph_capture (part 2/2)
```python
            with maybe_pynccl_context, maybe_pymscclpp_context:
                yield graph_capture_context
```
**EN:** This callable implements `GroupCoordinator.graph_capture`. It takes `graph_capture_context`, `stream` and mainly implements graph capture. This chunk is part 2 of 2 for the same logical block. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `GroupCoordinator.graph_capture`。它接收 `graph_capture_context`, `stream`，主要用于实现 graph capture 相关逻辑。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 559-618: Method GroupCoordinator.all_reduce (part 1/2)
```python
    def all_reduce(self, input_: torch.Tensor) -> torch.Tensor:
        """
        User-facing all-reduce function before we actually call the
        all-reduce operation.

        We need this because Dynamo does not support passing an arbitrary
        object (`self` in this case) to a custom op. We need to pass the
         group name as a string, and then look up the group coordinator from
         the group name, dispatch the all-reduce operation to the group
         coordinator.

        In addition, PyTorch custom ops do not support mutation or returning
        a new tensor in the same op. So we need to figure out if the op is
        in-place or out-of-place ahead of time.
        """
        # Bypass the function if we are using only 1 GPU.
        if self.world_size == 1:
            return input_

        if input_.is_cpu:
            if is_shm_available(input_.dtype, self.world_size, self.local_size):
                torch.ops.sgl_kernel.shm_allreduce(input_, REDUCE_OP_SUM)
            else:
                torch.distributed.all_reduce(input_, group=self.device_group)
            return input_

        if self.hpu_communicator is not None and not self.hpu_communicator.disabled:
            return self.hpu_communicator.all_reduce(input_)

        if self.xpu_communicator is not None and not self.xpu_communicator.disabled:
            return self.xpu_communicator.all_reduce(input_)

        if self.npu_communicator is not None and not self.npu_communicator.disabled:
            return self.npu_communicator.all_reduce(input_)

        if self.pynccl_comm is not None and self.is_symmetric_memory_enabled():
            self.debug_check_symmetric_mempool(self, {"input": input_}, "all_reduce")
            with self.pynccl_comm.change_state(enable=True):
                self.pynccl_comm.all_reduce(input_)
                return input_

        outplace_all_reduce_method = None
        if (
            self.ca_comm is not None
            and not self.ca_comm.disabled
            and self.ca_comm.should_custom_ar(input_)
        ):
            outplace_all_reduce_method = "ca"
        elif (
            self.qr_comm is not None
            and not self.qr_comm.disabled
            and self.qr_comm.should_quick_allreduce(input_)
        ):
            outplace_all_reduce_method = "qr"
        elif (
            self.pymscclpp_comm is not None
            and not self.pymscclpp_comm.disabled
            and self.pymscclpp_comm.should_mscclpp_allreduce(input_)
        ):
            outplace_all_reduce_method = "pymscclpp"
```
**EN:** This callable implements `GroupCoordinator.all_reduce`. It takes `input_` and mainly reduces or aggregates values. The docstring states: "User-facing all-reduce function before we actually call the all-reduce operation." This chunk is part 1 of 2 for the same logical block. In this range it coordinates distributed communication; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `GroupCoordinator.all_reduce`。它接收 `input_`，主要用于归约或聚合数值。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会协调分布式通信；记录指标或追踪信号。

### Lines 619-636: Method GroupCoordinator.all_reduce (part 2/2)
```python
        elif (
            self.torch_symm_mem_comm is not None
            and not self.torch_symm_mem_comm.disabled
            and self.torch_symm_mem_comm.should_torch_symm_mem_allreduce(input_)
        ):
            outplace_all_reduce_method = "torch_symm_mem"
        elif is_in_piecewise_cuda_graph() and self.pynccl_comm is not None:
            # For piecewise cuda graph, we use pynccl outplace allreduce
            outplace_all_reduce_method = "pynccl"
        if outplace_all_reduce_method is not None:
            return outplace_all_reduce(
                input_,
                group_name=self.unique_name,
                outplace_all_reduce_method=outplace_all_reduce_method,
            )
        else:
            inplace_all_reduce(input_, group_name=self.unique_name)
            return input_
```
**EN:** This callable implements `GroupCoordinator.all_reduce`. It takes `input_` and mainly reduces or aggregates values. The docstring states: "User-facing all-reduce function before we actually call the all-reduce operation." This chunk is part 2 of 2 for the same logical block. In this range it coordinates distributed communication; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `GroupCoordinator.all_reduce`。它接收 `input_`，主要用于归约或聚合数值。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会协调分布式通信；管理图捕获或回放逻辑。

### Lines 638-650: Method GroupCoordinator.quant_all_reduce
```python
    def quant_all_reduce(self, input_: torch.Tensor) -> torch.Tensor:
        """
        User-facing quant-all-reduce function similar to all-reduce. (NPU support only)
        """
        # Bypass the function if we are using only 1 GPU.
        if self.world_size == 1:
            return input_

        if self.npu_communicator is not None and not self.npu_communicator.disabled:
            return self.npu_communicator.quant_all_reduce(input_)
        else:
            inplace_all_reduce(input_, group_name=self.unique_name)
            return input_
```
**EN:** This callable implements `GroupCoordinator.quant_all_reduce`. It takes `input_` and mainly reduces or aggregates values. The docstring states: "User-facing quant-all-reduce function similar to all-reduce." In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.quant_all_reduce`。它接收 `input_`，主要用于归约或聚合数值。 在这一范围内，它会协调分布式通信。

### Lines 652-696: Method GroupCoordinator.fused_allreduce_rmsnorm
```python
    def fused_allreduce_rmsnorm(
        self,
        input_: torch.Tensor,
        residual_inp_: torch.Tensor,
        weight_: torch.Tensor,
        eps: float,
    ) -> Optional[Tuple[torch.Tensor, torch.Tensor]]:
        """Attempt fused all-reduce + RMSNorm via custom all-reduce communicator. ROCm/HIP Only"""
        ca_comm = self.ca_comm
        if ca_comm is None or getattr(ca_comm, "disabled", True):
            return None

        # Prefer communicator-native fused API when provided.
        if hasattr(ca_comm, "fused_allreduce_rmsnorm"):
            try:
                return ca_comm.fused_allreduce_rmsnorm(
                    input_, residual_inp_, weight_, eps
                )
            except Exception:
                # Fall back to custom_fused_ar_rms path below.
                pass

        if not hasattr(ca_comm, "custom_fused_ar_rms"):
            return None

        # 1-stage vs 2-stage selection for fused AR+RMSNorm:
        # The 1-stage kernel launches one block per token and is capped at
        # 80 tokens (kMaxBlocks).  Guard with a byte threshold so large
        # prefill batches fall through to the 2-stage kernel instead of
        # hitting a runtime error.  AITER's C++ dispatch already gates
        # which hidden_dims have valid 1-stage support.
        if envs.SGLANG_USE_1STAGE_ALLREDUCE.is_set():
            use_1stage_ar = envs.SGLANG_USE_1STAGE_ALLREDUCE.get()
        else:
            total_bytes = input_.numel() * input_.element_size()
            use_1stage_ar = total_bytes <= 128 * 1024

        fused_outputs = ca_comm.custom_fused_ar_rms(
            input_,
            residual_inp_,
            weight_,
            eps,
            use_1stage_ar,
        )
        return fused_outputs
```
**EN:** This callable implements `GroupCoordinator.fused_allreduce_rmsnorm`. It takes `input_`, `residual_inp_`, `weight_`, `eps` and mainly reduces or aggregates values. The docstring states: "Attempt fused all-reduce + RMSNorm via custom all-reduce communicator." In this range it reads environment-driven configuration; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `GroupCoordinator.fused_allreduce_rmsnorm`。它接收 `input_`, `residual_inp_`, `weight_`, `eps`，主要用于归约或聚合数值。 在这一范围内，它会读取环境变量驱动的配置；管理模型权重或检查点。

### Lines 698-723: Method GroupCoordinator._all_reduce_out_place
```python
    def _all_reduce_out_place(
        self, input_: torch.Tensor, outplace_all_reduce_method: str
    ) -> torch.Tensor:
        ca_comm = self.ca_comm
        qr_comm = self.qr_comm
        pymscclpp_comm = self.pymscclpp_comm
        torch_symm_mem_comm = self.torch_symm_mem_comm
        pynccl_comm = self.pynccl_comm
        assert any([qr_comm, ca_comm, pymscclpp_comm, torch_symm_mem_comm, pynccl_comm])
        if outplace_all_reduce_method == "ca":
            assert not ca_comm.disabled
            out = ca_comm.custom_all_reduce(input_)
        elif outplace_all_reduce_method == "qr":
            assert not qr_comm.disabled
            out = qr_comm.quick_all_reduce(input_)
        elif outplace_all_reduce_method == "torch_symm_mem":
            assert not torch_symm_mem_comm.disabled
            out = torch_symm_mem_comm.all_reduce(input_)
        elif outplace_all_reduce_method == "pymscclpp":
            assert not pymscclpp_comm.disabled
            out = pymscclpp_comm.all_reduce(input_)
        elif outplace_all_reduce_method == "pynccl":
            with pynccl_comm.change_state(enable=True):
                out = pynccl_comm.outplace_all_reduce(input_)
        assert out is not None
        return out
```
**EN:** This callable implements `GroupCoordinator._all_reduce_out_place`. It takes `input_`, `outplace_all_reduce_method` and mainly reduces or aggregates values. In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator._all_reduce_out_place`。它接收 `input_`, `outplace_all_reduce_method`，主要用于归约或聚合数值。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 725-733: Method GroupCoordinator._all_reduce_in_place
```python
    def _all_reduce_in_place(self, input_: torch.Tensor) -> None:
        pynccl_comm = self.pynccl_comm
        torch_symm_mem_comm = self.torch_symm_mem_comm
        if pynccl_comm is not None and not pynccl_comm.disabled:
            pynccl_comm.all_reduce(input_)
        elif torch_symm_mem_comm is not None and not torch_symm_mem_comm.disabled:
            torch_symm_mem_comm.all_reduce(input_)
        else:
            torch.distributed.all_reduce(input_, group=self.device_group)
```
**EN:** This callable implements `GroupCoordinator._all_reduce_in_place`. It takes `input_` and mainly reduces or aggregates values. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator._all_reduce_in_place`。它接收 `input_`，主要用于归约或聚合数值。 在这一范围内，它会协调分布式通信。

### Lines 735-753: Method GroupCoordinator._reduce_scatter_tensor
```python
    def _reduce_scatter_tensor(
        self,
        output: torch.Tensor,
        input: torch.Tensor,
    ) -> torch.Tensor:
        pynccl_comm = self.pynccl_comm
        if pynccl_comm is not None and (
            not pynccl_comm.disabled or self.is_symmetric_memory_enabled()
        ):
            self.debug_check_symmetric_mempool(
                self, {"output": output, "input": input}, "reduce_scatter_tensor"
            )
            with pynccl_comm.change_state(enable=True):
                pynccl_comm.reduce_scatter(output, input)
        else:
            torch.distributed.reduce_scatter_tensor(
                output, input, group=self.device_group
            )
        return output
```
**EN:** This callable implements `GroupCoordinator._reduce_scatter_tensor`. It takes `output`, `input` and mainly reduces or aggregates values. In this range it coordinates distributed communication; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `GroupCoordinator._reduce_scatter_tensor`。它接收 `output`, `input`，主要用于归约或聚合数值。 在这一范围内，它会协调分布式通信；记录指标或追踪信号。

### Lines 755-759: Method GroupCoordinator.reduce_scatter_tensor
```python
    def reduce_scatter_tensor(self, output: torch.Tensor, input: torch.Tensor):
        if _is_npu:
            self._reduce_scatter_tensor(output, input)
        else:
            reg_reduce_scatter_tensor(output, input, group_name=self.unique_name)
```
**EN:** This callable implements `GroupCoordinator.reduce_scatter_tensor`. It takes `output`, `input` and mainly reduces or aggregates values.
**CN:** 这一可调用对象实现了 `GroupCoordinator.reduce_scatter_tensor`。它接收 `output`, `input`，主要用于归约或聚合数值。

### Lines 761-768: Method GroupCoordinator.reduce_scatter
```python
    def reduce_scatter(
        self,
        output: torch.Tensor,
        input_list: List[torch.Tensor],
    ) -> None:
        # TODO(ch-wan): support other backends
        torch.distributed.reduce_scatter(output, input_list, group=self.device_group)
        return output
```
**EN:** This callable implements `GroupCoordinator.reduce_scatter`. It takes `output`, `input_list` and mainly reduces or aggregates values. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.reduce_scatter`。它接收 `output`, `input_list`，主要用于归约或聚合数值。 在这一范围内，它会协调分布式通信。

### Lines 770-801: Method GroupCoordinator.reduce_scatterv
```python
    def reduce_scatterv(
        self,
        input_: torch.Tensor,
        output: Optional[torch.Tensor] = None,
        sizes: Optional[List[int]] = None,
    ) -> torch.Tensor:
        world_size = self.world_size
        pynccl_comm = self.pynccl_comm

        with pynccl_comm.change_state(enable=True):
            assert (
                pynccl_comm is not None and not pynccl_comm.disabled
            ), "pynccl is required for reduce_scatterv"

            if sizes is not None:
                assert len(sizes) == world_size
                assert input_.shape[0] == sum(sizes)
                chunk_size = sizes[self.rank_in_group]
            else:
                assert input_.shape[0] % world_size == 0
                chunk_size = input_.shape[0] // world_size
            output_shape = (chunk_size,) + input_.shape[1:]

            if output is None:
                output = torch.empty(
                    output_shape, dtype=input_.dtype, device=input_.device
                )
            else:
                assert output.shape == output_shape

            pynccl_comm.reduce_scatter(output, input_, sizes=sizes)
            return output
```
**EN:** This callable implements `GroupCoordinator.reduce_scatterv`. It takes `input_`, `output`, `sizes` and mainly reduces or aggregates values. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `GroupCoordinator.reduce_scatterv`。它接收 `input_`, `output`, `sizes`，主要用于归约或聚合数值。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 803-816: Method GroupCoordinator._all_gather_into_tensor
```python
    def _all_gather_into_tensor(self, output: torch.Tensor, input: torch.Tensor):
        pynccl_comm = self.pynccl_comm
        if pynccl_comm is not None and (
            not pynccl_comm.disabled or self.is_symmetric_memory_enabled()
        ):
            self.debug_check_symmetric_mempool(
                self, {"output": output}, "all_gather_into_tensor"
            )
            with pynccl_comm.change_state(enable=True):
                pynccl_comm.all_gather(output, input)
        else:
            torch.distributed.all_gather_into_tensor(
                output, input, group=self.device_group
            )
```
**EN:** This callable implements `GroupCoordinator._all_gather_into_tensor`. It takes `output`, `input` and mainly converts data into another representation. In this range it coordinates distributed communication; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `GroupCoordinator._all_gather_into_tensor`。它接收 `output`, `input`，主要用于将数据转换为另一种表示。 在这一范围内，它会协调分布式通信；记录指标或追踪信号。

### Lines 818-822: Method GroupCoordinator.all_gather_into_tensor
```python
    def all_gather_into_tensor(self, output: torch.Tensor, input: torch.Tensor):
        if _is_npu or _is_xpu:
            self._all_gather_into_tensor(output, input)
        else:
            reg_all_gather_into_tensor(output, input, group_name=self.unique_name)
```
**EN:** This callable implements `GroupCoordinator.all_gather_into_tensor`. It takes `output`, `input` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `GroupCoordinator.all_gather_into_tensor`。它接收 `output`, `input`，主要用于将数据转换为另一种表示。

### Lines 824-837: Method GroupCoordinator.cp_all_gather_into_tensor_async
```python
    def cp_all_gather_into_tensor_async(
        self, output: torch.Tensor, input: torch.Tensor, stream: torch.cuda.Stream
    ):
        """
        Implement an asynchronous `allgather` operation on a specified stream.
        (the default `torch.distributed.all_gather_into_tensor` will trigger event synchronization),
        eliminating the CPU-side launch-kernel blocking issue caused by synchronization problems.
        The specific implementation uses the interface provided by pynccl to remove the synchronization logic of events.
        """
        pynccl_comm = self.pynccl_comm
        if pynccl_comm is None or pynccl_comm.disabled:
            self.all_gather_into_tensor(output, input)
        else:
            pynccl_comm.cp_all_gather_into_tensor(output, input, stream=stream)
```
**EN:** This callable implements `GroupCoordinator.cp_all_gather_into_tensor_async`. It takes `output`, `input`, `stream` and mainly converts data into another representation. The docstring states: "Implement an asynchronous `allgather` operation on a specified stream." In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.cp_all_gather_into_tensor_async`。它接收 `output`, `input`, `stream`，主要用于将数据转换为另一种表示。 在这一范围内，它会协调分布式通信。

### Lines 839-898: Method GroupCoordinator.all_gather (part 1/2)
```python
    def all_gather(
        self,
        input_: torch.Tensor,
        dim: int = -1,
        output_tensor_list: Optional[List[torch.Tensor]] = None,
    ) -> torch.Tensor:
        world_size = self.world_size
        # Bypass the function if we are using only 1 GPU.
        if world_size == 1:
            if output_tensor_list is not None:
                logger.warning(
                    "Performing in-place all-gather with a group size of 1. "
                    "This may be unnecessary; consider bypassing it for better efficiency."
                )
                output_tensor_list[0].copy_(input_)
                return None
            else:
                return input_

        if output_tensor_list is not None:
            # TODO(ch-wan): support other backends
            return torch.distributed.all_gather(
                output_tensor_list, input_, group=self.device_group
            )

        assert (
            -input_.dim() <= dim < input_.dim()
        ), f"Invalid dim ({dim}) for input tensor with shape {input_.size()}"

        # For HPUs, use HPU communicator.
        hpu_comm = self.hpu_communicator
        if hpu_comm is not None and not hpu_comm.disabled:
            return hpu_comm.all_gather(input_, dim)

        # For NPUs, use NPU communicator.
        npu_comm = self.npu_communicator
        if npu_comm is not None and not npu_comm.disabled:
            return npu_comm.all_gather(input_, dim)

        if dim < 0:
            # Convert negative dim to positive.
            dim += input_.dim()
        input_size = input_.size()
        # NOTE: we have to use concat-style all-gather here,
        # stack-style all-gather has compatibility issues with
        # torch.compile . see https://github.com/pytorch/pytorch/issues/138795
        output_size = (input_size[0] * world_size,) + input_size[1:]
        # Allocate output tensor.
        with self.use_symmetric_memory(
            self, disabled=not self.is_allocation_symmetric()
        ):
            output_tensor = torch.empty(
                output_size, dtype=input_.dtype, device=input_.device
            )

        # All-gather.
        if input_.is_cpu:
            if is_shm_available(input_.dtype, self.world_size, self.local_size):
                return torch.ops.sgl_kernel.shm_allgather(input_, dim)
            else:
```
**EN:** This callable implements `GroupCoordinator.all_gather`. It takes `input_`, `dim`, `output_tensor_list` and mainly implements all gather. This chunk is part 1 of 2 for the same logical block. In this range it performs defensive checks on invalid state; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.all_gather`。它接收 `input_`, `dim`, `output_tensor_list`，主要用于实现 all gather 相关逻辑。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会对非法状态执行防御性检查；输出日志以便诊断；协调分布式通信。

### Lines 899-911: Method GroupCoordinator.all_gather (part 2/2)
```python
                torch.distributed.all_gather_into_tensor(
                    output_tensor, input_, group=self.device_group
                )
        else:
            self.all_gather_into_tensor(output_tensor, input_)

        # Reshape
        output_tensor = output_tensor.reshape((world_size,) + input_size)
        output_tensor = output_tensor.movedim(0, dim)
        output_tensor = output_tensor.reshape(
            input_size[:dim] + (world_size * input_size[dim],) + input_size[dim + 1 :]
        )
        return output_tensor
```
**EN:** This callable implements `GroupCoordinator.all_gather`. It takes `input_`, `dim`, `output_tensor_list` and mainly implements all gather. This chunk is part 2 of 2 for the same logical block. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.all_gather`。它接收 `input_`, `dim`, `output_tensor_list`，主要用于实现 all gather 相关逻辑。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会协调分布式通信。

### Lines 913-965: Method GroupCoordinator.all_gatherv
```python
    def all_gatherv(
        self,
        input_: Union[torch.Tensor, List[torch.Tensor]],
        sizes: Optional[List[int]] = None,
    ) -> Union[torch.Tensor, List[torch.Tensor]]:
        """
        Supports varying sizes per rank and input tensor list.
        `sizes`: a list of len(world_size) with the number of items per rank to gather.
        """
        world_size = self.world_size
        pynccl_comm = self.pynccl_comm

        with pynccl_comm.change_state(enable=True):
            assert (
                pynccl_comm is not None and not pynccl_comm.disabled
            ), "pynccl is required for all_gatherv"

            def _all_gather_allocate_output(
                input_: torch.Tensor, sizes: Optional[List[int]] = None
            ):
                input_size = input_.size()
                if sizes is not None:
                    assert len(sizes) == world_size
                    assert input_.shape[0] == sizes[self.rank_in_group]
                    output_size = (sum(sizes),) + input_size[1:]
                    # 'sizes' is not needed if all inputs in the same group have the same shape
                    if all(s == sizes[0] for s in sizes):
                        sizes = None
                else:
                    output_size = (input_size[0] * world_size,) + input_size[1:]
                # Allocate output tensor.
                with self.use_symmetric_memory(self, disabled=sizes is not None):
                    output_tensor = torch.empty(
                        output_size, dtype=input_.dtype, device=input_.device
                    )
                return output_tensor, sizes

            if isinstance(input_, torch.Tensor):
                input_ = [input_]

            output_list = []
            size_list = []
            for inp in input_:
                output_tensor, s = _all_gather_allocate_output(inp, sizes=sizes)
                output_list.append(output_tensor)
                size_list.append(s)

            pynccl_comm.group_start()
            for i, inp in enumerate(input_):
                pynccl_comm.all_gather(output_list[i], inp, sizes=size_list[i])
            pynccl_comm.group_end()

            return output_list
```
**EN:** This callable implements `GroupCoordinator.all_gatherv`. It takes `input_`, `sizes` and mainly implements all gatherv. The docstring states: "Supports varying sizes per rank and input tensor list." In this range it performs defensive checks on invalid state; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `GroupCoordinator.all_gatherv`。它接收 `input_`, `sizes`，主要用于实现 all gatherv 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；记录指标或追踪信号。

### Lines 967-1000: Method GroupCoordinator.gather
```python
    def gather(
        self, input_: torch.Tensor, dst: int = 0, dim: int = -1
    ) -> Optional[torch.Tensor]:
        """
        NOTE: We assume that the input tensor is on the same device across
        all the ranks.
        NOTE: `dst` is the local rank of the destination rank.
        """
        world_size = self.world_size
        # Bypass the function if we are using only 1 GPU.
        if world_size == 1:
            return input_
        assert (
            -input_.dim() <= dim < input_.dim()
        ), f"Invalid dim ({dim}) for input tensor with shape {input_.size()}"
        if dim < 0:
            # Convert negative dim to positive.
            dim += input_.dim()
        if self.xpu_communicator is not None and not self.xpu_communicator.disabled:
            return self.xpu_communicator.gather(input_, self.rank_in_group, dst, dim)
        # Allocate output tensor.
        if self.rank_in_group == dst:
            gather_list = [torch.empty_like(input_) for _ in range(world_size)]
        else:
            gather_list = None
        # Gather.
        torch.distributed.gather(
            input_, gather_list, dst=self.ranks[dst], group=self.device_group
        )
        if self.rank_in_group == dst:
            output_tensor = torch.cat(gather_list, dim=dim)
        else:
            output_tensor = None
        return output_tensor
```
**EN:** This callable implements `GroupCoordinator.gather`. It takes `input_`, `dst`, `dim` and mainly implements gather. The docstring states: "NOTE: We assume that the input tensor is on the same device across all the ranks." In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.gather`。它接收 `input_`, `dst`, `dim`，主要用于实现 gather 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 1002-1015: Method GroupCoordinator.broadcast
```python
    def broadcast(self, input_: torch.Tensor, src: int = 0):
        """Broadcast the input tensor.
        NOTE: `src` is the local rank of the source rank.
        """
        assert src < self.world_size, f"Invalid src rank ({src})"

        # Bypass the function if we are using only 1 GPU.
        if self.world_size == 1:
            return input_
        # Broadcast.
        torch.distributed.broadcast(
            input_, src=self.ranks[src], group=self.device_group
        )
        return input_
```
**EN:** This callable implements `GroupCoordinator.broadcast`. It takes `input_`, `src` and mainly implements broadcast. The docstring states: "Broadcast the input tensor." In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.broadcast`。它接收 `input_`, `src`，主要用于实现 broadcast 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 1017-1039: Method GroupCoordinator.broadcast_object
```python
    def broadcast_object(self, obj: Optional[Any] = None, src: int = 0):
        """Broadcast the input object.
        NOTE: `src` is the local rank of the source rank.
        """
        assert src < self.world_size, f"Invalid src rank ({src})"

        # Bypass the function if we are using only 1 GPU.
        if self.world_size == 1:
            return obj
        if self.mq_broadcaster is not None:
            assert src == 0, "Message queue broadcaster only supports src=0"
            return self.mq_broadcaster.broadcast_object(obj)
        if self.rank_in_group == src:
            torch.distributed.broadcast_object_list(
                [obj], src=self.ranks[src], group=self.cpu_group
            )
            return obj
        else:
            recv = [None]
            torch.distributed.broadcast_object_list(
                recv, src=self.ranks[src], group=self.cpu_group
            )
            return recv[0]
```
**EN:** This callable implements `GroupCoordinator.broadcast_object`. It takes `obj`, `src` and mainly implements broadcast object. The docstring states: "Broadcast the input object." In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.broadcast_object`。它接收 `obj`, `src`，主要用于实现 broadcast object 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 1041-1056: Method GroupCoordinator.broadcast_object_list
```python
    def broadcast_object_list(
        self, obj_list: List[Any], src: int = 0, group: Optional[ProcessGroup] = None
    ):
        """Broadcast the input object list.
        NOTE: `src` is the local rank of the source rank.
        """
        assert src < self.world_size, f"Invalid src rank ({src})"

        # Bypass the function if we are using only 1 GPU.
        if self.world_size == 1:
            return obj_list
        # Broadcast.
        torch.distributed.broadcast_object_list(
            obj_list, src=self.ranks[src], group=self.device_group
        )
        return obj_list
```
**EN:** This callable implements `GroupCoordinator.broadcast_object_list`. It takes `obj_list`, `src`, `group` and mainly implements broadcast object list. The docstring states: "Broadcast the input object list." In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.broadcast_object_list`。它接收 `obj_list`, `src`, `group`，主要用于实现 broadcast object list 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 1058-1061: Method GroupCoordinator.all_gather_object
```python
    def all_gather_object(self, obj: Any) -> List[Any]:
        objs = [None] * self.world_size
        torch.distributed.all_gather_object(objs, obj, group=self.cpu_group)
        return objs
```
**EN:** This callable implements `GroupCoordinator.all_gather_object`. It takes `obj` and mainly implements all gather object. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.all_gather_object`。它接收 `obj`，主要用于实现 all gather object 相关逻辑。 在这一范围内，它会协调分布式通信。

### Lines 1063-1110: Method GroupCoordinator.send_object
```python
    def send_object(
        self,
        obj: Any,
        dst: int,
        async_send: bool = False,
    ) -> List[P2PWork]:
        """
        Send the input object list to the destination rank.
        This function uses the CPU group for all communications.

        TODO: If you want to use GPU communication, please add a new argument (e.g., data_group, group),
        use other functions (e.g., send), or implement a new function (e.g., send_object_device).

        NOTE: `dst` is the local rank of the destination rank.
        """

        assert dst < self.world_size, f"Invalid dst rank ({dst})"
        assert dst != self.rank_in_group, (
            "Invalid destination rank. Destination rank is the same "
            "as the current rank."
        )
        send_func = torch.distributed.isend if async_send else torch.distributed.send

        # Serialize object to tensor and get the size as well
        object_tensor = torch.frombuffer(pickle.dumps(obj), dtype=torch.uint8)
        size_tensor = torch.tensor(
            [object_tensor.numel()], dtype=torch.long, device="cpu"
        )

        # Send object size
        p2p_work = []
        size_work = send_func(
            size_tensor,
            self.ranks[dst],
            group=self.cpu_group,
        )
        if async_send:
            p2p_work.append(P2PWork(size_work, size_tensor))

        object_work = send_func(
            object_tensor,
            self.ranks[dst],
            group=self.cpu_group,
        )
        if async_send:
            p2p_work.append(P2PWork(object_work, object_tensor))

        return p2p_work
```
**EN:** This callable implements `GroupCoordinator.send_object`. It takes `obj`, `dst`, `async_send` and mainly sends data to another component. The docstring states: "Send the input object list to the destination rank." In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.send_object`。它接收 `obj`, `dst`, `async_send`，主要用于向其他组件发送数据。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 1112-1146: Method GroupCoordinator.recv_object
```python
    def recv_object(
        self,
        src: int,
    ) -> Any:
        """Receive the input object list from the source rank."""
        """NOTE: `src` is the local rank of the source rank."""

        assert src < self.world_size, f"Invalid src rank ({src})"
        assert (
            src != self.rank_in_group
        ), "Invalid source rank. Source rank is the same as the current rank."

        size_tensor = torch.empty(1, dtype=torch.long, device="cpu")

        # Receive object size
        # We have to use irecv here to make it work for both isend and send.
        work = torch.distributed.irecv(
            size_tensor, src=self.ranks[src], group=self.cpu_group
        )
        work.wait()

        # Tensor to receive serialized objects into.
        object_tensor: Any = torch.empty(  # type: ignore[call-overload]
            size_tensor.item(),  # type: ignore[arg-type]
            dtype=torch.uint8,
            device="cpu",
        )

        work = torch.distributed.irecv(
            object_tensor, src=self.ranks[src], group=self.cpu_group
        )
        work.wait()

        obj = pickle.loads(object_tensor.numpy())
        return obj
```
**EN:** This callable implements `GroupCoordinator.recv_object`. It takes `src` and mainly receives data from another component. The docstring states: "Receive the input object list from the source rank." In this range it sets up imports and shared symbols; performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.recv_object`。它接收 `src`，主要用于从其他组件接收数据。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；协调分布式通信。

### Lines 1148-1207: Method GroupCoordinator.broadcast_tensor_dict (part 1/2)
```python
    def broadcast_tensor_dict(
        self,
        tensor_dict: Optional[Dict[str, Union[torch.Tensor, Any]]] = None,
        src: int = 0,
        group: Optional[ProcessGroup] = None,
        metadata_group: Optional[ProcessGroup] = None,
    ) -> Optional[Dict[str, Union[torch.Tensor, Any]]]:
        """Broadcast the input tensor dictionary.
        NOTE: `src` is the local rank of the source rank.
        """
        # Bypass the function if we are using only 1 GPU.
        if not torch.distributed.is_initialized() or self.world_size == 1:
            return tensor_dict

        group = self.device_group
        metadata_group = self.cpu_group
        assert src < self.world_size, f"Invalid src rank ({src})"

        rank_in_group = self.rank_in_group
        if rank_in_group == src:
            metadata_list: List[Tuple[Any, Any]] = []
            assert isinstance(
                tensor_dict, dict
            ), f"Expecting a dictionary, got {type(tensor_dict)}"
            metadata_list, tensor_list = _split_tensor_dict(tensor_dict)
            # `metadata_list` lives in CPU memory.
            # `broadcast_object_list` has serialization & deserialization,
            # all happening on CPU. Therefore, we can use the CPU group.
            self.broadcast_object(metadata_list, src=src)
            async_handles = []
            for tensor in tensor_list:
                if tensor.numel() == 0:
                    # Skip broadcasting empty tensors.
                    continue
                if tensor.is_cpu:
                    # use metadata_group for CPU tensors
                    handle = torch.distributed.broadcast(
                        tensor, src=self.ranks[src], group=metadata_group, async_op=True
                    )
                else:
                    # use group for GPU tensors
                    handle = torch.distributed.broadcast(
                        tensor, src=self.ranks[src], group=group, async_op=True
                    )
                async_handles.append(handle)
            for async_handle in async_handles:
                async_handle.wait()

        else:
            metadata_list = self.broadcast_object(None, src=src)
            tensor_dict = {}
            async_handles = []
            for key, value in metadata_list:
                if isinstance(value, TensorMetadata):
                    tensor = torch.empty(
                        value.size, dtype=value.dtype, device=value.device
                    )
                    if tensor.numel() == 0:
                        # Skip broadcasting empty tensors.
                        tensor_dict[key] = tensor
```
**EN:** This callable implements `GroupCoordinator.broadcast_tensor_dict`. It takes `tensor_dict`, `src`, `group`, `metadata_group` and mainly implements broadcast tensor dict. The docstring states: "Broadcast the input tensor dictionary." This chunk is part 1 of 2 for the same logical block. In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.broadcast_tensor_dict`。它接收 `tensor_dict`, `src`, `group`, `metadata_group`，主要用于实现 broadcast tensor dict 相关逻辑。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 1208-1228: Method GroupCoordinator.broadcast_tensor_dict (part 2/2)
```python
                        continue
                    if tensor.is_cpu:
                        # use metadata_group for CPU tensors
                        handle = torch.distributed.broadcast(
                            tensor,
                            src=self.ranks[src],
                            group=metadata_group,
                            async_op=True,
                        )
                    else:
                        # use group for GPU tensors
                        handle = torch.distributed.broadcast(
                            tensor, src=self.ranks[src], group=group, async_op=True
                        )
                    async_handles.append(handle)
                    tensor_dict[key] = tensor
                else:
                    tensor_dict[key] = value
            for async_handle in async_handles:
                async_handle.wait()
        return tensor_dict
```
**EN:** This callable implements `GroupCoordinator.broadcast_tensor_dict`. It takes `tensor_dict`, `src`, `group`, `metadata_group` and mainly implements broadcast tensor dict. The docstring states: "Broadcast the input tensor dictionary." This chunk is part 2 of 2 for the same logical block. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.broadcast_tensor_dict`。它接收 `tensor_dict`, `src`, `group`, `metadata_group`，主要用于实现 broadcast tensor dict 相关逻辑。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会协调分布式通信。

### Lines 1230-1283: Method GroupCoordinator.send_tensor_dict
```python
    def send_tensor_dict(
        self,
        tensor_dict: Dict[str, Union[torch.Tensor, Any]],
        dst: Optional[int] = None,
        all_gather_group: Optional["GroupCoordinator"] = None,
        async_send: bool = False,
    ) -> Optional[List[P2PWork]]:
        """Send the input tensor dictionary.
        NOTE: `dst` is the local rank of the source rank.
        """
        # Bypass the function if we are using only 1 GPU.
        if self.world_size == 1:
            return tensor_dict

        all_gather_size = 1 if all_gather_group is None else all_gather_group.world_size
        all_gather_rank = (
            0 if all_gather_group is None else all_gather_group.rank_in_group
        )

        group = self.device_group
        metadata_group = self.cpu_group

        if dst is None:
            dst = (self.rank_in_group + 1) % self.world_size
        assert dst < self.world_size, f"Invalid dst rank ({dst})"

        assert isinstance(
            tensor_dict, dict
        ), f"Expecting a dictionary, got {type(tensor_dict)}"
        metadata_list, tensor_list = _split_tensor_dict(tensor_dict)
        # Note: While switching to Device-to-Device (D2D) would introduce an extra
        # Device-to-Host (D2H) memory copy overhead for serialization, our benchmarks
        # show better overall transmission performance with D2D due to:
        # 1. Superior D2D transfer bandwidth
        # 2. Ability to overlap send and recv operations
        # Thus the net performance gain justifies this approach.

        send_func = torch.distributed.isend if async_send else torch.distributed.send
        p2p_works = self.send_object(metadata_list, dst=dst, async_send=async_send)

        for tensor in tensor_list:
            if tensor.numel() == 0:
                # Skip sending empty tensors.
                continue

            # send-allgather: send only a slice, then do allgather.
            if all_gather_group is not None and tensor.numel() % all_gather_size == 0:
                tensor = tensor.reshape(all_gather_size, -1)[all_gather_rank]

            comm_group = metadata_group if tensor.is_cpu else group
            work = send_func(tensor, self.ranks[dst], group=comm_group)
            if async_send:
                p2p_works.append(P2PWork(work, tensor))
        return p2p_works
```
**EN:** This callable implements `GroupCoordinator.send_tensor_dict`. It takes `tensor_dict`, `dst`, `all_gather_group`, `async_send` and mainly sends data to another component. The docstring states: "Send the input tensor dictionary." In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.send_tensor_dict`。它接收 `tensor_dict`, `dst`, `all_gather_group`, `async_send`，主要用于向其他组件发送数据。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 1285-1343: Method GroupCoordinator.recv_tensor_dict
```python
    def recv_tensor_dict(
        self,
        src: Optional[int] = None,
        all_gather_group: Optional["GroupCoordinator"] = None,
    ) -> Optional[Dict[str, Union[torch.Tensor, Any]]]:
        """Recv the input tensor dictionary.
        NOTE: `src` is the local rank of the source rank.
        """
        # Bypass the function if we are using only 1 GPU.
        if not torch.distributed.is_initialized() or self.world_size == 1:
            return None

        all_gather_size = 1 if all_gather_group is None else all_gather_group.world_size
        all_gather_rank = (
            0 if all_gather_group is None else all_gather_group.rank_in_group
        )

        group = self.device_group
        metadata_group = self.cpu_group

        if src is None:
            src = (self.rank_in_group - 1) % self.world_size
        assert src < self.world_size, f"Invalid src rank ({src})"

        recv_metadata_list = self.recv_object(src=src)
        tensor_dict: Dict[str, Any] = {}
        for key, value in recv_metadata_list:
            if isinstance(value, TensorMetadata):
                tensor = torch.empty(value.size, dtype=value.dtype, device=value.device)
                if tensor.numel() == 0:
                    # Skip broadcasting empty tensors.
                    tensor_dict[key] = tensor
                    continue

                # send-allgather: send only a slice, then do allgather.
                use_all_gather = (
                    all_gather_group is not None
                    and tensor.numel() % all_gather_size == 0
                )

                if use_all_gather:
                    orig_shape = tensor.shape
                    tensor = tensor.reshape(all_gather_size, -1)[all_gather_rank]

                # We have to use irecv here to make it work for both isend and send.
                comm_group = metadata_group if tensor.is_cpu else group
                work = torch.distributed.irecv(
                    tensor, src=self.ranks[src], group=comm_group
                )
                work.wait()

                if use_all_gather:
                    tensor = all_gather_group.all_gather(tensor, dim=0)
                    tensor = tensor.reshape(orig_shape)

                tensor_dict[key] = tensor
            else:
                tensor_dict[key] = value
        return tensor_dict
```
**EN:** This callable implements `GroupCoordinator.recv_tensor_dict`. It takes `src`, `all_gather_group` and mainly receives data from another component. The docstring states: "Recv the input tensor dictionary." In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.recv_tensor_dict`。它接收 `src`, `all_gather_group`，主要用于从其他组件接收数据。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 1345-1352: Method GroupCoordinator.barrier
```python
    def barrier(self):
        """Barrier synchronization among the group.
        NOTE: don't use `device_group` here! `barrier` in NCCL is
        terrible because it is internally a broadcast operation with
        secretly created GPU tensors. It is easy to mess up the current
        device. Use the CPU group instead.
        """
        torch.distributed.barrier(group=self.cpu_group)
```
**EN:** This callable implements `GroupCoordinator.barrier` and mainly implements barrier. The docstring states: "Barrier synchronization among the group." In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.barrier`，主要用于实现 barrier 相关逻辑。 在这一范围内，它会协调分布式通信。

### Lines 1354-1364: Method GroupCoordinator.send
```python
    def send(self, tensor: torch.Tensor, dst: Optional[int] = None) -> None:
        """Sends a tensor to the destination rank in a non-blocking way"""
        """NOTE: `dst` is the local rank of the destination rank."""
        if dst is None:
            dst = (self.rank_in_group + 1) % self.world_size

        pynccl_comm = self.pynccl_comm
        if pynccl_comm is not None and not pynccl_comm.disabled:
            pynccl_comm.send(tensor, dst)
        else:
            torch.distributed.send(tensor, self.ranks[dst], self.device_group)
```
**EN:** This callable implements `GroupCoordinator.send`. It takes `tensor`, `dst` and mainly sends data to another component. The docstring states: "Sends a tensor to the destination rank in a non-blocking way" In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.send`。它接收 `tensor`, `dst`，主要用于向其他组件发送数据。 在这一范围内，它会协调分布式通信。

### Lines 1366-1380: Method GroupCoordinator.recv
```python
    def recv(
        self, size: torch.Size, dtype: torch.dtype, src: Optional[int] = None
    ) -> torch.Tensor:
        """Receives a tensor from the source rank."""
        """NOTE: `src` is the local rank of the source rank."""
        if src is None:
            src = (self.rank_in_group - 1) % self.world_size

        tensor = torch.empty(size, dtype=dtype, device=self.device)
        pynccl_comm = self.pynccl_comm
        if pynccl_comm is not None and not pynccl_comm.disabled:
            pynccl_comm.recv(tensor, src)
        else:
            torch.distributed.recv(tensor, self.ranks[src], self.device_group)
        return tensor
```
**EN:** This callable implements `GroupCoordinator.recv`. It takes `size`, `dtype`, `src` and mainly receives data from another component. The docstring states: "Receives a tensor from the source rank." In this range it sets up imports and shared symbols; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.recv`。它接收 `size`, `dtype`, `src`，主要用于从其他组件接收数据。 在这一范围内，它会建立导入关系并准备共享符号；协调分布式通信。

### Lines 1382-1394: Method GroupCoordinator.destroy
```python
    def destroy(self):
        if self.device_group is not None:
            torch.distributed.destroy_process_group(self.device_group)
            self.device_group = None
        if self.cpu_group is not None:
            torch.distributed.destroy_process_group(self.cpu_group)
            self.cpu_group = None
        if self.pynccl_comm is not None:
            self.pynccl_comm = None
        if self.ca_comm is not None:
            self.ca_comm = None
        if self.mq_broadcaster is not None:
            self.mq_broadcaster = None
```
**EN:** This callable implements `GroupCoordinator.destroy` and mainly implements destroy. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `GroupCoordinator.destroy`，主要用于实现 destroy 相关逻辑。 在这一范围内，它会协调分布式通信。

### Lines 1395-1399: Module-level constants and helpers
```python


_WORLD: Optional[GroupCoordinator] = None


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 1400-1402: Function get_world_group
```python
def get_world_group() -> GroupCoordinator:
    assert _WORLD is not None, "world group is not initialized"
    return _WORLD
```
**EN:** This callable implements `get_world_group` and mainly retrieves a value or derived view. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `get_world_group`，主要用于获取某个值或派生视图。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 1405-1421: Function init_world_group
```python
def init_world_group(
    ranks: List[int], local_rank: int, backend: str, recovered_rank: bool = False
) -> GroupCoordinator:
    return GroupCoordinator(
        group_ranks=[ranks],
        local_rank=local_rank,
        torch_distributed_backend=backend,
        use_pynccl=False,
        use_pymscclpp=False,
        use_custom_allreduce=False,
        use_torch_symm_mem_all_reduce=False,
        use_hpu_communicator=False,
        use_xpu_communicator=False,
        use_npu_communicator=False,
        group_name="world",
        recovered_rank=recovered_rank,
    )
```
**EN:** This callable implements `init_world_group`. It takes `ranks`, `local_rank`, `backend`, `recovered_rank` and mainly implements init world group. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `init_world_group`。它接收 `ranks`, `local_rank`, `backend`, `recovered_rank`，主要用于实现 init world group 相关逻辑。 在这一范围内，它会协调分布式通信。

### Lines 1424-1460: Function init_model_parallel_group
```python
def init_model_parallel_group(
    group_ranks: List[List[int]],
    local_rank: int,
    backend: str,
    use_pynccl: Optional[bool] = None,
    use_custom_allreduce: Optional[bool] = None,
    use_message_queue_broadcaster: bool = False,
    group_name: Optional[str] = None,
    use_mscclpp_allreduce: Optional[bool] = None,
    use_torch_symm_mem_allreduce: Optional[bool] = None,
    recovered_rank: bool = False,
) -> GroupCoordinator:
    if use_custom_allreduce is None:
        use_custom_allreduce = _ENABLE_CUSTOM_ALL_REDUCE
    if use_mscclpp_allreduce is None:
        use_mscclpp_allreduce = _ENABLE_MSCCLPP_ALL_REDUCE
    if use_torch_symm_mem_allreduce is None:
        use_torch_symm_mem_allreduce = _ENABLE_TORCH_SYMM_MEM_ALL_REDUCE
    return GroupCoordinator(
        group_ranks=group_ranks,
        local_rank=local_rank,
        torch_distributed_backend=backend,
        use_pynccl=(
            not (_is_npu or _is_xpu or backend == "mooncake")
            if use_pynccl is None
            else use_pynccl
        ),
        use_pymscclpp=use_mscclpp_allreduce,
        use_custom_allreduce=use_custom_allreduce,
        use_torch_symm_mem_all_reduce=use_torch_symm_mem_allreduce,
        use_hpu_communicator=True,
        use_xpu_communicator=True,
        use_npu_communicator=True,
        use_message_queue_broadcaster=use_message_queue_broadcaster,
        group_name=group_name,
        recovered_rank=recovered_rank,
    )
```
**EN:** This callable implements `init_model_parallel_group`. It takes `group_ranks`, `local_rank`, `backend`, `use_pynccl` and mainly implements init model parallel group. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `init_model_parallel_group`。它接收 `group_ranks`, `local_rank`, `backend`, `use_pynccl`，主要用于实现 init model parallel group 相关逻辑。 在这一范围内，它会协调分布式通信。

### Lines 1461-1472: Module-level constants and helpers
```python


_TP: Optional[GroupCoordinator] = None
_ATTN_TP: Optional[GroupCoordinator] = None
_ATTN_CP: Optional[GroupCoordinator] = None

# duplicate GroupCoordinator for prefill in PD-Multiplexing
_PDMUX_PREFILL_TP_GROUP: Optional[GroupCoordinator] = None

_ENABLE_PDMUX_P_TP: bool = False


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 1473-1475: Function set_pdmux_status
```python
def set_pdmux_status(enable_prefill_multiplexing: bool):
    global _ENABLE_PDMUX_P_TP
    _ENABLE_PDMUX_P_TP = enable_prefill_multiplexing
```
**EN:** This callable implements `set_pdmux_status`. It takes `enable_prefill_multiplexing` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `set_pdmux_status`。它接收 `enable_prefill_multiplexing`，主要用于将配置写入可变状态。

### Lines 1478-1485: Function get_tp_group
```python
def get_tp_group() -> GroupCoordinator:
    if _ENABLE_PDMUX_P_TP:
        assert (
            _PDMUX_PREFILL_TP_GROUP is not None
        ), "tensor model parallel group for PD-Multiplexing Prefill is not initialized"
        return _PDMUX_PREFILL_TP_GROUP
    assert _TP is not None, "tensor model parallel group is not initialized"
    return _TP
```
**EN:** This callable implements `get_tp_group` and mainly retrieves a value or derived view. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `get_tp_group`，主要用于获取某个值或派生视图。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 1488-1492: Function get_attn_tp_group
```python
def get_attn_tp_group() -> GroupCoordinator:
    assert (
        _ATTN_TP is not None
    ), "attention tensor model parallel group is not initialized"
    return _ATTN_TP
```
**EN:** This callable implements `get_attn_tp_group` and mainly retrieves a value or derived view. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `get_attn_tp_group`，主要用于获取某个值或派生视图。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 1495-1499: Function get_attn_cp_group
```python
def get_attn_cp_group() -> GroupCoordinator:
    assert (
        _ATTN_CP is not None
    ), "attention context model parallel group is not initialized"
    return _ATTN_CP
```
**EN:** This callable implements `get_attn_cp_group` and mainly retrieves a value or derived view. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `get_attn_cp_group`，主要用于获取某个值或派生视图。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 1500-1506: Module-level constants and helpers
```python


_MOE_DP: Optional[GroupCoordinator] = None
_MOE_EP: Optional[GroupCoordinator] = None
_MOE_TP: Optional[GroupCoordinator] = None


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 1507-1509: Function get_moe_dp_group
```python
def get_moe_dp_group() -> GroupCoordinator:
    assert _MOE_DP is not None, "moe data parallel group is not initialized"
    return _MOE_DP
```
**EN:** This callable implements `get_moe_dp_group` and mainly retrieves a value or derived view. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `get_moe_dp_group`，主要用于获取某个值或派生视图。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 1512-1514: Function get_moe_ep_group
```python
def get_moe_ep_group() -> GroupCoordinator:
    assert _MOE_EP is not None, "expert model parallel group is not initialized"
    return _MOE_EP
```
**EN:** This callable implements `get_moe_ep_group` and mainly retrieves a value or derived view. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `get_moe_ep_group`，主要用于获取某个值或派生视图。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 1517-1519: Function get_moe_tp_group
```python
def get_moe_tp_group() -> GroupCoordinator:
    assert _MOE_TP is not None, "expert model parallel group is not initialized"
    return _MOE_TP
```
**EN:** This callable implements `get_moe_tp_group` and mainly retrieves a value or derived view. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `get_moe_tp_group`，主要用于获取某个值或派生视图。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 1520-1527: Module-level constants and helpers
```python


# kept for backward compatibility
get_tensor_model_parallel_group = get_tp_group

_PP: Optional[GroupCoordinator] = None


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 1528-1530: Function get_pp_group
```python
def get_pp_group() -> GroupCoordinator:
    assert _PP is not None, "pipeline model parallel group is not initialized"
    return _PP
```
**EN:** This callable implements `get_pp_group` and mainly retrieves a value or derived view. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `get_pp_group`，主要用于获取某个值或派生视图。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 1531-1536: Module-level constants and helpers
```python


# kept for backward compatibility
get_pipeline_model_parallel_group = get_pp_group


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 1537-1546: Function get_mooncake_transfer_engine
```python
def get_mooncake_transfer_engine():
    """
    Return the shared MooncakeTransferEngine if initialized in device_communicators,
    else None. Used by disaggregation mooncake backend and mem_cache mooncake_store.
    """
    from sglang.srt.distributed.device_communicators.mooncake_transfer_engine import (
        get_mooncake_transfer_engine as _get_engine,
    )

    return _get_engine()
```
**EN:** This callable implements `get_mooncake_transfer_engine` and mainly retrieves a value or derived view. The docstring states: "Return the shared MooncakeTransferEngine if initialized in device_communicators, else None." In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `get_mooncake_transfer_engine`，主要用于获取某个值或派生视图。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 1549-1574: Function graph_capture
```python
@contextmanager
def graph_capture(stream: Optional[torch.cuda.Stream] = None):
    """
    `graph_capture` is a context manager which should surround the code that
    is capturing the CUDA graph. Its main purpose is to ensure that the
    some operations will be run after the graph is captured, before the graph
    is replayed. It returns a `GraphCaptureContext` object which contains the
    necessary data for the graph capture. Currently, it only contains the
    stream that the graph capture is running on. This stream is set to the
    current CUDA stream when the context manager is entered and reset to the
    default stream when the context manager is exited. This is to ensure that
    the graph capture is running on a separate stream from the default stream,
    in order to explicitly distinguish the kernels to capture
    from other kernels possibly launched on background in the default stream.
    """
    with (
        get_tp_group().graph_capture(stream=stream) as context,
        get_pp_group().graph_capture(context),
    ):
        with contextlib.ExitStack() as stack:
            seen = {id(_TP)}
            for group in (_MOE_EP, _MOE_TP):
                if group is not None and id(group) not in seen:
                    seen.add(id(group))
                    stack.enter_context(group.graph_capture(context))
            yield context
```
**EN:** This callable implements `graph_capture`. It takes `stream` and mainly implements graph capture. The docstring states: "`graph_capture` is a context manager which should surround the code that is capturing the CUDA graph." In this range it sets up imports and shared symbols; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `graph_capture`。它接收 `stream`，主要用于实现 graph capture 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；管理图捕获或回放逻辑。

### Lines 1575-1583: Module-level constants and helpers
```python


logger = logging.getLogger(__name__)

_ENABLE_CUSTOM_ALL_REDUCE = True
_ENABLE_MSCCLPP_ALL_REDUCE = False
_ENABLE_TORCH_SYMM_MEM_ALL_REDUCE = False


```
**EN:** This range organizes module-level state and shared setup. In this range it emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会输出日志以便诊断；协调分布式通信。

### Lines 1584-1586: Function set_custom_all_reduce
```python
def set_custom_all_reduce(enable: bool):
    global _ENABLE_CUSTOM_ALL_REDUCE
    _ENABLE_CUSTOM_ALL_REDUCE = enable
```
**EN:** This callable implements `set_custom_all_reduce`. It takes `enable` and mainly converts data into another representation. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `set_custom_all_reduce`。它接收 `enable`，主要用于将数据转换为另一种表示。 在这一范围内，它会协调分布式通信。

### Lines 1589-1591: Function set_mscclpp_all_reduce
```python
def set_mscclpp_all_reduce(enable: bool):
    global _ENABLE_MSCCLPP_ALL_REDUCE
    _ENABLE_MSCCLPP_ALL_REDUCE = enable
```
**EN:** This callable implements `set_mscclpp_all_reduce`. It takes `enable` and mainly applies configuration to mutable state. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `set_mscclpp_all_reduce`。它接收 `enable`，主要用于将配置写入可变状态。 在这一范围内，它会协调分布式通信。

### Lines 1594-1596: Function set_torch_symm_mem_all_reduce
```python
def set_torch_symm_mem_all_reduce(enable: bool):
    global _ENABLE_TORCH_SYMM_MEM_ALL_REDUCE
    _ENABLE_TORCH_SYMM_MEM_ALL_REDUCE = enable
```
**EN:** This callable implements `set_torch_symm_mem_all_reduce`. It takes `enable` and mainly converts data into another representation. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `set_torch_symm_mem_all_reduce`。它接收 `enable`，主要用于将数据转换为另一种表示。 在这一范围内，它会协调分布式通信。

### Lines 1597-1608: Module-level constants and helpers
```python


_DEVICE_TO_DISTRIBUTED_BACKEND = {
    "cuda": "nccl",
    "xpu": "xccl",
    "hpu": "hccl",
    "cpu": "gloo",
    "npu": "hccl" if not envs.SGLANG_ZBAL_LOCAL_MEM_SIZE.get() > 0 else "zbal",
    "musa": "mccl",
}


```
**EN:** This range organizes module-level state and shared setup. In this range it reads environment-driven configuration.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会读取环境变量驱动的配置。

### Lines 1609-1610: Function get_default_distributed_backend
```python
def get_default_distributed_backend(device: str) -> str:
    return _DEVICE_TO_DISTRIBUTED_BACKEND.get(device, "gloo")
```
**EN:** This callable implements `get_default_distributed_backend`. It takes `device` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `get_default_distributed_backend`。它接收 `device`，主要用于获取某个值或派生视图。

### Lines 1613-1665: Function _create_global_tcp_store
```python
def _create_global_tcp_store(rank: int, world_size: int) -> None:
    """Create a global TCPStore for coordination across ranks.

    This function creates a TCPStore that all ranks can use for coordination
    (e.g., for NIXL buffer setup).
    """
    from torch.distributed import TCPStore

    master_ip = os.environ.get("MASTER_ADDR")

    if not master_ip:
        logger.warning(
            "Could not determine master IP for global TCPStore. "
            "Broadcasting from rank 0 to all ranks."
        )

    base_store_port = envs.SGLANG_TCP_STORE_PORT.get()

    # Rank 0 gets its local IP and broadcasts it to all ranks
    # Use broadcast_object_list which works with any backend (handles CPU/GPU automatically)
    if not master_ip:
        if rank == 0:
            master_ip = get_local_ip_auto()
            ip_list = [master_ip]
        else:
            ip_list = [None]

        torch.distributed.broadcast_object_list(ip_list, src=0)
        master_ip = ip_list[0]

    try:
        tcp_store = TCPStore(
            host_name=master_ip,
            port=base_store_port,
            world_size=world_size,
            is_master=(rank == 0),
        )
        set_global_tcp_store(tcp_store)
        logger.info(
            "Created global TCPStore at %s:%d (rank=%d, world_size=%d)",
            master_ip,
            base_store_port,
            rank,
            world_size,
        )
    except Exception as e:
        logger.warning(
            "Failed to create global TCPStore at %s:%d: %s. "
            "Components requiring TCPStore (like NIXL) may not work.",
            master_ip,
            base_store_port,
            e,
        )
```
**EN:** This callable implements `_create_global_tcp_store`. It takes `rank`, `world_size` and mainly converts data into another representation. The docstring states: "Create a global TCPStore for coordination across ranks." In this range it sets up imports and shared symbols; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `_create_global_tcp_store`。它接收 `rank`, `world_size`，主要用于将数据转换为另一种表示。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；协调分布式通信。

### Lines 1668-1727: Function init_distributed_environment (part 1/2)
```python
def init_distributed_environment(
    world_size: int = -1,
    rank: int = -1,
    distributed_init_method: str = "env://",
    local_rank: int = -1,
    backend: str = "nccl",
    timeout: Optional[int] = None,
    moe_a2a_backend: Optional[str] = None,
    recovered_rank: bool = False,
):
    logger.debug(
        "world_size=%d rank=%d local_rank=%d " "distributed_init_method=%s backend=%s",
        world_size,
        rank,
        local_rank,
        distributed_init_method,
        backend,
    )
    if "mooncake" in backend:
        try:
            from mooncake import ep as mooncake_ep
        except ImportError as e:
            raise ImportError(
                "Please install mooncake by following the instructions at "
                "https://github.com/kvcache-ai/Mooncake/blob/main/doc/en/build.md "  # noqa: E501
                "to run SGLang with Mooncake Backend."
            ) from e
        mooncake_ep.set_host_ip(get_local_ip_auto())

    if not torch.distributed.is_initialized():
        global _MODEL_PARALLEL_GROUP_TIMEOUT
        assert distributed_init_method is not None, (
            "distributed_init_method must be provided when initializing "
            "distributed environment"
        )
        if timeout is not None:
            assert isinstance(timeout, (int)), "timeout must be a number"
            assert timeout > 0, "timeout must be positive"
            timeout = timedelta(seconds=timeout)

        _MODEL_PARALLEL_GROUP_TIMEOUT = timeout

        if backend == "mooncake":
            from mooncake.ep import MooncakeBackendOptions

            # Setting "cuda" as device here is safe, as it is guarded under the mooncake case
            active_ranks = torch.ones(world_size, dtype=torch.int32, device="cuda")
            pg_options = MooncakeBackendOptions(active_ranks, recovered_rank)
        else:
            pg_options = get_torch_distributed_pg_options()

        # this backend is used for WORLD
        torch.distributed.init_process_group(
            backend=backend,
            init_method=distributed_init_method,
            world_size=world_size,
            rank=rank,
            timeout=timeout,
            pg_options=pg_options,
        )
```
**EN:** This callable implements `init_distributed_environment`. It takes `world_size`, `rank`, `distributed_init_method`, `local_rank` and mainly implements init distributed environment. This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `init_distributed_environment`。它接收 `world_size`, `rank`, `distributed_init_method`, `local_rank`，主要用于实现 init distributed environment 相关逻辑。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；输出日志以便诊断。

### Lines 1728-1752: Function init_distributed_environment (part 2/2)
```python

        # Create a global TCPStore for coordination (used by NIXL)
        if moe_a2a_backend == "nixl":
            _create_global_tcp_store(rank, world_size)

    # set the local rank
    # local_rank is not available in torch ProcessGroup,
    # see https://github.com/pytorch/pytorch/issues/122816
    if local_rank == -1:
        # local rank not set, this usually happens in single-node
        # setting, where we can use rank as local rank
        if distributed_init_method == "env://":
            local_rank = int(os.environ.get("LOCAL_RANK", "0"))
        else:
            local_rank = rank
    global _WORLD
    if _WORLD is None:
        ranks = list(range(torch.distributed.get_world_size()))
        _WORLD = init_world_group(
            ranks, local_rank, backend, recovered_rank=recovered_rank
        )
    else:
        assert (
            _WORLD.world_size == torch.distributed.get_world_size()
        ), "world group already initialized with a different world size"
```
**EN:** This callable implements `init_distributed_environment`. It takes `world_size`, `rank`, `distributed_init_method`, `local_rank` and mainly implements init distributed environment. This chunk is part 2 of 2 for the same logical block. In this range it performs defensive checks on invalid state; coordinates distributed communication; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `init_distributed_environment`。它接收 `world_size`, `rank`, `distributed_init_method`, `local_rank`，主要用于实现 init distributed environment 相关逻辑。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信；读取环境变量驱动的配置。

### Lines 1755-1814: Function initialize_model_parallel (part 1/5)
```python
def initialize_model_parallel(
    tensor_model_parallel_size: int = 1,
    expert_model_parallel_size: int = 1,
    pipeline_model_parallel_size: int = 1,
    attention_data_parallel_size: int = 1,
    attention_context_model_parallel_size: int = 1,
    moe_data_model_parallel_size: int = 1,
    backend: Optional[str] = None,
    duplicate_tp_group: bool = False,
    enable_symm_mem: bool = False,
    recovered_rank: bool = False,
) -> None:
    """
    Initialize model parallel groups.

    Arguments:
        tensor_model_parallel_size: number of GPUs used for tensor model
            parallelism.
        expert_model_parallel_size: number of GPUs used for expert model
            parallelism.
        pipeline_model_parallel_size: number of GPUs used for pipeline model
            parallelism.
        attention_data_parallel_size: number of GPUs used for attention data
            parallelism.
        attention_context_model_parallel_size: number of GPUs used for attention context
            parallelism.
        moe_data_model_parallel_size: number of GPUs used for moe data
            parallelism.

    Let's say we have a total of 8 GPUs denoted by g0 ... g7 and we
    use 2 GPUs to parallelize the model tensor, and 4 GPUs to parallelize
    the model pipeline. The present function will
    create 4 tensor model-parallel groups and 2 pipeline model-parallel groups:
        4 tensor model-parallel groups:
            [g0, g1], [g2, g3], [g4, g5], [g6, g7]
        2 pipeline model-parallel groups:
            [g0, g2, g4, g6], [g1, g3, g5, g7]

    Let's say we use 2 GPUs for attention context parallelism (attn_cp_size=2) and 4 GPUs for
    attention tensor parallelism (attn_tp_size=4). As for MoE part, we use 2 GPUs for moe data
    parallelism (moe_dp_size=2) and 4 GPUs for moe expert parallelism (moe_ep_size=4). The present
    function will create the following groups:
        2 tensor model-parallel groups:
            [g0, g1, g2, g3], [g4, g5, g6, g7]
        4 attention context-parallel groups:
            [g0, g4], [g1, g5], [g2, g6], [g3, g7]
        2 moe expert-parallel groups:
            [g0, g1, g2, g3], [g4, g5, g6, g7]
        4 moe data-parallel groups:
            [g0, g4], [g1, g5], [g2, g6], [g3, g7]

    Note that for efficiency, the caller should make sure adjacent ranks
    are on the same DGX box. For example if we are using 2 DGX-1 boxes
    with a total of 16 GPUs, rank 0 to 7 belong to the first box and
    ranks 8 to 15 belong to the second box.
    """
    # Get world size and rank. Ensure some consistencies.
    assert torch.distributed.is_initialized()
    world_size: int = torch.distributed.get_world_size()
    backend = backend or torch.distributed.get_backend(get_world_group().device_group)
```
**EN:** This callable implements `initialize_model_parallel`. It takes `tensor_model_parallel_size`, `expert_model_parallel_size`, `pipeline_model_parallel_size`, `attention_data_parallel_size` and mainly implements initialize model parallel. The docstring states: "Initialize model parallel groups." This chunk is part 1 of 5 for the same logical block. In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `initialize_model_parallel`。它接收 `tensor_model_parallel_size`, `expert_model_parallel_size`, `pipeline_model_parallel_size`, `attention_data_parallel_size`，主要用于实现 initialize model parallel 相关逻辑。 该片段是同一逻辑块的第 1/5 部分。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 1815-1874: Function initialize_model_parallel (part 2/5)
```python

    if world_size != tensor_model_parallel_size * pipeline_model_parallel_size:
        raise RuntimeError(
            f"world_size ({world_size}) is not equal to "
            f"tensor_model_parallel_size ({tensor_model_parallel_size}) x "
            f"pipeline_model_parallel_size ({pipeline_model_parallel_size})"
        )

    # Build the tensor model-parallel groups.
    num_tensor_model_parallel_groups: int = world_size // tensor_model_parallel_size
    global _TP
    assert _TP is None, "tensor model parallel group is already initialized"
    group_ranks = []
    for tp_group_idx in range(num_tensor_model_parallel_groups):
        ranks = list(
            range(
                tp_group_idx * tensor_model_parallel_size,
                (tp_group_idx + 1) * tensor_model_parallel_size,
            )
        )
        group_ranks.append(ranks)

    # message queue broadcaster is only used in tensor model parallel group
    _TP = init_model_parallel_group(
        group_ranks,
        get_world_group().local_rank,
        backend,
        use_message_queue_broadcaster=envs.SGLANG_USE_MESSAGE_QUEUE_BROADCASTER.get(),
        group_name="tp",
        recovered_rank=recovered_rank,
    )

    if duplicate_tp_group:
        global _PDMUX_PREFILL_TP_GROUP
        assert (
            _PDMUX_PREFILL_TP_GROUP is None
        ), "tensor model parallel group for PD-Multiplexing Prefill is already initialized"
        _PDMUX_PREFILL_TP_GROUP = init_model_parallel_group(
            group_ranks,
            get_world_group().local_rank,
            backend,
            use_message_queue_broadcaster=envs.SGLANG_USE_MESSAGE_QUEUE_BROADCASTER.get(),
            group_name="pdmux_prefill_tp",
            recovered_rank=recovered_rank,
        )
        if _TP.pynccl_comm:
            _TP.pynccl_comm.disabled = False
            _PDMUX_PREFILL_TP_GROUP.pynccl_comm.disabled = False

    attn_dp_size = attention_data_parallel_size
    attn_cp_size = attention_context_model_parallel_size
    attn_tp_size = tensor_model_parallel_size // attn_cp_size // attn_dp_size

    global _ATTN_CP
    assert (
        _ATTN_CP is None
    ), "attention context model parallel group is already initialized"
    if attn_cp_size == tensor_model_parallel_size:
        _ATTN_CP = _TP
    else:
```
**EN:** This callable implements `initialize_model_parallel`. It takes `tensor_model_parallel_size`, `expert_model_parallel_size`, `pipeline_model_parallel_size`, `attention_data_parallel_size` and mainly implements initialize model parallel. The docstring states: "Initialize model parallel groups." This chunk is part 2 of 5 for the same logical block. In this range it performs defensive checks on invalid state; coordinates distributed communication; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `initialize_model_parallel`。它接收 `tensor_model_parallel_size`, `expert_model_parallel_size`, `pipeline_model_parallel_size`, `attention_data_parallel_size`，主要用于实现 initialize model parallel 相关逻辑。 该片段是同一逻辑块的第 2/5 部分。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信；读取环境变量驱动的配置。

### Lines 1875-1934: Function initialize_model_parallel (part 3/5)
```python
        group_ranks = []
        for tp_group_idx in range(num_tensor_model_parallel_groups):
            for dp_idx in range(attn_dp_size):
                for attn_tp_idx in range(attn_tp_size):
                    st = (
                        tp_group_idx * tensor_model_parallel_size
                        + dp_idx * attn_tp_size * attn_cp_size
                        + attn_tp_idx
                    )
                    en = (
                        tp_group_idx * tensor_model_parallel_size
                        + (dp_idx + 1) * attn_tp_size * attn_cp_size
                        + attn_tp_idx
                    )
                    ranks = list(range(st, en, attn_tp_size))
                    group_ranks.append(ranks)
        _ATTN_CP = init_model_parallel_group(
            group_ranks,
            get_world_group().local_rank,
            backend,
            use_message_queue_broadcaster=envs.SGLANG_USE_MESSAGE_QUEUE_BROADCASTER.get(),
            group_name="attn_cp",
            recovered_rank=recovered_rank,
        )

    from sglang.srt.layers.sampler import SYNC_TOKEN_IDS_ACROSS_TP

    global _ATTN_TP
    assert (
        _ATTN_TP is None
    ), "attention tensor model parallel group is already initialized"
    if attn_tp_size == tensor_model_parallel_size:
        _ATTN_TP = _TP
    else:
        group_ranks = []
        for tp_group_idx in range(num_tensor_model_parallel_groups):
            for cp_dp_combined_idx in range(attn_cp_size * attn_dp_size):
                st = (
                    tp_group_idx * tensor_model_parallel_size
                    + cp_dp_combined_idx * attn_tp_size
                )
                en = (
                    tp_group_idx * tensor_model_parallel_size
                    + (cp_dp_combined_idx + 1) * attn_tp_size
                )
                ranks = list(range(st, en))
                group_ranks.append(ranks)

        _ATTN_TP = init_model_parallel_group(
            group_ranks,
            get_world_group().local_rank,
            backend,
            use_pynccl=SYNC_TOKEN_IDS_ACROSS_TP or enable_symm_mem,
            use_mscclpp_allreduce=False,
            use_custom_allreduce=False,
            use_torch_symm_mem_allreduce=False,
            use_message_queue_broadcaster=envs.SGLANG_USE_MESSAGE_QUEUE_BROADCASTER.get(),
            group_name="attention_tp",
            recovered_rank=recovered_rank,
        )
```
**EN:** This callable implements `initialize_model_parallel`. It takes `tensor_model_parallel_size`, `expert_model_parallel_size`, `pipeline_model_parallel_size`, `attention_data_parallel_size` and mainly implements initialize model parallel. The docstring states: "Initialize model parallel groups." This chunk is part 3 of 5 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `initialize_model_parallel`。它接收 `tensor_model_parallel_size`, `expert_model_parallel_size`, `pipeline_model_parallel_size`, `attention_data_parallel_size`，主要用于实现 initialize model parallel 相关逻辑。 该片段是同一逻辑块的第 3/5 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；协调分布式通信。

### Lines 1935-1994: Function initialize_model_parallel (part 4/5)
```python

    moe_ep_size = expert_model_parallel_size
    moe_dp_size = moe_data_model_parallel_size
    moe_tp_size = tensor_model_parallel_size // moe_ep_size // moe_dp_size

    global _MOE_DP
    assert _MOE_DP is None, "moe data parallel group is already initialized"
    if attn_cp_size > moe_dp_size:
        # When moe_dp_size < attn_cp_size, CP ranks must share tokens before MoE.
        # The MOE_DP group includes these CP partners, so the existing DP
        # allgather/scatter handles the token sharing.
        _MOE_DP = _ATTN_CP
    elif moe_dp_size == tensor_model_parallel_size:
        _MOE_DP = _TP
    else:
        group_ranks = []
        for tp_group_idx in range(num_tensor_model_parallel_groups):
            for tp_ep_combined_idx in range(moe_tp_size * moe_ep_size):
                st = tp_group_idx * tensor_model_parallel_size + tp_ep_combined_idx
                en = (
                    tp_group_idx + 1
                ) * tensor_model_parallel_size + tp_ep_combined_idx
                ranks = list(range(st, en, moe_tp_size * moe_ep_size))
                group_ranks.append(ranks)
        _MOE_DP = init_model_parallel_group(
            group_ranks,
            get_world_group().local_rank,
            backend,
            group_name="moe_dp",
            recovered_rank=recovered_rank,
        )

    global _MOE_EP
    assert _MOE_EP is None, "expert model parallel group is already initialized"
    if moe_ep_size == tensor_model_parallel_size:
        _MOE_EP = _TP
    else:
        group_ranks = []
        for tp_group_idx in range(num_tensor_model_parallel_groups):
            for moe_dp_idx in range(moe_dp_size):
                for moe_tp_idx in range(moe_tp_size):
                    st = (
                        tp_group_idx * tensor_model_parallel_size
                        + moe_dp_idx * moe_ep_size * moe_tp_size
                        + moe_tp_idx
                    )
                    en = st + moe_ep_size * moe_tp_size
                    ranks = list(range(st, en, moe_tp_size))
                    group_ranks.append(ranks)
        _MOE_EP = init_model_parallel_group(
            group_ranks,
            get_world_group().local_rank,
            backend,
            use_pynccl=False,
            use_custom_allreduce=False,
            group_name="moe_ep",
            recovered_rank=recovered_rank,
        )

    global _MOE_TP
```
**EN:** This callable implements `initialize_model_parallel`. It takes `tensor_model_parallel_size`, `expert_model_parallel_size`, `pipeline_model_parallel_size`, `attention_data_parallel_size` and mainly implements initialize model parallel. The docstring states: "Initialize model parallel groups." This chunk is part 4 of 5 for the same logical block. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `initialize_model_parallel`。它接收 `tensor_model_parallel_size`, `expert_model_parallel_size`, `pipeline_model_parallel_size`, `attention_data_parallel_size`，主要用于实现 initialize model parallel 相关逻辑。 该片段是同一逻辑块的第 4/5 部分。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 1995-2040: Function initialize_model_parallel (part 5/5)
```python
    assert _MOE_TP is None, "expert model parallel group is already initialized"
    if moe_tp_size == tensor_model_parallel_size:
        _MOE_TP = _TP
    else:
        group_ranks = []
        for tp_group_idx in range(num_tensor_model_parallel_groups):
            for ep_dp_combined_idx in range(moe_ep_size * moe_dp_size):
                st = (
                    tp_group_idx * tensor_model_parallel_size
                    + ep_dp_combined_idx * moe_tp_size
                )
                en = (
                    tp_group_idx * tensor_model_parallel_size
                    + (ep_dp_combined_idx + 1) * moe_tp_size
                )
                ranks = list(range(st, en))
                group_ranks.append(ranks)
        _MOE_TP = init_model_parallel_group(
            group_ranks,
            get_world_group().local_rank,
            backend,
            use_pynccl=False,
            use_custom_allreduce=False,
            group_name="moe_tp",
            recovered_rank=recovered_rank,
        )

    # Build the pipeline model-parallel groups.
    num_pipeline_model_parallel_groups: int = world_size // pipeline_model_parallel_size
    global _PP
    assert _PP is None, "pipeline model parallel group is already initialized"
    group_ranks = []
    for pp_group_idx in range(num_pipeline_model_parallel_groups):
        ranks = list(
            range(pp_group_idx, world_size, num_pipeline_model_parallel_groups)
        )
        group_ranks.append(ranks)
    # pipeline parallel does not need custom allreduce
    _PP = init_model_parallel_group(
        group_ranks,
        get_world_group().local_rank,
        backend,
        use_custom_allreduce=False,
        group_name="pp",
        recovered_rank=recovered_rank,
    )
```
**EN:** This callable implements `initialize_model_parallel`. It takes `tensor_model_parallel_size`, `expert_model_parallel_size`, `pipeline_model_parallel_size`, `attention_data_parallel_size` and mainly implements initialize model parallel. The docstring states: "Initialize model parallel groups." This chunk is part 5 of 5 for the same logical block. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `initialize_model_parallel`。它接收 `tensor_model_parallel_size`, `expert_model_parallel_size`, `pipeline_model_parallel_size`, `attention_data_parallel_size`，主要用于实现 initialize model parallel 相关逻辑。 该片段是同一逻辑块的第 5/5 部分。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 2043-2089: Function create_custom_parallel_group
```python
def create_custom_parallel_group(
    group_ranks: List[int], backend: str = "gloo"
) -> Optional[torch.distributed.ProcessGroup]:
    """
    Create a custom parallel group based on the provided ranks.

    Args:
        group_ranks: The list of ranks that the CURRENT process wants to join.
                     (e.g., Rank 0 passes [0...7], Rank 8 passes [8...15])
        backend: The communication backend (default: "gloo").

    Returns:
        The ProcessGroup if the current rank is in group_ranks, else None.
    """
    assert torch.distributed.is_initialized()

    world_size = torch.distributed.get_world_size()
    rank = torch.distributed.get_rank()

    local_config = sorted(list(set(group_ranks)))
    gathered_configs = [None for _ in range(world_size)]

    torch.distributed.all_gather_object(gathered_configs, local_config)

    unique_groups = []
    seen_signatures = set()

    for config in gathered_configs:
        config_tuple = tuple(config)
        if config_tuple not in seen_signatures:
            seen_signatures.add(config_tuple)
            unique_groups.append(list(config_tuple))

    unique_groups.sort(key=lambda x: x[0])

    my_new_group = None

    for g_ranks in unique_groups:
        group = torch.distributed.new_group(ranks=g_ranks, backend=backend)

        if set(g_ranks) == set(local_config):
            my_new_group = group
            logger.debug(
                f"Rank {rank} successfully created/joined custom group: {g_ranks}"
            )

    return my_new_group
```
**EN:** This callable implements `create_custom_parallel_group`. It takes `group_ranks`, `backend` and mainly converts data into another representation. The docstring states: "Create a custom parallel group based on the provided ranks." In this range it performs defensive checks on invalid state; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `create_custom_parallel_group`。它接收 `group_ranks`, `backend`，主要用于将数据转换为另一种表示。 在这一范围内，它会对非法状态执行防御性检查；输出日志以便诊断；协调分布式通信。

### Lines 2092-2122: Function ensure_model_parallel_initialized
```python
def ensure_model_parallel_initialized(
    tensor_model_parallel_size: int,
    expert_model_parallel_size: int,
    pipeline_model_parallel_size: int,
    backend: Optional[str] = None,
) -> None:
    """Helper to initialize model parallel groups if they are not initialized,
    or ensure tensor-parallel and pipeline-parallel sizes are equal to expected
    values if the model parallel groups are initialized.
    """
    backend = backend or torch.distributed.get_backend(get_world_group().device_group)
    if not model_parallel_is_initialized():
        initialize_model_parallel(
            tensor_model_parallel_size,
            expert_model_parallel_size,
            pipeline_model_parallel_size,
            backend,
        )
        return

    assert get_tensor_model_parallel_world_size() == tensor_model_parallel_size, (
        "tensor parallel group already initialized, but of unexpected size: "
        f"{get_tensor_model_parallel_world_size()=} vs. "
        f"{tensor_model_parallel_size=}"
    )
    pp_world_size = get_pp_group().world_size
    assert pp_world_size == pipeline_model_parallel_size, (
        "pipeline parallel group already initialized, but of unexpected size: "
        f"{pp_world_size=} vs. "
        f"{pipeline_model_parallel_size=}"
    )
```
**EN:** This callable implements `ensure_model_parallel_initialized`. It takes `tensor_model_parallel_size`, `expert_model_parallel_size`, `pipeline_model_parallel_size`, `backend` and mainly ensures required state exists. The docstring states: "Helper to initialize model parallel groups if they are not initialized, or ensure tensor-parallel and pipeline-parallel sizes are equal to expected values if the model parallel groups are initialized." In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `ensure_model_parallel_initialized`。它接收 `tensor_model_parallel_size`, `expert_model_parallel_size`, `pipeline_model_parallel_size`, `backend`，主要用于确保所需状态已经存在。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 2125-2127: Function model_parallel_is_initialized
```python
def model_parallel_is_initialized():
    """Check if tensor and pipeline parallel groups are initialized."""
    return _TP is not None and _PP is not None
```
**EN:** This callable implements `model_parallel_is_initialized` and mainly implements model parallel is initialized. The docstring states: "Check if tensor and pipeline parallel groups are initialized."
**CN:** 这一可调用对象实现了 `model_parallel_is_initialized`，主要用于实现 model parallel is initialized 相关逻辑。

### Lines 2128-2132: Module-level constants and helpers
```python


_TP_STATE_PATCHED = False


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 2133-2155: Function patch_tensor_parallel_group
```python
@contextmanager
def patch_tensor_parallel_group(tp_group: GroupCoordinator):
    """Patch the tp group temporarily until this function ends.

    This method is for draft workers of speculative decoding to run draft model
    with different tp degree from that of target model workers.

    Args:
        tp_group (GroupCoordinator): the tp group coordinator
    """
    global _TP_STATE_PATCHED
    assert not _TP_STATE_PATCHED, "Should not call when it's already patched"

    _TP_STATE_PATCHED = True
    old_tp_group = get_tp_group()
    global _TP
    _TP = tp_group
    try:
        yield
    finally:
        # restore the original state
        _TP_STATE_PATCHED = False
        _TP = old_tp_group
```
**EN:** This callable implements `patch_tensor_parallel_group`. It takes `tp_group` and mainly implements patch tensor parallel group. The docstring states: "Patch the tp group temporarily until this function ends." In this range it sets up imports and shared symbols; performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `patch_tensor_parallel_group`。它接收 `tp_group`，主要用于实现 patch tensor parallel group 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查。

### Lines 2158-2160: Function get_world_size
```python
def get_world_size():
    """Return world size for the world group."""
    return get_world_group().world_size
```
**EN:** This callable implements `get_world_size` and mainly retrieves a value or derived view. The docstring states: "Return world size for the world group."
**CN:** 这一可调用对象实现了 `get_world_size`，主要用于获取某个值或派生视图。

### Lines 2163-2165: Function get_world_rank
```python
def get_world_rank():
    """Return my rank for the world group."""
    return get_world_group().rank_in_group
```
**EN:** This callable implements `get_world_rank` and mainly retrieves a value or derived view. The docstring states: "Return my rank for the world group."
**CN:** 这一可调用对象实现了 `get_world_rank`，主要用于获取某个值或派生视图。

### Lines 2168-2170: Function get_tensor_model_parallel_world_size
```python
def get_tensor_model_parallel_world_size():
    """Return world size for the tensor model parallel group."""
    return get_tp_group().world_size
```
**EN:** This callable implements `get_tensor_model_parallel_world_size` and mainly retrieves a value or derived view. The docstring states: "Return world size for the tensor model parallel group."
**CN:** 这一可调用对象实现了 `get_tensor_model_parallel_world_size`，主要用于获取某个值或派生视图。

### Lines 2173-2175: Function get_tensor_model_parallel_rank
```python
def get_tensor_model_parallel_rank():
    """Return my rank for the tensor model parallel group."""
    return get_tp_group().rank_in_group
```
**EN:** This callable implements `get_tensor_model_parallel_rank` and mainly retrieves a value or derived view. The docstring states: "Return my rank for the tensor model parallel group."
**CN:** 这一可调用对象实现了 `get_tensor_model_parallel_rank`，主要用于获取某个值或派生视图。

### Lines 2176-2178: Module-level logic
```python


# ATTN_TP
```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 2179-2181: Function get_attn_tensor_model_parallel_world_size
```python
def get_attn_tensor_model_parallel_world_size():
    """Return world size for the attention tensor model parallel group."""
    return get_attn_tp_group().world_size
```
**EN:** This callable implements `get_attn_tensor_model_parallel_world_size` and mainly retrieves a value or derived view. The docstring states: "Return world size for the attention tensor model parallel group."
**CN:** 这一可调用对象实现了 `get_attn_tensor_model_parallel_world_size`，主要用于获取某个值或派生视图。

### Lines 2184-2186: Function get_attn_tensor_model_parallel_rank
```python
def get_attn_tensor_model_parallel_rank():
    """Return my rank for the attention tensor model parallel group."""
    return get_attn_tp_group().rank_in_group
```
**EN:** This callable implements `get_attn_tensor_model_parallel_rank` and mainly retrieves a value or derived view. The docstring states: "Return my rank for the attention tensor model parallel group."
**CN:** 这一可调用对象实现了 `get_attn_tensor_model_parallel_rank`，主要用于获取某个值或派生视图。

### Lines 2187-2189: Module-level logic
```python


# ATTN_CP
```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 2190-2192: Function get_attn_context_model_parallel_world_size
```python
def get_attn_context_model_parallel_world_size():
    """Return world size for the attention context model parallel group."""
    return get_attn_cp_group().world_size
```
**EN:** This callable implements `get_attn_context_model_parallel_world_size` and mainly retrieves a value or derived view. The docstring states: "Return world size for the attention context model parallel group."
**CN:** 这一可调用对象实现了 `get_attn_context_model_parallel_world_size`，主要用于获取某个值或派生视图。

### Lines 2195-2197: Function get_attn_context_model_parallel_rank
```python
def get_attn_context_model_parallel_rank():
    """Return my rank for the attention context model parallel group."""
    return get_attn_cp_group().rank_in_group
```
**EN:** This callable implements `get_attn_context_model_parallel_rank` and mainly retrieves a value or derived view. The docstring states: "Return my rank for the attention context model parallel group."
**CN:** 这一可调用对象实现了 `get_attn_context_model_parallel_rank`，主要用于获取某个值或派生视图。

### Lines 2200-2202: Function get_pipeline_model_parallel_world_size
```python
def get_pipeline_model_parallel_world_size():
    """Return world size for the pipeline model parallel group."""
    return get_pp_group().world_size
```
**EN:** This callable implements `get_pipeline_model_parallel_world_size` and mainly retrieves a value or derived view. The docstring states: "Return world size for the pipeline model parallel group."
**CN:** 这一可调用对象实现了 `get_pipeline_model_parallel_world_size`，主要用于获取某个值或派生视图。

### Lines 2205-2207: Function get_pipeline_model_parallel_rank
```python
def get_pipeline_model_parallel_rank():
    """Return my rank for the pipeline model parallel group."""
    return get_pp_group().rank_in_group
```
**EN:** This callable implements `get_pipeline_model_parallel_rank` and mainly retrieves a value or derived view. The docstring states: "Return my rank for the pipeline model parallel group."
**CN:** 这一可调用对象实现了 `get_pipeline_model_parallel_rank`，主要用于获取某个值或派生视图。

### Lines 2208-2210: Module-level logic
```python


# MOE_DP
```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 2211-2213: Function get_moe_data_parallel_world_size
```python
def get_moe_data_parallel_world_size():
    """Return world size for the moe data parallel group."""
    return get_moe_dp_group().world_size
```
**EN:** This callable implements `get_moe_data_parallel_world_size` and mainly retrieves a value or derived view. The docstring states: "Return world size for the moe data parallel group."
**CN:** 这一可调用对象实现了 `get_moe_data_parallel_world_size`，主要用于获取某个值或派生视图。

### Lines 2216-2218: Function get_moe_data_parallel_rank
```python
def get_moe_data_parallel_rank():
    """Return my rank for the moe data parallel group."""
    return get_moe_dp_group().rank_in_group
```
**EN:** This callable implements `get_moe_data_parallel_rank` and mainly retrieves a value or derived view. The docstring states: "Return my rank for the moe data parallel group."
**CN:** 这一可调用对象实现了 `get_moe_data_parallel_rank`，主要用于获取某个值或派生视图。

### Lines 2219-2221: Module-level logic
```python


# MOE_EP
```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 2222-2224: Function get_moe_expert_parallel_world_size
```python
def get_moe_expert_parallel_world_size():
    """Return world size for the moe expert parallel group."""
    return get_moe_ep_group().world_size
```
**EN:** This callable implements `get_moe_expert_parallel_world_size` and mainly retrieves a value or derived view. The docstring states: "Return world size for the moe expert parallel group."
**CN:** 这一可调用对象实现了 `get_moe_expert_parallel_world_size`，主要用于获取某个值或派生视图。

### Lines 2227-2229: Function get_moe_expert_parallel_rank
```python
def get_moe_expert_parallel_rank():
    """Return my rank for the moe expert parallel group."""
    return get_moe_ep_group().rank_in_group
```
**EN:** This callable implements `get_moe_expert_parallel_rank` and mainly retrieves a value or derived view. The docstring states: "Return my rank for the moe expert parallel group."
**CN:** 这一可调用对象实现了 `get_moe_expert_parallel_rank`，主要用于获取某个值或派生视图。

### Lines 2230-2232: Module-level logic
```python


# MOE_TP
```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 2233-2235: Function get_moe_tensor_parallel_world_size
```python
def get_moe_tensor_parallel_world_size():
    """Return world size for the moe tensor parallel group."""
    return get_moe_tp_group().world_size
```
**EN:** This callable implements `get_moe_tensor_parallel_world_size` and mainly retrieves a value or derived view. The docstring states: "Return world size for the moe tensor parallel group."
**CN:** 这一可调用对象实现了 `get_moe_tensor_parallel_world_size`，主要用于获取某个值或派生视图。

### Lines 2238-2240: Function get_moe_tensor_parallel_rank
```python
def get_moe_tensor_parallel_rank():
    """Return my rank for the moe tensor parallel group."""
    return get_moe_tp_group().rank_in_group
```
**EN:** This callable implements `get_moe_tensor_parallel_rank` and mainly retrieves a value or derived view. The docstring states: "Return my rank for the moe tensor parallel group."
**CN:** 这一可调用对象实现了 `get_moe_tensor_parallel_rank`，主要用于获取某个值或派生视图。

### Lines 2243-2284: Function destroy_model_parallel
```python
def destroy_model_parallel():
    """Set the groups to none and destroy them."""
    global _TP
    if _TP:
        _TP.destroy()
    _TP = None

    global _PP
    if _PP:
        _PP.destroy()
    _PP = None

    global _MOE_EP
    if _MOE_EP:
        _MOE_EP.destroy()
    _MOE_EP = None

    global _MOE_TP
    if _MOE_TP:
        _MOE_TP.destroy()
    _MOE_TP = None

    global _ATTN_CP
    global _MOE_DP
    # Destroy _MOE_DP before _ATTN_CP since it may alias _ATTN_CP.
    # Only destroy if not aliasing another group.
    if _MOE_DP and _MOE_DP is not _ATTN_CP and _MOE_DP is not _TP:
        _MOE_DP.destroy()
    _MOE_DP = None
    if _ATTN_CP:
        _ATTN_CP.destroy()
    _ATTN_CP = None

    global _ATTN_TP
    if _ATTN_TP:
        _ATTN_TP.destroy()
    _ATTN_TP = None

    global _PDMUX_PREFILL_TP_GROUP
    if _PDMUX_PREFILL_TP_GROUP:  # type: ignore[union-attr]
        _PDMUX_PREFILL_TP_GROUP.destroy()
    _PDMUX_PREFILL_TP_GROUP = None
```
**EN:** This callable implements `destroy_model_parallel` and mainly implements destroy model parallel. The docstring states: "Set the groups to none and destroy them."
**CN:** 这一可调用对象实现了 `destroy_model_parallel`，主要用于实现 destroy model parallel 相关逻辑。

### Lines 2287-2294: Function destroy_distributed_environment
```python
def destroy_distributed_environment():
    global _WORLD, _MODEL_PARALLEL_GROUP_TIMEOUT
    if _WORLD:
        _WORLD.destroy()
    _WORLD = None
    _MODEL_PARALLEL_GROUP_TIMEOUT = None
    if torch.distributed.is_initialized():
        torch.distributed.destroy_process_group()
```
**EN:** This callable implements `destroy_distributed_environment` and mainly implements destroy distributed environment. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `destroy_distributed_environment`，主要用于实现 destroy distributed environment 相关逻辑。 在这一范围内，它会协调分布式通信。

### Lines 2297-2321: Function cleanup_dist_env_and_memory
```python
def cleanup_dist_env_and_memory(shutdown_ray: bool = False):
    destroy_model_parallel()
    destroy_distributed_environment()
    with contextlib.suppress(AssertionError):
        torch.distributed.destroy_process_group()
    if shutdown_ray:
        import ray  # Lazy import Ray

        ray.shutdown()
    gc.collect()
    if not _is_cpu:
        if hasattr(torch, "cuda") and torch.cuda.is_available():
            torch.cuda.empty_cache()
            if hasattr(torch._C, "_host_emptyCache"):
                torch._C._host_emptyCache()
            else:
                logger.warning(
                    "torch._C._host_emptyCache() only available in Pytorch >=2.5"
                )
        elif hasattr(torch, "xpu") and torch.xpu.is_available():
            torch.xpu.empty_cache()
        elif hasattr(torch, "npu") and torch.npu.is_available():
            torch.npu.empty_cache()
        elif hasattr(torch, "musa") and torch.musa.is_available():
            torch.musa.empty_cache()
```
**EN:** This callable implements `cleanup_dist_env_and_memory`. It takes `shutdown_ray` and mainly implements cleanup dist env and memory. In this range it sets up imports and shared symbols; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `cleanup_dist_env_and_memory`。它接收 `shutdown_ray`，主要用于实现 cleanup dist env and memory 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；协调分布式通信。

### Lines 2324-2383: Function in_the_same_node_as (part 1/2)
```python
def in_the_same_node_as(pg: ProcessGroup, source_rank: int = 0) -> List[bool]:
    """
    This is a collective operation that returns if each rank is in the same node
    as the source rank. It tests if processes are attached to the same
    memory system (shared access to shared memory).
    """
    assert (
        torch.distributed.get_backend(pg) != torch.distributed.Backend.NCCL
    ), "in_the_same_node_as should be tested with a non-NCCL group."
    # local rank inside the group
    rank = torch.distributed.get_rank(group=pg)
    world_size = torch.distributed.get_world_size(group=pg)

    # local tensor in each process to store the result
    is_in_the_same_node = torch.tensor([0] * world_size, dtype=torch.int32)

    # global ranks of the processes in the group
    ranks = torch.distributed.get_process_group_ranks(pg)

    magic_message = b"magic_message"
    shm = None

    try:
        with contextlib.suppress(OSError):
            if rank == source_rank:
                # create a shared memory segment
                shm = shared_memory.SharedMemory(create=True, size=128)
                shm.buf[: len(magic_message)] = magic_message
                torch.distributed.broadcast_object_list(
                    [shm.name], src=ranks[source_rank], group=pg
                )
                is_in_the_same_node[rank] = 1
            else:
                # try to open the shared memory segment
                recv = [None]
                torch.distributed.broadcast_object_list(
                    recv, src=ranks[source_rank], group=pg
                )
                name = recv[0]
                # fix to https://stackoverflow.com/q/62748654/9191338
                # Python incorrectly tracks shared memory even if it is not
                # created by the process. The following patch is a workaround.
                with patch(
                    "multiprocessing.resource_tracker.register",
                    lambda *args, **kwargs: None,
                ):
                    shm = shared_memory.SharedMemory(name=name)
                if shm.buf[: len(magic_message)] == magic_message:
                    is_in_the_same_node[rank] = 1
    except Exception as e:
        logger.error("Error ignored in is_in_the_same_node: %s", e)
    finally:
        if shm:
            shm.close()

    torch.distributed.barrier(group=pg)

    # clean up the shared memory segment
    with contextlib.suppress(OSError):
        if rank == source_rank and shm:
```
**EN:** This callable implements `in_the_same_node_as`. It takes `pg`, `source_rank` and mainly implements in the same node as. The docstring states: "This is a collective operation that returns if each rank is in the same node as the source rank." This chunk is part 1 of 2 for the same logical block. In this range it performs defensive checks on invalid state; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `in_the_same_node_as`。它接收 `pg`, `source_rank`，主要用于实现 in the same node as 相关逻辑。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会对非法状态执行防御性检查；输出日志以便诊断；协调分布式通信。

### Lines 2384-2387: Function in_the_same_node_as (part 2/2)
```python
            shm.unlink()
    torch.distributed.all_reduce(is_in_the_same_node, group=pg)

    return [x == 1 for x in is_in_the_same_node.tolist()]
```
**EN:** This callable implements `in_the_same_node_as`. It takes `pg`, `source_rank` and mainly implements in the same node as. The docstring states: "This is a collective operation that returns if each rank is in the same node as the source rank." This chunk is part 2 of 2 for the same logical block. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `in_the_same_node_as`。它接收 `pg`, `source_rank`，主要用于实现 in the same node as 相关逻辑。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会协调分布式通信。

### Lines 2388-2394: Module-level constants and helpers
```python


vllm_get_pp_group = None
vllm_get_tp_group = None
vllm_get_world_group = None


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 2395-2413: Function monkey_patch_vllm_parallel_state
```python
def monkey_patch_vllm_parallel_state(reverse: bool = False):
    try:
        import vllm.distributed.parallel_state as vllm_parallel_state
    except ImportError:
        return

    global vllm_get_pp_group, vllm_get_tp_group, vllm_get_world_group
    if vllm_get_pp_group is None:
        vllm_get_pp_group = vllm_parallel_state.get_pp_group
        vllm_get_tp_group = vllm_parallel_state.get_tp_group
        vllm_get_world_group = vllm_parallel_state.get_world_group
    if reverse:
        setattr(vllm_parallel_state, "get_pp_group", vllm_get_pp_group)
        setattr(vllm_parallel_state, "get_tp_group", vllm_get_tp_group)
        setattr(vllm_parallel_state, "get_world_group", vllm_get_world_group)
    else:
        setattr(vllm_parallel_state, "get_pp_group", get_pp_group)
        setattr(vllm_parallel_state, "get_tp_group", get_tp_group)
        setattr(vllm_parallel_state, "get_world_group", get_world_group)
```
**EN:** This callable implements `monkey_patch_vllm_parallel_state`. It takes `reverse` and mainly implements monkey patch vllm parallel state. In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `monkey_patch_vllm_parallel_state`。它接收 `reverse`，主要用于实现 monkey patch vllm parallel state 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

## Key Concepts / 关键概念
- `REDUCE_OP_SUM`: module constant or capability flag / 模块常量或能力标记
- `get_torch_distributed_pg_options`: converts data into another representation / 将数据转换为另一种表示
- `GraphCaptureContext`: core class or state container / 核心类或状态容器
- `P2PWork`: core class or state container / 核心类或状态容器
- `_split_tensor_dict`: implements split tensor dict / 实现 split tensor dict 相关逻辑
- `_get_unique_name`: retrieves a value or derived view / 获取某个值或派生视图
- `_register_group`: registers hooks, handlers, or operators / 注册钩子、处理器或算子
- `inplace_all_reduce`: reduces or aggregates values / 归约或聚合数值
- `outplace_all_reduce`: reduces or aggregates values / 归约或聚合数值
- `reg_all_gather_into_tensor`: converts data into another representation / 将数据转换为另一种表示

## Dependencies / 依赖关系
- **Standard library / 标准库**: `contextlib`, `gc`, `logging`, `os`, `pickle`, `weakref`, `collections`, `dataclasses`, `datetime`, `multiprocessing`, `typing`, `unittest.mock`
- **Third-party / 第三方**: `torch`, `torch.distributed`, `torch_npu`, `ray`, `vllm.distributed.parallel_state`, `mooncake`, `mooncake.ep`
- **Internal modules / 内部模块**: `sglang.srt.compilation.compilation_config`, `sglang.srt.compilation.piecewise_context_manager`, `sglang.srt.distributed.utils`, `sglang.srt.environ`, `sglang.srt.utils`, `sglang.srt.utils.custom_op`, `sglang.srt.utils.network`, `sglang.srt.distributed.device_communicators.mooncake_transfer_engine`, `sglang.srt.layers.sampler`, `sglang.srt.distributed.device_communicators.custom_all_reduce`, `sglang.srt.distributed.device_communicators.pymscclpp`, `sglang.srt.distributed.device_communicators.pynccl` + 8 more
