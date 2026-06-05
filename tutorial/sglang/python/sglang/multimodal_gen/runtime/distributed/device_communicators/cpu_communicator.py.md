# cpu_communicator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/distributed/device_communicators/cpu_communicator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the distributed runtime layer. It centers on `CpuCommunicator`, and `_CPUSHMDistributed`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于分布式运行时层。它围绕 `CpuCommunicator` 和 `_CPUSHMDistributed` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 7-12: module setup and imports / 模块初始化与导入
```python
import os

import torch
from torch.distributed import ProcessGroup

from .base_device_communicator import DeviceCommunicatorBase
```
**EN:** This block establishes the module context and imports `os`, `torch`, `torch.distributed`, and `.base_device_communicator`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `os`、`torch`、`torch.distributed` 和 `.base_device_communicator`。这些依赖为后续实现提供所需符号。

### Lines 15-16: `CpuCommunicator` class overview / `CpuCommunicator` 类概览
```python
class CpuCommunicator(DeviceCommunicatorBase):
```
**EN:** This block defines class `CpuCommunicator`. It encapsulates cpu communicator behavior. It inherits from `DeviceCommunicatorBase`.
**CN:** 该代码块定义了类 `CpuCommunicator`。 它用于封装 cpu communicator 相关行为。 它继承自 `DeviceCommunicatorBase`。

### Lines 17-35: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        cpu_group: ProcessGroup,
        device: torch.device | None = None,
        device_group: ProcessGroup | None = None,
        unique_name: str = "",
    ):
        from sglang.multimodal_gen.runtime.platforms import current_platform
        from sglang.multimodal_gen.runtime.platforms.interface import CpuArchEnum

        super().__init__(cpu_group, device, device_group, unique_name)
        self.dist_module = torch.distributed

        if (
            (current_platform.get_cpu_architecture() == CpuArchEnum.X86)
            and hasattr(torch.ops._C, "init_shm_manager")
            and unique_name.startswith("tp")
        ):
            self.dist_module = _CPUSHMDistributed(self)
```
**EN:** This block defines method `__init__` on `CpuCommunicator`. It initializes the instance state. Key calls include `super.__init__`, `hasattr`, `unique_name.startswith`, `_CPUSHMDistributed`, and `super`. The implementation branches on conditions. Parameters such as `cpu_group`, `device`, `device_group`, and `unique_name` drive the behavior in this section.
**CN:** 该代码块定义了 `CpuCommunicator` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`hasattr`、`unique_name.startswith`、`_CPUSHMDistributed` 和 `super`。 实现中包含条件分支。 本段逻辑主要由 `cpu_group`、`device`、`device_group` 和 `unique_name` 等参数驱动。

### Lines 37-43: `all_reduce` implementation / `all_reduce` 实现
```python
    def all_reduce(
        self,
        input_: torch.Tensor,
        op: torch.distributed.ReduceOp | None = torch.distributed.ReduceOp.SUM,
    ) -> torch.Tensor:
        self.dist_module.all_reduce(input_, group=self.device_group, op=op)
        return input_
```
**EN:** This block defines method `all_reduce` on `CpuCommunicator`. It handles all reduce logic. Key calls include `self.dist_module.all_reduce`. Parameters such as `input_`, and `op` drive the behavior in this section.
**CN:** 该代码块定义了 `CpuCommunicator` 的方法 `all_reduce`。 它用于处理 all reduce 相关逻辑。 关键调用包括 `self.dist_module.all_reduce`。 本段逻辑主要由 `input_` 和 `op` 等参数驱动。

### Lines 45-76: `gather` implementation / `gather` 实现
```python
    def gather(
        self, input_: torch.Tensor, dst: int = 0, dim: int = -1
    ) -> torch.Tensor | None:
        """
        NOTE: We assume that the input tensor is on the same device across
        all the ranks.
        NOTE: `dst` is the local rank of the destination rank.
        """
        world_size = self.world_size
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
        self.dist_module.gather(
            input_, gather_list, dst=self.ranks[dst], group=self.device_group
        )

        if self.rank_in_group == dst:
            output_tensor = torch.cat(gather_list, dim=dim)
        else:
            output_tensor = None
        return output_tensor
```
**EN:** This block defines method `gather` on `CpuCommunicator`. NOTE: We assume that the input tensor is on the same device across all the ranks. NOTE: `dst` is the local rank of the destination rank. Key calls include `self.dist_module.gather`, `input_.dim`, `torch.cat`, `input_.size`, and `torch.empty_like`. The implementation branches on conditions. Parameters such as `input_`, `dst`, and `dim` drive the behavior in this section.
**CN:** 该代码块定义了 `CpuCommunicator` 的方法 `gather`。 它用于处理 gather 相关逻辑。 关键调用包括 `self.dist_module.gather`、`input_.dim`、`torch.cat`、`input_.size` 和 `torch.empty_like`。 实现中包含条件分支。 本段逻辑主要由 `input_`、`dst` 和 `dim` 等参数驱动。

### Lines 78-104: `all_gather` implementation / `all_gather` 实现
```python
    def all_gather(self, input_: torch.Tensor, dim: int = -1) -> torch.Tensor:
        if dim < 0:
            # Convert negative dim to positive.
            dim += input_.dim()
        input_size = input_.size()
        # NOTE: we have to use concat-style all-gather here,
        # stack-style all-gather has compatibility issues with
        # torch.compile . see https://github.com/pytorch/pytorch/issues/138795
        output_size = (input_size[0] * self.world_size,) + input_size[1:]
        # Allocate output tensor.
        output_tensor = torch.empty(
            output_size, dtype=input_.dtype, device=input_.device
        )
        # All-gather.
        self.dist_module.all_gather_into_tensor(
            output_tensor, input_, group=self.device_group
        )

        # Reshape
        output_tensor = output_tensor.reshape((self.world_size,) + input_size)
        output_tensor = output_tensor.movedim(0, dim)
        output_tensor = output_tensor.reshape(
            input_size[:dim]
            + (self.world_size * input_size[dim],)
            + input_size[dim + 1 :]
        )
        return output_tensor
```
**EN:** This block defines method `all_gather` on `CpuCommunicator`. It handles all gather logic. Key calls include `input_.size`, `torch.empty`, `self.dist_module.all_gather_into_tensor`, `output_tensor.reshape`, and `output_tensor.movedim`. The implementation branches on conditions. Parameters such as `input_`, and `dim` drive the behavior in this section.
**CN:** 该代码块定义了 `CpuCommunicator` 的方法 `all_gather`。 它用于处理 all gather 相关逻辑。 关键调用包括 `input_.size`、`torch.empty`、`self.dist_module.all_gather_into_tensor`、`output_tensor.reshape` 和 `output_tensor.movedim`。 实现中包含条件分支。 本段逻辑主要由 `input_` 和 `dim` 等参数驱动。

### Lines 107-108: `_CPUSHMDistributed` class overview / `_CPUSHMDistributed` 类概览
```python
class _CPUSHMDistributed:
```
**EN:** This block defines class `_CPUSHMDistributed`. It encapsulates cpushmdistributed behavior.
**CN:** 该代码块定义了类 `_CPUSHMDistributed`。 它用于封装 cpushmdistributed 相关行为。

### Lines 109-119: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, communicator: CpuCommunicator):
        instance_identifier = os.environ["VLLM_DIST_IDENT"]
        unique_name = communicator.unique_name
        instance_identifier = f"{instance_identifier}-{unique_name}"
        self.communicator = communicator

        group_ranks = [str(rank) for rank in self.communicator.ranks]
        shm_group_identifier = f"[{'-'.join(group_ranks)}]"
        self.group_name = f"{instance_identifier}-{shm_group_identifier}-cpushm"

        self.handle = self._init_cpu_shm()
```
**EN:** This block defines method `__init__` on `_CPUSHMDistributed`. It initializes the instance state. Key calls include `self._init_cpu_shm`, `str`, and `join`. Parameters such as `communicator` drive the behavior in this section.
**CN:** 该代码块定义了 `_CPUSHMDistributed` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `self._init_cpu_shm`、`str` 和 `join`。 本段逻辑主要由 `communicator` 等参数驱动。

### Lines 121-134: `_init_cpu_shm` implementation / `_init_cpu_shm` 实现
```python
    def _init_cpu_shm(self) -> int:
        handle = torch.ops._C.init_shm_manager(
            self.group_name,
            self.communicator.world_size,
            self.communicator.rank,
        )
        torch.distributed.barrier(self.communicator.device_group)
        torch.ops._C.join_shm_manager(
            handle,
            self.group_name,
        )
        torch.distributed.barrier(self.communicator.device_group)

        return int(handle)
```
**EN:** This block defines method `_init_cpu_shm` on `_CPUSHMDistributed`. It initializes cpu shm. Key calls include `torch.ops._C.init_shm_manager`, `torch.distributed.barrier`, `torch.ops._C.join_shm_manager`, and `int`.
**CN:** 该代码块定义了 `_CPUSHMDistributed` 的方法 `_init_cpu_shm`。 它用于初始化cpu shm。 关键调用包括 `torch.ops._C.init_shm_manager`、`torch.distributed.barrier`、`torch.ops._C.join_shm_manager` 和 `int`。

### Lines 136-139: `all_reduce` implementation / `all_reduce` 实现
```python
    def all_reduce(
        self, input: torch.Tensor, group: ProcessGroup | None = None
    ) -> None:
        torch.ops._C.shm_allreduce(self.handle, input)
```
**EN:** This block defines method `all_reduce` on `_CPUSHMDistributed`. It handles all reduce logic. Key calls include `torch.ops._C.shm_allreduce`. Parameters such as `input`, and `group` drive the behavior in this section.
**CN:** 该代码块定义了 `_CPUSHMDistributed` 的方法 `all_reduce`。 它用于处理 all reduce 相关逻辑。 关键调用包括 `torch.ops._C.shm_allreduce`。 本段逻辑主要由 `input` 和 `group` 等参数驱动。

### Lines 141-154: `gather` implementation / `gather` 实现
```python
    def gather(
        self,
        input: torch.Tensor,
        gather_list: list[torch.Tensor] | None,
        dst: int = -1,
        group: ProcessGroup | None = None,
    ) -> None:
        # Note: different from the torch gather, here we use local dst rank.
        torch.ops._C.shm_gather(
            self.handle,
            input,
            gather_list,
            torch.distributed.get_group_rank(group, dst),
        )
```
**EN:** This block defines method `gather` on `_CPUSHMDistributed`. It handles gather logic. Key calls include `torch.ops._C.shm_gather`, and `torch.distributed.get_group_rank`. Parameters such as `input`, `gather_list`, `dst`, and `group` drive the behavior in this section.
**CN:** 该代码块定义了 `_CPUSHMDistributed` 的方法 `gather`。 它用于处理 gather 相关逻辑。 关键调用包括 `torch.ops._C.shm_gather` 和 `torch.distributed.get_group_rank`。 本段逻辑主要由 `input`、`gather_list`、`dst` 和 `group` 等参数驱动。

### Lines 156-162: `all_gather_into_tensor` implementation / `all_gather_into_tensor` 实现
```python
    def all_gather_into_tensor(
        self,
        output: torch.Tensor,
        input: torch.Tensor,
        group: ProcessGroup | None = None,
    ) -> None:
        torch.ops._C.shm_all_gather(self.handle, input, output)
```
**EN:** This block defines method `all_gather_into_tensor` on `_CPUSHMDistributed`. It handles all gather into tensor logic. Key calls include `torch.ops._C.shm_all_gather`. Parameters such as `output`, `input`, and `group` drive the behavior in this section.
**CN:** 该代码块定义了 `_CPUSHMDistributed` 的方法 `all_gather_into_tensor`。 它用于处理 all gather into tensor 相关逻辑。 关键调用包括 `torch.ops._C.shm_all_gather`。 本段逻辑主要由 `output`、`input` 和 `group` 等参数驱动。

## Key Concepts / 关键概念
- `CpuCommunicator`: Primary class that encapsulates cpu communicator behavior. / 核心类，用于封装 cpu communicator 相关行为。
- `_CPUSHMDistributed`: Primary class that encapsulates cpushmdistributed behavior. / 核心类，用于封装 cpushmdistributed 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`
- **Third-party / 第三方依赖**: `torch`, `torch.distributed`
- **Internal modules / 内部模块**: `.base_device_communicator`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.platforms.interface`

- **Total lines / 总行数**: 162
