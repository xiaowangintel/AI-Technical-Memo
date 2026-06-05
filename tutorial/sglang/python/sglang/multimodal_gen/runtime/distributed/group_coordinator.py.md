# group_coordinator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/distributed/group_coordinator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the distributed runtime layer. It centers on `GraphCaptureContext`, `GroupCoordinator`, and `PipelineGroupCoordinator`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于分布式运行时层。它围绕 `GraphCaptureContext`、`GroupCoordinator` 和 `PipelineGroupCoordinator` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 10-32: module setup and imports / 模块初始化与导入
```python
import pickle
from collections import namedtuple
from contextlib import contextmanager
from dataclasses import dataclass
from typing import Any, Dict, List, Optional, Tuple, Union

import torch
import torch.distributed
from torch.cuda import synchronize
from torch.distributed import Backend, ProcessGroup

from sglang.multimodal_gen.runtime.distributed.device_communicators.base_device_communicator import (
    DeviceCommunicatorBase,
)
from sglang.multimodal_gen.runtime.distributed.device_communicators.cpu_communicator import (
    CpuCommunicator,
)
from sglang.multimodal_gen.runtime.platforms import current_platform
from sglang.multimodal_gen.runtime.utils.logging_utils import (
    init_logger,
    suppress_stdout,
)
from sglang.srt.utils import is_shm_available
```
**EN:** This block establishes the module context and imports `pickle`, `collections`, `contextlib`, `dataclasses`, `typing`, and `torch`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `pickle`、`collections`、`contextlib`、`dataclasses`、`typing` 和 `torch`。这些依赖为后续实现提供所需符号。

### Lines 34-45: supporting statements / 辅助语句
```python
try:
    import torch_musa  # noqa: F401
    from torch_musa.core.device import synchronize
except ModuleNotFoundError:
    pass

logger = init_logger(__name__)

TensorMetadata = namedtuple("TensorMetadata", ["device", "dtype", "size"])


_group_name_counter: dict[str, int] = {}
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, `TensorMetadata`, and `_group_name_counter`. The code collaborates with `init_logger`, and `namedtuple`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger`、`TensorMetadata` 和 `_group_name_counter` 等名称。 代码会与 `init_logger` 和 `namedtuple` 协同工作。

### Lines 48-51: `get_local_torch_device` implementation / `get_local_torch_device` 实现
```python
def get_local_torch_device() -> torch.device:
    """Return the torch device for the current rank."""

    return current_platform.get_local_torch_device()
```
**EN:** This block defines function `get_local_torch_device`. Return the torch device for the current rank. Key calls include `current_platform.get_local_torch_device`.
**CN:** 该代码块定义了函数 `get_local_torch_device`。 它用于获取local torch device。 关键调用包括 `current_platform.get_local_torch_device`。

### Lines 54-64: `_get_unique_name` implementation / `_get_unique_name` 实现
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
**EN:** This block defines function `_get_unique_name`. Get a unique name for the group. Example: _get_unique_name("tp") -> "tp:0" _get_unique_name("tp") -> "tp:1" The implementation branches on conditions. Parameters such as `name` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_get_unique_name`。 它用于获取unique name。 实现中包含条件分支。 本段逻辑主要由 `name` 等参数驱动。

### Lines 67-108: `_split_tensor_dict` implementation / `_split_tensor_dict` 实现
```python
def _split_tensor_dict(
    tensor_dict: Dict[str, Union[torch.Tensor, Any]], prefix: str = ""
) -> Tuple[List[Tuple[str, Any]], List[torch.Tensor]]:
    """Split the tensor dictionary into two parts:
    1. A list of (key, value) pairs. If the value is a tensor, it is replaced
         by its metadata.
    2. A list of tensors.

    If the Tensor is nested under `tensor_dict["key1"]["key2"]`, the key of its
    metadata will be "key1%key2".
    """
    metadata_list: List[Tuple[str, Any]] = []
    tensor_list = []
    for key, value in tensor_dict.items():
        assert "%" not in key, (
            "Avoid having '%' in key "
            "as it is used as a separator for nested entries."
        )
        if isinstance(value, torch.Tensor):
            # Note: we cannot use `value.device` here,
            # because it contains not only the device type but also the device
            # index (e.g. "cuda:0"). We only need the device type.
            # receiving side will set the device index.
            device = value.device.type
            metadata_list.append(
                (
                    prefix + key,
                    TensorMetadata(device, value.dtype, value.size()),
                )
            )
            tensor_list.append(value)
        elif isinstance(value, dict):
            if len(value) == 0:
                metadata_list.append((prefix + key, value))
            inner_metadata_list, inner_tensor_list = _split_tensor_dict(
                value, prefix + key + "%"
            )
            metadata_list.extend(inner_metadata_list)
            tensor_list.extend(inner_tensor_list)
        else:
            metadata_list.append((prefix + key, value))
    return metadata_list, tensor_list
```
**EN:** This block defines function `_split_tensor_dict`. Split the tensor dictionary into two parts: 1. A list of (key, value) pairs. Key calls include `tensor_dict.items`, `isinstance`, `metadata_list.append`, `tensor_list.append`, and `_split_tensor_dict`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `tensor_dict`, and `prefix` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_split_tensor_dict`。 它用于拆分tensor dict。 关键调用包括 `tensor_dict.items`、`isinstance`、`metadata_list.append`、`tensor_list.append` 和 `_split_tensor_dict`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `tensor_dict` 和 `prefix` 等参数驱动。

### Lines 111-118: `_update_nested_dict` implementation / `_update_nested_dict` 实现
```python
def _update_nested_dict(nested_dict, flattened_key, value):
    key_splits = flattened_key.split("%")
    cur_dict = nested_dict
    for k in key_splits[:-1]:
        if k not in cur_dict:
            cur_dict[k] = {}
        cur_dict = cur_dict[k]
    cur_dict[key_splits[-1]] = value
```
**EN:** This block defines function `_update_nested_dict`. It updates nested dict. Key calls include `flattened_key.split`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `nested_dict`, `flattened_key`, and `value` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_update_nested_dict`。 它用于更新nested dict。 关键调用包括 `flattened_key.split`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `nested_dict`、`flattened_key` 和 `value` 等参数驱动。

### Lines 122-122: `GraphCaptureContext` class overview / `GraphCaptureContext` 类概览
```python
class GraphCaptureContext:
```
**EN:** This block defines class `GraphCaptureContext`. It encapsulates graph capture context behavior.
**CN:** 该代码块定义了类 `GraphCaptureContext`。 它用于封装 graph capture context 相关行为。

### Lines 123-123: supporting statements / 辅助语句
```python
    stream: torch.cuda.Stream | None
```
**EN:** This block gathers supporting statements inside `GraphCaptureContext`. It updates names such as `stream`.
**CN:** 该代码块汇集了位于 `GraphCaptureContext` 内部的辅助语句。 它会更新 `stream` 等名称。

### Lines 126-137: `GroupCoordinator` class overview / `GroupCoordinator` 类概览
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
```
**EN:** This block defines class `GroupCoordinator`. PyTorch ProcessGroup wrapper for a group of processes. PyTorch ProcessGroup is bound to one specific communication backend, e.g.
**CN:** 该代码块定义了类 `GroupCoordinator`。 它用于封装 group coordinator 相关行为。

### Lines 138-153: supporting statements / 辅助语句
```python
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
    local_rank: int  # local rank in the current node, used to assign devices
    rank_in_group: int  # rank inside the group
    cpu_group: ProcessGroup  # group for CPU communication
    device_group: ProcessGroup  # group for device communication
    use_device_communicator: bool  # whether to use device communicator
    device_communicator: DeviceCommunicatorBase  # device communicator
```
**EN:** This block gathers supporting statements inside `GroupCoordinator`. It updates names such as `rank`, `ranks`, `world_size`, `local_rank`, `rank_in_group`, and `cpu_group`.
**CN:** 该代码块汇集了位于 `GroupCoordinator` 内部的辅助语句。 它会更新 `rank`、`ranks`、`world_size`、`local_rank`、`rank_in_group` 和 `cpu_group` 等名称。

### Lines 155-219: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        group_ranks: List[List[int]],
        local_rank: int,
        torch_distributed_backend: Union[str, Backend],
        use_device_communicator: bool = True,
        use_message_queue_broadcaster: bool = False,
        group_name: str | None = None,
    ):
        self.unique_name = _get_unique_name(group_name)
        self.rank = torch.distributed.get_rank()
        self.local_rank = local_rank
        self.device_group = None
        self.cpu_group = None

        for ranks in group_ranks:
            device_group = torch.distributed.new_group(
                ranks, backend=torch_distributed_backend
            )
            # a group with `gloo` backend, to allow direct coordination between
            # processes through the CPU.
            with suppress_stdout():
                cpu_group = torch.distributed.new_group(ranks, backend="gloo")
            if self.rank in ranks:
                self.ranks = ranks
                self.world_size = len(ranks)
                self.rank_in_group = ranks.index(self.rank)
                self.device_group = device_group
                self.cpu_group = cpu_group

        assert self.cpu_group is not None, f"{group_ranks=}, {local_rank=}"
        assert self.device_group is not None

        # TODO: fix it for other platforms
        self.device = get_local_torch_device()

        self.use_device_communicator = use_device_communicator
        self.device_communicator: DeviceCommunicatorBase = None  # type: ignore
        if use_device_communicator and self.world_size > 1:
            # Platform-aware device communicator selection
            if current_platform.is_cuda_alike():
                from sglang.multimodal_gen.runtime.distributed.device_communicators.cuda_communicator import (
                    CudaCommunicator,
                )

                self.device_communicator = CudaCommunicator(
                    cpu_group=self.cpu_group,
                    device=self.device,
                    device_group=self.device_group,
                    unique_name=self.unique_name,
                )
            else:
                # For MPS and CPU, use the CPU communicator
                self.device_communicator = CpuCommunicator(
                    cpu_group=self.cpu_group,
                    device=self.device,
                    device_group=self.device_group,
                    unique_name=self.unique_name,
                )

        self.mq_broadcaster = None

        # TODO(will): check if this is needed
        # self.use_custom_op_call = current_platform.is_cuda_alike()
        self.use_custom_op_call = False
```
**EN:** This block defines method `__init__` on `GroupCoordinator`. It initializes the instance state. Key calls include `_get_unique_name`, `torch.distributed.get_rank`, `get_local_torch_device`, `torch.distributed.new_group`, and `current_platform.is_cuda_alike`. The implementation branches on conditions, iterates over collections or steps, uses context-managed resources. Parameters such as `group_ranks`, `local_rank`, `torch_distributed_backend`, `use_device_communicator`, and `use_message_queue_broadcaster` drive the behavior in this section.
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `_get_unique_name`、`torch.distributed.get_rank`、`get_local_torch_device`、`torch.distributed.new_group` 和 `current_platform.is_cuda_alike`。 实现中包含条件分支，会遍历集合或步骤，使用上下文管理资源。 本段逻辑主要由 `group_ranks`、`local_rank`、`torch_distributed_backend`、`use_device_communicator` 和 `use_message_queue_broadcaster` 等参数驱动。

### Lines 221-224: `first_rank` implementation / `first_rank` 实现
```python
    @property
    def first_rank(self):
        """Return the global rank of the first process in the group"""
        return self.ranks[0]
```
**EN:** This block defines method `first_rank` on `GroupCoordinator`. Return the global rank of the first process in the group
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `first_rank`。 它用于处理 first rank 相关逻辑。

### Lines 226-229: `last_rank` implementation / `last_rank` 实现
```python
    @property
    def last_rank(self):
        """Return the global rank of the last process in the group"""
        return self.ranks[-1]
```
**EN:** This block defines method `last_rank` on `GroupCoordinator`. Return the global rank of the last process in the group
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `last_rank`。 它用于处理 last rank 相关逻辑。

### Lines 231-234: `is_first_rank` implementation / `is_first_rank` 实现
```python
    @property
    def is_first_rank(self):
        """Return whether the caller is the first process in the group"""
        return self.rank == self.first_rank
```
**EN:** This block defines method `is_first_rank` on `GroupCoordinator`. Return whether the caller is the first process in the group
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `is_first_rank`。 它用于处理 is first rank 相关逻辑。

### Lines 236-239: `is_last_rank` implementation / `is_last_rank` 实现
```python
    @property
    def is_last_rank(self):
        """Return whether the caller is the last process in the group"""
        return self.rank == self.last_rank
```
**EN:** This block defines method `is_last_rank` on `GroupCoordinator`. Return whether the caller is the last process in the group
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `is_last_rank`。 它用于处理 is last rank 相关逻辑。

### Lines 241-246: `next_rank` implementation / `next_rank` 实现
```python
    @property
    def next_rank(self):
        """Return the global rank of the process that follows the caller"""
        rank_in_group = self.rank_in_group
        world_size = self.world_size
        return self.ranks[(rank_in_group + 1) % world_size]
```
**EN:** This block defines method `next_rank` on `GroupCoordinator`. Return the global rank of the process that follows the caller
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `next_rank`。 它用于处理 next rank 相关逻辑。

### Lines 248-253: `prev_rank` implementation / `prev_rank` 实现
```python
    @property
    def prev_rank(self):
        """Return the global rank of the process that precedes the caller"""
        rank_in_group = self.rank_in_group
        world_size = self.world_size
        return self.ranks[(rank_in_group - 1) % world_size]
```
**EN:** This block defines method `prev_rank` on `GroupCoordinator`. Return the global rank of the process that precedes the caller
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `prev_rank`。 它用于处理 prev rank 相关逻辑。

### Lines 255-260: `group_next_rank` implementation / `group_next_rank` 实现
```python
    @property
    def group_next_rank(self):
        """Return the group rank of the process that follows the caller"""
        rank_in_group = self.rank_in_group
        world_size = self.world_size
        return (rank_in_group + 1) % world_size
```
**EN:** This block defines method `group_next_rank` on `GroupCoordinator`. Return the group rank of the process that follows the caller
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `group_next_rank`。 它用于处理 group next rank 相关逻辑。

### Lines 262-267: `group_prev_rank` implementation / `group_prev_rank` 实现
```python
    @property
    def group_prev_rank(self):
        """Return the group rank of the process that precedes the caller"""
        rank_in_group = self.rank_in_group
        world_size = self.world_size
        return (rank_in_group - 1) % world_size
```
**EN:** This block defines method `group_prev_rank` on `GroupCoordinator`. Return the group rank of the process that precedes the caller
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `group_prev_rank`。 它用于处理 group prev rank 相关逻辑。

### Lines 269-274: `skip_rank` implementation / `skip_rank` 实现
```python
    @property
    def skip_rank(self):
        """Return the global rank of the process that skip connects with the caller"""
        rank_in_group = self.rank_in_group
        world_size = self.world_size
        return self.ranks[(world_size - rank_in_group - 1) % world_size]
```
**EN:** This block defines method `skip_rank` on `GroupCoordinator`. Return the global rank of the process that skip connects with the caller
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `skip_rank`。 它用于处理 skip rank 相关逻辑。

### Lines 276-281: `group_skip_rank` implementation / `group_skip_rank` 实现
```python
    @property
    def group_skip_rank(self):
        """Return the group rank of the process that skip connects with the caller"""
        rank_in_group = self.rank_in_group
        world_size = self.world_size
        return (world_size - rank_in_group - 1) % world_size
```
**EN:** This block defines method `group_skip_rank` on `GroupCoordinator`. Return the group rank of the process that skip connects with the caller
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `group_skip_rank`。 它用于处理 group skip rank 相关逻辑。

### Lines 283-305: `graph_capture` implementation / `graph_capture` 实现
```python
    @contextmanager
    def graph_capture(self, graph_capture_context: GraphCaptureContext | None = None):
        if current_platform.is_cuda_alike():
            if graph_capture_context is None:
                stream = torch.cuda.Stream()
                graph_capture_context = GraphCaptureContext(stream)
            else:
                stream = graph_capture_context.stream

            # ensure all initialization operations complete before attempting to
            # capture the graph on another stream
            curr_stream = torch.cuda.current_stream()
            if curr_stream != stream:
                stream.wait_stream(curr_stream)

            with torch.cuda.stream(stream):
                yield graph_capture_context
        else:
            # For non-CUDA platforms (MPS, CPU), just yield the context without stream management
            if graph_capture_context is None:
                # Create a dummy context for non-CUDA platforms
                graph_capture_context = GraphCaptureContext(None)
            yield graph_capture_context
```
**EN:** This block defines method `graph_capture` on `GroupCoordinator`. It handles graph capture logic. Key calls include `current_platform.is_cuda_alike`, `torch.cuda.current_stream`, `torch.cuda.Stream`, `GraphCaptureContext`, and `stream.wait_stream`. The implementation branches on conditions, uses context-managed resources. Parameters such as `graph_capture_context` drive the behavior in this section.
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `graph_capture`。 它用于处理 graph capture 相关逻辑。 关键调用包括 `current_platform.is_cuda_alike`、`torch.cuda.current_stream`、`torch.cuda.Stream`、`GraphCaptureContext` 和 `stream.wait_stream`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `graph_capture_context` 等参数驱动。

### Lines 307-312: `all_to_all_4D` implementation / `all_to_all_4D` 实现
```python
    def all_to_all_4D(
        self, input_: torch.Tensor, scatter_dim: int = 2, gather_dim: int = 1
    ) -> torch.Tensor:
        if self.world_size == 1:
            return input_
        return self.device_communicator.all_to_all_4D(input_, scatter_dim, gather_dim)
```
**EN:** This block defines method `all_to_all_4D` on `GroupCoordinator`. It handles all to all 4 d logic. Key calls include `self.device_communicator.all_to_all_4D`. The implementation branches on conditions. Parameters such as `input_`, `scatter_dim`, and `gather_dim` drive the behavior in this section.
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `all_to_all_4D`。 它用于处理 all to all 4 d 相关逻辑。 关键调用包括 `self.device_communicator.all_to_all_4D`。 实现中包含条件分支。 本段逻辑主要由 `input_`、`scatter_dim` 和 `gather_dim` 等参数驱动。

### Lines 314-342: `all_reduce` implementation / `all_reduce` 实现
```python
    def all_reduce(
        self,
        input_: torch.Tensor,
        op=torch._C._distributed_c10d.ReduceOp.SUM,
        async_op: bool = False,
    ) -> torch.Tensor:
        """
        NOTE: This operation will be applied in-place or out-of-place.
        Always assume this function modifies its input, but use the return
        value as the output.
        """
        # Bypass the function if we are using only 1 GPU.
        if self.world_size == 1:
            return input_
        else:
            if (
                current_platform.is_cpu()
                and is_shm_available(input_.dtype, self.world_size, len(self.ranks))
                and op is torch.distributed.ReduceOp.SUM
            ):
                # for CPU platform, intra-node case we could speedup with shared memory based comm ops
                torch.ops.sgl_kernel.shm_allreduce(
                    input_, int(torch.distributed.ReduceOp.SUM)
                )
            else:
                torch.distributed.all_reduce(
                    input_, op=op, group=self.device_group, async_op=async_op
                )
        return input_
```
**EN:** This block defines method `all_reduce` on `GroupCoordinator`. NOTE: This operation will be applied in-place or out-of-place. Always assume this function modifies its input, but use the return value as the output. Key calls include `current_platform.is_cpu`, `is_shm_available`, `torch.ops.sgl_kernel.shm_allreduce`, `torch.distributed.all_reduce`, and `len`. The implementation branches on conditions. Parameters such as `input_`, `op`, and `async_op` drive the behavior in this section.
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `all_reduce`。 它用于处理 all reduce 相关逻辑。 关键调用包括 `current_platform.is_cpu`、`is_shm_available`、`torch.ops.sgl_kernel.shm_allreduce`、`torch.distributed.all_reduce` 和 `len`。 实现中包含条件分支。 本段逻辑主要由 `input_`、`op` 和 `async_op` 等参数驱动。

### Lines 344-397: `all_gather` implementation / `all_gather` 实现
```python
    def all_gather(
        self, input_: torch.Tensor, dim: int = 0, separate_tensors: bool = False
    ) -> Union[torch.Tensor, List[torch.Tensor]]:
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
        # Allocate output tensor.
        input_size = list(input_.size())
        input_size[0] *= world_size
        output_tensor = torch.empty(
            input_size, dtype=input_.dtype, device=input_.device
        )

        # All-gather.
        if current_platform.is_cpu() and is_shm_available(
            input_.dtype, self.world_size, len(self.ranks)
        ):
            return torch.ops.sgl_kernel.shm_allgather(input_, dim)
        else:
            torch.distributed.all_gather_into_tensor(
                output_tensor, input_, group=self.device_group
            )

        if dim != 0:
            input_size[0] //= world_size
            output_tensor = output_tensor.reshape(
                [
                    world_size,
                ]
                + input_size
            )
            output_tensor = output_tensor.movedim(0, dim)

        if separate_tensors:
            tensor_list = [
                output_tensor.reshape(-1)
                .narrow(0, input_.numel() * i, input_.numel())
                .view_as(input_)
                for i in range(world_size)
            ]
            return tensor_list
        else:
            input_size = list(input_.size())
            input_size[dim] = input_size[dim] * world_size
            # Reshape
            output_tensor = output_tensor.reshape(input_size)
            return output_tensor
```
**EN:** This block defines method `all_gather` on `GroupCoordinator`. It handles all gather logic. Key calls include `list`, `torch.empty`, `input_.dim`, `input_.size`, and `current_platform.is_cpu`. The implementation branches on conditions. Parameters such as `input_`, `dim`, and `separate_tensors` drive the behavior in this section.
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `all_gather`。 它用于处理 all gather 相关逻辑。 关键调用包括 `list`、`torch.empty`、`input_.dim`、`input_.size` 和 `current_platform.is_cpu`。 实现中包含条件分支。 本段逻辑主要由 `input_`、`dim` 和 `separate_tensors` 等参数驱动。

### Lines 399-428: `gather` implementation / `gather` 实现
```python
    def gather(self, input_: torch.Tensor, dst: int = 0, dim: int = -1) -> torch.Tensor:
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
**EN:** This block defines method `gather` on `GroupCoordinator`. NOTE: We assume that the input tensor is on the same device across all the ranks. NOTE: `dst` is the local rank of the destination rank. Key calls include `torch.distributed.gather`, `input_.dim`, `torch.cat`, `input_.size`, and `torch.empty_like`. The implementation branches on conditions. Parameters such as `input_`, `dst`, and `dim` drive the behavior in this section.
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `gather`。 它用于处理 gather 相关逻辑。 关键调用包括 `torch.distributed.gather`、`input_.dim`、`torch.cat`、`input_.size` 和 `torch.empty_like`。 实现中包含条件分支。 本段逻辑主要由 `input_`、`dst` 和 `dim` 等参数驱动。

### Lines 430-448: `broadcast` implementation / `broadcast` 实现
```python
    def broadcast(self, input_: torch.Tensor, src: int = 0, async_op: bool = False):
        """Broadcast the input tensor.
        NOTE: `src` is the local rank of the source rank.
        """
        assert src < self.world_size, f"Invalid src rank ({src})"

        # Bypass the function if we are using only 1 GPU.
        if self.world_size == 1:
            return input_
        # Broadcast.
        if not input_.is_contiguous():
            input_ = input_.contiguous()
        torch.distributed.broadcast(
            input_,
            src=self.ranks[src],
            group=self.device_group,
            async_op=async_op,
        )
        return input_
```
**EN:** This block defines method `broadcast` on `GroupCoordinator`. Broadcast the input tensor. NOTE: `src` is the local rank of the source rank. Key calls include `torch.distributed.broadcast`, `input_.is_contiguous`, and `input_.contiguous`. The implementation branches on conditions. Parameters such as `input_`, `src`, and `async_op` drive the behavior in this section.
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `broadcast`。 它用于处理 broadcast 相关逻辑。 关键调用包括 `torch.distributed.broadcast`、`input_.is_contiguous` 和 `input_.contiguous`。 实现中包含条件分支。 本段逻辑主要由 `input_`、`src` 和 `async_op` 等参数驱动。

### Lines 450-472: `broadcast_object` implementation / `broadcast_object` 实现
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
            assert src == 0, "Shared memory broadcaster only supports src=0"
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
**EN:** This block defines method `broadcast_object` on `GroupCoordinator`. Broadcast the input object. NOTE: `src` is the local rank of the source rank. Key calls include `self.mq_broadcaster.broadcast_object`, and `torch.distributed.broadcast_object_list`. The implementation branches on conditions. Parameters such as `obj`, and `src` drive the behavior in this section.
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `broadcast_object`。 它用于处理 broadcast object 相关逻辑。 关键调用包括 `self.mq_broadcaster.broadcast_object` 和 `torch.distributed.broadcast_object_list`。 实现中包含条件分支。 本段逻辑主要由 `obj` 和 `src` 等参数驱动。

### Lines 474-492: `broadcast_object_list` implementation / `broadcast_object_list` 实现
```python
    def broadcast_object_list(
        self,
        obj_list: List[Any],
        src: int = 0,
        group: Optional[ProcessGroup] = None,
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
**EN:** This block defines method `broadcast_object_list` on `GroupCoordinator`. Broadcast the input object list. NOTE: `src` is the local rank of the source rank. Key calls include `torch.distributed.broadcast_object_list`. The implementation branches on conditions. Parameters such as `obj_list`, `src`, and `group` drive the behavior in this section.
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `broadcast_object_list`。 它用于处理 broadcast object list 相关逻辑。 关键调用包括 `torch.distributed.broadcast_object_list`。 实现中包含条件分支。 本段逻辑主要由 `obj_list`、`src` 和 `group` 等参数驱动。

### Lines 494-519: `send_object` implementation / `send_object` 实现
```python
    def send_object(self, obj: Any, dst: int) -> None:
        """Send the input object list to the destination rank."""
        """NOTE: `dst` is the local rank of the destination rank."""

        assert dst < self.world_size, f"Invalid dst rank ({dst})"

        assert dst != self.rank, (
            "Invalid destination rank. Destination rank is the same "
            "as the current rank."
        )

        # Serialize object to tensor and get the size as well
        object_tensor = torch.frombuffer(pickle.dumps(obj), dtype=torch.uint8)

        size_tensor = torch.tensor(
            [object_tensor.numel()], dtype=torch.long, device="cpu"
        )

        # Send object size

        torch.distributed.send(size_tensor, dst=self.ranks[dst], group=self.cpu_group)

        # Send object
        torch.distributed.send(object_tensor, dst=self.ranks[dst], group=self.cpu_group)

        return None
```
**EN:** This block defines method `send_object` on `GroupCoordinator`. Send the input object list to the destination rank. Key calls include `torch.frombuffer`, `torch.tensor`, `torch.distributed.send`, `pickle.dumps`, and `object_tensor.numel`. Parameters such as `obj`, and `dst` drive the behavior in this section.
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `send_object`。 它用于处理 send object 相关逻辑。 关键调用包括 `torch.frombuffer`、`torch.tensor`、`torch.distributed.send`、`pickle.dumps` 和 `object_tensor.numel`。 本段逻辑主要由 `obj` 和 `dst` 等参数驱动。

### Lines 521-555: `recv_object` implementation / `recv_object` 实现
```python
    def recv_object(self, src: int) -> Any:
        """Receive the input object list from the source rank."""
        """NOTE: `src` is the local rank of the source rank."""

        assert src < self.world_size, f"Invalid src rank ({src})"

        assert (
            src != self.rank
        ), "Invalid source rank. Source rank is the same as the current rank."

        size_tensor = torch.empty(1, dtype=torch.long, device="cpu")

        # Receive object size
        rank_size = torch.distributed.recv(
            size_tensor, src=self.ranks[src], group=self.cpu_group
        )

        # Tensor to receive serialized objects into.
        object_tensor = torch.empty(  # type: ignore[call-overload]
            size_tensor.item(),  # type: ignore[arg-type]
            dtype=torch.uint8,
            device="cpu",
        )

        rank_object = torch.distributed.recv(
            object_tensor, src=self.ranks[src], group=self.cpu_group
        )

        assert (
            rank_object == rank_size
        ), "Received object sender rank does not match the size sender rank."

        obj = pickle.loads(object_tensor.numpy().tobytes())

        return obj
```
**EN:** This block defines method `recv_object` on `GroupCoordinator`. Receive the input object list from the source rank. Key calls include `torch.empty`, `torch.distributed.recv`, `pickle.loads`, `size_tensor.item`, and `object_tensor.numpy.tobytes`. Parameters such as `src` drive the behavior in this section.
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `recv_object`。 它用于处理 recv object 相关逻辑。 关键调用包括 `torch.empty`、`torch.distributed.recv`、`pickle.loads`、`size_tensor.item` 和 `object_tensor.numpy.tobytes`。 本段逻辑主要由 `src` 等参数驱动。

### Lines 557-635: `broadcast_tensor_dict` implementation / `broadcast_tensor_dict` 实现
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
        src = self.ranks[src]

        rank = self.rank
        if rank == src:
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
                        tensor, src=src, group=metadata_group, async_op=True
                    )
                else:
                    # use group for GPU tensors
                    handle = torch.distributed.broadcast(
                        tensor, src=src, group=group, async_op=True
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
                        _update_nested_dict(tensor_dict, key, tensor)
                        continue
                    if tensor.is_cpu:
                        # use metadata_group for CPU tensors
                        handle = torch.distributed.broadcast(
                            tensor, src=src, group=metadata_group, async_op=True
                        )
                    else:
                        # use group for GPU tensors
                        handle = torch.distributed.broadcast(
                            tensor, src=src, group=group, async_op=True
                        )
                    async_handles.append(handle)
                    _update_nested_dict(tensor_dict, key, tensor)
                else:
                    _update_nested_dict(tensor_dict, key, value)
            for async_handle in async_handles:
                async_handle.wait()
        return tensor_dict
```
**EN:** This block defines method `broadcast_tensor_dict` on `GroupCoordinator`. Broadcast the input tensor dictionary. NOTE: `src` is the local rank of the source rank. Key calls include `isinstance`, `_split_tensor_dict`, `self.broadcast_object`, `torch.distributed.is_initialized`, and `async_handles.append`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `tensor_dict`, `src`, `group`, and `metadata_group` drive the behavior in this section.
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `broadcast_tensor_dict`。 它用于处理 broadcast tensor dict 相关逻辑。 关键调用包括 `isinstance`、`_split_tensor_dict`、`self.broadcast_object`、`torch.distributed.is_initialized` 和 `async_handles.append`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `tensor_dict`、`src`、`group` 和 `metadata_group` 等参数驱动。

### Lines 637-677: `send_tensor_dict` implementation / `send_tensor_dict` 实现
```python
    def send_tensor_dict(
        self,
        tensor_dict: Dict[str, Union[torch.Tensor, Any]],
        dst: Optional[int] = None,
    ) -> Optional[Dict[str, Union[torch.Tensor, Any]]]:
        """Send the input tensor dictionary.
        NOTE: `dst` is the local rank of the source rank.
        """
        # Bypass the function if we are using only 1 GPU.
        if not torch.distributed.is_initialized() or self.world_size == 1:
            return tensor_dict

        group = self.device_group
        metadata_group = self.cpu_group

        if dst is None:
            dst = self.group_next_rank
        assert dst < self.world_size, f"Invalid dst rank ({dst})"

        metadata_list: List[Tuple[Any, Any]] = []
        assert isinstance(
            tensor_dict, dict
        ), f"Expecting a dictionary, got {type(tensor_dict)}"
        metadata_list, tensor_list = _split_tensor_dict(tensor_dict)
        # `metadata_list` lives in CPU memory.
        # `send_object_list` has serialization & deserialization,
        # all happening on CPU. Therefore, we can use the CPU group.
        self.send_object(metadata_list, dst=dst)
        for tensor in tensor_list:
            if tensor.numel() == 0:
                # Skip sending empty tensors.
                continue
            if tensor.is_cpu:
                # use metadata_group for CPU tensors
                torch.distributed.send(
                    tensor, dst=self.ranks[dst], group=metadata_group
                )
            else:
                # use group for GPU tensors
                torch.distributed.send(tensor, dst=self.ranks[dst], group=group)
        return None
```
**EN:** This block defines method `send_tensor_dict` on `GroupCoordinator`. Send the input tensor dictionary. NOTE: `dst` is the local rank of the source rank. Key calls include `isinstance`, `_split_tensor_dict`, `self.send_object`, `torch.distributed.is_initialized`, and `type`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `tensor_dict`, and `dst` drive the behavior in this section.
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `send_tensor_dict`。 它用于处理 send tensor dict 相关逻辑。 关键调用包括 `isinstance`、`_split_tensor_dict`、`self.send_object`、`torch.distributed.is_initialized` 和 `type`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `tensor_dict` 和 `dst` 等参数驱动。

### Lines 679-716: `recv_tensor_dict` implementation / `recv_tensor_dict` 实现
```python
    def recv_tensor_dict(
        self, src: Optional[int] = None
    ) -> Optional[Dict[str, Union[torch.Tensor, Any]]]:
        """Recv the input tensor dictionary.
        NOTE: `src` is the local rank of the source rank.
        """
        # Bypass the function if we are using only 1 GPU.
        if not torch.distributed.is_initialized() or self.world_size == 1:
            return None

        group = self.device_group
        metadata_group = self.cpu_group

        if src is None:
            src = self.group_prev_rank
        assert src < self.world_size, f"Invalid src rank ({src})"

        recv_metadata_list = self.recv_object(src=src)
        tensor_dict: Dict[str, Any] = {}
        for key, value in recv_metadata_list:
            if isinstance(value, TensorMetadata):
                tensor = torch.empty(value.size, dtype=value.dtype, device=value.device)
                if tensor.numel() == 0:
                    # Skip broadcasting empty tensors.
                    _update_nested_dict(tensor_dict, key, tensor)
                    continue
                if tensor.is_cpu:
                    # use metadata_group for CPU tensors
                    torch.distributed.recv(
                        tensor, src=self.ranks[src], group=metadata_group
                    )
                else:
                    # use group for GPU tensors
                    torch.distributed.recv(tensor, src=self.ranks[src], group=group)
                _update_nested_dict(tensor_dict, key, tensor)
            else:
                _update_nested_dict(tensor_dict, key, value)
        return tensor_dict
```
**EN:** This block defines method `recv_tensor_dict` on `GroupCoordinator`. Recv the input tensor dictionary. NOTE: `src` is the local rank of the source rank. Key calls include `self.recv_object`, `isinstance`, `torch.distributed.is_initialized`, `torch.empty`, and `_update_nested_dict`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `src` drive the behavior in this section.
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `recv_tensor_dict`。 它用于处理 recv tensor dict 相关逻辑。 关键调用包括 `self.recv_object`、`isinstance`、`torch.distributed.is_initialized`、`torch.empty` 和 `_update_nested_dict`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `src` 等参数驱动。

### Lines 718-725: `barrier` implementation / `barrier` 实现
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
**EN:** This block defines method `barrier` on `GroupCoordinator`. Barrier synchronization among the group. NOTE: don't use `device_group` here! Key calls include `torch.distributed.barrier`.
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `barrier`。 它用于处理 barrier 相关逻辑。 关键调用包括 `torch.distributed.barrier`。

### Lines 727-741: `send` implementation / `send` 实现
```python
    def send(self, tensor: torch.Tensor, dst: Optional[int] = None) -> None:
        """Sends a tensor to the destination rank in a non-blocking way"""
        """NOTE: `dst` is the rank_in_group of the destination rank."""
        if dst is None:
            dst = self.group_next_rank

        torch.distributed.send(
            tensor,
            self.ranks[dst],
            group=(
                self.device_groups[self.rank_in_group % 2]
                if self.world_size == 2
                else self.device_group
            ),
        )
```
**EN:** This block defines method `send` on `GroupCoordinator`. Sends a tensor to the destination rank in a non-blocking way Key calls include `torch.distributed.send`. The implementation branches on conditions. Parameters such as `tensor`, and `dst` drive the behavior in this section.
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `send`。 它用于处理 send 相关逻辑。 关键调用包括 `torch.distributed.send`。 实现中包含条件分支。 本段逻辑主要由 `tensor` 和 `dst` 等参数驱动。

### Lines 743-761: `recv` implementation / `recv` 实现
```python
    def recv(
        self, size: torch.Size, dtype: torch.dtype, src: Optional[int] = None
    ) -> torch.Tensor:
        """Receives a tensor from the src rank."""
        """NOTE: `src` is the rank_in_group of the source rank."""
        if src is None:
            src = self.group_prev_rank

        tensor = torch.empty(size, dtype=dtype, device=self.device)
        torch.distributed.recv(
            tensor,
            self.ranks[src],
            (
                self.device_groups[(self.rank_in_group + 1) % 2]
                if self.world_size == 2
                else self.device_group
            ),
        )
        return tensor
```
**EN:** This block defines method `recv` on `GroupCoordinator`. Receives a tensor from the src rank. Key calls include `torch.empty`, and `torch.distributed.recv`. The implementation branches on conditions. Parameters such as `size`, `dtype`, and `src` drive the behavior in this section.
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `recv`。 它用于处理 recv 相关逻辑。 关键调用包括 `torch.empty` 和 `torch.distributed.recv`。 实现中包含条件分支。 本段逻辑主要由 `size`、`dtype` 和 `src` 等参数驱动。

### Lines 763-773: `destroy` implementation / `destroy` 实现
```python
    def destroy(self) -> None:
        if self.device_group is not None:
            torch.distributed.destroy_process_group(self.device_group)
            self.device_group = None
        if self.cpu_group is not None:
            torch.distributed.destroy_process_group(self.cpu_group)
            self.cpu_group = None
        if self.device_communicator is not None:
            self.device_communicator.destroy()
        if self.mq_broadcaster is not None:
            self.mq_broadcaster = None
```
**EN:** This block defines method `destroy` on `GroupCoordinator`. It handles destroy logic. Key calls include `torch.distributed.destroy_process_group`, and `self.device_communicator.destroy`. The implementation branches on conditions.
**CN:** 该代码块定义了 `GroupCoordinator` 的方法 `destroy`。 它用于处理 destroy 相关逻辑。 关键调用包括 `torch.distributed.destroy_process_group` 和 `self.device_communicator.destroy`。 实现中包含条件分支。

### Lines 776-794: `PipelineGroupCoordinator` class overview / `PipelineGroupCoordinator` 类概览
```python
class PipelineGroupCoordinator(GroupCoordinator):
    """
    available attributes:
    rank: int  # global rank
    ranks: List[int]  # global ranks in the group
    world_size: int  # size of the group
    difference between `local_rank` and `rank_in_group`:
    if we have a group of size 4 across two nodes:
    Process | Node | Rank | Local Rank | Rank in Group
      0     |   0  |  0   |     0      |       0
      1     |   0  |  1   |     1      |       1
      2     |   1  |  2   |     0      |       2
      3     |   1  |  3   |     1      |       3
    local_rank: int  # local rank used to assign devices
    rank_in_group: int  # rank inside the group
    cpu_group: ProcessGroup  # group for CPU communication
    device_group: ProcessGroup  # group for device communication
    """
```
**EN:** This block defines class `PipelineGroupCoordinator`. available attributes: rank: int # global rank ranks: List[int] # global ranks in the group world_size: int # size of the group difference between `local_rank` and `rank_in_group`: if we have a group of size 4 across two nodes: Process | Node | Rank | Local Rank | Rank in Group 0 | 0 | 0 | 0 | 0 1 | 0 | 1 | 1 | 1 2 | 1 | 2 | 0 | 2 3 | 1 | 3 | 1 | 3 local_rank: int # local rank used to assign devices rank_in_group: int # rank inside the group cpu_group: ProcessGroup # group for CPU communication device_group: ProcessGroup # group for device communication It inherits from `GroupCoordinator`.
**CN:** 该代码块定义了类 `PipelineGroupCoordinator`。 它用于封装 pipeline group coordinator 相关行为。 它继承自 `GroupCoordinator`。

### Lines 795-885: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        group_ranks: List[List[int]],
        local_rank: int,
        torch_distributed_backend: Union[str, Backend],
        group_name: str | None = None,
    ):
        super().__init__(
            group_ranks=group_ranks,
            local_rank=local_rank,
            torch_distributed_backend=torch_distributed_backend,
            group_name=group_name,
        )
        self.rank = torch.distributed.get_rank()
        self.local_rank = local_rank
        self.device_group = None
        self.cpu_group = None
        self.cpu_groups = []
        self.device_groups = []
        if len(group_ranks[0]) > 2 or len(group_ranks[0]) == 1:
            for ranks in group_ranks:
                device_group = torch.distributed.new_group(
                    ranks, backend=torch_distributed_backend
                )
                # a group with `gloo` backend, to allow direct coordination between
                # processes through the CPU.
                with suppress_stdout():
                    cpu_group = torch.distributed.new_group(ranks, backend="gloo")
                if self.rank in ranks:
                    self.ranks = ranks
                    self.world_size = len(ranks)
                    self.rank_in_group = ranks.index(self.rank)
                    self.device_group = device_group
                    self.cpu_group = cpu_group
        # when pipeline parallelism is 2, we need to create two groups to avoid
        #   communication stall.
        # *_group_0_1 represents the group for communication from device 0 to
        #   device 1.
        # *_group_1_0 represents the group for communication from device 1 to
        #   device 0.
        elif len(group_ranks[0]) == 2:
            for ranks in group_ranks:
                device_group_0_1 = torch.distributed.new_group(
                    ranks, backend=torch_distributed_backend
                )
                device_group_1_0 = torch.distributed.new_group(
                    ranks, backend=torch_distributed_backend
                )
                # a group with `gloo` backend, to allow direct coordination between
                # processes through the CPU.
                with suppress_stdout():
                    cpu_group_0_1 = torch.distributed.new_group(ranks, backend="gloo")
                    cpu_group_1_0 = torch.distributed.new_group(ranks, backend="gloo")
                if self.rank in ranks:
                    self.ranks = ranks
                    self.world_size = len(ranks)
                    self.rank_in_group = ranks.index(self.rank)
                    self.device_groups = [device_group_0_1, device_group_1_0]
                    self.cpu_groups = [cpu_group_0_1, cpu_group_1_0]
                    self.device_group = device_group_0_1
                    self.cpu_group = cpu_group_0_1

        assert self.cpu_group is not None
        assert self.device_group is not None

        self.device = current_platform.get_device(local_rank)

        self.recv_buffer_set: bool = False
        self.recv_tasks_queue: List[Tuple[str, int]] = []
        self.receiving_tasks: List[Tuple[torch.distributed.Work, str, int]] = []
        self.dtype: Optional[torch.dtype] = None
        self.num_pipefusion_patches: Optional[int] = None

        self.recv_shape: Dict[str, Dict[int, torch.Size]] = {}
        self.send_shape: Dict[str, Dict[int, torch.Size]] = {}
        self.recv_buffer: Dict[str, Dict[int, torch.Size]] = {}

        self.skip_tensor_recv_buffer_set: bool = False
        self.recv_skip_tasks_queue: List[Union[int, Tuple[str, int]]] = []
        self.receiving_skip_tasks: List[Tuple[torch.distributed.Work, str, int]] = []
        self.skip_tensor_recv_buffer: Optional[
            Union[List[torch.Tensor], torch.Tensor]
        ] = None
        self.skip_device_group = None
        for ranks in group_ranks:
            skip_device_group = torch.distributed.new_group(
                ranks, backend=torch_distributed_backend
            )
            if self.rank in ranks:
                self.skip_device_group = skip_device_group
        assert self.skip_device_group is not None
```
**EN:** This block defines method `__init__` on `PipelineGroupCoordinator`. It initializes the instance state. Key calls include `super.__init__`, `torch.distributed.get_rank`, `current_platform.get_device`, `torch.distributed.new_group`, and `super`. The implementation branches on conditions, iterates over collections or steps, uses context-managed resources. Parameters such as `group_ranks`, `local_rank`, `torch_distributed_backend`, and `group_name` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`torch.distributed.get_rank`、`current_platform.get_device`、`torch.distributed.new_group` 和 `super`。 实现中包含条件分支，会遍历集合或步骤，使用上下文管理资源。 本段逻辑主要由 `group_ranks`、`local_rank`、`torch_distributed_backend` 和 `group_name` 等参数驱动。

### Lines 887-896: `reset_buffer` implementation / `reset_buffer` 实现
```python
    def reset_buffer(self):
        self.recv_tasks_queue = []
        self.receiving_tasks = []
        self.recv_shape = {}
        self.send_shape = {}
        self.recv_buffer = {}

        self.recv_skip_tasks_queue = []
        self.receiving_skip_tasks = []
        self.skip_tensor_recv_buffer = {}
```
**EN:** This block defines method `reset_buffer` on `PipelineGroupCoordinator`. It handles reset buffer logic.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `reset_buffer`。 它用于处理 reset buffer 相关逻辑。

### Lines 898-899: `set_config` implementation / `set_config` 实现
```python
    def set_config(self, dtype: torch.dtype):
        self.dtype = dtype
```
**EN:** This block defines method `set_config` on `PipelineGroupCoordinator`. It configures config. Parameters such as `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `set_config`。 它用于设置config。 本段逻辑主要由 `dtype` 等参数驱动。

### Lines 901-921: `set_recv_buffer` implementation / `set_recv_buffer` 实现
```python
    def set_recv_buffer(
        self,
        num_pipefusion_patches: int,
        patches_shape_list: List[List[int]],
        feature_map_shape: List[int],
        dtype: torch.dtype,
    ):
        assert isinstance(dtype, torch.dtype), "dtype must be a torch.dtype object"
        assert (
            isinstance(num_pipefusion_patches, int) and num_pipefusion_patches >= 1
        ), "num_pipefusion_patches must be greater than or equal to 1"
        self.dtype = dtype
        self.num_pipefusion_patches = num_pipefusion_patches
        self.recv_buffer = [
            torch.zeros(*shape, dtype=self.dtype, device=self.device)
            for shape in patches_shape_list
        ]
        self.recv_buffer.append(
            torch.zeros(*feature_map_shape, dtype=self.dtype, device=self.device)
        )
        self.recv_buffer_set = True
```
**EN:** This block defines method `set_recv_buffer` on `PipelineGroupCoordinator`. It configures recv buffer. Key calls include `isinstance`, `self.recv_buffer.append`, and `torch.zeros`. Parameters such as `num_pipefusion_patches`, `patches_shape_list`, `feature_map_shape`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `set_recv_buffer`。 它用于设置recv buffer。 关键调用包括 `isinstance`、`self.recv_buffer.append` 和 `torch.zeros`。 本段逻辑主要由 `num_pipefusion_patches`、`patches_shape_list`、`feature_map_shape` 和 `dtype` 等参数驱动。

### Lines 923-933: `set_extra_tensors_recv_buffer` implementation / `set_extra_tensors_recv_buffer` 实现
```python
    def set_extra_tensors_recv_buffer(
        self,
        name: str,
        shape: List[int],
        num_buffers: int = 1,
        dtype: torch.dtype = torch.float16,
    ):
        self.extra_tensors_recv_buffer[name] = [
            torch.zeros(*shape, dtype=dtype, device=self.device)
            for _ in range(num_buffers)
        ]
```
**EN:** This block defines method `set_extra_tensors_recv_buffer` on `PipelineGroupCoordinator`. It configures extra tensors recv buffer. Key calls include `torch.zeros`, and `range`. Parameters such as `name`, `shape`, `num_buffers`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `set_extra_tensors_recv_buffer`。 它用于设置extra tensors recv buffer。 关键调用包括 `torch.zeros` 和 `range`。 本段逻辑主要由 `name`、`shape`、`num_buffers` 和 `dtype` 等参数驱动。

### Lines 935-985: `_check_shape_and_buffer` implementation / `_check_shape_and_buffer` 实现
```python
    def _check_shape_and_buffer(
        self,
        tensor_send_to_next=None,
        recv_prev=False,
        name: Optional[str] = None,
        segment_idx: int = 0,
    ):
        send_flag = False
        name = name or "latent"
        if tensor_send_to_next is not None:
            shape_list = self.send_shape.get(name, None)
            if shape_list is None:
                self.send_shape[name] = {segment_idx: tensor_send_to_next.shape}
                send_flag = True
            elif shape_list.get(segment_idx, None) is None:
                self.send_shape[name][segment_idx] = tensor_send_to_next.shape
                send_flag = True

        recv_flag = False
        if recv_prev:
            shape_list = self.recv_shape.get(name, None)
            if shape_list is None:
                recv_flag = True
            elif shape_list.get(segment_idx, None) is None:
                recv_flag = True

        recv_prev_shape = self._communicate_shapes(
            tensor_send_to_next=tensor_send_to_next if send_flag else None,
            recv_prev=recv_flag,
        )

        if recv_flag:
            if self.recv_shape.get(name, None) is None:
                self.recv_shape[name] = {segment_idx: recv_prev_shape}
            else:
                self.recv_shape[name][segment_idx] = recv_prev_shape

            if self.recv_buffer.get(name, None) is None:
                self.recv_buffer[name] = {
                    segment_idx: torch.zeros(
                        recv_prev_shape, device=self.device, dtype=self.dtype
                    )
                }
            else:
                if self.recv_buffer[name].get(segment_idx, None) is not None:
                    logger.warning(
                        f"Recv buffer [name: {name}, segment_idx: {segment_idx}] already exist. updating..."
                    )
                self.recv_buffer[name][segment_idx] = torch.zeros(
                    recv_prev_shape, device=self.device, dtype=self.dtype
                )
```
**EN:** This block defines method `_check_shape_and_buffer` on `PipelineGroupCoordinator`. It checks shape and buffer. Key calls include `self._communicate_shapes`, `self.send_shape.get`, `self.recv_shape.get`, `self.recv_buffer.get`, and `torch.zeros`. The implementation branches on conditions. Parameters such as `tensor_send_to_next`, `recv_prev`, `name`, and `segment_idx` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `_check_shape_and_buffer`。 它用于检查shape and buffer。 关键调用包括 `self._communicate_shapes`、`self.send_shape.get`、`self.recv_shape.get`、`self.recv_buffer.get` 和 `torch.zeros`。 实现中包含条件分支。 本段逻辑主要由 `tensor_send_to_next`、`recv_prev`、`name` 和 `segment_idx` 等参数驱动。

### Lines 987-1072: `_communicate_shapes` implementation / `_communicate_shapes` 实现
```python
    def _communicate_shapes(self, tensor_send_to_next=None, recv_prev=False):
        """Communicate tensor shapes between stages. Used to communicate
        tensor shapes before the actual tensor communication happens.

        Args:
            tensor_send_next: tensor to send to next rank (no tensor sent if
                              set to None).
            recv_prev: boolean for whether tensor should be received from
                       previous rank.
        """

        ops = []
        if recv_prev:
            recv_prev_dim_tensor = torch.empty(
                (1), device=self.device, dtype=torch.int64
            )
            recv_prev_dim_op = torch.distributed.P2POp(
                torch.distributed.irecv,
                recv_prev_dim_tensor,
                self.prev_rank,
                self.device_group,
            )
            ops.append(recv_prev_dim_op)

        if tensor_send_to_next is not None:
            send_next_dim_tensor = torch.tensor(
                tensor_send_to_next.dim(), device=self.device, dtype=torch.int64
            )
            send_next_dim_op = torch.distributed.P2POp(
                torch.distributed.isend,
                send_next_dim_tensor,
                self.next_rank,
                self.device_group,
            )
            ops.append(send_next_dim_op)

        if len(ops) > 0:
            reqs = torch.distributed.batch_isend_irecv(ops)
            for req in reqs:
                req.wait()

        # To protect against race condition when using batch_isend_irecv().
        # should take this out once the bug with batch_isend_irecv is resolved.
        synchronize()

        ops = []
        recv_prev_shape_tensor = None
        if recv_prev:
            recv_prev_shape_tensor = torch.empty(
                torch.Size(recv_prev_dim_tensor),
                device=self.device,
                dtype=torch.int64,
            )
            recv_prev_shape_op = torch.distributed.P2POp(
                torch.distributed.irecv,
                recv_prev_shape_tensor,
                self.prev_rank,
                self.device_group,
            )
            ops.append(recv_prev_shape_op)

        if tensor_send_to_next is not None:
            send_next_shape_tensor = torch.tensor(
                tensor_send_to_next.size(),
                device=self.device,
                dtype=torch.int64,
            )
            send_next_shape_op = torch.distributed.P2POp(
                torch.distributed.isend,
                send_next_shape_tensor,
                self.next_rank,
                self.device_group,
            )
            ops.append(send_next_shape_op)

        if len(ops) > 0:
            reqs = torch.distributed.batch_isend_irecv(ops)
            for req in reqs:
                req.wait()

        synchronize()

        recv_prev_shape = [0, 0, 0]
        if recv_prev_shape_tensor is not None:
            recv_prev_shape = recv_prev_shape_tensor
        return torch.Size(recv_prev_shape)
```
**EN:** This block defines method `_communicate_shapes` on `PipelineGroupCoordinator`. Communicate tensor shapes between stages. Used to communicate tensor shapes before the actual tensor communication happens. Key calls include `synchronize`, `torch.Size`, `torch.empty`, `torch.distributed.P2POp`, and `ops.append`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `tensor_send_to_next`, and `recv_prev` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `_communicate_shapes`。 它用于处理 communicate shapes 相关逻辑。 关键调用包括 `synchronize`、`torch.Size`、`torch.empty`、`torch.distributed.P2POp` 和 `ops.append`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `tensor_send_to_next` 和 `recv_prev` 等参数驱动。

### Lines 1074-1081: `pipeline_send` implementation / `pipeline_send` 实现
```python
    def pipeline_send(
        self, tensor: torch.Tensor, name: str = "latent", segment_idx: int = -1
    ) -> None:
        tensor = tensor.contiguous()
        self._check_shape_and_buffer(
            tensor_send_to_next=tensor, name=name, segment_idx=segment_idx
        )
        self._pipeline_isend(tensor).wait()
```
**EN:** This block defines method `pipeline_send` on `PipelineGroupCoordinator`. It handles pipeline send logic. Key calls include `tensor.contiguous`, `self._check_shape_and_buffer`, `self._pipeline_isend.wait`, and `self._pipeline_isend`. Parameters such as `tensor`, `name`, and `segment_idx` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `pipeline_send`。 它用于处理 pipeline send 相关逻辑。 关键调用包括 `tensor.contiguous`、`self._check_shape_and_buffer`、`self._pipeline_isend.wait` 和 `self._pipeline_isend`。 本段逻辑主要由 `tensor`、`name` 和 `segment_idx` 等参数驱动。

### Lines 1083-1090: `pipeline_isend` implementation / `pipeline_isend` 实现
```python
    def pipeline_isend(
        self, tensor: torch.Tensor, name: str = "latent", segment_idx: int = -1
    ) -> None:
        tensor = tensor.contiguous()
        self._check_shape_and_buffer(
            tensor_send_to_next=tensor, name=name, segment_idx=segment_idx
        )
        self._pipeline_isend(tensor)
```
**EN:** This block defines method `pipeline_isend` on `PipelineGroupCoordinator`. It handles pipeline isend logic. Key calls include `tensor.contiguous`, `self._check_shape_and_buffer`, and `self._pipeline_isend`. Parameters such as `tensor`, `name`, and `segment_idx` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `pipeline_isend`。 它用于处理 pipeline isend 相关逻辑。 关键调用包括 `tensor.contiguous`、`self._check_shape_and_buffer` 和 `self._pipeline_isend`。 本段逻辑主要由 `tensor`、`name` 和 `segment_idx` 等参数驱动。

### Lines 1092-1096: `pipeline_recv` implementation / `pipeline_recv` 实现
```python
    def pipeline_recv(self, idx: int = -1, name: str = "latent") -> torch.Tensor:
        name = name or "latent"
        self._check_shape_and_buffer(recv_prev=True, name=name, segment_idx=idx)
        self._pipeline_irecv(self.recv_buffer[name][idx]).wait()
        return self.recv_buffer[name][idx]
```
**EN:** This block defines method `pipeline_recv` on `PipelineGroupCoordinator`. It handles pipeline recv logic. Key calls include `self._check_shape_and_buffer`, `self._pipeline_irecv.wait`, and `self._pipeline_irecv`. Parameters such as `idx`, and `name` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `pipeline_recv`。 它用于处理 pipeline recv 相关逻辑。 关键调用包括 `self._check_shape_and_buffer`、`self._pipeline_irecv.wait` 和 `self._pipeline_irecv`。 本段逻辑主要由 `idx` 和 `name` 等参数驱动。

### Lines 1098-1100: `add_pipeline_recv_task` implementation / `add_pipeline_recv_task` 实现
```python
    def add_pipeline_recv_task(self, idx: int = -1, name: str = "latent"):
        name = name or "latent"
        self.recv_tasks_queue.append((name, idx))
```
**EN:** This block defines method `add_pipeline_recv_task` on `PipelineGroupCoordinator`. It handles add pipeline recv task logic. Key calls include `self.recv_tasks_queue.append`. Parameters such as `idx`, and `name` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `add_pipeline_recv_task`。 它用于处理 add pipeline recv task 相关逻辑。 关键调用包括 `self.recv_tasks_queue.append`。 本段逻辑主要由 `idx` 和 `name` 等参数驱动。

### Lines 1102-1110: `recv_next` implementation / `recv_next` 实现
```python
    def recv_next(self):
        if len(self.recv_tasks_queue) == 0:
            raise ValueError("No more tasks to receive")
        elif len(self.recv_tasks_queue) > 0:
            name, idx = self.recv_tasks_queue.pop(0)
            self._check_shape_and_buffer(recv_prev=True, name=name, segment_idx=idx)
            self.receiving_tasks.append(
                (self._pipeline_irecv(self.recv_buffer[name][idx]), name, idx)
            )
```
**EN:** This block defines method `recv_next` on `PipelineGroupCoordinator`. It handles recv next logic. Key calls include `len`, `ValueError`, `self.recv_tasks_queue.pop`, `self._check_shape_and_buffer`, and `self.receiving_tasks.append`. The implementation branches on conditions.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `recv_next`。 它用于处理 recv next 相关逻辑。 关键调用包括 `len`、`ValueError`、`self.recv_tasks_queue.pop`、`self._check_shape_and_buffer` 和 `self.receiving_tasks.append`。 实现中包含条件分支。

### Lines 1112-1123: `get_pipeline_recv_data` implementation / `get_pipeline_recv_data` 实现
```python
    def get_pipeline_recv_data(
        self, idx: int = -1, name: str = "latent"
    ) -> torch.Tensor:
        assert (
            len(self.receiving_tasks) > 0
        ), "No tasks to receive, call add_pipeline_recv_task first"
        receiving_task = self.receiving_tasks.pop(0)
        receiving_task[0].wait()
        assert (
            receiving_task[1] == name and receiving_task[2] == idx
        ), "Received tensor does not match the requested"
        return self.recv_buffer[name][idx]
```
**EN:** This block defines method `get_pipeline_recv_data` on `PipelineGroupCoordinator`. It retrieves pipeline recv data. Key calls include `self.receiving_tasks.pop`, `receiving_task.wait`, and `len`. Parameters such as `idx`, and `name` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `get_pipeline_recv_data`。 它用于获取pipeline recv data。 关键调用包括 `self.receiving_tasks.pop`、`receiving_task.wait` 和 `len`。 本段逻辑主要由 `idx` 和 `name` 等参数驱动。

### Lines 1125-1134: `_pipeline_irecv` implementation / `_pipeline_irecv` 实现
```python
    def _pipeline_irecv(self, tensor: torch.tensor):
        return torch.distributed.irecv(
            tensor,
            src=self.prev_rank,
            group=(
                self.device_groups[(self.rank_in_group + 1) % 2]
                if self.world_size == 2
                else self.device_group
            ),
        )
```
**EN:** This block defines method `_pipeline_irecv` on `PipelineGroupCoordinator`. It handles pipeline irecv logic. Key calls include `torch.distributed.irecv`. Parameters such as `tensor` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `_pipeline_irecv`。 它用于处理 pipeline irecv 相关逻辑。 关键调用包括 `torch.distributed.irecv`。 本段逻辑主要由 `tensor` 等参数驱动。

### Lines 1136-1145: `_pipeline_isend` implementation / `_pipeline_isend` 实现
```python
    def _pipeline_isend(self, tensor: torch.tensor):
        return torch.distributed.isend(
            tensor,
            dst=self.next_rank,
            group=(
                self.device_groups[self.rank_in_group % 2]
                if self.world_size == 2
                else self.device_group
            ),
        )
```
**EN:** This block defines method `_pipeline_isend` on `PipelineGroupCoordinator`. It handles pipeline isend logic. Key calls include `torch.distributed.isend`. Parameters such as `tensor` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `_pipeline_isend`。 它用于处理 pipeline isend 相关逻辑。 关键调用包括 `torch.distributed.isend`。 本段逻辑主要由 `tensor` 等参数驱动。

### Lines 1147-1159: `set_skip_tensor_recv_buffer` implementation / `set_skip_tensor_recv_buffer` 实现
```python
    def set_skip_tensor_recv_buffer(
        self,
        patches_shape_list: List[List[int]],
        feature_map_shape: List[int],
    ):
        self.skip_tensor_recv_buffer = [
            torch.zeros(*shape, dtype=self.dtype, device=self.device)
            for shape in patches_shape_list
        ]
        self.skip_tensor_recv_buffer.append(
            torch.zeros(*feature_map_shape, dtype=self.dtype, device=self.device)
        )
        self.skip_tensor_recv_buffer_set = True
```
**EN:** This block defines method `set_skip_tensor_recv_buffer` on `PipelineGroupCoordinator`. It configures skip tensor recv buffer. Key calls include `self.skip_tensor_recv_buffer.append`, and `torch.zeros`. Parameters such as `patches_shape_list`, and `feature_map_shape` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `set_skip_tensor_recv_buffer`。 它用于设置skip tensor recv buffer。 关键调用包括 `self.skip_tensor_recv_buffer.append` 和 `torch.zeros`。 本段逻辑主要由 `patches_shape_list` 和 `feature_map_shape` 等参数驱动。

### Lines 1161-1163: `pipeline_send_skip` implementation / `pipeline_send_skip` 实现
```python
    def pipeline_send_skip(self, tensor: torch.Tensor) -> None:
        tensor = tensor.contiguous()
        self._pipeline_isend_skip(tensor).wait()
```
**EN:** This block defines method `pipeline_send_skip` on `PipelineGroupCoordinator`. It handles pipeline send skip logic. Key calls include `tensor.contiguous`, `self._pipeline_isend_skip.wait`, and `self._pipeline_isend_skip`. Parameters such as `tensor` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `pipeline_send_skip`。 它用于处理 pipeline send skip 相关逻辑。 关键调用包括 `tensor.contiguous`、`self._pipeline_isend_skip.wait` 和 `self._pipeline_isend_skip`。 本段逻辑主要由 `tensor` 等参数驱动。

### Lines 1165-1167: `pipeline_isend_skip` implementation / `pipeline_isend_skip` 实现
```python
    def pipeline_isend_skip(self, tensor: torch.Tensor) -> None:
        tensor = tensor.contiguous()
        self._pipeline_isend_skip(tensor)
```
**EN:** This block defines method `pipeline_isend_skip` on `PipelineGroupCoordinator`. It handles pipeline isend skip logic. Key calls include `tensor.contiguous`, and `self._pipeline_isend_skip`. Parameters such as `tensor` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `pipeline_isend_skip`。 它用于处理 pipeline isend skip 相关逻辑。 关键调用包括 `tensor.contiguous` 和 `self._pipeline_isend_skip`。 本段逻辑主要由 `tensor` 等参数驱动。

### Lines 1169-1171: `pipeline_recv_skip` implementation / `pipeline_recv_skip` 实现
```python
    def pipeline_recv_skip(self, idx: int = -1) -> torch.Tensor:
        self._pipeline_irecv_skip(self.skip_tensor_recv_buffer[idx]).wait()
        return self.skip_tensor_recv_buffer[idx]
```
**EN:** This block defines method `pipeline_recv_skip` on `PipelineGroupCoordinator`. It handles pipeline recv skip logic. Key calls include `self._pipeline_irecv_skip.wait`, and `self._pipeline_irecv_skip`. Parameters such as `idx` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `pipeline_recv_skip`。 它用于处理 pipeline recv skip 相关逻辑。 关键调用包括 `self._pipeline_irecv_skip.wait` 和 `self._pipeline_irecv_skip`。 本段逻辑主要由 `idx` 等参数驱动。

### Lines 1173-1174: `add_pipeline_recv_skip_task` implementation / `add_pipeline_recv_skip_task` 实现
```python
    def add_pipeline_recv_skip_task(self, idx: int = -1):
        self.recv_skip_tasks_queue.append(idx)
```
**EN:** This block defines method `add_pipeline_recv_skip_task` on `PipelineGroupCoordinator`. It handles add pipeline recv skip task logic. Key calls include `self.recv_skip_tasks_queue.append`. Parameters such as `idx` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `add_pipeline_recv_skip_task`。 它用于处理 add pipeline recv skip task 相关逻辑。 关键调用包括 `self.recv_skip_tasks_queue.append`。 本段逻辑主要由 `idx` 等参数驱动。

### Lines 1176-1185: `get_pipeline_recv_skip_data` implementation / `get_pipeline_recv_skip_data` 实现
```python
    def get_pipeline_recv_skip_data(self, idx: int = -1) -> torch.Tensor:
        assert (
            len(self.receiving_skip_tasks) > 0
        ), "No tasks to receive, call add_pipeline_recv_skip_task first"
        receiving_skip_task = self.receiving_skip_tasks.pop(0)
        receiving_skip_task[0].wait()
        assert (
            receiving_skip_task[2] == idx
        ), "Received tensor does not match the requested"
        return self.skip_tensor_recv_buffer[idx]
```
**EN:** This block defines method `get_pipeline_recv_skip_data` on `PipelineGroupCoordinator`. It retrieves pipeline recv skip data. Key calls include `self.receiving_skip_tasks.pop`, `receiving_skip_task.wait`, and `len`. Parameters such as `idx` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `get_pipeline_recv_skip_data`。 它用于获取pipeline recv skip data。 关键调用包括 `self.receiving_skip_tasks.pop`、`receiving_skip_task.wait` 和 `len`。 本段逻辑主要由 `idx` 等参数驱动。

### Lines 1187-1199: `recv_skip_next` implementation / `recv_skip_next` 实现
```python
    def recv_skip_next(self):
        if len(self.recv_skip_tasks_queue) == 0:
            raise ValueError("No more tasks to receive")
        elif len(self.recv_skip_tasks_queue) > 0:
            task = self.recv_skip_tasks_queue.pop(0)
            idx = task
            self.receiving_skip_tasks.append(
                (
                    self._pipeline_irecv_skip(self.skip_tensor_recv_buffer[idx]),
                    None,
                    idx,
                )
            )
```
**EN:** This block defines method `recv_skip_next` on `PipelineGroupCoordinator`. It handles recv skip next logic. Key calls include `len`, `ValueError`, `self.recv_skip_tasks_queue.pop`, `self.receiving_skip_tasks.append`, and `self._pipeline_irecv_skip`. The implementation branches on conditions.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `recv_skip_next`。 它用于处理 recv skip next 相关逻辑。 关键调用包括 `len`、`ValueError`、`self.recv_skip_tasks_queue.pop`、`self.receiving_skip_tasks.append` 和 `self._pipeline_irecv_skip`。 实现中包含条件分支。

### Lines 1201-1204: `_pipeline_irecv_skip` implementation / `_pipeline_irecv_skip` 实现
```python
    def _pipeline_irecv_skip(self, tensor: torch.tensor):
        return torch.distributed.irecv(
            tensor, src=self.skip_rank, group=self.skip_device_group
        )
```
**EN:** This block defines method `_pipeline_irecv_skip` on `PipelineGroupCoordinator`. It handles pipeline irecv skip logic. Key calls include `torch.distributed.irecv`. Parameters such as `tensor` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `_pipeline_irecv_skip`。 它用于处理 pipeline irecv skip 相关逻辑。 关键调用包括 `torch.distributed.irecv`。 本段逻辑主要由 `tensor` 等参数驱动。

### Lines 1206-1209: `_pipeline_isend_skip` implementation / `_pipeline_isend_skip` 实现
```python
    def _pipeline_isend_skip(self, tensor: torch.tensor):
        return torch.distributed.isend(
            tensor, dst=self.skip_rank, group=self.skip_device_group
        )
```
**EN:** This block defines method `_pipeline_isend_skip` on `PipelineGroupCoordinator`. It handles pipeline isend skip logic. Key calls include `torch.distributed.isend`. Parameters such as `tensor` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineGroupCoordinator` 的方法 `_pipeline_isend_skip`。 它用于处理 pipeline isend skip 相关逻辑。 关键调用包括 `torch.distributed.isend`。 本段逻辑主要由 `tensor` 等参数驱动。

### Lines 1212-1212: `SequenceParallelGroupCoordinator` class overview / `SequenceParallelGroupCoordinator` 类概览
```python
class SequenceParallelGroupCoordinator(GroupCoordinator):
```
**EN:** This block defines class `SequenceParallelGroupCoordinator`. It encapsulates sequence parallel group coordinator behavior. It inherits from `GroupCoordinator`.
**CN:** 该代码块定义了类 `SequenceParallelGroupCoordinator`。 它用于封装 sequence parallel group coordinator 相关行为。 它继承自 `GroupCoordinator`。

### Lines 1213-1243: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        group_ranks: List[List[int]],
        local_rank: int,
        torch_distributed_backend: Union[str, Backend],
        group_name: str | None = None,
        **kwargs,
    ):
        super().__init__(
            group_ranks=group_ranks,
            local_rank=local_rank,
            torch_distributed_backend=torch_distributed_backend,
            group_name=group_name,
        )
        ulysses_group = kwargs.get("ulysses_group", None)
        ring_group = kwargs.get("ring_group", None)
        if ulysses_group is None:
            raise RuntimeError(
                f"Please pass argument 'ulysses_group' when calling init func of SequenceParallelGroupCoordinator"
            )
        if ring_group is None:
            raise RuntimeError(
                f"Please pass argument 'ring_group' when calling init func of SequenceParallelGroupCoordinator"
            )
        self.ulysses_group = ulysses_group
        self.ring_group = ring_group

        self.ulysses_world_size = torch.distributed.get_world_size(self.ulysses_group)
        self.ulysses_rank = torch.distributed.get_rank(self.ulysses_group)
        self.ring_world_size = torch.distributed.get_world_size(self.ring_group)
        self.ring_rank = torch.distributed.get_rank(self.ring_group)
```
**EN:** This block defines method `__init__` on `SequenceParallelGroupCoordinator`. It initializes the instance state. Key calls include `super.__init__`, `kwargs.get`, `torch.distributed.get_world_size`, `torch.distributed.get_rank`, and `RuntimeError`. The implementation branches on conditions. Parameters such as `group_ranks`, `local_rank`, `torch_distributed_backend`, and `group_name` drive the behavior in this section.
**CN:** 该代码块定义了 `SequenceParallelGroupCoordinator` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`kwargs.get`、`torch.distributed.get_world_size`、`torch.distributed.get_rank` 和 `RuntimeError`。 实现中包含条件分支。 本段逻辑主要由 `group_ranks`、`local_rank`、`torch_distributed_backend` 和 `group_name` 等参数驱动。

## Key Concepts / 关键概念
- `get_local_torch_device`: Return the torch device for the current rank. / 顶层函数，用于获取local torch device。
- `_get_unique_name`: Get a unique name for the group. / 顶层函数，用于获取unique name。
- `_split_tensor_dict`: Split the tensor dictionary into two parts: 1. / 顶层函数，用于拆分tensor dict。
- `_update_nested_dict`: Top-level function that updates nested dict. / 顶层函数，用于更新nested dict。
- `GraphCaptureContext`: Primary class that encapsulates graph capture context behavior. / 核心类，用于封装 graph capture context 相关行为。
- `GroupCoordinator`: PyTorch ProcessGroup wrapper for a group of processes. / 核心类，用于封装 group coordinator 相关行为。
- `PipelineGroupCoordinator`: available attributes: rank: int # global rank ranks: List[int] # global ranks in the group world_size: int # size of the group difference between `local_rank` and `rank_in_group`: if we have a group of size 4 across two nodes: Process | Node | Rank | Local Rank | Rank in Group 0 | 0 | 0 | 0 | 0 1 | 0 | 1 | 1 | 1 2 | 1 | 2 | 0 | 2 3 | 1 | 3 | 1 | 3 local_rank: int # local rank used to assign devices rank_in_group: int # rank inside the group cpu_group: ProcessGroup # group for CPU communication device_group: ProcessGroup # group for device communication / 核心类，用于封装 pipeline group coordinator 相关行为。
- `SequenceParallelGroupCoordinator`: Primary class that encapsulates sequence parallel group coordinator behavior. / 核心类，用于封装 sequence parallel group coordinator 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `pickle`, `collections`, `contextlib`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`, `torch.distributed`, `torch.cuda`, `torch_musa`, `torch_musa.core.device`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.distributed.device_communicators.base_device_communicator`, `sglang.multimodal_gen.runtime.distributed.device_communicators.cpu_communicator`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.srt.utils`, `sglang.multimodal_gen.runtime.distributed.device_communicators.cuda_communicator`

- **Total lines / 总行数**: 1243
